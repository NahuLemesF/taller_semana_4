# TEST_CASES_AI.md

## Introducción

Este documento presenta la matriz de casos de prueba generados por la **Gema B** (Asistente de IA para generación de casos de prueba) para el proyecto **SistemaTickets**, basándose en el contexto de negocio proporcionado y las tres historias de usuario seleccionadas:

- **US-01:** Procesar asignación de ticket (POST /api/assignments/)
- **US-02:** Consultar lista de asignaciones vía API (GET /api/assignments/)
- **US-03:** Consultar asignación específica por ID (GET /api/assignments/{id}/)

La Gema B utilizó técnicas del estándar **ISTQB** (International Software Testing Qualifications Board) para diseñar casos de prueba que cubren flujos básicos, reglas de negocio, valores límite, seguridad y excepciones.

## Refinamiento del Probador

Como parte del rigor analítico del equipo de QA, cada caso generado por la IA ha sido sometido a una **revisión crítica** considerando:
- Lógica de negocio específica del dominio
- Análisis de riesgos técnicos y operativos
- Cobertura de escenarios de concurrencia
- Validaciones de performance y escalabilidad
- Consideraciones de UX y mensajes de error

A continuación se presenta la matriz completa de ajustes realizados.

---

## Matriz de Casos de Prueba - US-01: Procesar Asignación de Ticket

| ID | Caso de Prueba generado por la Gema | Ajuste realizado por el probador | ¿Por qué se ajustó? |
|---|---|---|---|
| CP-001 | Asignación exitosa de prioridad por ADMIN con prioridad "Medium" | Asignación exitosa con validación de timestamp de "assigned_at" en formato ISO 8601 | La Gema no validó el formato de fecha/hora, crítico para auditorías y sincronización con otros servicios. El timestamp debe ser validado para evitar inconsistencias con RabbitMQ. |
| CP-002 | Asignación exitosa por servicio interno autenticado | Asignación exitosa por servicio interno con validación de trazabilidad (X-Request-ID header) | Se agregó validación del header X-Request-ID para mantener trazabilidad en arquitecturas de microservicios, esencial para debugging distribuido. |
| CP-003 | Rechazo por degradación de prioridad (High a Low) | Rechazo por degradación con validación de mensaje específico: "La prioridad solo puede avanzar: [Low->Medium->High]" | La Gema generó un mensaje genérico. Se especificó el mensaje exacto para mejorar la experiencia del usuario y facilitar el troubleshooting. |
| CP-004 | Rechazo por ticket en estado CLOSED | Rechazo por ticket CLOSED con validación de que no se persista el intento en logs de auditoría | Se añadió verificación de que los intentos fallidos no generen ruido en auditoría, considerando que tickets cerrados representan intentos posiblemente maliciosos. |
| CP-005 | Justificación con exactamente 255 caracteres | Justificación con 255 caracteres incluyendo caracteres especiales UTF-8 (ñ, á, é) | La Gema no consideró caracteres multibyte UTF-8. Se validó con caracteres especiales para evitar truncamientos en bases de datos mal configuradas. |
| CP-006 | Justificación excede 256 caracteres | Justificación de 256 caracteres con validación de respuesta antes de llegar a la base de datos (fail-fast) | Se especificó que la validación debe ocurrir en la capa de aplicación antes de consultar la BD, mejorando performance y reduciendo carga innecesaria. |
| CP-007 | Rechazo de asignación por usuario con rol "CLIENTE" | Rechazo con validación de que no se registre el intento en métricas de asignaciones fallidas | La Gema no distinguió entre errores operativos vs intentos no autorizados. Se ajustó para no contaminar métricas de negocio con eventos de seguridad. |
| CP-008 | Prevención de XSS en justificación con `<script>` | Prevención de XSS y validación de sanitización con múltiples vectores: `<img src=x onerror=alert(1)>`, `javascript:alert(1)`, y expresiones SQL | Se expandió para incluir más vectores de ataque (SQLi, Event Handlers) mencionados en las reglas de seguridad del contexto, asegurando cobertura completa. |
| CP-009 | Ticket ID inexistente devuelve 404 | Ticket ID inexistente con validación de tiempo de respuesta < 100ms para evitar timing attacks | Se agregó validación de performance consistente para prevenir que atacantes infieran existencia de tickets mediante análisis de latencia diferencial. |

---

## Matriz de Casos de Prueba - US-02: Consultar Lista de Asignaciones

| ID | Caso de Prueba generado por la Gema | Ajuste realizado por el probador | ¿Por qué se ajustó? |
|---|---|---|---|
| CP-010 | Obtención de página 1 con 20 elementos | Obtención de página 1 con validación de que el campo "next" apunte correctamente a page=2 | La Gema no validó los links de paginación HATEOAS. Se agregó verificación de navegabilidad para garantizar una UX fluida en el frontend. |
| CP-011 | Verificación de ordenamiento cronológico descendente | Verificación de ordenamiento con validación de que asignaciones del mismo segundo mantengan orden por ID descendente | Se añadió criterio de desempate por ID para garantizar resultados determinísticos en pruebas automatizadas y evitar flakiness. |
| CP-012 | Navegación a página 2 con 5 elementos restantes | Navegación a página 2 con validación de que el campo "previous" apunte a page=1 y "next" sea null | Se completó la validación de navegación bidireccional para verificar la estructura completa de paginación HATEOAS. |
| CP-013 | Consulta con base de datos vacía devuelve array vacío | Consulta con BD vacía y validación de código 200 (no 404), con contadores count=0 y next/previous=null | La Gema podría haber generado 404. Se clarificó que lista vacía es un estado válido (no error), importante para UX y manejo correcto en frontend. |
| CP-014 | Integridad de datos en el listado con campos obligatorios | Integridad de datos con validación adicional de tipos: `id` (integer), `assigned_at` (ISO 8601 string), `priority` (enum) | La Gema validó presencia pero no tipos de datos. Se agregó validación de schema para prevenir errores de serialización y cumplir con el contrato OpenAPI. |
| CP-015 | Consulta de página fuera de rango devuelve 404 | Consulta de página fuera de rango con validación de mensaje: "Página X no existe. Páginas disponibles: 1-Y" | Se mejoró la usabilidad del error indicando el rango válido, facilitando corrección por parte de clientes de la API. |
| CP-016 | Rechazo de usuario con rol estándar (403) | Rechazo de usuario estándar con validación de que la respuesta no filtre información (mensaje genérico "Acceso denegado") | Se ajustó para prevenir information disclosure. Mensajes detallados podrían revelar estructura interna del sistema a usuarios no autorizados. |
| CP-017 | Verificación de headers de seguridad (X-Frame-Options, X-Content-Type-Options) | Verificación de headers incluyendo Content-Security-Policy: "default-src 'self'" y Strict-Transport-Security | La Gema solo validó headers básicos. Se agregaron CSP y HSTS para cumplir con las políticas de seguridad técnica del contexto de negocio. |
| CP-018 | Rechazo sin cookie de autenticación (401) | Rechazo sin autenticación con validación de respuesta en < 50ms y sin consulta a base de datos | Se agregó requisito de fail-fast para prevenir ataques de enumeración y reducir carga en infraestructura ante intentos no autenticados masivos. |

---

## Matriz de Casos de Prueba - US-03: Consultar Asignación Específica por ID

| ID | Caso de Prueba generado por la Gema | Ajuste realizado por el probador | ¿Por qué se ajustó? |
|---|---|---|---|
| CP-019 | Consulta exitosa de asignación por Administrador | Consulta exitosa con validación de que el campo "assigned_to" incluya información enriquecida del agente (nombre, email) desde Users Service | La Gema devolvió solo el ID. Se enriqueció para verificar integración correcta con Users Service y mejorar usabilidad del frontend administrativo. |
| CP-020 | Consulta exitosa por Agente propietario | Consulta exitosa con validación de logs de auditoría que registren: user_id, timestamp, recurso_accedido | Se agregó verificación de logging para cumplir requisitos de compliance y auditoría de accesos, crítico en sistemas con datos sensibles. |
| CP-021 | Rechazo por IDOR - Agente intenta acceder a asignación ajena | Rechazo por IDOR con validación de rate limiting: máximo 5 intentos por minuto, luego bloqueo temporal 403 | La Gema no consideró intentos masivos de IDOR. Se agregó throttling para prevenir ataques de fuerza bruta para enumerar asignaciones ajenas. |
| CP-022 | Bloqueo de acceso por rol "Usuario Cliente" (403) | Bloqueo por rol con validación de que el intento no genere alertas de seguridad innecesarias (ruido operacional) | Se ajustó para diferenciar entre violaciones de seguridad críticas vs accesos denegados por diseño, optimizando la respuesta del equipo SOC. |
| CP-023 | Verificación de headers de seguridad en respuesta exitosa | Verificación de headers con validación adicional de ausencia de headers que filtren información: Server, X-Powered-By | La Gema validó headers presentes, pero no los ausentes. Se agregó verificación de que no se expongan detalles técnicos del stack (information disclosure). |
| CP-024 | Búsqueda de ID inexistente devuelve 404 | Búsqueda de ID inexistente con validación de mensaje amigable: "Asignación no encontrada" sin revelar si el ID alguna vez existió | Se mejoró el mensaje para prevenir information disclosure y ataques de enumeración temporal (saber si un ID fue eliminado vs nunca existió). |
| CP-025 | Validación de formato de ID inválido (abc-123) devuelve 400 | Validación de ID inválido incluyendo casos: negativos, 0, decimales (1.5), y caracteres especiales ($, %, &) | La Gema solo probó strings alfanuméricos. Se expandió para cubrir más casos de fuzzing y validar robustez del parser de rutas. |

---

## Resumen de Ajustes Realizados

### Estadísticas Generales

- **Total de casos generados por la Gema:** 25
- **Casos ajustados por el probador:** 25 (100%)
- **Ajustes por categoría:**
  - **Seguridad:** 10 ajustes (40%) - IDOR, XSS, Information Disclosure, Rate Limiting
  - **Performance:** 4 ajustes (16%) - Fail-fast, latencia consistente
  - **UX/Mensajes:** 5 ajustes (20%) - Mensajes específicos, guías de corrección
  - **Integración:** 3 ajustes (12%) - Trazabilidad, logs de auditoría, enriquecimiento de datos
  - **Robustez/Validaciones:** 3 ajustes (12%) - UTF-8, tipos de datos, casos de borde

### Justificación de la Estrategia de Refinamiento

1. **Análisis de Riesgos Técnicos:** Se priorizaron ajustes sobre vectores de ataque identificados en el contexto de negocio (XSS, SQLi, IDOR).

2. **Consideraciones Arquitectónicas:** Se validaron integraciones con microservicios (Users Service, RabbitMQ) que la IA no podía inferir sin contexto profundo.

3. **Compliance y Auditoría:** Se agregaron validaciones de logging y trazabilidad para cumplir con requisitos regulatorios implícitos en sistemas de tickets corporativos.

4. **Optimización de Performance:** Se incluyeron validaciones de fail-fast para reducir carga en infraestructura ante tráfico malicioso o errores de clientes.

5. **Experiencia de Usuario:** Se mejoraron mensajes de error para ser informativos sin comprometer seguridad, facilitando el troubleshooting de clientes de la API.

---

## Técnicas ISTQB Aplicadas en el Refinamiento

Durante el proceso de ajuste, el equipo aplicó las siguientes técnicas adicionales:

- **Security Testing:** Expansión de vectores de ataque (CP-008, CP-021)
- **Performance Testing:** Validaciones de latencia y fail-fast (CP-006, CP-009, CP-018)
- **Compliance Testing:** Verificación de auditoría y logging (CP-020, CP-004)
- **Usability Testing:** Mejora de mensajes de error (CP-003, CP-015, CP-024)
- **Integration Testing:** Validación de contratos entre microservicios (CP-002, CP-019)

---

## Conclusión

El trabajo colaborativo entre la **Gema B (IA)** y el **equipo de QA** resultó en una suite de pruebas robusta que cubre:
- ✅ Funcionalidad core del negocio
- ✅ Reglas de autorización y seguridad
- ✅ Casos de borde y excepciones
- ✅ Performance bajo condiciones adversas
- ✅ Experiencia de usuario en escenarios de error

La IA proporcionó una base sólida con cobertura técnica estándar, mientras que el refinamiento humano agregó la capa crítica de **contexto de negocio**, **análisis de riesgos** y **conocimiento arquitectónico** que solo la experiencia del equipo puede aportar.

**Este enfoque híbrido (IA + Human QA)** representa el estado del arte en automatización de pruebas moderna, maximizando eficiencia sin comprometer la calidad del análisis crítico.
