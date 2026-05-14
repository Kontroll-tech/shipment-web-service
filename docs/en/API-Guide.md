# Kila Platform API Guide - English

This guide provides comprehensive documentation for the Kila Platform API, including authentication, endpoints, and usage examples.

## Quick Start

### Authentication

All API requests (except login) require a valid JWT token:

```bash
Authorization: Bearer <your-token>
```

### Base URL

```
https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net
```

## Endpoints Overview

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/login` | Authenticate user and get token |
| POST | `/createShipmentWithTracking` | Create a new tracked shipment |
| GET | `/getShipmentById` | Retrieve shipment details |

## Detailed Endpoint Documentation

See [README.md](../../README.md) for complete endpoint documentation in English.

## Response Format

All responses follow this format:

### Success Response
```json
{
    "success": true,
    "data": {
        // Response data here
    }
}
```

### Error Response
```json
{
    "success": false,
    "error": {
        "code": 400,
        "message": "ERROR_CODE",
        "details": "Detailed error message"
    }
}
```

## Common HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK - Request successful |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Missing/invalid token |
| 403 | Forbidden - Token invalid or expired |
| 404 | Not Found - Resource not found |
| 405 | Method Not Allowed - Wrong HTTP method |

## Rate Limiting

Currently, no rate limits are enforced. However, it's recommended to implement reasonable request throttling in your application.

## Support

For issues or questions, please contact the development team or refer to the main documentation.
