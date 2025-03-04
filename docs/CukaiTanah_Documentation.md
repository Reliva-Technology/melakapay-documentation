# CukaiTanah Function Documentation

## Overview
The `CukaiTanah()` function is part of the `AccountCheckAPI` class and is responsible for handling requests related to "Cukai Tanah" (Land Tax) in the MelakaPay system. This function interfaces with external SOAP APIs to retrieve land tax information based on account number or title ID.

## Function Signature
```php
public function CukaiTanah()
```

## Input Parameters
The function accepts input via HTTP request parameters:

| Parameter | Required | Description |
|-----------|----------|-------------|
| account_no | Yes | The account number or title ID to search for |
| user_id | No | User ID for favorite functionality (optional) |

## Process Flow

1. **Input Validation**
   - Validates that the account number is provided
   - Returns an error message if validation fails

2. **Search Type Determination**
   - Determines the search type based on the format of the account number
   - If the account number has less than 11 characters, it's treated as `acc_number`
   - Otherwise, it's treated as `id_hakmilik` (title ID)

3. **Agency Verification**
   - Uses a fixed agency ID (8) for PTGNM (Pejabat Tanah dan Galian Negeri Melaka)
   - Checks if the agency service is enabled
   - Returns an error if the service is temporarily closed

4. **API Service Lookup**
   - Uses a fixed service ID (21)
   - Constructs the SOAP API endpoint URL

5. **SOAP API Call**
   - Based on the search type, calls the `checkAccountKPHM` SOAP method with either:
     - `idHakmilik` parameter (for title ID search)
     - `accountNo` parameter (for account number search)
   - Handles exceptions and logs errors

6. **API Request Logging**
   - Logs the API request details in the `ApiLog` table

7. **Cancelled Title Handling**
   - If the search is by title ID and the result contains multiple titles (indicating a cancelled title)
   - Saves the new titles as favorites for the user
   - Returns a special response with information about the cancelled title

8. **Response Processing**
   - Determines the appropriate merchant code and branch based on:
     - `jenisPegangan` (type of holding)
     - `kodCaw` (branch code)
   - Formats the land tax details including:
     - Account information
     - Land details
     - Payment breakdown
   - Checks for 6A Notice and payment status

9. **Response Generation**
   - Returns a formatted response with payment details and bill information
   - Returns appropriate error messages if no data is found or if there's an API error

## Response Structure
On successful API call, the function returns a JSON response with the following structure:

```json
{
  "success": true,
  "data": {
    "api": {
      "kod_cawangan": "XX",
      "cawangan": "Branch Name",
      "merchant_code": "merchant-code"
    },
    "details": {
      // Formatted bill details
    },
    "description": [
      {
        "Type of Revenue": "Quit Rent",
        "Current (RM)": "XXX.XX",
        "Arrears (RM)": "XXX.XX",
        "Total (RM)": "XXX.XX"
      },
      // Other revenue types...
    ],
    "payment": {
      "payment": 0/1,
      "amount": "XXX.XX",
      "account_no": "XXXXX",
      "merchant_code": "merchant-code",
      "holder_name": "Owner Name",
      "ic_no": "XXXXXX"
    },
    "raw": {
      // Original API response data
    }
  },
  "message": "Cukai Tanah retrieved successfully"
}
```

For cancelled titles, it returns:
```json
{
  "success": true,
  "data": {
    "link": [],
    "favourite": [],
    "batal": {
      "title": "Hakmilik Batal",
      "message": "Hakmilik ini telah batal dan disambungkan ke X hakmilik berikut. Sila bayar mengikut hakmilik ini.<ul>...</ul>"
    }
  },
  "message": "Cukai tanah batal"
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

## Branch and Merchant Code Mapping
The function maps branch codes and holding types to merchant codes and branch names:

### For "Hakmilik Pejabat Tanah dan Galian" (Land and Mine Office Title)
- Merchant Code: ptgnm-app
- Branch Name: Pejabat Tanah dan Galian Negeri Melaka

### For "Hakmilik Pejabat Tanah dan Daerah" (Land and District Office Title)
Based on branch code:
| Branch Code | Merchant Code | Branch Name |
|-------------|---------------|-------------|
| 01 | pdtmt-app | Pejabat Tanah dan Galian Melaka Tengah |
| 02 | pdtj-app | Pejabat Tanah dan Galian Jasin |
| 03 | pdtag-app | Pejabat Tanah dan Galian Alor Gajah |
| default | ptgnm-app | Pejabat Tanah dan Galian Negeri Melaka |

## Payment Breakdown
The function provides a detailed breakdown of the land tax components:

1. **Quit Rent** (Cukai Tanah)
   - Current amount
   - Arrears amount
   - Total amount

2. **Late Penalty** (Denda Lewat)
   - Current amount
   - Arrears amount
   - Total amount

3. **Canal Tax** (Cukai Parit)
   - Current amount
   - Arrears amount
   - Total amount

4. **6A Notice** (Notis 6A)
   - Current amount
   - Arrears amount (always 0)
   - Total amount

## Special Conditions
1. **6A Notice**: If the account has a 6A Notice (notis6a > 0), online payment is disabled and the user is directed to make payment at the counter.
2. **Already Paid**: If the payment status is "TELAH BAYAR", online payment is disabled.
3. **Cancelled Title**: If the title ID has been cancelled and replaced with new titles, the function returns information about the new titles and saves them as favorites for the user.

## Error Handling
- Validates input parameters and returns appropriate error messages
- Catches and logs SOAP faults and other exceptions
- Handles specific Java NullPointerException errors
- Returns user-friendly error messages for API failures

## Favorites Functionality
If a user ID is provided and the search returns multiple titles (indicating a cancelled title):
- The function saves the new titles as favorites for the user
- Deletes the old title from favorites
- Returns information about the new titles

## Dependencies
- Laravel Request
- DB facade
- Auth facade
- Carbon for timestamp generation
- SoapClient for SOAP API calls
- ApiLog model for logging API requests
- Favourite model for managing user favorites

## Notes for Developers
- The function uses SOAP 1.1 for API communication
- API timeout is set to 15 seconds
- All API requests are logged in the `ApiLog` table
- The function uses fixed agency ID (8) and service ID (21)
- The function handles cancelled titles by saving new titles as favorites
- Special handling for 6A Notice accounts (directing users to counter payment)
