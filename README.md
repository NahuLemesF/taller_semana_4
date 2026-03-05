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
| Material de socializacion tecnica (diapositivas, PDF, diagramas, enlaces, codigo) | [`Presentacion/`](./Presentacion/) | Completado |
| Proyecto de automatizacion con patron Screenplay | [`screenplay-example/`](./screenplay-example/) | Completado |

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
4. [`Presentacion/`](./Presentacion/) — Diapositivas sobre el patron Screenplay
5. [`screenplay-example/`](./screenplay-example/) — Proyecto de automatizacion con Serenity BDD + Screenplay + Gradle

## Estructura actual del repositorio

```text
.
├── BUSINESS_CONTEXT.md
├── USER_STORIES_REFINEMENT.md
├── TEST_CASES_AI.md
├── README.md
├── Presentacion/
│   └── Patron Screenplay - Presentacion.pdf
└── screenplay-example/
    ├── build.gradle
    ├── gradlew
    ├── gradlew.bat
    ├── serenity.properties
    ├── README.md
    ├── docker/
    │   └── docker-compose.yml
    ├── gradle/
    │   └── wrapper/
    │       ├── gradle-wrapper.jar
    │       └── gradle-wrapper.properties
    └── src/
        ├── main/
        │   └── java/net/serenitybdd/demos/todos/
        │       ├── pageobjects/
        │       │   ├── model/
        │       │   │   ├── TodoStatus.java
        │       │   │   └── TodoStatusFilter.java
        │       │   ├── pages/
        │       │   │   ├── TodoListPage.java
        │       │   │   └── TodoMVCHomePage.java
        │       │   └── steps/
        │       │       └── TodoUserSteps.java
        │       └── screenplay/
        │           ├── actions/
        │           │   └── JSClick.java
        │           ├── model/
        │           │   ├── ApplicationInformation.java
        │           │   ├── TodoStatus.java
        │           │   └── TodoStatusFilter.java
        │           ├── questions/
        │           │   ├── Application.java
        │           │   ├── ClearCompletedItems.java
        │           │   ├── CurrentFilter.java
        │           │   ├── ElementAvailability.java
        │           │   ├── Placeholder.java
        │           │   ├── TheItemStatus.java
        │           │   └── TheItems.java
        │           ├── tasks/
        │           │   ├── AddATodoItem.java
        │           │   ├── AddTodoItems.java
        │           │   ├── Clear.java
        │           │   ├── CompleteItem.java
        │           │   ├── DeleteAllTheItems.java
        │           │   ├── DeleteAnItem.java
        │           │   ├── FilterItems.java
        │           │   ├── Start.java
        │           │   └── ToggleStatus.java
        │           └── user_interface/
        │               ├── TodoList.java
        │               ├── TodoListApp.java
        │               └── TodoListItem.java
        └── test/
            ├── java/net/serenitybdd/demos/todos/
            │   ├── cucumber/
            │   │   ├── CucumberTestSuite.java
            │   │   ├── MissingTodoItemsException.java
            │   │   └── steps/
            │   │       ├── TodoUserActionSteps.java
            │   │       └── TodoUserSteps.java
            │   ├── pageobjects/
            │   │   ├── accessing_the_application/
            │   │   │   └── LearnAboutTheApplication.java
            │   │   ├── completing_todos/
            │   │   │   ├── CompleteATodo.java
            │   │   │   └── ToggleAllTodos.java
            │   │   ├── maintain_my_todo_list/
            │   │   │   ├── ClearCompletedTodos.java
            │   │   │   ├── DeletingTodos.java
            │   │   │   └── FilteringTodos.java
            │   │   └── record_todos/
            │   │       └── AddNewTodos.java
            │   └── screenplay/
            │       ├── accessing_the_application/
            │       │   └── LearnAboutTheApplication.java
            │       ├── completing_todos/
            │       │   └── CompleteATodo.java
            │       ├── maintain_my_todo_list/
            │       │   ├── ClearCompletedTodoItems.java
            │       │   ├── DeletingTodoItems.java
            │       │   ├── FilteringTodoItems.java
            │       │   └── HandlingTodosBelongingToSeveralUsers.java
            │       └── record_todos/
            │           └── AddNewTodos.java
            └── resources/
                ├── cucumber.properties
                ├── junit-platform.properties
                ├── logback-test.xml
                ├── serenity.conf
                └── features/
                    └── cucumber/
                        ├── maintain_my_todo_list/
                        │   ├── completing_todos.feature
                        │   ├── deleting_todos.feature
                        │   └── filtering_todos.feature
                        └── record_todos/
                            └── add_new_items_to_the_todo_list.feature
```
