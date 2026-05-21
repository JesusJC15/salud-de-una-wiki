# SaludDeUna — Preguntas Esperadas y Respuestas Defendibles

> Banco de 30 preguntas con respuestas técnicas, claras y defensibles para la sustentación.
> Clasificadas por categoría. Estudiar las marcadas con ⚠️ — son las más probables.

---

## Categoría 1: Sobre la IA y el alcance clínico

### ⚠️ P1. ¿La IA diagnostica enfermedades?

**Respuesta:**
> No. La IA clasifica prioridad de atención — LOW, MODERATE o HIGH — y detecta red flags predefinidas. No emite diagnósticos clínicos. Hay un guardrail implementado en código (`GuardrailService`) que evalúa cada respuesta de Gemini antes de persistirla. Si la respuesta contiene lenguaje de diagnóstico, prescripción o afirmación clínica, se bloquea y el campo `aiSummary` queda en `null`. Este comportamiento está probado en los tests E2E.

---

### ⚠️ P2. ¿Qué pasa si la IA clasifica mal la prioridad?

**Respuesta:**
> El sistema tiene dos capas de seguridad. Primero, el motor de red-flags en `red-flags-mg.json` es un conjunto de reglas deterministas que se aplican independientemente de Gemini. Segundo, si Gemini falla completamente, el sistema cae al modo `RULE_BASED` automáticamente. Además, el resultado del triage es una sugerencia de prioridad — el médico siempre puede ver el caso y reasignarlo. La responsabilidad clínica final es del médico, no del sistema.

---

### P3. ¿Qué datos del paciente le envían a Google Gemini?

**Respuesta:**
> Solo las respuestas del cuestionario de triage en forma de texto estructurado — sin nombre, sin número de identificación, sin datos de contacto. Los prompts que van a Gemini están diseñados con información clínica anónima. El `AiModule` tiene trazabilidad completa vía `aiauditlogs` — cada llamada a Gemini queda registrada con modelo, tokens, latencia y resultado.

---

### P4. ¿Qué diferencia a SaludDeUna de un chat médico simple como WhatsApp con un médico?

**Respuesta:**
> Cuatro diferencias estructurales. Primero, el triage guiado: antes del chat, la IA estructura la información del paciente para que el médico llegue informado. Segundo, la priorización automática: los casos más urgentes van primero en la cola. Tercero, el resumen clínico generado por IA al final de cada consulta — no hay que tomar notas manualmente. Y cuarto, el seguimiento automatizado post-consulta con escalación si el paciente empeora. Un chat con un médico no tiene ninguna de estas capas.

---

### P5. ¿Por qué usaron Gemini y no GPT-4 o Claude?

**Respuesta:**
> Evaluamos tres factores: capacidad de razonamiento clínico, costo de API y latencia. Gemini 2.5-flash tiene un balance favorable en los tres para el volumen de nuestro MVP. Además, la API de Google AI Studio tiene un tier gratuito generoso para pruebas. La arquitectura del `AiModule` usa un patrón de estrategia — si quisiéramos cambiar a otro modelo, el cambio es en la configuración, no en el código de negocio.

---

### P6. ¿El sistema podría causar daño si le dice a un paciente que su prioridad es baja pero en realidad es urgente?

**Respuesta:**
> El sistema siempre muestra el resultado como una recomendación, no como una instrucción médica. Hay un disclaimer clínico en toda salida de IA. Además, el sistema recomienda atención presencial inmediata si detecta red flags de prioridad CRITICAL. El texto final siempre incluye: "Esta clasificación es orientativa. Consulte a un médico si tiene dudas." La responsabilidad clínica es del profesional de salud, no de la plataforma.

---

## Categoría 2: Sobre la arquitectura técnica

### ⚠️ P7. ¿Por qué usaron arquitectura de microservicios vs. monolito?

**Respuesta:**
> No usamos microservicios — usamos un monolito modular con NestJS. Esta fue una decisión consciente. Para el volumen de un MVP con un equipo de cuatro personas, un monolito modular ofrece menor overhead operacional, deploy más simple y debugging más directo que microservicios. Los módulos NestJS tienen inyección de dependencias y límites claros — si en el futuro fuera necesario extraer un módulo como servicio independiente, la separación ya está hecha en código.

---

### P8. ¿Por qué tienen dos procesos de backend (API y Worker)?

**Respuesta:**
> El Worker maneja operaciones asíncronas que no deben bloquear el ciclo request-response de la API: los jobs de followup programados con BullMQ, el despacho del outbox transaccional, y las notificaciones push. Si el Worker falla, la API sigue funcionando. La variable `APP_RUNTIME_ROLE` controla qué proceso corre en cada instancia.

---

### ⚠️ P9. ¿Cómo manejan la escalabilidad del chat en tiempo real?

**Respuesta:**
> Actualmente el chat usa Socket.IO en modo single-instance. Esto funciona perfectamente con una sola instancia de backend, que es el escenario del MVP. Para escalar horizontalmente, la solución es el `@socket.io/redis-adapter` — tenemos la arquitectura documentada y el código de implementación en el plan técnico. Redis ya está en el stack, así que el cambio es agregar el adapter, no rediseñar el sistema.

---

### P10. ¿Por qué usaron MongoDB en lugar de PostgreSQL?

**Respuesta:**
> Tres razones. Primero, los documentos clínicos tienen esquemas variables — un documento de triage de Medicina General tiene campos distintos que uno de Odontología. MongoDB permite esa flexibilidad sin migraciones complejas. Segundo, MongoDB Atlas tiene Vector Search nativo HNSW, que usamos para el RAG. Tercero, el equipo tiene más experiencia con MongoDB y Mongoose para un MVP de desarrollo rápido.

---

### P11. ¿Qué es el patrón Outbox Transaccional y por qué lo usaron?

**Respuesta:**
> El Outbox Transaccional garantiza que cuando una consulta se cierra, el evento de dominio `CONSULTATION_CLOSED` se emite exactamente una vez — aunque el proceso falle a mitad de camino. La operación de negocio y la creación del evento outbox van en la misma transacción MongoDB. Un proceso dispatcher lee y procesa los eventos pendientes con reintentos y backoff exponencial hasta 5 veces. Sin esto, si el sistema se cae después de cerrar la consulta pero antes de crear los followups, los followups se perderían.

---

### P12. ¿Qué es el RAG y por qué lo tienen en el sistema?

**Respuesta:**
> RAG significa Retrieval-Augmented Generation. En lugar de depender solo del conocimiento general de Gemini, el sistema busca en una base de conocimiento clínica propia — documentos médicos indexados con embeddings vectoriales de 768 dimensiones usando gemini-embedding-001 en MongoDB Atlas Vector Search. El resultado es que el triage y el resumen clínico pueden enriquecerse con evidencia clínica relevante específica para la especialidad. El pipeline tiene 8 pasos, cache en Redis de 300 segundos, y trazas completas en la colección `ragtraces`.

---

## Categoría 3: Sobre la calidad técnica

### ⚠️ P13. ¿Por qué tienen 406 tests y qué nivel de confianza dan esas pruebas?

**Respuesta:**
> 406 tests en el backend pasando, ninguno fallando. La cobertura es 93% de statements, 80% de branches. Los tests están en dos capas: unit tests con mocks de NestJS para módulos individuales, y 12 suites de E2E con `mongodb-memory-server` que usan una base de datos MongoDB real en memoria. Los E2E nos dieron confianza en los flujos críticos — auth, triage, billing, chat. Detectaron tres bugs reales que los unit tests no habrían encontrado.

---

### P14. ¿Qué criterios de calidad definieron y cómo los midieron?

**Respuesta:**
> Definimos seis criterios:
>
> 1. Seguridad: dual auth JWT + Auth0, RBAC, rate limiting, bcrypt, max sesiones — medido por el éxito de los tests E2E de auth.
> 2. Confiabilidad: 406 tests, 93% cobertura, umbral 80% configurado en CI.
> 3. Observabilidad: logs JSON con correlationId, métricas técnicas en `/v1/dashboard/technical`, trazas OpenTelemetry.
> 4. Mantenibilidad: NestJS modular con DI, sin acoplamiento entre dominios, código en TypeScript estricto.
> 5. Rendimiento: p95 < 1500ms como SLO, cache Redis de 300s para RAG.
> 6. Disponibilidad: endpoints `/v1/health` y `/v1/ready` con checks de MongoDB y Redis.

---

### P15. ¿Cómo aseguran que no hay secrets hardcodeados en el repositorio?

**Respuesta:**
> Los archivos `.env` con valores reales están en `.gitignore`. Solo los archivos `.env.*.example` están en el repositorio — sin valores reales. Las variables sensibles en producción se manejan como Railway env vars y Vercel env vars — nunca en el código. El CI puede fallar si detecta secrets en el código mediante el paso de validación de entorno.

---

## Categoría 4: Sobre la implementación y el equipo

### ⚠️ P16. ¿Cuántas personas y cuánto tiempo tardaron en desarrollar esto?

**Respuesta:**
> Cuatro personas, aproximadamente cinco meses, con metodología Scrum de 10 sprints (Sprint 0 de inception hasta Sprint 9). El trabajo fue distribuido: backend principal, frontend web, app mobile, y DevOps/infraestructura. Usamos GitHub para el repositorio, Azure DevOps para el backlog y los sprints, y Claude Code como asistente de desarrollo.

---

### P17. ¿Cómo coordinaron el trabajo en equipo para evitar conflictos?

**Respuesta:**
> Tres mecanismos. Primero, la separación de repositorios por servicio (backend / web / mobile) minimiza los conflictos de código. Segundo, el contrato de API — los endpoints y los DTOs — se acordó antes de implementar, y las interfaces TypeScript generadas desde OpenAPI mantienen la coherencia. Tercero, las revisiones de PR en GitHub antes de mergear a main.

---

### P18. ¿Qué hubieran hecho diferente si tuvieran más tiempo?

**Respuesta:**
> Cuatro cosas concretas. Primero, implementar el `@socket.io/redis-adapter` para escalar el chat horizontalmente. Segundo, completar el flujo Auth0 PKCE en mobile para tener login con Google. Tercero, ampliar las especialidades de triage — actualmente Medicina General, Odontología y Urgencias; agregaríamos Pediatría y Urgencias especializadas. Cuarto, integrar una pasarela de pago real como Wompi en lugar del checkout simulado.

---

## Categoría 5: Sobre el despliegue y la infraestructura

### ⚠️ P19. ¿Dónde está desplegado el sistema?

**Respuesta:**
> El backend está en Railway, usando el Dockerfile existente con despliegue automático desde GitHub. El web está en Vercel con integración nativa de Next.js. El APK de Android se generó con EAS Build de Expo. MongoDB es MongoDB Atlas en un cluster gratuito para el MVP. Redis es Railway Redis addon. La arquitectura objetivo documentada es AWS ECS Fargate con ALB, pero para el MVP académico Railway y Vercel son más rápidos de configurar.

---

### P20. ¿Cómo hacen el despliegue? ¿Tienen CI/CD?

**Respuesta:**
> Tenemos CI/CD parcial. El CI está completo en GitHub Actions: lint → typecheck → unit tests → E2E tests → build para los tres servicios. El CD de deploy está configurado en Railway y Vercel con deployment automático en push a main. Lo que falta es el pipeline de deploy en GitHub Actions que coordine los tres servicios juntos — está documentado como deuda técnica en el Plan Técnico.

---

### P21. ¿Cómo manejan los secretos en producción?

**Respuesta:**
> Las variables de entorno sensibles — JWT_SECRET, GEMINI_API_KEY, MongoDB URI — viven como environment variables en Railway y Vercel, nunca en el código. El backend valida todas las variables con Joi al arrancar en modo producción: si falta alguna variable requerida, el proceso no arranca y falla rápido con un mensaje claro. Esto evita deploys a producción con configuración incompleta.

---

## Categoría 6: Sobre el producto y el negocio

### ⚠️ P22. ¿Cuál es el modelo de negocio?

**Respuesta:**
> El modelo implementado en el MVP es pago por consulta — el paciente paga una tarifa por especialidad antes de acceder a la consulta. Esta tarifa es configurable por el administrador. El ciclo de vida del pago es PENDING → COMPLETED → REFUNDED, con métricas de revenue en el dashboard admin. El modelo más amplio prevé suscripciones premium para pacientes y suscripciones para médicos, pero estos son parte del roadmap de producción.

---

### P23. ¿Cómo se protege la información del paciente?

**Respuesta:**
> Tres capas. Primero, autenticación con JWT — ningún endpoint de datos del paciente es público. Segundo, RBAC estricto — un médico no puede acceder a datos de otro médico, un paciente no puede ver datos de otro paciente. Tercero, los datos sensibles que van a Gemini son anonimizados — solo las respuestas del cuestionario, sin PII. Los logs están configurados para enmascarar PII antes de escritura.

---

### P24. ¿En qué se diferencia SaludDeUna de Doctor en Casa o similares?

**Respuesta:**
> Doctor en Casa y similares son plataformas de telemedicina completas — consulta video, historia clínica, prescripción digital. SaludDeUna es una capa de comunicación y priorización previa a la consulta. No pretende reemplazar esas plataformas — podría integrarse con ellas para mejorar la calidad de la información que llega al médico. El diferenciador es el triage asistido por IA con guardrail ético y el pipeline RAG con conocimiento clínico propio.

---

### P25. ¿Esto está listo para usuarios reales?

**Respuesta:**
> El sistema está funcionando en producción con un conjunto de datos de demo. Para usuarios reales, hay tres pasos pendientes: uno, validar el flujo Auth0 PKCE en dispositivos físicos para el login de pacientes. Dos, configurar el Redis adapter para que el chat escale con múltiples instancias del backend. Tres, hacer una revisión de seguridad formal y, en una etapa posterior, evaluar los requisitos regulatorios según la Resolución 2654 de 2019 sobre telemedicina en Colombia. El MVP no se presenta como una plataforma médica regulada.

---

## Categoría 7: Preguntas de comunicación técnica

### P26. ¿Por qué usaron Next.js 16 con App Router y no una versión más estable?

**Respuesta:**
> Next.js 16 con App Router es la versión estable actual — no es una versión inestable. El App Router ofrece streaming de respuestas, React Server Components nativos y layouts anidados que simplifican la arquitectura de rutas. Para un portal staff con múltiples secciones (admin, doctor) los route groups `(auth)/` y `(staff)/` hacen el código más mantenible que el Pages Router.

---

### P27. ¿Por qué usaron React Native + Expo y no Flutter?

**Respuesta:**
> Tres razones. Primero, el stack del proyecto es TypeScript en todos los servicios — usar React Native mantiene coherencia de lenguaje. Segundo, Expo simplifica significativamente el build y distribución para un MVP — EAS Build genera APKs sin necesidad de un entorno Android SDK completo. Tercero, el equipo tiene experiencia previa con React y las curvas de aprendizaje de React Native son menores que Flutter para este equipo.

---

### P28. ¿Qué es el Transactional Outbox y por qué es necesario?

**Respuesta:**
> Es un patrón de arquitectura que garantiza que los eventos de dominio se emiten exactamente una vez, incluso si el proceso falla a mitad de camino. Sin outbox, si el backend se cae después de cerrar una consulta pero antes de crear los jobs de followup, los followups nunca existirían y el paciente no recibiría su seguimiento. Con outbox, el cierre de consulta y la creación del evento outbox son atómicos. El Worker lee el outbox y reintenta hasta 5 veces con backoff si falla.

---

### P29. ¿Qué es un guardrail y cómo garantizan que funciona?

**Respuesta:**
> Un guardrail clínico es un filtro que evalúa la salida de la IA antes de usarla. El nuestro está implementado en `GuardrailService` con reglas en `guardrail-rules.json`. Las reglas detectan patrones de texto de tres categorías: diagnóstico, prescripción y afirmación clínica. La garantía es que el guardrail tiene tests unitarios propios — si las reglas fallan, los tests fallan. También hay tests de integración que verifican que el campo `guardrailApplied` se active correctamente.

---

### ⚠️ P30. ¿Cuál fue el mayor desafío técnico del proyecto y cómo lo resolvieron?

**Respuesta:**
> El mayor desafío fue garantizar que la integración de IA fuera ética y técnicamente robusta. No es trivial usar un LLM en un contexto de salud sin que genere lenguaje de diagnóstico. La solución fue el guardrail determinista — código, no prompts. El segundo desafío fue la arquitectura del chat en tiempo real con Socket.IO junto con el backend modular de NestJS. Lo resolvimos con el patrón de gateway de NestJS y el módulo Redis para el adapter de pub/sub. El chat funciona, los mensajes se persisten en MongoDB y el fallback de single-instance → Redis adapter está documentado y planeado.
