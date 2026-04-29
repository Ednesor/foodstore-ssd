## Why

Food Store necesita una estructura base organizada y consistente desde el primer día. Sin un scaffolding bien definido, el equipo comenzaría a escribir código en estructuras improvisadas, generando inconsistencias, imports cruzados, y deuda técnica que después cuesta años en reparar. Este change establece el punto de partida del monorepo con las convenciones de arquitectura que se usarán en todo el proyecto: feature-first en el backend y Feature-Sliced Design en el frontend.

## What Changes

- Se crea la estructura inicial del monorepo con `/backend` y `/frontend` como carpetas raíz
- Backend: estructura feature-first con carpetas por módulo (`auth/`, `usuarios/`, `productos/`, `categorias/`, `ingredientes/`, `pedidos/`, `pagos/`, `direcciones/`, `admin/`, `refreshtokens/`), cada una conteniendo `model.py`, `schemas.py`, `repository.py`, `service.py`, `router.py`
- Frontend: estructura Feature-Sliced Design con capas `app/`, `pages/`, `widgets/`, `features/`, `entities/`, `shared/`
- Se crea `.gitignore` completo para ambos proyectos
- Se crea `README.md` raíz con instrucciones de setup (clonar, instalar dependencias, ejecutar)
- Se crea `.env.example` en backend y frontend con todas las variables de entorno documentadas y valores de ejemplo

## Capabilities

### New Capabilities

- `project-structure`: Define la arquitectura de carpetas del monorepo, las convenciones de nombrado, y los archivos base que todo desarrollo posterior debe seguir. Esta es la columna vertebral que conecta todos los cambios futuros.

## Impact

- Todo el proyecto depende de esta estructura. Ningún otro change puede implementarse sin ella.
- Backend: establece cómo se organizan los módulos y qué archivos debe tener cada uno
- Frontend: establece el patrón FSD con sus reglas de imports entre capas
- Git: el `.gitignore` protege secretos y archivos generados de entrar al repositorio
- Setup: el README y `.env.example` reducen la fricción de onboarding para nuevos desarrolladores