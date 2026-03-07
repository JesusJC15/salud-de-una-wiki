## Objetivo
Reunir glosario, plantillas y referencias utiles para mantener consistencia de lenguaje y ejecucion en el proyecto.

## Alcance
Incluye anexos metodologicos (INVEST, Gherkin, MoSCoW, DoR/DoD) y glosario funcional/tecnico.

## Glosario
| Termino | Definicion |
|---|---|
| Triage predictivo no diagnostico | Priorizacion de riesgo con IA sin emitir diagnostico medico |
| Red flag | Patron de sintomas que sugiere riesgo alto y escalamiento de atencion |
| RAG | Recuperacion de contexto relevante para mejorar respuesta del modelo |
| RBAC | Control de acceso basado en roles |
| SLO | Objetivo de nivel de servicio tecnico |
| KPI | Indicador clave de negocio |
| DoR | Criterios para considerar historia lista para desarrollo |
| DoD | Criterios para cerrar una historia como terminada |
| MoSCoW | Metodo de priorizacion Must/Should/Could/Won't |

## Plantilla de Historia de Usuario
```md
ID: HU-XXX
Epica: EX
Feature: FX.Y
Como: <rol>
Quiero: <accion/capacidad>
Para: <beneficio>
Prioridad MoSCoW: Must/Should/Could/Won't
Dependencias: <ID o Ninguna>
Estimacion (SP): <numero>
```

## Plantilla de Gherkin
```gherkin
Feature: <nombre>

Scenario: Camino principal
  Given <contexto>
  When <accion>
  Then <resultado esperado>

Scenario: Camino alterno
  Given <contexto alterno>
  When <accion alterna>
  Then <resultado alterno esperado>
```

## Plantilla de Riesgo
```md
ID: R-XXX
Riesgo: <descripcion>
Probabilidad: Baja/Media/Alta
Impacto: Bajo/Medio/Alto
Mitigacion: <accion>
Dueño: <rol o persona>
```

## Referencias Metodologicas
- INVEST para historias de usuario.
- Gherkin (Given/When/Then).
- MoSCoW para priorizacion.
- Definition of Ready y Definition of Done.
- Practicas de documentacion en Azure DevOps Wiki.

## Estructura de Documentacion (mapa rapido)
1. Home: `00-Home.md`
2. Definicion de producto: `01` a `03`
3. Definicion tecnica y arquitectura: `04`
4. Backlog y calidad: `05` a `08`
5. Observabilidad y priorizacion: `09`, `10`
6. Plan operativo y riesgos: `11`, `12`
7. Anexos metodologicos: `13`
8. Mockups funcionales: `14`
9. Cumplimiento lineamientos 2026-1: `15`
10. Epicas Azure Boards (detalle): `docs/epics/Azure-Boards-Epica1.md` a `Azure-Boards-Epica7.md`
