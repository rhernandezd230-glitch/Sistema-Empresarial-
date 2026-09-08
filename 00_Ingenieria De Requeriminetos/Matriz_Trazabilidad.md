# DOCUMENTO 08 — MATRIZ DE TRAZABILIDAD
## Sistema Inteligente de Gestión y Análisis Documental

---

| Requisito | Funcionalidad | Endpoint / Componente | Prueba asociada | Estado |
|---|---|---|---|---|
| RF1 — Registro de usuarios | Registro con hash de contraseña | `POST /auth/register` | `test_registro_usuario`, `test_registro_correo_duplicado_falla` | ✅ Completado |
| RF2 — Login con JWT | Autenticación y emisión de token | `POST /auth/login` | `test_login_correcto`, `test_login_password_incorrecta` | ✅ Completado |
| RF3 — Protección de rutas | Validación de JWT en cada petición | `dependencies.get_current_user` (todas las rutas excepto `/auth/*`) | `test_ruta_protegida_sin_token`, `test_ruta_protegida_con_token_invalido`, `test_ruta_protegida_con_token_valido` | ✅ Completado |
| RF4 — CRUD de repositorios | Crear, listar, editar, eliminar repositorios | `/repositories` (POST/GET/PUT/DELETE) | Prueba manual (Fase 4), `test_listados_usados_por_el_dashboard` | ✅ Completado |
| RF5 — Gestión de carpetas | Crear, listar, eliminar carpetas | `/folders` (POST/GET/DELETE) | Prueba manual (Fase 4) | ✅ Completado |
| RF6 — Carga de documentos con validación | Extensión, tamaño, contenido vacío, firma real | `POST /documents/upload`, `document_service.validate_file` | `test_subir_txt_valido`, `test_subir_pdf_valido`, `test_subir_docx_valido`, `test_subir_archivo_con_extension_no_permitida`, `test_subir_archivo_vacio_es_rechazado`, `test_archivo_con_extension_falsificada_es_rechazado` | ✅ Completado |
| RF7 — Listar/consultar/descargar/eliminar documentos | CRUD de documentos | `/documents`, `/documents/{id}`, `/documents/{id}/download` | `test_listar_documentos_de_una_carpeta`, `test_descargar_documento`, `test_eliminar_documento` | ✅ Completado |
| RF8 — Extracción real de texto | PDF/DOCX/TXT | `pdf_service`, `docx_service`, `txt_service` | Prueba manual (Fase 4), evidencia en `extracted_text` real | ✅ Completado |
| RF9 — Clasificación automática | Facturas/Contratos/Informes | `ai/classifier.py` | Prueba manual (Fase 5, categoría "Facturas" correcta), `test_documento_queda_clasificado_con_resumen` | ✅ Completado |
| RF10 — Generación de resumen | Resumen en lenguaje natural | `ai/summarizer.py` | Prueba manual (Fase 5), `test_documento_queda_clasificado_con_resumen` | ✅ Completado |
| RF11 — Extracción de información estructurada | Campos según categoría | `ai/extractor.py` | Prueba manual (Fase 5, campos de factura correctos), `test_extraccion_de_informacion_estructurada` | ✅ Completado |
| RF12 — Preguntas sobre un documento | RAG por documento | `POST /documents/{id}/ask` | Prueba manual (Fase 6, respuesta correcta del total), `test_pregunta_con_informacion_relevante` | ✅ Completado |
| RF13 — Preguntas sobre todos los documentos | RAG global | `POST /search/ask` | Prueba manual (Fase 8), `test_busqueda_global_search_ask` | ✅ Completado |
| RF14 — Mensaje de "sin información suficiente" | No inventar respuestas | `ai/rag.py` (`NO_INFO_MESSAGE`) | Prueba manual (Fase 6, pregunta capital de Francia), `test_pregunta_sin_informacion_suficiente` | ✅ Completado |
| RF15 — Registro de errores | `processing_logs` | `document_service.log_event` | Consultas SQL directas durante depuración (Fases 4-6) | ✅ Completado |
| RF16 — Dashboard con estadísticas | Tarjetas y listados | `pages/Dashboard.jsx` (cálculo cliente sobre `/repositories` y `/documents`) | Prueba manual (Fase 9), `test_listados_usados_por_el_dashboard` | ✅ Completado |
| RF17 — Interfaz web completa | Frontend React conectado a todos los endpoints | `frontend/src/pages/*` | Pruebas manuales end-to-end (Fases 7-9) | ✅ Completado |
| RNF1 — Contraseñas con hash seguro | `bcrypt` | `utils/security.py` | Verificado: `password_hash` nunca se expone en respuestas | ✅ Completado |
| RNF2 — API keys solo en `.env` | Configuración centralizada | `config.py` | Revisión de código; ninguna clave literal en el repositorio | ✅ Completado |
| RNF3 — Validación real de tipo de archivo | "Magic bytes" | `document_service.validate_file` | `test_archivo_con_extension_falsificada_es_rechazado` | ✅ Completado |
| RNF4 — RAG no inventa información | Restricción en el prompt del sistema | `ai/rag.py` | `test_pregunta_sin_informacion_suficiente` | ✅ Completado |
| RNF5 — Diseño modular | Separación models/schemas/services/routers/ai | Estructura completa del backend | Revisión de código (Documento 03) | ✅ Completado |
| RNF6 — Registro de estado de procesamiento | Campo `status` en documentos | `models/document.py` (enum `DocumentStatus`) | Observado en todas las pruebas de subida | ✅ Completado |
| RNF7 — Desplegable en producción | Dockerfile, docker-compose | `backend/Dockerfile`, `docker-compose.yml` | No ejecutado; entregado como guía (Documento 05) | ⏳ Pendiente de ejecución real |
