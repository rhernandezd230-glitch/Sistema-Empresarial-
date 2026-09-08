# DOCUMENTO 05 — IMPLEMENTACIÓN Y DESPLIEGUE
## Sistema Inteligente de Gestión y Análisis Documental

---

## 1. Estado actual

El sistema está **completamente implementado y funcional en un entorno de desarrollo local** (Windows, PostgreSQL 18, Python 3.12, Node.js 24). **No se ha desplegado en un servidor de producción** — esta sección documenta el entorno de desarrollo real usado y deja la guía lista para un despliegue futuro, sin reportar un despliegue que no se realizó.

## 2. Requisitos del entorno de desarrollo (los realmente usados)

| Componente | Versión usada |
|---|---|
| Sistema operativo | Windows |
| Python | 3.12.10 |
| Node.js | v24.20.0 |
| PostgreSQL | 18 |
| Editor | Visual Studio Code |

## 3. Instalación — Backend (pasos reales seguidos)

```bash
cd backend
py -3.12 -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
# completar DATABASE_URL, SECRET_KEY y AI_API_KEY en .env
uvicorn app.main:app --reload --reload-dir app
```

Nota real de instalación: durante el desarrollo fue necesario fijar la versión de Python en 3.12 (no 3.14), porque algunas dependencias (`psycopg2-binary`, `pydantic-core`) no tenían paquetes precompilados para una versión tan reciente de Python.

## 4. Instalación — Base de datos

```bash
# En pgAdmin, crear la base de datos "sistema_documental"
psql -d sistema_documental -f database/schema.sql
psql -d sistema_documental -f database/seed.sql
```

## 5. Instalación — Frontend

```bash
cd frontend
npm install
npm run dev
```

## 6. Variables de entorno (`.env` del backend)

```text
DATABASE_URL=postgresql://usuario:password@localhost:5432/sistema_documental
SECRET_KEY=<clave aleatoria generada con secrets.token_hex(32)>
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
AI_API_KEY=<clave real de Google AI Studio>
AI_MODEL=gemini-flash-lite-latest
```

Ninguna de estas claves está escrita en el código fuente; todas se leen desde `.env`, el cual está excluido de git mediante `.gitignore`.

## 7. Guía de despliegue a producción (no ejecutada, entregada como preparación)

### Backend — Dockerfile
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### docker-compose.yml (raíz del proyecto)
```yaml
version: "3.9"
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: sistema_documental
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
  backend:
    build: ./backend
    env_file: ./backend/.env
    ports:
      - "8000:8000"
    depends_on:
      - db
volumes:
  db_data:
```

### Opciones de hosting recomendadas
| Componente | Opción sugerida |
|---|---|
| Backend | Render o Railway |
| Base de datos | Neon, Supabase, o la base administrada del mismo proveedor de backend |
| Frontend | Vercel o Netlify |

### Checklist antes de un despliegue real
- [ ] Generar un `SECRET_KEY` nuevo, distinto al de desarrollo.
- [ ] Cambiar `allow_origins` en `main.py` a la URL real del frontend desplegado (ya no `http://localhost:5173`).
- [ ] Confirmar que `.env` nunca se sube al repositorio.
- [ ] Ejecutar `schema.sql` contra la base de datos de producción.
- [ ] Actualizar la URL fija del backend en `frontend/src/services/api.js`.
- [ ] Configurar HTTPS (los proveedores sugeridos lo dan por defecto).

## 8. Riesgos identificados para el despliegue

- La capa gratuita de la API de Google Gemini tiene límites de cuota diarios que ya se evidenciaron durante el desarrollo (20 peticiones/día en modelos más nuevos); en producción con más de un usuario, esto podría requerir un plan de pago.
- El almacenamiento de archivos (`documents/`) es local al servidor; en un despliegue con múltiples instancias del backend, esto requeriría migrar a almacenamiento compartido (ej. S3) — no evaluado en este proyecto.
