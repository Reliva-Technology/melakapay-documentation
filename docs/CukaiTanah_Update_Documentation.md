# CukaiTanah (Update) Function Documentation

## Overview
The `CukaiTanah()` function in the `AccountUpdateAPI` class is responsible for updating payment information for "Cukai Tanah" (Land Tax) transactions in the MelakaPay system. This function interfaces with the eTanah SOAP API to update payment records after a successful transaction.

## Function Signature
```php
public function CukaiTanah($input)
```

## Input Parameters
The function accepts an array of transaction data:

| Parameter | Required | Description |
|-----------|----------|-------------|
| acc_number | Yes | The account number associated with the land tax |
| amount | Yes | The payment amount |
| eps_trans_id | Yes | The FPX transaction ID |
| transaction_id | Yes | The MelakaPay transaction ID |
| payment_datetime | Yes | The date and time of payment |
| receipt_no | Yes | The receipt number |
| payment_type | Yes | The type of payment (used for logging) |

## Process Flow

1. **API Endpoint Determination**
   - Determines the SOAP API endpoint URL based on the application environment
   - Uses different URLs for production and non-production environments

2. **Data Preparation**
   - Prepares the data for the SOAP API call, including:
     - Account number (accountNo)
     - Formatted payment amount
     - FPX transaction number (fpxNo)
     - MelakaPay transaction number (transactionNo)
     - Payment time
     - Payment type (fixed as 'K')
     - Receipt number
     - Source system (fixed as 'eBayar')

3. **SOAP API Call**
   - Calls the `updateUserAccount` SOAP method with the prepared data
   - Handles exceptions and logs errors
   - Captures exceptions with Sentry if available

4. **Transaction Update**
   - If the API call is successful (return value is not 'fail'):
     - Updates the transaction status in the EPS database
     - Logs the successful update
   - If the API call fails:
     - Logs the failure

5. **Transaction Logging**
   - Records the update attempt in the `UpdateTransaction` table
   - Stores both the request and response data

## Response Handling
The function does not return any value. Instead, it:

1. Updates the EPS transaction status if successful
2. Logs the result (success or failure)
3. Records the transaction update attempt in the database

## API Endpoints
The function uses different SOAP API endpoints based on the application environment:

| Environment | Endpoint URL |
|-------------|-------------|
| Production | https://etanah.melaka.gov.my/etanahwsa/QuitRentAgentService?wsdl |
| Non-production | https://etanah.melaka.gov.my/etanahwsastg/QuitRentAgentService?wsdl |

## Error Handling
- Catches and logs SOAP faults and other exceptions
- Handles specific Java NullPointerException errors
- Uses Sentry for exception tracking if available
- Logs detailed error information for troubleshooting

## Database Operations
The function performs two database operations:

1. **EPS Database Update**
   ```php
   DB::connection('eps')->table('eps_transactions')
      ->where('merchant_trans_id', $input['transaction_id'])
      ->update(['update_status' => 1]);
   ```

2. **Transaction Log Creation**
   ```php
   UpdateTransaction::insert([
       'payment_type' => $input['payment_type'],
       'receipt_no' => $input['receipt_no'],
       'request' => json_encode($input),
       'response' => json_encode($result['return']),
       'created_at' => Carbon::now()->toDateTimeString(),
   ]);
   ```

## Dependencies
- Laravel Log facade
- DB facade
- Carbon for timestamp generation
- SoapClient for SOAP API calls
- UpdateTransaction model for logging transaction updates
- Sentry for exception tracking (optional)
- Application configuration for environment detection

## Notes for Developers
- The function uses SOAP 1.1 for API communication
- API timeout is set to 15 seconds
- The payment type is hardcoded as 'K'
- The source system is hardcoded as 'eBayar'
- The function logs both successful and failed updates
- If the EPS update fails but the API call was successful, it logs that the transaction was updated manually from EPIC as a warning (unlike CukaiPetak which logs this as an error)
- This function is part of the payment reconciliation process, ensuring that the eTanah system is updated after a successful land tax payment in MelakaPay
- The API service name (QuitRentAgentService) indicates that this function is specifically for quit rent (land tax) payments
