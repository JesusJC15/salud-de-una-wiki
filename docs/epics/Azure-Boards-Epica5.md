# Azure Boards – Épica 5: Seguimiento y Evolución del Paciente

## Propósito del documento
Este documento contiene toda la información necesaria para implementar la **Épica 5 – Seguimiento y Evolución del Paciente** en Azure Boards, incluyendo la épica, sus features, historias de usuario con criterios Gherkin, y todas las tareas de desarrollo, pruebas y documentación asociadas. Cada sección indica los campos exactos que se deben completar al crear el ítem en Azure Boards.

---

## 1. ÉPICA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Epic |
| **ID**                 | E5 |
| **Title**              | Seguimiento y Evolución del Paciente |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Priority**           | 1 – Critical |
| **Business Value**     | Medir la progresión del paciente post-consulta, detectar cambios de riesgo oportunamente y aumentar la retención a 7 días mediante seguimiento automatizado |
| **Risk**               | Medium – Depende de que HU-006 (chat y consulta en tiempo real) esté cerrada; riesgo de abandono del paciente si los recordatorios no son relevantes |
| **Effort (Story Points total estimado)** | 16 SP |
| **Tags**               | seguimiento; evolucion; timeline; post-consulta; recordatorios; must; sprint6 |
| **Start Date**         | Sprint 6 |
| **Target Date**        | Cierre Sprint 6 |

### Descripción (campo Description)

```
La Épica 5 cubre el ciclo completo de seguimiento post-consulta y evolución
temporal del paciente en SaludDeUna, desde el cierre de la consulta hasta
el reporte de evolución y el ajuste de prioridad ante empeoramiento.

Objetivos de negocio:
- Aumentar la retención de pacientes a 7 días mediante recordatorios automáticos
  de seguimiento al cierre de cada consulta.
- Detectar oportunamente cambios de estado clínico (empeoramiento/mejora) a
  través de formularios de evolución estructurados y notificaciones al médico.
- Proveer al médico un timeline evolutivo del paciente que permita analizar la
  progresión de síntomas a lo largo del tiempo y ajustar el plan de cuidado.
- Generar el KPI de retención de seguimiento a 7 días como métrica de negocio
  obligatoria del MVP.

Funcionalidades cubiertas:
- Creación automática de un registro de seguimiento (Followup) al cerrar cada
  consulta, con fecha programada a las 72 horas y un recordatorio a los 7 días.
- Formulario de evolución para el paciente: intensidad actual de síntomas, cambios
  percibidos respecto a la consulta original, medicación tomada, nuevos síntomas.
- Motor de reglas de re-priorización: si el paciente reporta empeoramiento (aumento
  de intensidad >= 2 puntos en escala 1-10 o aparición de red flag), el sistema
  actualiza la prioridad del caso y notifica al médico responsable.
- Timeline evolutivo del paciente: vista cronológica de todos los eventos del caso
  (triage, consulta, mensajes, seguimientos, cambios de prioridad) accesible para
  médico y paciente.
- Endpoint POST /v1/followups para crear y registrar respuestas de seguimiento.
- Endpoint GET /v1/patients/{id}/timeline para obtener la línea de tiempo completa.
- WebSocket event consultation.followup.reminder.triggered para recordatorios
  en tiempo real en la app móvil.
- Soporte de perfil de cuidador: un familiar puede reportar evolución en nombre
  de un menor o adulto mayor (alcance Could, HU-012).

Restricciones:
- El seguimiento es un complemento comunicacional; no constituye diagnóstico ni
  prescripción automática por parte del sistema.
- La re-priorización automática notifica al médico pero no toma decisiones
  clínicas de forma autónoma; el médico confirma cualquier cambio de plan.
- El timeline es de solo lectura para el paciente; el médico puede agregar notas
  clínicas dentro del sistema pero no alterar registros históricos.
- No se integra con sistemas HCE externos en este sprint.
- La épica depende de que la consulta base (E3) y el resumen clínico (E4)
  estén implementados, ya que el seguimiento referencia la consulta cerrada.
```

### Criterios de aceptación de la épica

```
- Al cerrar una consulta, el sistema crea automáticamente un Followup con
  fecha programada a las 72 horas y un recordatorio a los 7 días.
- El paciente recibe una notificación (WebSocket / push) cuando llega la fecha
  de seguimiento y puede completar el formulario de evolución desde la app móvil.
- Al completar el formulario, el sistema persiste el documento Followup con todos
  los campos de evolución (intensidad, cambios, medicación, nuevos síntomas).
- Si el paciente reporta empeoramiento (criterios definidos en motor de reglas),
  el sistema actualiza la prioridad del caso y notifica al médico responsable
  dentro de los 5 minutos siguientes al reporte.
- El médico puede acceder al timeline evolutivo del paciente desde el panel web
  y ver todos los eventos ordenados cronológicamente.
- El paciente puede ver su propia línea de tiempo desde la app móvil.
- El KPI de retención a 7 días se calcula y muestra correctamente en el dashboard:
  porcentaje de consultas cerradas que tienen al menos un Followup completado
  dentro de los 7 días posteriores al cierre.
- El tiempo de respuesta del endpoint GET /v1/patients/{id}/timeline cumple
  el SLO: P95 < 1500 ms.
```

### Acceptance Criteria – formato Gherkin (nivel épica)

```gherkin
Feature: Seguimiento y evolución post-consulta en SaludDeUna

  Scenario: Creación automática de seguimiento al cerrar consulta
    Given una consulta en estado ACTIVE con médico y paciente asignados
    When el médico o el sistema cierra la consulta con estado CLOSED
    Then el sistema crea un documento Followup con scheduledAt = now + 72h
    And programa un recordatorio adicional a los 7 días
    And persiste el Followup en MongoDB vinculado a la consultationId

  Scenario: Paciente completa formulario de evolución sin empeoramiento
    Given un Followup con fecha de seguimiento alcanzada
    When el paciente completa el formulario indicando mejora o estabilidad
    Then el sistema persiste las respuestas en el documento Followup
    And actualiza el campo status a COMPLETED
    And no modifica la prioridad del caso

  Scenario: Re-priorización automática por empeoramiento reportado
    Given un paciente con una consulta cerrada de prioridad LOW o MODERATE
    When el paciente reporta empeoramiento con intensidad aumentada en 2 o más puntos
    Then el sistema actualiza la prioridad del caso al nivel inmediatamente superior
    And emite evento WebSocket consultation.priority.updated al médico
    And notifica al médico responsable mediante notificación interna

  Scenario: Médico accede al timeline evolutivo del paciente
    Given un paciente con al menos una consulta, un triage y un seguimiento registrados
    When el médico solicita GET /v1/patients/{patientId}/timeline
    Then el sistema retorna los eventos ordenados cronológicamente
    And cada evento incluye tipo, fecha, actor y referencia al documento fuente

  Scenario: Recordatorio de seguimiento por WebSocket
    Given un Followup cuya scheduledAt ha llegado y está en estado PENDING
    When el scheduler del sistema detecta el Followup vencido
    Then emite el evento consultation.followup.reminder.triggered al paciente
    And el estado del Followup cambia a REMINDED
```

---

## 2. FEATURES

### Feature F5.1 – Seguimiento Post-Consulta Automatizado

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F5.1 |
| **Title**              | Seguimiento Post-Consulta Automatizado |
| **Parent (Epic)**      | E5 – Seguimiento y Evolución del Paciente |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 8 SP |
| **Tags**               | seguimiento; post-consulta; formulario; re-priorizacion; recordatorios; websocket; must; sprint6 |

#### Descripción (campo Description)

```
Cubre la creación automática del registro de seguimiento al cierre de una consulta,
el envío de recordatorios al paciente, la captura del formulario de evolución y
el motor de reglas de re-priorización por empeoramiento.

Alcance funcional:
- Al cambiar el estado de una consulta a CLOSED, el sistema crea automáticamente
  un documento Followup en MongoDB con:
    · consultationId (ref)
    · patientId (ref)
    · doctorId (ref)
    · scheduledAt: Date = ahora + 72 horas
    · reminderAt: Date = ahora + 7 días
    · status: 'PENDING'
    · responses: null (se completa al responder el formulario)
- Scheduler NestJS (cron) que evalúa Followups con scheduledAt vencido y emite
  el evento WebSocket consultation.followup.reminder.triggered.
- Endpoint POST /v1/followups para que el paciente registre sus respuestas
  de evolución: intensidad actual (1-10), cambio percibido (BETTER/SAME/WORSE),
  medicación tomada, nuevos síntomas (string opcional).
- Motor de reglas de re-priorización integrado en FollowupService:
    · Si cambio = WORSE y (delta de intensidad >= 2 o presencia de red flag
      en nuevos síntomas), incrementar prioridad del caso.
    · Emitir evento consultation.priority.updated al médico por WebSocket.
    · Crear notificación interna para el médico responsable.
- El endpoint y el scheduler validan autenticación y autorización por rol.
- El paciente solo puede completar un Followup vinculado a una consulta propia.
- Registrar todos los eventos en log estructurado con correlation ID.

Criterios de salida del feature:
- Al cerrar una consulta, existe en MongoDB un Followup con status PENDING.
- El scheduler detecta Followups vencidos y emite el evento de recordatorio.
- El paciente puede completar el formulario y el sistema persiste las respuestas.
- Si hay empeoramiento, la prioridad del caso se actualiza y el médico es notificado.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Seguimiento post-consulta automatizado

  Scenario: Followup creado al cerrar consulta
    Given una consulta en estado ACTIVE
    When el estado cambia a CLOSED
    Then el sistema crea un Followup con scheduledAt = now + 72h
    And el Followup tiene status PENDING y el consultationId correcto

  Scenario: Recordatorio enviado al vencer la fecha programada
    Given un Followup con scheduledAt en el pasado y status PENDING
    When el scheduler ejecuta el job de recordatorios
    Then el paciente recibe el evento consultation.followup.reminder.triggered
    And el Followup cambia a status REMINDED

  Scenario: Paciente completa formulario con mejoría
    Given un Followup en estado REMINDED vinculado al paciente autenticado
    When el paciente envía POST /v1/followups con cambio=BETTER e intensidad menor
    Then el sistema persiste las respuestas y status=COMPLETED
    And no modifica la prioridad de la consulta

  Scenario: Re-priorización por empeoramiento reportado
    Given una consulta cerrada con prioridad LOW y un paciente con intensidad original 3
    When el paciente reporta cambio=WORSE con intensidad 6 en el formulario
    Then el sistema actualiza la prioridad del caso a MODERATE
    And emite evento WebSocket consultation.priority.updated al médico
    And crea notificación interna para el médico responsable

  Scenario: Paciente intenta completar Followup ajeno
    Given un Followup perteneciente al paciente A
    When el paciente B intenta enviarlo como propio
    Then el sistema retorna HTTP 403
    And no persiste ningún cambio
```

---

### Feature F5.2 – Timeline Evolutivo del Paciente

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F5.2 |
| **Title**              | Timeline Evolutivo del Paciente |
| **Parent (Epic)**      | E5 – Seguimiento y Evolución del Paciente |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 8 SP |
| **Tags**               | timeline; evolucion; historial; medico; paciente; must; sprint6 |

#### Descripción (campo Description)

```
Cubre la construcción y exposición del timeline evolutivo del paciente: una vista
cronológica unificada de todos los eventos del caso (triage, consultas, mensajes,
seguimientos, cambios de prioridad) accesible tanto para el médico en el panel web
como para el paciente en la app móvil.

Alcance funcional:
- Endpoint GET /v1/patients/{patientId}/timeline que agrega eventos de múltiples
  colecciones (TriageSession, Consultation, Message, Followup, PriorityChange) y
  los retorna ordenados por fecha descendente.
- Paginación con cursor para soportar pacientes con historial extenso.
- Cada evento del timeline incluye:
    · eventType: 'TRIAGE' | 'CONSULTATION_OPENED' | 'CONSULTATION_CLOSED' |
      'MESSAGE' | 'FOLLOWUP_COMPLETED' | 'PRIORITY_CHANGED'
    · occurredAt (Date)
    · actor (patientId | doctorId)
    · summary (string breve descriptivo del evento)
    · referenceId (ObjectId del documento fuente)
    · metadata (campo libre para datos adicionales por tipo de evento)
- Vista de timeline en la app React Native: lista vertical de eventos con iconos
  por tipo, fecha relativa y resumen. Filtro por tipo de evento.
- Vista de timeline en el panel médico (Next.js): tabla cronológica con acceso
  rápido al documento fuente (consulta, seguimiento, etc.).
- Control de acceso: el paciente ve solo su propio timeline; el médico ve el
  timeline de pacientes asignados a sus consultas; el admin puede ver cualquier
  timeline.
- Indicadores de cambio de síntomas: si entre dos Followups consecutivos hay
  un delta de intensidad significativo, el timeline marca el evento con un
  indicador visual de cambio de tendencia (mejora/empeoramiento).

Criterios de salida del feature:
- GET /v1/patients/{id}/timeline retorna todos los eventos en orden cronológico.
- La app móvil muestra el timeline con iconos y resúmenes por evento.
- El panel médico muestra el timeline del paciente seleccionado.
- El tiempo de respuesta del endpoint cumple el SLO: P95 < 1500 ms.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Timeline evolutivo del paciente

  Scenario: Médico consulta timeline completo de un paciente
    Given un paciente con triage, consulta y seguimiento registrados
    When el médico solicita GET /v1/patients/{patientId}/timeline
    Then el sistema retorna una lista ordenada por fecha descendente
    And cada evento incluye eventType, occurredAt, actor y summary
    And el tiempo de respuesta es menor a 1500 ms (SLO P95)

  Scenario: Paciente accede a su propio timeline desde la app
    Given un paciente autenticado con historial de consultas
    When abre la pantalla de historial en la app móvil
    Then ve sus eventos cronológicamente con iconos por tipo
    And puede filtrar por tipo de evento

  Scenario: Indicador de empeoramiento entre seguimientos
    Given dos Followups consecutivos del mismo paciente
    When el segundo reporta delta de intensidad >= 2 respecto al primero
    Then el evento en el timeline tiene el indicador de empeoramiento visible
    And el médico puede identificar la tendencia sin abrir cada Followup

  Scenario: Paciente intenta ver timeline de otro paciente
    Given el paciente A autenticado
    When solicita GET /v1/patients/{patientIdB}/timeline
    Then el sistema retorna HTTP 403
    And no expone datos del paciente B

  Scenario: Timeline con paginación para historial extenso
    Given un paciente con más de 50 eventos en su historial
    When el cliente solicita el timeline sin cursor
    Then el sistema retorna los primeros 20 eventos y un nextCursor
    And el cliente puede paginar usando el nextCursor para obtener el siguiente lote
```

---

## 3. HISTORIAS DE USUARIO

### HU-007 – Seguimiento y Reporte de Evolución Post-Consulta

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-007 |
| **Title**              | Como paciente en seguimiento quiero reportar mi evolución después de la consulta para que el sistema detecte empeoramiento o mejora de forma oportuna |
| **Parent (Feature)**   | F5.1 – Seguimiento Post-Consulta Automatizado / F5.2 – Timeline Evolutivo del Paciente |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Story Points**       | 8 |
| **Risk**               | Medium – Tasa de respuesta al formulario puede ser baja; mitigar con recordatorios pertinentes |
| **Dependencias**       | HU-006 (chat en tiempo real y consulta cerrada requerida como punto de partida) |
| **Tags**               | seguimiento; post-consulta; evolucion; timeline; recordatorios; must; sprint6 |

#### Descripción completa (campo Description)

```
Como paciente en seguimiento
Quiero reportar cómo me encuentro después de una consulta médica cerrada
Para que el sistema detecte si estoy mejorando o empeorando y notifique a mi médico
si necesito atención adicional de forma oportuna.

Contexto:
El seguimiento post-consulta es crítico para detectar deterioro clínico en pacientes
que no iniciaron una nueva consulta. Sin este mecanismo, el médico no tiene visibilidad
sobre la evolución del paciente entre consultas. El formulario de evolución es simple
y toma menos de 2 minutos en completarse desde la app móvil.

Flujo principal:
1. La consulta se cierra → el sistema crea automáticamente un Followup programado.
2. El paciente recibe una notificación (push/WebSocket) en la fecha programada.
3. El paciente abre la app y completa el formulario de evolución (intensidad, cambio,
   medicación, nuevos síntomas).
4. El sistema persiste las respuestas y evalúa si hay empeoramiento.
5. Si hay empeoramiento, el sistema notifica al médico y actualiza la prioridad.
6. El médico accede al timeline para revisar la evolución histórica del paciente.

Restricciones funcionales:
- La escala de intensidad de síntomas es de 1 (ninguno) a 10 (muy severo).
- El criterio de empeoramiento es: cambio = WORSE y delta de intensidad >= 2
  respecto al valor registrado en el triage original o en el último Followup.
- Un paciente solo puede completar un Followup por cada consulta cerrada.
- El formulario no sustituye una nueva consulta médica; el sistema incluye un
  mensaje de disclaimer si la intensidad es >= 8.
- El timeline es de solo lectura para el paciente; solo el médico puede agregar
  notas clínicas.

Notas de UX:
- La notificación push debe incluir el nombre del médico y la especialidad de la
  consulta para dar contexto al paciente.
- El formulario debe ser accesible en menos de 2 taps desde la notificación.
- Los indicadores de tendencia en el timeline deben ser visualmente claros (colores,
  iconos) para que el médico identifique rápidamente el estado del paciente.
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Seguimiento automatizado de evolución

  Scenario: Seguimiento principal completado con mejoría
    Given una consulta cerrada con prioridad MODERATE
    When llega la fecha de seguimiento y el paciente completa el formulario con mejoría
    Then el sistema guarda las respuestas en el Followup con status COMPLETED
    And el médico puede ver el evento en el timeline del paciente
    And la prioridad del caso no se modifica

  Scenario: Seguimiento alterno con empeoramiento detectado
    Given un paciente que reporta aumento de intensidad en 3 puntos respecto al triage
    When el sistema procesa la respuesta del formulario
    Then aumenta la prioridad del caso al nivel inmediatamente superior
    And notifica al médico responsable con los detalles del empeoramiento
    And emite el evento WebSocket consultation.priority.updated

  Scenario: Recordatorio enviado automáticamente al vencer el seguimiento
    Given un Followup con scheduledAt alcanzada y status PENDING
    When el scheduler ejecuta el job de recordatorios
    Then el paciente recibe notificación push y evento WebSocket
    And el Followup cambia a status REMINDED

  Scenario: Médico revisa timeline evolutivo completo
    Given un paciente con triage, consulta y seguimiento completado en el sistema
    When el médico accede al panel web y abre el timeline del paciente
    Then ve los eventos ordenados cronológicamente con tipos e iconos
    And puede acceder a cada documento fuente desde el timeline
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y features definidos (E5, F5.1, F5.2)
- [x] Prioridad MoSCoW asignada (Must)
- [x] Criterios Gherkin documentados (escenario principal y alterno)
- [x] Dependencias identificadas (HU-006 cerrada con consultas en estado CLOSED)
- [x] Estimación acordada (8 SP)
- [x] Datos/ambientes necesarios identificados (MongoDB dev, WebSocket dev, scheduler configurado)
- [x] Riesgos anotados (tasa de respuesta baja, recordatorios pertinentes)
- [x] Criterios de seguridad definidos (autorización por patientId, acceso solo al propio Followup)
- [x] KPI asociado identificado (KPI Retención de seguimiento a 7 días)
- [x] Responsable de validación funcional asignado

#### Definition of Done (DoD) – Checklist

- [ ] Documento Followup schema creado en MongoDB con todos los campos definidos
- [ ] Endpoint `POST /v1/followups` implementado y revisado por al menos un par
- [ ] Endpoint `GET /v1/patients/{patientId}/timeline` implementado con paginación
- [ ] Scheduler NestJS (cron) para recordatorios implementado y probado
- [ ] Motor de reglas de re-priorización implementado en FollowupService
- [ ] Evento WebSocket `consultation.followup.reminder.triggered` emitido correctamente
- [ ] Evento WebSocket `consultation.priority.updated` emitido al detectar empeoramiento
- [ ] Pantalla de formulario de evolución implementada en la app React Native
- [ ] Vista de timeline implementada en la app React Native (con filtros por tipo de evento)
- [ ] Vista de timeline implementada en el panel médico (Next.js)
- [ ] Código revisado por al menos un par
- [ ] Pruebas unitarias del FollowupService y motor de re-priorización en verde
- [ ] Pruebas de integración de los endpoints en verde
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] KPI retención a 7 días calculado y visible en el dashboard (referencia E6)
- [ ] SLO de timeline cumplido: P95 < 1500 ms verificado con prueba de carga básica
- [ ] Log de auditoría registra cada Followup completado y cada cambio de prioridad
- [ ] Validaciones de seguridad ejecutadas (autorización por rol y por ownership)
- [ ] Documentación técnica actualizada en Wiki (endpoints, contrato Followup, timeline)
- [ ] Demo funcional aprobada por el equipo
- [ ] Historia pasada a estado Done con evidencia enlazada (PR + test results)

---

### HU-012 – Gestión de Perfil de Familiar o Cuidador

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-012 |
| **Title**              | Como familiar cuidador quiero gestionar el perfil de un menor o adulto mayor para acompañar su atención dentro de la plataforma |
| **Parent (Feature)**   | F5.1 – Seguimiento Post-Consulta Automatizado |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Priority**           | 3 – Medium |
| **MoSCoW**             | Could |
| **Story Points**       | 8 |
| **Risk**               | Medium – Requiere modelo de datos adicional y control de permisos por custodia |
| **Dependencias**       | HU-001 (registro del cuidador como paciente base), HU-007 (seguimiento activo para el perfil gestionado) |
| **Tags**               | cuidador; perfil; menor; adulto-mayor; could; sprint6 |

#### Descripción completa (campo Description)

```
Como familiar cuidador
Quiero registrar y gestionar el perfil de salud de un menor o adulto mayor a mi cargo
Para poder reportar su evolución y acompañar su atención médica en la plataforma,
actuando en su nombre cuando el paciente no puede interactuar directamente.

Contexto:
Hay un segmento relevante de usuarios que no son el paciente directo sino un cuidador
(padre, madre, familiar) que actúa en representación. Sin este soporte, el cuidador
no puede completar el formulario de seguimiento ni consultar el timeline del paciente
a su cargo.

Flujo del proceso:
1. El cuidador (registrado como PACIENTE en el sistema) accede a la sección de
   "Perfiles gestionados" en su cuenta.
2. Crea un perfil dependiente: nombre, fecha de nacimiento, relación (padre/madre/
   cuidador/tutor legal), género (opcional).
3. La plataforma asocia el perfil dependiente al cuidador con el rol CAREGIVER_OF.
4. El cuidador puede iniciar triages, reportar seguimientos y ver el timeline
   en nombre del perfil dependiente.
5. El médico ve en el panel que la consulta es para un perfil gestionado, con
   el nombre del cuidador responsable identificado.

Restricciones:
- Un cuidador puede gestionar máximo 3 perfiles dependientes en el MVP.
- El perfil dependiente no tiene acceso autónomo a la plataforma (no puede
  iniciar sesión con sus propias credenciales en este sprint).
- La relación de custodia no se valida jurídicamente en el MVP; se declara
  como autodeclaración del cuidador.
- Esta historia es Could: solo se implementa si el sprint ha cumplido todas
  las historias Must y Should comprometidas con holgura suficiente.
- Si no se implementa en Sprint 6, se mueve al backlog de sprints posteriores.

Notas de UX:
- El cuidador debe poder cambiar de perfil activo desde la app sin necesidad
  de cerrar sesión.
- Las notificaciones de seguimiento se envían al cuidador, con el nombre del
  perfil dependiente en el asunto.
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Gestión de perfil de familiar cuidador

  Scenario: Cuidador crea perfil dependiente exitosamente
    Given un usuario con rol PACIENTE y cuenta activa
    When crea un perfil dependiente con nombre, fecha de nacimiento y relación
    Then el sistema persiste el perfil con status ACTIVE
    And asocia la relación CAREGIVER_OF entre el cuidador y el dependiente
    And retorna HTTP 201 con los datos del perfil creado

  Scenario: Cuidador reporta seguimiento en nombre de un dependiente
    Given un cuidador con un perfil dependiente activo
    And una consulta cerrada asociada al dependiente con un Followup PENDING
    When el cuidador completa el formulario de seguimiento en nombre del dependiente
    Then el sistema persiste el Followup con el patientId del dependiente
    And registra al cuidador como reporter del formulario
    And el médico puede ver en el timeline que el reporte fue hecho por el cuidador

  Scenario: Cuidador intenta crear más de 3 perfiles dependientes
    Given un cuidador que ya tiene 3 perfiles dependientes activos
    When intenta crear un cuarto perfil
    Then el sistema retorna HTTP 422 con mensaje de límite alcanzado
    And no crea el nuevo perfil

  Scenario: Médico identifica consulta de perfil gestionado
    Given una consulta iniciada por un cuidador en nombre de un dependiente
    When el médico abre la consulta en el panel web
    Then el sistema muestra el nombre del paciente dependiente
    And indica claramente que la consulta está siendo gestionada por un cuidador
    And muestra el nombre y relación del cuidador responsable
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y feature definidos (E5, F5.1)
- [x] Prioridad MoSCoW asignada (Could)
- [x] Criterios Gherkin documentados (principal y alterno)
- [x] Dependencias identificadas (HU-001 y HU-007)
- [x] Estimación acordada (8 SP)
- [x] Restricción de máximo 3 perfiles dependientes documentada
- [x] Riesgos anotados (modelo de datos de relación cuidador-dependiente)
- [x] Criterios de seguridad definidos (solo el cuidador propietario puede gestionar sus dependientes)
- [x] Condición de activación Sprint definida (solo si Must/Should completados con holgura)
- [x] Responsable de validación funcional asignado

#### Definition of Done (DoD) – Checklist

- [ ] Schema DependentProfile con relación CAREGIVER_OF creado en MongoDB
- [ ] Endpoint `POST /v1/patients/dependents` implementado para crear perfil dependiente
- [ ] Endpoint `GET /v1/patients/dependents` implementado para listar perfiles del cuidador
- [ ] Lógica de máximo 3 perfiles por cuidador implementada y probada
- [ ] Formulario de seguimiento permite seleccionar perfil dependiente en la app móvil
- [ ] Timeline del dependiente accesible al cuidador con identificación del reporter
- [ ] Panel médico muestra indicador de consulta gestionada por cuidador
- [ ] Código revisado por al menos un par
- [ ] Pruebas unitarias del CaregiverService en verde
- [ ] Pruebas de integración de los endpoints de cuidador en verde
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] Validaciones de seguridad ejecutadas (ownership, límite de perfiles)
- [ ] Documentación técnica actualizada en Wiki
- [ ] Demo funcional aprobada por el equipo
- [ ] Historia pasada a estado Done con evidencia enlazada (PR + test results)

---

## 4. TAREAS

### Tareas de HU-007 – Seguimiento y Reporte de Evolución Post-Consulta

---

#### T-007-01 – Diseño de modelo de datos Followup en MongoDB

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Diseñar esquema Mongoose Followup con todos los campos de seguimiento |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Crear el schema Mongoose para el modelo Followup:
- _id (ObjectId automático)
- consultationId (ObjectId, ref: 'Consultation', required, unique)
- patientId (ObjectId, ref: 'Patient', required)
- doctorId (ObjectId, ref: 'Doctor', required)
- scheduledAt (Date, required) – fecha programada de seguimiento (ahora + 72h)
- reminderAt (Date, required) – fecha de recordatorio a 7 días
- status: enum ['PENDING', 'REMINDED', 'COMPLETED', 'EXPIRED'], default: 'PENDING'
- responses (objeto embebido, nullable):
    · intensityScore (Number, min: 1, max: 10)
    · changePerceived: enum ['BETTER', 'SAME', 'WORSE']
    · medicationTaken (Boolean)
    · newSymptoms (String, optional, maxLength: 500)
    · completedAt (Date)
    · reportedBy (ObjectId, ref: 'Patient') – puede ser el cuidador
- priorityEscalated (Boolean, default: false)
- escalationReason (String, optional)
- timestamps

Índices:
- { consultationId: 1 } único
- { patientId: 1, status: 1 }
- { scheduledAt: 1, status: 1 } – para el scheduler

Ubicar en: apps/api/src/followups/schemas/followup.schema.ts
```

**Criterios de aceptación de la tarea:**
- Schema creado y exportado correctamente.
- Índices definidos y verificados en MongoDB local.
- Restricción unique en consultationId verificada (un Followup por consulta).
- El campo responses es nullable hasta que el paciente complete el formulario.

---

#### T-007-02 – Creación automática de Followup al cerrar consulta

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar creación automática de Followup al cambiar estado de consulta a CLOSED |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el hook en ConsultationService que crea el Followup al cerrar una consulta:
- Al ejecutar la transición de estado a CLOSED en una consulta, el servicio llama
  a FollowupService.createFromConsultation(consultation).
- FollowupService calcula:
    · scheduledAt = Date.now() + 72 * 60 * 60 * 1000 (72 horas)
    · reminderAt = Date.now() + 7 * 24 * 60 * 60 * 1000 (7 días)
- Persiste el Followup con status PENDING.
- Si ya existe un Followup para la consultationId (idempotencia), retorna el existente
  sin crear duplicados.
- Registrar el evento en log estructurado con correlation ID.
- La creación del Followup es parte de la misma operación lógica que el cierre
  de la consulta (misma sesión de MongoDB si es posible).

Ubicar en: apps/api/src/followups/followup.service.ts
         apps/api/src/consultations/consultation.service.ts (hook)
```

**Criterios de aceptación de la tarea:**
- Al cerrar una consulta, existe un Followup en MongoDB con los campos correctos.
- scheduledAt y reminderAt están correctamente calculados (±1 segundo de tolerancia).
- No se crea un segundo Followup si el método se llama dos veces para la misma consulta.
- El log registra el evento de creación con consultationId y patientId.

---

#### T-007-03 – Implementación del endpoint POST /v1/followups

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint POST /v1/followups para registro de evolución del paciente |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar el endpoint de registro del formulario de evolución:
- Módulo: FollowupsModule (NestJS)
- Ruta: POST /v1/followups
- Protegido con JwtAuthGuard y rol PACIENTE (o CAREGIVER si HU-012 activa).
- DTO: SubmitFollowupDto {
    followupId: string (ObjectId),
    intensityScore: number (1-10),
    changePerceived: 'BETTER' | 'SAME' | 'WORSE',
    medicationTaken: boolean,
    newSymptoms?: string (max 500 chars)
  }
- Lógica:
    1. Buscar el Followup por followupId → 404 si no existe.
    2. Verificar que el patientId del Followup coincide con el del JWT → 403 si no.
    3. Verificar que status != 'COMPLETED' → 409 si ya fue completado.
    4. Persistir responses con completedAt = Date.now() y reportedBy = patientId del JWT.
    5. Actualizar status a 'COMPLETED'.
    6. Llamar a FollowupService.evaluateEscalation(followup) para evaluar re-priorización.
    7. Si se dispara escalación, emitir evento WebSocket consultation.priority.updated.
    8. Registrar evento en log estructurado.
- Respuesta HTTP 200: { followupId, status, priorityEscalated, newPriority? }

Motor de re-priorización (evalueEscalation):
- Obtener intensityBaseline del triage original (TriageSession.intensityScore) o del
  último Followup completado previo.
- delta = intensityScore actual - intensityBaseline
- Si (changePerceived = 'WORSE' AND delta >= 2) O presencia de red flag en newSymptoms:
    · Incrementar prioridad: LOW → MODERATE, MODERATE → HIGH
    · Actualizar el campo priority de la Consultation.
    · Crear notificación interna para el médico responsable.
    · Marcar priorityEscalated = true en el Followup.
    · Emitir evento WebSocket consultation.priority.updated al room del médico.
- Si intensityScore >= 8 en cualquier cambio: agregar disclaimer en la respuesta HTTP
  recomendando iniciar nueva consulta.

Ubicar en: apps/api/src/followups/
```

**Criterios de aceptación de la tarea:**
- Endpoint retorna 200 con datos correctos al recibir payload válido.
- Retorna 404 si el followupId no existe.
- Retorna 403 si el paciente no es dueño del Followup.
- Retorna 409 si el Followup ya fue completado.
- Con delta >= 2 y WORSE, la prioridad se actualiza y el evento WebSocket se emite.
- El disclaimer aparece en la respuesta si intensityScore >= 8.
- Nunca dos Followups completados para la misma consulta.

---

#### T-007-04 – Scheduler de recordatorios de seguimiento

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar scheduler NestJS para envío de recordatorios de Followup vencidos |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar un job programado con @nestjs/schedule para procesar Followups vencidos:
- Cron expression: '*/15 * * * *' (cada 15 minutos).
- El job consulta Followups con:
    · status: 'PENDING'
    · scheduledAt <= Date.now()
- Para cada Followup encontrado:
    1. Emitir evento WebSocket consultation.followup.reminder.triggered al paciente
       (room: `patient:${patientId}`) con payload { followupId, consultationId, message }.
    2. Actualizar status a 'REMINDED'.
    3. Registrar el evento en log estructurado.
- El job es idempotente: si el paciente completó el Followup antes de que el job
  lo procese, el status ya es COMPLETED y el job lo omite.
- Segundo job para reminderAt (7 días): misma lógica pero para Followups con
  status 'REMINDED' y reminderAt <= Date.now(); emite recordatorio de 7 días y
  actualiza schedulerReminder7DaysSent = true (campo opcional).
- Manejo de errores: si el emit WebSocket falla, el job registra el error pero
  continúa con el siguiente Followup (no bloquea el batch).

Ubicar en: apps/api/src/followups/followup-scheduler.service.ts
```

**Criterios de aceptación de la tarea:**
- El job se ejecuta cada 15 minutos y procesa Followups vencidos.
- El evento WebSocket llega al paciente correcto al ejecutar el job.
- El Followup cambia de PENDING a REMINDED tras el procesamiento.
- Followups ya COMPLETED son omitidos por el job.
- Los errores de WebSocket no detienen el procesamiento del batch.

---

#### T-007-05 – Implementación del endpoint GET /v1/patients/{patientId}/timeline

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint GET /v1/patients/{patientId}/timeline con paginación por cursor |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 5 |

**Descripción:**
```
Implementar el endpoint de timeline evolutivo del paciente:
- Ruta: GET /v1/patients/{patientId}/timeline
- Protegido con JwtAuthGuard.
- Control de acceso:
    · PACIENTE: solo puede ver su propio timeline (patientId del JWT = patientId param).
    · MEDICO: solo puede ver el timeline de pacientes con los que tiene una consulta.
    · ADMIN: puede ver cualquier timeline.
- Query params: cursor? (string, ObjectId codificado en base64), limit? (number, default 20, max 50)
- Lógica de agregación:
    · Recopilar eventos de las colecciones:
        - TriageSession: eventType='TRIAGE', occurredAt=createdAt, summary=resumen de prioridad
        - Consultation: eventType='CONSULTATION_OPENED'/'CONSULTATION_CLOSED', occurredAt
        - Message: eventType='MESSAGE', occurredAt=sentAt, summary=primeros 80 chars del mensaje
        - Followup: eventType='FOLLOWUP_COMPLETED', occurredAt=responses.completedAt
        - PriorityChange (subdocumento o colección separada): eventType='PRIORITY_CHANGED'
    · Ordenar todos los eventos por occurredAt descendente.
    · Aplicar paginación con cursor (usar el ObjectId del último evento visto).
    · Calcular indicador de tendencia entre Followups consecutivos (delta de intensidad).
- Respuesta HTTP 200:
    {
      events: [{ eventType, occurredAt, actor, summary, referenceId, metadata }],
      nextCursor: string | null,
      total: number (aproximado, no exacto para performance)
    }
- Cachear la respuesta por 30 segundos en memoria (Map<patientId, cachedResponse>)
  para reducir carga en consultas frecuentes desde el panel médico.
- El tiempo de respuesta debe cumplir P95 < 1500 ms incluso con 100 eventos.

Ubicar en: apps/api/src/patients/patients.controller.ts
           apps/api/src/patients/timeline.service.ts
```

**Criterios de aceptación de la tarea:**
- Endpoint retorna eventos de todas las fuentes en orden cronológico descendente.
- Paginación con cursor funciona correctamente para historiales extensos.
- PACIENTE recibe 403 al intentar ver el timeline de otro paciente.
- MEDICO recibe 403 al intentar ver el timeline de un paciente sin consultas previas.
- El tiempo de respuesta con 100 eventos es < 1500 ms (verificado con prueba básica).
- El indicador de tendencia aparece en Followups donde hay delta >= 2.

---

#### T-007-06 – Pantalla de formulario de evolución en React Native

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar pantalla de formulario de evolución post-consulta en la app móvil |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 5 |

**Descripción:**
```
Implementar la pantalla de seguimiento en la app React Native:

FollowupFormScreen:
- Navegación: accesible desde notificación push, desde la pantalla de historial
  y desde el dashboard del paciente (sección "Seguimientos pendientes").
- Campos:
    · Slider de intensidad de síntomas: 1 (ninguno) a 10 (muy severo), con etiquetas.
    · Selector de cambio percibido: botones MEJOR / IGUAL / PEOR con iconos.
    · Toggle de medicación tomada (Sí / No).
    · Campo de texto libre: "¿Nuevos síntomas?" (opcional, max 500 chars, con contador).
- Botón "Enviar seguimiento" llama a POST /v1/followups.
- Si intensityScore >= 8, mostrar aviso antes de enviar: "Tu nivel de síntomas es alto.
  Te recomendamos iniciar una nueva consulta si el malestar es urgente."
- Al éxito, mostrar pantalla de confirmación con mensaje motivacional y opción de
  ver el timeline.
- Al error, mostrar mensaje de error sin perder los datos ingresados.

Manejo de notificación push:
- Al recibir el evento consultation.followup.reminder.triggered, mostrar una
  notificación local con deep link a FollowupFormScreen con el followupId.
- La app debe manejar el caso de recibir la notificación con la app cerrada y
  abierta en segundo plano.

Ubicar en: apps/mobile/src/screens/followup/
```

**Criterios de aceptación de la tarea:**
- El slider, selector y toggle funcionan correctamente en iOS y Android.
- El aviso de intensidad alta aparece solo si intensityScore >= 8.
- Envío exitoso muestra confirmación y navega al historial.
- La notificación push abre directamente el formulario correcto.
- La app no pierde los datos del formulario si hay un error de red al enviar.

---

#### T-007-07 – Vista de timeline en la app React Native

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar vista de timeline evolutivo del paciente en la app móvil |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar la pantalla de historial/timeline en la app React Native:

PatientTimelineScreen:
- Lista vertical de eventos en orden cronológico descendente (más reciente arriba).
- Cada ítem muestra:
    · Icono por eventType (triage, consulta, mensaje, seguimiento, cambio de prioridad).
    · Fecha relativa (ej. "hace 2 días") con tooltip de fecha exacta al pulsar.
    · Summary del evento (texto breve).
    · Indicador visual de tendencia para Followups (flecha verde/roja si hay delta >= 2).
- Filtro por tipo de evento: chips horizontales filtrables (Todos / Consultas /
  Seguimientos / Mensajes).
- Paginación: carga más eventos al llegar al final de la lista (infinite scroll con cursor).
- Navegación al documento fuente al pulsar un evento (consulta → pantalla de consulta,
  followup → pantalla de detalle del followup).
- Estado vacío: mensaje motivacional si no hay eventos aún.

Ubicar en: apps/mobile/src/screens/patient/PatientTimelineScreen.tsx
```

**Criterios de aceptación de la tarea:**
- La lista muestra todos los tipos de eventos con sus iconos correspondientes.
- El filtro por tipo de evento funciona correctamente.
- El infinite scroll carga más eventos al llegar al final.
- El indicador de tendencia es visible en Followups con delta >= 2.
- Tap en un evento navega al documento fuente correspondiente.

---

#### T-007-08 – Vista de timeline en el panel médico (Next.js)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar vista de timeline del paciente en el panel médico web (Next.js) |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Web |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar en el panel médico (Next.js) la vista de timeline del paciente:

PatientTimelinePage:
- Ruta del panel: /doctor/patients/{patientId}/timeline
- Tabla cronológica con columnas: Fecha, Tipo de evento, Actor, Resumen, Acciones.
- Badge de color por tipo de evento: azul (consulta), naranja (seguimiento), verde (triage),
  gris (mensaje), rojo/amarillo (cambio de prioridad).
- Indicador de tendencia: fila con fondo rojo claro si el Followup reportó empeoramiento
  con escalación de prioridad.
- Botón "Ver detalle" por fila que abre el documento fuente en una pestaña/modal.
- Filtro lateral: tipo de evento, rango de fechas.
- Paginación de tabla: 20 eventos por página con navegación.
- La página solo es visible para médicos con consultas activas o cerradas del paciente;
  redirige a /403 si no tiene acceso.

Ubicar en: apps/web/src/app/doctor/patients/[patientId]/timeline/
```

**Criterios de aceptación de la tarea:**
- La tabla muestra todos los eventos con badges de color por tipo.
- Las filas de Followup con empeoramiento tienen fondo de advertencia visible.
- El botón "Ver detalle" abre el documento fuente correcto.
- Los filtros de tipo y fecha funcionan correctamente.
- Acceso desde un médico sin consultas con el paciente redirige a /403.

---

#### T-007-09 – Pruebas unitarias del FollowupService y motor de re-priorización

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias para FollowupService incluyendo motor de re-priorización |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Escribir pruebas unitarias con Jest para FollowupService y el motor de re-priorización:

FollowupService:
- createFromConsultation: crea correctamente con scheduledAt/reminderAt calculados.
- createFromConsultation (idempotencia): retorna el existente si ya hay un Followup.
- submitFollowup: persiste responses y status=COMPLETED.
- submitFollowup: retorna 409 si ya está COMPLETED.
- submitFollowup: retorna 403 si patientId no coincide.

Motor de re-priorización (evaluateEscalation):
- delta < 2 con WORSE: no escala.
- delta >= 2 con WORSE: escala LOW → MODERATE.
- delta >= 2 con WORSE y prioridad MODERATE: escala a HIGH.
- delta >= 2 con BETTER: no escala (aunque delta sea > 2, la mejora no escala).
- newSymptoms con red flag keyword: escala independientemente del delta.
- intensityScore >= 8 con BETTER: no escala pero incluye disclaimer.

TimelineService:
- aggregateTimeline: retorna eventos ordenados descendentemente de todas las fuentes.
- aggregateTimeline: aplica paginación con cursor correctamente.
- aggregateTimeline: retorna 403 si el paciente no es dueño del timeline.

Mocks de todos los repositorios. Sin dependencias reales de BD.
Cobertura mínima objetivo: 85% del FollowupService y TimelineService.

Ubicar en: apps/api/src/followups/followup.service.spec.ts
           apps/api/src/patients/timeline.service.spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI (`npm run test`).
- Cobertura >= 85% de FollowupService y TimelineService.
- Los escenarios de re-priorización cubren todos los casos de la lógica de negocio.
- No hay dependencias reales de base de datos en estas pruebas.

---

#### T-007-10 – Pruebas de integración de los endpoints de seguimiento y timeline

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas de integración para POST /v1/followups y GET /v1/patients/{id}/timeline |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Pruebas de integración con supertest + MongoMemoryServer:

POST /v1/followups:
- Paciente completa seguimiento con BETTER → 200, status=COMPLETED, no escala.
- Paciente completa seguimiento con WORSE + delta 2 → 200, priorityEscalated=true.
- Paciente intenta completar Followup ajeno → 403.
- Followup inexistente → 404.
- Followup ya COMPLETED → 409.

GET /v1/patients/{patientId}/timeline:
- Médico con consulta activa del paciente → 200 con eventos ordenados.
- Paciente ve su propio timeline → 200 con eventos.
- Paciente intenta ver timeline ajeno → 403.
- Médico sin relación con el paciente → 403.
- Timeline vacío → 200 con events=[] y nextCursor=null.
- Paginación: timeline con más de 20 eventos retorna nextCursor.

Ubicar en: apps/api/test/followups.e2e-spec.ts
           apps/api/test/timeline.e2e-spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests de integración pasan en CI de forma aislada.
- Se cubre el flujo completo de seguimiento y re-priorización.
- La paginación del timeline es verificada con datos reales de prueba.

---

#### T-007-11 – Prueba de rendimiento básica del endpoint de timeline

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Ejecutar prueba de rendimiento básica del GET /v1/patients/{id}/timeline para verificar SLO P95 < 1500 ms |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Ejecutar una prueba básica de rendimiento para verificar el SLO del timeline:
- Herramienta: k6 o Artillery (la que esté disponible en el proyecto).
- Escenario: 20 usuarios virtuales concurrentes solicitando el timeline de un paciente
  con 100 eventos durante 60 segundos.
- Métrica objetivo: P95 latencia < 1500 ms.
- Si el SLO no se cumple, identificar el cuello de botella (consulta MongoDB, agregación,
  serialización) y aplicar optimización (índices adicionales, proyección limitada).
- Documentar resultados en el comentario de la tarea y en el PR.
- No bloquear el sprint si el SLO se cumple con margen; si no se cumple, abrir
  una tarea de deuda técnica para Sprint 9.

Ubicar resultado en: docs/tests/performance/timeline-slo-sprint6.md
```

**Criterios de aceptación de la tarea:**
- Prueba ejecutada y resultado documentado (pass o fail con causa).
- Si SLO cumplido: evidencia en PR (output de k6/Artillery).
- Si SLO no cumplido: tarea de deuda técnica abierta con causa identificada.

---

#### T-007-12 – Documentación técnica de endpoints de seguimiento y timeline en Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar endpoints de seguimiento y timeline del paciente en la Wiki del proyecto |
| **Parent (User Story)**| HU-007 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 1 |

**Descripción:**
```
Actualizar la Wiki de Azure DevOps con:
- POST /v1/followups: request body (SubmitFollowupDto), response 200, errores 403/404/409.
- GET /v1/patients/{patientId}/timeline: query params, respuesta con estructura de eventos,
  paginación con cursor, roles permitidos.
- Descripción del motor de re-priorización: criterios de escalación, niveles de prioridad.
- Contrato del objeto Followup con todos los campos.
- Estructura del evento de timeline con todos los eventTypes.
- Descripción del scheduler de recordatorios: frecuencia, lógica, idempotencia.
- Referencia a HU-007 y al sprint de implementación.
- Actualización del KPI "Retención de seguimiento a 7 días": definición, cálculo,
  fuente de datos.
```

**Criterios de aceptación de la tarea:**
- La Wiki refleja los contratos reales de los endpoints.
- El motor de re-priorización es comprensible para un desarrollador nuevo sin leer el código.
- El KPI de retención queda documentado con su fórmula de cálculo.

---

### Tareas de HU-012 – Gestión de Perfil de Familiar o Cuidador

> **Nota:** Las tareas de HU-012 solo se ejecutan si el sprint ha cumplido todas las historias Must y Should con holgura suficiente. De lo contrario, se mueven al backlog de sprints posteriores.

---

#### T-012-01 – Diseño de modelo de datos DependentProfile y relación CaregiverOf

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Diseñar esquema Mongoose DependentProfile con relación CaregiverOf |
| **Parent (User Story)**| HU-012 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Crear el schema Mongoose para el perfil dependiente:

DependentProfile schema:
- _id (ObjectId automático)
- caregiverId (ObjectId, ref: 'Patient', required) – el cuidador registrado
- firstName, lastName (String, required)
- birthDate (Date, required)
- gender (String, enum: ['M', 'F', 'OTHER'], optional)
- relationship: enum ['PARENT', 'SIBLING', 'LEGAL_GUARDIAN', 'OTHER'], required
- status: enum ['ACTIVE', 'INACTIVE'], default: 'ACTIVE'
- timestamps

Validación de negocio (a nivel de servicio):
- Máximo 3 perfiles ACTIVE por caregiverId (verificar antes de crear).

Índice:
- { caregiverId: 1, status: 1 }

Ubicar en: apps/api/src/patients/schemas/dependent-profile.schema.ts
```

**Criterios de aceptación de la tarea:**
- Schema creado y exportado correctamente.
- La relación con el cuidador por ObjectId está definida.
- El límite de 3 perfiles se valida en el servicio antes de persistir.

---

#### T-012-02 – Implementación de endpoints de gestión de perfiles dependientes

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoints POST y GET /v1/patients/dependents |
| **Parent (User Story)**| HU-012 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar los endpoints para gestión de perfiles dependientes:

POST /v1/patients/dependents:
- Protegido con JwtAuthGuard y rol PACIENTE.
- DTO: CreateDependentProfileDto { firstName, lastName, birthDate, gender?, relationship }
- Lógica:
    1. Contar perfiles ACTIVE del caregiverId → 422 si ya hay 3.
    2. Crear DependentProfile con caregiverId = userId del JWT.
    3. Respuesta HTTP 201 con los datos del perfil creado.

GET /v1/patients/dependents:
- Protegido con JwtAuthGuard y rol PACIENTE.
- Retorna todos los perfiles ACTIVE del cuidador autenticado.
- Respuesta HTTP 200: [{ id, firstName, lastName, birthDate, relationship }]

Adaptar POST /v1/followups para soportar reportedBy opcional:
- Si el payload incluye dependentProfileId, verificar que el cuidador es dueño
  del perfil → 403 si no lo es.
- Persistir el Followup con patientId = dependentProfile._id y reportedBy = cuidadorId.

Ubicar en: apps/api/src/patients/
```

**Criterios de aceptación de la tarea:**
- POST crea el perfil y retorna 201.
- POST retorna 422 si el cuidador ya tiene 3 perfiles activos.
- GET retorna solo los perfiles del cuidador autenticado.
- El Followup soporta reportedBy diferente al patientId.

---

#### T-012-03 – Pantalla de gestión de perfiles dependientes en React Native

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar pantalla de gestión de perfiles dependientes en la app móvil |
| **Parent (User Story)**| HU-012 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar la pantalla de gestión de perfiles en la app React Native:

DependentProfilesScreen:
- Ruta: accesible desde el perfil del usuario (sección "Perfiles a mi cargo").
- Lista de perfiles dependientes activos con nombre, fecha de nacimiento y relación.
- Botón "Agregar perfil" que abre un modal con el formulario de creación.
- Botón de cambio de perfil activo: al seleccionar un perfil, las acciones de triage,
  consulta y seguimiento se realizan en nombre del dependiente.
- Indicador visual del perfil activo actual (chip en el header de la app).
- Mensaje de límite si el cuidador intenta agregar un 4to perfil.

Selector de perfil en formularios de seguimiento:
- En FollowupFormScreen, si el cuidador tiene perfiles dependientes, mostrar un
  selector en la parte superior: "¿Para quién es este seguimiento?" con opción
  de elegir el propio cuidador o uno de sus dependientes.

Ubicar en: apps/mobile/src/screens/profile/DependentProfilesScreen.tsx
```

**Criterios de aceptación de la tarea:**
- La lista muestra los perfiles del cuidador con sus datos.
- El formulario de creación valida campos y muestra errores inline.
- El mensaje de límite aparece al intentar agregar un 4to perfil.
- El selector de perfil aparece en FollowupFormScreen cuando hay dependientes.
- El cambio de perfil activo se refleja visualmente en el header de la app.

---

#### T-012-04 – Pruebas unitarias y de integración de gestión de cuidador

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias e integración para gestión de perfiles dependientes |
| **Parent (User Story)**| HU-012 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 6 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Pruebas unitarias (Jest):
- createDependentProfile: crea correctamente el perfil.
- createDependentProfile: retorna 422 si ya hay 3 perfiles activos.
- submitFollowupForDependent: persiste con patientId del dependiente y reportedBy del cuidador.
- submitFollowupForDependent: retorna 403 si el cuidador no es dueño del perfil.

Pruebas de integración (supertest + MongoMemoryServer):
- POST /v1/patients/dependents: crea perfil y retorna 201.
- POST /v1/patients/dependents: límite de 3 retorna 422.
- GET /v1/patients/dependents: retorna solo perfiles del cuidador.
- POST /v1/followups con dependentProfileId: persiste correctamente.
- POST /v1/followups con perfil ajeno: retorna 403.

Ubicar en: apps/api/src/patients/caregiver.service.spec.ts
           apps/api/test/caregiver.e2e-spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI.
- El límite de 3 perfiles está cubierto por tests unitarios y de integración.
- La lógica de seguimiento para dependientes está cubierta.

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
📌 EPIC: E5 – Seguimiento y Evolución del Paciente (16 SP total)
│
├── 🔷 FEATURE: F5.1 – Seguimiento Post-Consulta Automatizado (8 SP)
│   │
│   └── 📖 USER STORY: HU-007 – Como paciente quiero reportar evolución post-consulta (8 SP)
│       ├── ✅ T-007-01 – Diseño schema Mongoose Followup (2h)
│       ├── ✅ T-007-02 – Creación automática de Followup al cerrar consulta (3h)
│       ├── ✅ T-007-03 – Endpoint POST /v1/followups con motor de re-priorización (4h)
│       ├── ✅ T-007-04 – Scheduler NestJS de recordatorios de Followup vencidos (3h)
│       ├── ✅ T-007-06 – Pantalla formulario de evolución en React Native (5h)
│       ├── ✅ T-007-09 – Pruebas unitarias FollowupService y re-priorización (4h)
│       ├── ✅ T-007-10 – Pruebas de integración POST /v1/followups (3h)
│       └── ✅ T-007-12 – Documentación técnica endpoints seguimiento en Wiki (1h)
│
├── 🔷 FEATURE: F5.2 – Timeline Evolutivo del Paciente (8 SP)
│   │
│   └── 📖 USER STORY: HU-007 – (continúa, cubre también F5.2)
│       ├── ✅ T-007-05 – Endpoint GET /v1/patients/{id}/timeline con paginación (5h)
│       ├── ✅ T-007-07 – Vista de timeline en app React Native (4h)
│       ├── ✅ T-007-08 – Vista de timeline en panel médico Next.js (4h)
│       ├── ✅ T-007-10 – Pruebas de integración GET /v1/patients/{id}/timeline (incluido arriba)
│       └── ✅ T-007-11 – Prueba de rendimiento SLO P95 < 1500 ms (2h)
│
└── 🔷 FEATURE: F5.1 (extensión Could) – Perfil de Familiar Cuidador
    │
    └── 📖 USER STORY: HU-012 – Como familiar cuidador quiero gestionar perfil de dependiente (8 SP)
        ├── ✅ T-012-01 – Diseño schema DependentProfile y relación CaregiverOf (2h)
        ├── ✅ T-012-02 – Endpoints POST/GET /v1/patients/dependents (4h)
        ├── ✅ T-012-03 – Pantalla de gestión de perfiles en React Native (4h)
        └── ✅ T-012-04 – Pruebas unitarias e integración de gestión de cuidador (3h)
```

**Total horas estimadas Sprint 6 (Épica 5):**

| Historia | Tareas | Horas |
|----------|--------|-------|
| HU-007   | T-007-01 a T-007-12 | 40 horas |
| HU-012   | T-012-01 a T-012-04 | 13 horas |
| **Total** | | **53 horas** |

**Total Story Points Épica 5:** 16 SP (HU-007: 8 SP + HU-012: 8 SP Could)  
**Sprint objetivo:** Sprint 6

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|-----------|-----------|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` – Sprint 6: Seguimiento y timeline |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` – Actividad 4: Continuidad; Slice C: E5+E6 |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` – HU-007, HU-012 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` – HU-007 |
| DoR / DoD | `docs/wiki/08-DoR-DoD.md` |
| KPIs y SLOs | `docs/wiki/09-Observabilidad-KPIs.md` |
| Plan de Sprints | `docs/wiki/11-Plan-Sprints-0-a-9.md` – Sprint 6 |
| Riesgos | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` |
| Épica anterior (dependencia) | `docs/epics/Azure-Boards-Epica3.md` – E3 Chat Clínico (HU-006) |
| Épica siguiente | `docs/epics/Azure-Boards-Epica4.md` – E4 Resumen Clínico (contexto del pipeline clínico) |

### API endpoints de esta épica (contratos del Plan Maestro)

| Endpoint | Tarea asociada | Descripción |
|----------|---------------|-------------|
| `POST /v1/followups` | T-007-03 | Registrar respuesta de formulario de evolución |
| `GET /v1/patients/{id}/timeline` | T-007-05 | Obtener timeline evolutivo paginado |
| `POST /v1/patients/dependents` | T-012-02 | Crear perfil dependiente (cuidador) |
| `GET /v1/patients/dependents` | T-012-02 | Listar perfiles dependientes del cuidador |

### WebSocket events de esta épica

| Evento | Namespace | Descripción |
|--------|-----------|-------------|
| `consultation.followup.reminder.triggered` | consultation | Recordatorio de seguimiento al paciente |
| `consultation.priority.updated` | consultation | Notificación al médico por re-priorización |

### KPIs impactados por esta épica

- **KPI Retención de seguimiento a 7 días** (obligatorio): porcentaje de consultas cerradas
  con al menos un Followup completado dentro de los 7 días posteriores al cierre.
  Fórmula: `(Followups COMPLETED con completedAt <= consultationClosedAt + 7d) / total Consultas CLOSED * 100`.
- **KPI Tasa de red flags relevantes confirmadas**: si el motor de re-priorización detecta
  empeoramiento, aporta datos a este KPI cuando el médico confirma la relevancia.
- **SLO P95 latencia < 1500 ms**: aplica al endpoint de timeline; verificado en T-007-11.
- **SLO Disponibilidad >= 98%**: el módulo de seguimiento es parte del core del MVP; su
  indisponibilidad afecta la retención y la detección de empeoramiento.

### Riesgos asociados

- **R-002 (Concurrencia real-time)**: el scheduler de recordatorios y los eventos WebSocket
  del módulo de seguimiento contribuyen a la carga de conexiones concurrentes. Verificar
  con prueba de concurrencia en Sprint 9.
- **R-005 (Sobrealcance)**: HU-012 (perfil de cuidador) es Could y puede removerse del
  sprint si el tiempo no alcanza, sin impacto en los Must del MVP.
- **R-007 (Calidad de datos de observabilidad)**: el KPI de retención a 7 días depende
  de que los Followups se registren con timestamps correctos; validar en Sprint 7 al
  implementar el dashboard.
