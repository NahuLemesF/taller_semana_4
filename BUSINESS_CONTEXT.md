# BUSINESS_CONTEXT.md

# Contexto de Negocio — Sistema de Gestión de Tickets (SistemaTickets)

---

## 1. Descripción del Proyecto

**Nombre del Proyecto:**  
Sistema de Gestión de Tickets (SistemaTickets)

**Objetivo del Proyecto:**  
Permitir a los usuarios reportar incidencias o solicitudes mediante tickets de soporte, consultarlas y hacer seguimiento hasta su resolución. El sistema permite a los administradores supervisar y gestionar todos los tickets, incluyendo cambio de prioridad, cambio de estado y respuestas administrativas.

La solución está diseñada bajo una arquitectura de microservicios desacoplada y escalable, con comunicación asíncrona para tolerancia a fallos.

---

## 2. Flujos Críticos del Negocio

### Principales Flujos de Trabajo

- Registro de usuarios.
- Autenticación (login) de usuarios.
- Creación de tickets de soporte por parte del usuario.
- Consulta de tickets propios por parte del usuario.
- Consulta global de tickets por parte de administradores (ADMIN).
- Actualización del estado del ticket (OPEN → IN_PROGRESS → CLOSED).
- Cierre de tickets.
- Cambio de prioridad de tickets (solo ADMIN).
- **Consulta de asignaciones de tickets (listado y detalle) para auditoría/gestión (solo ADMIN).**
- Generación de notificaciones ante eventos relevantes del sistema.
- Comunicación asíncrona entre servicios vía mensajería.

### Módulos o Funcionalidades Críticas

- Gestión de tickets (CRUD + cambio de estado + cierre).
- Gestión de usuarios (registro, autenticación, administración).
- Gestión de prioridad de tickets (solo ADMIN).
- Gestión/consulta de asignaciones de tickets (solo ADMIN).
- Notificaciones del sistema.
- Integración entre microservicios mediante mensajería.

---

## 3. Reglas de Negocio y Restricciones

### Reglas de Negocio Relevantes

- Un ticket sigue una máquina de estados estricta:

```
OPEN → IN_PROGRESS → CLOSED
```

- No se permiten retrocesos en el estado del ticket.
- Un ticket en estado **CLOSED** es **inmutable**:
  - no permite cambios de estado,
  - no permite cambios de prioridad,
  - no permite nuevas respuestas.

- **Solo ADMIN** puede:
  - cambiar prioridad,
  - agregar respuestas administrativas,
  - ver tickets de otros usuarios,
  - consultar asignaciones globales.

- Un **Usuario** solo puede:
  - crear tickets,
  - consultar sus propios tickets,
  - ver el estado de sus tickets,
  - recibir notificaciones relacionadas a sus tickets.

- La prioridad solo puede **avanzar** (no se permite retroceder):

```
Unassigned → Low → Medium → High
```

- No es posible volver a **Unassigned** una vez que la prioridad avanzó.
- La **justificación** del cambio de prioridad tiene un máximo de **255 caracteres**.
- Las **respuestas** a tickets tienen un máximo de **2000 caracteres**.
- No se permite contenido **HTML** en títulos ni descripciones (prevención XSS).
- Los microservicios **no comparten base de datos** (Database per Service).

### Regulaciones o Normativas

- Autenticación **stateless** mediante JWT almacenado en **cookie HttpOnly** (mitigación XSS).
- Headers de seguridad HTTP obligatorios:

```
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
X-XSS-Protection
```

- Cookies seguras habilitadas en entornos de producción (CSRF y SESSION cuando aplique).

---

## 4. Perfiles de Usuario y Roles

### Perfiles o Roles de Usuario en el Sistema

| Rol | Descripción |
|---|---|
| Usuario / Cliente | Crea tickets, consulta sus propios tickets y recibe notificaciones relacionadas |
| Administrador (ADMIN) | Gestiona todos los tickets, agrega respuestas, cambia prioridad y estado, y consulta asignaciones globales |

### Permisos y Limitaciones de Cada Perfil

**Usuario / Cliente**
- Puede:
  - registrarse e iniciar sesión,
  - crear tickets,
  - consultar únicamente sus tickets,
  - ver el estado de sus tickets,
  - recibir notificaciones propias.
- No puede:
  - ver tickets de otros usuarios,
  - cambiar prioridad,
  - cambiar estados que no correspondan a su flujo permitido,
  - agregar respuestas administrativas.

**Administrador (ADMIN)**
- Puede:
  - ver todos los tickets,
  - cambiar prioridad de tickets (con justificación),
  - cambiar estado de tickets respetando la máquina de estados,
  - agregar respuestas administrativas,
  - consultar listados y detalles de asignaciones.
- No puede:
  - modificar tickets que estén en estado CLOSED (por regla de inmutabilidad).

---

## 5. Condiciones del Entorno Técnico

### Plataformas Soportadas

- Aplicación web (navegador).
- APIs REST consumidas desde el frontend y herramientas como curl/Postman.

### Tecnologías o Integraciones Clave

| Capa | Tecnología |
|---|---|
| Frontend | React 19 + TypeScript + Vite |
| Backend | Django 6 + Django REST Framework (Python 3.12) |
| Autenticación | JWT stateless via cookie HttpOnly |
| Mensajería | RabbitMQ (fanout exchange) |
| Base de datos | PostgreSQL 16 (una por microservicio) |
| Contenerización | Docker + Docker Compose |
| Microservicios | Ticket Service (8000), Notification Service (8001), Assignment Service (8002), Users Service (8003) |

---

## 6. Casos Especiales o Excepciones (Opcional)

- Si un ticket está **CLOSED**, cualquier intento de modificación debe ser rechazado como error de dominio.
- Si un usuario intenta enviar **HTML** en el título o descripción del ticket, el sistema debe rechazarlo en dos capas (serializer y validaciones de dominio) para prevenir XSS.
- Si el **Notification Service** o el **Assignment Service** están caídos, el **Ticket Service** debe seguir funcionando gracias al desacoplamiento por RabbitMQ (tolerancia a fallos).
