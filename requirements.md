# Especificación de Requisitos - Agenda Salón MVP

## Información del Documento
- **Versión:** 1.0
- **Fecha:** 2026-02-04
- **Estado:** Borrador
- **Plataforma Objetivo:** Servidor Linux Ubuntu
- **Stack Tecnológico:** React 18+/TypeScript, FastAPI/Python 3.11+, PostgreSQL 15+, Infraestructura AWS

## 1. Introducción

### 1.1 Propósito
Este documento especifica los requisitos funcionales y no funcionales para Agenda Salón, un sistema web de gestión de citas para salones de belleza en Perú. Los requisitos siguen los patrones EARS (Easy Approach to Requirements Syntax) y las reglas de calidad INCOSE.

### 1.2 Alcance
El MVP permite a los clientes autogestionar sus reservas y a los dueños de salones gestionar servicios, personal y disponibilidad mediante una interfaz web.

### 1.3 Onboarding de Negocios
El registro e incorporación de negocios (salones) al sistema será ejecutado manualmente por el equipo de la aplicación. Durante este proceso de onboarding se recopilan:
- Datos del negocio (nombre, ubicación, contacto)
- Catálogo inicial de servicios (nombre, duración, precio)
- Personal del salón y sus asignaciones a servicios
- Horarios de atención

Este flujo manual está fuera del alcance de la implementación del MVP y no requiere interfaces de autoregistro para negocios.

### 1.4 Patrones EARS Utilizados
- **Ubicuo:** El `<sistema>` deberá `<capacidad>`
- **Dirigido por eventos:** CUANDO `<disparador>`, el `<sistema>` deberá `<capacidad>`
- **Dirigido por estado:** MIENTRAS `<estado>`, el `<sistema>` deberá `<capacidad>`
- **Opcional:** DONDE `<condición>`, el `<sistema>` deberá `<capacidad>`
- **No deseado:** SI `<condición no deseada>`, ENTONCES el `<sistema>` deberá `<capacidad>`

---

## 2. Requisitos de Frontend

### 2.1 Autenticación y Gestión de Usuarios

#### RF-FE-001: Registro de Usuario
El sistema deberá proporcionar un formulario de registro que acepte email, contraseña, nombre completo y número de teléfono.

#### RF-FE-002: Validación de Formato de Email
CUANDO un usuario ingrese un email durante el registro, el sistema deberá validar que siga el formato RFC 5322.

#### RF-FE-003: Validación de Fortaleza de Contraseña
CUANDO un usuario cree una contraseña, el sistema deberá validar que contenga al menos 8 caracteres, una letra mayúscula, una letra minúscula y un número.

#### RF-FE-004: Autenticación de Login
El sistema deberá proporcionar un formulario de login que acepte credenciales de email y contraseña.

#### RF-FE-005: Gestión de Sesión
CUANDO un usuario se autentique exitosamente, el sistema deberá almacenar el token JWT en memoria usando gestión de estado Zustand.

#### RF-FE-006: Persistencia de Sesión
CUANDO un usuario refresque el navegador, el sistema deberá mantener la sesión autenticada usando cookies seguras HTTP-only.

#### RF-FE-007: Funcionalidad de Logout
El sistema deberá proporcionar una acción de logout que limpie el token de autenticación y redirija a la página de login.

#### RF-FE-008: Rutas Protegidas
CUANDO un usuario no autenticado intente acceder a una ruta protegida, el sistema deberá redirigirlo a la página de login.

### 2.2 Visualización del Catálogo de Servicios

#### RF-FE-009: Listado de Servicios
El sistema deberá mostrar todos los servicios activos con nombre, duración y precio en la página de inicio.

#### RF-FE-010: Detalles de Servicio
CUANDO un cliente seleccione un servicio, el sistema deberá mostrar sus detalles completos incluyendo nombre, duración (en minutos) y precio (en moneda PEN).

#### RF-FE-011: Diseño de Tarjeta de Servicio
El sistema deberá renderizar cada servicio como una tarjeta conteniendo nombre del servicio, duración, precio y un botón de acción "Reservar".

### 2.3 Calendario de Disponibilidad

#### RF-FE-012: Filtro de Período de Tiempo
El sistema deberá proporcionar opciones de filtro para ver disponibilidad: 1 semana, 2 semanas o 1 mes.

#### RF-FE-013: Renderizado de Calendario
CUANDO un cliente seleccione un servicio y período de tiempo, el sistema deberá mostrar un calendario con las fechas disponibles.

#### RF-FE-014: Indicación de Disponibilidad de Fecha
El sistema deberá distinguir visualmente las fechas disponibles de las no disponibles usando colores o estilos distintos.

#### RF-FE-015: Fechas Pasadas Deshabilitadas
El sistema deberá deshabilitar la selección de fechas anteriores a la fecha actual.

### 2.4 Selección de Personal

#### RF-FE-016: Visualización de Miembros del Personal
CUANDO un cliente seleccione una fecha en el calendario, el sistema deberá mostrar los miembros del personal disponibles para el servicio seleccionado en esa fecha.

#### RF-FE-017: Selección de Personal Antes de Horarios
El sistema deberá requerir que el cliente seleccione un miembro del personal (o elija "Cualquier profesional") antes de mostrar los espacios de tiempo disponibles.

#### RF-FE-018: Opción de Cualquier Profesional
El sistema deberá proporcionar una opción "Cualquier profesional disponible" que muestre todos los espacios de tiempo donde al menos un miembro del personal calificado esté disponible.

#### RF-FE-019: Indicador de Asignación Automática
DONDE el cliente seleccione "Cualquier profesional disponible", el sistema deberá mostrar un mensaje indicando que se asignará automáticamente un miembro del personal disponible al confirmar.

### 2.5 Selección de Espacio de Tiempo

#### RF-FE-020: Agrupación de Espacios de Tiempo
El sistema deberá agrupar los espacios de tiempo disponibles en tres categorías: mañana (06:00-11:59), mediodía (12:00-15:59) y tarde (16:00-20:00).

#### RF-FE-021: Formato de Visualización de Espacio
El sistema deberá mostrar cada espacio de tiempo mostrando hora de inicio en formato de 24 horas y hora de fin calculada desde la duración del servicio.

#### RF-FE-022: Actualización de Espacios en Tiempo Real
CUANDO los datos de disponibilidad cambien, el sistema deberá actualizar los espacios de tiempo mostrados dentro de 3 segundos usando sondeo de TanStack Query.

#### RF-FE-023: Navegación de Semanas
DONDE el período de tiempo seleccionado exceda una semana, el sistema deberá proporcionar controles de navegación para moverse entre semanas.

#### RF-FE-024: Espacios No Disponibles Deshabilitados
El sistema deberá deshabilitar visualmente y prevenir la selección de espacios de tiempo no disponibles.

### 2.6 Confirmación de Reserva

#### RF-FE-025: Formulario de Información de Contacto
CUANDO un cliente seleccione un espacio de tiempo, el sistema deberá mostrar un formulario solicitando confirmación de nombre, número de teléfono y email.

#### RF-FE-026: Datos de Contacto Pre-llenados
DONDE el usuario esté autenticado, el sistema deberá pre-llenar el formulario de información de contacto con sus datos de perfil.

#### RF-FE-027: Visualización de Resumen de Reserva
El sistema deberá mostrar un resumen mostrando servicio seleccionado, duración, fecha, hora y miembro del personal asignado o seleccionado.

#### RF-FE-028: Acción de Confirmación de Reserva
El sistema deberá proporcionar un botón "Confirmar Reserva" que envíe la solicitud de cita al backend.

#### RF-FE-029: Retroalimentación de Éxito
CUANDO una reserva sea creada exitosamente, el sistema deberá mostrar un mensaje de éxito con el número de confirmación de la cita.

#### RF-FE-030: Retroalimentación de Error
SI una reserva falla, ENTONCES el sistema deberá mostrar un mensaje de error específico indicando la razón del fallo (ej. "Espacio ya no disponible", "Error de red").

#### RF-FE-031: Envío de Reserva Idempotente
El sistema deberá generar una clave de idempotencia única por intento de reserva e incluirla en el encabezado de la solicitud de reserva.

### 2.7 Gestión de Citas

#### RF-FE-032: Vista de Lista de Citas
El sistema deberá mostrar una lista de las citas del usuario autenticado mostrando fecha, hora, nombre del servicio y miembro del personal.

#### RF-FE-033: Filtrado de Citas
El sistema deberá permitir filtrar citas por estado: próximas, pasadas o canceladas.

#### RF-FE-034: Vista de Detalles de Cita
CUANDO un cliente seleccione una cita, el sistema deberá mostrar detalles completos incluyendo servicio, fecha, hora, duración, miembro del personal y estado.

#### RF-FE-035: Acción de Cancelar Cita
DONDE el estado de una cita sea "confirmada" o "pendiente", el sistema deberá proporcionar un botón "Cancelar".

#### RF-FE-036: Acción de Editar Cita
DONDE el estado de una cita sea "confirmada" o "pendiente", el sistema deberá proporcionar un botón "Editar".

#### RF-FE-037: Diálogo de Confirmación de Cancelación
CUANDO un cliente inicie la cancelación, el sistema deberá mostrar un diálogo de confirmación requiriendo confirmación explícita.

#### RF-FE-038: Mensaje de Cancelación Tardía
DONDE la cancelación se realice con menos de 2 horas de anticipación a la cita, el sistema deberá mostrar un mensaje adicional indicando que el cliente no está cumpliendo con su compromiso de cancelación anticipada.

#### RF-FE-039: Flujo de Editar Cita
CUANDO un cliente inicie la edición, el sistema deberá navegar al calendario de disponibilidad con el servicio de la cita actual pre-seleccionado.

#### RF-FE-040: Indicación de Estado de Edición
MIENTRAS se edita una cita, el sistema deberá mostrar un banner indicando que el espacio de la cita original se mantiene pendiente de confirmación.

### 2.8 Panel de Control del Dueño del Negocio (Interfaz Admin Futura)

#### RF-FE-041: Interfaz de Gestión de Servicios
El sistema deberá proporcionar formularios para crear, editar y desactivar servicios con campos para nombre, duración (minutos), precio (PEN) y personal asignado.

#### RF-FE-042: Configuración de Horarios de Negocio
El sistema deberá proporcionar una interfaz para establecer horarios de apertura y cierre para cada día de la semana.

#### RF-FE-043: Gestión de Fechas de Excepción
El sistema deberá permitir definir fechas específicas como días no laborables (feriados, vacaciones) con selector de fecha y campo de razón.

#### RF-FE-044: Vista de Calendario de Citas
El sistema deberá mostrar todas las citas en una vista de calendario con opciones de visualización diaria, semanal y quincenal.

#### RF-FE-045: Filtrado de Citas por Fecha
El sistema deberá proporcionar opciones de filtro: hoy, mañana, esta semana, próximas dos semanas.

### 2.9 Validación de Formularios (React Hook Form + Zod)

#### RF-FE-046: Validación del Lado del Cliente
El sistema deberá validar todas las entradas de formularios antes del envío usando esquemas Zod.

#### RF-FE-047: Retroalimentación de Validación en Tiempo Real
CUANDO un usuario complete un campo de formulario, el sistema deberá mostrar errores de validación dentro de 500ms.

#### RF-FE-048: Validación de Campos Requeridos
El sistema deberá marcar campos requeridos con un asterisco (*) y prevenir el envío si algún campo requerido está vacío.

### 2.10 Estados de Carga y Manejo de Errores

#### RF-FE-049: Indicadores de Carga
MIENTRAS se están obteniendo datos, el sistema deberá mostrar un spinner de carga o pantalla esqueleto.

#### RF-FE-050: Manejo de Errores de Red
SI una solicitud de red falla, ENTONCES el sistema deberá mostrar un mensaje de error con una opción "Reintentar".

#### RF-FE-051: Indicación de Obsolescencia de Datos
CUANDO los datos en caché excedan 30 segundos de antigüedad, el sistema deberá activar una actualización en segundo plano usando TanStack Query.

---

## 3. Requisitos de Backend

### 3.1 Autenticación y Autorización

#### RF-BE-001: Endpoint de Registro de Usuario
El sistema deberá proporcionar un endpoint POST `/api/v1/auth/registrar` que acepte email, contrasena, nombre_completo y telefono.

#### RF-BE-002: Validación de Unicidad de Email
CUANDO se procese el registro, el sistema deberá rechazar solicitudes si el email ya existe en la base de datos.

#### RF-BE-003: Hashing de Contraseña
El sistema deberá hashear contraseñas usando bcrypt con un factor de trabajo de 12 antes de almacenar en la base de datos.

#### RF-BE-004: Generación de Token JWT
CUANDO la autenticación tenga éxito, el sistema deberá generar un token JWT con claims user_id, email y role, expirando después de 24 horas.

#### RF-BE-005: Endpoint de Login
El sistema deberá proporcionar un endpoint POST `/api/v1/auth/login` que acepte credenciales de email y contrasena.

#### RF-BE-006: Verificación de Contraseña
CUANDO se procese el login, el sistema deberá verificar la contraseña usando comparación bcrypt.

#### RF-BE-007: Middleware de Validación de Token
El sistema deberá validar tokens JWT en todos los endpoints protegidos y rechazar solicitudes con tokens inválidos o expirados con HTTP 401.

#### RF-BE-008: Control de Acceso Basado en Roles
El sistema deberá aplicar permisos basados en roles: rol "cliente" para operaciones de reserva, rol "dueno_negocio" para operaciones de gestión.

### 3.2 Gestión de Servicios

#### RF-BE-009: Endpoint de Crear Servicio
El sistema deberá proporcionar un endpoint POST `/api/v1/servicios` aceptando nombre, duracion_minutos, precio_pen y personal_ids (solo dueno_negocio).

#### RF-BE-010: Validación de Servicio
CUANDO se cree un servicio, el sistema deberá validar que duracion_minutos esté entre 15 y 480, y precio_pen sea mayor que 0.

#### RF-BE-011: Endpoint de Listar Servicios
El sistema deberá proporcionar un endpoint GET `/api/v1/servicios` retornando todos los servicios activos con soporte de paginación.

#### RF-BE-012: Endpoint de Actualizar Servicio
El sistema deberá proporcionar un endpoint PUT `/api/v1/servicios/{servicio_id}` para actualizar atributos de servicio (solo dueno_negocio).

#### RF-BE-013: Endpoint de Desactivar Servicio
El sistema deberá proporcionar un endpoint DELETE `/api/v1/servicios/{servicio_id}` que establezca el estado del servicio como inactivo en lugar de eliminar el registro.

#### RF-BE-014: Asignación Servicio-Personal
El sistema deberá mantener relaciones muchos-a-muchos entre servicios y miembros del personal.

### 3.3 Cálculo de Disponibilidad

#### RF-BE-015: Endpoint de Consulta de Disponibilidad
El sistema deberá proporcionar un endpoint GET `/api/v1/disponibilidad` aceptando parámetros servicio_id, fecha_inicio, fecha_fin y personal_id opcional.

#### RF-BE-015A: Limpieza Inline de Ediciones Expiradas
CUANDO se procese una consulta de disponibilidad, el sistema deberá primero revertir automáticamente todas las citas en estado `pending_update` con más de 30 minutos de antigüedad a estado `confirmada` antes de calcular la disponibilidad.

#### RF-BE-016: Filtrado de Horarios de Negocio
CUANDO se calcule la disponibilidad, el sistema deberá retornar solo espacios dentro de los horarios de negocio configurados para las fechas solicitadas.

#### RF-BE-017: Filtrado de Fechas de Excepción
CUANDO se calcule la disponibilidad, el sistema deberá excluir fechas marcadas como días no laborables (feriados, vacaciones).

#### RF-BE-018: Exclusión de Citas Existentes
CUANDO se calcule la disponibilidad, el sistema deberá excluir espacios de tiempo que se solapen con citas confirmadas existentes para el mismo miembro del personal.

#### RF-BE-019: Consideración de Duración del Servicio
El sistema deberá retornar solo espacios de tiempo donde la duración del servicio quepa completamente dentro de los horarios de negocio (ej. no servicio de 90 minutos comenzando a las 19:30 si el cierre es a las 20:00).

#### RF-BE-020: Intervalo de Generación de Espacios
El sistema deberá generar espacios de tiempo en intervalos de 30 minutos alineados a la hora (ej. 09:00, 09:30, 10:00).

#### RF-BE-021: Disponibilidad Multi-personal
DONDE no se proporcione staff_id, el sistema deberá retornar espacios de tiempo disponibles para cualquier miembro del personal calificado para realizar el servicio solicitado.

#### RF-BE-022: Soporte de Citas Concurrentes
El sistema deberá permitir que diferentes miembros del personal tengan citas con horas de inicio y fin exactamente solapadas.

### 3.4 Reserva de Citas

#### RF-BE-023: Endpoint de Crear Cita
El sistema deberá proporcionar un endpoint POST `/api/v1/citas` aceptando servicio_id, personal_id (opcional), fecha_hora_cita, cliente_nombre, cliente_telefono, cliente_email.

#### RF-BE-024: Validación de Clave de Idempotencia
CUANDO se procese una solicitud de reserva, el sistema deberá verificar el encabezado `Idempotency-Key` y rechazar envíos duplicados con HTTP 409 si la clave fue usada dentro de las últimas 24 horas.

#### RF-BE-025: Bloqueo Optimista para Reserva de Espacio
CUANDO se cree una cita, el sistema deberá usar bloqueo a nivel de fila de base de datos (SELECT FOR UPDATE) para prevenir condiciones de carrera.

#### RF-BE-026: Re-validación de Disponibilidad de Espacio
CUANDO se procese una reserva, el sistema deberá re-validar que el espacio solicitado aún esté disponible antes de confirmar.

#### RF-BE-027: Asignación Automática de Personal
DONDE personal_id sea null en la solicitud de reserva, el sistema deberá asignar un miembro del personal disponible calificado para el servicio.

#### RF-BE-028: Creación de Cita
CUANDO una cita sea creada exitosamente, el sistema deberá retornar HTTP 201 con los detalles de la cita incluyendo personal asignado, número de confirmación y estado "pendiente".

#### RF-BE-029: Respuesta de Fallo de Reserva
SI el espacio ya no está disponible o falla la validación, ENTONCES el sistema deberá retornar HTTP 409 con detalles del error.

#### RF-BE-030: Creación de Registro de Idempotencia
CUANDO se procese una reserva válida, el sistema deberá crear un registro de idempotencia almacenando la clave, hash de solicitud y respuesta por 24 horas.

### 3.5 Gestión de Citas

#### RF-BE-031: Endpoint de Listar Citas de Usuario
El sistema deberá proporcionar un endpoint GET `/api/v1/citas/me` retornando citas para el usuario autenticado.

#### RF-BE-032: Filtrado de Estado de Citas
El sistema deberá soportar filtrado de citas por estado: pendiente, confirmada, cancelada, completada.

#### RF-BE-033: Endpoint de Obtener Detalles de Cita
El sistema deberá proporcionar un endpoint GET `/api/v1/citas/{cita_id}` retornando información completa de la cita.

#### RF-BE-034: Endpoint de Cancelar Cita
El sistema deberá proporcionar un endpoint DELETE `/api/v1/citas/{cita_id}` que actualice el estado de la cita a "cancelada".

#### RF-BE-035: Autorización de Cancelación
CUANDO se procese la cancelación, el sistema deberá verificar que el usuario solicitante posea la cita o tenga rol dueno_negocio.

#### RF-BE-036: Validación de Tiempo de Cancelación
El sistema deberá permitir cancelación solo si la fecha y hora de la cita es al menos 2 horas en el futuro.

#### RF-BE-037: Endpoint de Editar Cita
El sistema deberá proporcionar un endpoint PUT `/api/v1/citas/{cita_id}` aceptando nueva fecha_hora_cita y personal_id opcional.

#### RF-BE-038: Mantenimiento de Espacio en Edición
CUANDO se procese una solicitud de edición, el sistema deberá mantener la cita original en estado "pendiente_actualizacion" hasta que el nuevo espacio sea confirmado.

#### RF-BE-039: Confirmación de Edición
CUANDO se confirme un nuevo espacio para edición, el sistema deberá actualizar el registro de la cita y cambiar el estado de "pendiente_actualizacion" a "confirmada".

#### RF-BE-040: Manejo de Cancelación de Edición
SI la operación de edición es cancelada, ENTONCES el sistema deberá revertir la cita al estado "confirmada" con la fecha y hora original.

### 3.6 Configuración de Negocio

#### RF-BE-041: Endpoint de Establecer Horarios de Negocio
El sistema deberá proporcionar un endpoint POST `/api/v1/negocio/horarios` aceptando dia_semana (0-6), hora_apertura, hora_cierre (solo dueno_negocio).

#### RF-BE-042: Validación de Horarios de Negocio
CUANDO se establezcan horarios de negocio, el sistema deberá validar que hora_apertura sea antes que hora_cierre y ambos estén en formato HH:MM.

#### RF-BE-043: Endpoint de Crear Fecha de Excepción
El sistema deberá proporcionar un endpoint POST `/api/v1/negocio/excepciones` aceptando fecha_excepcion y motivo (solo dueno_negocio).

#### RF-BE-044: Unicidad de Fecha de Excepción
El sistema deberá aplicar restricción de unicidad en fecha_excepcion para prevenir entradas duplicadas de días no laborables.

#### RF-BE-045: Endpoint de Listar Horarios de Negocio
El sistema deberá proporcionar un endpoint GET `/api/v1/negocio/horarios` retornando horarios configurados para todos los días de la semana.

#### RF-BE-046: Endpoint de Listar Fechas de Excepción
El sistema deberá proporcionar un endpoint GET `/api/v1/negocio/excepciones` retornando todas las fechas de excepción futuras ordenadas por fecha.

### 3.7 Gestión de Personal

#### RF-BE-047: Endpoint de Crear Personal
El sistema deberá proporcionar un endpoint POST `/api/v1/personal` aceptando nombre y email (solo dueno_negocio).

#### RF-BE-048: Endpoint de Listar Personal
El sistema deberá proporcionar un endpoint GET `/api/v1/personal` retornando todos los miembros del personal activos.

#### RF-BE-049: Endpoint de Actualizar Personal
El sistema deberá proporcionar un endpoint PUT `/api/v1/personal/{personal_id}` para actualizar información del personal (solo dueno_negocio).

#### RF-BE-050: Endpoint de Desactivar Personal
El sistema deberá proporcionar un endpoint DELETE `/api/v1/personal/{personal_id}` estableciendo el estado del personal como inactivo (solo dueno_negocio).

### 3.8 Validación de Datos (Pydantic)

#### RF-BE-051: Validación de Esquema de Solicitud
El sistema deberá validar todos los cuerpos de solicitud entrantes usando modelos Pydantic v2 antes del procesamiento.

#### RF-BE-052: Coerción de Tipos
El sistema deberá intentar coerción de tipos para tipos compatibles (ej. string a integer) y rechazar tipos incompatibles con HTTP 422.

#### RF-BE-053: Respuesta de Error de Validación
CUANDO falle la validación, el sistema deberá retornar HTTP 422 con mensajes de error detallados a nivel de campo en formato JSON.

### 3.9 Manejo de Errores

#### RF-BE-054: Respuesta de Error Estructurada
El sistema deberá retornar errores en formato JSON consistente: `{"error": "codigo_error", "message": "descripción legible", "details": {}}`.

#### RF-BE-055: Manejo de Errores de Base de Datos
SI falla una operación de base de datos, ENTONCES el sistema deberá registrar el error con stack trace y retornar HTTP 500 con un mensaje de error genérico.

#### RF-BE-056: Rollback de Transacción
CUANDO ocurra un error de base de datos durante una transacción, el sistema deberá hacer rollback automático de todos los cambios.

### 3.10 Rendimiento

#### RF-BE-057: Operaciones de Base de Datos Asíncronas
El sistema deberá ejecutar todas las operaciones de base de datos usando la API async de SQLAlchemy para soportar solicitudes concurrentes.

#### RF-BE-058: Pool de Conexiones de Base de Datos
El sistema deberá mantener un pool de conexiones con mínimo 5 y máximo 20 conexiones.

#### RF-BE-059: Optimización de Consultas
El sistema deberá usar carga eager (selectinload/joinedload) para entidades relacionadas para prevenir problemas de consultas N+1.

---

## 4. Requisitos de Base de Datos (Modelo de Datos)

### 4.1 Tabla Usuario (usuarios)

#### RF-DB-001: Esquema de Usuario
El sistema deberá mantener una tabla usuarios con columnas: id (UUID, PK), email (VARCHAR(255), UNIQUE, NOT NULL), contrasena_hash (VARCHAR(255), NOT NULL), nombre_completo (VARCHAR(255), NOT NULL), telefono (VARCHAR(20), NOT NULL), rol (ENUM: cliente, dueno_negocio, NOT NULL), creado_en (TIMESTAMP, NOT NULL), actualizado_en (TIMESTAMP, NOT NULL), activo (BOOLEAN, DEFAULT TRUE).

#### RF-DB-002: Índice de Email
El sistema deberá crear un índice único en la columna email para búsqueda rápida.

#### RF-DB-003: Campos de Auditoría de Usuario
El sistema deberá poblar automáticamente creado_en en inserción y actualizado_en en actualización usando triggers de base de datos o eventos ORM.

### 4.2 Tabla Personal (personal)

#### RF-DB-004: Esquema de Personal
El sistema deberá mantener una tabla personal con columnas: id (UUID, PK), nombre (VARCHAR(255), NOT NULL), email (VARCHAR(255), UNIQUE, NOT NULL), activo (BOOLEAN, DEFAULT TRUE), creado_en (TIMESTAMP, NOT NULL), actualizado_en (TIMESTAMP, NOT NULL).

#### RF-DB-005: Unicidad de Email de Personal
El sistema deberá aplicar restricción única en personal.email para prevenir entradas duplicadas.

### 4.3 Tabla Servicio (servicios)

#### RF-DB-006: Esquema de Servicio
El sistema deberá mantener una tabla servicios con columnas: id (UUID, PK), nombre (VARCHAR(255), NOT NULL), duracion_minutos (INTEGER, NOT NULL, CHECK >= 15 AND <= 480), precio_pen (DECIMAL(10,2), NOT NULL, CHECK > 0), activo (BOOLEAN, DEFAULT TRUE), creado_en (TIMESTAMP, NOT NULL), actualizado_en (TIMESTAMP, NOT NULL).

#### RF-DB-007: Restricción de Duración de Servicio
El sistema deberá aplicar restricción CHECK asegurando que duracion_minutos esté entre 15 y 480.

#### RF-DB-008: Restricción de Precio de Servicio
El sistema deberá aplicar restricción CHECK asegurando que precio_pen sea mayor que 0.

### 4.4 Tabla de Asignación Servicio-Personal (servicio_personal)

#### RF-DB-009: Esquema Servicio-Personal
El sistema deberá mantener una tabla de unión servicio_personal con columnas: servicio_id (UUID, FK a servicios.id, NOT NULL), personal_id (UUID, FK a personal.id, NOT NULL), PRIMARY KEY (servicio_id, personal_id).

#### RF-DB-010: Claves Foráneas Servicio-Personal
El sistema deberá definir restricciones de clave foránea con ON DELETE CASCADE para remover asignaciones cuando se elimine servicio o personal.

### 4.5 Tabla Cita (citas)

#### RF-DB-011: Esquema de Cita
El sistema deberá mantener una tabla citas con columnas: id (UUID, PK), usuario_id (UUID, FK a usuarios.id, NULLABLE), servicio_id (UUID, FK a servicios.id, NOT NULL), personal_id (UUID, FK a personal.id, NOT NULL), fecha_hora_inicio (TIMESTAMP, NOT NULL), fecha_hora_fin (TIMESTAMP, NOT NULL), cliente_nombre (VARCHAR(255), NOT NULL), cliente_email (VARCHAR(255), NOT NULL), cliente_telefono (VARCHAR(20), NOT NULL), estado (ENUM: pendiente, confirmada, pendiente_actualizacion, cancelada, completada, NOT NULL), numero_confirmacion (VARCHAR(20), UNIQUE, NOT NULL), creado_en (TIMESTAMP, NOT NULL), actualizado_en (TIMESTAMP, NOT NULL).

#### RF-DB-012: Cálculo de Fecha/Hora de Fin de Cita
CUANDO se cree una cita, el sistema deberá calcular fecha_hora_fin como fecha_hora_inicio + servicio.duracion_minutos.

#### RF-DB-013: Estado por Defecto de Cita
El sistema deberá establecer el estado por defecto como "pendiente" para nuevas citas.

#### RF-DB-014: Generación de Número de Confirmación
El sistema deberá generar un numero_confirmacion alfanumérico único de 12 caracteres para cada cita.

#### RF-DB-015: Índice de Prevención de Solapamiento de Citas
El sistema deberá crear un índice compuesto en (personal_id, fecha_hora_inicio, fecha_hora_fin, estado) para optimizar consultas de solapamiento.

#### RF-DB-016: Asociación de Cita con Usuario
DONDE un usuario registrado cree una cita, el sistema deberá poblar usuario_id con su UUID.

#### RF-DB-017: Soporte de Cita de Invitado
El sistema deberá permitir que usuario_id sea NULL para soportar reservas de invitados (característica futura).

### 4.6 Tabla Horarios de Negocio (horarios_negocio)

#### RF-DB-018: Esquema de Horarios de Negocio
El sistema deberá mantener una tabla horarios_negocio con columnas: id (UUID, PK), dia_semana (INTEGER, NOT NULL, CHECK >= 0 AND <= 6), hora_apertura (TIME, NOT NULL), hora_cierre (TIME, NOT NULL), activo (BOOLEAN, DEFAULT TRUE), creado_en (TIMESTAMP, NOT NULL), actualizado_en (TIMESTAMP, NOT NULL).

#### RF-DB-019: Unicidad de Día de la Semana
El sistema deberá aplicar restricción única en (dia_semana, activo=true) para prevenir entradas activas duplicadas.

#### RF-DB-020: Validación de Tiempo de Horarios de Negocio
El sistema deberá aplicar restricción CHECK asegurando que hora_apertura < hora_cierre.

### 4.7 Tabla Fechas de Excepción (fechas_excepcion)

#### RF-DB-021: Esquema de Fechas de Excepción
El sistema deberá mantener una tabla fechas_excepcion con columnas: id (UUID, PK), fecha_excepcion (DATE, UNIQUE, NOT NULL), motivo (VARCHAR(255), NOT NULL), creado_en (TIMESTAMP, NOT NULL).

#### RF-DB-022: Unicidad de Fecha de Excepción
El sistema deberá aplicar restricción única en fecha_excepcion para prevenir entradas duplicadas.

### 4.8 Tabla de Idempotencia (claves_idempotencia)

#### RF-DB-023: Esquema de Idempotencia
El sistema deberá mantener una tabla claves_idempotencia con columnas: id (UUID, PK), clave_idempotencia (VARCHAR(255), UNIQUE, NOT NULL), hash_solicitud (VARCHAR(64), NOT NULL), estado_respuesta (INTEGER, NOT NULL), cuerpo_respuesta (JSONB, NOT NULL), creado_en (TIMESTAMP, NOT NULL), expira_en (TIMESTAMP, NOT NULL).

#### RF-DB-024: Índice de Clave de Idempotencia
El sistema deberá crear un índice único en clave_idempotencia para detección rápida de duplicados.

#### RF-DB-025: Expiración de Idempotencia
El sistema deberá establecer expira_en como creado_en + 24 horas para expiración automática.

#### RF-DB-026: Limpieza de Idempotencia (Inline)
El sistema deberá verificar y eliminar registros de idempotencia expirados (donde expira_en < current_timestamp) al procesar nuevas solicitudes de reserva, antes de insertar una nueva clave de idempotencia.

### 4.9 Rendimiento de Base de Datos

#### RF-DB-027: Índice en Fecha/Hora de Cita
El sistema deberá crear un índice en citas.fecha_hora_inicio para consultas de rango de fecha eficientes.

#### RF-DB-028: Índice en Email de Usuario
El sistema deberá crear un índice único en usuarios.email para consultas de autenticación.

#### RF-DB-029: Índices de Clave Foránea
El sistema deberá crear índices en todas las columnas de clave foránea para optimizar operaciones de join.

### 4.10 Integridad de Datos

#### RF-DB-030: Integridad Referencial
El sistema deberá aplicar restricciones de clave foránea con comportamientos ON DELETE apropiados: CASCADE para tablas de unión, RESTRICT para referencias críticas (ej. citas.servicio_id).

#### RF-DB-031: Aplicación de NOT NULL
El sistema deberá aplicar restricciones NOT NULL en todos los campos obligatorios para prevenir datos incompletos.

#### RF-DB-032: Aislamiento de Transacción
El sistema deberá usar nivel de aislamiento READ COMMITTED para todas las transacciones para balancear consistencia y rendimiento.

#### RF-DB-033: Protección de Reserva Concurrente
CUANDO múltiples transacciones intenten reservar el mismo espacio, el sistema deberá usar SELECT FOR UPDATE para bloquear filas y prevenir doble reserva.

### 4.11 Eliminación de Datos

#### RF-DB-034: Eliminación Suave
El sistema deberá usar eliminación suave (activo = FALSE) para usuarios, personal y servicios en lugar de eliminación física.

### 4.12 Migraciones de Base de Datos

#### RF-DB-035: Versionado de Migraciones
El sistema deberá usar Alembic para migraciones de base de datos con numeración de versión secuencial.

#### RF-DB-036: Reversibilidad de Migraciones
El sistema deberá implementar funciones tanto de upgrade como downgrade para todos los scripts de migración.

---

## 5. Requisitos No Funcionales

### 5.1 Rendimiento

#### RNF-001: Rendimiento Aceptable
El sistema deberá proporcionar tiempos de respuesta aceptables para interacciones de usuario bajo carga esperada de hasta 10 usuarios concurrentes, sin degradación perceptible de la experiencia del usuario.

### 5.2 Escalabilidad

#### RNF-002: Usuarios Concurrentes (MVP)
El sistema deberá soportar hasta 10 usuarios concurrentes sin degradación del rendimiento.

#### RNF-003: Pool de Conexiones de Base de Datos
El sistema deberá gestionar eficientemente conexiones de base de datos con un tamaño de pool de 5-20 conexiones.

#### RNF-004: Preparación para Escalado Horizontal
El sistema deberá ser diseñado para soportar escalado horizontal vía replicación de tareas AWS ECS.

### 5.3 Disponibilidad

#### RNF-005: Tiempo de Actividad del Sistema
El sistema deberá mantener 99% de tiempo de actividad durante horas de negocio (6:00-20:00 hora local).

#### RNF-006: Degradación Elegante
CUANDO la base de datos esté temporalmente no disponible, el sistema deberá mostrar información de servicio en caché y deshabilitar funcionalidad de reserva con un mensaje amigable al usuario.

### 5.4 Seguridad

#### RNF-007: Almacenamiento de Contraseña
El sistema nunca deberá almacenar contraseñas en texto plano; todas las contraseñas deberán ser hasheadas usando bcrypt con factor de trabajo >= 12.

#### RNF-008: Expiración de JWT
El sistema deberá emitir tokens JWT con expiración de 24 horas y requerir re-autenticación después de la expiración.

#### RNF-009: Validación de Entrada
El sistema deberá validar y sanitizar todas las entradas de usuario tanto en cliente como en servidor para prevenir ataques XSS e inyección SQL.

#### RNF-010: Encabezados de Seguridad Básicos
El sistema deberá implementar encabezados de seguridad básicos: X-Content-Type-Options, X-Frame-Options.

### 5.5 Usabilidad

#### RNF-011: Tiempo de Completar Reserva
Un cliente deberá poder completar una reserva en menos de 2 minutos para su primera reserva.

#### RNF-012: Diseño Web Responsivo
El sistema deberá proporcionar una experiencia de usuario funcional en navegadores web de escritorio con resoluciones estándar (1024px+).

#### RNF-013: Accesibilidad Básica
El sistema deberá proporcionar navegación por teclado básica y estructura semántica HTML apropiada.

#### RNF-014: Claridad de Error de Formulario
El sistema deberá mostrar errores de validación en línea junto al campo de formulario relevante con mensajes claros y accionables.

### 5.6 Mantenibilidad

#### RNF-015: Documentación de Código
El sistema deberá incluir docstrings para todos los endpoints API públicos y funciones de lógica de negocio complejas.

#### RNF-016: Seguridad de Tipos
El sistema deberá usar TypeScript (frontend) y type hints de Python (backend) con modo estricto habilitado.

#### RNF-017: Cobertura de Pruebas Automatizadas
El sistema deberá mantener al menos 70% de cobertura de código para lógica de negocio del backend.

#### RNF-018: Configuración de Entorno
El sistema deberá externalizar toda la configuración específica del entorno (URLs de base de datos, claves API) vía variables de entorno.

### 5.7 Despliegue y Operaciones

#### RNF-019: Containerización
El sistema deberá empaquetar tanto frontend como backend como contenedores Docker para despliegue en AWS ECS.

#### RNF-020: Hosting de Base de Datos
El sistema deberá usar AWS RDS PostgreSQL 15+ con respaldos automatizados habilitados.

#### RNF-021: Entrega de Assets Estáticos
El sistema deberá servir assets estáticos de frontend (JS, CSS, imágenes) desde AWS S3 vía CloudFront (optimización futura).

#### RNF-022: Registro de Logs
El sistema deberá registrar todos los errores, eventos de autenticación y transacciones de reserva en AWS CloudWatch con formato JSON estructurado.

#### RNF-023: Monitoreo
El sistema deberá exponer endpoints de health check en `/health` para monitoreo de salud AWS ECS.

### 5.8 Zona Horaria

#### RNF-024: Zona Horaria del Sistema
El sistema deberá operar en zona horaria America/Lima (UTC-5) para todos los cálculos de disponibilidad, horarios de negocio y timestamps de citas.

### 5.9 Privacidad de Datos

#### RNF-025: Protección de Datos Personales
El sistema deberá almacenar datos personales (email, teléfono, nombre) con encriptación en reposo en AWS RDS.

#### RNF-025: Registro de Acceso a Datos
El sistema deberá registrar todo acceso a datos personales de usuario para propósitos de auditoría.

### 5.9 Cumplimiento

#### RNF-026: Entregabilidad de Email
El sistema deberá usar AWS SES para emails transaccionales con configuración SPF y DKIM.

---

## 6. Sugerencias de Implementación y Decisiones de Diseño

### 6.1 Estrategia de Prevención de Doble Reserva

#### Enfoque: Idempotencia Multi-Capa

El sistema implementa un enfoque de dos capas para prevenir dobles reservas:

**Capa 1: Idempotencia a Nivel de Solicitud (Prevención de Duplicados del Cliente)**
- Frontend genera una clave de idempotencia UUID v4 para cada intento de reserva
- La clave se incluye en encabezado de solicitud: `Idempotency-Key: <uuid>`
- Backend almacena claves de idempotencia con hash de solicitud y respuesta en tabla `claves_idempotencia`
- Si se detecta clave duplicada dentro de ventana de 24 horas, retorna respuesta en caché (HTTP 200 con resultado original)
- Esto previene doble-clicks, re-envíos de formulario y reintentos de red de crear reservas duplicadas

**Capa 2: Bloqueo a Nivel de Espacio (Prevención de Reserva Concurrente)**
- Usar bloqueo a nivel de fila de base de datos con `SELECT FOR UPDATE` durante transacción de reserva:
  ```sql
  SELECT * FROM citas
  WHERE personal_id = ?
    AND estado IN ('confirmada', 'pendiente_actualizacion')
    AND fecha_hora_inicio < ?fecha_hora_fin
    AND fecha_hora_fin > ?fecha_hora_inicio
  FOR UPDATE;
  ```
- Esto asegura acceso serializado a espacios de tiempo solapados para el mismo miembro del personal
- La transacción se confirma solo si no existe cita conflictiva después de adquirir el bloqueo

**Capa 3: Validación Atómica + Inserción**
- Dentro de la misma transacción de base de datos:
  1. Bloquear y validar disponibilidad de espacio
  2. Insertar nuevo registro de cita
  3. Confirmar transacción
- Si la validación falla (espacio tomado), hacer rollback y retornar HTTP 409 Conflict

**Beneficios:**
- Capa 1 maneja errores de usuario (doble-clicks, reintentos)
- Capa 2 maneja condiciones de carrera (dos usuarios reservando mismo espacio simultáneamente)
- Capa 3 asegura atomicidad de validación + reserva

### 6.2 Implementación de Edición de Cita

Cuando un cliente edita una cita:

1. **Solicitud Inicial de Edición:**
   - Frontend envía PUT `/api/appointments/{id}` con nuevo datetime y staff_id opcional
   - Backend valida disponibilidad del nuevo espacio usando mismo mecanismo de bloqueo que reserva
   - Si está disponible, actualizar estado de cita a `pending_update` y almacenar datetime original en campo temporal
   - El espacio original permanece retenido (no liberado a otros clientes)

2. **Retención de Espacio:**
   - La cita permanece en sistema con datetime original pero status = `pending_update`
   - El cálculo de disponibilidad excluye citas `pending_update` de bloquear espacios
   - Frontend muestra banner: "Cita original retenida hasta confirmación"

3. **Confirmación:**
   - Cliente confirma nuevo espacio
   - Backend actualiza atómicamente appointment_datetime, end_datetime, staff_id, status = `confirmed`
   - El espacio original se libera (implícitamente, ya que la cita se movió a nuevo tiempo)

4. **Cancelación de Edición:**
   - Cliente cancela operación de edición
   - Backend revierte status a `confirmed`, mantiene datetime original
   - No se confirman cambios

**Manejo de Timeout (Inline):**
- Al consultar disponibilidad o al procesar cualquier operación de citas, el sistema deberá verificar si existen citas en estado `pending_update` con más de 30 minutos de antigüedad
- Si se detectan citas expiradas, el sistema deberá revertirlas automáticamente a estado `confirmada` con su datetime original antes de continuar con la operación solicitada
- Este enfoque elimina la necesidad de tareas en segundo plano (Celery) y garantiza consistencia on-demand

**Pseudocódigo de limpieza inline:**
```python
async def cleanup_expired_pending_updates(db: AsyncSession):
    threshold = datetime.now(tz=ZoneInfo("America/Lima")) - timedelta(minutes=30)
    await db.execute(
        update(Cita)
        .where(Cita.estado == "pending_update")
        .where(Cita.actualizado_en < threshold)
        .values(estado="confirmada")
    )
```

### 6.3 Lógica de Asignación de Personal

**Cuando personal_id es NULL en solicitud de reserva:**
1. Consultar todos los miembros del personal calificados para el servicio seleccionado (vía tabla de unión `servicio_personal`)
2. Filtrar al personal que no tiene citas conflictivas en el tiempo solicitado
3. Seleccionar el miembro del personal con menos citas en ese día (balanceo de carga)
4. Asignar ese personal_id a la cita

**Pseudocódigo de consulta:**
```sql
SELECT p.id, COUNT(c.id) as cantidad_citas
FROM personal p
JOIN servicio_personal sp ON p.id = sp.personal_id
LEFT JOIN citas c ON p.id = c.personal_id
  AND DATE(c.fecha_hora_inicio) = ?fecha_solicitada
  AND c.estado IN ('confirmada', 'pendiente_actualizacion')
WHERE sp.servicio_id = ?servicio_id
  AND p.activo = TRUE
  AND NOT EXISTS (
    SELECT 1 FROM citas c2
    WHERE c2.personal_id = p.id
      AND c2.estado IN ('confirmada', 'pendiente_actualizacion')
      AND c2.fecha_hora_inicio < ?fecha_hora_fin
      AND c2.fecha_hora_fin > ?fecha_hora_inicio
  )
GROUP BY p.id
ORDER BY cantidad_citas ASC
LIMIT 1;
```

### 6.4 Algoritmo de Cálculo de Disponibilidad

**Entrada:** servicio_id, fecha_inicio, fecha_fin, personal_id opcional

**Pasos:**
1. Recuperar duración del servicio de tabla `servicios`
2. Recuperar horarios de negocio para fechas en rango (considerando dia_semana)
3. Excluir fechas en tabla `fechas_excepcion`
4. Para cada fecha válida:
   - Generar espacios de intervalo de 30 minutos de hora_apertura a hora_cierre
   - Filtrar espacios donde (inicio_espacio + duracion) > hora_cierre
5. Consultar citas existentes para personal relevante (específico o todo personal calificado)
6. Filtrar espacios que se solapen con citas existentes:
   - Condición de solapamiento: `inicio_espacio < fin_existente AND fin_espacio > inicio_existente`
7. Retornar espacios disponibles agrupados por fecha

**Optimización:**
- Cachear horarios de negocio y fechas de excepción (raramente cambian)
- Usar índices de base de datos en (personal_id, fecha_hora_inicio, fecha_hora_fin)
- Limitar rango máximo de fecha a 2 meses para prevenir consultas costosas

### 6.5 Patrones de Transacción de Base de Datos

**Transacciones Críticas (ACID requerido):**
1. **Creación de Cita:**
   ```python
   async with db.begin():  # Iniciar transacción
       # Verificación de bloqueo
       conflicts = await db.execute(
           select(Cita)
           .where(...)
           .with_for_update()
       )
       if conflicts:
           raise ConflictError
       # Insertar
       nueva_cita = Cita(...)
       db.add(nueva_cita)
       # Auto-confirma al salir del contexto
   ```

2. **Verificación de Idempotencia + Reserva:**
   - Una sola transacción asegura que el registro de idempotencia se cree atómicamente con la cita
   - Previene carrera donde dos solicitudes idénticas ambas pasen la verificación de idempotencia

**Nivel de Aislamiento:**
- Usar `READ COMMITTED` para balance de consistencia y concurrencia
- Bloqueos a nivel de fila (SELECT FOR UPDATE) proporcionan seguridad adicional para secciones críticas

### 6.6 Estrategia de Gestión de Estado de Frontend

**TanStack Query (Estado del Servidor):**
- Gestionar todos los datos de API: servicios, disponibilidad, citas
- Caché automático, re-obtención en segundo plano, manejo de datos obsoletos
- Estructura de claves de consulta:
  - `['services']` - todos los servicios
  - `['availability', { serviceId, startDate, endDate, staffId }]` - espacios de disponibilidad
  - `['appointments', 'me']` - citas del usuario
  - `['appointments', appointmentId]` - cita específica

**Zustand (Estado del Cliente):**
- Estado de autenticación: `{ user, token, isAuthenticated }`
- Estado de flujo de reserva: `{ selectedService, selectedDate, selectedTime, selectedStaff }`
- Estado de UI: modales, toasts, overlays de carga

**Justificación de Separación:**
- Estado del servidor (TanStack) auto-sincroniza con backend, maneja caché
- Estado del cliente (Zustand) es efímero, específico de UI, no necesita caché

### 6.7 Formato de Respuesta de Error de API

**Estructura de Error Estandarizada:**
```json
{
  "error": "SLOT_UNAVAILABLE",
  "message": "El espacio de tiempo seleccionado ya no está disponible",
  "details": {
    "slot_datetime": "2026-02-10T14:00:00",
    "staff_id": "uuid-aqui"
  },
  "timestamp": "2026-02-04T10:30:00Z"
}
```

**Códigos de Error Comunes:**
- `SLOT_UNAVAILABLE` - conflicto de reserva (HTTP 409)
- `VALIDATION_ERROR` - entrada inválida (HTTP 422)
- `UNAUTHORIZED` - fallo de autenticación (HTTP 401)
- `FORBIDDEN` - permisos insuficientes (HTTP 403)
- `NOT_FOUND` - recurso no existe (HTTP 404)
- `INTERNAL_ERROR` - error inesperado del servidor (HTTP 500)

---

## 7. Trazabilidad de Requisitos

### 7.1 Cumplimiento de Criterios de Calidad INCOSE

Todos los requisitos en este documento adhieren a las reglas de calidad INCOSE:

1. **Necesario:** Cada requisito aborda una necesidad de negocio establecida en Initial_context.md
2. **Apropiado:** Los requisitos están limitados al MVP y coinciden con la fase de prototipo
3. **No ambiguo:** Los requisitos usan terminología precisa y patrones EARS
4. **Completo:** Los requisitos cubren todos los aspectos del sistema (frontend, backend, base de datos)
5. **Singular:** Cada requisito aborda una capacidad específica
6. **Factible:** Todos los requisitos son implementables con el stack tecnológico especificado
7. **Rastreable:** Los requisitos están identificados únicamente con formato RF-XX-NNN o RNF-NNN
8. **Verificable:** Cada requisito puede ser probado o demostrado

### 7.2 Resumen de Uso de Patrones EARS

| Patrón | IDs de Requisitos Ejemplo |
|---------|------------------------|
| Ubicuo | RF-FE-001, RF-BE-009, RF-DB-001 |
| Dirigido por eventos (CUANDO) | RF-FE-002, RF-BE-024, RF-DB-012 |
| Dirigido por estado (MIENTRAS) | RF-FE-021, RF-FE-038 |
| Opcional (DONDE) | RF-FE-018, RF-BE-016, RF-BE-021 |
| No deseado (SI-ENTONCES) | RF-FE-029, RF-BE-029, RF-BE-055 |

---

## 8. Criterios de Aceptación (Métricas de Éxito del MVP)

El MVP se considera exitoso cuando:

1. **CA-001:** Un cliente puede completar una reserva desde la selección de servicio hasta la confirmación en menos de 2 minutos (verificado vía pruebas de usuario)
2. **CA-002:** El sistema previene exitosamente dobles reservas en 100% de escenarios de reserva concurrentes (verificado vía pruebas de carga con solicitudes simultáneas)
3. **CA-003:** El dueño del negocio puede configurar horarios de negocio y ver agenda diaria dentro de 3 clicks (verificado vía pruebas de usabilidad)
4. **CA-004:** El sistema maneja 10 usuarios concurrentes sin degradación perceptible del rendimiento (verificado vía pruebas de carga)
5. **CA-005:** Todos los flujos críticos de usuario (reserva, edición, cancelación) funcionan correctamente en navegadores web estándar (verificado vía pruebas funcionales)

---

## 9. Fuera del Alcance (Explícitamente Excluido del MVP)

Las siguientes características están explícitamente excluidas del alcance del MVP:

1. Procesamiento de pagos en línea (Yape, Plin, tarjetas de crédito)
2. Integración de calendario (Google Calendar, exportación iCal)
3. Notificaciones automatizadas email/SMS/WhatsApp
4. Programas de lealtad de clientes o puntos de recompensa
5. Soporte multi-idioma (solo Español)
6. Analíticas avanzadas o dashboards de reportes
7. Aplicaciones nativas móviles (iOS/Android)
8. Integraciones de terceros (redes sociales, plataformas de marketing)
9. Horarios de trabajo específicos por personal (solo horarios a nivel de negocio)
10. Soporte multi-ubicación (solo un salón)
11. Optimización para dispositivos móviles (enfoque en web desktop)
12. Encriptación HTTPS (prototipo usará HTTP)

---

## 10. Glosario

| Término | Definición |
|------|------------|
| **Cita** | Una reserva de servicio programada con datetime específico, servicio y asignación de personal |
| **Dueño del Negocio** | Rol de usuario con permisos para gestionar servicios, personal, horarios y ver todas las citas |
| **Cliente** | Rol de usuario con permisos para ver servicios, reservar, editar y cancelar sus propias citas |
| **Número de Confirmación** | Identificador alfanumérico único de 12 caracteres para una cita |
| **Fecha de Excepción** | Una fecha específica cuando el salón está cerrado (feriado, vacaciones) |
| **Clave de Idempotencia** | Identificador único UUID v4 usado para prevenir envíos duplicados de reserva |
| **Servicio** | Una oferta del salón (ej. corte de pelo, manicure) con duración y precio definidos |
| **Espacio** | Un intervalo de tiempo específico disponible para reserva (generado en intervalos de 30 minutos) |
| **Personal** | Un empleado del salón calificado para realizar uno o más servicios |
| **Filtro de Período de Tiempo** | Rango seleccionable por usuario para ver disponibilidad: 1 semana, 2 semanas o 1 mes |

---

## Historial de Revisión del Documento

| Versión | Fecha | Autor | Cambios |
|---------|------|--------|---------|
| 1.0 | 2026-02-04 | Claude (Sonnet 4.5) | Especificación inicial de requisitos |

---

**Fin del Documento de Requisitos**
