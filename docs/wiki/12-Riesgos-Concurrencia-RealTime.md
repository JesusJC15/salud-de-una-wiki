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
- Availability mensual: `> 99%`.

## Escenarios de Calidad de Arquitectura (formato formal)
| Atributo | Estimulo | Entorno | Artefacto afectado | Respuesta del sistema | Medida de respuesta |
|---|---|---|---|---|---|
| Escalabilidad | 100 usuarios concurrentes solicitan triage y chat | Operacion normal en horario pico | API + WebSocket Hub | El sistema mantiene cola priorizada y entrega mensajes sin perdida | Tiempo de respuesta `< 2 s` en operaciones interactivas (objetivo interno `< 1500 ms`) |
| Disponibilidad | Falla transitoria de servicio externo de IA | Produccion durante consultas activas | IAOrchestrator + FollowupModule | El sistema entra en modo degradado, conserva flujo clinico y registra incidente | Disponibilidad mensual `> 99%` |
| Modificabilidad | Se integra nuevo proveedor LLM | Sprint de mejora tecnica | `IAProviderAdapter` | El cambio se encapsula sin afectar dominio clinico ni API publica | Cambios limitados a modulos de integracion IA |
| Seguridad | Intento de acceso de medico no verificado a cola clinica | Operacion normal | Auth/RBAC + endpoints de consulta | Se deniega acceso y se registra auditoria | `HTTP 403` + evento de auditoria trazable |
| Usabilidad | Paciente ejecuta tarea principal (login -> triage -> envio) | App movil Android | UX de app React Native | Flujo guiado con validacion de campos y feedback inmediato | Tarea principal en `<= 3 interacciones principales` por pantalla |

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
