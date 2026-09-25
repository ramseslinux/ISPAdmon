# Convenciones de desarrollo

## Idioma

Código, nombres de clases y endpoints en inglés. Documentación operativa puede estar en español.

## Nombres

- Clases: `PascalCase`.
- Funciones/variables Python: `snake_case`.
- Endpoints REST: sustantivos plurales.
- Eventos: pasado, por ejemplo `ServiceActivated`.
- Commands: verbo + objeto, por ejemplo `ActivateService`.
- Queries: `GetService`, `ListPlans`.

## Estructura

No colocar SQL ni llamadas RouterOS dentro de los endpoints.

Incorrecto:

```text
HTTP endpoint -> psycopg -> RouterOS
```

Correcto:

```text
HTTP -> Use Case -> Repository / Provider
```

## DTOs

Los DTOs de entrada/salida no deben convertirse en entidades de dominio. Validar formato en la interfaz y reglas de negocio en el dominio/aplicación.

## Excepciones

Usar excepciones de dominio/aplicación con códigos estables. Traducirlas a HTTP en una sola capa.

## Logs

Logs estructurados con `request_id`, `job_id`, `device_id` y `service_id` cuando existan. No incluir secretos.

## Pruebas

- Unitarias: dominio y casos de uso.
- Integración: PostgreSQL y repositorios.
- Provider: RouterOS CHR de laboratorio.
- API: contrato HTTP.
- E2E: creación de servicio -> job -> RouterOS -> reconciliación.

## Compatibilidad

Los cambios de API y base de datos deben tener migración explícita. Evitar cambios destructivos en una sola versión.

## Git

Commits pequeños y relacionados con una sola intención. No mezclar refactor arquitectónico con cambios funcionales de red sin necesidad.
