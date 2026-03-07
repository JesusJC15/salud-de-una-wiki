# Azure Boards – Épica 4: Resumen Clínico y Traductor IA

## Propósito del documento
Este documento contiene toda la información necesaria para implementar la **Épica 4 – Resumen Clínico y Traductor IA** en Azure Boards, incluyendo la épica, sus features, historias de usuario con criterios Gherkin, y todas las tareas de desarrollo, pruebas y documentación asociadas. Cada sección indica los campos exactos que se deben completar al crear el ítem en Azure Boards.

---

## 1. ÉPICA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Epic |
| **ID**                 | E4 |
| **Title**              | Resumen Clínico y Traductor IA |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Priority**           | 1 – Critical |
| **Business Value**     | Aumentar eficiencia médica reduciendo tiempo de recolección de antecedentes y mejorar la claridad de comunicación entre paciente y médico |
| **Risk**               | High – Inestabilidad de respuestas del LLM (R-003); cambio de condiciones del proveedor IA (R-004) |
| **Effort (Story Points total estimado)** | 13 SP |
| **Tags**               | ia; resumen-clinico; traduccion; gemini; rag; guardrails; must; sprint4 |
| **Start Date**         | Sprint 4 |
| **Target Date**        | Cierre Sprint 4 |

### Descripción (campo Description)

```
La Épica 4 cubre dos capacidades de inteligencia artificial aplicadas directamente
al flujo clínico de SaludDeUna: la generación automática del resumen clínico
preconsulta y la traducción bidireccional entre lenguaje del paciente y lenguaje
clínico estructurado.

Objetivos de negocio:
- Entregar al médico un resumen clínico estructurado y útil antes de responder al
  paciente, reduciendo el tiempo de recolección de antecedentes y mejorando la
  calidad de la primera respuesta.
- Permitir que el paciente comprenda las orientaciones médicas recibidas mediante
  traducciones a lenguaje simple, sin que la IA emita diagnósticos ni prescripciones.
- Mantener guardrails estrictos que impidan que el sistema genere contenido de
  diagnóstico o prescripción automática en ninguna de las dos funcionalidades.

Funcionalidades cubiertas:
- Servicio de orquestación IA que combina Gemini + RAG (índice vectorial de
  conocimiento clínico) para generar el objeto ClinicalSummary a partir de las
  respuestas del triage.
- Endpoint POST /v1/consultations/{id}/summary/generate que persiste y retorna
  el resumen estructurado al panel médico.
- Vista de resumen clínico en el panel médico (Next.js) con indicadores de
  confianza por campo y campo de marcado de utilidad para el KPI.
- Módulo de traducción bidireccional:
    · Dirección clínico → paciente: simplifica terminología médica a lenguaje
      comprensible para el usuario general.
    · Dirección paciente → clínico: estructura la descripción del paciente en
      términos clínicos normalizados para facilitar la documentación del médico.
- Guardrails activos en ambas direcciones: el sistema rechaza explícitamente
  cualquier solicitud de diagnóstico o prescripción y aplica disclaimer visible.
- Versión de prompts registrada en cada llamada para auditoría y pruebas de
  regresión semántica.

Restricciones:
- La IA no emite diagnósticos ni prescripciones en ningún escenario; las respuestas
  siempre son orientativas y están marcadas con disclaimer.
- El resumen clínico requiere un triage finalizado (E2) como precondición; no se
  genera a partir de datos incompletos.
- La traducción es un apoyo comunicacional; no altera ni reemplaza el registro
  clínico oficial del médico.
- No se integra con sistemas HCE externos en este sprint.
- La capa IAProviderAdapter encapsula la llamada a Gemini para mitigar lock-in
  ante cambios de proveedor (R-004).
```

### Criterios de aceptación de la épica

```
- Dado un triage finalizado, el médico puede generar el resumen clínico y verlo
  estructurado antes de responder al paciente.
- El resumen contiene: motivo de consulta, duración, intensidad, síntomas asociados,
  medicación actual, antecedentes relevantes, prioridad estimada y red flags.
- El médico puede marcar el resumen como "útil" o "no útil", alimentando el KPI
  de utilidad del resumen clínico.
- Cuando el sistema detecta baja confianza semántica en las respuestas del triage,
  marca campos individuales como "pendientes de confirmación" y advierte al médico.
- La IA rechaza explícitamente y con disclaimer cualquier solicitud de diagnóstico
  o prescripción en el resumen o en la traducción.
- Un paciente puede solicitar la traducción a lenguaje simple de una orientación
  médica y recibir la respuesta sin terminología especializada.
- Un médico puede solicitar la traducción al lenguaje clínico de la descripción
  libre del paciente.
- La versión del prompt utilizado queda registrada en el log de auditoría de cada
  llamada al servicio IA.
- El tiempo de generación del resumen cumple el SLO: P95 < 15 000 ms.
```

### Acceptance Criteria – formato Gherkin (nivel épica)

```gherkin
Feature: Resumen clínico y traducción IA en SaludDeUna

  Scenario: Médico genera resumen clínico preconsulta
    Given una consulta creada a partir de un triage finalizado con prioridad asignada
    When el médico solicita el resumen clínico desde el panel web
    Then el sistema genera el objeto ClinicalSummary con todos los campos obligatorios
    And lo persiste en MongoDB y lo retorna en la respuesta HTTP 200
    And el tiempo de generación es menor a 15 000 ms (SLO IA)

  Scenario: Resumen con baja confianza semántica
    Given un triage con respuestas ambiguas o incompletas
    When el sistema detecta que la confianza semántica es baja en uno o más campos
    Then genera el resumen con esos campos marcados como pendientes de confirmación
    And muestra advertencia visible al médico antes de que responda al paciente

  Scenario: Guardrail activo bloquea solicitud de diagnóstico
    Given una llamada al servicio IA que contiene una solicitud implícita de diagnóstico
    When el guardrail evalúa el prompt antes de enviarlo al LLM
    Then el sistema rechaza la solicitud con mensaje de disclaimer
    And registra el intento bloqueado en el log de auditoría con correlation ID

  Scenario: Paciente recibe orientación en lenguaje simple
    Given una consulta activa con una orientación médica redactada por el médico
    When el paciente solicita la traducción a lenguaje simple
    Then el sistema retorna la explicación sin terminología especializada
    And adjunta el disclaimer de que la orientación no constituye diagnóstico

  Scenario: Médico recibe descripción del paciente en lenguaje clínico
    Given la descripción libre de síntomas ingresada por el paciente
    When el médico solicita la traducción al lenguaje clínico estructurado
    Then el sistema retorna los términos normalizados correspondientes
    And no modifica el registro oficial de la consulta sin confirmación del médico
```

---

## 2. FEATURES

### Feature F4.1 – Resumen Clínico Automático Preconsulta

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F4.1 |
| **Title**              | Resumen Clínico Automático Preconsulta |
| **Parent (Epic)**      | E4 – Resumen Clínico y Traductor IA |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 8 SP |
| **Tags**               | ia; resumen-clinico; gemini; rag; guardrails; preconsulta; must; sprint4 |

#### Descripción (campo Description)

```
Genera automáticamente un resumen clínico estructurado a partir de las respuestas
del triage y lo presenta al médico antes de que inicie la interacción con el paciente.

Alcance funcional:
- Servicio IAOrchestrator (NestJS) que recibe el sessionId del triage finalizado
  y construye el prompt enriquecido con el contexto del índice vectorial RAG.
- Llamada a Gemini a través de IAProviderAdapter con política de retry (máx. 2
  reintentos con backoff exponencial de 2 s) y timeout de 20 s.
- Generación del objeto ClinicalSummary:
    { chiefComplaint, duration, intensity, associatedSymptoms, currentMedication,
      relevantHistory, estimatedPriority, redFlags, confidenceScore, promptVersion }
- Guardrails previos al envío del prompt: validación con lista de patrones prohibidos
  (diagnóstico, prescripción, pronóstico); si detecta solicitud prohibida, lanza
  ForbiddenIARequestException con disclaimer y registra en log.
- Validación de confianza semántica: si confidenceScore < 0.6 en algún campo,
  ese campo se marca con flag pending_confirmation y se incluye advertencia en
  el resumen.
- Endpoint POST /v1/consultations/{id}/summary/generate:
    · Protegido con JwtAuthGuard + @Roles('MEDICO').
    · Precondición: la consulta debe tener estado OPEN o IN_PROGRESS y el triage
      asociado debe estar en estado COMPLETED.
    · Persiste el ClinicalSummary en MongoDB (colección ClinicalSummaries).
    · Retorna HTTP 200 con el objeto generado.
    · Retorna HTTP 409 si ya existe un resumen para esa consulta (idempotente).
- Vista de resumen en el panel médico (Next.js):
    · Sección colapsable con todos los campos del ClinicalSummary.
    · Indicador visual de confianza por campo (verde / amarillo / rojo).
    · Advertencia destacada cuando hay campos con pending_confirmation.
    · Botón "Marcar como útil" / "Marcar como no útil" que llama a
      PATCH /v1/consultations/{id}/summary/feedback con { useful: boolean }.
- Registro del evento de generación en log estructurado con: correlation ID,
  consultationId, promptVersion, confidenceScore, latency_ms.
- Versión del prompt registrada en el documento ClinicalSummary para trazabilidad
  y pruebas de regresión semántica.

Criterios de salida del feature:
- El médico puede generar el resumen clínico con un clic desde el panel web.
- El resumen contiene todos los campos obligatorios del contrato ClinicalSummary.
- El tiempo de generación P95 es inferior a 15 000 ms en entorno de pruebas.
- El guardrail bloquea cualquier prompt que solicite diagnóstico o prescripción.
- El KPI de utilidad del resumen es alimentado por el botón de feedback.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Generación de resumen clínico preconsulta

  Scenario: Generación exitosa de resumen clínico
    Given una consulta con triage completado y prioridad asignada
    When el médico llama a POST /v1/consultations/{id}/summary/generate
    Then el sistema retorna HTTP 200 con el objeto ClinicalSummary completo
    And el resumen incluye chiefComplaint, duration, intensity, estimatedPriority y redFlags
    And el resumen queda persistido en la colección ClinicalSummaries de MongoDB

  Scenario: Resumen con campos de baja confianza semántica
    Given un triage con respuestas ambiguas en el campo de duración y medicación
    When el IAOrchestrator calcula un confidenceScore inferior a 0.6 para esos campos
    Then el resumen retorna esos campos con flag pending_confirmation = true
    And la vista médica muestra advertencia destacada sobre los campos incompletos

  Scenario: Guardrail bloquea solicitud de diagnóstico
    Given una consulta cuyo contexto de triage contiene patrones de solicitud de diagnóstico
    When el guardrail evalúa el prompt antes del envío a Gemini
    Then el sistema lanza ForbiddenIARequestException con HTTP 422
    And retorna el mensaje de disclaimer al médico
    And registra el intento bloqueado en el log con correlation ID

  Scenario: Idempotencia del endpoint de generación
    Given una consulta que ya tiene un ClinicalSummary generado
    When el médico llama nuevamente a POST /v1/consultations/{id}/summary/generate
    Then el sistema retorna HTTP 409 con el resumen existente
    And no genera una segunda llamada al LLM

  Scenario: Feedback de utilidad del resumen
    Given un médico que ha revisado el resumen clínico de una consulta
    When llama a PATCH /v1/consultations/{id}/summary/feedback con useful = true
    Then el sistema persiste el feedback en el documento ClinicalSummary
    And el contador del KPI de utilidad se actualiza correctamente

  Scenario: Acceso denegado para rol PACIENTE
    Given un paciente autenticado con rol PACIENTE
    When intenta llamar a POST /v1/consultations/{id}/summary/generate
    Then el sistema retorna HTTP 403
    And el log registra el intento de acceso no autorizado
```

---

### Feature F4.2 – Traducción Paciente-Clínico Bidireccional

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F4.2 |
| **Title**              | Traducción Paciente-Clínico Bidireccional |
| **Parent (Epic)**      | E4 – Resumen Clínico y Traductor IA |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Priority**           | 2 – High |
| **MoSCoW**             | Should |
| **Effort (SP)**        | 5 SP |
| **Tags**               | ia; traduccion; lenguaje-simple; lenguaje-clinico; guardrails; should; sprint4 |

#### Descripción (campo Description)

```
Habilita la traducción bidireccional entre el lenguaje coloquial del paciente y
la terminología clínica estructurada, mejorando la comprensión mutua durante la
consulta sin emitir diagnósticos ni prescripciones.

Alcance funcional:
- Módulo TranslationService (NestJS) con dos operaciones:
    · translateToPatient(text: string, context?: ClinicalContext): string
      Simplifica terminología médica a lenguaje comprensible para el usuario general.
      Ejemplo: "bradicardia sinusal" → "ritmo cardíaco más lento de lo normal".
    · translateToClinic(text: string): string
      Normaliza la descripción libre del paciente a términos clínicos estandarizados.
      Ejemplo: "me duele mucho la cabeza desde ayer y me mareo" →
      "cefalea intensa de 24 horas de evolución con vértigo asociado".
- Cada operación pasa por el guardrail antes de invocar Gemini:
    · Bloquea interpretación diagnóstica, pronóstica o prescriptiva.
    · Adjunta disclaimer automático al resultado: "Esta traducción es orientativa
      y no constituye diagnóstico médico ni indicación terapéutica."
- Endpoint POST /v1/consultations/{id}/translation:
    · Body: { direction: 'TO_PATIENT' | 'TO_CLINIC', text: string }
    · Protegido con JwtAuthGuard; accesible para roles PACIENTE y MEDICO.
    · Retorna HTTP 200 con { translatedText, disclaimer, promptVersion }.
    · No persiste la traducción como parte del registro clínico oficial; es
      solo de apoyo a la comunicación.
- Integración en la interfaz:
    · App React Native (paciente): botón "Explicar en lenguaje simple" junto a
      los mensajes del médico en el chat clínico.
    · Panel web médico (Next.js): botón "Traducir al lenguaje clínico" junto
      al campo de descripción libre del paciente en el detalle de la consulta.
- Registro de cada traducción en log estructurado con: correlation ID,
  consultationId, direction, promptVersion, latency_ms (sin registrar el
  texto original ni traducido para minimizar exposición de datos sensibles).

Criterios de salida del feature:
- Un paciente puede recibir la explicación de una orientación médica en lenguaje
  simple directamente desde la app, con disclaimer visible.
- Un médico puede obtener la normalización clínica de la descripción del paciente
  desde el panel web.
- El guardrail bloquea en ambas direcciones cualquier solicitud que implique
  diagnóstico o prescripción.
- La traducción no modifica el registro clínico oficial sin confirmación explícita
  del médico.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Traducción paciente-clínico bidireccional

  Scenario: Paciente recibe orientación en lenguaje simple
    Given una consulta activa donde el médico ha escrito una orientación con terminología especializada
    When el paciente solicita POST /v1/consultations/{id}/translation con direction = TO_PATIENT
    Then el sistema retorna HTTP 200 con translatedText en lenguaje coloquial
    And adjunta el disclaimer de que la orientación no constituye diagnóstico
    And el texto traducido no contiene términos de diagnóstico directo ni prescripción

  Scenario: Médico normaliza descripción del paciente a lenguaje clínico
    Given la descripción libre del paciente ingresada en el detalle de la consulta
    When el médico solicita POST /v1/consultations/{id}/translation con direction = TO_CLINIC
    Then el sistema retorna HTTP 200 con los términos clínicos normalizados
    And adjunta el disclaimer correspondiente
    And el registro oficial de la consulta no se modifica automáticamente

  Scenario: Guardrail bloquea traducción con contenido diagnóstico
    Given un texto de entrada que contiene una solicitud implícita de diagnóstico
    When el TranslationService evalúa el prompt con el guardrail
    Then el sistema rechaza la operación con HTTP 422
    And retorna mensaje de disclaimer al solicitante
    And registra el intento bloqueado en el log sin almacenar el texto original

  Scenario: Acceso denegado para usuario sin autenticación
    Given una solicitud sin JWT al endpoint de traducción
    When se envía POST /v1/consultations/{id}/translation
    Then el sistema retorna HTTP 401
    And no realiza ninguna llamada al servicio IA
```

---

## 3. HISTORIAS DE USUARIO

### Historia de Usuario HU-005 – Resumen Clínico Automático Preconsulta

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-005 |
| **Title**              | Como médico quiero recibir el resumen clínico automático preconsulta |
| **Parent (Feature)**   | F4.1 – Resumen Clínico Automático Preconsulta |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA / SaludDeUna\\Web |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 8 SP |
| **Risk**               | High – Inestabilidad del LLM (R-003); latencia de generación (SLO IA) |
| **Tags**               | ia; resumen-clinico; gemini; rag; medico; must; sprint4 |
| **Depends On**         | HU-003 (triage Medicina General), HU-004 (triage Odontología) |

#### Descripción (campo Description)

```
Como médico
quiero recibir automáticamente un resumen clínico estructurado del paciente
    antes de iniciar la consulta
para reducir el tiempo de recolección manual de antecedentes y mejorar la
    calidad de mi primera respuesta.

Contexto:
Al abrir una consulta desde su cola de casos, el médico necesita conocer
rápidamente el motivo de consulta, la evolución de los síntomas, los medicamentos
actuales, los antecedentes relevantes, la prioridad asignada y los red flags
detectados por el motor de triage. Hoy este proceso requiere que el médico lea
todas las respuestas crudas del triage, lo que consume tiempo valioso. El resumen
clínico generado por IA (Gemini + RAG) centraliza esta información en un objeto
estructurado y auditable.

Criterios de salida:
- El médico puede generar el resumen con un clic desde el panel web.
- El resumen contiene todos los campos del contrato ClinicalSummary.
- La generación P95 es menor a 15 000 ms.
- El guardrail bloquea contenido prohibido antes de cada llamada al LLM.
- El botón de feedback (útil / no útil) alimenta el KPI de utilidad del resumen.
- La versión del prompt y el confidenceScore quedan registrados en MongoDB
  y en el log estructurado para trazabilidad y regresión semántica.
```

#### Criterios de aceptación (campo Acceptance Criteria)

```gherkin
Feature: Generación de resumen clínico

  Scenario: Resumen principal preconsulta
    Given un caso con triage finalizado y prioridad asignada
    When el médico abre la consulta y solicita el resumen clínico
    Then el sistema presenta resumen clínico estructurado
    And muestra motivo, evolución, medicación, antecedentes, prioridad y red flags

  Scenario: Resumen alterno con baja confianza de datos
    Given un caso con respuestas ambiguas en el cuestionario de triage
    When el sistema detecta baja confianza semántica en uno o más campos
    Then marca esos campos como pendientes de confirmación
    And advierte al médico antes de que responda al paciente

  Scenario: Guardrail bloquea solicitud de diagnóstico en el contexto del triage
    Given un contexto de triage que activa un patrón de diagnóstico automático
    When el guardrail evalúa el prompt previo al envío al LLM
    Then el sistema rechaza la generación y retorna disclaimer
    And registra el intento bloqueado en log estructurado

  Scenario: Idempotencia al regenerar el resumen
    Given una consulta con resumen ya generado
    When el médico intenta generar el resumen nuevamente
    Then el sistema retorna el resumen existente con HTTP 409
    And no realiza una segunda llamada al LLM

  Scenario: Feedback de utilidad registrado
    Given un médico que ha revisado el resumen clínico de su paciente
    When marca el resumen como útil
    Then el sistema actualiza el campo useful en MongoDB
    And el KPI de utilidad de resumen clínico se incrementa correctamente
```

---

### Historia de Usuario HU-009 – Traducción a Lenguaje Simple para Paciente

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-009 |
| **Title**              | Como paciente quiero recibir explicaciones médicas en lenguaje simple |
| **Parent (Feature)**   | F4.2 – Traducción Paciente-Clínico Bidireccional |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Priority**           | 2 – High |
| **MoSCoW**             | Should |
| **Effort (SP)**        | 5 SP |
| **Risk**               | Medium – Guardrails deben prevenir que la traducción sea interpretada como diagnóstico (R-003, R-006) |
| **Tags**               | ia; traduccion; lenguaje-simple; paciente; should; sprint4 |
| **Depends On**         | HU-005 (resumen clínico activo como contexto de consulta) |

#### Descripción (campo Description)

```
Como paciente
quiero recibir una explicación en lenguaje simple de las orientaciones médicas
    que el médico me ha enviado durante la consulta
para entender mejor la orientación recibida y poder seguir las indicaciones
    sin necesidad de conocer terminología especializada.

Contexto:
Durante una consulta clínica, el médico puede utilizar términos especializados
que el paciente no comprende (p. ej. "bradicardia", "cefalea tensional",
"otalgia"). El módulo de traducción permite al paciente solicitar con un botón
en la app una versión del mensaje del médico en lenguaje cotidiano. El servicio
invoca a Gemini con guardrails activos para asegurar que la respuesta es
orientativa y no constituye un diagnóstico.

Adicionalmente, el médico puede utilizar la función inversa para normalizar la
descripción libre del paciente ("me duele la cabeza desde ayer y tengo náuseas")
a terminología clínica estandarizada, facilitando la documentación sin modificar
el registro oficial sin su confirmación.

Criterios de salida:
- El botón "Explicar en lenguaje simple" está disponible en la app para cada
  mensaje del médico en el chat clínico.
- La respuesta incluye siempre el disclaimer reglamentario visible.
- El guardrail bloquea cualquier texto de entrada que implique diagnóstico o
  prescripción, en ambas direcciones.
- El panel médico tiene el botón "Traducir al lenguaje clínico" junto al campo
  de descripción libre del paciente.
- El texto traducido no se persiste automáticamente en el registro oficial de
  la consulta.
```

#### Criterios de aceptación (campo Acceptance Criteria)

```gherkin
Feature: Traducción bidireccional de lenguaje clínico

  Scenario: Paciente recibe orientación en lenguaje simple
    Given una consulta activa con un mensaje del médico con terminología especializada
    When el paciente presiona el botón de traducción en la app
    Then la app llama a POST /v1/consultations/{id}/translation con direction TO_PATIENT
    And muestra la explicación en lenguaje coloquial
    And presenta el disclaimer de que la orientación no es un diagnóstico

  Scenario: Médico normaliza descripción del paciente a términos clínicos
    Given la descripción libre del paciente ingresada en el panel web
    When el médico presiona el botón de traducción al lenguaje clínico
    Then la pantalla muestra los términos clínicos normalizados
    And el registro oficial de la consulta permanece sin cambios hasta confirmación del médico

  Scenario: Guardrail bloquea texto con solicitud de diagnóstico
    Given un texto del paciente que incluye una solicitud implícita de diagnóstico
    When el servicio de traducción evalúa el prompt
    Then retorna HTTP 422 con mensaje de disclaimer
    And no realiza llamada al LLM
    And el log registra el intento sin almacenar el texto original

  Scenario: Traducción no disponible sin consulta activa
    Given un paciente sin consulta activa
    When intenta llamar al endpoint de traducción
    Then el sistema retorna HTTP 404 con mensaje de consulta no encontrada
    And no realiza ninguna llamada al servicio IA
```

---

## 4. TAREAS

### Tareas de HU-005 – Resumen Clínico Automático Preconsulta

---

#### T-005-01 – Diseño del modelo ClinicalSummary en MongoDB

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Diseñar y documentar el modelo ClinicalSummary en MongoDB |
| **Parent (User Story)**| HU-005 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Design |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Definir el schema Mongoose para la colección ClinicalSummaries:
- Campos obligatorios del contrato ClinicalSummary:
    { consultationId (ref: Consultation), triageSessionId (ref: TriageSession),
      chiefComplaint: string, duration: string, intensity: string,
      associatedSymptoms: string[], currentMedication: string[],
      relevantHistory: string[], estimatedPriority: PriorityLevel,
      redFlags: RedFlag[], confidenceScore: number,
      pendingConfirmationFields: string[], promptVersion: string,
      useful: boolean | null, generatedAt: Date }
- Índice único sobre consultationId para garantizar idempotencia.
- Índice sobre triageSessionId para trazabilidad.
- Anotaciones de auditoría: createdAt, updatedAt (timestamps de Mongoose).
Ubicar en: apps/api/src/consultations/schemas/clinical-summary.schema.ts
```

**Criterios de aceptación de la tarea:**
- El schema compila sin errores y pasa la validación de tipos TypeScript.
- El índice único sobre consultationId está declarado y validado en prueba de integración.
- Todos los campos del contrato ClinicalSummary están presentes con los tipos correctos.
- La documentación del modelo está actualizada en la Wiki (referencia al sprint 4).

---

#### T-005-02 – Implementación del IAOrchestrator: Gemini + RAG para generación de resumen

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar IAOrchestrator con Gemini + RAG para generación de ClinicalSummary |
| **Parent (User Story)**| HU-005 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 6 |

**Descripción:**
```
Implementar el servicio IAOrchestrator que genera el ClinicalSummary:
1. Recibe el triageSessionId y recupera las respuestas del triage de MongoDB.
2. Construye el contexto RAG: consulta el índice vectorial con los síntomas
   principales y recupera los fragmentos de conocimiento clínico relevantes
   (máx. 5 fragmentos, score de similitud >= 0.7).
3. Ensambla el prompt estructurado con: instrucciones del sistema (guardrails),
   contexto RAG, respuestas del triage y versión del prompt (PROMPT_SUMMARY_V1).
4. Invoca IAProviderAdapter.generateClinicalSummary(prompt) con:
   - Timeout: 20 000 ms.
   - Retry: 2 intentos con backoff exponencial de 2 s.
   - Si falla tras reintentos: lanza IAUnavailableException con estado
     AI_UNAVAILABLE para activar el plan de contingencia.
5. Parsea la respuesta del LLM al objeto ClinicalSummary.
6. Calcula el confidenceScore por campo (score de confianza 0–1 basado en
   completitud y coherencia de la respuesta).
7. Marca pendingConfirmationFields si algún campo tiene score < 0.6.
8. Registra en log: consultationId, promptVersion, confidenceScore, latency_ms,
   correlation ID.
Ubicar en: apps/api/src/ia/ia-orchestrator.service.ts
```

**Criterios de aceptación de la tarea:**
- El servicio genera un ClinicalSummary con todos los campos obligatorios.
- El índice RAG devuelve fragmentos relevantes; la prueba unitaria verifica la consulta vectorial.
- El retry se activa cuando el mock del proveedor falla en el primer intento.
- IAUnavailableException se lanza correctamente tras agotar los reintentos.
- El log incluye todos los campos requeridos (promptVersion, confidenceScore, latency_ms).
- El servicio no genera contenido de diagnóstico ni prescripción en ningún escenario de prueba.

---

#### T-005-03 – Implementación del endpoint POST /v1/consultations/{id}/summary/generate

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint de generación de resumen clínico |
| **Parent (User Story)**| HU-005 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar el endpoint de generación del resumen clínico:
- Ruta: POST /v1/consultations/{id}/summary/generate
- Guards: JwtAuthGuard + @Roles('MEDICO') + DoctorVerifiedGuard.
- Precondiciones (validadas en el servicio):
    1. La consulta debe existir → 404 si no.
    2. La consulta debe tener estado OPEN o IN_PROGRESS → 409 si está CLOSED.
    3. El triage asociado debe estar en estado COMPLETED → 409 si no.
    4. No debe existir un ClinicalSummary previo para esa consulta → 409 si ya existe.
- Flujo principal:
    1. Llamar a IAOrchestrator.generateSummary(consultationId, triageSessionId).
    2. Persistir el ClinicalSummary en MongoDB.
    3. Emitir evento WebSocket consultation.summary.ready al room de la consulta
       con { consultationId, summaryId }.
    4. Retornar HTTP 200 con el objeto ClinicalSummary.
- En caso de IAUnavailableException:
    · Persistir un ClinicalSummary con todos los campos vacíos y estado
      AI_UNAVAILABLE.
    · Retornar HTTP 503 con mensaje "Servicio IA no disponible; el médico puede
      continuar con plantilla manual."
- Registrar en log estructurado con correlation ID.
Ubicar en: apps/api/src/consultations/consultations.controller.ts y
           apps/api/src/consultations/consultations.service.ts
```

**Criterios de aceptación de la tarea:**
- Médico VERIFIED con consulta OPEN genera resumen → HTTP 200 con ClinicalSummary.
- Intento de generar resumen duplicado → HTTP 409 sin nueva llamada al LLM.
- Consulta CLOSED → HTTP 409 con mensaje descriptivo.
- Triage no COMPLETED → HTTP 409 con mensaje descriptivo.
- Rol PACIENTE → HTTP 403.
- IA no disponible → HTTP 503 y ClinicalSummary con estado AI_UNAVAILABLE persiste.
- Evento WebSocket consultation.summary.ready emitido correctamente.

---

#### T-005-04 – Implementación de los guardrails de IA para el resumen clínico

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar guardrails que bloquean diagnóstico/prescripción en el resumen IA |
| **Parent (User Story)**| HU-005 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el módulo IAGuardrailService que evalúa el prompt antes de enviarlo
al LLM y la respuesta al recibirla:
- Lista de patrones prohibidos en el prompt de entrada (regex y palabras clave):
    · Solicitudes de diagnóstico: "diagnostica", "qué enfermedad", "qué tiene",
      "confirma diagnóstico", entre otros.
    · Solicitudes de prescripción: "receta", "prescribe", "indica medicamento",
      "qué medicamento tomar", entre otros.
    · Solicitudes de pronóstico definitivo: "va a morir", "cuánto tiempo le queda",
      "es grave definitivamente", entre otros.
- Si detecta patrón prohibido en el prompt:
    · Lanza ForbiddenIARequestException(type, matchedPattern).
    · Registra en log: { event: 'guardrail_blocked', type, consultationId,
      promptVersion, correlation_id } sin incluir el texto original del prompt.
- Validación de la respuesta del LLM (post-procesamiento):
    · Si la respuesta contiene lenguaje diagnóstico o prescriptivo no esperado,
      elimina el fragmento y agrega disclaimer al campo afectado.
- Disclaimer reglamentario obligatorio adjuntado a toda respuesta:
    "El contenido generado es orientativo y no constituye diagnóstico médico,
     prescripción terapéutica ni recomendación definitiva."
- Versioning de prompts: PROMPT_SUMMARY_V1 (registrado en colección PromptVersions).
Ubicar en: apps/api/src/ia/guardrail.service.ts
```

**Criterios de aceptación de la tarea:**
- Un prompt con la palabra "diagnostica" activa el guardrail y lanza ForbiddenIARequestException.
- La respuesta del LLM con contenido diagnóstico inesperado es sanitizada.
- El disclaimer aparece en todos los ClinicalSummary generados.
- El log de bloqueo no incluye el texto original del prompt.
- La versión del prompt queda registrada en cada ClinicalSummary generado.

---

#### T-005-05 – Vista de resumen clínico en el panel médico (Next.js)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar vista de resumen clínico en el panel médico (Next.js) |
| **Parent (User Story)**| HU-005 |
| **Assigned To**        | Desarrollador Web |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar en el panel médico (Next.js + React) la sección de resumen clínico:
- Ruta del panel: /consultations/{id} (sección lateral o pestaña "Resumen Clínico").
- Botón "Generar Resumen Clínico" que llama a
  POST /v1/consultations/{id}/summary/generate con estado de carga (spinner).
- Sección colapsable con los campos del ClinicalSummary:
    · Motivo de consulta, duración, intensidad, síntomas asociados.
    · Medicación actual y antecedentes relevantes.
    · Prioridad estimada con etiqueta de color (LOW=verde, MODERATE=amarillo,
      HIGH=rojo).
    · Lista de red flags con su severidad.
    · Indicador de confianza por campo:
        - Verde: confidenceScore >= 0.8
        - Amarillo: 0.6 <= confidenceScore < 0.8
        - Rojo / "pendiente de confirmación": confidenceScore < 0.6
- Advertencia destacada en banner rojo si hay campos con pending_confirmation.
- Botones de feedback: "✓ Útil" y "✗ No útil" que llaman a
  PATCH /v1/consultations/{id}/summary/feedback.
- Estado de carga y manejo de errores:
    · Spinner mientras se genera el resumen.
    · Mensaje de error con opción de reintentar si el endpoint retorna 5xx.
    · Banner de aviso si el estado es AI_UNAVAILABLE.
- Solo visible para roles MEDICO y ADMIN.
Ubicar en: apps/web/src/app/consultations/[id]/components/ClinicalSummary/
```

**Criterios de aceptación de la tarea:**
- El médico puede generar el resumen con un clic y verlo en menos de 15 s (SLO).
- Los indicadores de confianza muestran el color correcto según el score.
- El banner de advertencia aparece cuando hay campos pending_confirmation.
- Los botones de feedback llaman correctamente al endpoint y actualizan la UI.
- El banner AI_UNAVAILABLE aparece cuando el endpoint retorna 503.
- La sección no es visible para el rol PACIENTE.

---

#### T-005-06 – Pruebas unitarias del IAOrchestrator y del IAGuardrailService

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias para IAOrchestrator y IAGuardrailService |
| **Parent (User Story)**| HU-005 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Escribir pruebas unitarias con Jest para los servicios de IA del resumen clínico:

IAOrchestrator:
- generateSummary con triage completo → ClinicalSummary con todos los campos.
- generateSummary con respuestas ambiguas → campos marcados pending_confirmation.
- generateSummary cuando el proveedor falla en el primer intento → retry exitoso.
- generateSummary cuando el proveedor falla en todos los reintentos → IAUnavailableException.
- Verificar que el log registra promptVersion, confidenceScore y latency_ms.

IAGuardrailService:
- evaluatePrompt con texto diagnóstico → ForbiddenIARequestException.
- evaluatePrompt con texto de prescripción → ForbiddenIARequestException.
- evaluatePrompt con texto válido → pasa sin excepción.
- sanitizeResponse con contenido diagnóstico en respuesta del LLM → fragmento eliminado.
- Verificar que el disclaimer está presente en toda respuesta sanitizada.
- Cobertura mínima objetivo: 80% en IAOrchestrator y 90% en IAGuardrailService.

Mocks requeridos: IAProviderAdapter, VectorIndexRepository, Logger.
Ubicar en:
  apps/api/src/ia/ia-orchestrator.service.spec.ts
  apps/api/src/ia/guardrail.service.spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI sin dependencias reales de LLM ni base de datos.
- Cobertura >= 80% en IAOrchestrator y >= 90% en IAGuardrailService.
- Los mocks de IAProviderAdapter simulan correctamente los escenarios de fallo y éxito.

---

#### T-005-07 – Pruebas de integración del endpoint de resumen clínico

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas de integración para POST /v1/consultations/{id}/summary/generate |
| **Parent (User Story)**| HU-005 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Pruebas de integración con supertest + MongoMemoryServer + mock de IAProviderAdapter:
- Médico VERIFIED + triage COMPLETED → HTTP 200, ClinicalSummary persiste en MongoDB.
- Resumen ya existente → HTTP 409, sin nueva llamada al LLM (verificar mock).
- Triage no COMPLETED → HTTP 409.
- Consulta CLOSED → HTTP 409.
- Rol PACIENTE → HTTP 403.
- Rol no autenticado → HTTP 401.
- IAProviderAdapter lanza IAUnavailableException → HTTP 503, ClinicalSummary con
  estado AI_UNAVAILABLE persiste.
- Evento WebSocket consultation.summary.ready emitido (verificar con mock del hub).
Ubicar en: apps/api/test/clinical-summary.e2e-spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI de forma aislada.
- El mock de IAProviderAdapter cubre escenarios de éxito, retry y fallo total.
- La idempotencia del endpoint está cubierta y verificada.

---

#### T-005-08 – Documentación técnica del resumen clínico en la Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar endpoint y flujo del resumen clínico en la Wiki |
| **Parent (User Story)**| HU-005 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 1 |

**Descripción:**
```
Actualizar la Wiki con:
- POST /v1/consultations/{id}/summary/generate: request (sin body), response 200
  con contrato ClinicalSummary, errores 403/404/409/503.
- PATCH /v1/consultations/{id}/summary/feedback: request { useful: boolean },
  response 200.
- Descripción del flujo completo: triage → IAOrchestrator → RAG → Gemini →
  guardrail → ClinicalSummary → panel médico.
- Diagrama de secuencia textual del flujo de generación.
- Descripción de guardrails activos y lista de patrones prohibidos (sin revelar
  la implementación exacta, solo el comportamiento).
- Política de versionado de prompts y ubicación del registro en MongoDB.
- Plan de contingencia cuando IA no está disponible (AI_UNAVAILABLE).
- Referencia a HU-005, sprint 4, y al SLO de tiempo de generación.
```

**Criterios de aceptación de la tarea:**
- La Wiki refleja el contrato real del endpoint con ejemplos de request/response.
- El flujo de generación es comprensible para un desarrollador nuevo sin leer el código.
- Los guardrails y el plan de contingencia están documentados con sus comportamientos esperados.

---

### Tareas de HU-009 – Traducción Paciente-Clínico Bidireccional

---

#### T-009-01 – Diseño del módulo TranslationService y su contrato de API

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Diseñar el módulo TranslationService y el contrato del endpoint de traducción |
| **Parent (User Story)**| HU-009 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Design |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Definir el contrato del endpoint y el diseño del módulo de traducción:
- Endpoint: POST /v1/consultations/{id}/translation
    · Body: { direction: 'TO_PATIENT' | 'TO_CLINIC', text: string }
    · Response 200: { translatedText: string, disclaimer: string, promptVersion: string }
    · Response 401: JWT ausente o inválido.
    · Response 403: rol sin acceso (solo PACIENTE y MEDICO permitidos en su consulta).
    · Response 404: consulta no encontrada.
    · Response 422: guardrail activa (contenido prohibido detectado).
- Prompts diferenciados por dirección:
    · PROMPT_TRANSLATION_TO_PATIENT_V1: instrucción de simplificar lenguaje médico
      sin emitir diagnóstico ni prescripción.
    · PROMPT_TRANSLATION_TO_CLINIC_V1: instrucción de normalizar descripción libre
      a terminología clínica estandarizada sin interpretar diagnósticamente.
- El TranslationService reutiliza IAProviderAdapter e IAGuardrailService.
- Política de no persistencia: el texto traducido no se guarda en la base de datos;
  solo se registran metadatos en el log (direction, consultationId, promptVersion,
  latency_ms, sin el texto original ni traducido).
Ubicar en: apps/api/src/ia/translation.service.ts
```

**Criterios de aceptación de la tarea:**
- El contrato del endpoint está documentado y aprobado en la sesión de refinamiento.
- Las interfaces TypeScript del body y la response están definidas sin errores.
- Los nombres de las versiones de prompt están registrados en la colección PromptVersions.

---

#### T-009-02 – Implementación de traducción clínico → paciente (lenguaje simple)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar traducción de lenguaje clínico a lenguaje simple para paciente |
| **Parent (User Story)**| HU-009 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el método translateToPatient en TranslationService:
1. Recibe el texto con terminología médica y el contexto opcional de la consulta.
2. Aplica IAGuardrailService.evaluatePrompt para detectar contenido prohibido.
   Si hay contenido prohibido → ForbiddenIARequestException.
3. Construye el prompt con PROMPT_TRANSLATION_TO_PATIENT_V1:
   - Instrucción explícita de no diagnosticar, no prescribir, no pronosticar.
   - Instrucción de usar vocabulario de nivel educativo básico.
   - Texto original a traducir.
4. Invoca IAProviderAdapter.translate(prompt) con timeout 10 000 ms.
5. Aplica IAGuardrailService.sanitizeResponse a la respuesta.
6. Adjunta el disclaimer reglamentario al resultado.
7. Registra en log: { event: 'translation', direction: 'TO_PATIENT', consultationId,
   promptVersion, latency_ms, correlation_id } sin incluir los textos.
Ubicar en: apps/api/src/ia/translation.service.ts
```

**Criterios de aceptación de la tarea:**
- El servicio transforma correctamente términos médicos en lenguaje coloquial en los tests.
- El disclaimer está presente en toda respuesta.
- El guardrail bloquea textos con patrones diagnósticos.
- El log no incluye el texto original ni el traducido.

---

#### T-009-03 – Implementación de traducción paciente → clínico (terminología médica)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar traducción de descripción del paciente a lenguaje clínico estructurado |
| **Parent (User Story)**| HU-009 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el método translateToClinic en TranslationService:
1. Recibe la descripción libre del paciente.
2. Aplica IAGuardrailService.evaluatePrompt.
3. Construye el prompt con PROMPT_TRANSLATION_TO_CLINIC_V1:
   - Instrucción de normalizar a terminología clínica estandarizada (CIE-10 como
     referencia, sin emitir código diagnóstico directo).
   - Instrucción explícita de no diagnosticar ni prescribir.
4. Invoca IAProviderAdapter.translate(prompt) con timeout 10 000 ms.
5. Aplica IAGuardrailService.sanitizeResponse.
6. Adjunta el disclaimer reglamentario.
7. Registra en log los metadatos (sin los textos).
Ubicar en: apps/api/src/ia/translation.service.ts
```

**Criterios de aceptación de la tarea:**
- El servicio normaliza descripciones coloquiales a términos clínicos reconocibles en los tests.
- El disclaimer está presente en toda respuesta.
- El resultado no incluye un código diagnóstico CIE-10 directo ni prescripción.
- El guardrail bloquea descripciones que contienen solicitudes explícitas de diagnóstico.

---

#### T-009-04 – Implementación del endpoint POST /v1/consultations/{id}/translation

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint de traducción bidireccional en ConsultationsController |
| **Parent (User Story)**| HU-009 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el endpoint de traducción:
- Ruta: POST /v1/consultations/{id}/translation
- Guards: JwtAuthGuard + @Roles('PACIENTE', 'MEDICO').
- Validaciones previas en el servicio:
    1. La consulta debe existir → 404 si no.
    2. El solicitante debe ser el paciente dueño o el médico asignado → 403 si no.
    3. La dirección TO_PATIENT solo puede ser solicitada por PACIENTE o MEDICO.
    4. La dirección TO_CLINIC solo puede ser solicitada por MEDICO.
- Flujo principal:
    1. Según direction, invocar TranslationService.translateToPatient o
       TranslationService.translateToClinic.
    2. Si ForbiddenIARequestException → HTTP 422 con mensaje de disclaimer.
    3. Retornar HTTP 200 con { translatedText, disclaimer, promptVersion }.
- No persistir el texto original ni el traducido en MongoDB.
- Registrar en log con correlation ID los metadatos de la operación.
Ubicar en: apps/api/src/consultations/consultations.controller.ts
```

**Criterios de aceptación de la tarea:**
- PACIENTE propietario con direction TO_PATIENT → HTTP 200 con traducción.
- MEDICO asignado con direction TO_CLINIC → HTTP 200 con normalización clínica.
- PACIENTE intenta direction TO_CLINIC → HTTP 403.
- Consulta no encontrada → HTTP 404.
- Texto con contenido diagnóstico → HTTP 422.
- No autenticado → HTTP 401.

---

#### T-009-05 – Integración del botón de traducción en la app React Native (paciente)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar botón "Explicar en lenguaje simple" en el chat de la app (React Native) |
| **Parent (User Story)**| HU-009 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Agregar en la pantalla de chat de la app React Native la función de traducción
para el paciente:
- Icono de ayuda (?) o botón "Explicar en lenguaje simple" junto a cada mensaje
  del médico en el hilo de chat.
- Al presionar, muestra un modal o panel inferior con:
    · Estado de carga (spinner) mientras se llama al endpoint.
    · Texto traducido en lenguaje simple.
    · Disclaimer visible en tipografía secundaria.
    · Botón "Cerrar".
- Llamada a POST /v1/consultations/{id}/translation con direction TO_PATIENT.
- Manejo de errores:
    · HTTP 422 (guardrail): mensaje "No es posible procesar esta solicitud."
    · HTTP 5xx: mensaje "Error al obtener la explicación; intenta de nuevo."
- No modifica el hilo de chat original.
Ubicar en: apps/mobile/src/screens/Chat/components/TranslationModal/
```

**Criterios de aceptación de la tarea:**
- El botón aparece junto a los mensajes del médico y no junto a los del paciente.
- El modal muestra el texto traducido y el disclaimer correctamente.
- Los errores de la API se muestran con mensajes amigables sin exponer detalles técnicos.
- El hilo de chat original no se modifica.

---

#### T-009-06 – Pruebas unitarias del TranslationService

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias para TranslationService |
| **Parent (User Story)**| HU-009 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Escribir pruebas unitarias con Jest para TranslationService:
- translateToPatient con texto médico válido → retorna texto simplificado con disclaimer.
- translateToPatient con texto diagnóstico → ForbiddenIARequestException.
- translateToClinic con descripción coloquial válida → retorna términos clínicos con disclaimer.
- translateToClinic con solicitud explícita de prescripción → ForbiddenIARequestException.
- Fallo del IAProviderAdapter → IAUnavailableException propagada.
- Verificar que el log no registra el texto original ni el traducido en ningún escenario.
- Cobertura mínima objetivo: 85% de TranslationService.
Mocks requeridos: IAProviderAdapter, IAGuardrailService, Logger.
Ubicar en: apps/api/src/ia/translation.service.spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI sin dependencias reales de LLM.
- Cobertura >= 85%.
- Los mocks cubren escenarios de éxito, guardrail activado y fallo del proveedor.

---

#### T-009-07 – Documentación técnica del módulo de traducción en la Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar endpoint y módulo de traducción bidireccional en la Wiki |
| **Parent (User Story)**| HU-009 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 4 |
| **Remaining Work (h)** | 1 |

**Descripción:**
```
Actualizar la Wiki con:
- POST /v1/consultations/{id}/translation: request body, response 200, errores
  401/403/404/422.
- Descripción de las dos direcciones de traducción (TO_PATIENT y TO_CLINIC) con
  ejemplos de entrada y salida esperada.
- Política de no persistencia de los textos y política de privacidad del log.
- Comportamiento del guardrail en el módulo de traducción.
- Disclaimer reglamentario y cuándo aparece.
- Referencia a HU-009, sprint 4, y a la dependencia con HU-005.
```

**Criterios de aceptación de la tarea:**
- La Wiki describe correctamente los dos casos de uso con ejemplos comprensibles.
- La política de privacidad del log (sin almacenar textos) está explícitamente documentada.
- El comportamiento del guardrail en traducción está diferenciado del comportamiento en el resumen.

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
📌 EPIC: E4 – Resumen Clínico y Traductor IA (13 SP total)
│
├── 🔷 FEATURE: F4.1 – Resumen Clínico Automático Preconsulta (8 SP)
│   │
│   └── 📖 USER STORY: HU-005 – Como médico quiero recibir resumen clínico automático (8 SP)
│       ├── [ ] T-005-01 – Diseño del modelo ClinicalSummary en MongoDB (2h)
│       ├── [ ] T-005-02 – IAOrchestrator: Gemini + RAG para generación de resumen (6h)
│       ├── [ ] T-005-03 – Endpoint POST /v1/consultations/{id}/summary/generate (4h)
│       ├── [ ] T-005-04 – Guardrails: bloquear diagnóstico/prescripción en resumen IA (3h)
│       ├── [ ] T-005-05 – Vista de resumen clínico en panel médico (Next.js) (4h)
│       ├── [ ] T-005-06 – Pruebas unitarias IAOrchestrator y IAGuardrailService (4h)
│       ├── [ ] T-005-07 – Pruebas de integración del endpoint de resumen (3h)
│       └── [ ] T-005-08 – Documentación técnica del resumen en la Wiki (1h)
│
└── 🔷 FEATURE: F4.2 – Traducción Paciente-Clínico Bidireccional (5 SP)
    │
    └── 📖 USER STORY: HU-009 – Como paciente quiero explicaciones en lenguaje simple (5 SP)
        ├── [ ] T-009-01 – Diseño del TranslationService y contrato de API (2h)
        ├── [ ] T-009-02 – Traducción clínico → paciente (lenguaje simple) (3h)
        ├── [ ] T-009-03 – Traducción paciente → clínico (terminología médica) (3h)
        ├── [ ] T-009-04 – Endpoint POST /v1/consultations/{id}/translation (3h)
        ├── [ ] T-009-05 – Botón "Explicar en lenguaje simple" en la app (React Native) (3h)
        ├── [ ] T-009-06 – Pruebas unitarias del TranslationService (3h)
        └── [ ] T-009-07 – Documentación técnica del módulo de traducción en la Wiki (1h)
```

**Total horas estimadas Sprint 4 (Épica 4):** 42 horas de trabajo  
**Total Story Points Épica 4:** 13 SP (HU-005: 8 SP + HU-009: 5 SP)  
**Sprint objetivo:** Sprint 4

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|-----------|-----------|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` – Sprint 4 y MoSCoW |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` – Actividad 3: Consulta (slice B) |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` – HU-005, HU-009 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` – HU-005 |
| DoR / DoD | `docs/wiki/08-DoR-DoD.md` |
| KPIs y SLOs | `docs/wiki/09-Observabilidad-KPIs.md` |
| Riesgos | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` |
| Épica dependiente (triage) | `docs/epics/Azure-Boards-Epica2.md` (HU-003, HU-004) |
| Épica dependiente (consulta) | `docs/epics/Azure-Boards-Epica3.md` (HU-006 consume el resumen) |

### API endpoints de esta épica (contratos del Plan Maestro)
- `POST /v1/consultations/{id}/summary/generate` → T-005-03
- `PATCH /v1/consultations/{id}/summary/feedback` → T-005-05
- `POST /v1/consultations/{id}/translation` → T-009-04

### KPIs impactados por esta épica
- **Utilidad de resumen clínico >= 75%**: el botón de feedback de HU-005 alimenta directamente este KPI.
- **Tiempo a primera respuesta médica (reducción >= 30%)**: el resumen clínico preconsulta reduce el tiempo de recolección de antecedentes, impactando directamente la velocidad de primera respuesta.
- **SLO Tiempo de generación IA < 15 000 ms P95**: aplicable al endpoint de generación del resumen y al de traducción.

### Riesgos asociados
- **R-003 (Resumen IA inestable)**: versionar prompts, pruebas de regresión semántica en cada sprint y política de retry con IAUnavailableException.
- **R-004 (Cambio de proveedor IA)**: encapsular todas las llamadas a Gemini en IAProviderAdapter; sin llamadas directas al SDK de Gemini fuera de este adaptador.
- **R-006 (Exposición de datos sensibles)**: política de no persistencia de textos en las traducciones; log anonimizado con solo metadatos; guardrails activos en ambos servicios.
