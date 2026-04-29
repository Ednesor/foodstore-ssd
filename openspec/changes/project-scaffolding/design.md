## Context

Food Store es un sistema de e-commerce full-stack con React + TypeScript (frontend) y FastAPI + PostgreSQL (backend). El proyecto se inicia desde cero y necesita una estructura que soporte desarrollo en equipo sinStep 1: establecer la estructura del monorepo con `/backend` y `/frontend` como carpetas raíz independientes. El frontend sigue Feature-Sliced Design (FSD) y el backend sigue arquitectura feature-first con capas (Router → Service → UoW → Repository → Model).

## Goals / Non-Goals

**Goals:**
- Establecer la estructura de carpetas que se usará en TODO el proyecto
- Definir las convenciones de nombrado (kebab-case para archivos, snake_case para Python, camelCase para TypeScript)
- Crear los archivos base vacíos o minimalmente documentados que cada módulo necesita
- Proteger el repositorio de secretos y archivos generados con `.gitignore`
- Reducir la fricción de onboarding con README y `.env.example`

**Non-Goals:**
- No se implementa ninguna funcionalidad de negocio en este change
- No se configura build ni se instalan dependencias todavía (eso corresponde a `backend-core-setup` y `frontend-core-setup`)
- No se crea código funcional en los módulos — solo la estructura y archivos base

## Decisions

### D1: Monorepo con `/backend` y `/frontend` como raíz

**Decisión:** Usar un monorepo con dos carpetas principales en la raíz en lugar de dos repositorios separados.

**Rationale:** Simplifica el setup de herramientas compartir archivos de configuración como `.gitignore` y `package.json` raíz si fuera necesario, y permite trabajar con GitOps simplificado (un solo repositorio para CI/CD).

**Alternativas consideradas:**
- Dos repositorios separados: complica el setup de CI/CD y el compartir convenciones
- Estructura flat (sin `/backend` ni `/frontend`): mezcla archivos de ambos stacks

### D2: Backend feature-first (vertical slice)

**Decisión:** Cada módulo funcional es una carpeta autocontenida con todos sus archivos (`model.py`, `schemas.py`, `repository.py`, `service.py`, `router.py`).

**Rationale:** Facilita la navegación (todo lo relacionado a "productos" está en un solo lugar), reduce los imports cruzados por diseño, y hace que agregar una nueva funcionalidad signifique crear una carpeta en lugar de tocar múltiples carpetas horizontales.

**Alternativas consideradas:**
- Estructura por capa (todos los models juntos, todos los services juntos): genera imports cruzados y dificultad para ubicar código relacionado
- Estructura flat: caos total en proyectos grandes

### D3: Frontend Feature-Sliced Design (FSD)

**Decisión:** Usar FSD con las capas `shared/` → `entities/` → `features/` → `widgets/` → `pages/` → `app/`.

**Rationale:** FSD escala bien en proyectos grandes, fuerza separación clara entre código compartido y código de negocio, y tiene reglas de imports entre capas que previenen acoplamiento.

**Alternativas consideradas:**
- Estructura por tipo de archivo (components/, hooks/, store/): escala mal, genera imports cruzados
- Feature folders planos: sin separación de capas, se mezcla todo

### D4: `.env.example` en ambos proyectos

**Decisión:** Documentar TODAS las variables de entorno con valores de ejemplo, sin valores reales.

**Rationale:** Permite a cualquier desarrollador copiar `.env.example` a `.env` y tener un punto de partida. Los secretos reales nunca van al repositorio (protegido por `.gitignore`).

### D5: Módulos iniciales del backend

**Decisión:** Crear carpetas para los 10 módulos funcionales desde el inicio: `auth/`, `usuarios/`, `productos/`, `categorias/`, `ingredientes/`, `pedidos/`, `pagos/`, `direcciones/`, `admin/`, `refreshtokens/`.

**Rationale:** Establece desde el inicio la lista completa de módulos y evita debates posteriores sobre cómo nombrar nuevas carpetas. Los módulos vacíos son cheap; la convención es lo valioso.

## Risks / Trade-offs

- **[Risk] Decisión de monorepo puede limitar escalabilidad futura** → Si el proyecto crece mucho y el monorepo se vuelve un problema, se puede migrar a un polyrepo más adelante sin mucho costo. La estructura interna de cada stack no cambia.
- **[Risk] FSD tiene curva de aprendizaje** → Invertir tiempo en documentar las reglas de imports entre capas y verificar que se cumplan (ESLint plugin existe).
- **[Trade-off] Estructura feature-first requiere más disciplina en imports cruzados** → Se mitiga con linting y convenciones claras. El beneficio de localize todo junto supera el costo.

## Migration Plan

Este change es puramente additive (nueva estructura). No hay migración. Los cambios se integran en un solo commit inicial.

**Rollback:** Si la estructura no funciona, se hace un commit que reorganiza. Como es solo estructura y no lógica, el rollback es rápido.

## Open Questions

- ¿Se necesita una carpeta compartida `/shared/` en el root para archivos comunes a ambos stacks (por ejemplo, tipos TypeScript compartidos)?
  - **Decisión pendiente:** Empezar sin ella. Si emerge la necesidad, se crea cuando surja.
- ¿Se usa convencional commits desde el primer commit o se permite otro formato?
  - **Decisión:** Conventional commits (`feat:`, `fix:`, `docs:`, etc.) desde el inicio para mantener historial limpio