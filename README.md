ISP Stack

Plataforma para administrar la operación de un ISP desde una sola consola, con PostgreSQL como fuente de verdad y MikroTik RouterOS v7 como infraestructura de acceso y aplicación de políticas.

Objetivo

El proyecto separa la gestión comercial y de servicios de la configuración de red. La consola administra clientes, planes, servicios, credenciales y estados; un motor de aprovisionamiento aplica los cambios en los equipos de red mediante proveedores (providers).

El diseño contempla dos formas de acceso que conviven:

PPPoE: mecanismo principal para clientes administrados.

HotSpot sobre VLAN 666: mecanismo de prueba de servicio, portal cautivo y venta temporal de acceso.

Una sesión autenticada en el HotSpot de la VLAN 666 debe poder conservar su acceso al desplazarse entre diferentes ONUs/casas que transporten esa misma VLAN. La movilidad depende de que la VLAN 666 sea una red L2 común y de que la autenticación/sesión se gestione de forma coherente; la ONU no debe convertirse en la fuente de verdad del usuario.

La arquitectura también deja preparado el camino para ONUs/OLTs, venta de tiempo, Telegram/chatbot IA y una futura conversión de una prueba HotSpot a un servicio PPPoE conservando el historial del cliente.

Estado actual

El laboratorio utiliza un CHR con RouterOS v7. En la prueba documentada:

WAN del CHR: ether1, 10.0.2.15/24, gateway 10.0.2.2.

Red de pruebas HotSpot: VLAN 666 sobre ether2.

Gateway VLAN 666: 10.66.6.1/24.

DHCP: 10.66.6.10-10.66.6.254.

ether2 queda como transporte L2 para la VLAN y no debe conservar una IP que contradiga el diseño VLAN.

El DHCP antiguo ligado directamente a ether2 quedó identificado como configuración inválida después del refactor.

El router de producción/laboratorio del ISP también usa RouterOS v7; el equipo documentado HAWEI HOGAR es un hEX S con RouterOS 7.24.4.

Arquitectura propuesta

                         +----------------------+
                         |     Web / API        |
                         |      FastAPI         |
                         +----------+-----------+
                                    |
                         +----------v-----------+
                         |       Domain         |
                         | clientes / servicios |
                         | planes / acceso      |
                         +----------+-----------+
                                    |
                    +---------------+---------------+
                    |                               |
             +------v------+                 +------v------+
             | PostgreSQL  |                 | Event Bus   |
             | fuente de   |                 | / Jobs      |
             | verdad      |                 +------+------+
             +-------------+                        |
                                                    v
                                           +-------------------+
                                           | Provisioning      |
                                           | Workers / Retry   |
                                           +---------+---------+
                                                     |
                          +--------------------------+-------------------+
                          |                                              |
                   +------v------+                                +------v------+
                   | MikroTik    |                                | ONU / OLT   |
                   | Provider    |                                | Provider    |
                   +------+------+                                +-------------+
                          |
             +------------+-------------+
             |                          |
        PPPoE                         HotSpot
                                      VLAN 666

Estructura documental

docs/README.md — índice de documentación.

docs/architecture.md — arquitectura y límites de módulos.

docs/domain.md — dominio, entidades, estados y casos de uso.

docs/mikrotik.md — integración RouterOS v7, PPPoE y HotSpot.

docs/provisioning.md — jobs, workers, eventos, reintentos e idempotencia.

docs/api.md — contrato REST y DTOs.

docs/database.md — modelo de datos y ERD lógico.

docs/security.md — autenticación, RBAC, secretos y auditoría.

docs/conventions.md — estructura y convenciones de desarrollo.

docs/adr/ — decisiones de arquitectura.

Arquitectura de código objetivo

isp-stack/
├── README.md
├── api/
│   └── app/
│       ├── main.py
│       ├── domain/
│       │   ├── entities/
│       │   ├── value_objects/
│       │   ├── services/
│       │   └── events/
│       ├── application/
│       │   ├── commands/
│       │   ├── queries/
│       │   ├── dto/
│       │   └── handlers/
│       ├── infrastructure/
│       │   ├── db/
│       │   ├── mikrotik/
│       │   ├── repositories/
│       │   └── events/
│       └── interfaces/
│           └── http/
├── workers/
│   ├── jobs/
│   └── workers/
├── migrations/
├── tests/
├── docs/
│   └── adr/
└── docker-compose.yml

El código existente de planes.py, servicios.py, provision.py, mikrotik.py y mikrotik_clientes.py es el punto de partida funcional. El refactor debe mover la lógica de negocio fuera de los routers HTTP y evitar que FastAPI conozca directamente detalles de RouterOS o SQL.

Flujo de provisión de cliente PPPoE

Crear o localizar el cliente.

Crear el servicio y asociarlo a un plan.

Validar que el plan esté activo.

Generar/validar la identidad de acceso PPPoE.

Crear o actualizar el recurso correspondiente en MikroTik.

Registrar el resultado y el estado de sincronización.

Emitir evento de dominio/integración.

Si falla una operación externa, registrar el job para reintento; no depender de un rollback SQL para deshacer una operación que ya ocurrió en MikroTik.

Flujo HotSpot VLAN 666

Cliente móvil
    |
    v
ONU / AP / casa A ----+
                      |
ONU / AP / casa B ----+---- VLAN 666 ---- MikroTik HotSpot
                      |
ONU / AP / casa C ----+
                                  |
                                  v
                           autenticación
                                  |
                                  v
                             Internet

La VLAN 666 representa el dominio de transporte del HotSpot. Las ONUs son puntos de acceso/transporte; el usuario HotSpot, su sesión, el tiempo restante y el estado comercial pertenecen al dominio del ISP.

Perfiles de prueba

El laboratorio contempla perfiles temporales como:

30 minutos

1 hora

3 horas

24 horas

Los nombres concretos de los perfiles deben configurarse en el router y registrarse en PostgreSQL como una política de acceso, no como una cadena dispersa por el código.

Reglas de diseño

PostgreSQL es la fuente de verdad de clientes, servicios, planes, credenciales lógicas, estados y auditoría.

MikroTik es un sistema externo de ejecución de configuración.

Los routers HTTP no deben contener lógica de negocio ni SQL complejo.

Los cambios externos se realizan mediante providers.

Toda operación de provisión debe ser idempotente.

Los trabajos deben poder reintentarse sin duplicar usuarios ni servicios.

Los eventos deben representar hechos ocurridos, no comandos pendientes.

La conversión HotSpot → PPPoE conserva el cliente y el historial; crea/cambia el servicio y su método de acceso.

La VLAN 666 es una red de acceso temporal y no sustituye la administración PPPoE.

Las credenciales y secretos nunca se almacenan en el repositorio.

Desarrollo local

El proyecto está pensado para ejecutarse con Docker Compose y PostgreSQL. Las variables de entorno deben contener las credenciales reales fuera del código.

Variables mínimas previstas:

DATABASE_URL=postgresql://<usuario>:<password>@postgres:5432/isp
MIKROTIK_HOST=<router>
MIKROTIK_PORT=8728
MIKROTIK_USER=<usuario-api>
MIKROTIK_PASSWORD=<secreto>

Para producción se debe preferir RouterOS API segura (8729) cuando la topología lo permita.

Próximo refactor

El refactor no debe ser una reescritura funcional sin control. Debe hacerse por capas:

estabilizar el contrato de dominio;

separar repositorios y providers;

introducir jobs y eventos;

migrar provisión PPPoE;

integrar HotSpot/VLAN 666;

agregar sincronización y reconciliación;

agregar RBAC y auditoría;

cubrir con pruebas unitarias, integración y pruebas contra CHR.

Principio operativo

El operador trabaja con clientes y servicios; el sistema traduce esas decisiones a configuraciones de red.

# Documentación ISP Stack

## Índice

| Documento | Contenido |
|---|---|
| [architecture.md](architecture.md) | Clean Architecture, DDD, Event Driven, Providers y límites de módulos |
| [domain.md](domain.md) | Entidades, estados, reglas de negocio y casos de uso |
| [mikrotik.md](mikrotik.md) | RouterOS v7, API, PPPoE, HotSpot, perfiles y sincronización |
| [provisioning.md](provisioning.md) | Jobs, Workers, eventos, reintentos e idempotencia |
| [api.md](api.md) | REST, DTOs, errores y ejemplos |
| [database.md](database.md) | Modelo relacional, ERD lógico e índices |
| [security.md](security.md) | Auth, RBAC, secretos, auditoría y seguridad de red |
| [conventions.md](conventions.md) | Estructura, nombres, patrones y reglas de código |
| [adr/](adr/) | Decisiones de arquitectura |

## Orden recomendado

1. architecture
2. domain
3. database
4. mikrotik
5. provisioning
6. api
7. security
8. conventions
9. ADRs
