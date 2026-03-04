## Objetivo
Definir el marco de observabilidad tecnica y de negocio del MVP, incluyendo historias asociadas, metrica de rendimiento/concurrencia y dashboard oficial.

## Alcance
Cubre logs estructurados, metricas tecnicas, 4 KPIs de negocio, alertas y vistas de dashboard.

## Historias de Observabilidad

| ID | Como | Quiero | Para | MoSCoW |
|---|---|---|---|---|
| OBS-001 | Equipo tecnico | registrar logs estructurados con correlation ID | rastrear errores end-to-end | Must |
| OBS-002 | Equipo tecnico | medir latencia, throughput y concurrencia | controlar rendimiento del sistema | Must |
| OBS-003 | Product Owner | visualizar 4 KPIs de negocio en dashboard | tomar decisiones de producto | Must |
| OBS-004 | Equipo de operacion | recibir alertas por degradacion | responder antes de afectar usuarios | Must |
  
## Logs Estructurados (obligatorio)

### Campos minimos por evento
- `timestamp`
- `level`
- `service`
- `endpoint_or_event`
- `correlation_id`
- `user_id` (anonimizado cuando aplique)
- `role`
- `latency_ms`
- `status_code`
- `error_code` (si aplica)

### Eventos minimos a registrar
- Login y cambios de sesion.
- Ejecucion de triage.
- Cambio de prioridad por red flags.
- Generacion de resumen IA.
- Mensajeria chat tiempo real.
- Creacion y cierre de seguimiento.
- Validacion REThUS por admin.

## Metricas Tecnicas (concurrencia y rendimiento)

| Metrica | Definicion | Objetivo |
|---|---|---|
| P95 API Latency | Latencia P95 de endpoints criticos | < 1500 ms |
| Chat Delivery Time | Tiempo de entrega de mensaje WS | < 1500 ms |
| AI Summary Time | Tiempo de generacion de resumen IA | < 15000 ms |
| Error Rate | Porcentaje de respuestas 5xx/total | < 2.0% |
| Concurrent Sessions | Sesiones activas simultaneas | Medicion continua |
| Availability | Tiempo de disponibilidad mensual | >= 98.0% |

## KPIs de Negocio (4 obligatorios)

| KPI | Formula | Meta Inicial | Fuente |
|---|---|---|---|
| Tiempo a primera respuesta medica | Promedio(`timestamp_primera_respuesta - timestamp_apertura`) | Reducir 30% vs baseline Sprint 2 | Consultations |
| Utilidad de resumen clinico | `% de consultas donde medico marca resumen como util` | >= 75% | Feedback medico |
| Red flags relevantes confirmadas | `% red flags HIGH confirmadas por medico` | >= 60% | Triage + validacion medico |
| Retencion de seguimiento 7 dias | `% pacientes que completan seguimiento en 7 dias` | >= 50% | FollowUp records |

## Definicion de Dashboard

### Panel Tecnico
- Latencia P95 por endpoint.
- Throughput por minuto.
- Error rate por servicio.
- Usuarios concurrentes activos.
- Estado de WebSocket hub.

### Panel Negocio
- Tiempo a primera respuesta medica.
- Utilidad de resumen clinico.
- Red flags relevantes confirmadas.
- Retencion de seguimiento 7 dias.

### Segmentaciones minimas
- Por especialidad.
- Por prioridad de caso.
- Por rango de fechas (dia/semana/sprint).

## Reglas de Alertamiento
- Alerta critica si `P95 API > 1500 ms` durante 10 minutos.
- Alerta critica si `error_rate > 2%` durante 5 minutos.
- Alerta warning si `AI Summary Time > 15000 ms` en 3 muestras consecutivas.
- Alerta warning si `availability < 98%` en ventana mensual proyectada.

## Operacion y Revision
- Revision diaria tecnica en daily scrum.
- Revision de KPIs de negocio semanal.
- Ajustes de backlog segun evidencia de dashboard.