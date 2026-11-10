# UD3 — CSS y sindicación de contenidos

**0373 · Lenguajes de Marcas y SGI** · Apuntes de la unidad · RA2 (f-j) · 1º DAW · Curso 2026-27

---

## Introducción

En la unidad anterior construiste el contenido y la estructura de una web: qué es cada cosa y cómo se organiza. Esta unidad tiene dos mitades distintas. En la primera aprendes **CSS**, el lenguaje que decide cómo se ve ese contenido: tipografía, colores, espacios, disposición en pantalla y adaptación al móvil. En la segunda aprendes **sindicación de contenidos**: cómo un sitio publica la lista de sus novedades en un formato XML estándar para que otros programas la lean, la combinen y avisen a quien se ha suscrito. Entre las dos hay una pieza que las une: la **validación**, que comprueba que el HTML, el CSS y los propios canales cumplen la especificación.

Estos apuntes están escritos para que puedas estudiar la unidad por tu cuenta, sin necesidad de haber asistido a clase. Cada apartado define los términos que usa, explica el porqué de las decisiones y trae ejemplos completos que puedes copiar, abrir en el navegador y comprobar. También sirven para consultar mientras haces el trabajo de la unidad, que es dar estilo a tu portfolio: **cada apartado contiene todo lo necesario para hacer esa parte del trabajo**, así que puedes avanzar al ritmo que necesites sin quedarte sin explicación.

La unidad se organiza en seis apartados:

- **Apartado 1:** Hojas de estilo: para qué sirven, cómo se escriben y cómo se aplican.
- **Apartado 2:** La cascada, la especificidad y el modelo de caja.
- **Apartado 3:** Disposición de la página y adaptación al móvil.
- **Apartado 4:** Validar HTML y CSS.
- **Apartado 5:** Sindicación de contenidos: RSS, Atom y JSON Feed.
- **Apartado 6:** Auditar un documento web y preparar la entrega del portfolio.

### Qué necesitas saber antes de empezar

- De la **UD1**: qué es un documento XML bien formado (un único elemento raíz, etiquetas cerradas y bien anidadas, atributos entre comillas) y qué es un espacio de nombres (*namespace*). Los necesitarás en el apartado 5, porque los canales RSS y Atom son documentos XML.
- De la **UD2**: la estructura de un documento HTML (`head`, `body`, `header`, `nav`, `main`, `footer`), las rutas relativas, el atributo `aria-current="page"`, el elemento `meta` con `viewport`, las tablas de datos y el uso del validador Nu Html Checker.
- Un editor de código (VS Code) y un navegador con herramientas de desarrollo (Chrome, Edge o Firefox).

### El portfolio sobre el que se trabaja

Todos los ejemplos de la unidad parten del mismo sitio: el portfolio de una persona ficticia, Ana Martín. Tiene tres páginas (inicio, sobre mí y contacto) con la estructura que aprendiste en la UD2. Tu portfolio será distinto del de Ana, pero tiene los mismos elementos, así que todo lo que se explica se aplica igual a lo tuyo: si un selector se escribe para el `nav` de Ana, funciona para el `nav` del tuyo.

La carpeta de partida tiene este aspecto. La hoja de estilo `css/estilos.css` todavía no existe: la crearás en el apartado 1.

```plaintext
portfolio/
├── index.html
├── sobre-mi.html
├── contacto.html
├── css/
│   └── estilos.css      ← la crearás en el apartado 1
└── img/
    └── avatar.svg       ← o la imagen que uses en tu portfolio
```

Y este es el contenido de las tres páginas. Fíjate en tres detalles que se usarán en la unidad: cada página enlaza `css/estilos.css` en el `head` (apartado 1.5), el menú marca la página actual con `aria-current="page"` (apartado 1.8) y el párrafo de presentación lleva `class="presentacion"` (apartado 1.8).

`index.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Inicio · Portfolio de Ana Martín</title>
  <link rel="stylesheet" href="css/estilos.css">
</head>
<body>
  <header>
    <p>Ana Martín · Desarrolladora web</p>
    <nav aria-label="Principal">
      <ul>
        <li><a href="index.html" aria-current="page">Inicio</a></li>
        <li><a href="sobre-mi.html">Sobre mí</a></li>
        <li><a href="contacto.html">Contacto</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <h1>Ana Martín</h1>
    <img src="img/avatar.svg" alt="Avatar de Ana: silueta azul sobre fondo oscuro" width="160" height="160">
    <p class="presentacion">Estudiante de Desarrollo de Aplicaciones Web. Aquí reúno lo que voy aprendiendo.</p>
    <p><a href="sobre-mi.html">Más sobre mí</a></p>
  </main>

  <footer>
    <p><small>© 2026 Ana Martín</small> · <a href="mailto:ana.martin@example.com">ana.martin@example.com</a></p>
  </footer>
</body>
</html>
```

`sobre-mi.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Sobre mí · Portfolio de Ana Martín</title>
  <link rel="stylesheet" href="css/estilos.css">
</head>
<body>
  <header>
    <p>Ana Martín · Desarrolladora web</p>
    <nav aria-label="Principal">
      <ul>
        <li><a href="index.html">Inicio</a></li>
        <li><a href="sobre-mi.html" aria-current="page">Sobre mí</a></li>
        <li><a href="contacto.html">Contacto</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <h1>Sobre mí</h1>
    <p class="presentacion">Estudio <abbr title="Desarrollo de Aplicaciones Web">DAW</abbr> y me gusta entender cómo funciona lo que construyo. Antes cursé el ciclo de <abbr title="Sistemas Microinformáticos y Redes">SMR</abbr>, así que las redes no me asustan.</p>

    <section>
      <h2>Qué sé hacer</h2>
      <ul>
        <li>Maquetar páginas con <strong>HTML semántico</strong>.</li>
        <li>Programar en Python: scripts de automatización y lectura de ficheros <abbr title="Comma-Separated Values">CSV</abbr>.</li>
        <li>Administrar equipos con Windows y Linux.</li>
      </ul>
    </section>

    <section>
      <h2>Mi forma de trabajar</h2>
      <ol>
        <li>Entender el problema antes de escribir código.</li>
        <li>Hacer una primera versión que funcione.</li>
        <li>Revisarla con el validador y mejorarla.</li>
      </ol>
    </section>
  </main>

  <footer>
    <p><small>© 2026 Ana Martín</small> · <a href="mailto:ana.martin@example.com">ana.martin@example.com</a></p>
  </footer>
</body>
</html>
```

`contacto.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Contacto · Portfolio de Ana Martín</title>
  <link rel="stylesheet" href="css/estilos.css">
</head>
<body>
  <header>
    <p>Ana Martín · Desarrolladora web</p>
    <nav aria-label="Principal">
      <ul>
        <li><a href="index.html">Inicio</a></li>
        <li><a href="sobre-mi.html">Sobre mí</a></li>
        <li><a href="contacto.html" aria-current="page">Contacto</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <h1>Contacto</h1>
    <p>Puedes escribirme por correo: <a href="mailto:ana.martin@example.com">ana.martin@example.com</a>.</p>
    <p><a href="index.html">Volver al inicio</a></p>
  </main>

  <footer>
    <p><small>© 2026 Ana Martín</small> · <a href="mailto:ana.martin@example.com">ana.martin@example.com</a></p>
  </footer>
</body>
</html>
```

### Convenciones

Los mensajes que aparecen en pantalla (por ejemplo, los del validador) van en un recuadro marcado como «salida». Los fragmentos de código van con su lenguaje indicado (`html`, `css`, `xml`, `json`). Los recuadros con título destacan alternativas, errores frecuentes, notas técnicas y preguntas para que compruebes lo que has entendido.

## Índice

1. Hojas de estilo: para qué sirven, cómo se escriben y cómo se aplican *(2.f, 2.g)*
2. La cascada, la especificidad y el modelo de caja *(2.g)*
3. Disposición de la página y adaptación al móvil *(2.g)*
4. Validar HTML y CSS *(2.h)*
5. Sindicación de contenidos: RSS, Atom y JSON Feed *(2.i, 2.j)*
6. Auditar un documento web y preparar la entrega del portfolio *(2.g, 2.h)*
- Glosario
- Referencias y recursos

---

## 1. Hojas de estilo: para qué sirven, cómo se escriben y cómo se aplican

`// Criterios de evaluación: 2.f, 2.g`

### 1.1 HTML dice qué es; CSS dice cómo se ve

En la UD2 escribiste páginas cuyo HTML describe **qué es** cada cosa: esto es un encabezado, esto es un párrafo, esto es un enlace, esto es la navegación. No dice nada de cómo debe verse. Aun así, cuando abres cualquiera de esas páginas, el título aparece grande y en negrita, los enlaces en azul y subrayados, y las listas con viñetas. ¿Quién ha decidido eso?

Lo ha decidido el propio navegador. Todos los navegadores incluyen una **hoja de estilo por defecto** (en inglés, *user agent stylesheet*) con reglas como «un `h1` es grande y en negrita», «un `a` es azul y subrayado» o «un `ul` lleva viñetas y sangría». Por eso una página sin ninguna línea de CSS no se muestra «sin formato»: se muestra con el formato del navegador. Esto tiene una consecuencia práctica importante: **tu hoja de estilo no empieza desde cero**. Lo que hace es contradecir, en los puntos que tú decidas, lo que dice la hoja del navegador.

**CSS** (*Cascading Style Sheets*, «hojas de estilo en cascada») es el lenguaje que describe la presentación de un documento: colores, tipografías, tamaños, espacios, bordes, disposición en la pantalla y su adaptación a distintos dispositivos. Lo de «en cascada» se refiere al mecanismo que decide qué hacer cuando dos reglas dan órdenes distintas sobre el mismo elemento. Lo estudiarás en el apartado 2; de momento basta con saber que existe.

Como viste en la UD2, CSS **no es un lenguaje de marcas**: no marca contenido, sino que aplica reglas de aspecto a elementos que ya existen. Vive junto al HTML, pero es otro lenguaje con otra sintaxis.

### 1.2 Ventajas de separar presentación y contenido

La alternativa a CSS sería escribir el aspecto dentro del propio HTML, como se hacía a finales de los noventa con elementos como `font` o `center` (que verás marcados como obsoletos en el validador). Separar el contenido (HTML) de la presentación (CSS) aporta cinco ventajas. Conviene saberlas explicar con un ejemplo, porque son el contenido del criterio de evaluación 2.f.

1. **Mantenimiento.** Si el color de los títulos está escrito una sola vez, en un único archivo, cambiarlo es cambiar una línea y las tres páginas se actualizan. Si estuviera escrito dentro de cada `h1` de cada página, habría que tocarlos todos, y siempre se olvida alguno.
2. **Coherencia.** Todas las páginas comparten la misma hoja de estilo, así que todas se ven igual sin esfuerzo adicional. La coherencia deja de depender de que recuerdes copiar los mismos valores.
3. **Rendimiento.** El navegador descarga el archivo CSS una vez y lo reutiliza en el resto de páginas del sitio. Si el aspecto estuviera repetido dentro de cada HTML, se descargaría una y otra vez junto con el contenido.
4. **Adaptación.** Con el mismo HTML se puede dar un aspecto distinto según el tamaño de pantalla, el modo claro u oscuro del sistema o la impresión. Esto es lo que harás en los apartados 3 y 1.12.
5. **HTML más limpio y más accesible.** Un HTML que solo describe qué es cada cosa lo entienden mejor los lectores de pantalla, los buscadores y cualquier otro programa que lo procese. Es la misma idea de «separar contenido y forma» que viste en la UD1 como ventaja general de los lenguajes de marcas.

Un ejemplo concreto de tu portfolio para la ventaja 1: el color de los títulos `h1` y `h2` de las tres páginas se cambia en una sola regla de `estilos.css`. Para la ventaja 4: la misma página puede mostrar el menú en columna en un móvil y en fila en un monitor, sin escribir dos HTML distintos.

### 1.3 Tres formas de aplicar CSS

Hay tres maneras de conectar una regla CSS con un documento HTML. El siguiente documento las usa las tres a la vez para que veas la diferencia; cada forma colorea un párrafo distinto:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Tres formas de aplicar CSS</title>
  <!-- Forma 3: externa. El navegador pide tres_formas.css y aplica sus reglas -->
  <link rel="stylesheet" href="tres_formas.css">
  <!-- Forma 2: interna. Las reglas van dentro del propio documento, en un elemento style -->
  <style>
    .interna {
      color: #b91c1c;
    }
  </style>
</head>
<body>
  <h1>Tres formas de aplicar CSS</h1>
  <!-- Forma 1: en línea. La declaración va dentro del propio elemento, en su atributo style -->
  <p style="color: #15803d;">Forma 1, en línea: el estilo está dentro de este párrafo.</p>
  <p class="interna">Forma 2, interna: el estilo está en el head de este documento.</p>
  <p class="externa">Forma 3, externa: el estilo está en otro archivo.</p>
</body>
</html>
```

Con su hoja externa, `tres_formas.css`:

```css
/* Forma 3: hoja de estilo externa. Un archivo .css aparte, enlazado con link en el head */
.externa {
  color: #1d4ed8;
}
```

- **En línea** (*inline*): la declaración va dentro del propio elemento, en su atributo `style`. En el ejemplo, el primer párrafo. Solo afecta a ese elemento y no se puede reutilizar: si hay diez párrafos así, hay que repetirla diez veces.
- **Interna**: las reglas van en un elemento `style` dentro del `head` de la propia página. En el ejemplo, la clase `interna`. Sirve para esa página y para ninguna otra.
- **Externa**: las reglas están en un archivo con extensión `.css`, que cada página enlaza desde su `head` con un elemento `link`. En el ejemplo, la clase `externa`. Un mismo archivo da estilo a todas las páginas que lo enlazan.

Si abres el documento en el navegador, verás tres párrafos de tres colores: verde (en línea), rojo (interna) y azul (externa). Para comprobar que la hoja externa es realmente independiente, cambia el color de `.externa` en `tres_formas.css`, guarda y recarga: cambia solo el tercer párrafo. En cambio, para cambiar el color del primer párrafo tienes que entrar en el HTML.

> **ALTERNATIVAS Y CRITERIO**
>
> Las tres formas son válidas y producen el mismo efecto sobre el elemento. Se elige la **externa** para un sitio de varias páginas porque es la única que cumple las cinco ventajas del apartado 1.2: un solo lugar para cambiar el aspecto, coherencia entre páginas, descarga una vez y HTML limpio. La interna tiene sentido en una página aislada o de prueba. La **en línea** no está «prohibida»: es HTML válido y a veces se usa, por ejemplo en correos electrónicos, donde muchos clientes de correo ignoran las hojas externas. Lo que no conviene es usarla como forma general de dar estilo a un sitio, porque cada declaración queda atada a un único elemento y, como verás en el apartado 2, además es muy difícil de contradecir desde la hoja de estilo.

### 1.4 Anatomía de una regla CSS

Una hoja de estilo es una lista de **reglas**. Cada regla tiene esta forma:

```plaintext
selector { propiedad: valor; propiedad: valor; }
   │         │          │
   │         │          └─ cómo lo cambia
   │         └─ qué cambia
   └─ a qué elementos se aplica
```

- El **selector** dice a qué elementos de la página se aplica la regla.
- Entre llaves va el **bloque de declaraciones**.
- Cada **declaración** es una **propiedad** (lo que quieres cambiar, como `color` o `font-size`), dos puntos, un **valor** (cómo quieres cambiarlo) y un punto y coma final.

Por ejemplo, esta regla pone en azul oscuro el texto de todos los `h1`:

```css
h1 {
  color: #1e3a8a;
}
```

Los dos puntos y el punto y coma importan. **El navegador no te avisa si te olvidas de uno**: simplemente ignora esa declaración, y a veces la siguiente. El validador de CSS sí lo detecta (apartado 4).

Dentro de una hoja de estilo puedes escribir **comentarios** entre `/*` y `*/`, que el navegador ignora. Los comentarios de los ejemplos de esta unidad explican cada declaración; úsalos en tu hoja de estilo para recordar por qué has tomado cada decisión.

### 1.5 Enlazar la hoja externa desde cada página

Una hoja externa no hace nada hasta que una página la enlaza. Para ello se añade esta línea en el `head` de **cada una de las tres páginas**:

```html
<link rel="stylesheet" href="css/estilos.css">
```

- `link` es un elemento vacío (no tiene contenido ni etiqueta de cierre) que relaciona el documento actual con otro archivo.
- `rel` (de *relationship*) dice qué relación tiene ese archivo con la página. El valor `stylesheet` significa «es la hoja de estilo de esta página».
- `href` es la ruta al archivo. Es una **ruta relativa**, igual que las de los enlaces y las imágenes de la UD2: las páginas están en la raíz del portfolio y la hoja en la carpeta `css`, así que la ruta es `css/estilos.css`, **sin barra inicial**. Una barra inicial (`/css/estilos.css`) significaría «desde la raíz del sitio web», no «desde la carpeta de esta página», y fallaría al abrir los archivos desde tu disco.

> **ERRORES FRECUENTES**
>
> **Enlazar la hoja solo en `index.html`.** Cada página es un documento independiente: si `sobre-mi.html` y `contacto.html` no tienen su propio `link`, se verán sin estilo. Es el error más habitual.
>
> **Error en la ruta del `href`.** Si la carpeta se llama `css` y escribes `CSS/estilos.css`, o `estilo.css` en lugar de `estilos.css`, el navegador no encuentra el archivo y no avisa en pantalla: simplemente no aplica nada. El diagnóstico es abrir las herramientas de desarrollo (F12), la pestaña **Red**, recargar la página y comprobar si `estilos.css` aparece cargado o con error. El validador HTML no comprueba que el archivo exista (apartado 4).

### 1.6 Valores: colores, tamaños y tipografía

Antes de escribir la primera hoja de estilo necesitas cuatro tipos de valores, para que ningún símbolo sea un misterio.

**Colores.** Se pueden escribir con un nombre (`red`, `navy`; el estándar define unos ciento cuarenta) o, de forma mucho más habitual, en **hexadecimal**: una almohadilla seguida de seis cifras, de dos en dos, que indican la cantidad de rojo, de verde y de azul de `00` (nada) a `ff` (el máximo). Así, `#1e3a8a` es poco rojo (`1e`), algo de verde (`3a`) y bastante azul (`8a`): un azul oscuro. `#000000` es negro, `#ffffff` es blanco. Cuando las cifras de cada pareja se repiten se puede abreviar a tres (`#fff`). Las propiedades de color más usadas son `color` (el del texto) y `background-color` (el del fondo).

**Tamaños.** La unidad `px` es el píxel CSS, una medida fija. La unidad `rem` significa «tantas veces el tamaño de letra base de la página»: el navegador suele traer 16 px de base, así que `1.125rem` son 18 px. La ventaja de `rem` es que respeta las preferencias de la persona: quien ha aumentado el tamaño de letra de su navegador porque ve mal obtiene una página proporcionalmente más grande, cosa que no ocurre si todo está escrito en píxeles. Por eso en esta unidad los tamaños de letra y los espacios se escriben en `rem`.

**Altura de línea.** La propiedad `line-height` es la altura de cada línea de texto. Escrita **sin unidad**, `1.6` significa «1,6 veces el tamaño de la letra». Da aire entre líneas y mejora la lectura. En el texto corrido, entre 1,5 y 1,6 es un valor habitual; los títulos suelen llevar menos (`1.2`) porque ocupan pocas líneas y de mayor tamaño.

**Tipografía.** La propiedad `font-family` recibe una **pila de fuentes**: una lista, separada por comas, en orden de preferencia. El navegador usa la primera que encuentra instalada en el ordenador de quien visita la página. En esta unidad se usa:

```css
font-family: system-ui, "Segoe UI", Roboto, Arial, sans-serif;
```

`system-ui` es la letra propia del sistema operativo, que siempre se ve bien y carga al instante, porque no hay que descargarla. `"Segoe UI"`, `Roboto` y `Arial` son el plan B (los nombres con espacios van entre comillas). La pila termina siempre con una **familia genérica** (`sans-serif`, «sin remates»; también existen `serif`, «con remates», y `monospace`, de ancho fijo), que garantiza que, si no hay ninguna de las anteriores, el navegador elige una del tipo indicado.

### 1.7 Herencia

Algunas propiedades, cuando se las das a un elemento, las **heredan** automáticamente los elementos que están dentro de él. Se heredan las que afectan al texto: `color`, `font-family`, `font-size`, `line-height`, `text-align`. No se heredan las que afectan a la caja del elemento: `margin`, `padding`, `border`, `background-color`, `width`.

La herencia es lo que permite escribir poco. Como todo el contenido visible de la página está dentro de `body`, si pones la tipografía, el tamaño, la altura de línea y el color de texto en una regla para `body`, **toda la página los hereda** y no hay que repetirlos en cada elemento. Es la regla más importante de una hoja de estilo, y por eso va siempre la primera.

Una precisión que explica un comportamiento que sorprende al principio: aunque pongas `color` en `body`, **los enlaces siguen siendo azules**. La razón es que la hoja de estilo del navegador (apartado 1.1) tiene una regla dirigida directamente a `a` que les da su color propio, y una regla dirigida a un elemento siempre gana frente a un valor heredado. Para cambiar el color de los enlaces hay que escribir una regla para `a`. Esto lo entenderás del todo en el apartado 2.

### 1.8 Selectores básicos

Con `body` ya has usado el primer selector. Con los seis siguientes cubres casi todo lo que necesitarás. Los verás aplicados al portfolio de Ana en el apartado 1.9.

| Selector | Se llama | A qué elementos se aplica | Ejemplo |
|---|---|---|---|
| `h1` | de tipo | A todos los elementos de ese tipo (el nombre de la etiqueta) | `h1 { … }` |
| `h1, h2` | de grupo | A varios selectores a la vez: la coma los separa y comparten las declaraciones | `h1, h2 { … }` |
| `.presentacion` | de clase | A cualquier elemento con `class="presentacion"` en el HTML. Empieza por un punto | `.presentacion { … }` |
| `header p` | descendiente | A los `p` que están dentro de un `header`, a cualquier profundidad. Lleva un espacio entre los dos | `header p { … }` |
| `a:hover` | de pseudoclase | A un elemento solo cuando está en cierto estado. Lleva dos puntos | `a:hover { … }` |
| `[aria-current="page"]` | de atributo | A los elementos que tienen ese atributo con ese valor. Va entre corchetes | `nav a[aria-current="page"] { … }` |

Algunas precisiones sobre cada uno:

- **Clase.** Se pone en el HTML con el atributo `class` y se puede usar en tantos elementos como quieras. Sirve para dar estilo solo a algunos elementos de un tipo: `.presentacion` afecta a los párrafos de presentación y a ningún otro `p`.
- **Descendiente.** `header p` no afecta al `p` de `main` ni al de `footer`, porque no están dentro de `header`. Es la forma de distinguir un `p` de otro por su posición en el árbol.
- **Pseudoclase.** `:hover` se aplica cuando el puntero del ratón está encima del elemento. `:focus` se aplica cuando el elemento tiene el **foco del teclado**, es decir, cuando se llega a él con la tecla Tab. Escribir `:hover` sin `:focus` deja fuera a quien navega con teclado, por eso se escriben juntos: `a:hover, a:focus`.
- **Atributo.** `[aria-current="page"]` es el enlace de la página donde estás, el que marcaste en el HTML de la UD2 para los lectores de pantalla. Encaja muy bien: el HTML ya sabe cuál es la página actual y el CSS solo tiene que decidir cómo se ve.

Existe también el selector de **identificador**, con almohadilla (`#oferta`), que se aplica al elemento con ese `id`. Conviene reconocerlo porque lo verás en código ajeno, pero **para dar estilo es mejor usar clases**: un `id` es único en la página y sirve sobre todo para anclas y para asociar un `label` a su control, y un selector de `id` es tan fuerte que luego cuesta contradecirlo (el porqué, en el apartado 2.3). También existe el selector universal `*`, que se aplica a todos los elementos y que usarás en el apartado 2.

### 1.9 La primera hoja de estilo del portfolio

Con todo lo anterior, esta es la primera versión de `css/estilos.css` para el portfolio de Ana. Créala en la carpeta `css` y recarga las tres páginas: con un solo archivo, la tipografía, el tamaño de letra, los colores y el fondo cambian en todas ellas.

```css
/* estilos.css · hoja de estilo del portfolio · UD3, primera versión */

/* body: todo el texto de la página hereda de aquí la fuente, el tamaño y el color */
body {
  font-family: system-ui, "Segoe UI", Roboto, Arial, sans-serif; /* varias fuentes: el navegador usa la primera que tenga */
  font-size: 1.125rem;       /* 1.125 veces el tamaño base del navegador (18 px si el base es 16) */
  line-height: 1.6;          /* altura de línea: 1,6 veces el tamaño de letra, sin unidad */
  color: #1f2933;            /* color del texto, en hexadecimal */
  background-color: #f8fafc; /* fondo casi blanco */
}

/* h1, h2: dos selectores separados por coma comparten las mismas declaraciones */
h1, h2 {
  color: #1e3a8a;            /* azul oscuro */
  line-height: 1.2;          /* los títulos, más apretados que el texto */
}

h1 {
  font-size: 2.25rem;
}

h2 {
  font-size: 1.5rem;
}

/* a: todos los enlaces */
a {
  color: #1d4ed8;
}

/* a:hover, a:focus: con el ratón encima, o con el foco del teclado (al llegar con Tab) */
a:hover,
a:focus {
  color: #1e3a8a;
  text-decoration-thickness: 3px;
}

/* header p: solo los p que están dentro de header (el nombre del sitio) */
header p {
  font-weight: 700;          /* negrita */
}

/* nav a[aria-current="page"]: el enlace de la página donde estás, marcado ya en el HTML */
nav a[aria-current="page"] {
  font-weight: 700;
  text-decoration: none;
  color: #1f2933;
}

/* .presentacion: la clase se pone en el HTML con class="presentacion" */
.presentacion {
  font-size: 1.25rem;
  color: #334155;
}

/* footer: texto secundario, algo más pequeño */
footer {
  font-size: 0.875rem;
  color: #475569;
}
```

Qué hace cada regla, de arriba abajo:

- `body` establece la base que hereda toda la página: la pila de fuentes, un tamaño de `1.125rem` (18 px si el base es 16), una altura de línea de 1,6, el color del texto y el fondo casi blanco.
- `h1, h2` comparte color y altura de línea entre los dos niveles de título. Después, `h1` y `h2` fijan cada uno su tamaño. Los selectores de grupo evitan repetir declaraciones.
- `a` da color a todos los enlaces (por la razón que se explicó en el apartado 1.7). `a:hover, a:focus` cambian el color y engrosan el subrayado al pasar el ratón o llegar con Tab.
- `header p` pone en negrita el nombre del sitio, y solo ese: es el único `p` dentro de `header`.
- `nav a[aria-current="page"]` marca visualmente la página actual: negrita, sin subrayado y color de texto normal.
- `.presentacion` agranda y suaviza el párrafo de presentación, y solo ese.
- `footer` reduce el tamaño y suaviza el color del pie.

Si hay algo que no se aplica, el orden de diagnóstico es: ¿está enlazada la hoja en esa página y con la ruta correcta?, ¿has guardado el archivo y recargado la página?, ¿están todos los `;` y todas las `}`?, ¿aparece `estilos.css` cargado en la pestaña Red? Es el mismo diagnóstico que se pide en una auditoría (apartado 6).

Un detalle que habrás notado al probar: **pasa el ratón por el enlace de la página actual. No cambia de color**, mientras que los otros enlaces sí. No es un fallo del navegador: es el resultado de dos reglas que dan órdenes distintas sobre el mismo elemento. Se explica y se corrige en el apartado 2. Si quieres anticiparlo, abre las herramientas de desarrollo (apartado 1.10) sobre ese enlace y mira qué declaración aparece tachada.

#### Lista de mínimos para tu portfolio

Cuando aplicas esta primera hoja a **tu** portfolio, estos son los puntos que debes cubrir. Es la base sobre la que se construye el resto de la unidad:

1. Crear la carpeta `css` y el archivo `estilos.css`, y enlazarlo en las tres páginas.
2. Una regla para `body` con tipografía, tamaño, altura de línea, color de texto y color de fondo.
3. Color para los títulos.
4. Color para los enlaces y un estilo para `:hover` y `:focus`.
5. La página actual del menú diferenciada con `[aria-current="page"]`. Si tu portfolio no marca la página actual, añade `aria-current="page"` al enlace correspondiente en el HTML: es una línea.
6. Una clase propia: ponla en el HTML con `class="..."` y dale estilo.
7. Recargar y hacer clic por las tres páginas para comprobar que todas comparten el aspecto.
8. Comprobar el contraste de los colores de texto y de fondo (apartado 1.11).

### 1.10 Las herramientas de desarrollo: el panel de estilos

Las herramientas de desarrollo del navegador (*DevTools*) son la herramienta con la que se diagnostica el CSS. Para abrirlas sobre un elemento: clic derecho sobre él y **Inspeccionar** (o F12). En Chrome y Edge, en la pestaña **Elementos** aparece el panel **Estilos**; en Firefox, en la pestaña **Inspector** el panel se llama **Reglas**.

Ese panel muestra, para el elemento seleccionado, la lista de **todas las reglas que le afectan**, de qué archivo y qué línea vienen, y sus declaraciones. Las declaraciones que **pierden** frente a otra aparecen **tachadas**. Puedes editar un valor en el propio panel y ver el resultado al instante (el cambio no se guarda en tu archivo: es para probar). En el apartado 2 lo usarás para entender por qué una regla gana a otra, y la pestaña **Calculado** te dirá el valor final de cada propiedad y qué regla lo ha puesto.

### 1.11 Contraste y accesibilidad

Elegir colores no es solo cuestión de gusto. El **contraste** es la diferencia de luminosidad entre el color del texto y el del fondo. Si es insuficiente, el texto se lee mal para mucha gente, no solo para quien tiene una discapacidad visual: una pantalla con reflejos o el brillo bajo del móvil bastan. Las pautas internacionales de accesibilidad web (**WCAG**, *Web Content Accessibility Guidelines*, del W3C) fijan los mínimos en el criterio 1.4.3 de la versión 2.2 (nivel AA):

- **4,5 : 1** para texto normal.
- **3 : 1** para texto grande, que las pautas definen como al menos 18 puntos, o 14 puntos en negrita; en píxeles, aproximadamente 24 px (o 18,5 px en negrita).

La relación de contraste se calcula con la fórmula `(L1 + 0,05) / (L2 + 0,05)`, donde `L1` es la luminosidad relativa del color más claro y `L2` la del más oscuro. No hace falta que la calcules a mano: el selector de color de las herramientas de desarrollo de Chrome y Edge muestra la relación cuando eliges un color, y existen comprobadores en línea. El umbral no se redondea: un 4,499 : 1 no cumple.

Estas son las relaciones de los colores del portfolio de Ana, calculadas con la fórmula de las pautas:

| Texto | Color sobre fondo `#f8fafc` | Relación | ¿Cumple 4,5 : 1? |
|---|---|---|---|
| Texto principal | `#1f2933` | 14,1 : 1 | Sí |
| Títulos | `#1e3a8a` | 9,9 : 1 | Sí |
| Enlaces | `#1d4ed8` | 6,4 : 1 | Sí |
| Texto del pie | `#475569` | 7,2 : 1 | Sí |

En cambio, un rojo claro como `#ff6666` sobre blanco se queda en 2,9 : 1: no cumple ni para texto normal. Y `#ffaaaa` sobre blanco, en 1,8 : 1, es prácticamente ilegible.

> **PARA SABER MÁS**
>
> El contraste es un caso típico de aquello que **un validador no comprueba**: una hoja de estilo con texto rosa claro sobre fondo blanco es CSS perfectamente válido. Por eso el contraste es una de las comprobaciones que hay que hacer a mano, y una de las que se espera ver en una auditoría (apartado 6). Las excepciones que contemplan las pautas incluyen texto puramente decorativo, logotipos y componentes de interfaz inactivos.

### 1.12 Extensión: variables CSS y modo oscuro

Cuando un mismo color aparece en varias reglas, repetirlo es una fuente de errores: si decides cambiar el azul de los títulos, hay que acordarse de todos los sitios. Una **propiedad personalizada**, o **variable CSS**, es un nombre que inventas, que empieza por dos guiones (`--titulo`), y que guarda un valor que luego usas con la función `var()`. Se declaran en el selector `:root`, que es el elemento raíz del documento (el `html`); allí quedan disponibles para toda la página, porque las propiedades personalizadas se heredan.

Con variables es muy fácil ofrecer un **modo oscuro**. Una regla `@media` es una regla que solo se aplica si se cumple una condición. La condición `(prefers-color-scheme: dark)` se cumple cuando la persona tiene el sistema operativo o el navegador en modo oscuro. Dentro de la regla `@media` se vuelven a declarar las variables con valores oscuros, y todo lo que las usa cambia solo.

Esta es la hoja de estilo del apartado 1.9 reescrita con variables y modo oscuro:

```css
/* estilos.css · hoja de estilo del portfolio · primera versión con variables y modo oscuro */

/* :root es el elemento raíz (html). Aquí se declaran las variables, con dos guiones delante */
:root {
  --texto: #1f2933;
  --texto-suave: #334155;
  --pie: #475569;
  --fondo: #f8fafc;
  --titulo: #1e3a8a;
  --enlace: #1d4ed8;
}

/* @media: estas reglas solo se aplican si el sistema de la persona está en modo oscuro */
@media (prefers-color-scheme: dark) {
  :root {
    --texto: #e2e8f0;
    --texto-suave: #cbd5e1;
    --pie: #cbd5e1;
    --fondo: #0f172a;
    --titulo: #bfdbfe;
    --enlace: #93c5fd;
  }
}

body {
  font-family: system-ui, "Segoe UI", Roboto, Arial, sans-serif;
  font-size: 1.125rem;
  line-height: 1.6;
  color: var(--texto);             /* var(): usa el valor guardado en la variable */
  background-color: var(--fondo);
}

h1, h2 {
  color: var(--titulo);
  line-height: 1.2;
}

h1 {
  font-size: 2.25rem;
}

h2 {
  font-size: 1.5rem;
}

a {
  color: var(--enlace);
}

a:hover,
a:focus {
  color: var(--titulo);
  text-decoration-thickness: 3px;
}

header p {
  font-weight: 700;
}

nav a[aria-current="page"] {
  font-weight: 700;
  text-decoration: none;
  color: var(--texto);
}

.presentacion {
  font-size: 1.25rem;
  color: var(--texto-suave);
}

footer {
  font-size: 0.875rem;
  color: var(--pie);
}
```

Para probarlo sin cambiar la configuración de tu sistema, en Chrome y Edge abre las herramientas de desarrollo, menú de tres puntos → **Más herramientas** → **Renderizado**, y en «Emular la función de medios CSS prefers-color-scheme» elige **dark**.

Hay un detalle importante: en el modo oscuro **hay que revisar el contraste de los colores nuevos**, no darlo por supuesto. En esta hoja, todos los textos que antes usaban un color fijo (el texto suave de `.presentacion`, el del pie y el del enlace de la página actual) pasan también a variables. Si se hubieran dejado con sus colores oscuros originales, habrían quedado casi ilegibles sobre el fondo oscuro `#0f172a`: `#334155` daría 1,7 : 1, `#475569` daría 2,4 : 1 y `#1f2933` daría 1,2 : 1. Con los valores de la hoja, las relaciones sobre `#0f172a` son: texto `#e2e8f0` 14,5 : 1, texto suave `#cbd5e1` 12,0 : 1, títulos `#bfdbfe` 12,6 : 1 y enlaces `#93c5fd` 9,9 : 1.

> **COMPRUEBA LO QUE HAS ENTENDIDO**
>
> 1. Explica con un ejemplo de tu portfolio dos ventajas de usar una hoja de estilo externa frente a poner estilos en línea en cada etiqueta.
>
>    **Respuesta.** Mantenimiento: para cambiar el color de los títulos en las tres páginas basta una regla en `estilos.css`, en lugar de editar cada `h1` de cada página. Coherencia: las tres páginas comparten tipografía y colores porque leen la misma hoja. (También valen rendimiento, porque el navegador descarga el archivo una vez, y adaptación, porque el mismo HTML puede tener otro aspecto en móvil.)
>
> 2. En `.aviso { color: #9a3412; font-size: 1.1rem; }`, ¿cuál es el selector, cuál la primera propiedad y cuál su valor?
>
>    **Respuesta.** El selector es `.aviso` (de clase). La primera propiedad es `color` y su valor es `#9a3412`. La segunda declaración es `font-size: 1.1rem`.
>
> 3. Para el HTML `<nav><a href="a.html">A</a></nav><p class="aviso">Hola</p>`, ¿qué selector da estilo solo al `p`: `p`, `.aviso`, `nav a` o `a:hover`?
>
>    **Respuesta.** `.aviso` es el más preciso, porque apunta exactamente a los elementos con esa clase. `p` también seleccionaría ese párrafo, pero en general afectaría a todos los `p` de la página. `nav a` selecciona el enlace y `a:hover` el enlace con el ratón encima: ninguno afecta al `p`.
>
> 4. ¿Por qué se pone la tipografía en `body` y no en cada elemento? ¿Y por qué, aun así, hay que escribir una regla para `a` si quieres cambiar el color de los enlaces?
>
>    **Respuesta.** Por la herencia: la tipografía, el tamaño, la altura de línea y el color de texto pasan a todo el contenido, y se mantienen en un solo sitio. Los enlaces necesitan su propia regla porque la hoja de estilo del navegador ya les da un color a `a` directamente, y una regla dirigida a un elemento gana al valor heredado.
>
> 5. ¿Cumple el contraste un texto `#ff6666` sobre fondo blanco? ¿Qué mínimo se exige para texto normal?
>
>    **Respuesta.** No: la relación es de unos 2,9 : 1 y el mínimo de WCAG 2.2 nivel AA para texto normal es 4,5 : 1 (3 : 1 solo para texto grande).

---

## 2. La cascada, la especificidad y el modelo de caja

`// Criterios de evaluación: 2.g`

### 2.1 El problema: dos reglas, un mismo elemento

En el apartado 1.9 dejaste una pregunta abierta: el enlace de la página actual no cambia de color al pasar el ratón, mientras que los demás sí. Estas son las dos reglas implicadas, tal como están en la primera hoja de estilo:

```css
a:hover,
a:focus {
  color: #1e3a8a;
  text-decoration-thickness: 3px;
}

nav a[aria-current="page"] {
  font-weight: 700;
  text-decoration: none;
  color: #1f2933;
}
```

Cuando el ratón está sobre el enlace de la página actual, **las dos reglas se aplican a la vez** a ese mismo elemento y **dan dos colores distintos**: la primera dice `#1e3a8a` y la segunda `#1f2933`. El navegador solo puede pintar uno. Decidir cuál es el trabajo de la **cascada**, que es el algoritmo con el que el navegador resuelve los conflictos entre declaraciones. Casi todos los «¿por qué no funciona mi CSS?» se reducen a un conflicto de este tipo, y la herramienta para entenderlos es el panel de estilos de las herramientas de desarrollo (apartado 1.10), donde la declaración que pierde aparece tachada.

### 2.2 Cómo decide la cascada

Para cada propiedad de cada elemento, el navegador reúne todas las declaraciones que podrían aplicarse y se queda con una. Lo hace aplicando estos criterios, **por orden**: cada uno solo se consulta si el anterior ha empatado.

1. **Relevancia.** Solo cuentan las declaraciones de reglas cuyo selector encaja con el elemento (y, para pseudoclases como `:hover`, solo mientras se cumple el estado).
2. **Origen e importancia.** Las declaraciones tienen tres posibles orígenes: la hoja de estilo del **navegador**, la de la **persona usuaria** (existe, aunque rara vez se usa: por ejemplo, una hoja de accesibilidad personal) y la del **autor**, que eres tú. En declaraciones normales, gana el autor sobre el usuario y el usuario sobre el navegador. Por eso tu hoja puede contradecir la del navegador. Las declaraciones marcadas con `!important` pasan a una categoría superior a la de todas las normales y, entre ellas, el orden de origen se invierte (la del navegador puede más que la del autor).
3. **Especificidad.** Entre declaraciones del mismo origen e importancia, gana la de selector más **específico**, es decir, el que apunta de forma más concreta. Se calcula como se explica en el apartado 2.3.
4. **Orden de aparición.** Si hay empate en todo lo anterior, gana la declaración que aparece **la última**. Si las reglas están en el mismo archivo, la que va más abajo; si están en distintos archivos o elementos `style`, la que se enlaza o aparece después en el `head`.

(Existen criterios adicionales, como las capas con `@layer` o la proximidad con `@scope`, que no se usan en esta unidad.)

El orden de los criterios tiene una consecuencia práctica muy importante: **la especificidad pesa más que el orden**. Una regla muy específica gana aunque esté escrita antes que otra menos específica. El orden solo desempata cuando la especificidad es idéntica.

El atributo `style` de un elemento (CSS en línea, apartado 1.3) se considera por encima de cualquier selector de una hoja de estilo. Eso es lo que lo hace tan difícil de contradecir desde fuera, y es otra razón para no abusar de él.

### 2.3 La especificidad

La **especificidad** de un selector se calcula contando tres tipos de componentes y escribiendo el resultado como tres números, uno por columna, de la columna más fuerte a la más débil. Se suele escribir con guiones: `ID-CLASE-TIPO`.

| Columna | Qué se cuenta | Cada uno suma |
|---|---|---|
| **ID** | Selectores de identificador (`#oferta`) | 1-0-0 |
| **CLASE** | Clases (`.aviso`), atributos (`[aria-current="page"]`) y pseudoclases (`:hover`) | 0-1-0 |
| **TIPO** | Tipos (`p`, `nav`) y pseudoelementos | 0-0-1 |

El selector universal `*` y los combinadores (el espacio del descendiente y otros como `>`) **no suman nada**. Para comparar dos selectores se miran las columnas **de izquierda a derecha**: gana el que tenga más en la columna ID, sin importar lo que haya en las demás; si empatan, se mira la columna CLASE, y así sucesivamente. **No son un número decimal**: `0-0-11` (once tipos) pierde frente a `0-1-0` (una clase).

| Selector | Cálculo | Especificidad |
|---|---|---|
| `*` | no suma | 0-0-0 |
| `p` | un tipo | 0-0-1 |
| `header p` | dos tipos | 0-0-2 |
| `nav a` | dos tipos | 0-0-2 |
| `.aviso` | una clase | 0-1-0 |
| `[aria-current="page"]` | un atributo | 0-1-0 |
| `p.aviso` | un tipo y una clase | 0-1-1 |
| `a:hover` | un tipo y una pseudoclase | 0-1-1 |
| `a[aria-current="page"]` | un tipo y un atributo | 0-1-1 |
| `nav a[aria-current="page"]` | dos tipos y un atributo | 0-1-2 |
| `#oferta` | un identificador | 1-0-0 |
| `#oferta .precio` | un identificador y una clase | 1-1-0 |

Ahora puedes resolver el caso de las dos reglas del apartado 2.1. Ambas coinciden en origen (autor) e importancia (normal), así que decide la especificidad: `a:hover` vale **0-1-1** y `nav a[aria-current="page"]` vale **0-1-2**. La segunda tiene más en la columna TIPO (dos tipos frente a uno) y las demás columnas empatan, así que **gana la segunda**: el color del enlace actual sigue siendo `#1f2933` aunque el ratón esté encima. Fíjate en que da igual el orden: en la hoja del apartado 1.9 la regla de `:hover` va primero, pero aunque fuera la última seguiría perdiendo.

Este cálculo también explica por qué se desaconseja usar `id` para dar estilo: una regla con `#oferta` tiene 1-0-0, y para contradecirla desde otra regla hay que escribir un selector con otro identificador, o recurrir a `!important`. Con clases, la especificidad se mantiene baja y es fácil de sobrescribir cuando hace falta.

> **PARA SABER MÁS**
>
> Las pseudoclases `:is()` y `:not()` no aportan especificidad por sí mismas: suman la del argumento más específico de su lista (por ejemplo, `:is(p, #fakeId)` vale 1-0-0). La pseudoclase `:where()` siempre vale 0-0-0, incluidos sus argumentos, lo que la hace útil para escribir reglas fáciles de sobrescribir. Y `:root` cuenta como una pseudoclase (0-1-0).

### 2.4 Tres formas de resolver el conflicto

Que el enlace de la página actual no reaccione al ratón puede ser exactamente lo que quieres: la persona ya está en esa página y no necesita pulsar ese enlace. Lo importante es que sea una **decisión** tuya y no un accidente. Si prefieres que sí reaccione, hay tres caminos:

**Opción 1: bajar la especificidad de la regla de la página actual y colocarla antes de `:hover`.** Si escribes `a[aria-current="page"]` en lugar de `nav a[aria-current="page"]`, su especificidad pasa a 0-1-1, igual que la de `a:hover`. Con empate en especificidad, decide el orden: si la regla de `:hover` va **después**, gana en el momento en que el ratón está encima. Y cuando no hay ratón encima, la regla `a[aria-current="page"]` (0-1-1) sigue ganando a la regla `a` (0-0-1), de modo que el enlace actual conserva su aspecto propio.

```css
a {
  color: #1d4ed8;
}

/* 0-1-1: un atributo y un tipo. Va ANTES de :hover */
a[aria-current="page"] {
  font-weight: 700;
  text-decoration: none;
  color: #1f2933;
}

/* 0-1-1: una pseudoclase y un tipo. Empata con la anterior y, al ir después, gana */
a:hover,
a:focus {
  color: #1e3a8a;
  text-decoration-thickness: 3px;
}
```

**Opción 2: declarar el caso explícitamente.** Se añade una regla específica para el enlace actual cuando el ratón está encima, con especificidad suficiente (0-2-2) para ganar:

```css
nav a[aria-current="page"]:hover,
nav a[aria-current="page"]:focus {
  color: #1e3a8a;
}
```

**Opción 3: `!important`.** Poner `color: #1e3a8a !important;` en la regla de `:hover` también funciona, porque una declaración importante gana a las normales. Pero es la peor opción.

> **ALTERNATIVAS Y CRITERIO**
>
> Las tres opciones consiguen que el enlace actual reaccione. Se prefiere la **opción 1** porque mantiene bajos los selectores y resuelve el conflicto con una regla simple y un orden claro. La opción 2 funciona, pero eleva la especificidad y obliga a repetir el selector largo, con lo que el siguiente conflicto exigirá otro selector aún más largo. La opción 3 se evita porque `!important` no resuelve el conflicto, sino que lo oculta: la siguiente regla que quiera contradecir esa declaración tendrá que usar también `!important`, y la hoja acaba como una carrera de declaraciones importantes imposible de mantener. Se reserva `!important` para casos muy concretos, como sobrescribir un CSS ajeno que no puedes modificar.

La opción 1 depende de un **orden**: la regla de `:hover` tiene que ir detrás. Si más adelante alguien reordena la hoja, el comportamiento cambia sin que haya ningún error. Por eso conviene dejar un comentario junto a las dos reglas, como se hace en la hoja del apartado 2.8.

### 2.5 Diagnosticar la cascada con las herramientas de desarrollo

El procedimiento para cualquier conflicto de estilos es el mismo:

1. Clic derecho sobre el elemento → **Inspeccionar**.
2. En el panel **Estilos** (Chrome y Edge) o **Reglas** (Firefox) se listan las reglas que afectan al elemento, de la más fuerte a la más débil. Las declaraciones que pierden aparecen **tachadas**.
3. Para ver el estado `:hover`, que desaparece en cuanto mueves el ratón fuera del elemento, usa el botón **:hov** del propio panel y marca `:hover`: fuerza ese estado sobre el elemento seleccionado sin necesidad de mantener el ratón encima.
4. Si dudas del valor final de una propiedad, la pestaña **Calculado** lo muestra, y al desplegar la propiedad indica qué regla lo establece.
5. Puedes cambiar valores en el propio panel para probar. Cuando encuentres la solución, aplícala en tu archivo: el panel no guarda nada.

### 2.6 El modelo de caja

Para el navegador, cada elemento de la página es una **caja rectangular**, y todos los problemas de espacios, huecos y anchos se resuelven entendiendo esa caja. Cada caja tiene cuatro capas, de dentro a fuera:

```plaintext
┌──────────────────── margin (margen) ────────────────────┐
│  ┌────────────── border (borde) ────────────────────┐   │
│  │  ┌────────── padding (relleno) ───────────────┐  │   │
│  │  │                                            │  │   │
│  │  │        contenido (texto, imagen…)          │  │   │
│  │  │                                            │  │   │
│  │  └────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

- El **contenido** es el texto, la imagen o los elementos hijos. Su tamaño lo fijan `width` (ancho) y `height` (alto).
- El **padding** (relleno) es el espacio entre el contenido y el borde. Es interior: si el elemento tiene fondo, el fondo también cubre el relleno.
- El **border** (borde) es la línea que rodea el relleno. Tiene grosor, estilo y color.
- El **margin** (margen) es el espacio exterior, que separa esta caja de las vecinas. Es transparente y no forma parte del tamaño visible de la caja.

**Propiedades abreviadas.** `margin` y `padding` aceptan de uno a cuatro valores, con un orden fijo que sigue las agujas del reloj desde arriba (arriba, derecha, abajo, izquierda). Con un valor, se aplica a los cuatro lados: `padding: 1rem`. Con dos, el primero es el vertical (arriba y abajo) y el segundo el horizontal: `margin: 0 auto`. Con tres, arriba, horizontal y abajo. Con cuatro, uno por lado en el orden arriba, derecha, abajo, izquierda. También se pueden fijar lados sueltos con `margin-top`, `padding-left`, etc.

La propiedad abreviada `border` lleva tres valores: grosor, estilo y color. Por ejemplo, `border: 1px solid #94a3b8`. **El estilo es obligatorio en la práctica**: si lo omites, el valor por defecto es `none` y el borde no se dibuja aunque hayas puesto grosor y color.

#### El ancho de una caja: `content-box` frente a `border-box`

Por defecto, `width` es el ancho del **contenido**, no de la caja entera: el relleno y el borde se **suman** a él. Este es el comportamiento llamado `content-box`. La propiedad `box-sizing` permite cambiarlo: con `box-sizing: border-box`, `width` es el ancho de la caja visible (contenido + relleno + borde), y el contenido se reduce para que todo quepa.

Compruébalo con este documento, que define dos cajas idénticas salvo por `box-sizing`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Dos modelos de caja</title>
  <style>
    .caja {
      width: 300px;
      padding: 20px;
      border: 5px solid #1e3a8a;
      margin: 10px;
      background-color: #dbeafe;
    }

    .caja-b {
      box-sizing: border-box;   /* width incluye el relleno y el borde */
    }
  </style>
</head>
<body>
  <div class="caja">Caja A: modelo estándar (content-box)</div>
  <div class="caja caja-b">Caja B: box-sizing: border-box</div>
</body>
</html>
```

Ábrelo y mide cada caja con el inspector, haciendo clic sobre ella y mirando el diagrama de la caja en la pestaña **Calculado**:

```text
Caja A (content-box):  contenido 300 px + relleno 2×20 px + borde 2×5 px = 350 px de ancho visible
Caja B (border-box):   300 px de ancho visible = contenido 250 px + relleno 2×20 px + borde 2×5 px
```

En ambas, el margen de 10 px queda fuera y no cuenta en el tamaño de la caja. El primer modelo obliga a hacer una cuenta cada vez que combinas un ancho con relleno o borde; con `border-box`, el ancho que escribes es el ancho que ves. Por eso es habitual aplicarlo a todos los elementos con el selector universal:

```css
* {
  box-sizing: border-box;
}
```

> **ALTERNATIVAS Y CRITERIO**
>
> Se podría trabajar con `content-box` y restar a mano el relleno y el borde de cada ancho. Se prefiere `border-box` porque el cálculo desaparece y los anchos son predecibles: si pones `width: 50%` a dos cajas con relleno, caben una al lado de la otra; con `content-box`, el relleno las desborda. Es la razón por la que casi toda hoja de estilo profesional empieza con esta regla.

#### Centrar con `margin: 0 auto` y colapso de márgenes

Para **centrar horizontalmente** un bloque, se le da un ancho máximo y márgenes laterales `auto`. El navegador reparte a partes iguales el espacio que sobra entre los dos márgenes. Por eso `margin: 0 auto` solo centra si la caja tiene un ancho menor que el de su contenedor (con `width` o `max-width`); si ocupa todo el ancho, no sobra nada que repartir.

Los **márgenes verticales** entre cajas hermanas tienen una peculiaridad: no se suman, sino que **colapsan** y prevalece el mayor. Si un párrafo tiene `margin-bottom: 1.5rem` y el siguiente `margin-top: 1rem`, el hueco entre ambos es de `1.5rem`, no de `2.5rem`. El relleno y el borde nunca colapsan. Si sospechas que un hueco no es la suma de lo que has escrito, probablemente es esto.

### 2.7 Cajas en bloque y cajas en línea

Otra propiedad que condiciona cómo se comporta una caja es `display`, que dice si es de **bloque** o **en línea**:

| | `display: block` (bloque) | `display: inline` (en línea) |
|---|---|---|
| Colocación | Empieza en una línea nueva y ocupa todo el ancho disponible | Fluye dentro de la línea de texto, junto a lo que tiene a los lados |
| `width` y `height` | Se aplican | Se ignoran |
| `margin` y `padding` verticales | Empujan a las cajas vecinas | No empujan a las líneas vecinas |
| Ejemplos | `p`, `h1`–`h6`, `ul`, `section`, `header`, `nav`, `main`, `footer` | `a`, `strong`, `em`, `abbr`, `span`, `small` |

Hay un tercer valor, `inline-block`, que combina ambos: la caja fluye en la línea, pero acepta ancho, alto y márgenes verticales. Y `display: flex`, que verás en el apartado 3, convierte un elemento en un contenedor flexible. Si una propiedad parece no hacer nada, comprueba el tipo de caja: por ejemplo, un `width` sobre un `a` en línea se ignora.

### 2.8 Aplicación al portfolio: espacios, bordes y la tabla de formación

Con lo anterior, esta es la segunda versión de `css/estilos.css`. Reemplaza el contenido de tu archivo por este (las diferencias con la primera: la regla `*`, el ancho máximo y el centrado de la página, los espacios de `header`, `nav`, `section` y `footer`, el arreglo del enlace actual con la opción 1 del apartado 2.4 y las reglas de la tabla):

```css
/* estilos.css · hoja de estilo del portfolio · UD3, segunda versión (cascada y modelo de caja) */

/* *: selector universal. El ancho de cada caja incluye su relleno y su borde */
* {
  box-sizing: border-box;
}

body {
  font-family: system-ui, "Segoe UI", Roboto, Arial, sans-serif;
  font-size: 1.125rem;
  line-height: 1.6;
  color: #1f2933;
  background-color: #f8fafc;
  max-width: 48rem;          /* la página no pasa de 48 rem de ancho (768 px si el base es 16) */
  margin: 0 auto;            /* arriba y abajo 0; izquierda y derecha auto: se reparte el espacio sobrante y la página queda centrada */
  padding: 1rem;             /* aire entre el borde de la caja y el contenido */
}

h1, h2 {
  color: #1e3a8a;
  line-height: 1.2;
}

h1 {
  font-size: 2.25rem;
}

h2 {
  font-size: 1.5rem;
}

a {
  color: #1d4ed8;
}

/* Enlace de la página actual. Especificidad 0-1-1 (un atributo, un tipo): se declara ANTES de :hover */
a[aria-current="page"] {
  font-weight: 700;
  text-decoration: none;
  color: #1f2933;
}

/* Especificidad 0-1-1 (una pseudoclase, un tipo). Empata con la regla anterior y, al ir después, gana */
a:hover,
a:focus {
  color: #1e3a8a;
  text-decoration-thickness: 3px;
}

header {
  border-bottom: 2px solid #cbd5e1;   /* borde: grosor, estilo y color */
  margin-bottom: 1.5rem;              /* separación con lo que hay debajo */
  padding-bottom: 0.5rem;
}

header p {
  font-weight: 700;
  margin: 0.5rem 0;
}

nav ul {
  list-style: none;          /* sin viñetas */
  margin: 0;
  padding: 0;                /* los ul traen un relleno a la izquierda por defecto */
}

nav li {
  margin: 0.25rem 0;
}

.presentacion {
  font-size: 1.25rem;
  color: #334155;
}

section {
  margin-bottom: 1.5rem;
}

/* Tabla de formación */
table {
  border-collapse: collapse; /* los bordes de celdas contiguas se fusionan en uno solo */
  width: 100%;
}

caption {
  font-weight: 700;
  text-align: left;
  margin-bottom: 0.5rem;
}

th,
td {
  border: 1px solid #94a3b8;
  padding: 0.5rem 0.75rem;
  text-align: left;
}

th {
  background-color: #e2e8f0;
}

footer {
  font-size: 0.875rem;
  color: #475569;
  border-top: 2px solid #cbd5e1;
  margin-top: 2rem;
  padding-top: 0.5rem;
}
```

Qué hay de nuevo, por bloques:

- **`*`** aplica `border-box` a todas las cajas.
- **`body`** con `max-width: 48rem` y `margin: 0 auto` limita el ancho de la página y la centra; `padding: 1rem` deja aire entre el borde de la ventana y el contenido. Las líneas de texto muy largas se leen mal, y un ancho máximo de este orden las mantiene razonables.
- **`a[aria-current="page"]` y `a:hover, a:focus`** son las dos reglas de la opción 1 del apartado 2.4, en el orden que se explicó. Ese orden **es importante** y está señalado en los comentarios.
- **`header`, `header p`, `nav ul`, `nav li`** añaden un borde inferior a la cabecera, separan el nombre del sitio del menú y quitan las viñetas y el relleno que los navegadores dan por defecto a las listas (`list-style: none`, `padding: 0`).
- **`section`** separa las secciones con un margen inferior.
- **Tabla.** `border-collapse: collapse` hace que los bordes de celdas contiguas se fundan en uno solo, en lugar de dibujarse dobles. `caption` y `th, td` fijan el título, los bordes y el relleno de las celdas. `th` tiene un fondo gris azulado, y `text-align: left` alinea a la izquierda los encabezados, que el navegador centra por defecto.
- **`footer`** añade un borde superior y espacio por encima.

Para probar la tabla, añade a `sobre-mi.html`, dentro de `main` y detrás de la sección «Mi forma de trabajar», esta nueva sección con la tabla de formación (si tu portfolio ya tiene una tabla propia de la UD2, estilízala con las mismas reglas):

```html
    <section>
      <h2>Formación</h2>
      <table>
        <caption>Estudios de Ana Martín</caption>
        <thead>
          <tr>
            <th scope="col">Estudios</th>
            <th scope="col">Periodo</th>
            <th scope="col">Estado</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Desarrollo de Aplicaciones Web</td>
            <td>2026–2028</td>
            <td>En curso</td>
          </tr>
          <tr>
            <td>Sistemas Microinformáticos y Redes</td>
            <td>2024–2026</td>
            <td>Finalizado</td>
          </tr>
        </tbody>
      </table>
    </section>
```

Comprueba con el inspector tres cosas: que al pasar el ratón por «Sobre mí» (la página actual) el color cambia ahora, que el modelo de caja de `body` muestra el relleno y el ancho máximo que has fijado, y que las celdas de la tabla tienen el relleno de `0.5rem 0.75rem`.

> **ERRORES FRECUENTES**
>
> **Escribir `border: 1px #94a3b8` y no ver el borde.** Falta el estilo (`solid`): sin él, el valor por defecto es `none`.
>
> **Centrar con `margin: 0 auto` una caja que ocupa todo el ancho.** No hay espacio que repartir, así que no se nota. Hay que limitar su ancho con `max-width` o `width`.
>
> **Dar `width` a un enlace y que no pase nada.** Los enlaces son cajas en línea y las cajas en línea ignoran `width`. Habría que cambiar su `display` a `inline-block` o a `block`.
>
> **Sumar márgenes verticales y sorprenderse con el resultado.** Los márgenes verticales colapsan: se queda el mayor.

> **COMPRUEBA LO QUE HAS ENTENDIDO**
>
> 1. Calcula la especificidad de `nav a:hover`, de `.menu a` y de `#cabecera .menu`. ¿Cuál gana si las tres aplican el mismo `color` a un mismo enlace?
>
>    **Respuesta.** `nav a:hover` = un tipo `nav`, un tipo `a` y una pseudoclase = 0-1-2. `.menu a` = una clase y un tipo = 0-1-1. `#cabecera .menu` = un identificador y una clase = 1-1-0. Gana la tercera, porque la columna ID (1) pesa más que cualquier número en las demás.
>
> 2. En el portfolio, `a:hover` (0-1-1) está escrito antes que `nav a[aria-current="page"]` (0-1-2). ¿Qué pasaría si se escribiera después y si se escribiera antes? ¿Por qué?
>
>    **Respuesta.** Lo mismo en los dos casos: gana la de `nav a[aria-current="page"]`, porque su especificidad es mayor y la especificidad se consulta antes que el orden de aparición. El orden solo desempataría si las especificidades fueran iguales.
>
> 3. Una caja tiene `width: 400px; padding: 10px; border: 2px solid`. ¿Qué ancho visible tiene con `content-box` y con `border-box`? ¿Y cuál es el ancho del contenido en cada caso?
>
>    **Respuesta.** Con `content-box`, el ancho visible es 400 + 2×10 + 2×2 = 424 px, y el contenido mide 400 px. Con `border-box`, el ancho visible es 400 px y el contenido mide 400 − 2×10 − 2×2 = 376 px.
>
> 4. ¿Qué significa `margin: 0 auto` y en qué condición centra una caja?
>
>    **Respuesta.** Márgenes verticales de 0 y márgenes laterales automáticos, que reparten el espacio sobrante a partes iguales. Solo centra si la caja es más estrecha que su contenedor, es decir, si tiene un `width` o `max-width` que deja espacio libre.
>
> 5. Un `p` con `margin-bottom: 2rem` va seguido de otro `p` con `margin-top: 1rem`. ¿Cuánto espacio hay entre ellos?
>
>    **Respuesta.** 2 rem, no 3 rem: los márgenes verticales entre hermanos colapsan y prevalece el mayor.

---

## 3. Disposición de la página y adaptación al móvil

`// Criterios de evaluación: 2.g`

### 3.1 El flujo normal del documento

Si no le dices nada al navegador, las cajas se colocan en el **flujo normal**: las cajas de bloque se apilan una debajo de otra, de arriba abajo, y las cajas en línea van una detrás de otra dentro de las líneas de texto (apartado 2.7). Es lo que ves en el portfolio hasta ahora: el menú es una lista, así que cada enlace aparece en su propia línea; las dos secciones de «Sobre mí» están una encima de otra. Para colocar cosas una al lado de otra hace falta un sistema de disposición. El que se usa en esta unidad es **Flexbox**.

### 3.2 Ancho máximo, centrado e imágenes flexibles

Dos reglas sencillas hacen que una página se comporte bien en pantallas de cualquier ancho. Ya has usado la primera en el apartado 2.8: `max-width` en el `body`. Un **ancho máximo** (en lugar de un ancho fijo) significa «ocupa todo el espacio disponible hasta este límite»: en un móvil, la página ocupa la pantalla entera; en un monitor grande, se queda en 48 rem y se centra con `margin: 0 auto`. Con un `width` fijo, en cambio, una página de 800 px de ancho en un móvil de 390 px obligaría a desplazarse en horizontal.

La segunda regla afecta a las imágenes. Una imagen tiene su propio ancho intrínseco y no se adapta sola al de su contenedor:

```css
img {
  max-width: 100%;   /* nunca más ancha que su contenedor */
  height: auto;      /* la altura se ajusta para mantener la proporción */
}
```

`max-width: 100%` impide que la imagen desborde su contenedor, y `height: auto` hace que, al reducirse el ancho, el alto se reduzca en proporción en lugar de deformar la imagen. (Los atributos `width` y `height` del HTML, que viste en la UD2, siguen siendo útiles: permiten al navegador reservar el espacio antes de que la imagen se cargue.)

### 3.3 Flexbox

**Flexbox** (*Flexible Box Layout*) es un sistema de disposición para colocar elementos en **una dimensión**: en fila o en columna, repartiendo el espacio entre ellos. Se activa con una sola declaración sobre el elemento contenedor:

```css
.contenedor {
  display: flex;
}
```

A partir de ahí, el elemento con `display: flex` es el **contenedor flex** (*flex container*) y **sus hijos directos** son los **elementos flex** (*flex items*). Esto último es importante y causa muchos errores: solo los hijos directos se convierten en elementos flex. En `nav ul { display: flex; }`, los elementos flex son los `li`, no los `a` que hay dentro de ellos.

Un contenedor flex tiene dos ejes. El **eje principal** es la dirección en la que se colocan los elementos; el **eje transversal** es el perpendicular. Estas son las propiedades que usarás:

| Propiedad (en el contenedor) | Qué hace | Valores habituales |
|---|---|---|
| `flex-direction` | Fija la dirección del eje principal | `row` (en fila, el valor por defecto), `column` (en columna) |
| `gap` | Separa los elementos entre sí, sin tocar los bordes exteriores | `1rem`, `0.5rem 1rem` (filas y columnas) |
| `justify-content` | Reparte los elementos a lo largo del eje principal | `flex-start`, `center`, `space-between` |
| `align-items` | Alinea los elementos en el eje transversal | `stretch` (por defecto), `center`, `flex-start` |
| `flex-wrap` | Permite que los elementos pasen a otra línea si no caben | `nowrap` (por defecto), `wrap` |

| Propiedad (en un elemento flex) | Qué hace |
|---|---|
| `flex: 1` | Abreviatura de «reparte el espacio sobrante a partes iguales con los demás elementos que también lo lleven». Equivale a `flex: 1 1 0%` (puede crecer, puede encogerse y parte de un tamaño base de cero) |

Si en un contenedor flex en fila varios elementos llevan `flex: 1`, todos terminan con el mismo ancho; si uno lleva `flex: 2` y otros `flex: 1`, el primero ocupa el doble. Es la forma sencilla de hacer columnas.

#### Dos usos en el portfolio

**El menú.** Un `ul` con tres `li` es una lista vertical. Con `display: flex` y `flex-direction: row`, los tres `li` pasan a ir uno al lado de otro, separados por el `gap` que indiques. Se quitan además las viñetas y el relleno (apartado 2.8).

**Las dos columnas de «Sobre mí».** Las dos secciones «Qué sé hacer» y «Mi forma de trabajar» son hermanas dentro de `main`, mezcladas con el título y el párrafo de presentación. Para convertirlas en columnas hay que agruparlas en un contenedor propio. Como no existe un elemento semántico para «grupo de columnas», aquí es correcto usar un `div`, porque su única función es de disposición (en la UD2 viste que la «sopa de `div`» es un problema cuando existe un elemento semántico adecuado; no es el caso). Se le da una clase, `columnas`, y se aplica Flexbox a esa clase.

### 3.4 Adaptación al móvil

Una página es **adaptable** (*responsive*) cuando se ve y se usa bien tanto en un móvil como en un monitor, con el mismo HTML. Tiene tres ingredientes.

**1. El `meta viewport`.** Ya lo tienes en el `head` de las tres páginas desde la UD2:

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

Sin esta línea, los navegadores de móvil asumen que la página está pensada para un escritorio y la dibujan en un ancho virtual grande (en torno a los 980 px), que después reducen para que quepa en la pantalla: se ve todo diminuto. Con `width=device-width`, el navegador usa el ancho real de la pantalla del dispositivo. Es la condición previa de todo lo demás.

**2. Estilos para pantalla estrecha por defecto, y extras para pantalla ancha: el enfoque *mobile first*.** Se escribe primero el aspecto pensado para el móvil (menú en columna, secciones apiladas) y después se añaden las reglas que solo hacen falta cuando hay más sitio. Es más sencillo que lo contrario, porque la disposición apilada es la natural del flujo normal y el diseño para pantalla ancha es un añadido.

**3. Las consultas de medios (`@media`).** Una **consulta de medios** es una regla que solo se aplica si se cumple una condición sobre el dispositivo. La más usada es la del ancho de la ventana:

```css
@media (min-width: 40rem) {
  /* reglas que solo se aplican si la ventana mide al menos 40 rem de ancho */
}
```

`min-width: 40rem` significa «ancho mínimo de 40 rem»: la condición se cumple en ventanas de 40 rem (640 px si el tamaño base es 16 px) o más. Dentro de las llaves va una lista de reglas completas, con sus selectores. El valor de corte (40 rem) se llama **punto de ruptura** (*breakpoint*). Una precisión técnica: en las consultas de medios, `rem` y `em` se calculan siempre con el tamaño de letra inicial del navegador (normalmente 16 px), no con el que hayas fijado tú en `html` o `body`.

> **ALTERNATIVAS Y CRITERIO**
>
> Se pueden escribir los estilos para pantalla ancha como base y usar `@media (max-width: …)` para corregirlos en pantallas estrechas. Se prefiere el enfoque *mobile first* con `min-width` porque el código base es el más simple (el flujo normal), cada consulta de medios solo añade lo que sobra en pantallas mayores y, en un móvil, el navegador no tiene que deshacer reglas pensadas para otro dispositivo. Los puntos de ruptura se eligen donde el diseño empieza a verse mal, no por la medida de ningún dispositivo en concreto.

### 3.5 Las dos versiones finales: HTML y CSS

Con todo lo anterior, esta es la versión final de `sobre-mi.html`. Los cambios respecto a la versión con la tabla son dos `div`: uno con la clase `columnas`, que agrupa las dos secciones, y otro con la clase `tabla-scroll`, que envuelve la tabla (el porqué, justo debajo). `index.html` y `contacto.html` no cambian (solo el contenido de la hoja de estilo que enlazan).

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Sobre mí · Portfolio de Ana Martín</title>
  <link rel="stylesheet" href="css/estilos.css">
</head>
<body>
  <header>
    <p>Ana Martín · Desarrolladora web</p>
    <nav aria-label="Principal">
      <ul>
        <li><a href="index.html">Inicio</a></li>
        <li><a href="sobre-mi.html" aria-current="page">Sobre mí</a></li>
        <li><a href="contacto.html">Contacto</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <h1>Sobre mí</h1>
    <p class="presentacion">Estudio <abbr title="Desarrollo de Aplicaciones Web">DAW</abbr> y me gusta entender cómo funciona lo que construyo. Antes cursé el ciclo de <abbr title="Sistemas Microinformáticos y Redes">SMR</abbr>, así que las redes no me asustan.</p>

    <div class="columnas">
      <section>
        <h2>Qué sé hacer</h2>
        <ul>
          <li>Maquetar páginas con <strong>HTML semántico</strong>.</li>
          <li>Programar en Python: scripts de automatización y lectura de ficheros <abbr title="Comma-Separated Values">CSV</abbr>.</li>
          <li>Administrar equipos con Windows y Linux.</li>
        </ul>
      </section>

      <section>
        <h2>Mi forma de trabajar</h2>
        <ol>
          <li>Entender el problema antes de escribir código.</li>
          <li>Hacer una primera versión que funcione.</li>
          <li>Revisarla con el validador y mejorarla.</li>
        </ol>
      </section>
    </div>

    <section>
      <h2>Formación</h2>
      <div class="tabla-scroll">
        <table>
          <caption>Estudios de Ana Martín</caption>
          <thead>
            <tr>
              <th scope="col">Estudios</th>
              <th scope="col">Periodo</th>
              <th scope="col">Estado</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>Desarrollo de Aplicaciones Web</td>
              <td>2026–2028</td>
              <td>En curso</td>
            </tr>
            <tr>
              <td>Sistemas Microinformáticos y Redes</td>
              <td>2024–2026</td>
              <td>Finalizado</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </main>

  <footer>
    <p><small>© 2026 Ana Martín</small> · <a href="mailto:ana.martin@example.com">ana.martin@example.com</a></p>
  </footer>
</body>
</html>
```

Y la versión final de `css/estilos.css`. Parte de la hoja del apartado 2.8 y añade la regla para las imágenes, el menú y las columnas con Flexbox en su disposición «de móvil» y, al final, la consulta de medios que los reorganiza en pantallas anchas:

```css
/* estilos.css · hoja de estilo del portfolio · UD3, versión final (disposición y adaptación al móvil) */

* {
  box-sizing: border-box;
}

body {
  font-family: system-ui, "Segoe UI", Roboto, Arial, sans-serif;
  font-size: 1.125rem;
  line-height: 1.6;
  color: #1f2933;
  background-color: #f8fafc;
  max-width: 48rem;
  margin: 0 auto;
  padding: 1rem;
}

h1, h2 {
  color: #1e3a8a;
  line-height: 1.2;
}

h1 {
  font-size: 2.25rem;
}

h2 {
  font-size: 1.5rem;
}

a {
  color: #1d4ed8;
}

a[aria-current="page"] {
  font-weight: 700;
  text-decoration: none;
  color: #1f2933;
}

a:hover,
a:focus {
  color: #1e3a8a;
  text-decoration-thickness: 3px;
}

/* Las imágenes nunca son más anchas que su contenedor; la altura se ajusta para no deformarlas */
img {
  max-width: 100%;
  height: auto;
}

header {
  border-bottom: 2px solid #cbd5e1;
  margin-bottom: 1.5rem;
  padding-bottom: 0.5rem;
}

header p {
  font-weight: 700;
  margin: 0.5rem 0;
}

/* Menú: sin viñetas y con flexbox. Por defecto (pantalla estrecha) los enlaces van uno debajo de otro */
nav ul {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;             /* nav ul pasa a ser un contenedor flex y sus li, elementos flex */
  flex-direction: column;    /* eje principal vertical: uno debajo de otro */
  gap: 0.25rem;              /* espacio entre elementos */
}

.presentacion {
  font-size: 1.25rem;
  color: #334155;
}

/* Dos secciones: por defecto, una debajo de otra */
.columnas {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

section {
  margin-bottom: 1.5rem;
}

/* Si la tabla es más ancha que la pantalla, se desplaza dentro de su contenedor y no toda la página */
.tabla-scroll {
  overflow-x: auto;
}

table {
  border-collapse: collapse;
  width: 100%;
}

caption {
  font-weight: 700;
  text-align: left;
  margin-bottom: 0.5rem;
}

th,
td {
  border: 1px solid #94a3b8;
  padding: 0.5rem 0.75rem;
  text-align: left;
}

th {
  background-color: #e2e8f0;
}

footer {
  font-size: 0.875rem;
  color: #475569;
  border-top: 2px solid #cbd5e1;
  margin-top: 2rem;
  padding-top: 0.5rem;
}

/* Pantalla ancha: a partir de 40 rem (640 px si el base es 16) */
@media (min-width: 40rem) {
  nav ul {
    flex-direction: row;     /* eje principal horizontal: los enlaces van en fila */
    gap: 1.5rem;
  }

  .columnas {
    flex-direction: row;     /* las dos secciones, una al lado de la otra */
  }

  .columnas section {
    flex: 1;                 /* cada sección reparte a partes iguales el ancho disponible */
  }
}
```

Qué resultado debes obtener:

- **Ventana de menos de 40 rem de ancho (un móvil):** el menú son tres enlaces, uno debajo de otro; las secciones «Qué sé hacer» y «Mi forma de trabajar» van una debajo de la otra; la tabla ocupa todo el ancho.
- **Ventana de 40 rem o más (un monitor):** el menú son tres enlaces en una sola fila, separados por 1,5 rem; las dos secciones van una al lado de la otra con el mismo ancho.
- **En cualquier ancho:** la página nunca supera los 48 rem y está centrada; la imagen nunca sobrepasa a su contenedor; no aparece barra de desplazamiento horizontal en la página (se ha comprobado en anchos entre 320 y 900 px).

#### Por qué la tabla lleva un contenedor

Una tabla es el elemento que más a menudo desborda en una pantalla estrecha: sus columnas tienen un ancho mínimo (el de la palabra más larga de cada una) y, si la suma supera el ancho disponible, la tabla se sale. En la tabla de formación de Ana ocurre ya en un móvil de 390 px de ancho: la palabra «Microinformáticos» y las otras dos columnas no caben en 358 px. La solución habitual es envolver la tabla en un contenedor con `overflow-x: auto`, que significa «si el contenido es más ancho que yo, muestra una barra de desplazamiento horizontal **dentro de mí**». La tabla se desplaza dentro de su caja y la página entera no se mueve. Es lo que hace la clase `tabla-scroll`.

### 3.6 Cómo probar la adaptación

Hay dos formas rápidas de probar el comportamiento en distintos anchos sin necesidad de un móvil:

1. **Cambiar el tamaño de la ventana** del navegador y observar el momento en que cambia el diseño.
2. **La vista de dispositivo de las herramientas de desarrollo.** En Chrome, Edge y Firefox se activa con la combinación **Ctrl + Mayús + M** y permite elegir un dispositivo (o un ancho concreto) y ver la página con ese tamaño de pantalla.

Prueba tres anchos: uno de móvil (alrededor de 390 px), uno intermedio (en torno a 640 px, para ver el cambio de diseño) y uno de monitor (a partir de 1000 px). Además de comprobar que el diseño cambia, mira siempre que **no haya desplazamiento horizontal**: es el síntoma de que algo (una imagen, una tabla, una palabra larga) es más ancho que la pantalla.

> **ERRORES FRECUENTES**
>
> **Aplicar `display: flex` al elemento equivocado.** Si `nav ul` es el contenedor, los elementos flex son los `li`. Si pones `display: flex` en el `li` o en el `a`, no ocurre lo que esperabas. Pregúntate siempre: ¿quién es el contenedor y quiénes son sus hijos directos?
>
> **Olvidar el `meta viewport`.** En el ordenador todo parece correcto, pero en el móvil la página aparece diminuta y las consultas de medios no se comportan como esperas.
>
> **Llaves mal cerradas en el `@media`.** Una consulta de medios contiene reglas completas, con sus propias llaves, y la suya propia. Una llave que falta hace que el navegador descarte o malinterprete todo lo que viene después.
>
> **Poner anchos fijos en píxeles.** `width: 800px` en un contenedor provoca desplazamiento horizontal en pantallas estrechas. Usa `max-width` y porcentajes.

> **PARA SABER MÁS**
>
> Flexbox ordena elementos en **una** dimensión. Cuando necesitas controlar filas y columnas a la vez (una cuadrícula de tarjetas, un diseño de página con cabecera, barra lateral y pie), la herramienta adecuada es **CSS Grid** (`display: grid`), que no se usa en esta unidad pero merece que lo conozcas. Antes de que existieran Flexbox y Grid, las columnas se hacían con `float` o, peor, con tablas de maquetación (los atributos y elementos obsoletos que viste en la UD2): ambos mecanismos están desaconsejados para disposición.

> **COMPRUEBA LO QUE HAS ENTENDIDO**
>
> 1. En `nav ul { display: flex; }`, ¿qué elementos son los elementos flex?
>
>    **Respuesta.** Los `li`, que son los hijos directos del `ul`. Los `a` que hay dentro de los `li` no son elementos flex.
>
> 2. ¿Qué diferencia hay entre `justify-content` y `align-items`?
>
>    **Respuesta.** `justify-content` reparte los elementos a lo largo del eje principal (en una fila, horizontalmente) y `align-items` los alinea en el eje transversal (en una fila, verticalmente).
>
> 3. Explica qué hace `@media (min-width: 40rem) { … }` y por qué se escribe primero la versión para pantallas estrechas.
>
>    **Respuesta.** Aplica las reglas de su interior solo cuando la ventana mide 40 rem o más de ancho. La versión estrecha es la base porque es la disposición natural del flujo normal (todo apilado), y cada consulta añade únicamente lo que hace falta cuando hay más espacio (enfoque *mobile first*).
>
> 4. Un compañero dice que su página «se ve bien en el ordenador pero diminuta en el móvil». ¿Qué comprobarías primero?
>
>    **Respuesta.** Que el `head` incluye `<meta name="viewport" content="width=device-width, initial-scale=1">`. Sin él, el móvil dibuja la página a un ancho virtual grande y la reduce.
>
> 5. ¿Por qué se pone `max-width: 100%` y `height: auto` a las imágenes?
>
>    **Respuesta.** Con `max-width: 100%` la imagen no puede ser más ancha que su contenedor (así no provoca desplazamiento horizontal), y con `height: auto` el alto se ajusta en proporción para que no se deforme.

---

## 4. Validar HTML y CSS

`// Criterios de evaluación: 2.h`

### 4.1 Qué es validar y por qué importa

**Validar** un documento es comprobar automáticamente que cumple las reglas de su especificación. Ya validaste HTML en la UD2 con el Nu Html Checker. En esta unidad validas también CSS, y lo haces por dos motivos. El primero, práctico: un error de sintaxis en CSS no avisa en pantalla. El navegador **descarta en silencio** la declaración errónea (y, según el error, también las siguientes) y muestra la página sin ese estilo. El validador es la única forma de ver esos descartes. El segundo, profesional: un documento válido es predecible, porque todos los navegadores lo interpretan igual.

El principio que ya conoces de la UD2 sigue vigente y es especialmente importante en CSS: **válido no significa bien hecho**. Un validador comprueba que el código respeta la gramática del lenguaje, no que sea una buena decisión de diseño ni que sea accesible. Un texto rosa claro sobre fondo blanco, un orden de reglas frágil o un menú que no se entiende en el móvil son CSS válido.

### 4.2 Las dos herramientas

**Nu Html Checker** (`validator.w3.org/nu/`). Es el validador de HTML que ya usas. Admite tres formas de entrada: una dirección web, la subida de un archivo o el texto pegado directamente. Además de HTML, comprueba el **CSS escrito dentro del propio documento**: el contenido de los elementos `style` y el de los atributos `style`. No sigue ni valida los archivos `.css` externos que el documento enlaza ni comprueba que existan: eso es tuyo.

**Validador de CSS del W3C** (`jigsaw.w3.org/css-validator/`). Comprueba hojas de estilo. Admite también tres entradas: una dirección web, la subida de un archivo `.css` o el texto pegado. Esta es la herramienta para validar tu `css/estilos.css`. Tiene opciones de perfil (versión de CSS) y de tipo de aviso que puedes dejar con sus valores por defecto. Distingue **errores**, que hay que corregir, de **avisos**, que hay que valorar.

Entre los dos, el procedimiento para validar un portfolio es: (1) cada una de las tres páginas HTML en el Nu Html Checker; (2) `css/estilos.css` en el validador de CSS; (3) cualquier CSS escrito dentro de las páginas (que no debería haber, si usas hoja externa) queda cubierto por el paso 1.

> **PARA SABER MÁS**
>
> No se ha comprobado en estos apuntes cómo trata el validador de CSS del W3C las características más recientes del lenguaje, como las variables CSS o la consulta `prefers-color-scheme` del apartado 1.12. Si el validador marca como error algo que sabes que existe y funciona, contrasta con la documentación de MDN antes de cambiarlo: puede ser una limitación del validador y no un fallo tuyo.

### 4.3 Leer los mensajes de error

El Nu Html Checker presenta cada mensaje con la posición (`línea.columna`), el tipo y la explicación. Tienes aquí una hoja de estilo con tres errores típicos, y el resultado de validarla:

```css
body {
  font-family: Arial, sans-serif
  font-size: 1.125rem;
  color: #12;
}

h1 {
  colour: navy;
  margin: 1rem;
}
```

```text
"errores.css":3.3-3.11: error: CSS: “font-family”: Missing a semicolon before the property name “font-size”.
"errores.css":4.10-4.12: error: CSS: “color”: “#12” is not a valid color 3 or 6 hexadecimals numbers.
"errores.css":8.11-8.14: error: CSS: “colour”: Property “colour” doesn't exist.
```

Los tres mensajes se leen así:

- **Línea 3: «Missing a semicolon before the property name “font-size”».** Falta el punto y coma al final de la línea anterior (`font-family: Arial, sans-serif`). El navegador, sin punto y coma, lee `Arial, sans-serif font-size: 1.125rem` como un único valor, no lo entiende y descarta la declaración entera. Efecto visible: se pierden **las dos** a la vez, la tipografía y el tamaño de letra (el texto vuelve a la letra con remates y a los 16 px por defecto del navegador).
- **Línea 4: «“#12” is not a valid color».** Un color hexadecimal necesita tres o seis cifras. Efecto visible: el color se descarta y se hereda o se usa el del navegador.
- **Línea 8: «Property “colour” doesn't exist».** Un error ortográfico en el nombre de la propiedad (el inglés escribe `color`, sin «u»). Efecto visible: ninguno, porque el navegador ignora las propiedades que no conoce, y por eso el error pasa inadvertido sin validador.

El mismo motor comprueba también el CSS escrito dentro de un documento HTML. Este documento tiene los mismos tipos de error, esta vez en un elemento `style` y en un atributo `style`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>CSS interno con errores</title>
  <style>
    p {
      color: #12;
      font-size: 14;
    }
  </style>
</head>
<body>
  <p style="colour: red;">Texto</p>
</body>
</html>
```

```text
"interno.html":8.14-8.16: error: CSS: “color”: “#12” is not a valid color 3 or 6 hexadecimals numbers.
"interno.html":9.18-9.19: error: CSS: “font-size”: only “0” can be a “unit”. You must put a unit after your number.
"interno.html":14.3-14.26: error: CSS: “colour”: Property “colour” doesn't exist.
```

Fíjate en que cada mensaje empieza por «CSS:» para distinguirlo de los de HTML, y que la posición se refiere a líneas del documento HTML. El error de la línea 9, «only “0” can be a “unit”. You must put a unit after your number», es otro clásico: `font-size: 14` sin unidad. La única cantidad que se puede escribir sin unidad es el `0`; `14` debe ser `14px` o `0.875rem`.

> **ERRORES FRECUENTES**
>
> **Interpretar «0 errores» como «el CSS está bien».** Solo significa que la sintaxis es correcta. No dice nada del contraste, de la cascada, de si la hoja realmente se aplica (¿está enlazada?) ni de si se ve bien en el móvil.
>
> **Corregir de abajo arriba.** Un error de sintaxis puede provocar mensajes en cascada (el punto y coma que falta afecta a la declaración siguiente). Empieza por el primero y vuelve a validar.
>
> **Validar solo `index.html`.** Hay que validar las tres páginas y la hoja de estilo.

### 4.4 Lo que el validador no detecta

Es tan importante saber qué no detecta el validador como saber usarlo. Estas comprobaciones las haces tú, con las herramientas de desarrollo y con criterio:

| Qué ocurre | Por qué el validador no lo ve | Cómo lo compruebas |
|---|---|---|
| La hoja `estilos.css` no se carga (ruta mal escrita en `href`) | El validador HTML no comprueba que el archivo enlazado exista | Pestaña **Red** de DevTools: la petición aparece con error |
| Contraste insuficiente (texto rosa sobre blanco) | Es CSS válido | Selector de color de DevTools, o un comprobador de contraste (apartado 1.11) |
| Una regla no se aplica por culpa de la cascada | Las dos reglas son válidas; la que pierde es un problema de especificidad u orden | Panel **Estilos** de DevTools: la declaración aparece tachada (apartado 2.5) |
| El diseño se rompe en el móvil | Es CSS válido | Vista de dispositivo de DevTools (apartado 3.6) |
| Uso de `!important` o de `id` para dar estilo | Son válidos; son decisiones de mantenimiento | Revisión del código |

### 4.5 Diagnóstico cuando «no se ve nada»

Cuando escribes CSS y no ves el efecto esperado, el orden de comprobación va del más habitual al más raro:

1. ¿La página tiene el `link` a la hoja de estilo, y está en **esa** página?
2. ¿La ruta del `href` es correcta (mayúsculas, nombre exacto, carpeta)? ¿Aparece el archivo cargado en la pestaña **Red**?
3. ¿Has **guardado** el archivo y **recargado** la página? (A veces el navegador conserva una copia anterior: prueba con recarga forzada, **Ctrl + F5**.)
4. ¿Falta algún punto y coma, alguna llave o alguna comilla? Valida la hoja.
5. ¿La declaración aparece **tachada** en el panel Estilos? Entonces es un conflicto de cascada (apartado 2).
6. ¿El selector encaja con el elemento? Si la regla no aparece en el panel, el selector no está seleccionando lo que crees (una clase con el nombre mal escrito, un descendiente en la jerarquía equivocada).

> **COMPRUEBA LO QUE HAS ENTENDIDO**
>
> 1. ¿Qué hace el navegador cuando una declaración CSS tiene un error de sintaxis? ¿Por qué eso hace necesario un validador?
>
>    **Respuesta.** La descarta en silencio, sin avisar en pantalla (a veces también descarta la siguiente), y muestra la página sin ese estilo. Sin validador es fácil no enterarse del error.
>
> 2. ¿Qué dos herramientas necesitas para validar un portfolio completo y qué valida cada una?
>
>    **Respuesta.** El Nu Html Checker valida cada página HTML (y el CSS escrito dentro de ellas, en `style` y en atributos `style`). El validador de CSS del W3C valida el archivo `css/estilos.css`.
>
> 3. ¿Qué indica el mensaje «only “0” can be a “unit”» sobre `font-size: 14`? ¿Cómo se corrige?
>
>    **Respuesta.** Que falta la unidad: solo el 0 puede escribirse sin ella. Se corrige escribiendo una unidad, por ejemplo `14px` o `0.875rem`.
>
> 4. Un portfolio pasa los dos validadores sin errores pero el texto de los enlaces es ilegible. ¿Es contradictorio?
>
>    **Respuesta.** No: validar solo comprueba la sintaxis. El contraste insuficiente es CSS válido y hay que comprobarlo con el selector de color de DevTools o un comprobador, calculando la relación frente al mínimo de 4,5 : 1.
>
> 5. Has escrito una regla, recargas y no ocurre nada, y el archivo CSS no aparece en la pestaña Red. ¿Qué miras primero?
>
>    **Respuesta.** Que la página tenga el `link rel="stylesheet"` en su `head` y que la ruta del `href` coincida exactamente con el nombre y la carpeta del archivo.

---

## 5. Sindicación de contenidos: RSS, Atom y JSON Feed

`// Criterios de evaluación: 2.i, 2.j`

### 5.1 El problema que resuelve la sindicación

Imagina que sigues veinte blogs, tres medios de comunicación y los lanzamientos de varios programas que usas. Para enterarte de las novedades tendrías que visitar las veinticuatro páginas, una por una, cada día, y fijarte en qué ha cambiado. La **sindicación de contenidos** (*content syndication*) resuelve esto: el sitio publica la lista de sus novedades en un archivo con un formato estándar que un programa puede leer, y una persona o aplicación se suscribe a ese archivo y se entera de lo nuevo sin visitar el sitio. El término está tomado del mundo de la prensa, donde un artículo o una columna se «sindica» cuando se distribuye a muchas publicaciones a la vez.

Hay tres piezas, con nombres que debes conocer:

- El **canal** o **feed** (en inglés, *feed* o *web feed*): el archivo publicado por el sitio, con la lista de entradas más recientes. Tiene una URL propia, como cualquier otro recurso web.
- El **agregador** o **lector de feeds** (*feed reader*): el programa o servicio (una aplicación, una extensión del navegador, un servicio en línea) que se suscribe a uno o varios feeds y muestra las novedades de todos juntos.
- La **suscripción**: la acción de dar al lector la URL del feed. El lector consulta ese archivo cada cierto tiempo y muestra las entradas que no había visto.

Es importante entender el mecanismo: **el lector va a mirar el archivo, no es el sitio quien avisa**. El feed es un archivo estático (o generado por el sitio) que se descarga con una petición HTTP normal. Por eso la sindicación no depende de ninguna plataforma: cualquiera puede publicar un feed y cualquiera puede escribir un lector.

### 5.2 Ámbitos de aplicación

La sindicación se usa en todos los lugares en los que alguien publica una secuencia de novedades que otros quieren seguir. Reconocer estos ámbitos es el contenido del criterio de evaluación 2.j:

| Ámbito | Qué publica el feed | Quién lo consume |
|---|---|---|
| Blogs y medios de comunicación | Cada artículo o noticia nueva, con título, enlace, fecha y resumen o texto completo | Lectores de feeds personales; servicios que agrupan noticias |
| Podcasts | Cada episodio nuevo, con un enlace al archivo de audio | Aplicaciones de podcasts, que descargan los episodios cuando aparecen |
| Seguimiento de software | Nuevas versiones o cambios de un proyecto. Algunos servicios de alojamiento de código ofrecen feeds de lanzamientos o de cambios | Desarrolladores que quieren enterarse de una actualización |
| Agregación de noticias | Titulares de muchas fuentes reunidos en una sola vista | Portales de noticias, paneles de información |
| Avisos e integración entre sistemas | Alertas de un servicio, novedades de una intranet, publicaciones de una institución | Otros programas que reaccionan al contenido nuevo, o personas que no quieren revisar la web a mano |

Los podcasts merecen una mención especial, porque son uno de los usos más extendidos: **un podcast es, técnicamente, un feed RSS** en el que cada entrada incluye, además del título y la descripción, un enlace al archivo de audio (en RSS, mediante el elemento `enclosure`, apartado 5.4).

Como límites a tener en cuenta: un feed no «empuja» notificaciones por sí mismo (el lector decide cada cuánto consulta), y puede ofrecer el contenido completo de cada entrada o solo un resumen con enlace al artículo original; es el publicador quien lo decide.

### 5.3 Tecnologías: XML, RSS, Atom y JSON Feed

Los feeds se escriben con un formato estándar que el lector sabe interpretar. Hay tres que debes distinguir (criterio 2.i):

- **RSS** es la familia de formatos de feed más antigua y extendida. Tiene varias ramas históricas; la más usada, y la que se estudia aquí, es **RSS 2.0**, mantenida por el RSS Advisory Board. La versión vigente de su especificación es la 2.0.11, publicada el 30 de marzo de 2009; el propio documento indica que el formato está en la práctica «congelado» y que las revisiones posteriores solo aclaran el texto. RSS 2.0 suele expandirse como *Really Simple Syndication*.
- **Atom** es un formato alternativo con las mismas funciones, estandarizado por el IETF (la organización que mantiene los estándares de internet) como **RFC 4287**, en diciembre de 2005, con la categoría *Standards Track*. Se diseñó para ofrecer una definición precisa y sin ambigüedades.
- **JSON Feed** es un formato más reciente, publicado como especificación independiente (no es un estándar de una organización como el IETF o el W3C). Su versión vigente es la 1.1, de agosto de 2020. Hace lo mismo que RSS y Atom, pero está escrito en **JSON** en lugar de XML.

RSS y Atom son **lenguajes de marcas basados en XML**: lo que aprendiste en la UD1 se aplica íntegramente. Un feed RSS o Atom es un documento XML, así que tiene que estar **bien formado** (un único elemento raíz, etiquetas cerradas y bien anidadas, atributos entre comillas, caracteres reservados escapados) y, en el caso de Atom, usa un **espacio de nombres** (UD1) para identificar sus elementos.

### 5.4 RSS 2.0

Un documento RSS 2.0 tiene esta estructura: un elemento raíz `rss` con el atributo `version="2.0"`; dentro, **un único** elemento `channel` que describe el canal y contiene tantos elementos `item` como entradas haya. Es un ejemplo completo y bien formado de feed para el portfolio de Ana, con dos novedades:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>Novedades del portfolio de Ana Martín</title>
    <link>https://example.org/ana/</link>
    <description>Cambios y nuevas páginas del portfolio de Ana Martín.</description>
    <language>es</language>
    <lastBuildDate>Wed, 25 Nov 2026 09:30:00 GMT</lastBuildDate>
    <atom:link href="https://example.org/ana/feed.xml" rel="self" type="application/rss+xml"/>
    <item>
      <title>El portfolio ya tiene estilos</title>
      <link>https://example.org/ana/novedades/estilos.html</link>
      <guid isPermaLink="true">https://example.org/ana/novedades/estilos.html</guid>
      <pubDate>Wed, 25 Nov 2026 09:30:00 GMT</pubDate>
      <description>Las tres páginas comparten ya una hoja de estilo externa y se adaptan al móvil.</description>
    </item>
    <item>
      <title>Nueva página: contacto</title>
      <link>https://example.org/ana/contacto.html</link>
      <guid isPermaLink="true">https://example.org/ana/contacto.html</guid>
      <pubDate>Thu, 29 Oct 2026 18:00:00 GMT</pubDate>
      <description>Añadida la página de contacto con un formulario que envía los datos por POST.</description>
    </item>
  </channel>
</rss>
```

Cada elemento significa lo siguiente:

| Elemento | Dónde | Qué es | ¿Obligatorio? |
|---|---|---|---|
| `rss` | raíz | Contiene todo el documento. El atributo `version` debe valer `2.0` | Sí |
| `channel` | dentro de `rss` | Describe el canal. Hay uno solo | Sí |
| `title` | `channel` | Nombre del canal | Sí |
| `link` | `channel` | URL del sitio web al que corresponde el canal | Sí |
| `description` | `channel` | Frase que describe el canal | Sí |
| `language`, `lastBuildDate` | `channel` | Idioma y fecha del último cambio del contenido | No |
| `item` | `channel` | Una entrada del canal. Hay tantos como se quiera | No (un canal puede estar vacío) |
| `title`, `link`, `description` | `item` | Título, URL y resumen (o texto) de la entrada | Todos son opcionales, pero **debe haber al menos `title` o `description`** |
| `guid` | `item` | Identificador único de la entrada | No, pero muy recomendable |
| `pubDate` | `item` | Fecha de publicación | No |
| `enclosure` | `item` | Archivo adjunto (audio, vídeo…) | No |

Algunos detalles que conviene entender:

- **Fechas.** RSS usa el formato de fecha del estándar RFC 822 (con año de cuatro cifras): `Wed, 25 Nov 2026 09:30:00 GMT`, es decir, día de la semana abreviado en inglés, día, mes abreviado en inglés, año, hora y zona. Se escribe en inglés aunque el feed esté en español.
- **`guid`.** Es el identificador de la entrada. El lector lo usa para saber si una entrada ya la había visto, así que **no debe cambiar nunca** una vez publicada. Su atributo `isPermaLink` vale `true` por defecto, lo que indica que el identificador es una URL permanente; si pones otro tipo de identificador, indica `isPermaLink="false"`.
- **`enclosure`.** Es el elemento que hace posible los podcasts. Tiene tres atributos obligatorios: `url` (de dónde descargar el archivo), `length` (su tamaño en bytes) y `type` (su tipo MIME). Por ejemplo, una entrada de un episodio de audio incluiría: `<enclosure url="https://example.org/ana/audio/episodio1.mp3" length="12216320" type="audio/mpeg"/>`.
- **`xmlns:atom` y `atom:link`.** El ejemplo declara en la raíz el espacio de nombres de Atom (`xmlns:atom="http://www.w3.org/2005/Atom"`) para poder usar su elemento `link` con `rel="self"` y decir cuál es la URL del propio feed. La propia especificación de RSS permite extensiones con espacios de nombres, y es una práctica habitual. Es el mismo mecanismo de prefijos que viste en la UD1.
- **Texto con HTML.** Si en el texto de una entrada quieres incluir marcado HTML (un enlace, una cursiva), tiene que ir **escapado** (`&lt;a href="…"&gt;`) o dentro de una sección `CDATA` (`<![CDATA[ … ]]>`), que es un bloque que el parser XML trata como texto literal sin interpretar su contenido. De lo contrario, esas etiquetas serían parte del XML del feed.

### 5.5 Atom

Atom resuelve lo mismo con otra estructura: el elemento raíz es `feed` (no hay `channel`) y las entradas son elementos `entry`. Está escrito en el **espacio de nombres** `http://www.w3.org/2005/Atom`, que se declara en la raíz como espacio por defecto. El mismo feed de Ana en Atom:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title>Novedades del portfolio de Ana Martín</title>
  <link href="https://example.org/ana/atom.xml" rel="self" type="application/atom+xml"/>
  <link href="https://example.org/ana/" rel="alternate" type="text/html"/>
  <updated>2026-11-25T09:30:00Z</updated>
  <id>https://example.org/ana/</id>
  <author>
    <name>Ana Martín</name>
  </author>
  <entry>
    <title>El portfolio ya tiene estilos</title>
    <link href="https://example.org/ana/novedades/estilos.html" rel="alternate" type="text/html"/>
    <id>https://example.org/ana/novedades/estilos.html</id>
    <updated>2026-11-25T09:30:00Z</updated>
    <summary>Las tres páginas comparten ya una hoja de estilo externa y se adaptan al móvil.</summary>
  </entry>
  <entry>
    <title>Nueva página: contacto</title>
    <link href="https://example.org/ana/contacto.html" rel="alternate" type="text/html"/>
    <id>https://example.org/ana/contacto.html</id>
    <updated>2026-10-29T18:00:00Z</updated>
    <summary>Añadida la página de contacto con un formulario que envía los datos por POST.</summary>
  </entry>
</feed>
```

Los elementos obligatorios son:

- En `feed`: `id`, `title` y `updated`. Además, `author` (al menos uno) salvo que **todas** las entradas lleven su propio autor.
- En cada `entry`: `id`, `title` y `updated`. Además, si la entrada no incluye un elemento `content` con su texto, debe tener al menos un `link` con `rel="alternate"` que apunte al contenido.

Diferencias importantes con RSS:

- **`id`.** Es un identificador permanente y único, escrito como una URL (o más exactamente un IRI). La especificación exige que **no cambie nunca**, aunque el feed se mueva de sitio o se republique. Se compara carácter a carácter y distingue mayúsculas de minúsculas. No hace falta que sea una dirección que funcione: puede ser solo un identificador.
- **Fechas.** Atom usa el formato **RFC 3339**, que es una variante del ISO 8601: `2026-11-25T09:30:00Z` (año, mes, día, `T`, hora, y `Z` para la zona UTC o un desplazamiento como `+01:00`).
- **`link`.** Es un elemento vacío con atributos: `href` (obligatorio, la URL), `rel` (la relación: `alternate` es la versión alternativa de la entrada, normalmente su página web, y es el valor por defecto si se omite; `self` es el propio feed) y `type` (el tipo MIME esperado).
- **Tipo de texto.** `title`, `summary` y `content` admiten el atributo `type` con los valores `text` (por defecto), `html` (HTML escapado) o `xhtml`, de modo que el lector sabe cómo interpretar su contenido. En RSS esa información no está definida.

### 5.6 JSON Feed

JSON Feed expresa la misma información como un objeto JSON. El mismo feed de Ana, escrito como JSON Feed 1.1:

```json
{
  "version": "https://jsonfeed.org/version/1.1",
  "title": "Novedades del portfolio de Ana Martín",
  "home_page_url": "https://example.org/ana/",
  "feed_url": "https://example.org/ana/feed.json",
  "language": "es",
  "authors": [
    { "name": "Ana Martín" }
  ],
  "items": [
    {
      "id": "https://example.org/ana/novedades/estilos.html",
      "url": "https://example.org/ana/novedades/estilos.html",
      "title": "El portfolio ya tiene estilos",
      "summary": "Las tres páginas comparten ya una hoja de estilo externa y se adaptan al móvil.",
      "content_text": "Las tres páginas comparten ya una hoja de estilo externa y se adaptan al móvil.",
      "date_published": "2026-11-25T09:30:00Z"
    },
    {
      "id": "https://example.org/ana/contacto.html",
      "url": "https://example.org/ana/contacto.html",
      "title": "Nueva página: contacto",
      "content_text": "Añadida la página de contacto con un formulario que envía los datos por POST.",
      "date_published": "2026-10-29T18:00:00Z"
    }
  ]
}
```

- `version` es la URL que identifica la versión de la especificación (`https://jsonfeed.org/version/1.1`), no un número.
- Los campos obligatorios en el nivel superior son `version`, `title` e `items`. `home_page_url` y `feed_url` son opcionales en general, pero la especificación los considera necesarios en un feed publicado en la web. `authors` es una lista de objetos con `name`, `url` y `avatar`.
- Cada elemento de `items` necesita un `id`, que es una cadena única y estable. Y debe tener `content_html` o `content_text` (o ambos): es el texto de la entrada. El HTML solo se permite en `content_html`.
- La fecha de publicación (`date_published`) usa el mismo formato RFC 3339 que Atom.
- El tipo de medio indicado para servirlo es `application/feed+json`.

### 5.7 Comparación y criterio de elección

| | RSS 2.0 | Atom 1.0 | JSON Feed 1.1 |
|---|---|---|---|
| Sintaxis | XML | XML con espacio de nombres | JSON |
| Elemento raíz | `rss` → `channel` | `feed` | objeto con `items` |
| Entrada | `item` | `entry` | cada objeto de `items` |
| Identificador | `guid` (opcional) | `id` (obligatorio) | `id` (obligatorio) |
| Fecha | RFC 822 (`pubDate`) | RFC 3339 (`updated`) | RFC 3339 (`date_published`) |
| Tipo de medio | `application/rss+xml` | `application/atom+xml` | `application/feed+json` |
| Quién lo define | RSS Advisory Board (2.0.11, 2009) | IETF, RFC 4287 (2005) | Especificación independiente (1.1, 2020) |
| Sirve para podcasts | Sí, es el formato habitual (`enclosure`) | Puede referenciar archivos con `link rel="enclosure"` | Sí, con `attachments` |

> **ALTERNATIVAS Y CRITERIO**
>
> Los tres formatos cumplen la función de sindicar. Para un criterio de elección profesional: **RSS 2.0** es el que prácticamente cualquier lector acepta y el que esperan los directorios de podcasts, así que es la opción segura para llegar al mayor número de consumidores. **Atom** se prefiere cuando se necesita una definición más rigurosa del modelo (identificador obligatorio, fechas obligatorias, tipo de contenido declarado), por ejemplo para integraciones entre sistemas. **JSON Feed** es cómodo cuando el consumidor es una aplicación escrita en JavaScript o con herramientas JSON, y se ahorra el análisis de XML; su adopción es más limitada que la de los otros dos. Muchos sitios publican dos formatos a la vez. Esto es una recomendación de uso, no una norma.

### 5.8 Autodescubrimiento: cómo sabe un lector dónde está tu feed

Publicar el feed no basta: tiene que poder encontrarse. El mecanismo estándar se llama **autodescubrimiento** (*autodiscovery*): en el `head` de la página se añade un `link` que anuncia la URL del feed. Un lector o una extensión del navegador que reciba la dirección de la web busca ese `link` y obtiene la dirección del feed sin que la persona tenga que saberla.

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Inicio · Portfolio de Ana Martín</title>
  <link rel="stylesheet" href="css/estilos.css">
  <link rel="alternate" type="application/rss+xml" title="Novedades (RSS)" href="feed.xml">
  <link rel="alternate" type="application/atom+xml" title="Novedades (Atom)" href="atom.xml">
</head>
<body>
  <h1>Ana Martín</h1>
</body>
</html>
```

Los atributos del `link` son:

- `rel="alternate"`: indica que se trata de una versión alternativa del contenido de la página (en minúsculas y sin otras palabras clave).
- `type`: identifica el formato. `application/rss+xml` para RSS, `application/atom+xml` para Atom y `application/feed+json` para JSON Feed.
- `title`: un nombre legible que el lector puede mostrar a la persona. Es opcional.
- `href`: la URL del feed (se recomienda que sea completa, no relativa).

Para el feed principal se recomienda un solo `link` de autodescubrimiento por página. Si publicas el mismo contenido en dos formatos, puedes enlazar los dos, como en el ejemplo.

### 5.9 Comprobar que un feed es correcto

Un feed tiene que superar tres comprobaciones, que van de la más básica a la más completa:

1. **Que esté bien formado como XML** (los de RSS y Atom). Un solo error, como una `&` sin escapar, hace que cualquier lector lo rechace por completo. Es el mismo criterio que aplicaste en la UD1. Se puede comprobar abriendo el archivo en un navegador, que muestra el error si el XML está mal, o con la herramienta `xmllint` en la línea de comandos. Este feed tiene un `&` sin escapar en la descripción:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
  <channel>
    <title>Novedades</title>
    <link>https://example.org/ana/</link>
    <description>Preguntas & respuestas del portfolio.</description>
  </channel>
</rss>
```

```text
feed_mal.xml:6: parser error : xmlParseEntityRef: no name
    <description>Preguntas & respuestas del portfolio.</description>
                            ^
```

El mensaje señala con un `^` el punto exacto: el carácter `&` está reservado en XML (inicia una entidad) y debe escribirse `&amp;`. Escrito así, el feed pasa a estar bien formado.

2. **Que cumpla la especificación del formato**: que estén todos los elementos obligatorios, con el formato de fecha correcto. Existe un servicio de validación de feeds del W3C (`validator.w3.org/feed/`); comprueba que está disponible en el momento en que lo uses. Un feed bien formado puede seguir siendo inválido, por ejemplo, un `channel` sin `description` o una fecha en formato ISO dentro de un RSS.
3. **Que un lector lo muestre bien.** Es la prueba final: suscríbete a tu propio feed con un lector real y comprueba los títulos, los enlaces y las fechas.

> **ERRORES FRECUENTES**
>
> **Cambiar el `guid` o el `id` de una entrada ya publicada.** El lector lo interpreta como una entrada nueva y la muestra duplicada.
>
> **Escribir las fechas de RSS en formato ISO** (`2026-11-25`) o las de Atom en formato RFC 822. Cada formato tiene la suya.
>
> **Poner HTML sin escapar dentro del texto de una entrada.** O se escapa o va en `CDATA`; de lo contrario, el XML deja de estar bien formado.
>
> **Olvidar el espacio de nombres de Atom.** Sin `xmlns="http://www.w3.org/2005/Atom"` en la raíz, los elementos no son de Atom y los lectores no los reconocen.

> **COMPRUEBA LO QUE HAS ENTENDIDO**
>
> 1. Define sindicación de contenidos y nombra las tres piezas que intervienen.
>
>    **Respuesta.** Publicar el contenido de un sitio en un formato estándar legible por programas para que otros lo consuman. Intervienen el canal o feed (el archivo publicado), el agregador o lector de feeds (el programa que lo consulta) y la suscripción (la acción de dar la URL del feed al lector).
>
> 2. ¿Quién avisa cuando hay una entrada nueva: el sitio o el lector? ¿Por qué importa?
>
>    **Respuesta.** Ninguno «avisa» en sentido estricto: el lector consulta periódicamente el feed y muestra las entradas nuevas. Importa porque el feed es un archivo normal descargable por HTTP, sin depender de ninguna plataforma.
>
> 3. Indica el elemento raíz y el elemento de entrada de RSS 2.0, de Atom y de JSON Feed.
>
>    **Respuesta.** RSS 2.0: raíz `rss` con un `channel` que contiene elementos `item`. Atom: raíz `feed` con elementos `entry`. JSON Feed: un objeto JSON cuyo campo `items` es una lista de objetos, uno por entrada.
>
> 4. ¿Qué formato de fecha usa RSS y cuál Atom? Da un ejemplo de cada uno.
>
>    **Respuesta.** RSS usa RFC 822: `Wed, 25 Nov 2026 09:30:00 GMT`. Atom usa RFC 3339: `2026-11-25T09:30:00Z`.
>
> 5. ¿Por qué un podcast es un feed RSS y qué elemento lo hace posible?
>
>    **Respuesta.** Porque cada episodio es una entrada (`item`) del canal que, además de título y descripción, incluye un `enclosure` con la `url` del archivo de audio, su tamaño en bytes (`length`) y su tipo MIME (`type`).
>
> 6. ¿Cómo anuncia una página web que tiene un feed RSS?
>
>    **Respuesta.** Con un `link` en el `head`: `<link rel="alternate" type="application/rss+xml" title="…" href="…">`. Es el autodescubrimiento.

---

## 6. Auditar un documento web y preparar la entrega del portfolio

`// Criterios de evaluación: 2.g, 2.h`

### 6.1 Qué es una auditoría

Una **auditoría** consiste en analizar un artefacto (un fragmento de código, un documento, un texto técnico) que alguien te entrega, y emitir un **juicio técnico razonado** sobre él: qué está mal, qué está bien y qué cambiarías, ordenado por importancia. El artefacto puede haberlo escrito una persona o una herramienta de IA; da igual, el análisis es el mismo. Es una competencia profesional central: revisar código ajeno, el de un compañero o el que ha generado una IA, y saber decir por qué se puede aceptar o no.

El resultado es un **informe en forma de tabla** con tres niveles de gravedad, **Crítico**, **Importante** y **Menor**, y para cada hallazgo cuatro datos: qué has observado, si es correcto o incorrecto, por qué (citando la **regla concreta**, no un simple «está mal») y qué cambiarías. Dos principios guían un buen informe:

- **Concluir que algo es correcto también es un resultado válido.** El informe no es una lista de defectos: si lo que parece sospechoso está bien, dilo y explica por qué.
- **Razonar importa más que contar.** Detectar menos errores pero explicarlos bien vale más que listar muchos sin justificar.

Como criterio orientativo para asignar la gravedad de un hallazgo en un documento web:

| Gravedad | Criterio | Ejemplos |
|---|---|---|
| **Crítico** | Impide que el contenido se vea, se lea o se use | Una hoja de estilo que no llega a cargarse; un texto con un contraste tan bajo que es ilegible |
| **Importante** | Incumple el estándar o la accesibilidad, pero el contenido sigue siendo utilizable | Un error de validación; contraste que no llega al mínimo; falta del `meta viewport` |
| **Menor** | Afecta al mantenimiento o a las buenas prácticas, sin consecuencia visible inmediata | `!important` innecesario; `id` en lugar de clase para dar estilo; tamaños en píxeles |

### 6.2 Un ejemplo completo: auditar una página con HTML y CSS

Este es el artefacto: una página de una tienda ficticia de plantas, con su hoja de estilo. Léelo antes de seguir, e intenta hacer tu propio listado de problemas.

`index.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>Verde y Hogar</title>
  <link rel="stylesheet" href="css/estilo.css">
</head>
<body>
  <center><h1>Verde y Hogar</h1></center>
  <p id="oferta">Hoy, 20 % de descuento en suculentas.</p>
  <img src="img/suculenta.jpg">
  <p class="precio">Desde 4,50 €</p>
</body>
</html>
```

`css/estilos.css`:

```css
body {
  font-family: Arial, sans-serif;
  font-size: 14;
  color: #ff6666;
  background-color: #ffffff;
}

#oferta {
  color: #ffaaaa !important;
  font-size: 12px;
}

.precio {
  color: #1f2933
  font-weight: 700;
}
```

**Paso 1: pasar los validadores.** El Nu Html Checker da este resultado para `index.html`:

```text
"index.html":9.3-9.10: error: The “center” element is obsolete. Use CSS instead.
"index.html":11.3-11.31: error: An “img” element must have an “alt” attribute, except under certain conditions. For details, consult guidance on providing text alternatives for images.
```

Y la hoja de estilo `css/estilos.css`, comprobada aquí con el Nu Html Checker en modo CSS (también puedes pasarla por el validador de CSS del W3C):

```text
"css/estilos.css":3.14-3.15: error: CSS: “font-size”: only “0” can be a “unit”. You must put a unit after your number.
"css/estilos.css":15.3-15.13: error: CSS: “color”: Missing a semicolon before the property name “font-weight”.
```

**Paso 2: lo que los validadores no ven.** Con las herramientas de desarrollo y con criterio se comprueban cuatro cosas más. La primera: el `link` del `head` apunta a `css/estilo.css`, pero el archivo se llama `css/estilos.css`; en la pestaña **Red** la petición falla y la página se ve con los estilos del navegador. La segunda: el contraste. El texto del `body` es `#ff6666` sobre `#ffffff` (2,9 : 1) y el de `#oferta` es `#ffaaaa` sobre `#ffffff` (1,8 : 1). La tercera: el `head` no tiene `meta viewport`. La cuarta: `#oferta` usa un selector de identificador y un `!important`.

**Paso 3: el informe.** Ordenado por gravedad, con la regla que justifica cada hallazgo:

| Gravedad | Qué he observado | Por qué | Qué cambiaría |
|---|---|---|---|
| Crítico | El `link` apunta a `css/estilo.css` y el archivo es `css/estilos.css` | Una ruta relativa debe coincidir exactamente con el nombre del archivo (apartado 1.5). Al no cargarse la hoja, no se aplica ninguna regla. El validador no lo detecta; se ve en la pestaña Red | `href="css/estilos.css"` |
| Crítico | El texto de `#oferta` es `#ffaaaa` sobre blanco: contraste 1,8 : 1 | El mínimo para texto normal es 4,5 : 1 (WCAG 2.2, 1.4.3). A 1,8 : 1 el texto es prácticamente ilegible | Un color oscuro, como `#b91c1c` (6,5 : 1) |
| Importante | El resto del texto es `#ff6666` sobre blanco: contraste 2,9 : 1 | No llega a 4,5 : 1. El contenido se puede leer con esfuerzo, pero incumple la accesibilidad | `#1f2933` (14,8 : 1) |
| Importante | `<img>` sin atributo `alt` | El validador lo marca como error: toda imagen necesita texto alternativo (UD2). Sin él, un lector de pantalla no puede describirla | `alt="Suculenta en una maceta de barro"` |
| Importante | `<center>` | Elemento obsoleto (el validador lo marca como error): el aspecto es cosa de CSS, no de HTML | Quitar `center` y centrar el título con `text-align: center` |
| Importante | `font-size: 14` en `body`, sin unidad | Solo el `0` puede escribirse sin unidad. La declaración se descarta y el cuerpo usa el tamaño por defecto | `font-size: 1rem` |
| Importante | Falta `;` tras `color: #1f2933` en `.precio` | El navegador lee `#1f2933 font-weight: 700` como un solo valor inválido y descarta **toda** la declaración: se pierden el color y la negrita del precio | Añadir el `;` |
| Importante | Sin `meta viewport` en el `head` | Sin él, el móvil dibuja la página a un ancho virtual grande y la reduce (apartado 3.4) | Añadir `<meta name="viewport" content="width=device-width, initial-scale=1">` |
| Menor | `#oferta` usa un selector de identificador y `!important` | Un `id` (1-0-0) y `!important` dificultan sobrescribir la regla (apartado 2.3 y 2.4) | Usar una clase (`.oferta`) y quitar `!important` |
| Menor | `font-size: 12px` en `#oferta` | Los píxeles no respetan el tamaño de letra que la persona ha elegido en su navegador (apartado 1.6) | `font-size: 0.875rem` |
| Correcto | `<!DOCTYPE html>`, `lang="es"`, `charset` y `title` | Son los elementos mínimos del esqueleto y están bien | Nada |

La versión corregida, que pasa los dos validadores sin errores:

`index.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Verde y Hogar</title>
  <link rel="stylesheet" href="css/estilos.css">
</head>
<body>
  <h1>Verde y Hogar</h1>
  <p class="oferta">Hoy, 20 % de descuento en suculentas.</p>
  <img src="img/suculenta.jpg" alt="Suculenta en una maceta de barro" width="320" height="240">
  <p class="precio">Desde 4,50 €</p>
</body>
</html>
```

`css/estilos.css`:

```css
body {
  font-family: Arial, sans-serif;
  font-size: 1rem;
  color: #1f2933;
  background-color: #ffffff;
}

h1 {
  text-align: center;
}

.oferta {
  color: #b91c1c;
  font-size: 0.875rem;
}

.precio {
  color: #1f2933;
  font-weight: 700;
}

img {
  max-width: 100%;
  height: auto;
}
```

> **ERRORES FRECUENTES**
>
> **Limitarse a copiar la salida del validador.** El validador cubre una parte de los hallazgos. Los dos críticos de este ejemplo (la ruta de la hoja de estilo y el contraste) no los detecta.
>
> **Poner todo como «crítico».** Una auditoría cuyo informe no distingue la gravedad no ayuda a decidir qué arreglar primero. Si todo es crítico, nada lo es.
>
> **Escribir «está mal» sin la regla.** Un hallazgo sin su porqué (la regla concreta, la especificación o el criterio de los apuntes que lo justifica) no puntúa completo.

### 6.3 Preparar la entrega del portfolio con CSS

La entrega del portfolio con CSS incluye las tres páginas y la hoja de estilo, junto con una defensa oral breve. La forma de entrega y la fecha las anuncia la persona que da el módulo. Antes de entregar, comprueba esta lista:

1. Las **tres páginas** enlazan `css/estilos.css` con la ruta relativa correcta, y se ven igual de coherentes.
2. Las tres páginas pasan el **Nu Html Checker con 0 errores**, y la hoja de estilo pasa el **validador de CSS** sin errores.
3. La hoja de estilo tiene la base en `body` (tipografía, tamaño, altura de línea, colores), color de títulos y enlaces con `:hover` y `:focus`, la página actual diferenciada con `[aria-current="page"]` y al menos una **clase propia**.
4. Los **colores de texto y de fondo** tienen un contraste de al menos 4,5 : 1 (texto normal), comprobado.
5. Hay **espacios** (márgenes y rellenos) razonables y `box-sizing: border-box`.
6. La página **se adapta al móvil**: tiene `meta viewport`, no hay desplazamiento horizontal y la disposición cambia con una consulta de medios.
7. Los **nombres de archivo y carpeta** no tienen espacios ni acentos y coinciden exactamente con las rutas de los `href` y los `src`.
8. El código está **comentado** lo suficiente como para explicar tus decisiones.

La defensa es una conversación sobre **tus decisiones**, no una recitación. Las preguntas que conviene saber contestar sobre tu propio portfolio son del tipo: por qué has elegido esos colores y cuánto contraste tienen; por qué un tamaño en `rem` y no en píxeles; qué selector gana si dos reglas coinciden y cómo lo has comprobado; qué hace `box-sizing: border-box` y por qué lo has puesto; cómo has probado la versión móvil; qué has cambiado respecto a una primera versión; y qué parte hiciste con ayuda de una herramienta, cómo la revisaste y qué has tenido que corregir. Si no puedes explicar algo de tu portfolio, ese algo no cuenta: revisa cada regla antes de entregar y asegúrate de que sabes qué hace.

> **COMPRUEBA LO QUE HAS ENTENDIDO**
>
> 1. ¿Qué cuatro datos debe tener cada hallazgo de un informe de auditoría?
>
>    **Respuesta.** Qué has observado, si es correcto o incorrecto, por qué (con la regla concreta) y qué cambiarías; todo ordenado por gravedad: crítico, importante, menor.
>
> 2. En el ejemplo, ¿por qué el `link` a `estilo.css` es crítico y el `font-size: 12px` es menor?
>
>    **Respuesta.** Con el `link` roto no se carga la hoja y no se aplica ninguna regla: la página se muestra sin estilo, así que impide el uso previsto. El `font-size: 12px` es un problema de buenas prácticas (no respeta la preferencia de tamaño de letra) sin consecuencia grave inmediata.
>
> 3. Un compañero entrega un informe que dice solo «el CSS está mal». ¿Qué le falta?
>
>    **Respuesta.** Qué elemento concreto falla, por qué (la regla o el criterio que incumple), su gravedad y qué cambiaría. Un «está mal» sin justificación no puntúa completo.
>
> 4. ¿Es válido concluir en una auditoría que un elemento sospechoso está bien?
>
>    **Respuesta.** Sí. Concluir que algo es correcto, razonándolo, es un resultado tan válido como detectar un error.
>
> 5. Nombra tres comprobaciones de la lista de entrega que el validador no puede hacer por ti.
>
>    **Respuesta.** El contraste de los colores, que la hoja de estilo esté realmente enlazada con la ruta correcta en las tres páginas, y que la página se vea bien y sin desplazamiento horizontal en un móvil.

---

## Glosario

- **Agregador (lector de feeds):** programa o servicio que se suscribe a feeds y muestra juntas las novedades de todos.
- **Atom:** formato XML de feed estandarizado por el IETF (RFC 4287). Raíz `feed`, entradas `entry`.
- **Auditoría:** análisis razonado de un artefacto, que emite un informe de hallazgos ordenados por gravedad.
- **Autodescubrimiento:** técnica por la que una página anuncia dónde está su feed con `<link rel="alternate" type="…" href="…">`.
- **Bloque de declaraciones:** parte de una regla CSS entre llaves, con las declaraciones.
- **box-sizing:** propiedad que define si el `width` incluye el relleno y el borde (`border-box`) o solo el contenido (`content-box`).
- **Cascada:** algoritmo con el que el navegador decide qué declaración gana cuando varias afectan a la misma propiedad de un elemento: origen e importancia, especificidad y orden.
- **CDATA:** sección de un documento XML cuyo contenido se trata como texto literal, sin interpretar etiquetas.
- **Canal (feed):** archivo publicado por un sitio con la lista de sus novedades en un formato estándar.
- **Clase:** valor del atributo `class` que permite dar estilo a varios elementos; su selector empieza por un punto (`.nombre`).
- **Consulta de medios (`@media`):** regla CSS cuyo contenido solo se aplica si se cumple una condición, como un ancho mínimo de ventana.
- **Contraste:** relación de luminosidad entre el color del texto y el del fondo; WCAG exige 4,5 : 1 para texto normal.
- **CSS:** *Cascading Style Sheets*; lenguaje que describe la presentación de un documento.
- **Declaración:** pareja `propiedad: valor;` dentro de una regla.
- **display:** propiedad que determina el tipo de caja de un elemento: `block`, `inline`, `inline-block`, `flex`…
- **Elemento flex:** hijo directo de un contenedor flex.
- **enclosure:** elemento de RSS que adjunta un archivo (audio, vídeo) a una entrada.
- **Especificidad:** peso de un selector, calculado como tres números `ID-CLASE-TIPO`, que decide qué regla gana entre dos del mismo origen e importancia.
- **Flexbox:** sistema de disposición unidimensional que se activa con `display: flex`.
- **gap:** propiedad que fija el espacio entre elementos de un contenedor flex.
- **guid:** identificador único y permanente de una entrada de RSS.
- **Herencia:** mecanismo por el que ciertas propiedades (color, tipografía) pasan de un elemento a su contenido.
- **Hoja de estilo del navegador:** estilos por defecto que trae cada navegador; la tuya los contradice.
- **JSON Feed:** formato de feed escrito en JSON, versión 1.1 de 2020.
- **Mobile first:** enfoque que escribe primero los estilos para pantalla estrecha y añade los de pantalla ancha con `@media (min-width: …)`.
- **Modelo de caja:** estructura de cada elemento: contenido, relleno (`padding`), borde (`border`) y margen (`margin`).
- **Nu Html Checker:** validador de HTML del W3C; comprueba también el CSS escrito dentro del documento.
- **Pila de fuentes:** lista de familias de letra de `font-family`, en orden de preferencia.
- **Propiedad personalizada (variable CSS):** valor con nombre (`--color`) que se usa con `var(--color)`.
- **Pseudoclase:** selector de estado de un elemento: `:hover`, `:focus`.
- **Punto de ruptura (*breakpoint*):** ancho de ventana a partir del cual cambia el diseño.
- **Regla:** selector más bloque de declaraciones.
- **rem:** unidad de longitud relativa al tamaño de letra base de la página (normalmente 16 px).
- **RSS:** familia de formatos XML de feed; RSS 2.0 usa `rss`, `channel` e `item`.
- **Selector:** parte de una regla que dice a qué elementos se aplica (de tipo, de grupo, de clase, descendiente, de pseudoclase, de atributo, de identificador).
- **Sindicación de contenidos:** publicar contenido en un formato estándar para que otros programas lo consuman.
- **Suscripción:** acción de dar a un lector la URL de un feed.
- **Validador de CSS del W3C:** herramienta que comprueba la sintaxis de una hoja de estilo.
- **Viewport (`meta viewport`):** etiqueta que indica al navegador del móvil que use el ancho real del dispositivo.
- **WCAG:** *Web Content Accessibility Guidelines*; pautas de accesibilidad web del W3C.

## Referencias y recursos

- **MDN Web Docs, especificidad:** developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascade/Specificity
- **MDN Web Docs, la cascada:** developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascade/Cascade
- **MDN Learn, el modelo de caja:** developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Box_model
- **WCAG 2.2, criterio 1.4.3 Contraste (mínimo), explicación del W3C:** w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- **Nu Html Checker:** validator.w3.org/nu
- **Validador de CSS del W3C:** jigsaw.w3.org/css-validator
- **Especificación RSS 2.0 (RSS Advisory Board):** rssboard.org/rss-specification
- **Autodescubrimiento de RSS:** rssboard.org/rss-autodiscovery
- **RFC 4287, The Atom Syndication Format:** rfc-editor.org/rfc/rfc4287
- **JSON Feed, versión 1.1:** jsonfeed.org/version/1.1
