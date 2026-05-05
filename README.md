# 🏢 RRHH-SOA — Sistema de Gestión de Recursos Humanos

> Arquitectura Orientada a Servicios (SOA) implementada con Node.js, Express, Docker y bases de datos especializadas por servicio.

---

## 📐 Arquitectura

```
                        ┌─────────────────────────────────┐
                        │         API GATEWAY             │
          Cliente ──▶   │  Puerto 3000  |  JWT + Rate Limit│
         (Postman)      │  Logger Winston | Proxy Router   │
                        └──────┬──────────────────┬────────┘
                               │                  │
              ┌────────────────┼──────────────────┼────────────────┐
              ▼                ▼                  ▼                │
     ┌────────────────┐ ┌────────────────┐ ┌─────────────────┐    │
     │   EMPLEADOS    │ │    NÓMINA      │ │  RECLUTAMIENTO  │    │
     │   Puerto 3001  │ │  Puerto 3002   │ │   Puerto 3003   │    │
     │   PostgreSQL   │ │  PostgreSQL    │ │    MongoDB      │    │
     └────────────────┘ └────────────────┘ └─────────────────┘    │
```

## 🗂️ Estructura del Proyecto

```
rrhh-soa/
├── gateway/                  # API Gateway (ESB)
│   ├── src/
│   │   ├── middlewares/
│   │   │   └── auth.js       # Validación JWT
│   │   ├── routes/
│   │   │   └── auth.js       # Login público
│   │   └── index.js          # Configuración principal
│   ├── .env
│   ├── Dockerfile
│   └── package.json
├── services/
│   ├── empleados/            # Servicio de empleados (Puerto 3001)
│   │   ├── src/
│   │   ├── .env
│   │   ├── Dockerfile
│   │   └── package.json
│   ├── nomina/               # Servicio de nómina (Puerto 3002)
│   │   ├── src/
│   │   ├── .env
│   │   ├── Dockerfile
│   │   └── package.json
│   └── reclutamiento/        # Servicio de reclutamiento (Puerto 3003)
│       ├── src/
│       ├── .env
│       ├── Dockerfile
│       └── package.json
├── docker-compose.yml
└── README.md
```

---

## 🛠️ Tecnologías Utilizadas

| Componente       | Tecnología                        |
|------------------|-----------------------------------|
| Runtime          | Node.js 18 LTS                    |
| Framework        | Express.js                        |
| Gateway/Proxy    | http-proxy-middleware             |
| Autenticación    | JSON Web Tokens (jsonwebtoken)    |
| ORM Relacional   | Sequelize v6                      |
| ODM Documentos   | Mongoose v8                       |
| Base de datos 1  | PostgreSQL 15 (Empleados + Nómina)|
| Base de datos 2  | MongoDB 7 (Reclutamiento)         |
| Logging          | Winston                           |
| Rate Limiting    | express-rate-limit                |
| Contenedores     | Docker + Docker Compose           |

---

## 🚀 Instalación y Ejecución

### Prerrequisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y corriendo
- Git

### Pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/rrhh-soa.git
cd rrhh-soa

# 2. Levantar todos los servicios
docker-compose up --build

# 3. Verificar que todos los contenedores estén corriendo
docker-compose ps
```

Deberías ver 7 contenedores activos:

| Contenedor             | Puerto | Estado   |
|------------------------|--------|----------|
| rrhh-soa-gateway       | 3000   | Up       |
| rrhh-soa-empleados     | 3001   | Up       |
| rrhh-soa-nomina        | 3002   | Up       |
| rrhh-soa-reclutamiento | 3003   | Up       |
| rrhh-soa-postgres_emp  | —      | Healthy  |
| rrhh-soa-postgres_nom  | —      | Healthy  |
| rrhh-soa-mongo         | —      | Up       |

---

## 🔐 Autenticación

Todas las rutas `/api/*` requieren un token JWT en el header `Authorization`.

### Obtener token

```http
POST http://localhost:3000/auth/login
Content-Type: application/json

{
  "usuario": "admin",
  "password": "1234"
}
```

**Respuesta:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "usuario": "admin",
  "rol": "admin"
}
```

### Usar el token

```http
Authorization: Bearer <token>
```

---

## 📡 Endpoints

### 🔑 Autenticación (público)

| Método | Ruta               | Descripción          |
|--------|--------------------|----------------------|
| POST   | `/auth/login`      | Iniciar sesión       |

### 👥 Empleados — `/api/empleados`

| Método | Ruta                    | Descripción                          |
|--------|-------------------------|--------------------------------------|
| POST   | `/api/empleados`        | Crear un nuevo empleado              |
| GET    | `/api/empleados`        | Listar empleados (filtro por depto.) |
| GET    | `/api/empleados/:id`    | Obtener empleado por ID              |
| PUT    | `/api/empleados/:id`    | Actualizar empleado                  |
| DELETE | `/api/empleados/:id`    | Desactivar empleado (soft delete)    |

**Ejemplo — Crear empleado:**
```json
{
  "cedula": "1712345678",
  "nombre": "Ana",
  "apellido": "Torres",
  "cargo": "Desarrolladora Backend",
  "departamento": "TI",
  "salario_base": 1500
}
```

### 💰 Nómina — `/api/nomina`

| Método | Ruta                    | Descripción                   |
|--------|-------------------------|-------------------------------|
| POST   | `/api/nomina/calcular`  | Calcular nómina de un empleado|
| GET    | `/api/nomina`           | Listar registros de nómina    |
| GET    | `/api/nomina/:id`       | Obtener nómina por ID         |

**Ejemplo — Calcular nómina:**
```json
{
  "empleado_id": 1,
  "periodo": "2025-04",
  "salario_base": 1500,
  "horas_extra": 5,
  "bono": 100,
  "descuentos": 67.50
}
```

**Fórmula de cálculo:**
```
Valor hora extra = salario_base / 240 * 1.5
Total extras     = horas_extra * valor_hora_extra
Neto a pagar     = salario_base + total_extras + bono - descuentos
```

### 📋 Reclutamiento — `/api/reclutamiento`

| Método | Ruta                                        | Descripción               |
|--------|---------------------------------------------|---------------------------|
| POST   | `/api/reclutamiento/vacantes`               | Publicar vacante          |
| GET    | `/api/reclutamiento/vacantes`               | Listar vacantes           |
| POST   | `/api/reclutamiento/candidatos`             | Registrar candidato       |
| GET    | `/api/reclutamiento/candidatos`             | Listar candidatos         |
| PUT    | `/api/reclutamiento/candidatos/:id/estado`  | Actualizar estado         |

**Estados válidos del candidato:** `recibido` → `entrevista` → `aprobado` / `rechazado`

---

## ⚙️ Variables de Entorno

### Gateway (`gateway/.env`)

```env
PORT=3000
JWT_SECRET=mi_secreto_jwt_super_seguro
EMPLEADOS_URL=http://empleados:3001
NOMINA_URL=http://nomina:3002
RECLUTAMIENTO_URL=http://reclutamiento:3003
```

### Servicio Empleados (`services/empleados/.env`)

```env
PORT=3001
DB_HOST=postgres_emp
DB_PORT=5432
DB_NAME=empleados_db
DB_USER=postgres
DB_PASS=postgres123
```

---

## 🧪 Pruebas con Postman

Importa la colección o ejecuta las pruebas en este orden:

1. ✅ `GET /health` — Verificar Gateway
2. ✅ `POST /auth/login` — Obtener token JWT
3. ✅ `GET /api/empleados` sin token — Debe devolver 401
4. ✅ `POST /api/empleados` — Crear empleado
5. ✅ `GET /api/empleados` — Listar empleados
6. ✅ `POST /api/empleados` cédula duplicada — Debe devolver 409
7. ✅ `POST /api/nomina/calcular` — Calcular nómina
8. ✅ `POST /api/reclutamiento/vacantes` — Crear vacante
9. ✅ `POST /api/reclutamiento/candidatos` — Registrar candidato
10. ✅ `PUT /api/reclutamiento/candidatos/:id/estado` — Cambiar estado

---

## 🩺 Health Check

```http
GET http://localhost:3000/health
```

```json
{
  "status": "ok",
  "servicios": {
    "empleados": "http://empleados:3001",
    "nomina": "http://nomina:3002",
    "reclutamiento": "http://reclutamiento:3003"
  }
}
```

---

Universidad de las Fuerzas Armadas ESPE — Período 202650
