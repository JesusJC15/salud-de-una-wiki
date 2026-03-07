# Azure Boards - Epica 7: Monetizacion Simulada y Gobierno

## Proposito del documento
Este documento contiene la informacion necesaria para implementar la **Epica 7 - Monetizacion Simulada y Gobierno** en Azure Boards, incluyendo epica, features, historias, criterios Gherkin, tareas y trazabilidad.

---

## 1. EPICA

| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Epic |
| **ID** | E7 |
| **Title** | Monetizacion Simulada y Gobierno |
| **State** | Active |
| **Area Path** | SaludDeUna\\Backend / SaludDeUna\\Web / SaludDeUna\\Producto |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Priority** | 2 - High |
| **Business Value** | Validar viabilidad de modelo y control de contenido medico para operacion del MVP |
| **Risk** | Medium - sobrealcance funcional en checkout y banco de conocimiento |
| **Effort (Story Points total estimado)** | 10 SP |
| **Tags** | monetizacion; checkout; governance; knowledge-base; sprint8 |
| **Start Date** | Sprint 8 |
| **Target Date** | Cierre Sprint 8 |

### Descripcion (campo Description)
```
La Epica 7 implementa el flujo de monetizacion simulada y el gobierno del banco
de conocimiento clinico validado.

Objetivos de negocio:
- Validar la conversion del flujo de compra simulada (consulta/plan) sin pago real.
- Estandarizar contenido reutilizable validado por medicos para reducir repeticion.
- Aportar evidencia para decisiones de modelo comercial del MVP.

Funcionalidades cubiertas:
- Checkout simulado (sin pasarela real): seleccion de plan/consulta, confirmacion,
  registro de transaccion simulada y habilitacion de beneficio.
- Banco de conocimiento con articulos clinicos y flujo de aprobacion medica.
- Endpoint de consulta de articulos validados y endpoint de aprobacion por medico.

Restricciones:
- No se integra pasarela de pago productiva.
- No se realiza facturacion real.
- El banco de conocimiento no reemplaza juicio clinico.
```

### Criterios de aceptacion de la epica
```
- El paciente puede simular compra y el sistema registra transaccion simulada.
- Si el paciente cancela, no se registra transaccion.
- El beneficio del plan/consulta se activa solo con simulacion exitosa.
- El medico puede aprobar contenido del banco de conocimiento.
- Solo contenido aprobado aparece en endpoint publico de consulta.
```

### Acceptance Criteria - formato Gherkin (nivel epica)
```gherkin
Feature: Monetizacion simulada y gobierno de contenido

  Scenario: Simulacion de compra exitosa
    Given un paciente autenticado
    When selecciona un plan y confirma checkout simulado
    Then el sistema registra la transaccion simulada como SUCCESS
    And habilita el beneficio contratado

  Scenario: Cancelacion de checkout simulado
    Given un paciente en pantalla de checkout
    When cancela la operacion
    Then el sistema no registra transaccion
    And el estado del plan permanece sin cambios

  Scenario: Aprobacion de articulo por medico
    Given un articulo en estado PENDING
    When un medico autorizado lo aprueba
    Then el articulo cambia a estado APPROVED
    And queda disponible en el endpoint publico de conocimiento
```

---

## 2. FEATURES

### Feature F7.1 - Flujo de pago simulado

| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Feature |
| **ID** | F7.1 |
| **Title** | Flujo de pago simulado |
| **Parent (Epic)** | E7 - Monetizacion Simulada y Gobierno |
| **State** | Active |
| **Area Path** | SaludDeUna\\Backend / SaludDeUna\\Mobile |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Priority** | 2 - High |
| **MoSCoW** | Must |
| **Effort (SP)** | 5 SP |
| **Tags** | checkout; billing; simulated; must |

#### Descripcion (campo Description)
```
Permite al paciente simular la compra de una consulta o plan sin pasarela real.
Incluye registro de estado de transaccion simulada y activacion de beneficio.
```

#### Acceptance Criteria del Feature
```gherkin
Feature: Checkout simulado

  Scenario: Registro de transaccion simulada
    Given un paciente autenticado con opcion de plan seleccionada
    When confirma simulacion de compra
    Then el sistema crea una transaccion simulada exitosa
    And asocia el beneficio al paciente
```

### Feature F7.2 - Banco de conocimiento validado

| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Feature |
| **ID** | F7.2 |
| **Title** | Banco de conocimiento validado |
| **Parent (Epic)** | E7 - Monetizacion Simulada y Gobierno |
| **State** | Active |
| **Area Path** | SaludDeUna\\Backend / SaludDeUna\\Web |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Priority** | 3 - Medium |
| **MoSCoW** | Should |
| **Effort (SP)** | 5 SP |
| **Tags** | knowledge; governance; content; should |

#### Descripcion (campo Description)
```
Habilita consulta de articulos de conocimiento clinico y flujo de aprobacion medica.
Solo contenido aprobado se expone para reutilizacion operacional.
```

#### Acceptance Criteria del Feature
```gherkin
Feature: Gobierno de conocimiento clinico

  Scenario: Publicacion de contenido aprobado
    Given un articulo en estado APPROVED
    When un cliente consulta /v1/knowledge/articles
    Then el articulo aparece en resultados
    And no aparecen articulos en estado PENDING o REJECTED
```

---

## 3. HISTORIAS DE USUARIO

### HU-010 - Flujo de monetizacion simulado
#### Descripcion completa (campo Description)
```
Como paciente quiero simular compra de consulta o plan
para validar el flujo de monetizacion del producto.
```

#### Criterios de Aceptacion - Gherkin
```gherkin
Feature: Simulacion de checkout

  Scenario: Compra simulada confirmada
    Given un paciente autenticado en checkout
    When confirma simulacion
    Then se registra transaccion simulada SUCCESS
    And se habilita beneficio

  Scenario: Compra simulada cancelada
    Given un paciente en checkout
    When cancela la simulacion
    Then no se registra transaccion
    And se mantiene estado previo
```

#### Definition of Ready (DoR) - Checklist
- [x] Historia escrita en formato Como/Quiero/Para.
- [x] Criterios Gherkin documentados.
- [x] Dependencias identificadas (HU-001).
- [x] Estimacion acordada por el equipo.

#### Definition of Done (DoD) - Checklist
- [ ] Endpoint de checkout simulado implementado.
- [ ] Estado de transaccion persistido y consultable.
- [ ] Criterios Gherkin verificados.
- [ ] Documentacion actualizada en Wiki.

### HU-011 - Banco de conocimiento validado por pares
#### Descripcion completa (campo Description)
```
Como medico quiero reutilizar respuestas validadas por pares
para reducir consultas repetitivas y mejorar consistencia.
```

#### Criterios de Aceptacion - Gherkin
```gherkin
Feature: Base de conocimiento validada

  Scenario: Medico aprueba articulo
    Given un articulo en estado PENDING
    When el medico lo aprueba
    Then cambia a APPROVED
    And queda disponible para consulta
```

---

## 4. TAREAS

### Tareas de HU-010 - Flujo de monetizacion simulado

#### T-010-01 - Disenar modelo de transaccion simulada
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Disenar modelo BillingSimulationTransaction en MongoDB |
| **Parent (User Story)** | HU-010 |
| **Assigned To** | Backend |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 2 |

#### T-010-02 - Implementar endpoint POST /v1/billing/simulate-checkout
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Implementar endpoint de checkout simulado |
| **Parent (User Story)** | HU-010 |
| **Assigned To** | Backend |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 4 |

#### T-010-03 - Implementar pantalla de checkout simulado en app movil
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Construir UI de seleccion y confirmacion de plan simulado |
| **Parent (User Story)** | HU-010 |
| **Assigned To** | Mobile |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 4 |

#### T-010-04 - Pruebas de integracion de checkout simulado
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Probar escenarios SUCCESS y CANCELLED del checkout simulado |
| **Parent (User Story)** | HU-010 |
| **Assigned To** | QA/Backend |
| **State** | To Do |
| **Activity** | Testing |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 3 |

### Tareas de HU-011 - Banco de conocimiento

#### T-011-01 - Disenar modelo KnowledgeArticle
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Disenar modelo KnowledgeArticle con estados PENDING/APPROVED/REJECTED |
| **Parent (User Story)** | HU-011 |
| **Assigned To** | Backend |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 2 |

#### T-011-02 - Implementar endpoint GET /v1/knowledge/articles
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Implementar consulta de articulos aprobados |
| **Parent (User Story)** | HU-011 |
| **Assigned To** | Backend |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 3 |

#### T-011-03 - Implementar endpoint POST /v1/knowledge/articles/{id}/approve
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Implementar aprobacion de articulo por medico |
| **Parent (User Story)** | HU-011 |
| **Assigned To** | Backend |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 3 |

#### T-011-04 - Vista web de revision y aprobacion de articulos
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Construir pantalla de aprobacion de conocimiento en panel medico/admin |
| **Parent (User Story)** | HU-011 |
| **Assigned To** | Frontend Web |
| **State** | To Do |
| **Activity** | Development |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 4 |

#### T-011-05 - Pruebas unitarias e integracion del modulo de conocimiento
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Probar filtros por estado y aprobacion por rol medico |
| **Parent (User Story)** | HU-011 |
| **Assigned To** | QA/Backend |
| **State** | To Do |
| **Activity** | Testing |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 3 |

#### T-011-06 - Documentacion tecnica de checkout y knowledge en Wiki
| Campo Azure Boards | Valor |
|---|---|
| **Work Item Type** | Task |
| **Title** | Documentar contratos de monetizacion simulada y banco de conocimiento |
| **Parent (User Story)** | HU-010 / HU-011 |
| **Assigned To** | Backend/PO |
| **State** | To Do |
| **Activity** | Documentation |
| **Iteration Path** | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 2 |

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
EPIC: E7 - Monetizacion Simulada y Gobierno (10 SP total)
|
|-- FEATURE: F7.1 - Flujo de pago simulado (5 SP)
|   |
|   |-- USER STORY: HU-010 - Checkout simulado
|       |-- [ ] T-010-01 - Modelo de transaccion simulada
|       |-- [ ] T-010-02 - POST /v1/billing/simulate-checkout
|       |-- [ ] T-010-03 - Pantalla checkout simulado
|       |-- [ ] T-010-04 - Pruebas de integracion checkout
|
|-- FEATURE: F7.2 - Banco de conocimiento validado (5 SP)
    |
    |-- USER STORY: HU-011 - Reutilizacion de respuestas validadas
        |-- [ ] T-011-01 - Modelo KnowledgeArticle
        |-- [ ] T-011-02 - GET /v1/knowledge/articles
        |-- [ ] T-011-03 - POST /v1/knowledge/articles/{id}/approve
        |-- [ ] T-011-04 - Vista de aprobacion en panel web
        |-- [ ] T-011-05 - Pruebas del modulo knowledge
        |-- [ ] T-011-06 - Documentacion tecnica
```

**Total horas estimadas Sprint 8 (Epica 7):** 30 horas  
**Total Story Points Epica 7:** 10 SP  
**Sprint objetivo:** Sprint 8

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|---|---|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` - Sprint 8 |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` - E7 |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` - HU-010, HU-011 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` - HU-010 |
| MoSCoW | `docs/wiki/10-Priorizacion-MoSCoW.md` |
| Riesgos | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` |

### API endpoints de esta epica
- `POST /v1/billing/simulate-checkout` -> T-010-02
- `GET /v1/knowledge/articles` -> T-011-02
- `POST /v1/knowledge/articles/{id}/approve` -> T-011-03

### KPIs impactados por esta epica
- KPI de conversion simulada (complementario).
- KPI tiempo a primera respuesta (indirecto por reutilizacion de conocimiento).

### Riesgos asociados
- R-001: sobrealcance por agregar pagos reales (fuera de alcance).
- R-007: calidad de contenido del banco de conocimiento.
