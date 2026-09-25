# Seguridad

## Autenticación

La API debe autenticar usuarios humanos mediante una sesión/token con expiración y rotación. Los workers y providers usan credenciales de servicio separadas.

## RBAC

Roles iniciales sugeridos:

- `SUPER_ADMIN`
- `NETWORK_ADMIN`
- `ISP_OPERATOR`
- `SUPPORT`
- `READ_ONLY`

Permisos deben asignarse por acción, no por pantalla.

Ejemplos:

```text
customer:read
customer:write
service:activate
service:suspend
hotspot:create
hotspot:revoke
network:read
network:provision
network:reconcile
billing:read
security:audit
```

## Secretos

Nunca:

- guardar contraseñas en Git;
- usar `DATABASE_URL` con contraseña real como valor por defecto en código;
- devolver passwords de MikroTik en JSON;
- escribir secretos en logs.

Usar variables de entorno en laboratorio y un secret manager en producción.

## MikroTik

Crear usuarios de API con permisos mínimos. Restringir por IP/origen cuando sea posible. Preferir API-SSL. Separar administración del tráfico de clientes.

## Red de clientes

La VLAN 666 debe considerarse red de acceso temporal. No debe permitir acceso administrativo al router. Las reglas de firewall deben limitar explícitamente el tráfico hacia la infraestructura de gestión.

## Auditoría

Registrar:

- quién realizó la acción;
- qué recurso cambió;
- estado anterior/relevante;
- estado solicitado;
- resultado;
- request ID;
- fecha/hora;
- dispositivo afectado cuando aplique.

No registrar contraseñas, tokens ni secretos.

## Idempotencia y seguridad operacional

La idempotencia evita duplicaciones cuando un operador reintenta una operación o un worker se recupera después de un timeout.

## Backups

PostgreSQL debe contar con backups y pruebas periódicas de restauración. Las credenciales de backup deben estar separadas de las de la aplicación.
