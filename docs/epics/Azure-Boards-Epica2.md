# Azure Boards – Épica 2: Triage Inteligente por Especialidad

## Propósito del documento
Este documento contiene toda la información necesaria para implementar la **Épica 2 – Triage Inteligente por Especialidad** en Azure Boards, incluyendo la épica, sus features, historias de usuario con criterios Gherkin, y todas las tareas de desarrollo, pruebas y documentación asociadas. Cada sección indica los campos exactos que se deben completar al crear el ítem en Azure Boards.

---

## 1. ÉPICA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Epic |
| **ID**                 | E2 |
| **Title**              | Triage Inteligente por Especialidad |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Priority**           | 1 – Critical |
| **Business Value**     | Priorizar casos y reducir incertidumbre del paciente; habilita la cola médica y el resumen clínico |
| **Risk**               | Medium – Precisión del motor IA/reglas puede requerir ajuste iterativo con criterio clínico |
| **Effort (Story Points total estimado)** | 16 SP |
| **Tags**               | triage; IA; red-flags; prioridad; medicina-general; odontologia; must |
| **Start Date**         | Sprint 2 |
| **Target Date**        | Cierre Sprint 3 |

### Descripción (campo Description)

```
La Épica 2 cubre el flujo completo de triage inteligente para las dos especialidades del MVP:
Medicina General y Odontología.

Objetivos de negocio:
- Permitir que el paciente describa sus síntomas de forma guiada y estructurada antes de
  contactar a un médico, reduciendo la incertidumbre y el tiempo de consulta.
- Clasificar automáticamente la urgencia del caso en tres niveles: LOW, MODERATE o HIGH.
- Detectar señales de alarma clínicas (red flags) que requieren atención presencial inmediata.
- Generar un registro estructurado de síntomas que alimenta el resumen clínico automático
  de la Épica 4.

Funcionalidades cubiertas:
- Cuestionario guiado adaptativo por especialidad (Medicina General y Odontología).
- Motor de análisis con IA (Gemini + RAG) para Medicina General.
- Motor de reglas determinísticas de red flags para Odontología y validación cruzada MG.
- Clasificación de prioridad: LOW | MODERATE | HIGH.
- Registro de sesión de triage con evidencia de síntomas y factores de riesgo.
- Cola médica ordenada por prioridad resultante del triage.

Restricciones:
- El triage NO emite diagnósticos ni prescripciones (guardrail obligatorio por alcance del MVP).
- La IA actúa solo como apoyo de priorización, no como sistema de diagnóstico.
- El flujo depende de que el paciente esté autenticado (Épica 1 completada).
- Los cuestionarios son fijos en el MVP; no hay cuestionarios dinámicos por historial.
```

### Criterios de aceptación de la épica

```
- Un paciente de Medicina General puede completar el cuestionario guiado y recibir
  una prioridad (LOW, MODERATE o HIGH) generada por el motor IA.
- Un paciente de Odontología puede completar el cuestionario y el motor de reglas
  detecta red flags y asigna la prioridad correspondiente.
- Un caso con al menos una red flag de severidad CRITICAL es clasificado como HIGH
  independientemente de las otras respuestas.
- Un médico ve la cola de casos ordenada por prioridad (HIGH primero).
- La sesión de triage queda persistida con todos los campos requeridos para alimentar
  el resumen clínico (Épica 4).
- El motor IA nunca devuelve texto que implique un diagnóstico o prescripción
  (guardrail verificado en pruebas).
- El log de auditoría registra cada ejecución de triage con correlation ID, especialidad,
  prioridad resultante y tiempo de procesamiento.
```

### Acceptance Criteria – formato Gherkin (nivel épica)

```gherkin
Feature: Triage inteligente por especialidad en SaludDeUna

  Scenario: Paciente de Medicina General recibe prioridad tras triage
    Given un paciente autenticado que selecciona Medicina General
    When completa el cuestionario guiado con síntomas y factores de riesgo
    Then el sistema genera una prioridad LOW, MODERATE o HIGH
    And guarda la sesión de triage con evidencia de síntomas
    And el caso aparece en la cola médica con la prioridad asignada

  Scenario: Paciente de Odontología con red flag crítica clasificado como HIGH
    Given un paciente autenticado que selecciona Odontología
    When responde el cuestionario indicando dolor intenso, fiebre y dificultad al tragar
    Then el motor de reglas detecta al menos una red flag de severidad CRITICAL
    And el sistema clasifica el caso como prioridad HIGH
    And recomienda atención presencial inmediata

  Scenario: Motor IA rechaza generar diagnóstico
    Given una sesión de triage activa
    When el sistema procesa los síntomas con el modelo Gemini
    Then la respuesta no contiene términos de diagnóstico ni prescripción
    And se aplica el guardrail de contenido antes de persistir el resultado

  Scenario: Cola médica ordenada por prioridad
    Given múltiples sesiones de triage completadas con diferentes prioridades
    When un médico VERIFIED accede a GET /v1/consultations/queue
    Then los casos HIGH aparecen primero, seguidos de MODERATE y luego LOW
```

---

## 2. FEATURES

### Feature F2.1 – Flujo Triage Medicina General

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F2.1 |
| **Title**              | Flujo Triage Medicina General |
| **Parent (Epic)**      | E2 – Triage Inteligente por Especialidad |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 8 SP |
| **Tags**               | triage; medicina-general; IA; Gemini; RAG; prioridad; must |

#### Descripción (campo Description)

```
Permite a un paciente de Medicina General responder un cuestionario guiado y obtener
una clasificación de prioridad asistida por IA.

Alcance funcional:
- Cuestionario estructurado de Medicina General: motivo de consulta, duración de síntomas,
  intensidad (escala 1-10), síntomas asociados, medicación actual, antecedentes relevantes.
- Endpoint POST /v1/triage/sessions → crea una sesión de triage con specialty=GENERAL_MEDICINE.
- Endpoint POST /v1/triage/sessions/{sessionId}/answers → guarda las respuestas del paciente.
- Endpoint POST /v1/triage/sessions/{sessionId}/analyze → ejecuta el análisis IA con Gemini +
  RAG y retorna la prioridad (LOW | MODERATE | HIGH) y los red flags detectados.
- Guardrail obligatorio: el resultado del modelo no puede contener diagnósticos ni
  prescripciones (filtro de contenido antes de persistir).
- Registro de auditoría: cada ejecución de triage se registra con correlation ID, userId,
  specialty, priority, latency_ms y timestamp.
- Pantalla de triage en la app React Native con flujo paso a paso.

Criterios de salida del feature:
- Un paciente puede completar el cuestionario de MG y recibir una prioridad válida.
- La sesión de triage queda persistida con todos los campos necesarios para el resumen
  clínico (F4.1 – Épica 4).
- El guardrail rechaza cualquier respuesta del modelo que contenga términos de diagnóstico.
- El log de auditoría refleja cada ejecución con latencia y prioridad resultante.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Triage de Medicina General con IA

  Scenario: Triage principal con priorización exitosa
    Given un paciente autenticado en Medicina General
    When completa el cuestionario con síntomas válidos y presiona "Analizar"
    Then el sistema retorna prioridad LOW, MODERATE o HIGH
    And guarda la sesión con motivo, síntomas, medicación y factores de riesgo
    And el log registra la ejecución con correlation ID y latencia

  Scenario: Triage con cuestionario incompleto
    Given un paciente en el flujo de triage de MG
    When intenta enviar respuestas con campos obligatorios vacíos
    Then el sistema retorna HTTP 400 con detalle de campos faltantes
    And no ejecuta el análisis IA

  Scenario: Guardrail rechaza diagnóstico en respuesta IA
    Given una respuesta del modelo Gemini que contiene lenguaje de diagnóstico
    When el guardrail de contenido procesa la respuesta
    Then el sistema descarta el texto y marca el campo de análisis como pendiente
    And registra una alerta en el log estructurado

  Scenario: Sesión de triage disponible para resumen clínico
    Given una sesión de triage completada para Medicina General
    When el módulo de resumen clínico (F4.1) solicita los datos de la sesión
    Then retorna los campos chiefComplaint, duration, intensity, associatedSymptoms,
         meds, history y priority correctamente estructurados
```

---

### Feature F2.2 – Flujo Triage Odontología

> **Nota Azure Boards:** HU-004 tiene como parent primario esta feature (F2.2). Crear además
> un vínculo de tipo **"Related"** entre HU-004 y F2.3 para reflejar que las tareas del motor
> de reglas y la cola médica (T-004-01, T-004-04) forman parte del alcance de F2.3.

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F2.2 |
| **Title**              | Flujo Triage Odontología |
| **Parent (Epic)**      | E2 – Triage Inteligente por Especialidad |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 5 SP |
| **Tags**               | triage; odontologia; red-flags; prioridad; must |

#### Descripción (campo Description)

```
Permite a un paciente de Odontología responder un cuestionario especializado y obtener
una clasificación de prioridad basada en reglas clínicas odontológicas.

Alcance funcional:
- Cuestionario odontológico: tipo de dolor (agudo/sordo/pulsátil), localización,
  intensidad (1-10), duración, fiebre asociada, dificultad al tragar, traumatismo
  reciente, sangrado, hinchazón facial.
- Endpoint POST /v1/triage/sessions → crea sesión con specialty=DENTISTRY.
- Endpoint POST /v1/triage/sessions/{sessionId}/answers → guarda respuestas odontológicas.
- Endpoint POST /v1/triage/sessions/{sessionId}/analyze → ejecuta el motor de reglas F2.3
  y retorna prioridad y red flags detectados.
- La prioridad HIGH se asigna automáticamente si existe al menos un red flag de
  severidad CRITICAL (ver F2.3).
- Pantalla de triage odontológico en la app React Native adaptada al cuestionario dental.
- Integración con cola médica priorizada para especialidad Odontología.

Criterios de salida del feature:
- Un paciente de Odontología puede completar el cuestionario y recibir su prioridad.
- Los red flags odontológicos son detectados y registrados en la sesión.
- La sesión queda disponible para el resumen clínico de la Épica 4.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Triage de Odontología con motor de reglas

  Scenario: Triage odontológico con prioridad alta por red flag
    Given un paciente de Odontología con dolor intenso, fiebre y dificultad al tragar
    When el motor de reglas evalúa las respuestas
    Then el sistema detecta red flags de severidad CRITICAL
    And asigna prioridad HIGH al caso
    And recomienda atención presencial inmediata

  Scenario: Triage odontológico sin red flags críticos
    Given un paciente de Odontología con molestia dental leve sin signos sistémicos
    When el motor de reglas evalúa las respuestas
    Then no se detectan red flags CRITICAL
    And el sistema asigna prioridad LOW o MODERATE según intensidad
    And el caso entra en la cola médica con seguimiento digital

  Scenario: Cuestionario odontológico incompleto
    Given un paciente en el flujo de triage de Odontología
    When intenta enviar respuestas sin completar campos obligatorios
    Then el sistema retorna HTTP 400
    And no ejecuta el análisis de reglas

  Scenario: Sesión odontológica integrada con cola médica
    Given una sesión de triage de Odontología completada con prioridad HIGH
    When un médico de Odontología VERIFIED accede a la cola
    Then el caso aparece en la posición más alta de la cola de esa especialidad
```

---

### Feature F2.3 – Motor de Red Flags por Especialidad

> **Nota Azure Boards:** Esta feature agrupa las tareas del motor de reglas (T-004-01),
> la cola médica priorizada (T-004-04) y las pruebas de concurrencia (T-004-07), que son
> compartidas con HU-004. Crear un vínculo **"Related"** desde HU-004 hacia F2.3 para
> mantener la trazabilidad completa en el tablero.

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F2.3 |
| **Title**              | Motor de Red Flags por Especialidad |
| **Parent (Epic)**      | E2 – Triage Inteligente por Especialidad |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 3 SP |
| **Tags**               | red-flags; reglas-clinicas; medicina-general; odontologia; seguridad-clinica; must |

#### Descripción (campo Description)

```
Implementa el motor centralizado de detección de señales de alarma clínicas (red flags)
para Medicina General y Odontología, usado por ambos flujos de triage.

Alcance funcional:
- Definición del contrato RedFlag { code, specialty, severity: CRITICAL|WARNING|INFO, evidence }.
- Catálogo de red flags de Medicina General (ejemplos):
    · RF-MG-001: dolor torácico + dificultad respiratoria → CRITICAL
    · RF-MG-002: pérdida de consciencia → CRITICAL
    · RF-MG-003: fiebre > 39°C + rigidez de nuca → CRITICAL
    · RF-MG-004: dolor abdominal intenso súbito → CRITICAL
    · RF-MG-005: alteraciones visuales repentinas → WARNING
- Catálogo de red flags de Odontología (ejemplos):
    · RF-OD-001: fiebre + dificultad para tragar → CRITICAL
    · RF-OD-002: hinchazón facial con trismus → CRITICAL
    · RF-OD-003: sangrado no controlado > 30 min → CRITICAL
    · RF-OD-004: traumatismo dental con pérdida de pieza → WARNING
    · RF-OD-005: dolor pulsátil intenso (>7/10) nocturno → WARNING
- Módulo RedFlagsEngine: función evaluate(answers, specialty) → RedFlag[].
- La función es determinística y testeable sin dependencia de IA externa.
- Si el resultado incluye al menos 1 red flag CRITICAL → prioridad = HIGH (override).
- Si solo hay red flags WARNING → prioridad = max(prioridad_base, MODERATE).
- Integración con el endpoint de análisis de triage de ambas especialidades.

Criterios de salida del feature:
- El motor evalúa correctamente los catálogos de ambas especialidades.
- El override de prioridad por red flag CRITICAL funciona en todos los casos.
- Cobertura de pruebas unitarias >= 90% del RedFlagsEngine.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Motor de detección de red flags clínicas

  Scenario: Red flag CRITICAL fuerza prioridad HIGH
    Given una sesión de triage con respuestas que activan RF-MG-001 (dolor torácico)
    When el RedFlagsEngine evalúa las respuestas
    Then retorna un array con al menos una RedFlag de severity=CRITICAL
    And la prioridad resultante es HIGH independientemente de otras respuestas

  Scenario: Solo red flags WARNING elevan prioridad a MODERATE
    Given una sesión de triage con respuestas que activan RF-OD-004 (traumatismo dental)
    When el RedFlagsEngine evalúa las respuestas
    Then retorna un array con RedFlag de severity=WARNING
    And la prioridad resultante es al menos MODERATE

  Scenario: Sin red flags la prioridad se mantiene según respuestas base
    Given una sesión de triage sin respuestas que activen ningún red flag
    When el RedFlagsEngine evalúa las respuestas
    Then retorna un array vacío de red flags
    And la prioridad es determinada solo por la intensidad de síntomas reportada

  Scenario: Motor no emite diagnósticos
    Given cualquier combinación de respuestas de triage
    When el RedFlagsEngine genera el resultado
    Then el campo evidence solo contiene descripciones de síntomas observados
    And no contiene términos de diagnóstico ni recomendaciones de medicación
```

---

## 3. HISTORIAS DE USUARIO

### HU-003 – Triage Guiado por IA para Medicina General

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-003 |
| **Title**              | Como paciente de medicina general quiero responder un triage guiado por IA para describir mejor mis síntomas y riesgo |
| **Parent (Feature)**   | F2.1 – Flujo Triage Medicina General |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\IA / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Story Points**       | 8 |
| **Risk**               | Medium – La calidad del análisis IA depende del prompt engineering y del corpus RAG |
| **Dependencias**       | HU-001 (autenticación de paciente activa) |
| **Tags**               | triage; medicina-general; IA; Gemini; RAG; must; sprint2 |

#### Descripción completa (campo Description)

```
Como paciente de Medicina General
Quiero responder un cuestionario guiado asistido por IA sobre mis síntomas
Para que el sistema clasifique la urgencia de mi caso antes de que un médico me atienda.

Contexto:
El paciente llega a la plataforma con una necesidad de salud no urgente pero relevante.
Sin un triage estructurado, el médico debe invertir tiempo recopilando antecedentes básicos
durante la consulta. El triage guiado resuelve esto de forma previa, asistido por IA para
detectar patrones de riesgo que el paciente podría no reportar espontáneamente.

Flujo funcional:
1. Paciente selecciona Medicina General en la pantalla principal de la app.
2. La app crea una sesión de triage (POST /v1/triage/sessions).
3. El paciente responde el cuestionario paso a paso en la app (POST /v1/triage/sessions/{id}/answers).
4. Al finalizar, la app dispara el análisis (POST /v1/triage/sessions/{id}/analyze).
5. El backend ejecuta el motor IA (Gemini + RAG) y el RedFlagsEngine.
6. El resultado (prioridad + red flags) se devuelve al paciente con indicaciones de siguiente paso.
7. Se crea automáticamente un caso en la cola médica con la prioridad asignada.

Restricciones funcionales:
- El cuestionario tiene un máximo de 10 preguntas en el MVP; preguntas adaptativas quedan
  para versiones futuras.
- El análisis IA debe completarse en < 15 segundos (SLO de resumen IA del Plan Maestro).
- El guardrail es obligatorio: ningún texto con lenguaje de diagnóstico puede ser devuelto
  al paciente ni persistido en la sesión.
- La sesión de triage debe quedar en estado COMPLETED para que el resumen clínico (E4)
  pueda consumirla; si el análisis falla, queda en FAILED con mensaje de error.

Notas de UX:
- El cuestionario debe mostrar una barra de progreso (paso X de Y).
- Cada pregunta tiene respuesta por selección múltiple o escala numérica; no se permite
  texto libre en el MVP para facilitar el análisis estructurado.
- Al recibir prioridad HIGH, la app muestra un mensaje destacado con recomendación de
  atención urgente y la opción de contactar al médico directamente.
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Triage IA para medicina general

  Scenario: Triage principal con priorización
    Given un paciente autenticado en Medicina General
    When responde el cuestionario guiado completo
    Then el sistema genera prioridad LOW, MODERATE o HIGH
    And guarda evidencia de síntomas y factores de riesgo
    And el caso entra en la cola médica con esa prioridad

  Scenario: Triage alterno incompleto
    Given un paciente que abandona el cuestionario
    When intenta enviar respuestas incompletas
    Then el sistema solicita completar campos obligatorios
    And no ejecuta análisis de prioridad

  Scenario: Guardrail de no-diagnóstico activo
    Given un cuestionario completado con síntomas que podrían inducir diagnóstico
    When el motor IA procesa las respuestas
    Then la respuesta no contiene lenguaje de diagnóstico ni prescripción
    And el resultado solo describe nivel de urgencia y síntomas reportados

  Scenario: Tiempo de análisis dentro del SLO
    Given una sesión de triage completada
    When se dispara el análisis con POST /v1/triage/sessions/{id}/analyze
    Then el sistema retorna la respuesta en menos de 15 segundos
    And el log registra la latencia del análisis IA
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y feature definidos (E2, F2.1)
- [x] Prioridad MoSCoW asignada (Must)
- [x] Criterios Gherkin documentados (principal y alterno)
- [x] Dependencias identificadas (HU-001 completada en Sprint 1)
- [x] Estimación acordada (8 SP)
- [x] Datos/ambientes necesarios identificados (MongoDB dev, API key Gemini dev, corpus RAG base)
- [x] Riesgos anotados (calidad IA requiere ajuste iterativo de prompts)
- [x] Guardrail de no-diagnóstico definido y acordado con el equipo
- [x] Estructura del cuestionario de MG acordada (máximo 10 preguntas, MVP)
- [x] Responsable de validación funcional asignado

#### Definition of Done (DoD) – Checklist

- [ ] Endpoints `POST /v1/triage/sessions`, `POST /v1/triage/sessions/{id}/answers` y `POST /v1/triage/sessions/{id}/analyze` implementados y revisados por al menos un par
- [ ] Motor IA integrado con Gemini + RAG; prompts versionados en repositorio
- [ ] Guardrail de no-diagnóstico implementado y verificado con pruebas
- [ ] RedFlagsEngine para Medicina General implementado con catálogo v1
- [ ] Sesión de triage persistida en MongoDB con todos los campos del contrato ClinicalSummary
- [ ] Pruebas unitarias del TriageService en verde (cobertura >= 80%)
- [ ] Pruebas de integración de los tres endpoints en verde
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] Log de auditoría registra cada ejecución de triage con correlation ID, specialty, priority y latency_ms
- [ ] Pantalla de triage en React Native implementada con barra de progreso y flujo paso a paso
- [ ] Tiempo de análisis < 15 s verificado en entorno de pruebas (al menos 10 ejecuciones)
- [ ] Documentación técnica actualizada en Wiki (endpoints, contrato de sesión, política de guardrail)
- [ ] Demo funcional aprobada por el equipo
- [ ] Historia pasada a estado Done con evidencia enlazada (PR + test results)

---

### HU-004 – Clasificación de Prioridad por Red Flags en Odontología

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-004 |
| **Title**              | Como paciente odontológico quiero recibir clasificación de prioridad por red flags para identificar casos que requieren atención presencial |
| **Parent (Feature)**   | F2.2 / F2.3 – Flujo Triage Odontología / Motor de Red Flags |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Story Points**       | 8 |
| **Risk**               | Medium – Las reglas clínicas odontológicas deben ser validadas con criterio profesional |
| **Dependencias**       | HU-003 (motor de triage base y sesiones activas) |
| **Tags**               | triage; odontologia; red-flags; prioridad; cola-medica; must; sprint3 |

#### Descripción completa (campo Description)

```
Como paciente odontológico
Quiero responder un cuestionario específico sobre mi problema dental y recibir una
clasificación de prioridad basada en señales de alarma clínicas
Para saber si necesito atención presencial urgente o puedo esperar una consulta digital.

Contexto:
Los problemas odontológicos pueden escalar rápidamente (absceso dental, celulitis
cervicofacial, traumatismo con pérdida de pieza). Un triage con reglas clínicas
odontológicas permite identificar estos casos graves antes de que el paciente espere
en una cola estándar, potencialmente reduciendo complicaciones.

Flujo funcional:
1. Paciente selecciona Odontología en la pantalla principal de la app.
2. La app crea sesión con specialty=DENTISTRY (POST /v1/triage/sessions).
3. El paciente responde el cuestionario odontológico (POST /v1/triage/sessions/{id}/answers).
4. Al finalizar, se dispara el análisis (POST /v1/triage/sessions/{id}/analyze).
5. El RedFlagsEngine evalúa el catálogo odontológico.
6. Si hay red flags CRITICAL → prioridad HIGH y alerta de atención presencial urgente.
7. Si solo hay red flags WARNING o ninguno → prioridad MODERATE/LOW según intensidad.
8. El caso entra en la cola médica de Odontología con la prioridad asignada.

Restricciones funcionales:
- El motor de reglas odontológico es determinístico; no usa IA generativa (solo el MG la usa).
- Las reglas son codificadas y versionadas en el repositorio; cambios requieren revisión
  con criterio clínico y PR aprobado.
- La cola médica filtra por especialidad: un médico de Odontología solo ve casos DENTISTRY.
- Si el paciente reporta traumatismo con pérdida de pieza permanente, el sistema informa
  de la ventana de reimplantación (60 minutos) como nota informativa, sin diagnosticar.

Notas de UX:
- El cuestionario odontológico tiene preguntas específicas de localización y tipo de dolor.
- Al detectar prioridad HIGH, la pantalla muestra un banner de advertencia con instrucciones
  para buscar atención presencial y el número de emergencias local como referencia.
- La app no bloquea la consulta digital aunque la prioridad sea HIGH; el paciente decide.
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Detección de red flags odontológicas

  Scenario: Caso principal de alto riesgo
    Given un paciente odontológico con dolor intenso y fiebre
    When el motor de reglas evalúa síntomas
    Then el sistema clasifica prioridad HIGH
    And recomienda atención presencial inmediata

  Scenario: Caso alterno sin red flags críticas
    Given un paciente odontológico con molestia leve
    When el motor de reglas evalúa síntomas
    Then el sistema asigna prioridad LOW o MODERATE
    And mantiene seguimiento digital

  Scenario: Cola médica de Odontología ordenada por prioridad
    Given múltiples casos de Odontología con diferentes prioridades
    When un médico VERIFIED de Odontología accede a GET /v1/consultations/queue
    Then los casos HIGH aparecen primero
    And la cola solo muestra casos de especialidad DENTISTRY

  Scenario: Nota informativa sobre traumatismo dental
    Given un paciente que reporta pérdida de pieza permanente por traumatismo
    When el sistema detecta el red flag RF-OD-004
    Then muestra una nota informativa sobre la ventana de reimplantación de 60 minutos
    And no emite diagnóstico ni indicación médica directa
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y features definidos (E2, F2.2 y F2.3)
- [x] Prioridad MoSCoW asignada (Must)
- [x] Criterios Gherkin documentados (principal y alterno)
- [x] Dependencias identificadas (HU-003 debe estar completada en Sprint 2)
- [x] Estimación acordada (8 SP)
- [x] Catálogo de red flags odontológicas v1 acordado (mínimo 5 reglas)
- [x] Datos/ambientes necesarios identificados (MongoDB dev, sin dependencia de IA externa)
- [x] Riesgos anotados (reglas clínicas requieren validación con profesional dental)
- [x] Responsable de validación funcional asignado

#### Definition of Done (DoD) – Checklist

- [ ] RedFlagsEngine extendido con catálogo odontológico v1 (mínimo 5 red flags)
- [ ] Endpoint `POST /v1/triage/sessions/{id}/analyze` soporta specialty=DENTISTRY
- [ ] Lógica de override de prioridad por red flag CRITICAL implementada y verificada
- [ ] Cola médica `GET /v1/consultations/queue` filtra por especialidad y ordena por prioridad
- [ ] Código revisado por al menos un par
- [ ] Pruebas unitarias del RedFlagsEngine odontológico en verde (cobertura >= 90%)
- [ ] Pruebas de integración del flujo odontológico completo en verde
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] Pantalla de triage odontológico en React Native implementada con banner de alerta HIGH
- [ ] Log de auditoría registra ejecución con specialty=DENTISTRY, red flags detectados y prioridad
- [ ] Reglas del catálogo versionadas en repositorio con comentarios de justificación clínica
- [ ] Documentación técnica actualizada en Wiki (catálogo de red flags, lógica de override, cola priorizada)
- [ ] Demo funcional aprobada por el equipo
- [ ] Historia pasada a estado Done con evidencia enlazada (PR + test results)

---

## 4. TAREAS

### Tareas de HU-003 – Triage Guiado por IA para Medicina General

---

#### T-003-01 – Diseño del modelo TriageSession en MongoDB

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Diseñar esquema Mongoose TriageSession con campos de especialidad, respuestas y análisis |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Crear el schema Mongoose para el modelo TriageSession incluyendo:
- _id (ObjectId generado automáticamente)
- patientId (ref Patient, required)
- specialty (String, enum: ['GENERAL_MEDICINE', 'DENTISTRY'], required)
- status (String, enum: ['IN_PROGRESS', 'COMPLETED', 'FAILED'], default: 'IN_PROGRESS')
- answers (Array de { questionId, questionText, answerValue, answeredAt })
- analysis (Object, optional):
    · priority (String, enum: ['LOW', 'MODERATE', 'HIGH'])
    · redFlags (Array de RedFlag { code, specialty, severity, evidence })
    · aiSummary (String, optional) – texto generado por IA, sin diagnósticos
    · analysisDurationMs (Number) – latencia del análisis IA
    · guardrailApplied (Boolean) – indica si el guardrail filtró contenido
- completedAt (Date, optional)
- createdAt, updatedAt (timestamps automáticos)

Contrato RedFlag (sub-documento):
- code (String, required) – e.g. 'RF-MG-001'
- specialty (String, enum: ['GENERAL_MEDICINE', 'DENTISTRY'])
- severity (String, enum: ['CRITICAL', 'WARNING', 'INFO'])
- evidence (String) – descripción del síntoma que activó el flag

Agregar índice compuesto en (patientId, status) para consultas de sesiones activas.
Ubicar en: apps/api/src/triage/schemas/triage-session.schema.ts
```

**Criterios de aceptación de la tarea:**
- Schema creado y exportado correctamente.
- El estado por defecto de una sesión nueva es IN_PROGRESS.
- Los sub-documentos de answers y redFlags se validan con los enums correctos.
- Índice compuesto (patientId, status) verificado en MongoDB local.

---

#### T-003-02 – Implementación del endpoint POST /v1/triage/sessions

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint POST /v1/triage/sessions para iniciar sesión de triage |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Implementar el endpoint de creación de sesión de triage:
- Módulo: TriageModule (NestJS)
- Ruta: POST /v1/triage/sessions
- Protegido con JwtAuthGuard + @Roles('PACIENTE')
- DTO de entrada: CreateTriageSessionDto { specialty: 'GENERAL_MEDICINE' | 'DENTISTRY' }
- Lógica:
    1. Verificar que el paciente no tenga una sesión IN_PROGRESS para la misma especialidad.
       Si existe → retornar HTTP 409 con mensaje "Ya tienes una sesión de triage activa".
    2. Crear documento TriageSession con status=IN_PROGRESS.
    3. Retornar HTTP 201 con { sessionId, specialty, status, questions[] }.
    · El campo questions[] contiene el cuestionario completo para la especialidad seleccionada
      (cargado desde el repositorio de preguntas; ver T-003-05).
- Registrar evento de inicio de triage en log estructurado con correlation ID.
Ubicar en: apps/api/src/triage/
```

**Criterios de aceptación de la tarea:**
- Endpoint retorna 201 con sessionId y cuestionario correcto según especialidad.
- Retorna 409 si ya existe sesión IN_PROGRESS para esa especialidad.
- Retorna 400 si specialty no es un valor válido del enum.
- Solo pacientes autenticados pueden crear sesiones (401 sin token, 403 con otro rol).

---

#### T-003-03 – Implementación del endpoint POST /v1/triage/sessions/{sessionId}/answers

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint de guardado de respuestas del cuestionario de triage |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Implementar el endpoint que guarda las respuestas del paciente:
- Ruta: POST /v1/triage/sessions/{sessionId}/answers
- Protegido con JwtAuthGuard + @Roles('PACIENTE')
- DTO: SaveTriageAnswersDto { answers: [{ questionId, answerValue }][] }
- Lógica:
    1. Validar que sessionId existe y pertenece al paciente autenticado → 404 si no.
    2. Validar que el status es IN_PROGRESS → 400 si ya está COMPLETED o FAILED.
    3. Validar que todos los questionIds son válidos para la especialidad de la sesión.
    4. Actualizar el array answers en el documento TriageSession.
    5. Retornar HTTP 200 con { sessionId, answersCount, isComplete }.
    · isComplete = true cuando se han respondido todas las preguntas obligatorias.
- Permite envío parcial (el paciente puede responder en múltiples llamadas).
Ubicar en: apps/api/src/triage/
```

**Criterios de aceptación de la tarea:**
- Endpoint actualiza las respuestas correctamente y retorna 200.
- Retorna 404 si la sesión no existe o no pertenece al paciente.
- Retorna 400 si la sesión no está en estado IN_PROGRESS.
- El campo isComplete refleja correctamente si todas las preguntas obligatorias fueron respondidas.

---

#### T-003-04 – Implementación del endpoint POST /v1/triage/sessions/{sessionId}/analyze

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint de análisis IA de la sesión de triage (Gemini + RedFlagsEngine) |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 5 |

**Descripción:**
```
Implementar el endpoint de análisis de triage que integra IA y motor de reglas:
- Ruta: POST /v1/triage/sessions/{sessionId}/analyze
- Protegido con JwtAuthGuard + @Roles('PACIENTE')
- Lógica:
    1. Validar que la sesión existe, pertenece al paciente y tiene todas las respuestas
       obligatorias (isComplete=true). Si no → HTTP 422.
    2. Ejecutar RedFlagsEngine.evaluate(answers, specialty) → redFlags[].
    3. Si specialty=GENERAL_MEDICINE: llamar a GeminiService.analyzeTriage(answers, redFlags)
       para obtener aiSummary y prioridad sugerida por el modelo.
    4. Aplicar guardrail de contenido (GuardrailService.check(aiSummary)):
       - Si detecta lenguaje de diagnóstico/prescripción → descartar aiSummary,
         marcar guardrailApplied=true, registrar alerta en log.
    5. Calcular prioridad final:
       - Si hay redFlag CRITICAL → prioridad = HIGH (override).
       - Si hay redFlag WARNING y prioridad_base < MODERATE → prioridad = MODERATE.
       - En otro caso → usar prioridad_base de la IA (MG) o de intensidad (Odontología).
    6. Actualizar TriageSession: status=COMPLETED, analysis{priority, redFlags, aiSummary,
       analysisDurationMs, guardrailApplied}.
    7. Crear automáticamente un documento Consultation en estado PENDING con la prioridad
       asignada para que entre en la cola médica.
    8. Retornar HTTP 200 con { sessionId, priority, redFlags, message }.
- Medir y registrar analysisDurationMs; emitir alerta en log si > 15000 ms.
- Registrar evento de auditoría con todos los campos de observabilidad requeridos.
Ubicar en: apps/api/src/triage/ y apps/api/src/triage/services/gemini-triage.service.ts
```

**Criterios de aceptación de la tarea:**
- Endpoint retorna 200 con prioridad válida y lista de red flags detectados.
- Retorna 422 si la sesión no está completa.
- El override de prioridad HIGH por red flag CRITICAL funciona correctamente.
- El guardrail bloquea respuestas con lenguaje de diagnóstico y registra la alerta.
- La sesión queda en estado COMPLETED con todos los campos del análisis.
- Se crea automáticamente un documento Consultation con la prioridad asignada.
- El tiempo de respuesta se registra y se alerta si supera 15 s.

---

#### T-003-05 – Implementación del RedFlagsEngine para Medicina General

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar RedFlagsEngine con catálogo de red flags para Medicina General |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el motor centralizado de detección de red flags:
- Clase RedFlagsEngine con método estático evaluate(answers: TriageAnswer[], specialty: Specialty): RedFlag[].
- Catálogo de red flags Medicina General (mínimo 5, versionado en archivo JSON separado):
    · RF-MG-001: dolor torácico + dificultad respiratoria → CRITICAL
    · RF-MG-002: pérdida de consciencia o síncope → CRITICAL
    · RF-MG-003: fiebre > 39°C + rigidez de nuca → CRITICAL
    · RF-MG-004: dolor abdominal intenso de inicio súbito → CRITICAL
    · RF-MG-005: alteraciones visuales repentinas → WARNING
- El catálogo se carga desde apps/api/src/triage/rules/red-flags-mg.json para facilitar
  actualización sin modificar código.
- La función es pura (sin efectos secundarios) y testeable de forma aislada.
- Cobertura de pruebas unitarias >= 90%.
Ubicar en: apps/api/src/triage/engines/red-flags.engine.ts
```

**Criterios de aceptación de la tarea:**
- El motor detecta correctamente cada uno de los 5 red flags de MG con sus respectivos
  síntomas disparadores.
- La función retorna un array vacío cuando no hay combinaciones de riesgo.
- El catálogo es modificable sin cambiar el código del motor.
- Cobertura de pruebas >= 90% en CI.

---

#### T-003-06 – Implementación del cuestionario de Medicina General y pantalla de triage en React Native

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar pantalla de triage MG en la app React Native con cuestionario paso a paso |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 5 |

**Descripción:**
```
Implementar el flujo de triage de Medicina General en la app móvil:

TriageSelectionScreen:
- Pantalla de selección de especialidad (Medicina General / Odontología).
- Botón "Iniciar Triage" llama a POST /v1/triage/sessions con la especialidad seleccionada.
- Al éxito, navega a TriageQuestionnaireScreen.

TriageQuestionnaireScreen (Medicina General):
- Muestra las preguntas del cuestionario una a una con barra de progreso (paso X de N).
- Tipos de respuesta soportados: selección única, selección múltiple, escala numérica (1-10).
- Botón "Siguiente" guarda la respuesta actual (POST /v1/triage/sessions/{id}/answers).
- Botón "Finalizar y analizar" (visible solo cuando isComplete=true) dispara el análisis.

TriageResultScreen:
- Muestra la prioridad resultante con color: verde (LOW), amarillo (MODERATE), rojo (HIGH).
- Lista de red flags detectados con su descripción.
- Para prioridad HIGH: banner de advertencia con recomendación de atención urgente.
- Botón "Ver mi caso en consultas" navega al listado de consultas del paciente.

Ubicar en: apps/mobile/src/screens/triage/
```

**Criterios de aceptación de la tarea:**
- El flujo completo (selección → cuestionario → resultado) funciona sin errores.
- La barra de progreso refleja el avance correcto.
- El banner HIGH se muestra correctamente cuando la prioridad es HIGH.
- La app no crashea con ninguna respuesta válida del backend.
- Los errores del backend se muestran con mensajes comprensibles al usuario.

---

#### T-003-07 – Implementación del GuardrailService para respuestas IA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar GuardrailService para filtrar respuestas IA con lenguaje de diagnóstico |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Backend / IA |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el servicio de guardrail para respuestas del modelo IA:
- Clase GuardrailService con método check(text: string): { safe: boolean, violations: string[] }.
- Lista de términos y patrones prohibidos (versionada en apps/api/src/triage/rules/guardrail-rules.json):
    · Términos de diagnóstico: "diagnóstico de", "padece de", "tiene [enfermedad]", etc.
    · Términos de prescripción: "tomar [medicamento]", "administrar", "recetar", etc.
    · Patrones de afirmación clínica: "es [condición médica]", "sufre de", etc.
- Si safe=false: registrar alerta en log estructurado con nivel WARN, los violations encontrados
  y el correlation ID de la sesión.
- El GuardrailService es independiente del modelo IA; puede reutilizarse en otros módulos.
Ubicar en: apps/api/src/triage/services/guardrail.service.ts
```

**Criterios de aceptación de la tarea:**
- El servicio detecta correctamente textos con lenguaje de diagnóstico (al menos 10 casos de prueba).
- El servicio clasifica como safe textos con lenguaje de priorización/urgencia sin diagnóstico.
- Cada violación queda registrada en el log con el correlation ID.

---

#### T-003-08 – Pruebas unitarias del TriageService y RedFlagsEngine MG

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias para TriageService y RedFlagsEngine (Medicina General) |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Escribir pruebas unitarias con Jest:
TriageService:
- createSession: creación exitosa con specialty válida.
- createSession: conflicto con sesión IN_PROGRESS existente → ConflictException.
- saveAnswers: guardado exitoso; retorna isComplete correcto.
- saveAnswers: sesión no encontrada → NotFoundException.
- analyzeSession: sesión incompleta → UnprocessableEntityException.
- analyzeSession: red flag CRITICAL → override a prioridad HIGH.
- analyzeSession: guardrail activo → aiSummary descartado, guardrailApplied=true.

RedFlagsEngine:
- Cada uno de los 5 red flags de MG: verificar detección correcta.
- Combinación sin red flags → array vacío.
- Combinación con múltiples red flags → todos retornados correctamente.

GuardrailService:
- Texto con diagnóstico → safe=false con violations.
- Texto con prioridad/urgencia → safe=true.

Cobertura mínima objetivo: 80% de TriageService, 90% de RedFlagsEngine.
Ubicar en: apps/api/src/triage/triage.service.spec.ts y red-flags.engine.spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI.
- Cobertura >= 80% TriageService, >= 90% RedFlagsEngine.
- No hay dependencias reales de base de datos ni de API IA en estas pruebas.

---

#### T-003-09 – Pruebas de integración de los endpoints de triage MG

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas de integración para los tres endpoints de triage de Medicina General |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Pruebas de integración con supertest + MongoMemoryServer (Gemini mockeado):
- POST /v1/triage/sessions con specialty=GENERAL_MEDICINE → 201 con sessionId y preguntas.
- POST /v1/triage/sessions con sesión IN_PROGRESS existente → 409.
- POST /v1/triage/sessions/{id}/answers con respuestas válidas → 200 con isComplete.
- POST /v1/triage/sessions/{id}/answers con sessionId inexistente → 404.
- POST /v1/triage/sessions/{id}/analyze con sesión completa → 200 con prioridad y redFlags.
- POST /v1/triage/sessions/{id}/analyze: mock de Gemini retorna diagnóstico → guardrail activo.
- POST /v1/triage/sessions/{id}/analyze: mock activa RF-MG-001 → prioridad=HIGH.
- POST /v1/triage/sessions/{id}/analyze con sesión incompleta → 422.
Ubicar en: apps/api/test/triage-mg.e2e-spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests de integración pasan en CI de forma aislada.
- El mock de Gemini permite probar el guardrail sin llamadas reales a la API.
- El flujo completo (crear → responder → analizar) se cubre end-to-end.

---

#### T-003-10 – Documentación técnica del módulo de triage MG en Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar endpoints de triage de Medicina General y política de guardrail en la Wiki |
| **Parent (User Story)**| HU-003 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 2 |
| **Remaining Work (h)** | 1 |

**Descripción:**
```
Actualizar la Wiki con:
- POST /v1/triage/sessions: request, response 201, errores 400/409.
- POST /v1/triage/sessions/{id}/answers: request, response 200 con isComplete, errores 400/404.
- POST /v1/triage/sessions/{id}/analyze: request, response 200 con prioridad y redFlags, errores 404/422.
- Contrato completo del objeto TriageSession con todos los campos.
- Catálogo de red flags de Medicina General v1.
- Política de guardrail: qué se filtra, cómo se registra, cómo se gestiona el resultado.
- Referencia a HU-003 y al sprint de implementación.
```

**Criterios de aceptación de la tarea:**
- La Wiki refleja los contratos reales de los endpoints implementados.
- Un nuevo desarrollador puede implementar un cliente del triage sin leer el código fuente.
- La política de guardrail es comprensible para el equipo de producto y clínico.

---

### Tareas de HU-004 – Clasificación de Prioridad por Red Flags en Odontología

---

#### T-004-01 – Extensión del RedFlagsEngine con catálogo de Odontología

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Extender RedFlagsEngine con catálogo de red flags para Odontología |
| **Parent (User Story)**| HU-004 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Extender el RedFlagsEngine existente (T-003-05) con el catálogo odontológico:
- Catálogo de red flags Odontología (mínimo 5, versionado en red-flags-od.json):
    · RF-OD-001: fiebre + dificultad para tragar → CRITICAL
    · RF-OD-002: hinchazón facial con trismus (dificultad para abrir boca) → CRITICAL
    · RF-OD-003: sangrado no controlado > 30 min → CRITICAL
    · RF-OD-004: traumatismo dental con pérdida de pieza permanente → WARNING
    · RF-OD-005: dolor pulsátil intenso (≥ 8/10) nocturno → WARNING
- El método evaluate(answers, specialty) ya carga el catálogo correcto según specialty.
- Agregar lógica de nota informativa para RF-OD-004: incluir campo specialNote con texto
  sobre la ventana de reimplantación sin lenguaje de diagnóstico.
- Mantener el catálogo modificable sin cambiar el código del motor.
Ubicar en: apps/api/src/triage/rules/red-flags-od.json y red-flags.engine.ts
```

**Criterios de aceptación de la tarea:**
- El motor detecta los 5 red flags de Odontología con sus síntomas disparadores.
- RF-OD-001 y RF-OD-002 fuerzan prioridad HIGH correctamente.
- RF-OD-004 incluye el campo specialNote con la nota informativa.
- La extensión no rompe los red flags de Medicina General existentes.
- Cobertura de pruebas del catálogo odontológico >= 90%.

---

#### T-004-02 – Adaptación del endpoint de análisis para especialidad Odontología

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Adaptar endpoint POST /v1/triage/sessions/{id}/analyze para specialty=DENTISTRY |
| **Parent (User Story)**| HU-004 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Adaptar la lógica del endpoint de análisis para el flujo de Odontología:
- Cuando specialty=DENTISTRY:
    1. Ejecutar RedFlagsEngine.evaluate(answers, 'DENTISTRY').
    2. NO llamar al modelo Gemini (el triage de Odontología es determinístico en el MVP).
    3. Calcular prioridad_base desde la intensidad del dolor reportada:
       · intensidad >= 8 → HIGH (sin necesidad de red flag)
       · intensidad 5-7 → MODERATE
       · intensidad < 5 → LOW
    4. Aplicar override de prioridad por red flags (misma lógica que MG).
    5. Si RF-OD-004 está presente, incluir specialNote en la respuesta al cliente.
    6. Actualizar TriageSession y crear Consultation con la prioridad final.
    7. Retornar HTTP 200 con { sessionId, priority, redFlags, specialNotes[] }.
- Registrar en log con specialty=DENTISTRY y todos los campos de auditoría requeridos.
```

**Criterios de aceptación de la tarea:**
- El análisis de Odontología no realiza ninguna llamada a la API de Gemini.
- La prioridad base se calcula correctamente desde la intensidad.
- El override por red flags CRITICAL funciona igual que en MG.
- El campo specialNotes se incluye en la respuesta cuando RF-OD-004 está presente.

---

#### T-004-03 – Cuestionario de Odontología y pantalla de triage en React Native

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar pantalla de triage odontológico en la app React Native |
| **Parent (User Story)**| HU-004 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Extender el flujo de triage en la app móvil para Odontología:

TriageQuestionnaireScreen (Odontología):
- Preguntas específicas de Odontología: tipo de dolor, localización en boca, intensidad,
  duración, fiebre, dificultad para tragar/abrir boca, traumatismo, sangrado, hinchazón.
- Mismo flujo paso a paso con barra de progreso que en MG.
- Al recibir prioridad HIGH con RF-OD-001 o RF-OD-002: mostrar banner de emergencia
  odontológica con instrucción de buscar atención presencial inmediata.
- Al recibir RF-OD-004: mostrar nota informativa sobre ventana de reimplantación
  con mensaje claro de que no es un diagnóstico médico.

TriageResultScreen (Odontología):
- Misma pantalla de resultado que MG adaptada para mostrar specialNotes si existen.
- El banner HIGH tiene color rojo y texto específico para urgencia dental.

Reutilizar los componentes genéricos de TriageQuestionnaireScreen donde sea posible;
solo personalizar las preguntas y los mensajes contextuales de Odontología.
Ubicar en: apps/mobile/src/screens/triage/ (extender los componentes existentes de MG)
```

**Criterios de aceptación de la tarea:**
- El cuestionario de Odontología muestra las preguntas específicas correctas.
- El banner de emergencia dental se muestra cuando la prioridad es HIGH.
- La nota de RF-OD-004 se muestra correctamente cuando está presente en la respuesta.
- No hay regresiones en el flujo de Medicina General.

---

#### T-004-04 – Implementación de la cola médica priorizada y filtrada por especialidad

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar GET /v1/consultations/queue con filtro de especialidad y orden por prioridad |
| **Parent (User Story)**| HU-004 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el endpoint de cola médica priorizada:
- Módulo: ConsultationsModule (NestJS)
- Ruta: GET /v1/consultations/queue
- Protegido con JwtAuthGuard + @Roles('MEDICO') + DoctorVerifiedGuard (implementado en T-002-05)
- Query params:
    · specialty (opcional): 'GENERAL_MEDICINE' | 'DENTISTRY' – si se omite, retorna la
      especialidad del médico autenticado (extraída del JWT).
    · status (opcional): 'PENDING' | 'IN_PROGRESS' – default: 'PENDING'.
    · limit (opcional): número de casos a retornar – default: 20, max: 50.
- Lógica de ordenamiento:
    1. Filtrar por specialty del médico y status=PENDING (o según query param).
    2. Ordenar por prioridad: HIGH (1) → MODERATE (2) → LOW (3).
    3. Dentro de la misma prioridad, ordenar por createdAt ASC (primero el más antiguo).
- Respuesta HTTP 200: { total, queue: [{ consultationId, patientName, priority, specialty,
  createdAt, waitingMinutes, redFlagsCount }] }
- El campo waitingMinutes es calculado como (now - createdAt) en minutos.
Ubicar en: apps/api/src/consultations/
```

**Criterios de aceptación de la tarea:**
- Un médico VERIFIED solo ve casos de su especialidad por defecto.
- Los casos HIGH aparecen antes que MODERATE, que aparecen antes que LOW.
- Dentro de la misma prioridad, el caso más antiguo aparece primero.
- Un médico PENDING recibe 403 (guard de verificación activo).
- El campo waitingMinutes se calcula correctamente.

---

#### T-004-05 – Pruebas unitarias del RedFlagsEngine Odontología y lógica de cola

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias para catálogo de red flags odontológicos y cola priorizada |
| **Parent (User Story)**| HU-004 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Escribir pruebas unitarias con Jest:
RedFlagsEngine (Odontología):
- Cada uno de los 5 red flags de OD: verificar detección correcta.
- RF-OD-001 + RF-OD-002: ambos generan CRITICAL y fuerzan HIGH.
- RF-OD-004: genera WARNING y specialNote presente.
- Sin red flags + intensidad 8 → prioridad HIGH por intensidad.
- Sin red flags + intensidad 5 → prioridad MODERATE.
- Sin red flags + intensidad 3 → prioridad LOW.

ConsultationsService:
- getQueue con casos de múltiples prioridades → orden correcto (HIGH, MODERATE, LOW).
- getQueue filtra por specialty del médico.
- getQueue calcula waitingMinutes correctamente.
- Médico PENDING lanza ForbiddenException (testeado via guard).

Cobertura mínima: 90% RedFlagsEngine OD, 80% ConsultationsService.
Ubicar en: apps/api/src/triage/red-flags.engine.spec.ts (extender) y
           apps/api/src/consultations/consultations.service.spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI.
- Las coberturas mínimas se cumplen.
- No hay regresiones en las pruebas de red flags de Medicina General.

---

#### T-004-06 – Pruebas de integración del flujo de triage odontológico y cola médica

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas de integración para el flujo de triage de Odontología y la cola médica |
| **Parent (User Story)**| HU-004 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Pruebas de integración con supertest + MongoMemoryServer:
Triage Odontología:
- POST /v1/triage/sessions con specialty=DENTISTRY → 201 con preguntas odontológicas.
- Flujo completo: crear sesión → responder → analizar con RF-OD-001 activo → prioridad HIGH.
- Flujo completo: sin red flags + intensidad 3 → prioridad LOW.
- Flujo completo: RF-OD-004 activo → specialNotes presente en respuesta.

Cola médica:
- GET /v1/consultations/queue como médico VERIFIED de Odontología → casos DENTISTRY ordenados.
- GET /v1/consultations/queue como médico PENDING → 403.
- GET /v1/consultations/queue con múltiples prioridades → orden HIGH > MODERATE > LOW.
- GET /v1/consultations/queue como médico de MG → no muestra casos DENTISTRY.

Ubicar en: apps/api/test/triage-od.e2e-spec.ts y consultations-queue.e2e-spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI de forma aislada.
- El flujo completo de Odontología (sin IA) se cubre end-to-end.
- La cola médica muestra el comportamiento correcto de filtrado y ordenamiento.

---

#### T-004-07 – Pruebas de concurrencia base del módulo de triage

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Ejecutar prueba de concurrencia base para el módulo de triage (10 sesiones simultáneas) |
| **Parent (User Story)**| HU-004 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Ejecutar una prueba de concurrencia base para el módulo de triage:
- Herramienta: k6 o Artillery (según el stack del equipo) o script de Node.js con Promise.all.
- Escenario: 10 pacientes diferentes ejecutan el flujo completo de triage (crear → responder →
  analizar) de forma simultánea.
- Verificar que:
    1. No hay errores 500 en ninguna solicitud.
    2. El tiempo de respuesta P95 del endpoint /analyze es < 15 s para MG
       (con Gemini mockeado para reproducibilidad).
    3. Los 10 documentos TriageSession quedan en estado COMPLETED sin mezcla de datos.
    4. Las 10 consultas se crean en la cola médica con la prioridad correcta.
- Registrar los resultados como artefacto del sprint.
Ubicar en: apps/api/test/performance/triage-concurrency.test.js
```

**Criterios de aceptación de la tarea:**
- 10 sesiones concurrentes se completan sin errores 500.
- P95 del análisis MG (con mock) < 15 s.
- Ninguna sesión mezcla respuestas de otro paciente.
- Los resultados quedan documentados como artefacto del sprint.

---

#### T-004-08 – Documentación técnica del módulo de triage odontológico y cola médica en Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar triage de Odontología, catálogo de red flags y cola médica priorizada en la Wiki |
| **Parent (User Story)**| HU-004 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 3 |
| **Remaining Work (h)** | 1 |

**Descripción:**
```
Actualizar la Wiki con:
- Catálogo completo de red flags de Odontología v1 con código, descripción, severidad
  y síntomas disparadores.
- Lógica de override de prioridad por red flags (CRITICAL → HIGH, WARNING → MODERATE).
- Diferencias entre triage MG (con IA) y triage OD (determinístico).
- GET /v1/consultations/queue: parámetros, ordenamiento, campos de respuesta.
- Descripción del campo specialNotes y cuándo se incluye.
- Referencia a HU-004 y al sprint de implementación.
```

**Criterios de aceptación de la tarea:**
- La Wiki explica claramente las diferencias entre el triage de MG y el de Odontología.
- El catálogo de red flags de OD está documentado con todos los campos.
- La lógica de la cola médica priorizada es comprensible sin leer el código.

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
📌 EPIC: E2 – Triage Inteligente por Especialidad (16 SP total)
│
├── 🔷 FEATURE: F2.1 – Flujo Triage Medicina General (8 SP)
│   │
│   └── 📖 USER STORY: HU-003 – Como paciente de MG quiero triage guiado por IA (8 SP)
│       ├── ✅ T-003-01 – Diseño modelo TriageSession en MongoDB (2h)
│       ├── ✅ T-003-02 – Endpoint POST /v1/triage/sessions (2h)
│       ├── ✅ T-003-03 – Endpoint POST /v1/triage/sessions/{id}/answers (2h)
│       ├── ✅ T-003-04 – Endpoint POST /v1/triage/sessions/{id}/analyze + IA (5h)
│       ├── ✅ T-003-05 – RedFlagsEngine catálogo Medicina General (3h)
│       ├── ✅ T-003-06 – Pantalla triage MG en React Native (5h)
│       ├── ✅ T-003-07 – GuardrailService para respuestas IA (3h)
│       ├── ✅ T-003-08 – Pruebas unitarias TriageService + RedFlagsEngine MG (3h)
│       ├── ✅ T-003-09 – Pruebas de integración endpoints triage MG (3h)
│       └── ✅ T-003-10 – Documentación técnica triage MG en Wiki (1h)
│
├── 🔷 FEATURE: F2.2 – Flujo Triage Odontología (5 SP)
│   │
│   └── 📖 USER STORY: HU-004 – Como paciente odontológico quiero clasificación por red flags (8 SP)*
│       ├── ✅ T-004-01 – RedFlagsEngine catálogo Odontología (3h)
│       ├── ✅ T-004-02 – Adaptar endpoint /analyze para DENTISTRY (2h)
│       ├── ✅ T-004-03 – Pantalla triage odontológico en React Native (4h)
│       └── [comparte tareas con F2.3]
│
└── 🔷 FEATURE: F2.3 – Motor de Red Flags por Especialidad (3 SP)
    │
    └── 📖 USER STORY: HU-004 – (continuación) Red flags y cola médica priorizada
        ├── ✅ T-004-04 – GET /v1/consultations/queue con prioridad y filtro (3h)
        ├── ✅ T-004-05 – Pruebas unitarias RedFlagsEngine OD + cola (3h)
        ├── ✅ T-004-06 – Pruebas de integración triage OD y cola médica (3h)
        ├── ✅ T-004-07 – Prueba de concurrencia base del módulo triage (2h)
        └── ✅ T-004-08 – Documentación técnica OD y cola en Wiki (1h)

* HU-004 tiene parent F2.2 en Azure Boards pero sus tareas cubren también F2.3.
  Crear el vínculo de "Related" entre HU-004 y F2.3 en Azure Boards.
```

**Total horas estimadas Sprint 2 (HU-003):** 29 horas de trabajo  
**Total horas estimadas Sprint 3 (HU-004):** 21 horas de trabajo  
**Total Story Points Épica 2:** 16 SP (HU-003: 8 SP + HU-004: 8 SP)  
**Sprints objetivo:** Sprint 2 (HU-003) y Sprint 3 (HU-004)

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|-----------|-----------|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` – Sprints 2 y 3, API /v1/triage/* |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` – Actividad 2: Captura y priorización |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` – HU-003, HU-004 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` – HU-003, HU-004 |
| DoR / DoD | `docs/wiki/08-DoR-DoD.md` |
| KPIs y SLOs | `docs/wiki/09-Observabilidad-KPIs.md` – AI Summary Time < 15 s, Red flags confirmadas >= 60% |
| Riesgos | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` |
| Épica anterior | `docs/epics/Azure-Boards-Epica1.md` – E1 debe estar completada antes de Sprint 2 |

### API endpoints de esta épica (contratos del Plan Maestro)
- `POST /v1/triage/sessions` → T-003-02
- `POST /v1/triage/sessions/{sessionId}/answers` → T-003-03
- `POST /v1/triage/sessions/{sessionId}/analyze` → T-003-04 / T-004-02
- `GET /v1/consultations/queue` → T-004-04

### KPIs impactados por esta épica
- **KPI Red flags relevantes confirmadas >= 60%**: el motor de red flags genera los casos que
  el médico debe validar en la consulta; la tasa de confirmación mide la calidad del catálogo.
- **KPI Tiempo a primera respuesta médica**: la cola priorizada reduce el tiempo de espera
  de los casos más urgentes.
- **SLO P95 latencia API < 1500 ms**: los endpoints de triage (excepto /analyze con IA)
  deben cumplir este SLO; /analyze tiene su propio SLO de < 15 s.

### Riesgos asociados
- **R-003 (Calidad de IA)**: El guardrail y los prompts del motor Gemini deben ser iterados;
  congelar el prompt base al inicio del Sprint 2 y registrar cambios como versionado explícito.
- **R-006 (Reglas clínicas)**: El catálogo de red flags debe ser revisado por alguien con
  criterio clínico antes del Sprint 3; documentar la fuente de cada regla en el JSON del catálogo.
- **R-002 (Concurrencia WebSocket)**: La cola médica priorizada es la base del chat en tiempo
  real (Épica 3); asegurar que la estructura de Consultation soporta el modelo de eventos WS.
