# Base de datos

PostgreSQL es la fuente de verdad.

## Modelo lógico

```text
customers 1---N services N---1 plans
                 |
                 N
                 |
          access_credentials
                 |
                 +---N provisioning_jobs
                 |
                 +---N audit_events

network_devices 1---N provisioning_jobs

hotspot_trials N---1 customers
hotspot_trials N---0..1 services   (después de conversión)
hotspot_trials 1---N hotspot_sessions
```

## Tablas principales

### customers

`id`, `name`, `phone`, `email`, `address`, `status`, timestamps.

### plans

`id`, `name`, `download_mbps`, `upload_mbps`, `monthly_price`, `active`, timestamps.

### services

`id`, `customer_id`, `plan_id`, `access_method`, `status`, `username`, `ip_address`, `mac_address`, `installed_at`, timestamps.

### access_credentials

`id`, `service_id`, `method`, `username`, `secret_reference`, `status`, timestamps.

La contraseña no se guarda en texto plano en esta tabla.

### hotspot_trials

`id`, `customer_id`, `profile`, `duration_seconds`, `started_at`, `expires_at`, `status`, `converted_service_id`, timestamps.

### hotspot_sessions

`id`, `trial_id`, `username`, `client_ip`, `client_mac`, `router_id`, `started_at`, `ended_at`, `bytes_in`, `bytes_out`.

### network_devices

`id`, `name`, `vendor`, `type`, `host`, `api_port`, `enabled`, `last_seen_at`, timestamps.

### provisioning_jobs

`id`, `type`, `aggregate_type`, `aggregate_id`, `device_id`, `idempotency_key`, `payload`, `status`, `attempts`, `next_run_at`, `last_error`, timestamps.

### outbox_events

`id`, `event_type`, `aggregate_type`, `aggregate_id`, `payload`, `published_at`, `created_at`.

### audit_events

`id`, `actor_id`, `action`, `resource_type`, `resource_id`, `metadata`, `created_at`.

## Índices

Mínimos:

- `customers(status)`
- `plans(active)`
- `services(customer_id)`
- `services(status)`
- unique sobre identidad de acceso según método/contexto
- `hotspot_trials(expires_at, status)`
- `hotspot_sessions(trial_id, started_at)`
- `provisioning_jobs(status, next_run_at)`
- unique `provisioning_jobs(idempotency_key)`
- `outbox_events(published_at, created_at)`
- `audit_events(resource_type, resource_id, created_at)`

## Migración desde el modelo actual

El código actual usa `clientes`, `planes` y `servicios`. El refactor debe conservar datos y hacer una migración controlada. No cambiar nombres y columnas en la misma etapa que la migración de lógica sin una estrategia de compatibilidad.
