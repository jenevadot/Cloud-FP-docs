# Documento de Diseño - Agenda Salón MVP

## Información del Documento
- **Versión:** 1.0
- **Fecha:** 2026-02-04
- **Estado:** Borrador
- **Basado en:** requirements.md, Initial_context.md, tech.md

---

## Estructura de Repositorios

El proyecto se divide en **dos repositorios independientes**:

```
agenda-salon/
├── agenda-salon-frontend/     # Repositorio Frontend
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── vite.config.ts
│   ├── Dockerfile
│   ├── .env.example
│   └── README.md
│
└── agenda-salon-backend/      # Repositorio Backend
    ├── app/
    ├── tests/
    ├── migrations/
    ├── requirements.txt
    ├── Dockerfile
    ├── alembic.ini
    ├── .env.example
    └── README.md
```

| Repositorio | Tecnología | Puerto Local | Descripción |
|-------------|------------|--------------|-------------|
| `agenda-salon-frontend` | React 18 + TypeScript + Vite | 5173 | Aplicación web del cliente y panel admin |
| `agenda-salon-backend` | FastAPI + Python 3.11 | 8000 | API REST y lógica de negocio |

### Comunicación entre Repositorios

```
┌─────────────────────────┐         ┌─────────────────────────┐
│  agenda-salon-frontend  │  HTTP   │  agenda-salon-backend   │
│  (React SPA)            │◄───────►│  (FastAPI REST API)     │
│  Puerto: 5173           │  JSON   │  Puerto: 8000           │
└─────────────────────────┘         └─────────────────────────┘
                                              │
                                              ▼
                                    ┌─────────────────────────┐
                                    │     PostgreSQL          │
                                    │     Puerto: 5432        │
                                    └─────────────────────────┘
```

### Variables de Entorno por Repositorio

**Frontend (`.env`)**
```env
VITE_API_URL=http://localhost:8000/api/v1
VITE_APP_NAME=Agenda Salón
```

**Backend (`.env`)**
```env
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/agenda_salon
JWT_CLAVE_SECRETA=tu-clave-secreta-aqui
JWT_ALGORITMO=HS256
JWT_HORAS_EXPIRACION=24
CORS_ORIGINS=http://localhost:5173
DEBUG=true
ENTORNO=development
```

---

## 1. Arquitectura

### 1.1 Arquitectura de Componentes

El sistema sigue una arquitectura de tres capas con separación clara de responsabilidades:

```
┌─────────────────────────────────────────────────────────────────────┐
│                           CLIENTE                                    │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    Frontend (React + TypeScript)               │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐   │  │
│  │  │   Páginas   │  │ Componentes │  │  Gestión de Estado  │   │  │
│  │  │  - Auth     │  │  - Calendario│  │  - Zustand (auth)   │   │  │
│  │  │  - Reserva  │  │  - Servicios │  │  - TanStack Query   │   │  │
│  │  │  - Panel    │  │  - Horarios  │  │    (estado servidor)│   │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ HTTPS/REST API
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                           SERVIDOR                                   │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    Backend (FastAPI + Python)                  │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐   │  │
│  │  │   Routers   │  │  Servicios  │  │    Middleware       │   │  │
│  │  │  - auth     │  │  - auth     │  │  - Validación JWT   │   │  │
│  │  │  - services │  │  - reservas │  │  - CORS             │   │  │
│  │  │  - disponib.│  │  - disponib.│  │  - Manejo errores   │   │  │
│  │  │  - citas    │  │  - personal │  │  - Logging          │   │  │
│  │  │  - negocio  │  │  - negocio  │  └─────────────────────┘   │  │
│  │  │  - personal │  └─────────────┘                            │  │
│  │  └─────────────┘         │                                    │  │
│  │                          ▼                                    │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │          Capa de Acceso a Datos (SQLAlchemy)            │  │  │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌───────────┐  │  │  │
│  │  │  │ Modelos │  │ Esquemas│  │ Repos   │  │Migraciones│  │  │  │
│  │  │  │  (ORM)  │  │(Pydantic)│ │ (CRUD)  │  │ (Alembic) │  │  │  │
│  │  │  └─────────┘  └─────────┘  └─────────┘  └───────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ SQLAlchemy Async
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        BASE DE DATOS                                 │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                   PostgreSQL 15+ (AWS RDS)                     │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │  │
│  │  │usuarios │ │servicios│ │ citas   │ │personal │ │horarios │ │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

#### Estructura del Repositorio Frontend (`agenda-salon-frontend`)

```
agenda-salon-frontend/
├── public/
│   └── favicon.ico
├── src/
│   ├── components/
│   │   ├── auth/
│   │   │   ├── FormularioLogin.tsx
│   │   │   ├── FormularioRegistro.tsx
│   │   │   └── RutaProtegida.tsx
│   │   ├── reservas/
│   │   │   ├── TarjetaServicio.tsx
│   │   │   ├── ListaServicios.tsx
│   │   │   ├── Calendario.tsx
│   │   │   ├── SelectorPersonal.tsx
│   │   │   ├── GrillaHorarios.tsx
│   │   │   ├── GrupoHorarios.tsx
│   │   │   └── ConfirmacionReserva.tsx
│   │   ├── citas/
│   │   │   ├── ListaCitas.tsx
│   │   │   ├── TarjetaCita.tsx
│   │   │   └── DetalleCita.tsx
│   │   ├── admin/
│   │   │   ├── GestorServicios.tsx
│   │   │   ├── ConfiguradorHorarios.tsx
│   │   │   ├── GestorExcepciones.tsx
│   │   │   └── CalendarioAgenda.tsx
│   │   └── comunes/
│   │       ├── SpinnerCarga.tsx
│   │       ├── MensajeError.tsx
│   │       ├── DialogoConfirmacion.tsx
│   │       └── Notificacion.tsx
│   ├── pages/
│   │   ├── PaginaInicio.tsx
│   │   ├── PaginaLogin.tsx
│   │   ├── PaginaRegistro.tsx
│   │   ├── PaginaReserva.tsx
│   │   ├── PaginaCitas.tsx
│   │   └── PanelAdmin.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useServicios.ts
│   │   ├── useDisponibilidad.ts
│   │   └── useCitas.ts
│   ├── stores/
│   │   ├── authStore.ts
│   │   └── flujoReservaStore.ts
│   ├── api/
│   │   ├── cliente.ts
│   │   ├── auth.ts
│   │   ├── servicios.ts
│   │   ├── disponibilidad.ts
│   │   └── citas.ts
│   ├── tipos/
│   │   └── index.ts
│   ├── utils/
│   │   ├── fecha.ts
│   │   └── idempotencia.ts
│   ├── schemas/
│   │   └── reserva.ts
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── .env.example
├── .gitignore
├── Dockerfile
├── nginx.conf
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

#### Estructura del Repositorio Backend (`agenda-salon-backend`)

```
agenda-salon-backend/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── excepciones.py
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── negocios.py
│   │   ├── servicios.py
│   │   ├── disponibilidad.py
│   │   ├── citas.py
│   │   └── personal.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   ├── negocio_service.py
│   │   ├── servicio_service.py
│   │   ├── disponibilidad_service.py
│   │   ├── cita_service.py
│   │   └── personal_service.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── usuario.py
│   │   ├── negocio.py
│   │   ├── servicio.py
│   │   ├── personal.py
│   │   ├── cita.py
│   │   ├── horario_negocio.py
│   │   ├── fecha_excepcion.py
│   │   └── idempotencia.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── negocio.py
│   │   ├── servicio.py
│   │   ├── disponibilidad.py
│   │   ├── cita.py
│   │   └── personal.py
│   ├── middleware/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── manejador_errores.py
│   └── utils/
│       ├── __init__.py
│       ├── seguridad.py
│       ├── tiempo.py
│       ├── codigo_confirmacion.py
│       └── idempotencia.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_auth.py
│   ├── test_servicios.py
│   ├── test_disponibilidad.py
│   └── test_citas.py
├── migrations/
│   ├── versions/
│   │   ├── 001_crear_tabla_usuarios.py
│   │   ├── 002_crear_tabla_personal.py
│   │   ├── 003_crear_tabla_servicios.py
│   │   └── ...
│   ├── env.py
│   └── script.py.mako
├── .env.example
├── .gitignore
├── alembic.ini
├── Dockerfile
├── requirements.txt
├── requirements-dev.txt
├── pytest.ini
└── README.md
```

### 1.2 Flujo de Datos

#### Flujo de Reserva de Cita

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Cliente  │     │ Frontend │     │ Backend  │     │   BD     │
└────┬─────┘     └────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │                │
     │ 1. Ver Servicios               │                │
     │───────────────>│                │                │
     │                │ GET /api/v1/servicios          │
     │                │───────────────>│                │
     │                │                │ SELECT servicios
     │                │                │───────────────>│
     │                │                │<───────────────│
     │                │<───────────────│                │
     │<───────────────│                │                │
     │                │                │                │
     │ 2. Seleccionar Servicio + Rango de Fechas      │
     │───────────────>│                │                │
     │                │ GET /api/v1/disponibilidad     │
     │                │   ?servicio_id=X               │
     │                │   &fecha_inicio=Y              │
     │                │   &fecha_fin=Z                 │
     │                │───────────────>│                │
     │                │                │ Limpiar ediciones
     │                │                │ expiradas      │
     │                │                │───────────────>│
     │                │                │<───────────────│
     │                │                │ Calcular slots │
     │                │                │───────────────>│
     │                │                │<───────────────│
     │                │<───────────────│                │
     │<───────────────│ Mostrar Calendario             │
     │                │                │                │
     │ 3. Seleccionar Personal (opcional)             │
     │───────────────>│                │                │
     │                │ GET /api/v1/disponibilidad     │
     │                │   ?personal_id=X (filtro)      │
     │                │───────────────>│                │
     │                │<───────────────│                │
     │<───────────────│ Mostrar Horarios               │
     │                │                │                │
     │ 4. Seleccionar Horario         │                │
     │───────────────>│                │                │
     │                │ Generar UUID  │                │
     │                │ (idempotencia)│                │
     │                │                │                │
     │ 5. Confirmar Reserva           │                │
     │───────────────>│                │                │
     │                │ POST /api/v1/citas             │
     │                │ Encabezados:                   │
     │                │   Idempotency-Key: <uuid>      │
     │                │ Cuerpo: {servicio_id, datetime,│
     │                │          personal_id?, contacto}│
     │                │───────────────>│                │
     │                │                │ INICIAR TRANSACCIÓN
     │                │                │───────────────>│
     │                │                │ Verificar idempotencia
     │                │                │───────────────>│
     │                │                │<───────────────│
     │                │                │ SELECT FOR UPDATE
     │                │                │ (bloquear slot)│
     │                │                │───────────────>│
     │                │                │<───────────────│
     │                │                │ Validar slot   │
     │                │                │ disponible     │
     │                │                │                │
     │                │                │ INSERT cita    │
     │                │                │───────────────>│
     │                │                │<───────────────│
     │                │                │ INSERT idempotencia
     │                │                │───────────────>│
     │                │                │<───────────────│
     │                │                │ COMMIT         │
     │                │                │───────────────>│
     │                │<───────────────│ 201 Creado     │
     │<───────────────│ Mostrar confirmación           │
     │                │                │                │
```

#### Flujo de Edición de Cita

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Cliente  │     │ Frontend │     │ Backend  │     │   BD     │
└────┬─────┘     └────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │                │
     │ 1. Solicitar Edición           │                │
     │───────────────>│                │                │
     │                │ PUT /api/v1/citas/{id}         │
     │                │ {nueva_fecha, personal_id?}    │
     │                │───────────────>│                │
     │                │                │ Validar nuevo slot
     │                │                │───────────────>│
     │                │                │<───────────────│
     │                │                │ UPDATE estado= │
     │                │                │ 'pending_update'
     │                │                │───────────────>│
     │                │<───────────────│ 200 OK         │
     │<───────────────│ Mostrar banner │                │
     │                │ "Cita original retenida"       │
     │                │                │                │
     │ 2. Confirmar Nuevo Horario     │                │
     │───────────────>│                │                │
     │                │ PUT /api/v1/citas/{id}/confirmar
     │                │───────────────>│                │
     │                │                │ UPDATE cita    │
     │                │                │ fecha, estado  │
     │                │                │ = 'confirmada' │
     │                │                │───────────────>│
     │                │<───────────────│ 200 OK         │
     │<───────────────│ Mostrar éxito  │                │
     │                │                │                │
     ─ ─ O BIEN ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
     │                │                │                │
     │ 2b. Cancelar Edición           │                │
     │───────────────>│                │                │
     │                │ DELETE /api/v1/citas/{id}/edicion
     │                │───────────────>│                │
     │                │                │ REVERTIR estado│
     │                │                │ = 'confirmada' │
     │                │                │───────────────>│
     │                │<───────────────│ 200 OK         │
     │<───────────────│ Mostrar original               │
     │                │                │                │
```

#### Flujo de Cálculo de Disponibilidad

```
                    ┌─────────────────────────────────┐
                    │   GET /api/v1/disponibilidad    │
                    │   ?servicio_id=X                │
                    │   &fecha_inicio=2026-02-10      │
                    │   &fecha_fin=2026-02-17         │
                    │   &personal_id=Y (opcional)     │
                    └─────────────────┬───────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │  1. Limpiar Ediciones Expiradas │
                    │  UPDATE citas SET estado =      │
                    │  'confirmada' WHERE estado =    │
                    │  'pending_update' AND           │
                    │  actualizado_en < NOW() - 30min │
                    └─────────────────┬───────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │  2. Obtener Duración Servicio   │
                    │  SELECT duracion_minutos        │
                    │  FROM servicios WHERE id = X    │
                    └─────────────────┬───────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │  3. Obtener Horarios Negocio    │
                    │  SELECT dia_semana,             │
                    │         hora_apertura,          │
                    │         hora_cierre             │
                    │  FROM horarios_negocio          │
                    │  WHERE activo = TRUE            │
                    └─────────────────┬───────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │  4. Obtener Fechas de Excepción │
                    │  SELECT fecha_excepcion         │
                    │  FROM fechas_excepcion          │
                    │  WHERE fecha_excepcion          │
                    │  BETWEEN start_date AND end_date│
                    └─────────────────┬───────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │  5. Obtener Personal Calificado │
                    │  SELECT p.id FROM personal p    │
                    │  JOIN servicio_personal sp      │
                    │  ON p.id = sp.personal_id       │
                    │  WHERE sp.servicio_id = X       │
                    │  AND p.activo = TRUE            │
                    └─────────────────┬───────────────┘
                                      │
                                      ▼
                    ┌─────────────────────────────────┐
                    │  6. Obtener Citas Existentes    │
                    │  SELECT personal_id,            │
                    │         fecha_hora_inicio,      │
                    │         fecha_hora_fin          │
                    │  FROM citas                     │
                    │  WHERE estado IN ('confirmada', │
                    │        'pendiente_actualizacion')
                    │  AND fecha_hora_inicio          │
                    │  BETWEEN start_date AND end_date│
                    └─────────────────┬───────────────┘
                                      │
                                      ▼
              ┌───────────────────────────────────────────┐
              │  7. Generar Slots Disponibles             │
              │                                           │
              │  PARA cada fecha EN rango_fechas:         │
              │    SI fecha EN fechas_excepcion: SALTAR   │
              │    SI dia_semana NO EN horarios: SALTAR   │
              │                                           │
              │    hora_apertura = horarios[dia_semana]   │
              │    hora_cierre = horarios[dia_semana]     │
              │                                           │
              │    PARA inicio_slot DESDE hora_apertura   │
              │         HASTA hora_cierre PASO 30min:     │
              │      fin_slot = inicio_slot + duracion    │
              │      SI fin_slot > hora_cierre: SALTAR    │
              │                                           │
              │      PARA cada personal EN calificados:   │
              │        SI NO hay solapamiento CON citas:  │
              │          AGREGAR slot a disponibles       │
              │                                           │
              │  RETORNAR slots_disponibles por fecha     │
              └───────────────────────────────────────────┘
```

### 1.3 Funciones Utilitarias Principales

#### Utilidades del Backend

**1. Utilidades de Seguridad (`app/utils/seguridad.py`)**

- **Propósito:** Centralizar las operaciones de autenticación y seguridad de contraseñas/tokens.
- **Dependencias:** `bcrypt`, `python-jose` (JWT)
- **Funciones a implementar:**
  - `hashear_contrasena`: Hashea contraseñas usando bcrypt con factor de trabajo 12
  - `verificar_contrasena`: Compara contraseña plana contra hash almacenado
  - `crear_token_acceso`: Genera token JWT incluyendo claims del usuario (id, email, rol) con expiración configurable (default 24 horas)
  - `decodificar_token_acceso`: Valida y decodifica token JWT; lanza excepción si es inválido o expirado

**2. Utilidades de Tiempo (`app/utils/tiempo.py`)**

- **Propósito:** Manejar todas las operaciones de fecha/hora con zona horaria consistente (Lima, UTC-5).
- **Dependencias:** `zoneinfo` (stdlib Python 3.9+)
- **Funciones a implementar:**
  - `ahora_lima`: Retorna datetime actual en zona horaria de Lima
  - `a_zona_lima`: Convierte cualquier datetime a zona horaria de Lima
  - `generar_slots_tiempo`: Genera lista de slots cada 30 minutos entre hora de apertura y cierre, filtrando aquellos donde el servicio no cabe antes del cierre
  - `slots_se_solapan`: Determina si dos rangos de tiempo (inicio/fin) tienen intersección
  - `agrupar_slots_por_periodo`: Clasifica slots en tres grupos: mañana (06:00-11:59), mediodía (12:00-15:59), tarde (16:00-20:00)

**3. Generador de Código de Confirmación (`app/utils/codigo_confirmacion.py`)**

- **Propósito:** Generar códigos únicos legibles para identificar citas.
- **Dependencias:** `secrets`, `string` (stdlib)
- **Comportamiento:**
  - Genera código de 12 caracteres usando solo letras mayúsculas y dígitos
  - Usa `secrets.choice` para garantizar aleatoriedad criptográficamente segura
  - Ejemplo de salida: `A7X9B2K4M1P3`

**4. Utilidades de Idempotencia (`app/utils/idempotencia.py`)**

- **Propósito:** Prevenir procesamiento duplicado de solicitudes de reserva.
- **Dependencias:** `hashlib`, `json` (stdlib)
- **Funciones a implementar:**
  - `calcular_hash_solicitud`: Genera hash SHA-256 del cuerpo de solicitud serializado (JSON ordenado por keys) para detectar solicitudes idénticas
  - `limpiar_claves_idempotencia_expiradas`: Función async que elimina registros de idempotencia vencidos de la base de datos. Se ejecuta inline antes de insertar nueva clave (no requiere job separado)

#### Utilidades del Frontend

**1. Cliente API (`src/api/cliente.ts`)**

- **Propósito:** Centralizar comunicación HTTP con el backend.
- **Dependencias:** `axios`
- **Comportamiento:**
  - Crear instancia de Axios con baseURL desde variable de entorno `VITE_API_URL` y timeout de 10 segundos
  - Interceptor de solicitud: obtiene token del store de autenticación (Zustand) y lo agrega como header `Authorization: Bearer {token}`
  - Interceptor de respuesta: detecta errores 401 y ejecuta `cerrarSesion()` del store de auth para limpiar sesión expirada

**2. Generador de Clave de Idempotencia (`src/utils/idempotencia.ts`)**

- **Propósito:** Generar identificador único por intento de reserva.
- **Dependencias:** Web Crypto API (nativo del navegador)
- **Comportamiento:** Retorna UUID v4 usando `crypto.randomUUID()`. Se genera una vez cuando el usuario llega al paso de confirmación y se envía en header `Idempotency-Key`

**3. Utilidades de Fecha (`src/utils/fecha.ts`)**

- **Propósito:** Formateo y manipulación de fechas para la UI.
- **Dependencias:** `date-fns`, `date-fns/locale/es`
- **Funciones a implementar:**
  - `formatearFechaMostrar`: Formato legible en español (ej: "Lunes 10 de Febrero")
  - `formatearSlotHora`: Formato HH:mm para mostrar horarios
  - `obtenerRangoFechas`: Calcula fecha inicio (hoy) y fin según semanas seleccionadas (1, 2 o 4)
  - `agruparSlotsPorPeriodo`: Misma lógica que backend - clasifica slots en mañana/mediodía/tarde

**4. Esquemas de Validación de Formularios (`src/schemas/reserva.ts`)**

- **Propósito:** Validar datos de formularios antes de enviar al backend.
- **Dependencias:** `zod`
- **Esquemas a implementar:**
  - `esquemaInfoContacto`: Valida nombre (mínimo 2 caracteres), email (formato válido), teléfono (regex `^9\d{8}$` para números peruanos de 9 dígitos)
  - `esquemaReserva`: Compuesto por servicioId (UUID), personalId (UUID opcional), fechaHoraCita (ISO datetime), y contacto (usando esquemaInfoContacto)

### 1.4 Puntos de Integración

#### Endpoints de la API

##### Autenticación

| Endpoint | Método | Auth | Descripción |
|----------|--------|------|-------------|
| `/api/v1/auth/registrar` | POST | No | Registro de usuario |
| `/api/v1/auth/login` | POST | No | Login y obtención de JWT |

##### Negocios

| Endpoint | Método | Auth | Descripción |
|----------|--------|------|-------------|
| `/api/v1/negocios` | POST | Usuario | Crear negocio (creador = dueño) |
| `/api/v1/negocios` | GET | Usuario | Listar negocios del usuario |
| `/api/v1/negocios/{negocio_id}` | GET | No | Info pública del negocio |
| `/api/v1/negocios/{negocio_id}` | PUT | Dueño | Actualizar negocio |

##### Servicios (bajo contexto de negocio)

| Endpoint | Método | Auth | Descripción |
|----------|--------|------|-------------|
| `/api/v1/negocios/{negocio_id}/servicios` | GET | No | Lista servicios activos |
| `/api/v1/negocios/{negocio_id}/servicios` | POST | Dueño | Crear servicio |
| `/api/v1/negocios/{negocio_id}/servicios/{id}` | PUT | Dueño | Actualizar servicio |
| `/api/v1/negocios/{negocio_id}/servicios/{id}` | DELETE | Dueño | Desactivar servicio |

##### Personal (bajo contexto de negocio)

| Endpoint | Método | Auth | Descripción |
|----------|--------|------|-------------|
| `/api/v1/negocios/{negocio_id}/personal` | GET | No | Lista personal activo |
| `/api/v1/negocios/{negocio_id}/personal` | POST | Dueño | Crear personal |
| `/api/v1/negocios/{negocio_id}/personal/{id}` | PUT | Dueño | Actualizar personal |
| `/api/v1/negocios/{negocio_id}/personal/{id}` | DELETE | Dueño | Desactivar personal |

##### Configuración del Negocio

| Endpoint | Método | Auth | Descripción |
|----------|--------|------|-------------|
| `/api/v1/negocios/{negocio_id}/horarios` | GET | No | Horarios del negocio |
| `/api/v1/negocios/{negocio_id}/horarios` | POST | Dueño | Configurar horarios |
| `/api/v1/negocios/{negocio_id}/excepciones` | GET | No | Fechas de excepción |
| `/api/v1/negocios/{negocio_id}/excepciones` | POST | Dueño | Crear excepción |
| `/api/v1/negocios/{negocio_id}/excepciones/{id}` | DELETE | Dueño | Eliminar excepción |

##### Disponibilidad y Citas (bajo contexto de negocio)

| Endpoint | Método | Auth | Descripción |
|----------|--------|------|-------------|
| `/api/v1/negocios/{negocio_id}/disponibilidad` | GET | No | Consulta disponibilidad |
| `/api/v1/negocios/{negocio_id}/citas` | POST | Cliente | Crear cita |
| `/api/v1/negocios/{negocio_id}/citas/{id}` | GET | Cliente/Dueño | Detalle de cita |
| `/api/v1/negocios/{negocio_id}/citas/{id}` | PUT | Cliente | Editar cita |
| `/api/v1/negocios/{negocio_id}/citas/{id}` | DELETE | Cliente/Dueño | Cancelar cita |

##### Citas del Usuario (cross-negocio)

| Endpoint | Método | Auth | Descripción |
|----------|--------|------|-------------|
| `/api/v1/citas/me` | GET | Cliente | Mis citas (todos los negocios) |

##### Health Check

| Endpoint | Método | Auth | Descripción |
|----------|--------|------|-------------|
| `/health` | GET | No | Verificación de salud |

#### Integración con Servicios Externos

```
┌─────────────────────────────────────────────────────────────────┐
│                     Infraestructura AWS                          │
│                                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │CloudFront│───>│   ALB    │───>│   ECS    │───>│   RDS    │  │
│  │  (CDN)   │    │          │    │(Backend) │    │(Postgres)│  │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘  │
│       │                               │                         │
│       │                               │                         │
│  ┌──────────┐                    ┌──────────┐                   │
│  │    S3    │                    │   SES    │                   │
│  │(Estático)│                    │ (Emails) │                   │
│  └──────────┘                    └──────────┘                   │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CloudWatch                           │   │
│  │  - Logs de aplicación (JSON estructurado)                 │   │
│  │  - Seguimiento de errores                                 │   │
│  │  - Métricas de health check                               │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

#### Conexión a Base de Datos

```python
# app/database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

DATABASE_URL = os.getenv("DATABASE_URL")

engine = create_async_engine(
    DATABASE_URL,
    pool_size=5,
    max_overflow=15,  # Total máximo: 20 conexiones
    pool_pre_ping=True,
)

AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

async def obtener_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()
```

---

## 2. Manejo de Errores

### 2.1 Estrategia de Errores del Backend

#### Estructura de Respuesta de Error

- **Ubicación:** Definir en `app/schemas/error.py`
- **Propósito:** Estandarizar todas las respuestas de error de la API.
- **Campos obligatorios:**
  - `error`: Código de error en mayúsculas con guiones bajos (ej: `SLOT_NO_DISPONIBLE`, `ERROR_VALIDACION`)
  - `mensaje`: Texto legible para mostrar al usuario en español
  - `detalles`: Diccionario con información adicional del contexto (fecha solicitada, ID de recurso, etc.)
  - `timestamp`: Fecha/hora del error en formato ISO 8601 con zona horaria

#### Jerarquía de Excepciones

- **Ubicación:** `app/excepciones.py`
- **Propósito:** Mapear errores de negocio a códigos HTTP apropiados.
- **Estructura:**
  - Clase base `ExcepcionApp` que almacena: código de error, mensaje, código HTTP y detalles
  - Subclases especializadas:
    - `ErrorValidacion` → HTTP 422: Datos de entrada inválidos
    - `ErrorConflicto` → HTTP 409: Recurso en conflicto (slot ya reservado)
    - `ErrorNoEncontrado` → HTTP 404: Recurso no existe
    - `ErrorNoAutorizado` → HTTP 401: Token inválido o ausente
    - `ErrorProhibido` → HTTP 403: Usuario sin permisos suficientes

#### Manejador Global de Excepciones

- **Ubicación:** `app/middleware/manejador_errores.py`
- **Propósito:** Interceptar todas las excepciones y convertirlas en respuestas JSON consistentes.
- **Comportamientos:**
  - Registrar manejador para `ExcepcionApp`: extraer campos y retornar JSONResponse con código HTTP apropiado
  - Registrar manejador para `Exception` genérica: loguear stack trace completo con `logger.exception()`, retornar HTTP 500 con mensaje genérico (sin exponer detalles internos)
  - Incluir timestamp en zona horaria de Lima en todas las respuestas

#### Manejo de Errores en Transacciones de Base de Datos

- **Propósito:** Garantizar atomicidad y manejo correcto de errores en operaciones críticas.
- **Patrón a implementar:**
  1. Usar context manager `async with db.begin()` para transacciones atómicas
  2. Dentro de la transacción: limpiar idempotencia expirada → verificar clave existente → bloquear con `SELECT FOR UPDATE` → validar → insertar → guardar idempotencia
  3. La transacción hace commit automático al salir del bloque sin errores
  4. Capturar excepciones específicas de SQLAlchemy:
     - `IntegrityError`: Convertir a `ErrorConflicto` (HTTP 409)
     - `OperationalError`: Convertir a error de base de datos (HTTP 503)

### 2.2 Estrategia de Errores del Frontend

#### Tipo de Error de API

- **Ubicación:** `src/api/tipos.ts`
- **Propósito:** Tipar las respuestas de error del backend.
- **Estructura:** Interface TypeScript con los 4 campos del backend (error, mensaje, detalles, timestamp)

#### Hook de Manejo de Errores

- **Ubicación:** `src/hooks/useErrorApi.ts`
- **Propósito:** Centralizar la lógica de respuesta a errores de API.
- **Comportamientos por código de error:**
  - `SLOT_NO_DISPONIBLE`: Mostrar toast de error + invalidar queries de disponibilidad para refrescar datos
  - `ERROR_VALIDACION`: No mostrar toast (los errores de campo los maneja el formulario)
  - `NO_AUTORIZADO`: Mostrar toast + ejecutar cerrarSesion() + navegar a /login
  - `ERROR_RED`: Mostrar toast indicando problema de conexión
  - Default: Mostrar mensaje del backend o mensaje genérico

#### Integración con TanStack Query

- **Ubicación:** Hooks de mutación (ej: `src/hooks/useCitas.ts`)
- **Propósito:** Manejar estados de éxito/error en operaciones de escritura.
- **Patrón de `useMutation`:**
  - `mutationFn`: Generar clave de idempotencia antes de llamar al API
  - `onSuccess`: Invalidar queries relacionadas (citas, disponibilidad) + mostrar toast de éxito con número de confirmación
  - `onError`: Delegar al hook useErrorApi
  - `retry`: Función que retorna false para errores 4xx, permite hasta 2 reintentos para errores 5xx

#### Validación de Formularios

- **Ubicación:** Componentes de formulario
- **Propósito:** Validar datos antes de enviar y mostrar errores inline.
- **Patrón:**
  - Usar `useForm` de react-hook-form con `zodResolver` para conectar esquemas Zod
  - Acceder a errores via `formState.errors`
  - Renderizar mensaje de error condicionalmente debajo de cada campo con estilos de texto rojo

#### Wrapper de Estados de Query

- **Ubicación:** `src/components/comunes/EnvolvedorQuery.tsx`
- **Propósito:** Abstraer el manejo de estados loading/error/success para queries.
- **Comportamiento:**
  - Props: recibe resultado de useQuery y función children que recibe los datos
  - Si `isLoading`: renderizar componente `SpinnerCarga`
  - Si `isError`: renderizar componente `MensajeError` con botón de reintentar que llama a `refetch()`
  - Si éxito: renderizar children pasando los datos

---

## 3. Enfoque de Implementación

### 3.1 Fases de Desarrollo

#### Fase 1: Fundamentos (Semana 1-2)

**Configuración del Backend**
1. Inicializar proyecto FastAPI con estructura de carpetas
2. Configurar SQLAlchemy async con PostgreSQL
3. Crear modelos de base de datos (usuarios, servicios, personal, citas)
4. Implementar migraciones con Alembic
5. Configurar middleware de CORS y manejo de errores
6. Implementar endpoint de health check

**Configuración del Frontend**
1. Inicializar proyecto React + Vite + TypeScript
2. Configurar Tailwind CSS
3. Configurar TanStack Query y Zustand
4. Crear estructura de carpetas y enrutamiento básico
5. Implementar cliente API con Axios

#### Fase 2: Autenticación (Semana 2-3)

**Backend**
1. Implementar registro de usuario (RF-BE-001 a RF-BE-003)
2. Implementar login con JWT (RF-BE-004 a RF-BE-006)
3. Crear middleware de validación de token (RF-BE-007)
4. Implementar control de acceso por roles (RF-BE-008)

**Frontend**
1. Crear formularios de registro y login (RF-FE-001 a RF-FE-004)
2. Implementar gestión de sesión con Zustand (RF-FE-005, RF-FE-006)
3. Crear componente RutaProtegida (RF-FE-008)
4. Implementar logout (RF-FE-007)

#### Fase 3: Servicios y Personal (Semana 3-4)

**Backend**
1. CRUD de servicios (RF-BE-009 a RF-BE-014)
2. CRUD de personal (RF-BE-047 a RF-BE-050)
3. Relación servicio-personal

**Frontend**
1. Vista de catálogo de servicios (RF-FE-009 a RF-FE-011)
2. Panel de gestión de servicios (RF-FE-041)

#### Fase 4: Disponibilidad (Semana 4-5)

**Backend**
1. Endpoint de consulta de disponibilidad (RF-BE-015)
2. Implementar limpieza inline de ediciones expiradas (RF-BE-015A)
3. Lógica de cálculo de slots (RF-BE-016 a RF-BE-022)
4. CRUD de horarios de negocio (RF-BE-041, RF-BE-042)
5. CRUD de fechas de excepción (RF-BE-043 a RF-BE-046)

**Frontend**
1. Componente Calendario (RF-FE-012 a RF-FE-015)
2. Selector de personal (RF-FE-016 a RF-FE-019)
3. Grilla de slots de tiempo (RF-FE-020 a RF-FE-024)
4. Configuración de horarios de negocio (RF-FE-042, RF-FE-043)

#### Fase 5: Reservas (Semana 5-6)

**Backend**
1. Endpoint de crear cita (RF-BE-023)
2. Sistema de idempotencia (RF-BE-024, RF-BE-030)
3. Bloqueo optimista y validación (RF-BE-025, RF-BE-026)
4. Asignación automática de personal (RF-BE-027)
5. Generación de código de confirmación (RF-BE-028)

**Frontend**
1. Formulario de confirmación (RF-FE-025 a RF-FE-028)
2. Generación de clave de idempotencia (RF-FE-031)
3. Retroalimentación de éxito/error (RF-FE-029, RF-FE-030)

#### Fase 6: Gestión de Citas (Semana 6-7)

**Backend**
1. Listar citas de usuario (RF-BE-031, RF-BE-032)
2. Detalle de cita (RF-BE-033)
3. Cancelar cita (RF-BE-034 a RF-BE-036)
4. Editar cita con pending_update (RF-BE-037 a RF-BE-040)

**Frontend**
1. Lista de citas (RF-FE-032, RF-FE-033)
2. Detalle de cita (RF-FE-034)
3. Flujo de cancelación (RF-FE-035, RF-FE-037, RF-FE-038)
4. Flujo de edición (RF-FE-036, RF-FE-039, RF-FE-040)

#### Fase 7: Panel de Administración (Semana 7-8)

**Backend**
1. Endpoints para vista de agenda del negocio
2. Filtros por fecha

**Frontend**
1. Vista de calendario de citas (RF-FE-044)
2. Filtros de agenda (RF-FE-045)

#### Fase 8: Pruebas y Despliegue (Semana 8-9)

1. Pruebas unitarias backend (>70% cobertura)
2. Pruebas de integración de flujos críticos
3. Configuración de Docker
4. Despliegue a AWS ECS
5. Configuración de logging en CloudWatch

### 3.2 Detalles Técnicos de Implementación

#### Estrategia de Migraciones de Base de Datos

- **Ubicación:** `migrations/versions/`
- **Herramienta:** Alembic
- **Orden de ejecución:** Las migraciones deben ejecutarse en orden secuencial respetando dependencias entre tablas
- **Secuencia:**
  1. `001_crear_tabla_usuarios` - Tabla base de usuarios
  2. `002_crear_tabla_negocios` - Tabla de negocios (depende de usuarios para dueno_id)
  3. `003_crear_tabla_personal` - Personal del salón (incluye negocio_id)
  4. `004_crear_tabla_servicios` - Catálogo de servicios (incluye negocio_id)
  5. `005_crear_tabla_servicio_personal` - Relación muchos-a-muchos entre servicios y personal
  6. `006_crear_tabla_horarios_negocio` - Configuración de horarios por día (incluye negocio_id)
  7. `007_crear_tabla_fechas_excepcion` - Días no laborables (incluye negocio_id)
  8. `008_crear_tabla_citas` - Reservas de clientes (incluye negocio_id, depende de usuarios, servicios, personal)
  9. `009_crear_tabla_idempotencia` - Registro de claves de idempotencia
  10. `010_agregar_indices` - Índices para optimización de consultas (incluir índices por negocio_id)

#### Estrategia de Versionado de API

- **Propósito:** Permitir evolución del API sin romper clientes existentes.
- **Patrón:** Prefijo `/api/v1/` en todas las rutas
- **Routers a registrar:**
  - `/api/v1/auth` - Autenticación
  - `/api/v1/negocios` - CRUD de negocios y recursos anidados
  - `/api/v1/negocios/{negocio_id}/servicios` - CRUD de servicios
  - `/api/v1/negocios/{negocio_id}/personal` - Gestión de personal
  - `/api/v1/negocios/{negocio_id}/horarios` - Configuración de horarios
  - `/api/v1/negocios/{negocio_id}/excepciones` - Fechas de excepción
  - `/api/v1/negocios/{negocio_id}/disponibilidad` - Consulta de slots
  - `/api/v1/negocios/{negocio_id}/citas` - Gestión de citas
  - `/api/v1/citas/me` - Citas del usuario (cross-negocio)

#### Configuración de Entorno

- **Ubicación:** `app/config.py`
- **Patrón:** Usar `pydantic_settings.BaseSettings` para validación automática de variables
- **Variables requeridas:**
  - `DATABASE_URL`: Cadena de conexión PostgreSQL con driver asyncpg
  - `JWT_CLAVE_SECRETA`: Clave para firmar tokens (sin valor por defecto, forzar configuración)
  - `JWT_ALGORITMO`: Algoritmo de firma (default: HS256)
  - `JWT_HORAS_EXPIRACION`: Duración del token (default: 24)
  - `DEBUG`: Modo debug (default: false)
  - `ENTORNO`: development/staging/production
  - `AWS_REGION`: Región de AWS (default: us-east-1)
- **Carga automática:** Desde archivo `.env` usando `env_file` en Config interna

#### Configuración de Docker

**Backend Dockerfile**
- **Imagen base:** `python:3.11-slim` (liviana y segura)
- **Pasos de build:**
  1. Establecer directorio de trabajo `/app`
  2. Copiar `requirements.txt` e instalar dependencias (para cache de capas)
  3. Copiar código de aplicación (`app/`), migraciones y `alembic.ini`
  4. Exponer puerto 8000
  5. Comando de inicio: uvicorn apuntando a `app.main:app` en host 0.0.0.0

**Frontend Dockerfile**
- **Patrón:** Multi-stage build para optimizar tamaño de imagen
- **Stage 1 (builder):**
  - Imagen: `node:20-alpine`
  - Instalar dependencias con `npm ci` (reproducible)
  - Ejecutar `npm run build` para generar assets
- **Stage 2 (producción):**
  - Imagen: `nginx:alpine`
  - Copiar `dist/` del stage anterior a directorio html de nginx
  - Copiar configuración nginx personalizada
  - Exponer puerto 80

**Docker Compose para Desarrollo**
- **Propósito:** Levantar ambiente completo localmente con un solo comando
- **Servicios:**
  - `postgres`: PostgreSQL 15 Alpine con volumen persistente para datos
  - `backend`: Build desde Dockerfile del backend, variables de entorno para DB/JWT/CORS, volumen para hot-reload de código
  - `frontend`: Build desde Dockerfile del frontend, depende del backend
- **Networking:** Red interna automática permite comunicación entre servicios por nombre
- **Puertos expuestos:** PostgreSQL (5432), Backend (8000), Frontend (80)

### 3.3 Estrategia de Pruebas

#### Pruebas Unitarias del Backend

- **Herramientas:** pytest, pytest-asyncio
- **Ubicación:** `tests/`
- **Cobertura objetivo:** >70%
- **Casos críticos a probar:**
  - Cálculo de disponibilidad: verificar que slots ocupados por citas existentes se excluyen correctamente
  - Prevención de doble reserva: simular transacciones concurrentes y verificar que solo una tiene éxito
  - Validación de horarios: verificar que slots fuera del horario del negocio no se ofrecen
  - Fechas de excepción: verificar que días no laborables no muestran disponibilidad
  - Expiración de ediciones: verificar que citas en estado `pending_update` por más de 30 minutos se revierten

#### Pruebas de Componentes del Frontend

- **Herramientas:** Vitest o Jest, React Testing Library
- **Ubicación:** Junto a componentes en carpeta `__tests__/`
- **Componentes críticos a probar:**
  - `GrillaHorarios`: Verificar agrupación correcta de slots en mañana/mediodía/tarde, verificar que slots no disponibles aparecen deshabilitados
  - `Calendario`: Verificar navegación entre semanas, selección de fechas
  - `FormularioContacto`: Verificar validación de campos y mensajes de error
  - `ConfirmacionReserva`: Verificar que muestra resumen correcto antes de confirmar

### 3.4 Lista de Verificación para Despliegue

#### Repositorio Backend

**Seguridad:**
- JWT_CLAVE_SECRETA configurada como variable de entorno segura (no en código)
- Credenciales de base de datos en AWS Secrets Manager
- CORS_ORIGINS configurado solo para dominios permitidos
- Rate limiting configurado si es necesario

**Base de Datos:**
- RDS PostgreSQL creado con respaldos automáticos habilitados
- Migraciones ejecutadas exitosamente con `alembic upgrade head`
- Índices creados según especificaciones de requerimientos

**Pruebas:**
- Pruebas unitarias pasando (`pytest`)
- Cobertura >70% (`pytest --cov=app`)
- Pruebas de integración de flujos críticos pasando

**Contenedor:**
- Dockerfile construye sin errores
- Imagen subida a Amazon ECR
- Health check en `/health` respondiendo correctamente

#### Repositorio Frontend

**Configuración:**
- VITE_API_URL apuntando a URL de producción del backend
- Variables de entorno de producción configuradas

**Build:**
- `npm run build` ejecuta sin errores
- Bundle size optimizado (code splitting activo)
- Assets estáticos generados correctamente

**Contenedor:**
- Dockerfile construye sin errores
- nginx.conf configurado para SPA (redirigir rutas a index.html)
- Imagen subida a Amazon ECR

#### Infraestructura AWS

**ECS:**
- Task Definitions configuradas para backend y frontend
- Servicios ECS creados y ejecutándose
- Auto-scaling configurado según carga esperada

**Networking:**
- Application Load Balancer configurado con health checks
- Target groups para backend (:8000) y frontend (:80)
- Certificado SSL configurado para dominio de producción

**Monitoreo:**
- Log groups de CloudWatch creados
- Alarmas de CloudWatch configuradas para errores y latencia
- Logging estructurado en formato JSON funcionando

---

## Apéndice A: Diagramas de Referencia

### Diagrama Entidad-Relación (Multi-Tenant)

```
┌───────────────┐
│   usuarios    │
├───────────────┤
│ id (PK)       │
│ email         │
│ contrasena    │
│ nombre        │
│ telefono      │
│ rol           │
│ activo        │
│ creado_en     │
│ actualizado_en│
└───────┬───────┘
        │
        │ 1:N (un usuario puede ser dueño de varios negocios)
        ▼
┌───────────────────┐
│     negocios      │
├───────────────────┤
│ id (PK)           │
│ dueno_id (FK)     │──────► usuarios.id
│ nombre            │
│ slug (UNIQUE)     │
│ activo            │
│ creado_en         │
│ actualizado_en    │
└─────────┬─────────┘
          │
          │ 1:N (un negocio tiene múltiples recursos)
          │
    ┌─────┴─────┬─────────────┬─────────────┬─────────────┐
    ▼           ▼             ▼             ▼             ▼
┌─────────┐ ┌─────────┐ ┌───────────┐ ┌───────────┐ ┌─────────┐
│personal │ │servicios│ │ horarios  │ │excepciones│ │  citas  │
└────┬────┘ └────┬────┘ └─────┬─────┘ └─────┬─────┘ └────┬────┘
     │           │            │             │            │
     └───────────┴────────────┴─────────────┴────────────┘
                              │
                    Todos tienen negocio_id (FK)
```

#### Detalle de Tablas

```
┌───────────────────┐       ┌───────────────────┐       ┌───────────────────┐
│     negocios      │       │     personal      │       │    servicios      │
├───────────────────┤       ├───────────────────┤       ├───────────────────┤
│ id (PK, UUID)     │       │ id (PK, UUID)     │       │ id (PK, UUID)     │
│ dueno_id (FK)     │◄──┐   │ negocio_id (FK)   │──────►│ negocio_id (FK)   │
│ nombre            │   │   │ nombre            │       │ nombre            │
│ slug (UNIQUE)     │   │   │ email             │       │ duracion_min      │
│ activo            │   │   │ activo            │       │ precio_pen        │
│ creado_en         │   │   │ creado_en         │       │ activo            │
│ actualizado_en    │   │   │ actualizado_en    │       │ creado_en         │
└───────────────────┘   │   └─────────┬─────────┘       │ actualizado_en    │
                        │             │                 └─────────┬─────────┘
                        │             │                           │
                        │             │   ┌───────────────────────┴─────────┐
                        │             │   │      servicio_personal          │
                        │             │   ├─────────────────────────────────┤
                        │             └──►│ servicio_id (FK)                │
                        │                 │ personal_id (FK)                │
                        │                 │ PK (servicio_id, personal_id)   │
                        │                 └─────────────────────────────────┘
                        │
                        │   ┌───────────────────────────────────────────────┐
                        │   │                    citas                       │
                        │   ├───────────────────────────────────────────────┤
                        └──►│ id (PK, UUID)                                 │
                            │ negocio_id (FK)  ◄── NUEVO                    │
                            │ usuario_id (FK, nullable)                      │
                            │ servicio_id (FK)                               │
                            │ personal_id (FK)                               │
                            │ fecha_hora_inicio                              │
                            │ fecha_hora_fin                                 │
                            │ cliente_nombre                                 │
                            │ cliente_email                                  │
                            │ cliente_telefono                               │
                            │ estado (pendiente|confirmada|pending_update|..)│
                            │ numero_confirmacion                            │
                            │ creado_en                                      │
                            │ actualizado_en                                 │
                            └───────────────────────────────────────────────┘

┌───────────────────┐       ┌───────────────────┐
│ horarios_negocio  │       │ fechas_excepcion  │
├───────────────────┤       ├───────────────────┤
│ id (PK, UUID)     │       │ id (PK, UUID)     │
│ negocio_id (FK)   │◄─┐    │ negocio_id (FK)   │◄── NUEVO
│ dia_semana (0-6)  │  │    │ fecha_excepcion   │
│ hora_apertura     │  │    │ motivo            │
│ hora_cierre       │  │    │ creado_en         │
│ activo            │  │    └───────────────────┘
│ creado_en         │  │
│ actualizado_en    │  │    ┌───────────────────┐
└───────────────────┘  │    │claves_idempotencia│
                       │    ├───────────────────┤
                       │    │ id (PK)           │
                       │    │ clave_idempotencia│
                       │    │ hash_solicitud    │
                       │    │ estado_respuesta  │
                       │    │ cuerpo_respuesta  │
                       │    │ creado_en         │
                       │    │ expira_en         │
                       │    └───────────────────┘
                       │
                       └──── Todos apuntan a negocios.id
```

---

**Fin del Documento de Diseño**
