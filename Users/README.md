# Usuarios del Sistema de Control "Save Chernobyl"

> El sistema es **autónomo**: decide y actúa solo. Las personas supervisan e intervienen manualmente solo si la automatización falla.

## Usuarios humanos (personas)

| Persona | Rol | Archivo |
|---|---|---|
| Operador de Sala de Control | Supervisa la automatización, reconoce alarmas y opera en modo manual cuando se activa | [operador-sala-control.md](operador-sala-control.md) |
| Jefe de Turno | Toma el control manual si la automatización falla y configura sus reglas | [jefe-de-turno.md](jefe-de-turno.md) |
| Ingeniero de Mantenimiento e Instrumentación | Mantiene y reemplaza sensores, controladores y sistemas de refrigeración | [ingeniero-mantenimiento.md](ingeniero-mantenimiento.md) |
| Oficial de Seguridad Nuclear / Auditor | Revisa la trazabilidad (acciones automáticas y manuales) y genera reportes regulatorios | [auditor-seguridad.md](auditor-seguridad.md) |
| Ingeniero de Plataforma (SRE / TI) | Mantiene el sistema disponible y conduce la migración del legacy | [ingeniero-plataforma.md](ingeniero-plataforma.md) |

## Actores externos (no humanos)

Son equipos o sistemas **fuera del sistema**: no se implementan, solo intercambian datos con él. El Main Controller y los controladores locales no son actores, sino componentes internos del sistema.

| Actor | Rol | Interactúa con |
|---|---|---|
| Sensor (hasta 100 por reactor) | Envía una lectura de temperatura cada ~0.05 s. Puede fallar o enviar datos erróneos | Reading Ingestion del Main Controller y del controlador local |
| Cooling System (×3) | Refrigera el reactor. Recibe comandos, los confirma y reporta su estado | Actuator Control del controlador local |
| Backup Cooling System (×3) | Refrigeración de respaldo, independiente de la principal. Se activa si la principal falla | Actuator Control del controlador local |
| Shutdown System (×3) | Barras de control. Recibe la orden de apagado del reactor | Actuator Control del controlador local |
| Sistema Legacy | Control actual de la planta; coexiste durante la migración | Anti-Corruption Layer y CDC / ETL |

## Permisos por rol

| Acción | Operador | Jefe de Turno | Mantenimiento | Auditor | Plataforma |
|---|:-:|:-:|:-:|:-:|:-:|
| Ver panel en tiempo real | ✓ | ✓ | ✓ | ✓ (solo lectura) | ✓ |
| Reconocer alarmas y escalamientos | ✓ | ✓ | — | — | — |
| Pasar un reactor a modo manual / volver a automático | — | ✓ | — | — | — |
| Enviar comandos en modo manual | ✓ | ✓ | — | — | — |
| Encender o apagar un reactor en modo manual | Solicita | **Autoriza** | — | — | — |
| Configurar reglas y umbrales de la automatización | — | ✓ | — | — | — |
| Marcar sensor en mantenimiento | — | ✓ | ✓ | — | — |
| Confirmar regreso del modo autónomo local | — | ✓ | — | — | — |
| Consultar y exportar auditoría | — | ✓ | — | ✓ | — |
| Feature flags y migración | — | Aprueba | — | — | ✓ |
