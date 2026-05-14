# Kila Platform API Documentation

> **Documentation available in multiple languages:**
>
> - 🇬🇧 [English](README.md) (Main Documentation)
> - 🇪🇸 [Español](README.es.md) (Documentación en español)
>
> **Quick Start Guides:**
> - [English Quick Guide](docs/en/API-Guide.md)
> - [Guía Rápida en Español](docs/es/Guía-API.md)

---

## Table of Contents
1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [Endpoints](#endpoints)
   - [Login](#login)
   - [Create Shipment with Tracking](#create-shipment-with-tracking)
   - [Get Shipment by ID](#get-shipment-by-id)
4. [Error Handling](#error-handling)
5. [Examples](#examples)

## Introduction

This API provides services for managing shipments and tracking on the Kila platform. All endpoints return responses in JSON format and use standard HTTP status codes.

## Authentication

The API uses JWT token-based authentication. For most endpoints, you need to include the token in the authorization header:

```
Authorization: Bearer <your-token>
```

## Endpoints

### Login

**Endpoint:** `/login`

**Method:** `POST`

**Description:** Authenticates a user using email and password.

#### Request

```json
{
    "email": "user@example.com",
    "password": "password123"
}
```

#### Successful Response (200 OK)

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Possible Errors

- **400 Bad Request**
  ```json
  {
      "error": {
          "code": 400,
          "message": "EMAIL_AND_PASSWORD_REQUIRED",
          "errors": [
              {
                  "message": "EMAIL_AND_PASSWORD_REQUIRED",
                  "domain": "global",
                  "reason": "invalid"
              }
          ]
      }
  }
  ```

- **405 Method Not Allowed**
  ```json
  {
      "error": {
          "code": 405,
          "message": "METHOD_NOT_ALLOWED",
          "errors": []
      }
  }
  ```

### Create Shipment with Tracking

**Endpoint:** `/createShipmentWithTracking`

**Method:** `POST`

**Authentication required:** Yes

**Description:** Creates a new shipment and configures automatic tracking. The service verifies authentication using the Bearer token and extracts the companyId and uid from it.

**Validations:**
- **Date Validation:** The system validates that milestone dates (ETA, ETD, ATA, ATD) are not older than 80 days from the current date to ensure data accuracy.
- **Route Validation:** The optional originPort and destinationPort fields allow validation that the found route matches the expected country codes (ISO 2-character format).

#### Required Headers

```
Authorization: Bearer <your-token>
Content-Type: application/json
```

#### Request

```json
{
    "shipmentNumber": "SHIP123456",
    "carrier": "MAEU",
    "type": "container",
    "transportMode": "OCEAN",
    "originPort": "CO",
    "destinationPort": "US"
}
```

| Field | Type | Description | Required |
|-------|------|-------------|----------|
| shipmentNumber | string | Shipment, booking, or container number | Yes |
| carrier | string | Carrier code | No |
| type | string | Shipment type (booking, mbl, container) for OCEAN and mawb for AIR | Yes |
| transportMode | string | Transport mode (AIR, OCEAN) | Yes |
| originPort | string | Origin country code (2-character ISO) - Optional for ocean shipments. Used to validate the route | No |
| destinationPort | string | Destination country code (2-character ISO) - Optional for ocean shipments. Used to validate the route | No |

#### Successful Response (200 OK)

```json
{
    "success": true,
    "data": {
        "shipmentId": "MwfBkld0pQAXoHvEBP1y"
    }
}
```

#### Possible Errors

- **400 Bad Request** - Missing parameters
  ```json
  {
      "success": false,
      "error": {
          "code": 400,
          "message": "MISSING_PARAMS"
      }
  }
  ```

- **400 Bad Request** - Tracking information too old
  ```json
  {
      "success": false,
      "error": {
          "code": 400,
          "message": "OUTDATED_INFORMATION",
          "details": "The tracking information found is too old (more than 80 days). Please verify that the information sent is correct and consider providing the carrier."
      }
  }
  ```

- **401 Unauthorized** - Token not provided
  ```json
  {
      "success": false,
      "error": {
          "code": 401,
          "message": "No valid authentication token provided",
          "details": "A Bearer token is required in the authorization header"
      }
  }
  ```

- **403 Forbidden** - Invalid token
  ```json
  {
      "success": false,
      "error": {
          "code": 403,
          "message": "Invalid authentication token",
          "details": "The provided token is invalid or has expired"
      }
  }
  ```

### Get Shipment by ID

**Endpoint:** `/getShipmentById`

**Method:** `GET`

**Authentication required:** Yes

**Description:** Retrieves complete shipment information by ID, including tracking data, milestone dates (ETA, ETD, ATA, ATD), carrier information, and container details.

#### Required Headers

```
Authorization: Bearer <your-token>
```

#### Query Parameters

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| shipmentId | string | ID of the shipment to query | Yes |

#### Successful Response (200 OK)

```json
{
    "success": true,
    "data": {
        "id": "abc123",
        "status": "IN_TRANSIT",
        "mbl": "MBLNUMBER123",
        "booking": "BOOK123456",
        "carrierCode": "MSK",
        "carrierName": "Maersk Line",
        "createdAt": "2025-02-23 19:30:00",
        "eta": "2025-02-24 00:30:00",
        "etd": "2025-02-12 08:48:00",
        "ata": "2025-02-24 00:30:00",
        "atd": "2025-02-12 08:48:00",
        "portOfLoading": "Shanghai",
        "portOfDischarge": "Rotterdam",
        "flag": "Panama",
        "vessel": "MSC ANNA",
        "voyage": "VOY123",
        "locations": {
            "Origin": {
                "LocationCode": "USNYC",
                "LocationName": "NEW YORK",
                "CountryCode": "USA"
            },
            "Destination": {
                "LocationCode": "COCTG",
                "LocationName": "CARTAGENA",
                "CountryCode": "Colombia"
            }
        },
        "containers": [
            {
                "containerNumber": "CONT123456",
                "containerType": "40HC"
            }
        ],
        "trackingEvents": [
            {
                "TimestampDescription": "Gate out empty",
                "TimestampDateTime": "2025-01-23T14:59:00-05:00",
                "TimestampLocation": {
                    "name": "DETROIT",
                    "state": "Michigan",
                    "country": "USA",
                    "countryCode": "US",
                    "locode": "USDET",
                    "coordinates": {
                        "lat": 42.258,
                        "lng": -83.1225
                    },
                    "timezone": "America/Detroit"
                },
                "source": "carrier",
                "transportMode": "TRUCK",
                "planned": false
            }
        ]
    }
}
```

#### Possible Errors

- **400 Bad Request** - Missing parameters
  ```json
  {
      "success": false,
      "error": {
          "code": 400,
          "message": "MISSING_PARAMS",
          "details": "The shipmentId parameter is required"
      }
  }
  ```

- **401 Unauthorized** - Token not provided
  ```json
  {
      "success": false,
      "error": {
          "code": 401,
          "message": "UNAUTHORIZED",
          "details": "An authentication token is required"
      }
  }
  ```

- **403 Forbidden** - Token missing companyId
  ```json
  {
      "success": false,
      "error": {
          "code": 403,
          "message": "FORBIDDEN",
          "details": "The provided token does not contain the necessary company information"
      }
  }
  ```

- **404 Not Found** - Shipment not found
  ```json
  {
      "success": false,
      "error": {
          "code": 404,
          "message": "SHIPMENT_NOT_FOUND",
          "details": "The shipment with the provided ID was not found"
      }
  }
  ```

- **405 Method Not Allowed** - Method not allowed
  ```json
  {
      "success": false,
      "error": {
          "code": 405,
          "message": "METHOD_NOT_ALLOWED",
          "details": "The HTTP method used is not allowed for this endpoint"
      }
  }
  ```

## Error Handling

All errors follow a consistent structure:

```json
{
    "success": false,
    "error": {
        "code": "number",
        "message": "string",
        "details": "string",
        "errors": "array (optional)"
    }
}
```

## Examples

### Login Example

```bash
curl --location 'https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/login' \
--header 'Content-Type: application/json' \
--data-raw '{
    "email": "user@example.com",
    "password": "password123"
}'
```

### Create Shipment Example

```bash
curl --location 'https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/createShipmentWithTracking' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIs...' \
--header 'Content-Type: application/json' \
--data-raw '{
    "shipmentNumber": "SHIP123456",
    "carrier": "MAEU",
    "type": "container",
    "transportMode": "OCEAN",
    "originPort": "CO",
    "destinationPort": "US"
}'
```

### Get Shipment by ID Example

```bash
curl --location 'https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/getShipmentById/MwfBkld0pQAXoHvEBP1y' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIs...'
```

**Example response (Ocean Shipment):**

```json
{
    "success": true,
    "data": {
        "id": "abc123",
        "status": "IN_TRANSIT",
        "mbl": "MBLNUMBER123",
        "booking": "BOOK123456",
        "carrierCode": "MSK",
        "carrierName": "Maersk Line",
        "createdAt": "2025-02-23 19:30:00",
        "eta": "2025-02-24T00:30:00",
        "etd": "2025-02-12T08:48:00",
        "ata": "2025-02-24T00:30:00",
        "atd": "2025-02-12T08:48:00",
        "portOfLoading": "Shanghai",
        "portOfDischarge": "Rotterdam",
        "flag": "Panama",
        "vessel": "MSC ANNA",
        "voyage": "VOY123",
        "locations": {
            "Origin": {
                "LocationCode": "USNYC",
                "LocationName": "NEW YORK",
                "CountryCode": "USA"
            },
            "Destination": {
                "LocationCode": "COCTG",
                "LocationName": "CARTAGENA",
                "CountryCode": "Colombia"
            }
        },
        "containers": [
            {
                "type": "45G1",
                "number": "HLBU1767477"
            }
        ],
        "trackingEvents": [
            {
                "TimestampDescription": "Gate out empty",
                "TimestampDateTime": "2025-01-23T14:59:00-05:00",
                "TimestampLocation": {
                    "name": "DETROIT",
                    "state": "Michigan",
                    "country": "USA",
                    "countryCode": "US",
                    "locode": "USDET",
                    "coordinates": {
                        "lat": 42.258,
                        "lng": -83.1225
                    },
                    "timezone": "America/Detroit"
                },
                "source": "carrier",
                "transportMode": "TRUCK",
                "planned": false
            }
        ]
    }
}
```

### Get Air Shipment by ID Example

**Example response:**

```json
{
    "success": true,
    "data": {
        "id": "n1IP7703P31ISWDuNchE",
        "status": "BOOKING_REQUEST",
        "createdAt": "2025-04-08 11:55:13",
        "eta": "2025-02-24T00:30:00",
        "etd": "2025-02-12T08:48:00",
        "ata": "2025-02-24T00:30:00",
        "atd": "2025-02-12T08:48:00",
        "locations": {
            "Origin": null,
            "Destination": null
        },
        "containers": [],
        "trackingEvents": [],
        "transportMode": "Air Transport",
        "mawb": "TLLU5154957",
        "hawb": null,
        "carrierCode": "EMC",
        "carrierName": "247 Aviation",
        "airportOfDeparture": null,
        "airportOfArrival": null
    }
}
```

---

**Last Updated:** May 14, 2026
