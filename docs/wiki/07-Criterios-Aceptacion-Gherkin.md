## Objetivo
Definir criterios de aceptacion concretos en formato Gherkin para historias Must, incluyendo escenario principal y escenario alterno por historia.

## Alcance
Cubre historias Must del backlog `HU-001` a `HU-008` y `HU-010`, y aplica tambien a historias Should/Could cuando sean comprometidas en sprint. No incluye pruebas tecnicas de rendimiento detalladas.

## Regla de aplicacion
- Todo item comprometido en sprint debe tener 2 escenarios Gherkin (principal y alterno) antes de iniciar desarrollo.
- Las historias no comprometidas pueden mantener criterios en borrador hasta entrar a planificacion formal.

## HU-001 Registro e inicio de sesion paciente
```gherkin
Feature: Registro y login de paciente

Scenario: Registro principal exitoso
  Given un usuario paciente sin cuenta
  When completa nombre, correo y contrasena validos
  Then el sistema crea la cuenta con rol PACIENTE
  And permite iniciar sesion con token valido

Scenario: Registro alterno por correo duplicado
  Given un correo ya registrado
  When el usuario intenta crear una nueva cuenta con ese correo
  Then el sistema rechaza la operacion
  And muestra mensaje de conflicto de identidad
```

## HU-002 Validacion REThUS de medico
```gherkin
Feature: Validacion profesional de medico

Scenario: Validacion principal aprobada por admin
  Given un medico en estado PENDING
  When el admin registra evidencia de verificacion REThUS
  Then el estado del medico cambia a VERIFIED
  And se habilita acceso a cola de casos

Scenario: Validacion alterna rechazada
  Given un medico en estado PENDING
  When el admin registra inconsistencia documental
  Then el estado cambia a REJECTED
  And el medico no puede atender consultas
```

## HU-003 Triage IA Medicina General
```gherkin
Feature: Triage IA para medicina general
  
Scenario: Triage principal con priorizacion
  Given un paciente autenticado en Medicina General
  When responde el cuestionario guiado
  Then el sistema genera prioridad LOW, MODERATE o HIGH
  And guarda evidencia de sintomas y factores de riesgo

Scenario: Triage alterno incompleto
  Given un paciente que abandona el cuestionario
  When intenta enviar respuestas incompletas
  Then el sistema solicita completar campos obligatorios
  And no ejecuta analisis de prioridad
```

## HU-004 Red flags Odontologia
```gherkin
Feature: Deteccion de red flags odontologicas

Scenario: Caso principal de alto riesgo
  Given un paciente odontologico con dolor intenso y fiebre
  When el motor de reglas evalua sintomas
  Then el sistema clasifica prioridad HIGH
  And recomienda atencion presencial inmediata

Scenario: Caso alterno sin red flags criticas
  Given un paciente odontologico con molestia leve
  When el motor de reglas evalua sintomas
  Then el sistema asigna prioridad LOW o MODERATE
  And mantiene seguimiento digital
```

## HU-005 Resumen clinico automatico
```gherkin
Feature: Generacion de resumen clinico

Scenario: Resumen principal preconsulta
  Given un caso con triage finalizado
  When el medico abre la consulta
  Then el sistema presenta resumen clinico estructurado
  And muestra motivo, evolucion, medicacion y prioridad
  
Scenario: Resumen alterno con baja confianza de datos
  Given un caso con respuestas ambiguas
  When el sistema detecta baja confianza semantica
  Then marca campos como pendientes de confirmacion
  And advierte al medico antes de responder
```

## HU-006 Chat clinico tiempo real
```gherkin
Feature: Mensajeria clinica en tiempo real

Scenario: Interaccion principal paciente-medico
  Given una consulta activa
  When paciente y medico envian mensajes
  Then ambos ven mensajes en tiempo real
  And el estado de consulta se actualiza sin refrescar pagina

Scenario: Interaccion alterna con desconexion temporal
  Given una consulta activa con perdida momentanea de red
  When el usuario recupera conectividad
  Then el cliente sincroniza mensajes pendientes
  And conserva orden cronologico
```

## HU-007 Seguimiento post-consulta
```gherkin
Feature: Seguimiento automatizado de evolucion

Scenario: Seguimiento principal completado
  Given una consulta cerrada
  When llega la fecha de seguimiento
  Then el paciente recibe formulario de evolucion
  And el sistema guarda cambios de sintomas

Scenario: Seguimiento alterno con empeoramiento
  Given un paciente que reporta aumento de intensidad
  When el sistema procesa la respuesta
  Then aumenta prioridad del caso
  And notifica al medico responsable
```

## HU-008 Dashboard tecnico y de negocio
```gherkin
Feature: Visualizacion de observabilidad y KPIs

Scenario: Dashboard principal disponible
  Given datos de logs y metricas del periodo
  When el equipo abre el panel de observabilidad
  Then ve latencia, errores, concurrencia y 4 KPIs de negocio
  And cada KPI muestra valor, meta y estado

Scenario: Dashboard alterno con fuente incompleta
  Given una fuente de metricas caida
  When el panel intenta cargar datos
  Then muestra estado de degradacion controlada
  And mantiene visibles las fuentes disponibles
```

## HU-010 Flujo de monetizacion simulado
```gherkin
Feature: Checkout simulado de consulta o plan

Scenario: Simulacion principal de compra
  Given un paciente autenticado
  When selecciona plan o consulta y confirma simulacion
  Then el sistema registra transaccion simulada exitosa
  And habilita el beneficio contratado

Scenario: Simulacion alterna cancelada
  Given un paciente en pantalla de checkout simulado
  When cancela la operacion
  Then no se crea transaccion
  And el estado del plan se mantiene sin cambios
```