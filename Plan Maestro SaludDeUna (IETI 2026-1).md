# Plan Maestro SaludDeUna (IETI 2026-1)

## Resumen
Este plan cubre **Sprint 0 + 9 sprints de ejecucion** (10 en total), alineado con `Project-Definition IETI-2026` y con `lineamientos_proyecto_innovacion_ia_movil.md`.
Objetivo del semestre: entregar un MVP funcional de comunicacion clinica inteligente para **Medicina General y Odontologia**, con **IA (RAG + LLM + reglas clinicas)**, **chat en tiempo real**, observabilidad completa y trazabilidad Scrum en Azure DevOps (Wiki/HU) con codigo en GitHub.

Fechas criticas:
1. Inception: **25 de febrero de 2026 a 4 de marzo de 2026**.
2. Presentacion Inception: **9 de marzo de 2026**.

## Equivalencia de sprints con lineamiento 2026-1
Se conserva la convencion interna `Sprint 0..9` para continuidad del equipo, con mapeo oficial:

| Esquema interno | Lineamiento |
|---|---|
| Sprint 0 (Inception) | Inception |
| Sprint 1 | Sprint 1 |
| Sprint 2 | Sprint 2 |
| Sprint 3 | Sprint 3 |
| Sprint 4 | Sprint 4 |
| Sprint 5 | Sprint 5 |
| Sprint 6 | Sprint 6 |
| Sprint 7 | Sprint 7 |
| Sprint 8 | Sprint 8 |
| Sprint 9 | Sprint 10 |

## Alcance cerrado

### In Scope (MVP semestre)
1. App paciente en React Native.
2. Panel medico/admin web en React + Next.
3. Backend NestJS + MongoDB.
4. IA con Gemini + RAG + reglas clinicas de seguridad.
5. Triage predictivo no diagnostico con prioridad `LOW|MODERATE|HIGH`.
6. Red flags por especialidad (Medicina General y Odontologia).
7. Resumen clinico automatico preconsulta.
8. Chat clinico asincrono con actualizaciones en tiempo real (WebSocket).
9. Seguimiento post-consulta y evolucion del caso.
10. Flujo de monetizacion simulado.
11. Validacion semiautomatica REThUS por admin.
12. Observabilidad: logs estructurados, metricas tecnicas, dashboard y 4 KPIs de negocio.

### Out of Scope
1. Diagnostico o prescripcion automatica por IA.
2. Videoconsulta en vivo.
3. Integracion real con pasarela de pagos productiva.
4. Integracion oficial directa certificada con sistemas hospitalarios/HCE productivos.

## Cambios y adiciones de interfaces publicas

### Canales cliente
1. `PacienteApp` (React Native): registro, triage, chat, seguimiento, historial.
2. `DoctorAdminWeb` (React + Next): cola clinica, chat medico, validacion REThUS, dashboards.

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
11. `PATCH /v1/consultations/{id}/attend`
12. `PATCH /v1/consultations/{id}/close`
13. `POST /v1/consultations/{id}/summary/generate`
14. `PATCH /v1/consultations/{id}/summary/feedback`
15. `POST /v1/consultations/{id}/translation`
16. `POST /v1/followups`
17. `POST /v1/patients/medical-history`
18. `GET /v1/patients/{id}/timeline`
19. `POST /v1/patients/dependents`
20. `GET /v1/patients/dependents`
21. `GET /v1/dashboard/technical`
22. `GET /v1/dashboard/business`
23. `POST /v1/billing/simulate-checkout`
24. `GET /v1/knowledge/articles`
25. `POST /v1/knowledge/articles/{id}/approve`

### WebSocket events (`namespace: consultation`)
1. `consultation.message.created`
2. `consultation.status.changed`
3. `consultation.priority.updated`
4. `consultation.queue.updated`
5. `consultation.followup.reminder.triggered`

### Tipos/contratos clave
1. `PriorityLevel = LOW | MODERATE | HIGH`
2. `Specialty = GENERAL_MEDICINE | DENTISTRY`
3. `TranslationDirection = TO_PATIENT | TO_CLINIC`
4. `RedFlag { code, specialty, severity, evidence }`
5. `ClinicalSummary { chiefComplaint, duration, intensity, associatedSymptoms, meds, history, priority, redFlags }`
6. `SummaryFeedback { useful, comments? }`
7. `RethusVerification { doctorId, status, checkedBy, checkedAt, evidenceUrl }`
8. `KpiSnapshot { periodStart, periodEnd, metricName, value, target, status }`

## KPIs y SLOs cerrados

### KPIs de negocio (obligatorios)
1. Tiempo a primera respuesta medica.
2. Calidad/uso del resumen clinico (porcentaje de casos donde el medico lo marca util).
3. Tasa de red flags relevantes confirmadas por medico.
4. Retencion de seguimiento a 7 dias.

### SLO tecnico (nivel acordado)
1. P95 latencia chat `< 1.5 s`.
2. Tiempo de generacion de resumen IA `< 15 s`.
3. Latencia interactiva (lineamiento) `< 2 s` en operaciones de usuario.
4. Disponibilidad mensual `> 99.0%`.

## Plan por sprint (decision complete)
1. **Sprint 0 (Inception, hasta 4 marzo 2026):** wiki completa en Azure DevOps con titulo, integrantes, resumen, problema, Canvas, arquitectura base, epicas/features, HU INVEST con Gherkin, DoR/DoD, MoSCoW, story map, plan de observabilidad, 2 HU principales con IA, guion de presentacion del 9 marzo.
2. **Sprint 1:** base tecnica y repositorios; monorepo; auth paciente/medico; RBAC; estructura Mongo; skeleton RN y Web; CI GitHub Actions; despliegue inicial en Azure.
3. **Sprint 2:** modulo triage para Medicina General; cuestionario estructurado; motor de reglas red flags v1; clasificacion de prioridad; registro de auditoria IA.
4. **Sprint 3:** modulo triage para Odontologia; ajustes de reglas por especialidad; cola clinica por prioridad para panel medico; pruebas de concurrencia base.
5. **Sprint 4:** generacion de resumen clinico automatico con Gemini + RAG; guardrails (no diagnostico/no prescripcion); traduccion paciente-clinico y clinico-paciente.
6. **Sprint 5:** chat clinico real-time con WebSocket; estados de consulta; persistencia de mensajes; notificaciones internas; hardening de seguridad de sesion.
7. **Sprint 6:** seguimiento post-consulta automatizado; timeline evolutivo del paciente; indicadores de cambio de sintomas; recordatorios.
8. **Sprint 7:** panel medico/admin avanzado; dashboard tecnico y de negocio; validacion semiautomatica REThUS; gestion de banco de conocimiento validado.
9. **Sprint 8:** monetizacion simulada (planes/pago por consulta); analitica de conversion simulada; optimizacion de performance para cumplir SLO.
10. **Sprint 9 (hardening y cierre):** pruebas E2E completas, prueba de carga y concurrencia, correcciones, documentacion final, demo final, retrospectiva y backlog postcurso.

## MoSCoW del producto

### Must
1. Registro/login paciente y medico con RBAC.
2. Verificacion semiautomatica REThUS.
3. Triage IA + red flags + prioridad en 2 especialidades.
4. Resumen clinico automatico.
5. Chat real-time y cola clinica.
6. Seguimiento post-consulta y timeline.
7. Logs estructurados + metricas + dashboard.
8. 4 KPIs de negocio implementados y medibles.
9. Flujo de monetizacion simulado.

### Should
1. Banco de conocimiento validado con aprobaciones medicas.
2. Traduccion bidireccional de lenguaje clinico.
3. Alertas inteligentes de seguimiento.

### Could
1. Recomendaciones de contenido preventivo personalizadas.
2. Versionado avanzado de prompts y evaluacion automatica offline.

### Won't (este semestre)
1. Videollamadas.
2. Prescripcion/diagnostico automatico.
3. Integraciones clinicas productivas certificadas.
4. Pasarela real de pagos productiva.

## Pruebas y criterios de aceptacion

### Pruebas obligatorias
1. Unitarias para reglas de prioridad y red flags.
2. Integracion API para flujos de consulta, resumen, seguimiento y REThUS admin.
3. E2E de paciente y medico (alta prioridad, moderada, baja).
4. Carga/concurrencia para chat y cola de casos.
5. Seguridad basica: auth, autorizacion por rol, validacion de input, sanitizacion de logs.

### Escenarios de aceptacion minimos
1. Caso critico odontologico genera prioridad alta y alerta en cola medica.
2. Medico recibe resumen util antes de responder.
3. Seguimiento post-consulta detecta empeoramiento y re-prioriza.
4. Dashboard muestra metricas tecnicas y 4 KPIs de negocio con datos reales del entorno de pruebas.
5. IA rechaza solicitudes de diagnostico/prescripcion y aplica disclaimer.

## DevOps y gobernanza
1. Codigo en GitHub (monorepo) con ramas `main`, `develop`, `feature/*`.
2. Repositorio documentado: `https://github.com/JesusJC15/salud-de-una-wiki`.
3. GitHub Actions para lint, test, build y deploy por ambiente.
4. Azure DevOps para Wiki, Boards, HU, trazabilidad de sprint y presentacion.
5. Ambientes: `dev`, `staging-demo`.
6. Gestion de secretos por variable segura en Azure/GitHub.
7. Datos mixtos con politica: desarrollo sintetico, demos con datos anonimizados/controlados.

## Riesgos y mitigacion
1. Riesgo multi-cloud (Azure + Gemini): encapsular proveedor IA mediante `IAProviderAdapter`.
2. Riesgo REThUS automatico: mantener validacion semiautomatica trazable por admin.
3. Riesgo de sobrealcance: congelar Must al cierre de Sprint 1 y mover extras a Should/Could.
4. Riesgo de performance en tiempo real: pruebas de carga desde Sprint 3 en cada release candidate.
5. Riesgo legal/comunicacion clinica: guardrails estrictos, disclaimers y auditoria de prompts/respuestas.

## Supuestos y defaults explicitos
1. Se planifica en formato **10 sprints totales (Sprint 0 + Sprint 1-9)**.
2. Equipo de **4 integrantes** con distribucion sugerida: movil, backend/IA, web/admin, QA-DevOps rotativo.
3. Se acepta arquitectura **multi-cloud** por uso de Gemini junto con infraestructura Azure institucional.
4. MongoDB se mantiene como base principal; si analitica exige optimizacion, se agrega capa de agregacion sin migrar dominio principal.
5. No se usaran datos clinicos personales sin anonimización y trazabilidad documental.
6. Si Gemini no esta disponible institucionalmente, fallback operativo: proveedor LLM compatible mediante `IAProviderAdapter` sin rediseño de dominio.
