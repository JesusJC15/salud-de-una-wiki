# Azure Boards - Epica 7: Monetizacion Simulada y Gobierno

## Proposito del documento
Este documento contiene toda la informacion necesaria para implementar la **Epica 7 - Monetizacion Simulada y Gobierno** en Azure Boards, incluyendo la epica, sus features, historias de usuario con criterios Gherkin, y todas las tareas de desarrollo, pruebas y documentacion asociadas. Cada seccion indica los campos exactos que se deben completar al crear el item en Azure Boards.

---

## 1. EPICA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Epic |
| **ID**                 | E7 |
| **Title**              | Monetizacion Simulada y Gobierno |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Web / SaludDeUna\\Producto |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Priority**           | 2 - High |
| **Business Value**     | Validar la viabilidad del modelo hibrido de monetizacion del MVP y establecer controles operativos para reutilizar contenido clinico validado |
| **Risk**               | Medium - Riesgo de sobrealcance si se intenta convertir el checkout simulado en cobro real o si el gobierno de contenido agrega flujos editoriales no comprometidos (R-001) |
| **Effort (Story Points total estimado)** | 10 SP |
| **Tags**               | monetizacion; checkout; governance; knowledge-base; sprint8 |
| **Start Date**         | Sprint 8 |
| **Target Date**        | Cierre Sprint 8 |

### Descripcion (campo Description)

```
La Epica 7 cubre el cierre operativo del MVP desde dos perspectivas: la validacion
del flujo de monetizacion sin pasarela productiva y el gobierno del banco de
conocimiento clinico validado para reutilizacion controlada.

Objetivos de negocio:
- Validar el comportamiento del flujo de compra simulada de consulta o plan sin
  introducir pagos reales, para medir interes y friccion del usuario.
- Reducir repeticion operativa en la atencion medica mediante un banco de
  conocimiento de respuestas o contenidos clinicos aprobados por pares.
- Generar evidencia para decisiones futuras de modelo comercial y controles de
  calidad del contenido reutilizable.

Funcionalidades cubiertas:
- Checkout simulado: seleccion de consulta o plan, confirmacion, cancelacion,
  registro de transaccion simulada exitosa y activacion de beneficio asociado.
- Persistencia de transacciones simuladas con trazabilidad minima para analitica
  complementaria de conversion.
- Banco de conocimiento con articulos en estados PENDING, APPROVED o REJECTED.
- Flujo de revision y aprobacion por medico autorizado o admin desde panel web.
- Endpoint publico de consulta que expone solo contenido APPROVED.

Restricciones:
- No se integra pasarela de pago productiva.
- No se realiza cobro, facturacion ni recaudo real.
- La cancelacion del checkout no genera transaccion de compra exitosa.
- La autoria inicial de articulos puede provenir de carga manual o seed interno;
  esta epica se centra en aprobacion y consulta, no en un CMS completo.
- El banco de conocimiento no reemplaza juicio clinico, diagnostico ni decision
  medica individual.
```

### Criterios de aceptacion de la epica

```
- El paciente puede simular una compra de consulta o plan y el sistema registra
  una transaccion simulada exitosa con el beneficio correspondiente.
- Si el paciente cancela el checkout, no se crea transaccion de compra exitosa y
  el estado del plan o consulta no cambia.
- El beneficio del plan o consulta se activa solo cuando la simulacion finaliza
  exitosamente.
- Un medico autorizado o admin puede aprobar contenido del banco de conocimiento.
- Solo el contenido APPROVED aparece en el endpoint de consulta publica.
- El sistema registra auditoria de checkout simulado y decisiones de aprobacion.
```

### Acceptance Criteria - formato Gherkin (nivel epica)

```gherkin
Feature: Monetizacion simulada y gobierno de contenido en SaludDeUna

  Scenario: Simulacion de compra exitosa
    Given un paciente autenticado con una opcion de consulta o plan seleccionada
    When confirma el checkout simulado
    Then el sistema registra la transaccion simulada como SUCCESS
    And habilita el beneficio contratado para el paciente

  Scenario: Cancelacion de checkout simulado
    Given un paciente en pantalla de checkout simulado
    When cancela la operacion
    Then el sistema no crea una transaccion simulada exitosa
    And el estado del plan o consulta permanece sin cambios

  Scenario: Aprobacion de articulo por medico
    Given un articulo del banco de conocimiento en estado PENDING
    When un medico autorizado lo aprueba
    Then el articulo cambia a estado APPROVED
    And queda disponible en el endpoint publico de conocimiento
```

---

## 2. FEATURES

### Feature F7.1 - Flujo de pago simulado

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F7.1 |
| **Title**              | Flujo de pago simulado |
| **Parent (Epic)**      | E7 - Monetizacion Simulada y Gobierno |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Producto |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Priority**           | 2 - High |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 5 SP |
| **Tags**               | checkout; billing; simulated; must; sprint8 |

#### Descripcion (campo Description)

```
Permite al paciente simular la compra de una consulta o plan dentro del MVP sin
conectarse a una pasarela real. El objetivo no es cobrar, sino validar el flujo,
la intencion de conversion y la activacion del beneficio asociado.

Alcance funcional:
- Pantalla de seleccion de opcion comercial (consulta puntual o plan).
- Resumen de checkout con nombre del producto, beneficio, costo referencial y
  confirmacion explicita de que es una simulacion.
- Endpoint POST /v1/billing/simulate-checkout para confirmar la simulacion.
- Persistencia de BillingSimulationTransaction con estado SUCCESS.
- Activacion del beneficio comprado en el perfil del paciente o en la consulta.
- Cancelacion del flujo desde la UI sin generar compra exitosa.
- Registro en log estructurado y auditoria del evento de simulacion.

Criterios de salida del feature:
- La simulacion exitosa crea una transaccion consultable.
- El beneficio asociado queda activo para el paciente.
- La cancelacion no crea compra exitosa ni altera beneficios.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Checkout simulado

  Scenario: Registro de transaccion simulada exitosa
    Given un paciente autenticado con una opcion comercial seleccionada
    When confirma la simulacion de compra
    Then el sistema crea una transaccion simulada SUCCESS
    And asocia el beneficio correspondiente al paciente

  Scenario: Cancelacion sin efectos laterales
    Given un paciente en la pantalla de checkout simulado
    When cancela la simulacion antes de confirmar
    Then no existe una transaccion simulada exitosa asociada a esa operacion
    And el estado del beneficio se mantiene sin cambios

  Scenario: Checkout protegido por autenticacion
    Given un usuario no autenticado
    When intenta ejecutar POST /v1/billing/simulate-checkout
    Then el sistema rechaza la solicitud
    And no crea ninguna transaccion
```

---

### Feature F7.2 - Banco de conocimiento validado

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F7.2 |
| **Title**              | Banco de conocimiento validado |
| **Parent (Epic)**      | E7 - Monetizacion Simulada y Gobierno |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Web / SaludDeUna\\Producto |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Priority**           | 3 - Medium |
| **MoSCoW**             | Should |
| **Effort (SP)**        | 5 SP |
| **Tags**               | knowledge; governance; content; should; sprint8 |

#### Descripcion (campo Description)

```
Habilita la consulta de articulos de conocimiento clinico aprobados y el flujo de
gobierno minimo para que un medico autorizado o admin revise y apruebe contenido
antes de exponerlo para reutilizacion operacional.

Alcance funcional:
- Modelo KnowledgeArticle con estados PENDING, APPROVED y REJECTED.
- Endpoint GET /v1/knowledge/articles que retorna solo articulos APPROVED.
- Endpoint POST /v1/knowledge/articles/{id}/approve para aprobar o rechazar.
- Vista web para revision, aprobacion y filtro por estado.
- Registro de reviewedBy, reviewedAt y notas de revision.
- Control de acceso por rol para evitar aprobaciones no autorizadas.

Criterios de salida del feature:
- Los articulos aprobados son visibles para consulta.
- Los articulos pendientes o rechazados no aparecen en la consulta publica.
- Toda aprobacion queda trazable con actor y fecha.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Gobierno de conocimiento clinico

  Scenario: Publicacion de contenido aprobado
    Given un articulo en estado APPROVED
    When un cliente consulta GET /v1/knowledge/articles
    Then el articulo aparece en los resultados
    And no aparecen articulos en estado PENDING o REJECTED

  Scenario: Rechazo de articulo por medico revisor
    Given un articulo en estado PENDING
    When un medico autorizado lo marca como REJECTED
    Then el articulo cambia a estado REJECTED
    And no queda visible en el endpoint publico

  Scenario: Aprobacion protegida por rol
    Given un usuario sin rol MEDICO ni ADMIN
    When intenta aprobar un articulo del banco de conocimiento
    Then el sistema retorna HTTP 403
    And no modifica el estado del articulo
```

---

## 3. HISTORIAS DE USUARIO

### HU-010 - Flujo de monetizacion simulado

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-010 |
| **Title**              | Como paciente quiero simular compra de consulta o plan para validar el flujo de monetizacion del producto |
| **Parent (Feature)**   | F7.1 - Flujo de pago simulado |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Producto |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Priority**           | 2 - High |
| **MoSCoW**             | Must |
| **Story Points**       | 5 |
| **Risk**               | Medium - Riesgo de confusion con pago real o de activar beneficios duplicados si el flujo no es idempotente |
| **Dependencias**       | HU-001 |
| **Tags**               | monetizacion; checkout; simulated; patient; must; sprint8 |

#### Descripcion completa (campo Description)

```
Como paciente
Quiero simular la compra de una consulta o un plan
Para validar el flujo de monetizacion del producto sin necesidad de una pasarela
de pago real en esta etapa del MVP.

Contexto:
El equipo necesita validar si el usuario entiende la propuesta comercial y es capaz
de completar un flujo de compra simple. Esta historia mide interes y friccion del
checkout, pero no ejecuta cobro real. El valor esta en probar la experiencia, no en
la transaccion financiera productiva.

Flujo principal:
1. El paciente autenticado selecciona una consulta o plan disponible.
2. La app muestra resumen del producto, costo referencial y mensaje de simulacion.
3. El paciente confirma el checkout simulado.
4. El backend registra una BillingSimulationTransaction en estado SUCCESS.
5. El sistema activa el beneficio correspondiente en la cuenta o flujo del paciente.
6. Se registra auditoria para posterior analitica de conversion simulada.

Restricciones funcionales:
- No existe integracion con pasarela de pago ni cobro real.
- Si el paciente cancela antes de confirmar, no se genera compra exitosa.
- El beneficio solo se activa cuando el backend confirma SUCCESS.
- El checkout debe dejar claro que se trata de una simulacion.
- El flujo debe ser idempotente para no activar dos veces el mismo beneficio.

Notas de UX:
- La pantalla debe mostrar claramente el nombre del producto y el beneficio.
- Debe existir una accion explicita de confirmar y otra de cancelar.
- La confirmacion debe devolver un mensaje claro de exito y proximo paso.
```

#### Criterios de Aceptacion - Gherkin

```gherkin
Feature: Simulacion de checkout

  Scenario: Compra simulada confirmada
    Given un paciente autenticado en la pantalla de checkout simulado
    When confirma la simulacion de compra
    Then el sistema registra una transaccion simulada SUCCESS
    And habilita el beneficio asociado a la opcion seleccionada

  Scenario: Compra simulada cancelada
    Given un paciente en la pantalla de checkout simulado
    When cancela la operacion
    Then no se crea una transaccion simulada exitosa
    And el estado previo del beneficio se mantiene sin cambios

  Scenario: Reintento de confirmacion del mismo checkout
    Given un paciente que ya confirmo exitosamente una simulacion para la misma opcion
    When reintenta confirmar el mismo checkout de forma inmediata
    Then el sistema evita duplicar la activacion del beneficio
    And responde con el estado existente de la simulacion
```

#### Definition of Ready (DoR) - Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, epica y feature definidos (E7, F7.1)
- [x] Prioridad MoSCoW asignada (Must)
- [x] Criterios Gherkin documentados
- [x] Dependencias identificadas (HU-001)
- [x] Estimacion acordada (5 SP)
- [x] Restriccion explicita de no usar pasarela real documentada
- [x] Riesgos anotados (confusion con cobro real, duplicidad de beneficio)
- [x] Responsable de validacion funcional asignado

#### Definition of Done (DoD) - Checklist

- [ ] Schema BillingSimulationTransaction implementado y persistido en MongoDB
- [ ] Endpoint `POST /v1/billing/simulate-checkout` implementado y protegido con JWT
- [ ] Activacion de beneficio implementada de forma idempotente
- [ ] Pantalla de checkout simulado implementada en la app movil
- [ ] Mensaje de confirmacion explicita de simulacion visible en UI
- [ ] Pruebas unitarias e integracion del flujo SUCCESS/CANCELLED en verde
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] Log de auditoria registra la simulacion y la activacion del beneficio
- [ ] Documentacion tecnica actualizada en Wiki
- [ ] Demo funcional aprobada por el equipo

---

### HU-011 - Banco de conocimiento validado por pares

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-011 |
| **Title**              | Como medico quiero reutilizar respuestas validadas por pares para reducir consultas repetitivas y mejorar consistencia |
| **Parent (Feature)**   | F7.2 - Banco de conocimiento validado |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Web / SaludDeUna\\Producto |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Priority**           | 3 - Medium |
| **MoSCoW**             | Should |
| **Story Points**       | 5 |
| **Risk**               | Medium - Riesgo de reutilizar contenido desactualizado o sin revision suficiente si no se controla el flujo de aprobacion |
| **Dependencias**       | HU-005 |
| **Tags**               | knowledge; governance; doctor; should; sprint8 |

#### Descripcion completa (campo Description)

```
Como medico
Quiero reutilizar respuestas validadas por pares
Para reducir consultas repetitivas y mejorar la consistencia del contenido clinico
que se usa como apoyo operativo dentro de la plataforma.

Contexto:
Muchos mensajes y orientaciones se repiten entre consultas de baja o mediana
complejidad. Un banco de conocimiento validado permite tener contenido revisado y
trazable, disminuyendo repeticion operativa y mejorando consistencia. Esta historia
no construye un CMS completo; se enfoca en aprobacion y consulta de articulos.

Flujo principal:
1. Existe un articulo candidato en estado PENDING dentro del modulo Knowledge.
2. Un medico autorizado o admin abre el panel de revision.
3. El revisor aprueba o rechaza el articulo y deja trazabilidad de la decision.
4. El articulo APPROVED queda disponible para consulta mediante GET /v1/knowledge/articles.
5. El contenido aprobado puede reutilizarse como soporte operacional en otros flujos.

Restricciones funcionales:
- Solo roles MEDICO autorizado o ADMIN pueden aprobar o rechazar contenido.
- El endpoint publico no expone articulos PENDING ni REJECTED.
- El articulo aprobado no sustituye juicio clinico ni decision individual del medico.
- La creacion editorial de articulos fuera de la aprobacion minima no forma parte
  del alcance principal de esta historia.
- Si el sprint requiere recorte, esta historia puede moverse por ser Should.

Notas de UX / operacion:
- El panel web debe permitir filtrar por estado y especialidad.
- Debe ser posible ver quien aprobo o rechazo y cuando lo hizo.
- Los articulos aprobados deben tener metadata suficiente para futura busqueda.
```

#### Criterios de Aceptacion - Gherkin

```gherkin
Feature: Base de conocimiento validada

  Scenario: Medico aprueba articulo pendiente
    Given un articulo del banco de conocimiento en estado PENDING
    When un medico autorizado lo aprueba
    Then el articulo cambia a estado APPROVED
    And queda disponible para consulta en GET /v1/knowledge/articles

  Scenario: Revisor rechaza articulo
    Given un articulo del banco de conocimiento en estado PENDING
    When un medico autorizado lo marca como REJECTED
    Then el articulo cambia a estado REJECTED
    And no aparece en el endpoint de consulta publica

  Scenario: Usuario sin permisos intenta aprobar
    Given un usuario autenticado sin rol MEDICO ni ADMIN
    When intenta aprobar un articulo del banco de conocimiento
    Then el sistema retorna HTTP 403
    And el estado del articulo permanece sin cambios
```

#### Definition of Ready (DoR) - Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, epica y feature definidos (E7, F7.2)
- [x] Prioridad MoSCoW asignada (Should)
- [x] Criterios Gherkin documentados
- [x] Dependencias identificadas (HU-005)
- [x] Estimacion acordada (5 SP)
- [x] Reglas de rol y gobierno documentadas
- [x] Restriccion de alcance del banco de conocimiento definida
- [x] Riesgos anotados (contenido desactualizado, flujo editorial excesivo)
- [x] Responsable de validacion funcional asignado

#### Definition of Done (DoD) - Checklist

- [ ] Schema KnowledgeArticle implementado con estados PENDING, APPROVED y REJECTED
- [ ] Endpoint `GET /v1/knowledge/articles` implementado con filtro solo APPROVED
- [ ] Endpoint `POST /v1/knowledge/articles/{id}/approve` implementado con control por rol
- [ ] Vista web de revision y aprobacion implementada con filtros por estado
- [ ] Aprobaciones y rechazos registran reviewedBy, reviewedAt y notas
- [ ] Pruebas unitarias e integracion del modulo knowledge en verde
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] Validaciones de seguridad ejecutadas para control por rol
- [ ] Documentacion tecnica actualizada en Wiki
- [ ] Demo funcional aprobada por el equipo

---

## 4. TAREAS

### Tareas de HU-010 - Flujo de monetizacion simulado

---

#### T-010-01 - Disenar modelo de transaccion simulada

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Disenar modelo BillingSimulationTransaction en MongoDB |
| **Parent (User Story)**| HU-010 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 2 |

**Descripcion:**
```
Crear el schema Mongoose BillingSimulationTransaction con campos minimos:
- _id (ObjectId)
- patientId (ObjectId, ref Patient)
- checkoutType: 'CONSULTATION' | 'PLAN'
- productCode o planCode
- referenceAmount (Number)
- currency: 'COP'
- status: 'SUCCESS'
- benefitType
- benefitStatus: 'ACTIVE'
- simulatedAt
- correlationId
- timestamps

Indices sugeridos:
- { patientId: 1, simulatedAt: -1 }
- { patientId: 1, productCode: 1 }

Ubicar en: apps/api/src/billing/schemas/
```

**Criterios de aceptacion de la tarea:**
- El schema se crea y exporta correctamente.
- Los campos permiten trazabilidad de la simulacion y del beneficio activado.
- Los indices soportan consulta por paciente y producto.

---

#### T-010-02 - Implementar endpoint POST /v1/billing/simulate-checkout

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint de checkout simulado |
| **Parent (User Story)**| HU-010 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 4 |

**Descripcion:**
```
Implementar POST /v1/billing/simulate-checkout:
- protegido con JwtAuthGuard y rol PACIENTE
- DTO con opcion seleccionada, acceptedTerms y metadata minima
- persistir BillingSimulationTransaction en estado SUCCESS
- activar beneficio asociado para el paciente
- registrar auditoria y log estructurado con correlation_id
- aplicar idempotencia basica para evitar doble activacion del mismo checkout

Ubicar en: apps/api/src/billing/
```

**Criterios de aceptacion de la tarea:**
- El endpoint retorna exito con la transaccion simulada creada.
- El beneficio queda activo al finalizar SUCCESS.
- Un reintento inmediato no duplica beneficio ni transaccion activa equivalente.
- El log registra patientId, checkoutType y correlation_id.

---

#### T-010-03 - Implementar pantalla de checkout simulado en app movil

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Construir UI de seleccion y confirmacion de plan o consulta simulada |
| **Parent (User Story)**| HU-010 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 5 |

**Descripcion:**
```
Construir la pantalla de checkout simulado en la app React Native:
- selector de opcion comercial
- resumen del producto y costo referencial
- mensaje visible de que no es cobro real
- botones Confirmar simulacion y Cancelar
- manejo de estado de carga y confirmacion exitosa

Ubicar en: apps/mobile/src/screens/billing/ o modulo equivalente
```

**Criterios de aceptacion de la tarea:**
- La pantalla permite seleccionar y confirmar una simulacion.
- La cancelacion vuelve al estado previo sin crear compra exitosa.
- La UI deja claro que el flujo es simulado.

---

#### T-010-04 - Pruebas de integracion de checkout simulado

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Probar escenarios SUCCESS, CANCELLED e idempotencia del checkout simulado |
| **Parent (User Story)**| HU-010 |
| **Assigned To**        | QA / Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 3 |

**Descripcion:**
```
Ejecutar pruebas de integracion y validacion funcional para:
- confirmacion exitosa
- cancelacion sin efectos laterales
- doble confirmacion del mismo checkout
- rechazo de acceso sin JWT
```

**Criterios de aceptacion de la tarea:**
- Los escenarios SUCCESS y CANCELLED quedan cubiertos.
- La idempotencia evita activacion duplicada.
- Los resultados quedan documentados para la demo del sprint.

---

#### T-010-05 - Documentacion tecnica del flujo de monetizacion simulada

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar contrato del checkout simulado y estados de transaccion |
| **Parent (User Story)**| HU-010 |
| **Assigned To**        | Backend / PO |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 1 |

**Descripcion:**
```
Actualizar la Wiki con:
- endpoint de checkout simulado
- contrato BillingSimulationTransaction
- restricciones de no cobro real
- evidencia necesaria para analitica complementaria de conversion
```

**Criterios de aceptacion de la tarea:**
- La documentacion coincide con el endpoint implementado.
- El alcance no financiero queda explicitado.

---

### Tareas de HU-011 - Banco de conocimiento validado

---

#### T-011-01 - Disenar modelo KnowledgeArticle

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Disenar modelo KnowledgeArticle con estados PENDING, APPROVED y REJECTED |
| **Parent (User Story)**| HU-011 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 2 |

**Descripcion:**
```
Crear el schema KnowledgeArticle con campos minimos:
- _id
- title
- specialty
- summary
- body
- tags
- status: PENDING | APPROVED | REJECTED
- authorName o sourceLabel
- reviewedBy
- reviewedAt
- reviewNotes
- timestamps

Indices sugeridos:
- { status: 1, specialty: 1 }
- { tags: 1 }

Ubicar en: apps/api/src/knowledge/schemas/
```

**Criterios de aceptacion de la tarea:**
- El schema soporta estados y metadata de revision.
- Los indices soportan consulta por estado y especialidad.
- reviewedBy y reviewedAt quedan listos para auditoria.

---

#### T-011-02 - Implementar endpoint GET /v1/knowledge/articles

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar consulta de articulos aprobados del banco de conocimiento |
| **Parent (User Story)**| HU-011 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 3 |

**Descripcion:**
```
Implementar GET /v1/knowledge/articles:
- retornar solo articulos con status APPROVED
- soportar filtros basicos por specialty y tag
- paginacion simple para evitar listas extensas
- no exponer reviewNotes internas ni articulos pendientes

Ubicar en: apps/api/src/knowledge/
```

**Criterios de aceptacion de la tarea:**
- El endpoint retorna solo articulos APPROVED.
- Los filtros basicos funcionan correctamente.
- No se exponen articulos PENDING ni REJECTED.

---

#### T-011-03 - Implementar endpoint POST /v1/knowledge/articles/{id}/approve

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar aprobacion o rechazo de articulo por medico autorizado |
| **Parent (User Story)**| HU-011 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 3 |

**Descripcion:**
```
Implementar POST /v1/knowledge/articles/{id}/approve con body de decision:
- status: APPROVED | REJECTED
- notes?: string

Reglas:
- solo MEDICO autorizado o ADMIN puede ejecutar la accion
- actualizar reviewedBy, reviewedAt y reviewNotes
- registrar auditoria del cambio de estado

Ubicar en: apps/api/src/knowledge/
```

**Criterios de aceptacion de la tarea:**
- El endpoint permite aprobar o rechazar articulos pendientes.
- Los roles no autorizados reciben 403.
- La decision queda trazable con actor y fecha.

---

#### T-011-04 - Vista web de revision y aprobacion de articulos

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Construir pantalla de revision y aprobacion de conocimiento en panel web |
| **Parent (User Story)**| HU-011 |
| **Assigned To**        | Desarrollador Web |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 4 |

**Descripcion:**
```
Construir una vista web para revision del banco de conocimiento:
- listado de articulos por estado
- filtro por specialty
- detalle del articulo
- acciones Aprobar y Rechazar
- visualizacion de reviewedBy y reviewedAt cuando aplique

Ubicar en: apps/web/src/app/knowledge/ o modulo equivalente
```

**Criterios de aceptacion de la tarea:**
- El panel muestra articulos por estado y especialidad.
- Aprobar y rechazar actualiza la UI correctamente.
- La pantalla refleja quien reviso el articulo y cuando.

---

#### T-011-05 - Pruebas unitarias e integracion del modulo de conocimiento

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Probar filtros por estado y aprobacion por rol medico |
| **Parent (User Story)**| HU-011 |
| **Assigned To**        | QA / Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 3 |

**Descripcion:**
```
Ejecutar pruebas unitarias e integracion del modulo Knowledge para validar:
- consulta solo APPROVED
- aprobacion por rol MEDICO o ADMIN
- rechazo y no exposicion publica
- intento de aprobacion por usuario no autorizado
```

**Criterios de aceptacion de la tarea:**
- Los filtros por estado quedan cubiertos por pruebas.
- La aprobacion y el rechazo quedan validados por rol.
- Los articulos no aprobados no se exponen publicamente.

---

#### T-011-06 - Documentacion tecnica de checkout y knowledge en Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar contratos de monetizacion simulada y banco de conocimiento |
| **Parent (User Story)**| HU-011 |
| **Related (User Story)**| HU-010 |
| **Assigned To**        | Backend / PO |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 8 |
| **Remaining Work (h)** | 2 |

**Descripcion:**
```
Actualizar la Wiki con:
- endpoint de checkout simulado
- endpoint de consulta y aprobacion de knowledge
- modelo KnowledgeArticle
- restricciones de gobierno y alcance editorial del sprint
```

**Criterios de aceptacion de la tarea:**
- La documentacion cubre ambos contratos del sprint.
- Las restricciones y roles quedan claros para el equipo.

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
EPIC: E7 - Monetizacion Simulada y Gobierno (10 SP total)
|
|-- FEATURE: F7.1 - Flujo de pago simulado (5 SP)
|   |
|   |-- USER STORY: HU-010 - Simulacion de checkout de consulta o plan (5 SP)
|       |-- [ ] T-010-01 - Modelo BillingSimulationTransaction (2h)
|       |-- [ ] T-010-02 - POST /v1/billing/simulate-checkout (4h)
|       |-- [ ] T-010-03 - Pantalla de checkout simulado en app movil (5h)
|       |-- [ ] T-010-04 - Pruebas SUCCESS/CANCELLED/idempotencia (3h)
|       |-- [ ] T-010-05 - Documentacion tecnica del flujo simulado (1h)
|
|-- FEATURE: F7.2 - Banco de conocimiento validado (5 SP)
    |
    |-- USER STORY: HU-011 - Reutilizacion de respuestas validadas por pares (5 SP)
        |-- [ ] T-011-01 - Modelo KnowledgeArticle (2h)
        |-- [ ] T-011-02 - GET /v1/knowledge/articles (3h)
        |-- [ ] T-011-03 - POST /v1/knowledge/articles/{id}/approve (3h)
        |-- [ ] T-011-04 - Vista web de revision y aprobacion (4h)
        |-- [ ] T-011-05 - Pruebas del modulo knowledge (3h)
        |-- [ ] T-011-06 - Documentacion tecnica checkout + knowledge (2h)
```

**Total horas estimadas Sprint 8 (Epica 7):** 32 horas de trabajo  
**Total Story Points Epica 7:** 10 SP (HU-010: 5 SP + HU-011: 5 SP)  
**Sprint objetivo:** Sprint 8

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|-----------|-----------|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` - alcance MVP y endpoints publicos |
| Contexto del problema | `docs/wiki/02-Descripcion-Problema-Contexto.md` - monetizacion simulada fuera de pasarela real |
| Modelo Canvas | `docs/wiki/03-Modelo-Canvas.md` - validacion inicial del modelo hibrido por simulacion |
| Arquitectura base | `docs/wiki/04-Arquitectura-Despliegue-Base.md` - BillingModule y KnowledgeModule |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` - E7, F7.1, F7.2 |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` - HU-010, HU-011 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` - HU-010 |
| MoSCoW | `docs/wiki/10-Priorizacion-MoSCoW.md` |
| Plan de sprints | `docs/wiki/11-Plan-Sprints-0-a-9.md` - Sprint 8 |
| Riesgos | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` |

### API endpoints de esta epica
- `POST /v1/billing/simulate-checkout` -> T-010-02
- `GET /v1/knowledge/articles` -> T-011-02
- `POST /v1/knowledge/articles/{id}/approve` -> T-011-03

### KPIs impactados por esta epica
- **KPI de conversion simulada (complementario):** la transaccion simulada permite medir interes comercial sin pasarela real.
- **KPI tiempo a primera respuesta medica (indirecto):** el contenido reutilizable validado puede reducir repeticion operativa en atencion.

### Riesgos asociados
- **R-001 (Sobrecarga de alcance):** evitar que el checkout simulado derive a pagos reales o a un CMS editorial completo.
- **R-005 (Uso indebido de rol medico):** la aprobacion del banco de conocimiento debe estar protegida por rol MEDICO autorizado o ADMIN.
- **R-006 (Exposicion de datos sensibles):** los articulos y transacciones simuladas no deben exponer informacion clinica o personal innecesaria.
