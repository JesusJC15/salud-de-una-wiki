# Azure Boards – Épica 3: Consulta Clínica en Tiempo Real

## Propósito del documento
Este documento contiene toda la información necesaria para implementar la **Épica 3 – Consulta Clínica en Tiempo Real** en Azure Boards, incluyendo la épica, sus features, historias de usuario con criterios Gherkin, y todas las tareas de desarrollo, pruebas y documentación asociadas. Cada sección indica los campos exactos que se deben completar al crear el ítem en Azure Boards.

---

## 1. ÉPICA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Epic |
| **ID**                 | E3 |
| **Title**              | Consulta Clínica en Tiempo Real |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Priority**           | 1 – Critical |
| **Business Value**     | Mejorar velocidad y continuidad de atención; habilitar la interacción directa paciente-médico en tiempo real |
| **Risk**               | High – Latencia en WebSocket y manejo de concurrencia de sesiones simultáneas |
| **Effort (Story Points total estimado)** | 13 SP |
| **Tags**               | chat; websocket; tiempo-real; consulta; cola; must; sprint5 |
| **Start Date**         | Sprint 5 |
| **Target Date**        | Cierre Sprint 5 |

### Descripción (campo Description)

```
La Épica 3 cubre el ciclo completo de la consulta clínica en tiempo real entre paciente
y médico dentro de SaludDeUna.

Objetivos de negocio:
- Permitir a pacientes y médicos intercambiar mensajes clínicos en tiempo real dentro
  de una consulta activa, reduciendo el tiempo a primera respuesta médica.
- Brindar al médico una cola de casos priorizada según el nivel de urgencia asignado
  por el motor de triage, facilitando la gestión de múltiples consultas concurrentes.
- Garantizar la persistencia y el orden cronológico de los mensajes aunque haya
  reconexiones o pérdidas momentáneas de red.

Funcionalidades cubiertas:
- Creación de consultas clínicas a partir de un triage finalizado.
- Cola de casos priorizada (HIGH → MODERATE → LOW) visible para el médico desde
  el panel web.
- Canal de mensajería bidireccional paciente-médico mediante WebSocket.
- Persistencia de mensajes en MongoDB y entrega con acuse lógico (ACK).
- Cambio de estado de consulta (OPEN → IN_PROGRESS → CLOSED) con eventos
  en tiempo real para ambos clientes.
- Notificaciones internas de nuevos mensajes y cambios de estado.
- Actualización automática de la cola cuando se incorpora una nueva consulta de
  alta prioridad.

Restricciones:
- No incluye videollamada ni audio (fuera de alcance MVP).
- El resumen clínico preconsulta es generado en E4 (HU-005); E3 solo lo consume y
  lo muestra al médico al abrir la consulta.
- La autenticación de la sesión WebSocket reutiliza el JWT emitido en E1 (HU-001).
- No se implementa cifrado extremo a extremo de mensajes en este sprint; se planifica
  para el hardening de Sprint 9.
```

### Criterios de aceptación de la épica

```
- Un paciente puede crear una consulta tras completar el triage y ver el estado
  de la misma en tiempo real.
- Un médico VERIFIED puede ver la cola de casos ordenada por prioridad y abrir
  cualquiera de ellos para iniciar la consulta.
- Paciente y médico intercambian mensajes que aparecen en ambas interfaces sin
  necesidad de refrescar la página o la app.
- Cuando un mensaje se envía y la contraparte está desconectada temporalmente,
  al reconectarse recibe todos los mensajes pendientes en orden cronológico.
- El estado de la consulta (OPEN, IN_PROGRESS, CLOSED) se refleja en tiempo real
  en ambos clientes al cambiar.
- La cola del médico se actualiza automáticamente cuando se agrega una consulta
  de prioridad HIGH.
- El log de auditoría registra la creación de consultas, el envío de mensajes y
  cada cambio de estado.
```

### Acceptance Criteria – formato Gherkin (nivel épica)

```gherkin
Feature: Consulta clínica en tiempo real en SaludDeUna

  Scenario: Paciente inicia consulta tras triage completado
    Given un paciente autenticado con un triage finalizado y prioridad asignada
    When crea una nueva consulta desde la app
    Then el sistema registra la consulta en estado OPEN
    And la inserta en la cola médica en la posición correspondiente a su prioridad

  Scenario: Médico ve cola priorizada y abre consulta
    Given un médico VERIFIED autenticado en el panel web
    When accede a su cola de casos
    Then ve las consultas ordenadas de mayor a menor prioridad (HIGH → MODERATE → LOW)
    And al abrir una consulta puede ver el resumen clínico preconsulta

  Scenario: Intercambio de mensajes en tiempo real
    Given una consulta en estado IN_PROGRESS con paciente y médico conectados
    When cualquiera de los dos envía un mensaje
    Then el otro lo recibe en tiempo real sin recargar la interfaz
    And el mensaje queda persistido en la base de datos

  Scenario: Sincronización tras reconexión
    Given un paciente que pierde conectividad durante una consulta activa
    When recupera la conexión
    Then la app sincroniza todos los mensajes enviados durante la desconexión
    And los muestra en orden cronológico correcto

  Scenario: Médico cierra consulta
    Given una consulta en estado IN_PROGRESS
    When el médico cambia el estado a CLOSED
    Then el paciente recibe el evento consultation.status.changed en tiempo real
    And la consulta deja de aparecer en la cola activa del médico
```

---

## 2. FEATURES

### Feature F3.1 – Chat Clínico en Tiempo Real

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F3.1 |
| **Title**              | Chat Clínico en Tiempo Real |
| **Parent (Epic)**      | E3 – Consulta Clínica en Tiempo Real |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 9 SP |
| **Tags**               | chat; websocket; mensajeria; tiempo-real; must; sprint5 |

#### Descripción (campo Description)

```
Habilita el canal de mensajería clínica bidireccional entre el paciente (app React Native)
y el médico (panel web Next.js) a través de WebSocket.

Alcance funcional:
- Endpoint POST /v1/consultations/{id}/messages para enviar mensajes REST (fallback y
  persistencia garantizada).
- Gateway WebSocket NestJS (namespace: consultation) que emite y recibe eventos:
    · consultation.message.created  → nuevo mensaje disponible para ambas partes.
    · consultation.status.changed   → cambio de estado de consulta.
    · consultation.priority.updated → cambio de prioridad del caso.
- Autenticación de la conexión WebSocket mediante el JWT del paciente o médico
  (validado en el handshake).
- Autorización por consulta: solo el paciente dueño y el médico asignado pueden
  conectarse al room de esa consulta.
- Entrega con ACK lógico: el servidor confirma la recepción de cada mensaje;
  el cliente reintenta si no recibe ACK en 5 segundos.
- Sincronización al reconectar: al establecerse la conexión, el cliente solicita
  mensajes desde su último messageId recibido.
- Persistencia de mensajes en colección ConsultationMessage de MongoDB con
  timestamp de servidor para garantizar orden cronológico.
- Notificación interna al receptor cuando llega un mensaje y la app/web está
  en background.

Criterios de salida del feature:
- Dos usuarios en la misma consulta intercambian mensajes que aparecen en tiempo
  real en ambos clientes.
- Los mensajes persisten en MongoDB y son recuperables tras reconexión.
- La conexión WebSocket es rechazada si el JWT no corresponde a un participante
  de la consulta.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Mensajería clínica en tiempo real

  Scenario: Envío y recepción exitosa de mensaje en tiempo real
    Given una consulta activa con paciente y médico conectados al WebSocket
    When el paciente envía un mensaje de texto
    Then el médico lo recibe mediante el evento consultation.message.created
    And el servidor retorna ACK al paciente confirmando entrega

  Scenario: Mensaje persistido en base de datos
    Given un mensaje enviado correctamente durante una consulta
    When el médico o paciente solicita el historial de mensajes
    Then el sistema retorna todos los mensajes en orden cronológico por timestamp de servidor

  Scenario: Reconexión y sincronización de mensajes pendientes
    Given un paciente que pierde conexión durante una consulta activa
    When el cliente se reconecta al WebSocket con su último messageId
    Then el servidor envía todos los mensajes posteriores en orden cronológico
    And no se duplican mensajes ya recibidos

  Scenario: Conexión WebSocket rechazada por JWT inválido
    Given un usuario con un JWT expirado o inválido
    When intenta conectarse al namespace consultation
    Then el servidor rechaza la conexión con código 401
    And no se establece el room de consulta

  Scenario: Usuario no autorizado bloqueado del room de consulta
    Given un paciente autenticado que no es dueño de la consulta solicitada
    When intenta suscribirse al room de esa consulta
    Then el servidor rechaza con código 403
    And no recibe ningún evento de esa consulta
```

---

### Feature F3.2 – Cola de Casos Priorizada para Médico

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F3.2 |
| **Title**              | Cola de Casos Priorizada para Médico |
| **Parent (Epic)**      | E3 – Consulta Clínica en Tiempo Real |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 4 SP |
| **Tags**               | cola; prioridad; medico; websocket; must; sprint5 |

#### Descripción (campo Description)

```
Proporciona al médico una vista dinámica de la cola de consultas ordenadas por prioridad
clínica, con actualizaciones en tiempo real cuando se incorporan nuevos casos.

Alcance funcional:
- Endpoint GET /v1/consultations/queue: retorna consultas OPEN o IN_PROGRESS del médico
  autenticado, ordenadas por: prioridad (HIGH → MODERATE → LOW) y, dentro del mismo
  nivel, por fecha de creación (FIFO).
- Evento WebSocket consultation.queue.updated: notifica al médico cuando la cola cambia
  (nueva consulta, cambio de prioridad, cierre de consulta).
- Endpoint POST /v1/consultations: crea una consulta a partir del sessionId de triage
  finalizado; valida que el triage existe y está completado; asigna prioridad y especialidad
  desde el resultado del triage.
- Panel web (Next.js): vista /doctor/queue con tabla de consultas ordenada, indicador
  visual de prioridad (rojo=HIGH, naranja=MODERATE, verde=LOW) y botón "Atender".
- Al hacer clic en "Atender", el estado de la consulta cambia a IN_PROGRESS y el médico
  es redirigido a la pantalla de chat de esa consulta.
- Filtro por especialidad (Medicina General / Odontología) en el panel.
- Solo visible para médicos con rol MEDICO y estado rethusStatus = VERIFIED.

Criterios de salida del feature:
- La cola muestra consultas ordenadas correctamente por prioridad.
- La cola se actualiza en tiempo real cuando llega una nueva consulta HIGH.
- Un médico PENDING no puede acceder a la cola (HTTP 403).
- Al atender una consulta, el estado cambia a IN_PROGRESS y ambos participantes
  reciben el evento consultation.status.changed.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Cola de casos priorizada para médico

  Scenario: Médico VERIFIED accede a cola ordenada por prioridad
    Given un médico con estado VERIFIED autenticado en el panel web
    When accede a GET /v1/consultations/queue
    Then recibe HTTP 200 con consultas ordenadas HIGH primero, luego MODERATE, luego LOW
    And dentro del mismo nivel de prioridad, las consultas están ordenadas por fecha de creación

  Scenario: Cola actualizada en tiempo real por nueva consulta HIGH
    Given un médico conectado al WebSocket con su cola visible
    When un paciente crea una nueva consulta con prioridad HIGH
    Then el médico recibe el evento consultation.queue.updated
    And la nueva consulta aparece al inicio de la cola sin recargar la página

  Scenario: Médico atiende consulta y cambia estado a IN_PROGRESS
    Given un médico en la vista de cola con una consulta OPEN
    When hace clic en "Atender" sobre una consulta
    Then el sistema actualiza el estado a IN_PROGRESS
    And emite el evento consultation.status.changed a paciente y médico
    And el médico es redirigido a la pantalla de chat de esa consulta

  Scenario: Médico PENDING bloqueado de la cola de casos
    Given un médico con estado PENDING
    When intenta acceder a GET /v1/consultations/queue
    Then el sistema retorna HTTP 403
    And el mensaje indica que debe completar la verificación profesional

  Scenario: Creación de consulta a partir de triage completado
    Given un paciente autenticado con un triage en estado COMPLETED
    When llama a POST /v1/consultations con el sessionId del triage
    Then el sistema crea la consulta con prioridad y especialidad del triage
    And retorna HTTP 201 con el id de la consulta y su estado inicial OPEN
    And la consulta aparece en la cola del médico correspondiente
```

---

## 3. HISTORIAS DE USUARIO

### HU-006 – Chat Clínico en Tiempo Real

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-006 |
| **Title**              | Como paciente y médico quiero intercambiar mensajes en tiempo real con estado de caso para mejorar la continuidad y oportunidad de respuesta |
| **Parent (Feature)**   | F3.1 – Chat Clínico en Tiempo Real / F3.2 – Cola de Casos Priorizada |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Story Points**       | 13 |
| **Risk**               | High – Latencia WebSocket, concurrencia de sesiones y sincronización tras reconexión |
| **Dependencias**       | HU-005 (resumen clínico preconsulta debe estar disponible al abrir la consulta) |
| **Tags**               | chat; websocket; cola; consulta; tiempo-real; must; sprint5 |

#### Descripción completa (campo Description)

```
Como paciente y como médico
Quiero intercambiar mensajes clínicos en tiempo real dentro de una consulta activa
y ver el estado actualizado de la consulta en todo momento
Para mejorar la continuidad y la oportunidad de respuesta durante la atención médica.

Contexto:
El chat clínico es el núcleo de la interacción entre paciente y médico en SaludDeUna.
Sin este canal, el flujo de atención no puede completarse. La entrega en tiempo real
es un requisito de negocio y un KPI crítico: el tiempo a primera respuesta médica
debe ser reducido y medible. La cola priorizada asegura que los médicos atiendan
primero los casos más urgentes (HIGH), cumpliendo el SLO de P95 < 1500 ms para
la entrega de mensajes.

Flujo principal:
1. Paciente completa el triage (HU-003 / HU-004) → el sistema genera prioridad.
2. Paciente crea una consulta desde la app (POST /v1/consultations).
3. La consulta entra a la cola del médico ordenada por prioridad.
4. El médico abre la consulta desde su panel → estado cambia a IN_PROGRESS.
5. Paciente y médico intercambian mensajes mediante WebSocket.
6. El médico cierra la consulta → estado cambia a CLOSED.
7. Se dispara el flujo de seguimiento (HU-007) automáticamente.

Restricciones funcionales:
- Solo puede haber una consulta activa (OPEN o IN_PROGRESS) por paciente a la vez.
- La sesión WebSocket debe autenticarse con el mismo JWT del paciente o médico.
- Solo el médico puede cambiar el estado a CLOSED.
- Los mensajes deben tener timestamp de servidor para garantizar orden; no se
  usa el timestamp del cliente.
- El SLO de entrega de mensajes es P95 < 1500 ms bajo carga normal (ver
  docs/wiki/12-Riesgos-Concurrencia-RealTime.md).

Notas de UX:
- La app móvil debe mostrar indicador de "escribiendo..." cuando el médico está
  redactando una respuesta.
- El panel web debe mostrar el resumen clínico preconsulta en un panel lateral
  al abrir la consulta.
- El botón "Cerrar consulta" solo debe estar visible para el médico.
- Los mensajes deben diferenciarse visualmente: los del paciente alineados a la
  derecha (burbuja azul) y los del médico a la izquierda (burbuja gris).
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Mensajería clínica en tiempo real

  Scenario: Interacción principal paciente-médico en consulta activa
    Given una consulta activa (IN_PROGRESS) con paciente y médico conectados al WebSocket
    When el paciente envía un mensaje de texto
    Then el médico lo recibe en tiempo real mediante el evento consultation.message.created
    And el estado de la consulta se mantiene IN_PROGRESS sin recargar ninguna interfaz

  Scenario: Interacción alterna con desconexión temporal del paciente
    Given una consulta activa con el paciente con pérdida momentánea de red
    When el paciente recupera la conectividad y se reconecta al WebSocket
    Then el cliente sincroniza todos los mensajes enviados durante la desconexión
    And los mensajes aparecen en orden cronológico correcto sin duplicados

  Scenario: Médico cierra consulta
    Given una consulta en estado IN_PROGRESS
    When el médico envía la acción de cierre
    Then el estado cambia a CLOSED
    And ambos participantes reciben el evento consultation.status.changed con newStatus=CLOSED
    And la consulta desaparece de la cola activa del médico

  Scenario: Creación de consulta con prioridad correcta desde triage
    Given un paciente con triage completado con prioridad HIGH
    When llama a POST /v1/consultations con su sessionId de triage
    Then el sistema crea la consulta con priority=HIGH
    And la consulta aparece al inicio de la cola del médico de la especialidad correspondiente

  Scenario: Solo un participante autorizado accede al room WebSocket
    Given un usuario autenticado que no es el paciente ni el médico de la consulta
    When intenta conectarse al room de esa consulta en el WebSocket
    Then el servidor rechaza la conexión con código 403
    And no recibe ningún evento de esa consulta
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y features definidos (E3, F3.1, F3.2)
- [x] Prioridad MoSCoW asignada (Must)
- [x] Criterios Gherkin documentados (principal y alternos)
- [x] Dependencias identificadas (HU-005 debe estar completada antes del inicio del Sprint 5)
- [x] Estimación acordada (13 SP)
- [x] Datos/ambientes necesarios identificados (MongoDB dev, WebSocket dev, Azure dev)
- [x] Riesgos anotados (alto – latencia WS, concurrencia, sincronización)
- [x] Criterios de seguridad definidos (JWT en handshake WS, autorización por consulta, auditoría de mensajes)
- [x] SLO de rendimiento definido (P95 entrega mensaje < 1500 ms)
- [x] Responsable de validación funcional asignado

#### Definition of Done (DoD) – Checklist

- [ ] Endpoint `POST /v1/consultations` implementado y revisado por al menos un par
- [ ] Endpoint `GET /v1/consultations/queue` implementado con ordenamiento por prioridad
- [ ] Endpoint `POST /v1/consultations/{id}/messages` implementado con persistencia en MongoDB
- [ ] Gateway WebSocket NestJS implementado con autenticación y autorización por consulta
- [ ] Eventos `consultation.message.created`, `consultation.status.changed`, `consultation.priority.updated` y `consultation.queue.updated` funcionando correctamente
- [ ] Sincronización de mensajes al reconectar implementada y probada
- [ ] Pantalla de chat en React Native (app paciente) implementada con buenas prácticas de UX
- [ ] Vista de cola y pantalla de chat en Next.js (panel médico) implementada
- [ ] Pruebas unitarias del ConsultationService en verde (cobertura >= 80%)
- [ ] Pruebas de integración de endpoints REST en verde
- [ ] Prueba de concurrencia inicial (20 sesiones WebSocket simultáneas sin pérdida de mensajes)
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] Log de auditoría registra creación de consultas, mensajes y cambios de estado
- [ ] Guard `DoctorVerifiedGuard` aplicado en `GET /v1/consultations/queue`
- [ ] Validaciones de seguridad ejecutadas (auth WS, autorización por room, input sanitization)
- [ ] Documentación técnica actualizada en Wiki (endpoints, eventos WS, contratos)
- [ ] Demo funcional aprobada por el equipo
- [ ] Historia pasada a estado Done con evidencia enlazada (PR + test results)

---

## 4. TAREAS

### Tareas de HU-006 – Chat Clínico en Tiempo Real

---

#### T-006-01 – Diseño de modelos de datos Consultation y ConsultationMessage en MongoDB

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Diseñar esquemas Mongoose Consultation y ConsultationMessage |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Crear los esquemas Mongoose para la persistencia de consultas y mensajes:

Consultation schema:
- _id (ObjectId generado automáticamente)
- patientId (ref Patient, required)
- doctorId (ref Doctor, optional – se asigna cuando el médico abre la consulta)
- specialty: enum ['GENERAL_MEDICINE', 'DENTISTRY'], required
- priority: enum ['LOW', 'MODERATE', 'HIGH'], required (tomado del triage)
- triageSessionId (ref TriageSession, required)
- status: enum ['OPEN', 'IN_PROGRESS', 'CLOSED'], default 'OPEN'
- clinicalSummaryId (ref ClinicalSummary, optional – generado por E4/HU-005)
- closedAt (Date, optional)
- timestamps (createdAt, updatedAt automáticos)

ConsultationMessage schema:
- _id (ObjectId)
- consultationId (ref Consultation, required)
- senderId (String, required) – userId del emisor
- senderRole: enum ['PACIENTE', 'MEDICO'], required
- content (String, required, trim)
- serverTimestamp (Date, required, default: Date.now) – usado para orden cronológico;
  no se acepta timestamp del cliente para garantizar consistencia.
- deliveredAt (Date, optional) – momento del ACK lógico
- timestamps

Índices recomendados:
- Consultation: índice compuesto { doctorId, status, priority, createdAt } para
  eficiencia en la consulta de cola.
- ConsultationMessage: índice compuesto { consultationId, serverTimestamp } para
  paginación y sincronización eficiente.
- Consultation: índice único { patientId, status: 'OPEN' | 'IN_PROGRESS' } para
  validar que no haya más de una consulta activa por paciente.

Ubicar en: apps/api/src/consultations/schemas/
```

**Criterios de aceptación de la tarea:**
- Ambos esquemas creados y exportados correctamente.
- Los índices están definidos en el schema y verificados en MongoDB local.
- El campo serverTimestamp es el único timestamp de referencia para ordenamiento.
- No se puede crear una segunda consulta activa para el mismo paciente (índice único validado).

---

#### T-006-02 – Implementación del endpoint POST /v1/consultations

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint POST /v1/consultations para crear consulta desde triage |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el endpoint de creación de consulta clínica:
- Módulo: ConsultationsModule (NestJS)
- Ruta: POST /v1/consultations
- Protegido con @Roles('PACIENTE') y JwtAuthGuard.
- DTO de entrada: CreateConsultationDto { triageSessionId }
- Lógica:
    1. Validar que el triageSessionId existe y pertenece al paciente autenticado.
    2. Validar que el triage esté en estado COMPLETED; retornar HTTP 422 si no.
    3. Validar que el paciente no tenga ya una consulta activa (OPEN o IN_PROGRESS);
       retornar HTTP 409 con mensaje "Ya tienes una consulta activa" si la tiene.
    4. Crear el documento Consultation con patientId, specialty y priority del triage.
    5. Emitir evento WebSocket consultation.queue.updated a los médicos de la
       especialidad correspondiente.
    6. Registrar evento de auditoría en log estructurado con correlation ID.
- Respuesta HTTP 201: { id, patientId, specialty, priority, status, triageSessionId, createdAt }
Ubicar en: apps/api/src/consultations/
```

**Criterios de aceptación de la tarea:**
- Endpoint retorna 201 con consulta en estado OPEN y prioridad correcta del triage.
- Retorna 404 si el triageSessionId no existe o no pertenece al paciente.
- Retorna 422 si el triage no está completado.
- Retorna 409 si el paciente ya tiene una consulta activa.
- El evento `consultation.queue.updated` se emite correctamente al médico correspondiente.

---

#### T-006-03 – Implementación del endpoint GET /v1/consultations/queue

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint GET /v1/consultations/queue con ordenamiento por prioridad |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el endpoint de cola de casos para el médico:
- Ruta: GET /v1/consultations/queue
- Protegido con @Roles('MEDICO'), JwtAuthGuard y DoctorVerifiedGuard.
- Query params opcionales: specialty (GENERAL_MEDICINE | DENTISTRY), status (OPEN | IN_PROGRESS).
- Lógica de ordenamiento:
    1. Filtrar consultas por specialty (si se indica) y status (default: OPEN e IN_PROGRESS).
    2. Ordenar por prioridad: HIGH(1) → MODERATE(2) → LOW(3).
    3. Dentro del mismo nivel de prioridad, ordenar por createdAt ASC (FIFO).
- Incluir en cada ítem de respuesta:
    { id, patientId, patientName, specialty, priority, status, createdAt,
      clinicalSummaryAvailable: Boolean }
- Paginar con limit (default 20) y cursor-based pagination via lastId.
- Registrar acceso en log con rol y userId del médico.
Ubicar en: apps/api/src/consultations/
```

**Criterios de aceptación de la tarea:**
- Retorna 200 con consultas en el orden correcto (HIGH primero, FIFO dentro del mismo nivel).
- Médico PENDING recibe 403 (DoctorVerifiedGuard activo).
- Filtro por specialty funciona correctamente.
- Paginación funciona con parámetro lastId.
- El log registra cada acceso al endpoint.

---

#### T-006-04 – Implementación del endpoint POST /v1/consultations/{id}/messages

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint REST de envío de mensajes en consulta |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el endpoint de mensajería clínica REST (complementario al WebSocket,
garantiza persistencia independiente del estado de la conexión WS):
- Ruta: POST /v1/consultations/{id}/messages
- Protegido con JwtAuthGuard; roles permitidos: PACIENTE y MEDICO.
- DTO de entrada: SendMessageDto { content: string } — máximo 2000 caracteres.
- Lógica:
    1. Validar que la consulta existe y que el userId del JWT es el paciente o el médico.
       Retornar 403 si no es un participante de la consulta.
    2. Validar que la consulta esté en estado OPEN o IN_PROGRESS; retornar 422 si está CLOSED.
    3. Crear el documento ConsultationMessage con serverTimestamp = Date.now().
    4. Emitir evento WebSocket consultation.message.created al room de la consulta.
    5. Registrar en log estructurado con consultationId, senderId y senderRole.
- Respuesta HTTP 201:
    { id, consultationId, senderId, senderRole, content, serverTimestamp }
Ubicar en: apps/api/src/consultations/
```

**Criterios de aceptación de la tarea:**
- Retorna 201 con el mensaje creado y serverTimestamp del servidor.
- Retorna 403 si el userId no es participante de la consulta.
- Retorna 422 si la consulta está CLOSED.
- El evento `consultation.message.created` se emite al room correcto.
- El campo content es sanitizado para prevenir XSS antes de persistir.

---

#### T-006-05 – Implementación del Gateway WebSocket (ConsultationGateway)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar ConsultationGateway WebSocket en NestJS |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 5 |

**Descripción:**
```
Implementar el gateway WebSocket para el namespace consultation:
- Clase: ConsultationGateway decorada con @WebSocketGateway({ namespace: 'consultation' }).
- Autenticación en el handshake:
    · Extraer el JWT del header Authorization o del query param token.
    · Validar firma con JwtService; rechazar conexión (disconnect) si el JWT es
      inválido o expirado, respondiendo con WsException('Unauthorized').
- Gestión de rooms por consulta:
    · Al conectarse, el cliente envía el evento join-consultation con { consultationId }.
    · El gateway valida que el userId del JWT sea participante de esa consulta (paciente
      dueño o médico asignado); si no, emite WsException('Forbidden') y cierra el room.
    · El gateway agrega al cliente al room socket.io identificado por consultationId.
- Eventos emitidos desde el servidor:
    · consultation.message.created: { messageId, consultationId, senderId, senderRole,
      content, serverTimestamp }
    · consultation.status.changed: { consultationId, oldStatus, newStatus, timestamp }
    · consultation.priority.updated: { consultationId, oldPriority, newPriority, reason }
    · consultation.queue.updated: { doctorId, specialty, queueSize, topCases }
- Sincronización al reconectar:
    · El cliente envía evento sync-messages con { consultationId, lastMessageId }.
    · El gateway responde con todos los mensajes posteriores a lastMessageId en orden
      cronológico por serverTimestamp.
- ACK lógico:
    · Al procesar un mensaje entrante del cliente, el gateway emite message-ack con
      { messageId, receivedAt } al emisor.
- Aplicar límite de reconexiones: máximo 5 intentos con backoff exponencial en cliente.
Ubicar en: apps/api/src/consultations/consultation.gateway.ts
```

**Criterios de aceptación de la tarea:**
- El gateway autentica correctamente la conexión mediante JWT en el handshake.
- Un usuario no participante de la consulta no puede unirse al room.
- Los cuatro eventos definidos se emiten correctamente a los clientes en el room.
- La sincronización al reconectar entrega mensajes en orden y sin duplicados.
- El ACK lógico es emitido tras cada mensaje procesado.
- La desconexión abrupta del cliente es manejada sin causar errores en el servidor.

---

#### T-006-06 – Implementación del cambio de estado de consulta (IN_PROGRESS / CLOSED)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoints de cambio de estado de consulta (atender y cerrar) |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Implementar las transiciones de estado de la consulta:

PATCH /v1/consultations/{id}/attend (OPEN → IN_PROGRESS):
- Solo rol MEDICO + DoctorVerifiedGuard.
- Asigna doctorId al documento Consultation con el userId del médico autenticado.
- Emite evento WebSocket consultation.status.changed a los participantes del room.
- Retorna HTTP 200: { id, status: 'IN_PROGRESS', doctorId, updatedAt }

PATCH /v1/consultations/{id}/close (IN_PROGRESS → CLOSED):
- Solo rol MEDICO; validar que el médico autenticado es el asignado a la consulta.
- Actualiza status a CLOSED y closedAt = Date.now().
- Emite evento WebSocket consultation.status.changed con newStatus=CLOSED.
- Dispara evento interno follow-up.triggered para que el FollowUpService (HU-007)
  programe el seguimiento post-consulta.
- Retorna HTTP 200: { id, status: 'CLOSED', closedAt }
- Registrar en log estructurado con correlation ID.
Ubicar en: apps/api/src/consultations/
```

**Criterios de aceptación de la tarea:**
- `attend`: retorna 200 con status IN_PROGRESS; médico incorrecto recibe 403.
- `close`: retorna 200 con status CLOSED y closedAt; solo el médico asignado puede cerrar.
- Ambos emiten el evento `consultation.status.changed` correctamente al room.
- El evento `follow-up.triggered` se emite tras el cierre para HU-007.
- Intentar cerrar una consulta ya CLOSED retorna 422 con mensaje de estado inválido.

---

#### T-006-07 – Pantalla de chat clínico en React Native (app paciente)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar pantalla de chat clínico en la app React Native del paciente |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 5 |

**Descripción:**
```
Implementar la pantalla de consulta activa en la app móvil del paciente:

ConsultationScreen (React Native):
- Mostrar historial de mensajes en orden cronológico (serverTimestamp).
- Burbujas de mensaje diferenciadas: paciente a la derecha (azul), médico a la izquierda (gris).
- Indicador de estado de la consulta (OPEN / IN_PROGRESS / CLOSED) visible en el header.
- Input de texto con botón de envío; deshabilitar si la consulta está CLOSED.
- Indicador de "el médico está escribiendo..." (evento typing recibido por WS).
- Conectar al WebSocket al montar la pantalla y desconectar al desmontar.
- Emitir join-consultation al conectarse con el consultationId.
- Al enviar, llamar al endpoint REST POST /v1/consultations/{id}/messages para
  garantizar persistencia; el WS es complementario para tiempo real.
- Al recibir el evento consultation.status.changed con CLOSED: deshabilitar el input
  y mostrar mensaje "Consulta cerrada por el médico".
- Sincronización al reconectar: al recuperar conexión WS, emitir sync-messages con
  el lastMessageId recibido.
- Gestión de errores: mostrar banner de error si el envío falla; permitir reintentar.

Almacenamiento local:
- Guardar mensajes en el estado de React (no persisten en dispositivo; se recargan
  desde el historial REST al montar la pantalla).

Ubicar en: apps/mobile/src/screens/consultation/ConsultationScreen.tsx
```

**Criterios de aceptación de la tarea:**
- Los mensajes enviados por el paciente aparecen en la pantalla inmediatamente.
- Los mensajes del médico llegan en tiempo real sin recargar la pantalla.
- Tras reconexión, los mensajes pendientes se sincronizan y aparecen en orden correcto.
- El input se deshabilita cuando la consulta está CLOSED.
- La app no crashea en ningún flujo de error del backend o del WebSocket.

---

#### T-006-08 – Vista de cola y pantalla de chat en Next.js (panel médico)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar vista de cola de casos y pantalla de chat en el panel médico (Next.js) |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Web |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 5 |

**Descripción:**
```
Implementar en el panel web (Next.js + React) las dos pantallas principales del médico:

1. QueuePage (/doctor/queue):
- Tabla con consultas ordenadas por prioridad (HIGH=rojo, MODERATE=naranja, LOW=verde).
- Columnas: prioridad, nombre del paciente, especialidad, tiempo en cola, estado.
- Filtro por especialidad (Medicina General / Odontología).
- Indicador de actualización en tiempo real mediante evento consultation.queue.updated.
- Botón "Atender" por fila que llama a PATCH /v1/consultations/{id}/attend.
- Al hacer clic, redirigir al médico a /doctor/consultation/{id}.
- Solo visible para médicos VERIFIED; redirigir a /403 si el rol o estado no corresponde.

2. ConsultationPage (/doctor/consultation/{id}):
- Panel lateral izquierdo: resumen clínico preconsulta (ClinicalSummary de HU-005),
  datos del paciente y prioridad del caso.
- Panel derecho: historial de mensajes en orden cronológico.
- Input de texto con botón de envío y botón "Cerrar consulta".
- Burbujas de mensaje: médico a la derecha (azul), paciente a la izquierda (gris).
- Indicador de estado (IN_PROGRESS / CLOSED) en el header.
- Conexión WebSocket al montar la página; emitir join-consultation con consultationId.
- Botón "Cerrar consulta": llama a PATCH /v1/consultations/{id}/close; al confirmar,
  deshabilitar input y mostrar "Consulta cerrada".
- Actualización en tiempo real de mensajes entrantes via evento consultation.message.created.

Ubicar en: apps/web/src/app/doctor/
```

**Criterios de aceptación de la tarea:**
- La cola muestra consultas ordenadas correctamente y se actualiza en tiempo real.
- El indicador visual de prioridad (color) es claro y correcto.
- Los mensajes del paciente llegan en tiempo real en la ConsultationPage.
- El panel lateral muestra el resumen clínico preconsulta cuando está disponible.
- Acceso de un rol no-MEDICO o médico no-VERIFIED redirige a /403.

---

#### T-006-09 – Pruebas unitarias del ConsultationService y ConsultationGateway

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias para ConsultationService y ConsultationGateway |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Escribir pruebas unitarias con Jest para ConsultationService y ConsultationGateway:

ConsultationService:
- createConsultation: creación exitosa con prioridad correcta.
- createConsultation: triage inexistente → NotFoundException.
- createConsultation: triage no completado → UnprocessableEntityException.
- createConsultation: paciente con consulta activa → ConflictException.
- getQueue: retorna lista ordenada correctamente (HIGH primero, FIFO).
- sendMessage: mensaje creado con serverTimestamp del servidor.
- sendMessage: usuario no participante → ForbiddenException.
- sendMessage: consulta CLOSED → UnprocessableEntityException.
- attendConsultation: actualiza estado a IN_PROGRESS correctamente.
- closeConsultation: actualiza estado a CLOSED y emite follow-up.triggered.

ConsultationGateway:
- handleConnection: JWT válido → conexión permitida.
- handleConnection: JWT inválido → WsException Unauthorized.
- handleJoinConsultation: usuario participante → unido al room.
- handleJoinConsultation: usuario no participante → WsException Forbidden.
- handleSyncMessages: retorna mensajes posteriores al lastMessageId en orden.

Mocks: ConsultationRepository, MessageRepository, JwtService, EventEmitter2.
Cobertura mínima objetivo: 80% de ConsultationService y ConsultationGateway.
Ubicar en: apps/api/src/consultations/
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI (`npm run test`).
- Cobertura >= 80% de los dos módulos.
- No hay dependencias reales de base de datos ni de WebSocket en estas pruebas.

---

#### T-006-10 – Pruebas de integración de endpoints REST de consulta

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas de integración para endpoints REST de consulta |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Escribir pruebas de integración con supertest + MongoMemoryServer:

POST /v1/consultations:
- Triage completado → HTTP 201 con prioridad correcta.
- Triage no completado → HTTP 422.
- Paciente con consulta activa → HTTP 409.
- TriageSessionId inexistente → HTTP 404.

GET /v1/consultations/queue:
- Médico VERIFIED → HTTP 200 con consultas ordenadas.
- Médico PENDING → HTTP 403.
- Filtro por specialty funciona correctamente.

POST /v1/consultations/{id}/messages:
- Participante envía mensaje → HTTP 201 con serverTimestamp.
- No participante → HTTP 403.
- Consulta CLOSED → HTTP 422.

PATCH /v1/consultations/{id}/attend:
- Médico VERIFIED → HTTP 200 con status IN_PROGRESS.
- No médico → HTTP 403.

PATCH /v1/consultations/{id}/close:
- Médico asignado → HTTP 200 con status CLOSED.
- Médico diferente → HTTP 403.
- Consulta ya CLOSED → HTTP 422.

Ubicar en: apps/api/test/consultations.e2e-spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests de integración pasan en CI de forma aislada.
- Se cubre el flujo completo desde la creación de consulta hasta el cierre.
- Los tests de estado HTTP y estructura de respuesta son precisos.

---

#### T-006-11 – Prueba de concurrencia inicial (smoke test WebSocket)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Ejecutar prueba de carga con 100 sesiones WebSocket simultáneas (Sprint 5) |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar y ejecutar el smoke test de concurrencia definido en el plan de pruebas
(docs/wiki/12-Riesgos-Concurrencia-RealTime.md – Sprint 5):

Objetivo: 100 sesiones WebSocket concurrentes + triage simultáneo, P95 dentro del SLO.

Configuración del test:
- Herramienta: artillery (si aún no hay herramienta de carga configurada en el repo,
  adoptarla como estándar del proyecto; añadir dependencia de desarrollo y script en package.json).
- Escenario: 100 usuarios virtuales conectados simultáneamente al namespace consultation.
- Cada usuario envía 10 mensajes de 200 caracteres en intervalos aleatorios de 1-3s.
- Medir:
    · Latencia P50, P95 y P99 de entrega de mensajes (desde envío hasta ACK).
    · Tasa de mensajes perdidos (sin ACK en 5s).
    · Errores de conexión WebSocket.
- Criterio de salida del Sprint 5 (plan de riesgos):
    · P95 latencia de entrega < 1500 ms.
    · Tasa de pérdida de mensajes < 0.1%.
    · Sin errores de conexión inesperados.

Instrucciones de ejecución:
- El test debe poder ejecutarse con un solo comando desde la raíz del repositorio.
- Los resultados deben exportarse a un archivo JSON en apps/api/test/load/results/.
Ubicar en: apps/api/test/load/consultation-concurrency.yml (o .js según herramienta)
```

**Criterios de aceptación de la tarea:**
- El script de prueba de concurrencia existe y puede ejecutarse en el entorno de desarrollo.
- Los resultados muestran P95 < 1500 ms en condiciones normales.
- La tasa de pérdida de mensajes es < 0.1%.
- El resultado del test queda registrado y referenciado en la Wiki.

---

#### T-006-12 – Documentación técnica de la consulta clínica en Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar endpoints REST, Gateway WebSocket y flujo de consulta en la Wiki |
| **Parent (User Story)**| HU-006 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 5 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Actualizar o crear sección en la Wiki de Azure DevOps con:

Endpoints REST:
- POST /v1/consultations: request body, response 201, errores 404/409/422.
- GET /v1/consultations/queue: query params, response 200, orden y paginación.
- POST /v1/consultations/{id}/messages: request body, response 201, errores 403/422.
- PATCH /v1/consultations/{id}/attend: response 200, errores.
- PATCH /v1/consultations/{id}/close: response 200, errores, evento follow-up.

Gateway WebSocket:
- Namespace y URL de conexión.
- Autenticación en el handshake (JWT).
- Evento join-consultation: payload y respuesta.
- Evento sync-messages: payload y respuesta.
- Eventos emitidos por el servidor: consultation.message.created,
  consultation.status.changed, consultation.priority.updated, consultation.queue.updated.
- Diagrama de secuencia simplificado del flujo de mensajes.

Flujo completo de consulta:
- Diagrama textual: triage completado → POST /v1/consultations → cola médica
  → médico atiende → mensajes en tiempo real → médico cierra → seguimiento programado.

Resultados de prueba de concurrencia:
- Resumen del smoke test del Sprint 5 con métricas P95 y tasa de pérdida.

Referencias a HU-006, HU-005 y HU-007, y al sprint de implementación.
```

**Criterios de aceptación de la tarea:**
- La Wiki refleja los contratos reales de todos los endpoints y eventos WebSocket.
- El diagrama de secuencia es comprensible para un desarrollador nuevo.
- Los resultados del smoke test están referenciados.
- Un desarrollador nuevo puede consumir el chat sin leer el código fuente.

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
📌 EPIC: E3 – Consulta Clínica en Tiempo Real (13 SP total)
│
├── 🔷 FEATURE: F3.1 – Chat Clínico en Tiempo Real (9 SP)
│   │
│   └── 📖 USER STORY: HU-006 – Como paciente y médico quiero intercambiar mensajes en tiempo real (13 SP)
│       ├── ✅ T-006-01 – Diseño de modelos Consultation y ConsultationMessage en MongoDB (2h)
│       ├── ✅ T-006-02 – Endpoint POST /v1/consultations (3h)
│       ├── ✅ T-006-03 – Endpoint GET /v1/consultations/queue (3h)
│       ├── ✅ T-006-04 – Endpoint POST /v1/consultations/{id}/messages (3h)
│       ├── ✅ T-006-05 – ConsultationGateway WebSocket NestJS (5h)
│       ├── ✅ T-006-06 – Endpoints de cambio de estado (attend y close) (2h)
│       ├── ✅ T-006-07 – Pantalla de chat en React Native (paciente) (5h)
│       ├── ✅ T-006-08 – Vista de cola y chat en Next.js (médico) (5h)
│       ├── ✅ T-006-09 – Pruebas unitarias ConsultationService y Gateway (4h)
│       ├── ✅ T-006-10 – Pruebas de integración endpoints REST consulta (3h)
│       ├── ✅ T-006-11 – Prueba de carga 100 sesiones WS concurrentes (Sprint 5) (3h)
│       └── ✅ T-006-12 – Documentación técnica en Wiki (2h)
│
└── 🔷 FEATURE: F3.2 – Cola de Casos Priorizada para Médico (4 SP)
    │
    └── (HU-006 cubre también F3.2 – los endpoints y tareas de cola están incluidos arriba)
```

**Total horas estimadas Sprint 5 (Épica 3):** 40 horas de trabajo  
**Total Story Points Épica 3:** 13 SP (HU-006: 13 SP)  
**Sprint objetivo:** Sprint 5

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|-----------|-----------|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` – Sprint 5 y MoSCoW |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` – Actividad 3: Consulta |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` – HU-006 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` – HU-006 |
| DoR / DoD | `docs/wiki/08-DoR-DoD.md` |
| KPIs y SLOs | `docs/wiki/09-Observabilidad-KPIs.md` |
| Riesgos y Concurrencia | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` – R-002, Plan Sprint 5 |
| Épica dependiente (precondición) | `docs/epics/Azure-Boards-Epica1.md` – E1 (auth y RBAC) |

### API endpoints de esta épica (contratos del Plan Maestro)
- `POST /v1/consultations` → T-006-02
- `GET /v1/consultations/queue` → T-006-03
- `POST /v1/consultations/{id}/messages` → T-006-04
- `PATCH /v1/consultations/{id}/attend` → T-006-06
- `PATCH /v1/consultations/{id}/close` → T-006-06

### Eventos WebSocket de esta épica
- `consultation.message.created` → T-006-05
- `consultation.status.changed` → T-006-05 / T-006-06
- `consultation.priority.updated` → T-006-05
- `consultation.queue.updated` → T-006-02 / T-006-05

### KPIs impactados por esta épica
- **KPI Tiempo a primera respuesta médica**: el chat en tiempo real y la cola priorizada
  son los mecanismos directos para reducir este indicador.
- **SLO P95 Chat Delivery < 1500 ms**: medido en el smoke test de concurrencia (T-006-11).
- **SLO Disponibilidad >= 98%**: el módulo de consulta es crítico para el flujo completo
  del MVP.

### Riesgos asociados
- **R-002 (Latencia alta en chat real-time)**: mitigado con prueba de concurrencia en
  Sprint 5 (T-006-11), optimización de payloads WebSocket y tuning progresivo.
- **R-003 (Inestabilidad del resumen IA)**: E3 consume el resumen de E4 (HU-005);
  si el resumen no está disponible, la consulta muestra un aviso al médico sin bloquearlo.

### Dependencias hacia otras épicas
- **E1 (HU-001, HU-002)**: la autenticación JWT y el guard `DoctorVerifiedGuard` deben
  estar operativos antes de iniciar el Sprint 5.
- **E2 (HU-003, HU-004)**: el triage debe estar completado para crear una consulta;
  la prioridad y especialidad del triage alimentan la cola clínica.
- **E4 (HU-005)**: el resumen clínico preconsulta generado por E4 es consumido en la
  pantalla del médico al abrir la consulta.
- **E5 (HU-007)**: el evento `follow-up.triggered` emitido al cerrar la consulta
  (T-006-06) activa el módulo de seguimiento de E5.
