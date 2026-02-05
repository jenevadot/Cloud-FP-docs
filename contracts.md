# Contratos de API - Agenda Salon MVP

## Informacion del Documento
- **Version:** 2.0
- **Fecha:** 2026-02-05
- **Base URL:** `/api/v1`
- **Formato:** JSON
- **Autenticacion:** JWT Bearer Token
- **Arquitectura:** Multi-Tenant (soporte para multiples negocios)

---

## 1. Informacion General

### 1.1 Arquitectura Multi-Tenant

El sistema soporta multiples negocios. Cada usuario puede:
- Crear y administrar uno o mas negocios (rol `dueno_negocio`)
- Reservar citas en cualquier negocio (rol `cliente`)

Los recursos (servicios, personal, horarios, citas) pertenecen a un negocio especifico y se acceden bajo el contexto del negocio.

### 1.2 Encabezados Comunes

**Request Headers:**
```
Content-Type: application/json
Authorization: Bearer <jwt_token>  # Solo endpoints protegidos
Idempotency-Key: <uuid>            # Solo POST /negocios/{negocio_id}/citas
```

**Response Headers:**
```
Content-Type: application/json
X-Request-Id: <uuid>
```

### 1.3 Formato de Respuesta de Error

Todas las respuestas de error siguen este formato:

```json
{
  "error": "CODIGO_ERROR",
  "mensaje": "Descripcion legible para el usuario",
  "detalles": {},
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

**Codigos de Error:**
| Codigo | HTTP | Descripcion |
|--------|------|-------------|
| `ERROR_VALIDACION` | 422 | Datos de entrada invalidos |
| `SLOT_NO_DISPONIBLE` | 409 | Espacio de tiempo ya reservado |
| `NO_AUTORIZADO` | 401 | Token invalido o ausente |
| `PROHIBIDO` | 403 | Permisos insuficientes |
| `NO_ENCONTRADO` | 404 | Recurso no existe |
| `CLAVE_IDEMPOTENCIA_DUPLICADA` | 409 | Solicitud duplicada detectada |
| `ERROR_INTERNO` | 500 | Error inesperado del servidor |

### 1.4 Tipos de Datos Comunes

```typescript
// UUID v4
type UUID = string; // Formato: "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx"

// Timestamps en formato ISO 8601 con zona horaria Lima (UTC-5)
type Timestamp = string; // Formato: "2026-02-04T14:30:00-05:00"

// Fecha sin hora
type Date = string; // Formato: "2026-02-04"

// Hora sin fecha
type Time = string; // Formato: "14:30"

// Moneda peruana
type PEN = number; // Decimal con 2 posiciones, ej: 45.00

// Roles de usuario
type RolUsuario = "cliente" | "dueno_negocio";

// Estados de cita
type EstadoCita = "pendiente" | "confirmada" | "pendiente_actualizacion" | "cancelada" | "completada";
```

---

## 2. Autenticacion (`/auth`)

### 2.1 Registrar Usuario

Crea una nueva cuenta de usuario.

**Endpoint:** `POST /api/v1/auth/registrar`

**Autenticacion:** No requerida

**Request Body:**
```json
{
  "email": "usuario@ejemplo.com",
  "contrasena": "MiContrasena123",
  "nombre_completo": "Juan Perez Garcia",
  "telefono": "912345678"
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `email` | Requerido, formato RFC 5322, unico en sistema |
| `contrasena` | Requerido, min 8 caracteres, 1 mayuscula, 1 minuscula, 1 numero |
| `nombre_completo` | Requerido, min 2 caracteres, max 255 |
| `telefono` | Requerido, formato peruano: `^9\d{8}$` (9 digitos empezando con 9) |

**Response 201 Created:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "usuario@ejemplo.com",
  "nombre_completo": "Juan Perez Garcia",
  "telefono": "912345678",
  "rol": "cliente",
  "creado_en": "2026-02-04T10:30:00-05:00"
}
```

**Response 409 Conflict (Email duplicado):**
```json
{
  "error": "EMAIL_DUPLICADO",
  "mensaje": "Ya existe una cuenta con este email",
  "detalles": {
    "email": "usuario@ejemplo.com"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

**Response 422 Unprocessable Entity:**
```json
{
  "error": "ERROR_VALIDACION",
  "mensaje": "Los datos proporcionados no son validos",
  "detalles": {
    "contrasena": "Debe contener al menos 8 caracteres, una mayuscula, una minuscula y un numero",
    "telefono": "Formato invalido. Debe ser un numero de 9 digitos empezando con 9"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 2.2 Login

Autentica un usuario y retorna un token JWT.

**Endpoint:** `POST /api/v1/auth/login`

**Autenticacion:** No requerida

**Request Body:**
```json
{
  "email": "usuario@ejemplo.com",
  "contrasena": "MiContrasena123"
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `email` | Requerido, formato email valido |
| `contrasena` | Requerido |

**Response 200 OK:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 86400,
  "usuario": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "usuario@ejemplo.com",
    "nombre_completo": "Juan Perez Garcia",
    "telefono": "912345678",
    "rol": "cliente"
  }
}
```

**JWT Payload:**
```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "email": "usuario@ejemplo.com",
  "rol": "cliente",
  "exp": 1738713600,
  "iat": 1738627200
}
```

**Response 401 Unauthorized:**
```json
{
  "error": "CREDENCIALES_INVALIDAS",
  "mensaje": "Email o contrasena incorrectos",
  "detalles": {},
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

## 3. Negocios (`/negocios`)

### 3.1 Crear Negocio

Crea un nuevo negocio. El usuario autenticado se convierte en el dueno.

**Endpoint:** `POST /api/v1/negocios`

**Autenticacion:** Requerida

**Request Body:**
```json
{
  "nombre": "Salon Bella Vista",
  "slug": "bella-vista"
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `nombre` | Requerido, max 255 caracteres |
| `slug` | Requerido, max 100 caracteres, unico, solo letras minusculas, numeros y guiones |

**Response 201 Created:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440050",
  "nombre": "Salon Bella Vista",
  "slug": "bella-vista",
  "dueno": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "nombre_completo": "Juan Perez Garcia"
  },
  "activo": true,
  "creado_en": "2026-02-04T10:30:00-05:00",
  "actualizado_en": "2026-02-04T10:30:00-05:00"
}
```

**Response 409 Conflict (Slug duplicado):**
```json
{
  "error": "SLUG_DUPLICADO",
  "mensaje": "Ya existe un negocio con este identificador",
  "detalles": {
    "slug": "bella-vista"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 3.2 Listar Mis Negocios

Retorna los negocios del usuario autenticado.

**Endpoint:** `GET /api/v1/negocios`

**Autenticacion:** Requerida

**Response 200 OK:**
```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440050",
      "nombre": "Salon Bella Vista",
      "slug": "bella-vista",
      "activo": true,
      "creado_en": "2026-02-04T10:30:00-05:00"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440051",
      "nombre": "Spa Relax",
      "slug": "spa-relax",
      "activo": true,
      "creado_en": "2026-02-05T09:00:00-05:00"
    }
  ],
  "total": 2
}
```

---

### 3.3 Obtener Negocio (Publico)

Retorna informacion publica de un negocio.

**Endpoint:** `GET /api/v1/negocios/{negocio_id}`

**Autenticacion:** No requerida

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440050",
  "nombre": "Salon Bella Vista",
  "slug": "bella-vista",
  "activo": true,
  "creado_en": "2026-02-04T10:30:00-05:00"
}
```

**Response 404 Not Found:**
```json
{
  "error": "NO_ENCONTRADO",
  "mensaje": "Negocio no encontrado",
  "detalles": {
    "negocio_id": "550e8400-e29b-41d4-a716-446655440099"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 3.4 Actualizar Negocio

Actualiza informacion de un negocio.

**Endpoint:** `PUT /api/v1/negocios/{negocio_id}`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Request Body:**
```json
{
  "nombre": "Salon Bella Vista Premium",
  "slug": "bella-vista-premium"
}
```

**Validaciones:** Mismas que crear negocio. Todos los campos son opcionales.

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440050",
  "nombre": "Salon Bella Vista Premium",
  "slug": "bella-vista-premium",
  "dueno": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "nombre_completo": "Juan Perez Garcia"
  },
  "activo": true,
  "creado_en": "2026-02-04T10:30:00-05:00",
  "actualizado_en": "2026-02-04T11:00:00-05:00"
}
```

**Response 403 Forbidden:**
```json
{
  "error": "PROHIBIDO",
  "mensaje": "No tienes permisos para modificar este negocio",
  "detalles": {
    "negocio_id": "550e8400-e29b-41d4-a716-446655440050"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

## 4. Servicios (`/negocios/{negocio_id}/servicios`)

### 4.1 Listar Servicios

Retorna todos los servicios activos de un negocio.

**Endpoint:** `GET /api/v1/negocios/{negocio_id}/servicios`

**Autenticacion:** No requerida

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Query Parameters:**
| Parametro | Tipo | Requerido | Descripcion |
|-----------|------|-----------|-------------|
| `pagina` | integer | No | Numero de pagina (default: 1) |
| `por_pagina` | integer | No | Items por pagina (default: 20, max: 100) |

**Response 200 OK:**
```json
{
  "negocio": {
    "id": "550e8400-e29b-41d4-a716-446655440050",
    "nombre": "Salon Bella Vista"
  },
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440001",
      "nombre": "Corte de Cabello",
      "duracion_minutos": 45,
      "precio_pen": 35.00,
      "personal": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440010",
          "nombre": "Maria Lopez"
        },
        {
          "id": "550e8400-e29b-41d4-a716-446655440011",
          "nombre": "Carlos Sanchez"
        }
      ]
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440002",
      "nombre": "Manicure",
      "duracion_minutos": 60,
      "precio_pen": 45.00,
      "personal": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440012",
          "nombre": "Ana Torres"
        }
      ]
    }
  ],
  "total": 15,
  "pagina": 1,
  "por_pagina": 20,
  "total_paginas": 1
}
```

---

### 4.2 Obtener Servicio

Retorna detalles de un servicio especifico.

**Endpoint:** `GET /api/v1/negocios/{negocio_id}/servicios/{servicio_id}`

**Autenticacion:** No requerida

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `servicio_id` | UUID | ID del servicio |

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
  "nombre": "Corte de Cabello",
  "duracion_minutos": 45,
  "precio_pen": 35.00,
  "activo": true,
  "personal": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440010",
      "nombre": "Maria Lopez"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440011",
      "nombre": "Carlos Sanchez"
    }
  ],
  "creado_en": "2026-01-15T09:00:00-05:00",
  "actualizado_en": "2026-01-20T14:30:00-05:00"
}
```

**Response 404 Not Found:**
```json
{
  "error": "NO_ENCONTRADO",
  "mensaje": "Servicio no encontrado",
  "detalles": {
    "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
    "servicio_id": "550e8400-e29b-41d4-a716-446655440099"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 4.3 Crear Servicio

Crea un nuevo servicio en el catalogo del negocio.

**Endpoint:** `POST /api/v1/negocios/{negocio_id}/servicios`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Request Body:**
```json
{
  "nombre": "Tinte de Cabello",
  "duracion_minutos": 120,
  "precio_pen": 150.00,
  "personal_ids": [
    "550e8400-e29b-41d4-a716-446655440010",
    "550e8400-e29b-41d4-a716-446655440011"
  ]
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `nombre` | Requerido, max 255 caracteres |
| `duracion_minutos` | Requerido, entre 15 y 480 |
| `precio_pen` | Requerido, mayor que 0 |
| `personal_ids` | Opcional, lista de UUIDs validos de personal activo del mismo negocio |

**Response 201 Created:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440003",
  "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
  "nombre": "Tinte de Cabello",
  "duracion_minutos": 120,
  "precio_pen": 150.00,
  "activo": true,
  "personal": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440010",
      "nombre": "Maria Lopez"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440011",
      "nombre": "Carlos Sanchez"
    }
  ],
  "creado_en": "2026-02-04T10:30:00-05:00",
  "actualizado_en": "2026-02-04T10:30:00-05:00"
}
```

**Response 403 Forbidden:**
```json
{
  "error": "PROHIBIDO",
  "mensaje": "No tienes permisos para crear servicios en este negocio",
  "detalles": {
    "negocio_id": "550e8400-e29b-41d4-a716-446655440050"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 4.4 Actualizar Servicio

Actualiza un servicio existente.

**Endpoint:** `PUT /api/v1/negocios/{negocio_id}/servicios/{servicio_id}`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `servicio_id` | UUID | ID del servicio a actualizar |

**Request Body:**
```json
{
  "nombre": "Tinte de Cabello Premium",
  "duracion_minutos": 150,
  "precio_pen": 180.00,
  "personal_ids": [
    "550e8400-e29b-41d4-a716-446655440010"
  ]
}
```

**Validaciones:** Mismas que crear servicio. Todos los campos son opcionales.

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440003",
  "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
  "nombre": "Tinte de Cabello Premium",
  "duracion_minutos": 150,
  "precio_pen": 180.00,
  "activo": true,
  "personal": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440010",
      "nombre": "Maria Lopez"
    }
  ],
  "creado_en": "2026-02-04T10:30:00-05:00",
  "actualizado_en": "2026-02-04T11:00:00-05:00"
}
```

---

### 4.5 Desactivar Servicio

Desactiva un servicio (soft delete).

**Endpoint:** `DELETE /api/v1/negocios/{negocio_id}/servicios/{servicio_id}`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `servicio_id` | UUID | ID del servicio a desactivar |

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440003",
  "mensaje": "Servicio desactivado exitosamente"
}
```

---

## 5. Personal (`/negocios/{negocio_id}/personal`)

### 5.1 Listar Personal

Retorna todo el personal activo del negocio.

**Endpoint:** `GET /api/v1/negocios/{negocio_id}/personal`

**Autenticacion:** No requerida

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Query Parameters:**
| Parametro | Tipo | Requerido | Descripcion |
|-----------|------|-----------|-------------|
| `servicio_id` | UUID | No | Filtrar por personal calificado para un servicio |

**Response 200 OK:**
```json
{
  "negocio": {
    "id": "550e8400-e29b-41d4-a716-446655440050",
    "nombre": "Salon Bella Vista"
  },
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440010",
      "nombre": "Maria Lopez",
      "servicios": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440001",
          "nombre": "Corte de Cabello"
        },
        {
          "id": "550e8400-e29b-41d4-a716-446655440003",
          "nombre": "Tinte de Cabello"
        }
      ]
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440011",
      "nombre": "Carlos Sanchez",
      "servicios": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440001",
          "nombre": "Corte de Cabello"
        }
      ]
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440012",
      "nombre": "Ana Torres",
      "servicios": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440002",
          "nombre": "Manicure"
        }
      ]
    }
  ],
  "total": 3
}
```

---

### 5.2 Crear Personal

Crea un nuevo miembro del personal en el negocio.

**Endpoint:** `POST /api/v1/negocios/{negocio_id}/personal`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Request Body:**
```json
{
  "nombre": "Laura Martinez",
  "email": "laura@salon.com"
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `nombre` | Requerido, max 255 caracteres |
| `email` | Requerido, formato email valido, unico dentro del negocio |

**Response 201 Created:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440013",
  "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
  "nombre": "Laura Martinez",
  "email": "laura@salon.com",
  "activo": true,
  "servicios": [],
  "creado_en": "2026-02-04T10:30:00-05:00",
  "actualizado_en": "2026-02-04T10:30:00-05:00"
}
```

---

### 5.3 Actualizar Personal

Actualiza informacion de un miembro del personal.

**Endpoint:** `PUT /api/v1/negocios/{negocio_id}/personal/{personal_id}`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `personal_id` | UUID | ID del personal a actualizar |

**Request Body:**
```json
{
  "nombre": "Laura Martinez Gomez",
  "email": "laura.martinez@salon.com"
}
```

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440013",
  "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
  "nombre": "Laura Martinez Gomez",
  "email": "laura.martinez@salon.com",
  "activo": true,
  "servicios": [],
  "creado_en": "2026-02-04T10:30:00-05:00",
  "actualizado_en": "2026-02-04T11:00:00-05:00"
}
```

---

### 5.4 Desactivar Personal

Desactiva un miembro del personal (soft delete).

**Endpoint:** `DELETE /api/v1/negocios/{negocio_id}/personal/{personal_id}`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `personal_id` | UUID | ID del personal a desactivar |

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440013",
  "mensaje": "Personal desactivado exitosamente"
}
```

---

## 6. Disponibilidad (`/negocios/{negocio_id}/disponibilidad`)

### 6.1 Consultar Disponibilidad

Retorna espacios de tiempo disponibles para un servicio en un rango de fechas.

**Endpoint:** `GET /api/v1/negocios/{negocio_id}/disponibilidad`

**Autenticacion:** No requerida

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Query Parameters:**
| Parametro | Tipo | Requerido | Descripcion |
|-----------|------|-----------|-------------|
| `servicio_id` | UUID | Si | ID del servicio |
| `fecha_inicio` | Date | Si | Fecha de inicio del rango (YYYY-MM-DD) |
| `fecha_fin` | Date | Si | Fecha de fin del rango (YYYY-MM-DD) |
| `personal_id` | UUID | No | Filtrar por personal especifico |

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `servicio_id` | UUID valido de servicio activo del negocio |
| `fecha_inicio` | Fecha >= hoy |
| `fecha_fin` | Fecha <= fecha_inicio + 60 dias |
| `personal_id` | UUID valido de personal activo del negocio y calificado para el servicio |

**Response 200 OK:**
```json
{
  "negocio": {
    "id": "550e8400-e29b-41d4-a716-446655440050",
    "nombre": "Salon Bella Vista"
  },
  "servicio": {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "nombre": "Corte de Cabello",
    "duracion_minutos": 45
  },
  "fecha_inicio": "2026-02-10",
  "fecha_fin": "2026-02-17",
  "disponibilidad": [
    {
      "fecha": "2026-02-10",
      "dia_semana": "Lunes",
      "slots": [
        {
          "hora_inicio": "09:00",
          "hora_fin": "09:45",
          "personal_disponible": [
            {
              "id": "550e8400-e29b-41d4-a716-446655440010",
              "nombre": "Maria Lopez"
            },
            {
              "id": "550e8400-e29b-41d4-a716-446655440011",
              "nombre": "Carlos Sanchez"
            }
          ]
        },
        {
          "hora_inicio": "09:30",
          "hora_fin": "10:15",
          "personal_disponible": [
            {
              "id": "550e8400-e29b-41d4-a716-446655440011",
              "nombre": "Carlos Sanchez"
            }
          ]
        },
        {
          "hora_inicio": "10:00",
          "hora_fin": "10:45",
          "personal_disponible": [
            {
              "id": "550e8400-e29b-41d4-a716-446655440010",
              "nombre": "Maria Lopez"
            },
            {
              "id": "550e8400-e29b-41d4-a716-446655440011",
              "nombre": "Carlos Sanchez"
            }
          ]
        }
      ],
      "slots_agrupados": {
        "manana": [
          {
            "hora_inicio": "09:00",
            "hora_fin": "09:45",
            "cantidad_personal": 2
          },
          {
            "hora_inicio": "09:30",
            "hora_fin": "10:15",
            "cantidad_personal": 1
          },
          {
            "hora_inicio": "10:00",
            "hora_fin": "10:45",
            "cantidad_personal": 2
          }
        ],
        "mediodia": [
          {
            "hora_inicio": "12:00",
            "hora_fin": "12:45",
            "cantidad_personal": 2
          }
        ],
        "tarde": [
          {
            "hora_inicio": "16:00",
            "hora_fin": "16:45",
            "cantidad_personal": 1
          }
        ]
      }
    },
    {
      "fecha": "2026-02-11",
      "dia_semana": "Martes",
      "slots": [],
      "slots_agrupados": {
        "manana": [],
        "mediodia": [],
        "tarde": []
      }
    }
  ],
  "fechas_sin_disponibilidad": ["2026-02-11", "2026-02-15"]
}
```

**Nota sobre agrupacion de slots:**
- `manana`: 06:00 - 11:59
- `mediodia`: 12:00 - 15:59
- `tarde`: 16:00 - 20:00

**Response 422 Unprocessable Entity:**
```json
{
  "error": "ERROR_VALIDACION",
  "mensaje": "Los parametros de busqueda no son validos",
  "detalles": {
    "fecha_inicio": "La fecha no puede ser anterior a hoy",
    "fecha_fin": "El rango maximo es de 60 dias"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

## 7. Citas (`/negocios/{negocio_id}/citas`)

### 7.1 Crear Cita

Crea una nueva reserva de cita en un negocio.

**Endpoint:** `POST /api/v1/negocios/{negocio_id}/citas`

**Autenticacion:** Requerida (rol: `cliente`)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Headers Requeridos:**
```
Authorization: Bearer <jwt_token>
Idempotency-Key: <uuid_v4>
```

**Request Body:**
```json
{
  "servicio_id": "550e8400-e29b-41d4-a716-446655440001",
  "personal_id": "550e8400-e29b-41d4-a716-446655440010",
  "fecha_hora_cita": "2026-02-10T09:00:00-05:00",
  "cliente_nombre": "Juan Perez Garcia",
  "cliente_email": "juan@ejemplo.com",
  "cliente_telefono": "912345678"
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `servicio_id` | Requerido, UUID de servicio activo del negocio |
| `personal_id` | Opcional, UUID de personal activo del negocio y calificado para el servicio. Si es null, se asigna automaticamente |
| `fecha_hora_cita` | Requerido, timestamp futuro, dentro de horario del negocio |
| `cliente_nombre` | Requerido, min 2 caracteres |
| `cliente_email` | Requerido, formato email valido |
| `cliente_telefono` | Requerido, formato peruano: `^9\d{8}$` |

**Response 201 Created:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440100",
  "negocio": {
    "id": "550e8400-e29b-41d4-a716-446655440050",
    "nombre": "Salon Bella Vista"
  },
  "servicio": {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "nombre": "Corte de Cabello",
    "duracion_minutos": 45,
    "precio_pen": 35.00
  },
  "personal": {
    "id": "550e8400-e29b-41d4-a716-446655440010",
    "nombre": "Maria Lopez"
  },
  "fecha_hora_inicio": "2026-02-10T09:00:00-05:00",
  "fecha_hora_fin": "2026-02-10T09:45:00-05:00",
  "cliente_nombre": "Juan Perez Garcia",
  "cliente_email": "juan@ejemplo.com",
  "cliente_telefono": "912345678",
  "estado": "confirmada",
  "numero_confirmacion": "A7X9B2K4M1P3",
  "creado_en": "2026-02-04T10:30:00-05:00"
}
```

**Response 409 Conflict (Slot no disponible):**
```json
{
  "error": "SLOT_NO_DISPONIBLE",
  "mensaje": "El espacio de tiempo seleccionado ya no esta disponible",
  "detalles": {
    "fecha_hora_solicitada": "2026-02-10T09:00:00-05:00",
    "personal_id": "550e8400-e29b-41d4-a716-446655440010"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

**Response 409 Conflict (Idempotencia duplicada):**
```json
{
  "error": "CLAVE_IDEMPOTENCIA_DUPLICADA",
  "mensaje": "Esta solicitud ya fue procesada anteriormente",
  "detalles": {
    "clave_idempotencia": "123e4567-e89b-12d3-a456-426614174000",
    "respuesta_original": {
      "id": "550e8400-e29b-41d4-a716-446655440100",
      "numero_confirmacion": "A7X9B2K4M1P3"
    }
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 7.2 Obtener Detalle de Cita

Retorna informacion detallada de una cita.

**Endpoint:** `GET /api/v1/negocios/{negocio_id}/citas/{cita_id}`

**Autenticacion:** Requerida (cliente dueno de la cita o dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `cita_id` | UUID | ID de la cita |

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440100",
  "negocio": {
    "id": "550e8400-e29b-41d4-a716-446655440050",
    "nombre": "Salon Bella Vista"
  },
  "usuario_id": "550e8400-e29b-41d4-a716-446655440000",
  "servicio": {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "nombre": "Corte de Cabello",
    "duracion_minutos": 45,
    "precio_pen": 35.00
  },
  "personal": {
    "id": "550e8400-e29b-41d4-a716-446655440010",
    "nombre": "Maria Lopez",
    "email": "maria@salon.com"
  },
  "fecha_hora_inicio": "2026-02-10T09:00:00-05:00",
  "fecha_hora_fin": "2026-02-10T09:45:00-05:00",
  "cliente_nombre": "Juan Perez Garcia",
  "cliente_email": "juan@ejemplo.com",
  "cliente_telefono": "912345678",
  "estado": "confirmada",
  "numero_confirmacion": "A7X9B2K4M1P3",
  "puede_editar": true,
  "puede_cancelar": true,
  "creado_en": "2026-02-04T10:30:00-05:00",
  "actualizado_en": "2026-02-04T10:30:00-05:00"
}
```

**Response 404 Not Found:**
```json
{
  "error": "NO_ENCONTRADO",
  "mensaje": "Cita no encontrada",
  "detalles": {
    "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
    "cita_id": "550e8400-e29b-41d4-a716-446655440999"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 7.3 Editar Cita (Iniciar Edicion)

Inicia el proceso de edicion de una cita existente.

**Endpoint:** `PUT /api/v1/negocios/{negocio_id}/citas/{cita_id}`

**Autenticacion:** Requerida (cliente dueno de la cita)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `cita_id` | UUID | ID de la cita a editar |

**Request Body:**
```json
{
  "fecha_hora_cita": "2026-02-11T10:00:00-05:00",
  "personal_id": "550e8400-e29b-41d4-a716-446655440011"
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `fecha_hora_cita` | Requerido, timestamp futuro, dentro de horario del negocio |
| `personal_id` | Opcional, UUID de personal activo del negocio y calificado |

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440100",
  "estado": "pendiente_actualizacion",
  "fecha_hora_original": "2026-02-10T09:00:00-05:00",
  "fecha_hora_nueva": "2026-02-11T10:00:00-05:00",
  "personal_original": {
    "id": "550e8400-e29b-41d4-a716-446655440010",
    "nombre": "Maria Lopez"
  },
  "personal_nuevo": {
    "id": "550e8400-e29b-41d4-a716-446655440011",
    "nombre": "Carlos Sanchez"
  },
  "mensaje": "Cita en proceso de edicion. Confirme el nuevo horario para completar.",
  "expira_en": "2026-02-04T11:00:00-05:00"
}
```

**Response 409 Conflict (Nuevo slot no disponible):**
```json
{
  "error": "SLOT_NO_DISPONIBLE",
  "mensaje": "El nuevo espacio de tiempo no esta disponible",
  "detalles": {
    "fecha_hora_solicitada": "2026-02-11T10:00:00-05:00"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 7.4 Confirmar Edicion de Cita

Confirma el nuevo horario de una cita en edicion.

**Endpoint:** `PUT /api/v1/negocios/{negocio_id}/citas/{cita_id}/confirmar`

**Autenticacion:** Requerida (cliente dueno de la cita)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `cita_id` | UUID | ID de la cita |

**Precondiciones:**
- La cita debe estar en estado `pendiente_actualizacion`
- La edicion no debe haber expirado (30 minutos)

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440100",
  "negocio": {
    "id": "550e8400-e29b-41d4-a716-446655440050",
    "nombre": "Salon Bella Vista"
  },
  "servicio": {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "nombre": "Corte de Cabello",
    "duracion_minutos": 45,
    "precio_pen": 35.00
  },
  "personal": {
    "id": "550e8400-e29b-41d4-a716-446655440011",
    "nombre": "Carlos Sanchez"
  },
  "fecha_hora_inicio": "2026-02-11T10:00:00-05:00",
  "fecha_hora_fin": "2026-02-11T10:45:00-05:00",
  "estado": "confirmada",
  "numero_confirmacion": "A7X9B2K4M1P3",
  "mensaje": "Cita actualizada exitosamente"
}
```

**Response 400 Bad Request (Edicion expirada):**
```json
{
  "error": "EDICION_EXPIRADA",
  "mensaje": "El tiempo para confirmar la edicion ha expirado. La cita ha sido revertida al horario original.",
  "detalles": {
    "estado_actual": "confirmada",
    "fecha_hora_actual": "2026-02-10T09:00:00-05:00"
  },
  "timestamp": "2026-02-04T11:30:00-05:00"
}
```

---

### 7.5 Cancelar Edicion de Cita

Cancela el proceso de edicion y revierte al horario original.

**Endpoint:** `DELETE /api/v1/negocios/{negocio_id}/citas/{cita_id}/edicion`

**Autenticacion:** Requerida (cliente dueno de la cita)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `cita_id` | UUID | ID de la cita |

**Precondiciones:**
- La cita debe estar en estado `pendiente_actualizacion`

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440100",
  "estado": "confirmada",
  "fecha_hora_inicio": "2026-02-10T09:00:00-05:00",
  "fecha_hora_fin": "2026-02-10T09:45:00-05:00",
  "mensaje": "Edicion cancelada. La cita mantiene su horario original."
}
```

---

### 7.6 Cancelar Cita

Cancela una cita existente.

**Endpoint:** `DELETE /api/v1/negocios/{negocio_id}/citas/{cita_id}`

**Autenticacion:** Requerida (cliente dueno de la cita o dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `cita_id` | UUID | ID de la cita a cancelar |

**Precondiciones:**
- La cita debe estar en estado `pendiente` o `confirmada`
- La fecha de la cita debe ser al menos 2 horas en el futuro

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440100",
  "estado": "cancelada",
  "mensaje": "Cita cancelada exitosamente"
}
```

**Response 400 Bad Request (Muy cerca de la hora):**
```json
{
  "error": "CANCELACION_TARDIA",
  "mensaje": "No es posible cancelar citas con menos de 2 horas de anticipacion",
  "detalles": {
    "fecha_hora_cita": "2026-02-04T12:00:00-05:00",
    "hora_limite_cancelacion": "2026-02-04T10:00:00-05:00",
    "hora_actual": "2026-02-04T10:30:00-05:00"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

## 8. Citas del Usuario (`/citas/me`)

### 8.1 Listar Mis Citas (Cross-Negocio)

Retorna todas las citas del usuario autenticado en todos los negocios.

**Endpoint:** `GET /api/v1/citas/me`

**Autenticacion:** Requerida (rol: `cliente`)

**Query Parameters:**
| Parametro | Tipo | Requerido | Descripcion |
|-----------|------|-----------|-------------|
| `estado` | string | No | Filtrar por estado: `pendiente`, `confirmada`, `cancelada`, `completada` |
| `tipo` | string | No | `proximas` (futuras), `pasadas`, `todas` (default: `todas`) |
| `pagina` | integer | No | Numero de pagina (default: 1) |
| `por_pagina` | integer | No | Items por pagina (default: 20, max: 100) |

**Response 200 OK:**
```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440100",
      "negocio": {
        "id": "550e8400-e29b-41d4-a716-446655440050",
        "nombre": "Salon Bella Vista"
      },
      "servicio": {
        "id": "550e8400-e29b-41d4-a716-446655440001",
        "nombre": "Corte de Cabello",
        "duracion_minutos": 45,
        "precio_pen": 35.00
      },
      "personal": {
        "id": "550e8400-e29b-41d4-a716-446655440010",
        "nombre": "Maria Lopez"
      },
      "fecha_hora_inicio": "2026-02-10T09:00:00-05:00",
      "fecha_hora_fin": "2026-02-10T09:45:00-05:00",
      "estado": "confirmada",
      "numero_confirmacion": "A7X9B2K4M1P3",
      "puede_editar": true,
      "puede_cancelar": true
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440101",
      "negocio": {
        "id": "550e8400-e29b-41d4-a716-446655440051",
        "nombre": "Spa Relax"
      },
      "servicio": {
        "id": "550e8400-e29b-41d4-a716-446655440020",
        "nombre": "Masaje Relajante",
        "duracion_minutos": 60,
        "precio_pen": 80.00
      },
      "personal": {
        "id": "550e8400-e29b-41d4-a716-446655440030",
        "nombre": "Pedro Gomez"
      },
      "fecha_hora_inicio": "2026-02-15T14:00:00-05:00",
      "fecha_hora_fin": "2026-02-15T15:00:00-05:00",
      "estado": "confirmada",
      "numero_confirmacion": "B8Y0C3L5N2Q4",
      "puede_editar": true,
      "puede_cancelar": true
    }
  ],
  "total": 2,
  "pagina": 1,
  "por_pagina": 20,
  "total_paginas": 1
}
```

---

## 9. Configuracion del Negocio (`/negocios/{negocio_id}/horarios`, `/negocios/{negocio_id}/excepciones`)

### 9.1 Obtener Horarios del Negocio

Retorna los horarios de atencion configurados.

**Endpoint:** `GET /api/v1/negocios/{negocio_id}/horarios`

**Autenticacion:** No requerida

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Response 200 OK:**
```json
{
  "negocio": {
    "id": "550e8400-e29b-41d4-a716-446655440050",
    "nombre": "Salon Bella Vista"
  },
  "horarios": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440200",
      "dia_semana": 1,
      "dia_nombre": "Lunes",
      "hora_apertura": "09:00",
      "hora_cierre": "19:00",
      "activo": true
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440201",
      "dia_semana": 2,
      "dia_nombre": "Martes",
      "hora_apertura": "09:00",
      "hora_cierre": "19:00",
      "activo": true
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440202",
      "dia_semana": 3,
      "dia_nombre": "Miercoles",
      "hora_apertura": "09:00",
      "hora_cierre": "19:00",
      "activo": true
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440203",
      "dia_semana": 4,
      "dia_nombre": "Jueves",
      "hora_apertura": "09:00",
      "hora_cierre": "19:00",
      "activo": true
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440204",
      "dia_semana": 5,
      "dia_nombre": "Viernes",
      "hora_apertura": "09:00",
      "hora_cierre": "19:00",
      "activo": true
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440205",
      "dia_semana": 6,
      "dia_nombre": "Sabado",
      "hora_apertura": "09:00",
      "hora_cierre": "17:00",
      "activo": true
    },
    {
      "dia_semana": 0,
      "dia_nombre": "Domingo",
      "activo": false
    }
  ],
  "zona_horaria": "America/Lima"
}
```

---

### 9.2 Configurar Horarios del Negocio

Crea o actualiza los horarios de atencion.

**Endpoint:** `POST /api/v1/negocios/{negocio_id}/horarios`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Request Body:**
```json
{
  "horarios": [
    {
      "dia_semana": 1,
      "hora_apertura": "08:00",
      "hora_cierre": "20:00",
      "activo": true
    },
    {
      "dia_semana": 0,
      "activo": false
    }
  ]
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `dia_semana` | Requerido, entre 0 (Domingo) y 6 (Sabado) |
| `hora_apertura` | Requerido si activo=true, formato HH:MM |
| `hora_cierre` | Requerido si activo=true, formato HH:MM, mayor que hora_apertura |
| `activo` | Opcional, default true |

**Response 200 OK:**
```json
{
  "mensaje": "Horarios actualizados exitosamente",
  "horarios_actualizados": 2
}
```

---

### 9.3 Listar Fechas de Excepcion

Retorna las fechas de excepcion (dias no laborables) del negocio.

**Endpoint:** `GET /api/v1/negocios/{negocio_id}/excepciones`

**Autenticacion:** No requerida

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Query Parameters:**
| Parametro | Tipo | Requerido | Descripcion |
|-----------|------|-----------|-------------|
| `desde` | Date | No | Filtrar desde esta fecha (default: hoy) |
| `hasta` | Date | No | Filtrar hasta esta fecha (default: hoy + 1 ano) |

**Response 200 OK:**
```json
{
  "negocio": {
    "id": "550e8400-e29b-41d4-a716-446655440050",
    "nombre": "Salon Bella Vista"
  },
  "excepciones": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440300",
      "fecha_excepcion": "2026-03-30",
      "motivo": "Semana Santa - Viernes Santo",
      "creado_en": "2026-01-15T09:00:00-05:00"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440301",
      "fecha_excepcion": "2026-05-01",
      "motivo": "Dia del Trabajo",
      "creado_en": "2026-01-15T09:00:00-05:00"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440302",
      "fecha_excepcion": "2026-07-28",
      "motivo": "Fiestas Patrias",
      "creado_en": "2026-01-15T09:00:00-05:00"
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440303",
      "fecha_excepcion": "2026-07-29",
      "motivo": "Fiestas Patrias",
      "creado_en": "2026-01-15T09:00:00-05:00"
    }
  ],
  "total": 4
}
```

---

### 9.4 Crear Fecha de Excepcion

Registra una nueva fecha de excepcion (dia no laborable).

**Endpoint:** `POST /api/v1/negocios/{negocio_id}/excepciones`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |

**Request Body:**
```json
{
  "fecha_excepcion": "2026-12-25",
  "motivo": "Navidad"
}
```

**Validaciones:**
| Campo | Reglas |
|-------|--------|
| `fecha_excepcion` | Requerido, formato YYYY-MM-DD, fecha futura, unica por negocio |
| `motivo` | Requerido, max 255 caracteres |

**Response 201 Created:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440304",
  "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
  "fecha_excepcion": "2026-12-25",
  "motivo": "Navidad",
  "creado_en": "2026-02-04T10:30:00-05:00"
}
```

**Response 409 Conflict (Fecha duplicada):**
```json
{
  "error": "FECHA_DUPLICADA",
  "mensaje": "Ya existe una excepcion para esta fecha en este negocio",
  "detalles": {
    "negocio_id": "550e8400-e29b-41d4-a716-446655440050",
    "fecha_excepcion": "2026-12-25"
  },
  "timestamp": "2026-02-04T10:30:00-05:00"
}
```

---

### 9.5 Eliminar Fecha de Excepcion

Elimina una fecha de excepcion.

**Endpoint:** `DELETE /api/v1/negocios/{negocio_id}/excepciones/{excepcion_id}`

**Autenticacion:** Requerida (dueno del negocio)

**Path Parameters:**
| Parametro | Tipo | Descripcion |
|-----------|------|-------------|
| `negocio_id` | UUID | ID del negocio |
| `excepcion_id` | UUID | ID de la excepcion a eliminar |

**Response 200 OK:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440304",
  "mensaje": "Fecha de excepcion eliminada exitosamente"
}
```

---

## 10. Health Check

### 10.1 Verificacion de Salud

Endpoint para monitoreo de salud del servicio.

**Endpoint:** `GET /health`

**Autenticacion:** No requerida

**Response 200 OK:**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "timestamp": "2026-02-04T10:30:00-05:00",
  "dependencies": {
    "database": "healthy",
    "cache": "healthy"
  }
}
```

**Response 503 Service Unavailable:**
```json
{
  "status": "unhealthy",
  "version": "1.0.0",
  "timestamp": "2026-02-04T10:30:00-05:00",
  "dependencies": {
    "database": "unhealthy",
    "cache": "healthy"
  },
  "error": "Database connection failed"
}
```

---

## 11. Resumen de Endpoints

### Autenticacion
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| POST | `/api/v1/auth/registrar` | No | - | Registro de usuario |
| POST | `/api/v1/auth/login` | No | - | Login |

### Negocios
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| POST | `/api/v1/negocios` | Si | Usuario | Crear negocio (creador = dueno) |
| GET | `/api/v1/negocios` | Si | Usuario | Listar mis negocios |
| GET | `/api/v1/negocios/{negocio_id}` | No | - | Info publica del negocio |
| PUT | `/api/v1/negocios/{negocio_id}` | Si | Dueno | Actualizar negocio |

### Servicios (bajo contexto de negocio)
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| GET | `/api/v1/negocios/{negocio_id}/servicios` | No | - | Listar servicios |
| GET | `/api/v1/negocios/{negocio_id}/servicios/{id}` | No | - | Detalle de servicio |
| POST | `/api/v1/negocios/{negocio_id}/servicios` | Si | Dueno | Crear servicio |
| PUT | `/api/v1/negocios/{negocio_id}/servicios/{id}` | Si | Dueno | Actualizar servicio |
| DELETE | `/api/v1/negocios/{negocio_id}/servicios/{id}` | Si | Dueno | Desactivar servicio |

### Personal (bajo contexto de negocio)
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| GET | `/api/v1/negocios/{negocio_id}/personal` | No | - | Listar personal |
| POST | `/api/v1/negocios/{negocio_id}/personal` | Si | Dueno | Crear personal |
| PUT | `/api/v1/negocios/{negocio_id}/personal/{id}` | Si | Dueno | Actualizar personal |
| DELETE | `/api/v1/negocios/{negocio_id}/personal/{id}` | Si | Dueno | Desactivar personal |

### Disponibilidad (bajo contexto de negocio)
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| GET | `/api/v1/negocios/{negocio_id}/disponibilidad` | No | - | Consultar disponibilidad |

### Citas (bajo contexto de negocio)
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| POST | `/api/v1/negocios/{negocio_id}/citas` | Si | Cliente | Crear cita |
| GET | `/api/v1/negocios/{negocio_id}/citas/{id}` | Si | Cliente/Dueno | Detalle de cita |
| PUT | `/api/v1/negocios/{negocio_id}/citas/{id}` | Si | Cliente | Iniciar edicion |
| PUT | `/api/v1/negocios/{negocio_id}/citas/{id}/confirmar` | Si | Cliente | Confirmar edicion |
| DELETE | `/api/v1/negocios/{negocio_id}/citas/{id}/edicion` | Si | Cliente | Cancelar edicion |
| DELETE | `/api/v1/negocios/{negocio_id}/citas/{id}` | Si | Cliente/Dueno | Cancelar cita |

### Citas del Usuario (cross-negocio)
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| GET | `/api/v1/citas/me` | Si | Cliente | Mis citas (todos los negocios) |

### Configuracion del Negocio
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| GET | `/api/v1/negocios/{negocio_id}/horarios` | No | - | Obtener horarios |
| POST | `/api/v1/negocios/{negocio_id}/horarios` | Si | Dueno | Configurar horarios |
| GET | `/api/v1/negocios/{negocio_id}/excepciones` | No | - | Listar excepciones |
| POST | `/api/v1/negocios/{negocio_id}/excepciones` | Si | Dueno | Crear excepcion |
| DELETE | `/api/v1/negocios/{negocio_id}/excepciones/{id}` | Si | Dueno | Eliminar excepcion |

### Health Check
| Metodo | Endpoint | Auth | Rol | Descripcion |
|--------|----------|------|-----|-------------|
| GET | `/health` | No | - | Health check |

---

## 12. Esquemas Pydantic (Backend)

### 12.1 Esquemas de Autenticacion

```python
# app/schemas/auth.py

class RegistroRequest(BaseModel):
    email: EmailStr
    contrasena: str = Field(..., min_length=8)
    nombre_completo: str = Field(..., min_length=2, max_length=255)
    telefono: str = Field(..., pattern=r"^9\d{8}$")

class LoginRequest(BaseModel):
    email: EmailStr
    contrasena: str

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
    expires_in: int
    usuario: "UsuarioResponse"

class UsuarioResponse(BaseModel):
    id: UUID
    email: str
    nombre_completo: str
    telefono: str
    rol: RolUsuario
```

### 12.2 Esquemas de Negocio

```python
# app/schemas/negocio.py

class NegocioCreate(BaseModel):
    nombre: str = Field(..., max_length=255)
    slug: str = Field(..., max_length=100, pattern=r"^[a-z0-9-]+$")

class NegocioUpdate(BaseModel):
    nombre: str | None = Field(None, max_length=255)
    slug: str | None = Field(None, max_length=100, pattern=r"^[a-z0-9-]+$")

class NegocioBasico(BaseModel):
    id: UUID
    nombre: str

class NegocioResponse(BaseModel):
    id: UUID
    nombre: str
    slug: str
    dueno: "UsuarioBasico" | None = None
    activo: bool
    creado_en: datetime
    actualizado_en: datetime

class UsuarioBasico(BaseModel):
    id: UUID
    nombre_completo: str
```

### 12.3 Esquemas de Servicio

```python
# app/schemas/servicio.py

class ServicioCreate(BaseModel):
    nombre: str = Field(..., max_length=255)
    duracion_minutos: int = Field(..., ge=15, le=480)
    precio_pen: Decimal = Field(..., gt=0)
    personal_ids: list[UUID] = []

class ServicioUpdate(BaseModel):
    nombre: str | None = Field(None, max_length=255)
    duracion_minutos: int | None = Field(None, ge=15, le=480)
    precio_pen: Decimal | None = Field(None, gt=0)
    personal_ids: list[UUID] | None = None

class ServicioResponse(BaseModel):
    id: UUID
    negocio_id: UUID
    nombre: str
    duracion_minutos: int
    precio_pen: Decimal
    activo: bool
    personal: list["PersonalBasico"]
    creado_en: datetime
    actualizado_en: datetime
```

### 12.4 Esquemas de Disponibilidad

```python
# app/schemas/disponibilidad.py

class SlotDisponible(BaseModel):
    hora_inicio: time
    hora_fin: time
    personal_disponible: list["PersonalBasico"]

class SlotAgrupado(BaseModel):
    hora_inicio: time
    hora_fin: time
    cantidad_personal: int

class SlotsAgrupados(BaseModel):
    manana: list[SlotAgrupado]
    mediodia: list[SlotAgrupado]
    tarde: list[SlotAgrupado]

class DisponibilidadDia(BaseModel):
    fecha: date
    dia_semana: str
    slots: list[SlotDisponible]
    slots_agrupados: SlotsAgrupados

class DisponibilidadResponse(BaseModel):
    negocio: "NegocioBasico"
    servicio: "ServicioBasico"
    fecha_inicio: date
    fecha_fin: date
    disponibilidad: list[DisponibilidadDia]
    fechas_sin_disponibilidad: list[date]
```

### 12.5 Esquemas de Cita

```python
# app/schemas/cita.py

class CitaCreate(BaseModel):
    servicio_id: UUID
    personal_id: UUID | None = None
    fecha_hora_cita: datetime
    cliente_nombre: str = Field(..., min_length=2, max_length=255)
    cliente_email: EmailStr
    cliente_telefono: str = Field(..., pattern=r"^9\d{8}$")

class CitaUpdate(BaseModel):
    fecha_hora_cita: datetime
    personal_id: UUID | None = None

class CitaResponse(BaseModel):
    id: UUID
    negocio: "NegocioBasico"
    servicio: "ServicioBasico"
    personal: "PersonalBasico"
    fecha_hora_inicio: datetime
    fecha_hora_fin: datetime
    cliente_nombre: str
    cliente_email: str
    cliente_telefono: str
    estado: EstadoCita
    numero_confirmacion: str
    puede_editar: bool
    puede_cancelar: bool
    creado_en: datetime
    actualizado_en: datetime

class CitaListItem(BaseModel):
    id: UUID
    negocio: "NegocioBasico"
    servicio: "ServicioBasico"
    personal: "PersonalBasico"
    fecha_hora_inicio: datetime
    fecha_hora_fin: datetime
    estado: EstadoCita
    numero_confirmacion: str
    puede_editar: bool
    puede_cancelar: bool
```

### 12.6 Esquemas de Configuracion del Negocio

```python
# app/schemas/configuracion.py

class HorarioNegocioItem(BaseModel):
    dia_semana: int = Field(..., ge=0, le=6)
    hora_apertura: time | None = None
    hora_cierre: time | None = None
    activo: bool = True

class HorarioNegocioCreate(BaseModel):
    horarios: list[HorarioNegocioItem]

class HorarioNegocioResponse(BaseModel):
    id: UUID | None
    dia_semana: int
    dia_nombre: str
    hora_apertura: time | None
    hora_cierre: time | None
    activo: bool

class FechaExcepcionCreate(BaseModel):
    fecha_excepcion: date
    motivo: str = Field(..., max_length=255)

class FechaExcepcionResponse(BaseModel):
    id: UUID
    negocio_id: UUID
    fecha_excepcion: date
    motivo: str
    creado_en: datetime
```

---

## 13. Tipos TypeScript (Frontend)

```typescript
// src/tipos/index.ts

// Enums
type RolUsuario = "cliente" | "dueno_negocio";
type EstadoCita = "pendiente" | "confirmada" | "pendiente_actualizacion" | "cancelada" | "completada";

// Respuestas base
interface PaginatedResponse<T> {
  items: T[];
  total: number;
  pagina: number;
  por_pagina: number;
  total_paginas: number;
}

interface ErrorResponse {
  error: string;
  mensaje: string;
  detalles: Record<string, unknown>;
  timestamp: string;
}

// Usuario
interface Usuario {
  id: string;
  email: string;
  nombre_completo: string;
  telefono: string;
  rol: RolUsuario;
}

interface UsuarioBasico {
  id: string;
  nombre_completo: string;
}

interface TokenResponse {
  access_token: string;
  token_type: string;
  expires_in: number;
  usuario: Usuario;
}

// Negocio
interface NegocioBasico {
  id: string;
  nombre: string;
}

interface Negocio extends NegocioBasico {
  slug: string;
  dueno?: UsuarioBasico;
  activo: boolean;
  creado_en: string;
  actualizado_en: string;
}

interface NegocioListItem {
  id: string;
  nombre: string;
  slug: string;
  activo: boolean;
  creado_en: string;
}

// Personal
interface PersonalBasico {
  id: string;
  nombre: string;
}

interface Personal extends PersonalBasico {
  negocio_id: string;
  email: string;
  activo: boolean;
  servicios: ServicioBasico[];
  creado_en: string;
  actualizado_en: string;
}

// Servicio
interface ServicioBasico {
  id: string;
  nombre: string;
  duracion_minutos: number;
  precio_pen: number;
}

interface Servicio extends ServicioBasico {
  negocio_id: string;
  activo: boolean;
  personal: PersonalBasico[];
  creado_en: string;
  actualizado_en: string;
}

// Disponibilidad
interface SlotDisponible {
  hora_inicio: string;
  hora_fin: string;
  personal_disponible: PersonalBasico[];
}

interface SlotAgrupado {
  hora_inicio: string;
  hora_fin: string;
  cantidad_personal: number;
}

interface SlotsAgrupados {
  manana: SlotAgrupado[];
  mediodia: SlotAgrupado[];
  tarde: SlotAgrupado[];
}

interface DisponibilidadDia {
  fecha: string;
  dia_semana: string;
  slots: SlotDisponible[];
  slots_agrupados: SlotsAgrupados;
}

interface DisponibilidadResponse {
  negocio: NegocioBasico;
  servicio: ServicioBasico;
  fecha_inicio: string;
  fecha_fin: string;
  disponibilidad: DisponibilidadDia[];
  fechas_sin_disponibilidad: string[];
}

// Cita
interface Cita {
  id: string;
  negocio: NegocioBasico;
  servicio: ServicioBasico;
  personal: PersonalBasico;
  fecha_hora_inicio: string;
  fecha_hora_fin: string;
  cliente_nombre: string;
  cliente_email: string;
  cliente_telefono: string;
  estado: EstadoCita;
  numero_confirmacion: string;
  puede_editar: boolean;
  puede_cancelar: boolean;
  creado_en: string;
  actualizado_en: string;
}

interface CitaListItem {
  id: string;
  negocio: NegocioBasico;
  servicio: ServicioBasico;
  personal: PersonalBasico;
  fecha_hora_inicio: string;
  fecha_hora_fin: string;
  estado: EstadoCita;
  numero_confirmacion: string;
  puede_editar: boolean;
  puede_cancelar: boolean;
}

// Configuracion del Negocio
interface HorarioNegocio {
  id?: string;
  dia_semana: number;
  dia_nombre: string;
  hora_apertura?: string;
  hora_cierre?: string;
  activo: boolean;
}

interface FechaExcepcion {
  id: string;
  negocio_id: string;
  fecha_excepcion: string;
  motivo: string;
  creado_en: string;
}
```

---

**Fin del Documento de Contratos de API**
