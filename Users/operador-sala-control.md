# Persona: Operador de Sala de Control

**Ejemplo:** Lucía, 32 años, técnica en operación de plantas, 6 años en la planta, turnos rotativos de 8 h.

## Descripción
Supervisa en tiempo real el sistema autónomo que controla los 3 reactores. El sistema decide y actúa solo; Lucía verifica que las acciones automáticas tengan sentido, reconoce las alarmas e interviene cuando la automatización escala un problema que no logra controlar.

## Objetivos
- Ver en un solo lugar el estado de los 3 reactores, sus sensores y el **modo** de cada uno (automático, manual, autónomo local, apagado).
- Entender qué hizo el sistema y por qué (qué regla disparó cada acción automática).
- Enterarse de inmediato cuando la automatización escala un problema a intervención manual.
- Que el sistema no la llene de alarmas falsas por sensores dañados.

## Tareas en el sistema
- Supervisar el panel y las acciones automáticas recientes (RF-05).
- Reconocer alarmas y escalamientos (RF-04).
- Cuando el Jefe de Turno pasa un reactor a modo manual, enviar los comandos de refrigeración desde el panel (RF-03).
- Avisar al Jefe de Turno si observa una acción automática incorrecta (RF-03).

## Pain points
| Tipo | Pain point |
|---|---|
| Logístico | Hoy el control es muy manual: revisa medidores, registra lecturas a mano y pierde tiempo en tareas repetitivas que el sistema podría hacer solo. |
| Emocional | Estrés y fatiga por alarmas poco claras o falsas (un sensor dañado dispara alertas). No sabe si una alerta es real. |
| Físico | No tiene una vista consolidada: debe consultar varios controladores o pantallas por separado. |
| Emocional | Si el sistema actúa solo sin explicar por qué, no puede confiar en él ni detectar cuándo se equivoca. |

## Requisitos relacionados
RF-03, RF-04, RF-05, RF-06, RF-07 · RNF-02, RNF-06, RNF-10
