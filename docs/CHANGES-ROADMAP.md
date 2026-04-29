# Changes — Roadmap Food Store

---

## change: project-scaffolding

### 🎯 Objetivo
Sentar la estructura base del monorepo con carpetas backend (feature-first) y frontend (Feature-Sliced Design), ready para que el equipo empiece a trabajar.

### 🧩 Funcionalidad cubierta
- Estructura monorepo con `/backend` y `/frontend`
- Backend feature-first: carpetas por módulo (`auth/`, `usuarios/`, `productos/`, `categorias/`, `ingredientes/`, `pedidos/`, `pagos/`, `direcciones/`, `admin/`, `refreshtokens/`), cada una con `model.py`, `schemas.py`, `repository.py`, `service.py`, `router.py`
- Frontend FSD: `app/`, `pages/`, `widgets/`, `features/`, `entities/`, `shared/`
- `.gitignore` completo
- `README.md` raíz con instrucciones de setup
- `.env.example` en backend y frontend

### 👤 Historias de usuario implementadas
- Como **Líder Técnico**, quiero tener el repositorio Git inicializado con la estructura de carpetas del backend (feature-first) y del frontend (Feature-Sliced Design), para que el equipo pueda comenzar a desarrollar sobre una base organizada y consistente. (US-000)

### 🔗 Dependencias
- Ninguna (punto de partida)

### 🏗 Tipo de incremento
`foundation`

---

## change: backend-core-setup

### 🎯 Objetivo
Configurar FastAPI con todas las dependencias core del backend, middleware de CORS, rate limiting, y configuración centralizada via variables de entorno.

### 🧩 Funcionalidad cubierta
- FastAPI app con CORS middleware configurado
- Rate limiting con slowapi (5 intentos/15min en login)
- Configuración centralizada en `core/config.py` (DATABASE_URL, SECRET_KEY, JWT_ACCESS_TOKEN_EXPIRE_MINUTES=30, JWT_REFRESH_TOKEN_EXPIRE_DAYS=7, CORS_ORIGINS, MERCADOPAGO_ACCESS_TOKEN, MERCADOPAGO_PUBLIC_KEY)
- Database engine y session factory en `core/database.py`
- Security utilities (hashing bcrypt, JWT) en `core/security.py`
- RFC 7807 error handler global
- Documentación Swagger en `/docs` y ReDoc en `/redoc`
- Registro de routers con prefijo `/api/v1`

### 👤 Historias de usuario implementadas
- Como **Desarrollador**, quiero tener el proyecto backend configurado con FastAPI, SQLModel, Alembic y todas las dependencias necesarias, para poder comenzar a implementar los módulos funcionales. (US-000a)

### 🔗 Dependencias
- `project-scaffolding` — necesita la estructura de carpetas existente

### 🏗 Tipo de incremento
`foundation`

---

## change: database-setup

### 🎯 Objetivo
Crear el esquema completo de PostgreSQL con todas las tablas del ERD v5, migraciones Alembic versionadas, y script de seed idempotente con los datos iniciales obligatorios.

### 🧩 Funcionalidad cubierta
- Todas las tablas: Usuario, Rol, UsuarioRol, RefreshToken, DireccionEntrega, Categoria, Producto, Ingrediente, ProductoCategoria, ProductoIngrediente, FormaPago, EstadoPedido, Pedido, DetallePedido, HistorialEstadoPedido, Pago
- Campos de auditoría (`creado_en`, `actualizado_en`) en todas las tablas principales
- Soft delete (`eliminado_en`) en tablas que lo soportan
- Tipos correctos: precios NUMERIC(10,2), personalizacion INTEGER[], email UNIQUE
- Claves foráneas y restricciones de integridad referencial
- Categoria con `padre_id` FK autoreferencial
- UsuarioRol con UNIQUE compuesta (usuario_id, rol_id)
- Seed data: 4 Roles (ADMIN=1, STOCK=2, PEDIDOS=3, CLIENT=4), 6 EstadosPedido, 2 FormasPago, 1 admin seed
- Migraciones reversibles con Alembic
- Script de seed idempotente (INSERT ON CONFLICT DO NOTHING)

### 👤 Historias de usuario implementadas
- Como **Desarrollador**, quiero tener PostgreSQL configurado con Alembic para migraciones y un script de seed que cargue los datos iniciales, para que el sistema tenga las tablas y datos catálogo necesarios para funcionar. (US-000b)

### 🔗 Dependencias
- `backend-core-setup` — necesita FastAPI configurado para definir modelos

### 🏗 Tipo de incremento
`foundation`

---

## change: backend-patterns

### 🎯 Objetivo
Implementar los patrones de infraestructura base del backend: BaseRepository[T] genérico, Unit of Work como context manager, get_current_user y require_role.

### 🧩 Funcionalidad cubierta
- `BaseRepository[T]` genérico con: `get_by_id`, `list_all` (excluye eliminados por defecto), `count`, `create` (flush), `update`, `soft_delete`, `hard_delete`
- `UnitOfWork` como context manager (`async with`): abre sesión, expone repos como atributos, commit al salir, rollback automático en excepción
- Dependencia `get_current_user`: extrae Bearer token, decodifica JWT, verifica firma y expiración, retorna Usuario o lanza 401
- Dependencia factory `require_role(roles: list[str])`: verifica que el usuario tenga al menos un rol, lanza 403 si no
- Exception handler RFC 7807 con clases: ValidationError, UnauthorizedError, ForbiddenError, NotFoundError

### 👤 Historias de usuario implementadas
- Como **Desarrollador**, quiero tener implementados el BaseRepository genérico, el Unit of Work y las dependencias de FastAPI (get_current_user, require_role), para que los módulos funcionales puedan construirse sobre una base sólida y consistente. (US-000d)

### 🔗 Dependencias
- `database-setup` — necesita las tablas y modelos SQLModel existentes

### 🏗 Tipo de incremento
`foundation`

---

## change: frontend-core-setup

### 🎯 Objetivo
Configurar el proyecto frontend con React + TypeScript + Vite, todas las dependencias core (TanStack Query, TanStack Form, Zustand, Axios, Tailwind, MercadoPago SDK), y la configuración base de Axios con interceptors JWT.

### 🧩 Funcionalidad cubierta
- React 18 + TypeScript (strict mode)
- Vite con SWC para fast refresh
- TanStack Query con QueryClientProvider en App root
- TanStack Form configurado
- Axios instance centralizada con:
  - Base URL desde `VITE_API_BASE_URL`
  - Request interceptor que adjunta access token al header `Authorization: Bearer`
  - Response interceptor que al recibir 401 hace refresh automático, actualiza authStore y reintenta la request original
- Tailwind CSS con PostCSS y purging en producción
- Configuración de routing con react-router-dom
- `.env.example` con todas las variables documentadas

### 👤 Historias de usuario implementadas
- Como **Desarrollador**, quiero tener el proyecto frontend configurado con React, TypeScript, Vite y todas las librerías necesarias, para poder comenzar a construir la interfaz de usuario. (US-000c)

### 🔗 Dependencias
- `project-scaffolding` — necesita la estructura de carpetas del frontend

### 🏗 Tipo de incremento
`foundation`

---

## change: frontend-state-stores

### 🎯 Objetivo
Implementar los cuatro stores de Zustand con persistencia selectiva y separación correcta entre estado del cliente (Zustand) y estado del servidor (TanStack Query).

### 🧩 Funcionalidad cubierta
- `authStore`: estado (accessToken, refreshToken, user, isAuthenticated), acciones (login, logout, updateTokens), selectores (isAuthenticated, hasRole), persistencia en localStorage con `partialize` excluyendo estados transitorios
- `cartStore`: estado (items array con productoId, producto, cantidad, personalizacion), acciones (addItem, removeItem, updateQuantity, clearCart), selectores (totalItems, totalPrice, getItem), persistencia completa en localStorage
- `paymentStore`: estado (checkoutStep, preferenceId, paymentStatus, error), SIN persistencia (se resetea al recargar)
- `uiStore`: estado (theme, sidebarOpen, toasts), persistencia selectiva solo de theme
- Acceso fuera de React via `useAuthStore.getState()` para interceptores

### 👤 Historias de usuario implementadas
- Como **Desarrollador**, quiero tener los cuatro stores de Zustand configurados con sus acciones base y persistencia, para que el frontend tenga una gestión de estado consistente desde el inicio. (US-000e)

### 🔗 Dependencias
- `frontend-core-setup` — necesita Zustand instalado y Axios configurado

### 🏗 Tipo de incremento
`foundation`

---

## change: authentication-backend

### 🎯 Objetivo
Implementar el flujo completo de autenticación JWT: registro de cliente (con rol CLIENT automático), login con rate limiting, refresh con rotación de tokens, y logout con invalidación de refresh token.

### 🧩 Funcionalidad cubierta
- `POST /api/v1/auth/register`: crea usuario con contraseña hasheada bcrypt, asigna rol CLIENT automáticamente, retorna tokens
- `POST /api/v1/auth/login`: valida credenciales, retorna access token (30 min) + refresh token (7 días), rate limited 5/15min por IP
- `POST /api/v1/auth/refresh`: rota refresh token (invalida anterior, genera nuevo par), detecta replay attack y revoca todos los tokens del usuario
- `POST /api/v1/auth/logout`: marca refresh token como revocado en BD
- `GET /api/v1/auth/me`: retorna datos del usuario autenticado
- bcrypt con cost factor ≥ 10, salt automático
- Access token JWT con payload: userId, email, roles, exp (30 min)
- Refresh token UUID v4 opaco almacenado en tabla RefreshToken

### 👤 Historias de usuario implementadas
- Como **Cliente**, quiero registrarme en la plataforma con mi email y contraseña, para poder acceder a las funcionalidades de compra. (US-001)
- Como **Cliente**, quiero iniciar sesión con mis credenciales, para acceder a mi cuenta y realizar compras. (US-002)
- Como **Sistema**, quiero rotar los tokens de acceso usando el refresh token, para mantener la sesión del usuario activa de forma segura. (US-003)
- Como **Cliente**, quiero cerrar mi sesión, para proteger mi cuenta cuando dejo de usar la plataforma. (US-004)

### 🔗 Dependencias
- `backend-patterns` — necesita BaseRepository, UoW, get_current_user
- `database-setup` — necesita tabla RefreshToken y seed de roles

### 🏗 Tipo de incremento
`feature`

---

## change: rbac-backend

### 🎯 Objetivo
Implementar la gestión de roles RBAC: asignación de roles por Admin, verificación de permisos por endpoint, y protecciones específicas (no quitarse ADMIN a sí mismo si es el último, ownership checks).

### 🧩 Funcionalidad cubierta
- `PUT /api/v1/admin/users/{id}/role`: Admin asigna roles a un usuario (ADMIN, STOCK, PEDIDOS, CLIENT)
- Validación: un ADMIN no puede quitarse el rol ADMIN a sí mismo si es el último administrador
- Validación: solo ADMIN puede modificar roles de otros usuarios
- Refresh tokens del usuario modificado se invalidan al cambiar su rol
- Middleware/protection para endpoints que requieren roles específicos
- Validación de `activo` en login: si usuario está desactivado, retorna 403

### 👤 Historias de usuario implementadas
- Como **Admin**, quiero asignar roles a los usuarios del sistema, para controlar el acceso a las distintas funcionalidades. (US-005)
- Como **Sistema**, quiero proteger cada endpoint según el rol requerido, para garantizar que solo usuarios autorizados accedan a cada recurso. (US-006)

### 🔗 Dependencias
- `authentication-backend` — necesita el flujo de login y get_current_user
- `database-setup` — necesita seed de roles

### 🏗 Tipo de incremento
`feature`

---

## change: navigation-frontend

### 🎯 Objetivo
Implementar la navegación adaptativa por rol y los guards de ruta en el frontend para proteger rutas según autenticación y rol del usuario.

### 🧩 Funcionalidad cubierta
- Componente Navigation/Sidebar que muestra opciones según rol del JWT decodificado:
  - CLIENT: Catálogo, Mi Carrito, Mis Pedidos, Mi Perfil, Mis Direcciones
  - STOCK: Productos, Categorías, Ingredientes, Stock
  - PEDIDOS: Panel de Pedidos
  - ADMIN: todas las opciones + Usuarios + Métricas + Configuración
  - No autenticado: Catálogo, Login, Registrarse
- Route guards basados en rol: redirigir a login si no autenticado, mostrar 403 o redirigir si rol insuficiente
- Lazy loading de módulos por rol
- ProtectedRoute HOC

### 👤 Historias de usuario implementadas
- Como **usuario del sistema**, quiero ver solo las opciones de menú correspondientes a mi rol, para tener una interfaz limpia y enfocada en mis tareas. (US-075)
- Como **Sistema**, quiero proteger las rutas del frontend según autenticación y rol, para evitar que usuarios accedan a vistas no autorizadas. (US-076)

### 🔗 Dependencias
- `authentication-backend` — necesita login funcional para probar los flows
- `frontend-state-stores` — necesita authStore con roles

### 🏗 Tipo de incremento
`feature`

---

## change: axios-interceptors

### 🎯 Objetivo
Implementar la renovación transparente de sesión: interceptor que detecta 401, renueva token con queue de requests pendientes, y maneja errores globales de forma consistente.

### 🧩 Funcionalidad cubierta
- Interceptor Axios que detecta 401 y llama refresh endpoint automáticamente
- Singleton de refresh en progreso: si múltiples requests fallan concurrentemente, solo se hace un refresh y las demás esperan
- Cola de requests pendientes que se reintentan post-refresh
- Si refresh token también falla: redirigir a login
- Proceso transparente para el usuario (no ve errores 401 intermitentes)
- Interceptor de response que mapea códigos a mensajes de error usables

### 👤 Historias de usuario implementadas
- Como **Sistema**, quiero interceptar respuestas 401 en el frontend y renovar el token automáticamente, para que el cliente no pierda su sesión por expiración del access token. (US-066)
- Como **Cliente**, quiero ver mensajes de error claros cuando algo falla, para entender qué pasó y qué puedo hacer. (US-067)

### 🔗 Dependencias
- `authentication-backend` — necesita endpoint de refresh funcionando
- `frontend-state-stores` — necesita authStore para guardar tokens

### 🏗 Tipo de incremento
`feature`

---

## change: categories-management

### 🎯 Objetivo
Implementar el CRUD completo de categorías con soporte jerárquico (FK autoreferencial, profundidad arbitraria via CTE recursiva) y validación de ciclos.

### 🧩 Funcionalidad cubierta
- `POST /api/v1/categorias`: crea categoría con nombre, descripción, imagen opcional, padre_id (nullable para raíces)
- `GET /api/v1/categorias`: listado público del árbol jerárquico completo (CTE recursiva), paginación
- `GET /api/v1/categorias/{id}`: detalle de categoría con subcategorías anidadas
- `PUT /api/v1/categorias/{id}`: edita nombre, descripción, imagen, padre_id; valida que no genera ciclos
- `DELETE /api/v1/categorias/{id}`: soft delete solo si no tiene productos activos asociados; reasignar o eliminar subcategorías primero
- No permitir padre de sí misma ni ciclos

### 👤 Historias de usuario implementadas
- Como **Gestor de Stock**, quiero crear categorías para organizar los productos, para que los clientes encuentren lo que buscan más fácilmente. (US-007)
- Como **Cliente**, quiero ver las categorías organizadas en forma jerárquica, para navegar el catálogo de manera intuitiva. (US-008)
- Como **Gestor de Stock**, quiero editar el nombre o la jerarquía de una categoría, para mantener el catálogo organizado correctamente. (US-009)
- Como **Gestor de Stock**, quiero dar de baja una categoría que ya no se usa, para mantener el catálogo limpio sin perder datos históricos. (US-010)

### 🔗 Dependencias
- `backend-patterns` — necesita BaseRepository y UoW
- `rbac-backend` — necesita protección de rutas por rol

### 🏗 Tipo de incremento
`feature`

---

## change: ingredients-management

### 🎯 Objetivo
Implementar el CRUD completo de ingredientes con flag de alergeno para gestionar la composición de productos y las restricciones alimentarias de los clientes.

### 🧩 Funcionalidad cubierta
- `POST /api/v1/ingredientes`: crea ingrediente con nombre (único) y flag esAlergeno (booleano)
- `GET /api/v1/ingredientes`: listado con filtros opcionales (solo alergenos), paginación
- `GET /api/v1/ingredientes/{id}`: detalle
- `PUT /api/v1/ingredientes/{id}`: edita nombre o flag de alergeno; nombre único
- `DELETE /api/v1/ingredientes/{id}`: soft delete

### 👤 Historias de usuario implementadas
- Como **Gestor de Stock**, quiero registrar ingredientes indicando si son alergenos, para informar correctamente a los clientes sobre la composición de los productos. (US-011)
- Como **Gestor de Stock**, quiero ver todos los ingredientes registrados, para gestionar su asociación con productos. (US-012)
- Como **Gestor de Stock**, quiero editar un ingrediente existente, para corregir datos o actualizar su clasificación como alergeno. (US-013)
- Como **Gestor de Stock**, quiero dar de baja un ingrediente, para que no se pueda asociar a nuevos productos sin perder los registros históricos. (US-014)

### 🔗 Dependencias
- `backend-patterns` — necesita BaseRepository y UoW
- `rbac-backend` — necesita protección de rutas por rol

### 🏗 Tipo de incremento
`feature`

---

## change: products-management

### 🎯 Objetivo
Implementar el CRUD completo de productos con gestión de stock, asociación a categorías (M2M) y a ingredientes (M2M con flag es_removible), y visibilidad según disponibilidad.

### 🧩 Funcionalidad cubierta
- `POST /api/v1/productos`: alta de producto con nombre, descripción, precio (NUMERIC), stock_cantidad (entero ≥ 0), imagen, disponible (default true)
- `PUT /api/v1/productos/{id}`: edición de cualquier campo; valida precio > 0, stock ≥ 0
- `PATCH /api/v1/productos/{id}/stock`: actualiza stock_cantidad; operación atómica
- `PATCH /api/v1/productos/{id}/disponibilidad`: toggla disponible (true/false)
- `DELETE /api/v1/productos/{id}`: soft delete
- `PUT /api/v1/productos/{id}/categorias`: asocia/desasicia categorías (M2M via ProductoCategoria)
- `PUT /api/v1/productos/{id}/ingredientes`: asocia/desasicia ingredientes (M2M via ProductoIngrediente con es_removible)

### 👤 Historias de usuario implementadas
- Como **Gestor de Stock**, quiero dar de alta un producto con su precio, stock, imagen y descripción, para que los clientes puedan verlo y comprarlo. (US-015)
- Como **Gestor de Stock**, quiero asociar un producto a una o más categorías, para que aparezca en las secciones correctas del catálogo. (US-016)
- Como **Gestor de Stock**, quiero asociar ingredientes a un producto, para que los clientes conozco su composición y alergenos. (US-017)
- Como **Gestor de Stock**, quiero editar los datos de un producto existente, para mantener la información del catálogo actualizada. (US-020)
- Como **Gestor de Stock**, quiero actualizar la cantidad en stock de un producto, para reflejar entradas y salidas de mercancía. (US-021)
- Como **Gestor de Stock**, quiero dar de baja un producto, para que no aparezca en el catálogo sin perder los datos históricos asociados a pedidos. (US-022)
- Como **Admin**, quiero tener acceso completo a la gestión del catálogo (productos, categorías, ingredientes), para intervenir cuando sea necesario sin depender del Gestor de Stock. (US-064)

### 🔗 Dependencias
- `categories-management` — necesita categorías existentes para asociar
- `ingredients-management` — necesita ingredientes existentes para asociar
- `rbac-backend` — necesita protección de rutas por rol

### 🏗 Tipo de incremento
`feature`

---

## change: product-catalog-public

### 🎯 Objetivo
Exponer el catálogo público de productos con filtros, búsqueda y paginación, incluyendo detalle con ingredientes y alérgenos para que los clientes tomen decisiones informadas.

### 🧩 Funcionalidad cubierta
- `GET /api/v1/productos`: listado público (solo disponible=true, eliminado=false) con paginación, filtros por categoría, búsqueda por nombre (ILIKE), filtro por rango de precio, filtro por exclusión de alérgenos
- `GET /api/v1/productos/{id}`: detalle público con ingredientes (con flag es_alergeno destacado), categorías, stock disponible (sin revelar cantidad exacta)
- Filtro por alérgenos: `GET /api/v1/productos?excluirAlergenos=1,3,7`
- Incluye conteo total para paginación en frontend

### 👤 Historias de usuario implementadas
- Como **Cliente**, quiero ver los productos disponibles con su precio, imagen y disponibilidad, para decidir qué comprar. (US-018)
- Como **Cliente**, quiero ver el detalle completo de un producto incluyendo ingredientes y alergenos, para tomar una decisión de compra informada. (US-019)
- Como **Cliente**, quiero filtrar productos que contengan determinados alergenos, para evitar alimentos que me generen reacciones alérgicas. (US-023)

### 🔗 Dependencias
- `products-management` — necesita productos dados de alta para listar

### 🏗 Tipo de incremento
`feature`

---

## change: customer-profile

### 🎯 Objetivo
Permitir al cliente ver y editar su propio perfil (nombre, teléfono) y cambiar su contraseña con invalidación forzada de todos los refresh tokens.

### 🧩 Funcionalidad cubierta
- `GET /api/v1/perfil`: cliente ve sus datos (nombre, email, teléfono, fecha de registro)
- `PUT /api/v1/perfil`: edita nombre y teléfono; email no modificable
- `PUT /api/v1/perfil/password`: cambia contraseña (password actual + nueva); valida contraseña actual con bcrypt; invalida TODOS los refresh tokens del usuario
- Validación de formato de teléfono

### 👤 Historias de usuario implementadas
- Como **Cliente**, quiero ver los datos de mi perfil, para verificar que mi información sea correcta. (US-061)
- Como **Cliente**, quiero editar mis datos personales (nombre, teléfono), para mantener mi información actualizada. (US-062)
- Como **Cliente**, quiero cambiar mi contraseña, para mantener la seguridad de mi cuenta. (US-063)

### 🔗 Dependencias
- `authentication-backend` — necesita login y get_current_user

### 🏗 Tipo de incremento
`feature`

---

## change: delivery-addresses

### 🎯 Objetivo
Implementar el CRUD completo de direcciones de entrega por cliente con dirección principal/predeterminada y validación de ownership.

### 🧩 Funcionalidad cubierta
- `POST /api/v1/direcciones`: crea dirección (calle, número, piso/depto opcional, ciudad, código postal, referencia opcional); si es la primera se marca predeterminada automáticamente
- `GET /api/v1/direcciones`: listado de direcciones propias del cliente; indica cuál es predeterminada
- `PUT /api/v1/direcciones/{id}`: edita dirección propia; valida ownership
- `DELETE /api/v1/direcciones/{id}`: elimina dirección propia; si era predeterminada, reasigna otra
- `PATCH /api/v1/direcciones/{id}/predeterminada`: establece como predeterminada; atómicamente quita el flag de la anterior

### 👤 Historias de usuario implementadas
- Como **Cliente**, quiero agregar direcciones de entrega a mi perfil, para seleccionarlas al realizar un pedido. (US-024)
- Como **Cliente**, quiero ver todas mis direcciones guardadas, para gestionar dónde recibo mis pedidos. (US-025)
- Como **Cliente**, quiero editar una dirección existente, para corregir o actualizar mis datos de entrega. (US-026)
- Como **Cliente**, quiero eliminar una dirección que ya no uso, para mantener limpio mi listado de direcciones. (US-027)
- Como **Cliente**, quiero marcar una dirección como predeterminada, para que se preseleccione al crear un pedido. (US-028)

### 🔗 Dependencias
- `authentication-backend` — necesita get_current_user para ownership

### 🏗 Tipo de incremento
`feature`

---

## change: shopping-cart

### 🎯 Objetivo
Implementar el carrito de compras client-side con Zustand: agregar productos con personalización (exclusión de ingredientes), persistencia en localStorage, y cálculo de totales.

### 🧩 Funcionalidad cubierta
- Agregar producto al carrito con cantidad; si ya existe, incrementa cantidad (no duplica)
- Personalización: excluir ingredientes (array de IDs); solo ingredientes que el producto efectivamente tiene
- Modificar cantidad de item (≥1); cantidad 0 elimina
- Eliminar item del carrito
- Ver resumen: nombre, cantidad, precio unitario, exclusiones, subtotal por item, total general
- Vaciar carrito con confirmación modal
- Persistencia en localStorage: sobrevive a cierre de navegador, refresh, logout/login
- Carrito es client-side ONLY (no existe en backend)

### 👤 Historias de usuario implementadas
- Como **Cliente**, quiero agregar productos al carrito indicando cantidad, para ir armando mi pedido. (US-029)
- Como **Cliente**, quiero excluir ingredientes de un producto al agregarlo al carrito, para personalizar mi pedido según mis preferencias o restricciones alimentarias. (US-030)
- Como **Cliente**, quiero cambiar la cantidad de un producto en el carrito, para ajustar mi pedido antes de confirmarlo. (US-031)
- Como **Cliente**, quiero quitar un producto del carrito, para descartar algo que ya no quiero pedir. (US-032)
- Como **Cliente**, quiero ver un resumen del carrito con todos los productos, cantidades, exclusiones y el total, para revisar mi pedido antes de confirmarlo. (US-033)
- Como **Cliente**, quiero vaciar el carrito de una vez, para empezar de cero si cambié de opinión. (US-034)

### 🔗 Dependencias
- `product-catalog-public` — necesita listar productos para agregar al carrito
- `products-management` — necesita tener ingredientes asociados a productos para la personalización

### 🏗 Tipo de incremento
`feature`

---

## change: order-creation

### 🎯 Objetivo
Implementar la creación atómica de pedidos desde el carrito con snapshots inmutables de precios y dirección, validación de stock DENTRO de la transacción, y registro en audit trail.

### 🧩 Funcionalidad cubierta
- `POST /api/v1/pedidos`: crea pedido atómico (UoW) con items del carrito, dirección seleccionada, forma de pago
  - Valida productos existen y disponibles
  - Valida stock suficiente (SELECT FOR UPDATE) dentro de la transacción
  - Genera snapshots: precio de cada producto (DetallePedido.precio_snapshot), datos de dirección (Pedido.direccion_snapshot)
  - Calcula total = suma(subtotales) + costo_envio
  - Crea Pedido en estado PENDIENTE
  - Crea DetallePedido por cada item
  - Crea primer HistorialEstadoPedido con estado_desde=NULL
  - Commit atómico; si falla algo, rollback total
- `GET /api/v1/pedidos`: listado propio (CLIENT) o todos (ADMIN/PEDIDOS) con filtros por estado, paginación
- `GET /api/v1/pedidos/{id}`: detalle completo con items, historial, pagos

### 👤 Historias de usuario implementadas
- Como **Cliente**, quiero confirmar mi carrito y crear un pedido, para proceder al pago y recibir mis productos. (US-035)
- Como **Sistema**, quiero validar que haya stock suficiente de cada producto al crear un pedido, para evitar ventas de productos agotados. (US-036)
- Como **Sistema**, quiero almacenar el precio de cada producto al momento de crear el pedido, para que cambios futuros de precios no afecten pedidos existentes. (US-037)
- Como **Sistema**, quiero almacenar la dirección de entrega al momento de crear el pedido, para que modificaciones futuras de la dirección no afecten pedidos en curso. (US-038)
- Como **Cliente**, quiero ver el listado de todos mis pedidos con su estado actual, para hacer seguimiento de mis compras. (US-049)
- Como **Cliente**, quiero ver el detalle completo de uno de mis pedidos, para conocer los productos, cantidades, exclusiones, dirección y estado de pago. (US-050)
- Como **Gestor de Pedidos**, quiero ver todos los pedidos del sistema con filtros por estado, para gestionar el flujo de preparación y entrega. (US-051)
- Como **Gestor de Pedidos**, quiero ver el detalle completo de cualquier pedido, para tomar decisiones sobre su procesamiento. (US-052)

### 🔗 Dependencias
- `backend-patterns` — necesita UoW para transacción atómica
- `shopping-cart` — necesita carrito del cliente
- `delivery-addresses` — necesita direcciones del cliente

### 🏗 Tipo de incremento
`feature`

---

## change: order-state-machine

### 🎯 Objetivo
Implementar la máquina de estados finitos (FSM) de 6 estados para pedidos con transiciones validadas, decremento/restauración atómica de stock, y audit trail append-only.

### 🧩 Funcionalidad cubierta
- `PATCH /api/v1/pedidos/{id}/estado` (avanzar): valida transición contra mapa FSM, ejecuta cambio de estado, registra en HistorialEstadoPedido
- `PATCH /api/v1/pedidos/{id}/cancelar`: cancela pedido según reglas; si estaba CONFIRMADO, restaura stock atómicamente
- Transiciones válidas:
  - PENDIENTE → CONFIRMADO (automático por pago aprobado)
  - CONFIRMADO → EN_PREPARACIÓN (gestor Pedidos/Admin)
  - EN_PREPARACIÓN → EN_CAMINO (gestor Pedidos/Admin)
  - EN_CAMINO → ENTREGADO (gestor Pedidos/Admin)
  - PENDIENTE → CANCELADO (Cliente/Gestor/Admin)
  - CONFIRMADO → CANCELADO (Gestor/Admin, restaura stock)
  - EN_PREPARACIÓN → CANCELADO (solo Admin, restaura stock)
- Estados terminales (no admiten transiciones salientes): ENTREGADO, CANCELADO
- Decremento de stock al confirmar: `UPDATE Producto SET stock = stock - :cant WHERE id = :id AND stock >= :cant`
- Restauración de stock al cancelar: `UPDATE Producto SET stock = stock + :cant WHERE id = :id`
- HistorialEstadoPedido append-only: solo INSERT, nunca UPDATE/DELETE
- Cada registro incluye: estado anterior, estado nuevo, timestamp, usuario/SISTEMA, observación

### 👤 Historias de usuario implementadas
- Como **Sistema**, quiero que el pedido pase automáticamente de PENDIENTE a CONFIRMADO cuando el pago es aprobado, para iniciar su preparación. (US-039)
- Como **Gestor de Pedidos**, quiero marcar un pedido confirmado como en preparación, para que el equipo de cocina comience a trabajar en él. (US-040)
- Como **Gestor de Pedidos**, quiero marcar un pedido como en camino, para indicar que fue despachado para entrega. (US-041)
- Como **Gestor de Pedidos**, quiero marcar un pedido como entregado, para cerrar su ciclo de vida. (US-042)
- Como **Gestor de Pedidos**, quiero cancelar un pedido en estado PENDIENTE o CONFIRMADO, para gestionar pedidos que no se van a completar. (US-043)
- Como **Admin**, quiero ver el historial completo de estados de un pedido, para auditar su procesamiento y resolver incidentes. (US-044)

### 🔗 Dependencias
- `order-creation` — necesita que existan pedidos para transicionar
- `rbac-backend` — necesita permisos por rol para cada transición

### 🏗 Tipo de incremento
`feature`

---

## change: payment-mercadopago

### 🎯 Objetivo
Integrar MercadoPago Checkout API con Orders para procesar pagos de forma segura (PCI SAQ-A), manejar webhooks IPN, y garantizar idempotencia con UUIDs.

### 🧩 Funcionalidad cubierta
- `POST /api/v1/pagos/crear-preferencia`: recibe pedidoId en PENDIENTE, crea preferencia de pago en MercadoPago via Orders API, devuelve URL de checkout para redirigir
- `POST /api/v1/pagos/webhook` (IPN): recibe notificaciones de MercadoPago, valida firma, consulta estado real en API MP, actualiza tabla Pago, si approved dispara transición PENDIENTE→CONFIRMADO automáticamente
- `GET /api/v1/pagos/pedido/{pedido_id}`: historial de pagos de un pedido (múltiples intentos)
- Idempotency key UUID generado por backend almacenado en Pago para evitar cobros duplicados
- Datos de tarjeta tokenizados en browser via SDK MercadoPago.js (PCI SAQ-A compliant)
- Tabla Pago completa: mp_payment_id, mp_status, external_reference, idempotency_key
- Estados MP: approved → avanza pedido, rejected → permanece PENDIENTE, pending/in_process → permanece PENDIENTE, cancelled → queda disponible para reintento
- Reintento de pago rechazado: genera nueva preferencia con nuevo idempotency key

### 👤 Historias de usuario implementadas
- Como **Cliente**, quiero pagar mi pedido a través de MercadoPago, para completar la compra de forma segura. (US-045)
- Como **Sistema**, quiero procesar las notificaciones IPN de MercadoPago, para actualizar el estado del pedido según el resultado del pago. (US-046)
- Como **Cliente**, quiero ver el estado de pago de mi pedido, para saber si el pago fue procesado correctamente. (US-047)
- Como **Cliente**, quiero poder reintentar el pago si fue rechazado, para completar mi compra sin tener que crear un nuevo pedido. (US-048)

### 🔗 Dependencias
- `order-creation` — necesita pedidos en estado PENDIENTE para crear preferencias
- `order-state-machine` — necesita la transición automática PENDIENTE→CONFIRMADO

### 🏗 Tipo de incremento
`integration`

---

## change: order-ui-feedback

### 🎯 Objetivo
Implementar la experiencia de usuario en torno a pedidos: feedback visual post-creación, retorno de MercadoPago con mensajes apropiados, y timeline de seguimiento del pedido.

### 🧩 Funcionalidad cubierta
- Pantalla de confirmación post-creación de pedido: número de pedido, resumen de items, total, dirección, estado "PENDIENTE - Esperando pago", botón para ir a pagar, botón para ver detalle
- Página de retorno de MercadoPago (success/failure/pending URLs):
  - Success: mensaje de éxito con estado actualizado del pedido
  - Failure: mensaje de rechazo con opción de reintentar
  - Pending: mensaje de proceso con indicación de esperar
- Timeline de seguimiento del pedido: visualización cronológica de estados con fechas y actores
- Polling cada 30s para actualizar estado del pedido en tiempo real mientras está PENDIENTE

### 👤 Historias de usuario implementadas
- Como **Cliente**, quiero recibir una confirmación visual clara cuando mi pedido se crea exitosamente, para saber que todo salió bien. (US-071)
- Como **Cliente**, quiero ver el resultado de mi pago al volver de MercadoPago, para saber si debo reintentar o si el pago fue exitoso. (US-072)

### 🔗 Dependencias
- `order-creation` — necesita endpoint de creación de pedido
- `payment-mercadopago` — necesita webhooks y callback URLs

### 🏗 Tipo de incremento
`feature`

---

## change: admin-dashboard

### 🎯 Objetivo
Implementar el panel de administración con métricas, gráficos con recharts, y gestión integral de pedidos, stock, usuarios y configuración.

### 🧩 Funcionalidad cubierta
- Dashboard KPIs: total ventas período, pedidos por estado, usuarios registrados, productos más vendidos
- `GET /api/v1/admin/metricas/resumen`: métricas generales con filtros por rango de fechas
- `GET /api/v1/admin/metricas/ventas`: gráfico de evolución de ventas (línea) por día/semana/mes
- `GET /api/v1/admin/metricas/productos-top`: ranking de top N productos más vendidos
- `GET /api/v1/admin/metricas/pedidos-por-estado`: distribución por estado (gráfico de torta/barras)
- Panel de gestión de pedidos: filtro por estado, fechas, búsqueda por número/cliente, paginación
- `GET /api/v1/admin/usuarios`: listado de usuarios con búsqueda, filtro por rol, paginación
- `PUT /api/v1/admin/usuarios/{id}`: editar datos y rol de usuario; no puede degradar último ADMIN
- `PATCH /api/v1/admin/usuarios/{id}/estado`: activar/desactivar usuario; invalida refresh tokens
- `GET /api/v1/admin/configuracion`: ver parámetros configurables
- `PUT /api/v1/admin/configuracion`: modificar parámetros (horarios, zona de entrega, mensajes)

### 👤 Historias de usuario implementadas
- Como **Admin**, quiero ver métricas generales del sistema (ventas, pedidos, usuarios), para tomar decisiones informadas sobre el negocio. (US-056)
- Como **Admin**, quiero ver un gráfico de evolución de ventas por día/semana/mes, para entender las tendencias del negocio. (US-057)
- Como **Admin**, quiero ver el ranking de productos más vendidos, para entender qué productos tienen mayor demanda. (US-058)
- Como **Admin**, quiero ver la distribución de pedidos por estado, para identificar cuellos de botella en el proceso. (US-059)
- Como **Admin**, quiero ver todos los usuarios registrados con su rol y estado, para gestionar el acceso al sistema. (US-053)
- Como **Admin**, quiero editar los datos y rol de cualquier usuario, para corregir información o ajustar permisos. (US-054)
- Como **Admin**, quiero desactivar un usuario, para impedir su acceso sin eliminar sus datos históricos. (US-055)
- Como **Admin**, quiero gestionar configuraciones generales del sistema, para ajustar parámetros operativos sin tocar código. (US-060)
- Como **Admin**, quiero poder gestionar cualquier pedido (ver, avanzar estado, cancelar), para resolver situaciones excepcionales. (US-065)

### 🔗 Dependencias
- `order-state-machine` — necesita FSM de pedidos
- `rbac-backend` — necesita protección ADMIN en endpoints de admin

### 🏗 Tipo de incremento
`feature`

---

## Resumen del roadmap

| # | change | tipo | dependencias clave |
|---|--------|------|-------------------|
| 01 | `project-scaffolding` | foundation | — |
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

**Total: 22 changes** organizados en 4 fases: Foundation (6) → Auth + Nav (4) → Catálogo (4) → Pedidos + Pagos (8)

Las dependencias respetan el flujo: `foundation` → `feature` → `integration`, sin ciclos. Cada change es implementable en horas, no días.