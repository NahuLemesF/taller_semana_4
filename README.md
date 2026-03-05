# Taller Semana 4: QA moderno, automatizacion robusta y refinamiento inteligente

### Entregable institucional de Ingenieria de Calidad (QA)

## Ficha institucional

| Campo | Detalle |
|---|---|
| Institucion | Sofka U |
| Taller | Semana 4 - QA moderno, automatizacion robusta y refinamiento inteligente |
| Proyecto base | [`SistemaTickets`](https://github.com/equipo-6-uruguay) |
| Fecha | 5 de marzo de 2026 |
| Tipo de entrega | Documentacion de contexto, refinamiento y diseno de pruebas asistido por IA |

## Integrantes

- Nahuel Lemes
- Cristian Davila
- Jean Pierre Villacís

## Objetivo del taller

Evolucionar la estrategia de calidad de un producto real para que los requisitos sean tecnicamente solidos y la ejecucion de pruebas sea escalable, integrando IA generativa y criterio experto de QA.

## Stack de referencia solicitado

- Gemini (Gemas IA)
- Java
- Selenium WebDriver
- Serenity BDD
- Patrones POM y Screenplay
- Gradle
- IDE con asistencia de IA (VS Code / IntelliJ + Copilot)

## Entregables institucionales

| Entregable | Archivo / Ubicacion | Estado |
|---|---|---|
| Contexto de negocio diligenciado | [`BUSINESS_CONTEXT.md`](./BUSINESS_CONTEXT.md) | Completado |
| Refinamiento de historias de usuario (antes vs despues) | [`USER_STORIES_REFINEMENT.md`](./USER_STORIES_REFINEMENT.md) | Completado |
| Matriz de casos de prueba IA + ajustes del probador | [`TEST_CASES_AI.md`](./TEST_CASES_AI.md) | Completado |
| Material de socializacion tecnica (diapositivas, PDF, diagramas, enlaces, codigo) | Carpeta `presentacion/` (sugerida) | Pendiente de anexar |

## Alcance funcional evaluado

- **US-01:** `POST /api/assignments/` (creacion de asignacion y prioridad)
- **US-02:** `GET /api/assignments/` (listado global de asignaciones)
- **US-03:** `GET /api/assignments/{id}/` (detalle por ID)

## Resultados consolidados

- Reglas de negocio formalizadas: inmutabilidad de tickets `CLOSED`, avance estricto de prioridad y control por roles.
- Refinamiento INVEST aplicado a las 3 historias con criterios mas precisos y verificables.
- **25 casos de prueba generados por IA** y **25 casos ajustados por el probador (100%)**.
- Cobertura reforzada en seguridad, performance, integracion, robustez y mensajes de error.

## Trazabilidad con las fases del taller

- **Fase 0 - Preparacion:** alineacion con contexto de negocio y stack.
- **Fase 1 - Socializacion tecnica:** base conceptual para automatizacion.
- **Fase 2 - Diseno inteligente:** uso de Gemas (A: INVEST, B: casos de prueba).
- **Fase 3 - Refinamiento humano:** validacion critica y consolidacion documental.

## Orden recomendado de revision

1. [`BUSINESS_CONTEXT.md`](./BUSINESS_CONTEXT.md)
2. [`USER_STORIES_REFINEMENT.md`](./USER_STORIES_REFINEMENT.md)
3. [`TEST_CASES_AI.md`](./TEST_CASES_AI.md)

## Estructura actual del repositorio

```text
.
├── BUSINESS_CONTEXT.md
├── USER_STORIES_REFINEMENT.md
├── TEST_CASES_AI.md
└── README.md
```
