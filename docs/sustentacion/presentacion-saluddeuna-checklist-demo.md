# SaludDeUna — Checklist de Demo en Vivo

> Ejecutar este checklist la noche anterior Y 30 minutos antes de la sustentación.
> Fecha del evento: **Jueves 21 de mayo de 2026 · 11:00 a.m. · IETI Grupo 1**

---

## NOCHE ANTERIOR (lunes 20 mayo — 9:00–11:00 p.m.)

### ✅ 1. Verificar servicios en producción

- [ ] **Backend responde:**

  ```
  curl https://salud-de-una-backend-production.up.railway.app/v1
  ```

  Debe retornar `{"status":"ok"}` con HTTP 200.

- [ ] **Web carga sin errores:**
  Abrir la URL de Vercel en browser. No debe haber errores en consola.

- [ ] **AI_ENABLED=true:**
  Verificar en Railway Variables que `AI_ENABLED=true` y `GEMINI_API_KEY` tiene valor.

- [ ] **MongoDB conectado:**
  Revisar Railway Logs — sin errores de conexión a MongoDB al arrancar.

- [ ] **Redis conectado:**
  Revisar Railway Logs — sin errores de conexión a Redis.

---

### ✅ 2. Preparar datos de demo

- [ ] **Verificar APK en dispositivo:**
  Login con `paciente.demo@saluddeuna.com` / `Dem0.P4c1ente!` → home carga.

- [ ] **Verificar login doctor en web:**
  Tab incógnito → `doctor.demo@saluddeuna.com` / `Dem0.D0ct0r!` → cola de consultas carga.

- [ ] **Verificar login admin en web:**
  Admin login → dashboard con KPIs carga y muestra números reales.

---

### ✅ 3. APK y QR

- [ ] **APK instalado en dispositivo Android físico** (no emulador — se ve mejor al proyectar).

- [ ] **QR del APK listo para proyectar:**
  `https://expo.dev/accounts/saluddeuna/projects/salud-de-una-mobile/builds`
  → Copiar URL del build `preview` más reciente → guardar como imagen o tener tab abierta.

- [ ] **QR legible desde 3 metros:**
  Proyectar en pantalla y verificar que se puede escanear desde lejos.

---

### ✅ 4. Screenshots de respaldo

Guardar en `Downloads/demo-backup/`:

- [ ] Dashboard admin con KPIs reales (pacientes, revenue, doctores verificados)
- [ ] Lista de doctores con doctor VERIFIED
- [ ] Triage en progreso (pregunta con specialty seleccionada)
- [ ] Resultado de triage con prioridad ALTA y red flag visible
- [ ] Cola de consultas del doctor (consulta en tope)
- [ ] Chat en tiempo real (ambos mensajes visibles)
- [ ] Resumen clínico generado por Gemini
- [ ] Pantalla de billing / revenue metrics
- [ ] Swagger docs `https://...backend.../v1/docs` (si está disponible en staging, NO en prod)

---

### ✅ 5. Preparar tabs del browser (noche anterior)

| Tab # | URL | Usuario | Notas |
|-------|-----|---------|-------|
| 1 | Admin dashboard | admin.demo@ | Dashboard KPIs visible |
| 2 | Doctor queue (incógnito) | doctor.demo@ | Cola vacía aún |
| 3 | QR del APK (EAS) | — | Solo imagen |
| 4 | Health check backend | — | `.../v1` → 200 |
| 5 | Presentación (Canva/Slides) | — | Slide 1 en pantalla |

---

### ✅ 6. Preparar credenciales (en papel o celular)

```
BACKEND:   https://salud-de-una-backend-production.up.railway.app/v1
WEB:       [URL de Vercel]
EAS BUILD: https://expo.dev/accounts/saluddeuna/projects/salud-de-una-mobile/builds

PACIENTE:  paciente.demo@saluddeuna.com  /  Dem0.P4c1ente!
DOCTOR:    doctor.demo@saluddeuna.com    /  Dem0.D0ct0r!
ADMIN:     [email admin real]            /  [password admin real]
```

---

## 30 MINUTOS ANTES (21 mayo — 10:30 a.m.)

### ✅ Dispositivos y conectividad

- [ ] WiFi disponible — conectar todos los dispositivos
- [ ] **Plan B:** hotspot del celular activado como respaldo
- [ ] Cable HDMI o adaptador preparado si hay proyector físico
- [ ] Dispositivo Android encendido y con batería ≥ 80%
- [ ] **Modo No Molestar** activado en todos los dispositivos
- [ ] Brillo de pantalla al máximo en el teléfono

### ✅ Verificación final

- [ ] Health check: `https://.../v1` → 200 OK
- [ ] Login admin funciona en Tab 1
- [ ] Login doctor funciona en Tab 2 (incógnito)
- [ ] Login paciente funciona en el APK
- [ ] QR escaneable desde 3 metros
- [ ] Presentación en Slide 1, lista para comenzar

---

## RUTA DETALLADA DEL DEMO (12 pasos · ~7 minutos)

### Preparación antes de empezar el demo

1. Slide 22 (Ruta del demo) en pantalla
2. Dispositivo Android en mano, APK abierto en home del paciente demo
3. Tab 2 (doctor en incógnito) minimizado
4. Tab 1 (admin) minimizado

---

### PASO 1 — Proyectar QR (30 s)

**Acción:** Slide 23 en pantalla (QR grande + URLs)

**Qué decir:**
> "Aquí el QR del APK. Está disponible para descarga directa desde EAS Build de Expo."

**Verificar:** QR visible y escaneable desde el fondo del salón

---

### PASO 2 — Login del paciente en APK (30 s)

**Acción:** Mostrar teléfono al jurado. Abrir app → pantalla de login.

**Qué decir:**
> "El paciente entra con sus credenciales. Esto es JWT — acceso directo al sistema."

**Credenciales:** `paciente.demo@saluddeuna.com` / `Dem0.P4c1ente!`

---

### PASO 3 — Seleccionar especialidad e iniciar triage (30 s)

**Acción:** Toca "Iniciar consulta" → seleccionar "Medicina General"

**Qué decir:**
> "El paciente selecciona la especialidad. El sistema crea una sesión de triage."

---

### PASO 4 — Responder cuestionario (1 min)

**Acción:** Responder 5 preguntas con respuestas preparadas para activar prioridad ALTA:

```
Pregunta 1 — Síntoma principal: "dolor en el pecho y dificultad para respirar"
Pregunta 2 — Duración: "3 horas"
Pregunta 3 — Intensidad (1-10): 8
Pregunta 4 — ¿Tiene fiebre?: sí
Pregunta 5 — ¿Toma medicamentos?: no
```

**Qué decir:**
> "Las preguntas varían por especialidad. El sistema guía al paciente — no le pide que describa desde cero."

---

### PASO 5 — Ver resultado del triage (30 s)

**Acción:** Tocar "Analizar" → esperar respuesta Gemini → mostrar resultado

**Qué mostrar:** Prioridad ALTA (rojo), red flag `RF-MG-001` visible

**Qué decir:**
> "Gemini analizó las respuestas. Detectó una combinación de dolor en el pecho con dificultad respiratoria — esto es un red flag de posible urgencia cardiovascular. El sistema asigna prioridad ALTA."

> "Lo que no dice el sistema: no dice 'usted tiene un infarto'. Dice: 'su caso requiere atención urgente'. El médico decide."

---

### PASO 6 — Checkout simulado (30 s)

**Acción:** Continuar al checkout → confirmar pago → consulta creada

**Qué decir:**
> "El paciente paga el precio de la especialidad — en este caso Medicina General. El billing está simulado para el MVP — el ciclo de vida de la transacción sí es real: PENDING → COMPLETED."

---

### PASO 7 — Login del doctor en web (30 s)

**Acción:** Cambiar al Tab 2 (incógnito) → login con doctor demo

**Qué decir:**
> "En el panel web, el doctor inicia sesión. Esto es el portal staff, construido con Next.js 16."

**Credenciales:** `doctor.demo@saluddeuna.com` / `Dem0.D0ct0r!`

---

### PASO 8 — Ver cola de consultas (20 s)

**Acción:** Navegar a la cola → mostrar la consulta con prioridad ALTA al tope

**Qué decir:**
> "La cola está ordenada por prioridad. La consulta que acabamos de crear aparece arriba — prioridad ALTA, con el red flag visible."

---

### PASO 9 — Asignar y abrir chat (20 s)

**Acción:** Asignar consulta → abrir sala de chat

**Qué decir:**
> "El doctor toma la consulta. Se abre la sala de chat — Socket.IO, tiempo real."

---

### PASO 10 — Chat en tiempo real (1 min)

**Acción:**

- Doctor escribe desde web: "Hola, cuénteme más sobre su dolor en el pecho"
- Paciente responde desde APK (mostrar al jurado)

**Qué mostrar:** Mensaje aparece instantáneamente en ambas pantallas

**Qué decir:**
> "Tiempo real — los mensajes aparecen instantáneamente. Cada mensaje está persistido en MongoDB. El historial completo queda disponible para el resumen clínico."

---

### PASO 11 — Resumen clínico con IA (30 s)

**Acción:** Doctor hace clic en "Generar resumen" → esperar 2–4 segundos → mostrar nota médica

**Qué decir:**
> "El doctor pide el resumen. Gemini genera una nota médica estructurada basada en el chat. Aquí ven el disclaimer clínico al final — el guardrail garantizó que no hay diagnóstico en el texto."

> "El médico puede editar este resumen y guardarlo como nota oficial de la consulta."

---

### PASO 12 — Dashboard admin con KPIs (30 s)

**Acción:** Cambiar al Tab 1 (admin) → mostrar dashboard con KPIs

**Qué mostrar:**

- Total pacientes registrados
- Revenue del período
- Doctores verificados vs. pendientes
- p95 latencia y error rate en dashboard técnico

**Qué decir:**
> "Estas son métricas reales de la base de datos — no hardcodeadas. El admin ve KPIs de negocio y métricas técnicas en tiempo real."

---

## CONTINGENCIAS

### Si falla la conexión a internet

1. Activar hotspot en 30 segundos
2. Reconectar todos los dispositivos
3. Si el hotspot no resuelve → abrir `Downloads/demo-backup/`
4. **Nunca dejar de hablar** — continuar el guion mientras se resuelve
5. > "Tenemos un pequeño inconveniente de red. Mientras se resuelve, les muestro los screenshots del flujo completo."

---

### Si el APK no conecta al backend

1. Mostrar screenshots del flujo mobile desde `demo-backup/`
2. Simular el flujo del paciente desde el browser web (si hay acceso mobile desde web)
3. > "En producción el APK conecta al mismo backend. Aquí les muestro el mismo flujo desde el browser."

---

### Si Gemini no responde durante el triage

1. No hacer nada extra — el sistema usa `RULE_BASED` automáticamente sin error visible
2. El resultado puede tomar unos segundos más
3. > "El sistema usó el fallback basado en reglas — este es precisamente el criterio de resiliencia en acción. Si Gemini no está disponible, el sistema sigue funcionando con las reglas deterministas."

---

### Si el seed de datos no corrió y la cola está vacía

1. Desde la app, crear una nueva consulta en vivo (el demo funciona igual)
2. Si no hay tiempo: mostrar screenshot de la cola con datos reales

---

### Si el jurado pide ver el código durante la demo

1. Tener una tab con el repositorio de GitHub abierta
2. Ir directamente a `src/triage/` para mostrar el guardrail
3. Ir a `src/chat/chat.gateway.ts` para mostrar el WebSocket
4. Ir a `test/e2e/` para mostrar los tests E2E

---

## Qué mostrar si el jurado pregunta por el Swagger

> "El Swagger está disponible en entorno de desarrollo en `/v1/docs`. En producción está deshabilitado automáticamente cuando `NODE_ENV=production` — esto es un control de seguridad. Si quieren, puedo mostrar el endpoint en local o el archivo OpenAPI exportado en la wiki."

---

## Checklist de cierre del evento

- [ ] Agradecer al jurado
- [ ] QR sigue visible en pantalla durante las preguntas
- [ ] Responder preguntas con las respuestas del archivo `preguntas-respuestas.md`
- [ ] Si no saben algo: "Es una muy buena pregunta. No lo implementamos en este MVP, pero la forma en que lo abordaríamos sería..."
