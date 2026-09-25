# API REST

La API debe seguir un contrato versionado, por ejemplo `/api/v1`.

## Salud

### GET `/api/v1/health`

```json
{"status":"ok"}
```

## Clientes

### POST `/api/v1/customers`

```json
{
  "name": "Cliente Demo",
  "phone": "9990000000",
  "email": "cliente@example.com",
  "address": "Dirección"
}
```

### GET `/api/v1/customers/{id}`

Devuelve datos comerciales sin secretos.

## Planes

### POST `/api/v1/plans`

```json
{
  "name": "Fibra 100",
  "download_mbps": 100,
  "upload_mbps": 20,
  "monthly_price": 499.00
}
```

### GET `/api/v1/plans`

Lista planes activos/inactivos según filtros.

## Servicios

### POST `/api/v1/services`

```json
{
  "customer_id": 1,
  "plan_id": 2,
  "access_method": "PPPOE",
  "username": "cliente001"
}
```

### POST `/api/v1/services/{id}/suspend`
### POST `/api/v1/services/{id}/activate`

## HotSpot

### POST `/api/v1/hotspot/trials`

```json
{
  "customer_id": 1,
  "profile": "trial-1h",
  "duration_minutes": 60
}
```

### GET `/api/v1/hotspot/sessions/{id}`
### POST `/api/v1/hotspot/trials/{id}/revoke`
### POST `/api/v1/hotspot/trials/{id}/convert-to-pppoe`

La conversión devuelve la nueva referencia de servicio y conserva el ID de la prueba original.

## Provisioning

### GET `/api/v1/provisioning/jobs/{id}`

### POST `/api/v1/provisioning/jobs/{id}/retry`

## MikroTik

### GET `/api/v1/network/devices`
### GET `/api/v1/network/devices/{id}/health`
### POST `/api/v1/network/devices/{id}/reconcile`

## Errores

Formato común:

```json
{
  "error": {
    "code": "SERVICE_NOT_FOUND",
    "message": "Servicio no encontrado",
    "request_id": "01J..."
  }
}
```

Códigos HTTP previstos:

- `400` validación;
- `401` no autenticado;
- `403` sin permiso;
- `404` recurso inexistente;
- `409` conflicto de estado/identidad;
- `422` payload inválido;
- `429` límite;
- `500` error interno;
- `502/503` dependencia de red no disponible.

## Compatibilidad con API actual

Los endpoints actuales `/planes`, `/servicios`, `/provision/cliente` y `/mikrotik/clientes` son funcionales y deben migrarse gradualmente al contrato versionado. No romperlos durante una migración sin una etapa de compatibilidad.
