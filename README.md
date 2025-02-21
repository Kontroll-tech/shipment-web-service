# Documentación API Kila Platform

## Índice
1. [Introducción](#introducción)
2. [Autenticación](#autenticación)
3. [Endpoints](#endpoints)
   - [Login](#login)
   - [Crear Envío con Tracking](#crear-envío-con-tracking)
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
    "carrier": "MAERSK",
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
    "message": "Shipment created successfully"
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
    "carrier": "MAERSK",
    "type": "container"
}'
```
