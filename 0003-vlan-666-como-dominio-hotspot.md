# ADR-0003: VLAN 666 como red común de HotSpot

- Estado: Aceptado
- Fecha: 2026-09-24

## Contexto

Un usuario que inicia una prueba desde una casa/ONU debe poder desplazarse a otras casas/ONUs que también transportan el HotSpot.

## Decisión

VLAN 666 representa el dominio de transporte del acceso temporal. Las ONUs que participan deben transportar esa VLAN de forma compatible. La identidad comercial y el estado de la prueba viven en el sistema ISP, no en una ONU concreta.

## Consecuencias

El diseño de capa 2 debe preservar el dominio de HotSpot entre los puntos de acceso. Si se cambia la topología a un diseño enrutado, debe definirse otro mecanismo explícito para conservar la sesión.
