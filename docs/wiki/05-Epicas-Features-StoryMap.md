## Objetivo
Organizar el producto en epicas y features para garantizar trazabilidad de valor y facilitar la planificacion Scrum del semestre.

## Alcance
Incluye mapeo de epicas/features y un story map textual por flujo de usuario. No reemplaza el backlog detallado de historias.

## Mapa de Epicas
| ID Epica | Nombre | Resultado de negocio | Documento |
|---|---|---|---|
| E1 | Onboarding y acceso seguro | Incorporar usuarios validos con control de rol | [E1](../epics/Azure-Boards-Epica1.md) |
| E2 | Triage inteligente por especialidad | Priorizar casos y reducir incertidumbre del paciente | [E2](../epics/Azure-Boards-Epica2.md) |
| E3 | Consulta clinica en tiempo real | Mejorar velocidad y continuidad de atencion | [E3](../epics/Azure-Boards-Epica3.md) |
| E4 | Resumen clinico y traductor IA | Aumentar eficiencia medica y claridad de comunicacion | [E4](../epics/Azure-Boards-Epica4.md) |
| E5 | Seguimiento y evolucion | Medir progresion y detectar cambios de riesgo | [E5](../epics/Azure-Boards-Epica5.md) |
| E6 | Observabilidad y analitica | Medir salud tecnica y valor de negocio | [E6](../epics/Azure-Boards-Epica6.md) |
| E7 | Monetizacion simulada y gobierno | Validar viabilidad de modelo y controles operativos | [E7](../epics/Azure-Boards-Epica7.md) |

## Feature Mapping por Epica
| Epica | Feature ID | Feature | Prioridad |
|---|---|---|---|
| E1 | F1.1 | Registro/login paciente | Must |
| E1 | F1.2 | Registro medico + verificacion REThUS admin | Must |
| E2 | F2.1 | Flujo triage Medicina General | Must |
| E2 | F2.2 | Flujo triage Odontologia | Must |
| E2 | F2.3 | Motor de red flags por especialidad | Must |
| E3 | F3.1 | Chat clinico en tiempo real | Must |
| E3 | F3.2 | Cola de casos priorizada para medico | Must |
| E4 | F4.1 | Resumen clinico automatico preconsulta | Must |
| E4 | F4.2 | Traduccion paciente-clinico bidireccional | Should |
| E5 | F5.1 | Seguimiento post-consulta automatizado | Must |
| E5 | F5.2 | Timeline evolutivo por paciente | Must |
| E6 | F6.1 | Logs estructurados y auditoria | Must |
| E6 | F6.2 | Metricas de concurrencia/rendimiento | Must |
| E6 | F6.3 | Dashboard tecnico y de negocio | Must |
| E7 | F7.1 | Flujo de pago simulado | Must |
| E7 | F7.2 | Banco de conocimiento validado | Should |

## Story Map (texto estructurado)

### Actividad 1: Onboarding
- Usuario paciente se registra.
- Usuario medico se registra.
- Admin valida medico contra REThUS.

### Actividad 2: Captura y priorizacion
- Paciente selecciona especialidad.
- Paciente responde preguntas guiadas.
- Sistema detecta red flags y prioridad.

### Actividad 3: Consulta
- Se crea caso y entra a cola medica.
- Medico recibe resumen clinico.
- Paciente y medico interactuan por chat en tiempo real.

### Actividad 4: Continuidad
- Sistema dispara seguimiento.
- Paciente reporta evolucion.
- Medico revisa timeline y ajusta plan.

### Actividad 5: Analitica y operacion
- Plataforma registra logs y metricas.
- Dashboard muestra KPIs tecnicos y de negocio.
- Equipo revisa alertas y acciones de mejora.

## Slices de Entrega (por release interno)
| Slice | Cobertura | Sprints objetivo |
|---|---|---|
| Slice A | E1 + E2 base | 1 a 3 |
| Slice B | E3 + E4 | 4 a 5 |
| Slice C | E5 + E6 | 6 a 7 |
| Slice D | E7 + hardening | 8 a 9 |

## Matriz de Trazabilidad Transversal
| Actividad Story Map | Historias | Criterios Gherkin | Sprint objetivo | KPI/SLO impactado | Riesgo asociado |
|---|---|---|---|---|---|
| Actividad 1: Onboarding | HU-001, HU-002 | HU-001, HU-002 | 1 | SLO Availability, KPI Retencion 7 dias (habilitador) | R-005 |
| Actividad 2: Captura y priorizacion | HU-003, HU-004 | HU-003, HU-004 | 2-3 | KPI Red flags confirmadas, SLO P95 API | R-003, R-006 |
| Actividad 3: Consulta | HU-005, HU-006 | HU-005, HU-006 | 4-5 | KPI Tiempo primera respuesta, SLO Chat Delivery | R-002, R-003 |
| Actividad 4: Continuidad | HU-007 | HU-007 | 6 | KPI Retencion 7 dias, SLO P95 API | R-002 |
| Actividad 5: Analitica y operacion | HU-008, OBS-001..004 | HU-008 | 7 | KPI Utilidad resumen, SLO Availability, Error Rate | R-007 |
| Actividad 5: Monetizacion y gobierno | HU-010, HU-011 | HU-010 (HU-011 al comprometerse) | 8 | KPI Conversion simulada (complementario), KPI Tiempo primera respuesta (indirecto) | R-001 |
