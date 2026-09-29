# UD1 — Introducción a los lenguajes de marcas

**0373 · Lenguajes de Marcas y SGI** · Apuntes de la unidad · RA1 (a-i) · 1º DAW · Curso 2026-27

---

> Para practicar: [cuadernillo de ejercicios de la UD1](cuadernillo.md) (con soluciones plegadas) · [versión para imprimir, solo enunciados (PDF)](descargas/UD1_cuadernillo_enunciados.pdf).

## Introducción

En esta unidad vas a aprender a reconocer las características de los lenguajes de marcas, analizando e interpretando fragmentos de código. El trabajo se organiza en cinco sesiones, cada una centrada en un bloque de contenido:

- **Sesión 1:** ¿Qué es un lenguaje de marcas? Características generales y ventajas.
- **Sesión 2:** Clasificación de los lenguajes de marcas y ámbitos de aplicación.
- **Sesión 3:** Comparativa de lenguajes concretos: XML, HTML y otros.
- **Sesión 4:** Estructura y sintaxis de un documento de marcas.
- **Sesión 5:** Documentos bien formados y espacios de nombres.

Estos apuntes son un material de apoyo y consulta: úsalos para repasar y consolidar lo trabajado en clase, no como primer contacto con cada tema.

## Índice

1. ¿Qué es un lenguaje de marcas? *(1.a, 1.b)*
2. Clasificación de los lenguajes de marcas y ámbitos de aplicación *(1.c, 1.d, 1.e)*
3. Comparativa de lenguajes concretos: XML, HTML y otros *(1.f)*
4. Estructura y sintaxis de un documento de marcas *(1.g)*
5. Documentos bien formados y espacios de nombres *(1.h, 1.i)*
- Glosario
- Referencias y recursos

---

## 1. ¿Qué es un lenguaje de marcas?

### 1.1 Origen y definición

El término "marcar" un texto es anterior a la informática. En la edición tradicional de libros, el corrector de pruebas escribía anotaciones a mano en los márgenes del manuscrito para indicar al tipógrafo cómo componer el texto: "esto va en cursiva", "aquí empieza una sección nueva". Los primeros lenguajes de marcas informáticos (GML, desarrollado por IBM en los años 60-70) trasladaron esa misma idea al texto digital, dando lugar a la familia de lenguajes que se sigue usando hoy: GML → SGML → HTML / XML. Por eso HTML y XML comparten un aspecto sintáctico similar —elementos delimitados por `< >`— pese a tener propósitos distintos: ambos descienden de SGML.

Un **lenguaje de marcas** (markup language) es, por tanto, un sistema de anotación que permite combinar, dentro de un mismo documento de texto, contenido y un conjunto de marcas o etiquetas que indican cómo debe interpretarse, estructurarse o procesarse ese contenido, sin recurrir a un formato binario propietario. Las marcas conviven con el contenido pero son sintácticamente distinguibles de él, de modo que un programa (parser) puede separar de forma automática el contenido de las instrucciones que lo acompañan.

### 1.2 Marcas explícitas frente a formato oculto

Conviene distinguir un lenguaje de marcas de lo que hace un procesador de textos como Word cuando se pone una palabra en negrita. En ese caso, el formato se gestiona de forma interna y oculta, dentro de un formato binario o empaquetado propio de la aplicación. En un lenguaje de marcas, en cambio, la marca es texto visible, explícito y legible, tanto por una persona como por cualquier programa que entienda su sintaxis.

Compara la misma información sobre un producto, sin marcar y marcada con un lenguaje de marcas. Sin marcar, el texto es ambiguo: ¿cuál es el precio? ¿y el nombre?

> Teclado mecánico compacto 54.90 Periféricos

Marcada, cada dato queda identificado sin ambigüedad:

```xml
<producto>
    <nombre>Teclado mecánico compacto</nombre>
    <precio>54.90</precio>
    <categoria>Periféricos</categoria>
</producto>
```

Un programa que reciba el segundo documento puede extraer el precio con total fiabilidad, buscando el contenido del elemento `precio`; con el primero tendría que adivinarlo.

### 1.3 Ventajas de trabajar con lenguajes de marcas

- **Legibilidad humana:** al ser texto plano, se puede leer y editar sin herramienta especializada, incluso con un editor de texto genérico.
- **Independencia de la aplicación:** cualquier programa que entienda la sintaxis puede procesar el documento; no depende de un fabricante concreto.
- **Separación de contenido y forma:** el mismo contenido marcado puede reutilizarse para distintas salidas (pantalla, impresión, lectura por voz) sin reescribirlo.
- **Interoperabilidad:** al ser un formato de texto estandarizado, sistemas y lenguajes de programación distintos pueden intercambiar información sin conversiones complejas.
- **Procesabilidad automática:** la estructura explícita permite que un programa navegue, valide, transforme o extraiga información del documento de forma fiable.
- **Durabilidad:** un archivo de texto plano con marcas sigue siendo legible dentro de veinte años aunque el software que lo creó haya desaparecido; un binario propietario puede quedar "atrapado" si el fabricante deja de existir.

> **ALTERNATIVAS Y CRITERIO**
>
> La misma información podría almacenarse en un formato binario propietario (por ejemplo, el .doc antiguo de Word) o en el formato interno de una base de datos. Un lenguaje de marcas de texto plano es preferible cuando el objetivo es el intercambio, la interoperabilidad y la durabilidad de la información a largo plazo, por encima de la eficiencia de espacio o velocidad que sí ofrece un binario a medida. Cuando el único requisito es el rendimiento interno de una única aplicación —por ejemplo, el motor de almacenamiento de una base de datos—, un formato binario propio puede ser preferible, pero entonces deja de ser un lenguaje de marcas.

> **ERRORES FRECUENTES**
>
> Es fácil confundir "lenguaje de marcas" con "dar formato" en el sentido de un procesador de textos: negrita, cursiva, tamaño de fuente. Poner texto en negrita en Word es aplicar un formato visual gestionado internamente por la aplicación, no escribir marcas legibles como texto. Un lenguaje de marcas es una sintaxis explícita, visible y procesable por terceros, no una decoración oculta dentro de un binario.

## 2. Clasificación de los lenguajes de marcas y ámbitos de aplicación

### 2.1 Cuatro formas de marcar un texto

Los lenguajes de marcas se clasifican, según el tipo de información que codifican sus marcas, en cuatro categorías funcionales:

- **Marcado presentacional:** la marca indica directamente un efecto visual sobre el propio texto, ligado a un formato o dispositivo de salida concreto. Es el caso de RTF (Rich Text Format), donde `{\b texto en negrita}` indica el efecto visual sin decir nada sobre qué es ese texto.
- **Marcado de procedimiento:** las marcas son instrucciones directas para el programa de salida ("pon esto en negrita", "salta de página aquí"). Ejemplos: troff, PostScript, los comandos de TeX/LaTeX. Ventaja: control preciso de la salida; inconveniente: la marca no dice qué ES el contenido, solo cómo se ve, así que no es reutilizable para otro medio de salida.
- **Marcado descriptivo o generalizado:** las marcas describen qué ES cada parte del contenido —un título, una fecha, un precio— sin decir cómo debe mostrarse; la presentación queda para una hoja de estilo o un procesador posterior. La familia SGML/XML pertenece aquí (DocBook es un ejemplo de vocabulario descriptivo para documentación técnica), y HTML se le aproxima aunque mezcla algo de marcado de procedimiento por herencia histórica. Ventaja: el mismo documento sirve para pantalla, impresión o lectura por voz sin cambios.
- **Marcado referencial:** las marcas no describen el contenido, sino relaciones entre partes de un documento o entre documentos distintos: un hipervínculo (`<a href="...">` en HTML), una referencia cruzada, la inclusión de un fragmento externo.

### 2.2 La familia de lenguajes de marcas

- **SGML** (Standard Generalized Markup Language): metalenguaje original, normalizado por ISO (norma ISO 8879:1986); muy potente pero complejo de implementar; predecesor directo de HTML y XML.
- **XML** (eXtensible Markup Language): subconjunto simplificado de SGML, de propósito general, normalizado por el W3C; permite definir vocabularios de marcado propios.
- **HTML** (HyperText Markup Language): vocabulario fijo y específico para la web, orientado a presentación e hipertexto. Se desarrollará en profundidad en la UD2.
- **XHTML:** reformulación de HTML como aplicación de XML, con sintaxis más estricta; hoy en gran medida en desuso frente a HTML5.
- **Otros vocabularios XML de propósito específico:** SVG (gráficos vectoriales), MathML (notación matemática), DocBook (documentación técnica), RSS/Atom (sindicación de contenidos, se estudiará en la UD3).

SGML y XML no son de un único organismo: SGML se formalizó como norma ISO, mientras que XML se desarrolla y mantiene como recomendación del W3C (World Wide Web Consortium) — los dos organismos que, junto con WHATWG para HTML, regulan hoy esta familia de lenguajes.

### 2.3 Dónde se usa cada lenguaje

| Ámbito | Lenguaje(s) habitual(es) |
|---|---|
| Presentación de contenido web | HTML / XHTML |
| Intercambio de datos entre sistemas | XML (facturación electrónica, servicios SOAP) |
| Documentación técnica y editorial | DocBook, DITA |
| Formatos de oficina | OOXML (.docx/.xlsx/.pptx), ODF |
| Configuración de aplicaciones y sistemas | XML (layouts de Android, Maven pom.xml, Spring) |
| Gráficos vectoriales | SVG |
| Sindicación de contenidos | RSS / Atom (UD3) |

### 2.4 Por qué hace falta un lenguaje de propósito general

Cuando ningún vocabulario existente encaja "de fábrica" con el dominio de un problema —una factura, una partitura musical, la configuración de una aplicación concreta—, se necesita un metalenguaje que permita definir las propias etiquetas manteniendo unas reglas sintácticas comunes, y opcionalmente una gramática propia (DTD o XML Schema, que se estudiarán en la UD5) para validar los documentos de ese vocabulario. Ese es exactamente el papel de XML: no es un lenguaje con vocabulario fijo como HTML, sino una plantilla de reglas para crear lenguajes de marcas a medida.

Observa la diferencia entre expresar la información de una factura con un vocabulario XML propio o forzarla dentro del vocabulario fijo de HTML:

```xml
<!-- Vocabulario XML propio: semántica real -->
<factura numero="2026-045">
    <importe moneda="EUR">350.00</importe>
    <cliente>Academia Delta</cliente>
</factura>
```

```html
<!-- Forzado en HTML: la semántica depende de una convención,
     no existe la etiqueta <factura> -->
<div class="factura" data-numero="2026-045">
    <span class="importe">350.00 EUR</span>
    <span class="cliente">Academia Delta</span>
</div>
```

En el segundo caso, un programa solo sabe que `data-numero` es "el número de factura" porque alguien se lo dijo por convención; en el primero, la propia sintaxis lo dice.

> **ALTERNATIVAS Y CRITERIO**
>
> Existe una alternativa a diseñar un vocabulario XML propio para intercambiar datos: usar JSON, que no es un lenguaje de marcas sino una notación de objetos. XML sigue siendo preferible cuando el documento necesita mezclar contenido narrativo con estructura (texto con marcas incrustadas dentro de un párrafo) o cuando se requiere una gramática formal validable y espacios de nombres para combinar vocabularios de distinto origen; JSON resulta más ligero y es preferible para estructuras de datos puras (objetos, listas) en APIs web modernas, pero no está pensado para marcar texto narrativo ni dispone de espacios de nombres.

> **ERRORES FRECUENTES**
>
> Es habitual pensar que "lenguaje de marcas" es sinónimo de HTML. HTML es solo uno de los lenguajes de marcas posibles —el más conocido por ser el de la web—, pero la familia es mucho más amplia. Confundir el concepto general con un caso particular impide entender por qué existen XML, SVG o los formatos de configuración basados en marcas.

> **PARA SABER MÁS**
>
> Aunque XML dominó el intercambio de datos entre 2000 y 2010 (SOAP, servicios web empresariales), buena parte de las APIs REST actuales usan JSON por su menor peso. Sin embargo, XML sigue siendo el estándar de facto en la facturación electrónica (Facturae en España), en los formatos de oficina —un .docx es, literalmente, un ZIP de archivos XML— y en la configuración de numerosos sistemas empresariales: el terreno que se recorrerá en las UD5 a UD8 de este módulo.

## 3. Comparativa de lenguajes concretos: XML, HTML y otros

### 3.1 Extensibilidad, rigor y propósito

Aunque SGML, XML, HTML y XHTML comparten origen y una sintaxis de marcas similar en apariencia —elementos delimitados por `< >`—, difieren en tres características clave: el grado de **extensibilidad** (vocabulario fijo o definible por quien escribe el documento), el **rigor sintáctico** exigido (tolerancia del procesador ante errores) y el **propósito** para el que fueron diseñados.

| Lenguaje | Vocabulario | Rigor sintáctico | Propósito principal |
|---|---|---|---|
| SGML | Definible (vía DTD) | Estricto en teoría; variantes toleradas en la práctica | Definir lenguajes de marcas complejos |
| XML | Definible (metalenguaje) | Estricto — el parser aborta ante el primer error | Intercambio de datos, vocabularios propios |
| HTML5 | Fijo (estándar WHATWG) | Tolerante — el navegador intenta reparar errores | Presentación y estructura de páginas web |
| XHTML | Fijo (igual que HTML) | Estricto — hereda las reglas de XML | Web procesable con herramientas XML (en desuso) |

### 3.2 Qué implica ese rigor en la práctica

Fíjate en un fragmento HTML con una etiqueta sin cerrar:

```html
<p>Este párrafo no cierra su etiqueta
<p>Y este tampoco</p>
```

Ese fragmento se renderiza igualmente en cualquier navegador: el motor de HTML "adivina" dónde debería cerrarse el primer `<p>`. El mismo fragmento, interpretado por un parser XML estricto, produce un error de análisis (parsing error) y el documento no se procesa en absoluto: no hay reparación automática.

> **ALTERNATIVAS Y CRITERIO**
>
> Para el desarrollo web actual, la alternativa histórica a HTML5 es XHTML. Se prefiere HTML5 en la inmensa mayoría de proyectos porque su tolerancia a errores facilita el desarrollo y porque es el estándar activamente mantenido por WHATWG con soporte universal en los navegadores; XHTML solo se justifica cuando el documento debe procesarse además con herramientas XML genéricas (XSLT, XPath) sin pasar antes por un parser HTML, un caso cada vez más raro en proyectos nuevos.

> **ERRORES FRECUENTES**
>
> No des por hecho que todo documento HTML es automáticamente un documento XML válido, es decir, que "HTML es un tipo de XML". Esto solo es cierto para XHTML. Un documento HTML5 normal puede tener atributos sin comillas, etiquetas sin cerrar como `<br>` o `<img>`, y anidamiento que un parser XML estricto rechazaría de inmediato.

> **PARA SABER MÁS**
>
> Este tema se retomará en la UD2: que el navegador tolere errores no significa que esos errores sean inofensivos. Un marcado mal formado, aunque "funcione" visualmente, dificulta el mantenimiento, la accesibilidad y el procesamiento automático del documento por otras herramientas.

## 4. Estructura y sintaxis de un documento de marcas

### 4.1 El documento como árbol

Todo documento de marcas se organiza como un árbol jerárquico de elementos: un único **elemento raíz** contiene, anidado dentro de sí, el resto de elementos del documento. Este modelo de árbol —el mismo que usa el DOM (Document Object Model) para representar un documento en memoria— tiene su propio vocabulario, que conviene manejar con soltura: el **nodo raíz** es el único elemento sin padre, del que cuelga todo lo demás; un **nodo padre** es el que contiene directamente a otro; los elementos contenidos directamente en él son sus **nodos hijos**; y dos o más elementos que comparten el mismo padre son **nodos hermanos** entre sí.

Cada elemento se delimita mediante una etiqueta de apertura y una de cierre —o una etiqueta vacía autocontenida—, puede llevar atributos que aportan información adicional sobre ese elemento, y puede contener texto, otros elementos, o ambos.

### 4.2 Sintaxis básica

- **Etiqueta de apertura:** `<elemento>`
- **Etiqueta de cierre:** `</elemento>`
- **Etiqueta vacía o autocontenida:** `<elemento/>`, equivalente a `<elemento></elemento>`
- **Atributos:** pares nombre="valor" dentro de la etiqueta de apertura: `<elemento atributo="valor">`
- **Anidamiento:** los elementos deben cerrarse en el orden inverso al que se abrieron; estructura de árbol, nunca de solapamiento
- **Declaración o prólogo** (específico de XML): `<?xml version="1.0" encoding="UTF-8"?>`, indica la versión de XML y la codificación de caracteres; debe ser la primera línea del documento
- **Comentarios:** `<!-- comentario -->`, válidos en XML y en HTML, ignorados por el procesador
- **Sensibilidad a mayúsculas:** XML distingue mayúsculas de minúsculas en los nombres de elemento (`<Producto>` y `<producto>` son elementos distintos); HTML no la distingue en la práctica, aunque la convención moderna es escribir siempre en minúsculas

### 4.3 Un documento completo, elemento a elemento

El siguiente documento XML está bien formado y reúne todos los elementos sintácticos anteriores:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Ficha de un producto del catálogo -->
<producto id="P-1024" disponible="true">
    <nombre>Teclado mecánico compacto</nombre>
    <categoria>Periféricos</categoria>
    <precio moneda="EUR">54.90</precio>
    <descripcion>
        Teclado mecánico de perfil bajo con conexión USB-C
        y retroiluminación regulable.
    </descripcion>
</producto>
```

Aplicando el vocabulario del árbol de la sección 4.1: `producto` es el nodo raíz, y `id="P-1024"` y `disponible="true"` son sus atributos —metadatos sobre el producto, no contenido narrativo—. `nombre`, `categoria`, `precio` y `descripcion` son nodos hijos de `producto`, y a la vez hermanos entre sí, porque comparten el mismo padre. `moneda="EUR"` es un atributo del nodo `precio`. La línea `<!-- Ficha de un producto del catálogo -->` es un comentario: no forma parte de los datos, solo documenta el archivo.

> **ALTERNATIVAS Y CRITERIO**
>
> Una alternativa a delimitar un dato mediante un atributo es convertirlo en un elemento hijo: por ejemplo, `<precio><valor>54.90</valor><moneda>EUR</moneda></precio>` en lugar de un atributo `moneda`. No es una regla sintáctica obligatoria sino una convención de diseño: se suele usar un atributo para metadatos sobre el elemento (identificadores, unidades, banderas) y un elemento hijo para datos que forman parte del contenido principal o que podrían necesitar su propia estructura interna en el futuro.

> **ERRORES FRECUENTES**
>
> Vigila el anidamiento cruzado: `<precio><moneda>EUR</precio></moneda>` en lugar de `<precio><moneda>EUR</moneda></precio>`. Un parser XML rechaza este documento de inmediato. Acostúmbrate a leer y escribir el cierre de etiquetas "de dentro hacia fuera", en el orden inverso al de apertura.

> **PARA SABER MÁS**
>
> Los editores de código profesionales (VSCode y, en general, cualquier IDE) resaltan automáticamente el emparejamiento de etiquetas y avisan de anidamientos incorrectos antes incluso de ejecutar o abrir el documento en otra herramienta: una de las razones prácticas por las que conviene escribir marcado en un editor de texto o código, y no en un procesador de textos como Word.

## 5. Documentos bien formados y espacios de nombres

### 5.1 Qué significa estar "bien formado"

Un documento de marcas está **bien formado** (well-formed) cuando cumple todas las reglas sintácticas básicas del lenguaje: tiene un único elemento raíz, todas las etiquetas están correctamente cerradas y anidadas, los valores de los atributos van entre comillas, y los caracteres especiales reservados están correctamente escapados cuando aparecen como texto. Ser bien formado es una condición puramente sintáctica, independiente de si el contenido tiene sentido o se ajusta a un vocabulario concreto —eso sería estar **validado**, y se abordará con DTD y XML Schema en la UD5.

- Un único elemento raíz que contiene a todos los demás.
- Toda etiqueta abierta se cierra, y el anidamiento respeta el orden inverso de apertura.
- Los valores de los atributos van siempre entre comillas (simples o dobles, de forma consistente).
- Los caracteres especiales reservados se escapan cuando forman parte del texto: `&lt;` por `<`, `&gt;` por `>`, `&amp;` por `&`, `&quot;` por `"`, `&apos;` por `'`.

Un procesador XML estricto se detiene y lanza un error de análisis (fatal error) ante el primer fallo de buena formación; no intenta "adivinar" ni reparar el documento, a diferencia de un navegador procesando HTML. Es una decisión de diseño deliberada: garantiza que cualquier programa conforme al estándar interprete un documento XML exactamente igual que cualquier otro, sin ambigüedad. La implicación práctica es directa: cuando un programa genera XML automáticamente —por ejemplo, una exportación de datos—, un solo carácter mal escapado puede romper la lectura completa del archivo por cualquier sistema que lo consuma después.

### 5.2 Espacios de nombres

Un **espacio de nombres** (namespace) es un mecanismo de XML para evitar colisiones entre nombres de elemento o atributo cuando se combinan, dentro de un mismo documento, vocabularios definidos por orígenes distintos. Si dos vocabularios usan, cada uno con su propio significado, el mismo nombre de elemento —por ejemplo `titulo`—, el documento resultante sería ambiguo sin un mecanismo que los distinga.

La sintaxis se basa en el atributo `xmlns`: `xmlns:prefijo="URI"` declara un espacio de nombres con un prefijo, que se aplica al elemento donde se declara y a todos sus descendientes; los elementos de ese espacio se escriben como `prefijo:elemento`. La URI es únicamente un identificador único de texto, no tiene por qué ser una dirección real que el procesador visite. También existe el espacio de nombres por defecto (`xmlns="URI"`, sin prefijo), que se aplica a los elementos sin prefijo dentro de su ámbito.

El siguiente documento combina un vocabulario propio (`catalogo`) con SVG embebido, ambos con un elemento de nombre similar, sin colisión gracias a los espacios de nombres:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalogo:producto
    xmlns:catalogo="http://ejemplo.org/catalogo"
    xmlns:svg="http://www.w3.org/2000/svg">
    <catalogo:nombre>Teclado mecánico compacto</catalogo:nombre>
    <catalogo:icono>
        <svg:svg width="24" height="24">
            <svg:title>Icono de teclado</svg:title>
        </svg:svg>
    </catalogo:icono>
</catalogo:producto>
```

`catalogo:nombre` y `svg:title` conviven sin ambigüedad precisamente porque cada uno declara a qué espacio de nombres pertenece.

> **ALTERNATIVAS Y CRITERIO**
>
> JSON no dispone de un mecanismo equivalente a los espacios de nombres: si dos fuentes de datos JSON usan la misma clave con significados distintos y se combinan en un mismo objeto, la única forma de evitar la colisión es una convención manual —prefijar las claves a mano, anidar en sub-objetos—. Cuando un documento debe combinar de forma fiable vocabularios de orígenes distintos y potencialmente desconocidos de antemano, como ocurre en SOAP o al incrustar SVG dentro de XML, XML con espacios de nombres ofrece una solución estandarizada; para estructuras de datos internas de una única aplicación, sin riesgo real de colisión, esta complejidad adicional no suele compensar.

> **ERRORES FRECUENTES**
>
> Vigila dos errores típicos: declarar dos espacios de nombres por defecto distintos en el mismo ámbito, o repetir un prefijo con una URI diferente en un elemento anidado sin darte cuenta de que eso redefine su significado para ese subárbol. También es frecuente confundir la URI de un espacio de nombres con una dirección web real "que hay que visitar": es solo un identificador de texto, el procesador nunca la descarga.

> **PARA SABER MÁS**
>
> Los espacios de nombres son la base de tecnologías que se usarán más adelante en este módulo y en el resto del ciclo: SVG embebido en HTML5, las extensiones de RSS/Atom de la UD3, y los servicios web SOAP que en su día dominaron la integración empresarial: el mismo mecanismo que aquí se presenta de forma introductoria.

## Glosario

- **Marcado (markup):** anotación insertada en un texto para indicar estructura, semántica o presentación.
- **Marcado presentacional:** marcado que indica directamente un efecto visual, ligado a un formato de salida concreto (ejemplo: RTF).
- **Marcado de procedimiento:** marcado que da instrucciones directas al programa de salida sobre cómo mostrar el contenido (ejemplos: troff, PostScript, TeX/LaTeX).
- **Marcado descriptivo:** marcado que indica qué es cada parte del contenido, sin decir cómo debe mostrarse (familia SGML/XML).
- **Marcado referencial:** marcado que expresa relaciones entre partes de un documento o entre documentos (ejemplo: un hipervínculo).
- **Etiqueta (tag):** marca delimitadora de un elemento, de apertura o de cierre.
- **Elemento:** unidad estructural de un documento de marcas, compuesta por su etiqueta (o etiquetas), sus atributos y su contenido.
- **Atributo:** par nombre-valor que aporta información adicional dentro de una etiqueta de apertura.
- **Nodo raíz:** único elemento de nivel superior que contiene a todos los demás en un documento de marcas.
- **Nodo padre / nodo hijo / nodos hermanos:** vocabulario del árbol del documento (DOM): un nodo padre contiene directamente a sus nodos hijos; dos nodos con el mismo padre son hermanos entre sí.
- **Documento bien formado (well-formed):** documento que cumple las reglas sintácticas básicas del lenguaje de marcas.
- **Validación:** comprobación de que un documento, además de bien formado, se ajusta a una gramática o vocabulario concreto (DTD, XML Schema) — se desarrollará en la UD5.
- **SGML:** Standard Generalized Markup Language, metalenguaje normalizado por ISO del que derivan HTML y XML.
- **XML:** eXtensible Markup Language, metalenguaje de propósito general normalizado por el W3C para definir vocabularios de marcado propios.
- **HTML:** HyperText Markup Language, vocabulario fijo orientado a la web.
- **XHTML:** reformulación de HTML como aplicación de XML.
- **Espacio de nombres (namespace):** mecanismo de XML para evitar colisiones de nombres al combinar vocabularios de distinto origen.
- **W3C:** World Wide Web Consortium, organismo que desarrolla y mantiene estándares web como XML, los espacios de nombres o SVG.
- **Parser (procesador):** programa que analiza sintácticamente un documento de marcas.

---

## Referencias y recursos

- **W3C — Extensible Markup Language (XML) 1.0** (especificación oficial): w3.org/TR/xml/
- **W3C — Namespaces in XML 1.0** (especificación oficial): w3.org/TR/xml-names/
- **WHATWG — HTML Living Standard**: html.spec.whatwg.org
- **MDN Web Docs — sección HTML**: developer.mozilla.org/es/docs/Web/HTML
