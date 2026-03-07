## Objetivo
Definir el backlog oficial de historias de usuario del MVP con estructura INVEST y campos obligatorios para desarrollo y trazabilidad.

## Alcance
Incluye historias Must, Should, Could y Won't de alcance semestral. Cada historia contiene `ID`, `Como`, `Quiero`, `Para`, prioridad, dependencia y estimacion.

## Backlog Priorizado
| ID | Epica | Feature | Como | Quiero | Para | MoSCoW | Dependencia | Estimacion (SP) |
|---|---|---|---|---|---|---|---|---|
| HU-001 | E1 | F1.1 | Paciente nuevo | registrarme e iniciar sesion | gestionar mis consultas de forma segura | Must | Ninguna | 5 |
| HU-002 | E1 | F1.2 | Admin de plataforma | validar medico con soporte REThUS | asegurar autenticidad profesional | Must | HU-001 | 8 |
| HU-003 | E2 | F2.1 | Paciente de medicina general | responder un triage guiado por IA | describir mejor sintomas y riesgo | Must | HU-001 | 8 |
| HU-004 | E2 | F2.2/F2.3 | Paciente odontologico | recibir clasificacion de prioridad por red flags | identificar casos que requieren atencion presencial | Must | HU-003 | 8 |
| HU-005 | E4 | F4.1 | Medico | recibir resumen clinico automatico preconsulta | reducir tiempo de recoleccion de antecedentes | Must | HU-003/HU-004 | 8 |
| HU-006 | E3 | F3.1/F3.2 | Paciente y medico | intercambiar mensajes en tiempo real con estado de caso | mejorar continuidad y oportunidad de respuesta | Must | HU-005 | 13 |
| HU-007 | E5 | F5.1/F5.2 | Paciente en seguimiento | reportar evolucion despues de consulta | detectar empeoramiento o mejora oportuna | Must | HU-006 | 8 |
| HU-008 | E6 | F6.1/F6.2/F6.3 | Equipo de producto | visualizar metricas tecnicas y KPIs de negocio | tomar decisiones basadas en datos | Must | HU-006/HU-007 | 8 |
| HU-009 | E4 | F4.2 | Paciente | recibir explicaciones en lenguaje simple | entender mejor la orientacion medica | Should | HU-005 | 5 |
| HU-010 | E7 | F7.1 | Paciente | simular compra de consulta o plan | validar flujo de monetizacion | Must | HU-001 | 5 |
| HU-011 | E7 | F7.2 | Medico | reutilizar respuestas validadas por pares | reducir consultas repetitivas | Should | HU-005 | 5 |
| HU-012 | E5 | - | Familiar cuidador | gestionar perfil de menor o adulto mayor | acompanar atencion de terceros | Could | HU-001 | 8 |
| HU-013 | E6 | - | Equipo de datos | exportar tablero a reporte semanal | seguir tendencias de valor y riesgo | Could | HU-008 | 3 |
| HU-014 | E3 | - | Paciente y medico | tener videollamada integrada | resolver consulta en sincronico completo | Won't | HU-006 | 13 |


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

## Referencias por Epica (Azure Boards)
- E1: [Azure-Boards-Epica1](../epics/Azure-Boards-Epica1.md)
- E2: [Azure-Boards-Epica2](../epics/Azure-Boards-Epica2.md)
- E3: [Azure-Boards-Epica3](../epics/Azure-Boards-Epica3.md)
- E4: [Azure-Boards-Epica4](../epics/Azure-Boards-Epica4.md)
- E5: [Azure-Boards-Epica5](../epics/Azure-Boards-Epica5.md)
- E6: [Azure-Boards-Epica6](../epics/Azure-Boards-Epica6.md)
- E7: [Azure-Boards-Epica7](../epics/Azure-Boards-Epica7.md)
