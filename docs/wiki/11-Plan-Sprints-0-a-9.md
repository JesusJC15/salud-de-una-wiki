# Existing Content of 11-Plan-Sprints-0-a-9.md

... (preserving previous sections) ...

## Detalle de Epicas y Story Points Totales

### Resumen de Epicas
| ID  | Titulo                                    | Sprint     | Story Points | Prioridad | Riesgo | Business Value                                                   | Funcionalidades Clave                                           |
|-----|-------------------------------------------|------------|--------------|-----------|--------|-----------------------------------------------------------------|----------------------------------------------------------------|
| E1  | Onboarding y Acceso Seguro                | Sprint 1  | 13           | Critical  | Medium | Incorporar usuarios validos con control de rol; habilita todas las demas epicas | Registro paciente/medico, Login JWT, RBAC, Verificacion REThUS semiautomatica  |
| E2  | Triage Inteligente por Especialidad       | Sprint 2-3| 16           | Critical  | Medium | Priorizar casos y reducir incertidumbre del paciente            | Cuestionario adaptativo MG/Odontologia, Motor IA Gemini+RAG MG, Reglas red flags, Clasificacion LOW/MODERATE/HIGH|
| E3  | Consulta Clinica en Tiempo Real           | Sprint 5  | 13           | Critical  | High   | Mejorar velocidad y continuidad de atencion                    | Chat WebSocket, Cola priorizada, Estado consulta, Persistencia MongoDB|
| E4  | Resumen Clinico y Traductor IA            | Sprint 4  | 13           | Critical  | High   | Aumentar eficiencia medica y claridad de comunicacion           | Generacion resumen Gemini+RAG, Traduccion bidireccional, Guardrails antiDiagnostico|
| E5  | Seguimiento y Evolucion del Paciente      | Sprint 6  | 16           | Critical  | Medium | Medir progresion, detectar cambios riesgo, retension 7 dias    | Seguimiento auto 72h + 7 dias, Timeline evolutivo, Motor repiorizacion|
| E6  | Observabilidad y Analitica                 | Sprint 7  | 12           | Critical  | Medium | Medir salud tecnica y valor negocio para decisiones basadas evidencia | Logs estructurados, Metricas latencia/error/concurrencia, Dashboards tecnico/negocio, Alertas SLO|
| E7  | Monetizacion Simulada y Gobierno           | Sprint 8  | 10           | High     | Medium | Validar viabilidad modelo y controles operativos               | Checkout simulado, Banco conocimiento, Flujo aprobacion medico|

### Resumen de Story Points
| Sprint     | Total Story Points |
|------------|-------------------|
| Sprint 1  | 13                |
| Sprint 2-3| 16                |
| Sprint 4  | 13                |
| Sprint 5  | 13                |
| Sprint 6  | 16                |
| Sprint 7  | 12                |
| Sprint 8  | 10                |
| **Total**  | **99**            |

### Epic Dependencies and Blocker Assessment
- **E1**: No dependencies.
- **E2**: Depends on E1.
- **E3**: Depends on E2.
- **E4**: Depends on E3.
- **E5**: Depends on E3.
- **E6**: No blocking issues identified.
- **E7**: Depends on E6.