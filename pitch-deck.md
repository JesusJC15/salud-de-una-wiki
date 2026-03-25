# 🏥 SaludDeUna — Pitch Deck

> **Comunicación Clínica Inteligente, de un solo golpe.**

---

## 1. Portada

| Campo | Detalle |
|---|---|
| **Producto** | SaludDeUna |
| **Tagline** | La capa inteligente que conecta pacientes y médicos antes de la consulta |
| **Contexto** | Proyecto de innovación con IA Móvil — IETI 2026-1 |
| **Fecha de Inception** | Presentación: 9 de marzo de 2026 |
| **Repositorio Wiki** | https://github.com/JesusJC15/salud-de-una-wiki |

---

## 2. El Equipo

| Nombre | Correo |
|---|---|
| Jesús Alberto Jauregui Conde | jesus.jauregui-c@mail.escuelaing.edu.co |
| Mayerlly Suárez Correa | mayerlly.suarez-c@mail.escuelaing.edu.co |
| Natalia Espitia Espinel | natalia.espitia-e@mail.escuelaing.edu.co |
| Santiago Hurtado Martínez | santiago.hurtado-m@mail.escuelaing.edu.co |

**Distribución sugerida:** Móvil · Backend/IA · Web/Admin · QA-DevOps (rotativo)

---

## 3. El Problema

### Del lado del paciente
- ❌ No distingue urgencia real de síntomas no urgentes.
- ❌ Describe mal los síntomas por falta de estructura.
- ❌ Omite datos relevantes por nervios o desconocimiento.
- ❌ Consultas cortas y costosas con información incompleta.

### Del lado del médico
- ❌ Recibe información desorganizada al iniciar cada consulta.
- ❌ Repite las mismas preguntas base de manera constante.
- ❌ Alta carga operativa por triage manual sin herramientas.
- ❌ Falta continuidad estructurada entre consultas del mismo paciente.

> **El médico invierte parte importante de la consulta en recolectar antecedentes, reduciendo el tiempo clínico efectivo y afectando la continuidad de la atención.**

---

## 4. La Solución: SaludDeUna

**SaludDeUna** es una plataforma inteligente de comunicación clínica que conecta pacientes y médicos mediante inteligencia artificial. Es una capa de triage predictivo, estructuración clínica y seguimiento evolutivo que **optimiza la información antes de que el médico intervenga**.

### ¿Qué hace?
1. **Guía al paciente** con preguntas estructuradas por especialidad antes de la consulta.
2. **Identifica red flags** y clasifica el nivel de prioridad: `LOW | MODERATE | HIGH`.
3. **Genera un resumen clínico** preconsulta para el médico de forma automática.
4. **Conecta paciente y médico** en un chat clínico en tiempo real.
5. **Hace seguimiento** post-consulta y construye una línea evolutiva del caso.

### Regla ética explícita
> La IA **no diagnostica** · La IA **no prescribe** · La decisión final siempre es del **médico**

---

## 5. Propuesta de Valor

| Actor | Beneficio |
|---|---|
| **Paciente** | Orientación temprana, mejor expresión de síntomas, continuidad del cuidado |
| **Médico** | Menos tiempo recolectando antecedentes, mejor información previa, priorización automática de casos |
| **Negocio** | Modelo híbrido con pago por consulta + suscripción, reducción de fricción operativa, datos de eficiencia clínica |

---

## 6. Producto — Capacidades Clave del MVP

### Especialidades cubiertas
- 🩺 **Medicina General**
- 🦷 **Odontología**

### Módulos principales

| Módulo | Descripción |
|---|---|
| **Triage IA** | Cuestionario guiado por especialidad + motor de reglas de red flags + clasificación de prioridad con Gemini + RAG |
| **Resumen Clínico Automático** | Genera historia clínica preconsulta con queja principal, duración, intensidad, síntomas asociados, medicación y antecedentes relevantes |
| **Chat Clínico en Tiempo Real** | WebSocket bidireccional con estados de consulta (`OPEN → IN_PROGRESS → CLOSED`) y cola médica priorizada |
| **Seguimiento Post-Consulta** | Formularios evolutivos, timeline del paciente, alertas automáticas por empeoramiento |
| **Panel Médico/Admin Web** | Cola priorizada de casos, validación REThUS, dashboards técnicos y de negocio |
| **Monetización Simulada** | Pago por consulta + suscripción (flujo simulado para validación del modelo) |
| **Banco de Conocimiento** | Artículos validados por médicos con aprobación interna |

---

## 7. Flujos de Usuario (Mockups Clave)

### Flujo del Paciente
```
Login/Registro → Inicio → Triage Guiado → Resultado de Prioridad → Chat Clínico → Seguimiento → Timeline
```

### Pantallas Principales

**Login**
```
+---------------------------------------+
| SaludDeUna — Ingreso seguro           |
| [ Correo electrónico             ]    |
| [ Contraseña                    ]     |
| [ Iniciar sesión ]                    |
| No tienes cuenta? [ Registrarme ]     |
+---------------------------------------+
```

**Triage Guiado**
```
+---------------------------------------+
| Triage - Medicina General  Paso 2/4   |
| Síntoma principal                     |
| [ Dolor de cabeza               ]     |
| Intensidad (1-10)                     |
| [-----|----*-----] 7                  |
| [ Atras ]             [ Siguiente ]   |
+---------------------------------------+
```

**Chat Clínico en Tiempo Real**
```
+---------------------------------------+
| Consulta #C-1024    Estado: ACTIVA    |
| Medico: Cuentame desde cuando inicio  |
| Paciente: Desde ayer en la tarde      |
| Medico: Tienes fiebre?                |
| Paciente: Si, 38.4                    |
| [ Escribe tu mensaje...         ] [>] |
+---------------------------------------+
```

**Seguimiento Post-Consulta**
```
+---------------------------------------+
| Seguimiento (72h)                     |
| Como te sientes hoy?                  |
| Intensidad actual (1-10): [ 6 ]       |
| Hay nuevos sintomas? [ Si/No ]        |
| Tomaste medicacion? [ Si/No ]         |
| [ Enviar seguimiento ]                |
+---------------------------------------+
```

---

## 8. Arquitectura Técnica

### Stack Tecnológico
| Capa | Tecnología |
|---|---|
| App Paciente | React Native (Android objetivo principal) |
| Panel Médico/Admin | React + Next.js |
| Backend | NestJS (Node.js) |
| Base de Datos | MongoDB |
| Tiempo Real | WebSocket |
| Inteligencia Artificial | Google Gemini + RAG + Reglas Clínicas |
| Índice Vectorial | Vector Index para contexto RAG |
| Cloud | Azure institucional + IAProviderAdapter |
| CI/CD | GitHub Actions |
| Gestión de Proyecto | Azure DevOps (Wiki + Boards) |

### Diagrama General del Sistema
```
Paciente App (React Native) ──┐
                               ├──► API Backend (NestJS) ──► MongoDB
Panel Médico/Admin (Next.js) ──┘         │
                                         ├──► WebSocket Hub
                                         ├──► IA Orchestrator (RAG + Gemini)
                                         │         └──► Vector Index
                                         └──► Observabilidad (Logs + Métricas + Dashboard)
```

### Usuarios del Sistema
- 👤 **Paciente** → App Móvil
- 🩺 **Médico** → Panel Web
- 🔐 **Administrador** → Panel Web (validación REThUS, gobierno)

### Seguridad
- RBAC estricto por rol (paciente, médico, admin).
- Auditoría de acciones sensibles (cambios de prioridad, cierre de consulta, validación REThUS).
- Guardrails de IA: bloqueo automático de contenido de diagnóstico/prescripción.
- Datos mínimos necesarios, trazables y bajo control de acceso por rol.

---

## 9. Inteligencia Artificial — El Corazón del Producto

### Componentes de IA
| Componente | Función |
|---|---|
| **Gemini (LLM)** | Motor de lenguaje para análisis de síntomas y generación de resúmenes |
| **RAG (Retrieval-Augmented Generation)** | Recupera contexto clínico relevante del índice vectorial para enriquecer respuestas |
| **Motor de Reglas Clínicas** | Detecta red flags por especialidad y clasifica prioridad (`LOW/MODERATE/HIGH`) |
| **IAProviderAdapter** | Capa de abstracción que previene lock-in con un solo proveedor de IA |

### Guardrails de Seguridad Clínica
- La IA **nunca** emite un diagnóstico.
- La IA **nunca** prescribe medicamentos.
- Cada respuesta incluye **disclaimer** clínico.
- Auditoría completa de prompts y respuestas.
- Si Gemini no está disponible: fallback a proveedor LLM compatible sin rediseño de dominio.

### Tipos Clave (Contratos de IA)
```
PriorityLevel  = LOW | MODERATE | HIGH
Specialty      = GENERAL_MEDICINE | DENTISTRY

RedFlag {
  code, specialty, severity (MEDIUM | HIGH | CRITICAL), evidence[]
}

ClinicalSummary {
  chiefComplaint, duration, intensity,
  associatedSymptoms[], currentMedication[],
  relevantHistory[], estimatedPriority, redFlags[]
}
```

---

## 10. Modelo de Negocio (Business Model Canvas)

### Propuesta de Valor Central
> Reducir el tiempo improductivo del médico en recolección inicial de información, mejorar la priorización de casos y garantizar continuidad del cuidado del paciente.

| Bloque Canvas | Detalle |
|---|---|
| **Segmentos de Clientes** | Pacientes (ambulatorio), Médicos de Medicina General y Odontología, Administradores de clínicas |
| **Propuesta de Valor** | Triage IA + resumen clínico automático + chat en tiempo real + seguimiento evolutivo |
| **Canales** | App móvil (paciente), Web (médico/admin) |
| **Relación con Clientes** | Autoservicio guiado + alertas automatizadas + seguimiento proactivo |
| **Fuentes de Ingresos** | Pago por consulta + suscripción mensual (modelo híbrido, validado con simulación) |
| **Recursos Clave** | IA (Gemini + RAG), equipo clínico-técnico, base de conocimiento médico validado |
| **Actividades Clave** | Desarrollo de flujos de triage, mantenimiento de motor de reglas clínicas, observabilidad |
| **Socios Clave** | Google (Gemini), Azure (infraestructura), profesionales de salud para validación de contenido |
| **Estructura de Costos** | Desarrollo del producto, infraestructura cloud, costos de API de IA, validación clínica |

### Supuestos de Negocio Validables
1. Los pacientes aceptan responder flujos guiados si perciben beneficio de tiempo.
2. Los médicos usan más la plataforma si la calidad del resumen inicial es consistente.
3. El modelo híbrido de monetización puede validarse inicialmente con simulación.

---

## 11. Métricas e Indicadores de Éxito

### KPIs de Negocio (4 Obligatorios)

| KPI | Fórmula | Meta |
|---|---|---|
| **Tiempo a primera respuesta médica** | Promedio(`timestamp_primera_respuesta − timestamp_apertura`) | Reducir 30% vs baseline Sprint 2 |
| **Utilidad del resumen clínico** | % consultas donde el médico marca el resumen como útil | ≥ 75% |
| **Red flags relevantes confirmadas** | % red flags HIGH confirmadas por médico | ≥ 60% |
| **Retención de seguimiento a 7 días** | % pacientes que completan seguimiento en 7 días | ≥ 50% |

### SLOs Técnicos

| Métrica | Objetivo |
|---|---|
| P95 Latencia API | < 1.500 ms |
| Tiempo de entrega de mensaje WebSocket | < 1.500 ms |
| Tiempo de generación de resumen IA | < 15.000 ms |
| Error Rate (respuestas 5xx) | < 2.0% |
| Disponibilidad mensual | > 99.0% |
| Sesiones concurrentes | Medición continua |

### Dashboard Operacional
- **Panel Técnico:** latencia P95 por endpoint, throughput/minuto, error rate, usuarios concurrentes, estado de WebSocket Hub.
- **Panel de Negocio:** los 4 KPIs anteriores, segmentados por especialidad, prioridad de caso y rango de fechas.

---

## 12. Diferenciación Competitiva

| Dimensión | Chat médico simple | Telemedicina tradicional | **SaludDeUna** |
|---|---|---|---|
| Priorización clínica previa | ❌ | ❌ | ✅ Triage IA + red flags |
| Resumen clínico automático | ❌ | ❌ | ✅ Generado antes de la consulta |
| Seguimiento estructurado | ❌ | Parcial | ✅ Timeline evolutivo |
| Dashboard de eficiencia clínica | ❌ | Parcial | ✅ 4 KPIs + métricas técnicas |
| Verificación del profesional (REThUS) | ❌ | Variable | ✅ Validación semiautomática |
| Guardrails éticos de IA | ❌ | N/A | ✅ Integrados al modelo |

---

## 13. Épicas y Funcionalidades del Producto

| Épica | Nombre | Resultado de Negocio |
|---|---|---|
| **E1** | Onboarding y Acceso Seguro | Incorporar usuarios válidos con control de rol |
| **E2** | Triage Inteligente por Especialidad | Priorizar casos y reducir incertidumbre del paciente |
| **E3** | Consulta Clínica en Tiempo Real | Mejorar velocidad y continuidad de atención |
| **E4** | Resumen Clínico y Traductor IA | Aumentar eficiencia médica y claridad de comunicación |
| **E5** | Seguimiento y Evolución del Paciente | Medir progresión y detectar cambios de riesgo |
| **E6** | Observabilidad y Analítica | Medir salud técnica y valor de negocio |
| **E7** | Monetización Simulada y Gobierno | Validar viabilidad del modelo y controles operativos |

### Priorización MoSCoW

**🔴 Must (núcleo del MVP)**
- Registro/login paciente y médico con RBAC.
- Verificación semiautomática REThUS.
- Triage IA + red flags + prioridad en 2 especialidades.
- Resumen clínico automático.
- Chat en tiempo real y cola clínica.
- Seguimiento post-consulta y timeline.
- Logs estructurados + métricas + dashboard.
- 4 KPIs de negocio implementados y medibles.
- Flujo de monetización simulado.

**🟡 Should (mejora experiencia)**
- Banco de conocimiento validado con aprobaciones médicas.
- Traducción bidireccional de lenguaje clínico (paciente ↔ clínico).
- Alertas inteligentes de seguimiento.

**🟢 Could (diferenciación futura)**
- Recomendaciones de contenido preventivo personalizadas.
- Versionado avanzado de prompts y evaluación automática offline.

**⚫ Won't (fuera del alcance del semestre)**
- Videollamadas en vivo.
- Prescripción/diagnóstico automático.
- Integraciones clínicas productivas certificadas.
- Pasarela real de pagos productiva.

---

## 14. Roadmap — Plan de 10 Sprints

| Sprint | Objetivo | Entregables Clave |
|---|---|---|
| **Sprint 0** (Inception) | Definición técnica y funcional | Wiki completa, Canvas, arquitectura base, backlog INVEST, Gherkin, DoR/DoD, MoSCoW |
| **Sprint 1** | Base técnica y acceso seguro | Auth paciente/médico, RBAC, flujo admin REThUS, repositorio y CI base |
| **Sprint 2** | Triage Medicina General | Flujo guiado MG, análisis IA, prioridad LOW/MODERATE/HIGH |
| **Sprint 3** | Triage Odontología + red flags | Reglas odontológicas, ajuste priorización, prueba inicial concurrencia WS |
| **Sprint 4** | Resumen clínico IA | Resumen preconsulta, guardrails IA, telemetría de modelo |
| **Sprint 5** | Consulta en tiempo real | Chat WS funcional, estado de consulta, cola médica priorizada |
| **Sprint 6** | Seguimiento y timeline | Formularios post-consulta, línea de evolución, alertas por empeoramiento |
| **Sprint 7** | Observabilidad y dashboard | Logs estructurados, métricas técnicas, alertas y 4 KPIs de negocio visibles |
| **Sprint 8** | Monetización simulada | Checkout simulado y, si hay capacidad, banco de conocimiento validado |
| **Sprint 9** | Hardening y cierre | Pruebas E2E, carga final, correcciones, demo integral para presentación final |

### Hitos Clave
1. ✅ Inception completo: hasta 4 de marzo de 2026.
2. 🎯 Presentación Inception: 9 de marzo de 2026.
3. 🔗 Primer flujo end-to-end funcional (registro → triage → resumen → cola): cierre Sprint 5.
4. 📊 Observabilidad completa (técnica + negocio): cierre Sprint 7.
5. 🚀 MVP estable para cierre académico: cierre Sprint 9.

---

## 15. Alcance del MVP y Lo Que No Es

### ✅ Dentro del MVP
- App paciente en React Native.
- Panel médico/admin web en React + Next.
- Backend NestJS + MongoDB.
- IA con Gemini + RAG + reglas clínicas de seguridad.
- Triage predictivo no diagnóstico con prioridad `LOW|MODERATE|HIGH`.
- Red flags por especialidad (Medicina General y Odontología).
- Resumen clínico automático preconsulta.
- Chat clínico asíncrono con actualizaciones en tiempo real (WebSocket).
- Seguimiento post-consulta y evolución del caso.
- Flujo de monetización simulado.
- Validación semiautomática REThUS por admin.
- Observabilidad: logs estructurados, métricas técnicas, dashboard y 4 KPIs de negocio.

### ❌ Fuera del MVP
- Diagnóstico o prescripción automática por IA.
- Videoconsulta en vivo.
- Integración real con pasarela de pagos productiva.
- Integración oficial directa certificada con sistemas hospitalarios/HCE productivos.

---

## 16. Contexto Regulatorio y Escalabilidad

| Aspecto | Detalle |
|---|---|
| **Mercado objetivo inicial** | Colombia (ambulatorio, precomercial) |
| **Visión de expansión** | Diseño escalable a Latinoamérica |
| **Manejo de datos** | Datos sintéticos en desarrollo · datos anonimizados en demo |
| **Trazabilidad** | Datos sensibles mínimos, trazables y con control de acceso por rol |
| **Cumplimiento clínico** | Guardrails estrictos, disclaimers y auditoría de prompts/respuestas |
| **Multi-cloud** | Azure institucional + IAProviderAdapter (adaptable a cualquier LLM) |

---

## 17. Gestión de Riesgos

| ID | Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|---|
| R-001 | Sobrecarga de alcance en 10 sprints | Media | Alta | Congelar Must en Sprint 1, aplicar regla de corte MoSCoW |
| R-002 | Latencia alta en chat en tiempo real | Media | Alta | Optimizar payloads WS, pruebas de carga desde Sprint 3 |
| R-003 | Resumen IA inestable por prompts | Media | Alta | Versionar prompts y pruebas de regresión semántica |
| R-004 | Cambio de condiciones del proveedor IA | Baja | Media | Capa `IAProviderAdapter` para reducir lock-in |
| R-005 | Uso indebido del rol médico | Media | Alta | Validación REThUS semiautomática + RBAC estricto |
| R-006 | Exposición de datos sensibles | Baja | Alta | Minimización de datos, anonimizado y auditoría |
| R-007 | Caída parcial de servicios dashboard | Media | Media | Modo degradado, caché temporal y alertamiento |

---

## 18. DevOps y Gobernanza

| Aspecto | Configuración |
|---|---|
| **Control de versiones** | GitHub (monorepo) — ramas `main`, `develop`, `feature/*` |
| **Integración Continua** | GitHub Actions: lint, test, build y deploy por ambiente |
| **Gestión de Proyecto** | Azure DevOps: Wiki, Boards, HU, trazabilidad de sprint |
| **Ambientes** | `dev` y `staging-demo` |
| **Gestión de Secretos** | Variables seguras en Azure/GitHub |
| **Wiki** | https://github.com/JesusJC15/salud-de-una-wiki |

---

## 19. Resumen Ejecutivo para el Pitch

> **SaludDeUna** es una plataforma de comunicación clínica asistida por IA que resuelve la ineficiencia informacional en la atención ambulatoria. Conecta pacientes y médicos con triage predictivo no diagnóstico, estructuración clínica automática y seguimiento inteligente.

### Los 5 porqués para invertir o adoptar SaludDeUna:
1. **Problema real y masivo:** La ineficiencia en la recolección de antecedentes clínicos afecta millones de consultas diariamente en Latinoamérica.
2. **IA con guardrails:** No diagnóstica, no prescribe — pero sí prioriza, estructura y hace seguimiento mejor que cualquier sistema actual.
3. **Doble tracción:** Beneficia simultáneamente al paciente (mejor experiencia) y al médico (mayor eficiencia).
4. **Modelo de negocio validable:** Pago por consulta + suscripción, medible desde el primer sprint.
5. **Escalabilidad técnica y geográfica:** Arquitectura desacoplada con adapter de IA, diseñada para LatAm desde Colombia.

---

*Documento generado a partir de la documentación oficial del proyecto SaludDeUna — IETI 2026-1.*
*Fuentes: Plan Maestro, Wiki del Proyecto (docs/wiki/), Épicas Azure Boards, Business Model Canvas.*
