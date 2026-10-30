## Contexto

- La planta tiene **3 reactores**.
- Cada reactor tiene hasta **100 sensores** de temperatura.
- Cada sensor envía una lectura cada **0.05 s** en promedio.
- Hoy el sistema legacy controla la planta, pero casi todo se hace **a mano**. Eso hace perder tiempo a los empleados.
- El legacy **no se puede apagar** durante la modernización.
- Una decisión errónea puede causar un **desastre**.

**Objetivo:** automatizar el control que hoy es manual, sin detener la planta.

**Load del sistema:** son las lecturas de los sensores, no los usuarios.
3 reactores × 100 sensores × 20 lecturas/s = **6,000 lecturas/s** (2,000 por reactor).

**Equipos de la planta:** los sensores, la refrigeración principal, la refrigeración de respaldo y el sistema de apagado son **actores externos**. El sistema les envía comandos y recibe sus datos, pero no los diseña. La refrigeración de respaldo es un supuesto del equipo.

**Modos de control de un reactor:**
1. **Automático:** el Main Controller decide.
2. **Autónomo local:** si el Main Controller se desconecta, decide el controlador local del reactor.
3. **Manual:** el Jefe de Turno toma el control si la automatización falla.

---

## Requisitos Funcionales

### RF-01. Envío de lecturas de temperatura
- Cada sensor envía una lectura cada 0.05 s en promedio.
- Cada lectura indica el sensor, el reactor, la temperatura y la hora.
- El Main Controller recibe todas las lecturas.

### RF-02. Controlador local por reactor
- Cada reactor tiene un controlador local.
- El controlador local gestiona la refrigeración y las alertas de su reactor.
- Ejecuta los comandos del Main Controller.
- Rechaza cualquier comando que rompa los límites de seguridad.
- Recibe las lecturas de su reactor, para poder controlarlo solo si hace falta (RF-07).

### RF-03. Decisiones globales y comandos
- El Main Controller recibe los datos de los 3 reactores.
- Toma las decisiones globales de forma automática, según reglas configuradas.
- Envía los comandos: ajustar la refrigeración, activar la de respaldo o apagar un reactor.
- Si la automatización falla, el Jefe de Turno puede pasar un reactor a **modo manual**.
- En modo manual, se envían comandos desde el panel.
- Los límites de seguridad siguen activos en modo manual.
- El Jefe de Turno confirma el regreso al modo automático.

### RF-04. Alertas por temperatura
- El sistema alerta si la temperatura supera un umbral.
- Hay dos niveles de alerta: **Advertencia** y **Crítico**.
- También alerta cada acción automática. Ejemplo: "refrigeración del reactor 2 aumentada".
- Si la automatización no logra controlar un reactor, avisa al Jefe de Turno para que intervenga.

### RF-05. Panel para operadores
- El panel muestra en **tiempo real** la temperatura de cada reactor y sensor.
- Muestra las **alarmas** activas y permite reconocerlas.
- Muestra el modo de cada reactor: automático, autónomo local, manual o apagado.
- Genera **reportes** e historial.
- Cada usuario ve y hace solo lo que permite su rol.

### RF-06. Falla de un sensor o de la refrigeración
- El sistema detecta un sensor que no envía lecturas por más de 500 ms.
- También detecta un sensor que envía datos erróneos (fuera de rango, saltos imposibles o valor congelado).
- Ese sensor se deja de usar para decidir y se avisa a Mantenimiento.
- La falla de un sensor no apaga el reactor.
- Si un reactor queda con menos del 80% de sensores sanos, se hace un **apagado controlado**.
- Si la refrigeración principal falla, se activa la **refrigeración de respaldo**.
- Si no hay refrigeración o la temperatura es crítica, se hace un **shutdown de emergencia**.

### RF-07. Desconexión del Main Controller
- Si el Main Controller se desconecta, ningún reactor queda sin control.
- Cada controlador local pasa a controlar su reactor por su cuenta.
- El controlador local guarda sus acciones y las envía al registro al reconectarse.
- El Jefe de Turno confirma el regreso al control del Main Controller.
- Si falla un controlador local, su control también se recupera solo.

### RF-08. Registro y trazabilidad
- El sistema registra **todas las acciones y datos**.
- El registro responde **quién prendió o apagó qué**, y cuándo.
- En una acción automática, guarda qué controlador la hizo y qué regla la disparó.
- En una acción manual, guarda qué usuario la hizo y con qué resultado.
- También guarda los cambios de modo.
- El registro se puede consultar y exportar.

### RF-09. Convivencia con el legacy
- El sistema nuevo convive con el legacy sin detenerlo.
- Desde el inicio recibe los datos del legacy. Así, el monitoreo, las alarmas y el registro dejan de ser manuales.
- Antes de controlar un reactor, el sistema nuevo funciona en paralelo **sin actuar**.
- En ese periodo, sus decisiones se comparan con las reglas de operación.
- Toma el control **un reactor a la vez**, y se puede volver atrás.
- Los datos históricos del legacy se migran completos y marcados como de origen legacy.

### RF-10. Piloto de la nueva librería de eventos
- Se prueba la nueva librería de eventos como **piloto**.
- El piloto no puede poner en riesgo el control de la planta.
- Se adopta solo si mejora la latencia y da los mismos resultados que la actual.
- Se adopta de forma gradual y se puede revertir al instante.

---

## Requisitos No Funcionales

> Los valores son **supuestos** del equipo: aún no hay datos reales de la planta. Se ajustarán con pruebas de carga y monitoreo.

| ID | Atributo | Requisito | Supuesto / métrica |
|---|---|---|---|
| **RNF-01** | Disponibilidad | El control de los reactores debe estar disponible casi siempre. El panel y los reportes pueden tolerar caídas cortas. | Control de reactores ≥ 99.999% (≈ 5 min de caída al año). Panel y reportes ≥ 99.9%. Availability = 2xx / (2xx + 5xx). |
| **RNF-02** | Tiempo real | Las decisiones críticas se toman con latencia mínima. Los comandos críticos nunca esperan detrás de otros datos. | Lectura → decisión: P99 ≤ 100 ms (llegan lecturas nuevas cada 50 ms). Decisión → ejecución: P99 ≤ 100 ms. Lectura → panel: P99 ≤ 500 ms. |
| **RNF-03** | Tolerancia a fallos | Ningún componente es un punto único de falla. Nunca dos controladores dan órdenes distintas a un mismo reactor. La falla de un reactor no afecta a los otros. Ante una falla múltiple que impida decidir quién controla un reactor, el sistema no actúa sobre ese reactor y alerta. | Si falla el Main Controller, el control se recupera en ≤ 5 s. Refrigeración de respaldo activa en ≤ 1 s. 0 órdenes contradictorias en simulacros. |
| **RNF-04** | Seguridad operativa | Ante datos dudosos, el sistema no actúa y alerta. Ningún comando manual desactiva los límites de seguridad. El shutdown de emergencia es el último recurso. | 0 comandos ejecutados con datos dudosos. 0 shutdowns causados por la falla de un sensor. Shutdown de emergencia en ≤ 1 s. |
| **RNF-05** | Escalabilidad | El sistema soporta la carga actual y un crecimiento sin perder latencia. Se escala solo cuando hace falta. | Carga actual: 6,000 lecturas/s. Margen: 18,000 lecturas/s (3×) por si se agregan sensores o reactores. Escalar si sube el tráfico, baja la disponibilidad y sube la latencia. |
| **RNF-06** | Confiabilidad de datos | No se pierde ninguna lectura ni comando. Cada comando se confirma y nunca se ejecuta dos veces. Un comando vencido se descarta. Ante sobrecarga, se posterga el panel, nunca el control. | 0 lecturas perdidas. Reliability = 2xx / (2xx + 4xx + 5xx) ≥ 99.9% en comandos. |
| **RNF-07** | Trazabilidad | El registro no se puede modificar ni borrar. Si el registro falla, el control sigue funcionando. | 100% de acciones registradas. Se guarda ≥ 10 años (supuesto regulatorio nuclear). |
| **RNF-08** | Seguridad de acceso | Solo usuarios autenticados usan el sistema. Cada rol hace solo lo que le corresponde. La comunicación va cifrada. La red de control está aislada de la red de oficina. | 0 comandos sin usuario autenticado. Más de 3 intentos fallidos de login → bloqueo y alerta. |
| **RNF-09** | Modernización sin detener la planta | La planta nunca se detiene por la modernización. Cada paso se puede revertir. Los datos migrados llegan completos. | Downtime del control = 0. Rollback ≤ 1 min. 100% de registros migrados verificados. |
| **RNF-10** | Observabilidad | El equipo de plataforma ve el estado del sistema en dashboards y recibe alertas antes de que fallen los usuarios. | Dashboards con availability, reliability, tráfico y latencia. Incidente detectado en ≤ 1 min. |
