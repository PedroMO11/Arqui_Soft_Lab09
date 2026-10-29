# Persona: Oficial de Seguridad Nuclear / Auditor

**Ejemplo:** Carmen, 52 años, especialista en seguridad nuclear y cumplimiento regulatorio. Interlocutora con el ente regulador.

## Descripción
Verifica que la operación de la planta cumpla las normas de seguridad. Investiga incidentes (RCA) y prepara reportes para gerencia y el regulador.

## Objetivos
- Reconstruir con exactitud **quién encendió o apagó qué, cuándo y por qué**: una persona o el sistema automático y, en ese caso, qué regla disparó la acción.
- Contar con evidencia inalterable de las acciones y lecturas.
- Generar reportes periódicos y de incidentes sin depender de TI.

## Tareas en el sistema
- Consultar y filtrar el registro de auditoría por usuario, reactor, comando o rango de fechas (RF-08).
- Exportar reportes de cumplimiento e incidentes (RF-05, RF-08).
- Revisar las alertas críticas y su tiempo de respuesta (RF-04).
- Validar y firmar la migración de la auditoría legacy: hash coincidente y registros completos (RF-09).

## Pain points
| Tipo | Pain point |
|---|---|
| Logístico | La trazabilidad actual es incompleta: hay registros manuales y faltan datos de contexto. |
| Emocional | No puede asegurar que los registros no fueron modificados. |
| Financiero | Una falta de cumplimiento regulatorio expone a sanciones y a la detención de la planta. |
| Logístico | Armar un RCA toma días porque la información está dispersa entre el legacy y las bitácoras. |

## Requisitos relacionados
RF-04, RF-08, RF-09 · RNF-07, RNF-08, RNF-09
