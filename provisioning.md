# Aprovisionamiento

## Problema

Una operación comercial puede requerir varios pasos: PostgreSQL, RouterOS y posteriormente ONU/OLT. No se debe intentar mantener una transacción SQL abierta mientras se realizan llamadas de red.

## Patrón

```text
Command
  |
  v
DB transaction -> persist desired state + outbox event
  |
  v
Worker
  |
  +--> Provider
  |
  +--> success -> mark synchronized
  |
  +--> transient error -> retry
  |
  +--> permanent error -> failed + alert
```

## Job

Campos mínimos:

- `id`
- `type`
- `aggregate_type`
- `aggregate_id`
- `device_id`
- `payload`
- `idempotency_key`
- `status`
- `attempts`
- `next_run_at`
- `last_error`
- `created_at`
- `updated_at`

## Workers

Un worker obtiene jobs `PENDING` o `RETRYING`, los bloquea de forma segura, cambia a `RUNNING`, ejecuta el provider y persiste el resultado.

Debe existir un mecanismo de lease/timeout para recuperar jobs cuyo worker murió.

## Reintentos

Usar backoff progresivo para errores transitorios. Ejemplos:

- timeout de conexión;
- router temporalmente no disponible;
- límite temporal de API.

No reintentar ciegamente errores de validación, credenciales inválidas permanentes o recursos mal configurados.

## Idempotencia

La clave debe derivarse de la intención, por ejemplo:

```text
provision-service:{service_id}:revision:{desired_revision}
```

Una segunda ejecución de la misma revisión debe dejar el mismo estado.

## Outbox

El cambio de estado y el evento pendiente deben guardarse en la misma transacción PostgreSQL. Un worker publica/procesa después.

Esto evita el problema:

```text
COMMIT DB
   X
evento perdido
```

## Reconciliación

Debe existir un job periódico para detectar diferencias entre el estado deseado y RouterOS. La reconciliación no sustituye los jobs de provisión; corrige divergencias.
