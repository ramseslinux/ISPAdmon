# ADR-0001: PostgreSQL como fuente de verdad

- Estado: Aceptado
- Fecha: 2026-09-24

## Contexto

El sistema administra clientes y servicios en PostgreSQL y aplica configuración en MikroTik. Si ambos se consideran fuente de verdad, las divergencias son difíciles de resolver.

## Decisión

PostgreSQL es la fuente de verdad del estado deseado y comercial. MikroTik es un sistema externo que ejecuta ese estado.

## Consecuencias

Se requiere provisioning, reconciliación, jobs y auditoría. Un cambio manual en MikroTik puede ser detectado y corregido según la política de reconciliación.
