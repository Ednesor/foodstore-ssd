---
name: trello-task-tracker
description: >
  Gestiona tareas del equipo en Trello directamente desde el chat usando la API REST de Trello.
  Usar SIEMPRE que el usuario mencione tareas, cards, tablero, Trello, pendientes, standup, o quiera crear/mover/consultar trabajo del equipo.
  Activar ante frases como: "creá una card", "qué tenemos pendiente", "mové esto a done", "abrí una tarea para", "qué está en progreso", "resumen del equipo", "agregá al tablero", "¿qué falta hacer?", "marcá como terminado".
  También activar cuando el usuario termine de implementar algo y quiera registrarlo, o cuando pregunte sobre el estado del trabajo del equipo.
---

# Trello Task Tracker

Skill para gestionar tareas del equipo en Trello desde opencode. Permite crear cards, consultar el estado del tablero y mover tareas entre listas sin salir del flujo de código.

---

## Configuración inicial (hacer UNA VEZ por miembro del equipo)

Antes de cualquier operación, verificar si el usuario tiene sus credenciales configuradas. Si no las tiene, guiarlo con estas instrucciones:

### Obtener credenciales de Trello

1. **API Key**: Ir a https://trello.com/power-ups/admin → crear una Power-Up → copiar la API Key
   - O directamente: https://trello.com/app-key
2. **Token**: En la misma página, hacer clic en "Token" → autorizar → copiar el token generado
3. **Board ID**: Abrir el tablero en Trello → agregar `.json` al final de la URL → copiar el valor del campo `"id"`

### Variables de entorno recomendadas

Pedir al usuario que agregue estas variables a su entorno (`.env`, `.zshrc`, o config de opencode):

```bash
TRELLO_API_KEY=tu_api_key_aqui
TRELLO_TOKEN=tu_token_aqui
TRELLO_BOARD_ID=id_del_tablero_aqui
```

Si no están configuradas como variables de entorno, pedirlas una sola vez y usarlas durante la sesión. Nunca mostrarlas en outputs ni logs.

---

## Estructura del tablero recomendada

Si el equipo todavía no tiene un tablero, sugerir crear uno con estas listas en este orden:

```
📋 Backlog       → tareas identificadas pero no iniciadas
🔄 En Progreso   → en lo que alguien está trabajando ahora
👀 En Revisión   → listo para review o testing
✅ Hecho         → completado
🚫 Bloqueado     → no puede avanzar por algún motivo externo
```

Ofrecer crear el tablero y todas las listas automáticamente con la API si el usuario lo pide.

---

## Operaciones principales

### 1. Crear una card

**Cuándo usar**: El usuario menciona una tarea nueva, termina de identificar un bug, o quiere registrar algo mientras codea.

**Flujo**:
1. Inferir el título de la card desde el contexto del chat (no pedir confirmación si es obvio)
2. Preguntar solo si no está claro: ¿en qué lista va? (default: `Backlog`)
3. Crear la card y confirmar con el link directo

**Llamada a la API**:
```
POST https://api.trello.com/1/cards
  ?key={TRELLO_API_KEY}
  &token={TRELLO_TOKEN}
  &idList={id_de_la_lista}
  &name={titulo}
  &desc={descripcion_opcional}
```

Ver referencia completa en `references/api.md` → sección "Cards"

---

### 2. Consultar el tablero

**Cuándo usar**: El usuario pregunta qué hay pendiente, en progreso, o quiere un resumen.

**Flujo**:
1. Obtener todas las listas del tablero
2. Para cada lista relevante, obtener sus cards
3. Presentar en formato limpio y legible (ver formato de respuesta más abajo)

**Llamadas a la API**:
```
GET https://api.trello.com/1/boards/{BOARD_ID}/lists
  ?key={TRELLO_API_KEY}&token={TRELLO_TOKEN}

GET https://api.trello.com/1/lists/{LIST_ID}/cards
  ?key={TRELLO_API_KEY}&token={TRELLO_TOKEN}
```

---

### 3. Mover una card

**Cuándo usar**: El usuario termina algo, quiere avanzar una tarea, o dice "pasá X a done/en progreso/etc."

**Flujo**:
1. Identificar la card por nombre (buscar en todas las listas si es necesario)
2. Si hay ambigüedad (dos cards parecidas), mostrar opciones antes de mover
3. Mover y confirmar

**Llamada a la API**:
```
PUT https://api.trello.com/1/cards/{CARD_ID}
  ?key={TRELLO_API_KEY}
  &token={TRELLO_TOKEN}
  &idList={id_lista_destino}
```

---

## Formato de respuesta al consultar el tablero

Usar este formato compacto y legible:

```
📋 **Backlog** (3)
  • Integrar login con Google
  • Revisar performance de queries
  • Documentar endpoints de API

🔄 **En Progreso** (2)
  • Módulo de facturación → asignado a: Leandro
  • Fix bug en formulario de registro

✅ **Hecho hoy** (1)
  • Setup inicial del proyecto
```

Si hay cards en **Bloqueado**, siempre mencionarlas con énfasis aunque no se hayan pedido explícitamente.

---

## Resumen de standup

Si el usuario pide un "resumen del equipo" o "standup", generar este formato:

```
📊 Estado del tablero — [fecha]

✅ Completado recientemente: X cards
🔄 En progreso ahora: X cards
📋 Backlog pendiente: X cards
🚫 Bloqueado: X cards  ← si hay alguno, siempre mencionar

🔴 Atención: [lista de cards bloqueadas o sin movimiento en más de 3 días, si aplica]
```

---

## Reglas de comportamiento

- **No pedir confirmación para crear cards obvias**: si el usuario dice "abrí una card para el bug del login", hacerlo directamente.
- **Inferir la lista correcta** según el contexto: si el usuario está empezando algo → "En Progreso"; si lo terminó → "Hecho"; si lo identificó pero no lo va a hacer ya → "Backlog".
- **Usar el contexto del código**: si hay un error o feature discutida en el chat, usar ese contexto para completar el título y descripción de la card automáticamente.
- **Ser conciso en las confirmaciones**: después de crear o mover una card, responder en una línea con el nombre y la lista destino. No repetir toda la info.
- **Manejar errores de API con gracia**: si falla por credenciales, guiar al usuario a re-configurar. Si falla por otro motivo, mostrar el mensaje de error de Trello y sugerir qué revisar.

---

## Dependencias y compatibilidad

- No requiere librerías externas: usa `curl` o `fetch` nativo según el entorno
- Compatible con cualquier entorno que pueda hacer HTTP requests
- La API de Trello no requiere OAuth para operaciones básicas con API Key + Token personal

Para referencia completa de endpoints, ver `references/api.md`.
