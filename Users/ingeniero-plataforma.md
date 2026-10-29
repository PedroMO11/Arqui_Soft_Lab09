# Persona: Ingeniero de Plataforma (SRE / TI)

**Ejemplo:** Valeria, 29 años, ingeniera de software y SRE, a cargo de la infraestructura del sistema de control y de su modernización.

## Descripción
Mantiene disponible la plataforma de control: infraestructura, despliegues, monitoreo y failover. Conduce la migración gradual desde el legacy y el piloto de la nueva librería de eventos.

## Objetivos
- Modernizar el sistema **sin detener nunca** el control de la planta.
- Detectar degradaciones antes de que afecten la operación.
- Evaluar con datos si la nueva librería de eventos mejora la latencia.

## Tareas en el sistema
- Operar el Parallel Run y el traspaso reactor por reactor entre el sistema nuevo y el legacy (RF-09).
- Conducir la migración de datos: CDC para lo nuevo, ETL por fases para lo histórico, y validación con conteos y checksums (RF-09).
- Correr el piloto de la librería en Parallel Run, comparar P95/P99 y adoptarla con feature flag (RF-10).
- Monitorear dashboards de availability, reliability, tráfico y latencia, y ejecutar o verificar el failover (RF-07).

## Pain points
| Tipo | Pain point |
|---|---|
| Logístico | El legacy tiene poca documentación, alto acoplamiento y protocolos antiguos difíciles de integrar. |
| Emocional | Cualquier cambio en producción es de alto riesgo: no hay ambiente de pruebas con carga real ni un rollback claro. |
| Físico | No tiene observabilidad: no puede medir la latencia ni encontrar el bottleneck del sistema actual. |
| Financiero | Adoptar la nueva librería sin evidencia puede costar más que el beneficio. Se necesita un POC y un benchmark. |

## Requisitos relacionados
RF-07, RF-09, RF-10 · RNF-01, RNF-02, RNF-05, RNF-09, RNF-10
