# SaludDeUna — Diagramas Arquitectónicos en Mermaid
> Versión corregida — 19 mayo 2026  
> Para visualizar: VS Code con extensión **Mermaid Preview** o pega en [mermaid.live](https://mermaid.live)

---

## 1. C4 Nivel 1 — Contexto del Sistema

```mermaid
C4Context
    title C4 Nivel 1 — Contexto del Sistema SaludDeUna

    Person(paciente, "Paciente", "Necesita atención médica primaria")
    Person(medico, "Médico", "Profesional de salud verificado REThUS")
    Person(admin, "Administrador", "Gestiona plataforma, doctores y precios")

    System(saluddeuna, "SaludDeUna", "Plataforma clínica IA: triage guiado, chat tiempo real, seguimiento post-consulta y facturación")

    System_Ext(gemini, "Google Gemini 2.5", "IA generativa: triage + embeddings + resumen clínico")
    System_Ext(mongodb, "MongoDB Atlas", "Persistencia documentos + Vector Search HNSW 768d")
    System_Ext(redis, "Redis Cloud", "Cache · BullMQ queues · Socket.IO pub/sub")
    System_Ext(auth0, "Auth0", "Identidad · JWT RS256 · JWKS endpoint")
    System_Ext(rethus, "RETHUS Colombia", "Registro de credenciales médicas")
    System_Ext(expopush, "Expo Notifications", "Push notifications · FCM / APNs")

    Rel(paciente, saluddeuna, "Realiza triage y consultas", "HTTPS / WebSocket")
    Rel(medico, saluddeuna, "Atiende consultas y chatea", "HTTPS / WebSocket")
    Rel(admin, saluddeuna, "Gestiona operaciones y precios", "HTTPS")

    Rel(saluddeuna, gemini, "Triage IA · RAG · resumen clínico", "HTTPS API")
    Rel(saluddeuna, mongodb, "Persiste datos clínicos y vectores", "Mongoose/BSON")
    Rel(saluddeuna, redis, "Cache · throttling · chat pub/sub", "ioredis")
    Rel(saluddeuna, auth0, "Valida tokens JWT RS256", "HTTPS/JWKS")
    Rel(saluddeuna, rethus, "Verifica credenciales médicas", "API / Manual")
    Rel(saluddeuna, expopush, "Envía push notifications", "HTTPS API")
```

---

## 2. C4 Nivel 2 — Vista de Contenedores

```mermaid
C4Container
    title C4 Nivel 2 — Vista de Contenedores — SaludDeUna Platform

    Person(paciente, "Paciente", "")
    Person(medico, "Médico", "")
    Person(admin, "Administrador", "")

    System_Boundary(saluddeuna, "SaludDeUna Platform") {
        Container(alb, "ALB", "AWS Application Load Balancer", "HTTPS termination · sticky sessions para WebSocket")
        Container(mobile, "Expo Mobile", "React Native / Expo 55", "App pacientes iOS/Android — patient-only")
        Container(web, "Next.js Web", "Next.js 16 / App Router", "Staff Portal: Médicos y Administradores únicamente")
        Container(api, "NestJS API", "NestJS 11 / Node 20 · REST+WSS", "Auth · Triage IA · Consultas · Chat · Billing · Dashboard")
        Container(worker, "NestJS Worker", "NestJS 11 / BullMQ", "Followup jobs · Outbox dispatcher · Push notifications")
        ContainerDb(mongodb, "MongoDB Atlas", "MongoDB 7 / Atlas", "Datos clínicos · Vector Search HNSW 768d")
        ContainerDb(redis, "Redis Cloud", "Redis 7 / ioredis", "Cache 300s · BullMQ · Socket.IO adapter · Rate limit")
    }

    System_Ext(gemini, "Google Gemini 2.5", "gemini-2.5-flash · embedding-001")
    System_Ext(auth0, "Auth0", "JWT RS256 · JWKS · Provisioning")
    System_Ext(expopush, "Expo Push", "FCM/APNs push notifications")
    System_Ext(rethus, "RETHUS", "Verificación médicos Colombia")

    Rel(paciente, mobile, "Usa", "HTTPS / WebSocket")
    Rel(medico, web, "Usa", "HTTPS / WebSocket")
    Rel(admin, web, "Gestiona", "HTTPS")
    Rel(mobile, alb, "API calls · chat socket", "HTTPS / WSS")
    Rel(web, alb, "API calls · chat socket", "HTTPS / WSS")
    Rel(alb, api, "Enruta /v1/* y /socket.io/*", "HTTP")
    Rel(alb, web, "Enruta /* (default)", "HTTP")
    Rel(api, mongodb, "Lee y escribe", "Mongoose ODM")
    Rel(api, redis, "Cache · pub/sub · throttle", "ioredis")
    Rel(worker, mongodb, "Lee y escribe", "Mongoose ODM")
    Rel(worker, redis, "Consume queues BullMQ", "ioredis")
    Rel(api, gemini, "Triage · RAG · resumen", "HTTPS API")
    Rel(api, auth0, "Valida tokens RS256", "HTTPS/JWKS")
    Rel(worker, expopush, "Envía push notifications", "HTTPS")
    Rel(api, rethus, "Verifica médicos", "API ext.")
```

---

## 3. Módulos Backend NestJS

```mermaid
graph TB
    subgraph CORE["🔵 CORE GLOBAL — app.module.ts"]
        AppModule["AppModule\nBootstrap · PORT 3000\nDI Root · Global prefix /v1"]
        ConfigModule["ConfigModule\nJoi Validation · .env global\nFail-fast en startup"]
        CommonModule["Common (Guards/Filters)\nGlobalExceptionFilter\nCorrelationId · Logging"]
        DatabaseModule["DatabaseModule\nMongoose · MongoDB Atlas\nAtlas URI · 15+ schemas"]
        RedisModule["RedisModule\nioredis · BullMQ\nSocket.IO adapter"]
        ThrottlerModule["ThrottlerModule\n20 req / 60s / IP\nRedis Store distribuido"]
    end

    subgraph USUARIOS["🟢 DOMINIO: USUARIOS"]
        AuthModule["AuthModule\nJWT HS256+RS256 · Auth0\nLogin · Register · Refresh\nMax 3 sessions · Provisioning"]
        PatientsModule["PatientsModule\nPerfil paciente\nTimeline clínico"]
        DoctorsModule["DoctorsModule\nEspecialidad · Schedule\nREThUS verify · Availability"]
        AdminsModule["AdminsModule\nAdmin seeder\nPerfil administrador"]
    end

    subgraph CLINICO["🟡 DOMINIO: CLÍNICO"]
        TriageModule["TriageModule\nAI Questionnaire\nRedFlagsEngine\nGuardrailService\nSpecialty questions"]
        ConsultationsModule["ConsultationsModule\nQueue · PENDING→COMPLETED\nAssign · Close · Rate · Feedback"]
        ChatModule["ChatModule\nSocket.IO rooms\nWebSocket msgs\nRedis IO adapter"]
        BillingModule["BillingModule\nTransacciones COP\nRevenue metrics\nBillingPriceSeeder"]
        FollowupsModule["FollowupsModule\nBullMQ jobs\n72h + 7d auto-followup\nEscalación si empeora"]
        NotificationsModule["NotificationsModule\nPush + in-app\nExpo Push tokens\nFCM / APNs"]
    end

    subgraph IARAGROUP["🟣 DOMINIO: IA / RAG"]
        AiModule["AiModule\nGemini 2.5-flash\nGenerate · Summarize\nPrompt versioning · Audit log"]
        KnowledgeModule["KnowledgeModule\nKnowledgeChunks · Chunking\ngemini-embedding-001\nAtlas Vector Search HNSW 768d"]
        RagModule["RagModule\nPipeline 8 pasos\nCache Redis 300s\nRagTrace · Feedback\nbuildTriageEvidence()"]
    end

    subgraph INFRA["🔴 INFRAESTRUCTURA"]
        OutboxModule["OutboxModule\nTransactional Outbox\nRetry · Backoff x5\nExactly-once semantics"]
        DashboardModule["DashboardModule\nKPIs negocio · AI metrics\nRevenue metrics · Alerts"]
        AdminModule["AdminModule\nDoctor verification\nUser management\nCSV export"]
        OtelBootstrap["OTel Bootstrap\nmain.ts init · NO NestJS module\nNodeSDK · OTLP HTTP\nAuto-instrumentación"]
    end

    subgraph CC["⚫ CROSS-CUTTING CONCERNS vía APP_GUARD / APP_INTERCEPTOR"]
        JwtAuthGuard["JwtAuthGuard — Global · @Public() bypass"]
        RolesGuard["RolesGuard — PATIENT / DOCTOR / ADMIN"]
        ThrottlerGuard2["ThrottlerGuard — 20 req / 60s / IP"]
        LoggingInterceptor["LoggingInterceptor — Structured JSON · CorrelationId"]
        GlobalExceptionFilter2["GlobalExceptionFilter — statusCode · message · CID"]
        ValidationPipe2["ValidationPipe — class-validator · whitelist · transform"]
    end

    %% Key dependencies
    TriageModule --> AiModule
    TriageModule --> RagModule
    RagModule --> KnowledgeModule
    RagModule --> AiModule
    ConsultationsModule --> OutboxModule
    FollowupsModule --> OutboxModule
    ChatModule --> RedisModule
```

---

## 4. Arquitectura de Despliegue AWS (Objetivo)

```mermaid
graph TD
    Internet["🌐 Internet / Usuarios Finales"]

    subgraph EDGE["AWS Edge"]
        Route53["Route 53\nDNS Management"]
        CloudFront["CloudFront + WAF\nCDN · DDoS · Static Assets\n⚠️ WebSocket behavior requerido"]
        ALB["ALB\nLoad Balancer · HTTPS\nSticky Sessions Socket.IO"]
    end

    subgraph VPC["VPC — us-east-1 (Multi-AZ 1a+1b)  Private Subnets"]
        subgraph ECS["ECS Fargate Cluster — FARGATE_SPOT"]
            ApiTask["API Task ×2 min\nNestJS 11 · Node 20 Alpine\n512MB · 0.5vCPU\nAuto Scale CPU>70%"]
            WebTask["Web Task ×2 min\nNext.js 16 SSR · App Router\n512MB · 0.5vCPU"]
            WorkerTask["Worker Task ×1\nNestJS 11 · BullMQ · Outbox\n256MB · 0.25vCPU\n🚫 Sin acceso desde ALB"]
        end
    end

    subgraph DATA["Capa de Datos — Servicios Gestionados"]
        MongoDB["MongoDB Atlas M10+\nMulti-AZ · Vector Search HNSW"]
        RedisCloud["Redis Cloud HA\n6GB · Cache · BullMQ"]
        ECR["ECR\nContainer Registry\ndocker images"]
        SecretsManager["AWS Secrets Manager\nAPI Keys · JWT Secrets"]
    end

    subgraph OBS["Observabilidad"]
        OTelCollector["OpenTelemetry Collector\nOTLP HTTP :4318"]
        Jaeger["Jaeger / Tempo\nDistributed Tracing"]
        Prometheus["Prometheus + Grafana\nMétricas · Alertas"]
        CloudWatch["CloudWatch\nLogs + Alarms · 7d retention"]
    end

    Internet --> Route53
    Route53 --> CloudFront
    CloudFront --> ALB
    ALB -- "/v1/* y /socket.io/*" --> ApiTask
    ALB -- "/* default" --> WebTask
    ApiTask --> MongoDB
    ApiTask --> RedisCloud
    WorkerTask --> MongoDB
    WorkerTask --> RedisCloud
    ApiTask --> OTelCollector
    OTelCollector --> Jaeger
    OTelCollector --> Prometheus
    ApiTask --> CloudWatch

    style WorkerTask fill:#4C1D95,color:#fff
    style CloudFront fill:#FF9900,color:#fff
    note["Deploy ACTUAL: Railway + Vercel\nDeploy OBJETIVO: ECS Fargate"]
```

---

## 5. Pipeline RAG Clínico

```mermaid
flowchart LR
    Q["1️⃣ Query\nEntrada\nUsuario"]
    N["2️⃣ Normalize\nLimpieza\nSHA-256 key"]
    C{"3️⃣ Cache?\nRedis TTL\n300s hit?"}
    E["4️⃣ Embed\nGemini\nembedding-001\n768 dims"]
    S["5️⃣ Search\nAtlas\n&#36;vectorSearch\ncosine sim"]
    R["6️⃣ Re-rank\nVector score\nTOP_K=8"]
    G["7️⃣ Generate\nGemini\n2.5-flash"]
    Resp["8️⃣ Response\nCitations\n+ RagTrace"]

    Q --> N --> C
    C -- "MISS" --> E --> S --> R --> G --> Resp
    C -- "HIT ⚡\nSaltar E+S" --> G

    subgraph Guardrails["🛡 Guardrails aplicados en paso 7"]
        GR1["Prompt injection protection"]
        GR2["System instruction framing"]
        GR3["CLINICAL_AI_DISCLAIMER appended"]
        GR4["No diagnostica · No prescribe"]
    end

    subgraph UseCases["Funciones que invocan el pipeline"]
        UC1["buildTriageEvidence()\nEvidencia clínica para triage IA"]
        UC2["buildConsultationSummary()\nResumen clínico post-consulta"]
        UC3["RAG_PATIENT_EVIDENCE\nEvidencia clínica para paciente"]
    end

    S -- "Sin evidencia" --> Fallback["⚠️ Fallback message\nSin evidencia disponible"]

    style C fill:#F59E0B,color:#fff
    style E fill:#1A73E8,color:#fff
    style S fill:#00684A,color:#fff
    style R fill:#7C3AED,color:#fff
    style G fill:#1A73E8,color:#fff
    style Resp fill:#0077B6,color:#fff
```

---

## 6. Flujo de Autenticación — Secuencia UML

```mermaid
sequenceDiagram
    autonumber
    participant C as Client<br/>(Web / Mobile)
    participant API as NestJS API<br/>AuthModule · Guards
    participant DB as MongoDB<br/>Users · Sessions
    participant Redis as Redis<br/>Throttle · Blacklist
    participant Auth0 as Auth0 JWKS<br/>RS256 Endpoint

    Note over C,API: Flujo de Login (JWT Legacy)
    C->>API: POST /v1/auth/patient/login<br/>OR /v1/auth/staff/login<br/>{ email, password }
    API->>Redis: ThrottlerGuard — check rate (20 req/60s)
    Redis-->>API: OK / 429 TooManyRequests
    API->>DB: find user by email + role
    DB-->>API: user doc | null → 401
    Note over API: bcrypt.compare(password, hash)<br/>→ 401 if mismatch
    API->>DB: count refresh_sessions (max 3)
    DB-->>API: sessions list | revoke oldest LIFO
    Note over API: sign accessToken (HS256 · 1h)<br/>sub:userId · role · exp
    API->>DB: insert refresh_sessions doc<br/>tokenHash · deviceInfo · TTL 7d
    API-->>C: 200 OK<br/>{ accessToken, refreshToken }

    Note over C,Auth0: Flujo de Request Protegido
    C->>API: GET /v1/... Authorization: Bearer token
    Note over API: JwtStrategy HS256 — verify signature
    alt HS256 válido
        Note over API: RolesGuard check @Roles() decorator
        API-->>C: 200 OK — response data
    else HS256 falla → intenta RS256 (Auth0)
        API->>Auth0: fetch JWKS (RS256 / Auth0 token)
        Auth0-->>API: public key → verify → user payload
        Note over API: RolesGuard check @Roles() decorator
        API-->>C: 200 OK — response data
    end

    Note over C,Auth0: Estándares: JWT RFC 7519 · HS256 (secret) · RS256 JWKS (Auth0)<br/>bcrypt cost 12 · Max 3 sesiones · Refresh rotation LIFO
```

---

## 7. Modelo de Datos — Usuarios, Auth y Billing

```mermaid
erDiagram
    patients {
        ObjectId _id PK
        String email UK
        String passwordHash
        String auth0Sub "Auth0 integration"
        String firstName
        String lastName
        String phone
        Date birthDate
        String gender
        String role "PATIENT"
        Boolean isActive
        Date createdAt
        Date updatedAt
    }

    doctors {
        ObjectId _id PK
        String email UK
        String passwordHash
        String auth0Sub "Auth0 integration"
        String firstName
        String lastName
        String personalId UK
        String phoneNumber
        String specialty "GENERAL_MEDICINE|ODONTOLOGY|URGENT_CARE"
        String doctorStatus "PENDING|VERIFIED|REJECTED"
        String professionalLicense
        String role "DOCTOR"
        Boolean isActive
        Date createdAt
        Date updatedAt
    }

    admins {
        ObjectId _id PK
        String email UK
        String passwordHash
        String role "ADMIN"
        Boolean isActive
        Date createdAt
        Date updatedAt
    }

    refresh_sessions {
        ObjectId _id PK
        ObjectId userId FK
        String userRole
        String tokenHash UK
        String deviceInfo
        String ipAddress
        Date expiresAt "TTL index auto-cleanup"
        Boolean isRevoked
        Date createdAt
    }

    billing_prices {
        ObjectId _id PK
        String specialty UK
        Number priceInCOP
        Boolean isActive
        ObjectId updatedBy
        Date createdAt
        Date updatedAt
    }

    billing_transactions {
        ObjectId _id PK
        ObjectId patientId FK
        ObjectId consultationId FK
        String specialty
        Number amount
        String status "PENDING|COMPLETED|REFUNDED"
        Date completedAt
        Date createdAt
        Date updatedAt
    }

    patients ||--o{ refresh_sessions : "tiene sesiones"
    doctors  ||--o{ refresh_sessions : "tiene sesiones"
    admins   ||--o{ refresh_sessions : "tiene sesiones"
    patients ||--o{ billing_transactions : "paga"
```

---

## 8. Modelo de Datos — Dominio Clínico

```mermaid
erDiagram
    triage_sessions {
        ObjectId _id PK
        ObjectId patientId FK
        String specialty
        String status "DRAFT|IN_PROGRESS|COMPLETED|EXPIRED"
        Array answers
        String aiSummary
        String priority "LOW|MODERATE|HIGH"
        Array redFlags
        Boolean ragContextUsed
        String analysisMode "AI_ASSISTED|RULE_BASED"
        Date completedAt
        Date createdAt
        Date updatedAt
    }

    consultations {
        ObjectId _id PK
        ObjectId patientId FK
        ObjectId doctorId FK
        ObjectId triageSessionId FK
        String specialty
        String status "PENDING_PAYMENT|WAITING|IN_PROGRESS|COMPLETED|CANCELLED"
        String clinicalNotes
        String aiSummary
        String summaryFeedback
        Date scheduledAt
        Date createdAt
        Date updatedAt
    }

    consult_messages {
        ObjectId _id PK
        ObjectId consultationId FK
        ObjectId senderId FK
        String senderRole
        String content
        String type "TEXT|ATTACHMENT|AI_SUMMARY"
        Boolean isRead
        Date createdAt
    }

    followups {
        ObjectId _id PK
        ObjectId consultationId FK
        ObjectId patientId FK
        String type "72h|7d"
        String status "PENDING|COMPLETED|MISSED"
        Number symptomSeverity
        Number baselineSeverity
        String notes
        Date scheduledFor
        Date submittedAt
        Date createdAt
    }

    knowledge_chunks {
        ObjectId _id PK
        String title
        String content
        String specialty
        String source
        Array embedding "768 dims · gemini-embedding-001"
        Array tags
        Date createdAt
        Date updatedAt
    }

    rag_traces {
        ObjectId _id PK
        ObjectId sessionId FK
        ObjectId patientId FK
        String query
        Boolean cacheHit
        Number chunksRetrieved
        Number vectorScore
        Number responseMs
        Number feedbackRating
        Boolean guardrailTripped
        Date createdAt
    }

    outbox_events {
        ObjectId _id PK
        String eventType
        String aggregateId
        String aggregateType
        Object payload
        String status "PENDING|PROCESSING|PUBLISHED|FAILED"
        Number retryCount
        Number maxRetries "5"
        Number visibilityTimeoutSec "30"
        String errorMessage
        Date processedAt
        Date createdAt
        Date updatedAt
    }

    patients ||--o{ triage_sessions : "realiza"
    triage_sessions ||--o| consultations : "genera"
    consultations ||--o{ consult_messages : "contiene"
    consultations ||--o{ followups : "crea post-cierre"
    consultations ||--o{ outbox_events : "publica via outbox"
    triage_sessions ||--o{ rag_traces : "genera trazas RAG"
```

---

## 9. Pipeline CI/CD — GitHub Actions → AWS ECS

```mermaid
flowchart LR
    subgraph GIT["Estrategia Git — GitFlow"]
        Main["main\nProducción"]
        Feat["feat/xxx\nfeature branches"]
        Fix["fix/xxx\nbugfixes"]
        Chore["chore/xxx\ninfra/config"]
        Feat -->|PR| Main
        Fix  -->|PR| Main
        Chore-->|PR| Main
    end

    subgraph CI["CI — IMPLEMENTADO ✅ (GitHub Actions)"]
        Push["1 git push\nfeature branch\nPull Request"]
        Lint["2 Lint\nESLint autofix\nPrettier · TS check"]
        Unit["3 Unit Tests\nJest · Node 20\nAI_ENABLED=false\nAiService mocked\nCoverage ≥80% BE"]
        E2E["4 E2E Tests\nSupertest · Playwright\nmongodb-memory-server\nNo DB mock\nRedis 7 service"]
        Build["5 Build Image\ndocker build multi-stage\nAlpine · non-root USER\npush → ECR"]
    end

    subgraph CD_ROADMAP["CD — ROADMAP 🔲 (no implementado aún)"]
        Security["6 Security Scan\nTrivy · OWASP\nSecrets detect\n[ROADMAP]"]
        Staging["7 Staging Deploy\nECS task update\ndevelop branch\nSmoke tests\n[ROADMAP]"]
        Approval["8 ✋ Manual Approval\nGitHub Review Gate\nmain branch PR\n[ROADMAP]"]
        Prod["9 PROD Deploy\nECS Blue/Green\nRolling update\nHealth check wait\n[ROADMAP]"]
    end

    Push --> Lint --> Unit --> E2E --> Build
    Build --> Security --> Staging --> Approval --> Prod

    style Security fill:#9CA3AF
    style Staging fill:#9CA3AF
    style Approval fill:#9CA3AF
    style Prod fill:#9CA3AF
```

---

## 10. Stack de Observabilidad — OpenTelemetry

```mermaid
graph TB
    subgraph APP["NestJS API — OpenTelemetry NodeSDK (OTEL_ENABLED=true para activar)"]
        HTTP_I["HTTP Instrumentation\n@opentelemetry/instrumentation-http"]
        Mongo_I["Mongoose Instrumentation\nMongoDB queries · duration"]
        Redis_I["ioredis Instrumentation\nRedis cmds · cache hit/miss"]
        Log_I["LoggingInterceptor\nStructured JSON · CorrelationId\nPII masked antes de log"]
        AI_I["AI Tracer\nRAG pipeline spans\ncache hit · chunks · ms"]
        RAG_I["rag_traces collection\nMongoDB persist\nHuman feedback loop"]
    end

    subgraph COLLECTOR["OpenTelemetry Collector — OTLP HTTP :4318"]
        C1["Recibe spans · batches metrics\nenv: OTEL_EXPORTER_OTLP_ENDPOINT"]
    end

    subgraph LOGS["📋 LOGS"]
        L1["Formato JSON:\ntimestamp · level\ncorrelationId · userId\nmethod · path\nstatusCode · durationMs\nerror.stack (si error)"]
        L2["Destinos:\nstdout → CloudWatch\nLogtail / Axiom (roadmap)"]
        L3["PII masked · Retention: 30d"]
    end

    subgraph METRICS["📊 MÉTRICAS"]
        M1["Técnicas:\nhttp_request_duration_ms\nhttp_requests_total\ndb_query_duration_ms\nredis_cache_hit_ratio\nbullmq_queue_depth"]
        M2["Negocio (Dashboard):\nactive_consultations\ntriage_completed_today\nrevenue_total_cop\nai_rag_cache_hit_rate\nai_avg_response_ms"]
    end

    subgraph TRACES["🔍 TRAZAS"]
        T1["Span hierarchy ejemplo:\nPOST /v1/triage/analyze\n├─ JwtAuthGuard.verify\n├─ ThrottlerGuard.check\n├─ MongoDB.findOne\n├─ rag.embed (768d)\n├─ rag.vectorSearch\n├─ gemini.generate\n└─ outbox.publish"]
        T2["W3C TraceContext header\ntraceId + spanId\nJaeger UI · 7d retention"]
    end

    subgraph AI_AUDIT["🤖 AI AUDIT"]
        A1["RagTrace doc:\nquery · cacheHit\nchunksRetrieved · vectorScore\nresponseMs · feedbackRating\nguardrailTripped"]
        A2["Guardrails:\nClinical domain filter\nPII detector (regex)\nNo diagnose · No prescribe\nConfidence threshold"]
    end

    subgraph ALERTS["🚨 ALERTAS — umbrales wiki 09-Observabilidad-KPIs.md"]
        AL1["🔴 CRÍTICO:\nError Rate > 2% por 5min\nP95 Latency > 1500ms por 10min"]
        AL2["🟡 ADVERTENCIA:\nAI Summary > 15s × 3 consecutivos\nDisponibilidad < 99%"]
        AL3["🟣 AI:\nRAG cache hit < 40%\nOutbox failure spike\nBullMQ dead letter"]
        AL4["✅ Compliance:\nLey 1581 audit log\nAcceso datos · 90d retention"]
    end

    HTTP_I --> C1
    Mongo_I --> C1
    Redis_I --> C1
    Log_I --> LOGS
    AI_I --> C1
    RAG_I --> AI_AUDIT
    C1 --> TRACES
    C1 --> METRICS
```

---

## Instrucciones para visualización local

### VS Code
1. Instalar extensión **Bierner.markdown-mermaid** (Markdown Preview Mermaid Support)
2. Abrir este archivo → `Ctrl+Shift+V` para preview

### Mermaid Live Editor
- Pegar cualquier bloque en [mermaid.live](https://mermaid.live)

### GitHub
- Los bloques ` ```mermaid ``` ` se renderizan automáticamente en GitHub

### CLI (exportar a PNG/SVG)
```bash
npm install -g @mermaid-js/mermaid-cli
mmdc -i DIAGRAMAS-MERMAID.md -o output/ -f svg
```

---

*Generado: 2026-05-19 | Versión corregida post-auditoría arquitectónica*
