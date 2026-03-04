## Objetivo
Identificar riesgos tecnicos/funcionales del proyecto y definir la estrategia concreta para concurrencia y operaciones en tiempo real.

## Alcance
Incluye registro de riesgos, mitigaciones, plan de pruebas de carga y lineamientos de resiliencia del chat clinico y servicios IA.

## Registro de Riesgos
| ID | Riesgo | Probabilidad | Impacto | Mitigacion |
|---|---|---|---|---|
| R-001 | Sobrecarga de alcance en 10 sprints | Media | Alta | Congelar Must en Sprint 1 y aplicar regla de corte MoSCoW |
| R-002 | Latencia alta en chat real-time | Media | Alta | Optimizar payloads WS, pruebas de carga desde Sprint 3, tuning por sprint |
| R-003 | Resumen IA inestable por prompts | Media | Alta | Versionar prompts y pruebas de regresion semantica |
| R-004 | Cambio de condiciones de proveedor IA | Baja | Media | Capa `IAProviderAdapter` para reducir lock-in |
| R-005 | Uso indebido de rol medico | Media | Alta | Validacion REThUS semiautomatica + RBAC estricto |
| R-006 | Exposicion de datos sensibles | Baja | Alta | Minimizacion de datos, anonimizado y auditoria |
| R-007 | Caida parcial de servicios dashboard | Media | Media | Modo degradado, cache temporal y alertamiento |

## Estrategia de Concurrencia y Tiempo Real

### Canal de chat
- WebSocket con autenticacion de sesion y autorizacion por consulta.
- Entrega de mensajes con ACK logico para confirmar recepcion.
- Reintento controlado y sincronizacion al reconectar.

### Manejo de carga
- Limite de conexiones por instancia.
- Escalamiento horizontal del backend segun sesiones concurrentes.
- Colas internas para operaciones costosas de IA cuando aplique.

### Consistencia de estado
- Estado de consulta centralizado en backend.
- Eventos `consultation.status.changed` y `consultation.priority.updated` como fuente de verdad de UI.
- Timestamp de servidor para orden cronologico.

## Objetivos de Rendimiento (SLO)
- P95 API Latency: `< 1500 ms`.
- Chat delivery: `< 1500 ms`.
- AI summary time: `< 15000 ms`.
- Availability mensual: `>= 98%`.

## Plan de Pruebas de Concurrencia
| Fase | Prueba | Criterio de salida |
|---|---|---|
| Sprint 3 | Smoke de WebSocket con 20 sesiones concurrentes | Sin perdida de mensajes |
| Sprint 5 | Carga intermedia 100 sesiones + triage simultaneo | P95 dentro de objetivo |
| Sprint 7 | Carga combinada chat + resumen IA + dashboard | Error rate < 2% |
| Sprint 9 | Prueba final de endurance 60 minutos | Disponibilidad estable y sin degradacion severa |

## Plan de Contingencia
- Si falla IA: permitir respuesta medica con plantilla manual y marcar estado `AI_UNAVAILABLE`.
- Si falla WebSocket: fallback a polling corto temporal.
- Si falla dashboard: mantener log crudo exportable para seguimiento.