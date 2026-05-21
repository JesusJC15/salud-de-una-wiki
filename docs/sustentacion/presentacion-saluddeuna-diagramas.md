# SaludDeUna — Diagramas para la Presentación

> Todos los diagramas están en Mermaid — renderizables en GitHub, VS Code y mermaid.live.
> Para exportar a SVG/PNG: `mmdc -i DIAGRAMAS-MERMAID.md -o output/ -f svg`
> Los diagramas C4 completos están en: `diagrams/DIAGRAMAS-MERMAID.md`

---

## DÓNDE USAR CADA DIAGRAMA

| Slide | Diagrama | Fuente |
|-------|----------|--------|
| 10 | Flujo E2E del paciente | Este archivo §1 |
| 11 | C4 Nivel 1 — Contexto | DIAGRAMAS-MERMAID.md §1 |
| 12 | C4 Nivel 2 — Contenedores | DIAGRAMAS-MERMAID.md §2 |
| 13 | Módulos backend NestJS | DIAGRAMAS-MERMAID.md §3 |
| 14 | Pipeline de triage IA | Este archivo §2 |
| 16 | Flujo de autenticación | DIAGRAMAS-MERMAID.md §6 |
| 18 | Stack observabilidad | DIAGRAMAS-MERMAID.md §10 |
| 19 | Pipeline RAG | DIAGRAMAS-MERMAID.md §5 |
| 20 | Guardrail clínico | Este archivo §3 |

---

## §1 — Flujo E2E del Paciente (para Slide 10)

```mermaid
flowchart LR
    P(["👤 Paciente\nLogin"])

    subgraph T["📋 TRIAGE IA"]
        T1["Selecciona\nespecialidad"]
        T2["Responde\ncuestionario\n(5–8 preguntas)"]
        T3["Gemini\nanáliza +\nred-flags"]
        T4{"Prioridad"}
    end

    subgraph B["💳 BILLING"]
        B1["Checkout\nsimulado"]
        B2["Consulta\ncreada\nWAITING"]
    end

    subgraph C["🩺 CONSULTA"]
        C1["Cola\nordenada\npor prioridad"]
        C2["Doctor\nasigna"]
        C3["Chat\nSocket.IO\ntiempo real"]
        C4["Resumen IA\nGemini +\nGuardrail"]
        C5["Doctor\ncierra"]
    end

    subgraph F["🔄 FOLLOWUP"]
        F1["72h + 7d\nBullMQ"]
        F2["Push\nnotif."]
        F3["Paciente\nresponde"]
        F4{"¿Empeoró?"}
        F5["✅ Completo"]
        F6["🚨 Nueva\nconsulta\nALTA"]
    end

    P --> T1 --> T2 --> T3 --> T4
    T4 -- "LOW/MOD/HIGH" --> B1 --> B2
    B2 --> C1 --> C2 --> C3 --> C4 --> C5
    C5 --> F1 --> F2 --> F3 --> F4
    F4 -- "No" --> F5
    F4 -- "Sí (+2pts)" --> F6 -.->|"Re-ingresa\na cola"| C1

    style T4 fill:#F59E0B,color:#000
    style F4 fill:#F59E0B,color:#000
    style F6 fill:#DC2626,color:#fff
    style F5 fill:#16A34A,color:#fff
    style C3 fill:#1D4ED8,color:#fff
```

---

## §2 — Pipeline de Triage IA (para Slide 14)

```mermaid
flowchart TD
    A["Paciente completa\ncuestionario\n(5–8 respuestas)"]

    subgraph AI["⚙️ MOTOR IA — TriageModule"]
        B["Reglas deterministas\nred-flags-mg.json\n(siempre activas)"]
        C["Gemini 2.5-flash\nPrompt por especialidad\n+ respuestas del paciente"]
        D["GuardrailService\nguardrail-rules.json"]
    end

    E{"Guardrail\npasa?"}
    F["✅ aiSummary\npersistido\n(resumen neutral\nde urgencia)"]
    G["❌ aiSummary = null\nguardrailApplied = true\nlog WARN + correlationId"]

    H["Resultado final\nprioridad: LOW/MOD/HIGH\nredFlags: []\naiSummary: '...' o null"]

    A --> B
    A --> C
    B --> D
    C --> D
    D --> E
    E -- "safe=true" --> F --> H
    E -- "safe=false" --> G --> H

    note1["Si Gemini falla:\n→ analysisMode=RULE_BASED\n→ solo reglas deterministas\n→ sin interrupción para el usuario"]
    C -. "falla" .-> note1 --> H

    style E fill:#F59E0B,color:#000
    style F fill:#16A34A,color:#fff
    style G fill:#DC2626,color:#fff
    style D fill:#7C3AED,color:#fff
```

---

## §3 — Guardrail Clínico (para Slide 20)

```mermaid
flowchart LR
    A["Gemini 2.5-flash\ngenera texto"]

    subgraph GR["🛡️ GuardrailService"]
        B["Evalúa contra\nguardrail-rules.json"]
        C["Categorías bloqueadas:\n✗ diagnóstico\n✗ prescripción\n✗ afirmación clínica"]
    end

    D{"safe?"}
    E["✅ safe = true\naiSummary = texto\n(resumen neutral)"]
    F["🚫 safe = false\naiSummary = null\nguardrailApplied = true\nlog WARN estructurado"]

    A --> B --> C --> D
    D -- "Sí" --> E
    D -- "No" --> F

    style D fill:#F59E0B,color:#000
    style E fill:#16A34A,color:#fff
    style F fill:#DC2626,color:#fff
    style GR fill:#1E293B,color:#fff
```

---

## §4 — Criterios de Calidad (para Slide 15 — alternativa visual)

```mermaid
mindmap
  root((Calidad\nSaludDeUna))
    Seguridad
      JWT HS256 + Auth0 RS256
      RBAC PATIENT/DOCTOR/ADMIN
      Rate limiting 20req/60s Redis
      bcrypt cost 12
      Max 3 sesiones activas
    Confiabilidad
      406+ tests pasando
      93% statements
      80% branches mínimo
      E2E con mongodb-memory-server
    Observabilidad
      Logs JSON + correlationId
      p95 latencia en dashboard
      OpenTelemetry OTLP listo
      AI Audit - RagTrace docs
    Mantenibilidad
      NestJS modular 17 módulos
      DI sin acoplamiento directo
      TypeScript estricto
      Sin barrel exports
    Rendimiento
      p95 < 1500ms SLO
      Redis cache RAG 300s
      BullMQ async jobs
    Disponibilidad
      /v1/health + /v1/ready
      Fallback RULE_BASED si IA falla
      Outbox retry x5 backoff
```

---

## §5 — Arquitectura de Despliegue Simplificada (para presentación)

```mermaid
graph TD
    Internet["🌐 Usuarios"]

    subgraph CLIENTS["Clientes"]
        Mobile["📱 Expo Mobile\nReact Native\niOS / Android"]
        Web["🖥️ Next.js Web\nDoctor + Admin\nVercel"]
    end

    subgraph BACKEND["Backend — Railway"]
        API["NestJS API\n:3000 · /v1\n2 instancias"]
        Worker["NestJS Worker\nBullMQ · Outbox\n1 instancia"]
    end

    subgraph DATA["Datos"]
        Mongo["MongoDB Atlas\nDatos + Vector Search\n768d HNSW"]
        Redis["Redis\nCache · BullMQ\nSocket.IO adapter"]
    end

    subgraph EXT["Servicios externos"]
        Gemini["Google Gemini\n2.5-flash · embedding-001"]
        Auth0["Auth0\nJWT RS256"]
        Expo["Expo Push\nFCM / APNs"]
    end

    Internet --> Mobile
    Internet --> Web
    Mobile -->|"HTTPS / WSS"| API
    Web -->|"HTTPS / WSS"| API
    API --> Mongo
    API --> Redis
    Worker --> Mongo
    Worker --> Redis
    API -->|"triage · RAG · resumen"| Gemini
    API -->|"valida tokens"| Auth0
    Worker -->|"push notifications"| Expo

    style API fill:#1D4ED8,color:#fff
    style Worker fill:#7C3AED,color:#fff
    style Mongo fill:#00684A,color:#fff
    style Redis fill:#DC2626,color:#fff
    style Gemini fill:#1A73E8,color:#fff
```

---

## §6 — Flujo de Chat en Tiempo Real (para explicar durante el demo)

```mermaid
sequenceDiagram
    participant P as 📱 Paciente<br/>(Mobile)
    participant API as 🔌 ChatGateway<br/>(Socket.IO)
    participant DB as 🗄️ MongoDB<br/>(consultation_messages)
    participant D as 🖥️ Doctor<br/>(Web)

    Note over P,D: Ambos conectados a la sala "consultation:{id}"

    P->>API: emit('joinRoom', {consultationId})
    D->>API: emit('joinRoom', {consultationId})

    P->>API: emit('sendMessage', {content: "Tengo dolor..."})
    API->>DB: insert consultationMessage doc
    DB-->>API: saved
    API-->>D: emit('receiveMessage', {sender: 'PATIENT', content, sentAt})
    API-->>P: emit('receiveMessage', {ack: true})

    D->>API: emit('sendMessage', {content: "Cuénteme más..."})
    API->>DB: insert consultationMessage doc
    API-->>P: emit('receiveMessage', {sender: 'DOCTOR', content, sentAt})
    API-->>D: emit('receiveMessage', {ack: true})

    Note over P,D: Mensajes persistidos · Historial disponible<br/>para resumen IA al cerrar la consulta
```

---

## Instrucciones para exportar a SVG (presentación)

```bash
# Instalar CLI de Mermaid
npm install -g @mermaid-js/mermaid-cli

# Exportar todos los diagramas de DIAGRAMAS-MERMAID.md
mmdc -i salud-de-una-wiki/diagrams/DIAGRAMAS-MERMAID.md \
     -o salud-de-una-wiki/docs/sustentacion/output/ \
     -f svg

# Exportar los diagramas de este archivo
mmdc -i salud-de-una-wiki/docs/sustentacion/presentacion-saluddeuna-diagramas.md \
     -o salud-de-una-wiki/docs/sustentacion/output/ \
     -f svg
```

Los SVGs exportados se pueden insertar directamente en Canva, Google Slides o PowerPoint.

---

## Recomendaciones de uso en la presentación

| Diagrama | Recomendación visual |
|----------|----------------------|
| C4 L1 y L2 | Exportar SVG, insertar en slide con fondo oscuro, leyenda al pie |
| Flujo E2E paciente | Usar versión simplificada (este §1) — no el de 30 nodos |
| Pipeline triage IA | Animación slide-by-slide si la herramienta lo permite |
| Guardrail | Simple y directo — el punto clave es el bloqueo en rojo |
| Módulos backend | Exportar del §3 de DIAGRAMAS-MERMAID.md como SVG |
| Mindmap calidad | Solo si la herramienta renderiza mindmaps — alternativa: tabla de 6 tarjetas |
