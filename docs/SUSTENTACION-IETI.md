# Guion de Sustentación Técnica — SaludDeUna

## IETI Grupo 1 · Jueves 21 de mayo 2026 · 11:00 a.m

> **Cómo usar este documento:** Cada sección tiene el texto para hablar en voz alta (en cursiva) y las notas técnicas de respaldo. No leer literalmente — hablar con confianza usando los puntos clave.

---

## SECCIÓN 1: Contexto General del Problema (2 minutos exactos)

### Guion de apertura

> *"En Colombia, un paciente que necesita atención médica enfrenta tres barreras simultáneas: no sabe si su urgencia realmente es urgente, no puede comunicar sus síntomas con precisión al médico, y cuando finalmente llega a consulta, invierte la mitad del tiempo explicando antecedentes que ya debería haber registrado.*
>
> *Del lado del médico, el problema espeja: recibe pacientes sin contexto estructurado, repite las mismas preguntas de anamnesis, y no tiene continuidad clínica entre consultas.*
>
> *SaludDeUna resuelve este problema con una plataforma inteligente de comunicación clínica. Usamos inteligencia artificial — específicamente Gemini 2.5-flash de Google — para guiar al paciente a través de un triage estructurado antes de la consulta: identifica red flags clínicos, clasifica la prioridad de atención, y genera un resumen médico que el doctor recibe antes de que empiece la consulta.*
>
> *Nuestros usuarios son: pacientes que necesitan orientación clínica rápida, médicos generales que quieren consultas más eficientes, y administradores clínicos que necesitan visibilidad operacional.*
>
> *El impacto esperado es una reducción del 30% en el tiempo hasta la primera respuesta médica, con al menos el 60% de las alertas de red flag confirmadas como relevantes por los doctores. Eso se traduce en menos consultas desperdiciadas y mejor continuidad clínica."*

### Datos de respaldo

- **Usuarios afectados:** Pacientes colombianos (atención ambulatoria), médicos generales, odontólogos, personal administrativo clínico
- **Especialidades implementadas:** Medicina General, Odontología, Urgencias
- **La IA no diagnostica, no prescribe** — es guía y estructuración. La decisión final es del médico (ver `wiki/02-Descripcion-Problema-Contexto.md`)
- **Restricción ética explícita** en el código: `GuardrailService` valida que el output de Gemini no contenga diagnósticos directos

---

## SECCIÓN 2: Criterios de Calidad (≥ 3 requeridos — tenemos 5)

### Criterio 1: Seguridad

> *"El primero es seguridad. Trabajamos con datos médicos sensibles, por lo que implementamos defensa en profundidad. El backend tiene autenticación JWT requerida en todos los endpoints por defecto — para desactivarla en un endpoint hay que usar el decorador explícito `@Public()`. Sobre eso, tenemos autorización basada en roles con tres roles: PATIENT, DOCTOR y ADMIN. Y encima de eso, un guard específico para doctores: `DoctorVerifiedGuard` que verifica que el doctor esté validado por REThUS antes de acceder a funcionalidades clínicas. El rate limiting es distribuido en Redis — 20 requests por 60 segundos por cliente — por lo que funciona correctamente con múltiples instancias del backend."*

**Archivos de evidencia:**

- [salud-de-una-backend/src/common/guards/jwt-auth.guard.ts](../salud-de-una-backend/src/common/guards/jwt-auth.guard.ts) — JWT requerido globalmente
- [salud-de-una-backend/src/common/guards/roles.guard.ts](../salud-de-una-backend/src/common/guards/roles.guard.ts) — RBAC (PATIENT/DOCTOR/ADMIN)
- [salud-de-una-backend/src/common/guards/doctor-verified.guard.ts](../salud-de-una-backend/src/common/guards/doctor-verified.guard.ts) — estado de verificación REThUS
- [salud-de-una-backend/src/redis/redis-throttler.storage.ts](../salud-de-una-backend/src/redis/redis-throttler.storage.ts) — throttling distribuido con Redis
- [salud-de-una-backend/src/app.module.ts:109-116](../salud-de-una-backend/src/app.module.ts) — guards globales en el módulo raíz

---

### Criterio 2: Observabilidad

> *"El segundo es observabilidad. Tenemos tres capas: primero, trazas distribuidas con OpenTelemetry — el SDK inicializa con auto-instrumentación de Node.js y exporta a cualquier colector OTLP compatible con Jaeger o Grafana Tempo. Segundo, logs JSON estructurados con correlation ID: cada request genera un UUID que viaja en todos los logs relacionados, lo que permite reconstruir el ciclo de vida completo de un request en producción. Tercero, un dashboard de negocio con 4 KPIs reales calculados desde la base de datos: tiempo promedio de respuesta médica, tasa de red flags confirmados, retención en seguimiento post-consulta, y utilidad del resumen clínico según feedback del doctor."*

**Archivos de evidencia:**

- [salud-de-una-backend/src/observability/telemetry.ts](../salud-de-una-backend/src/observability/telemetry.ts) — OpenTelemetry NodeSDK
- [salud-de-una-backend/src/common/interceptors/request-logging.interceptor.ts](../salud-de-una-backend/src/common/interceptors/request-logging.interceptor.ts) — correlation ID propagation
- [salud-de-una-backend/src/common/logging/structured-json.logger.ts](../salud-de-una-backend/src/common/logging/structured-json.logger.ts) — JSON logger
- [salud-de-una-backend/src/dashboard/](../salud-de-una-backend/src/dashboard/) — 4 KPIs de negocio
- **Diagrama:** `diagrams/sad_observability_stack.svg`

---

### Criterio 3: Mantenibilidad y Modularidad

> *"El tercero es mantenibilidad. El backend tiene 17 módulos NestJS independientes con inyección de dependencia explícita. Cada módulo encapsula su propio controller, service, schemas de Mongoose y DTOs. Los módulos opcionales como el servicio de IA y el de RAG están registrados con el decorador `@Optional()` — eso significa que el sistema funciona correctamente aunque no estén disponibles, aplicando degradación graceful. Este diseño nos permitió desarrollar billing, triage, chat y followup en paralelo sin interferencia entre equipos."*

**Archivos de evidencia:**

- [salud-de-una-backend/src/app.module.ts](../salud-de-una-backend/src/app.module.ts) — 17 módulos registrados (AiModule, AuthModule, PatientsModule, DoctorsModule, AdminModule, NotificationsModule, FollowupsModule, DashboardModule, ChatModule, ConsultationsModule, TriageModule, AdminsModule, OutboxModule, BillingModule, KnowledgeModule, RagModule, RedisModule)
- [salud-de-una-backend/src/triage/triage.service.ts](../salud-de-una-backend/src/triage/triage.service.ts) — `@Optional() private geminiTriageService?`
- **Diagrama:** `diagrams/sad_backend_modules_diagram.svg`

---

### Criterio 4: Testing

> *"El cuarto es testing. El backend tiene 406 tests pasando con una cobertura del 93% de statements y 80% de branches — superando el umbral mínimo del 80% que tenemos configurado. Y lo más importante: los tests E2E no mockean la base de datos. Usamos `mongodb-memory-server` que levanta una instancia real de MongoDB en memoria para cada suite de tests. Eso nos protege de la divergencia entre el mock y el comportamiento real — una lección aprendida de proyectos anteriores donde los mocks pasaban pero la migración de producción fallaba."*

**Archivos de evidencia:**

- [salud-de-una-backend/test/e2e/](../salud-de-una-backend/test/e2e/) — 18 suites E2E organizadas por módulo
- [salud-de-una-backend/jest.config.js](../salud-de-una-backend/jest.config.js) — threshold 80% enforced
- [salud-de-una-backend/test/e2e/support/](../salud-de-una-backend/test/e2e/support/) — harness, builders, contracts reutilizables
- CI/CD: [.github/workflows/backend-ci.yml](../.github/workflows/backend-ci.yml) — pipeline que falla si tests fallan

---

### Criterio 5: Resiliencia / Disponibilidad

> *"El quinto es resiliencia. El flujo de triage funciona incluso cuando Gemini no está disponible: si el servicio de IA falla, el sistema cae a un modo basado en reglas con el `RedFlagsEngine`, que evalúa catálogos clínicos estáticos por especialidad. El response incluye un campo `analysisMode` que indica si fue `AI_ASSISTED` o `RULE_BASED`, y un `noticeCode` que explica por qué usó el fallback. El chat en tiempo real usa Socket.IO con Redis adapter, lo que garantiza que los mensajes se distribuyen correctamente entre múltiples instancias del backend — nada se pierde si el paciente y el doctor están conectados a pods distintos."*

**Archivos de evidencia:**

- [salud-de-una-backend/src/triage/services/gemini-triage.service.ts](../salud-de-una-backend/src/triage/services/gemini-triage.service.ts) — fallback a rule-based
- [salud-de-una-backend/src/triage/engines/red-flags.engine.ts](../salud-de-una-backend/src/triage/engines/red-flags.engine.ts) — motor de reglas clínicas
- [salud-de-una-backend/src/chat/redis-io.adapter.ts](../salud-de-una-backend/src/chat/redis-io.adapter.ts) — Redis adapter para Socket.IO
- **Diagrama:** `diagrams/sad_c4_container_view.svg`

---

## SECCIÓN 3: Casos de Uso

### Guion del flujo principal

> *"El flujo principal de negocio tiene 6 pasos. El paciente descarga la app móvil, se registra y hace un triage: selecciona especialidad, responde preguntas estructuradas generadas dinámicamente por especialidad, y Gemini analiza el contexto clínico, detecta red flags y clasifica la prioridad como LOW, MODERATE o HIGH. Ese resultado crea automáticamente una consulta en la cola de atención.*
>
> *El doctor en la plataforma web ve la cola, ve el resumen clínico del triage antes de abrir la consulta, la asigna, y se abre una sala de chat en tiempo real vía Socket.IO. Al cerrar la consulta, el sistema crea automáticamente dos eventos de seguimiento post-consulta a 72 horas y 7 días. Si el paciente reporta empeoramiento significativo en el followup, el sistema crea automáticamente una nueva consulta con prioridad alta.*
>
> *Toda la facturación está implementada: hay precios por especialidad en COP configurables por el admin, el paciente paga al confirmar la consulta, y el admin tiene un dashboard de revenue con métricas mensuales y por especialidad."*

### Tabla de casos de uso implementados

| Caso de Uso | Mobile (paciente) | Web (doctor/admin) | Backend endpoint |
|-------------|-------------------|--------------------|------------------|
| Registro / Login | `app/(auth)/login.tsx` | `app/(auth)/login.tsx` | `POST /v1/auth/register/patient` |
| Triage con IA | `app/triage/[sessionId]/questions.tsx` | — | `POST /v1/triage/sessions` → `PATCH .../answers` → `PATCH .../complete` |
| Cola de consultas | `app/(tabs)/consultations.tsx` | `app/(dashboard)/doctor/queue` | `GET /v1/consultations` |
| Chat en tiempo real | `app/consultations/[id]/chat.tsx` | `app/(dashboard)/doctor/[id]` | Socket.IO gateway |
| Seguimiento post-consulta | `app/followup/[id]/timeline.tsx` | — | `POST /v1/followups/:id/submit` |
| Dashboard admin | — | `app/(dashboard)/admin/dashboard` | `GET /v1/dashboard/business` |
| Verificación doctores | — | `app/(dashboard)/admin/doctors` | `PATCH /v1/admin/doctors/:id/verify` |
| Facturación | — | `app/(dashboard)/admin/billing` | `GET/PATCH /v1/billing/admin/prices` |

---

## SECCIÓN 4: Diagrama de Arquitectura

### Secuencia para mostrar los diagramas (abrir los SVGs en el browser)

**Paso 1 — Contexto del sistema** (`sad_c4_context_diagram.svg`):
> *"Empezamos con el nivel más alto: contexto del sistema. Los tres actores externos son el paciente en mobile, el doctor y admin en web, y los sistemas externos: MongoDB Atlas para persistencia, Redis Cloud para estado distribuido y cache, Gemini AI para procesamiento clínico y Auth0 para identidad."*

**Paso 2 — Contenedores** (`sad_c4_container_view.svg`):
> *"Un nivel más abajo, vemos los contenedores: la app Expo/React Native patient-only, el portal Next.js staff-only, y el backend NestJS. La comunicación es HTTPS para REST y WebSocket para el chat. Importante: hay un Application Load Balancer frente al backend que maneja sticky sessions para WebSocket — sin eso, Socket.IO se rompe en múltiples instancias."*

**Paso 3 — Infraestructura de despliegue** (`sad_aws_deployment_diagram.svg`):
> *"El despliegue está en AWS con ECS Fargate SPOT, que nos da contenedores sin gestionar servidores y con SPOT reducimos costos en ~70%. El ALB enruta `/v1/*` y `/socket.io/*` al backend y todo lo demás al web. El Dockerfile es multi-stage: builder con Alpine y runner sin dependencias de desarrollo."*

**Paso 4 — Pipeline de IA** (`sad_rag_pipeline_diagram.svg`):
> *"El pipeline de IA del triage: el paciente responde preguntas, el `RedFlagsEngine` evalúa reglas deterministas primero — sin latencia de IA — y si detecta síntomas críticos como dolor torácico con disnea, eleva la prioridad a HIGH inmediatamente. Luego `GeminiTriageService` envía el contexto clínico estructurado a Gemini 2.5-flash con los prompts versionados almacenados en MongoDB. Si el RAG está activo, recupera chunks relevantes del knowledge base con vector search antes de la llamada a Gemini para enriquecer el contexto."*

**Paso 5 — Módulos backend** (`sad_backend_modules_diagram.svg`):
> *"El backend tiene 17 módulos organizados por dominio. Cada módulo es independiente con inyección de dependencia explícita. Los módulos de infraestructura como Redis son compartidos. Destacamos el `OutboxModule` que implementa el patrón Transactional Outbox: los domain events como CONSULTATION_CLOSED_EVENT se persisten en MongoDB primero y luego se despachan de forma asíncrona, garantizando que el followup se crea incluso si el proceso falla a mitad."*

### Patrones arquitectónicos a mencionar

| Patrón | Módulo | Por qué se usó |
|--------|--------|---------------|
| Transactional Outbox | `outbox/` | Garantiza consistencia entre cierre de consulta y creación de followup |
| Repository | `triage/questions/` | Aísla acceso a datos del catálogo de preguntas |
| Strategy | `auth/strategies/` | jwt, jwt-legacy, jwt-provision — intercambiables sin cambiar el controller |
| Dual Store | `dashboard/metrics/` | In-memory para acceso rápido, Redis como fuente de verdad |
| Circuit Breaker (soft) | `triage/services/` | `@Optional()` + fallback a rule-based si Gemini no responde |

---

## SECCIÓN 5: Script de Demo en Tiempo Real

> **IMPORTANTE:** QR del APK visible desde el minuto 0.  
> URL EAS Build: `https://expo.dev/accounts/saluddeuna/projects/salud-de-una-mobile/builds`  
> Backend producción: `https://salud-de-una-backend-production.up.railway.app/v1`  
> Web producción: `[URL-VERCEL]` ← completar con URL real de Vercel

### Credenciales de demo (preparar esta noche)

| Rol | Email | Contraseña |
|-----|-------|------------|
| Admin | `admin.demo@saluddeuna.com` | `Adm1n.D3m0!` |
| Doctor (verificado) | `doctor.demo@saluddeuna.com` | `D0ct0r.D3m0!` |
| Paciente | `paciente.demo@saluddeuna.com` | `P4c1ent3.D3m0!` |

### Flujo de demo (total: ~16 minutos)

**T+00 — Proyectar QR del APK**

- Mostrar el QR del EAS Build en pantalla
- Decir: *"Este es el QR para descargar el APK del build de producción. Está instalado aquí en el dispositivo físico."*
- Abrir la app en el dispositivo Android

**T+01 — Admin Web: Dashboard**

- Browser Tab 1: Abrir `[URL-VERCEL]`
- Login como `admin.demo@saluddeuna.com`
- Mostrar dashboard: KPIs de negocio (consultas activas, revenue total, doctores activos)
- Decir: *"El admin tiene visibilidad completa de la operación en tiempo real."*

**T+03 — Admin Web: Verificar doctor**

- Navegar a `/admin/doctors`
- Mostrar lista de doctores con estados (PENDING, VERIFIED)
- Si hay uno PENDING: hacer click en "Verificar" — mostrar el flujo
- Decir: *"El admin valida los doctores revisando sus credenciales REThUS manualmente. En producción esto se integraría con la API pública del Ministerio de Salud."*

**T+05 — Mobile: Login paciente**

- Dispositivo Android: abrir SaludDeUna
- Login con `paciente.demo@saluddeuna.com`
- Mostrar home: próximas consultas, acceso rápido a triage

**T+06 — Mobile: Iniciar triage**

- Click en "Nueva consulta" → seleccionar "Medicina General"
- Decir: *"El triage empieza con preguntas estructuradas generadas dinámicamente por especialidad."*
- Responder 4-5 preguntas (tener respuestas preparadas con síntomas moderados)
- En la pregunta sobre fiebre: responder "38.5°C, hace 3 días"
- Completar triage

**T+09 — Mobile: Ver resultado de IA**

- Mostrar pantalla de resultado: prioridad MODERATE, resumen clínico en español
- Mostrar que el `analysisMode` dice `AI_ASSISTED`
- Decir: *"Gemini 2.5-flash analizó el contexto clínico, identificó que la fiebre sostenida por 3 días merece atención en 24 horas, y generó este resumen para el doctor. La IA no diagnostica — orienta y prioriza."*
- La consulta ya está creada en la cola

**T+11 — Doctor Web: Cola de consultas**

- Browser Tab 2: Abrir `/doctor/queue`
- Mostrar que la consulta del paciente aparece con el resumen de triage
- Decir: *"El doctor ve el resumen antes de abrir la consulta. Ya llega con contexto."*
- Asignar la consulta

**T+12 — Chat en tiempo real**

- Doctor web escribe: "Hola, veo que llevas 3 días con fiebre. ¿Has tomado algún medicamento?"
- Mostrar que aparece en la app mobile en tiempo real (sin recargar)
- Responder desde mobile
- Decir: *"El chat usa Socket.IO con Redis adapter. Funciona aunque el backend esté en múltiples instancias porque Redis actúa como bus de mensajes."*

**T+14 — Cerrar consulta + Billing**

- Doctor: cerrar consulta
- Mostrar que se genera el cobro automáticamente
- Admin: abrir `/admin/billing` → mostrar la transacción nueva
- Mostrar revenue metrics por especialidad

**T+15 — Mostrar followup creado**

- En Mobile: navegar a notificaciones
- Decir: *"Al cerrar la consulta, el sistema crea automáticamente dos eventos de seguimiento: a 72 horas y a 7 días. Si el paciente reporta empeoramiento, se crea una nueva consulta de alta prioridad automáticamente."*

**T+16 — FIN DEMO**

- Mostrar brevemente el Swagger: `[BACKEND-URL]/v1/docs` (si está accesible)
- O mostrar el dashboard de métricas técnicas

---

## SECCIÓN 6: Preguntas Difíciles — Q&A Preparado

### P1: "¿Cómo garantizan que la IA no da diagnósticos incorrectos que ponen en riesgo al paciente?"

**Respuesta:** *"Tenemos tres capas de protección. Primero, el `GuardrailService` en `src/triage/services/guardrail.service.ts` aplica reglas regex contra el output de Gemini antes de enviarlo al paciente — detecta patrones como 'tienes X enfermedad' y los bloquea. Segundo, el prompt de Gemini instruye explícitamente que la IA NO diagnostica ni prescribe — solo orienta y prioriza. Tercero, el disclaimer legal es visible en la pantalla de resultado. La decisión clínica final siempre es del médico."*

---

### P2: "¿Por qué NestJS y no Spring Boot que es más maduro para healthcare?"

**Respuesta:** *"Para nuestro equipo y el tiempo disponible, TypeScript end-to-end fue la decisión correcta. Tenemos tipos compartidos desde el backend OpenAPI hasta ambos frontends con generación automática vía `openapi-typescript` — eso reduce bugs de contrato en tiempo de compilación, no en producción. En una plataforma clínica real escalaríamos el core crítico a Java, pero para un MVP académico con 4 servicios y 36 semanas, la productividad de TypeScript unificado fue superior."*

---

### P3: "¿Cómo escala el sistema si tienen 10,000 pacientes simultáneos?"

**Respuesta:** *"La arquitectura es stateless por diseño. El estado de sesión está en Redis y MongoDB, no en memoria del proceso. El chat usa Redis pub/sub con el `redis-io.adapter` — múltiples instancias del backend sincronizadas. En ECS Fargate configuramos auto-scaling basado en CPU. El throttling distribuido en Redis evita thundering herd. Gemini y MongoDB Atlas escalan independientemente como servicios externos. Para 10,000 usuarios necesitaríamos al menos 4-6 instancias del backend y MongoDB Atlas M10+."*

---

### P4: "¿Por qué MongoDB y no PostgreSQL para datos médicos sensibles?"

**Respuesta:** *"Los registros clínicos son inherentemente polimórficos: una consulta de odontología tiene campos completamente distintos a una de urgencias. MongoDB con schemas flexibles nos evita 40+ tablas join en SQL. Para las métricas financieras de billing usamos aggregation pipelines que son suficientemente potentes. Si escalamos a producción real, consideraríamos un modelo híbrido: MongoDB para datos clínicos y PostgreSQL para transacciones financieras auditables."*

---

### P5: "¿La integración con REThUS es real?"

**Respuesta:** *"Implementamos el flujo completo de verificación con todos los estados: `PENDING`, `VERIFIED`, `REJECTED`, y el proceso de revisión por admin. La integración automatizada con la API del RETHUS del Ministerio de Salud requiere credenciales institucionales que no están disponibles en contexto académico, por lo que el admin verifica manualmente revisando la documentación del doctor. El código está preparado para reemplazar esa verificación manual con una llamada HTTP cuando se tengan las credenciales."*

---

### P6: "¿Qué pasa si Gemini falla durante un triage activo?"

**Respuesta:** *"El `GeminiTriageService` está registrado con `@Optional()` en el módulo de triage. Si falla o no responde, el sistema usa automáticamente el `RedFlagsEngine` con sus catálogos de reglas clínicas estáticas. El response incluye `analysisMode: 'RULE_BASED'` y un `noticeCode` que explica por qué usó el fallback. El paciente obtiene su clasificación de prioridad de todas formas — solo pierde el análisis semántico de Gemini. También tenemos retry con backoff: 150ms y 400ms antes de declarar fallback."*

---

### P7: "¿Cuánto cuesta la infraestructura en producción?"

**Respuesta:** *"Aproximadamente $26-31 USD/mes usando ECS Fargate SPOT en AWS. SPOT nos reduce el costo en ~70% comparado con On-Demand. El desglose: ECS (backend + web) ~$15/mes, ALB ~$7/mes, ECR almacenamiento ~$1/mes, MongoDB Atlas M0 gratuito (límite 512MB, suficiente para el MVP), Redis Cloud gratuito. Para el plan de producción con usuarios reales estimamos $60-80/mes con MongoDB M10 y Redis plan de pago."*

---

### P8: "¿Cómo manejan la privacidad de los datos de salud bajo la Ley 1581?"

**Respuesta:** *"Implementamos el principio de mínimo privilegio: el paciente solo ve sus datos, el doctor solo los de sus pacientes asignados, el admin tiene acceso completo pero auditable. Los tokens JWT expiran en 1 hora. Las contraseñas usan bcrypt con salt. Los datos van cifrados en tránsito por TLS. Para producción completa se requiere política de privacidad formal, consentimiento de habeas data en el onboarding, y cifrado at-rest en MongoDB Atlas disponible desde el plan M10."*

---

### P9: "¿Por qué el lineamiento de IA en proceso de desarrollo si IETI no lo requiere?"

*[Solo si el profesor lo pregunta para IETI]*

**Respuesta:** *"Usamos Claude Code — la CLI de Anthropic — durante todo el desarrollo. El `CLAUDE.md` en la raíz del repositorio es el 'prompt de contexto persistente' que manteníamos actualizado para que el agente entendiera la arquitectura completa del monorepo. Esto nos permitió generar módulos completos con tests E2E, refactorizar código manteniendo coherencia entre los 4 servicios, y mantener 406 tests pasando con 93% de cobertura en un proyecto de esta escala durante el semestre."*

---

### P10: "¿El chat funciona si el backend tiene más de una instancia?"

**Respuesta:** *"Sí, por el Redis adapter. `src/chat/redis-io.adapter.ts` implementa `createAdapter` de `@socket.io/redis-adapter`. Cuando el doctor en la instancia A envía un mensaje, Socket.IO publica en el canal de Redis. La instancia B que tiene conectado al paciente recibe el evento del canal y lo reenvía al WebSocket correspondiente. Sin este adapter, si paciente y doctor están en instancias distintas, el chat sería silencioso — un bug que detectamos en producción de proyectos similares."*

---

### P11: "¿Por qué Expo y no Flutter o nativo para mobile?"

**Respuesta:** *"Tres razones: primero, TypeScript compartido con el backend y la web — menos cambios de contexto para el equipo. Segundo, Expo SDK 55 con React Native New Architecture y React Compiler activos, lo que nos da rendimiento comparable a nativo para nuestro caso de uso. Tercero, EAS Build que genera APK/IPA sin necesitar macOS ni Android Studio configurados localmente — fundamental para agilidad en un proyecto académico."*

---

### P12: "¿Qué significa el Transactional Outbox pattern y por qué lo usan?"

**Respuesta:** *"El problema es este: cuando el doctor cierra una consulta, necesitamos hacer dos cosas: marcar la consulta como CLOSED en MongoDB y crear dos Followup documents programados. Si hacemos las dos operaciones independientes y el proceso falla entre medio, la consulta queda cerrada pero sin followup — inconsistencia de datos médicos. El Transactional Outbox resuelve esto: primero guardamos el event `CONSULTATION_CLOSED_EVENT` en la misma transacción MongoDB que cierra la consulta. Luego un dispatcher asíncrono procesa el event y crea los followups. Si el dispatcher falla, el event sigue ahí para reintentar. Garantía de exactly-once processing."*

---

### P13: "¿Cuántas horas de desarrollo tiene el proyecto?"

**Respuesta:** *"Estimamos entre 400 y 600 horas distribuidas en el semestre. Usamos Claude Code como multiplicador de productividad: generó el boilerplate de módulos NestJS, los tests E2E, y parte de la documentación técnica. Pero las decisiones arquitectónicas, el diseño del flujo de triage, los criterios de calidad y la integración de Gemini fueron diseño y revisión del equipo. La IA fue un acelerador, no un sustituto."*

---

### P14: "¿Qué harían diferente si lo rehacieran desde cero?"

**Respuesta:** *"Dos cosas. Primero, habríamos configurado el pipeline de CI/CD para deploy automático desde el día 1 en lugar de hacerlo al final — el tiempo de integración tardía fue costoso. Segundo, habríamos validado Auth0 en el dispositivo móvil mucho antes — las incompatibilidades del flujo PKCE en React Native solo se manifiestan en un APK real, no en el emulador, y descubrirlo tarde genera deuda técnica de estrés."*

---

### P15: "¿Por qué no hay videos en tiempo real entre médico y paciente?"

**Respuesta:** *"Está explícitamente en el No Alcance del MVP, documentado en `wiki/02-Descripcion-Problema-Contexto.md`. La razón es que WebRTC tiene complejidad significativa de infraestructura (servidores TURN/STUN, codec negotiation, mobile constraints) que excede el tiempo disponible. La decisión fue: hacer el chat de texto excepcionalmente bien — con persistencia, historial, entrega garantizada via Redis — antes de añadir video. El sistema está preparado para añadirlo: el canal Socket.IO ya existe, solo faltaría agregar el signaling de WebRTC."*

---

## SECCIÓN 7: Checklist 30 Minutos Antes

### En el salón (10:30 a.m.)

- [ ] **QR del APK proyectado** y visible desde cualquier ángulo del salón
  - URL: `https://expo.dev/accounts/saluddeuna/projects/salud-de-una-mobile/builds`
- [ ] **APK en el dispositivo Android físico** — login con `paciente.demo@saluddeuna.com` hecho
- [ ] **Browser Tab 1:** Web admin `[URL-VERCEL]` — login como admin hecho
- [ ] **Browser Tab 2:** Web doctor — login como doctor hecho (en incógnito)
- [ ] **Browser Tab 3:** Swagger en `https://salud-de-una-backend-production.up.railway.app/v1/docs` (si aplica)
- [ ] **Health check manual:** `https://salud-de-una-backend-production.up.railway.app/v1` → debe responder
- [ ] **Consulta de demo pre-cargada** en la cola del doctor (ejecutar seed-demo.ts la noche anterior)
- [ ] **Diagramas SVG** abiertos en tabs del browser listos para cambiar:
  - `sad_c4_context_diagram.svg`
  - `sad_c4_container_view.svg`
  - `sad_aws_deployment_diagram.svg`
  - `sad_rag_pipeline_diagram.svg`
  - `sad_backend_modules_diagram.svg`
- [ ] **Backup offline:** carpeta con screenshots de cada pantalla lista en caso de fallo de red
- [ ] **Cronómetro** preparado para controlar los 2 minutos del contexto
- [ ] **Credenciales en papel** — no depender de memoria bajo estrés

### Si falla la red del salón (Plan B)

1. Usar hotspot del celular como WiFi
2. Usar screenshots de backup para mostrar flujos
3. Mostrar código directamente en VS Code para criterios de calidad
4. El diagrama de arquitectura se puede mostrar offline desde los SVGs locales

---

## URLs de Referencia Rápida

| Recurso | URL |
|---------|-----|
| Backend producción (health) | `https://salud-de-una-backend-production.up.railway.app/v1` |
| Web producción | `[URL-VERCEL]` ← completar |
| Swagger (si habilitado) | `https://salud-de-una-backend-production.up.railway.app/v1/docs` |
| EAS Build (QR APK) | `https://expo.dev/accounts/saluddeuna/projects/salud-de-una-mobile/builds` |
| Auth0 tenant | `salud-de-una.us.auth0.com` |
| GitHub repo | `https://github.com/JesusJC15/salud-de-una-wiki` |

---

*Documento generado: 2026-05-19 | Versión: 1.0 para IETI Grupo 1*
