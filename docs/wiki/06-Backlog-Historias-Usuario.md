## Objetivo
Definir el backlog oficial de historias de usuario del MVP con estructura INVEST y campos obligatorios para desarrollo y trazabilidad.

## Alcance
Incluye historias Must, Should, Could y Won't de alcance semestral. Cada historia contiene `ID`, `Como`, `Quiero`, `Para`, prioridad, dependencia y estimacion. Para Azure Boards, cada historia funcional tiene un solo feature parent; cualquier feature adicional se documenta como relacionado.

## Backlog Priorizado
| ID | Epica | Feature | Como | Quiero | Para | MoSCoW | Dependencia | Estimacion (SP) |
|---|---|---|---|---|---|---|---|---|
| HU-001 | E1 | F1.1 | Paciente nuevo | registrarme e iniciar sesion | gestionar mis consultas de forma segura | Must | Ninguna | 5 |
| HU-002 | E1 | F1.2 | Admin de plataforma | validar medico con soporte REThUS | asegurar autenticidad profesional | Must | HU-001 | 8 |
| HU-003 | E2 | F2.1 | Paciente de medicina general | responder un triage guiado por IA | describir mejor sintomas y riesgo | Must | HU-001 | 8 |
| HU-004 | E2 | F2.2 (relacionada: F2.3) | Paciente odontologico | recibir clasificacion de prioridad por red flags | identificar casos que requieren atencion presencial | Must | HU-003 | 8 |
| HU-005 | E4 | F4.1 | Medico | recibir resumen clinico automatico preconsulta | reducir tiempo de recoleccion de antecedentes | Must | HU-003/HU-004 | 8 |
| HU-006 | E3 | F3.1 (relacionada: F3.2) | Paciente y medico | intercambiar mensajes en tiempo real con estado de caso | mejorar continuidad y oportunidad de respuesta | Must | HU-005 | 13 |
| HU-007 | E5 | F5.1 (relacionada: F5.2) | Paciente en seguimiento | reportar evolucion despues de consulta | detectar empeoramiento o mejora oportuna | Must | HU-006 | 8 |
| HU-008 | E6 | F6.3 (relacionadas: F6.1, F6.2) | Equipo de producto | visualizar metricas tecnicas y KPIs de negocio | tomar decisiones basadas en datos | Must | HU-006/HU-007 | 8 |
| HU-009 | E4 | F4.2 | Paciente | recibir explicaciones en lenguaje simple | entender mejor la orientacion medica | Should | HU-005 | 5 |
| HU-010 | E7 | F7.1 | Paciente | simular compra de consulta o plan | validar flujo de monetizacion | Must | HU-001 | 5 |
| HU-011 | E7 | F7.2 | Medico | reutilizar respuestas validadas por pares | reducir consultas repetitivas | Should | HU-005 | 5 |
| HU-012 | E5 | F5.1 | Familiar cuidador | gestionar perfil de menor o adulto mayor | acompanar atencion de terceros | Could | HU-001 | 8 |
| HU-013 | E6 | F6.3 | Equipo de datos | exportar tablero a reporte semanal | seguir tendencias de valor y riesgo | Could | HU-008 | 3 |
| HU-014 | E3 | - | Paciente y medico | tener videollamada integrada | resolver consulta en sincronico completo | Won't | HU-006 | 13 |

## Historias complementarias de observabilidad (Azure Boards)
Estas historias tecnicas complementan HU-008 y permiten trazabilidad separada en Azure Boards para el trabajo de observabilidad. Su estimacion es referencial y debe ajustarse en refinement segun capacidad real del Sprint 7.

| ID | Epica | Feature parent | Como | Quiero | Para | MoSCoW | Dependencia | Estimacion referencial (SP) |
|---|---|---|---|---|---|---|---|---|
| OBS-001 | E6 | F6.1 | Equipo tecnico | registrar logs estructurados con correlation ID | rastrear errores end-to-end | Must | HU-006/HU-007 | 2 |
| OBS-002 | E6 | F6.2 | Equipo tecnico | medir latencia, throughput y concurrencia | controlar rendimiento del sistema | Must | OBS-001 | 3 |
| OBS-003 | E6 | F6.3 | Product Owner | visualizar 4 KPIs de negocio en dashboard | tomar decisiones de producto | Must | OBS-001/OBS-002 | 2 |
| OBS-004 | E6 | F6.2 | Equipo de operacion | recibir alertas por degradacion | responder antes de afectar usuarios | Must | OBS-001/OBS-002 | 2 |


## Historias Principales con IA (obligatorias)
- HU-003: triage IA para Medicina General.
- HU-004: red flags y prioridad IA para Odontologia.
- HU-005: resumen clinico automatico con IA.

## Validacion INVEST (resumen)
| Criterio | Evidencia en backlog |
|---|---|
| Independent | Historias separadas por capacidad funcional y feature |
| Negotiable | Alcance y reglas afinables por refinement |
| Valuable | Cada historia expresa beneficio de paciente, medico o negocio |
| Estimable | Todas las historias tienen puntos estimados |
| Small | Historias divididas por slice de valor implementable en sprint |
| Testable | Criterios Gherkin en documento 07 |

## Politica de Dependencias
- Se permite iniciar historias Should solo si el sprint ya cumple historias Must comprometidas.
- Historias Could no bloquean hitos de presentacion.
- Historias Won't quedan fuera y no se estiman para compromiso.

## Sprint comprometido o sprint candidato

| ID | Estado | Sprint |
|---|---|---|
| HU-001 | Comprometida | Sprint 1 |
| HU-002 | Comprometida | Sprint 1 |
| HU-003 | Comprometida | Sprint 2 |
| HU-004 | Comprometida | Sprint 3 |
| HU-005 | Comprometida | Sprint 4 |
| HU-006 | Comprometida | Sprint 5 |
| HU-007 | Comprometida | Sprint 6 |
| HU-008 | Comprometida | Sprint 7 |
| OBS-001 | Comprometida | Sprint 7 |
| OBS-002 | Comprometida | Sprint 7 |
| OBS-003 | Comprometida | Sprint 7 |
| OBS-004 | Comprometida | Sprint 7 |
| HU-009 | Candidata | Sprint 8 |
| HU-010 | Comprometida | Sprint 8 |
| HU-011 | Candidata | Sprint 8 |
| HU-012 | Candidata | Sprint 9 |
| HU-013 | Candidata | Sprint 9 |
| HU-014 | No aplica | Won't |

Nota:
- Que una historia tenga sprint candidato no implica compromiso; el compromiso formal sigue dependiendo de capacidad y regla de corte MoSCoW.

## Referencias por Epica (Azure Boards)
- E1: [Azure-Boards-Epica1](../epics/Azure-Boards-Epica1.md)
- E2: [Azure-Boards-Epica2](../epics/Azure-Boards-Epica2.md)
- E3: [Azure-Boards-Epica3](../epics/Azure-Boards-Epica3.md)
- E4: [Azure-Boards-Epica4](../epics/Azure-Boards-Epica4.md)
- E5: [Azure-Boards-Epica5](../epics/Azure-Boards-Epica5.md)
- E6: [Azure-Boards-Epica6](../epics/Azure-Boards-Epica6.md)
- E7: [Azure-Boards-Epica7](../epics/Azure-Boards-Epica7.md)
