# UD1 — Cuadernillo de ejercicios

**0373 · Lenguajes de Marcas y SGI** · Introducción a los lenguajes de marcas · RA1 (a-i) · Curso 2026-27

Ejercicios de consolidación para trabajar en casa, distintos de las actividades vistas en clase. Se resuelven de forma autónoma usando los apuntes de la UD1 como referencia. Cada ejercicio tiene su solución plegada debajo: inténtalo antes de abrirla.

**Índice**

- **Bloque 1** — ¿Qué es un lenguaje de marcas? (CE 1.a, 1.b) — 5 ejercicios

- **Bloque 2** — Clasificación y ámbitos de aplicación (CE 1.c, 1.d, 1.e) — 5 ejercicios

- **Bloque 3** — Comparativa XML/HTML (CE 1.f) — 5 ejercicios

- **Bloque 4** — Estructura y sintaxis (CE 1.g) — 5 ejercicios

- **Bloque 5** — Documentos bien formados y espacios de nombres (CE 1.h, 1.i) — 5 ejercicios

## Bloque 1 — ¿Qué es un lenguaje de marcas?

`// Criterios cubiertos: CE 1.a, 1.b`

### Ejercicio 1 · `<básico>`

`// Refuerza: CE 1.a`

Aquí tienes la ficha de un curso, sin marcar: "Curso de fotografía digital. Profesor: Marta Gil. Duración: 20 horas. Plazas: 15." Márcala tú mismo con un lenguaje de marcas, inventando las etiquetas que consideres razonables. El documento no tiene que ser válido según ningún estándar concreto — solo debe quedar sin ambigüedad qué es cada dato.

<details markdown="1">
<summary>Ver solución</summary>

```xml
<curso>
  <titulo>Curso de fotografía digital</titulo>
  <profesor>Marta Gil</profesor>
  <duracion unidad="horas">20</duracion>
  <plazas>15</plazas>
</curso>
```

Cualquier estructura razonable es válida siempre que cada dato quede identificado sin ambigüedad y el anidamiento sea correcto. Fíjate en si has usado un atributo o un elemento hijo con criterio (ver Bloque 4).

</details>

### Ejercicio 2 · `<básico>`

`// Refuerza: CE 1.b`

Elige 4 de las 6 ventajas del texto marcado vistas en los apuntes (legibilidad humana, independencia de la aplicación, separación de contenido y forma, interoperabilidad, procesabilidad automática, durabilidad) y, para cada una, escribe un ejemplo TUYO, distinto de los que aparecen en los apuntes.

<details markdown="1">
<summary>Ver solución</summary>

Respuesta abierta. Ejemplo de referencia para cada ventaja: **Legibilidad humana** — un archivo de configuración .xml se puede leer y corregir con el Bloc de notas. **Independencia de la aplicación** — un dibujo .svg se puede abrir con un navegador, con Inkscape o editarse a mano, da igual qué programa lo generó. **Separación de contenido y forma** — la letra de una canción marcada con XML podría mostrarse en una web, imprimirse en un cancionero o leerse con un sintetizador de voz sin cambiar el archivo. **Interoperabilidad** — un sistema de facturación y la Agencia Tributaria intercambian facturas electrónicas en XML sin que ninguno tenga que "traducir" el formato del otro. **Procesabilidad automática** — un script puede recorrer un catálogo XML de productos y calcular el precio medio automáticamente. **Durabilidad** — un archivo .html de 2005 se sigue abriendo perfectamente en cualquier navegador actual.

</details>

### Ejercicio 3 · `<medio>`

`// Refuerza: CE 1.b`

Una aplicación de gestión de inventario guarda sus datos en un formato binario propio. Construye una tabla con dos columnas — "Ventaja" y "¿Se cumple con este formato binario? ¿Por qué?" — repasando las 6 ventajas del texto marcado una a una para este caso.

<details markdown="1">
<summary>Ver solución</summary>

Un formato binario propietario incumple, en mayor o menor medida, casi todas las ventajas: no es legible por una persona sin la aplicación (incumple legibilidad humana), solo lo puede procesar esa aplicación concreta (incumple independencia e interoperabilidad), y si la empresa desaparece o descontinúa el programa, los datos pueden quedar inaccesibles (incumple durabilidad). La procesabilidad automática solo la tiene la propia aplicación, no terceros. La separación de contenido y forma depende del diseño interno, pero normalmente no está pensada para reutilizarse fuera de esa aplicación.

</details>

### Ejercicio 4 · `<avanzado (ampliación, opcional)>`

`// Refuerza: CE 1.a, 1.b`

Busca (en la documentación de alguna aplicación que uses, o en internet) un formato de archivo real, distinto de los ejemplos vistos en clase, que sea un lenguaje de marcas. Explica en 5-6 líneas qué ventajas de las vistas en clase aprovecha ese formato y por qué crees que se eligió texto marcado en vez de un binario.

<details markdown="1">
<summary>Ver solución</summary>

Respuesta abierta — comprueba que el ejemplo que elijas sea realmente un lenguaje de marcas (no una base de datos ni un binario) y que el razonamiento conecte con ventajas concretas, no genéricas. Ejemplo válido: KML (Google Earth) es XML; permite que cualquier programa de mapas lo lea, no solo Google Earth (interoperabilidad e independencia de la aplicación).

</details>

### Ejercicio 5 · `<auditoría>`

`// Refuerza: CE 1.a, 1.b`

Lee esta nota interna de una empresa y audítala: identifica qué está bien razonado, qué es cuestionable o incorrecto, y qué cambiarías, ordenado por gravedad.

```text
NOTA INTERNA — Departamento de Sistemas
A partir de este trimestre, el catálogo de productos de la
tienda se guardará en el formato binario propio de nuestra
aplicación de gestión, en lugar de en XML como hasta ahora.
Así el archivo pesará menos y se abrirá más rápido. Los
archivos XML anteriores se convertirán al nuevo formato y
se eliminarán.
```

<details markdown="1">
<summary>Ver solución</summary>

**Crítico** — eliminar los archivos XML originales sin conservar una copia es una decisión de alto riesgo: si en el futuro se cambia de aplicación de gestión (o el proveedor actual desaparece), los datos pueden quedar atrapados en un formato propietario sin posibilidad de recuperación — justo el problema de durabilidad visto en el Bloque 1. **Importante** — la justificación ("pesará menos y se abrirá más rápido") es una ventaja real del binario, pero se está priorizando el rendimiento sobre la interoperabilidad y la durabilidad sin haber evaluado ese trade-off explícitamente ni haber consultado si el catálogo se comparte con otros sistemas (proveedores, la propia web de la tienda). **Menor/aceptable** — no hay nada de malo, en sí, en usar un binario si el único consumidor de ese archivo es siempre esa misma aplicación y el rendimiento importa de verdad; el problema no es la elección del formato sino la falta de plan de conservación de los datos originales.

</details>

## Bloque 2 — Clasificación y ámbitos de aplicación

`// Criterios cubiertos: CE 1.c, 1.d, 1.e`

### Ejercicio 1 · `<básico>`

`// Refuerza: CE 1.c`

Clasifica cada una de estas marcas como presentacional, de procedimiento, descriptiva o referencial (las cuatro categorías de los apuntes):

- `<pagebreak/>` en un procesador de textos antiguo

- `<autor>Miguel Delibes</autor>`

- `<xref target="capitulo-3"/>`

- `<b>texto en negrita</b>` en HTML

- `<fecha-nacimiento>1990-04-12</fecha-nacimiento>`

<details markdown="1">
<summary>Ver solución</summary>

- `<pagebreak/>` → **procedimiento** (instrucción directa de salida: salta de página)

- `<autor>...</autor>` → **descriptivo** (dice QUÉ ES el contenido)

- `<xref target="...">` → **referencial** (relación con otra parte del documento)

- `<b>...</b>` → **procedimiento** (dice cómo se ve, no qué es)

- `<fecha-nacimiento>...</fecha-nacimiento>` → **descriptivo** (dice qué es el dato)

</details>

### Ejercicio 2 · `<básico>`

`// Refuerza: CE 1.d`

Para cada archivo, indica su ámbito de aplicación habitual según la tabla de los apuntes: `factura-2026.xml`, `index.html`, `logo.svg`, `activity_main.xml` (Android), `feed.rss`.

<details markdown="1">
<summary>Ver solución</summary>

- `factura-2026.xml` → intercambio de datos entre sistemas

- `index.html` → web y presentación de contenido

- `logo.svg` → gráficos vectoriales

- `activity_main.xml` → configuración de aplicaciones

- `feed.rss` → sindicación de contenidos (se desarrollará en la UD3)

</details>

### Ejercicio 3 · `<medio>`

`// Refuerza: CE 1.e`

Una empresa de logística quiere compartir con sus transportistas externos los datos de cada envío (destinatario, dirección, peso, fecha de entrega prevista), cada uno usando su propio software. Justifica en un párrafo si usarías XML o HTML para este intercambio, explicando la idea de "lenguaje de propósito general" vista en el Bloque 2.

<details markdown="1">
<summary>Ver solución</summary>

XML es la opción adecuada: no existe un vocabulario fijo (como en HTML) con etiquetas para "destinatario" o "peso del envío", y cada transportista usa un software distinto. XML, como metalenguaje de propósito general, permite definir un vocabulario propio (`<envio><destinatario>...`) que cualquier sistema puede procesar con solo conocer esas reglas, sin depender de que todos usen el mismo programa.

</details>

### Ejercicio 4 · `<avanzado (ampliación, opcional)>`

`// Refuerza: CE 1.c, 1.d`

Investiga uno de estos vocabularios XML de propósito específico, distinto de los vistos en clase: DocBook, MathML o KML. Describe en 5-6 líneas para qué se usa y pon un ejemplo (inventado, pero plausible) de una etiqueta suya.

<details markdown="1">
<summary>Ver solución</summary>

Respuesta abierta. Ejemplo de referencia — MathML: vocabulario XML para representar fórmulas matemáticas de forma que se puedan procesar (renderizar, validar, buscar) en vez de como una simple imagen. Ejemplo de etiqueta: `<mfrac><mn>1</mn><mn>2</mn></mfrac>` representaría la fracción 1/2.

</details>

### Ejercicio 5 · `<auditoría>`

`// Refuerza: CE 1.c`

En un foro de estudiantes alguien escribe: "HTML es un lenguaje de programación de propósito general, como XML — con HTML puedes definir tus propias etiquetas igual que con XML, solo que se usa para la web." Audita esta afirmación: señala qué es correcto y qué es incorrecto, ordenado por gravedad.

<details markdown="1">
<summary>Ver solución</summary>

**Crítico** — HTML no es un lenguaje de programación: es un lenguaje de marcas (no ejecuta lógica, describe estructura de contenido). Confundir ambos conceptos es un error de base. **Crítico** — HTML NO es de propósito general: tiene un vocabulario fijo y cerrado, definido por el estándar (WHATWG); no se pueden inventar libremente etiquetas nuevas y esperar que signifiquen algo para el navegador o para otros programas (los navegadores toleran etiquetas desconocidas visualmente, pero no les dan significado). **Correcto/aceptable** — es cierto que ambos, HTML y XML, usan una sintaxis parecida de marcas delimitadas por `< >`, y que HTML se usa específicamente para la web — eso sí está bien dicho.

</details>

## Bloque 3 — Comparativa XML/HTML

`// Criterios cubiertos: CE 1.f`

### Ejercicio 1 · `<básico>`

`// Refuerza: CE 1.f`

Indica si cada afirmación es verdadera o falsa, y justifica en una línea:

- a) Todo documento HTML5 es también un documento XML válido.

- b) Un parser XML estricto repara automáticamente una etiqueta sin cerrar.

- c) XHTML es HTML reformulado con las reglas sintácticas de XML.

- d) Un navegador tolera etiquetas mal anidadas porque intenta reparar el documento.

<details markdown="1">
<summary>Ver solución</summary>

- a) **Falsa** — solo XHTML lo es; un HTML5 normal puede tener atributos sin comillas o etiquetas sin cerrar, que un parser XML rechazaría.

- b) **Falsa** — un parser XML estricto NO repara nada: aborta con un error de análisis ante el primer fallo.

- c) **Verdadera** — es exactamente su definición.

- d) **Verdadera** — es la tolerancia característica de HTML, a diferencia del rigor de XML.

</details>

### Ejercicio 2 · `<básico>`

`// Refuerza: CE 1.f`

Completa esta tabla comparativa con "estricto" o "tolerante" (rigor sintáctico) para cada lenguaje: SGML, XML, HTML5, XHTML.

<details markdown="1">
<summary>Ver solución</summary>

- SGML → estricto en teoría, con variantes toleradas en la práctica

- XML → estricto

- HTML5 → tolerante

- XHTML → estricto (hereda las reglas de XML)

</details>

### Ejercicio 3 · `<medio>`

`// Refuerza: CE 1.f`

Sin usar el ordenador: predice si un navegador renderizaría sin problema este fragmento, y explica por qué.

```xml
<ul>
  <li>Primer elemento
    <li>Segundo elemento
    </ul>
```

<details markdown="1">
<summary>Ver solución</summary>

Sí, un navegador lo renderiza sin problema: aunque los dos `<li>` no están cerrados explícitamente, HTML tolera este patrón y el navegador cierra automáticamente cada `<li>` al encontrar el siguiente `<li>` o el `</ul>` final — la misma tolerancia vista con los `<p>` en el bloque 3 de los apuntes. Un parser XML estricto, en cambio, rechazaría este mismo fragmento.

</details>

### Ejercicio 4 · `<avanzado (ampliación, opcional)>`

`// Refuerza: CE 1.f`

Investiga por qué XHTML, pese a sus ventajas de rigor, nunca llegó a sustituir a HTML en la práctica. Resume en 5-6 líneas los motivos principales.

<details markdown="1">
<summary>Ver solución</summary>

Respuesta abierta — puntos que deberías recoger: el rigor de XHTML (un solo error rompe toda la página) resultaba poco práctico para la web real, donde el contenido se genera dinámicamente y con muchas fuentes distintas; HTML5 incorporó gran parte de las mejoras estructurales de XHTML sin exigir ese nivel de rigor; y el ecosistema (navegadores, herramientas, desarrolladores) terminó consolidándose en torno a HTML5 y WHATWG en lugar de la vía XML del W3C.

</details>

### Ejercicio 5 · `<auditoría>`

`// Refuerza: CE 1.f`

Audita este fragmento HTML: identifica los problemas ordenados por gravedad, indica cómo se comportaría en un navegador normal, y cómo se comportaría si se procesara con un parser XML estricto.

```xml
<article>
  <h3>Horario de tutorías
    <p>Lunes de 10 a 11, <b>sala 204</i>.
      <a href=horario.pdf>Descargar horario completo</a>
    </article>
```

<details markdown="1">
<summary>Ver solución</summary>

**Crítico** — anidamiento cruzado: `<b>` se abre y se cierra con `</i>`, una etiqueta distinta a la que lo abrió. Es el mismo problema visto en el bloque 3 de los apuntes — el navegador decide una interpretación propia, no garantizada entre navegadores. **Crítico** — `<h3>` y `<p>` no se cierran; el navegador los cierra automáticamente al llegar al siguiente elemento de bloque o al `</article>` final. **Importante** — el atributo `href=horario.pdf` no lleva comillas. **En un navegador normal**: la página se renderiza igualmente, sin ningún mensaje de error visible, gracias a la tolerancia de HTML. **Como XML estricto**: el documento sería rechazado en la primera etiqueta sin cerrar (`<h3>`), sin llegar siquiera a procesar el resto.

</details>

## Bloque 4 — Estructura y sintaxis de un documento de marcas

`// Criterios cubiertos: CE 1.g`

### Ejercicio 1 · `<básico>`

`// Refuerza: CE 1.g`

Localiza los errores de sintaxis de este documento (hay más de uno):

```xml
<curso nivel=avanzado>
  <nombre>Iniciación a XML<nombre>
    <horas>20</horas>
    <profesor id="P-3">Marta Gil</profesor
  </curso>
```

<details markdown="1">
<summary>Ver solución</summary>

- El valor del atributo `nivel` no lleva comillas: debería ser `nivel="avanzado"`.

- `<nombre>` se abre pero se "cierra" repitiendo `<nombre>` en vez de `</nombre>`.

- La etiqueta de cierre de `<profesor>` está incompleta: falta el `>` final (`</profesor>`).

</details>

### Ejercicio 2 · `<básico>`

`// Refuerza: CE 1.g`

Escribe un documento XML completo y bien formado (con declaración incluida) para una receta de cocina, con al menos: nombre de la receta, tiempo de preparación, y una lista de al menos dos ingredientes.

<details markdown="1">
<summary>Ver solución</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<receta>
  <nombre>Tortilla de patatas</nombre>
  <tiempo unidad="min">45</tiempo>
  <ingredientes>
    <ingrediente>Patatas</ingrediente>
    <ingrediente>Huevos</ingrediente>
  </ingredientes>
</receta>
```

Cualquier estructura bien formada equivalente es válida: elemento raíz único, todas las etiquetas cerradas y correctamente anidadas.

</details>

### Ejercicio 3 · `<medio>`

`// Refuerza: CE 1.g`

Parte de este documento y añade: un atributo `id` al elemento raíz, un atributo `idioma` al elemento `titulo`, y un comentario al principio explicando de qué trata el archivo.

```xml
<libro>
  <titulo>Cien años de soledad</titulo>
  <autor>Gabriel García Márquez</autor>
</libro>
```

<details markdown="1">
<summary>Ver solución</summary>

```xml
<!-- Ficha de un libro del catálogo de la biblioteca -->
<libro id="L-045">
  <titulo idioma="es">Cien años de soledad</titulo>
  <autor>Gabriel García Márquez</autor>
</libro>
```

</details>

### Ejercicio 4 · `<medio>`

`// Refuerza: CE 1.g`

Para cada uno de estos tres datos de un pedido online, decide si lo pondrías como atributo o como elemento hijo, y justifica con el criterio visto en los apuntes (Bloque 4): (a) el número de pedido, (b) la lista de productos comprados, (c) la moneda en la que está expresado el importe total.

<details markdown="1">
<summary>Ver solución</summary>

(a) **Atributo** del elemento raíz `pedido` (`<pedido numero="2026-045">`) — es un identificador, metadato sobre el pedido en sí. (b) **Elemento hijo** — la lista de productos tiene estructura propia (varios productos, cada uno con sus datos), no cabe como un simple valor de atributo. (c) **Atributo** del elemento `importe` (`<importe moneda="EUR">`) — es información sobre ese dato concreto, siguiendo el mismo criterio que el ejemplo de precio de los apuntes.

</details>

### Ejercicio 5 · `<auditoría>`

`// Refuerza: CE 1.g`

Audita este documento XML generado automáticamente por un script de exportación: identifica todos los problemas, ordenados por gravedad, y explica qué cambiarías.

```xml
<alumno id=A-102>
  <nombre>Laura</nombre>
  <asignatura>
    <nombre>LMSGI</nombre>
    <nota>7.5</nota>
  </asignatura>
</alumno
```

<details markdown="1">
<summary>Ver solución</summary>

**Crítico** — la etiqueta de cierre final `</alumno>` está incompleta (falta el `>`), lo que por sí solo ya rompe la buena formación del documento completo. **Crítico** — el atributo `id=A-102` no lleva comillas. **Importante** — el documento no tiene ninguna indentación, lo que hace muy difícil comprobar a simple vista qué elemento cierra a cuál: conviene reescribirlo indentado. **Menor** — sería más legible añadir la declaración `<?xml version="1.0" encoding="UTF-8"?>` al principio, aunque no imprescindible según el contexto en que se use el archivo.

</details>

## Bloque 5 — Documentos bien formados y espacios de nombres

`// Criterios cubiertos: CE 1.h, 1.i`

### Ejercicio 1 · `<básico>`

`// Refuerza: CE 1.h`

Para cada documento, indica si está bien formado o no, y por qué:

```text
DOCUMENTO A:
<a><b>texto</b></a>
DOCUMENTO B:
<a><b>texto</a></b>
DOCUMENTO C:
<a>Precio: 10 &lt; 20</a>
```

<details markdown="1">
<summary>Ver solución</summary>

- Documento A → **bien formado**: un único elemento raíz, cierre y anidamiento correctos.

- Documento B → **NO bien formado**: anidamiento cruzado, `<b>` se cierra fuera de orden.

- Documento C → **bien formado**: el carácter `<` está correctamente escapado como `&lt;`.

</details>

### Ejercicio 2 · `<básico>`

`// Refuerza: CE 1.h`

Explica con tus propias palabras, en 3-4 líneas, la diferencia entre que un documento esté "bien formado" y que esté "validado" (este segundo concepto se desarrollará con detalle en la UD5, pero ya se ha adelantado en los apuntes de esta unidad).

<details markdown="1">
<summary>Ver solución</summary>

Estar bien formado es una condición puramente sintáctica: que el documento respete las reglas básicas de XML (raíz única, etiquetas cerradas, atributos entrecomillados, caracteres escapados), sin importar si el contenido tiene sentido. Estar validado va un paso más allá: comprobar que el documento, además, se ajusta a una gramática o vocabulario concreto (por ejemplo, que un pedido tenga siempre un número y un cliente, definido en un DTD o un XML Schema).

</details>

### Ejercicio 3 · `<medio>`

`// Refuerza: CE 1.i`

Este documento combina dos vocabularios (datos de un alumno y datos de su tutor) sin espacios de nombres, con una colisión de nombre en `<nombre>` y `<telefono>`. Reescríbelo usando espacios de nombres para eliminar la ambigüedad.

```xml
<ficha>
  <alumno>
    <nombre>Laura Ibáñez</nombre>
    <telefono>600111222</telefono>
  </alumno>
  <tutor>
    <nombre>Carlos Ibáñez</nombre>
    <telefono>600333444</telefono>
  </tutor>
</ficha>
```

<details markdown="1">
<summary>Ver solución</summary>

En este caso concreto, al estar `<alumno>` y `<tutor>` en ramas separadas, `<nombre>` y `<telefono>` no colisionan técnicamente (el anidamiento ya los distingue) — pero si se quisiera dejar explícito el origen de cada vocabulario, o prepararlo para una futura fusión con otros orígenes de datos, se resolvería así:

```xml
<ficha xmlns:est="http://ejemplo.org/estudiante"
       xmlns:tut="http://ejemplo.org/tutor">
  <est:alumno>
    <est:nombre>Laura Ibáñez</est:nombre>
    <est:telefono>600111222</est:telefono>
  </est:alumno>
  <tut:tutor>
    <tut:nombre>Carlos Ibáñez</tut:nombre>
    <tut:telefono>600333444</tut:telefono>
  </tut:tutor>
</ficha>
```

</details>

### Ejercicio 4 · `<avanzado (ampliación, opcional)>`

`// Refuerza: CE 1.i`

Investiga (o razona a partir de lo visto en el Bloque 5 de los apuntes) qué ocurriría si dos ramas distintas del mismo documento XML declararan el mismo prefijo de espacio de nombres (por ejemplo `cat:`) apuntando a dos URIs diferentes. Explica el resultado con un ejemplo propio de 4-5 líneas de XML.

<details markdown="1">
<summary>Ver solución</summary>

Es sintácticamente válido: el ámbito de una declaración `xmlns:prefijo` es el elemento donde se declara y sus descendientes, así que un prefijo puede "redefinirse" en una rama distinta sin error — pero es una fuente de confusión grave, porque el mismo prefijo significa cosas distintas según en qué parte del árbol se lea. Ejemplo: `<raiz><a xmlns:cat="URI-1"><cat:x/></a><b xmlns:cat="URI-2"><cat:x/></b></raiz>` — los dos `<cat:x/>` tienen el mismo prefijo pero pertenecen a espacios de nombres distintos.

</details>

### Ejercicio 5 · `<auditoría>`

`// Refuerza: CE 1.h, 1.i`

Audita este documento, supuestamente exportado combinando el perfil de un profesor con su horario de tutorías:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<profesor id="P-12">
  <nombre>Cobra & Asociados — María P.</nombre>
</profesor>
<horario>
  <nombre>Horario de tutorías</nombre>
  <dia>Miércoles</dia>
</horario>
```

<details markdown="1">
<summary>Ver solución</summary>

**Crítico** — hay dos elementos de nivel superior (`<profesor>` y `<horario>`), sin una raíz común: el documento no está bien formado. **Crítico** — el carácter `&` en "Cobra & Asociados" no está escapado; debería ser `&amp;`. **Importante** — `<nombre>` aparece en ambos vocabularios con significados distintos (nombre del profesor / nombre del horario); si se fusionaran correctamente bajo una raíz común, haría falta resolverlo con espacios de nombres, igual que en el ejemplo del bloque 5 de los apuntes. **Menor** — conviene revisar si "Cobra & Asociados" es realmente el nombre que se quiere exportar en un campo de nombre de profesor, más allá del problema sintáctico del `&` — parece un error de origen de datos, no solo de escritura del XML.

</details>