# ADR-0004: Provisionamiento asíncrono

- Estado: Propuesto
- Fecha: 2026-09-24

## Contexto

Crear un servicio puede implicar PostgreSQL y uno o más equipos de red. Una llamada HTTP no debe permanecer bloqueada esperando toda la cadena.

## Decisión

Persistir el estado deseado y un evento Outbox dentro de una transacción. Workers ejecutan jobs idempotentes contra providers.

## Consecuencias

La API puede responder con un job pendiente. Se requiere observabilidad, reintentos, leases y reconciliación.
