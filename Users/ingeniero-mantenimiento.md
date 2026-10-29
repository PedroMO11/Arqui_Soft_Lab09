# Persona: Ingeniero de Mantenimiento e Instrumentación

**Ejemplo:** Diego, 38 años, ingeniero electrónico, especialista en instrumentación y sensores industriales.

## Descripción
Mantiene, calibra y reemplaza los sensores de temperatura, los controladores locales y los sistemas de refrigeración (principal y de respaldo). Atiende las notificaciones de sensores degradados y las fallas de refrigeración.

## Objetivos
- Saber de inmediato qué sensor falló, en qué reactor y desde cuándo.
- Reemplazar o calibrar un sensor sin afectar el control del reactor.
- Ver el historial de lecturas de un sensor para diagnosticar fallas.

## Tareas en el sistema
- Recibir las notificaciones de sensores degradados (RF-06).
- Marcar un sensor "en mantenimiento" para excluirlo de las decisiones mientras se interviene (RF-06).
- Consultar el historial de lecturas y fallas por sensor (RF-05, RF-08).
- Atender con urgencia las fallas del sistema de refrigeración y restablecerlo mientras opera el respaldo (RF-06).

## Pain points
| Tipo | Pain point |
|---|---|
| Logístico | Las fallas de sensores se detectan tarde, cuando un operador nota lecturas raras. |
| Físico | No tiene herramientas para ver el historial de un sensor ni para distinguir una falla del sensor de un problema real del reactor. |
| Emocional | Teme que intervenir un sensor dispare alarmas o acciones automáticas no deseadas. |
| Logístico | Con hasta 100 sensores por reactor, ubicar cuál falló es lento. |

## Requisitos relacionados
RF-05, RF-06, RF-08 · RNF-03, RNF-10
