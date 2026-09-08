# DOCUMENTO 01 — ANÁLISIS
## Sistema Inteligente de Gestión y Análisis Documental
**UTS — Tecnología en Desarrollo de Software — VI semestre**
**Docente:** Wilson Castaño Galviz

---

## 1. Introducción

Las empresas acumulan grandes volúmenes de documentos (facturas, contratos, informes) en carpetas sin organización ni análisis automático, lo que dificulta encontrar información, clasificar documentos y responder preguntas sobre su contenido de forma rápida. Este proyecto plantea un sistema que permite convertir una carpeta de documentos en un repositorio inteligente, capaz de organizar, procesar, clasificar, resumir, extraer información y responder preguntas en lenguaje natural sobre los documentos cargados, usando Inteligencia Artificial.

## 2. Objetivo general

Desarrollar un sistema web que permita a una empresa cargar, organizar y analizar documentos (PDF, DOCX, TXT) de forma automática mediante Inteligencia Artificial, incluyendo clasificación, resumen, extracción de información estructurada y respuesta a preguntas en lenguaje natural sobre su contenido.

## 3. Objetivos específicos

- Implementar autenticación segura de usuarios mediante JWT.
- Permitir la organización de documentos en repositorios y carpetas.
- Implementar la carga, validación, almacenamiento y extracción de texto de documentos PDF, DOCX y TXT.
- Integrar una API de Inteligencia Artificial para clasificar documentos, generar resúmenes y extraer información estructurada según el tipo de documento.
- Implementar un sistema de preguntas y respuestas (RAG) basado únicamente en el contenido real de los documentos.
- Construir un frontend web que permita interactuar con todas las funcionalidades anteriores (pendiente — Fases 7 a 9).
- Documentar y desplegar el sistema (pendiente — Fases 11 y 12).

## 4. Alcance del proyecto

### 4.1 Incluido y ya implementado (Fases 1 a 6)
- Arquitectura backend con FastAPI + PostgreSQL.
- Autenticación de usuarios (registro, login, JWT, rutas protegidas).
- Gestión de repositorios y carpetas.
- Carga de documentos con validación de extensión, tamaño, contenido vacío y firma real del archivo.
- Extracción de texto real de PDF (PyMuPDF), DOCX (python-docx) y TXT.
- Clasificación automática en Facturas / Contratos / Informes mediante IA.
- Generación de resúmenes mediante IA.
- Extracción de información estructurada específica por tipo de documento.
- Sistema RAG: fragmentación de texto, generación de embeddings, búsqueda por similitud y respuesta a preguntas basada únicamente en el contenido recuperado, con mensaje explícito cuando no hay evidencia suficiente.
- Registro de errores y estados de procesamiento (`processing_logs`, `status`).

### 4.2 Pendiente (Fases 7 a 13)
- Frontend en React (login, dashboard, gestión de repositorios/carpetas/documentos, buscador, chat).
- Integración frontend-backend.
- Dashboard con estadísticas.
- Pruebas automatizadas (actualmente las pruebas se han hecho de forma manual vía Swagger UI, documentadas en el Documento 04).
- Despliegue en un entorno de producción (actualmente el sistema corre en un entorno de desarrollo local).
- Documentación final y sustentación.

## 5. Justificación

Un sistema de este tipo reduce el tiempo que el personal administrativo invierte en buscar, clasificar y resumir documentos manualmente, y permite consultar el contenido de los documentos mediante preguntas en lenguaje natural en vez de leerlos completos. Además, sirve como caso de estudio integral de una aplicación empresarial que combina backend, base de datos relacional, procesamiento documental e Inteligencia Artificial.

## 6. Actores del sistema

| Actor | Descripción |
|---|---|
| Usuario | Persona que se registra, inicia sesión y gestiona sus propios repositorios, carpetas y documentos. |
| Sistema de IA (externo) | Servicio de modelo de lenguaje (Google Gemini) consumido por el backend para clasificar, resumir, extraer información y responder preguntas. No es un actor humano, pero interviene en el flujo. |

*Nota: el diseño actual no distingue roles administrador/usuario en la lógica de negocio (todo usuario gestiona solo sus propios datos); el campo `role` existe en el modelo de usuario para una futura diferenciación de permisos si el proyecto lo requiere.*

## 7. Requisitos funcionales

| # | Requisito | Estado |
|---|---|---|
| RF1 | El sistema debe permitir el registro de usuarios | ✅ Implementado |
| RF2 | El sistema debe permitir el inicio de sesión con JWT | ✅ Implementado |
| RF3 | El sistema debe proteger rutas que requieren autenticación | ✅ Implementado |
| RF4 | El sistema debe permitir crear, listar, editar y eliminar repositorios | ✅ Implementado |
| RF5 | El sistema debe permitir crear, listar y eliminar carpetas dentro de un repositorio | ✅ Implementado |
| RF6 | El sistema debe permitir subir documentos PDF, DOCX y TXT con validación | ✅ Implementado |
| RF7 | El sistema debe permitir listar, consultar, descargar y eliminar documentos | ✅ Implementado |
| RF8 | El sistema debe extraer el texto real de cada documento subido | ✅ Implementado |
| RF9 | El sistema debe clasificar automáticamente el documento (Facturas/Contratos/Informes) | ✅ Implementado |
| RF10 | El sistema debe generar un resumen del documento | ✅ Implementado |
| RF11 | El sistema debe extraer información estructurada según el tipo de documento | ✅ Implementado |
| RF12 | El sistema debe permitir preguntar sobre un documento específico | ✅ Implementado |
| RF13 | El sistema debe permitir preguntar sobre todos los documentos del usuario | ✅ Implementado |
| RF14 | El sistema debe responder "no encontró información suficiente" cuando corresponda | ✅ Implementado |
| RF15 | El sistema debe registrar errores de procesamiento | ✅ Implementado |
| RF16 | El sistema debe mostrar un dashboard con estadísticas | ⏳ Pendiente (Fase 9) |
| RF17 | El sistema debe tener una interfaz web para todas las funciones anteriores | ⏳ Pendiente (Fases 7-8) |

## 8. Requisitos no funcionales

| # | Requisito | Estado |
|---|---|---|
| RNF1 | Las contraseñas deben almacenarse con hash seguro (bcrypt) | ✅ Implementado |
| RNF2 | Las claves de API nunca deben estar escritas en el código, solo en `.env` | ✅ Implementado |
| RNF3 | El sistema debe validar el tipo real de archivo, no solo su extensión | ✅ Implementado |
| RNF4 | El sistema no debe inventar información al responder preguntas (RAG) | ✅ Implementado |
| RNF5 | El sistema debe ser modular (separación de rutas, servicios, modelos, IA) | ✅ Implementado |
| RNF6 | El sistema debe registrar el estado de procesamiento de cada documento | ✅ Implementado |
| RNF7 | El sistema debe poder desplegarse en un servidor de producción | ⏳ Pendiente (Fase 11) |

## 9. Restricciones y supuestos

- El proyecto usa la API de Google Gemini como proveedor de IA (decisión tomada en la Fase 5 por disponibilidad de capa gratuita), en vez de un proveedor específico no indicado en el enunciado original.
- La búsqueda semántica (RAG) se calcula en Python sin la extensión `pgvector`, decisión tomada para no depender de que el servidor de despliegue la tenga instalada.
- El entorno de desarrollo usado es Windows con PostgreSQL 18 y Python 3.12.
- El sistema, a la fecha de este documento, corre únicamente en entorno de desarrollo local; no ha sido desplegado en producción.

## 10. Glosario

| Término | Definición |
|---|---|
| JWT | JSON Web Token, mecanismo de autenticación sin estado usado por el backend. |
| RAG | Retrieval-Augmented Generation: técnica que recupera fragmentos relevantes de un documento antes de generar una respuesta con IA. |
| Embedding | Representación numérica (vector) del significado de un texto, usada para calcular similitud semántica. |
| Chunk | Fragmento de texto en el que se divide un documento para su procesamiento por el sistema RAG. |
