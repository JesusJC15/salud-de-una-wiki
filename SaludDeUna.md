# Salud De Una

**Salud De Una** es una plataforma inteligente de comunicacion clinica que conecta pacientes y medicos mediante IA que estructura, prioriza y optimiza la informacion medica antes de que el profesional intervenga.

No es telemedicina tradicional.
No es un chat medico.

Es un sistema de **triage asistido por IA + estructuracion clinica + seguimiento inteligente**.

> **Nota sobre el alcance del MVP:** Este documento describe la vision completa del producto. El MVP academico implementado en IETI-2026-I cubre un subconjunto priorizaado de estas capacidades. Las secciones marcadas con `[MVP]` estan implementadas; las marcadas con `[Roadmap]` son parte de la vision de producto pero no del MVP actual. Ver [16-Plan-Tecnico-Actualizado-2026-05.md](docs/wiki/16-Plan-Tecnico-Actualizado-2026-05.md) para el estado real de implementacion.

---

# Problematica

## Del lado del paciente

- No sabe si su sintoma es urgente.
- No sabe explicar correctamente lo que siente.
- Olvida informacion importante.
- Tiene consultas cortas y costosas.
- Tiene acceso limitado a orientacion medica oportuna.

## Del lado del medico

- Informacion desorganizada al recibir al paciente.
- Consultas repetitivas con recoleccion manual de antecedentes.
- Tiempo desperdiciado en anamnesis basica.
- Saturacion de agenda sin prioridad clara.
- Falta de continuidad estructurada entre consultas.

---

# Propuesta de Valor

**"IA como asistente clinico de comunicacion — no como diagnosticador"**

La IA:

- Guia al paciente con preguntas estructuradas por especialidad. `[MVP]`
- Detecta *red flags* y clasifica prioridad. `[MVP]`
- Genera resumen clinico profesional para el medico. `[MVP]`
- Apoya la comunicacion entre paciente y medico con informacion organizada. `[MVP]`

La IA **nunca**:

- **Diagnostica**
- **Prescribe**
- **Reemplaza al medico**

Toda salida de la IA pasa por un guardrail que filtra contenido de diagnostico, prescripcion y afirmacion clinica.

---

# Capacidades del MVP (Implementadas)

## `[MVP]` Triage Asistido por IA — No diagnostico

La plataforma guia al paciente con un cuestionario estructurado por especialidad. Una vez completado, el motor de IA (Gemini 2.5-flash + reglas clinicas) analiza las respuestas y:

- Detecta combinaciones de sintomas que corresponden a *red flags* predefinidas.
- Clasifica la prioridad de atencion: **Baja**, **Moderada** o **Alta**.
- Genera un resumen clinico neutral de urgencia (sin diagnostico ni prescripcion).

Especialidades disponibles en el MVP: **Medicina General**, **Odontologia**, **Urgencias**.

**Ejemplo en odontologia:**
Dolor intenso + fiebre + inflamacion facial → red flag de posible urgencia infecciosa → prioridad Alta.

**Impacto:**
- Proteccion del paciente ante signos de alarma no reconocidos.
- Optimizacion del flujo medico por prioridad real.
- Reduccion del riesgo clinico y legal.

## `[MVP]` Resumen Clinico Automatizado

Antes de que el medico atienda la consulta, recibe informacion estructurada del paciente:

- Motivo de consulta.
- Duracion e intensidad de sintomas.
- Sintomas asociados y factores agravantes.
- Medicacion actual reportada.
- Antecedentes relevantes.
- Nivel de prioridad estimado.
- Red flags detectadas (si aplica).

Formato de nota medica generado por IA (Gemini 2.5-flash), filtrado por guardrail clinico.

## `[MVP]` Chat Clinico en Tiempo Real

Canal de comunicacion directo entre paciente y medico via Socket.IO, dentro del contexto de una consulta activa. Los mensajes se persisten en MongoDB para trazabilidad clinica.

## `[MVP]` Seguimiento Post-Consulta (Followup)

Despues del cierre de consulta, la plataforma crea automaticamente dos seguimientos programados:

- **72 horas:** evaluacion inicial de recuperacion.
- **7 dias:** evaluacion de evolucion a mediano plazo.

Si la severidad de sintomas aumenta 2 o mas puntos respecto a la baseline, se escala automaticamente creando una nueva consulta de alta prioridad.

## `[MVP]` Historia Clinica Evolutiva

No es solo una consulta aislada. La plataforma construye un historial acumulativo del paciente con:

- Timeline de consultas y seguimientos.
- Evolucion de sintomas entre consultas.
- Trazabilidad de prioridades y red flags.

## `[MVP]` Facturacion Simulada

Modelo de pago por consulta (simulado para el MVP academico):

- Precios configurables por especialidad por el administrador.
- Ciclo de vida de transacciones: PENDING → COMPLETED → REFUNDED.
- Metricas de revenue en el dashboard administrativo.

La integracion con pasarela de pago real (Stripe/Wompi) es parte del roadmap de produccion.

## `[MVP]` Panel Administrativo

Para administradores:

- Dashboard con KPIs de negocio (pacientes, doctores verificados, revenue).
- Metricas tecnicas (p95 latencia, error rate).
- Bandeja de verificacion de credenciales medicas (REThUS).
- Gestion de usuarios (activar/desactivar).
- Administracion de precios por especialidad.

---

# Capacidades del Roadmap (No en MVP)

Estas capacidades son parte de la vision del producto pero no estan implementadas en el MVP IETI-2026-I:

## `[Roadmap]` Especialidades Adicionales

Especialidades planeadas para iteraciones futuras: Ginecologia, Psicologia, Dermatologia, Pediatria, Cardiologia.

## `[Roadmap]` Perfil de Salud Dinamico

Registro de condiciones cronicas, medicacion habitual, alergias y antecedentes relevantes para que la IA tenga contexto en futuras consultas.

## `[Roadmap]` Modo Acompanante Familiar

Un usuario puede gestionar consultas de adultos mayores o menores a su cargo (maximo 3 dependientes).

## `[Roadmap]` Banco de Conocimiento Validado con RAG

La infraestructura RAG (Knowledge base + embeddings) esta preparada en el backend (KnowledgeModule + RagModule), pero el flujo completo de aprobacion de contenido medico y la UI de gestion son parte del roadmap.

## `[Roadmap]` Traduccion Bidireccional Paciente-Clinico

Traduccion de lenguaje cotidiano a lenguaje medico y viceversa. La arquitectura soporta este caso de uso via AiModule pero no esta expuesta como endpoint en el MVP.

## `[Roadmap]` Integracion con Pasarela de Pago Real

Reemplazo del checkout simulado por integracion real con Stripe o Wompi cuando haya usuarios de pago.

---

# Innovaciones Clave del MVP

1. **Triage predictivo etico:** clasifica prioridad sin diagnosticar.
2. **Guardrail clinico integrado:** filtra automaticamente contenido de diagnostico o prescripcion.
3. **Resumen clinico automatizado:** el medico llega a la consulta con informacion organizada.
4. **Historia evolutiva acumulativa:** seguimiento longitudinal del paciente.
5. **Seguimiento automatizado post-consulta:** escalacion inteligente por deterioro.
6. **Pipeline RAG preparado:** infraestructura de embeddings y Vector Search lista para enriquecer el contexto de la IA.

Es **una capa inteligente de comunicacion entre paciente y medico**.

---

# Modelo de Negocio (Simulado en MVP)

## Pago por consulta `[MVP - simulado]`

- Paciente paga por acceso a consulta medica.
- Precios configurables por especialidad.
- Revenue calculado y visible en dashboard admin.

## Suscripcion Premium paciente `[Roadmap]`

- Consultas ilimitadas.
- Seguimiento avanzado.
- Historial descargable.

## Suscripcion medica `[Roadmap]`

- Acceso a pacientes priorizados.
- Panel profesional con insights de practica.
- Comision reducida sobre consultas.

---

# Limites Importantes

- La plataforma **no es un servicio de telemedicina completo** en terminos regulatorios colombianos.
- La IA **no diagnostica, no prescribe y no reemplaza al juicio clinico del medico**.
- El MVP es un prototipo academico/profesional con funcionalidad real pero **sin usuarios de produccion validados**.
- Los pagos en el MVP son **simulados** (no se procesan transacciones financieras reales).
- La verificacion REThUS es **manual/administrativa** por el equipo de la plataforma, no automatizada via API oficial.
