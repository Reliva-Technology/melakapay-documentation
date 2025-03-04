# CarianPersendirian Function Documentation

## Overview
The `CarianPersendirian()` function is part of the `AccountCheckAPI` class and is responsible for handling requests related to "Carian Persendirian" (Private Search) in the MelakaPay system. This function interfaces with external SOAP APIs to retrieve land title information based on various search criteria.

## Function Signature
```php
public function CarianPersendirian()
```

## Input Parameters
The function accepts input via HTTP request parameters:

| Parameter | Required | Description |
|-----------|----------|-------------|
| agency_id | Yes | The ID of the agency to query |
| account_no | Yes* | The account number or title ID to search for (*required for acc_number/id_hakmilik type) |
| acc_type | No | The type of search to perform (defaults to 'acc_number') |
| idHakmilik | No | Alternative to account_no for some search types |

### Search Types (acc_type)
The function supports several search types:

1. **acc_number/id_hakmilik** - Search by title ID
2. **id_hakmilik_landed** - Search by title number with land details
3. **id_hakmilik_strata** - Search by title number with strata details
4. **id_lot_landed** - Search by lot number with land details
5. **id_lot_strata** - Search by lot number with strata details

### Additional Parameters Based on Search Type

#### For id_hakmilik_landed
- idHakmilik (title ID)
- daerah (district)
- mukim (subdistrict)
- seksyen (section, optional)
- jenisHakmilik (title type)

#### For id_hakmilik_strata
- idHakmilik (title ID)
- daerah (district)
- mukim (subdistrict)
- seksyen (section, optional)
- jenisHakmilik (title type)
- noBangunan (building number)
- noTingkat (floor number)
- noPetak (parcel number)

#### For id_lot_landed
- daerah (district)
- mukim (subdistrict)
- seksyen (section, optional)
- jenisLot (lot type)
- noLot (lot number)

#### For id_lot_strata
- daerah (district)
- mukim (subdistrict)
- seksyen (section, optional)
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
   - Based on the search type (`acc_type`), calls one of five SOAP methods:
     - `carianHakmilik`
     - `carianHakmilikByNoHakmilik`
     - `carianHakmilikByNoHakmilik` (for strata)
     - `carianHakmilikByNoLot`
     - `carianHakmilikStrataByNoLot`
   - Handles exceptions and logs errors

5. **API Request Logging**
   - Logs the API request details in the `ApiLog` table

6. **Response Processing**
   - Processes the API response
   - Maps branch codes to merchant codes and branch names
   - Formats the response data

7. **Response Generation**
   - Returns a formatted response with payment details and search information
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
    "details": {
      // Formatted search details
    },
    "payment": {
      "payment": 1,
      "amount": "30.00",
      "merchant_code": "merchant-code",
      "account_no": "XXXXX",
      "holder_name": "Owner Name"
    }
  },
  "message": "Carian Persendirian from [Agency Name] retrieved successfully"
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

## Payment Details
- The service has a fixed fee of RM 30.00
- Payment is always required (payment flag is set to 1)

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
- The function contains duplicate code for checking if the agency is enabled (can be refactored)
- Error messages direct users to try again later when API calls fail
