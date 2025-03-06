# Documentación API Kila Platform

## Índice
1. [Introducción](#introducción)
2. [Autenticación](#autenticación)
3. [Endpoints](#endpoints)
   - [Login](#login)
   - [Crear Envío con Tracking](#crear-envío-con-tracking)
   - [Obtener Embarque por ID](#obtener-embarque-por-id)
4. [Manejo de Errores](#manejo-de-errores)
5. [Ejemplos](#ejemplos)

## Introducción

Esta API proporciona servicios para la gestión de envíos y tracking en la plataforma Kila. Todos los endpoints devuelven respuestas en formato JSON y utilizan los códigos de estado HTTP estándar.

## Autenticación

La API utiliza autenticación basada en tokens JWT. Para la mayoría de los endpoints, se requiere incluir el token en el header de autorización:

```
Authorization: Bearer <tu-token>
```

## Endpoints

### Login

**Endpoint:** `/login`

**Método:** `POST`

**Descripción:** Autentica a un usuario utilizando email y contraseña.

#### Request

```json
{
    "email": "usuario@ejemplo.com",
    "password": "contraseña123"
}
```

#### Response Exitosa (200 OK)

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### Errores Posibles

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

### Crear Envío con Tracking

**Endpoint:** `/createShipmentWithTracking`

**Método:** `POST`

**Autenticación requerida:** Sí

**Descripción:** Crea un nuevo envío y configura su seguimiento automático. El servicio verifica la autenticación mediante el token Bearer y extrae el companyId y uid del mismo.

#### Headers Requeridos

```
Authorization: Bearer <tu-token>
Content-Type: application/json
```

#### Request

```json
{
    "shipmentNumber": "SHIP123456",
    "carrier": "MAEU",
    "type": "container"
}
```

| Campo | Tipo | Descripción | Requerido |
|-------|------|-------------|-----------|
| shipmentNumber | string | Número de envío, booking o contenedor | Sí |
| carrier | string | Código del transportista | Sí |
| type | string | Tipo de envío (booking, mbl, container) | Sí |

#### Response Exitosa (200 OK)

```json
{
    "success": true,
    "data": {
        "shipmentId": "MwfBkld0pQAXoHvEBP1y"
    }
}
```

#### Errores Posibles

- **400 Bad Request** - Parámetros faltantes
  ```json
  {
      "success": false,
      "error": {
          "code": 400,
          "message": "MISSING_PARAMS"
      }
  }
  ```

- **401 Unauthorized** - Token no proporcionado
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

- **403 Forbidden** - Token inválido
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

### Obtener Embarque por ID

**Endpoint:** `/getShipmentById`

**Método:** `GET`

**Autenticación requerida:** Sí

**Descripción:** Obtiene la información completa de un embarque por su ID, incluyendo datos de tracking, fechas de milestones (ETA, ETD, ATA, ATD), información de la naviera y detalles de contenedores. El servicio verifica la autenticación mediante el token Bearer y extrae el companyId del mismo. Las fechas se devuelven en formato "YYYY-MM-DD HH:MM:SS" en zona horaria UTC-5.

#### Headers Requeridos

```
Authorization: Bearer <tu-token>
```

#### Parámetros de consulta

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| shipmentId | string | ID del embarque a consultar | Sí |

#### Response Exitosa (200 OK)

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
        "flag": "Panamá",
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

#### Errores Posibles

- **400 Bad Request** - Parámetros faltantes
  ```json
  {
      "success": false,
      "error": {
          "code": 400,
          "message": "MISSING_PARAMS",
          "details": "Se requiere el parámetro shipmentId"
      }
  }
  ```

- **401 Unauthorized** - Token no proporcionado
  ```json
  {
      "success": false,
      "error": {
          "code": 401,
          "message": "UNAUTHORIZED",
          "details": "Un token de autenticación es requerido"
      }
  }
  ```

- **403 Forbidden** - Token sin companyId
  ```json
  {
      "success": false,
      "error": {
          "code": 403,
          "message": "FORBIDDEN",
          "details": "El token proporcionado no contiene la información de compañía necesaria"
      }
  }
  ```

- **404 Not Found** - Embarque no encontrado
  ```json
  {
      "success": false,
      "error": {
          "code": 404,
          "message": "SHIPMENT_NOT_FOUND",
          "details": "No se encontró el embarque con el ID proporcionado"
      }
  }
  ```

- **405 Method Not Allowed** - Método no permitido
  ```json
  {
      "success": false,
      "error": {
          "code": 405,
          "message": "METHOD_NOT_ALLOWED",
          "details": "El método HTTP utilizado no está permitido para este endpoint"
      }
  }
  ```

## Manejo de Errores

Todos los errores siguen una estructura consistente:

```json
{
    "success": false,
    "error": {
        "code": number,
        "message": string,
        "details": string,
        "errors": array (opcional)
    }
}
```

## Ejemplos

### Ejemplo de Login

```bash
curl --location 'https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/login' \
--header 'Content-Type: application/json' \
--data-raw '{
    "email": "usuario@ejemplo.com",
    "password": "contraseña123"
}'
```

### Ejemplo de Crear Envío

```bash
curl --location 'https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/createShipmentWithTracking' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIs...' \
--header 'Content-Type: application/json' \
--data-raw '{
    "shipmentNumber": "SHIP123456",
    "carrier": "MAEU",
    "type": "container"
}'
```

### Ejemplo de Obtener Embarque por ID

```bash
curl --location 'https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/getShipmentById/MwfBkld0pQAXoHvEBP1y' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIs...'
```

**Respuesta de ejemplo:**

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
        "flag": "Panamá",
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
            },
    }
}
```
