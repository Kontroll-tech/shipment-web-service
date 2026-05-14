# Guía Rápida API Kila Platform

> **Documentación completa disponible en:**
> 
> - 🇪🇸 [Documentación Completa en Español](../../README.es.md)
> - 🇬🇧 [Complete Documentation in English](../../README.md)

---

## Inicio Rápido

### 1. Autenticación

Primero, obtenga un token JWT:

```bash
curl -X POST https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "usuario@ejemplo.com",
    "password": "contraseña123"
  }'
```

**Respuesta:**
```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### 2. Crear un Envío con Tracking

```bash
curl -X POST https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/createShipmentWithTracking \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "shipmentNumber": "SHIP123456",
    "carrier": "MAEU",
    "type": "container",
    "transportMode": "OCEAN",
    "originPort": "CO",
    "destinationPort": "US"
  }'
```

**Respuesta:**
```json
{
    "success": true,
    "data": {
        "shipmentId": "MwfBkld0pQAXoHvEBP1y"
    }
}
```

### 3. Obtener Detalles del Envío

```bash
curl -X GET "https://us-central1-kontroll-platform-qa-ba160.cloudfunctions.net/getShipmentById?shipmentId=MwfBkld0pQAXoHvEBP1y" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

## Tabla de Referencia Rápida

| Operación | Método | Endpoint | Autenticación |
|-----------|--------|----------|----------------|
| Login | POST | `/login` | No |
| Crear Envío | POST | `/createShipmentWithTracking` | Sí |
| Obtener Envío | GET | `/getShipmentById` | Sí |

## Códigos de Error Comunes

| Código | Significado |
|--------|-------------|
| 400 | Parámetros inválidos o faltantes |
| 401 | Token no proporcionado |
| 403 | Token inválido o expirado |
| 404 | Recurso no encontrado |
| 405 | Método HTTP no permitido |

## Estructuras de Datos Principales

### Parámetros de Envío

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

### Respuesta de Envío

```json
{
    "success": true,
    "data": {
        "id": "abc123",
        "status": "IN_TRANSIT",
        "mbl": "MBLNUMBER123",
        "booking": "BOOK123456",
        "eta": "2025-02-24 00:30:00",
        "etd": "2025-02-12 08:48:00",
        "portOfLoading": "Shanghai",
        "portOfDischarge": "Rotterdam",
        "vessel": "MSC ANNA"
    }
}
```

---

Para documentación completa, consulte los archivos README principal.
