# Persona: Jefe de Turno (Supervisor de Planta)

**Ejemplo:** Andrés, 47 años, ingeniero nuclear, 15 años de experiencia, responsable de la planta durante su turno.

## Descripción
Es responsable de la planta durante su turno. El sistema controla los reactores de forma autónoma; Andrés supervisa que la automatización funcione bien y es quien **toma el control manual** de un reactor cuando la automatización falla. Coordina la respuesta ante incidentes.

## Objetivos
- Tener una visión global y confiable del estado de la planta y de lo que está haciendo el sistema.
- Poder tomar el control manual de un reactor en cualquier momento si la automatización falla.
- Que los límites de seguridad nunca se desactiven, ni siquiera por un comando manual.
- Saber qué hizo el sistema y qué hizo cada persona durante el turno.

## Tareas en el sistema
- Pasar un reactor a **modo manual** cuando la automatización falla, y confirmar su regreso a modo automático (RF-03).
- Configurar las reglas y umbrales de la automatización (RF-03, RF-04).
- Atender los escalamientos que la automatización no logra resolver (RF-04).
- Supervisar los apagados automáticos (por pérdida de cobertura o falla de refrigeración) y el failover del Main Controller (RF-06, RF-07).
- Confirmar el regreso del modo autónomo local al control del Main Controller (RF-07).
- Revisar el historial de acciones automáticas y manuales del turno (RF-08).
- Aprobar los pasos de la migración del legacy (RF-09).

## Pain points
| Tipo | Pain point |
|---|---|
| Emocional | Una decisión errónea puede causar un desastre. Necesita certeza de que el sistema decide con datos correctos y actuales. |
| Logístico | Hoy reconstruye "quién hizo qué" con bitácoras en papel o preguntando al personal. |
| Logístico | Si el controlador central se cae, no hay un procedimiento automático claro: depende de la reacción manual. |
| Financiero | Una parada no planificada de un reactor genera grandes pérdidas. Necesita evitar falsos apagados. |

## Requisitos relacionados
RF-03, RF-04, RF-06, RF-07, RF-08, RF-09 · RNF-03, RNF-04, RNF-07
