# SaludDeUna - Entregables de sustentación

## Archivos

- `SaludDeUna-Sustentacion-Tecnica.pptx`: presentación PowerPoint editable con notas del expositor.
- `presentacion-web/index.html`: presentación HTML interactiva sin dependencias CDN.

## Uso de la versión web

Abrir `presentacion-web/index.html` directamente o publicarlo en GitHub Pages.

Controles:

- Flechas / PageUp / PageDown / espacio: navegación.
- `F`: pantalla completa.
- `B`: mostrar u ocultar slides backup.
- Imprimir desde el navegador para exportar a PDF.

## Estructura de slides

Cada slide del PowerPoint incluye notas de expositor con objetivo, mensaje principal, guion sugerido y animación sugerida.

| # | Tipo | Sección | Objetivo visual | Mensaje principal |
|---|------|---------|-----------------|-------------------|
| 1 | Principal | APERTURA | SaludDeUna | Una plataforma real para priorizar, estructurar y comunicar información clínica antes de la consulta. |
| 2 | Principal | NARRATIVA | El recorrido prueba problema, solución y solidez técnica. | La sustentación se concentra en 7 bloques con demo en vivo como evidencia final. |
| 3 | Principal | PROBLEMA | La consulta pierde valor cuando la información llega tarde o incompleta. | Pacientes no saben estructurar síntomas; médicos pierden tiempo reconstruyendo contexto clínico. |
| 4 | Principal | IMPACTO | El mismo desorden afecta al paciente, al médico y al sistema. | La oportunidad está en convertir síntomas dispersos en contexto clínico accionable. |
| 5 | Principal | SOLUCIÓN | SaludDeUna funciona como una capa inteligente entre paciente y médico. | Triage asistido, resumen clínico y seguimiento post-consulta reducen fricción sin reemplazar al profesional. |
| 6 | Principal | MVP | El MVP cubre tres roles y nueve casos de uso principales. | Paciente mobile, médico web y administrador web comparten un backend clínico con flujo E2E implementado. |
| 7 | Principal | FLUJO | El negocio se sostiene en un flujo E2E trazable. | Cada consulta nace en el triage, pasa por pago simulado y cola médica, y termina con cierre y seguimiento. |
| 8 | Principal | ARQUITECTURA | La arquitectura separa experiencia, dominio clínico, IA y datos. | Expo, Next.js y NestJS operan sobre MongoDB, Redis, Gemini, Auth0 y servicios externos con límites claros. |
| 9 | Principal | CLOUD | El despliegue operativo es simple; la ruta de escala ya está diseñada. | Railway + Vercel cubren el MVP; AWS ECS Fargate + ALB queda como arquitectura objetivo. |
| 10 | Principal | BACKEND | El backend agrupa 18 módulos NestJS por dominio. | Los límites de módulo permiten mantener seguridad, IA, clínica, billing, outbox y observabilidad sin acoplar responsabilidades. |
| 11 | Principal | IA PRODUCTO | La IA prioriza y resume; no diagnostica ni prescribe. | Gemini 2.5-flash genera análisis, pero un guardrail determinista bloquea salidas clínicas indebidas. |
| 12 | Principal | CALIDAD | Seis criterios de calidad están implementados con evidencia técnica. | Seguridad, disponibilidad, escalabilidad, mantenibilidad, observabilidad y rendimiento tienen mecanismos concretos. |
| 13 | Principal | SEGURIDAD | La defensa en profundidad empieza antes del controlador. | JWT/Auth0, RBAC, verificación médica, rate limiting, CORS, Helmet y correlationId protegen datos y trazabilidad. |
| 14 | Principal | CONFIABILIDAD | Testing y observabilidad hacen defendible el funcionamiento real. | 406 tests backend, E2E con MongoDB real, logs JSON, OpenTelemetry y KPIs técnicos reducen riesgo operativo. |
| 15 | Principal | IA DESARROLLO | La IA también aceleró arquitectura, código, tests y documentación. | Claude Code, Copilot y ChatGPT se usaron como copilotos, con decisiones revisadas por el equipo. |
| 16 | Principal | DEMO | La demo prueba el flujo completo con APK, QR, web y backend. | Paciente completa triage, paga, entra a cola; médico atiende por chat y genera resumen; admin valida KPIs. |
| 17 | Principal | VIABILIDAD | El producto tiene base técnica para evolucionar de MVP a plataforma. | Billing simulado, KPIs, modularidad, Redis, outbox y cloud objetivo reducen el camino hacia mercado. |
| 18 | Principal | CIERRE | SaludDeUna resuelve un problema real con software medible. | La combinación de IA controlada, arquitectura modular y demo funcional sostiene la defensa técnica. |
| 19 | Backup | BACKUP C4 | Contexto C4: actores humanos y sistemas externos. | Paciente, médico y administrador interactúan con un sistema central apoyado por Gemini, MongoDB, Redis, Auth0 y REThUS. |
| 20 | Backup | BACKUP C4 | Contenedores C4: frontends, API, worker y datos. | La separación API/worker mantiene el request-response liviano y mueve followups/outbox a jobs. |
| 21 | Backup | BACKUP CLOUD | Despliegue objetivo AWS: ECS Fargate + ALB. | La arquitectura objetivo soporta escalamiento horizontal y sticky sessions para WebSocket. |
| 22 | Backup | BACKUP DATOS | Modelo de datos: usuarios, billing y dominio clínico. | MongoDB permite registros clínicos polimórficos y Atlas Vector Search habilita RAG. |
| 23 | Backup | BACKUP IA | Pipeline RAG clínico y guardrails. | El contexto recuperado mejora la respuesta, pero el guardrail conserva límites clínicos. |
| 24 | Backup | BACKUP DEVOPS | CI/CD y observabilidad reducen riesgo operacional. | GitHub Actions, Railway/Vercel/EAS y OpenTelemetry forman la ruta de entrega y diagnóstico. |
| 25 | Backup | BACKUP JURADO | Preguntas probables: IA, seguridad, escala y costos. | Las respuestas clave deben defender límites clínicos, monolito modular, MongoDB, Redis y regulación. |
| 26 | Backup | BACKUP PRODUCTO | Business Model Canvas y roadmap de mercado. | Pago por consulta, especialidades ampliables y alianzas clínicas sostienen la ruta de viabilidad. |

## Recursos reutilizados

- Logos oficiales: `logoSaludDeUna.png`, `iconoSaludDeUna.png`.
- Diagramas SVG técnicos: C4 contexto, C4 contenedores, despliegue AWS, módulos backend, RAG, auth, datos, CI/CD y observabilidad.
- Business Model Canvas del proyecto.

## Criterios explícitamente cubiertos

- Contexto del problema, usuarios afectados, impacto social, oportunidad de mercado y propuesta de valor.
- Casos de uso: registro pacientes/médicos, síntomas, triage, gestión clínica, historial, chat, seguimiento y administración.
- Arquitectura: mobile, web, backend, IA, MongoDB, Redis, cloud, Auth0, REThUS, observabilidad y APIs.
- Calidad: seguridad, disponibilidad, escalabilidad, mantenibilidad, observabilidad y rendimiento.
- IA en producto y desarrollo: Gemini, RAG, guardrails, generación de código, testing, documentación, UX/UI y automatización.
- Demo: APK/QR, flujo E2E y contingencias.
- Backup técnico: preguntas del jurado, costos, seguridad, escalabilidad, regulación y decisiones arquitectónicas.
