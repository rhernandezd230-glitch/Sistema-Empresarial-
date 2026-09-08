# DOCUMENTO 03 — DESARROLLO TÉCNICO
## Sistema Inteligente de Gestión y Análisis Documental

---

## 1. Stack tecnológico final

| Capa | Tecnología | Versión usada |
|---|---|---|
| Backend | Python | 3.12.10 |
| Framework API | FastAPI | 0.115.0 |
| Servidor ASGI | Uvicorn | 0.30.6 |
| ORM | SQLAlchemy | 2.0.35 |
| Validación | Pydantic | 2.9.2 |
| Autenticación | python-jose (JWT), bcrypt | 3.3.0 / 4.2.0 |
| Base de datos | PostgreSQL | 18 |
| Driver DB | psycopg2-binary | 2.9.9 |
| Extracción PDF | PyMuPDF | 1.24.11 |
| Extracción DOCX | python-docx | 1.1.2 |
| IA | google-generativeai | 0.8.6 |
| Modelo de lenguaje | gemini-flash-lite-latest | — |
| Modelo de embeddings | models/gemini-embedding-001 | — |
| Pruebas | pytest | 8.3.3 |
| Frontend | React + Vite | Vite 8.2.2 |
| Cliente HTTP frontend | Axios | — |
| Enrutamiento frontend | react-router-dom | — |
| Node.js | — | v24.20.0 |

## 2. Estructura de carpetas (real, tal como quedó)

```text
sistema-documental/
├── backend/
│   ├── app/
│   │   ├── ai/            (ai_service, classifier, summarizer, extractor, rag)
│   │   ├── models/        (user, repository, folder, document, category, document_category, ai_result, document_chunk, processing_log)
│   │   ├── schemas/       (user, token, repository, folder, document, rag)
│   │   ├── routers/       (auth, users, repositories, folders, documents, search)
│   │   ├── services/      (document_service, pdf_service, docx_service, txt_service, chunking_service)
│   │   ├── utils/         (security)
│   │   ├── main.py, config.py, database.py, dependencies.py
│   ├── tests/              (conftest, test_auth, test_documents, test_ai_pipeline, test_search)
│   ├── venv/, requirements.txt, .env, pytest.ini
├── frontend/
│   └── src/ (pages, components, context, services, styles)
├── database/ (schema.sql, seed.sql)
├── documents/  (almacenamiento físico de archivos subidos)
└── docker-compose.yml
```

## 3. Explicación de módulos clave

- **`config.py`**: única fuente de configuración, lee `.env` vía `pydantic-settings`. Nunca hay valores reales hardcodeados.
- **`dependencies.py`**: `get_current_user`, valida el JWT en cada ruta protegida usando `HTTPBearer`.
- **`document_service.py`**: orquesta todo el pipeline de un documento — validación, guardado, extracción, IA, chunking/embeddings — y registra cada paso en `processing_logs`.
- **`ai/ai_service.py`**: única capa que habla con la API externa de IA (`ask_ai`, `ask_ai_json`, `embed_text`). Si se cambia de proveedor, solo se toca este archivo.
- **`ai/rag.py`**: calcula similitud coseno en Python (sin `pgvector`) y aplica el umbral que decide si hay "información suficiente".

## 4. Cambios técnicos respecto al plan original, con justificación

| Cambio | Motivo |
|---|---|
| Passlib/Bcrypt → `bcrypt` directo | `passlib` 1.7.4 tiene un bug conocido de incompatibilidad con `bcrypt >= 4.1` |
| API de Anthropic (Claude) → Google Gemini | Anthropic requiere suscripción de pago; Gemini ofrece capa gratuita sin tarjeta |
| Python 3.14 → Python 3.12 | `psycopg2-binary` y `pydantic-core` no tenían wheels precompilados para 3.14 (muy reciente), obligando a compilar desde cero y requiriendo Visual C++ Build Tools |
| `OAuth2PasswordBearer` → `HTTPBearer` | El login del proyecto usa JSON (no form-data de OAuth2 estándar); `HTTPBearer` da un campo simple de "pegar token" coherente con eso |
| `models/text-embedding-004` → `models/gemini-embedding-001` | El primero fue descontinuado por Google durante el desarrollo |
| `gemini-2.0-flash` → `gemini-3.6-flash` → `gemini-2.5-flash` → `gemini-flash-lite-latest` | Descontinuaciones sucesivas de Google y límite de cuota gratuita (20 peticiones/día) en el modelo más nuevo; el modelo "lite" resultó con cuota más generosa |
| `pgvector` → `embedding` como JSONB + similitud en Python | Evitar depender de que el servidor de despliegue tenga la extensión instalada |

## 5. Bitácora de incidencias reales resueltas durante el desarrollo

Registro honesto de problemas encontrados y su solución, tal como ocurrieron (evidencia del proceso real de desarrollo, no simulado):

| # | Incidencia | Solución |
|---|---|---|
| 1 | `python` no reconocido en PowerShell (alias de Microsoft Store) | Se instaló Python desde el instalador oficial marcando "Add python.exe to PATH"; se usó `py` como lanzador mientras se resolvía |
| 2 | `venv\Scripts\activate` bloqueado por política de ejecución de PowerShell | `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` |
| 3 | `psycopg2-binary` y `pydantic-core` fallaban al compilar en Python 3.14 | Se recreó el entorno virtual con Python 3.12 (`py -3.12 -m venv venv`) |
| 4 | `UnicodeDecodeError` al conectar a PostgreSQL | Causa raíz real: el `.env` tenía la contraseña incorrecta; PostgreSQL respondía el error de autenticación en español con tildes, y `psycopg2` fallaba al decodificarlo como UTF-8 en vez de mostrar el error real |
| 5 | Contenido de `.env` pegado accidentalmente en una sola línea (variables concatenadas) | Se reescribió el archivo verificando salto de línea real después de cada variable |
| 6 | Contraseña de PostgreSQL olvidada | Reseteo temporal del método de autenticación a `trust` en `pg_hba.conf`, cambio de contraseña vía `psql`, y reversión inmediata a `scram-sha-256` por seguridad |
| 7 | `ModuleNotFoundError: No module named 'app.ai'` | La carpeta se había creado como `ia` en vez de `ai`; se corrigió el nombre |
| 8 | `uvicorn --reload` recargando constantemente por vigilar la carpeta `venv/` | Se usó `--reload-dir app` para limitar el watcher solo al código propio |
| 9 | `node`/`npm` no reconocidos tras instalar Node.js | Requería reinicio completo del computador para actualizar el PATH del sistema |
| 10 | Modelos de IA descontinuados durante las pruebas (`text-embedding-004`, `gemini-2.0-flash`, `gemini-2.5-flash`) | Se consultó `genai.list_models()` para obtener los modelos vigentes en cada caso |
| 11 | Límite de cuota gratuita (`429 RESOURCE_EXHAUSTED`, 20 peticiones/día) en `gemini-3.6-flash` | Cambio a `gemini-flash-lite-latest`, con cuota gratuita más amplia |
| 12 | Frontend sin menú de navegación (solo se veía el Dashboard) | Se agregó `components/Layout.jsx` con sidebar y se envolvieron las rutas protegidas en él |

## 6. Estado del desarrollo por fase

| Fase | Estado |
|---|---|
| 1. Arquitectura | ✅ Completa |
| 2. Base de datos | ✅ Completa |
| 3. Backend y autenticación | ✅ Completa y probada |
| 4. Carga y procesamiento documental | ✅ Completa y probada |
| 5. Inteligencia Artificial | ✅ Completa y probada |
| 6. RAG y búsqueda inteligente | ✅ Completa y probada |
| 7. Frontend | ✅ Completa y probada |
| 8. Integración frontend + backend | ✅ Completa y probada |
| 9. Dashboard | ✅ Completa (cálculo del lado del cliente, sin endpoint dedicado) |
| 10. Pruebas | ✅ Suite de 18 pruebas automatizadas con `pytest` |
| 11. Despliegue | ⏳ Guía y archivos (`Dockerfile`, `docker-compose.yml`) entregados; despliegue real en servidor de producción no realizado |
| 12. Documentación | ⏳ En curso (este documento forma parte de ella) |
| 13. Presentación | ⏳ Guion y estructura entregados |
