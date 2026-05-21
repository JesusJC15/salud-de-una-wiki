# SaludDeUna — Estructura de la Presentación Técnica

**Evento:** Sustentación Técnica IETI-2026-I — Grupo 1
**Fecha:** 21 de mayo de 2026, 11:00 a.m. – 1:00 p.m.
**Duración estimada por equipo:** 15–20 minutos de presentación + preguntas
**Plataforma sugerida:** Canva / Google Slides / Gamma / PowerPoint

---

## Diseño Visual Recomendado

### Paleta de colores

| Uso | Color | Hex |
|-----|-------|-----|
| Primario (fondo oscuro) | Azul medianoche | `#0F172A` |
| Acento principal | Azul eléctrico | `#3B82F6` |
| Acento salud | Verde menta | `#10B981` |
| Alerta / IA | Violeta | `#8B5CF6` |
| Texto | Blanco / Gris claro | `#F1F5F9` |
| Tarjetas / Fondos secundarios | Gris oscuro | `#1E293B` |

> Modo oscuro tecnológico. Transmite confianza, innovación y salud digital.

### Tipografías

- **Títulos:** Inter Bold o Space Grotesk Bold — moderna, tecnológica
- **Cuerpo:** Inter Regular — legible, limpia
- **Código / datos:** JetBrains Mono — evidencia técnica

### Elementos visuales

- Íconos lineales (Phosphor Icons o Heroicons)
- Tarjetas con bordes redondeados y fondo gris oscuro
- Diagramas Mermaid exportados como SVG
- Mockups de la app mobile (frames tipo teléfono)
- Badges de tecnología (NestJS, React Native, Gemini, etc.)
- QR proyectado en esquina derecha de la slide de demo y siempre visible en la última slide

---

## Índice de Diapositivas — 25 Slides

### Sección 0: Apertura (3 slides · ~2 min)

| # | Título | Lineamiento | Duración |
|---|--------|-------------|----------|
| 1 | Portada — SaludDeUna | Identidad del equipo | 30 s |
| 2 | Agenda de la sustentación | Orientación | 30 s |
| 3 | El problema en una frase | Contexto del problema (§2) | 1 min |

### Sección 1: Problema y Solución (4 slides · ~3 min)

| # | Título | Lineamiento | Duración |
|---|--------|-------------|----------|
| 4 | El caos de la consulta médica — Del lado del paciente | Contexto, usuarios (§2) | 45 s |
| 5 | El costo oculto — Del lado del médico | Contexto, usuarios (§2) | 45 s |
| 6 | SaludDeUna — La capa inteligente | Propuesta de valor (§2) | 45 s |
| 7 | ¿Qué NO es SaludDeUna? | Claridad de alcance | 45 s |

### Sección 2: El Sistema (3 slides · ~2.5 min)

| # | Título | Lineamiento | Duración |
|---|--------|-------------|----------|
| 8 | Alcance real del MVP | Casos de uso (§4) | 45 s |
| 9 | Tres roles, un sistema | Casos de uso (§4) | 45 s |
| 10 | Flujo del negocio — De principio a fin | Flujo del negocio (§4) | 1 min |

### Sección 3: Arquitectura (4 slides · ~3 min)

| # | Título | Lineamiento | Duración |
|---|--------|-------------|----------|
| 11 | Arquitectura general — Contexto C4 | Diagrama de arquitectura (§5) | 45 s |
| 12 | Arquitectura de contenedores C4 | Diagrama de arquitectura (§5) | 45 s |
| 13 | Backend por dentro — 17 módulos NestJS | Frontend/backend/APIs (§5) | 1 min |
| 14 | El Pipeline de Triage IA | IA y persistencia (§5) | 30 s |

### Sección 4: Calidad Técnica (4 slides · ~3 min)

| # | Título | Lineamiento | Duración |
|---|--------|-------------|----------|
| 15 | Criterios de Calidad — Resumen | Criterios de calidad (§3) | 30 s |
| 16 | Seguridad y Autenticación | Criterio: Seguridad (§3) | 45 s |
| 17 | Testing — 406+ tests, 93% cobertura | Criterio: Confiabilidad/Testeabilidad (§3) | 45 s |
| 18 | Observabilidad y Trazabilidad | Criterio: Observabilidad (§3) | 45 s |

### Sección 5: IA en el Proyecto (3 slides · ~2.5 min)

| # | Título | Lineamiento | Duración |
|---|--------|-------------|----------|
| 19 | IA en el producto — Triage asistido | IA en el sistema (§5) | 1 min |
| 20 | Guardrail Clínico — La IA no diagnostica | Claridad ética/técnica | 45 s |
| 21 | IA en el proceso de desarrollo | IA en desarrollo (§6 SWNT) | 45 s |

### Sección 6: Demo (2 slides · ~1 min de setup)

| # | Título | Lineamiento | Duración |
|---|--------|-------------|----------|
| 22 | Ruta del Demo en Vivo | Demo funcional (§7) | 45 s |
| 23 | Demo activo — APK + QR siempre visible | APK y QR obligatorio (§7) | Demo real |

### Sección 7: Cierre (2 slides · ~1 min)

| # | Título | Lineamiento | Duración |
|---|--------|-------------|----------|
| 24 | Resultados logrados | Funcionamiento real (§7) | 30 s |
| 25 | Conclusiones + ¿Preguntas? | Cierre técnico (§8) | 30 s |

---

## Distribución del tiempo recomendada

| Sección | Slides | Tiempo |
|---------|--------|--------|
| Apertura | 1–3 | 2 min |
| Problema y solución | 4–7 | 3 min |
| El sistema | 8–10 | 2.5 min |
| Arquitectura | 11–14 | 3 min |
| Calidad técnica | 15–18 | 3 min |
| IA | 19–21 | 2.5 min |
| Demo (vivo) | 22–23 | 5–7 min |
| Cierre | 24–25 | 1 min |
| **Total** | **25** | **~22 min** |

---

## Checklist de cobertura por lineamiento oficial

| Lineamiento (PDF) | Slide que lo cubre |
|-------------------|--------------------|
| §2 Contexto del problema (2 min) | 3, 4, 5 |
| §2 Usuarios afectados | 4, 5, 9 |
| §2 Propuesta de valor | 6, 7 |
| §3 Criterios de calidad (mín. 3) | 15, 16, 17, 18 |
| §4 Casos de uso principales | 8, 9 |
| §4 Flujo principal del negocio | 10 |
| §5 Diagrama de arquitectura | 11, 12, 13 |
| §5 Frontend, backend, APIs, IA, persistencia | 11, 12, 13, 14 |
| §6 IA en el proceso de desarrollo | 21 |
| §7 Demo funcional en tiempo real | 22, 23 |
| §7 APK funcional | 23 |
| §7 QR visible | 23 (+ corner badge en todas las slides del demo) |
| §8 Comunicación técnica clara | Todo el guion |
| §9 Evitar reducción de calificación | APK listo, arquitectura en slide 11–13, justificación técnica en 15–18 |
