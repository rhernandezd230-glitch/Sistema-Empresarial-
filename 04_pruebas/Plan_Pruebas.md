# Plan de Pruebas
## Sistema Inteligente de Gestión y Análisis Documental — "Legajo"
**Fase 10 — Pruebas**

---

## 1. Objetivo

Verificar que los módulos implementados (autenticación, repositorios, carpetas, documentos, procesamiento con IA, búsqueda/RAG y dashboard) cumplen los requisitos funcionales definidos en la Fase 1, tanto en la capa de frontend (con datos simulados) como en los contratos de API que consumirá el backend real (Fases 2-6).

## 2. Alcance

**Incluye:**
- Pruebas funcionales de las 8 páginas del frontend (`Login`, `Register`, `Dashboard`, `Repositories`, `Documents`, `DocumentDetail`, `Search`, `Chat`).
- Pruebas de los flujos descritos en los diagramas de secuencia (auth, carga, procesamiento IA, RAG, búsqueda, eliminación).
- Pruebas de validación de datos (formularios, extensión/tamaño de archivo, mensajes de error).
- Pruebas de la capa `services/api.js` en modo simulado (`MOCK_MODE = true`), como sustituto verificable del contrato de API mientras el backend no está desplegado.

**No incluye (fuera de alcance de esta fase):**
- Pruebas de carga/rendimiento sobre el backend real.
- Pruebas de seguridad ofensiva (pentesting) sobre la API en producción.
- Pruebas con el modelo de lenguaje real (se prueba el pipeline RAG con la lógica simulada, no la calidad del LLM en sí).

## 3. Tipos de prueba aplicados

| Tipo | Descripción | Herramienta / método |
|---|---|---|
| Funcional | Verificar que cada flujo produce el resultado esperado | Ejecución manual guiada por casos de prueba |
| Validación de formularios | Campos requeridos, formatos, mensajes de error | Inspección de `Login.jsx`, `Register.jsx`, `Repositories.jsx` |
| Prueba de escritorio (desk check) | Trazado manual de la lógica de `services/api.js` contra las entradas de los casos de prueba | Lectura de código + trazas de ejecución esperada |
| Exploratoria | Navegación libre para detectar comportamientos no contemplados | Sesión manual sobre la app corriendo con `npm run dev` |
| Regresión | Reejecución de casos críticos tras cada corrección registrada en la bitácora | Casos marcados como `Regresión` en la bitácora |

## 4. Ambiente de pruebas

| Elemento | Detalle |
|---|---|
| Frontend | React + Vite, modo `MOCK_MODE = true` (sin backend real) |
| Navegador | Chrome/Edge actualizados, resolución de escritorio y móvil (375px) |
| Datos | Conjunto semilla de `mocks/mockData.js` (3 repositorios, 7 documentos, 4 resultados de IA) |
| Persistencia | `localStorage` del navegador (clave `sd_mock_store_v1`) |
| Comando de arranque | `npm install && npm run dev` desde `frontend/` |

## 5. Roles y responsabilidades

| Rol | Responsable | Actividad |
|---|---|---|
| Diseño de casos | Estudiante(s) del proyecto | Elaborar y mantener `Casos_Prueba.md` |
| Ejecución | Estudiante(s) del proyecto | Ejecutar casos y registrar evidencia |
| Registro de errores | Estudiante(s) del proyecto | Documentar en `Bitacora_Errores_Soluciones.md` |
| Revisión | Docente Wilson Castaño Galviz | Validar cobertura y cierre de la fase |

## 6. Criterios de entrada

- Fase 7 (Frontend) completada y funcional en modo simulado.
- Endpoints de la API especificados (`04_Especificaciones_API.md`).
- Casos de prueba redactados y priorizados.

## 7. Criterios de salida (definición de "hecho")

- 100% de los casos de prioridad **Alta** ejecutados y en estado `Aprobado`.
- Ningún defecto abierto de severidad **Crítica** o **Alta** en la bitácora.
- Evidencia registrada para cada caso ejecutado.

## 8. Priorización

| Prioridad | Criterio |
|---|---|
| Alta | Bloquea el uso del sistema si falla (login, carga de documentos, RAG) |
| Media | Afecta la experiencia pero tiene alternativa (mensajes de validación, búsqueda) |
| Baja | Detalles visuales o de conveniencia |

## 9. Matriz de trazabilidad (módulo → casos)

| Módulo | Casos relacionados (ver `Casos_Prueba.md`) |
|---|---|
| Autenticación | CP-01 a CP-06 |
| Repositorios y carpetas | CP-07 a CP-10 |
| Documentos y carga | CP-11 a CP-17 |
| Procesamiento IA | CP-18 a CP-20 |
| Búsqueda y RAG | CP-21 a CP-25 |
| Dashboard | CP-26 a CP-27 |
| Navegación/sesión | CP-28 a CP-30 |

## 10. Riesgos de la fase de pruebas

| Riesgo | Mitigación |
|---|---|
| No hay backend real disponible aún | Pruebas contra la capa mock, con contrato idéntico al de la API especificada |
| Persistencia en `localStorage` puede quedar en estado inconsistente entre sesiones de prueba | Caso CP-30 (reinicio de datos) y limpieza manual de `localStorage` entre rondas |
| Ambigüedad entre "error de UI" y "error de lógica simulada" | Cada defecto en la bitácora indica explícitamente el módulo y si aplica solo al modo mock |
