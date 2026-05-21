# SaludDeUna — Contenido Completo de Diapositivas

> Listo para copiar a Canva, Google Slides, PowerPoint o Gamma.
> Cada slide incluye: título, texto principal, bullets, elementos visuales y notas del presentador.

---

## SLIDE 1 — PORTADA

**Título grande:**

```
SaludDeUna
```

**Subtítulo:**

```
Plataforma de comunicación y priorización clínica asistida por IA
```

**Equipo:**

```
Jesús Jauregui · Mayerlly Suárez · Natalia Espitia · Santiago Hurtado
IETI 2026-1 · Escuela Colombiana de Ingeniería Julio Garavito
```

**Elementos visuales:**

- Logo o ícono de cruz médica + símbolo de IA (chip o red neuronal) fusionados
- Fondo oscuro con degradado azul-violeta sutil
- QR del APK en esquina inferior derecha (pequeño pero visible)
- Badge tecnológico: NestJS · Expo · Gemini AI · MongoDB

**Notas del presentador:**
> Proyectar esta slide al entrar al salón. QR debe ser escaneable. El primer texto que el jurado lee define el tono: no es telemedicina, es priorización clínica con IA.

---

## SLIDE 2 — AGENDA

**Título:** `Agenda`

**Bullets:**

```
01  El problema identificado
02  Propuesta de valor — SaludDeUna
03  Arquitectura del sistema
04  Criterios de calidad
05  IA en el producto y en el desarrollo
06  Demo en vivo  ← APK + QR siempre visible
07  Resultados y conclusiones
```

**Elementos visuales:**

- Lista numerada con íconos a la izquierda
- Timeline horizontal con 7 puntos (opción alternativa)

**Notas del presentador:**
> "En los próximos 20 minutos vamos a recorrer estos 7 temas. Empezamos con el problema que motivó el proyecto."

---

## SLIDE 3 — EL PROBLEMA EN UNA FRASE

**Título:** `El problema`

**Frase central grande:**

```
"Cada día, médicos y pacientes pierden tiempo y calidad
porque la información clínica llega mal organizada,
tarde o incompleta."
```

**Elementos visuales:**

- Fondo oscuro con esa frase en tipografía grande, centrada
- Pequeño ícono de reloj roto o conversación confusa
- Subtexto pequeño: "Colombia · Atención primaria · Consultas de 10 minutos"

**Notas del presentador:**
> Esta frase sintetiza todo. No más de 30 segundos aquí — el impacto está en el silencio después de leerla.

---

## SLIDE 4 — EL CAOS DE LA CONSULTA (PACIENTE)

**Título:** `Del lado del paciente`

**Columna izquierda — El problema:**

```
🤕  No sabe si su síntoma es urgente
💬  No sabe cómo explicar lo que siente
🧠  Olvida información importante en la cita
⏰  Consultas cortas que no resuelven
❌  Sin orientación médica oportuna
```

**Columna derecha — Consecuencia:**

```
→  Visitas innecesarias a urgencias
→  Diagnósticos tardíos
→  Saturación del sistema
→  Frustración y desconfianza
```

**Elementos visuales:**

- Layout de 2 columnas con iconos lineales
- Una foto o ilustración de persona mirando síntomas en el celular
- Fondo oscuro, texto blanco

**Notas del presentador:**
> "El paciente llega a la cita sin poder describir bien lo que siente. Esto no es un problema de educación — es un problema de estructura."

---

## SLIDE 5 — EL COSTO OCULTO (MÉDICO)

**Título:** `Del lado del médico`

**Tres tarjetas horizontales:**

```
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│  INFORMACIÓN        │  │  TIEMPO              │  │  CONTINUIDAD        │
│  DESORGANIZADA      │  │  DESPERDICIADO       │  │  ROTA               │
│                     │  │                      │  │                     │
│  El médico recolecta│  │  40–60% del tiempo   │  │  Sin historia       │
│  antecedentes desde │  │  de consulta es      │  │  estructurada entre │
│  cero en cada cita  │  │  anamnesis básica    │  │  citas              │
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
```

**Elementos visuales:**

- 3 tarjetas con íconos: 📋 🕐 🔗
- Fondo oscuro, tarjetas gris oscuro con borde azul tenue

**Notas del presentador:**
> "El médico no tiene el problema de información — tiene el problema de recibirla sin estructura, repetida y sin contexto previo."

---

## SLIDE 6 — PROPUESTA DE VALOR

**Título:** `SaludDeUna — La capa inteligente`

**Frase central:**

```
Una capa de IA entre paciente y médico
que estructura, prioriza y comunica
la información clínica antes de la consulta.
```

**Tres pilares (tarjetas):**

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  🎯 TRIAGE   │  │  📋 RESUMEN  │  │  📈 SEGUIM.  │
│  ASISTIDO    │  │  CLÍNICO     │  │  POST-       │
│  POR IA      │  │  AUTOMÁTICO  │  │  CONSULTA    │
│              │  │              │  │              │
│ Clasifica    │  │ Doctor llega │  │ Followups    │
│ prioridad    │  │ informado    │  │ 72h y 7d     │
│ LOW·MOD·HIGH │  │ antes de la  │  │ automáticos  │
│              │  │ consulta     │  │ + escalación │
└──────────────┘  └──────────────┘  └──────────────┘
```

**Elementos visuales:**

- 3 tarjetas con gradiente de fondo (azul, verde, violeta)
- Ícono central grande en cada tarjeta

**Notas del presentador:**
> "SaludDeUna no reemplaza al médico. Le ahorra tiempo y le da mejor información."

---

## SLIDE 7 — QUÉ NO ES SALUDDEUNA

**Título:** `Límites claros — esto no es SaludDeUna`

**Layout: Columna NO vs. SÍ:**

```
❌ NO ES                          ✅ SÍ ES
─────────────────────────────     ───────────────────────────
✗ Telemedicina completa           ✓ Plataforma de comunicación
✗ Diagnóstico automático por IA   ✓ Triage asistido (prioridad)
✗ Prescripción de medicamentos    ✓ Resumen clínico estructurado
✗ Reemplazo del médico            ✓ Apoyo al juicio clínico
✗ Sistema regulado (Res. 2654)    ✓ MVP académico/profesional
```

**Frase al pie:**

```
"La IA no diagnostica. La IA organiza, clasifica y comunica."
```

**Elementos visuales:**

- Dos columnas con checkmarks y X en verde y rojo
- Frase al pie en grande, color verde menta

**Notas del presentador:**
> "Esta slide es importante para el jurado. Si la IA diagnosticara, habría implicaciones regulatorias. Aquí la IA clasifica prioridad — no hace diagnóstico clínico."

---

## SLIDE 8 — ALCANCE REAL DEL MVP

**Título:** `Alcance del MVP — Lo que está implementado`

**Tabla de features:**

```
FEATURE                    BACKEND   WEB     MOBILE   ESTADO
──────────────────────────────────────────────────────────────
Registro y autenticación     ✅       ✅       ✅      Completo
Triage asistido por IA       ✅       —        ✅      Completo
Cola de consultas médicas    ✅       ✅       —       Completo
Chat clínico en tiempo real  ✅       ✅       ✅      Completo
Resumen clínico IA           ✅       ✅       —       Completo
Seguimiento post-consulta    ✅       —        ✅      Completo
Verificación REThUS (admin)  ✅       ✅       —       Completo
Dashboard KPIs               ✅       ✅       —       Completo
Billing simulado             ✅       ✅       ✅      Completo
```

**Nota al pie:**

```
3 especialidades: Medicina General · Odontología · Urgencias
406+ tests · Cobertura ≥ 80% en backend
```

**Elementos visuales:**

- Tabla con checkmarks en verde y colores de columna diferenciados
- Badges de tecnología al pie

**Notas del presentador:**
> "Todo esto está funcionando. No es mockup — el backend tiene 406 tests y 93% de cobertura."

---

## SLIDE 9 — TRES ROLES, UN SISTEMA

**Título:** `Tres roles — Una plataforma`

**Tres columnas:**

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  👤 PACIENTE    │  │  🩺 MÉDICO       │  │  🛡️ ADMIN       │
│  App Mobile     │  │  Panel Web      │  │  Panel Web      │
│  (Expo 55)      │  │  (Next.js 16)   │  │  (Next.js 16)   │
│                 │  │                 │  │                 │
│ • Registro      │  │ • Cola verific. │  │ • REThUS verify │
│ • Triage IA     │  │ • Chat clínico  │  │ • Gestión users │
│ • Checkout      │  │ • Resumen IA    │  │ • Billing       │
│ • Chat          │  │ • Historial     │  │ • Dashboard KPI │
│ • Followup      │  │   del paciente  │  │ • Métricas IA   │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

**Elementos visuales:**

- 3 columnas con ícono de rol arriba
- Fondo de tarjeta diferente por rol (azul, verde, violeta)
- Mockup pequeño de cada interfaz al lado

**Notas del presentador:**
> "Cada rol tiene su propio canal. El paciente usa mobile. El médico y admin usan la web."

---

## SLIDE 10 — FLUJO DEL NEGOCIO E2E

**Título:** `Flujo de principio a fin`

**Diagrama de flujo horizontal:**

```
[Paciente]     [Triage IA]    [Checkout]    [Cola]    [Doctor]    [Cierre]    [Followup]
    │               │              │            │          │           │            │
  Login      Cuestionario      Pago simu.   Espera    Chat real   Resumen IA    72h + 7d
    │         + Gemini          billing       FIFO      Socket.IO   Gemini      BullMQ
    │         + red-flags         │            │          │           │            │
    │         + prioridad         │            │       asigna       cierra     escalación
    └────────────┴────────────────┴────────────┴──────────┴───────────┴────────────┘
```

**Elementos visuales:**

- Diagrama horizontal con flechas y etiquetas de tecnología
- Colores por actor (azul=paciente, verde=doctor, violeta=admin/IA)
- Usar el Diagrama 11 de DIAGRAMAS-MERMAID.md exportado como SVG

**Notas del presentador:**
> "Este es el flujo completo. Vamos a ver cada parte en el demo. La IA entra en el paso 2 — el triage — y también al cierre cuando el doctor pide el resumen clínico."

---

## SLIDE 11 — ARQUITECTURA GENERAL (C4 L1)

**Título:** `Arquitectura — Contexto del sistema`

**Diagrama:** C4 Nivel 1 de DIAGRAMAS-MERMAID.md (exportar §1 como SVG)

**Leyenda al pie:**

```
Paciente (Mobile)   →  Backend NestJS  →  Google Gemini 2.5
Médico/Admin (Web)  →  Backend NestJS  →  MongoDB Atlas
                                        →  Redis Cloud
                                        →  Auth0
                                        →  RETHUS Colombia
```

**Nota técnica lateral:**

```
Protocolo: HTTPS / WebSocket (Socket.IO)
Auth: JWT HS256 + Auth0 RS256
```

**Elementos visuales:**

- Diagrama SVG del C4 L1 (ya existe en DIAGRAMAS-MERMAID.md §1)
- Leyenda limpia al pie
- Fondo oscuro

**Notas del presentador:**
> "Tres actores humanos interactúan con un sistema central. El sistema se apoya en servicios externos: la IA de Google, la base de datos, el caché y la autenticación."

---

## SLIDE 12 — ARQUITECTURA DE CONTENEDORES (C4 L2)

**Título:** `Arquitectura — Vista de contenedores`

**Diagrama:** C4 Nivel 2 de DIAGRAMAS-MERMAID.md (exportar §2 como SVG)

**Tabla de contenedores:**

```
Contenedor          Tecnología              Responsabilidad
──────────────────────────────────────────────────────────
Expo Mobile         React Native / Expo 55  App del paciente
Next.js Web         Next.js 16 / App Router Portal staff (Dr + Admin)
NestJS API          NestJS 11 / Node 20     Toda la lógica de negocio
NestJS Worker       BullMQ                  Jobs async + Outbox
MongoDB Atlas       MongoDB 7               Datos + Vector Search
Redis Cloud         Redis 7                 Cache + BullMQ + throttling
```

**Elementos visuales:**

- Diagrama SVG del C4 L2
- Tabla compacta al pie derecho

**Notas del presentador:**
> "Dos frontends separados. Un backend con un proceso API y un proceso Worker. La base de datos tiene búsqueda vectorial para el RAG."

---

## SLIDE 13 — BACKEND POR DENTRO

**Título:** `Backend — 17 módulos NestJS`

**Diagrama de módulos agrupados:**

```
USUARIOS          CLÍNICO            IA / RAG          INFRAESTRUCTURA
──────────        ──────────         ──────────         ──────────
AuthModule        TriageModule        AiModule           OutboxModule
PatientsModule    ConsultationsModule KnowledgeModule    DashboardModule
DoctorsModule     ChatModule          RagModule          AdminModule
AdminsModule      BillingModule                          OTel Bootstrap
                  FollowupsModule
                  NotificationsModule
```

**Elementos visuales:**

- Diagrama del Módulo Backend de DIAGRAMAS-MERMAID.md §3 (exportar como SVG)
- 4 grupos de colores (azul, amarillo, violeta, rojo)

**Puntos clave al pie:**

```
✓ Prefijo global: /v1        ✓ ValidationPipe global (DTOs)
✓ Guards globales: JWT + RBAC + Throttler
✓ Filtro global de excepciones con correlationId
```

**Notas del presentador:**
> "Todo el backend sigue el patrón modular de NestJS. Los módulos se comunican vía inyección de dependencias — no hay acoplamiento directo entre dominios."

---

## SLIDE 14 — PIPELINE DE TRIAGE IA

**Título:** `Triage asistido por IA — El corazón del producto`

**Diagrama de flujo del pipeline RAG:**

```
Paciente responde           Gemini 2.5-flash          Resultado
el cuestionario    ──────►  analiza respuestas  ─────► prioridad:
(5–8 preguntas               + reglas de               LOW / MOD / HIGH
por especialidad)             red-flags               + red flags detectadas
                                   │
                                   ▼
                           Guardrail clínico
                        (filtra diagnóstico /
                         prescripción / afirmación)
                                   │
                                   ▼
                         Resumen neutral de urgencia
                         (sin diagnóstico · sin Rx)
```

**Tres puntos clave:**

```
1. La IA nunca diagnostica — clasifica prioridad
2. El guardrail bloquea cualquier salida de diagnóstico/prescripción
3. Si el guardrail activa → fallback a reglas basadas en código
```

**Elementos visuales:**

- Diagrama de flujo simple con colores
- Ícono de escudo para el guardrail
- Usar diagrama §5 de DIAGRAMAS-MERMAID.md (pipeline RAG) como apoyo

**Notas del presentador:**
> "Este es el punto técnico más sensible. La IA no genera un diagnóstico — genera una clasificación de prioridad y un resumen de urgencia. El guardrail es código, no una promesa."

---

## SLIDE 15 — CRITERIOS DE CALIDAD (RESUMEN)

**Título:** `Criterios de Calidad — 6 implementados`

**Grid de 6 tarjetas:**

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  🔒          │  │  🧪          │  │  🔍          │
│  SEGURIDAD   │  │ CONFIABILIDAD│  │OBSERVABILIDAD│
│              │  │              │  │              │
│ JWT+Auth0    │  │ 406+ tests   │  │ OpenTelemetry│
│ RBAC+Guards  │  │ 93% cobertura│  │ + Correlation│
└──────────────┘  └──────────────┘  └──────────────┘
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  🏗️          │  │  ⚡          │  │  🔄          │
│MANTENIBILIDAD│  │ RENDIMIENTO  │  │DISPONIBILIDAD│
│              │  │              │  │              │
│ NestJS modular│  │ p95 < 1500ms │  │ Health +     │
│ + DI + SOLID │  │ Redis cache  │  │ Readiness    │
└──────────────┘  └──────────────┘  └──────────────┘
```

**Elementos visuales:**

- 6 tarjetas en grid 3×2 con ícono grande y 2 líneas de evidencia

**Notas del presentador:**
> "Vamos a ver los tres más relevantes en detalle."

---

## SLIDE 16 — SEGURIDAD Y AUTENTICACIÓN

**Título:** `Criterio de calidad: Seguridad`

**Diagrama de flujo de auth (simplificado):**

```
Cliente               NestJS API            MongoDB/Redis/Auth0
  │                      │                        │
  │ POST /auth/login      │                        │
  ├──────────────────────►│                        │
  │                      │── Throttler (20/60s) ──►│
  │                      │── bcrypt verify ────────►│
  │                      │── Max 3 sesiones ───────►│
  │                      │── sign JWT HS256         │
  │◄── accessToken ───────│                        │
  │    refreshToken       │                        │
```

**5 controles implementados:**

```
✓ JWT HS256 (legacy) + Auth0 RS256 — dual auth
✓ RBAC: PATIENT · DOCTOR · ADMIN (guards globales)
✓ Rate limiting: 20 req/60s por IP (Redis throttler)
✓ bcrypt cost 12 para contraseñas
✓ Max 3 sesiones activas por usuario — revocación LIFO
✓ Correlation ID en todas las respuestas
```

**Elementos visuales:**

- Mini diagrama de secuencia de login
- Lista con checkmarks

**Notas del presentador:**
> "La seguridad no es opcional en un sistema de salud. Cada request pasa por throttling, autenticación JWT y verificación de rol antes de llegar al controlador."

---

## SLIDE 17 — TESTING Y COBERTURA

**Título:** `Criterio de calidad: Confiabilidad`

**Métricas grandes (números visibles):**

```
     406+              93%               82               ≥ 80%
   Tests              Statements        Tests web        Umbral
  backend             cobertura         pasando          todos
  pasando             backend                            módulos
```

**Estrategia de testing:**

```
UNIT TESTS                  E2E TESTS (Backend)
──────────────              ────────────────────────────────────
Jest + mocks NestJS         mongodb-memory-server (BD real)
AiService siempre           12 suites: auth · admin · triage ·
mockeado en unit            chat · billing · followups · system
                            NO se mockea la capa de BD
```

**Elementos visuales:**

- 4 números grandes en boxes destacados
- Tabla con dos columnas (unit vs e2e)
- Badge de SonarCloud (quality gate)

**Notas del presentador:**
> "406 tests pasando, ninguno fallando. Los E2E usan una base de datos MongoDB real en memoria — no hay mocks de BD. Esto significa que si algo falla en producción, los tests lo habrían detectado."

---

## SLIDE 18 — OBSERVABILIDAD Y TRAZABILIDAD

**Título:** `Criterio de calidad: Observabilidad`

**Stack de observabilidad:**

```
NestJS API                    →  OpenTelemetry NodeSDK (OTLP)
  HTTP instrumentation             │
  MongoDB instrumentation          ▼
  Redis instrumentation       OTLP Collector → Jaeger/Tempo (trazas)
  LoggingInterceptor                       → Prometheus/Grafana (métricas)
  AI Tracer (RAG spans)                    → CloudWatch (logs)
```

**4 capacidades:**

```
📋  Logs estructurados JSON con correlationId en todas las respuestas
📊  Métricas técnicas: p95 latencia, error rate, cache hit rate
🔍  Trazas distribuidas: span hierarchy completa por request
🤖  AI Audit: RagTrace docs — query, chunks, score, guardrail, ms
```

**Dashboard API:**

```
GET /v1/dashboard/technical  →  p95 latency, error rate (tiempo real)
GET /v1/dashboard/business   →  4 KPIs: pacientes, doctores, revenue, IA
```

**Elementos visuales:**

- Diagrama de stack de observabilidad (simplificado del §10 de DIAGRAMAS-MERMAID.md)
- 4 bullets con íconos

**Notas del presentador:**
> "Cada request tiene un correlationId único. Podemos trazar cualquier error desde el cliente hasta la base de datos."

---

## SLIDE 19 — IA EN EL PRODUCTO

**Título:** `IA en el producto — Tres funciones`

**Tres columnas:**

```
┌───────────────────────┐  ┌───────────────────────┐  ┌───────────────────────┐
│  🎯 TRIAGE            │  │  📋 RESUMEN CLÍNICO    │  │  📚 RAG               │
│  ANÁLISIS             │  │  POST-CHAT             │  │  KNOWLEDGE BASE       │
│                       │  │                        │  │                       │
│ Modelo: gemini-2.5    │  │ Modelo: gemini-2.5     │  │ Modelo: embedding-001 │
│ Input: 5–8 respuestas │  │ Input: mensajes del    │  │ Dimensiones: 768      │
│ Output: prioridad     │  │ chat clínico           │  │ Índice: Atlas HNSW    │
│ + red flags           │  │ Output: nota médica    │  │ Cache: Redis 300s     │
│ + resumen neutral     │  │ + disclaimer clínico   │  │                       │
│                       │  │                        │  │ Enriquece contexto    │
│ Guardrail: siempre    │  │ Guardrail: siempre     │  │ del triage y resumen  │
└───────────────────────┘  └───────────────────────┘  └───────────────────────┘
```

**Nota al pie:**

```
Gemini API Key activa · AI_ENABLED=true en producción · Fallback a reglas si Gemini falla
```

**Elementos visuales:**

- 3 tarjetas de color diferente por función de IA
- Logo de Google Gemini pequeño en cada tarjeta

**Notas del presentador:**
> "Gemini tiene tres roles distintos en el sistema. El más crítico es el triage. El guardrail es código que se ejecuta antes de persistir cualquier respuesta de la IA."

---

## SLIDE 20 — GUARDRAIL CLÍNICO

**Título:** `Guardrail — La IA bajo control`

**Diagrama simple:**

```
Gemini genera texto
        │
        ▼
┌───────────────────────────────────────────────────┐
│  GUARDRAIL CLÍNICO                                │
│  guardrail-rules.json · GuardrailService          │
│                                                    │
│  Detecta categorías bloqueadas:                    │
│    ✗ "diagnóstico": "Usted tiene..."              │
│    ✗ "prescripción": "Tome este medicamento..."   │
│    ✗ "afirmación clínica": "Su condición es..."   │
└───────────────────────────────────────────────────┘
        │                    │
        ▼                    ▼
   safe=true             safe=false
        │                    │
   persiste resumen      aiSummary = null
   neutral de urgencia   guardrailApplied = true
                         log WARN + correlationId
```

**Frase clave:**

```
"El guardrail no es una promesa — es código que se ejecuta en cada respuesta de IA."
```

**Elementos visuales:**

- Diagrama de flujo con escudo verde/rojo
- Frase grande al pie

**Notas del presentador:**
> "Si alguien pregunta '¿cómo garantizan que la IA no diagnostica?', esta es la respuesta técnica."

---

## SLIDE 21 — IA EN EL PROCESO DE DESARROLLO

**Título:** `IA en el proceso de desarrollo`

**4 evidencias concretas:**

```
🤖 GENERACIÓN DE CÓDIGO
   Claude Code (Anthropic) como asistente principal durante el desarrollo.
   Generación de módulos NestJS, tests E2E, hooks React, esquemas Zod.

🧪 TESTING ASISTIDO
   Generación y revisión de suites de tests E2E con mongodb-memory-server.
   Cobertura: 406 tests backend · 82 tests web.

📚 DOCUMENTACIÓN AUTOMÁTICA
   Auditoría completa de documentación, detección de inconsistencias
   y actualización de wiki, READMEs y diagramas vía IA.

🏗️ ARQUITECTURA Y REVISIÓN
   Análisis arquitectónico, identificación de deuda técnica y
   roadmap técnico generado con asistencia de IA.
```

**Nota al pie:**

```
Herramientas: Claude Code (Anthropic) · GitHub Copilot · ChatGPT
```

**Elementos visuales:**

- 4 bloques con ícono y descripción corta
- Logos de herramientas de IA usadas

**Notas del presentador:**
> "La IA no solo es parte del producto — fue parte del proceso. Usamos Claude Code para generar código, revisar arquitectura y mantener la documentación."

---

## SLIDE 22 — RUTA DEL DEMO

**Título:** `Demo en vivo — Ruta planeada`

**Flujo numerado:**

```
1. APK + QR  ─────────────────────────► Proyectar QR, mostrar APK en dispositivo
2. Login paciente (mobile)             → paciente.demo@saluddeuna.com
3. Seleccionar especialidad            → Medicina General
4. Completar triage (3–4 preguntas)    → respuestas preparadas
5. Ver resultado: PRIORIDAD ALTA 🔴    → con red flag detectado
6. Checkout simulado                   → confirmar pago
7. LOGIN DOCTOR (web) — Tab incógnito  → doctor.demo@saluddeuna.com
8. Ver cola → consulta aparece arriba  → prioridad HIGH visible
9. Chat en tiempo real (ambas apps)    → mensaje demo preparado
10. Doctor genera resumen IA           → Gemini → nota médica
11. Doctor cierra consulta             → followup auto-creado
12. LOGIN ADMIN — KPIs en dashboard    → revenue, pacientes, doctores
```

**Nota de respaldo:**

```
⚠️  Si falla red → screenshots backup en Downloads/demo-backup/
⚠️  Si falla Gemini → sistema usa RULE_BASED automáticamente (sin error visible)
⚠️  Si falla APK → mostrar el mismo flujo en browser web
```

**Elementos visuales:**

- Lista numerada con íconos de usuario/dispositivo/acción
- Caja de respaldo en rojo tenue al pie

**Notas del presentador:**
> "Tenemos 12 pasos claros. Si algo falla, tenemos respaldo. El demo empieza siempre con el QR proyectado."

---

## SLIDE 23 — DEMO ACTIVO

**Título:** `Demo en vivo — SaludDeUna`

**Esta slide permanece en pantalla durante toda la demo.**

**Layout:**

```
┌─────────────────────────────────────────────────────────────────┐
│                                               ┌───────────┐     │
│                                               │  [QR APK] │     │
│  DEMO EN VIVO                                 │           │     │
│                                               │ Escanear  │     │
│  Backend: https://api.saluddeuna.com/v1       │ para APK  │     │
│  Web: https://staff.saluddeuna.com            └───────────┘     │
│                                                                  │
│  paciente.demo@saluddeuna.com  /  Dem0.P4c1ente!                 │
│  doctor.demo@saluddeuna.com   /  Dem0.D0ct0r!                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Elementos visuales:**

- QR del APK grande y visible (mínimo 6cm en pantalla proyectada)
- URLs de producción visibles
- Credenciales de demo visibles (no es seguridad crítica, son datos de demo)

**Notas del presentador:**
> "El QR queda visible todo el tiempo. El jurado puede escanear cuando quiera."

---

## SLIDE 24 — RESULTADOS LOGRADOS

**Título:** `Resultados — Lo que logramos`

**Métricas en números grandes:**

```
     17           406+           3              2h
Módulos         Tests          Roles         Frontends
NestJS          pasando        de usuario    separados
backend         0 fallando
```

```
   93%           ≥80%           12             5
Statements     Branches        Endpoints      Meses de
cobertura      cobertura       triage + chat  desarrollo
```

**Logros cualitativos:**

```
✓ Sistema funcionando en producción (Railway + Vercel)
✓ APK Android disponible via EAS Build
✓ Chat en tiempo real con Socket.IO
✓ Pipeline RAG con MongoDB Atlas Vector Search (768 dims)
✓ Guardrail clínico activo en producción
✓ 4 KPIs de negocio con datos reales
```

**Elementos visuales:**

- Grid de números grandes y métricas
- Lista de logros con checkmarks

**Notas del presentador:**
> "Estas son cifras reales — no estimadas. Todo está desplegado y funcionando."

---

## SLIDE 25 — CONCLUSIONES Y CIERRE

**Título:** `SaludDeUna — Más que un proyecto`

**Frase central:**

```
Construimos una plataforma real
que resuelve un problema real
con tecnología de producción
y criterios de calidad medibles.
```

**Lo que aprendimos:**

```
→ La IA en salud requiere guardrails éticos explícitos — no solo técnicos
→ La arquitectura modular nos permitió paralelizar el desarrollo sin conflictos
→ Los tests E2E detectaron bugs reales que los unit tests no habrían encontrado
→ La observabilidad desde el inicio ahorra horas de debugging en producción
```

**Invitación final:**

```
Código abierto · Tests pasando · Demo en vivo · APK disponible

¿Preguntas?
```

**Elementos visuales:**

- Frase grande centrada sobre fondo oscuro
- Bullets de aprendizaje en tipografía más pequeña
- QR del APK visible en esquina (siempre)

**Notas del presentador:**
> "Cerramos con esto: no es solo un proyecto académico. Es un sistema que podría escalar a usuarios reales. Las decisiones técnicas que tomamos — arquitectura, testing, seguridad, observabilidad — son decisiones que se toman en producción real."
