ISP Management Stack

Sistema de gestión y automatización para un ISP basado en MikroTik, FastAPI, PostgreSQL, Redis y Telegram, desplegado mediante Docker Compose.

Arquitectura

                         ┌──────────────────────┐
                         │      INTERNET        │
                         └──────────┬───────────┘
                                    │
                            ┌───────▼────────┐
                            │ MikroTik CHR   │
                            │ Router / ISP   │
                            └───────┬────────┘
                                    │
                         MikroTik API / SNMP
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
             ┌──────▼──────┐                 ┌──────▼──────┐
             │   FastAPI   │                 │ Monitoring  │
             │     API     │                 │ Prometheus  │
             └──────┬──────┘                 └─────────────┘
                    │
          ┌─────────┼──────────┐
          │         │          │
    ┌─────▼────┐ ┌──▼─────┐ ┌──▼─────────────┐
    │PostgreSQL│ │ Redis  │ │ Telegram Worker│
    └──────────┘ └────────┘ └────────────────┘

Componentes

MikroTik CHR — Router, HotSpot y control de clientes.

FastAPI — API principal y lógica de negocio.

PostgreSQL 16 — Base de datos.

Redis 7 — Cache y servicios auxiliares.

Telegram Bot — Administración y notificaciones.

Docker Compose — Orquestación de servicios.

Prometheus — Monitorización.

SNMP Exporter — Métricas del MikroTik.

Node Exporter — Métricas del servidor Linux.

Blackbox Exporter — Comprobación de conectividad.

Estructura del proyecto

isp-stack/
├── api/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│       ├── main.py
│       ├── mikrotik.py
│       ├── mikrotik_clientes.py
│       ├── pagos.py
│       ├── planes.py
│       ├── provision.py
│       ├── servicios.py
│       ├── telegram_bot.py
│       ├── telegram_worker.py
│       └── ...
├── database/
│   └── init.sql
├── compose.yml
├── .env.example
├── .gitignore
├── verificar_vencidos.sh
└── README.md

Requisitos

Linux / Ubuntu Server

Docker Engine

Docker Compose

MikroTik RouterOS 7.x o MikroTik CHR

Acceso API al MikroTik

SNMP v2c para monitorización

Cuenta/bot de Telegram

Configuración

Las credenciales y secretos se almacenan en .env.

Nunca se debe subir .env al repositorio.

Crear el archivo a partir del ejemplo:

cp .env.example .env
nano .env

Variables principales:

TELEGRAM_BOT_TOKEN=
TELEGRAM_CHAT_ID=

POSTGRES_DB=isp
POSTGRES_USER=ispadmin
POSTGRES_PASSWORD=

MIKROTIK_HOST=10.66.20.1
MIKROTIK_PORT=8728
MIKROTIK_USER=isp-api
MIKROTIK_PASSWORD=

Docker Compose

Validar la configuración:

docker compose config --quiet

Levantar los servicios:

docker compose up -d

Ver estado:

docker compose ps

Ver logs:

docker compose logs -f

Ver logs de la API:

docker compose logs -f api

Ver logs del bot:

docker compose logs -f telegram-worker

Reiniciar:

docker compose restart

Actualizar y reconstruir:

docker compose up -d --build

Detener:

docker compose down

API

La API FastAPI utiliza el puerto:

8000

Comprobación básica:

curl http://127.0.0.1:8000/

La API comprueba la disponibilidad de:

FastAPI

PostgreSQL

Redis

Base de datos

PostgreSQL contiene las principales entidades del sistema:

clientes

dispositivos

pagos

planes

servicios

tickets

La estructura inicial se encuentra en:

database/init.sql

Gestión de clientes

El sistema permite asociar clientes con:

Nombre

Datos de contacto

Usuario HotSpot

Dirección IP

MAC

Plan contratado

Estado del servicio

Ejemplo de cliente de laboratorio:

Cliente: cliente001
IP: 10.66.7.100
Plan MikroTik: Fibra200

Planes de Internet

Los planes se pueden asociar con perfiles de MikroTik.

Ejemplo:

Fibra50   → 50 Mbps
Fibra100  → 100 Mbps
Fibra200  → 200 Mbps

Los perfiles correspondientes pueden configurarse en MikroTik:

/ip hotspot user profile
add name=Fibra50 rate-limit=50M/50M
add name=Fibra100 rate-limit=100M/100M
add name=Fibra200 rate-limit=200M/200M

Pagos

El sistema registra:

Cliente

Importe

Fecha de vencimiento

Estado del pago

Servicio asociado

Cuando se registra un pago válido, el servicio puede ser reactivado automáticamente.

Suspensión automática

El sistema incluye un proceso de verificación de vencimientos:

verificar_vencidos.sh

El proceso consulta los pagos y detecta servicios vencidos.

Cuando corresponde:

Detecta el servicio vencido.

Suspende al usuario HotSpot en MikroTik.

Actualiza el estado en PostgreSQL.

Registra el resultado.

El proceso está preparado para ejecutarse mediante systemd.

Ejemplo:

sudo systemctl status isp-verificar-vencidos.timer

Ejecutar manualmente:

sudo systemctl start isp-verificar-vencidos.service

Consultar el log:

sudo tail -f /var/log/isp-verificar-vencidos.log

Telegram

El sistema incluye un worker para administración mediante Telegram.

Comandos disponibles:

/start
/health
/clientes
/cliente ID
/estado ID
/suspender ID
/reactivar ID
/vencidos
/pago ID monto

Ejemplo:

/cliente 1

Muestra la información del cliente.

Ejemplo:

/estado 1

Consulta el estado del servicio y su relación con MikroTik.

Ejemplo:

/suspender 1

Suspende el servicio del cliente.

Ejemplo:

/reactivar 1

Reactiva el servicio.

Ejemplo:

/pago 1 399

Registra un pago para el cliente.

MikroTik API

La aplicación se comunica con MikroTik mediante RouterOS API.

Configuración:

MIKROTIK_HOST=10.66.20.1
MIKROTIK_PORT=8728
MIKROTIK_USER=isp-api
MIKROTIK_PASSWORD=

El acceso API debe restringirse únicamente al servidor de gestión.

Ejemplo de restricción:

Servidor ISP → 10.66.20.10
MikroTik     → 10.66.20.1

Red del laboratorio

Direcciones principales:

Red de gestión:
10.66.20.0/24

MikroTik CHR:
10.66.20.1

Servidor Ubuntu:
10.66.20.10

Red de clientes:
10.66.7.0/24

Gateway HotSpot:
10.66.7.1

Monitorización

La plataforma puede integrarse con Prometheus para monitorizar:

Disponibilidad del servidor

Disponibilidad del MikroTik

Tráfico de interfaces

Estado de servicios

Métricas SNMP

Conectividad IP

SNMP

El MikroTik utiliza SNMP v2c con una comunidad específica para monitorización.

El acceso debe limitarse al servidor de monitorización.

Ejemplo de prueba:

snmpwalk -v2c -c isp-monitor 10.66.20.1 1.3.6.1.2.1.1

Tráfico de interfaces

Se pueden obtener métricas mediante:

ifInOctets
ifOutOctets

Y convertirlas a Mbps mediante Prometheus.

Seguridad

Secretos

No almacenar credenciales directamente en:

compose.yml

código Python

README

repositorio Git

Utilizar:

.env

El archivo .env está excluido mediante .gitignore.

Archivos ignorados

También se excluyen:

*.bak
*.log
__pycache__/
.venv/
postgres_data/
redis_data/
.pem
.key
.crt

Git

Inicializar el repositorio:

cd ~/isp-stack
git init
git branch -M main

Comprobar archivos:

git status

Verificar que .env está ignorado:

git check-ignore -v .env

Agregar archivos:

git add .

Crear commit:

git commit -m "Initial ISP management stack"

Agregar el repositorio remoto:

git remote add origin git@github.com:ramseslinux/isp-stack.git

Subir el proyecto:

git push -u origin main

Sincronización posterior:

git add .
git commit -m "Update ISP stack"
git push

Para actualizar desde GitHub:

git pull --rebase

Desarrollo

Para modificar la API:

cd ~/isp-stack
nano api/app/main.py

Después reconstruir:

docker compose up -d --build api

Para modificar el worker de Telegram:

docker compose up -d --build telegram-worker

Estado del proyecto

Actualmente el proyecto incluye:

MikroTik CHR

Acceso RouterOS API

Gestión de clientes

Gestión de planes

Registro de pagos

Suspensión por vencimiento

Reactivación mediante pago

Telegram Bot

PostgreSQL

Redis

Docker Compose

Verificación automática de vencimientos

SNMP

Prometheus

Monitorización del MikroTik

Control de secretos mediante .env

Preparación para Git/GitHub

Próximos objetivos

Notificaciones automáticas de vencimiento:

3 días antes

1 día antes

Día de vencimiento

Notificación de suspensión

Panel web para administración

Dashboard de clientes

Reportes de pagos

Facturación

Automatización avanzada de provisión

Integración con sistemas de monitoreo

Backups automáticos

Auditoría de operaciones

Gestión multi-MikroTik

Licencia

Proyecto privado / laboratorio ISP.

La licencia y condiciones de distribución deberán definirse antes de publicar el proyecto como software abierto.
