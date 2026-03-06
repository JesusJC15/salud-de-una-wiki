# Azure Boards – Épica 1: Onboarding y Acceso Seguro

## Propósito del documento
Este documento contiene toda la información necesaria para implementar la **Épica 1 – Onboarding y Acceso Seguro** en Azure Boards, incluyendo la épica, sus features, historias de usuario con criterios Gherkin, y todas las tareas de desarrollo, pruebas y documentación asociadas. Cada sección indica los campos exactos que se deben completar al crear el ítem en Azure Boards.

---

## 1. ÉPICA

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Epic |
| **ID**                 | E1 |
| **Title**              | Onboarding y Acceso Seguro |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Seguridad |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Priority**           | 1 – Critical |
| **Business Value**     | Incorporar usuarios válidos con control de rol; habilita todas las demás épicas |
| **Risk**               | Medium – Verificación REThUS depende de proceso administrativo externo |
| **Effort (Story Points total estimado)** | 13 SP |
| **Tags**               | auth; onboarding; security; RBAC; REThUS; must |
| **Start Date**         | Sprint 1 |
| **Target Date**        | Cierre Sprint 1 |

### Descripción (campo Description)

```
La Épica 1 cubre el ciclo completo de incorporación y autenticación de los dos perfiles
principales de SaludDeUna: el paciente y el médico.

Objetivos de negocio:
- Garantizar que solo usuarios registrados y autenticados accedan a la plataforma.
- Asegurar que los médicos habilitados hayan sido verificados contra el registro REThUS
  antes de atender cualquier consulta.
- Establecer la base de control de acceso por rol (RBAC) que protege todas las funciones
  clínicas del MVP.

Funcionalidades cubiertas:
- Registro de paciente con validación de correo y contraseña.
- Login con emisión de JWT y gestión de sesión.
- Registro de médico con perfil profesional.
- Flujo de verificación semiautomática REThUS gestionado por el administrador.
- Asignación de roles PACIENTE, MEDICO y ADMIN con permisos diferenciados.

Restricciones:
- No incluye login con redes sociales (fuera de alcance MVP).
- La verificación REThUS es semiautomática: el admin revisa evidencia y aprueba/rechaza;
  no hay consulta automatizada a la API pública de REThUS en este MVP.
- No incluye videollamadas ni integraciones con HCE externos.
```

### Criterios de aceptación de la épica

```
- Al menos un paciente puede registrarse, iniciar sesión y acceder a su panel.
- Al menos un médico puede registrarse, ser verificado (VERIFIED) por el admin y acceder
  a la cola de casos.
- Un médico no verificado (PENDING o REJECTED) no puede ver ni responder consultas.
- Los tokens JWT caducan según la política de seguridad definida en el sprint.
- El log de auditoría registra cada acción de autenticación y cambio de estado REThUS.
```

### Acceptance Criteria – formato Gherkin (nivel épica)

```gherkin
Feature: Control de acceso por rol en SaludDeUna

  Scenario: Paciente autenticado accede a su panel
    Given un paciente con cuenta registrada y correo verificado
    When inicia sesión con credenciales correctas
    Then recibe un JWT válido con rol PACIENTE
    And puede navegar a su panel de consultas

  Scenario: Médico no verificado bloqueado
    Given un médico recién registrado en estado PENDING
    When intenta acceder a la cola de casos
    Then el sistema rechaza la solicitud con HTTP 403
    And muestra mensaje de verificación pendiente

  Scenario: Admin cambia estado REThUS a VERIFIED
    Given un médico en estado PENDING con evidencia cargada
    When el admin aprueba la verificación
    Then el estado cambia a VERIFIED
    And el médico puede atender consultas a partir de ese momento
```

---

## 2. FEATURES

### Feature F1.1 – Registro y Login de Paciente

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F1.1 |
| **Title**              | Registro y Login de Paciente |
| **Parent (Epic)**      | E1 – Onboarding y Acceso Seguro |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 5 SP |
| **Tags**               | auth; paciente; registro; login; JWT; must |

#### Descripción (campo Description)

```
Permite a un usuario nuevo registrarse como paciente en la plataforma SaludDeUna
y posteriormente iniciar sesión.

Alcance funcional:
- Formulario de registro: nombre, apellido, correo electrónico, contraseña, fecha
  de nacimiento, género (opcional).
- Validación de unicidad de correo en base de datos.
- Hash de contraseña con bcrypt (cost factor >= 12).
- Endpoint POST /v1/auth/patient/register.
- Login con correo + contraseña → JWT (access token + refresh token).
- Endpoint POST /v1/auth/login.
- Pantalla de registro y login en la app móvil (React Native).

Criterios de salida del feature:
- Un paciente puede registrarse y hacer login en la app desde un dispositivo móvil.
- Los tokens son válidos y contienen el claim de rol PACIENTE.
- Correos duplicados son rechazados con mensaje de error apropiado.
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Registro y login de paciente

  Scenario: Registro exitoso de nuevo paciente
    Given un usuario sin cuenta en la plataforma
    When completa el formulario con nombre, correo único y contraseña válida
    Then el sistema crea la cuenta con rol PACIENTE
    And retorna HTTP 201 con datos del perfil creado (sin contraseña)
    And permite iniciar sesión inmediatamente

  Scenario: Registro rechazado por correo duplicado
    Given un correo ya registrado en la base de datos
    When un usuario intenta registrarse con ese mismo correo
    Then el sistema retorna HTTP 409
    And muestra el mensaje "El correo ya está registrado"

  Scenario: Login exitoso
    Given un paciente con cuenta activa
    When envía correo y contraseña correctos al endpoint de login
    Then recibe HTTP 200 con access_token y refresh_token
    And el JWT contiene el claim role=PACIENTE

  Scenario: Login fallido por credenciales incorrectas
    Given un paciente con cuenta activa
    When envía una contraseña incorrecta
    Then recibe HTTP 401
    And el sistema no revela si el correo existe o no
```

---

### Feature F1.2 – Registro de Médico y Verificación REThUS

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Feature |
| **ID**                 | F1.2 |
| **Title**              | Registro de Médico y Verificación REThUS |
| **Parent (Epic)**      | E1 – Onboarding y Acceso Seguro |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Web-Admin |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Effort (SP)**        | 8 SP |
| **Tags**               | auth; medico; REThUS; admin; registro; verificacion; must |

#### Descripción (campo Description)

```
Permite a un médico registrarse en la plataforma con su información profesional y
al administrador validar sus credenciales contra el registro REThUS.

Alcance funcional:
- Formulario médico: nombre, apellido, correo, contraseña, especialidad
  (GENERAL_MEDICINE | DENTISTRY), número REThUS, matrícula profesional.
- Endpoint POST /v1/auth/doctor/register → estado inicial PENDING.
- Panel admin web (Next.js) con lista de médicos PENDING y botón de verificar.
- Endpoint POST /v1/admin/doctors/{doctorId}/rethus-verify con campos:
    { status: VERIFIED | REJECTED, checkedBy, evidenceUrl, notes }.
- Envío de notificación interna al médico con resultado de verificación.
- Persistencia del objeto RethusVerification en MongoDB.
- RBAC: solo el rol ADMIN puede ejecutar la verificación.

Criterios de salida del feature:
- Un médico puede registrarse. Estado inicial = PENDING.
- Un admin puede ver la lista de médicos PENDING y verificar/rechazar.
- Un médico VERIFIED puede iniciar sesión y ver la cola de casos.
- Un médico REJECTED o PENDING no puede atender consultas (HTTP 403).
```

#### Acceptance Criteria del Feature

```gherkin
Feature: Registro de médico y verificación REThUS

  Scenario: Registro exitoso de médico
    Given un profesional médico sin cuenta
    When completa el formulario con datos válidos incluyendo número REThUS
    Then el sistema crea la cuenta con rol MEDICO y estado PENDING
    And retorna HTTP 201

  Scenario: Admin aprueba verificación REThUS
    Given un médico en estado PENDING y un admin autenticado
    When el admin envía POST /v1/admin/doctors/{doctorId}/rethus-verify con status=VERIFIED
    Then el estado del médico cambia a VERIFIED en la base de datos
    And se registra el objeto RethusVerification con checkedBy y checkedAt
    And el médico recibe notificación de habilitación

  Scenario: Admin rechaza verificación REThUS
    Given un médico en estado PENDING y documentación inconsistente
    When el admin envía POST /v1/admin/doctors/{doctorId}/rethus-verify con status=REJECTED
    Then el estado del médico cambia a REJECTED
    And el médico recibe notificación de rechazo con notas del admin

  Scenario: Médico PENDING bloqueado de cola de casos
    Given un médico con estado PENDING
    When intenta acceder a GET /v1/consultations/queue
    Then el sistema retorna HTTP 403
    And el mensaje indica que debe completar la verificación profesional
```

---

## 3. HISTORIAS DE USUARIO

### HU-001 – Registro e Inicio de Sesión del Paciente

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-001 |
| **Title**              | Como paciente nuevo quiero registrarme e iniciar sesión para gestionar mis consultas de forma segura |
| **Parent (Feature)**   | F1.1 – Registro y Login de Paciente |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Mobile |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Story Points**       | 5 |
| **Risk**               | Low |
| **Dependencias**       | Ninguna |
| **Tags**               | auth; paciente; registro; login; must; sprint1 |

#### Descripción completa (campo Description)

```
Como paciente nuevo
Quiero registrarme con mis datos personales e iniciar sesión con mis credenciales
Para gestionar mis consultas médicas de forma segura desde la aplicación móvil.

Contexto:
El paciente es el usuario principal del producto. Sin un registro funcional y seguro,
ninguna funcionalidad clínica puede ejecutarse. Este es el punto de entrada del MVP.

Restricciones funcionales:
- La contraseña debe cumplir la política centralizada (ver T-001-03): mínimo 8 caracteres,
  al menos una mayúscula, un número y un carácter especial.
- El correo electrónico debe ser único en el sistema.
- El token JWT tiene una duración de 1 hora para access y 7 días para refresh.
- No se guarda la contraseña en texto plano: se usa bcrypt con factor 12.

Notas de UX:
- La app móvil debe mostrar errores de validación inline en el formulario.
- Al registrarse exitosamente, el usuario entra directamente al dashboard sin
  necesidad de hacer login por separado.
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Registro y login de paciente

  Scenario: Registro principal exitoso
    Given un usuario paciente sin cuenta
    When completa nombre, correo y contraseña válidos
    Then el sistema crea la cuenta con rol PACIENTE
    And permite iniciar sesión con token válido

  Scenario: Registro alterno por correo duplicado
    Given un correo ya registrado
    When el usuario intenta crear una nueva cuenta con ese correo
    Then el sistema rechaza la operación
    And muestra mensaje de conflicto de identidad

  Scenario: Login con credenciales correctas
    Given un paciente con cuenta activa
    When ingresa correo y contraseña correctos
    Then recibe access_token con claim role=PACIENTE
    And puede acceder a su panel de consultas

  Scenario: Login con contraseña incorrecta
    Given un paciente con cuenta activa
    When ingresa una contraseña incorrecta
    Then recibe mensaje de error genérico sin revelar si el correo existe
    And no se emite ningún token
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y feature definidos (E1, F1.1)
- [x] Prioridad MoSCoW asignada (Must)
- [x] Criterios Gherkin documentados (principal y alterno)
- [x] Dependencias identificadas (ninguna)
- [x] Estimación acordada (5 SP)
- [x] Datos/ambientes necesarios identificados (MongoDB dev, Azure dev)
- [x] Riesgos anotados (bajo – flujo de autenticación estándar)
- [x] Criterios de seguridad definidos (bcrypt, JWT, validación de input)
- [x] Responsable de validación funcional asignado

#### Definition of Done (DoD) – Checklist

- [ ] Endpoints `POST /v1/auth/patient/register` y `POST /v1/auth/login` implementados y revisados por al menos un par
- [ ] Hash de contraseña con bcrypt factor >= 12 verificado
- [ ] Pruebas unitarias del servicio de autenticación en verde
- [ ] Pruebas de integración de los endpoints en verde
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] Log de auditoría registra eventos de registro y login
- [ ] Validaciones de seguridad ejecutadas (SQL injection, input sanitization)
- [ ] Pantalla de registro y login en app React Native implementada
- [ ] Documentación técnica actualizada en Wiki (endpoints, contratos)
- [ ] Demo funcional aprobada por el equipo
- [ ] Historia pasada a estado Done con evidencia enlazada (PR + test results)

---

### HU-002 – Validación REThUS de Médico por Administrador

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | User Story |
| **ID**                 | HU-002 |
| **Title**              | Como admin de plataforma quiero validar a un médico con soporte REThUS para asegurar autenticidad profesional |
| **Parent (Feature)**   | F1.2 – Registro de Médico y Verificación REThUS |
| **State**              | Active |
| **Area Path**          | SaludDeUna\\Backend / SaludDeUna\\Web-Admin |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Priority**           | 1 – Critical |
| **MoSCoW**             | Must |
| **Story Points**       | 8 |
| **Risk**               | Medium – depende de que el admin tenga acceso a la evidencia REThUS |
| **Dependencias**       | HU-001 (base de auth y RBAC activa) |
| **Tags**               | auth; medico; REThUS; admin; verificacion; RBAC; must; sprint1 |

#### Descripción completa (campo Description)

```
Como administrador de la plataforma
Quiero revisar y validar el registro profesional de un médico contra el sistema REThUS
Para asegurar que solo profesionales con matrícula vigente puedan atender consultas.

Contexto:
La verificación REThUS es un requisito regulatorio y de confianza del producto.
Sin este control, cualquier persona podría registrarse como médico y atender pacientes.
La validación en el MVP es semiautomática: el admin revisa el número de matrícula,
la especialidad y la evidencia cargada, y decide aprobar o rechazar.

Flujo del proceso:
1. Médico se registra → estado PENDING.
2. Admin recibe notificación o ve la lista de médicos PENDING en el panel.
3. Admin revisa el número REThUS y la evidencia adjunta.
4. Admin aprueba (VERIFIED) o rechaza (REJECTED) con notas.
5. El sistema actualiza el estado y notifica al médico.

Restricciones:
- Solo el rol ADMIN puede ejecutar el endpoint de verificación.
- El objeto RethusVerification debe persistir con trazabilidad completa
  (checkedBy, checkedAt, evidenceUrl, notes).
- Un médico REJECTED puede reenviar evidencia y volver a PENDING (en alcance de HU-002).
- La verificación no es automática: no hay consulta a API REThUS en este MVP.
```

#### Criterios de Aceptación – Gherkin

```gherkin
Feature: Validación profesional de médico por admin

  Scenario: Validación principal aprobada por admin
    Given un médico en estado PENDING
    When el admin registra evidencia de verificación REThUS y aprueba
    Then el estado del médico cambia a VERIFIED
    And se habilita acceso a la cola de casos
    And el médico recibe notificación de habilitación

  Scenario: Validación alterna rechazada
    Given un médico en estado PENDING
    When el admin registra inconsistencia documental y rechaza
    Then el estado cambia a REJECTED
    And el médico no puede atender consultas
    And el médico recibe notificación de rechazo con notas del admin

  Scenario: Médico VERIFIED accede a cola de casos
    Given un médico con estado VERIFIED
    When inicia sesión y accede a GET /v1/consultations/queue
    Then recibe HTTP 200 con la lista de casos asignados

  Scenario: Médico reenvía evidencia tras rechazo
    Given un médico con estado REJECTED
    When carga nueva evidencia y solicita revisión
    Then el estado vuelve a PENDING
    And el admin puede reiniciar el proceso de verificación
```

#### Definition of Ready (DoR) – Checklist

- [x] Historia escrita en formato Como/Quiero/Para
- [x] ID, épica y feature definidos (E1, F1.2)
- [x] Prioridad MoSCoW asignada (Must)
- [x] Criterios Gherkin documentados (principal y alterno)
- [x] Dependencias identificadas (HU-001 desbloqueada en el mismo sprint)
- [x] Estimación acordada (8 SP)
- [x] Datos/ambientes necesarios identificados (MongoDB dev, panel admin web dev)
- [x] Riesgos anotados (proceso admin externo, sin API REThUS automática)
- [x] Criterios de seguridad definidos (RBAC ADMIN, auditoría de cambios de estado)
- [x] Responsable de validación funcional asignado

#### Definition of Done (DoD) – Checklist

- [ ] Endpoint `POST /v1/auth/doctor/register` implementado (estado inicial PENDING)
- [ ] Endpoint `POST /v1/admin/doctors/{doctorId}/rethus-verify` implementado con RBAC ADMIN
- [ ] Objeto `RethusVerification` persistido en MongoDB con todos los campos requeridos
- [ ] Código revisado por al menos un par
- [ ] Pruebas unitarias del servicio de verificación en verde
- [ ] Pruebas de integración del endpoint admin en verde (incluyendo rol no-admin → 403)
- [ ] Criterios Gherkin verificados en entorno de pruebas
- [ ] Panel admin web muestra lista de médicos PENDING y acción de verificar/rechazar
- [ ] Log de auditoría registra cada cambio de estado REThUS con actor y timestamp
- [ ] Validaciones de seguridad ejecutadas (auth, autorización por rol ADMIN, input sanitization)
- [ ] Documentación técnica actualizada en Wiki (endpoints, contrato RethusVerification)
- [ ] Demo funcional aprobada por el equipo
- [ ] Historia pasada a estado Done con evidencia enlazada (PR + test results)

---

## 4. TAREAS

### Tareas de HU-001 – Registro e Inicio de Sesión del Paciente

---

#### T-001-01 – Diseño de modelo de datos Patient en MongoDB

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Diseñar esquema Mongoose Patient con campos requeridos y validaciones |
| **Parent (User Story)**| HU-001 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Crear el schema Mongoose para el modelo Patient incluyendo:
- _id (ObjectId generado automáticamente)
- firstName, lastName (String, required)
- email (String, unique, required, lowercase, trim)
- passwordHash (String, required, select: false)
- role (String, referencia al enum centralizado UserRole.PACIENTE, default: UserRole.PACIENTE)
- birthDate (Date, optional)
- gender (String, enum: ['M', 'F', 'OTHER'], optional)
- isActive (Boolean, default: true)
- createdAt, updatedAt (timestamps automáticos)

IMPORTANTE – Enum centralizado:
Definir UserRole = { PACIENTE: 'PACIENTE', MEDICO: 'MEDICO', ADMIN: 'ADMIN' } en
apps/api/src/common/enums/user-role.enum.ts y referenciar desde Patient, Doctor y
JwtPayload para garantizar consistencia.

Agregar índice único en email.
No incluir la contraseña en las respuestas de la API (select: false).
Ubicar en: apps/api/src/patients/schemas/patient.schema.ts
```

**Criterios de aceptación de la tarea:**
- Schema creado y exportado correctamente.
- Índice único en email verificado en MongoDB local.
- No expone passwordHash en ninguna consulta por defecto.

---

#### T-001-02 – Implementación del endpoint POST /v1/auth/patient/register

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint POST /v1/auth/patient/register en NestJS |
| **Parent (User Story)**| HU-001 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el endpoint de registro de paciente:
- Módulo: AuthModule (NestJS)
- Ruta: POST /v1/auth/patient/register
- DTO de entrada: RegisterPatientDto { firstName, lastName, email, password, birthDate?, gender? }
- Validaciones con class-validator usando la política de contraseñas centralizada definida
  en apps/api/src/common/validators/password-policy.ts (ver sección de política de contraseñas
  en T-001-03).
- Hash de contraseña con bcrypt (cost factor 12).
- Verificar unicidad de correo antes de crear → lanzar ConflictException si existe.
- Respuesta exitosa HTTP 201: { id, firstName, lastName, email, role, createdAt }
- No incluir passwordHash en la respuesta.
- Registrar evento de auditoría en el log estructurado con nivel INFO.
Ubicar en: apps/api/src/auth/
```

**Criterios de aceptación de la tarea:**
- Endpoint responde 201 con datos correctos al recibir payload válido.
- Retorna 409 si el correo ya existe.
- Retorna 400 con errores de validación si el payload es inválido.
- La contraseña nunca aparece en logs ni en la respuesta.

---

#### T-001-03 – Implementación del endpoint POST /v1/auth/login

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint POST /v1/auth/login con emisión de JWT |
| **Parent (User Story)**| HU-001 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el endpoint de login para todos los roles:
- Ruta: POST /v1/auth/login
- DTO: LoginDto { email, password }
- Verificar existencia del usuario por email (paciente o médico).
- Comparar contraseña con bcrypt.compare.
- Si las credenciales son incorrectas → lanzar UnauthorizedException con mensaje
  genérico ("Credenciales inválidas") sin revelar si el correo existe.
- Emitir JWT de acceso (exp: 1h) y refresh token (exp: 7d) firmados con clave secreta
  almacenada en variable de entorno JWT_SECRET.
  Política de contraseñas y JWT (definir en apps/api/src/common/validators/password-policy.ts):
    · Contraseña: mínimo 8 caracteres, al menos 1 mayúscula, 1 número y 1 carácter especial.
    · JWT_SECRET: mínimo 32 caracteres alfanuméricos aleatorios; generado con `openssl rand -hex 32`.
    · Rotación recomendada: semestral o ante compromiso confirmado.
- Payload del JWT: { sub: userId, role, email, iat, exp }
- Respuesta HTTP 200: { access_token, refresh_token, user: { id, email, role } }
- Registrar evento de auditoría (login exitoso / fallido) en log estructurado.
Ubicar en: apps/api/src/auth/
```

**Criterios de aceptación de la tarea:**
- Login exitoso retorna 200 con access_token y refresh_token válidos.
- JWT contiene claim role=PACIENTE para pacientes.
- Login fallido retorna 401 con mensaje genérico.
- La clave JWT nunca está hardcodeada; se lee de variable de entorno.

---

#### T-001-04 – Guard JWT y decorador @Roles en NestJS

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar JwtAuthGuard y decorador @Roles para RBAC |
| **Parent (User Story)**| HU-001 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Implementar la infraestructura de autorización reutilizable:
- JwtAuthGuard: valida la firma del JWT en el header Authorization Bearer.
- RolesGuard: verifica que el rol en el JWT coincide con el rol requerido por el endpoint.
- Decorador @Roles(...roles): permite decorar controladores o métodos con los roles permitidos.
- Aplicar JwtAuthGuard globalmente con APP_GUARD excepto en rutas públicas.
- Crear decorador @Public() para excluir rutas de auth (register, login).
- Ubicar en: apps/api/src/common/guards/
```

**Criterios de aceptación de la tarea:**
- Un endpoint protegido con @Roles('ADMIN') retorna 403 si el JWT tiene rol PACIENTE.
- Una ruta decorada con @Public() no requiere JWT.
- El guard extrae correctamente el userId y role del payload del JWT.

---

#### T-001-05 – Pantallas de Registro y Login en React Native

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar pantallas de Registro y Login en la app React Native |
| **Parent (User Story)**| HU-001 |
| **Assigned To**        | Desarrollador Mobile |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar las dos pantallas principales de onboarding en la app móvil:

RegisterScreen:
- Campos: nombre, apellido, correo, contraseña, confirmación de contraseña.
- Validación inline con mensajes de error debajo de cada campo.
- Botón "Registrarme" llama a POST /v1/auth/patient/register.
- Al éxito, navega al Dashboard; al error, muestra el mensaje del backend.

LoginScreen:
- Campos: correo, contraseña.
- Botón "Iniciar sesión" llama a POST /v1/auth/login.
- Al éxito, guarda los tokens en SecureStore y navega al Dashboard.
- Al error, muestra "Correo o contraseña incorrectos".
- Link a RegisterScreen para nuevos usuarios.

Almacenamiento de tokens:
- Usar expo-secure-store para guardar access_token y refresh_token.
- No guardar tokens en AsyncStorage ni en variables de estado globales sin cifrado.

Ubicar en: apps/mobile/src/screens/auth/
```

**Criterios de aceptación de la tarea:**
- Formulario muestra errores inline al ingresar datos inválidos.
- Login exitoso almacena tokens y navega al Dashboard.
- Contraseña nunca visible en texto plano; usar `secureTextEntry`.
- La app no crashea en ningún flujo de error del backend.

---

#### T-001-06 – Pruebas unitarias del servicio de autenticación (paciente)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias para AuthService (registro y login de paciente) |
| **Parent (User Story)**| HU-001 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Escribir pruebas unitarias con Jest para el AuthService, cubriendo:
- registerPatient: registro exitoso → verifica creación en DB mockeada.
- registerPatient: correo duplicado → verifica ConflictException.
- login: credenciales correctas → verifica emisión de JWT.
- login: contraseña incorrecta → verifica UnauthorizedException.
- login: usuario no existe → verifica UnauthorizedException genérica.
- Usar mocks de PatientRepository y JwtService.
- Cobertura mínima objetivo: 80% de líneas del AuthService.
Ubicar en: apps/api/src/auth/auth.service.spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI (`npm run test`).
- Cobertura >= 80% del AuthService.
- No hay dependencias reales de base de datos en estas pruebas (todo mockeado).

---

#### T-001-07 – Pruebas de integración de endpoints de auth (paciente)

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas de integración para POST /v1/auth/patient/register y POST /v1/auth/login |
| **Parent (User Story)**| HU-001 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Escribir pruebas de integración con supertest + Jest contra una instancia real de la API
con MongoDB de prueba (en memoria con @shelf/jest-mongodb o MongoMemoryServer):
- Registro exitoso → HTTP 201 con campos correctos.
- Registro duplicado → HTTP 409.
- Registro con payload inválido → HTTP 400.
- Login exitoso → HTTP 200 con tokens válidos.
- Login con contraseña incorrecta → HTTP 401.
Ubicar en: apps/api/test/auth.e2e-spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests de integración pasan en CI.
- Los tests se ejecutan de forma aislada sin depender de datos externos.

---

#### T-001-08 – Documentación técnica de endpoints de auth en Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar endpoints de autenticación de paciente en la Wiki del proyecto |
| **Parent (User Story)**| HU-001 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 1 |

**Descripción:**
```
Actualizar o crear sección en la Wiki de Azure DevOps con:
- Descripción de POST /v1/auth/patient/register: request body, response 201, errores 400/409.
- Descripción de POST /v1/auth/login: request body, response 200 con tokens, errores 401.
- Política de JWT: duración de access y refresh token, claims incluidos.
- Política de contraseñas: requisitos mínimos.
- Referencia a la historia HU-001 y al sprint de implementación.
```

**Criterios de aceptación de la tarea:**
- La Wiki refleja los contratos reales de los endpoints implementados.
- Un nuevo desarrollador puede usar la documentación para consumir los endpoints sin leer el código fuente.

---

### Tareas de HU-002 – Validación REThUS de Médico

---

#### T-002-01 – Diseño de modelo de datos Doctor y RethusVerification en MongoDB

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Diseñar esquemas Mongoose Doctor y RethusVerification |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Crear los schemas Mongoose para el modelo médico:

Doctor schema:
- _id, firstName, lastName, email (unique), passwordHash (select:false)
- role: referencia al enum centralizado UserRole.MEDICO (ver T-001-01)
- specialty: enum ['GENERAL_MEDICINE', 'DENTISTRY'], required
- rethusNumber (String, required)
- professionalLicense (String, optional)
- rethusStatus: enum ['PENDING', 'VERIFIED', 'REJECTED'], default 'PENDING'
- rethusVerification (ref a RethusVerification, optional)
- isActive (Boolean, default: true)
- timestamps

RethusVerification schema:
- _id, doctorId (ref Doctor, required)
- status: enum ['VERIFIED', 'REJECTED'], required
- checkedBy (String, required) – email del admin que verificó
- checkedAt (Date, required)
- evidenceUrl (String, optional)
- notes (String, optional)
- timestamps

Ubicar en: apps/api/src/doctors/schemas/
```

**Criterios de aceptación de la tarea:**
- Ambos schemas creados y exportados correctamente.
- El campo rethusStatus por defecto es PENDING.
- RethusVerification referencia el Doctor por ObjectId.

---

#### T-002-02 – Implementación del endpoint POST /v1/auth/doctor/register

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint POST /v1/auth/doctor/register en NestJS |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Implementar el endpoint de registro de médico:
- Ruta: POST /v1/auth/doctor/register
- DTO: RegisterDoctorDto { firstName, lastName, email, password, specialty, rethusNumber, professionalLicense? }
- Validaciones con class-validator.
- Hash de contraseña con bcrypt (factor 12).
- Estado inicial rethusStatus = PENDING.
- Respuesta HTTP 201: { id, firstName, lastName, email, role, specialty, rethusStatus }
- Registrar evento de auditoría en log estructurado.
Ubicar en: apps/api/src/auth/
```

**Criterios de aceptación de la tarea:**
- Endpoint retorna 201 con rethusStatus=PENDING.
- Correo duplicado retorna 409.
- Payload inválido retorna 400 con detalle de errores.

---

#### T-002-03 – Implementación del endpoint POST /v1/admin/doctors/{doctorId}/rethus-verify

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar endpoint de verificación REThUS para admin |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar el endpoint de verificación REThUS:
- Ruta: POST /v1/admin/doctors/{doctorId}/rethus-verify
- Protegido con @Roles('ADMIN') y JwtAuthGuard.
- DTO: RethusVerifyDto { status: 'VERIFIED' | 'REJECTED', evidenceUrl?, notes? }
- Lógica:
    1. Buscar doctor por doctorId → 404 si no existe.
    2. Crear documento RethusVerification con { doctorId, status, checkedBy (del JWT),
       checkedAt (Date.now()), evidenceUrl, notes }.
    3. Actualizar rethusStatus del Doctor al valor del DTO.
    4. Guardar ambos documentos en una sesión de MongoDB (transacción).
       Si la transacción falla, hacer rollback y retornar HTTP 500 con mensaje
       "Error interno al actualizar verificación; sin cambios aplicados".
    5. Disparar notificación interna al médico (evento interno o log).
- Respuesta HTTP 200: { doctorId, rethusStatus, checkedAt }
- Registrar en log estructurado con correlation ID.
Ubicar en: apps/api/src/admin/
```

**Criterios de aceptación de la tarea:**
- Admin con rol ADMIN puede aprobar o rechazar. Retorna 200 con datos actualizados.
- Usuario con rol PACIENTE o MEDICO recibe 403.
- DoctorId inexistente recibe 404.
- RethusVerification persiste en MongoDB con todos los campos.
- El rethusStatus del Doctor se actualiza atómicamente.
- Si la transacción falla a mitad de camino, ningún documento queda modificado (rollback verificado).

---

#### T-002-04 – Panel admin web: lista de médicos PENDING y acción de verificar

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar vista de verificación REThUS en el panel admin (Next.js) |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | Desarrollador Web |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 4 |

**Descripción:**
```
Implementar en el panel web (Next.js + React) la pantalla de verificación REThUS:
- Ruta del panel: /admin/doctors/verification
- Tabla con médicos en estado PENDING: nombre, especialidad, rethusNumber, fecha de registro.
- Botón "Verificar" y botón "Rechazar" por fila.
- Modal de confirmación con campo de notas y campo opcional de URL de evidencia.
- Al confirmar, llama a POST /v1/admin/doctors/{doctorId}/rethus-verify.
- Actualiza el estado en la tabla sin recargar la página (optimistic update o refetch).
- Solo visible para usuarios con rol ADMIN; redirige a /403 si el rol no coincide.
Ubicar en: apps/web/src/app/admin/doctors/verification/
```

**Criterios de aceptación de la tarea:**
- La tabla muestra solo médicos PENDING.
- El modal permite ingresar notas y URL de evidencia.
- Tras la acción, el médico desaparece de la lista PENDING.
- Acceso desde un rol no-ADMIN redirige a /403.

---

#### T-002-05 – Implementación del guard de estado VERIFIED en endpoints clínicos

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar guard que bloquea médicos no verificados en endpoints clínicos |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Crear un guard NestJS DoctorVerifiedGuard que:
- Verifica que el usuario autenticado con rol MEDICO tenga rethusStatus = VERIFIED.
- Si rethusStatus = PENDING o REJECTED → lanza ForbiddenException con mensaje
  "Acceso restringido: verificación profesional pendiente".
- Aplicar el guard en: GET /v1/consultations/queue y en todos los endpoints de consulta.
- El guard solo aplica al rol MEDICO; ADMIN y PACIENTE no son afectados.
Ubicar en: apps/api/src/common/guards/doctor-verified.guard.ts
```

**Criterios de aceptación de la tarea:**
- Médico PENDING recibe 403 al intentar acceder a la cola de casos.
- Médico VERIFIED accede correctamente.
- El guard no afecta a pacientes ni admins.

---

#### T-002-06 – Notificación interna de resultado de verificación REThUS

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Implementar notificación interna al médico tras cambio de estado REThUS |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Development |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Al cambiar el estado REThUS del médico, emitir una notificación interna:
- MVP: registrar en la colección Notifications de MongoDB un documento con
  { userId: doctorId, type: 'RETHUS_STATUS_CHANGE', status, message, createdAt, read: false }.
- El médico puede consultar sus notificaciones mediante GET /v1/notifications (fuera
  del alcance de HU-002, pero el documento debe crearse correctamente).
- Registrar en log estructurado el evento de notificación con correlation ID.
- No se requiere email externo en este sprint; solo persistencia interna.
Ubicar en: apps/api/src/notifications/
```

**Criterios de aceptación de la tarea:**
- Tras aprobar/rechazar, existe un documento en Notifications para el médico.
- El documento tiene los campos correctos (type, status, message, read=false).
- El log registra el evento de notificación.

---

#### T-002-07 – Pruebas unitarias del servicio de verificación REThUS

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas unitarias para AdminService (verificación REThUS) |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | Desarrollador Backend / QA |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 3 |

**Descripción:**
```
Escribir pruebas unitarias con Jest para AdminService y DoctorVerifiedGuard:
- verifyDoctor VERIFIED: actualiza estado, crea RethusVerification, crea Notification.
- verifyDoctor REJECTED: actualiza estado, crea RethusVerification con status REJECTED.
- verifyDoctor con doctorId inexistente: lanza NotFoundException.
- DoctorVerifiedGuard con médico PENDING: lanza ForbiddenException.
- DoctorVerifiedGuard con médico VERIFIED: permite paso.
- Cobertura mínima objetivo: 80% del AdminService.
Ubicar en: apps/api/src/admin/admin.service.spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI.
- Mocks de repositorios usados; sin dependencias reales de BD.
- Cobertura >= 80%.

---

#### T-002-08 – Pruebas de integración del endpoint de verificación REThUS

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Escribir pruebas de integración para POST /v1/admin/doctors/{doctorId}/rethus-verify |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | QA / Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Testing |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 2 |

**Descripción:**
```
Pruebas de integración con supertest + MongoMemoryServer:
- Admin aprueba médico PENDING → 200, rethusStatus=VERIFIED, RethusVerification creado.
- Admin rechaza médico PENDING → 200, rethusStatus=REJECTED.
- No-admin intenta verificar → 403.
- DoctorId inexistente → 404.
- Médico PENDING accede a cola de casos → 403.
- Médico VERIFIED accede a cola de casos → 200 (stub de cola vacía).
Ubicar en: apps/api/test/admin-rethus.e2e-spec.ts
```

**Criterios de aceptación de la tarea:**
- Todos los tests pasan en CI de forma aislada.
- Se cubre el flujo completo de verificación y bloqueo.

---

#### T-002-09 – Documentación técnica de endpoints de médico y verificación REThUS en Wiki

| Campo Azure Boards     | Valor |
|------------------------|-------|
| **Work Item Type**     | Task |
| **Title**              | Documentar endpoints de registro de médico y verificación REThUS en la Wiki |
| **Parent (User Story)**| HU-002 |
| **Assigned To**        | Desarrollador Backend |
| **State**              | To Do |
| **Activity**           | Documentation |
| **Iteration Path**     | SaludDeUna\\Sprint 1 |
| **Remaining Work (h)** | 1 |

**Descripción:**
```
Actualizar la Wiki con:
- POST /v1/auth/doctor/register: request, response 201, errores.
- POST /v1/admin/doctors/{doctorId}/rethus-verify: request, response 200, errores 403/404.
- Descripción del flujo semiautomático REThUS (PENDING → VERIFIED/REJECTED).
- Contrato del objeto RethusVerification con todos los campos.
- Roles requeridos por endpoint.
- Referencia a HU-002 y al sprint de implementación.
```

**Criterios de aceptación de la tarea:**
- La Wiki refleja los contratos reales de los endpoints.
- El flujo REThUS es comprensible para un desarrollador nuevo sin leer el código.

---

## 5. RESUMEN DE ESTRUCTURA EN AZURE BOARDS

```
📌 EPIC: E1 – Onboarding y Acceso Seguro (13 SP total)
│
├── 🔷 FEATURE: F1.1 – Registro y Login de Paciente (5 SP)
│   │
│   └── 📖 USER STORY: HU-001 – Como paciente nuevo quiero registrarme e iniciar sesión (5 SP)
│       ├── [ ] T-001-01 – Diseño de modelo Patient en MongoDB (2h)
│       ├── [ ] T-001-02 – Endpoint POST /v1/auth/patient/register (3h)
│       ├── [ ] T-001-03 – Endpoint POST /v1/auth/login con JWT (3h)
│       ├── [ ] T-001-04 – JwtAuthGuard y @Roles para RBAC (2h)
│       ├── [ ] T-001-05 – Pantallas Registro y Login en React Native (4h)
│       ├── [ ] T-001-06 – Pruebas unitarias AuthService paciente (3h)
│       ├── [ ] T-001-07 – Pruebas de integración endpoints auth paciente (2h)
│       └── [ ] T-001-08 – Documentación técnica en Wiki (1h)
│
└── 🔷 FEATURE: F1.2 – Registro de Médico y Verificación REThUS (8 SP)
    │
    └── 📖 USER STORY: HU-002 – Como admin quiero validar médico con soporte REThUS (8 SP)
        ├── [ ] T-002-01 – Diseño de modelos Doctor y RethusVerification (2h)
        ├── [ ] T-002-02 – Endpoint POST /v1/auth/doctor/register (3h)
        ├── [ ] T-002-03 – Endpoint POST /v1/admin/doctors/{id}/rethus-verify (4h)
        ├── [ ] T-002-04 – Panel admin: lista PENDING y acción verificar (Next.js) (4h)
        ├── [ ] T-002-05 – Guard DoctorVerifiedGuard en endpoints clínicos (2h)
        ├── [ ] T-002-06 – Notificación interna de resultado REThUS (2h)
        ├── [ ] T-002-07 – Pruebas unitarias AdminService (3h)
        ├── [ ] T-002-08 – Pruebas de integración verificación REThUS (2h)
        └── [ ] T-002-09 – Documentación técnica en Wiki (1h)
```

**Total horas estimadas Sprint 1 (Épica 1):** 42 horas de trabajo  
**Total Story Points Épica 1:** 13 SP (HU-001: 5 SP + HU-002: 8 SP)  
**Sprint objetivo:** Sprint 1

---

## 6. REFERENCIAS CRUZADAS

| Artefacto | Referencia |
|-----------|-----------|
| Plan Maestro | `Plan Maestro SaludDeUna (IETI 2026-1).md` – Sprints y MoSCoW |
| Story Map | `docs/wiki/05-Epicas-Features-StoryMap.md` – Actividad 1: Onboarding |
| Backlog completo | `docs/wiki/06-Backlog-Historias-Usuario.md` – HU-001, HU-002 |
| Criterios Gherkin | `docs/wiki/07-Criterios-Aceptacion-Gherkin.md` – HU-001, HU-002 |
| DoR / DoD | `docs/wiki/08-DoR-DoD.md` |
| KPIs y SLOs | `docs/wiki/09-Observabilidad-KPIs.md` |
| Riesgos | `docs/wiki/12-Riesgos-Concurrencia-RealTime.md` |

### API endpoints de esta épica (contratos del Plan Maestro)
- `POST /v1/auth/patient/register` → T-001-02
- `POST /v1/auth/doctor/register` → T-002-02
- `POST /v1/auth/login` → T-001-03
- `POST /v1/admin/doctors/{doctorId}/rethus-verify` → T-002-03

### KPIs impactados por esta épica
- **SLO Disponibilidad > 99%**: el módulo de auth es crítico; su indisponibilidad bloquea toda la plataforma.
- **KPI Retención a 7 días**: un onboarding simple y seguro reduce abandono inicial.

### Riesgos asociados
- **R-005 (Sobrealcance)**: Congelar Must al cierre de Sprint 1; no agregar flujos OAuth en este sprint.
- **R-001 (Multi-cloud)**: La verificación REThUS es semiautomática; no depende de servicios externos en este sprint.
