## Objetivo
Explicar el problema real que aborda `SaludDeUna`, su contexto de uso y el alcance funcional del MVP para el curso.

## Alcance
Describe antecedentes, dolores de paciente/medico, alcance y no alcance. No contiene implementacion tecnica detallada.

## Antecedentes
En contextos de atencion ambulatoria, el paciente llega con informacion incompleta o desordenada y el medico invierte parte importante de la consulta en recolectar antecedentes. Esto reduce el tiempo clinico efectivo y afecta la continuidad.

## Problema a Resolver

### Del lado del paciente
- No distingue urgencia real de sintomas no urgentes.
- Describe mal sintomas por falta de estructura.
- Omite datos relevantes por nervios o desconocimiento.
- Tiene consultas cortas y costosas.

### Del lado del medico
- Recibe informacion desorganizada.
- Repite preguntas base de manera constante.
- Tiene alta carga operativa por triage manual.
- Falta continuidad estructurada entre consultas.

## Solucion Propuesta
`SaludDeUna` usa IA con guardrails clinicos para:

- Guiar al paciente con preguntas estructuradas por especialidad.
- Identificar red flags y clasificar prioridad (`LOW`, `MODERATE`, `HIGH`).
- Generar resumen clinico preconsulta.
- Soportar seguimiento posterior y linea de evolucion.

Regla etica explicita:
- La IA no diagnostica.
- La IA no prescribe.
- La decision final es del medico.

## Alcance MVP
- Especialidades: Medicina General y Odontologia.
- Cliente paciente: app React Native.
- Cliente medico/admin: web React + Next.
- Backend: NestJS + MongoDB + WebSocket.
- IA: Gemini + RAG + reglas clinicas.
- Verificacion profesional: validacion semiautomatica REThUS por admin.
- Monetizacion: flujo simulado (no pasarela productiva).

## No Alcance MVP
- Videollamada en vivo.
- Prescripcion automatica.
- Diagnostico automatizado.
- Integracion productiva con HCE de terceros.
- Cobro productivo real.

## Valor Agregado vs Alternativas

Comparado con chat medico simple o telemedicina tradicional:
- Priorizacion clinica previa de casos.
- Resumen clinico automatico y reutilizable.
- Continuidad estructurada por seguimiento.
- Dashboard de eficiencia clinica y seguridad operacional.

## Restricciones y Contexto Regulatorio
- Contexto principal: Colombia precomercial con diseno escalable a LatAm.
- Manejo de datos: mixto (sinteticos en desarrollo y datos anonimizados en demo).
- Cualquier dato sensible debe seguir minimizacion, trazabilidad y control de acceso por rol.