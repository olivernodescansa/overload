# Cómo funciona Mindscape

Documentación de la arquitectura del carrusel de espejos en `mindscape.html`
(capítulo IV — Overload: Venom and Glass). Pensado para que en el futuro,
vos u otra persona, pueda agregar capítulos nuevos sin tener que volver a
tocar el HTML principal.

---

## 1. Idea general

`mindscape.html` es un **shell vacío**: no tiene ninguna card ni ningún
capítulo escrito adentro. Al cargar la página, un script:

1. Hace `fetch('mindscape/cards.json')` y arma las 13 cards del carrusel
   en JavaScript, una por cada objeto del archivo.
2. Compara la fecha de cada card contra el reloj de quien está mirando la
   página. Si la fecha ya llegó, la card se desbloquea sola (imagen/video
   real, sin la pill violeta, clickeable). Si no llegó, se muestra
   bloqueada.
3. Cuando alguien clickea una card desbloqueada, hace un segundo
   `fetch()` — esta vez al archivo HTML del capítulo correspondiente — y
   lo inyecta debajo del carrusel, en el panel de lectura.

Por eso, para agregar un capítulo nuevo **no hace falta tocar
`mindscape.html` para nada**. Solo hay que:

- Escribir el archivo del capítulo (un `.html` chiquito, ver sección 3).
- Sumar su fila correspondiente en `cards.json` (ver sección 2).
- Si tiene imagen o video propio, ponerlo en `mindscape/media/`.

---

## 2. Estructura de carpetas

```
overload-web/                         ← raíz del sitio
├── index.html
├── architecture.html
├── medical-record.html
├── academic-files.html
├── mindscape.html                    ← el shell del carrusel
├── ...(las fuentes y videos de fondo de siempre)...
│
└── mindscape/                        ← TIENE que estar al lado de mindscape.html
    ├── cards.json                    ← la lista de los 13 espejos
    ├── COMO-FUNCIONA.md              ← este archivo
    ├── chapters/
    │   ├── 01-unsealed.html
    │   ├── 02-silent-company.html
    │   └── ...(uno por capítulo escrito)
    └── media/
        ├── 01-unsealed.mp4
        ├── default-glass.png         ← imagen genérica para las cards sin media propia
        └── ...(la imagen/video de cada capítulo, si tiene uno propio)
```

**Todas las rutas dentro de `cards.json` son relativas a la raíz del
sitio** (donde vive `mindscape.html`), no relativas a la carpeta
`mindscape/`. Por eso se escriben como `"mindscape/media/..."` y
`"mindscape/chapters/..."`, no `"media/..."` a secas.

---

## 3. `cards.json` — cómo se arma cada fila

Cada objeto del array es una card del carrusel:

```json
{
  "id": "02",
  "titulo": "Silent Company",
  "eyebrow": "Capa 02 — Livie a su lado, en silencio",
  "fecha": "2026-09-08",
  "fechaDisplay": "08/09/2026",
  "media": { "type": "image", "src": "mindscape/media/default-glass.png" },
  "chapter": "mindscape/chapters/02-silent-company.html"
}
```

| Campo         | Qué es |
|---------------|--------|
| `id`          | Identificador de dos dígitos (`"01"`, `"02"`...). Se usa para el `id` del botón y para cachear el capítulo ya abierto. |
| `titulo`      | El nombre que se ve en la card (`Unsealed`, `Silent Company`, etc.). |
| `eyebrow`     | La línea chica que aparece arriba del título cuando se abre el capítulo en el reader (`"Capa 02 — ..."`). Es la misma que tiene que ir adentro del archivo del capítulo (ver sección 4). |
| `fecha`       | Fecha ISO (`"AAAA-MM-DD"`) que se compara contra el reloj del visitante. **`null`** si la card nunca se desbloquea por fecha (caso especial, ver más abajo). |
| `fechaDisplay`| La fecha tal como se muestra en la card (`"08/09/2026"`). Es solo texto — no se usa para calcular nada, así que tiene que coincidir a mano con `fecha`. |
| `media`       | `{ "type": "video" \| "image", "src": "ruta/al/archivo" }`. Si el capítulo no tiene media propia, dejalo apuntando a `mindscape/media/default-glass.png` con `"type": "image"`. |
| `chapter`     | Ruta al archivo HTML del capítulo (sección 4). **`null`** si todavía no está escrito, o si la card no tiene capítulo (como "El Veneno"). |

### Caso especial: card sin fecha ("El Veneno")

```json
{
  "id": "10",
  "titulo": "Venom",
  "eyebrow": "Capa 10 — El Veneno",
  "fecha": null,
  "fechaDisplay": "—",
  "media": { "type": "image", "src": "mindscape/media/default-glass.png" },
  "chapter": null
}
```

Con `"fecha": null`, la card queda bloqueada **para siempre** (no es un
capítulo con fecha pendiente, es algo que temáticamente "no es un
espejo"). En vez de la pill violeta *"This mirror opens soon"* muestra
*"Not a mirror"*. El modo preview (sección 6) no la desbloquea a
propósito.

### Card con fecha pero sin capítulo escrito todavía

Es perfectamente válido tener `"fecha"` puesta y `"chapter": null` (o
apuntando a un archivo que todavía no existe). Cuando llegue la fecha, la
card se va a desbloquear igual (imagen real, clickeable), pero al
clickearla el sitio va a mostrar:

> *"Este espejo ya está abierto, pero el capítulo todavía se está
> terminando de escribir."*

en vez de romperse. Así podés cargar el calendario completo de fechas
desde ahora y sumar los textos reales más adelante, sin apuro.

---

## 4. Cómo armar el archivo de un capítulo

Cada capítulo es un **fragmento de HTML** (no una página completa — sin
`<html>`, sin `<head>`, sin video de fondo propio). Va en
`mindscape/chapters/NN-nombre.html`.

### Estructura mínima

```html
<span class="mirror-reader-eyebrow">Capa 02 — Livie a su lado, en silencio</span>
<h3 class="mirror-reader-title">Silent Company</h3>

<p class="story-narration">Acá va un párrafo de narración normal.</p>

<div class="story-dialogue-block">
  <span class="story-speaker story-speaker--oliver">Oliver</span>
  <p class="story-dialogue story-dialogue--oliver">—Una línea que dice Oliver.</p>
</div>
```

### Clases disponibles

| Clase | Para qué sirve |
|---|---|
| `story-narration` | Un párrafo normal de narración (en un `<p>`). |
| `story-dialogue-block` → `story-speaker` + `story-dialogue` | Una línea hablada. El `story-speaker` es la etiqueta con el nombre del personaje — **solo hace falta ponerla cuando cambia el turno** (si el mismo personaje habla en dos párrafos seguidos, el segundo bloque va sin `story-speaker`, dejando el `<div>` vacío de ese span). |
| `story-speaker--oliver` / `story-dialogue--oliver` | Color celeste. Reservado para Oliver, que es la voz constante en todos los capítulos. |
| `story-speaker--other` / `story-dialogue--other` | Color violeta genérico. **Usalo para cualquier otro personaje** (Livie, Tabatha, Zahir, quien sea) — no hace falta crear una clase nueva por personaje, el texto de la etiqueta (`Livie`, `Tabatha`...) ya identifica quién habla. |
| `story-divider` | Separador de corte de escena (el rombito violeta entre líneas). Usalo donde el capítulo tenga un `---` en el documento original. Es un `<div>` vacío, mirá el ejemplo abajo. |
| `story-lede` | Frase de apertura destacada, en itálica y centrada (la usan `index.html` y algunos capítulos para la primera línea). Opcional, no hace falta en todos. |

### Ejemplo de un separador de escena

```html
<div class="story-divider">
  <svg viewBox="0 0 10 10" fill="none" xmlns="http://www.w3.org/2000/svg">
    <path d="M5 0L10 5L5 10L0 5Z" fill="currentColor"/>
  </svg>
</div>
```

### Reglas rápidas al pasar el texto a HTML

- Cada párrafo del documento original es un `<p>` separado (`story-narration`
  o dentro de un `story-dialogue-block`).
- Un párrafo es diálogo **solo si arranca con guión de diálogo (`—`)**.
  Si es reflexión en primera persona sin guión, va como `story-narration`
  aunque esté en el mismo tono íntimo — no toda introspección es una
  línea hablada.
- Cuando un mismo parlante sigue hablando en el párrafo siguiente
  (marcado con `»` en el documento, o simplemente porque no cambió el
  turno), no se repite el `story-speaker`.
- Los caracteres especiales (`'`, `"`, `&`) tienen que ir escapados como
  entidades HTML (`&#x27;`, `&quot;`, `&amp;`) si los escribís a mano.

---

## 5. Paso a paso: agregar un capítulo nuevo

Siempre se parte de un **`.md` con el texto del capítulo tal cual se
escribió** (narración y diálogo en formato normal de manuscrito, con
`---` para los cortes de escena) — el mismo tipo de archivo que se pasó
para "Unsealed" y "Silent Company". Ese `.md` no se sube a ningún lado
del sitio, es solo la fuente de la que se parte.

1. A partir de ese `.md`, armá el archivo
   `mindscape/chapters/NN-nombre-del-capitulo.html` pasando cada párrafo
   a las clases de la sección 4 (narración vs. diálogo, quién habla en
   cada línea, dónde van los separadores de escena).
2. Si tiene imagen o video propio, subilo a `mindscape/media/` y
   actualizá el campo `media` de esa card en `cards.json`. Si no,
   dejalo apuntando a `default-glass.png`.
3. En `cards.json`, buscá la fila con el `id` correspondiente y
   actualizá su `"chapter"` para que apunte al archivo que acabás de
   crear.
4. Listo — no hay que tocar `mindscape.html`.

---

## 6. Cómo probarlo en local

### Con Live Server

`mindscape.html` usa `fetch()` para traer `cards.json` y los capítulos,
así que **no funciona abriendo el archivo con doble clic** (protocolo
`file://` — el navegador bloquea esos `fetch()` por seguridad). Hay que
sevirlo por http:

1. Abrí la carpeta completa del sitio en VS Code (`File → Open Folder`).
2. Con la extensión **Live Server** instalada, clic derecho sobre
   `mindscape.html` → *Open with Live Server* (o el botón *Go Live* de
   la barra de abajo).
3. Se abre algo como `http://127.0.0.1:5500/mindscape.html`.

### Modo preview (para no esperar a que llegue la fecha)

Agregá `?preview` al final de la URL:

```
http://127.0.0.1:5500/mindscape.html?preview
```

Con eso, **todas las cards con fecha quedan desbloqueadas** sin importar
qué día sea — podés clickear cualquiera y ver su capítulo. Va a aparecer
un cartel violeta arriba del carrusel (*"Modo vista previa — todas las
fechas están desbloqueadas"*) para que no te confundas si sacás una
captura pensando que es el sitio real.

`"El Veneno"` (la card sin fecha) sigue bloqueada incluso en preview,
porque no es un capítulo con fecha pendiente — es un elemento temático
que no se abre nunca.

Sacá el `?preview` de la URL para volver a ver el sitio como lo va a ver
cualquier visitante real, respetando las fechas.

---

## 7. Publicarlo (GitHub Pages)

Una vez que subas toda la carpeta (`overload-web/`, con `mindscape/`
adentro tal cual está) a un repo y actives GitHub Pages, todo funciona
igual que en Live Server — ahí también se sirve por https, así que el
`fetch()` no tiene problema. No hace falta ningún build ni configuración
extra: es HTML, CSS y JS estático.
