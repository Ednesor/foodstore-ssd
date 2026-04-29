# Referencia API de Trello

Base URL: `https://api.trello.com/1`
Autenticación: siempre incluir `?key={API_KEY}&token={TOKEN}` en cada llamada.

---

## Boards

### Obtener info del tablero
```
GET /boards/{BOARD_ID}
```

### Listar todas las listas del tablero
```
GET /boards/{BOARD_ID}/lists
  ?key=...&token=...
```
Respuesta: array de objetos `{ id, name, closed, pos }`

### Listar todos los miembros del tablero
```
GET /boards/{BOARD_ID}/members
  ?key=...&token=...
```
Respuesta: array de `{ id, fullName, username }`

### Crear el tablero desde cero
```
POST /boards
  ?key=...&token=...
  &name=Nombre del tablero
  &defaultLists=false
```
`defaultLists=false` evita que Trello cree listas por defecto ("To Do", "Doing", "Done").

---

## Lists (listas)

### Crear una lista en el tablero
```
POST /lists
  ?key=...&token=...
  &name=Nombre de la lista
  &idBoard={BOARD_ID}
  &pos=bottom
```
`pos` puede ser `top`, `bottom`, o un número para posición específica.

### Archivar una lista
```
PUT /lists/{LIST_ID}/closed
  ?key=...&token=...
  &value=true
```

---

## Cards

### Listar cards de una lista
```
GET /lists/{LIST_ID}/cards
  ?key=...&token=...
  &fields=id,name,desc,idMembers,labels,due,shortUrl
```

### Buscar cards por nombre en el tablero
```
GET /boards/{BOARD_ID}/cards
  ?key=...&token=...
  &fields=id,name,idList,desc,shortUrl
```
Luego filtrar por nombre en el cliente. La API de Trello no tiene búsqueda por nombre en un board directamente.

### Crear una card
```
POST /cards
  ?key=...&token=...
  &idList={LIST_ID}        ← obligatorio
  &name=Título de la card  ← obligatorio
  &desc=Descripción        ← opcional
  &due=2024-12-31T23:59:00.000Z  ← opcional, formato ISO 8601
  &idMembers={MEMBER_ID}   ← opcional, puede ser array separado por coma
  &idLabels={LABEL_ID}     ← opcional
```
Respuesta incluye `id`, `shortUrl` (link directo a la card), `name`.

### Mover una card a otra lista
```
PUT /cards/{CARD_ID}
  ?key=...&token=...
  &idList={LISTA_DESTINO_ID}
```

### Actualizar nombre o descripción
```
PUT /cards/{CARD_ID}
  ?key=...&token=...
  &name=Nuevo título
  &desc=Nueva descripción
```

### Agregar un comentario a una card
```
POST /cards/{CARD_ID}/actions/comments
  ?key=...&token=...
  &text=Texto del comentario
```

### Agregar checklist a una card
```
POST /checklists
  ?key=...&token=...
  &idCard={CARD_ID}
  &name=Nombre del checklist

POST /checklists/{CHECKLIST_ID}/checkItems
  ?key=...&token=...
  &name=Ítem del checklist
  &checked=false
```

### Archivar (no eliminar) una card
```
PUT /cards/{CARD_ID}
  ?key=...&token=...
  &closed=true
```

---

## Labels

### Listar labels del tablero
```
GET /boards/{BOARD_ID}/labels
  ?key=...&token=...
```
Colores disponibles: `green`, `yellow`, `orange`, `red`, `purple`, `blue`, `sky`, `lime`, `pink`, `black`

### Crear un label
```
POST /labels
  ?key=...&token=...
  &name=Bug
  &color=red
  &idBoard={BOARD_ID}
```

---

## Ejemplos de flujo completo

### Crear tablero desde cero con estructura recomendada

```bash
# 1. Crear el tablero
curl -s -X POST "https://api.trello.com/1/boards?key=$KEY&token=$TOKEN&name=Mi%20Equipo&defaultLists=false"
# → guardar el "id" del tablero

# 2. Crear las listas en orden
for lista in "📋 Backlog" "🔄 En Progreso" "👀 En Revisión" "✅ Hecho" "🚫 Bloqueado"; do
  curl -s -X POST "https://api.trello.com/1/lists?key=$KEY&token=$TOKEN&idBoard=$BOARD_ID&name=$lista&pos=bottom"
done
```

### Crear una card en Backlog

```bash
curl -s -X POST "https://api.trello.com/1/cards" \
  -G \
  --data-urlencode "key=$TRELLO_API_KEY" \
  --data-urlencode "token=$TRELLO_TOKEN" \
  --data-urlencode "idList=$BACKLOG_LIST_ID" \
  --data-urlencode "name=Fix bug en formulario de registro" \
  --data-urlencode "desc=El campo email no valida correctamente dominios con subdominio"
```

### Mover card a "Hecho"

```bash
curl -s -X PUT "https://api.trello.com/1/cards/$CARD_ID" \
  -G \
  --data-urlencode "key=$TRELLO_API_KEY" \
  --data-urlencode "token=$TRELLO_TOKEN" \
  --data-urlencode "idList=$DONE_LIST_ID"
```

---

## Manejo de errores comunes

| Error | Causa probable | Solución |
|---|---|---|
| `401 Unauthorized` | Token o API Key incorrectos | Re-verificar credenciales en https://trello.com/app-key |
| `404 Not Found` | ID de board, lista o card incorrecto | Verificar el ID con GET /boards/{id} |
| `429 Too Many Requests` | Rate limit alcanzado (300 req/10seg) | Agregar delay entre llamadas o reducir frecuencia |
| `400 Bad Request` | Parámetro faltante o inválido | Revisar que `idList` esté presente en creación de cards |

---

## Rate limits de la API de Trello

- **Por token**: 300 requests cada 10 segundos
- **Por API key**: 100 requests cada 10 segundos por IP
- Para uso normal de equipo (no bulk), estos límites no deberían ser un problema.
