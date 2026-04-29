# Food Store — Repositorio Base

Sistema de e-commerce de productos alimenticios desarrollado con **Spec-Driven Development (SDD)** usando OPSX y Claude Code.

---

## Documentación del sistema

Antes de escribir una línea de código, leé los documentos en `docs/`:

| Archivo | Contenido |
|---------|-----------|
| `docs/Descripcion.txt` | Visión general, actores del sistema y stack tecnológico |
| `docs/Integrador.txt` | Arquitectura en capas, ERD, API REST y patrones de diseño |
| `docs/Historias_de_usuario.txt` | US-000 a US-076 con criterios de aceptación y reglas de negocio |
| `docs/CHANGES-ROADMAP.md` | Roadmap completo de 22 changes ordenados por dependencias |

Estos documentos son la fuente de verdad del sistema.

---

## Stack tecnológico

**Backend**: FastAPI · SQLModel · PostgreSQL · Alembic · bcrypt · python-jose · slowapi · MercadoPago SDK
**Frontend**: React · TypeScript · Vite · TanStack Query · TanStack Form · Zustand · Axios · Tailwind CSS · Recharts

---

## Estructura del proyecto

```
food-store/
├── backend/                    # FastAPI — Feature-First
│   ├── auth/                  # Login, registro, refresh, logout
│   ├── usuarios/              # CRUD usuarios, asignación de roles
│   ├── direcciones/           # Direcciones de entrega
│   ├── categorias/           # Categorías jerárquicas (CTE recursiva)
│   ├── productos/             # Catálogo con ingredientes y stock
│   ├── ingredientes/          # Ingredientes con flag es_alergeno
│   ├── pedidos/               # Pedidos con FSM y audit trail
│   ├── pagos/                 # Integración MercadoPago
│   ├── admin/                 # Dashboard y métricas
│   ├── refreshtokens/         # Gestión de refresh tokens
│   └── core/                  # Config, database, security (UOW base)
│
├── frontend/src/              # React — Feature-Sliced Design
│   ├── app/                   # App root, providers, routing
│   ├── pages/                 # Vistas por ruta
│   ├── widgets/                # Composición de features
│   ├── features/               # Interacciones de usuario
│   ├── entities/               # Modelos de dominio
│   └── shared/                 # UI genérica, API, types
│
├── docs/                      # Documentación del sistema
├── openspec/                  # Artefactos OPSX (changes archivados)
│
├── README.md                  # Este archivo
└── .gitignore
```

### Convenciones de arquitectura

**Backend — Feature-First**
Cada módulo es una carpeta autocontenida con: `model.py`, `schemas.py`, `repository.py`, `service.py`, `router.py`.
Flujo de dependencias: `Router → Service → UoW → Repository → Model`

**Frontend — Feature-Sliced Design**
Imports fluyen de mayor a menor abstracción: `app` → `pages` → `widgets` → `features` → `entities` → `shared`

---

## Setup del entorno de desarrollo

### Requisitos previos
- Python 3.11+
- Node.js 18+
- PostgreSQL 15+
- Claude Code: `npm install -g @anthropic-ai/claude-code`
- OpenSpec CLI: `npm install -g @fission-ai/openspec`

### 1. Clonar

```bash
git clone <url-del-repo> food-store
cd food-store
```

### 2. Backend

```bash
cd backend
cp .env.example .env
# Completar las variables de entorno en .env

python -m venv .venv
.venv\Scripts\activate      # Windows
# source .venv/bin/activate  # Linux/Mac

pip install -r requirements.txt
alembic upgrade head
python -m app.db.seed
uvicorn app.main:app --reload
```

API: `http://localhost:8000` · Swagger: `http://localhost:8000/docs`

### 3. Frontend

```bash
cd frontend
cp .env.example .env
# Completar VITE_API_BASE_URL=http://localhost:8000/api/v1

npm install
npm run dev
```

App: `http://localhost:5173`

---

## Flujo de desarrollo con OPSX

Todo cambio al sistema sigue este ciclo:

```
/opsx:explore   →  pensar antes de comprometerse (opcional)
/opsx:propose   →  generar propuesta + diseño + tareas
/opsx:apply     →  implementar tarea por tarea
/opsx:verify    →  auditar que el código coincide con las specs
/opsx:archive   →  sincronizar specs y cerrar el change
```

---

## Roadmap de changes

| # | Change | Tipo | Dependencias |
|---|--------|------|-------------|
| 01 | `project-scaffolding` | foundation | — ✅ |
| 02 | `backend-core-setup` | foundation | 01 |
| 03 | `database-setup` | foundation | 02 |
| 04 | `backend-patterns` | foundation | 03 |
| 05 | `frontend-core-setup` | foundation | 01 |
| 06 | `frontend-state-stores` | foundation | 05 |
| 07 | `authentication-backend` | feature | 04, 03 |
| 08 | `rbac-backend` | feature | 07, 03 |
| 09 | `navigation-frontend` | feature | 07, 06 |
| 10 | `axios-interceptors` | feature | 07, 06 |
| 11 | `categories-management` | feature | 04, 08 |
| 12 | `ingredients-management` | feature | 04, 08 |
| 13 | `products-management` | feature | 11, 12, 08 |
| 14 | `product-catalog-public` | feature | 13 |
| 15 | `customer-profile` | feature | 07 |
| 16 | `delivery-addresses` | feature | 07 |
| 17 | `shopping-cart` | feature | 14, 13 |
| 18 | `order-creation` | feature | 04, 17, 16 |
| 19 | `order-state-machine` | feature | 18, 08 |
| 20 | `payment-mercadopago` | integration | 18, 19 |
| 21 | `order-ui-feedback` | feature | 18, 20 |
| 22 | `admin-dashboard` | feature | 19, 08 |

Ver `docs/CHANGES-ROADMAP.md` para detalle completo de cada change.

---

## Convenciones de commits

```
feat: add project scaffolding structure
fix(auth): correct password validation logic
docs(productos): update API endpoint descriptions
refactor(pedidos): extract FSM logic to service layer
test(categorias): add tests for recursive CTE query
chore: update dependencies
```

---

## Variables de entorno

**Backend** — copiar de `backend/.env.example`:

| Variable | Descripción |
|----------|-------------|
| `DATABASE_URL` | Connection string PostgreSQL |
| `SECRET_KEY` | Clave JWT (mín. 32 chars) |
| `ALGORITHM` | Algoritmo JWT (HS256) |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Duración access token (30) |
| `REFRESH_TOKEN_EXPIRE_DAYS` | Duración refresh token (7) |
| `CORS_ORIGINS` | Orígenes permitidos (JSON array) |
| `MERCADOPAGO_ACCESS_TOKEN` | Access token MercadoPago |
| `MERCADOPAGO_PUBLIC_KEY` | Public key MercadoPago |
| `MP_NOTIFICATION_URL` | URL webhook IPN |

**Frontend** — copiar de `frontend/.env.example`:

| Variable | Descripción |
|----------|-------------|
| `VITE_API_BASE_URL` | URL base del backend |
| `VITE_MERCADOPAGO_PUBLIC_KEY` | Public key MercadoPago |

---

## Historial de changes archivados

| Date | Change | Commit |
|------|--------|--------|
| 2026-04-29 | `project-scaffolding` | `e747e4f` |

Los artefactos de cada change se encuentran en `openspec/changes/archive/`.
