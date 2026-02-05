# Arquitectura AWS - Agenda Salon MVP

## Resumen

Arquitectura cloud simplificada para un **prototipo funcional** de Agenda Salon. Optimizada para:
- Hasta 10 usuarios concurrentes
- Bajo costo operativo
- Mantenimiento simple
- Sin requisitos de alta disponibilidad (99%+)

---

## Diagrama de Arquitectura

```
                         ARQUITECTURA AWS - AGENDA SALON MVP
    ┌────────────────────────────────────────────────────────────────────────┐
    │                         AWS Cloud (us-east-1)                          │
    │                                                                        │
    │   ┌──────────────────────────────────────────────────────────────┐    │
    │   │                        VPC (10.0.0.0/16)                      │    │
    │   │                                                              │    │
    │   │   ┌─────────────────────────────────────────────────────┐   │    │
    │   │   │              Public Subnet (10.0.1.0/24)            │   │    │
    │   │   │                                                     │   │    │
Internet   │   │   ┌─────────────────────────────────────────────┐│   │    │
    │      │   │   │         Application Load Balancer           ││   │    │
────┼──────┼───┼──►│              (ALB)                          ││   │    │
HTTPS:443  │   │   │  - HTTPS termination                        ││   │    │
    │      │   │   │  - Health checks                            ││   │    │
    │      │   │   │  - Route: /* -> Frontend                    ││   │    │
    │      │   │   │  - Route: /api/* -> Backend                 ││   │    │
    │      │   │   └──────────────┬──────────────────────────────┘│   │    │
    │      │   │                  │                               │   │    │
    │      │   │                  ▼                               │   │    │
    │      │   │   ┌─────────────────────────────────────────────┐│   │    │
    │      │   │   │            ECS Fargate Cluster              ││   │    │
    │      │   │   │                                             ││   │    │
    │      │   │   │  ┌─────────────────┐ ┌─────────────────┐   ││   │    │
    │      │   │   │  │   Frontend      │ │    Backend      │   ││   │    │
    │      │   │   │  │   Service       │ │    Service      │   ││   │    │
    │      │   │   │  │                 │ │                 │   ││   │    │
    │      │   │   │  │ ┌─────────────┐ │ │ ┌─────────────┐ │   ││   │    │
    │      │   │   │  │ │   Task      │ │ │ │   Task      │ │   ││   │    │
    │      │   │   │  │ │  (React +   │ │ │ │  (FastAPI)  │ │   ││   │    │
    │      │   │   │  │ │   nginx)    │ │ │ │             │ │   ││   │    │
    │      │   │   │  │ │  :80        │ │ │ │  :8000      │ │   ││   │    │
    │      │   │   │  │ └─────────────┘ │ │ └──────┬──────┘ │   ││   │    │
    │      │   │   │  └─────────────────┘ └────────┼────────┘   ││   │    │
    │      │   │   └───────────────────────────────┼─────────────┘│   │    │
    │      │   │                                   │              │   │    │
    │      │   └───────────────────────────────────┼──────────────┘   │    │
    │      │                                       │                  │    │
    │      │   ┌───────────────────────────────────┼──────────────┐   │    │
    │      │   │           Private Subnet (10.0.2.0/24)           │   │    │
    │      │   │                                   │              │   │    │
    │      │   │                                   ▼              │   │    │
    │      │   │            ┌─────────────────────────────┐      │   │    │
    │      │   │            │      RDS PostgreSQL 15      │      │   │    │
    │      │   │            │                             │      │   │    │
    │      │   │            │   Instance: db.t3.micro     │      │   │    │
    │      │   │            │   Storage: 20 GB SSD        │      │   │    │
    │      │   │            │   Single-AZ (no replica)    │      │   │    │
    │      │   │            │   Automated backups: 7 days │      │   │    │
    │      │   │            │   Port: 5432                │      │   │    │
    │      │   │            └─────────────────────────────┘      │   │    │
    │      │   │                                                  │   │    │
    │      │   └──────────────────────────────────────────────────┘   │    │
    │      │                                                          │    │
    │      └──────────────────────────────────────────────────────────┘    │
    │                                                                      │
    │   ┌────────────────────────────────────────────────────────────┐    │
    │   │                    Servicios de Soporte                     │    │
    │   │                                                            │    │
    │   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │    │
    │   │   │     ECR     │  │ CloudWatch  │  │  Secrets Manager │   │    │
    │   │   │             │  │             │  │   (opcional)     │   │    │
    │   │   │  frontend/  │  │  /ecs/      │  │                  │   │    │
    │   │   │  backend/   │  │  agenda-    │  │  DB credentials  │   │    │
    │   │   │             │  │  salon      │  │  JWT secret      │   │    │
    │   │   └─────────────┘  └─────────────┘  └─────────────────┘   │    │
    │   │                                                            │    │
    │   └────────────────────────────────────────────────────────────┘    │
    │                                                                      │
    └──────────────────────────────────────────────────────────────────────┘
```

---

## Servicios AWS y Proposito

| # | Servicio | Proposito | Configuracion MVP |
|---|----------|-----------|-------------------|
| 1 | **Amazon ECS (Fargate)** | Ejecutar contenedores Docker sin administrar servidores EC2. Escala automaticamente y paga solo por uso. | 2 servicios (frontend + backend), 0.25 vCPU y 0.5 GB RAM por tarea |
| 2 | **Amazon RDS PostgreSQL** | Base de datos relacional gestionada con respaldos automaticos, parches y mantenimiento manejado por AWS. | db.t3.micro, 20 GB gp2, Single-AZ, retention 7 dias |
| 3 | **Application Load Balancer (ALB)** | Distribuir trafico HTTP/HTTPS entre servicios. Terminar SSL, enrutar por path y verificar health de contenedores. | 1 ALB con 2 target groups (frontend :80, backend :8000) |
| 4 | **Amazon ECR** | Almacenar imagenes Docker de forma segura. Integrado con ECS para despliegues automaticos. | 2 repositorios: `agenda-frontend`, `agenda-backend` |
| 5 | **Amazon CloudWatch** | Centralizar logs de aplicacion y metricas basicas. Permite debugging y monitoreo de errores. | Log groups para ECS tasks, retention 14 dias |
| 6 | **Amazon VPC** | Aislamiento de red. Subnets publicas para ALB y privadas para base de datos. Security groups para control de acceso. | VPC default o custom, 2 subnets (public + private) |
| 7 | **AWS Secrets Manager** | (Opcional) Almacenar credenciales de forma segura en lugar de variables de entorno. | Solo si se requiere rotacion de secretos |

---

## Flujo de Trafico

```
Usuario (Browser)
       │
       │ HTTPS (443)
       ▼
┌─────────────────┐
│       ALB       │
│  (agenda.com)   │
└────────┬────────┘
         │
         ├─── /* ───────────► Frontend (React/nginx:80)
         │
         └─── /api/* ───────► Backend (FastAPI:8000)
                                    │
                                    │ PostgreSQL (5432)
                                    ▼
                              ┌───────────┐
                              │    RDS    │
                              │ PostgreSQL│
                              └───────────┘
```

---

## Detalle por Servicio

### 1. Amazon ECS Fargate

**Que es:** Servicio de contenedores serverless. Ejecuta Docker sin provisionar ni administrar servidores.

**Por que usarlo:**
- No hay que manejar EC2, parches, ni escalado de instancias
- Pago por segundo de uso (ideal para prototipos)
- Integracion nativa con ALB y CloudWatch

**Configuracion MVP:**

| Parametro | Frontend | Backend |
|-----------|----------|---------|
| CPU | 0.25 vCPU | 0.25 vCPU |
| Memoria | 0.5 GB | 0.5 GB |
| Tareas deseadas | 1 | 1 |
| Puerto | 80 | 8000 |
| Health check | `/` | `/health` |

---

### 2. Amazon RDS PostgreSQL

**Que es:** Base de datos PostgreSQL gestionada por AWS.

**Por que usarlo:**
- Respaldos automaticos (point-in-time recovery)
- Parches de seguridad automaticos
- Monitoreo integrado en CloudWatch
- Facil escalamiento vertical si crece el proyecto

**Configuracion MVP:**

| Parametro | Valor |
|-----------|-------|
| Engine | PostgreSQL 15 |
| Instance class | db.t3.micro |
| Storage | 20 GB gp2 (SSD) |
| Multi-AZ | No (Single-AZ) |
| Backup retention | 7 dias |
| Encryption | Habilitado (at rest) |
| Public access | No (solo desde VPC) |

---

### 3. Application Load Balancer (ALB)

**Que es:** Balanceador de carga de capa 7 (HTTP/HTTPS).

**Por que usarlo:**
- Terminacion SSL en un solo punto
- Enrutamiento basado en paths (`/api/*` vs `/*`)
- Health checks automaticos
- Integrado con ECS para descubrimiento de tareas

**Reglas de enrutamiento:**

| Prioridad | Condicion | Accion |
|-----------|-----------|--------|
| 1 | Path `/api/*` | Forward to Backend target group |
| 2 | Path `/*` (default) | Forward to Frontend target group |

---

### 4. Amazon ECR

**Que es:** Registro privado de imagenes Docker.

**Por que usarlo:**
- Almacenamiento seguro de imagenes
- Escaneo de vulnerabilidades (opcional)
- Integrado con ECS para pull automatico

**Repositorios:**
- `agenda-salon/frontend`
- `agenda-salon/backend`

---

### 5. Amazon CloudWatch

**Que es:** Servicio de monitoreo y logs.

**Por que usarlo:**
- Centraliza logs de todos los contenedores
- Permite buscar errores facilmente
- Metricas basicas sin costo adicional

**Log Groups:**
- `/ecs/agenda-salon-frontend`
- `/ecs/agenda-salon-backend`

---

### 6. VPC y Networking

**Estructura de red:**

```
VPC: 10.0.0.0/16
│
├── Public Subnet: 10.0.1.0/24
│   └── ALB, NAT Gateway (si es necesario)
│
└── Private Subnet: 10.0.2.0/24
    └── RDS PostgreSQL
```

**Security Groups:**

| Nombre | Inbound | Outbound |
|--------|---------|----------|
| `sg-alb` | 443 from 0.0.0.0/0 | All |
| `sg-ecs` | 80,8000 from sg-alb | All |
| `sg-rds` | 5432 from sg-ecs | None |

---

## Estimacion de Costos (USD/mes)

| Servicio | Estimacion |
|----------|------------|
| ECS Fargate (2 tareas, 24/7) | ~$15 |
| RDS db.t3.micro (24/7) | ~$15 |
| ALB | ~$16 + $0.008/LCU |
| ECR (< 1 GB) | ~$0.10 |
| CloudWatch Logs | ~$1-2 |
| Data Transfer | ~$1-5 |
| **Total estimado** | **~$50-55/mes** |

> Nota: Estos costos son aproximados para la region us-east-1 con uso minimo. Pueden variar segun uso real.

---

## Alternativa: Aun Mas Simple

Si el presupuesto es muy limitado, se puede simplificar mas:

```
┌──────────────────────────────────────────────┐
│              EC2 t3.micro (Free Tier)        │
│                                              │
│  ┌────────────┐    ┌────────────────────┐   │
│  │   nginx    │───►│ Docker Compose     │   │
│  │ (reverse   │    │                    │   │
│  │  proxy)    │    │ - frontend         │   │
│  └────────────┘    │ - backend          │   │
│                    │ - postgres         │   │
│                    └────────────────────┘   │
└──────────────────────────────────────────────┘

Costo: ~$0 (Free Tier) o ~$8/mes (t3.micro)
```

Esta alternativa es viable para desarrollo/demo pero no recomendada para produccion real.

---

## Notas para el Prototipo

1. **Sin alta disponibilidad:** Single-AZ es suficiente para MVP. Si falla la zona, hay downtime.

2. **Sin CDN:** CloudFront se puede agregar despues si se necesita mejor performance global.

3. **Sin WAF:** Para un prototipo no es necesario. Agregar si se detecta trafico malicioso.

4. **Escalamiento manual:** Las tareas de ECS estan fijas en 1. Escalar manualmente si crece la demanda.

5. **HTTPS:** Usar AWS Certificate Manager (ACM) para certificado SSL gratuito.

---

## Diagrama Simplificado

```
     Internet
         │
         ▼
    ┌─────────┐
    │   ALB   │  ← HTTPS (ACM Certificate)
    └────┬────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────┐
│Frontend│ │Backend│  ← ECS Fargate
│ :80   │ │ :8000 │
└───────┘ └───┬───┘
              │
              ▼
         ┌────────┐
         │  RDS   │  ← PostgreSQL
         │ :5432  │
         └────────┘
```

---

**Fin del Documento de Arquitectura AWS**
