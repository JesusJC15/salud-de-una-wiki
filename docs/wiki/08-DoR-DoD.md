## Objetivo
Establecer reglas de calidad y preparacion obligatorias para que una historia entre a desarrollo (DoR) y para que se considere terminada (DoD).

## Alcance
Aplica a las historias comprometidas en cada sprint del semestre (Sprint 0 a Sprint 9).
  
## Definition of Ready (DoR)
Una historia entra a sprint solo si cumple todos los puntos:

1. Historia escrita en formato `Como/Quiero/Para`.
2. ID, epica y feature definidos.
3. Prioridad MoSCoW asignada.
4. Criterios Gherkin (principal y alterno) documentados para la historia comprometida.
5. Dependencias identificadas y desbloqueadas o con plan explicito.
6. Estimacion en Story Points acordada por el equipo.
7. Datos/ambientes necesarios identificados.
8. Riesgos funcionales y tecnicos anotados.
9. Criterios de seguridad y privacidad aplicables definidos.
10. Responsable de validacion funcional asignado.

## Definition of Done (DoD)
Una historia se cierra solo si cumple todos los puntos:

1. Codigo implementado y revisado por al menos un par.
2. Pruebas unitarias e integracion del flujo critico en verde.
3. Criterios Gherkin de la historia verificados.
4. Observabilidad instrumentada cuando aplique (logs/metricas/eventos).
5. Validaciones de seguridad basica ejecutadas (auth, autorizacion, input).
6. Documentacion tecnica/funcional actualizada en Wiki.
7. Demo funcional aprobada por el equipo.
8. No quedan defectos severidad alta sin plan.
9. Telemetria de la historia visible en dashboard si impacta KPIs.
10. Historia pasa a estado `Done` con evidencia enlazada.

## DoD Especial para Historias de IA
Adicional a DoD general:

1. Prompt/version de plantilla documentados.
2. Guardrails activos para no diagnostico/no prescripcion.
3. Registro de trazas IA con correlation ID.
4. Evaluacion de salida con casos de prueba controlados.

## Politica de Calidad por Sprint
- Ninguna historia Must puede cerrarse sin pasar DoD completo.
- Historias Should/Could pueden moverse de sprint solo con acuerdo de PO y Scrum Master.
- Si falla DoD en cierre de sprint, la historia retorna a `In Progress` y se planifica debt visible.

## Evidencias Minimas por Historia Cerrada
- Link a PR o commit.
- Resultado de pruebas.
- Captura o registro de demo.
- Link de actualizacion en Wiki.