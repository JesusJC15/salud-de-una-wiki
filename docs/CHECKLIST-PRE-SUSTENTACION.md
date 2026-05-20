# Checklist Pre-Sustentación — SaludDeUna

## IETI Grupo 1 · Jueves 21 mayo 2026 · 11:00 a.m

> Ejecuta este checklist completo la **noche anterior** y de nuevo **30 minutos antes** del evento.  
> Marca cada ítem conforme lo verifiques. Si algo falla, aplica el Plan B correspondiente.

---

## NOCHE ANTERIOR (martes 20 mayo — 9:00 p.m. – 11:00 p.m.)

### 1. Servicios en Producción

- [ ] **Backend Railway responde:**

  ```
  curl https://salud-de-una-backend-production.up.railway.app/v1
  ```

  Debe retornar `{"status":"ok"}` o similar con HTTP 200.

- [ ] **Web Vercel carga correctamente:**  
  Abrir `[URL-VERCEL]` en el browser — debe cargar la pantalla de login sin errores en consola.

- [ ] **AI_ENABLED=true en producción:**  
  Verificar en el panel de Railway (Variables de entorno) que `AI_ENABLED=true` y `GEMINI_API_KEY` tiene un valor.

- [ ] **MongoDB Atlas conectado:**  
  Verificar en Railway Logs que no hay errores de conexión a MongoDB al iniciar el servicio.

- [ ] **Redis conectado:**  
  Verificar en Railway Logs que no hay errores de conexión a Redis.

---

### 2. Seed de Datos de Demo

- [ ] **Ejecutar seed-demo.ts:**

  ```bash
  cd salud-de-una-backend
  SEED_API_URL=https://salud-de-una-backend-production.up.railway.app/v1 \
  SEED_ADMIN_EMAIL=TU_ADMIN_EMAIL \
  SEED_ADMIN_PASSWORD="TU_ADMIN_PASSWORD" \
  npx ts-node scripts/seed-demo.ts
  ```

  Debe imprimir `✅ Datos de demo listos` al final.

- [ ] **Verificar doctor demo en panel admin web:**  
  Login como admin → `/admin/doctors` → confirmar que `doctor.demo@saluddeuna.com` aparece como **VERIFIED**.  
  Si no: hacer click en "Verificar" manualmente.

- [ ] **Login paciente en APK:**  
  Abrir APK en dispositivo Android → login con `paciente.demo@saluddeuna.com` / `Dem0.P4c1ente!` → confirmar que carga el home.

- [ ] **Login doctor en web:**  
  Tab incógnito → login con `doctor.demo@saluddeuna.com` / `Dem0.D0ct0r!` → confirmar que carga la cola de consultas.

- [ ] **Login admin en web:**  
  Login con las credenciales de admin → confirmar que carga el dashboard con KPIs.

---

### 3. APK y QR

- [ ] **APK instalado en dispositivo Android físico** (no emulador):  
  Confirmar que el icono de SaludDeUna aparece en la pantalla de inicio del teléfono.

- [ ] **QR del APK preparado:**  
  Abrir `https://expo.dev/accounts/saluddeuna/projects/salud-de-una-mobile/builds`  
  → Copiar la URL del build `preview` más reciente  
  → Guardar en una tab del browser o como imagen para proyectar.

- [ ] **QR visible desde 3 metros de distancia:**  
  Proyectar en pantalla y verificar que se puede escanear desde lejos.

---

### 4. Diagramas SVG

Abrir cada uno en el browser y verificar que se renderizan correctamente:

- [ ] `salud-de-una-wiki/diagrams/sad_c4_context_diagram.svg`
- [ ] `salud-de-una-wiki/diagrams/sad_c4_container_view.svg`
- [ ] `salud-de-una-wiki/diagrams/sad_aws_deployment_diagram.svg`
- [ ] `salud-de-una-wiki/diagrams/sad_rag_pipeline_diagram.svg`
- [ ] `salud-de-una-wiki/diagrams/sad_backend_modules_diagram.svg`
- [ ] `salud-de-una-wiki/diagrams/sad_auth_sequence_flow.svg`
- [ ] `salud-de-una-wiki/diagrams/sad_observability_stack.svg`

> **Tip:** Abrir todos en tabs separadas con Ctrl+T y arrastrar las pestañas para tener el orden de presentación.

---

### 5. Screenshots de Backup (offline)

En caso de fallo de red en el salón, tener screenshots listos en una carpeta local:

- [ ] Dashboard admin con KPIs reales
- [ ] Lista de doctores (con doctor VERIFIED)
- [ ] Pantalla de triage en mobile (resultado con IA)
- [ ] Cola de consultas en doctor web
- [ ] Chat en tiempo real (doctor y paciente)
- [ ] Pantalla de billing / revenue metrics
- [ ] Swagger documentado (`/v1/docs`)

> Guardar en `Downloads/demo-backup/` o equivalente.

---

### 6. Guion y Q&A

- [ ] Leer completo el documento `docs/SUSTENTACION-IETI.md`
- [ ] Practicar el pitch de 2 minutos (contexto del problema) — cronometrar
- [ ] Ensayar el demo flow completo una vez sin interrupciones
- [ ] Repasar las 15 preguntas difíciles de la Sección 6
- [ ] Guardar las credenciales de demo en papel o en el celular (no solo en la cabeza)

---

## 30 MINUTOS ANTES (jueves 21 mayo — 10:30 a.m.)

### Dispositivos y Conectividad

- [ ] **WiFi disponible** en el salón — conectar todos los dispositivos
- [ ] **Plan B WiFi listo:** Hotspot del celular activado como respaldo
- [ ] **Cable HDMI o adaptador** preparado si hay proyector físico

### Browser (computador de presentación)

Tener estas tabs abiertas Y con login activo:

| Tab # | URL | Login |
|-------|-----|-------|
| 1 | `[URL-VERCEL]` — Admin dashboard | <admin.demo@saluddeuna.com> |
| 2 | `[URL-VERCEL]` — Doctor queue (incógnito) | <doctor.demo@saluddeuna.com> |
| 3 | QR del APK (EAS build) | — |
| 4 | `sad_c4_context_diagram.svg` (local o GitHub) | — |
| 5 | `sad_c4_container_view.svg` | — |
| 6 | `sad_aws_deployment_diagram.svg` | — |
| 7 | `sad_rag_pipeline_diagram.svg` | — |
| 8 | `sad_backend_modules_diagram.svg` | — |
| 9 | `https://salud-de-una-backend-production.up.railway.app/v1` (health) | — |

### Mobile

- [ ] **Dispositivo Android encendido y cargado** (≥ 80% batería)
- [ ] **APK abierto** con `paciente.demo@saluddeuna.com` logueado
- [ ] **Modo no molestar activado** para evitar interrupciones durante el demo
- [ ] **Brillo al máximo** para que se vea bien en el proyector

### Verificación final de producción

- [ ] Health check del backend: `https://salud-de-una-backend-production.up.railway.app/v1` → HTTP 200
- [ ] Login admin funciona en Tab 1
- [ ] Login doctor funciona en Tab 2 (incógnito)
- [ ] Login paciente funciona en el APK

---

## EN EL EVENTO

### Inicio (minuto 0)

1. Proyectar **Tab 3 (QR del APK)** inmediatamente — debe estar visible durante TODA la presentación
2. Mostrar el dispositivo Android con la app abierta
3. Decir: *"El APK está disponible en este QR para descarga directa desde EAS Build."*

### Si falla la red durante la demo

1. Activar hotspot del celular en 30 segundos
2. Reconectar todos los dispositivos al hotspot
3. Si el hotspot no resuelve: usar screenshots de backup en `Downloads/demo-backup/`
4. Nunca dejar de hablar — seguir con el guion verbal mientras se resuelve

### Si el APK no conecta al backend

1. Mostrar screenshots del flujo de mobile
2. Hacer la demo en el browser web simulando el flujo del paciente
3. Mencionar: *"En producción el APK conecta al mismo backend. Aquí estamos mostrando el mismo flujo desde el browser."*

### Si Gemini no responde durante el triage en vivo

1. El sistema automáticamente usa RULE_BASED (no hay error visible para el usuario)
2. Mencionar: *"Aquí el sistema usó el fallback basado en reglas — este es precisamente nuestro criterio de resiliencia en acción."*

---

## Datos de Referencia Rápida

```
Backend:  https://salud-de-una-backend-production.up.railway.app/v1
Swagger:  https://salud-de-una-backend-production.up.railway.app/v1/docs
EAS:      https://expo.dev/accounts/saluddeuna/projects/salud-de-una-mobile/builds
Auth0:    salud-de-una.us.auth0.com

PACIENTE: paciente.demo@saluddeuna.com  /  Dem0.P4c1ente!
DOCTOR:   doctor.demo@saluddeuna.com   /  Dem0.D0ct0r!
ADMIN:    [tu email de admin]          /  [tu password]
```

---

*Documento generado: 2026-05-19 | Para uso interno del equipo SaludDeUna*
