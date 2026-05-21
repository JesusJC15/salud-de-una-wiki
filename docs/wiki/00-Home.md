## Objetivo
Centralizar la documentacion oficial del proyecto `SaludDeUna` para cumplir todos los requisitos del proyecto de `IETI-2026-I`.

## Alcance
Este documento define el indice, el estado ejecutivo y la trazabilidad general de la documentacion.

## Responsables
- `Jesús Alberto Jauregui Conde`
- `Mayerlly Suaréz Correa`
- `Natalia Espitia Espinel`
- `Santiago Hurtado Martínez`

## Resumen Ejecutivo
`SaludDeUna` es una plataforma de comunicacion y priorizacion clinica asistida por IA que conecta pacientes y medicos mediante triage guiado, chat en tiempo real, seguimiento post-consulta y facturacion simulada.

El MVP implementado cubre:
- 3 especialidades de triage: Medicina General, Odontologia y Urgencias.
- App movil para pacientes (React Native / Expo 55): triage, consultas, chat clinico, followups, notificaciones y perfil.
- Panel web para staff (Next.js 16): flujos de doctor (cola de consultas, chat, resumen IA) y administrador (usuarios, verificacion REThUS, billing, dashboard KPIs).
- Backend NestJS 11 con MongoDB Atlas, Redis, WebSocket (Socket.IO), modulo IA (RAG + Gemini 2.5-flash + reglas clinicas), outbox transaccional y billing simulado.
- Observabilidad: logs estructurados, metricas tecnicas (p95 latencia, error rate), 4 KPIs de negocio, preparacion OpenTelemetry.
- 406+ tests en backend (cobertura statements 93%, branches 80%); 82 tests en web.

## Estado del Proyecto
- **Estado actual:** `Version Final — Proyecto completado (mayo 2026)`.
- **Sustentacion IETI-2026-I:** `21 de mayo de 2026`.
- **Funcionalidad core:** ~95% implementada y probada (triage → consulta → chat → followup).
- **Billing simulado:** completado el 2026-05-09 (checkout, transacciones, metricas de revenue, admin CRUD de precios).
- **Deuda tecnica pendiente:** produccion/infraestructura (Redis adapter Socket.IO, CI/CD deploy, EAS Build, Auth0 mobile), no de funcionalidad.

## Convencion de Sprints (alineacion lineamientos 2026-1)
Se mantuvo el esquema operativo del equipo `Sprint 0..9` con equivalencia al lineamiento oficial:

| Esquema interno | Esquema lineamiento |
|---|---|
| Sprint 0 (Inception) | Inception |
| Sprint 1 | Sprint 1 |
| Sprint 2 | Sprint 2 |
| Sprint 3 | Sprint 3 |
| Sprint 4 | Sprint 4 |
| Sprint 5 | Sprint 5 |
| Sprint 6 | Sprint 6 |
| Sprint 7 | Sprint 7 |
| Sprint 8 | Sprint 8 |
| Sprint 9 | Sprint 10 |

## Indice Navegable
1. [Producto, equipo y Resumen](01-Producto-Equipo-Resumen.md)

2. [Descripcion, Problema y Contexto](02-Descripcion-Problema-Contexto.md)

3. [Modelo Canvas](03-Modelo-Canvas.md)

4. [Arquitectura y Despliegue Base](04-Arquitectura-Despliegue-Base.md)

5. [Epicas, Features y Story Map](05-Epicas-Features-StoryMap.md)

6. [Backlog de Historias de Usuario](06-Backlog-Historias-Usuario.md)

7. [Criterios de Aceptacion en Gherkin](07-Criterios-Aceptacion-Gherkin.md)

8. [Definition of Ready y Definition of Done](08-DoR-DoD.md)

9. [Observabilidad y KPIs](09-Observabilidad-KPIs.md)

10. [Priorizacion MoSCoW](10-Priorizacion-MoSCoW.md)

11. [Plan de Sprints 0 a 9](11-Plan-Sprints-0-a-9.md)

12. [Riesgos, Concurrencia y Real-Time](12-Riesgos-Concurrencia-RealTime.md)

13. [Anexos y Referencias](13-Anexos-Referencias.md)

14. [Mockups y Flujos Moviles](14-Mockups-Flujos-Moviles.md)

15. [Cumplimiento de Lineamientos 2026-1](15-Cumplimiento-Lineamientos-2026-1.md)

16. [Plan Tecnico Actualizado — Version Final (mayo 2026)](16-Plan-Tecnico-Actualizado-2026-05.md)

## Documentos Azure Boards (detalle por epica)
1. [Epica 1 - Onboarding y Acceso Seguro](../epics/Azure-Boards-Epica1.md)
2. [Epica 2 - Triage Inteligente por Especialidad](../epics/Azure-Boards-Epica2.md)
3. [Epica 3 - Consulta Clinica en Tiempo Real](../epics/Azure-Boards-Epica3.md)
4. [Epica 4 - Resumen Clinico y Traductor IA](../epics/Azure-Boards-Epica4.md)
5. [Epica 5 - Seguimiento y Evolucion del Paciente](../epics/Azure-Boards-Epica5.md)
6. [Epica 6 - Observabilidad y Analitica](../epics/Azure-Boards-Epica6.md)
7. [Epica 7 - Monetizacion Simulada y Gobierno](../epics/Azure-Boards-Epica7.md)

## Diagramas Arquitectonicos
- [Diagramas Mermaid — C4, RAG, Backend, AWS, Auth, Datos, CI/CD, Observabilidad](../../diagrams/DIAGRAMAS-MERMAID.md)
