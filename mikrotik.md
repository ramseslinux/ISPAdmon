# Integración MikroTik RouterOS v7

## Principio

MikroTik es un sistema de ejecución de configuración. PostgreSQL conserva la intención y el estado esperado. El provider traduce esa intención a RouterOS.

## API

La implementación existente usa `routeros_api.RouterOsApiPool`. El código actual conecta con `8728` y `plaintext_login=True`; esto debe considerarse una etapa de laboratorio. Para una instalación real se debe evaluar `8729`/API-SSL y restringir el origen de administración.

Variables:

```env
MIKROTIK_HOST=10.66.20.1
MIKROTIK_PORT=8728
MIKROTIK_USER=admin
MIKROTIK_PASSWORD=<secret>
```

No usar un usuario administrador global desde la aplicación en producción. Crear un usuario de API con permisos mínimos.

## Recursos RouterOS relevantes

### PPPoE

- `/ppp/profile`
- `/ppp/secret`
- `/interface/pppoe-server/server`
- `/ppp/active`

El plan comercial debe mapearse a un PPP profile mediante una política explícita. No construir nombres de perfiles con `replace(" ", "")` como regla definitiva; debe existir un campo de mapeo o una entidad de política.

### HotSpot

- `/ip/hotspot/user`
- `/ip/hotspot/user/profile`
- `/ip/hotspot/active`
- `/ip/hotspot/host`

El código existente ya implementa creación, cambio de perfil, suspensión y activación de usuarios HotSpot. Esa funcionalidad debe migrarse al `RouterOSProvider`.

## VLAN 666

Laboratorio documentado:

```text
ether2
  |
  +-- vlan666
        IP: 10.66.6.1/24
        DHCP: 10.66.6.10-10.66.6.254
        HotSpot: sobre la red VLAN 666
```

`ether2` debe transportar L2. No mantener simultáneamente un DHCP/servicio IP directo sobre `ether2` si el diseño final exige que los clientes entren por `vlan666`.

Las ONUs que participen en el HotSpot deben transportar/taggear VLAN 666 según el diseño de la red de acceso. Todas las casas/ONUs que deban compartir el acceso temporal tienen que alcanzar el mismo dominio L2 lógico o un diseño equivalente que preserve la sesión.

## Sincronización

La sincronización debe ser explícita:

```text
Desired state (PostgreSQL)
          |
          v
Reconciler
          |
          v
RouterOS actual
          |
          v
Diff
          |
          v
Provisioning Job
```

No asumir que una operación `POST /cliente` equivale a estado sincronizado para siempre. RouterOS puede cambiarse fuera de la aplicación.

## Idempotencia

Antes de crear un recurso:

1. buscar por identidad estable;
2. si existe y coincide, devolver éxito idempotente;
3. si existe y difiere, actualizar;
4. si no existe, crear;
5. guardar el identificador externo (`routeros_id`) cuando exista.

## Buenas prácticas

- Timeouts de conexión.
- Pool controlado.
- Reintentos solo para errores transitorios.
- Logs sin contraseñas.
- Límites de concurrencia por router.
- Health check del dispositivo.
- Reconciliación periódica.
- Auditoría de cambios.
- API de administración aislada de redes de clientes.
