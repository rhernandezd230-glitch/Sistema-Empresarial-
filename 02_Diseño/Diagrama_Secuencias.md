# Diagramas de Secuencia
## Sistema Inteligente de Gestión y Análisis Documental — "Legajo"

---

## 1. Registro de usuario

```mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Register.jsx
    participant BE as POST /auth/register
    participant DB as PostgreSQL
    U->>FE: Completa nombre, correo, contraseña
    FE->>FE: Valida longitud mínima de contraseña
    FE->>BE: POST /auth/register {name, email, password}
    BE->>DB: Verifica que el correo no exista
    alt correo ya registrado
        DB-->>BE: Conflicto
        BE-->>FE: 409 Conflict
        FE-->>U: "Ya existe una cuenta con este correo"
    else correo disponible
        BE->>BE: Genera hash Bcrypt de la contraseña
        BE->>DB: INSERT INTO users
        DB-->>BE: Usuario creado
        BE-->>FE: 201 {token, user}
        FE->>FE: Guarda token y user en el AuthContext
        FE-->>U: Redirige al Panel
    end
```

## 2. Inicio de sesión

```mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Login.jsx
    participant BE as POST /auth/login
    participant DB as PostgreSQL
    U->>FE: Ingresa email y contraseña
    FE->>BE: POST /auth/login {email, password}
    BE->>DB: SELECT usuario por email
    DB-->>BE: Registro (o vacío)
    BE->>BE: Compara password con password_hash (bcrypt)
    alt credenciales inválidas
        BE-->>FE: 401 Unauthorized
        FE-->>U: "Credenciales inválidas"
    else credenciales correctas
        BE->>BE: Firma JWT (sub=user_id, exp)
        BE-->>FE: 200 {token, user}
        FE->>FE: Guarda token (localStorage) y user (AuthContext)
        FE->>FE: Middleware de rutas protegidas habilita el acceso
        FE-->>U: Redirige a la ruta solicitada originalmente
    end
```

## 3. Carga de documento

```mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Documents.jsx
    participant BE as POST /documents/upload
    participant DB as PostgreSQL
    participant FS as Almacenamiento (documents/)
    U->>FE: Selecciona archivo y confirma carga
    FE->>FE: Valida extensión, tamaño y MIME en cliente
    FE->>BE: POST /documents/upload (multipart: file, repositoryId, folderId)
    BE->>BE: Revalida extensión/MIME real/tamaño máximo
    alt archivo inválido
        BE-->>FE: 400 Bad Request
        BE->>DB: INSERT processing_logs (level=error)
        FE-->>U: Muestra error de validación
    else archivo válido
        BE->>FS: Guarda archivo físico
        BE->>DB: INSERT INTO documents (status='pendiente')
        DB-->>BE: Documento creado
        BE-->>FE: 201 {document}
        FE->>FE: Agrega el documento a la lista con status "Pendiente"
        FE->>BE: POST /documents/{id}/process (disparo asíncrono)
    end
```

## 4. Procesamiento con IA

```mermaid
sequenceDiagram
    participant FE as Documents.jsx / DocumentDetail.jsx
    participant BE as POST /documents/{id}/process
    participant PROC as Servicio de Procesamiento
    participant AI as Módulo IA
    participant LLM as API de Modelo de Lenguaje
    participant DB as PostgreSQL
    FE->>BE: POST /documents/{id}/process
    BE->>DB: UPDATE documents SET status='procesando'
    BE-->>FE: 202 Accepted
    BE->>PROC: Extrae texto (PyMuPDF / python-docx / TXT)
    PROC->>PROC: Limpieza de texto
    PROC->>DB: INSERT document_chunks (con overlap definido)
    PROC->>AI: Clasificar, resumir, extraer datos
    AI->>LLM: Prompt de clasificación/resumen/extracción
    LLM-->>AI: Resultado estructurado
    AI->>DB: INSERT/UPDATE ai_results
    AI->>DB: INSERT document_categories (con confidence)
    DB-->>BE: Confirmación
    BE->>DB: UPDATE documents SET status='procesado', processed_at=now()
    FE->>BE: (poll) GET /documents/{id}
    BE-->>FE: {status:'procesado', aiResult:{...}}
    FE-->>FE: Actualiza StatusBadge y muestra pestaña de resultados IA
    alt error durante extracción o IA
        PROC-->>BE: Excepción
        BE->>DB: UPDATE documents SET status='error'
        BE->>DB: INSERT processing_logs (level=error)
    end
```

## 5. Pregunta sobre un documento específico (RAG)

```mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Chat.jsx
    participant BE as POST /documents/{id}/ask
    participant RAG as Módulo RAG
    participant DB as PostgreSQL
    participant LLM as API de Modelo de Lenguaje
    U->>FE: Escribe una pregunta sobre el documento
    FE->>BE: POST /documents/{id}/ask {question}
    BE->>RAG: Ejecuta pipeline RAG (scope=documento)
    RAG->>RAG: Genera embedding de la pregunta
    RAG->>DB: Busca chunks más similares en document_chunks (filtrado por document_id)
    DB-->>RAG: Fragmentos relevantes ordenados por similitud
    RAG->>RAG: Construye contexto con los fragmentos recuperados
    RAG->>LLM: Envía contexto + pregunta
    LLM-->>RAG: Respuesta generada
    RAG->>RAG: Verifica evidencia suficiente
    alt evidencia insuficiente
        RAG-->>BE: {answer: "No se encontró información suficiente...", sources: []}
    else evidencia suficiente
        RAG-->>BE: {answer, sources:[{documentId, filename, excerpt}]}
    end
    BE-->>FE: 200 {answer, sources}
    FE-->>U: Muestra respuesta con fuentes citadas
```

## 6. Búsqueda global (semántica + palabra clave)

```mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Search.jsx
    participant BE as GET /search o POST /search/ask
    participant DB as PostgreSQL
    U->>FE: Escribe término o pregunta
    alt búsqueda por palabra clave
        FE->>BE: GET /search?q=...
        BE->>DB: Filtra documents por filename/contenido indexado
        DB-->>BE: Lista de documentos coincidentes
        BE-->>FE: 200 [documents]
    else pregunta en lenguaje natural
        FE->>BE: POST /search/ask {question}
        BE->>DB: Ejecuta pipeline RAG sin filtrar por documento (scope=repositorio/global)
        DB-->>BE: Fragmentos relevantes de varios documentos
        BE-->>FE: 200 {answer, sources}
    end
    FE-->>U: Renderiza resultados o respuesta con fuentes
```

## 7. Eliminación de repositorio (con documentos asociados)

```mermaid
sequenceDiagram
    participant U as Usuario
    participant FE as Repositories.jsx
    participant BE as DELETE /repositories/{id}
    participant DB as PostgreSQL
    participant FS as Almacenamiento
    U->>FE: Confirma eliminación del repositorio
    FE->>BE: DELETE /repositories/{id}
    BE->>DB: Verifica pertenencia del repositorio al usuario autenticado
    alt no autorizado
        BE-->>FE: 403 Forbidden
    else autorizado
        BE->>DB: DELETE CASCADE folders, documents, document_chunks, ai_results, processing_logs
        BE->>FS: Elimina archivos físicos asociados
        DB-->>BE: Confirmación
        BE-->>FE: 204 No Content
        FE-->>U: Retira el repositorio de la lista
    end
```
