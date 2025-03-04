# CarianPersendirianDokumen Function Documentation

## Overview
The `CarianPersendirianDokumen()` function is part of the `AccountCheckAPI` class and is responsible for retrieving a list of documents related to personal searches in the MelakaPay system. This function interfaces with the eTanah SOAP API to fetch document information based on a user ID.

## Function Signature
```php
public function CarianPersendirianDokumen()
```

## Input Parameters
The function accepts input via HTTP request parameters:

| Parameter | Required | Description |
|-----------|----------|-------------|
| id_pengguna | Yes | The user ID to search for documents |

## Process Flow

1. **Input Validation**
   - Validates that the user ID is provided
   - Returns an error message if validation fails

2. **API Endpoint Determination**
   - Determines the SOAP API endpoint URL based on the application environment
   - Uses different URLs for production and non-production environments

3. **SOAP API Call**
   - Calls the `senaraiDokumen` SOAP method with the user ID parameter
   - Handles exceptions and logs errors

4. **API Request Logging**
   - Logs the API request details in the `ApiLog` table
   - Uses a fixed agency ID (8)

5. **Response Generation**
   - Returns the API response directly to the client
   - Includes a success message with the user ID

## Response Structure
On successful API call, the function returns a JSON response with the following structure:

```json
{
  "success": true,
  "data": {
    // Original API response data from the senaraiDokumen method
  },
  "message": "Senarai Dokumen for [id_pengguna] retrieved successfully"
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

## API Endpoints
The function uses different SOAP API endpoints based on the application environment:

| Environment | Endpoint URL |
|-------------|-------------|
| Production | https://etanah.melaka.gov.my/etanahwsa/CarianPersendirianService?wsdl |
| Non-production | https://etanah.melaka.gov.my/etanahwsastg/CarianPersendirianService?wsdl |

## Error Handling
- Validates input parameters and returns appropriate error messages
- Catches and logs SOAP faults and other exceptions
- Handles specific Java NullPointerException errors
- Logs all API requests regardless of success or failure

## Dependencies
- Laravel Request
- DB facade
- Auth facade
- Carbon for timestamp generation
- SoapClient for SOAP API calls
- ApiLog model for logging API requests
- Application configuration for environment detection

## Notes for Developers
- The function uses SOAP 1.1 for API communication
- API timeout is set to 15 seconds
- All API requests are logged in the `ApiLog` table with agency ID 8
- The function returns the raw API response without additional processing
- Unlike other API functions, this one does not perform complex data transformation or formatting
- The function depends on the application environment configuration to determine the API endpoint
- This function is likely used to retrieve a list of documents associated with a user's personal searches in the eTanah system
