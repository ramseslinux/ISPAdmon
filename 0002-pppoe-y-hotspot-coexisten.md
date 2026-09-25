# ADR-0002: PPPoE y HotSpot son métodos de acceso distintos

- Estado: Aceptado
- Fecha: 2026-09-24

## Contexto

El ISP necesita administrar clientes permanentes mediante PPPoE y ofrecer pruebas/venta temporal mediante HotSpot sobre VLAN 666.

## Decisión

Ambos métodos pertenecen al mismo dominio de acceso, pero se modelan por separado. Un cliente puede tener historial de prueba HotSpot y posteriormente un servicio PPPoE.

## Consecuencias

La conversión no elimina la prueba. Se conserva el historial y se crea/activa el nuevo servicio PPPoE.
