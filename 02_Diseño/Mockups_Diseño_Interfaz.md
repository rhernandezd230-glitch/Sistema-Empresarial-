# Mockups y Diseño de Interfaz
## Sistema Inteligente de Gestión y Análisis Documental — "Legajo"

---

## 1. Concepto de diseño

**Legajo** evoca el archivo institucional: fólderes, ledgers y sellos, pero resuelto con una interfaz digital limpia. Se evita el look genérico de "SaaS card kit" (tarjetas redondeadas idénticas con sombra) a favor de listas tipo ledger con líneas divisorias finas, más cercanas a un libro de registros.

### Paleta de color

| Token | Hex | Uso |
|---|---|---|
| `--ink` | `#1F2A24` | Texto principal, fondo del sidebar |
| `--paper` | `#E9E4D8` | Fondo general de la aplicación |
| `--surface` | `#FDFCF9` | Fondo de tarjetas y formularios |
| `--primary` | `#2F6F62` | Acciones principales, enlaces activos (verde ledger) |
| `--accent` | `#C08A28` | Acentos de IA / marca (ocre sello) |
| `--danger` | `#B2392F` | Errores, acciones destructivas |
| `--success` | `#3D7A4C` | Estado "procesado" |
| `--muted` | `#6E675B` | Texto secundario |

### Tipografía

- **Source Serif 4** — títulos y encabezados (carácter documental/editorial).
- **IBM Plex Sans** — cuerpo de texto, formularios, navegación.
- **IBM Plex Mono** — valores de datos extraídos (montos, códigos, fechas) cuando se requiere alineación tabular.

### Layout

Sidebar fijo de 15.5rem en tinta oscura, contenido principal sobre fondo papel. Listas en formato ledger (borde superior + fila con borde inferior) en lugar de tarjetas con sombra repetidas.

```
┌────────────┬──────────────────────────────────────────┐
│  L Legajo  │  Panel                                    │
│            │  ────────────────────────────────────────│
│  Panel     │  [stat] [stat] [stat] [stat]              │
│  Repositor.│  ────────────────────────────────────────│
│  Documentos│  Documentos por categoría   Cargas/día    │
│  Buscar    │  [gráfico barras]           [gráfico línea]│
│  Chat      │                                            │
│            │                                            │
│  [avatar]  │                                            │
└────────────┴──────────────────────────────────────────┘
```

---

## 2. Login (`/login`)

Pantalla dividida: panel de formulario a la izquierda, panel editorial (cita + color ink) a la derecha.

```
┌───────────────────────────┬───────────────────────────┐
│  L                        │                            │
│  Ingresa a Legajo         │   "Un repositorio no es    │
│  Gestiona repositorios,   │    un cajón donde se       │
│  procesa documentos...    │    guardan documentos..."  │
│                           │                            │
│  Correo electrónico       │        (fondo ink,         │
│  [______________]         │      tipografía serif      │
│  Contraseña               │        grande)             │
│  [______________]         │                            │
│  [ Iniciar sesión ]       │                            │
│                           │                            │
│  ¿No tienes cuenta?       │                            │
│  Crear una cuenta         │                            │
└───────────────────────────┴───────────────────────────┘
```
Incluye credencial de demostración visible (`demo@uts.edu.co` / `demo1234`) para facilitar la sustentación.

## 3. Registro (`/register`)
Misma estructura visual que Login; formulario con nombre, correo y contraseña (mínimo 8 caracteres, validado en cliente).

## 4. Panel / Dashboard (`/`)

```
┌────────────────────────────────────────────────────────┐
│  Panel                                                  │
│  Resumen de tu actividad documental                     │
│                                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │
│  │Repositor.│ │Documentos│ │Procesados│ │Pendientes│    │
│  │    3     │ │    7     │ │    4     │ │    2     │    │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │
│                                                          │
│  Documentos por categoría        Cargas por día         │
│  ┌────────────────────────┐   ┌────────────────────┐   │
│  │  ▇▇▇▇  Factura          │   │   ╭─╮    ╭──╮       │   │
│  │  ▇▇    Contrato         │   │  ╭╯ ╰╮  ╭╯  ╰╮      │   │
│  │  ▇     Informe          │   │ ╭╯   ╰──╯    ╰╮     │   │
│  └────────────────────────┘   └────────────────────┘   │
└────────────────────────────────────────────────────────┘
```
Los cuatro indicadores clave (repositorios, documentos, procesados, pendientes) usan tipografía serif grande para el número, sans-serif pequeña para la etiqueta. Los gráficos (Recharts) usan la paleta de marca, no colores por defecto.

## 5. Repositorios (`/repositorios`)

Lista ledger de repositorios (nombre, descripción, fecha de creación, cantidad de documentos) con botón "Nuevo repositorio" que abre un formulario inline. Cada fila tiene acción de eliminar (ícono de papelera, color `--danger` solo en hover).

```
Repositorios                              [+ Nuevo repositorio]
──────────────────────────────────────────────────────────────
Auditoría Financiera 2025          12 documentos      🗑
Facturas y soportes contables...
──────────────────────────────────────────────────────────────
Contratos Proveedores               5 documentos      🗑
Contratos vigentes con proveedores...
──────────────────────────────────────────────────────────────
```

## 6. Documentos (`/documentos`)

Selector de repositorio/carpeta en la parte superior, zona de carga (drag & drop o selección de archivo) y tabla ledger con columnas: nombre, tamaño, estado (`StatusBadge`), fecha de carga, acciones (ver detalle, descargar, eliminar).

```
Documentos › Auditoría Financiera 2025 › Enero - Marzo
                                          [ Subir documento ]
──────────────────────────────────────────────────────────────
factura_energia_marzo.pdf   214 KB   ● Procesado   20/08  👁 ⬇ 🗑
factura_internet_febrero.pdf 189 KB  ● Procesado   18/08  👁 ⬇ 🗑
factura_arriendo_abril.pdf  156 KB   ● Procesando  05/09  👁 ⬇ 🗑
──────────────────────────────────────────────────────────────
```
El `StatusBadge` usa cuatro variantes de color (pendiente=ocre, procesando=verde marca claro, procesado=verde éxito, error=rojo), consistentes con el estado real de `documents.status`.

## 7. Detalle de documento (`/documentos/{id}`)

Vista de dos columnas: metadatos + vista previa a la izquierda, pestañas de resultados de IA a la derecha (Clasificación, Resumen, Datos extraídos). El estado del pipeline se representa como una secuencia de pasos numerada (extracción → clasificación → resumen → extracción de datos), ya que sí corresponde a un proceso secuencial real.

```
factura_energia_marzo.pdf                    ● Procesado
────────────────────────────────────────────────────────
① Extracción  ② Clasificación  ③ Resumen  ④ Datos extraídos

Clasificación          Resumen
Factura (96%)          "Factura de energía eléctrica
                        correspondiente a marzo de 2026..."

Datos extraídos
proveedor        Electrificadora del Oriente S.A.
numero_factura   FE-88213
valor_total      $184.200
fecha_vencimiento 2026-04-10

[ Preguntar sobre este documento → ]
```

## 8. Buscar (`/buscar`)

Un único campo de búsqueda con conmutador entre "Palabra clave" y "Pregunta (IA)". En modo pregunta, el resultado se presenta como respuesta con fuentes citadas (nombre de archivo + fragmento), igual que en Chat.

```
Buscar                    [ Palabra clave | Pregunta (IA) ]
[ ¿Cuánto se pagó por el servicio de internet en febrero? ]

Respuesta:
"Según los fragmentos recuperados: ConectaNet S.A.S. factura
el servicio de internet dedicado de 200MB..."

Fuentes:
📄 factura_internet_febrero.pdf — "...valor de $312.500..."
```

## 9. Chat / Preguntar a un documento (`/chat`)

Selector de documento en la parte superior + historial de conversación estilo mensajería, con las respuestas del sistema mostrando siempre sus fuentes debajo del mensaje (nunca como texto suelto sin atribución, para cumplir el requisito de RAG con evidencia).

```
Documento: informe_autoevaluacion_software.docx  [cambiar]
────────────────────────────────────────────────────────
                                   ¿Cuál fue la calificación
                                   global del programa?  🧑
🤖  La calificación global fue de 4.3 sobre 5.0.
    Fuente: informe_autoevaluacion_software.docx

[ Escribe tu pregunta...                        ] [ Enviar ]
```

---

## 10. Componentes reutilizables

| Componente | Uso |
|---|---|
| `Sidebar` | Navegación principal + sesión del usuario |
| `AppLayout` | Envuelve las rutas protegidas con el sidebar |
| `StatusBadge` | Estado visual de un documento (4 variantes) |
| `ProtectedRoute` | Redirige a `/login` si no hay sesión activa |
| Iconos inline (`Icons.jsx`) | Set propio de SVG, sin dependencia externa de íconos |

## 11. Accesibilidad y estados vacíos

- Foco de teclado visible (`outline` verde marca) en todos los elementos interactivos.
- Estados vacíos (`.empty-state`) con borde punteado e invitación a la acción ("Sube tu primer documento"), nunca solo un texto plano de "no hay datos".
- Contraste verificado entre `--ink`/`--paper` y `--surface`/`--ink` para cumplir legibilidad AA.
