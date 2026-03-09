## Objetivo
Definir la ejecucion de 10 sprints del semestre con metas, entregables y criterios de salida por sprint.

## Alcance
Incluye Sprint 0 (Inception) y Sprint 1 a 9 (construccion, hardening y cierre).

## Equivalencia con Lineamiento 2026-1
Para mantener continuidad historica del equipo se conserva `Sprint 0..9` y se mapea al esquema del lineamiento (`Inception + Sprint 1..10`):

| Esquema interno | Lineamiento |
|---|---|
| Sprint 0 | Inception |
| Sprint 1 | Sprint 1 |
| Sprint 2 | Sprint 2 |
| Sprint 3 | Sprint 3 |
| Sprint 4 | Sprint 4 |
| Sprint 5 | Sprint 5 |
| Sprint 6 | Sprint 6 |
| Sprint 7 | Sprint 7 |
| Sprint 8 | Sprint 8 |
| Sprint 9 | Sprint 10 |

## Cadencia Scrum
- Sprint planning semanal.
- Daily scrum diario.
- Review y retrospectiva al cierre de cada sprint.
- Refinement de backlog una vez por sprint.

## Roadmap por Sprint
| Sprint | Objetivo principal | Historias foco | Entregables clave |
|---|---|---|---|
| Sprint 0 | Definicion tecnica y funcional | Documentacion inicial | Wiki completa Inception, Canvas, arquitectura base, backlog INVEST, Gherkin, DoR/DoD, MoSCoW |
| Sprint 1 | Base tecnica y acceso seguro | HU-001, HU-002 | Auth paciente/medico, RBAC, flujo admin de verificacion REThUS, repositorio y CI base |
| Sprint 2 | Triage Medicina General | HU-003 | Flujo guiado MG, analisis IA inicial, prioridad LOW/MODERATE/HIGH |
| Sprint 3 | Triage Odontologia + red flags | HU-004 | Reglas odontologicas, ajuste priorizacion, prueba inicial concurrencia WS |
| Sprint 4 | Resumen clinico IA | HU-005 | Resumen preconsulta, guardrails IA, telemetria de modelo |
| Sprint 5 | Consulta en tiempo real | HU-006 | Chat WS funcional, estado de consulta, cola medica priorizada |
| Sprint 6 | Seguimiento y timeline | HU-007 | Formularios post-consulta, linea de evolucion, alertas por empeoramiento |
| Sprint 7 | Observabilidad y dashboard | HU-008, OBS-001..004 | Logs estructurados, metricas tecnicas, 4 KPIs de negocio visibles |
| Sprint 8 | Monetizacion simulada y knowledge | HU-010, HU-011 | Checkout simulado, gestion de contenido validado medico |
| Sprint 9 | Hardening y cierre | Ajustes Must pendientes | Pruebas E2E, carga final, correcciones, evidencia para presentacion final |

## Definition of Done por Sprint (resumen)
- Sprint 0: documentos obligatorios completos y presentables.
- Sprint 1 a 8: historias comprometidas con DoD completo.
- Sprint 9: cierre de deuda critica, resultados de performance y demo integral.

## Hitos Clave
1. Inception completo: hasta 4 marzo 2026.
2. Presentacion Inception: 9 marzo 2026.
3. Primer flujo end-to-end funcional (registro -> triage -> resumen -> cola): al cierre de Sprint 5.
4. Observabilidad completa (tecnica + negocio): al cierre de Sprint 7.
5. MVP estable para cierre academico: al cierre de Sprint 9.

## Dependencias de Alto Impacto
- Si HU-002 no se cierra temprano, se afecta validacion de rol medico.
- Si HU-005 se retrasa, bloquea HU-006 y se compromete reto de concurrencia/real-time.
- Si HU-008 se retrasa, se compromete evidencia de observabilidad exigida.
