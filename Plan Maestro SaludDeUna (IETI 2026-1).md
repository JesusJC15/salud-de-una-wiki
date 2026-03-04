# Plan Maestro SaludDeUna (IETI 2026-1)

## Resumen
Este plan cubre **Sprint 0 + 9 sprints de ejecución** (10 en total), alineado con `Project-Definition IETI-2026` y el contexto de `SaludDeUna`.  
Objetivo del semestre: entregar un MVP funcional de comunicación clínica inteligente para **Medicina General y Odontología**, con **IA (RAG + LLM + reglas clínicas)**, **chat en tiempo real**, observabilidad completa y trazabilidad Scrum en Azure DevOps (Wiki/HU) con código en GitHub.

Fechas críticas cerradas:
1. Inception: **25 de febrero de 2026 a 4 de marzo de 2026**.
2. Presentación Inception: **9 de marzo de 2026**.

## Alcance cerrado

### In Scope (MVP semestre)
1. App paciente en React Native.
2. Panel médico/admin web en React + Next.
3. Backend NestJS + MongoDB.
4. IA con Gemini + RAG + reglas clínicas de seguridad.
5. Triage predictivo no diagnóstico con prioridad `baja|moderada|alta`.
6. Red flags por especialidad (Medicina General y Odontología).
7. Resumen clínico automático preconsulta.
8. Chat clínico asíncrono con actualizaciones en tiempo real (WebSocket).
9. Seguimiento post-consulta y evolución del caso.
10. Flujo de monetización simulado.
11. Validación semiautomática REThUS por admin.
12. Observabilidad: logs estructurados, métricas técnicas, dashboard y 4 KPIs de negocio.

### Out of Scope
1. Diagnóstico o prescripción automática por IA.
2. Videoconsulta en vivo.
3. Integración real con pasarela de pagos productiva.
4. Integración oficial directa certificada con sistemas hospitalarios/HCE productivos.

## Cambios y adiciones de interfaces públicas

### Canales cliente
1. `PacienteApp` (React Native): registro, triage, chat, seguimiento, historial.
2. `DoctorAdminWeb` (React + Next): cola clínica, chat médico, validación REThUS, dashboards.

### API REST `v1` (NestJS)
1. `POST /v1/auth/patient/register`
2. `POST /v1/auth/doctor/register`
3. `POST /v1/auth/login`
4. `POST /v1/admin/doctors/{doctorId}/rethus-verify`
5. `POST /v1/triage/sessions`
6. `POST /v1/triage/sessions/{sessionId}/answers`
7. `POST /v1/triage/sessions/{sessionId}/analyze`
8. `POST /v1/consultations`
9. `GET /v1/consultations/queue`
10. `POST /v1/consultations/{id}/messages`
11. `POST /v1/consultations/{id}/summary/generate`
12. `POST /v1/followups`
13. `POST /v1/patients/medical-history`
14. `GET /v1/patients/{id}/timeline`
15. `GET /v1/dashboard/technical`
16. `GET /v1/dashboard/business`
17. `POST /v1/billing/simulate-checkout`
18. `GET /v1/knowledge/articles`
19. `POST /v1/knowledge/articles/{id}/approve`

### WebSocket events (`namespace: consultation`)
1. `consultation.message.created`
2. `consultation.status.changed`
3. `consultation.priority.updated`
4. `consultation.queue.updated`
5. `consultation.followup.reminder.triggered`

### Tipos/contratos clave
1. `PriorityLevel = LOW | MODERATE | HIGH`
2. `Specialty = GENERAL_MEDICINE | DENTISTRY`
3. `RedFlag { code, specialty, severity, evidence }`
4. `ClinicalSummary { chiefComplaint, duration, intensity, associatedSymptoms, meds, history, priority, redFlags }`
5. `RethusVerification { doctorId, status, checkedBy, checkedAt, evidenceUrl }`
6. `KpiSnapshot { periodStart, periodEnd, metricName, value, target, status }`

## KPIs y SLOs cerrados

### KPIs de negocio (obligatorios)
1. Tiempo a primera respuesta médica.
2. Calidad/uso del resumen clínico (porcentaje de casos donde el médico lo marca útil).
3. Tasa de red flags relevantes confirmadas por médico.
4. Retención de seguimiento a 7 días.

### SLO técnico (nivel básico acordado)
1. P95 latencia chat < 1.5 s.
2. Tiempo de generación de resumen IA < 15 s.
3. Disponibilidad mensual >= 98.0%.

## Plan por sprint (decision complete)

1. **Sprint 0 (Inception, hasta 4 marzo 2026):** wiki completa en Azure DevOps con título, integrantes, resumen, problema, Canvas, arquitectura base, épicas/features, HU INVEST con Gherkin, DoR/DoD, MoSCoW, story map, plan de observabilidad, 2 HU principales con IA, guion de presentación del 9 marzo.
2. **Sprint 1:** base técnica y repositorios; monorepo; auth paciente/médico; RBAC; estructura Mongo; skeleton RN y Web; CI GitHub Actions; despliegue inicial en Azure.
3. **Sprint 2:** módulo triage para Medicina General; cuestionario estructurado; motor de reglas red flags v1; clasificación de prioridad; registro de auditoría IA.
4. **Sprint 3:** módulo triage para Odontología; ajustes de reglas por especialidad; cola clínica por prioridad para panel médico; pruebas de concurrencia base.
5. **Sprint 4:** generación de resumen clínico automático con Gemini + RAG; guardrails (no diagnóstico/no prescripción); traducción paciente-clínico y clínico-paciente.
6. **Sprint 5:** chat clínico real-time con WebSocket; estados de consulta; persistencia de mensajes; notificaciones internas; hardening de seguridad de sesión.
7. **Sprint 6:** seguimiento post-consulta automatizado; timeline evolutivo del paciente; indicadores de cambio de síntomas; recordatorios.
8. **Sprint 7:** panel médico/admin avanzado; dashboard técnico y de negocio; validación semiautomática REThUS; gestión de banco de conocimiento validado.
9. **Sprint 8:** monetización simulada (planes/pago por consulta); analítica de conversión simulada; optimización de performance para cumplir SLO.
10. **Sprint 9 (hardening y cierre):** pruebas E2E completas, prueba de carga y concurrencia, correcciones, documentación final, demo final, retrospectiva y backlog postcurso.

## MoSCoW del producto

### Must
1. Registro/login paciente y médico con RBAC.
2. Verificación semiautomática REThUS.
3. Triage IA + red flags + prioridad en 2 especialidades.
4. Resumen clínico automático.
5. Chat real-time y cola clínica.
6. Seguimiento post-consulta y timeline.
7. Logs estructurados + métricas + dashboard.
8. 4 KPIs de negocio implementados y medibles.
9. Flujo de monetización simulado.

### Should
1. Banco de conocimiento validado con aprobaciones médicas.
2. Traducción bidireccional de lenguaje clínico.
3. Alertas inteligentes de seguimiento.

### Could
1. Recomendaciones de contenido preventivo personalizadas.
2. Versionado avanzado de prompts y evaluación automática offline.

### Won’t (este semestre)
1. Videollamadas.
2. Prescripción/diagnóstico automático.
3. Integraciones clínicas productivas certificadas.
4. Pasarela real de pagos productiva.

## Pruebas y criterios de aceptación

### Pruebas obligatorias
1. Unitarias para reglas de prioridad y red flags.
2. Integración API para flujos de consulta, resumen, seguimiento y REThUS admin.
3. E2E de paciente y médico (alta prioridad, moderada, baja).
4. Carga/concurrencia para chat y cola de casos.
5. Seguridad básica: auth, autorización por rol, validación de input, sanitización de logs.

### Escenarios de aceptación mínimos
1. Caso crítico odontológico genera prioridad alta y alerta en cola médica.
2. Médico recibe resumen útil antes de responder.
3. Seguimiento post-consulta detecta empeoramiento y re-prioriza.
4. Dashboard muestra métricas técnicas y 4 KPIs de negocio con datos reales del entorno de pruebas.
5. IA rechaza solicitudes de diagnóstico/prescripción y aplica disclaimer.

## DevOps y gobernanza

1. Código en GitHub (monorepo) con ramas `main`, `develop`, `feature/*`.
2. GitHub Actions para lint, test, build y deploy por ambiente.
3. Azure DevOps para Wiki, Boards, HU, trazabilidad de sprint y presentación.
4. Ambientes: `dev`, `staging-demo`.
5. Gestión de secretos por variable segura en Azure/GitHub.
6. Datos mixtos con política: desarrollo sintético, demos con datos anonimizados/controlados.

## Riesgos y mitigación

1. Riesgo multi-cloud (Azure + Gemini): encapsular proveedor IA mediante `IAProviderAdapter`.
2. Riesgo REThUS automático: mantener validación semiautomática trazable por admin.
3. Riesgo de sobrealcance: congelar Must al cierre de Sprint 1 y mover extras a Should/Could.
4. Riesgo de performance en tiempo real: pruebas de carga desde Sprint 3 en cada release candidate.
5. Riesgo legal/comunicación clínica: guardrails estrictos, disclaimers y auditoría de prompts/respuestas.

## Supuestos y defaults explícitos

1. Se planifica en formato **10 sprints totales (Sprint 0 + Sprint 1-9)**.
2. Equipo de **4 integrantes** con distribución sugerida: móvil, backend/IA, web/admin, QA-DevOps rotativo.
3. Se acepta arquitectura **multi-cloud** por uso de Gemini junto con infraestructura Azure institucional.
4. MongoDB se mantiene como base principal; si analítica exige optimización, se agrega capa de agregación sin migrar dominio principal.
5. No se usarán datos clínicos personales sin anonimización y trazabilidad documental.
6. Si Gemini no está disponible institucionalmente, fallback operativo: proveedor LLM compatible mediante `IAProviderAdapter` sin rediseño de dominio.
