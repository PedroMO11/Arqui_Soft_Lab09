# Requisitos — Sistema de Control "Save Chernobyl"

> Los requisitos describen **qué** debe hacer el sistema según el enunciado del Lab #12, no **cómo** se implementa. Las decisiones de diseño (componentes, patrones, tecnologías) están en el diagrama de arquitectura.
>
> **Contexto:** la planta tiene 3 reactores. Cada uno puede tener hasta 100 sensores de temperatura que envían una lectura en promedio cada 0.05 s. El sistema legacy funciona, pero el control es demasiado manual; debe seguir operando durante la modernización.
>
> **Load del sistema:** el load no son los usuarios sino las **lecturas de sensores**.
> 3 reactores × 100 sensores × 20 lecturas/s = **6,000 mensajes/s** (2,000 por reactor).
>
> **Modelo de control: el sistema es autónomo.** Toma las decisiones por su cuenta y las personas lo supervisan. Hay tres niveles:
> 1. **Automático (normal):** el Main Controller recibe los datos, toma las decisiones globales y envía los comandos.
> 2. **Automático (fallback):** si el Main Controller se desconecta, el controlador local de cada reactor controla su reactor por su cuenta.
> 3. **Manual:** los operadores intervienen solo si la automatización falla o no logra controlar la situación.

---

## Requisitos Funcionales

| ID | Requisito | Descripción | Usuarios |
|---|---|---|---|
| **RF-01** | Envío de lecturas de temperatura | Cada sensor de temperatura (hasta 100 por reactor) envía una lectura en promedio cada **0.05 s**. El Main Controller recibe todas las lecturas. Cada lectura identifica el sensor, el reactor, el valor y el momento de la medición. | Sistema |
| **RF-02** | Controlador local por reactor | Cada reactor tiene un **controlador local** que gestiona su refrigeración y sus alertas. Ejecuta los comandos del Main Controller y **rechaza cualquier comando que viole los límites de seguridad**. Dispone siempre de las lecturas actuales de su reactor, para poder controlarlo por su cuenta si el Main Controller se desconecta (RF-07). | Sistema |
| **RF-03** | Decisiones automáticas y control manual | El Main Controller **toma las decisiones globales de forma automática**. Según las lecturas de los 3 reactores y reglas configuradas, ajusta la refrigeración, activa la refrigeración de respaldo o apaga un reactor, y envía los comandos. **Control manual:** si la automatización falla, el Jefe de Turno puede pasar un reactor a **modo manual** y enviar comandos desde el panel. Los límites de seguridad (RF-06) siguen activos también en modo manual. El regreso a modo automático lo confirma el Jefe de Turno. | Sistema, Jefe de Turno, Operador |
| **RF-04** | Alertas y escalamiento | Alertar cuando la temperatura supera un umbral (**Advertencia** y **Crítico**) y en cada **acción automática** (p.ej. "refrigeración del reactor 2 aumentada por 312 °C"). **Escalar a intervención manual** cuando la automatización no logra controlar la situación: la temperatura sigue subiendo después de las acciones automáticas, los datos son insuficientes o contradictorios, o no se puede ejecutar un comando. | Operador, Jefe de Turno |
| **RF-05** | Panel de operadores en tiempo real | Panel con la temperatura de cada reactor y sensor en **tiempo real**, el **modo de cada reactor** (automático, manual, autónomo local, apagado), las acciones automáticas recientes, las **alarmas** activas, el historial y **reportes**. Desde el panel se reconocen alarmas y, en modo manual, se envían comandos según el rol. | Operador, Jefe de Turno, Auditor |
| **RF-06** | Fallas de sensores y de refrigeración | **Sensores:** detectar un sensor que **deja de responder** (sin lectura por > 500 ms) o que **envía datos erróneos** (fuera de rango, salto imposible, valor congelado o desvío respecto de los sensores vecinos). Ese sensor se excluye de las decisiones y se alerta a Mantenimiento, **sin apagar el reactor**. Si un reactor queda con menos del 80% de sensores sanos o con una zona sin sensores, se ejecuta un **apagado controlado automático** y se notifica al Jefe de Turno. **Refrigeración:** confirmar que cada comando de refrigeración se ejecutó y detectar si la refrigeración falla o deja de responder. En ese caso, se activa automáticamente la **refrigeración de respaldo** y se alerta. Si además la temperatura alcanza el umbral crítico, se ejecuta un **shutdown automático** del reactor. La refrigeración de respaldo sigue activa para disipar el calor residual. | Sistema, Ing. Mantenimiento, Jefe de Turno |
| **RF-07** | Desconexión del Main Controller | Si el Main Controller se desconecta, **el control no se interrumpe**: sus funciones se recuperan automáticamente, sin intervención manual. Mientras no se recuperen, cada controlador local **controla su reactor por su cuenta**: refrigera, activa la refrigeración de respaldo y ejecuta el shutdown si es necesario (RF-06). Registra sus acciones y las envía al registro de auditoría al reconectarse. El regreso al control del Main Controller lo confirma el Jefe de Turno. Si el controlador local también falla, el reactor queda en **estado seguro (fail-safe)**. | Sistema, Jefe de Turno |
| **RF-08** | Registro y trazabilidad | Registrar **todas las acciones y datos**, automáticos y manuales, para responder **quién prendió o apagó qué**. En una acción automática, qué componente la ejecutó (Main Controller o controlador local), **qué regla la disparó** y con qué lecturas. En una manual, quién la ejecutó, cuándo, desde dónde y con qué resultado. También los cambios de modo (automático ↔ manual). El registro se puede consultar y exportar como reporte. | Auditor, Jefe de Turno |
| **RF-09** | Coexistencia con el legacy | El sistema nuevo **convive con el legacy** sin reemplazarlo de golpe ni detenerlo. Antes de tomar el control de un reactor, se verifica que sus decisiones automáticas coincidan con las del legacy. Toma el control **reactor por reactor**, de forma reversible. Los datos del legacy (lecturas históricas y registro de acciones) se migran completos y sin alteraciones, y los registros migrados quedan identificados como de origen legacy. | Ing. Plataforma, Jefe de Turno, Auditor |
| **RF-10** | Piloto de la nueva librería de eventos | Evaluar como **piloto** la nueva librería de manejo de eventos propuesta para mejorar la latencia, **sin poner en riesgo el control** de la planta. Se adopta solo si demuestra menor latencia y los mismos resultados que la actual. La adopción es gradual (reactor por reactor) y se puede revertir al instante. | Ing. Plataforma |

---

## Requisitos No Funcionales

> Los valores numéricos son **hipotéticos**: todavía no hay datos reales de la planta. Se ajustarán con performance testing y monitoreo.

| ID | Atributo | Requisito | Métrica / Fitness function |
|---|---|---|---|
| **RNF-01** | Disponibilidad | El control de cada reactor debe estar disponible **≥ 99.999%** y las funciones del Main Controller **≥ 99.99%**. | Availability = 2xx / (2xx + 5xx), medida por reactor y por componente. |
| **RNF-02** | Tiempo real (latencia) | Desde la lectura del sensor hasta la **decisión automática**: **P99 ≤ 100 ms**. Desde la decisión hasta su ejecución en el reactor: **P99 ≤ 100 ms**. Desde la lectura hasta que se ve en el panel: **P99 ≤ 500 ms**. Los comandos críticos nunca esperan detrás de otros datos. | P95 y P99, nunca el promedio. Fitness: Good ≤ 100 ms · OK ≤ 200 ms · Bad > 300 ms (lectura → comando ejecutado). |
| **RNF-03** | Tolerancia a fallos (sin SPOF) | Ningún componente puede ser un **punto único de falla**. Si el Main Controller falla, el control se recupera automáticamente en **≤ 5 s** y ningún reactor queda sin control en ese lapso. Nunca hay dos controladores dando órdenes contradictorias a un mismo reactor. Cada reactor tiene **refrigeración de respaldo**. La falla de un reactor no afecta a los otros. | Tiempo de recuperación medido en simulacros de falla. 0 órdenes contradictorias. Activación de la refrigeración de respaldo **≤ 1 s** después de detectar la falla de la principal. |
| **RNF-04** | Seguridad operativa (fail-safe) | Una decisión errónea puede causar un desastre. Ante datos inconsistentes, pérdida de comunicación o una falla no clasificada, el sistema lleva el reactor a un **estado seguro**. Se prefiere **rechazar un comando antes que ejecutarlo con datos dudosos**. El **shutdown automático** se reserva para cuando ya no se puede refrigerar el reactor. La falla de un solo sensor nunca apaga un reactor. **Ningún comando manual puede desactivar los límites de seguridad.** | 0 comandos ejecutados con datos de sensores degradados. 0 shutdowns causados por un solo sensor. Shutdown ejecutado **≤ 1 s** después de detectar refrigeración caída con temperatura crítica. |
| **RNF-05** | Escalabilidad | El sistema debe soportar la carga base de **6,000 lecturas/s** con un margen de **3× (18,000 lecturas/s)** sin degradar la latencia de RNF-02. | Load test a 6,000 msg/s y spike test a 18,000 msg/s. Criterio hipotético para escalar: el tráfico ↑ coincide con availability ↓ y latencia ↑. |
| **RNF-06** | Reliability e integridad de datos | Ninguna lectura ni ningún comando se pierden. Ante sobrecarga, el **Load Shedding** del Main Controller solo descarta trabajo no crítico (refresco del panel, reportes), nunca lecturas ni comandos del control. Cada comando se **confirma** al ejecutarse, **nunca se ejecuta dos veces** y, si vence su plazo de validez, se descarta en lugar de ejecutarse tarde. | Reliability = 2xx / (2xx + 4xx + 5xx) **≥ 99.9%** en comandos. % de lecturas perdidas = 0. 0 comandos ejecutados después de su plazo de validez. |
| **RNF-07** | Trazabilidad / auditoría | El registro de acciones **no se puede modificar ni borrar** y se conserva por **≥ 10 años**. Una falla del registro **no debe detener ni ralentizar** el control. | 100% de las acciones (automáticas y manuales) registradas. Latencia del control sin cambio con el registro caído. |
| **RNF-08** | Seguridad | Autenticación de todos los usuarios y control de acceso por rol. Comunicación **cifrada y autenticada** entre sensores, controladores y el sistema. Red de control aislada de la red corporativa. | 0 comandos manuales sin usuario autenticado. Fitness: más de 3 intentos de login fallidos → bloqueo y alerta. |
| **RNF-09** | Modernización sin detener la planta | La modernización convive con el legacy **sin detener la planta**. Cada paso es **reversible (rollback ≤ 1 min)** y se valida antes de tomar el control. Los datos migrados llegan completos y sin alteraciones. | Downtime del control durante la modernización = 0. Divergencia entre las decisiones del sistema nuevo y del legacy < umbral definido. 100% de los registros de acciones migrados con integridad verificada. Diferencia en el conteo de lecturas históricas < 0.01%. |
| **RNF-10** | Observabilidad | Monitoreo y dashboards con **availability, reliability, tráfico y latencia** por reactor y por componente, con alertas proactivas al superar un umbral. | Tiempo de detección de un incidente ≤ 1 min. |

---

## Preguntas del caso y requisitos que las responden

| Pregunta | Requisitos |
|---|---|
| ¿Qué pasa si un sensor falla, deja de responder o envía datos erróneos? | RF-06, RNF-04 |
| ¿Qué pasa si falla la refrigeración? ¿Alerta o shutdown? | RF-06, RNF-03, RNF-04 |
| ¿Qué pasa si el controlador central se desconecta? | RF-02, RF-07, RNF-03, RNF-04 |
| ¿Cómo decide el sistema sin intervención manual? | RF-03, RF-06, RNF-02 |
| ¿Cuándo y cómo interviene una persona? | RF-03, RF-04, RF-05 |
| ¿Quién prendió o apagó qué (persona o sistema)? | RF-08, RNF-07 |
| ¿Cómo convive el sistema nuevo con el legacy sin detener la planta? | RF-09, RNF-09 |
| ¿Cómo evaluar la nueva librería de eventos? | RF-10, RNF-02 |
