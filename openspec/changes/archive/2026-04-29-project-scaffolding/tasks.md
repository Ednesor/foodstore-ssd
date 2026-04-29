## 1. Monorepo root structure

- [x] 1.1 Create root directories: `backend/` and `frontend/`
- [x] 1.2 Verify root contains only README.md, docs/, openspec/, backend/, frontend/ and no application code

## 2. Backend structure

- [x] 2.1 Create the 10 backend module folders: `auth/`, `usuarios/`, `productos/`, `categorias/`, `ingredientes/`, `pedidos/`, `pagos/`, `direcciones/`, `admin/`, `refreshtokens/`
- [x] 2.2 For each module folder, create empty `model.py`, `schemas.py`, `repository.py`, `service.py`, `router.py`
- [x] 2.3 Create `backend/core/` directory for shared infrastructure
- [x] 2.4 Add a `backend/__init__.py` to make it a Python package

## 3. Frontend structure

- [x] 3.1 Create the 6 FSD layer folders inside `frontend/src/`: `app/`, `pages/`, `widgets/`, `features/`, `entities/`, `shared/`
- [x] 3.2 Create placeholder `frontend/src/app/App.tsx` (minimal React component)
- [x] 3.3 Create `frontend/src/main.tsx` entry point
- [x] 3.4 Verify no additional layer folders exist outside the defined list

## 4. Git configuration

- [x] 4.1 Create root `.gitignore` covering both stacks: `.env`, `__pycache__/`, `node_modules/`, `.venv/`, `*.pyc`, `dist/`, `.DS_Store`, `*.log`, `.env.local`
- [x] 4.2 Verify `.env` is not tracked after running `git status`

## 5. Documentation

- [x] 5.1 Create root `README.md` with sections: Descripción del proyecto, Requisitos, Instalación (backend + frontend por separado), Ejecución (dev servers), Convenciones de commits
- [x] 5.2 Create `backend/.env.example` documenting all vars: DATABASE_URL, SECRET_KEY, ALGORITHM, ACCESS_TOKEN_EXPIRE_MINUTES, REFRESH_TOKEN_EXPIRE_DAYS, CORS_ORIGINS, MERCADOPAGO_ACCESS_TOKEN, MERCADOPAGO_PUBLIC_KEY, MP_NOTIFICATION_URL
- [x] 5.3 Create `frontend/.env.example` documenting: VITE_API_BASE_URL, VITE_MERCADOPAGO_PUBLIC_KEY

## 6. Git commit

- [x] 6.1 Stage all new files and directories
- [x] 6.2 Create commit with message following conventional commits format: `feat: add project scaffolding structure`
- [x] 6.3 Verify `git log` shows the commit with correct format