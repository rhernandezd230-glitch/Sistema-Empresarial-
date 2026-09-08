# Especificaciones de la API
## Sistema Inteligente de Gestión y Análisis Documental — "Legajo"

**Base URL (desarrollo):** `http://localhost:8000`
**Formato:** JSON (excepto `/documents/upload`, que usa `multipart/form-data`)
**Autenticación:** Bearer JWT en el header `Authorization: Bearer <token>` para todos los endpoints excepto `/auth/register` y `/auth/login`.
**Documentación interactiva:** generada automáticamente por FastAPI en `/docs` (Swagger) y `/redoc`.

---

## Convenciones generales

| Código | Significado |
|---|---|
| 200 | OK |
| 201 | Recurso creado |
| 202 | Aceptado, procesamiento asíncrono en curso |
| 204 | Eliminado, sin contenido |
| 400 | Solicitud inválida (validación de Pydantic) |
| 401 | No autenticado / token inválido o ausente |
| 403 | Autenticado pero sin permiso sobre el recurso |
| 404 | Recurso no encontrado |
| 409 | Conflicto (ej. correo ya registrado) |
| 422 | Error de validación de esquema |
| 500 | Error interno del servidor |

Formato estándar de error:
```json
{
  "detail": "Mensaje descriptivo del error"
}
```

---

## 1. Autenticación

### `POST /auth/register`
Crea una cuenta nueva.

**Body**
```json
{
  "name": "Wilson Castaño Galviz",
  "email": "demo@uts.edu.co",
  "password": "demo1234"
}
```

**Respuesta 201**
```json
{
  "token": "eyJhbGciOi...",
  "user": { "id": "uuid", "name": "Wilson Castaño Galviz", "email": "demo@uts.edu.co", "role": "docente" }
}
```
**Errores:** `409` correo ya registrado · `422` contraseña muy corta o email inválido.

---

### `POST /auth/login`
Autentica un usuario existente.

**Body**
```json
{ "email": "demo@uts.edu.co", "password": "demo1234" }
```

**Respuesta 200:** igual forma que `/auth/register`.
**Errores:** `401` credenciales inválidas.

---

## 2. Repositorios

### `GET /repositories`
Lista los repositorios del usuario autenticado.
**Respuesta 200:** `[{ id, name, description, ownerId, createdAt }]`

### `POST /repositories`
**Body:** `{ "name": "string", "description": "string" }`
**Respuesta 201:** repositorio creado.

### `PUT /repositories/{id}`
**Body:** `{ "name": "string", "description": "string" }` (campos parciales permitidos)
**Respuesta 200:** repositorio actualizado.
**Errores:** `403` si el repositorio no pertenece al usuario · `404` si no existe.

### `DELETE /repositories/{id}`
Elimina el repositorio y, en cascada, sus carpetas, documentos, fragmentos y resultados de IA asociados.
**Respuesta:** `204`.

---

## 3. Carpetas

### `GET /folders?repositoryId=`
**Respuesta 200:** `[{ id, repositoryId, parentId, name }]`

### `POST /folders`
**Body:** `{ "repositoryId": "uuid", "parentId": "uuid|null", "name": "string" }`
**Respuesta 201:** carpeta creada.

### `DELETE /folders/{id}`
**Respuesta:** `204`. Si la carpeta tiene documentos, estos quedan sin `folder_id` (se mueven a la raíz del repositorio) salvo que se indique `?cascade=true`.

---

## 4. Documentos

### `POST /documents/upload`
`multipart/form-data`: `file`, `repositoryId`, `folderId` (opcional).

**Validaciones del backend:**
- Extensión permitida: `.pdf`, `.docx`, `.txt`.
- MIME real verificado (no solo la extensión declarada).
- Tamaño máximo configurable (por defecto 20 MB).
- Extensiones ejecutables bloqueadas explícitamente.

**Respuesta 201**
```json
{
  "id": "uuid",
  "filename": "factura_marzo.pdf",
  "status": "pendiente",
  "sizeKb": 214,
  "uploadedAt": "2026-09-08T10:00:00Z"
}
```
**Errores:** `400` archivo inválido (extensión, MIME o tamaño).

### `GET /documents?repositoryId=&folderId=`
**Respuesta 200:** lista de documentos con sus campos base y `status`.

### `GET /documents/{id}`
**Respuesta 200:** documento completo, incluyendo `aiResult` (si `status='procesado'`) y `categories`.

### `GET /documents/{id}/download`
Devuelve el archivo binario original (`Content-Disposition: attachment`).

### `DELETE /documents/{id}`
Elimina el documento, su archivo físico, sus `document_chunks` y su `ai_results`.
**Respuesta:** `204`.

### `POST /documents/{id}/process`
Dispara (o reintenta) el pipeline de IA de forma asíncrona.
**Respuesta 202:** `{ "status": "procesando" }`
El resultado final se consulta luego vía `GET /documents/{id}` (polling) o WebSocket si se implementa en fases posteriores.

### `POST /documents/{id}/ask`
Pregunta en lenguaje natural sobre un documento específico (RAG acotado a ese documento).

**Body:** `{ "question": "¿Cuál es el valor total de la factura?" }`

**Respuesta 200**
```json
{
  "answer": "Según los fragmentos recuperados: el valor a pagar es de $184.200...",
  "sources": [
    { "documentId": "uuid", "filename": "factura_marzo.pdf", "excerpt": "..." }
  ]
}
```
Si no hay evidencia suficiente, `answer` devuelve el mensaje estándar y `sources` es un arreglo vacío.

---

## 5. Búsqueda

### `GET /search?q=`
Búsqueda por palabra clave sobre nombre de archivo y texto extraído.
**Respuesta 200:** `[{ id, filename, repositoryId, status }]`

### `POST /search/ask`
RAG sin acotar a un documento: busca en todos los `document_chunks` accesibles por el usuario.
**Body:** `{ "question": "string" }`
**Respuesta 200:** igual forma que `/documents/{id}/ask`.

---

## 6. Dashboard

### `GET /dashboard/stats`
**Respuesta 200**
```json
{
  "totalRepositories": 3,
  "totalDocuments": 7,
  "byStatus": { "pendiente": 1, "procesando": 1, "procesado": 4, "error": 1 },
  "byCategory": [{ "name": "Factura", "total": 2 }, { "name": "Contrato", "total": 1 }],
  "uploadsByDay": [{ "date": "2026-08-18", "total": 1 }]
}
```

---

## 7. Correspondencia con el frontend

La capa `frontend/src/services/api.js` implementa una función por cada endpoint anterior (`login`, `register`, `getRepositories`, `createRepository`, `deleteRepository`, `getFolders`, `createFolder`, `getDocuments`, `getDocument`, `uploadDocument`, `processDocument`, `deleteDocument`, `searchKeyword`, `searchAsk`, `askDocument`, `getDashboardStats`). Mientras el backend no está desplegado, estas funciones resuelven contra datos simulados (`MOCK_MODE = true`); al integrarse el backend real basta con cambiar esa bandera para que usen la instancia de Axios ya configurada.
