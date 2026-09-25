# Arquitectura

## Objetivo

Separar el dominio ISP de FastAPI, PostgreSQL y RouterOS. La aplicación debe poder cambiar el mecanismo de acceso o el fabricante de red sin modificar las reglas comerciales.

## Capas

### Domain
Contiene entidades, value objects, estados, reglas, servicios de dominio y eventos. No importa FastAPI, psycopg ni librerías MikroTik.

### Application
Implementa casos de uso mediante commands/queries, DTOs, handlers y puertos. Coordina repositorios y providers.

### Infrastructure
Implementa PostgreSQL, RouterOS API, colas, workers y otros adaptadores.

### Interfaces
Expone REST, autenticación y serialización. Los endpoints llaman casos de uso, no SQL ni RouterOS directamente.

## DDD

Bounded contexts principales:

- **Customer:** identidad y datos del cliente.
- **Catalog:** planes y políticas comerciales.
- **Access:** métodos de acceso PPPoE y HotSpot.
- **Network:** routers, ONUs/OLTs y recursos de red.
- **Provisioning:** ejecución de cambios y reconciliación.
- **Billing/Time Access:** consumo temporal y futuras ventas de tiempo.
- **Audit:** trazabilidad de acciones.

## Providers

El dominio define puertos, por ejemplo:

```python
class AccessProvider(Protocol):
    def provision(self, command): ...
    def suspend(self, command): ...
    def activate(self, command): ...
    def reconcile(self, identity): ...
```

La implementación actual puede usar `RouterOSProvider`. Más adelante pueden existir `HuaweiProvider`, `GenericOltProvider` u otros sin cambiar los casos de uso.

## Event Driven

Los eventos representan hechos:

- `CustomerCreated`
- `ServiceCreated`
- `ServiceActivated`
- `ServiceSuspended`
- `AccessCredentialChanged`
- `ProvisioningRequested`
- `ProvisioningSucceeded`
- `ProvisioningFailed`
- `HotspotSessionStarted`
- `HotspotSessionExpired`
- `AccessMethodChanged`

Para operaciones que cruzan PostgreSQL y un equipo externo se recomienda Outbox + Worker, en lugar de una transacción distribuida.

## Arquitectura HotSpot VLAN 666

VLAN 666 es un dominio L2 de acceso temporal. Puede atravesar varias ONUs/casas. El MikroTik que presta el HotSpot es el punto lógico de autenticación y salida a Internet.

La movilidad entre ONUs requiere que todas transporten la misma VLAN de forma compatible y que el usuario mantenga una identidad/sesión reconocible por el HotSpot. La aplicación registra el acceso y su tiempo, pero no usa la MAC de una ONU concreta como identidad principal.

## Coexistencia PPPoE + HotSpot

PPPoE administra el servicio permanente. HotSpot administra pruebas/venta temporal. Ambos pueden pertenecer al mismo cliente y tener historial separado.

Una conversión debe ser explícita:

```text
HotSpotTrial
     |
     | convert
     v
PPPoEService
```

No se borra la prueba original.

## Componentes

```text
API -> Application -> Domain
             |           |
             v           v
       Repositories   Providers
             |           |
             v           v
        PostgreSQL   RouterOS / ONU
             |
          Outbox
             |
          Worker
```
