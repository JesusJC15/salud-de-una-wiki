# Azure Boards – Épica 6: Observabilidad y Analítica

## Propósito del documento
Este documento contiene toda la información necesaria para implementar la **Épica 6 – Observabilidad y Analítica** en Azure Boards, incluyendo la épica, sus features, historias de usuario con criterios Gherkin, y todas las tareas de desarrollo, pruebas y documentación asociadas. Cada sección indica los campos exactos que se deben completar al crear el ítem en Azure Boards.
## Propósito del documento
Este documento contiene toda la información necesaria para implementar la **Épica 6 – Observabilidad y Analítica** en Azure Boards, incluyendo la épica, sus features, historias de usuario con criterios Gherkin, y todas las tareas de desarrollo, pruebas y documentación asociadas. Cada sección indica los campos exactos que se deben completar al crear el ítem en Azure Boards.

---

## 1. ÉPICA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Epic |
| **ID**                 | E6 |
| **Title**              | Observabilidad y Analítica |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\DevOps / SaludDeUna\\Data / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Priority**           | 1 – Critical |
| **Business Value**     | Medir salud técnica y valor de negocio para decisiones de producto basadas en evidencia, y reaccionar antes de afectar la experiencia clínica del MVP |
| **Risk**               | Medium – La calidad de datos y la consistencia de eventos emitidos por E1-E5 pueden degradar dashboards y alertas (R-007) |
| **Effort (Story Points total estimado)** | 11 SP |
| **Tags**               | observability; logs; metrics; dashboard; kpi; alerting; must; sprint7 |
| **Start Date**         | Sprint 7 |
| **Target Date**        | Cierre Sprint 7 |

### Descripción (campo Description)

```
La Épica 6 implementa la capa de observabilidad técnica y analítica de negocio
del MVP SaludDeUna, consolidando trazabilidad operativa, alertamiento y
visualización de KPIs para el equipo técnico y de producto.

Objetivos de negocio:
- Habilitar visibilidad operativa del sistema para reaccionar tempranamente ante
  degradación de rendimiento, errores o caídas parciales.
- Exponer en un dashboard los 4 KPIs obligatorios del curso para soportar
  decisiones de priorización, validación de hipótesis y mejora continua.
- Asegurar que cada flujo crítico del producto (onboarding, triage, consulta,
  IA y seguimiento) quede trazable de punta a punta con correlation_id.

Funcionalidades cubiertas:
- Logs estructurados con campos mínimos obligatorios y correlación entre REST,
  WebSocket y jobs programados.
- Métricas técnicas de latencia P95, throughput, error rate, sesiones
  concurrentes, delivery de chat, tiempo de resumen IA y disponibilidad.
- Dashboard técnico con segmentación por endpoint, especialidad, prioridad de
  caso y rango de fechas.
- Dashboard de negocio con los 4 KPIs obligatorios: tiempo a primera respuesta
  médica, utilidad del resumen clínico, red flags relevantes confirmadas y
  retención de seguimiento a 7 días.
- Reglas de alertamiento con umbrales documentados y vista de incidentes en modo
  degradado cuando una fuente falle.
- Exportación semanal opcional del tablero como historia Could (HU-013).

Restricciones:
- No reemplaza observabilidad de infraestructura avanzada ni un stack corporativo
  completo de BI; el alcance es MVP y nivel aplicación.
- No se exponen datos sensibles del paciente en dashboards; user_id debe llegar
  anonimizado o pseudonimizado cuando aplique.
- El dashboard depende de que E1-E5 emitan eventos consistentes; si una fuente no
  emite o llega incompleta, el panel debe mostrar degradación controlada.
- La retención histórica y analítica exploratoria de largo plazo quedan fuera de
  alcance de este sprint.
```

### Criterios de aceptación de la épica

```
- Cada flujo crítico (auth, triage, chat, seguimiento, IA y validación REThUS)
  emite logs estructurados con correlation_id y campos mínimos definidos.
- El dashboard técnico muestra latencia P95, throughput, error rate,
  concurrencia, estado de WebSocket hub y disponibilidad.
- El dashboard de negocio muestra los 4 KPIs obligatorios con fórmula, meta,
  valor actual y estado OK/WARNING/CRITICAL.
- Las alertas críticas se disparan según umbrales documentados: P95 API > 1500 ms,
  error_rate > 2%, AI Summary Time > 15000 ms y availability <= 99% proyectada.
- Si una fuente de datos falla, el panel mantiene visibles las demás fuentes y
  explicita el estado degradado sin romper la experiencia de consulta.
- La data del dashboard es trazable a eventos, colecciones o fuentes identificables.
```

### Acceptance Criteria – formato Gherkin (nivel épica)

```gherkin
Feature: Observabilidad integral de SaludDeUna

  Scenario: Equipo técnico consulta panel de salud operativa
    Given que existen eventos y métricas del último sprint
    When el equipo abre el dashboard técnico
    Then visualiza latencia P95, throughput, error rate, concurrencia y disponibilidad
    And puede filtrar por endpoint, especialidad y rango de fechas

  Scenario: Product owner consulta KPIs de negocio
    Given que existen consultas y seguimientos registrados
    When el product owner abre el dashboard de negocio
    Then visualiza los 4 KPIs obligatorios con valor y meta
    And identifica estado OK, WARNING o CRITICAL por KPI

  Scenario: Dashboard en modo degradado por fuente incompleta
    Given que una fuente de métricas no responde
    When el panel intenta cargar los datos del periodo
    Then mantiene visibles las fuentes disponibles
    And muestra un indicador de degradación controlada

  Scenario: Alerta por degradación de rendimiento
    Given que la latencia P95 supera el umbral configurado durante 10 minutos
    When el sistema evalúa reglas de alertamiento
    Then se genera una alerta crítica
    And se registra evento de alerta con correlation_id
```

---

## 2. FEATURES

### Feature F6.1 – Logs Estructurados y Auditoría

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F6.1 |
| **Title**              | Logs Estructurados y Auditoría |
| **Parent (Epic)**      | E6 – Observabilidad y Analítica |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\DevOps |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 3 SP |
| **Tags**               | observability; logging; audit; correlation-id; must; sprint7 |

#### Descripción (campo Description)

```
Cubre la estandarización de logs de aplicación y auditoría end-to-end para todos
los flujos críticos del MVP, alineado con OBS-001.

Alcance funcional:
- Middleware/interceptor global en NestJS para enriquecer cada request/evento con:
    · timestamp
    · level
    · service
    · endpoint_or_event
    · correlation_id
    · user_id (anonimizado cuando aplique)
    · role
    · latency_ms
    · status_code
    · error_code (si aplica)
- Propagación del correlation_id entre endpoints REST, eventos WebSocket y jobs
  programados (scheduler de seguimiento, jobs de dashboard).
- Registro obligatorio de eventos mínimos:
    · login y cambios de sesión
    · ejecución de triage
    · cambio de prioridad por red flags
    · generación de resumen IA
    · mensajería de chat en tiempo real
    · creación y cierre de seguimiento
    · validación REThUS por admin
- Separación entre log técnico y evento de auditoría, evitando duplicados innecesarios.
- Política de anonimización para evitar que el dashboard o los logs operativos
  expongan PHI de forma directa.

Criterios de salida del feature:
- Todos los eventos mínimos definidos llegan con correlation_id.
- Los errores 4xx/5xx incluyen contexto suficiente para trazabilidad.
- Los eventos críticos son consultables por flujo y por rango de fechas.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Logs estructurados y auditoría técnica

  Scenario: Login emite log estructurado completo
    Given un paciente autenticado que inicia sesión
    When el backend procesa POST /v1/auth/login
    Then registra un log con correlation_id, latency_ms y status_code
    And el evento puede trazarse en el panel técnico

  Scenario: Evento WebSocket mantiene correlation_id
    Given una consulta activa con mensajería en tiempo real
    When paciente y médico intercambian mensajes por WebSocket
    Then cada mensaje registra endpoint_or_event, role y correlation_id
    And la correlación se conserva entre recepción y ACK lógico

  Scenario: Error de IA queda auditado
    Given una solicitud de resumen clínico al módulo IA
    When el proveedor externo falla
    Then el sistema registra error_code y contexto del flujo afectado
    And el evento queda disponible para análisis posterior
```

---

### Feature F6.2 – Métricas de Concurrencia y Rendimiento

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F6.2 |
| **Title**              | Métricas de Concurrencia y Rendimiento |
| **Parent (Epic)**      | E6 – Observabilidad y Analítica |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\DevOps / SaludDeUna\\Data |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 4 SP |
| **Tags**               | metrics; performance; concurrency; slo; alerts; must; sprint7 |

#### Descripción (campo Description)

```
Cubre la captura de métricas técnicas del sistema, agregaciones operativas y reglas
de alertamiento alineadas con OBS-002 y OBS-004.

Alcance funcional:
- Medición continua de:
    · P95 API Latency (< 1500 ms)
    · Chat Delivery Time (< 1500 ms)
    · AI Summary Time (< 15000 ms)
    · Error Rate (< 2.0%)
    · Concurrent Sessions (medición continua)
    · Availability (> 99.0%)
- Cálculo de throughput por minuto para endpoints y eventos críticos.
- Contador de sesiones concurrentes activas en WebSocket Hub.
- Métrica de delivery/ACK de chat para validar rendimiento real-time.
- Reglas de alertamiento:
    · crítica si P95 API > 1500 ms durante 10 minutos
    · crítica si error_rate > 2% durante 5 minutos
    · warning si AI Summary Time > 15000 ms en 3 muestras consecutivas
    · warning si availability <= 99% en ventana mensual proyectada
- Persistencia de snapshots agregados para consumo por dashboard.
- Modo degradado si un colector falla, registrando incidente sin detener el resto.

Criterios de salida del feature:
- Las métricas se actualizan automáticamente sin intervención manual.
- Las reglas de alertamiento generan eventos y quedan visibles en panel técnico.
- La prueba de carga combinada Sprint 7 cumple error rate < 2%.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Métricas técnicas y alertamiento operativo

  Scenario: Panel técnico muestra métricas de rendimiento
    Given que existen snapshots de métricas de las últimas 24 horas
    When el equipo solicita el resumen técnico
    Then ve P95 API Latency, Error Rate, Chat Delivery Time y Concurrent Sessions
    And cada métrica indica valor actual, tendencia y umbral

  Scenario: Alerta crítica por error rate
    Given que el error_rate supera 2% durante 5 minutos
    When el evaluador de alertas procesa la ventana actual
    Then crea una alerta crítica
    And la deja disponible para consulta por el equipo de operación

  Scenario: Fuente de métricas falla parcialmente
    Given que el colector de sesiones concurrentes no responde
    When el agregador genera el snapshot técnico
    Then marca la fuente como DEGRADED
    And conserva el resto de métricas disponibles
```

---

### Feature F6.3 – Dashboard Técnico y de Negocio

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F6.3 |
| **Title**              | Dashboard Técnico y de Negocio |
| **Parent (Epic)**      | E6 – Observabilidad y Analítica |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Data / SaludDeUna\\Web / SaludDeUna\\DevOps |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 4 SP |
| **Tags**               | dashboard; kpi; analytics; product; must; sprint7 |

#### Descripción (campo Description)

```
Cubre la construcción del dashboard oficial de observabilidad del MVP, incluyendo
panel técnico, panel de negocio y una historia Could de exportación semanal.

Alcance funcional:
- Dashboard técnico con:
    · latencia P95 por endpoint
    · throughput por minuto
    · error rate por servicio
    · usuarios concurrentes activos
    · estado del WebSocket hub
    · disponibilidad proyectada
- Dashboard de negocio con los 4 KPIs obligatorios:
    · tiempo a primera respuesta médica
    · utilidad de resumen clínico
    · red flags relevantes confirmadas
    · retención de seguimiento 7 días
- Segmentaciones mínimas:
    · por especialidad
    · por prioridad de caso
    · por rango de fechas (día/semana/sprint)
- Estado visual por tarjeta: OK, WARNING o CRITICAL.
- Vista de degradación controlada cuando alguna fuente no esté disponible.
- Exportación semanal de reporte como extensión Could vinculada a HU-013.

Criterios de salida del feature:
- El panel técnico y el de negocio están disponibles en web con filtros mínimos.
- Cada KPI muestra fórmula, meta, valor actual y fuente.
- El equipo puede distinguir rápidamente entre estado normal y degradado.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Dashboard técnico y de negocio

  Scenario: Product owner consulta KPIs con segmentación
    Given que existen datos agregados por especialidad y prioridad
    When el product owner filtra el dashboard por Odontología y casos HIGH
    Then el panel recalcula los KPIs del segmento seleccionado
    And mantiene visible la meta y el estado de cada KPI

  Scenario: Equipo técnico consulta panel con degradación controlada
    Given que una fuente de métricas está caída
    When el dashboard se renderiza
    Then muestra un banner de degradación controlada
    And sigue mostrando las tarjetas con datos disponibles

  Scenario: Visualización semanal de negocio
    Given que hay datos de consultas, triages, resúmenes y seguimientos
    When el usuario abre el dashboard de negocio del sprint
    Then ve los 4 KPIs obligatorios con su valor del periodo
    And puede identificar rápidamente los KPI en estado WARNING o CRITICAL
```

---

## 3. HISTORIAS DE USUARIO

### HU-008 – Dashboard Técnico y de Negocio

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-008 |
| **Title**              | Como equipo de producto quiero visualizar métricas técnicas y KPIs de negocio para tomar decisiones basadas en datos |
| **Parent (Feature)**   | F6.3 – Dashboard Técnico y de Negocio |
| **Related (Feature)**  | F6.1 – Logs Estructurados y Auditoría / F6.2 – Métricas de Concurrencia y Rendimiento |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\DevOps / SaludDeUna\\Data / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Story Points**       | 8 |
| **Risk**               | Medium – Dependencia alta de datos emitidos por E1-E5; si una fuente falla puede sesgar la lectura del tablero |
| **Dependencias**       | HU-006 / HU-007 / OBS-001..004 y emisión consistente de eventos de E1-E5 |
| **Tags**               | observability; dashboard; kpi; metrics; alerts; must; sprint7 |

#### Descripción completa (campo Description)

```
Como equipo de producto
Quiero visualizar métricas técnicas y KPIs de negocio en un tablero único
Para tomar decisiones basadas en datos, detectar degradación operativa y priorizar
mejoras del backlog con evidencia observable.

Contexto:
Hasta Sprint 6 el MVP ya cuenta con los flujos clave de onboarding, triage,
consulta en tiempo real, resumen IA y seguimiento. La épica E6 transforma esos
flujos en señales medibles para que el equipo no dependa de observaciones manuales.
La historia se apoya en el trabajo habilitador de OBS-001, OBS-002, OBS-003 y
OBS-004, que se trackean por separado en el backlog tecnico de observabilidad.

Flujo principal:
1. El backend instrumenta logs estructurados y métricas para REST, WebSocket y jobs.
2. Un agregador genera snapshots técnicos y de negocio para el periodo solicitado.
3. El dashboard técnico expone latencia, throughput, error rate, concurrencia,
   estado de hub WebSocket y disponibilidad.
4. El dashboard de negocio expone los 4 KPIs obligatorios con fórmula, meta,
   valor actual y estado.
5. El sistema evalúa reglas de alertamiento y genera incidentes visibles cuando
   un umbral se incumple.
6. Si una fuente cae, el tablero sigue disponible en modo degradado.

Restricciones funcionales:
- El tablero es de solo lectura; no permite editar métricas ni eventos fuente.
- Los datos deben poder filtrarse por especialidad, prioridad de caso y rango de fechas.
- user_id debe mostrarse anonimizado cuando llegue al almacenamiento analítico.
- Si una fuente está incompleta, el dashboard debe explicitarla como DEGRADED y
  no presentar datos inexistentes como si fueran válidos.
- La visualización de negocio no reemplaza análisis financiero ni BI corporativo.

Notas de UX / operación:
- Cada tarjeta muestra valor, meta, tendencia y color de estado (OK/WARNING/CRITICAL).
- El panel debe cargarse con métricas visibles en menos de 2 segundos en condiciones normales.
- Las alertas críticas deben verse destacadas al entrar al dashboard.
- Debe existir una indicación clara de fuente y timestamp de última actualización.
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Visualización de observabilidad y KPIs

  Scenario: Dashboard principal disponible
    Given datos de logs y métricas del periodo
    When el equipo abre el panel de observabilidad
    Then ve latencia, errores, concurrencia y 4 KPIs de negocio
    And cada KPI muestra valor, meta y estado

  Scenario: Dashboard alterno con fuente incompleta
    Given una fuente de métricas caída
    When el panel intenta cargar datos
    Then muestra estado de degradación controlada
    And mantiene visibles las fuentes disponibles

  Scenario: Alerta crítica visible por incumplimiento de SLO
    Given que el error_rate supera 2% durante 5 minutos
    When el equipo abre el panel técnico
    Then ve una alerta crítica activa asociada al servicio afectado
    And puede identificar el umbral incumplido

  Scenario: Product owner filtra KPI por especialidad y prioridad
    Given que existen datos segmentados del sprint
    When filtra el dashboard por Medicina General y prioridad HIGH
    Then el sistema recalcula el tablero para ese segmento
    And mantiene visible la fórmula y meta de cada KPI
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y features definidos (E6, F6.1, F6.2, F6.3)
- [x] Prioridad MoSCoW asignada (Must)
- [x] Criterios Gherkin documentados (principal y alterno)
- [x] Dependencias identificadas (HU-006, HU-007, eventos de E1-E5)
- [x] Estimación acordada (8 SP)
- [x] Métricas, KPIs y umbrales definidos en documento de observabilidad
- [x] Riesgos anotados (R-007 y calidad de datos)
- [x] Criterios de seguridad definidos (anonimización, acceso solo a equipo interno)
- [x] Segmentaciones mínimas documentadas (especialidad, prioridad, fechas)
- [x] Responsable de validación funcional asignado

#### Definition of Done (DoD) – Checklist

- [ ] Logging estructurado global implementado con correlation_id y campos mínimos
- [ ] Eventos mínimos de auth, triage, chat, seguimiento, IA y REThUS instrumentados
- [ ] Snapshots de métricas técnicas implementados y persistidos
- [ ] Reglas de alertamiento implementadas con umbrales documentados
- [ ] Endpoint `GET /v1/dashboard/technical` implementado
- [ ] Endpoint `GET /v1/dashboard/business` implementado
- [ ] Endpoint `GET /v1/observability/alerts` implementado
- [ ] Dashboard web técnico y de negocio implementado con filtros mínimos
- [ ] Estado visual OK/WARNING/CRITICAL implementado por tarjeta
- [ ] Modo degradado implementado cuando falle una fuente
- [ ] Código revisado por al menos un par
- [ ] Pruebas unitarias e integración de collectors, agregadores y endpoints en verde
- [ ] Prueba de carga combinada Sprint 7 ejecutada con error rate < 2%
- [ ] KPIs obligatorios visibles con fórmula, meta, valor y fuente
- [ ] Log de auditoría registra la generación de alertas y consultas al dashboard
- [ ] Documentación técnica actualizada en Wiki
- [ ] Demo funcional aprobada por el equipo
- [ ] Historia pasada a Done con evidencia enlazada (PR + test results + capturas)

---

### HU-013 – Exportación de Reporte Semanal del Tablero

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-013 |
| **Title**              | Como equipo de datos quiero exportar el tablero a un reporte semanal para seguir tendencias de valor y riesgo |
| **Parent (Feature)**   | F6.3 – Dashboard Técnico y de Negocio |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Data / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 9 |
| **Estado de planificación** | Candidata – Sprint 9 |
| **Priority**           | 3 – Medium |
| **MoSCoW**             | Could |
| **Story Points**       | 3 |
| **Risk**               | Low – Historia opcional; no debe comprometer la entrega Must del tablero principal |
| **Dependencias**       | HU-008 |
| **Tags**               | analytics; reporting; export; could; sprint9 |

#### Descripción completa (campo Description)

```
Como equipo de datos
Quiero exportar el tablero de observabilidad a un reporte semanal
Para seguir tendencias de valor y riesgo fuera del panel interactivo y facilitar
la revisión del equipo en sesiones semanales.

Contexto:
Una vez disponible HU-008, el equipo puede requerir un snapshot semanal reutilizable
para revisión, archivo o soporte de retrospectivas. Esta historia es Could, queda
propuesta como candidata para Sprint 9 y solo debe ejecutarse si las historias Must
comprometidas ya están estables.

Flujo principal:
1. El usuario selecciona un rango semanal desde el dashboard.
2. El sistema genera un snapshot con KPIs, alertas activas y métricas técnicas clave.
3. El reporte se descarga en un formato simple del MVP (CSV + resumen Markdown o PDF ligero).
4. El archivo queda identificable por semana, fecha de generación y filtros aplicados.

Restricciones:
- No incluye distribución automática por correo en este sprint.
- No reemplaza el dashboard interactivo; es una salida resumida para consulta offline.
- Solo puede usar datos ya calculados por HU-008; no debe recalcular todo el pipeline.
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Exportación semanal del tablero

  Scenario: Exportación principal de reporte semanal
    Given un dashboard con datos agregados de una semana cerrada
    When el usuario solicita exportar el reporte semanal
    Then el sistema genera un archivo con métricas técnicas, KPIs y alertas del periodo
    And el nombre del archivo identifica la semana exportada

  Scenario: Exportación con fuente degradada
    Given una fuente marcada como DEGRADED en el dashboard
    When el usuario exporta el reporte semanal
    Then el archivo incluye una nota de degradación del periodo
    And no omite silenciosamente la fuente incompleta
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y feature definidos (E6, F6.3)
- [x] Prioridad MoSCoW asignada (Could)
- [x] Dependencia de HU-008 documentada
- [x] Estimación acordada (3 SP)
- [x] Formato de salida inicial definido (CSV + resumen)
- [x] Criterios Gherkin documentados
- [x] Restricción de no afectar el camino crítico Must explícita

#### Definition of Done (DoD) – Checklist

- [ ] Servicio de snapshot semanal implementado usando datos agregados existentes
- [ ] Endpoint `GET /v1/observability/reports/weekly/export` implementado
- [ ] Acción de exportación visible en el dashboard web
- [ ] Archivo generado con KPIs, métricas y alertas del periodo
- [ ] Estado DEGRADED reflejado en el reporte cuando aplique
- [ ] Pruebas unitarias e integración del flujo de exportación en verde
- [ ] Documentación técnica actualizada en Wiki
- [ ] Historia pasada a Done con evidencia enlazada

---

## 4. TAREAS

### Tareas de HU-008 – Dashboard Técnico y de Negocio

---

#### T-008-01 – Interceptor global de logging estructurado

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar interceptor global de logging estructurado con correlation_id |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Crear un interceptor/middleware global en NestJS para enriquecer cada request con
los campos mínimos de observabilidad:
- timestamp
- level
- service
- endpoint_or_event
- correlation_id
- user_id (anonimizado)
- role
- latency_ms
- status_code
- error_code (si aplica)

Reglas:
- Si el cliente envía x-correlation-id, reutilizarlo; si no, generarlo.
- Propagar el correlation_id a request context, eventos WebSocket y jobs.
- Usar JSON como formato estándar de salida.

Ubicar en: apps/api/src/observability/logging/
```

**Criterios de aceptación de la tarea:**
- Cada request HTTP queda registrado en formato estructurado JSON.
- correlation_id queda disponible para servicios y controladores descendentes.
- Los errores 4xx/5xx incluyen status_code y error_code cuando aplique.

---

#### T-008-02 – Instrumentación de eventos mínimos obligatorios

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Instrumentar eventos mínimos de auth, triage, chat, seguimiento, IA y REThUS |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Agregar puntos de instrumentación explícitos en los módulos existentes:
- AuthModule: login y refresh de sesión
- TriageModule: inicio/completitud de triage y cambio de prioridad
- ConsultationsModule: apertura/cierre de consulta
- ConsultationGateway: envío/recepción/ACK de mensajes chat
- AI module: generación de resumen clínico
- FollowupsModule: creación/completitud de seguimiento
- AdminModule: validación REThUS

Cada evento debe emitir endpoint_or_event consistente y reutilizar correlation_id.

Ubicar en: apps/api/src/**/services y gateway existentes
```

**Criterios de aceptación de la tarea:**
- Todos los eventos mínimos aparecen en logs estructurados.
- Los nombres de endpoint_or_event son consistentes y documentados.
- Se puede trazar un flujo completo auth -> triage -> consulta -> seguimiento.

---

#### T-008-03 – Collector de métricas API, error rate y disponibilidad

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar collector para latencia API, throughput, error rate y disponibilidad |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Construir el collector principal de métricas técnicas para endpoints REST:
- calcular P95 API Latency por endpoint crítico
- calcular throughput por minuto
- calcular Error Rate = 5xx / total respuestas
- calcular availability proyectada a partir de health checks y errores severos

Persistir snapshots agregados con granularidad de 1 minuto y resumen por día/sprint.

Ubicar en: apps/api/src/observability/metrics/
```

**Criterios de aceptación de la tarea:**
- Existen snapshots consultables por rango de tiempo.
- P95 API, throughput y error rate se calculan correctamente.
- Availability queda disponible como métrica proyectada del periodo.

---

#### T-008-04 – Métricas de WebSocket, concurrencia y resumen IA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Medir chat delivery, sesiones concurrentes y AI Summary Time |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Agregar collectors específicos para componentes no REST:
- Chat Delivery Time: tiempo entre recepción y ACK lógico de mensaje
- Concurrent Sessions: contador de sesiones activas simultáneas en WebSocket hub
- AI Summary Time: duración de generación del resumen clínico

Reglas:
- registrar snapshots continuos para consumo de panel técnico
- si AI Summary Time > 15000 ms, marcar muestra como candidate_warning
- si el hub no puede reportar concurrencia, marcar fuente DEGRADED

Ubicar en: apps/api/src/consultations/gateways/
         apps/api/src/ai/
         apps/api/src/observability/metrics/
```

**Criterios de aceptación de la tarea:**
- Chat Delivery Time queda disponible como métrica agregada.
- Concurrent Sessions refleja sesiones activas del hub.
- AI Summary Time registra duración real por solicitud.

---

#### T-008-05 – Motor de alertamiento operativo

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar reglas de alertamiento para SLO y degradación |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Construir un evaluador de alertas que procese snapshots y genere incidentes:
- crítica si P95 API > 1500 ms durante 10 minutos
- crítica si error_rate > 2% durante 5 minutos
- warning si AI Summary Time > 15000 ms en 3 muestras consecutivas
- warning si availability <= 99% en ventana mensual proyectada

El evaluador debe:
- crear registros de alerta con severidad, fuente, correlation_id y threshold
- evitar duplicados activos del mismo incidente
- registrar estado RESOLVED cuando el valor vuelva a rango normal

Ubicar en: apps/api/src/observability/alerts/
```

**Criterios de aceptación de la tarea:**
- Las alertas se crean con severidad correcta y umbral incumplido.
- No se duplican alertas activas del mismo incidente.
- Las alertas pueden pasar a estado RESOLVED automáticamente.

---

#### T-008-06 – Endpoints del dashboard técnico y alertas

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoints GET del dashboard técnico y listado de alertas |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar los endpoints backend para consumo del panel técnico:
- GET /v1/dashboard/technical
- GET /v1/observability/alerts

Filtros soportados:
- specialty?
- priority?
- from?
- to?

Respuesta esperada technical:
{
  metrics: [{ key, value, target, state, trend, source }],
  lastUpdatedAt,
  degradedSources: []
}

Ubicar en: apps/api/src/observability/observability.controller.ts
         apps/api/src/observability/observability.service.ts
```

**Criterios de aceptación de la tarea:**
- Los endpoints retornan datos agregados del periodo solicitado.
- degradedSources refleja colectores caídos o incompletos.
- Los filtros aplican correctamente sobre el resumen técnico.

---

#### T-008-07 – Endpoint del dashboard de negocio

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint GET del dashboard de negocio con 4 KPIs obligatorios |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar GET /v1/dashboard/business con cálculo de:
- Tiempo a primera respuesta médica
- Utilidad de resumen clínico
- Red flags relevantes confirmadas
- Retención de seguimiento 7 días

Cada KPI debe incluir:
- key
- formula
- value
- target
- state
- source
- segmentApplied

Segmentaciones mínimas:
- especialidad
- prioridad de caso
- rango de fechas (día/semana/sprint)

Ubicar en: apps/api/src/observability/business-kpi.service.ts
```

**Criterios de aceptación de la tarea:**
- Los 4 KPIs se calculan con las fórmulas del documento de observabilidad.
- Cada KPI expone target, value y state.
- El endpoint soporta segmentaciones mínimas del sprint.

---

#### T-008-08 – Implementación del dashboard web

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar dashboard web técnico y de negocio con filtros y estados visuales |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Desarrollador Web |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 5 |

**Descripción:**
```
Construir la UI web del dashboard en Next.js o panel web del proyecto:
- vista técnica con tarjetas de métricas y alertas activas
- vista negocio con 4 KPIs obligatorios
- filtros por especialidad, prioridad y rango de fechas
- timestamp de última actualización
- banner de degradación controlada si degradedSources no está vacío

Estados visuales:
- OK: verde
- WARNING: amarillo
- CRITICAL: rojo

Ubicar en: apps/web/src/app/observability/ o ruta equivalente del panel
```

**Criterios de aceptación de la tarea:**
- La UI muestra métricas técnicas y KPIs de negocio en vistas diferenciadas.
- Los filtros actualizan correctamente la información del tablero.
- El banner DEGRADED aparece cuando una fuente falla.

---

#### T-008-09 – Pruebas integrales y carga combinada Sprint 7

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Ejecutar pruebas unitarias, integración y carga combinada para observabilidad |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | QA / Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Ejecutar validación integral de la capa de observabilidad:
- pruebas unitarias de collectors, agregadores y alert evaluator
- pruebas de integración de endpoints technical/business/alerts
- prueba de carga combinada chat + resumen IA + dashboard, según plan Sprint 7

Objetivos:
- error rate < 2%
- panel responde con datos agregados consistentes
- modo degradado visible cuando una fuente simulada falla
```

**Criterios de aceptación de la tarea:**
- Tests unitarios e integración quedan en verde.
- La prueba de carga Sprint 7 documenta error rate < 2%.
- Se verifica comportamiento de degradación controlada.

---

#### T-008-10 – Documentación técnica de observabilidad en Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar contratos, métricas, alertas y dashboard de observabilidad en la Wiki |
| **Parent (User Story)**| HU-008 |
| **Assigned To**        | Tech Lead / Backend |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Actualizar la documentación técnica del proyecto con:
- endpoints del dashboard
- contrato de snapshots de métricas
- reglas de alertamiento
- definición de KPI y fuentes
- ejemplos de log estructurado

Referenciar docs/wiki/09-Observabilidad-KPIs.md y el plan de Sprint 7.
```

**Criterios de aceptación de la tarea:**
- La wiki contiene endpoints, ejemplos y umbrales actualizados.
- Los nombres de métricas y KPIs coinciden con implementación y tablero.
- El equipo puede usar la documentación para demo y soporte del sprint.

---

### Tareas de HU-013 – Exportación de Reporte Semanal del Tablero

> **Nota:** Las tareas de HU-013 quedan como trabajo candidato de Sprint 9 y solo se ejecutan si HU-008 y el hardening de observabilidad ya están estables.

---

#### T-013-01 – Servicio de snapshot semanal exportable

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar servicio de snapshot semanal para exportación de tablero |
| **Parent (User Story)**| HU-013 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 9 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Construir un servicio que tome el resultado agregado de HU-008 y genere un snapshot
semanal reutilizable para exportación.

Debe incluir:
- métricas técnicas clave
- 4 KPIs obligatorios
- alertas activas o resueltas del periodo
- nota de degradación si alguna fuente estuvo en estado DEGRADED
```

**Criterios de aceptación de la tarea:**
- El snapshot reutiliza datos agregados existentes.
- Incluye degradación del periodo cuando aplique.
- No recalcula innecesariamente todo el pipeline analítico.

---

#### T-013-02 – Endpoint de exportación semanal

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint GET /v1/observability/reports/weekly/export |
| **Parent (User Story)**| HU-013 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 9 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Implementar un endpoint para descargar el snapshot semanal en formato simple del MVP.

Ruta:
- GET /v1/observability/reports/weekly/export

Formato inicial:
- CSV para datos tabulares
- resumen Markdown o PDF ligero para lectura ejecutiva
```

**Criterios de aceptación de la tarea:**
- El endpoint retorna un archivo identificable por semana.
- El archivo contiene KPIs, métricas y alertas del periodo.
- La exportación indica si existió degradación de fuentes.

---

#### T-013-03 – Acción de exportación en dashboard web

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Agregar acción de descarga de reporte semanal en el dashboard web |
| **Parent (User Story)**| HU-013 |
| **Assigned To**        | Desarrollador Web |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 9 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Agregar al dashboard una acción visible para exportar el reporte semanal del rango
seleccionado, reutilizando los filtros activos.

Ubicar en: apps/web/src/app/observability/ o ruta equivalente del panel
```

**Criterios de aceptación de la tarea:**
- El usuario puede descargar el reporte sin salir del dashboard.
- El archivo respeta los filtros del periodo seleccionado.
- La UI informa si la exportación contiene fuentes degradadas.

---

#### T-013-04 – Pruebas y documentación de exportación

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Validar exportación semanal y documentar uso del reporte |
| **Parent (User Story)**| HU-013 |
| **Assigned To**        | QA / Tech Lead |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 9 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Ejecutar pruebas del flujo Could de exportación semanal:
- validación del archivo generado
- validación de filtros aplicados
- validación de nota de degradación
- actualización de guía de uso en Wiki
```

**Criterios de aceptación de la tarea:**
- La exportación se descarga correctamente con el contenido esperado.
- La documentación explica cuándo usar el reporte y sus limitaciones.
- La historia queda lista para demo si el sprint lo permite.

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
📌 EPIC: E6 – Observabilidad y Analítica (11 SP total)
│
├── 🔷 FEATURE: F6.1 – Logs Estructurados y Auditoría (3 SP)
├── 🔷 FEATURE: F6.2 – Métricas de Concurrencia y Rendimiento (4 SP)
└── 🔷 FEATURE: F6.3 – Dashboard Técnico y de Negocio (4 SP)
    │
    ├── 📖 USER STORY: HU-008 – Como equipo de producto quiero visualizar métricas técnicas y KPIs de negocio (8 SP)
    │   ├── [ ] T-008-01 – Interceptor global de logging estructurado (3h)
    │   ├── [ ] T-008-02 – Instrumentación de eventos mínimos obligatorios (4h)
    │   ├── [ ] T-008-03 – Collector API, error rate y availability (4h)
    │   ├── [ ] T-008-04 – Métricas WS, concurrencia y resumen IA (4h)
    │   ├── [ ] T-008-05 – Motor de alertamiento operativo (3h)
    │   ├── [ ] T-008-06 – Endpoints dashboard técnico y alertas (4h)
    │   ├── [ ] T-008-07 – Endpoint dashboard de negocio (4h)
    │   ├── [ ] T-008-08 – Dashboard web con filtros y estados visuales (5h)
    │   ├── [ ] T-008-09 – Pruebas integrales y carga combinada Sprint 7 (4h)
    │   └── [ ] T-008-10 – Documentación técnica en Wiki (2h)
    │
    └── 📖 USER STORY: HU-013 – Como equipo de datos quiero exportar el tablero a un reporte semanal (3 SP, candidata Sprint 9)
        ├── [ ] T-013-01 – Servicio de snapshot semanal exportable (3h)
      ├── [ ] T-013-02 – Endpoint GET /v1/observability/reports/weekly/export (2h)
        ├── [ ] T-013-03 – Acción de exportación en dashboard web (2h)
        └── [ ] T-013-04 – Pruebas y documentación de exportación (2h)
```

  **Total horas estimadas Épica 6:** 46 horas de trabajo  
**Total Story Points Épica 6:** 11 SP (HU-008: 8 SP + HU-013: 3 SP)  
  **Sprint objetivo:** Sprint 7 (HU-008 comprometida) / Sprint 9 (HU-013 candidata)

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|-----------|-----------|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` – Sprint 7 y roadmap del MVP |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` – Actividad 5: Analítica y operación |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` – HU-008, HU-013 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` – HU-008, HU-013 |
| KPIs y observabilidad | `docs/wiki/09-Observabilidad-KPIs.md` |
| Plan de sprints | `docs/wiki/11-Plan-Sprints-0-a-9.md` – Sprint 7 |
| Riesgos | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` |
| Cumplimiento lineamientos | `docs/wiki/15-Cumplimiento-Lineamientos-2026-1.md` |

### API endpoints de esta épica (contratos propuestos del MVP)
- `GET /v1/dashboard/technical` → T-008-06
- `GET /v1/dashboard/business` → T-008-07
- `GET /v1/observability/alerts` → T-008-06
- `GET /v1/observability/reports/weekly/export` → T-013-02

### Eventos y fuentes instrumentadas por esta épica
- `POST /v1/auth/login` y cambios de sesión (E1)
- ejecución de triage y red flags (E2)
- mensajería y estado de consulta en tiempo real (E3)
- generación de resumen clínico IA (E4)
- creación/completitud de followup y re-priorización (E5)
- validación REThUS por admin (E1)

### KPIs y SLOs impactados por esta épica
- **P95 API Latency < 1500 ms**: visible y alertable desde panel técnico.
- **Chat Delivery Time < 1500 ms**: monitorea salud del flujo real-time.
- **AI Summary Time < 15000 ms**: controla desempeño del módulo IA.
- **Error Rate < 2.0%**: criterio crítico de estabilidad técnica.
- **Availability > 99.0%**: SLO operativo del MVP.
- **Tiempo a primera respuesta médica**: KPI de continuidad y oportunidad de atención.
- **Utilidad de resumen clínico**: KPI de valor del módulo IA.
- **Red flags relevantes confirmadas**: KPI de calidad del triage.
- **Retención de seguimiento 7 días**: KPI de continuidad post-consulta.

### Riesgos asociados
- **R-007 (Caída parcial de servicios dashboard)**: Mitigar con modo degradado, caché temporal y alertamiento.
- **R-002 (Latencia alta en chat real-time)**: Impacta directamente métricas y alertas del panel técnico.
- **R-003 (Resumen IA inestable)**: Puede degradar AI Summary Time y sesgar KPI de utilidad del resumen.
- **R-006 (Exposición de datos sensibles)**: Requiere anonimización de user_id y disciplina de logging.
