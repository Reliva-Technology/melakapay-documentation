# CarianPersendirian (Update) Function Documentation

## Overview
The `CarianPersendirian()` function in the `AccountUpdateAPI` class is responsible for updating payment information for "Carian Persendirian" (Private Search) transactions in the MelakaPay system. This function interfaces with the eTanah SOAP API to update payment records after a successful transaction and also updates local records.

## Function Signature
```php
public function CarianPersendirian($input)
```

## Input Parameters
The function accepts an array of transaction data:

| Parameter | Required | Description |
|-----------|----------|-------------|
| acc_number | Yes | The account number associated with the private search |
| amount | Yes | The payment amount |
| transaction_id | Yes | The MelakaPay transaction ID |
| payment_datetime | Yes | The date and time of payment |
| full_name | Yes | The name of the person making the payment |
| receipt_no | Yes | The receipt number |
| user_id | Yes | The user ID associated with the search |
| payment_type | Yes | The type of payment (used for logging) |

## Process Flow

1. **API Endpoint Determination**
   - Determines the SOAP API endpoint URL based on the application environment
   - Uses different URLs for production and non-production environments

2. **Data Preparation**
   - Prepares the data for the SOAP API call, including:
     - Account number (accountNo)
     - Payment amount (amaun)
     - Transaction ID (transId)
     - Payment date and time
     - Payer name (namaPenyerah)
     - Receipt number
     - User ID (idPengguna)

3. **SOAP API Call**
   - Calls the `updateBayaranCarian` SOAP method with the prepared data
   - Handles exceptions and logs errors
   - Captures exceptions with Sentry if available

4. **Transaction Update**
   - If the API call is successful (return value is 'success'):
     - Updates the transaction status in the EPS database
     - Updates the local CarianPersendirian records by calling the `updateCarianPersendirian` method
     - Logs the successful update
   - If the API call fails:
     - Logs the failure

5. **Transaction Logging**
   - Records the update attempt in the `UpdateTransaction` table
   - Stores both the request and response data

## Response Handling
The function does not return any value. Instead, it:

1. Updates the EPS transaction status if successful
2. Updates local CarianPersendirian records if successful
3. Logs the result (success or failure)
4. Records the transaction update attempt in the database

## API Endpoints
The function uses different SOAP API endpoints based on the application environment:

| Environment | Endpoint URL |
|-------------|-------------|
| Production | https://etanah.melaka.gov.my/etanahwsa/CarianPersendirianService?wsdl |
| Non-production | https://etanah.melaka.gov.my/etanahwsastg/CarianPersendirianService?wsdl |

## Error Handling
- Catches and logs SOAP faults and other exceptions
- Handles specific Java NullPointerException errors
- Uses Sentry for exception tracking if available
- Logs detailed error information for troubleshooting

## Database Operations
The function performs three database operations:

1. **EPS Database Update**
   ```php
   DB::connection('eps')->table('eps_transactions')
      ->where('merchant_trans_id', $input['transaction_id'])
      ->update(['update_status' => 1]);
   ```

2. **Local CarianPersendirian Update**
   ```php
   $this->updateCarianPersendirian($input['acc_number'], $input['user_id']);
   ```

3. **Transaction Log Creation**
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
- Internal `updateCarianPersendirian` method for updating local records

## Notes for Developers
- The function uses SOAP 1.1 for API communication
- API timeout is set to 15 seconds
- The function logs both successful and failed updates
- Unlike the CukaiTanah and CukaiPetak update functions, this function:
  - Uses a different SOAP method (`updateBayaranCarian`)
  - Includes the payer's name in the API call
  - Updates local records in addition to the external system
  - Uses 'success' as the success indicator (vs. checking for 'fail')
  - Includes the transaction ID in log messages
- The function calls an internal method `updateCarianPersendirian` which is not shown in this snippet but is responsible for updating local records
- This function is part of the payment reconciliation process, ensuring that both the eTanah system and local MelakaPay records are updated after a successful private search payment
