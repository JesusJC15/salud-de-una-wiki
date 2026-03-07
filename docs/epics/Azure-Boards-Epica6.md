# Azure Boards - Epica 6: Observabilidad y Analitica

## Proposito del documento
Este documento contiene la informacion necesaria para implementar la **Epica 6 - Observabilidad y Analitica** en Azure Boards, incluyendo epica, features, historias, criterios Gherkin, tareas y trazabilidad.

---

## 1. EPICA

| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Epic |
| **ID** | E6 |
| **Title** | Observabilidad y Analitica |
| **State** | Active |
| **Area Path** | SaludDeUna\\Backend / SaludDeUna\\DevOps / SaludDeUna\\Data |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Priority** | 1 - Critical |
| **Business Value** | Medir salud tecnica y valor de negocio para decisiones de producto basadas en evidencia |
| **Risk** | Medium - calidad de datos y consistencia de eventos pueden degradar KPIs |
| **Effort (Story Points total estimado)** | 12 SP |
| **Tags** | observability; logs; metrics; dashboard; kpi; must; sprint7 |
| **Start Date** | Sprint 7 |
| **Target Date** | Cierre Sprint 7 |

### Descripcion (campo Description)
```
La Epica 6 implementa la capa de observabilidad tecnica y de negocio del MVP.

Objetivos de negocio:
- Habilitar visibilidad operativa del sistema para reaccion temprana ante degradacion.
- Exponer los 4 KPIs obligatorios del curso en un dashboard de negocio.
- Sostener decisiones de priorizacion con evidencia cuantitativa.

Funcionalidades cubiertas:
- Logs estructurados con correlation_id y eventos minimos obligatorios.
- Metricas tecnicas de latencia, error rate, concurrencia y disponibilidad.
- Dashboards tecnico y de negocio con segmentaciones minimas.
- Alertas operativas para degradacion de SLO y KPIs.

Restricciones:
- No reemplaza observabilidad de infraestructura avanzada (solo nivel MVP).
- No incluye analitica historica de largo plazo ni BI corporativo.
- El dashboard depende de eventos consistentes emitidos por E1-E5.
```

### Criterios de aceptacion de la epica
```
- Cada flujo critico (auth, triage, chat, seguimiento, IA) emite logs estructurados.
- El dashboard tecnico muestra latencia P95, error rate, concurrencia y disponibilidad.
- El dashboard de negocio muestra los 4 KPIs obligatorios del curso.
- Las alertas criticas se disparan segun umbrales documentados.
- La data del dashboard es trazable a eventos y fuentes identificables.
```

### Acceptance Criteria - formato Gherkin (nivel epica)
```gherkin
Feature: Observabilidad integral de SaludDeUna

  Scenario: Equipo tecnico consulta panel de salud operativa
    Given que existen eventos y metricas del ultimo sprint
    When el equipo abre el dashboard tecnico
    Then visualiza latencia P95, error rate, concurrencia y disponibilidad
    And puede filtrar por especialidad y rango de fechas

  Scenario: Product owner consulta KPIs de negocio
    Given que existen consultas y seguimientos registrados
    When el product owner abre el dashboard de negocio
    Then visualiza los 4 KPIs obligatorios con valor y meta
    And identifica estado OK, WARNING o CRITICAL por KPI

  Scenario: Alerta por degradacion de rendimiento
    Given que la latencia P95 supera el umbral configurado
    When el sistema evalua reglas de alertamiento
    Then se genera una alerta critica
    And se registra evento de alerta con correlation_id
```

---

## 2. FEATURES

### Feature F6.1 - Logs estructurados y auditoria

| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Feature |
| **ID** | F6.1 |
| **Title** | Logs estructurados y auditoria |
| **Parent (Epic)** | E6 - Observabilidad y Analitica |
| **State** | Active |
| **Area Path** | SaludDeUna\\Backend / SaludDeUna\\DevOps |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Priority** | 1 - Critical |
| **MoSCoW** | Must |
| **Effort (SP)** | 4 SP |
| **Tags** | logs; audit; correlation-id; must |

#### Descripcion (campo Description)
```
Implementa el estandar de logs estructurados y auditoria para eventos criticos:
auth, triage, summary IA, chat, followup y validacion REThUS.
```

#### Acceptance Criteria del Feature
```gherkin
Feature: Logging estructurado

  Scenario: Emision de log por endpoint critico
    Given una solicitud a un endpoint critico
    When la solicitud termina
    Then se emite un log estructurado con timestamp, service, endpoint_or_event y correlation_id
    And incluye status_code y latency_ms
```

### Feature F6.2 - Metricas de concurrencia y rendimiento

| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Feature |
| **ID** | F6.2 |
| **Title** | Metricas de concurrencia y rendimiento |
| **Parent (Epic)** | E6 - Observabilidad y Analitica |
| **State** | Active |
| **Area Path** | SaludDeUna\\Backend / SaludDeUna\\DevOps |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Priority** | 1 - Critical |
| **MoSCoW** | Must |
| **Effort (SP)** | 4 SP |
| **Tags** | metrics; slos; perf; must |

#### Descripcion (campo Description)
```
Implementa instrumentacion de metricas de latencia, concurrencia, disponibilidad y error rate
con objetivos alineados a lineamientos del curso y metas internas.
```

#### Acceptance Criteria del Feature
```gherkin
Feature: Metricas operativas

  Scenario: Medicion de latencia P95
    Given trafico de endpoints criticos
    When se calcula la latencia del periodo
    Then el sistema registra P95 por endpoint
    And expone el valor en el panel tecnico
```

### Feature F6.3 - Dashboard tecnico y de negocio

| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Feature |
| **ID** | F6.3 |
| **Title** | Dashboard tecnico y de negocio |
| **Parent (Epic)** | E6 - Observabilidad y Analitica |
| **State** | Active |
| **Area Path** | SaludDeUna\\Web / SaludDeUna\\Data |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Priority** | 1 - Critical |
| **MoSCoW** | Must |
| **Effort (SP)** | 4 SP |
| **Tags** | dashboard; kpi; analytics; must |

#### Descripcion (campo Description)
```
Habilita dos vistas:
- Dashboard tecnico: latencia, errores, concurrencia, disponibilidad.
- Dashboard negocio: tiempo primera respuesta, utilidad resumen, red flags confirmadas, retencion 7 dias.
```

#### Acceptance Criteria del Feature
```gherkin
Feature: Dashboard dual tecnico/negocio

  Scenario: Visualizacion de KPIs de negocio
    Given datos consolidados del periodo
    When el usuario abre el dashboard de negocio
    Then visualiza los 4 KPIs obligatorios con meta y estado
    And puede segmentar por especialidad y fechas
```

---

## 3. HISTORIAS DE USUARIO

### HU-008 - Dashboard tecnico y de negocio
#### Descripcion completa (campo Description)
```
Como equipo de producto quiero visualizar metricas tecnicas y KPIs de negocio
para tomar decisiones basadas en datos y reaccionar ante degradaciones.
```

#### Criterios de Aceptacion - Gherkin
```gherkin
Feature: Dashboard de observabilidad

  Scenario: Panel tecnico disponible
    Given que existen metricas tecnicas del periodo
    When el equipo abre /v1/dashboard/technical
    Then ve latencia, error rate, concurrencia y disponibilidad

  Scenario: Panel negocio disponible
    Given que existen eventos de negocio consolidados
    When el equipo abre /v1/dashboard/business
    Then ve los 4 KPIs obligatorios con metas y estado
```

#### Definition of Ready (DoR) - Checklist
- [x] Historia en formato Como/Quiero/Para.
- [x] Criterios Gherkin principal y alterno.
- [x] Dependencias identificadas (E1-E5).
- [x] Metricas y alertas objetivo definidas.

#### Definition of Done (DoD) - Checklist
- [ ] Endpoints de dashboard implementados y probados.
- [ ] Datos de metricas y KPIs visibles en entorno de pruebas.
- [ ] Criterios Gherkin verificados.
- [ ] Wiki actualizada con contratos y definiciones.

### OBS-001 - Logging estructurado
#### Descripcion completa (campo Description)
```
Como equipo tecnico quiero registrar logs estructurados con correlation_id
para rastrear errores end-to-end y soportar auditoria.
```

### OBS-002 - Medicion de rendimiento
#### Descripcion completa (campo Description)
```
Como equipo tecnico quiero medir latencia, throughput y concurrencia
para controlar la salud operacional del sistema.
```

### OBS-003 - KPIs de negocio
#### Descripcion completa (campo Description)
```
Como product owner quiero visualizar 4 KPIs de negocio
para priorizar mejoras del producto con evidencia.
```

### OBS-004 - Alertamiento operativo
#### Descripcion completa (campo Description)
```
Como equipo de operacion quiero recibir alertas por degradacion
para responder antes de afectar usuarios.
```

---

## 4. TAREAS

### Tareas de HU-008 y OBS-001..004

#### T-008-01 - Definir esquema comun de logs estructurados
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Definir esquema comun de logs estructurados por servicio |
| **Parent (User Story)** | OBS-001 |
| **Assigned To** | Backend/DevOps |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 3 |

#### T-008-02 - Instrumentar logs en auth, triage, consulta, seguimiento e IA
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Instrumentar logs estructurados en flujos criticos |
| **Parent (User Story)** | OBS-001 |
| **Assigned To** | Backend |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 5 |

#### T-008-03 - Implementar agregacion de metricas tecnicas
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Implementar pipeline de metricas P95/error/concurrencia/disponibilidad |
| **Parent (User Story)** | OBS-002 |
| **Assigned To** | Backend/DevOps |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 6 |

#### T-008-04 - Implementar endpoint GET /v1/dashboard/technical
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Construir endpoint de dashboard tecnico |
| **Parent (User Story)** | HU-008 |
| **Assigned To** | Backend |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

#### T-008-05 - Implementar endpoint GET /v1/dashboard/business
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Construir endpoint de dashboard de negocio |
| **Parent (User Story)** | HU-008 |
| **Assigned To** | Backend/Data |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

#### T-008-06 - Construir vista web de dashboard tecnico
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Implementar pantalla Dashboard Tecnico en Next.js |
| **Parent (User Story)** | HU-008 |
| **Assigned To** | Frontend Web |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 5 |

#### T-008-07 - Construir vista web de dashboard de negocio
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Implementar pantalla Dashboard Negocio en Next.js |
| **Parent (User Story)** | HU-008 |
| **Assigned To** | Frontend Web |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 5 |

#### T-008-08 - Configurar reglas de alertamiento operativo
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Configurar alertas criticas y warning por SLO |
| **Parent (User Story)** | OBS-004 |
| **Assigned To** | DevOps |
| **State** | To Do |
| **Activity** | DevOps |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 3 |

#### T-008-09 - Pruebas de integracion de endpoints de dashboard
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Escribir pruebas de integracion para dashboards tecnico y negocio |
| **Parent (User Story)** | HU-008 |
| **Assigned To** | QA/Backend |
| **State** | To Do |
| **Activity** | Testing |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 4 |

#### T-008-10 - Documentacion tecnica de observabilidad en Wiki
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Actualizar Wiki con contratos de dashboard, metricas y alertas |
| **Parent (User Story)** | HU-008 |
| **Assigned To** | Backend/PO |
| **State** | To Do |
| **Activity** | Documentation |
| **Iteration Path** | SaludDeUna\\Sprint 7 |
| **Remaining Work (h)** | 2 |

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
EPIC: E6 - Observabilidad y Analitica (12 SP total)
|
|-- FEATURE: F6.1 - Logs estructurados y auditoria (4 SP)
|   |
|   |-- USER STORY: OBS-001 - Logging estructurado
|       |-- [ ] T-008-01 - Esquema comun de logs
|       |-- [ ] T-008-02 - Instrumentacion de logs
|
|-- FEATURE: F6.2 - Metricas de concurrencia y rendimiento (4 SP)
|   |
|   |-- USER STORY: OBS-002 - Medicion de rendimiento
|       |-- [ ] T-008-03 - Pipeline de metricas
|       |-- [ ] T-008-08 - Reglas de alertamiento
|
|-- FEATURE: F6.3 - Dashboard tecnico y de negocio (4 SP)
    |
    |-- USER STORY: HU-008 + OBS-003/004
        |-- [ ] T-008-04 - GET /v1/dashboard/technical
        |-- [ ] T-008-05 - GET /v1/dashboard/business
        |-- [ ] T-008-06 - Vista dashboard tecnico
        |-- [ ] T-008-07 - Vista dashboard negocio
        |-- [ ] T-008-09 - Pruebas integracion dashboard
        |-- [ ] T-008-10 - Documentacion tecnica
```

**Total horas estimadas Sprint 7 (Epica 6):** 41 horas  
**Total Story Points Epica 6:** 12 SP  
**Sprint objetivo:** Sprint 7

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|---|---|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` - Sprint 7 |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` - E6 |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` - HU-008 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` - HU-008 |
| Observabilidad y KPIs | `docs/wiki/09-Observabilidad-KPIs.md` |
| Riesgos | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` |

### API endpoints de esta epica
- `GET /v1/dashboard/technical` -> T-008-04
- `GET /v1/dashboard/business` -> T-008-05

### KPIs impactados por esta epica
- Tiempo a primera respuesta medica.
- Utilidad de resumen clinico.
- Red flags relevantes confirmadas.
- Retencion de seguimiento a 7 dias.

### Riesgos asociados
- R-007: calidad/consistencia de telemetria.
- R-002: latencia bajo carga concurrente.
