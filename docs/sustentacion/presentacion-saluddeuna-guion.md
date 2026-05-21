# SaludDeUna — Guión Completo de Exposición

> Tiempo estimado: 20–22 minutos de presentación + preguntas del jurado.
> Cada bloque indica la slide correspondiente y el tiempo objetivo.

---

## SLIDE 1 — Portada (30 s)

*[Mientras se carga la presentación, QR ya visible en pantalla]*

> "Buenos días/tardes. Somos el equipo SaludDeUna. Les vamos a mostrar una plataforma de comunicación clínica asistida por IA que desarrollamos durante este semestre como proyecto final de IETI 2026-1."

*[Pausa breve. Dejar que lean el subtítulo.]*

> "El QR que ven en pantalla conecta directamente al APK de nuestra aplicación móvil. Pueden escanearlo ahora o en cualquier momento durante la presentación."

---

## SLIDE 2 — Agenda (30 s)

> "En los próximos 20 minutos vamos a cubrir: el problema que identificamos, nuestra solución, la arquitectura técnica del sistema, los criterios de calidad que implementamos, cómo usamos la IA, y vamos a cerrar con una demo en vivo del sistema funcionando."

> "Empecemos con el problema."

---

## SLIDE 3 — El problema en una frase (1 min)

*[Mostrar la frase grande, esperar 3 segundos]*

> "Esta frase resume lo que encontramos: la información clínica llega al médico mal organizada, tarde, o incompleta."

> "Pero ¿qué significa eso en la práctica?"

*[Transición a siguiente slide con pausa deliberada]*

---

## SLIDE 4 — Del lado del paciente (45 s)

> "Del lado del paciente, el problema es de estructura. No es que no sepa lo que siente — es que no sabe cómo comunicarlo de forma útil para el médico."

> "No sabe si su dolor de cabeza es una cefalea tensional o una señal de algo más serio. No recuerda qué medicamentos tomó la semana pasada. En una consulta de 10 minutos, esa información se pierde."

> "El resultado: visitas innecesarias a urgencias, diagnósticos tardíos, y un sistema saturado."

---

## SLIDE 5 — Del lado del médico (45 s)

> "Del lado del médico, el problema es de tiempo y continuidad. El médico dedica entre el 40 y el 60 por ciento de cada consulta a recolectar información básica: antecedentes, medicación actual, duración de los síntomas."

> "Esa es información que el paciente ya tiene — solo que no está organizada."

> "Y además, entre una consulta y la siguiente, no hay continuidad estructurada. El médico tiene que reconstruir el contexto desde cero cada vez."

---

## SLIDE 6 — Propuesta de valor (45 s)

> "SaludDeUna propone ser una capa inteligente entre paciente y médico."

> "Tres cosas concretas: primero, triage asistido por IA que clasifica la prioridad del caso antes de que el médico lo vea. Segundo, un resumen clínico automático para que el médico llegue informado a la consulta. Y tercero, seguimiento post-consulta automatizado a las 72 horas y a los 7 días."

> "No es una idea — está implementado y funcionando."

---

## SLIDE 7 — Qué NO es SaludDeUna (45 s)

> "Antes de seguir, quiero ser muy explícito sobre lo que SaludDeUna NO es."

> "No es telemedicina en el sentido regulatorio. No es un sistema de diagnóstico automático. La IA no prescribe medicamentos, no diagnostica enfermedades, y no reemplaza al médico."

> "Lo que la IA hace es clasificar prioridad, estructurar información y generar resúmenes de urgencia. Nada más. Y esto es una decisión técnica y ética — no una limitación del proyecto."

*[Señalar la frase al pie de la slide]*

> "La IA organiza, clasifica y comunica. El médico decide."

---

## SLIDE 8 — Alcance del MVP (45 s)

> "¿Qué está implementado? Todo lo que ven en esta tabla."

> "Autenticación, triage con IA, cola de consultas médicas, chat en tiempo real, resumen clínico generado por Gemini, seguimiento post-consulta, verificación de médicos con REThUS, dashboard de KPIs, y facturación simulada."

> "Tres especialidades: Medicina General, Odontología y Urgencias. 406 tests pasando. Cobertura por encima del 80% en todos los módulos del backend."

---

## SLIDE 9 — Tres roles, un sistema (45 s)

> "El sistema tiene tres actores con interfaces separadas."

> "El paciente usa la aplicación móvil — construida con Expo y React Native. Tiene acceso al triage, al checkout, al chat y al seguimiento."

> "El médico usa el panel web — construido con Next.js. Ve la cola de consultas, chatea con el paciente y genera el resumen clínico."

> "El administrador también usa el panel web. Verifica credenciales médicas vía REThUS, gestiona usuarios y ve las métricas del negocio."

---

## SLIDE 10 — Flujo del negocio E2E (1 min)

> "Este es el flujo completo de principio a fin."

> "Empieza cuando el paciente hace login en la app. Responde el cuestionario de triage — entre 5 y 8 preguntas dependiendo de la especialidad. Gemini analiza las respuestas, detecta red flags si los hay, y asigna una prioridad."

> "El paciente hace el checkout — pago simulado — y la consulta entra a la cola."

> "El médico ve la cola ordenada por prioridad. Asigna la consulta. Abre el chat. Al cerrar, puede pedir el resumen clínico a Gemini."

> "Al cerrar la consulta, el sistema crea automáticamente dos followups: uno a las 72 horas y otro a los 7 días. Si el paciente reporta que empeoró, el sistema escala creando una nueva consulta de alta prioridad."

> "Este flujo es el que vamos a mostrar en la demo."

---

## SLIDE 11 — Arquitectura general C4 L1 (45 s)

> "La arquitectura sigue el modelo C4. Este es el nivel de contexto."

> "Tres actores humanos: paciente, médico y administrador. Un sistema central. Y seis servicios externos: Google Gemini para la IA, MongoDB Atlas para los datos, Redis Cloud para cache y colas, Auth0 para la identidad, RETHUS para verificar médicos, y Expo Notifications para las notificaciones push."

> "Todo el tráfico va por HTTPS. El chat usa WebSocket via Socket.IO."

---

## SLIDE 12 — Arquitectura de contenedores C4 L2 (45 s)

> "En el nivel de contenedores vemos los procesos. Dos frontends: la app móvil y el portal web. Dos procesos de backend: la API y el Worker."

> "El Worker es importante — maneja los jobs asincrónicos: los recordatorios de followup, el despacho del outbox transaccional, y las notificaciones push. Corre con BullMQ sobre Redis."

> "La base de datos es MongoDB Atlas con un índice vectorial de 768 dimensiones para el RAG."

---

## SLIDE 13 — Backend por dentro (1 min)

> "El backend tiene 17 módulos NestJS organizados en cuatro dominios."

> "Dominio de usuarios: autenticación, pacientes, médicos y administradores. Dominio clínico: triage, consultas, chat, billing, followups y notificaciones. Dominio de IA: el módulo AI con Gemini, el módulo de Knowledge base y el módulo RAG. Y el dominio de infraestructura: outbox transaccional, dashboard y el bootstrap de OpenTelemetry."

> "Todos los módulos pasan por tres capas transversales: JWT con RBAC, throttling de 20 requests por 60 segundos, y el filtro global de excepciones con correlationId."

---

## SLIDE 14 — Pipeline de triage IA (30 s)

> "El triage funciona así: el paciente responde el cuestionario. Esas respuestas van a Gemini con un prompt estructurado por especialidad. Gemini genera el análisis. Antes de persistir cualquier cosa, el guardrail revisa la respuesta."

> "Si el guardrail detecta lenguaje de diagnóstico o prescripción, bloquea la respuesta y guarda null. Si pasa el guardrail, se persiste el resumen neutral de urgencia."

> "El fallback también está implementado: si Gemini falla, el motor de reglas asume el análisis sin interrupción visible para el usuario."

---

## SLIDE 15 — Criterios de calidad resumen (30 s)

> "Implementamos seis criterios de calidad. Vamos a ver los tres más importantes: seguridad, confiabilidad y observabilidad."

---

## SLIDE 16 — Seguridad (45 s)

> "Seguridad. Cinco controles en la capa de autenticación."

> "Dual auth: JWT HS256 para el flujo legacy y Auth0 RS256 para el flujo OAuth. RBAC con tres roles y guards globales. Rate limiting de 20 requests por 60 segundos por IP, usando Redis como storage distribuido. bcrypt con cost 12 para contraseñas. Y máximo tres sesiones activas por usuario — si se supera, las más antiguas se revocan automáticamente."

> "Cada respuesta de la API incluye un correlationId único para trazabilidad."

---

## SLIDE 17 — Testing (45 s)

> "Confiabilidad. 406 tests pasando en el backend, ninguno fallando. 93% de cobertura de statements. Umbral mínimo del 80% configurado en Jest — si baja del 80%, el CI falla."

> "La estrategia de testing tiene dos capas. Unit tests con mocks de NestJS — la IA siempre se mockea en unit tests. Y E2E tests con mongodb-memory-server, que levanta una base de datos MongoDB real en memoria para cada suite. No se mockea la capa de base de datos en E2E."

> "¿Por qué importa esto? Porque los mocks de BD no detectan bugs de migración ni de queries complejas. El E2E real sí."

---

## SLIDE 18 — Observabilidad (45 s)

> "Observabilidad. Tres niveles."

> "Logs estructurados en JSON con correlationId, userId, método, path, statusCode y duración en milisegundos. Cada request queda trazable."

> "Métricas técnicas en tiempo real: p95 de latencia y error rate accesibles via API en /v1/dashboard/technical. Umbral de alerta: si el p95 supera 1500ms durante 10 minutos, es una alerta crítica."

> "Y trazas distribuidas con OpenTelemetry — configurado y listo para exportar a Jaeger o Tempo cuando se habilite el collector en producción."

---

## SLIDE 19 — IA en el producto (1 min)

> "La IA tiene tres funciones en el producto."

> "La primera es el triage: Gemini 2.5-flash recibe las respuestas del cuestionario y genera la clasificación de prioridad con los red flags. El guardrail valida la salida antes de persistir."

> "La segunda es el resumen clínico: después del chat entre paciente y médico, el doctor puede pedir a Gemini que genere una nota médica estructurada basada en la conversación. El guardrail también se aplica aquí."

> "La tercera es el RAG — Retrieval-Augmented Generation. Tenemos un Knowledge base con documentos clínicos, chunking, y embeddings vectoriales de 768 dimensiones usando gemini-embedding-001. El pipeline RAG enriquece el contexto del triage con evidencia clínica relevante. Hay un cache en Redis de 300 segundos para no repetir búsquedas vectoriales idénticas."

---

## SLIDE 20 — Guardrail clínico (45 s)

> "El guardrail merece una slide propia porque es la respuesta técnica a la pregunta que más van a hacer: '¿cómo garantizan que la IA no diagnostica?'"

> "La respuesta no es una promesa — es código. GuardrailService evalúa cada respuesta de Gemini contra tres categorías bloqueadas: lenguaje de diagnóstico, lenguaje de prescripción, y afirmaciones clínicas."

> "Si cualquiera de esas categorías aparece, la respuesta no se persiste. El campo aiSummary queda en null, se registra el evento con log WARN y correlationId, y el campo guardrailApplied queda en true. Esto es auditable."

---

## SLIDE 21 — IA en el desarrollo (45 s)

> "Más allá del producto, usamos IA durante todo el proceso de desarrollo."

> "Usamos Claude Code de Anthropic como asistente principal. Con él generamos módulos de NestJS, tests E2E, hooks de React, esquemas Zod y documentación. También hicimos revisiones arquitectónicas completas del código antes de cada sprint."

> "Los 406 tests del backend fueron revisados y en muchos casos generados con asistencia de IA. La documentación que ven — wiki, READMEs, diagramas — fue auditada y actualizada con asistencia de IA."

> "Esto es evidencia concreta del ítem 6 de los lineamientos."

---

## SLIDE 22 — Ruta del demo (45 s)

> "La demo va a seguir esta ruta. 12 pasos, aproximadamente 7 minutos."

> "Empezamos con el QR del APK proyectado. Luego login del paciente en el teléfono, triage completo, resultado con prioridad ALTA y red flag visible, checkout, y cambio al panel del doctor."

> "Desde el panel del doctor vemos la consulta al tope de la cola por prioridad alta. Abrimos el chat. Enviamos mensajes en tiempo real desde ambos dispositivos. El doctor genera el resumen clínico con Gemini. Cierra la consulta."

> "Cerramos con el panel del admin viendo los KPIs reales del sistema."

> "Si algo falla, tenemos screenshots de respaldo. Si Gemini no responde, el sistema usa el motor de reglas — sin error visible para el usuario."

---

## SLIDE 23 — Demo en vivo

*[Esta slide permanece durante toda la demo]*

> "Empezamos con el QR en pantalla. El APK está disponible para descarga directa desde EAS Build."

*[Ejecutar la demo según el checklist en presentacion-saluddeuna-checklist-demo.md]*

*[Durante la demo, narrar cada paso con voz clara y sin apresurarse]*

**Frases clave durante el demo:**

Al mostrar el resultado del triage:
> "El sistema detectó señales de alarma en las respuestas — aquí los ven como red flags. La IA los clasificó como prioridad ALTA. Esta consulta va al tope de la cola."

Al mostrar el chat:
> "Los mensajes van por Socket.IO en tiempo real. Están persistidos en MongoDB. El doctor ve exactamente lo mismo que el paciente."

Al generar el resumen:
> "El doctor pide el resumen. Gemini genera la nota médica. El guardrail valida que no tenga diagnóstico ni prescripción antes de mostrarse."

Al mostrar el dashboard:
> "Estas son métricas reales de la base de datos. No son datos hardcodeados."

---

## SLIDE 24 — Resultados (30 s)

> "En números: 17 módulos backend, 406 tests pasando, 3 roles de usuario, 2 frontends separados. 93% de cobertura de statements, mínimo 80% en branches. 12 suites E2E."

> "Todo esto en 5 meses de desarrollo en equipo de 4 personas."

---

## SLIDE 25 — Conclusiones y cierre (30 s)

> "SaludDeUna es un sistema que funciona, que tiene tests que lo respaldan, que tiene arquitectura que se puede explicar y defender, y que resuelve un problema real de comunicación en salud."

> "Lo que aprendimos: la IA en salud no puede ser un black box — necesita guardrails explícitos. Los tests E2E con base de datos real son críticos. La observabilidad desde el primer día ahorra horas de debugging. Y la modularidad del backend nos permitió trabajar en paralelo sin conflictos."

> "El código es abierto, los tests están pasando, y la demo acaba de funcionar en vivo."

*[Pausa. Mirar al jurado.]*

> "¿Preguntas?"

---

## Frases de transición entre secciones

| Transición | Frase |
|------------|-------|
| Problema → Solución | "Entonces, ¿qué propone SaludDeUna?" |
| Solución → Arquitectura | "Veamos cómo está construido técnicamente." |
| Arquitectura → Calidad | "Una arquitectura sin calidad medible no sirve de mucho. ¿Cómo validamos la calidad?" |
| Calidad → IA | "Ahora, el componente que hace única a esta plataforma: la integración con IA." |
| IA → Demo | "La mejor forma de entender el sistema es verlo funcionar." |
| Demo → Cierre | "Eso es lo que construimos. Ahora las conclusiones." |

---

## Cómo manejar preguntas difíciles durante la exposición

**Si preguntan sobre diagnóstico:**
> "La IA no diagnostica. El guardrail es código — no una promesa. Si Gemini genera lenguaje de diagnóstico, el sistema lo bloquea y registra el evento. Podemos mostrar el código si quieren."

**Si preguntan por qué no usaron otra tecnología:**
> "Evaluamos alternativas. Elegimos NestJS por su madurez en TypeScript y su sistema de inyección de dependencias. Expo por la velocidad de desarrollo multiplataforma. Gemini porque tiene el mejor balance entre capacidad de lenguaje clínico y costo de API en este momento."

**Si preguntan por el despliegue:**
> "El sistema está desplegado en Railway para el backend y Vercel para el web. La infraestructura objetivo para escala es AWS ECS Fargate — tenemos los Dockerfiles y los diagramas de la arquitectura cloud documentados."

**Si preguntan por regulación:**
> "El sistema es un MVP académico/profesional. No está registrado como dispositivo médico ni como plataforma de telemedicina en el sentido de la Resolución 2654 de 2019. En una etapa de producción real, el análisis regulatorio sería un paso obligatorio antes del lanzamiento."
