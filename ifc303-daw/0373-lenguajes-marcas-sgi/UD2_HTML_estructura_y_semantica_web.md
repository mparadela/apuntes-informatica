# UD2 — HTML: estructura y semántica web

**0373 · Lenguajes de Marcas y SGI** · Apuntes de la unidad · RA2 (a-e) · 1º DAW · Curso 2026-27

---

## Introducción

En esta unidad vas a aprender a usar HTML para transmitir y presentar información a través de la web. Analizarás la estructura de los documentos, reconocerás para qué sirve cada etiqueta y usarás las herramientas con las que se trabaja hoy de forma profesional. Ya conoces los lenguajes de marcas en general (UD1). Ahora te centras en el que más se usa en el mundo: HTML.

El trabajo se organiza en nueve sesiones y avanza a la vez que tu propio proyecto, un portfolio personal que irás construyendo página a página:

- **Sesión 1:** Los lenguajes de la web y quién los define.
- **Sesión 2:** Anatomía de un documento HTML y el entorno de trabajo.
- **Sesión 3:** Texto con significado: encabezados, párrafos, listas y semántica en línea.
- **Sesión 4:** Enlaces, rutas e imágenes.
- **Sesión 5:** Estructura semántica de la página.
- **Sesión 6:** Tablas de datos.
- **Sesión 7:** Formularios.
- **Sesión 8:** Versiones de HTML en la práctica: de HTML 4 y XHTML a HTML actual.
- **Sesión 9:** Defensa del portfolio.

Estos apuntes son un material de apoyo y consulta. Úsalos para repasar y consolidar lo trabajado en clase, no como primer contacto con cada tema.

## Índice

1. Los lenguajes de la web y sus estándares *(2.a, 2.d)*
2. Estructura de un documento HTML y herramientas de trabajo *(2.b, 2.e)*
3. Texto con significado *(2.c)*
4. Enlaces, rutas e imágenes *(2.c, 2.e)*
5. Estructura semántica de la página *(2.b, 2.c)*
6. Tablas de datos *(2.c)*
7. Formularios *(2.c, 2.d)*
8. Versiones de HTML en la práctica *(2.d, 2.a)*
- Glosario
- Referencias y recursos

---

## 1. Los lenguajes de la web y sus estándares

### 1.1 Una página web es un documento con varios lenguajes

Cuando un navegador muestra una página, casi nunca está leyendo un solo lenguaje. Lo habitual es que en una misma página convivan tres lenguajes con responsabilidades separadas, y a veces algunos más:

- **HTML** (*HyperText Markup Language*) describe la **estructura y el significado** del contenido: esto es un encabezado, esto es un párrafo, esto es un enlace, esto es un formulario.
- **CSS** (*Cascading Style Sheets*) describe la **presentación**: colores, tipografías, márgenes, disposición en pantalla. Lo trabajarás en la UD3.
- **JavaScript** describe el **comportamiento**: qué pasa cuando se pulsa un botón, cómo se modifica la página sin recargarla. Lo trabajarás en la UD4.

A esta separación se la llama a menudo **separación de responsabilidades** (*separation of concerns*). Es la misma idea de «separar contenido y forma» que viste en la UD1 como ventaja de los lenguajes de marcas, aplicada a la web. El mismo HTML puede tener dos hojas de estilo distintas (pantalla e impresión, por ejemplo) sin tocar una sola línea de contenido.

Además de esos tres, hay otros lenguajes de marcas que pueden aparecer dentro de una página o a su alrededor:

- **SVG** (*Scalable Vector Graphics*): gráficos vectoriales descritos con marcas. Un icono SVG es texto, no una imagen de píxeles, y se puede escribir directamente dentro del HTML.
- **MathML** (*Mathematical Markup Language*): notación matemática estructurada. También se puede escribir dentro del HTML.
- **XHTML**: HTML reformulado con las reglas de XML (ver apartado 1.4).
- **Formatos XML de sindicación** (RSS y Atom): no forman parte de la página, pero la página puede enlazarlos para que otras aplicaciones se suscriban a sus novedades. Los verás en la UD3.

### 1.2 ¿Cuáles de ellos son lenguajes de marcas?

El criterio es el de la UD1: un lenguaje de marcas combina contenido y marcas que lo anotan, dentro de un documento de texto. Con ese criterio, **HTML, XHTML, SVG, MathML, RSS y Atom son lenguajes de marcas**. CSS y JavaScript **no lo son**: CSS es un lenguaje de reglas de estilo (selector + declaraciones) y JavaScript es un lenguaje de programación. Viven junto al HTML, pero no marcan contenido.

El siguiente documento es válido según el validador oficial Nu Html Checker y combina casi todos estos lenguajes en una sola página. Fíjate en que cada uno tiene su propia sintaxis:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Ficha del módulo 0373</title>
  <link rel="alternate" type="application/rss+xml" title="Novedades" href="novedades.xml">
  <style>
    h1 { color: #1e3a8a; }
  </style>
</head>
<body>
  <h1>Lenguajes de Marcas y SGI</h1>
  <p>Horas totales: <strong>67</strong></p>
  <svg width="120" height="20" role="img" aria-label="Progreso: 30 %">
    <rect width="120" height="20" fill="#e5e7eb"/>
    <rect width="36" height="20" fill="#1e3a8a"/>
  </svg>
  <p>Fórmula de la nota:
    <math>
      <mi>N</mi><mo>=</mo>
      <mfrac><mrow><mo>∑</mo><msub><mi>p</mi><mi>i</mi></msub><msub><mi>n</mi><mi>i</mi></msub></mrow><mn>100</mn></mfrac>
    </math>
  </p>
  <button type="button" id="saludo">Saludar</button>
  <script>
    document.getElementById("saludo").addEventListener("click", () => {
      alert("Hola, 1º DAW");
    });
  </script>
</body>
</html>
```

| Fragmento | Lenguaje | ¿Lenguaje de marcas? | Función |
|---|---|---|---|
| `<h1>`, `<p>`, `<strong>`, `<button>` | HTML | Sí | Estructura y significado |
| `h1 { color: #1e3a8a; }` dentro de `<style>` | CSS | No (reglas de estilo) | Presentación |
| `<svg>`, `<rect>` | SVG | Sí | Gráfico vectorial |
| `<math>`, `<mfrac>`, `<mi>` | MathML | Sí | Notación matemática |
| `document.getElementById(...)` dentro de `<script>` | JavaScript | No (programación) | Comportamiento |
| `novedades.xml` enlazado con `<link rel="alternate">` | RSS (XML) | Sí | Sindicación (fuera de la página) |

### 1.3 Quién define los lenguajes de la web

Un estándar solo es útil si todos los navegadores lo implementan igual. Para eso hacen falta organismos que redacten las especificaciones. En la web intervienen cuatro principales:

| Organismo | Qué es | Qué define (relevante para el módulo) |
|---|---|---|
| **W3C** (*World Wide Web Consortium*) | Consorcio fundado en 1994 por Tim Berners-Lee, el creador de la web | CSS, SVG, MathML, XML y XHTML. Accesibilidad (WCAG). Publicó las versiones de HTML hasta HTML 5.2 |
| **WHATWG** (*Web Hypertext Application Technology Working Group*) | Grupo creado en 2004 por Apple, Mozilla y Opera | El estándar de HTML actual (*HTML Living Standard*) y el DOM |
| **IETF** (*Internet Engineering Task Force*) | Organismo de estándares de Internet, que publica documentos llamados RFC | El protocolo HTTP y la sintaxis de los URI. Publicó HTML 2.0 como RFC 1866 (1995) |
| **Ecma International** | Organismo de estandarización (comité TC39) | ECMAScript, el estándar en el que se basa JavaScript |

La relación entre W3C y WHATWG explica por qué hoy no existe «HTML6». A principios de los 2000, el W3C apostaba por un XHTML 2.0 estricto e incompatible con las páginas existentes. Los fabricantes de navegadores no estaban de acuerdo, crearon el WHATWG y empezaron a desarrollar por su cuenta lo que acabaría siendo HTML5. Durante años hubo dos especificaciones de HTML en paralelo. En **2019, W3C y WHATWG firmaron un acuerdo** por el que el WHATWG mantiene una **única versión de HTML y del DOM**, y el W3C colabora en ella en lugar de publicar la suya.

El resultado es el ***HTML Living Standard*** («estándar vivo»): una especificación que no tiene números de versión y se actualiza continuamente. Cuando se incorpora una novedad, entra en el estándar y los navegadores la van implementando. Por eso, cuando hoy se dice «HTML5», casi siempre se quiere decir simplemente «HTML moderno».

### 1.4 Evolución de HTML

| Año | Versión | Quién | Qué aportó o qué cambió |
|---|---|---|---|
| 1991 | Primeras etiquetas de HTML | Tim Berners-Lee (CERN) | Unas pocas etiquetas derivadas de SGML (la familia que viste en la UD1): encabezados, párrafos, enlaces |
| 1995 | HTML 2.0 | IETF (RFC 1866) | Primera versión estandarizada. Incluye formularios |
| 1997 | HTML 3.2 | W3C | Tablas y etiquetas presentacionales como `font` y `center`, fruto de la «guerra de navegadores» |
| 1997-1999 | HTML 4.0 / 4.01 | W3C | Separa presentación y contenido: CSS pasa a ser la vía recomendada. Tres variantes (*Strict*, *Transitional* y *Frameset*) |
| 2000 | XHTML 1.0 | W3C | HTML 4.01 reescrito con las reglas de XML: bien formado obligatorio |
| 2000s | XHTML 2.0 (abandonado) | W3C | Incompatible con la web existente. Nunca llegó a publicarse como estándar |
| 2014 | HTML5 | W3C (a partir del trabajo del WHATWG) | Etiquetas semánticas (`header`, `nav`, `main`, `article`…), audio y vídeo nativos, nuevos tipos de formulario y DOCTYPE simplificado |
| 2016-2017 | HTML 5.1 / 5.2 | W3C | Revisiones del W3C, que dejan de mantenerse tras el acuerdo de 2019 |
| Desde 2019 | *HTML Living Standard* | WHATWG (con W3C) | Versión única, sin números, en evolución continua |

La forma más rápida de reconocer a qué versión pertenece un documento es su **declaración DOCTYPE**, la primera línea del archivo. Compara estas tres:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
```

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
```

```html
<!DOCTYPE html>
```

Las dos primeras hacen referencia a una **DTD** (*Document Type Definition*), un documento que define qué etiquetas y atributos son válidos. Las DTD las trabajarás a fondo en la UD5, con XML. HTML actual ya no se basa en una DTD, porque su especificación define directamente cómo debe procesarse cualquier documento, incluso uno con errores. Por eso su DOCTYPE es tan corto: su única función es que el navegador procese la página en **modo estándar** y no en el modo de compatibilidad con páginas antiguas (*quirks mode*).

### 1.5 HTML y XHTML: dos sintaxis para el mismo lenguaje

En la UD1 viste que un parser XML se detiene en cuanto encuentra un error de buen formado. HTML funciona al revés: el navegador **tolera los errores** y los corrige como puede, porque la web está llena de páginas imperfectas que tienen que seguir funcionando. Una etiqueta `<p>` sin cerrar, atributos sin comillas o mayúsculas y minúsculas mezcladas: HTML lo acepta todo.

XHTML es HTML escrito con las reglas de XML: todas las etiquetas cerradas (también las vacías, como `<br />`), minúsculas, atributos siempre entre comillas y anidamiento correcto. El estándar actual de HTML sigue contemplando las dos sintaxis: la **sintaxis HTML** (la habitual) y la **sintaxis XML** (lo que se llamaba XHTML). La diferencia real depende de cómo el servidor envía el archivo:

- Si lo envía como `text/html` (lo normal), el navegador usa el parser de HTML, tolerante, **aunque el código esté escrito con estilo XHTML**.
- Solo si lo envía como `application/xhtml+xml` se usa el parser XML, estricto: un solo error de buen formado y la página no se muestra.

> **ALTERNATIVAS Y CRITERIO**
>
> Se puede escribir una página con sintaxis XHTML estricta, con un DOCTYPE de HTML 4.01, o con HTML actual. Hoy se elige HTML actual (`<!DOCTYPE html>`) porque es el único estándar mantenido, el que implementan los navegadores y el que comprueban los validadores. Escribir con disciplina XML (cerrar etiquetas, comillas siempre, minúsculas) sigue siendo una buena costumbre, porque hace el código más legible y más fácil de mantener. Pero es una elección de estilo: no convierte la página en XHTML ni la hace «más correcta» para el navegador. Solo tiene sentido servir una página como XHTML real si otro sistema tiene que procesarla como XML, algo poco frecuente hoy.

> **ERRORES FRECUENTES**
>
> - **Copiar el DOCTYPE de un tutorial antiguo.** Mucho material en Internet (y algunos apuntes) sigue usando DOCTYPE de HTML 4.01 o XHTML 1.0. No rompe la página, pero la ata a un estándar abandonado, y el validador actual lo marca como error. En un documento nuevo, siempre `<!DOCTYPE html>`.
> - **Creer que «HTML5» y «HTML» son lenguajes distintos.** HTML5 fue el nombre de una etapa. Hoy hay un solo HTML, el *Living Standard*, que incluye todo lo que se llamó HTML5.
> - **Llamar «lenguaje de marcas» a CSS o JavaScript.** Van dentro de las páginas, pero no marcan contenido.

> **PARA SABER MÁS**
>
> Como el estándar evoluciona continuamente, la pregunta útil no es «¿en qué versión de HTML está esto?», sino «¿lo soportan ya los navegadores?». Para responderla se consultan las tablas de compatibilidad de **MDN Web Docs** (la documentación de referencia de Mozilla, en developer.mozilla.org) y la web **Can I use** (caniuse.com). La especificación oficial está en html.spec.whatwg.org. Es muy extensa y no está pensada para aprender, pero es la fuente que manda en caso de duda.

---

## 2. Estructura de un documento HTML y herramientas de trabajo

### 2.1 Elementos, etiquetas y atributos

Un documento HTML está formado por **elementos**. Un elemento se compone normalmente de una **etiqueta de apertura**, un **contenido** y una **etiqueta de cierre**:

```html
<a href="contacto.html">Escríbeme</a>
```

Aquí `<a href="contacto.html">` es la etiqueta de apertura, `Escríbeme` es el contenido y `</a>` es la etiqueta de cierre. El conjunto completo es el elemento `a`. La distinción importa porque se suele decir «etiqueta» cuando se quiere decir «elemento». El elemento es la pieza del documento, y las etiquetas son solo las marcas que la delimitan.

Los **atributos** van siempre en la etiqueta de apertura, con la forma `nombre="valor"`, y aportan información adicional sobre el elemento. En el ejemplo, `href` indica el destino del enlace. Hay dos grandes grupos de atributos:

- **Atributos globales**: se pueden poner en cualquier elemento. Los más usados son `id` (identificador único en la página), `class` (una o varias clases, separadas por espacios, que se usarán desde CSS y JavaScript), `lang` (idioma del contenido), `title` (información complementaria, que el navegador suele mostrar como texto emergente), `hidden` (oculta el elemento) y `data-*` (datos propios, por ejemplo `data-precio="54.90"`).
- **Atributos específicos**: solo tienen sentido en ciertos elementos, como `href` en `a` o `src` y `alt` en `img`.

Algunos atributos son **booleanos**: su sola presencia significa «verdadero» y no necesitan valor. `<input required>` y `<input required="">` son equivalentes. Escribir `required="false"` **no** lo desactiva, porque el atributo sigue presente. Para desactivarlo hay que quitarlo.

Hay también **elementos vacíos** (*void elements*), que no pueden tener contenido y por tanto no llevan etiqueta de cierre: `area`, `base`, `br`, `col`, `embed`, `hr`, `img`, `input`, `link`, `meta`, `source`, `track` y `wbr`. En sintaxis HTML se escriben `<br>`. La forma `<br />`, heredada de XHTML, también se acepta: la barra final se ignora.

### 2.2 El esqueleto de un documento HTML

Este es el esqueleto completo de la página principal de un portfolio. Es válido según el Nu Html Checker, y cada línea está ahí por un motivo concreto:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Portfolio de Ana Martín · Desarrolladora web</title>
  <meta name="description" content="Portfolio de Ana Martín, estudiante de Desarrollo de Aplicaciones Web en el IES Río Arba.">
  <link rel="icon" href="img/favicon.svg" type="image/svg+xml">
</head>
<body>
  <h1>Ana Martín</h1>
  <p>Estudiante de 1º de Desarrollo de Aplicaciones Web.</p>
</body>
</html>
```

| Línea | Qué hace | Qué pasa si falta |
|---|---|---|
| `<!DOCTYPE html>` | Indica al navegador que procese el documento en **modo estándar** | El navegador entra en *quirks mode* (modo de compatibilidad con páginas de los 90) y algunas reglas de CSS se aplican de otra forma |
| `<html lang="es">` | Elemento raíz. `lang` declara el idioma del contenido | Los lectores de pantalla pueden pronunciar el texto con reglas de otro idioma, y los traductores automáticos y buscadores no saben en qué idioma está la página |
| `<head>` | Contenedor de **metadatos**: información *sobre* el documento, que no se muestra en la página | — |
| `<meta charset="utf-8">` | Declara la codificación de caracteres (lo que viste en la UD1 con la declaración XML). Debe ir al principio del `head` | Las tildes y la ñ pueden verse como caracteres extraños («Ã³» en lugar de «ó») |
| `<meta name="viewport" ...>` | Indica a los móviles que usen el ancho real de la pantalla | En el móvil, la página se muestra como si fuera de escritorio y encogida |
| `<title>` | Título del documento: aparece en la pestaña, en los marcadores y en los resultados de los buscadores. Es obligatorio | El validador da error. La pestaña muestra el nombre del archivo |
| `<meta name="description" ...>` | Resumen del contenido que los buscadores pueden mostrar bajo el título | El buscador elige por su cuenta un fragmento de la página |
| `<link rel="icon" ...>` | Relaciona el documento con otro recurso: aquí, el icono de la pestaña (*favicon*). También se usa para enlazar hojas de estilo (UD3) | El navegador intenta cargar `/favicon.ico` por su cuenta |
| `<body>` | Contenedor de **todo el contenido visible** de la página | — |

Un documento HTML, igual que un XML, tiene **un único elemento raíz** (`html`) con dos hijos directos: `head` y `body`. Visto como árbol, lo que el navegador construye en memoria (el **DOM**, *Document Object Model*) es este:

```text
html
├── head
│   ├── meta (charset)
│   ├── meta (viewport)
│   ├── title
│   ├── meta (description)
│   └── link (icon)
└── body
    ├── h1
    └── p
```

Es el mismo vocabulario de nodos (raíz, padre, hijo, hermano) que usaste en la UD1. En la UD4, JavaScript trabajará directamente sobre este árbol.

### 2.3 El navegador corrige, el validador avisa

En la UD1 comprobaste que el parser XML se detiene en el primer error. Los navegadores hacen justo lo contrario: la especificación de HTML define **con exactitud cómo debe recuperarse el parser de cualquier error**, para que todos los navegadores construyan el mismo árbol a partir de un documento defectuoso. Por eso una página mal escrita casi siempre «se ve bien», y eso es precisamente lo que hace peligrosos los errores: no se notan.

El caso extremo es este documento, que **el validador acepta como válido** (solo da un aviso por no declarar el idioma):

```html
<!DOCTYPE html>
<title>Documento mínimo</title>
<p>Este documento no tiene etiquetas html, head ni body.
<p>Y los párrafos no están cerrados.
```

No hay `html`, ni `head`, ni `body`, y los párrafos no se cierran. Es válido porque la especificación declara **opcionales** esas etiquetas en determinados contextos, y el navegador crea los elementos igualmente. Si lo abres y miras el inspector de DevTools, verás el árbol completo con `html`, `head` y `body`. Que sea válido no quiere decir que sea recomendable. En el módulo se escriben siempre todas las etiquetas, por legibilidad y por coherencia con XML.

Otra cosa son los **errores reales**, que el navegador tolera pero el validador detecta. Esta es la salida del Nu Html Checker sobre un documento con tres errores típicos:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>Con errores</title>
</head>
<body>
  <p>Texto en <b>negrita y <i>cursiva</b> mal anidadas</i>.</p>
  <img src="foto.jpg">
  <p id="intro">Primero</p>
  <p id="intro">Segundo</p>
</body>
</html>
```

```text
8.38-8.41: error: End tag “b” violates nesting rules.
9.3-9.22: error: An “img” element must have an “alt” attribute, except under certain conditions. For details, consult guidance on providing text alternatives for images.
11.3-11.16: error: Duplicate ID “intro”.
10.3-10.16: info warning: The first occurrence of ID “intro” was here.
```

El formato es el mismo que el del parser de Python en la UD1: `línea.columna` de inicio y de fin del fragmento, tipo de mensaje y explicación. La diferencia es que aquí **el validador sigue hasta el final** y lista todos los problemas, no solo el primero. Los tres errores son el anidamiento cruzado de `b` e `i` (el mismo error de buen formado que en XML), una imagen sin texto alternativo y un `id` repetido.

### 2.4 Herramientas para crear documentos web

Para escribir HTML basta un editor de texto plano, pero trabajar con herramientas profesionales ahorra tiempo y evita errores. En este módulo se usa este conjunto:

**Editor de código: Visual Studio Code.** Es gratuito, multiplataforma y el más extendido hoy. Lo que aporta frente a un editor genérico: resaltado de sintaxis, autocompletado de etiquetas y atributos, cierre automático de etiquetas, vista del documento como árbol plegable y **Emmet** (ya integrado, sin instalar nada).

**Emmet** convierte abreviaturas en código. La más útil es `!` seguida de la tecla Tab, que genera el esqueleto de un documento. Con la versión actual de Emmet, el resultado es:

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Document</title>
</head>
<body>
	
</body>
</html>
```

Fíjate en `lang="en"`: Emmet declara la página en inglés. Hay que cambiarlo a `es` cada vez o, mejor, configurarlo una sola vez en los ajustes de VS Code (`settings.json`):

```json
{
  "emmet.variables": {
    "lang": "es"
  }
}
```

Otras abreviaturas útiles: `p.intro` genera `<p class="intro"></p>`, y `nav>ul>li*3>a` genera un menú completo:

```html
<nav>
	<ul>
		<li><a href=""></a></li>
		<li><a href=""></a></li>
		<li><a href=""></a></li>
	</ul>
</nav>
```

El símbolo `>` significa «dentro de» y `*3` significa «repetido 3 veces». Es la misma idea de árbol padre-hijo, escrita en una sola línea.

**Navegador y herramientas de desarrollo (DevTools).** Se abren con F12 en Chrome, Edge y Firefox. Las pestañas que se usan en esta unidad son:

- **Elements / Inspector:** muestra el árbol DOM real que ha construido el navegador (no el código fuente que escribiste, que puede ser distinto si había errores corregidos). Al pasar el ratón sobre un nodo se resalta en la página.
- **Console / Consola:** mensajes y errores. También permite ejecutar JavaScript sobre la página.
- **Emulación de dispositivos:** muestra la página como se vería en un móvil (en Chrome y Edge, Ctrl+Mayús+M con DevTools abiertas).
- **Árbol de accesibilidad:** muestra cómo interpreta la página un lector de pantalla (lo usarás en la sesión 5).

**Validador: Nu Html Checker.** Es el validador oficial de HTML del W3C, disponible en validator.w3.org/nu. Permite comprobar una URL, subir un archivo o pegar código. Conviene pasar cada página por el validador antes de darla por terminada.

**Vista previa en vivo (opcional).** Hay extensiones de VS Code que abren la página en el navegador y la recargan cada vez que guardas (por ejemplo, *Live Preview*, de Microsoft, o *Live Server*). Son cómodas, pero no imprescindibles: basta con abrir el archivo en el navegador y pulsar F5.

**Control de versiones y publicación: Git y GitHub.** El portfolio se guarda en un repositorio y se puede publicar gratis como web con **GitHub Pages**. Lo verás en clase al organizar la entrega.

> **ALTERNATIVAS Y CRITERIO**
>
> Existen editores visuales (WYSIWYG, *What You See Is What You Get*), constructores de webs y asistentes de IA que generan HTML sin escribirlo a mano. Para un profesional son útiles en contextos concretos, pero en este módulo se trabaja con un editor de código por tres motivos. Primero, se ve y se controla cada línea. Segundo, el código resultante es limpio. Tercero, sin saber leer HTML no se puede evaluar si lo que genera otra herramienta está bien. Los editores visuales antiguos que aparecen en materiales de hace años (Dreamweaver en sus versiones de diseño, Amaya del W3C, ya discontinuado) generaban código con atributos presentacionales y tablas de maquetación, justo lo que el validador marca hoy como obsoleto. Con la IA pasa algo parecido: suele producir código que funciona pero con estructura pobre (ver sección 5). Se puede usar, pero siempre revisando su resultado con criterio.

> **ERRORES FRECUENTES**
>
> - **Escribir HTML en Word o en el Bloc de notas con otra codificación.** Word no guarda texto plano. El Bloc de notas antiguo podía guardar en ANSI aunque declarases `utf-8`, y entonces las tildes salen rotas. En VS Code, la codificación aparece en la barra inferior (debe decir UTF-8).
> - **Nombres de archivo con espacios, tildes o mayúsculas.** `Sobre Mí.html` funciona en tu Windows, pero al publicarla en un servidor Linux (que distingue mayúsculas y minúsculas) los enlaces fallan. Usa minúsculas, sin tildes y con guiones: `sobre-mi.html`.
> - **Pensar que si se ve bien, está bien.** El navegador corrige errores sin avisar. Solo el validador y el inspector te muestran lo que realmente hay.

> **PARA SABER MÁS**
>
> Estructura de carpetas recomendada para el portfolio. La página principal se llama siempre `index.html`, porque es el archivo que los servidores web entregan por defecto cuando se pide una carpeta:
>
> ```text
> portfolio/
> ├── index.html
> ├── sobre-mi.html
> ├── contacto.html
> ├── img/
> │   └── ana.jpg
> └── docs/
>     └── cv-ana-martin.pdf
> ```

---

## 3. Texto con significado

### 3.1 La regla de oro: semántica antes que apariencia

HTML describe **qué es** cada fragmento de contenido, no cómo se ve. Que un título aparezca grande y en negrita es una decisión del navegador, que CSS puede cambiar por completo. Lo que HTML aporta es que ese texto **es un encabezado**, y eso lo aprovechan otros programas además del navegador:

- **Lectores de pantalla:** una persona ciega puede saltar de encabezado en encabezado o listar todos los enlaces de la página, pero solo si están marcados como tales.
- **Buscadores:** dan más peso al texto de los encabezados para entender de qué trata una página.
- **Modo lectura, traductores, asistentes de IA:** extraen el contenido principal gracias a su estructura.

Por eso la pregunta al elegir una etiqueta nunca es «¿cómo quiero que se vea?», sino **«¿qué es este contenido?»**.

HTML clasifica sus elementos en **categorías de contenido** que determinan qué puede ir dentro de qué. Las dos que más se usan son el **contenido de flujo** (casi todo lo que puede ir en `body`: párrafos, listas, tablas, secciones) y el **contenido de fraseo** (*phrasing content*: el texto y los elementos que van dentro de un párrafo, como `strong`, `a` o `abbr`). Un párrafo solo admite contenido de fraseo. Por eso `<p><ul>...</ul></p>` no es correcto: una lista no puede ir dentro de un párrafo.

### 3.2 Encabezados

Hay seis niveles, de `h1` a `h6`, que forman la **jerarquía** del documento, como el índice de un libro:

- `h1` es el título principal de la página. La práctica recomendada es que haya **uno**.
- Los niveles **no se saltan** al bajar: debajo de un `h2` va un `h3`, no un `h4`. Al subir sí se puede volver a cualquier nivel superior.
- Se elige el nivel por su **posición en la jerarquía**, nunca por su tamaño. Si un `h2` te parece demasiado grande, se cambia con CSS, no se cambia por `h4`.

### 3.3 Párrafos, saltos y separadores

- `p`: párrafo. Es la unidad básica de texto.
- `br`: salto de línea **que forma parte del contenido**, como en los versos de un poema o las líneas de una dirección postal. No se usa para separar párrafos ni para crear espacio vertical (eso es CSS).
- `hr`: **cambio temático** entre párrafos (por ejemplo, un cambio de escena en un relato). Aunque el navegador lo dibuje como una línea, su significado no es «línea horizontal».

### 3.4 Listas

| Elemento | Uso | Atributos propios |
|---|---|---|
| `ul` + `li` | Lista sin orden: el orden de los elementos no cambia el significado | — |
| `ol` + `li` | Lista ordenada: pasos, clasificaciones | `start` (número inicial), `reversed` (cuenta atrás), `type` (`1`, `a`, `A`, `i`, `I`) |
| `dl` + `dt` + `dd` | Lista de descripciones: pares término-definición, pregunta-respuesta, nombre-valor | — |

Las listas se pueden **anidar**: la sublista va **dentro** del `li` al que pertenece, no suelta entre dos `li`. Un menú de navegación también es, semánticamente, una lista de enlaces.

### 3.5 Semántica en línea

Estos elementos marcan fragmentos dentro de un párrafo. Varios de ellos se ven igual en el navegador pero significan cosas distintas:

| Elemento | Significado | Ejemplo de uso |
|---|---|---|
| `strong` | Importancia, gravedad o urgencia | «**No** desconectes el equipo durante la actualización» |
| `em` | Énfasis que cambia el sentido de la frase (el que pondrías con la voz) | «Yo *no* he dicho eso» |
| `b` | Llamar la atención sin añadir importancia | Palabras clave de un resumen, nombres de producto |
| `i` | Voz o tono distinto: términos técnicos, palabras en otro idioma, pensamientos | «Esto se llama *parsing*» (mejor con `lang="en"`) |
| `mark` | Resaltado por relevancia en el contexto actual | Coincidencias de una búsqueda |
| `small` | Letra pequeña: avisos legales, créditos | «© 2026» |
| `s` | Contenido que ya no es correcto o relevante | Precio anterior en una oferta |
| `del` / `ins` | Texto eliminado o insertado en una revisión del documento | Correcciones de un texto, con `datetime` |
| `abbr` | Abreviatura o sigla, con la forma completa en `title` | `<abbr title="Desarrollo de Aplicaciones Web">DAW</abbr>` |
| `time` | Fecha u hora, con valor legible por máquina en `datetime` | `<time datetime="2026-10-14">14 de octubre</time>` |
| `code` | Fragmento de código | `<code>&lt;br&gt;</code>` |
| `kbd` | Tecla o entrada de teclado | `<kbd>F12</kbd>` |
| `samp` | Salida de un programa | Un mensaje de error |
| `var` | Variable (matemática o de programación) | *x* |
| `sub` / `sup` | Subíndice / superíndice, cuando forman parte del significado | H<sub>2</sub>O, m<sup>2</sup> |
| `q` / `cite` | Cita corta en línea / título de una obra | `<cite>El Quijote</cite>` |
| `dfn` | Término que se está definiendo en esa misma frase | «Un *parser* es…» |
| `span` | Ninguno: contenedor genérico en línea, para aplicar estilo o script | Último recurso, cuando no hay elemento semántico adecuado |

`b`, `i`, `s` y `small` existían en HTML 4 como etiquetas puramente visuales. En HTML actual **no son obsoletas**: se redefinieron con un significado propio, y se pueden usar cuando ese significado encaja.

### 3.6 Citas, código preformateado y entidades

- `blockquote` marca una cita larga, de bloque. El atributo `cite` puede indicar la URL de origen, pero no se muestra. Si quieres que el lector vea la fuente, escríbela en el texto.
- `pre` conserva los espacios y saltos de línea tal como están escritos. Combinado con `code` (`<pre><code>…</code></pre>`) es la forma estándar de mostrar un bloque de código.
- Los caracteres `<` y `&` tienen significado especial en HTML, igual que en XML. Para mostrarlos como texto se usan **entidades**: `&lt;` (<), `&gt;` (>) y `&amp;` (&). En valores de atributos entre comillas, `&quot;` ("). También existe `&nbsp;` (espacio que impide un salto de línea, útil entre un número y su unidad: «67&nbsp;h») y `&copy;` (©).
- Con `utf-8` **no hace falta** escribir las tildes como entidades (`&aacute;`), como se hacía en páginas antiguas. Se escriben directamente.
- Los **comentarios** se escriben `<!-- así -->`. No se muestran en la página, pero **sí se envían al navegador** y cualquiera puede leerlos con «Ver código fuente». Nunca se dejan contraseñas ni información interna en un comentario.

### 3.7 Ejemplo completo: la página «Sobre mí»

Documento válido según el Nu Html Checker, que reúne casi todos los elementos de este bloque:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Sobre mí · Portfolio de Ana Martín</title>
</head>
<body>
  <h1>Sobre mí</h1>
  <p>Me llamo Ana y estudio <abbr title="Desarrollo de Aplicaciones Web">DAW</abbr> en el IES Río Arba de Tauste. Antes cursé el ciclo de <abbr title="Sistemas Microinformáticos y Redes">SMR</abbr>, así que las redes no me asustan.</p>

  <h2>Qué sé hacer</h2>
  <ul>
    <li>Maquetar páginas con <strong>HTML semántico</strong>.</li>
    <li>Programar en Python:
      <ul>
        <li>scripts de automatización,</li>
        <li>lectura de ficheros <abbr title="Comma-Separated Values">CSV</abbr>.</li>
      </ul>
    </li>
    <li>Administrar equipos con Windows y Linux.</li>
  </ul>

  <h2>Mi forma de trabajar</h2>
  <ol>
    <li>Entender el problema antes de escribir código.</li>
    <li>Hacer una primera versión que funcione.</li>
    <li>Revisarla con el validador y mejorarla.</li>
  </ol>
  <p>Una regla que aprendí en clase: <em>nunca</em> se elige una etiqueta por cómo se ve, sino por lo que significa.</p>

  <h2>Glosario personal</h2>
  <dl>
    <dt>Parser</dt>
    <dd>Programa que lee un documento y construye su estructura en memoria.</dd>
    <dt>DOM</dt>
    <dd>Árbol de objetos que representa el documento dentro del navegador.</dd>
  </dl>

  <h2>Una cita que me gusta</h2>
  <blockquote cite="https://www.w3.org/WAI/">
    <p>The power of the Web is in its universality. Access by everyone regardless of disability is an essential aspect.</p>
  </blockquote>
  <p>— Tim Berners-Lee, inventor de la web, citado por la <cite>Iniciativa de Accesibilidad Web (WAI) del W3C</cite></p>

  <h2>Un fragmento de código</h2>
  <p>Para abrir la consola del navegador pulso <kbd>F12</kbd>. Esta es la primera línea de cualquier página que escribo:</p>
  <pre><code>&lt;!DOCTYPE html&gt;</code></pre>

  <hr>
  <p><small>Última actualización: <time datetime="2026-10-14">14 de octubre de 2026</time>.</small></p>
  <!-- Pendiente: añadir enlace a mi GitHub en la S4 -->
</body>
</html>
```

> **ALTERNATIVAS Y CRITERIO**
>
> Casi cualquier cosa de este bloque podría hacerse con `div` y `span` y darles apariencia con CSS. Visualmente el resultado sería idéntico, pero el documento perdería todo su significado para los lectores de pantalla, los buscadores y cualquier programa que lo procese. Se elige el elemento semántico siempre que exista uno cuyo significado encaje. `div` y `span` se reservan para cuando no hay ninguno. Entre `strong` y `b`, el criterio es preguntarse si el texto es **más importante** que el que lo rodea (`strong`) o solo se quiere **destacar visualmente** sin cambiar su importancia (`b`).

> **ERRORES FRECUENTES**
>
> - **Elegir el encabezado por su tamaño**, poniendo `h4` porque `h2` «es muy grande». Rompe la jerarquía que usan los lectores de pantalla.
> - **Usar `br` o `&nbsp;` repetidos para crear espacio.** `<br><br><br>` para separar bloques, o varios `&nbsp;` para sangrar. El espaciado es cosa de CSS.
> - **Meter listas o encabezados dentro de un párrafo.** El navegador cierra el párrafo automáticamente antes de la lista, y el árbol resultante no es el que creías.
> - **Escribir `<` sin escapar dentro de `code`.** El navegador lo interpreta como el comienzo de una etiqueta y el ejemplo desaparece de la página.

> **PARA SABER MÁS**
>
> Los lectores de pantalla (NVDA en Windows, que es gratuito, o VoiceOver en macOS y iOS) permiten navegar por la página saltando de encabezado en encabezado. Probar tu portfolio con uno de ellos durante un par de minutos es la forma más rápida de entender por qué importa la semántica.

---

## 4. Enlaces, rutas e imágenes

### 4.1 El elemento `a`

El enlace (*anchor*, ancla) es lo que convierte un conjunto de documentos en una **web**: es la «H» de HTML (*HyperText*). Su atributo esencial es `href`, el destino. El **texto del enlace** debe describir adónde lleva, porque los lectores de pantalla permiten listar todos los enlaces de una página fuera de contexto. Una lista con cinco enlaces que dicen «pulsa aquí» no sirve para nada.

| Tipo de destino | Ejemplo de `href` | Qué hace |
|---|---|---|
| Otra página del mismo sitio | `sobre-mi.html` | Navega a esa página |
| Página externa | `https://github.com/` | Navega a otro sitio |
| Un punto de la misma página | `#ultimo-proyecto` | Salta al elemento con ese `id` |
| Un punto de otra página | `sobre-mi.html#formacion` | Navega y salta al `id` |
| Correo electrónico | `mailto:ana@example.com?subject=Hola` | Abre el cliente de correo (los espacios del asunto se escriben `%20`) |
| Teléfono | `tel:+34976000000` | En móvil, ofrece llamar |

Otros atributos de `a`:

- `download`: indica que el recurso debe **descargarse** en vez de abrirse. Solo funciona con archivos del mismo sitio.
- `target="_blank"`: abre el destino en una pestaña nueva. Hay que usarlo con moderación, porque quita al usuario la decisión de dónde abrirlo. Si se usa, se acompaña de `rel="noopener noreferrer"`. `noopener` impide que la página abierta pueda manipular la pestaña de origen a través de JavaScript. Los navegadores actuales ya aplican `noopener` por defecto con `_blank`, pero se sigue indicando explícitamente por claridad. `noreferrer` evita que la página de destino sepa desde qué página llegó el usuario. También conviene avisar en el texto de que se abrirá una pestaña nueva.

### 4.2 Rutas absolutas y relativas

Una **URL absoluta** incluye el protocolo y el dominio: `https://github.com/usuario`. Se usa para enlazar a otros sitios.

Una **ruta relativa** se interpreta **a partir de la ubicación del documento actual**. Se usa para todo lo que está dentro del propio sitio, porque sigue funcionando aunque el sitio cambie de dominio o se abra desde el disco. Para este árbol de carpetas:

```text
portfolio/
├── index.html
├── sobre-mi.html
├── img/
│   └── ana.jpg
└── proyectos/
    ├── index.html
    └── xml.html
```

| Desde | Hacia | Ruta relativa |
|---|---|---|
| `index.html` | `sobre-mi.html` | `sobre-mi.html` (o `./sobre-mi.html`) |
| `index.html` | `img/ana.jpg` | `img/ana.jpg` |
| `index.html` | `proyectos/xml.html` | `proyectos/xml.html` |
| `proyectos/xml.html` | `index.html` (el principal) | `../index.html` |
| `proyectos/xml.html` | `img/ana.jpg` | `../img/ana.jpg` |
| `proyectos/xml.html` | `proyectos/index.html` | `index.html` |

`./` significa «esta carpeta» y `../` significa «la carpeta de arriba». Se pueden encadenar (`../../`).

Existe una tercera forma, la ruta **relativa a la raíz del sitio**, que empieza por `/`: `/img/ana.jpg`. Se resuelve desde la raíz del dominio. Es útil en sitios grandes, pero **no funciona al abrir los archivos directamente desde el disco** (con `file://`), porque la «raíz» pasa a ser la raíz de la unidad. Tampoco funciona cuando el sitio se publica en una subcarpeta, como ocurre en GitHub Pages con `usuario.github.io/portfolio/`. En el portfolio se usan rutas relativas.

### 4.3 Imágenes

```html
<img src="img/ana.jpg" alt="Ana Martín sonriendo delante de un ordenador" width="320" height="320">
```

- `src`: ruta de la imagen.
- `alt`: **texto alternativo**, obligatorio. Es lo que lee un lector de pantalla, lo que se muestra si la imagen no carga y lo que usan los buscadores. Debe transmitir **la misma información que la imagen en su contexto**, no describir que es una imagen («foto de…», «imagen de…» sobra). Si la imagen es **puramente decorativa**, se escribe `alt=""` (vacío), y el lector de pantalla la ignora. No es lo mismo omitir el atributo, que es un error, que dejarlo vacío, que es una decisión.
- `width` y `height`: dimensiones intrínsecas en píxeles, **sin unidad**. No sirven para cambiar el tamaño en pantalla (eso lo hará CSS), sino para que el navegador **reserve el hueco** antes de que la imagen se descargue. Así la página no «salta» cuando llega la imagen.
- `loading="lazy"`: la imagen no se descarga hasta que el usuario se acerca a ella al desplazarse. Se usa en imágenes que no están visibles al cargar la página.

| Formato | Tipo | Cuándo usarlo |
|---|---|---|
| JPEG | Mapa de bits, con pérdida | Fotografías |
| PNG | Mapa de bits, sin pérdida, con transparencia | Capturas de pantalla, imágenes con texto o transparencia |
| GIF | Mapa de bits, 256 colores, animación | Prácticamente sustituido por vídeo o WebP animado |
| SVG | Vectorial (XML) | Logotipos, iconos, gráficos: se escala sin perder calidad |
| WebP | Mapa de bits, con o sin pérdida | Alternativa moderna a JPEG y PNG, con archivos más ligeros |
| AVIF | Mapa de bits, con o sin pérdida | Aún más ligero que WebP, compatible con los navegadores actuales |

`figure` agrupa un contenido autónomo (una imagen, un diagrama, un fragmento de código) con su pie, `figcaption`. El pie es visible para todo el mundo y **no sustituye** al `alt`: son complementarios.

`picture` permite ofrecer **varias versiones** de una imagen para que el navegador elija. Contiene elementos `source` (con `srcset`, `media` y `type`) y, al final, un `img` obligatorio, que es el que se usa si ninguna fuente encaja y el que lleva el `alt`.

### 4.4 Audio y vídeo

`video` y `audio` reproducen contenido multimedia sin complementos. Antes de HTML5 hacía falta Flash, que desapareció a finales de 2020.

- `controls` muestra los controles de reproducción. Sin él, el usuario no puede pausar.
- Varios `source` con distintos formatos (`type`): el navegador usa el primero que sabe reproducir.
- `track` añade subtítulos en formato WebVTT (`.vtt`), necesarios para la accesibilidad.
- El texto dentro del elemento solo se muestra en navegadores que no soportan `video`.
- `autoplay` con sonido está bloqueado por los navegadores actuales. Solo se permite con el vídeo silenciado (`muted`).

Para incrustar un vídeo de una plataforma externa (YouTube, por ejemplo) se usa el código `iframe` que proporciona la propia plataforma.

### 4.5 Ejemplo completo: la página principal del portfolio

Documento válido según el Nu Html Checker. El validador no comprueba que los archivos enlazados existan: esa comprobación es tuya.

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Portfolio de Ana Martín · Desarrolladora web</title>
</head>
<body>
  <p>
    <a href="index.html">Inicio</a> ·
    <a href="sobre-mi.html">Sobre mí</a> ·
    <a href="proyectos/index.html">Proyectos</a> ·
    <a href="contacto.html">Contacto</a>
  </p>

  <h1>Ana Martín</h1>
  <figure>
    <img src="img/ana.jpg" alt="Ana Martín sonriendo delante de un ordenador" width="320" height="320">
    <figcaption>En el aula de informática del IES Río Arba.</figcaption>
  </figure>
  <p>Estudiante de Desarrollo de Aplicaciones Web. Puedes leer <a href="sobre-mi.html">más sobre mí y lo que sé hacer</a> o ir directamente a <a href="#ultimo-proyecto">mi último proyecto</a>.</p>

  <h2>Dónde encontrarme</h2>
  <ul>
    <li><a href="https://github.com/" target="_blank" rel="noopener noreferrer">Mi perfil de GitHub</a> (se abre en una pestaña nueva)</li>
    <li><a href="mailto:ana.martin@example.com?subject=Contacto%20desde%20el%20portfolio">Escríbeme un correo</a></li>
    <li><a href="docs/cv-ana-martin.pdf" download>Descarga mi currículum (PDF, 120 KB)</a></li>
  </ul>

  <h2 id="ultimo-proyecto">Último proyecto</h2>
  <p>Una ficha de producto en XML, validada con el parser de Python.</p>
  <picture>
    <source srcset="img/proyecto-movil.webp" media="(max-width: 600px)" type="image/webp">
    <img src="img/proyecto.jpg" alt="Captura de la ficha de producto mostrada en el navegador" width="800" height="450" loading="lazy">
  </picture>

  <h2>Vídeo de presentación</h2>
  <video controls width="640" height="360" poster="img/portada-video.jpg">
    <source src="media/presentacion.webm" type="video/webm">
    <source src="media/presentacion.mp4" type="video/mp4">
    <track src="media/presentacion.es.vtt" kind="subtitles" srclang="es" label="Español" default>
    Tu navegador no puede reproducir el vídeo. <a href="media/presentacion.mp4">Descárgalo aquí</a>.
  </video>
</body>
</html>
```

> **ALTERNATIVAS Y CRITERIO**
>
> Una misma imagen puede ir en HTML (`img`) o como fondo desde CSS (`background-image`, UD3). El criterio es el significado. Si la imagen **es contenido** (la foto del autor, una captura de un proyecto), va en HTML con su `alt`, porque forma parte de la información. Si es **decoración** (una textura, un degradado), va en CSS, y así no molesta a los lectores de pantalla.

> **ERRORES FRECUENTES**
>
> - **Rutas absolutas de tu disco**: `src="C:\Users\ana\Desktop\portfolio\img\foto.jpg"`. Funciona en tu ordenador y en ningún otro. Además, la barra invertida `\` no es un separador válido en URL.
> - **Mayúsculas que no coinciden**: `Foto.JPG` en el disco y `foto.jpg` en el `src`. Windows lo perdona, pero el servidor Linux donde se publica no.
> - **`alt` que no dice nada**: `alt="imagen"`, `alt="foto1.jpg"`, o repetir literalmente el pie de foto.
> - **Enlaces con texto «aquí» o «pulsa aquí».** El texto del enlace debe tener sentido leído de forma aislada.

> **PARA SABER MÁS**
>
> Un **servidor web** entrega `index.html` cuando se pide una carpeta. Por eso `https://ejemplo.com/proyectos/` muestra `proyectos/index.html`. Si publicas el portfolio con GitHub Pages, la dirección `https://tu-usuario.github.io/portfolio/` mostrará tu `index.html`.

---

## 5. Estructura semántica de la página

### 5.1 El problema: la «sopa de divs»

Antes de HTML5 no había elementos para las zonas de una página (cabecera, menú, contenido, pie), así que todo se construía con `div`, el contenedor genérico, diferenciado solo por el nombre de sus clases. Hoy se sigue viendo mucho, en plantillas antiguas y en código generado por herramientas y por IA. Este documento es un ejemplo:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Blog de Ana</title>
</head>
<body>
  <div class="cabecera">
    <div class="logo">Blog de Ana</div>
    <div class="menu">
      <div><a href="index.html">Inicio</a></div>
      <div><a href="archivo.html">Archivo</a></div>
    </div>
  </div>
  <div class="contenido">
    <div class="entrada">
      <div class="titulo">Mi primer documento XML</div>
      <div class="fecha">30/09/2026</div>
      <div class="texto">Hoy he construido un documento XML desde un archivo vacío.</div>
    </div>
    <div class="lateral">
      <div class="titulo">Sobre mí</div>
      <div class="texto">Estudio DAW en Tauste.</div>
    </div>
  </div>
  <div class="pie">© 2026 Ana Martín</div>
</body>
</html>
```

**Este documento es válido**: el Nu Html Checker no da ni un aviso. Ese es el punto clave de esta sección. **Válido no significa bien estructurado.** El validador comprueba que se cumplan las reglas del lenguaje, no que se hayan elegido los elementos adecuados.

¿Qué problemas tiene? Para el navegador, para un lector de pantalla o para un buscador, `class="cabecera"` y `class="menu"` no significan nada: las clases son nombres inventados por quien escribió el código. Un lector de pantalla no puede ofrecer «saltar al contenido principal» ni «ir al menú», porque no sabe dónde están. Y no hay ni un solo encabezado: el «título» de la entrada es un `div` que solo parece un título si CSS lo pinta grande.

### 5.2 Los elementos de sección

| Elemento | Significado | Criterio de uso |
|---|---|---|
| `header` | Cabecera introductoria de la página o de una sección/artículo | Logo, título, menú principal. Puede haber varios: uno para la página y uno dentro de cada `article` |
| `nav` | Bloque de **navegación principal** | Menú del sitio, índice de la página. No cualquier grupo de enlaces: los enlaces a redes sociales del pie no necesitan `nav` |
| `main` | **Contenido principal** y único de la página | Solo uno visible por página. No va dentro de `header`, `footer`, `article`, `aside` ni `nav` |
| `article` | Contenido **autónomo**, con sentido por sí solo si se sacara de la página | Una entrada de blog, una noticia, un comentario, una ficha de producto |
| `section` | Agrupación **temática** de contenido, normalmente con su propio encabezado | Los capítulos de un artículo, las secciones de una página de inicio |
| `aside` | Contenido **relacionado de forma tangencial** con lo que lo rodea | Barra lateral, cuadro de «sabías que», publicidad |
| `footer` | Pie de la página o de una sección/artículo | Autoría, copyright, contacto, enlaces legales |
| `address` | Información de **contacto** del autor de la página o del artículo | No se usa para cualquier dirección postal que aparezca en el texto |
| `search` | Zona de **búsqueda o filtrado** | Contiene el formulario de búsqueda. Se incorporó al estándar en 2023 |
| `div` | Ninguno | Solo para agrupar con fines de estilo o script cuando ningún elemento anterior encaja |

Para decidir entre `article`, `section` y `div`, un criterio práctico en tres preguntas:

1. ¿Tendría sentido este contenido **publicado solo**, por ejemplo en otra web o en un lector de noticias? → `article`.
2. Si no, ¿es una **parte temática** del documento que merece su propio encabezado (que aparecería en un índice)? → `section`.
3. Si solo lo agrupas para darle estilo o manipularlo con JavaScript → `div`.

### 5.3 La misma página, con estructura semántica

Refactorización del documento anterior. También es válido, pero ahora la estructura significa algo:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Blog de Ana</title>
</head>
<body>
  <header>
    <p class="logo">Blog de Ana</p>
    <nav aria-label="Principal">
      <ul>
        <li><a href="index.html" aria-current="page">Inicio</a></li>
        <li><a href="archivo.html">Archivo</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <h1>Últimas entradas</h1>
    <article>
      <header>
        <h2>Mi primer documento XML</h2>
        <p>Publicado el <time datetime="2026-09-30">30 de septiembre de 2026</time></p>
      </header>
      <p>Hoy he construido un documento XML desde un archivo vacío.</p>
      <section>
        <h3>Lo que aprendí</h3>
        <p>Que un solo carácter &amp; sin escapar basta para que el parser se detenga.</p>
      </section>
    </article>
  </main>

  <aside>
    <h2>Sobre mí</h2>
    <p>Estudio DAW en Tauste.</p>
  </aside>

  <footer>
    <p>© 2026 Ana Martín · <a href="mailto:ana.martin@example.com">ana.martin@example.com</a></p>
  </footer>
</body>
</html>
```

Cambios, uno a uno:

- La cabecera, el menú, el contenido, la barra lateral y el pie usan su elemento propio.
- El menú es una **lista** de enlaces dentro de `nav`. `aria-label="Principal"` da nombre a la zona de navegación. Es útil cuando hay más de un `nav`. `aria-current="page"` indica en qué página está el usuario.
- La página tiene un `h1` y la entrada tiene su `h2`. La jerarquía de encabezados ya no depende de CSS.
- La entrada es un `article` con su propio `header`, y la fecha es un `time` con valor legible por máquina.
- El logo de texto pasa a ser un `p`, no un `h1`, porque el `h1` es el título de *esta* página («Últimas entradas»). Es una decisión de diseño que se puede discutir: en la página de inicio de muchos sitios el nombre del sitio sí es el `h1`.

### 5.4 Árbol de accesibilidad y roles implícitos

Además del DOM, el navegador construye un **árbol de accesibilidad**, que es lo que reciben los lectores de pantalla. En él, cada elemento semántico tiene un **rol**:

| Elemento | Rol implícito |
|---|---|
| `header` (hijo directo de `body`) | `banner` |
| `nav` | `navigation` |
| `main` | `main` |
| `aside` | `complementary` |
| `footer` (hijo directo de `body`) | `contentinfo` |
| `article` | `article` |
| `search` | `search` |
| `div` | ninguno (`generic`) |

Estas zonas con rol se llaman **regiones** o *landmarks*, y son las que un lector de pantalla ofrece como atajos. Puedes verlas en DevTools: en Chrome y Edge, en el panel de accesibilidad del inspector. En la versión con `div` no aparece ninguna región.

Existe el atributo `role` para asignar roles a mano (`<div role="navigation">`), dentro de la especificación **WAI-ARIA**. La primera regla de uso de ARIA, según el propio W3C, es **no usar ARIA si existe un elemento HTML nativo** con ese significado. `<nav>` es siempre preferible a `<div role="navigation">`.

> **ALTERNATIVAS Y CRITERIO**
>
> Las dos versiones de este bloque se ven igual con el CSS adecuado y las dos son válidas. Se elige la semántica porque hace el documento **comprensible para programas**, no solo para personas que lo ven. Esto tiene consecuencias legales, además de técnicas: la accesibilidad es obligatoria en las webs del sector público y cada vez en más servicios privados. Un `div` sigue siendo la opción correcta cuando no hay significado que expresar: un contenedor para colocar dos columnas con CSS no es una `section`.

> **ERRORES FRECUENTES**
>
> - **Cambiar todos los `div` por `section`.** `section` no es un `div` con otro nombre: debe ser una parte temática con encabezado. Usarla como contenedor genérico es tan poco semántico como la sopa de divs.
> - **Poner más de un `main` visible**, o meter `main` dentro de `header` o de `article`.
> - **Confundir `header` con `h1` o con `head`.** `head` son los metadatos del documento. `header` es la zona de cabecera visible. `h1` es un encabezado de texto.
> - **Envolver en `nav` cualquier grupo de enlaces.** Se reserva para la navegación principal y los índices.

> **PARA SABER MÁS**
>
> En materiales de hace unos años se lee que dentro de cada `section` o `article` se puede volver a empezar por `h1`, porque el navegador calcularía los niveles automáticamente (el llamado *algoritmo de outline*). **Ningún navegador llegó a implementarlo** y se eliminó de la especificación. La jerarquía la marcan los niveles `h1`-`h6` que escribas.

---

## 6. Tablas de datos

### 6.1 Cuándo usar una tabla

Una tabla sirve para **datos tabulares**: información que tiene sentido leída por filas y por columnas a la vez, como un horario, una comparativa o un listado de notas. Durante los años 90 y 2000, las tablas se usaron también para **maquetar** (colocar la cabecera, el menú lateral y el contenido en celdas). Esa práctica es incorrecta: un lector de pantalla lee la página celda a celda, anunciando «fila 2, columna 1», y la estructura no tiene ningún significado. La disposición visual se hace con CSS (UD3).

### 6.2 Elementos de una tabla

| Elemento | Función |
|---|---|
| `table` | Contenedor de la tabla |
| `caption` | Título de la tabla. Es el primer hijo de `table`, y los lectores de pantalla lo anuncian al llegar a ella |
| `thead`, `tbody`, `tfoot` | Grupos de filas: cabecera, cuerpo y pie (totales, notas). Permiten, por ejemplo, repetir la cabecera en cada página al imprimir |
| `tr` | Fila (*table row*) |
| `th` | Celda de **encabezado**. Con `scope="col"` encabeza una columna. Con `scope="row"`, una fila |
| `td` | Celda de **datos** (*table data*) |
| `colgroup`, `col` | Agrupan columnas para aplicarles estilo |

`scope` es lo que permite a un lector de pantalla anunciar, al llegar a una celda, a qué encabezados pertenece («Horas: 2000»). En tablas muy complejas, donde `scope` no basta, se asocia cada celda a sus encabezados con los atributos `id` (en `th`) y `headers` (en `td`).

### 6.3 Celdas combinadas

- `colspan="n"`: la celda ocupa `n` **columnas** hacia la derecha.
- `rowspan="n"`: la celda ocupa `n` **filas** hacia abajo.

La clave para no equivocarse es que **cada fila tiene que sumar el mismo número de columnas**, contando las celdas que «bajan» desde filas anteriores con `rowspan`. Esas celdas ya no se escriben en las filas siguientes. Lo más fiable es **dibujar la tabla en papel** antes de escribirla, marcando qué celdas se fusionan.

Ejemplo: un horario de 6 columnas (hora + 5 días), 2 filas de datos:

```text
+-------+--------+--------+-----------+-------------+-----------+
| Hora  | Lunes  | Martes | Miércoles | Jueves      | Viernes   |
+-------+--------+--------+-----------+-------------+-----------+
| 9:25  |  Otros módulos  | LMSGI     | Otro módulo |  Otros    |
+-------+  colspan="2"    +-----------+-------------+  módulos  |
| 10:20 |  rowspan="2"    | Otro mód. | LMSGI       | rowspan=2 |
+-------+-----------------+-----------+-------------+-----------+
```

En la segunda fila solo se escriben 3 celdas (hora, miércoles, jueves), porque lunes, martes y viernes vienen ocupados desde la fila anterior.

### 6.4 Ejemplo completo

Documento válido según el Nu Html Checker, con una tabla con `thead`/`tbody`/`tfoot` y el horario con celdas combinadas:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Formación · Portfolio de Ana Martín</title>
</head>
<body>
  <h1>Formación</h1>
  <table>
    <caption>Estudios cursados y en curso</caption>
    <thead>
      <tr>
        <th scope="col">Periodo</th>
        <th scope="col">Estudios</th>
        <th scope="col">Centro</th>
        <th scope="col">Horas</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th scope="row">2024-2026</th>
        <td>CFGM Sistemas Microinformáticos y Redes</td>
        <td>IES Río Arba</td>
        <td>2000</td>
      </tr>
      <tr>
        <th scope="row">2026-2028</th>
        <td>CFGS Desarrollo de Aplicaciones Web</td>
        <td>IES Río Arba</td>
        <td>2000</td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <th scope="row" colspan="3">Total</th>
        <td>4000</td>
      </tr>
    </tfoot>
  </table>

  <h2>Horario de Lenguajes de Marcas</h2>
  <table>
    <caption>Sesiones semanales de 0373 en 1º DAW</caption>
    <thead>
      <tr>
        <th scope="col">Hora</th>
        <th scope="col">Lunes</th>
        <th scope="col">Martes</th>
        <th scope="col">Miércoles</th>
        <th scope="col">Jueves</th>
        <th scope="col">Viernes</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th scope="row">9:25</th>
        <td rowspan="2" colspan="2">Otros módulos</td>
        <td>LMSGI</td>
        <td>Otro módulo</td>
        <td rowspan="2">Otros módulos</td>
      </tr>
      <tr>
        <th scope="row">10:20</th>
        <td>Otro módulo</td>
        <td>LMSGI</td>
      </tr>
    </tbody>
  </table>
</body>
</html>
```

### 6.5 Atributos de tabla obsoletos

En materiales antiguos aparecen tablas con atributos de presentación. Esta es la salida real del validador sobre una tabla escrita así:

```html
<table border="1" cellpadding="4" width="100%" bgcolor="#eeeeee">
<tr><td align="center">A</td></tr>
</table>
```

```text
5.1-5.65: info warning: The “border” attribute on the “table” element is obsolete. Consider specifying “img { border: 0; }” in CSS instead.
5.1-5.65: info warning: The “cellpadding” attribute on the “table” element is obsolete. Use CSS instead.
5.1-5.65: info warning: The “width” attribute on the “table” element is obsolete. Use CSS instead.
5.1-5.65: info warning: The “bgcolor” attribute on the “table” element is obsolete. Use CSS instead.
6.5-6.23: info warning: The “align” attribute on the “td” element is obsolete. Use CSS instead.
```

Son avisos (*info warning*), no errores: los atributos son obsoletos y todo eso se hace con CSS. Un detalle curioso: el mensaje sobre `border` en `table` sugiere una regla CSS para `img`. Es una errata del propio validador, que reutiliza un texto pensado para imágenes. Las herramientas también se equivocan, y hay que leer sus mensajes con criterio.

> **ALTERNATIVAS Y CRITERIO**
>
> Unos datos se pueden presentar como tabla, como lista (`ul` o `dl`) o como tarjetas maquetadas con CSS. El criterio es la **dimensión** de los datos. Si cada dato depende de **dos ejes** (fila y columna: qué módulo hay el martes a las 10:20), es una tabla. Si solo hay un eje (una lista de habilidades) o pares nombre-valor (una ficha), una lista es más sencilla y más accesible.

> **ERRORES FRECUENTES**
>
> - **Usar tablas para colocar elementos en la página.** Es el error clásico de la web de los 2000, y el validador no lo detecta.
> - **Usar `td` con negrita en lugar de `th`** para los encabezados. Visualmente queda igual, pero se pierde la relación entre encabezado y datos.
> - **Filas que no suman lo mismo** por olvidar las celdas ocupadas por un `rowspan`. La tabla se descuadra y las celdas «sobrantes» se desplazan a la derecha.
> - **Olvidar `caption`.** Sin él, quien usa un lector de pantalla no sabe de qué es la tabla hasta haber leído varias celdas.

---

## 7. Formularios

### 7.1 Qué hace un formulario

Un formulario recoge datos del usuario y los **envía a un programa en un servidor** que los procesa (PHP, Python, Node.js…). HTML solo define la parte del cliente: los campos y cómo se envían. El procesamiento se verá en otros módulos del ciclo.

Al enviar, el navegador recoge **cada campo que tenga atributo `name`** y construye pares `nombre=valor`. Un campo sin `name` **no se envía**, aunque el usuario lo haya rellenado. El elemento `form` decide adónde y cómo se envían:

- `action`: URL del programa que recibe los datos.
- `method`: `get` o `post`.

### 7.2 GET frente a POST

Con **GET**, los datos viajan **en la propia URL**, a continuación de un `?`, separados por `&`, y con los caracteres especiales codificados (la `@` pasa a ser `%40` y los espacios, `+` o `%20`):

```text
https://ejemplo.com/contacto?nombre=Ana+Mart%C3%ADn&email=ana%40example.com&telefono=612345678
```

Con **POST**, los datos viajan **en el cuerpo de la petición HTTP**, y no aparecen en la URL.

| | GET | POST |
|---|---|---|
| Dónde viajan los datos | En la URL | En el cuerpo de la petición |
| Visibles en la barra de direcciones, historial y marcadores | Sí | No |
| Quedan en los registros (*logs*) de servidores y proxies | Sí, porque se registran las URL | Normalmente no |
| Se pueden enlazar o guardar como marcador | Sí | No |
| Límite de tamaño | Limitado por la longitud máxima de URL | Sin límite práctico. Permite subir archivos |
| Uso correcto | **Consultas que no cambian nada**: búsquedas, filtros, paginación | **Enviar o modificar datos**: registros, mensajes, datos personales, contraseñas |

**Criterio de seguridad.** En algunos materiales antiguos aparecen formularios de matrícula que envían datos personales con GET sobre HTTP sin cifrar. Es un patrón **incorrecto** y no debe reproducirse. Los datos personales se envían siempre con **POST** y siempre sobre **HTTPS**. HTTPS cifra la comunicación, pero no impide que la URL quede guardada en el historial y en los registros del servidor. Por eso, aun con HTTPS, los datos personales no van en la URL.

### 7.3 Etiquetas: `label`

Cada campo necesita una etiqueta visible asociada. La forma habitual es relacionar el atributo `for` del `label` con el `id` del campo:

```html
<label for="email">Correo electrónico</label>
<input type="email" id="email" name="email">
```

La asociación tiene dos efectos. Primero, un lector de pantalla anuncia «Correo electrónico» al entrar en el campo. Segundo, al hacer clic en el texto se activa el campo, lo que se agradece mucho en casillas pequeñas o en el móvil. También se puede envolver el campo dentro del `label`, sin `for`.

`id` y `name` suelen coincidir, pero cumplen funciones distintas. `id` identifica el elemento **en la página** (para el `label`, CSS y JavaScript) y debe ser único. `name` es el nombre con que el dato **llega al servidor**, y puede repetirse, como en un grupo de botones de opción.

### 7.4 Tipos de `input`

HTML5 amplió mucho los tipos de campo. Cada tipo aporta **validación automática** y, en el móvil, **el teclado adecuado**:

| `type` | Para qué | Qué aporta |
|---|---|---|
| `text` | Texto de una línea | — |
| `email` | Correo | Comprueba el formato `algo@dominio`. Teclado con @ en el móvil |
| `password` | Contraseña | Oculta lo que se escribe (no lo cifra) |
| `tel` | Teléfono | Teclado numérico. **No valida el formato**, porque cada país tiene el suyo: se combina con `pattern` |
| `url` | Dirección web | Comprueba que sea una URL absoluta |
| `number` | Número | Flechas de incremento. `min`, `max`, `step` |
| `range` | Valor aproximado en un intervalo | Control deslizante |
| `date`, `time`, `datetime-local`, `month`, `week` | Fechas y horas | Selector nativo. El valor se envía en formato estándar (`2026-10-28`) aunque se muestre en formato local |
| `color` | Color | Selector de color |
| `checkbox` | Casilla de verificación | Solo se envía si está marcada |
| `radio` | Una opción entre varias | Los botones del mismo grupo comparten `name` |
| `file` | Subir archivo | `accept` limita los tipos. Requiere `method="post"` y `enctype="multipart/form-data"` |
| `search` | Campo de búsqueda | Botón para borrar en algunos navegadores |
| `hidden` | Dato que se envía sin mostrarse | **No es secreto**: se ve en el código fuente |
| `submit`, `reset`, `button` | Botones | Mejor usar el elemento `button` |

### 7.5 Otros controles y atributos

- `select` con `option` (y `optgroup` para agruparlas): lista desplegable. `selected` marca la opción por defecto. Con `multiple` se pueden elegir varias.
- `textarea`: texto de varias líneas. El valor inicial va **entre las etiquetas**, no en un atributo `value`.
- `fieldset` y `legend`: agrupan campos relacionados con un título. Son imprescindibles en grupos de `radio` y `checkbox`, porque el lector de pantalla anuncia la pregunta (`legend`) antes de cada opción.
- `button`: su `type` por defecto dentro de un formulario es **`submit`**. Un botón pensado para otra cosa (abrir un menú con JavaScript, por ejemplo) debe llevar `type="button"`, o enviará el formulario.
- `datalist`: sugerencias para un campo de texto, que el usuario puede ignorar.

| Atributo | Efecto |
|---|---|
| `required` | Campo obligatorio |
| `minlength`, `maxlength` | Longitud mínima y máxima del texto |
| `min`, `max`, `step` | Límites para números y fechas |
| `pattern` | Expresión regular que debe cumplir el valor. Se acompaña de un `title` que explique el formato |
| `placeholder` | Texto de ejemplo dentro del campo. **No sustituye al `label`**: desaparece al escribir y suele tener poco contraste |
| `autocomplete` | Indica al navegador qué dato es (`name`, `email`, `tel`…) para autorrellenarlo |
| `value` | Valor inicial o valor que se envía |
| `checked` | Casilla u opción marcada por defecto |
| `disabled` | Campo inactivo: **no se envía** |
| `readonly` | Campo no editable, pero **sí se envía** |

### 7.6 Validación en el navegador: comodidad, no seguridad

Con los atributos anteriores, el navegador impide enviar el formulario si algún campo no cumple su restricción, y muestra un mensaje. Es una gran ventaja para el usuario, que se entera del error al momento y sin recargar la página.

Pero **no es una medida de seguridad**. Todo lo que ocurre en el navegador está bajo el control del usuario: basta con abrir DevTools, borrar el atributo `required` o añadir `novalidate` al formulario, y se puede enviar cualquier cosa. También se puede enviar una petición al servidor sin pasar por el formulario. Por tanto, **el servidor debe volver a validar siempre todos los datos**, como si la validación del cliente no existiera. La validación del navegador mejora la experiencia de uso. La seguridad depende del servidor.

### 7.7 Ejemplo completo: la página de contacto

Documento válido según el Nu Html Checker. Incluye también un formulario de búsqueda, que es el caso típico de GET:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Contacto · Portfolio de Ana Martín</title>
</head>
<body>
  <h1>Contacto</h1>
  <p>Los campos marcados con * son obligatorios.</p>

  <form action="/contacto" method="post">
    <fieldset>
      <legend>Tus datos</legend>

      <p>
        <label for="nombre">Nombre *</label>
        <input type="text" id="nombre" name="nombre" autocomplete="name" required maxlength="80">
      </p>
      <p>
        <label for="email">Correo electrónico *</label>
        <input type="email" id="email" name="email" autocomplete="email" required>
      </p>
      <p>
        <label for="telefono">Teléfono</label>
        <input type="tel" id="telefono" name="telefono" autocomplete="tel"
               pattern="[6-9][0-9]{8}" title="Nueve cifras, empezando por 6, 7, 8 o 9">
      </p>
    </fieldset>

    <fieldset>
      <legend>Tu mensaje</legend>

      <p>
        <label for="motivo">Motivo</label>
        <select id="motivo" name="motivo">
          <option value="practicas">Oferta de prácticas</option>
          <option value="proyecto">Propuesta de proyecto</option>
          <option value="otro" selected>Otro</option>
        </select>
      </p>
      <p>
        <label for="fecha">Fecha preferida para hablar</label>
        <input type="date" id="fecha" name="fecha" min="2026-10-01" max="2027-06-30">
      </p>
      <p>
        <label for="mensaje">Mensaje *</label>
        <textarea id="mensaje" name="mensaje" rows="6" cols="40" required minlength="20"></textarea>
      </p>
    </fieldset>

    <fieldset>
      <legend>¿Cómo prefieres que te responda?</legend>
      <p>
        <input type="radio" id="resp-email" name="respuesta" value="email" checked>
        <label for="resp-email">Por correo</label>
        <input type="radio" id="resp-tel" name="respuesta" value="telefono">
        <label for="resp-tel">Por teléfono</label>
      </p>
    </fieldset>

    <p>
      <input type="checkbox" id="privacidad" name="privacidad" value="acepto" required>
      <label for="privacidad">He leído la <a href="privacidad.html">política de privacidad</a> y acepto que se usen mis datos solo para responder a este mensaje *</label>
    </p>

    <p>
      <button type="submit">Enviar mensaje</button>
      <button type="reset">Borrar el formulario</button>
    </p>
  </form>

  <h2>Buscar en el portfolio</h2>
  <search>
    <form action="/buscar" method="get">
      <label for="q">Término de búsqueda</label>
      <input type="search" id="q" name="q">
      <button type="submit">Buscar</button>
    </form>
  </search>
</body>
</html>
```

Detalles a observar:

- Los datos personales van por **POST**. La búsqueda va por **GET**, porque no modifica nada y conviene que la URL de resultados se pueda compartir.
- El `pattern` del teléfono admite nueve cifras empezando por 6, 7, 8 o 9 (móviles y fijos en España), y el `title` explica el formato al usuario.
- La casilla de privacidad es obligatoria y el texto dice para qué se usarán los datos. La normativa de protección de datos exige informar de la finalidad y **no recoger más datos de los necesarios**. Por eso el teléfono no es obligatorio.
- El formulario de búsqueda va dentro de `search`, el elemento de región de búsqueda.
- Un sitio estático, como el portfolio publicado en GitHub Pages, **no tiene programa en el servidor** que reciba el formulario. En el portfolio el formulario se construye igual, sabiendo que enviarlo no tendrá efecto hasta que haya un servidor detrás.

> **ALTERNATIVAS Y CRITERIO**
>
> Para dar un medio de contacto hay tres opciones: un enlace `mailto:` (sencillo y sin servidor, pero depende de que el usuario tenga un cliente de correo configurado, y deja la dirección visible para programas que recopilan correos), un formulario propio (control total, pero necesita un programa en el servidor) o un servicio externo de formularios (cómodo, pero los datos personales pasan por un tercero). Para el portfolio se construye el formulario porque es el objetivo didáctico. En un proyecto real, la elección depende de si hay servidor propio y de dónde es aceptable que se traten los datos.

> **ERRORES FRECUENTES**
>
> - **Campos sin `name`**, que no llegan al servidor aunque se rellenen.
> - **Usar `placeholder` en lugar de `label`.**
> - **Datos personales por GET.**
> - **Confiar en `required` o `pattern` como seguridad.**
> - **Un `button` sin `type` dentro de un formulario**, que lo envía sin querer.
> - **Grupos de `radio` con `name` distinto**, que dejan marcar varias opciones a la vez.

---

## 8. Versiones de HTML en la práctica

### 8.1 Qué cambió entre HTML 4 / XHTML y el HTML actual

En el bloque 1 viste la historia. Este bloque se centra en las diferencias concretas que te encontrarás al leer o actualizar páginas antiguas, que siguen siendo muchas en Internet:

| Aspecto | HTML 4.01 / XHTML 1.0 | HTML actual |
|---|---|---|
| DOCTYPE | Largo, con referencia a una DTD | `<!DOCTYPE html>` |
| Codificación | `<meta http-equiv="Content-Type" content="text/html; charset=utf-8">` | `<meta charset="utf-8">` |
| Presentación | Elementos y atributos visuales: `font`, `center`, `bgcolor`, `align`, `border`, `width` en celdas | Solo CSS. Esos elementos y atributos son **obsoletos** |
| Maquetación | Con tablas | Con CSS (flexbox y grid, UD3) sobre elementos semánticos |
| Estructura | Solo `div` con clases | `header`, `nav`, `main`, `article`, `section`, `aside`, `footer` |
| Multimedia | Complementos externos (Flash) | `audio` y `video` nativos |
| Formularios | Solo `text`, `password`, `checkbox`, `radio`, `file`, `hidden` | Tipos `email`, `tel`, `url`, `number`, `date`… y validación nativa (`required`, `pattern`) |
| Scripts | `<script type="text/javascript">` con el código envuelto en comentarios `<!-- -->` | `<script>`. El truco del comentario era para navegadores de los 90 y ya no sirve para nada |
| Elementos vacíos | Obligatorio `<br />` en XHTML | `<br>` (se acepta `<br />`) |
| Mayúsculas | Admitidas en HTML 4, prohibidas en XHTML | Admitidas, pero por convención se escribe todo en minúsculas |

**Elementos obsoletos** que conviene reconocer: `font`, `center`, `big`, `strike`, `tt`, `acronym` (sustituido por `abbr`), `frame`, `frameset` y `noframes` (páginas divididas en marcos), `applet` (applets de Java), `marquee` (texto en movimiento) y `blink` (texto parpadeante, que nunca fue estándar). **Elementos redefinidos** que siguen siendo válidos con otro significado: `b`, `i`, `s`, `small`, `hr` (bloque 3).

### 8.2 Actualizar una página antigua con el validador, paso a paso

Este es el artefacto de la sesión 8: la página de una academia ficticia tal como se escribía en 2005, en HTML 4.01 Transitional, con maquetación en tablas y formato con `font`.

**Paso 1: el documento original.**

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<HTML>
<HEAD>
<META http-equiv="Content-Type" content="text/html; charset=utf-8">
<TITLE>Academia Cinco Villas</TITLE>
<SCRIPT type="text/javascript" language="javascript">
<!--
function saludo() { alert("Bienvenido"); }
//-->
</SCRIPT>
</HEAD>
<BODY bgcolor="#FFFFCC" onload="saludo()">
<TABLE width="800" border="0" cellpadding="5" align="center">
<TR>
<TD colspan="2" bgcolor="#003366"><CENTER><FONT face="Verdana" size="6" color="white">Academia Cinco Villas</FONT></CENTER></TD>
</TR>
<TR>
<TD width="150" valign="top" bgcolor="#CCCCCC">
<A href="index.html">Inicio</A><BR>
<A href="cursos.html">Cursos</A><BR>
<A href="contacto.html">Contacto</A>
</TD>
<TD valign="top">
<FONT face="Verdana" size="4"><B>Nuestros cursos</B></FONT><BR><BR>
<IMG src="aula.jpg" width="300" height="200" border="0">
<P>Ofrecemos cursos de <I>ofimática</I> e <I>internet</I>.
<P><STRIKE>Matrícula abierta</STRIKE> <FONT color="red"><BLINK>¡Plazas agotadas!</BLINK></FONT>
</TD>
</TR>
<TR>
<TD colspan="2" align="center"><FONT size="2">&copy; 2005 Academia Cinco Villas</FONT></TD>
</TR>
</TABLE>
</BODY>
</HTML>
```

Salida del Nu Html Checker: **10 errores y 15 avisos**.

```text
1.1-1.102: error: Almost standards mode doctype. Expected “<!DOCTYPE html>”.
6.1-6.53: info warning: The “language” attribute on the “script” element is obsolete. Use the “type” attribute instead.
6.1-6.53: info warning: The “type” attribute is unnecessary for JavaScript resources.
12.1-12.42: info warning: The “bgcolor” attribute on the “body” element is obsolete. Use CSS instead.
13.1-13.61: info warning: The “width” attribute on the “table” element is obsolete. Use CSS instead.
13.1-13.61: info warning: The “border” attribute on the “table” element is obsolete. Consider specifying “img { border: 0; }” in CSS instead.
13.1-13.61: info warning: The “cellpadding” attribute on the “table” element is obsolete. Use CSS instead.
13.1-13.61: info warning: The “align” attribute on the “table” element is obsolete. Use CSS instead.
14.5-15.34: info warning: The “bgcolor” attribute on the “td” element is obsolete. Use CSS instead.
15.35-15.42: error: The “center” element is obsolete. Use CSS instead.
15.43-15.86: error: The “font” element is obsolete. Use CSS instead.
17.5-18.47: info warning: The “width” attribute on the “td” element is obsolete. Use CSS instead.
17.5-18.47: info warning: The “valign” attribute on the “td” element is obsolete. Use CSS instead.
17.5-18.47: info warning: The “bgcolor” attribute on the “td” element is obsolete. Use CSS instead.
22.6-23.17: info warning: The “valign” attribute on the “td” element is obsolete. Use CSS instead.
24.1-24.30: error: The “font” element is obsolete. Use CSS instead.
25.1-25.56: info warning: The “border” attribute on the “img” element is obsolete. Consider specifying “img { border: 0; }” in CSS instead.
25.1-25.56: error: An “img” element must have an “alt” attribute, except under certain conditions. For details, consult guidance on providing text alternatives for images.
27.4-27.11: error: The “strike” element is obsolete. Use “del” or “s” element instead.
27.39-27.56: error: The “font” element is obsolete. Use CSS instead.
27.57-27.63: error: The “blink” element is a completely-unknown element that is not allowed anywhere in any HTML content.
27.57-27.63: error: The “blink” element is obsolete. Use CSS instead.
30.5-31.31: info warning: The “align” attribute on the “td” element is obsolete. Use CSS instead.
31.32-31.46: error: The “font” element is obsolete. Use CSS instead.
1.103-2.6: info warning: Consider adding a “lang” attribute to the “html” start tag to declare the language of this document.
```

Cómo se lee esta salida:

- **Errores** (*error*): lo que el estándar actual no permite. El DOCTYPE antiguo, los elementos obsoletos (`center`, `font`, `strike`), una imagen sin `alt` y `blink`, que el validador marca dos veces: como elemento desconocido y como obsoleto.
- **Avisos** (*info warning*): atributos presentacionales obsoletos, el atributo `language` y el `type` innecesario del `script`, y la falta de `lang` en `html`.
- **Lo que el validador no dice**: nada sobre la **maquetación con tablas**, porque cada tabla, por sí sola, es válida. Tampoco protesta por las **mayúsculas** en las etiquetas, que HTML admite. Ni por el comentario `<!-- -->` dentro del `script`: no es un error, solo algo inútil.

**Paso 2: modernizar la cabecera.** Se cambia el DOCTYPE, se añade `lang="es"`, se simplifica la declaración de codificación y se limpia el `script`. El `body` no se toca:

```html
<!DOCTYPE html>
<HTML lang="es">
<HEAD>
<META charset="utf-8">
<TITLE>Academia Cinco Villas</TITLE>
<SCRIPT>
function saludo() { alert("Bienvenido"); }
</SCRIPT>
</HEAD>
```

Salida: **9 errores y 12 avisos**. Desaparecen los problemas de la cabecera, pero los del cuerpo siguen todos, porque son el problema de fondo: **la presentación está mezclada con el contenido**. Ahí no hay arreglo rápido: hay que reescribir.

**Paso 3: reescritura con HTML actual.** Estructura semántica en lugar de tablas, sin ningún atributo de presentación (los colores y la tipografía irían en `estilos.css`, UD3), `alt` en la imagen y `del` en lugar de `strike`. El saludo con `alert` al cargar la página se elimina: era molesto para el usuario, y si hiciera falta algún comportamiento iría en un archivo de JavaScript aparte (UD4).

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Academia Cinco Villas</title>
  <link rel="stylesheet" href="estilos.css">
</head>
<body>
  <header>
    <h1>Academia Cinco Villas</h1>
    <nav aria-label="Principal">
      <ul>
        <li><a href="index.html">Inicio</a></li>
        <li><a href="cursos.html">Cursos</a></li>
        <li><a href="contacto.html">Contacto</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <h2>Nuestros cursos</h2>
    <img src="aula.jpg" alt="Aula de la academia con diez ordenadores" width="300" height="200">
    <p>Ofrecemos cursos de ofimática e internet.</p>
    <p><del>Matrícula abierta</del> <strong class="aviso">¡Plazas agotadas!</strong></p>
  </main>

  <footer>
    <p><small>© 2005-2026 Academia Cinco Villas</small></p>
  </footer>
</body>
</html>
```

Salida del Nu Html Checker: **sin errores ni avisos**.

### 8.3 Modo estándar y *quirks mode*

El DOCTYPE no es decorativo. Los navegadores tienen tres modos de procesar una página:

- **Modo estándar** (*no-quirks*): el que se activa con `<!DOCTYPE html>`.
- **Modo casi estándar** (*limited-quirks*): el que activa el DOCTYPE de HTML 4.01 Transitional. Por eso el validador lo llama *Almost standards mode doctype*.
- **Modo de compatibilidad** (*quirks mode*): sin DOCTYPE, o con DOCTYPE muy antiguos. El navegador imita errores de los navegadores de finales de los 90, y el mismo CSS da resultados distintos.

Puedes comprobar en qué modo está cualquier página escribiendo en la consola de DevTools:

```javascript
document.compatMode
```

Devuelve `"CSS1Compat"` en modo estándar (y en el casi estándar) y `"BackCompat"` en *quirks mode*.

> **ALTERNATIVAS Y CRITERIO**
>
> Ante una página antigua caben dos estrategias: **parchearla** (paso 2: arreglar lo mínimo para que el validador dé menos errores) o **reescribirla** (paso 3). Parchear es rápido y tiene sentido si la página va a retirarse pronto o si es enorme y funciona. Reescribir es la opción correcta si la página va a seguir mantenida, porque los problemas de fondo (presentación mezclada con contenido, maquetación con tablas, falta de semántica) no se arreglan con retoques.

> **ERRORES FRECUENTES**
>
> - **Dar por buena una página porque el validador no da errores.** El validador no detecta la maquetación con tablas, la sopa de divs, un `alt` sin sentido ni una jerarquía de encabezados absurda. Validar es necesario, pero no suficiente.
> - **Copiar código de tutoriales antiguos sin revisarlo.** DOCTYPE largos, `<script type="text/javascript">`, `<center>`, `<font>` o `border="0"` en imágenes son señales de que el material es de otra época.

---

## Glosario

- **Atributo:** información adicional de un elemento, escrita en su etiqueta de apertura con la forma `nombre="valor"`.
- **Atributo booleano:** atributo cuyo valor es verdadero por el mero hecho de estar presente (`required`, `disabled`, `checked`).
- **Atributo global:** atributo que se puede usar en cualquier elemento (`id`, `class`, `lang`, `title`, `hidden`, `data-*`).
- **Árbol de accesibilidad:** representación del documento que el navegador entrega a las tecnologías de apoyo, con el rol y el nombre de cada elemento.
- **Categoría de contenido:** clasificación de los elementos HTML (flujo, fraseo, sección, encabezado, incrustado, interactivo, metadatos) que determina dónde puede ir cada uno.
- **DevTools:** herramientas de desarrollo integradas en el navegador (inspector, consola, red, accesibilidad).
- **DOCTYPE:** declaración inicial del documento. En HTML actual, `<!DOCTYPE html>`, que activa el modo estándar.
- **DOM (*Document Object Model*):** árbol de objetos que el navegador construye en memoria a partir del documento.
- **Elemento:** unidad del documento formada por etiqueta de apertura, contenido y etiqueta de cierre.
- **Elemento obsoleto:** elemento que el estándar actual no permite en documentos nuevos (`font`, `center`, `frame`…).
- **Elemento vacío (*void element*):** elemento que no puede tener contenido ni etiqueta de cierre (`br`, `img`, `input`, `meta`…).
- **Emmet:** sistema de abreviaturas que genera código HTML y CSS, integrado en VS Code.
- **Entidad:** secuencia que representa un carácter especial (`&lt;`, `&amp;`, `&nbsp;`).
- **Etiqueta:** marca que delimita un elemento: `<p>` (apertura) o `</p>` (cierre).
- **GET / POST:** métodos de envío de un formulario: en la URL o en el cuerpo de la petición HTTP.
- **HTML Living Standard:** especificación única y sin números de versión de HTML, mantenida por el WHATWG desde el acuerdo con el W3C de 2019.
- **HTTPS:** HTTP sobre una conexión cifrada.
- **Landmark (región):** zona de la página con un rol de accesibilidad (`banner`, `navigation`, `main`…) que sirve de atajo en los lectores de pantalla.
- **Lector de pantalla:** programa que lee en voz alta o en braille el contenido de la pantalla.
- **Metadatos:** información sobre el documento (codificación, título, descripción), contenida en `head`.
- **Nu Html Checker:** validador oficial de HTML del W3C (validator.w3.org/nu).
- **Quirks mode:** modo de compatibilidad con páginas antiguas que activa un navegador cuando falta el DOCTYPE o es muy antiguo.
- **Ruta relativa:** ruta que se interpreta a partir de la ubicación del documento actual (`img/foto.jpg`, `../index.html`).
- **Semántica:** significado de un elemento, independiente de su apariencia.
- **Texto alternativo (`alt`):** texto que sustituye a una imagen para quien no puede verla.
- **URL absoluta:** dirección completa, con protocolo y dominio.
- **Validador:** programa que comprueba que un documento cumple las reglas de un estándar.
- **Viewport:** zona visible de la página en el dispositivo.
- **W3C:** consorcio que desarrolla estándares web (CSS, SVG, XML, accesibilidad).
- **WAI-ARIA:** especificación del W3C que añade roles y propiedades de accesibilidad a HTML.
- **WHATWG:** grupo de fabricantes de navegadores que mantiene el estándar de HTML y del DOM.
- **XHTML:** HTML escrito con las reglas sintácticas de XML.

---

## Referencias y recursos

- **HTML Living Standard** (especificación oficial): html.spec.whatwg.org. Es la fuente de referencia en caso de duda, aunque no está pensada para aprender.
- **MDN Web Docs, sección HTML** (documentación de Mozilla, con versión en español): developer.mozilla.org. Incluye referencia de cada elemento y atributo, con ejemplos y tablas de compatibilidad.
- **Nu Html Checker** (validador oficial del W3C): validator.w3.org/nu.
- **Can I use** (compatibilidad de cada característica en cada navegador): caniuse.com.
- **W3C Web Accessibility Initiative (WAI)**: w3.org/WAI. Tutoriales de accesibilidad y las pautas WCAG.
- **Documentación de Emmet en VS Code**: code.visualstudio.com/docs/languages/emmet.
- **Documentación de GitHub Pages**: docs.github.com/pages.
- **Lenguaje HTML** (web de Manz, en español): lenguajehtml.com. Tienes su chuleta de HTML en la carpeta del módulo.
