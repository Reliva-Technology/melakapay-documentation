# CukaiPetak Function Documentation

## Overview
The `CukaiPetak()` function is part of the `AccountCheckAPI` class and is responsible for handling requests related to "Cukai Petak" in the MelakaPay system. This function interfaces with external SOAP APIs to retrieve parcel tax information based on various search criteria.

## Function Signature
```php
public function CukaiPetak()
```

## Input Parameters
The function accepts input via HTTP request parameters:

| Parameter | Required | Description |
|-----------|----------|-------------|
| account_no | Yes | The account number or title ID to search for |
| agency_id | Yes | The ID of the agency to query |
| acc_type | Yes | The type of search to perform (acc_number, id_hakmilik, id_hakmilik_strata, id_lot_strata) |
| seksyen | No | Section information (optional) |

### Additional Parameters Based on Search Type
Depending on the `acc_type` value, additional parameters may be required:

#### For id_hakmilik_strata
- id_hakmilik
- daerah (district)
- mukim (subdistrict)
- jenisHakmilik (title type)
- noBangunan (building number)
- noTingkat (floor number)
- noPetak (parcel number)

#### For id_lot_strata
- daerah (district)
- mukim (subdistrict)
- jenisLot (lot type)
- noLot (lot number)
- noBangunan (building number)
- noTingkat (floor number)
- noPetak (parcel number)

## Process Flow

1. **Input Validation**
   - Validates that required parameters are provided
   - Returns appropriate error messages if validation fails

2. **Agency Verification**
   - Checks if the specified agency is enabled
   - Returns an error if the service is temporarily closed

3. **API Service Lookup**
   - Retrieves the API service details for the specified agency
   - Constructs the SOAP API endpoint URL

4. **SOAP API Call**
   - Based on the search type (`acc_type`), calls one of three SOAP methods:
     - `checkAccountIdStrata`
     - `checkAccountStrataByNoHakmilik`
     - `checkAccountStrataByNoLot`
   - Handles exceptions and logs errors

5. **API Request Logging**
   - Logs the API request details in the `ApiLog` table

6. **Response Processing**
   - Processes the API response
   - Determines payment status
   - Maps branch codes to merchant codes and branch names
   - Formats the response data

7. **Response Generation**
   - Returns a formatted response with payment details and bill information
   - Returns appropriate error messages if no data is found or if there's an API error

## Response Structure
On successful API call, the function returns a JSON response with the following structure:

```json
{
  "success": true,
  "data": {
    "return": {
      // Original API response data
    },
    "api": {
      "kod_cawangan": "XX",
      "cawangan": "Branch Name",
      "merchant_code": "merchant-code"
    },
    "details": {
      // Formatted bill details
    },
    "payment": {
      "payment": 0/1,
      "amount": "XXX.XX",
      "merchant_code": "merchant-code",
      "account_no": "XXXXX",
      "holder_name": "Owner Name",
      "ic_no": "-"
    }
  },
  "message": "Cukai Petak from [Agency Name] retrieved successfully"
}
```

On failure, it returns:
```json
{
  "success": false,
  "data": null,
  "message": "Error message"
}
```

## Branch Code Mapping
The function maps branch codes to merchant codes and branch names:

| Branch Code | Merchant Code | Branch Name |
|-------------|---------------|-------------|
| 00 | ptgnm-app | Pejabat Tanah dan Galian Negeri Melaka |
| 01 | pdtmt-app | Pejabat Tanah dan Galian Melaka Tengah |
| 02 | pdtj-app | Pejabat Tanah dan Galian Jasin |
| 03 | pdtag-app | Pejabat Tanah dan Galian Alor Gajah |

## Error Handling
- Validates input parameters and returns appropriate error messages
- Catches and logs SOAP faults and other exceptions
- Handles specific Java NullPointerException errors
- Returns user-friendly error messages for API failures

## Dependencies
- Laravel Request
- DB facade
- Auth facade
- Carbon for timestamp generation
- SoapClient for SOAP API calls
- ApiLog model for logging API requests

## Notes for Developers
- The function uses SOAP 1.1 for API communication
- API timeout is set to 15 seconds
- All API requests are logged in the `ApiLog` table
- Payment status is determined by checking if `statusBayaran` is 'TELAH BAYAR' or if `jumlahKeseluruhan` is 0
- For error support, users are directed to contact 06-3333333 5016/5017/5052 or email ptg@melaka.gov.my
