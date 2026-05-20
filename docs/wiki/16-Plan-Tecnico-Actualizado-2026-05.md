# Plan Técnico Actualizado — SaludDeUna

**Versión:** 2.1  
**Fecha de auditoría:** 2026-05-06 | **Última actualización:** 2026-05-09  
**Rol:** Arquitecto Senior + Tech Lead  
**Reemplaza:** plan anterior (2026-05-03)  
**Estado:** Documento maestro de continuidad técnica

---

## 0. Metodología de esta Auditoría

Este documento fue construido mediante:

1. Lectura directa de todos los archivos del monorepo (backend, web, mobile, wiki)
2. Contraste con el plan anterior (2026-05-03)
3. Entrevista estructurada al responsable técnico del proyecto
4. Análisis de gaps entre lo planeado y lo implementado
5. Validación de CI/CD, infraestructura, testing y configuración

**Principio rector:** Ninguna conclusión se basa en el plan anterior. Todo se valida contra el código real.

---

## 1. Resumen Ejecutivo

SaludDeUna es una plataforma de comunicación clínica inteligente que conecta pacientes con médicos mediante triage guiado por IA, chat en tiempo real y seguimiento post-consulta. El proyecto pasó de fase académica a modo **producto/startup** y requiere despliegue en producción en un plazo urgente de **menos de 1 mes**.

### Estado actual (2026-05-06)

| Dimensión | Estado | Nota |
|-----------|--------|------|
| Funcionalidad core (triage → consulta → chat → followup) | ✅ Completa | Flujo E2E implementado y conectado |
| Dashboard + 4 KPIs de negocio | ✅ Completa | Datos reales de BD |
| Admin console web | ✅ Implementada | API conectada; posibles bugs de UX |
| Auth0 integración (web) | ✅ Funcional | Provisioning, claims, guards OK |
| Auth0 integración (mobile) | ⚠️ Incompleta | Servicio existe pero flujo PKCE sin validar en dispositivo |
| Socket.io WebSocket (chat) | ⚠️ Single-instance | Sin Redis adapter — bloqueante para escalar |
| CI/CD | ⚠️ Parcial | Solo SonarQube; sin pipeline de deploy |
| Infraestructura cloud | ⚠️ Parcial | MongoDB Atlas + Vercel listos; backend sin configurar |
| EAS Build (mobile) | ❌ No configurado | Necesario para builds de producción Expo |
| Usuarios reales en producción | ❌ Sin usuarios | Solo datos de prueba |

### Diagnóstico en una línea

> El producto está **funcionalmente completo** (~95% de features implementadas). El cuello de botella es la **preparación para producción** (infraestructura, deploy pipeline, Socket.io scaling, validación Auth0 mobile) — no el desarrollo de nuevas features.

---

## 2. Arquitectura Actual Validada

### 2.1 Stack tecnológico

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENTES                                │
│                                                                 │
│  ┌─────────────────────┐    ┌─────────────────────────────────┐ │
│  │   Mobile (Expo 55)  │    │       Web (Next.js 16)          │ │
│  │   React Native 0.85 │    │       React 19 / App Router     │ │
│  │   Expo Router       │    │       Auth0 SPA SDK             │ │
│  │   Zustand + RQ      │    │       TanStack Query + RHF      │ │
│  │   expo-auth-session │    │       Tailwind 4 + shadcn/ui    │ │
│  └──────────┬──────────┘    └────────────────┬────────────────┘ │
│             │                                │                  │
└─────────────┼────────────────────────────────┼──────────────────┘
              │ HTTPS / WebSocket              │ HTTPS / WebSocket
              ▼                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BACKEND (NestJS 11)                          │
│                    Global prefix: /v1                           │
│                                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐   │
│  │  auth/   │ │ triage/  │ │  chat/   │ │  consultations/  │   │
│  │  (JWT +  │ │  (AI +   │ │ (Socket  │ │  (queue, assign, │   │
│  │  Auth0)  │ │ red-flag)│ │   .io)   │ │   close, rate)   │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐   │
│  │followups/│ │patients/ │ │ doctors/ │ │   dashboard/     │   │
│  │(BullMQ)  │ │(timeline)│ │(rethus)  │ │  (KPIs + métr.)  │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐   │
│  │ outbox/  │ │   ai/    │ │  admin/  │ │  notifications/  │   │
│  │(transact)│ │(Gemini)  │ │(console) │ │  (push + in-app) │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘   │
│  ┌──────────┐ ┌──────────┐                                      │
│  │  redis/  │ │ common/  │                                      │
│  │(throttle)│ │(guards,  │                                      │
│  │          │ │ filters, │                                      │
│  │          │ │intercept)│                                      │
│  └──────────┘ └──────────┘                                      │
└─────────────────────────┬───────────────────────────────────────┘
                          │
           ┌──────────────┼──────────────────┐
           ▼              ▼                  ▼
     MongoDB Atlas      Redis           Auth0 Tenant
     (Mongoose ODM)    (BullMQ +        (JWT issuance,
     15 schemas        throttler)        provisioning,
                                         social login)
           │
           ▼
     Google Gemini 2.5-flash
     (triage AI + clinical summary)
```

### 2.2 Patrones arquitectónicos en uso

| Patrón | Módulo | Estado |
|--------|--------|--------|
| Transactional Outbox | `outbox/` | ✅ Implementado |
| Domain Events | `outbox/domain-events-handler.service.ts` | ✅ Implementado |
| Repository Pattern | `triage/questions/triage-questions.repository.ts` | ✅ Implementado |
| Strategy Pattern | `auth/strategies/` (jwt, jwt-legacy, jwt-provision) | ✅ Implementado |
| Dual Store (in-memory / Redis) | `dashboard/metrics/` | ✅ Implementado |
| Global Guards + Decorators | `common/guards/`, `common/decorators/` | ✅ Implementado |
| Correlation ID propagation | `common/interceptors/request-logging.interceptor.ts` | ✅ Implementado |
| BullMQ async job processing | `followups/followups.processor.ts` | ✅ Implementado |

### 2.3 Flujo crítico validado: Triage → Consulta → Followup

```
[Mobile] Paciente inicia triage
    → POST /v1/triage/sessions                  (crear sesión)
    → PUT  /v1/triage/sessions/:id/answers      (responder preguntas)
    → POST /v1/triage/sessions/:id/analyze      (Gemini analiza → red flags)
    → POST /v1/consultations                    (crear consulta)

[Web] Doctor asigna consulta
    → PATCH /v1/consultations/:id/assign
    → Socket.io: sala 'consultation:${id}' creada

[Mobile+Web] Chat en tiempo real
    → Socket.io: eventos 'sendMessage', 'receiveMessage'
    → Mensajes persistidos en MongoDB (consultation-message schema)

[Web] Doctor cierra consulta
    → PATCH /v1/consultations/:id/close
    → outboxService.createConsultationClosedEvent()          ← outbox
    → domain-events-handler: CONSULTATION_CLOSED_EVENT
    → followupsService.handleConsultationClosedEvent()
    → Crea 2 Followup docs: {type: '72h', scheduledFor: now+72h}
                            {type: '7d',  scheduledFor: now+7d}

[Scheduler - BullMQ] En el momento programado
    → Job 'followup-reminder' se activa
    → push-notifications.service.ts: Expo push token → notificación

[Mobile] Paciente responde seguimiento
    → POST /v1/followups/:id/submit
    → Si symptomSeverity - baseline >= 2: crear nueva consulta ALTA prioridad
```

---

## 3. Estado Detallado por Módulo/Feature

### 3.1 Backend — Inventario completo

| Módulo | Controller | Service | Schema | Tests | Estado | Observaciones |
|--------|-----------|---------|--------|-------|--------|---------------|
| `auth` | ✅ | ✅ | ✅ | ✅ | Completo | JWT + Auth0 provisioning |
| `triage` | ✅ | ✅ | ✅ | ✅ | Completo | Gemini, red-flags, guardrails |
| `consultations` | ✅ | ✅ | ✅ | ✅ | Completo | Queue, assign, close, rate, feedback |
| `chat` | ✅ | ✅ | ✅ | ✅ | Completo | Socket.io (single-instance — ver gap) |
| `followups` | ✅ | ✅ | ✅ | ✅ | Completo | BullMQ, auto-creation, escalación |
| `patients` | ✅ | ✅ | ✅ | ✅ | Completo | Timeline implementado |
| `doctors` | ✅ | ✅ | ✅ | ✅ | Completo | RETHUS, availability, push token |
| `admin` | ✅ | ✅ | — | ✅ | Completo | CSV export, REThUS review |
| `dashboard` | ✅ | ✅ | — | ✅ | Completo | 4 KPIs con datos reales |
| `notifications` | ✅ | ✅ | ✅ | ✅ | Completo | Push + in-app |
| `outbox` | ✅ | ✅ | ✅ | ✅ | Completo | Transactional outbox pattern |
| `ai` | ✅ | ✅ | ✅ | ✅ | Completo | Gemini, prompts versionados, audit log |
| `admins` | — | ✅ | ✅ | ✅ | Completo | Seeder inicial |
| `billing` | ✅ | ✅ | ✅ | ✅ | Completo | Checkout simulado, BillingPrice seeder, Transaction lifecycle, revenue metrics, admin PATCH price |
| `redis` | — | ✅ | — | ✅ | Completo | Health, throttler, lifecycle |
| `common` | — | — | — | ✅ | Completo | Guards, filters, interceptors, decorators |

**Cobertura de tests backend:** 406+ tests pasando, 0 fallando (billing.service.spec.ts añadido)  
**Cobertura global:** Statements 93.11% | Branches 80.15% | Functions 94.33% | Lines 93.03%  
**Todos los umbrales ≥ 80%**: ✅

### 3.2 Web — Inventario completo

| Feature | Pages | Hooks | Service | Tests | Estado |
|---------|-------|-------|---------|-------|--------|
| `auth` | ✅ login, register | ✅ | ✅ | ✅ | Completo |
| `doctor-queue` | ✅ queue, consultation, history | ✅ (10 hooks) | ✅ | ✅ | Completo |
| `admin-console` | ✅ users, doctors, verify, reports, **billing** | ✅ (use-admin-billing) | ✅ (admin-service) | ⚠️ | Funcional, validar UX |
| `admin-home` | ✅ KPIs + revenue + AI metrics | ✅ (use-dashboard-metrics, use-revenue-metrics, use-ai-metrics) | — | — | Completo |
| `dashboard` | ✅ | — | — | — | Completo |
| `doctor-home` | ✅ | — | — | — | Completo |

**Rutas duplicadas detectadas:** Existe `app/dashboard/` y `app/(dashboard)/` simultáneamente → refactoring pendiente.

**Admin billing** (`src/app/(dashboard)/admin/billing/page.tsx`): página de facturación con tabla de precios por especialidad (editable in-line) y listado de transacciones paginado.

**Nuevos tipos** (`src/types/admin.ts`): `BillingPrice`, `BillingTransaction`, `RevenueMetrics`, `ListTransactionsResponse`.

**Cobertura web:** 82 tests pasando | Statements 85.71% | Branches 73.55% | Functions 85.91% | Lines 85.71%  
⚠️ **Branches 73.55% < 80%** — por debajo del umbral. Requiere atención.

### 3.3 Mobile — Inventario completo

| Feature | Screen | Hooks/Service | Estado | Observaciones |
|---------|--------|---------------|--------|---------------|
| `patient-auth` | ✅ login, register | ✅ | Funcional | Auth0 no validado en dispositivo |
| `patient-triage` | ✅ specialty, session, result | ✅ | Completo | |
| `patient-chat` | ✅ | ✅ | Completo | |
| `patient-consultations` | ✅ history | ✅ | Completo | |
| `patient-followup` | ✅ `[followupId].tsx` | ✅ | Completo | |
| `patient-notifications` | ✅ | ✅ | Completo | |
| `patient-profile` | ✅ | — | Completo | |
| `patient-timeline` | — | ✅ | Parcial | Hook existe, falta pantalla dedicada |
| Auth0 PKCE | — | `auth0-service.ts` | ⚠️ Sin validar | Flujo principal usa JWT legacy |

---

## 4. Problemas Críticos Detectados

### 4.1 [CRÍTICO - Bloqueante para producción] Socket.io sin Redis Adapter

**Archivo:** `src/chat/chat.gateway.ts`  
**Descripción:** El gateway WebSocket usa broadcast local (`this.server.to(room).emit()`). En un entorno de múltiples instancias (horizontal scaling en Railway/Cloud Run), los mensajes no se distribuyen entre pods. Paciente y médico podrían conectarse a instancias distintas y no recibir mensajes del otro.  
**Impacto:** Chat completamente roto en producción con más de 1 instancia.  
**Solución:** Agregar `@socket.io/redis-adapter` con la instancia Redis ya existente.  
**Esfuerzo estimado:** 3-4 horas.

### 4.2 [CRÍTICO - Bloqueante para producción] Sin pipeline de deploy en CI/CD

**Descripción:** Solo existe `.github/workflows/sonarqube.yml` que corre tests + SonarQube. No hay ningún workflow que construya la imagen Docker del backend y la despliegue en Railway/AWS, ni que despliegue el web en Vercel, ni que inicie un EAS Build para mobile.  
**Impacto:** Sin CI/CD de deploy, cada actualización requiere operación manual. Inaceptable para producción.  
**Solución:** Crear workflows de deploy para cada servicio.  
**Esfuerzo estimado:** 6-8 horas.

### 4.3 [ALTO] Auth0 Mobile sin validar en dispositivo real

**Archivo:** `salud-de-una-mobile/src/services/auth/auth0-service.ts`  
**Descripción:** El servicio Auth0 existe y tiene funciones correctas (`refreshAuth0Session`, `provisionPatient`) pero no está claro cómo se conecta con el flujo principal de login. El app podría estar usando JWT legacy sin activar Auth0 PKCE.  
**Impacto:** Login social (Google/Apple) no disponible en mobile. Si el backend solo acepta tokens Auth0 en el futuro, el mobile quedaría roto.  
**Solución:** Conectar `auth0-service.ts` al flujo de login en `patient-auth.ts`, probar en simulador iOS y Android.  
**Esfuerzo estimado:** 8-12 horas.

### 4.4 [ALTO] EAS Build no configurado

**Descripción:** No existe `eas.json` en el proyecto mobile. Sin EAS Build, no es posible generar builds de producción firmadas para distribución en App Store o Google Play, ni builds de prueba para TestFlight/Firebase App Distribution.  
**Impacto:** Mobile no puede llegar a usuarios reales.  
**Solución:** Inicializar EAS, crear perfiles `development`, `preview` y `production`.  
**Esfuerzo estimado:** 3-4 horas.

### 4.5 [MEDIO] Rutas duplicadas en web (`dashboard/` vs `(dashboard)/`)

**Descripción:** Existe tanto `app/dashboard/` como `app/(dashboard)/` en Next.js, lo cual puede generar conflictos de routing en producción con App Router.  
**Impacto:** Posibles redirecciones incorrectas, errores 404 o comportamiento inesperado.  
**Solución:** Eliminar `app/dashboard/` (directorio plano) y consolidar todo en `app/(dashboard)/`.  
**Esfuerzo estimado:** 1-2 horas.

### 4.6 [MEDIO] Cobertura de branches en web por debajo del umbral

**Descripción:** Branches coverage en web: 73.55% (umbral mínimo: 80%).  
**Impacto:** CI podría fallar si se habilita el umbral de branches en la configuración de Jest de la web.  
**Solución:** Agregar tests para cubrir ramas condicionales no cubiertas.  
**Esfuerzo estimado:** 4-6 horas.

### 4.7 [BAJO] Variables CORS hardcodeadas a localhost en `.env.development.example`

**Descripción:** `CORS_ORIGINS_PATIENT=http://localhost:5173,http://localhost:8081`. En producción estas deben apuntar a los dominios reales.  
**Impacto:** CORS bloqueará requests en producción si no se configuran correctamente.  
**Solución:** Configurar estas variables en el Secret Manager/Railway env antes del deploy.

---

## 5. Deuda Técnica Priorizada

| # | Item | Severidad | Esfuerzo | Cuándo atacar |
|---|------|-----------|----------|---------------|
| 1 | Socket.io Redis adapter | CRÍTICA | 4h | Sprint 1 — antes del deploy |
| 2 | Deploy CI/CD pipeline | CRÍTICA | 8h | Sprint 1 |
| 3 | Auth0 mobile validado E2E | ALTA | 12h | Sprint 1 |
| 4 | EAS Build configuración | ALTA | 4h | Sprint 1 |
| 5 | Branches coverage web < 80% | MEDIA | 6h | Sprint 1 / Sprint 2 |
| 6 | Rutas duplicadas `dashboard/` | MEDIA | 2h | Sprint 1 |
| 7 | `triage/rules/` — reglas hardcoded | MEDIA | 8h | Sprint 2 |
| 8 | Especialidades limitadas (solo MG + Odontología) | MEDIA | 16h | Sprint 2 |
| 9 | Prompts IA únicos (no especializados por especialidad) | BAJA | 8h | Sprint 2 |
| 10 | Knowledge base / RAG infraestructura | BAJA | 20h | Sprint 3 |
| 11 | ~~Monetización simulada (checkout)~~ | ~~BAJA~~ | ~~24h~~ | ✅ **COMPLETADO 2026-05-09** — billing/ implementado con checkout, transacciones y admin price management |
| 12 | OpenTelemetry / observabilidad distribuida | BAJA | 16h | Sprint 3 |

---

## 6. Riesgos Técnicos

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|---------|------------|
| Chat roto en producción multi-instancia (sin Redis adapter) | Alta | Crítico | Implementar Redis adapter antes del deploy |
| Auth0 PKCE falla en mobile en producción | Media | Alto | Probar en simulador + dispositivo físico antes del release |
| MongoDB Atlas IP whitelist bloquea backend en Railway | Alta | Alto | Configurar IP estática en Railway o `Allow access from anywhere` (temporal) |
| EAS Build falla por credenciales de firma no configuradas | Media | Alto | Inicializar EAS con perfiles correctos y certificados |
| CORS bloqueando requests de dominios de producción | Alta | Alto | Configurar CORS_ORIGINS en Railway env antes del primer request |
| Gemini API key agotada o rate-limit en producción | Baja | Alto | Configurar cuota y alertas en Google AI Studio |
| Redis Cloud plan gratuito insuficiente para BullMQ + throttler | Media | Medio | Usar Railway Redis addon o Redis Cloud plan de pago |
| Apple Sign In requiere Apple Developer Program ($99/año) | Alta | Medio | Lanzar con Google login primero; Apple como segunda iteración |
| Exposición de secrets en `.env` files commitados | Baja | Crítico | Verificar `.gitignore` — los `.env.*.example` no deben contener valores reales |

---

## 7. Decisiones Arquitectónicas Recomendadas

### 7.1 Backend en Railway (no GCP Cloud Run inicialmente)

**Decisión:** Deploy en Railway en vez de GCP Cloud Run para el sprint de producción urgente.

**Razones:**
- Menor tiempo de configuración (15 min vs 2-3 horas para GCP)
- Railway soporta Dockerfile existente sin cambios
- Railway Redis addon disponible como servicio gestionado
- Para el volumen actual (0 usuarios reales) Railway es más que suficiente
- Migración a GCP/AWS en Sprint 3 cuando haya usuarios reales y datos de carga

**Configuración Railway:**
```
Servicios a crear:
  - salud-de-una-api      (Dockerfile existente → deploy automático desde GitHub)
  - salud-de-una-redis    (Railway Redis addon)

Variables de entorno en Railway:
  NODE_ENV=production
  PORT=3000
  MONGODB_URI=${{MongoDB.MONGODB_URI}}      (variable de MongoDB Atlas)
  REDIS_URL=${{Redis.REDIS_URL}}             (variable de Railway Redis)
  JWT_SECRET=<64+ chars>
  JWT_REFRESH_SECRET=<64+ chars>
  GEMINI_API_KEY=<key>
  AI_ENABLED=true
  AI_PROVIDER=gemini
  GEMINI_MODEL=gemini-2.5-flash
  CORS_ORIGINS_PATIENT=https://app.saluddeuna.com,https://<mobile-origin>
  CORS_ORIGINS_STAFF=https://staff.saluddeuna.com
  ENABLE_BOOTSTRAP_ADMIN=false
  AUTH0_DOMAIN=salud-de-una.us.auth0.com
  AUTH0_AUDIENCE=https://api.salud-de-una.com
  OUTBOX_DISPATCH_INTERVAL_MS=5000
```

### 7.2 Web en Vercel (ya decidido — sin cambios)

**Configuración Vercel:**
```
Project settings:
  Root Directory: salud-de-una-web
  Framework: Next.js
  Build Command: npm run build
  Output Directory: .next

Environment Variables (Vercel dashboard):
  NEXT_PUBLIC_API_BASE_URL=https://api.saluddeuna.com
  NEXT_PUBLIC_AUTH0_DOMAIN=salud-de-una.us.auth0.com
  NEXT_PUBLIC_AUTH0_CLIENT_ID=<web-spa-client-id>
  NEXT_PUBLIC_AUTH0_AUDIENCE=https://api.salud-de-una.com
  NEXT_PUBLIC_AUTH0_REDIRECT_URI=https://staff.saluddeuna.com/callback
  NEXT_PUBLIC_ID_ENCRYPTION_KEY=<32+ chars>
```

### 7.3 Socket.io con Redis Adapter (antes del deploy)

**Decisión:** Implementar `@socket.io/redis-adapter` usando la instancia Redis de Railway.

```typescript
// chat.module.ts — agregar provider de adapter
// chat.gateway.ts — conectar adapter en afterInit()
import { createAdapter } from '@socket.io/redis-adapter';

@WebSocketGateway({ cors: { origin: '*' } })
export class ChatGateway implements OnGatewayInit {
  afterInit(server: Server) {
    const pubClient = redisClient;
    const subClient = pubClient.duplicate();
    server.adapter(createAdapter(pubClient, subClient));
  }
}
```

### 7.4 Auth0 Mobile — Flujo definitivo

**Decisión:** Implementar flujo Auth0 PKCE como flujo **principal** en mobile (no como fallback). Eliminar JWT legacy de mobile una vez validado.

**Secuencia de implementación:**
1. Crear Application tipo "Native" en Auth0 dashboard
2. Configurar `app.config.ts`: scheme `com.saluddeuna`, Allowed Callback URLs
3. Conectar `auth0-service.ts` → `use-patient-auth.ts` → pantalla de login
4. Probar en simulador iOS + Android
5. Validar provisioning automático para nuevos usuarios
6. Validar refresh token flow

---

## 8. Roadmap por Fases — Actualizado

### Sprint 1 — Producción (semanas 1-3, deadline: ~28 días)

**Objetivo:** Plataforma completa corriendo en producción con dominio real y CI/CD funcional.

**Prioridad absoluta — BLOQUEANTES:**

#### 1.1 Agregar Socket.io Redis Adapter

```
Archivos a modificar:
  - salud-de-una-backend/package.json: agregar @socket.io/redis-adapter
  - src/chat/chat.gateway.ts: conectar adapter en afterInit()
  - src/chat/chat.module.ts: inyectar cliente Redis al gateway

Test de validación:
  - Iniciar 2 instancias del backend localmente con mismo Redis
  - Verificar que mensajes de una instancia llegan a sockets de la otra
```

#### 1.2 Fix rutas duplicadas web

```
Eliminar: salud-de-una-web/app/dashboard/ (directorio plano, obsoleto)
Consolidar todo en: salud-de-una-web/app/(dashboard)/
```

#### 1.3 Validar y conectar Auth0 mobile

```
1. Crear Auth0 Application "Native" para mobile
2. Conectar auth0-service.ts al flujo de login en use-patient-auth.ts
3. Actualizar app/(auth)/login.tsx con botones Google / Email via Auth0
4. Probar en simulador iOS y Android
5. Validar que provisioning funciona para usuario nuevo
6. Validar refresh token automático
```

#### 1.4 Deploy backend en Railway

```
Checklist:
  [ ] Crear proyecto Railway con servicio "salud-de-una-api"
  [ ] Conectar repositorio GitHub (rama main)
  [ ] Agregar Railway Redis addon
  [ ] Configurar todas las variables de entorno de producción
  [ ] Configurar MongoDB Atlas: whitelist IP de Railway (o 0.0.0.0/0 temporal)
  [ ] Verificar health check: GET /v1 → 200 OK
  [ ] Verificar Swagger deshabilitado (NODE_ENV=production)
  [ ] Configurar dominio personalizado: api.saluddeuna.com → Railway
```

#### 1.5 Deploy web en Vercel

```
Checklist:
  [ ] Importar proyecto salud-de-una-web en Vercel
  [ ] Configurar Root Directory: salud-de-una-web
  [ ] Configurar todas las variables de entorno de producción
  [ ] Verificar Auth0 Allowed Callback URLs incluye dominio Vercel
  [ ] Verificar Auth0 Allowed Logout URLs
  [ ] Probar flujo login → consulta en producción
  [ ] Configurar dominio personalizado: staff.saluddeuna.com
```

#### 1.6 CI/CD — Workflows de deploy

```yaml
# Crear: salud-de-una-backend/.github/workflows/deploy.yml
Trigger: push a main
Steps:
  1. Tests (npm run test:cov + npm run test:e2e)
  2. Docker build
  3. Railway deploy (via railway CLI o webhook de Railway)
  
# Crear: salud-de-una-web/.github/workflows/deploy.yml
Trigger: push a main
Steps:
  1. npm run check:types
  2. npm run test
  3. Vercel deploy (via vercel CLI o integración nativa GitHub)
```

#### 1.7 Configurar EAS Build

```
Archivos a crear:
  - salud-de-una-mobile/eas.json (perfiles: development, preview, production)

Pasos:
  1. eas init (asociar app en expo.dev)
  2. Configurar eas.json con perfiles
  3. eas build --platform android --profile preview (primer build de prueba)
  4. Verificar APK descargable y funcional
  5. Para iOS: configurar Apple Developer credentials
```

#### 1.8 Auth0 producción hardening

```
Checklist en Auth0 dashboard:
  [ ] Allowed Callback URLs: https://staff.saluddeuna.com/callback, com.saluddeuna://callback
  [ ] Allowed Logout URLs: https://staff.saluddeuna.com, https://app.saluddeuna.com
  [ ] Allowed Web Origins: https://staff.saluddeuna.com
  [ ] Action post-login (ID: 113e227d) en estado DEPLOYED
  [ ] CORS configurado para dominios de producción
  [ ] Google social connection habilitada
```

**Criterios de aceptación Sprint 1:**

- [ ] GET https://api.saluddeuna.com/v1 → 200 OK
- [ ] Flujo completo en web: login Admin/Doctor → cola → consulta → chat → close → dashboard KPIs
- [ ] Flujo completo en mobile: login → triage → consulta → chat → followup → notificación
- [ ] Chat funciona con 2 instancias del backend corriendo simultáneamente (Redis adapter)
- [ ] Push a `main` → deploy automático en Railway + Vercel
- [ ] EAS Build genera APK funcional en Android
- [ ] Swagger deshabilitado en producción (`/v1/docs` retorna 404)

---

### Sprint 2 — Calidad y Ampliación (semanas 4-8)

**Objetivo:** Elevar calidad de IA, ampliar especialidades y cubrir deuda técnica.

#### 2.1 Mejorar cobertura de branches en web (≥ 80%)

```
Objetivo: elevar branches de 73.55% a ≥ 80%
Archivos con menores branches: revisar con npm run test:cov en web
```

#### 2.2 Nuevas especialidades de triage

```
Especialidades a agregar: PEDIATRICS, GENERAL_EMERGENCY
Archivos:
  - src/common/enums/specialty.enum.ts
  - src/triage/questions/triage-questions.repository.ts
  - src/ai/ai-prompt-seeder.service.ts (prompts especializados)
  - src/triage/services/gemini-triage.service.ts
```

#### 2.3 Prompts IA especializados por especialidad

```
Estado actual: 1 prompt genérico
Objetivo: prompts específicos para MG, Odontología, Pediatría, Urgencias
Almacenamiento: AiPromptDefinition schema (ya existe)
Seed: ai-prompt-seeder.service.ts (ya existe)
```

#### 2.4 Reglas de triage dinámicas

```
Estado actual: triage/rules/ tiene guardrail-rules.json y red-flags-mg.json
Gap: solo medicina general tiene reglas de red-flags; las otras especialidades no
Objetivo: agregar red-flags-odontologia.json, red-flags-pediatria.json
```

#### 2.5 Sistema de auditoría IA (feedback médico)

```
Estado: campo summaryFeedback existe en schema y endpoint PATCH /consultations/:id/summary/feedback existe
Gap: UI en web (botones feedback debajo del resumen clínico)
Archivo: salud-de-una-web/src/features/doctor-queue/components/clinical-summary-panel.tsx
```

#### 2.6 Pantalla de timeline en mobile

```
Estado: usePatientTimeline hook existe; falta pantalla dedicada
Crear: salud-de-una-mobile/src/features/patient-timeline/patient-timeline-screen.tsx
Ruta: agregar tab o navegación desde patient-home
```

**Criterios de aceptación Sprint 2:**

- [ ] Branches coverage web ≥ 80%
- [ ] 4 especialidades disponibles en triage (MG, Odontología, Pediatría, Urgencias)
- [ ] Dashboard KPI 2 muestra % de feedback útil con datos reales de producción
- [ ] Pantalla timeline visible en mobile

---

### Sprint 3 — Escala y Observabilidad (semanas 9-16)

**Objetivo:** Preparar la plataforma para primeros usuarios reales con observabilidad completa.

#### 3.1 Observabilidad centralizada

```
Herramientas recomendadas (Railway-compatible):
  - Logging: Railway Logs + Logtail (o Axiom)
  - APM: Sentry (SDK para NestJS + Next.js + Expo)
  - Uptime: Better Uptime o Railway health checks

Implementación:
  - npm install @sentry/nestjs @sentry/nextjs @sentry/react-native
  - Configurar DSN en todas las apps
  - Agregar error boundary en web y mobile
  - Configurar alertas en Sentry para errores críticos
```

#### 3.2 Knowledge Base / RAG (infraestructura)

```
Módulo: src/knowledge/
Alcance: CRUD artículos médicos, flujo de aprobación (médico/admin)
Sin embeddings en esta fase — solo infraestructura de contenido

Archivos a crear:
  - src/knowledge/schemas/knowledge-article.schema.ts
  - src/knowledge/knowledge.service.ts
  - src/knowledge/knowledge.controller.ts
  - salud-de-una-web/src/features/admin-knowledge/
```

#### 3.3 Migración a GCP Cloud Run (si crece la carga)

```
Condición de trigger: > 100 usuarios activos simultáneos o costo Railway > $50/mes
Stack GCP:
  - Backend: Cloud Run (usa Dockerfile existente sin cambios)
  - Redis: Memorystore for Redis
  - Logs: Cloud Logging (nativo)
  - CI/CD: Actualizar GitHub Actions para deploy a Cloud Run
```

#### 3.4 Apple Sign In (mobile)

```
Pre-requisito: Apple Developer Program ($99/año)
Implementación: Auth0 social connection "Sign in with Apple"
Si no hay cuenta Apple Developer disponible: skip esta iteración
```

---

### Sprint 4+ — Monetización Real y Escala (mes 4+)

**Condición de entrada:** Plataforma en producción con usuarios reales validando el modelo.

> **Nota 2026-05-09:** El módulo de facturación simulada ya está implementado (billing/ en backend + admin billing UI en web). Lo que queda para este sprint es la integración con pasarela real.

- ~~Checkout simulado~~ ✅ **Implementado** — ver `billing/` en backend y `/admin/billing` en web
- Integración Stripe/Wompi cuando haya usuarios de pago (reemplaza el checkout simulado)
- Banco de conocimiento con RAG (embeddings Vertex AI o OpenAI)
- Perfil cuidador familiar (máx 3 dependientes)
- Nuevas especialidades (Cardiología, Dermatología, etc.)

---

## 9. Estrategia de Testing

### 9.1 Estado actual

| Capa | Herramienta | Tests | Cobertura |
|------|-------------|-------|-----------|
| Backend unitarios | Jest | 406 | 93% statements, 80% branches ✅ |
| Backend E2E | mongodb-memory-server | 10 suites | Por módulo |
| Web unitarios | Jest | 82 | 85% statements, 73% branches ⚠️ |
| Mobile unitarios | Jest | Activos | Sin dato exacto |
| E2E multi-servicio | — | ❌ No existe | — |

### 9.2 Reglas de testing que no se cambian

- Backend: **no mockear la capa de base de datos** en E2E. Solo en tests unitarios de módulo.
- AI: mockear `AiService` al nivel del módulo NestJS en tests que involucren IA.
- Chat gateway: cubrir en E2E, no en unitarios (depende de WebSocket real).

### 9.3 Mejoras a implementar en Sprint 1-2

```
1. Agregar tests de branches faltantes en web para superar 80%
2. Agregar test E2E de flujo Auth0 PKCE (simulado con mock de Auth0)
3. Agregar test de Socket.io con Redis adapter (2 instancias locales)
4. Agregar test E2E del flujo followup completo: close → followup creado → submit → escalación
```

### 9.4 Test de humo pre-deploy (checklist manual)

```
Ejecutar antes de cada deploy a producción:
  [ ] npm run test:cov (backend) — 0 fallos, todos los umbrales ≥ 80%
  [ ] npm run test (web) — 0 fallos
  [ ] npm run test (mobile) — 0 fallos
  [ ] npm run check:types (web) — 0 errores
  [ ] npm run typecheck (mobile) — 0 errores
  [ ] npm run lint (backend) — 0 warnings
  [ ] Flujo manual: login → triage → consulta → chat → close → followup
```

---

## 10. Estrategia de Despliegue

### 10.1 Arquitectura de despliegue Sprint 1

```
┌─────────────────────────────────────────────────┐
│                  GitHub (main)                  │
│                      │                          │
│         ┌────────────┼────────────┐             │
│         ▼            ▼            ▼             │
│    Railway CI    Vercel CI    EAS Build          │
│  (backend push) (web push)  (manual trigger)    │
└─────────────────────────────────────────────────┘
         │              │
         ▼              ▼
   Railway App      Vercel App
   NestJS :3000     Next.js :443
   api.saluddeuna.com  staff.saluddeuna.com
         │
    ┌────┴────┐
    ▼         ▼
MongoDB    Railway Redis
 Atlas     (BullMQ + throttler + Socket.io adapter)
```

### 10.2 Variables de entorno por ambiente

| Variable | Dev | Staging | Producción |
|----------|-----|---------|------------|
| `NODE_ENV` | development | production | production |
| `MONGODB_URI` | localhost | Atlas M0 (free) | Atlas M10+ |
| `REDIS_URL` | localhost | Railway Redis | Railway Redis |
| `AI_ENABLED` | false | true | true |
| `ENABLE_BOOTSTRAP_ADMIN` | true | false | false |
| `CORS_ORIGINS_*` | localhost:* | staging-domains | prod-domains |

### 10.3 Hardening pre-producción (checklist)

```
SEGURIDAD:
  [ ] JWT_SECRET mínimo 64 caracteres (no valor de ejemplo)
  [ ] JWT_REFRESH_SECRET mínimo 64 caracteres
  [ ] GEMINI_API_KEY configurada en Railway secrets (no en .env commiteado)
  [ ] MongoDB Atlas: network access restringido (no 0.0.0.0/0 en producción estable)
  [ ] ENABLE_BOOTSTRAP_ADMIN=false
  [ ] NODE_ENV=production (Swagger deshabilitado automáticamente)
  [ ] Verificar que ningún .env con valores reales está en git

AUTH0:
  [ ] Action post-login (ID: 113e227d) en estado DEPLOYED en tenant producción
  [ ] Allowed Callback URLs completos
  [ ] Google social connection habilitada y testeada
  [ ] Rate limits de Auth0 revisados (plan free: 7,500 MAU)

INFRAESTRUCTURA:
  [ ] Health check endpoint verificado: GET /v1 → 200
  [ ] Railway: configurar healthcheck path=/v1
  [ ] Railway: configurar restart policy on-failure
  [ ] Vercel: configurar dominio custom con SSL automático
  [ ] MongoDB Atlas: configurar alertas de conexiones y almacenamiento
```

---

## 11. Estrategia de Observabilidad

### 11.1 Estado actual (baseline)

- **Logging:** `request-logging.interceptor.ts` con correlation_id, user_id, role, latency_ms, status_code
- **Métricas:** Dashboard `/v1/dashboard/technical` + `/v1/dashboard/business` con 4 KPIs reales
- **Health:** Endpoints de health para MongoDB y Redis

### 11.2 Objetivo Sprint 1 (mínimo viable de observabilidad)

```
1. Configurar Railway Logs: los logs del interceptor ya están; solo verificar que
   se muestran correctamente en el panel de Railway.

2. Configurar Sentry en backend:
   npm install @sentry/nestjs
   - Capturar excepciones no manejadas con correlation_id como contexto
   - Alertas para errores 5xx

3. Configurar uptime monitoring: Railway tiene checks nativos de healthcheck.
   Alternativa: Better Uptime (free tier) para alertas de downtime.
```

### 11.3 Objetivo Sprint 3 (observabilidad completa)

```
- Sentry en los 3 servicios (backend, web, mobile)
- Dashboard de Sentry con métricas de Performance
- Alertas configuradas: error rate > 1%, p95 latency > 2s
- Logtail o Axiom para búsqueda de logs por correlation_id
- KPI 3 y KPI 4 con alertas si caen bajo umbral (≥ 60% y ≥ 50%)
```

---

## 12. Estrategia de Seguridad

### 12.1 Controles actuales verificados

| Control | Estado | Notas |
|---------|--------|-------|
| JWT access + refresh tokens | ✅ | Max 3 sesiones activas |
| Auth0 para web | ✅ | JWT validado con JWKS |
| Rate limiting | ✅ | 20 req/60s, Redis throttler storage |
| Role-based access | ✅ | Guards: PATIENT, DOCTOR, ADMIN |
| Doctor verification guard | ✅ | Solo doctores VERIFIED acceden a consultas |
| Global HTTP exception filter | ✅ | No expone stack traces en producción |
| Input validation | ✅ | class-validator en todos los DTOs |
| Zod en frontend/mobile | ✅ | Validación de respuestas de API |
| Correlation ID propagation | ✅ | Trazabilidad de requests |

### 12.2 Riesgos de seguridad a resolver en Sprint 1

```
1. CORS_ORIGINS con dominios reales (no localhost) → configurar en Railway
2. Verificar que .gitignore excluye todos los .env con valores reales
3. Secrets en Railway env vars (no en código)
4. MongoDB Atlas network access configurado apropiadamente
5. Swagger deshabilitado en NODE_ENV=production (ya implementado automáticamente)
```

### 12.3 Para Sprint 2+

```
- Agregar rate limiting más granular por endpoint sensible (login, register)
- Audit log de acciones admin (quién aprobó qué doctor, cuándo)
- Revisión de dependencias con npm audit en CI
- Rotation de JWT_SECRET en caso de compromiso
```

---

## 13. Convenciones Técnicas del Proyecto

### 13.1 Código

- **Idioma de código:** Inglés. **Documentación:** Español.
- **Imports:** directos a archivos, no barrel exports (`index.ts`)
- **TypeScript:** Sin `any` implícito. Nunca usar `as any` salvo en tests con mocks
- **Comentarios:** Solo cuando el "por qué" no es obvio. Nunca el "qué"
- **DTOs:** Siempre decorados con `@ApiProperty()` para Swagger
- **Tests:** 1 describe por módulo, it/test por caso. Nombres en formato "should [behavior]"

### 13.2 Git y branching

```
main      → producción (deploy automático)
develop   → integración (opcional, agregar si el equipo crece)
feat/xxx  → features (PR → main o develop)
fix/xxx   → bugfixes
chore/xxx → infra, dependencias, config
```

### 13.3 Endpoints backend

```
GET    /v1/{resource}           → listar (paginado)
GET    /v1/{resource}/:id       → obtener uno
POST   /v1/{resource}           → crear
PATCH  /v1/{resource}/:id       → actualizar parcial
DELETE /v1/{resource}/:id       → eliminar
POST   /v1/{resource}/:id/{action} → acción de dominio (close, assign, submit)
```

### 13.4 Respuesta de errores

```json
{
  "statusCode": 400,
  "message": "Descripción del error",
  "correlationId": "uuid-v4"
}
```

---

## 14. Próximos Pasos Accionables

### Esta semana (días 1-7)

| # | Tarea | Responsable | Esfuerzo |
|---|-------|-------------|----------|
| 1 | Agregar `@socket.io/redis-adapter` al backend | Backend | 4h |
| 2 | Eliminar `app/dashboard/` duplicado en web | Frontend | 1h |
| 3 | Crear proyecto Railway + addon Redis | DevOps | 2h |
| 4 | Configurar todas las env vars de producción en Railway | DevOps | 2h |
| 5 | Conectar MongoDB Atlas a Railway (whitelist IP) | DevOps | 1h |
| 6 | Deploy inicial backend en Railway + verificar healthcheck | DevOps | 2h |
| 7 | Importar proyecto web en Vercel + configurar env vars | DevOps | 2h |
| 8 | Configurar dominio custom en Railway y Vercel | DevOps | 2h |

### Semana 2 (días 8-14)

| # | Tarea | Responsable | Esfuerzo |
|---|-------|-------------|----------|
| 9 | Crear Auth0 Application Native para mobile | Backend | 1h |
| 10 | Conectar auth0-service.ts al flujo de login mobile | Mobile | 6h |
| 11 | Probar Auth0 PKCE en simulador iOS y Android | Mobile | 4h |
| 12 | Configurar EAS Build con perfiles development/preview/production | Mobile | 3h |
| 13 | Primer EAS Build Android preview | Mobile | 2h |
| 14 | Crear workflow CI/CD de deploy backend (Railway) | DevOps | 3h |
| 15 | Crear workflow CI/CD de deploy web (Vercel) | DevOps | 3h |

### Semana 3 (días 15-21)

| # | Tarea | Responsable | Esfuerzo |
|---|-------|-------------|----------|
| 16 | QA completo del flujo E2E en producción | QA | 4h |
| 17 | Configurar Auth0 Allowed URLs de producción | Backend | 1h |
| 18 | Configurar Sentry en backend y web | Backend | 3h |
| 19 | Configurar uptime monitoring (Railway healthcheck) | DevOps | 1h |
| 20 | Documentar credenciales de producción (en lugar seguro, no en repo) | Todos | 1h |
| 21 | Primer EAS Build Android production (si hay usuario Android de prueba) | Mobile | 2h |

---

## 15. Checklist Técnico por Fase

### Sprint 1 — Producción

```
BACKEND:
  [ ] @socket.io/redis-adapter instalado y conectado en chat.gateway.ts
  [ ] Chat verificado en 2 instancias simultáneas con mismo Redis
  [ ] Deploy en Railway funcionando
  [ ] Health check: GET /v1 → 200 en Railway
  [ ] Swagger: GET /v1/docs → 404 en Railway (NODE_ENV=production)
  [ ] Todos los tests pasando en CI antes del deploy
  [ ] Variables de entorno de producción configuradas en Railway

WEB:
  [ ] Rutas duplicadas `dashboard/` eliminadas
  [ ] Deploy en Vercel funcionando
  [ ] Login con Auth0 funciona en dominio de producción
  [ ] Doctor puede ver cola, asignar y cerrar consulta
  [ ] Admin puede ver usuarios, doctores, reportes
  [ ] Dashboard KPIs muestran datos reales

MOBILE:
  [ ] Auth0 PKCE conectado y probado en simulador
  [ ] Provisioning automático funciona para usuario nuevo
  [ ] Refresh token funciona correctamente
  [ ] EAS Build Android funcional (APK descargable)
  [ ] Push notifications funcionan en build de producción

CI/CD:
  [ ] Push a main → deploy automático backend en Railway
  [ ] Push a main → deploy automático web en Vercel
  [ ] CI falla si tests fallan (bloquea deploy)
  [ ] Secrets configurados en GitHub Actions (no hardcodeados)

AUTH0:
  [ ] Action post-login DEPLOYED
  [ ] Google social connection habilitada
  [ ] Allowed URLs completos para producción
```

### Sprint 2 — Calidad

```
  [ ] Branches coverage web ≥ 80%
  [ ] 4 especialidades en triage (+ Pediatría + Urgencias)
  [ ] Prompts especializados por especialidad en DB
  [ ] UI de feedback IA en panel del doctor
  [ ] Pantalla de timeline en mobile
  [ ] Red flags rules para nuevas especialidades
```

---

## 16. Módulos Huérfanos y Código Muerto

| Archivo | Tipo | Acción recomendada |
|---------|------|-------------------|
| `app/dashboard/` (web) | Ruta duplicada | Eliminar; mantener solo `(dashboard)/` |
| `app/(auth)/register.tsx` (web) | Revisar si se usa | Validar flujo de registro de doctores via Auth0 |
| `src/strategies/jwt-legacy.strategy.ts` | Legacy | Mantener mientras mobile use JWT propio. Eliminar en Sprint 2 tras migrar mobile a Auth0 |
| `src/strategies/jwt-provision.strategy.ts` | Provisioning | Mantener, es parte del flujo Auth0 provision |

---

## 17. Consistencia Backend / Web / Mobile

| Campo / Enum | Backend | Web | Mobile | Estado |
|--------------|---------|-----|--------|--------|
| Especialidades (specialty.enum) | MG, Odontología | ✅ | ✅ | Sincronizados |
| Estados de consulta | Completo | ✅ | ✅ | Sincronizados |
| Estados de followup | PENDING, COMPLETED, MISSED | N/A | ✅ | OK |
| User roles | PATIENT, DOCTOR, ADMIN | ✅ | PATIENT | OK |
| Error response format | {statusCode, message, correlationId} | ✅ | ✅ | Sincronizados |
| Doctor status (RETHUS) | PENDING, VERIFIED, REJECTED | ✅ | N/A | OK |

---

## 18. Historial de Versiones del Plan

| Versión | Fecha | Resumen de cambios |
|---------|-------|--------------------|
| 1.0 | 2026-05-03 | Plan inicial post-fase académica. Fases 3-7 definidas. |
| 2.0 | 2026-05-06 | Auditoría completa contra código real. Confirmado: followups, timeline, KPIs y admin console implementados correctamente. Gaps reales identificados: Socket.io adapter, Auth0 mobile sin validar, CI/CD sin deploy, EAS Build no configurado. Roadmap reestructurado con deadline urgente de producción. |
| 2.1 | 2026-05-09 | **Módulo billing implementado:** checkout simulado completo (backend: billing/, billing-price-seeder, BillingPrice + Transaction schemas; web: admin-billing-page, use-admin-billing hook, admin-service endpoints, tipos en admin.ts). **Specialty URGENT_CARE** agregado al enum. **Admin dashboard ampliado:** useRevenueMetrics, useAiMetrics, useConsultationMetrics, useDashboardAlerts. Deuda técnica #11 (monetización simulada) marcada como COMPLETADA. Mobile audit (2026-05-07): 8 errores críticos corregidos. |

---

*Documento generado el 2026-05-06 mediante auditoría técnica completa del monorepo SaludDeUna.*  
*Próxima revisión recomendada: al finalizar Sprint 1 (semana 3).*
