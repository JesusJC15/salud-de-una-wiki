## Objetivo
Priorizar el alcance del semestre con metodologia MoSCoW para proteger entregables criticos y controlar riesgo de sobrealcance.

## Alcance
Define que entra en Must, Should, Could y Won't, junto con criterios de corte de alcance.

## Criterios de Priorizacion
1. Impacto en valor clinico.
2. Impacto en seguridad del paciente.
3. Dependencias tecnicas.
4. Riesgo de implementacion en horizonte de 10 sprints.

## Matriz MoSCoW
| Categoria | Historias | Justificacion |
|---|---|---|
| Must | HU-001, HU-002, HU-003, HU-004, HU-005, HU-006, HU-007, HU-008, HU-010 | Nucleo funcional del MVP, historias IA obligatorias, tiempo real, observabilidad y cumplimiento de evaluacion |
| Should | HU-009, HU-011 | Mejoran experiencia y productividad, pero no bloquean valor core |
| Could | HU-012, HU-013 | Aportan diferenciacion, no son criticas para demo final |
| Won't | HU-014 | Alto costo tecnico y no obligatorio para objetivo del curso |

## Regla de Corte de Alcance
- Si el equipo detecta desviacion de capacidad > 15% en un sprint:
  - Primero mover Could.
  - Luego mover Should.
  - Nunca retirar Must sin replaneacion formal y aprobacion PO.

## Dependencias Criticas Must
- HU-005 depende de HU-003/HU-004.
- HU-006 depende de HU-005.
- HU-007 y HU-008 dependen de HU-006.
- HU-010 puede ir paralelo a HU-001.

## Control de Cambios
- Toda propuesta de cambio de prioridad se documenta en retrospectiva y planning siguiente.
- Cualquier cambio de Must a Should requiere evidencia de riesgo real y aval de equipo completo.