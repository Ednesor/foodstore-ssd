## ADDED Requirements

### Requirement: Monorepo root structure
The system SHALL organize the codebase as a monorepo with two main directories at root: `/backend` and `/frontend`.

#### Scenario: Root directory contains backend and frontend
- **WHEN** a developer lists the root directory contents
- **THEN** the directories `backend/` and `frontend/` are present
- **AND** no other application code exists at root level

### Requirement: Backend module structure
Each backend functional module SHALL be a self-contained folder containing exactly: `model.py`, `schemas.py`, `repository.py`, `service.py`, and `router.py`.

#### Scenario: Backend module has all required files
- **WHEN** a backend module folder is created
- **THEN** it contains `model.py`, `schemas.py`, `repository.py`, `service.py`, and `router.py`
- **AND** all files are empty or contain only minimal documentation

### Requirement: Backend initial module list
The backend SHALL start with exactly these 10 module folders: `auth/`, `usuarios/`, `productos/`, `categorias/`, `ingredientes/`, `pedidos/`, `pagos/`, `direcciones/`, `admin/`, `refreshtokens/`.

#### Scenario: All initial modules exist
- **WHEN** a developer lists the backend directory
- **THEN** all 10 module folders are present

### Requirement: Backend core directory
The backend SHALL have a `/core/` directory at the same level as the modules for shared infrastructure code.

#### Scenario: Core directory exists
- **WHEN** a developer lists the backend directory
- **THEN** a `core/` folder exists alongside the module folders

### Requirement: Frontend FSD structure
The frontend SHALL follow Feature-Sliced Design with exactly these layer directories: `app/`, `pages/`, `widgets/`, `features/`, `entities/`, `shared/`.

#### Scenario: Frontend has all FSD layers
- **WHEN** a developer lists the frontend/src directory
- **THEN** all 6 layer folders exist
- **AND** no additional layer folders are created outside this list

### Requirement: Frontend import direction
The frontend SHALL enforce that imports only flow from higher-abstraction layers to lower-abstraction layers: `app` → `pages` → `widgets` → `features` → `entities` → `shared`.

#### Scenario: No reverse imports
- **WHEN** code in `features/` attempts to import from `pages/`
- **THEN** the import SHALL be rejected by linting rules
- **AND** an error SHALL be thrown during build

### Requirement: Gitignore protection
The repository SHALL have a `.gitignore` that excludes at minimum: `.env`, `__pycache__/`, `node_modules/`, `.venv/`, `*.pyc`, `dist/`, `.DS_Store`, and OS-specific temporary files.

#### Scenario: Env file is not tracked
- **WHEN** a developer creates a `.env` file
- **AND** runs `git status`
- **THEN** the `.env` file is NOT shown as an untracked file

### Requirement: Root README with setup instructions
The root directory SHALL contain a `README.md` with instructions to: clone the repo, install backend dependencies, install frontend dependencies, and run each stack.

#### Scenario: README covers all setup steps
- **WHEN** a new developer reads the README
- **THEN** they can clone, install, and run both stacks without additional documentation

### Requirement: Backend env example
The backend SHALL have a `.env.example` file documenting all environment variables with example values, including: DATABASE_URL, SECRET_KEY, ALGORITHM, ACCESS_TOKEN_EXPIRE_MINUTES, REFRESH_TOKEN_EXPIRE_DAYS, CORS_ORIGINS, MERCADOPAGO_ACCESS_TOKEN, MERCADOPAGO_PUBLIC_KEY.

#### Scenario: Env example is complete
- **WHEN** a developer copies `.env.example` to `.env`
- **THEN** all required variables are present with documented example values

### Requirement: Frontend env example
The frontend SHALL have a `.env.example` file documenting all environment variables with example values, including: VITE_API_BASE_URL, VITE_MERCADOPAGO_PUBLIC_KEY.

#### Scenario: Frontend env example is complete
- **WHEN** a developer copies `.env.example` to `.env`
- **THEN** all required variables are present with documented example values

### Requirement: Conventional commits
All commits SHALL follow Conventional Commits format: `type(scope): description` where type is one of: feat, fix, docs, style, refactor, test, chore, perf, build, ci.

#### Scenario: Commit follows conventional format
- **WHEN** a developer makes a commit
- **THEN** the message format is `type(scope): description`
- **AND** the type is from the allowed list