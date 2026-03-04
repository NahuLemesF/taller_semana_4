# USER_STORIES_REFINEMENT.md

Informe comparativo que incluye las 3 Historias de Usuario analizadas: versión original, versión refinada por la Gema que diagnostica las historias de usuario y el cuadro de diferencias detectadas.

---

## US-01

### Historia de Usuario Original

**US-01 — Crear una asignación de ticket vía API**  
Como agente de soporte quiero crear una nueva asignación de ticket a través de la API REST para registrar formalmente que un ticket ha sido asignado con una prioridad específica.

**Criterios de Aceptación (Gherkin)**

```gherkin
@epic:api-rest-asignaciones @story:US-01 @priority:alta @risk:medio

Feature: Crear asignación de ticket vía API  
Como agente de soporte  
Quiero crear una nueva asignación de ticket  
Para registrar formalmente la asignación con prioridad  

Scenario: Creación exitosa de asignación con datos válidos  
Given el sistema está operativo y la base de datos accesible  
And no existe una asignación previa para el ticket "TK-100"  
When envío una petición POST a "/api/assignments/" con body {"ticket_id": "TK-100", "priority": "high"}  
Then el sistema responde con código de estado 201 Created  
And el cuerpo de respuesta contiene el campo "id" con un valor numérico  
And el campo "ticket_id" es "TK-100"  
And el campo "priority" es "high"  
And el campo "assigned_at" contiene una fecha ISO válida  

Scenario: Creación idempotente cuando ya existe asignación para el ticket  
Given existe una asignación para el ticket "TK-100" con prioridad "high"  
When envío una petición POST a "/api/assignments/" con body {"ticket_id": "TK-100", "priority": "medium"}  
Then el sistema responde con código de estado 201 Created  
And la asignación retornada mantiene la prioridad original "high"  
And no se crea un registro duplicado en la base de datos  

Scenario: Rechazo por prioridad inválida  
Given el sistema está operativo  
When envío una petición POST a "/api/assignments/" con body {"ticket_id": "TK-101", "priority": "critical"}  
Then el sistema responde con código de estado 400 Bad Request  
And el cuerpo contiene un mensaje de error indicando las prioridades válidas  

Scenario: Rechazo por ticket_id vacío  
Given el sistema está operativo  
When envío una petición POST a "/api/assignments/" con body {"ticket_id": "", "priority": "low"}  
Then el sistema responde con código de estado 400 Bad Request  
And el cuerpo contiene un mensaje de error indicando que "ticket_id" es requerido  
```

Notas  
Valor de negocio: Permite el registro formal de asignaciones, habilitando trazabilidad completa del ciclo de vida del ticket.  
Supuestos confirmados: Las prioridades válidas son high, medium, low y unassigned. La idempotencia se aplica por ticket_id.  
Dependencias: Requiere que el ticket-service publique IDs de ticket válidos.

Validación INVEST  
✅ INVEST — US-01: Crear asignación de ticket vía API  
I: ✅ Se puede implementar y desplegar sin depender de otras historias del backlog.  
N: ✅ Describe intención de negocio (registrar asignación); la implementación (endpoint, serializer) es negociable.  
V: ✅ Valor explícito: trazabilidad formal de asignaciones de tickets.  
E: ✅ Alcance claro: un endpoint POST con validaciones definidas y prioridades conocidas.  
S: ✅ Cabe en un sprint; es un solo endpoint con lógica de creación y validación.  
T: ✅ Criterios Gherkin observables con 4 escenarios verificables por Postman o pytest.

---

### Historia de Usuario Refinada por la Gema

**US-01 — Procesar asignación de ticket y prioridad**  
Como Sistema de Asignación (Assignment Service)  
Quiero procesar la creación de una asignación y establecer su prioridad  
Para garantizar que cada ticket sea atendido bajo las reglas de negocio y jerarquía de prioridades.

**Criterios de Aceptación (Gherkin)**

```gherkin
Feature: Procesar asignación de ticket

Background:  
Given el usuario está autenticado con rol "ADMIN" o es un servicio interno  
And el ticket "TK-100" está en estado "OPEN"

Scenario: Asignación exitosa con prioridad ascendente  
When envío una petición POST a "/api/assignments/" con:  
| campo | valor |  
| ticket_id | "TK-100" |  
| priority | "Medium" |  
| justification | "Asignación inicial" |  
Then el sistema responde con código 201 Created  
And el campo "priority" es "Medium"

Scenario: Rechazo por intento de bajar prioridad (Regla de Avance)  
Given el ticket "TK-100" ya tiene prioridad "High"  
When envío una petición POST a "/api/assignments/" con:  
| campo | valor |  
| priority | "Low" |  
Then el sistema responde con código 400 Bad Request  
And el mensaje indica que la prioridad solo puede avanzar

Scenario: Rechazo por ticket cerrado (Inmutabilidad)  
Given el ticket "TK-200" tiene estado "CLOSED"  
When envío una petición POST a "/api/assignments/" con body {"ticket_id": "TK-200", ...}  
Then el sistema responde con código 400 Bad Request  
And el mensaje indica que un ticket cerrado es inmutable

Scenario: Validación de longitud de justificación  
When envío una petición POST con una "justification" de 300 caracteres  
Then el sistema responde con código 400 Bad Request  
And el error indica que el máximo es 255 caracteres

> Nota: Las reglas de inmutabilidad y avance de prioridad fueron inferidas del contexto de negocio.
```

---

### Diferencias detectadas

| Historia original | Historia refinada | Diferencias detectadas |
|---|---|---|
| Actor: Agente. | Actor: Assignment Service / ADMIN. | El agente no asigna; es un proceso automático o administrativo. |
| Permite bajar prioridad en escenario idempotente. | Prohíbe el retroceso de prioridad. | Se alineó con la regla "La prioridad solo puede avanzar". |
| No menciona el estado del ticket. | Valida que el ticket no esté CLOSED. | Se incorporó la regla de inmutabilidad de tickets cerrados. |
| Sin campo de justificación. | Incluye justification (max 255 chars). | Se agregó el campo obligatorio según regla de negocio. |
| Prioridades: high, medium, low. | Prioridades: Low, Medium, High. | Ajuste de nomenclatura según el estándar del proyecto. |

---

## US-02

### Historia de Usuario Original

**US-02 — Original**  
Como supervisor del equipo de soporte quiero consultar la lista completa de asignaciones de tickets para tener visibilidad del estado actual de la distribución de trabajo.

**Criterios de Aceptación (Gherkin)**

```gherkin
@epic:api-rest-asignaciones @story:US-02 @priority:alta @risk:bajo

Feature: Consultar lista de asignaciones  
Como supervisor del equipo de soporte  
Quiero consultar todas las asignaciones  
Para tener visibilidad de la distribución de trabajo  

Scenario: Listado exitoso con asignaciones existentes  
Given existen 3 asignaciones registradas en el sistema  
When envío una petición GET a "/api/assignments/"  
Then el sistema responde con código de estado 200 OK  
And el cuerpo contiene un arreglo con 3 elementos  
And cada elemento tiene los campos "id", "ticket_id", "priority", "assigned_at" y "assigned_to"  

Scenario: Listado vacío cuando no hay asignaciones  
Given no existen asignaciones registradas en el sistema  
When envío una petición GET a "/api/assignments/"  
Then el sistema responde con código de estado 200 OK  
And el cuerpo contiene un arreglo vacío  

Scenario: Las asignaciones se ordenan por fecha más reciente primero  
Given existen asignaciones creadas en orden: "TK-001" (hace 2 horas), "TK-002" (hace 1 hora), "TK-003" (hace 5 minutos)  
When envío una petición GET a "/api/assignments/"  
Then el primer elemento del arreglo corresponde al ticket "TK-003"  
And el último elemento corresponde al ticket "TK-001"  
```

Notas  
Valor de negocio: Visibilidad operativa para supervisores sobre la carga de trabajo asignada.  
Supuestos confirmados: El ordenamiento es por assigned_at descendente (más reciente primero).  
Dependencias: Ninguna directa.

Validación INVEST  
✅ INVEST — US-02: Consultar todas las asignaciones  
I: ✅ Independiente; el GET no depende del POST para su implementación (puede usar fixtures).  
N: ✅ Describe intención de consulta; formato de respuesta y paginación son negociables.  
V: ✅ Visibilidad operativa para supervisores.  
E: ✅ Alcance mínimo: un endpoint GET que retorna lista serializada.  
S: ✅ Muy pequeña; lectura directa del repositorio.  
T: ✅ 3 escenarios verificables con datos de prueba controlados.

---

### Historia de Usuario Refinada por la Gema

**Título**  
US-02 — Consultar listado global de asignaciones (Vista Administrativa)

**Descripción**  
Como Administrador  
Quiero consultar la lista completa de asignaciones de tickets  
Para supervisar la distribución de trabajo entre los agentes y asegurar el cumplimiento de las prioridades.

**Criterios de Aceptación (Gherkin)**

```gherkin
Feature: Consultar lista de asignaciones vía API

Background:  
Given el sistema está operativo  
And el usuario se ha autenticado con rol "ADMIN" mediante cookie HttpOnly

Scenario: Listado exitoso con datos paginados  
Given existen 25 asignaciones registradas  
When envío una petición GET a "/api/assignments/?page=1"  
Then el sistema responde con código 200 OK  
And el cuerpo contiene un campo "results" con 20 elementos (tamaño de página)  
And cada elemento incluye "id", "ticket_id", "priority", "assigned_at", "assigned_to" y "justification"

Scenario: Ordenamiento cronológico descendente  
Given existen asignaciones: "A" (10:00 AM), "B" (10:30 AM), "C" (11:00 AM)  
When envío una petición GET a "/api/assignments/"  
Then el primer elemento en "results" es la asignación "C"  
And el último elemento en "results" es la asignación "A"

Scenario: Rechazo por falta de privilegios (Rol Usuario)  
Given el usuario se ha autenticado con rol "Usuario"  
When envío una petición GET a "/api/assignments/"  
Then el sistema responde con código 403 Forbidden  
And el mensaje indica que no tiene permisos para ver todas las asignaciones

Scenario: Verificación de headers de seguridad  
When envío una petición GET a "/api/assignments/"  
Then la respuesta debe incluir el header "X-Frame-Options" con valor "DENY"  
And el header "Content-Type" debe ser "application/json"

> Nota: Las validaciones de rol y headers fueron incorporadas basándose en las secciones 3 y 4 del Contexto de Negocio.
```

---

### Diferencias detectadas

| Historia original | Historia refinada | Diferencias detectadas |
|---|---|---|
| Actor: Supervisor. | Actor: Administrador (ADMIN). | Se ajustó al modelo de roles oficial del proyecto. |
| Retorno de lista simple. | Retorno paginado (results). | Mejora de performance y escalabilidad para microservicios. |
| Sin validación de permisos. | Validación de rol y JWT. | Se alineó con la restricción de que el usuario común no ve tickets ajenos. |
| Campos básicos. | Agregado campo justification. | Consistencia con la US-01 donde la justificación es obligatoria. |
| Sin contexto de seguridad. | Verificación de Headers HTTP. | Cumplimiento de la normativa de seguridad técnica del proyecto. |

---

## US-03

### Historia de Usuario Original

**US-03: Original**  
Como agente de soporte quiero consultar los detalles de una asignación específica por su ID para verificar la prioridad, el ticket asociado y el usuario asignado.

**Criterios de Aceptación (Gherkin)**

```gherkin
@epic:api-rest-asignaciones @story:US-03 @priority:media @risk:bajo

Feature: Consultar asignación por ID  
Como agente de soporte  
Quiero consultar una asignación específica  
Para verificar sus detalles  

Scenario: Consulta exitosa de asignación existente  
Given existe una asignación con ID 1 para el ticket "TK-200" con prioridad "medium"  
When envío una petición GET a "/api/assignments/1/"  
Then el sistema responde con código de estado 200 OK  
And el cuerpo contiene "ticket_id" igual a "TK-200"  
And el cuerpo contiene "priority" igual a "medium"  

Scenario: Consulta de asignación inexistente  
Given no existe una asignación con ID 999  
When envío una petición GET a "/api/assignments/999/"  
Then el sistema responde con código de estado 404 Not Found  

Scenario: Consulta incluye campo assigned_to cuando está asignado a un usuario  
Given existe una asignación con ID 2 asignada al usuario "agent-42"  
When envío una petición GET a "/api/assignments/2/"  
Then el sistema responde con código de estado 200 OK  
And el campo "assigned_to" es "agent-42"
```

Notas  
Valor de negocio: Permite verificación puntual de asignaciones por parte de agentes y supervisores.  
Supuestos confirmados: El endpoint usa el ID numérico interno de la asignación, no el ticket_id.  
Dependencias: Ninguna directa.

Validación INVEST  
✅ INVEST — US-03: Consultar asignación por ID  
I: ✅ Independiente; consulta por clave primaria sin dependencias funcionales.  
N: ✅ Intención de negocio clara; estructura de respuesta negociable.  
V: ✅ Permite a agentes verificar estado de asignaciones puntuales.  
E: ✅ Alcance mínimo: un endpoint GET con parámetro de ruta.  
S: ✅ Muy pequeña.  
T: ✅ 3 escenarios observables con respuestas HTTP verificables.

---

### Historia de Usuario Refinada por la Gema

**Título**  
US-03 — Consulta detallada de asignación por ID (Acceso Agente/Admin)

**Descripción**  
Como Agente de Soporte o Administrador  
Quiero consultar los detalles de una asignación específica mediante su ID  
Para verificar la prioridad, la justificación del cambio y el estado de la asignación de un ticket.

**Criterios de Aceptación (Gherkin)**

```gherkin
Feature: Consultar asignación específica por ID

Background:  
Given el sistema está operativo  
And existe una asignación con ID 10 para el ticket "TK-200" con prioridad "High"  
And la asignación tiene la justificación "Urgencia cliente VIP"

Scenario: Consulta exitosa por parte de un Administrador  
Given el usuario está autenticado con rol "ADMIN"  
When envío una petición GET a "/api/assignments/10/"  
Then el sistema responde con código 200 OK  
And el cuerpo contiene los campos:  
| campo | valor |  
| id | 10 |  
| ticket_id | "TK-200" |  
| priority | "High" |  
| justification | "Urgencia cliente VIP" |  
And la respuesta incluye los headers de seguridad nosniff y XSS Filter

Scenario: Consulta exitosa por parte del Agente asignado  
Given el usuario está autenticado como "agent-42" con rol "Agente"  
And la asignación 10 está vinculada a "agent-42"  
When envío una petición GET a "/api/assignments/10/"  
Then el sistema responde con código 200 OK  
And el campo "assigned_to" es "agent-42"

Scenario: Rechazo por intentar ver asignación ajena (Rol Agente)  
Given el usuario está autenticado como "agent-99" con rol "Agente"  
And la asignación 10 está vinculada a "agent-42"  
When envío una petición GET a "/api/assignments/10/"  
Then el sistema responde con código 403 Forbidden  
And el mensaje indica que no tiene permiso para ver esta asignación

Scenario: Error al buscar ID inexistente  
Given el usuario es "ADMIN"  
When envío una petición GET a "/api/assignments/9999/"  
Then el sistema responde con código 404 Not Found

> Nota: Se añadieron validaciones de propiedad de la asignación y el campo "justification" basado en las reglas de negocio.
```

---

### Diferencias detectadas

| Historia original | Historia refinada | Diferencias detectadas |
|---|---|---|
| Campos: ID, ticket_id, priority, assigned_to. | Campos: Se añade justification. | Coherencia con la regla de negocio de prioridad. |
| Seguridad: No especificada. | Seguridad: Validación de pertenencia y Rol. | Se evitó que un agente vea trabajo de otros sin ser ADMIN. |
| Criterios: Solo éxito/error 404. | Criterios: Incluye 403 Forbidden y headers. | Cumplimiento de normativas técnicas del proyecto. |
| Actor: Agente de soporte. | Actor: Agente o Administrador. | Clarificación de permisos según la jerarquía del sistema. |
