# Modelo de dominio

## Entidades

### Customer
Representa a la persona o empresa contratante.

Campos conceptuales: `id`, `name`, `phone`, `email`, `address`, `status`, fechas.

### Plan
Define velocidad, precio y políticas de acceso.

### Service
Une un cliente con un plan y representa un servicio contratado.

Estados sugeridos:

```text
PENDING -> ACTIVE -> SUSPENDED -> ACTIVE
                  \-> CANCELLED
```

### AccessCredential
Representa la identidad usada para acceder a la red. Debe indicar el método (`PPPOE` o `HOTSPOT`) y nunca exponer la contraseña en respuestas o logs.

### HotspotTrial
Representa una prueba temporal. Tiene inicio, expiración, perfil, identidad de acceso y estado.

Estados:

```text
CREATED -> ACTIVE -> EXPIRED
              \-> REVOKED
```

### NetworkDevice
Representa un RouterOS, OLT u otro equipo administrado.

### ProvisioningJob
Representa una operación pendiente o ejecutada contra infraestructura.

Estados:

```text
PENDING -> RUNNING -> SUCCEEDED
                  \-> FAILED -> RETRYING -> RUNNING
```

## Reglas de negocio

1. Un servicio pertenece a un cliente.
2. Un servicio activo debe tener un plan activo.
3. Un username de acceso debe ser único dentro del contexto del proveedor/método donde se utilice.
4. Suspender un servicio debe impedir el acceso según el método de acceso.
5. Reactivar un servicio debe reconstruir la configuración necesaria si fue eliminada.
6. Una prueba HotSpot tiene una expiración obligatoria.
7. Una sesión de prueba no se convierte automáticamente en PPPoE sin una acción explícita.
8. Convertir una prueba conserva su historial y crea la nueva relación de acceso.
9. VLAN 666 identifica el transporte del HotSpot, no al cliente.
10. La MAC puede servir para diagnóstico/sesión, pero no debe ser la identidad comercial principal.

## Casos de uso

- Crear cliente.
- Crear plan.
- Crear servicio PPPoE.
- Suspender/reactivar servicio.
- Crear prueba HotSpot.
- Vender tiempo de HotSpot.
- Consultar sesión HotSpot.
- Expirar prueba.
- Convertir HotSpot a PPPoE.
- Sincronizar router.
- Reconciliar usuarios locales vs MikroTik.
- Reintentar provisión.
- Consultar auditoría.

## Eventos de dominio

Los eventos deben contener identificadores y metadatos mínimos. No deben transportar contraseñas.
