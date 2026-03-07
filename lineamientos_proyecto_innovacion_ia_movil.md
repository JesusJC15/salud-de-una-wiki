# Universidad Escuela Colombiana de Ingeniería Julio Garavito

## Asignatura
**Innovación y Nuevas Tecnologías**

# Lineamientos del Proyecto de Curso  
## Desarrollo de una Aplicación Móvil con Integración de Inteligencia Artificial

**Profesor:** Rodrigo Gualtero  
**Semestre:** 2026-1

---

# 1. Objetivo del Proyecto

El objetivo del proyecto es diseñar, planificar e implementar una **aplicación móvil innovadora** que incorpore casos de uso de **Inteligencia Artificial (IA)** para resolver un problema real.

El desarrollo técnico debe respaldar el **diseño de producto elaborado previamente durante las sesiones teóricas del curso**, asegurando coherencia entre:

- Problema identificado  
- Propuesta de valor  
- Experiencia de usuario  
- Arquitectura tecnológica  

---

# 2. Metodología de Desarrollo

El proyecto será gestionado utilizando la **metodología Scrum**.

Toda la planificación y seguimiento deberá realizarse en **Azure DevOps**, incluyendo:

- Backlog
- Épicas
- Features
- Historias de usuario

La documentación del proyecto deberá mantenerse en la **Wiki del repositorio**.

| Fase | Descripción |
|-----|-----|
| **Inception** | Definición del producto, problema, valor y planificación inicial. |
| **Sprint 1** | Diseño técnico, arquitectura del sistema y definición de integración con IA. |
| **Sprint 2 a la 10** | Implementación del prototipo funcional con avances verificables. |

---

# 3. Organización del Equipo

- Los grupos **no podrán modificarse durante el semestre**.  
- Todos los integrantes deben participar en:
  - Diseño
  - Desarrollo
  - Documentación
  - Presentación

---

# 4. Requisitos del Proyecto

## 4.1 Aplicación Móvil

La aplicación final debe ser una **aplicación móvil funcional** desarrollada en tecnologías como:

- Android
- iOS
- Flutter
- React Native

Debe incluir:

- Autenticación
- Persistencia de datos
- Interacción con APIs
- Interfaz usable

Al final debe ser una **aplicación que se pueda desplegar en un dispositivo Android**.

---

## 4.2 Integración de Inteligencia Artificial

La aplicación debe integrar **al menos un caso de uso de Inteligencia Artificial dentro del flujo principal del producto**.

Ejemplos:

- Recomendaciones personalizadas
- Análisis de texto
- Clasificación automática de contenido
- Reconocimiento de imágenes

---

# 5. Planificación del Proyecto en Azure DevOps

Cada equipo deberá **crear y gestionar el backlog del producto en Azure DevOps**.

Las historias de usuario deberán cumplir con los **principios INVEST**.

## Ejemplo de historia de usuario

**Como** usuario  
**quiero** recibir recomendaciones personalizadas  
**para** encontrar contenido relevante según mis preferencias.

Los **criterios de aceptación** deberán escribirse utilizando **lenguaje Gherkin**.

### Ejemplo
- Given el usuario ha iniciado sesión
- When solicita recomendaciones
- Then el sistema muestra contenido personalizado

---

# 6. Documentación en Wiki

Toda la documentación deberá estar en la **Wiki de Azure DevOps** e incluir:

- Descripción del producto
- Problema que resuelve
- Propuesta de valor
- Arquitectura del sistema
- Tecnologías utilizadas
- Casos de uso
- Integración con IA

---

# 7. Repositorios en GitHub

Cada equipo deberá **crear repositorios privados en GitHub** e invitar al profesor:

**rodrigogualtero92@gmail.com**

---

# 8. Diagramas de Arquitectura

El proyecto deberá incluir diagramas de arquitectura como:

- Diagrama general del sistema
- Diagramas de componentes
- Modelo **C4**

---

# 9. Escenarios de Calidad de Arquitectura

Cada equipo deberá definir **escenarios de calidad** que permitan justificar y defender la arquitectura propuesta.

Los escenarios deben describir:

- Estímulo
- Entorno
- Artefacto afectado
- Respuesta del sistema
- Medida de respuesta

| Atributo de Calidad | Escenario | Métrica |
|---|---|---|
| **Escalabilidad** | La aplicación debe soportar múltiples usuarios concurrentes solicitando recomendaciones. | Tiempo de respuesta menor a **2 segundos** |
| **Disponibilidad** | El sistema debe permanecer operativo incluso ante fallos de un servicio externo. | Disponibilidad mayor al **99%** |
| **Modificabilidad** | Debe ser posible integrar un nuevo modelo de IA sin afectar otros componentes. | Cambios limitados a módulos específicos |
| **Seguridad** | Los datos del usuario deben almacenarse y transmitirse de manera segura. | Uso de autenticación y comunicación cifrada |
| **Usabilidad** | El usuario debe poder completar la tarea principal en pocos pasos. | En máximo **3 interacciones principales** |

---

# 10. Entregable de la Próxima Semana

Cada equipo deberá entregar la **planificación completa del proyecto** incluyendo:

- Backlog en **Azure DevOps**
- **Épicas**
- **Features**
- **Historias de usuario con criterios Gherkin**
- **Diagramas de arquitectura**
- **Mockups**
- **Repositorios creados en GitHub**
- **Documentación en la Wiki**