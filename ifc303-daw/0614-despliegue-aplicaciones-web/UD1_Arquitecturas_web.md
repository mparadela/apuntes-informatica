# 0614 · Despliegue de Aplicaciones Web

## UD1 — Arquitecturas web

Apuntes de la unidad · RA1 (a-i) · 2º DAW · Curso 2026-27

*Estos apuntes desarrollan el contenido de RA1 — Implanta arquitecturas web analizando y aplicando criterios de funcionalidad. Son material de referencia para consolidar lo trabajado sobre Solvia, el artefacto ancla del módulo: un sistema interno de gestión de incidencias/tickets IT que se despliega progresivamente durante todo el curso, capa a capa, con cada UD. No sustituyen la sesión de clase, sino que fijan con precisión lo que en ella se ve en contexto sobre el propio Solvia.*

## 1. Modelos de arquitectura web

### Definición formal

«El modelo de desarrollo web se apoya, en una primera aproximación desde un punto de vista centrado en el hardware, en lo que se conoce como arquitectura cliente-servidor, que define un patrón de arquitectura donde existen dos actores, cliente y servidor» (Santiago Faci, apuntes «Despliegue de Aplicaciones Web», licencia CC BY-NC-SA 4.0). Una arquitectura web, en sentido más amplio, es la organización estructural de los componentes software — y del hardware que los soporta — que intervienen en la publicación, ejecución y consumo de una aplicación accesible mediante HTTP/HTTPS. Definir una arquitectura significa decidir qué responsabilidad asume cada componente — presentación, lógica de negocio, acceso a datos — y cómo se comunican entre sí.

### Explicación desarrollada

- **Cliente-servidor:** el paradigma base de toda la web: el servidor se ejecuta continuamente esperando solicitudes de múltiples clientes, y la comunicación se articula en peticiones (requests) del cliente y respuestas (responses) del servidor. Los roles son asimétricos — el cliente siempre inicia, el servidor siempre atiende — y no comparten memoria ni estado por defecto.

- **Monolítico:** toda la aplicación (interfaz, lógica de negocio, acceso a datos) se empaqueta y despliega como una única unidad, normalmente ejecutándose en un solo proceso sobre un solo servidor.

- **Cliente-servidor de 2 capas:** el cliente se comunica directamente con el servidor de datos, sin una capa intermedia de lógica de negocio explícita.

- **N capas (N-tier), típicamente 3:** presentación (interfaz de usuario) → lógica de negocio (servidor de aplicaciones) → datos (servidor de base de datos). Cada capa puede ejecutarse en un proceso o una máquina distinta, comunicándose por red.

- **Microservicios:** la capa de lógica de negocio se descompone en servicios pequeños e independientes, cada uno con responsabilidad única y a menudo con su propia base de datos, comunicados mediante API (REST, mensajería asíncrona).

Es importante no tratar estos modelos como alternativas excluyentes en un mismo eje: cliente-servidor es el paradigma de comunicación; monolito, N-capas y microservicios son formas distintas de organizar internamente el lado servidor de ese paradigma.

### Ejemplo resuelto

Despliegue monolítico — Solvia (el artefacto ancla del módulo: un sistema interno de gestión de incidencias/tickets IT) tal como se despliega en esta UD1:

```text
[Navegador]  --HTTP-->  [Apache + PHP (mod_php) + MySQL, misma máquina]
```

Despliegue en 3 capas — la misma aplicación Solvia, en la fase en que se trabajará en UD3 al incorporar el servidor de aplicaciones:

```text
[Navegador] -> [Apache :80, proxy] -> [Tomcat :8080] -> [MySQL :3306]
                (resto de Solvia,        (módulo Java,       (datos)
                 PHP)                     p. ej. informes)
```

En UD3, Apache sigue siendo el servidor que recibe todas las peticiones del navegador: la mayor parte de Solvia (PHP) la sirve directamente, y sólo las peticiones dirigidas al módulo Java las reenvía a Tomcat. Es el mismo patrón conceptual de 3 capas aplicado con las tecnologías concretas que usará este módulo — no un ejemplo genérico aparte, sino la propia evolución de Solvia.

### Caso práctico

El artefacto ancla del módulo es Solvia: los usuarios internos abren tickets de incidencia, los técnicos los atienden y existe un panel de administración. En esta UD1, Solvia se despliega como una única capa — Apache + PHP + MySQL en la misma máquina —, con el formulario de alta de tickets como funcionalidad dinámica básica que ya estará en pie antes de la primera sesión. En UD2 se añadirá HTTPS y gestión de logs sobre este mismo despliegue; en UD3, un módulo Java en Tomcat se incorporará detrás de Apache como servidor de aplicaciones independiente. No son ejercicios distintos: es la evolución progresiva del mismo sistema.

### ⚖️ Alternativa y criterio

Monolito/N-capas frente a microservicios: los microservicios añaden complejidad operativa real — descubrimiento de servicios, transacciones distribuidas, latencia de red entre componentes, más superficie que asegurar y monitorizar — que sólo se amortiza a partir de cierta escala de equipo y de tráfico.

Criterio: para un equipo pequeño y un sistema que no necesita escalar sus partes de forma independiente, un monolito bien modularizado o una arquitectura N-capas es la opción preferible sobre microservicios. Se eligen microservicios cuando distintas partes del sistema tienen necesidades de escalado, disponibilidad o cadencia de despliegue claramente distintas, y existe una estructura de equipo que sostenga esa fragmentación.

Este módulo trabaja con arquitecturas monolíticas y N-capas como base, porque es lo que exige Solvia y lo que es realista operar y defender con las horas de clase disponibles.

### ⚠️ Error común

Confundir «cliente-servidor» y «N-capas» como si fueran alternativas del mismo tipo. Cliente-servidor describe quién inicia la comunicación; N-capas describe cómo se organiza internamente el lado servidor. Toda arquitectura N-capas es cliente-servidor, pero no toda arquitectura cliente-servidor tiene varias capas — puede ser perfectamente monolítica, como Solvia en esta UD1.

### 💡 Nota técnica

En 2026, los sistemas cloud-native a gran escala tienden hacia microservicios contenerizados orquestados con Kubernetes, pero el monolito sigue siendo la opción pragmática por defecto en la inmensa mayoría de proyectos reales de tamaño pequeño o mediano. El «monolito modular» — separación interna clara de responsabilidades sin llegar a desplegar servicios independientes — gana terreno como término medio entre ambos extremos.

## 2. Protocolo HTTP/HTTPS y funcionamiento de un servidor web

### Definición formal

«El protocolo HTTP se usa para enviar y recibir datos en la Web» (logongas.es, «El protocolo HTTP», licencia CC BY-SA 4.0). Formalmente, HTTP (HyperText Transfer Protocol) es un protocolo de la capa de aplicación que define un formato de mensajes de petición y de respuesta entre un cliente y un servidor. Es un protocolo sin estado (stateless): cada petición se procesa de forma independiente, sin memoria de peticiones anteriores, y se transporta habitualmente sobre una conexión TCP fiable.

### Explicación desarrollada

- **Posición en la pila de protocolos:** aplicación (HTTP) sobre transporte (TCP, puerto 80 por defecto) sobre red (IP).

- **Características de HTTP (logongas.es):** sencillo — «es en modo texto y fácil de usar directamente por una persona»; extensible — «se pueden enviar más metadatos que los que están por defecto»; sin estado — «cada petición es independiente».

- **Ciclo petición-respuesta:** el cliente abre una conexión TCP con el servidor y envía una petición compuesta por una línea de petición, cabeceras y, opcionalmente, un cuerpo. El servidor la procesa y devuelve una respuesta con una línea de estado, cabeceras y, opcionalmente, un cuerpo. La conexión puede cerrarse o mantenerse abierta para más peticiones (keep-alive).

- **Métodos HTTP:** logongas.es resume los cuatro métodos básicos así — GET: «queremos obtener los datos»; POST: «queremos añadir los datos»; PUT: «queremos actualizar nuevos datos»; DELETE: «queremos borrar los datos». El estándar HTTP añade además HEAD (como GET pero sólo cabeceras, sin cuerpo), PATCH (modificar parcialmente) y OPTIONS (consultar qué métodos admite el servidor para un recurso).

- **Códigos de estado:** por rangos (logongas.es) — 2xx: «la petición ha tenido éxito» (200 «todo ha ido bien», 201 «se ha creado el recurso», 204 «la petición no retorna datos»); 3xx: «redirección de los datos»; 4xx: «los datos que ha enviado el cliente no son correctos» (400 «los datos no son correctos», 401 «hay que estar logueado», 403 «usuario logueado pero con acceso prohibido», 404 «no encuentra el documento»); 5xx: «se ha producido un error en el servidor» (500, y también 502/503 en despliegues con proxy inverso, como el que se usará en UD3).

- **Cabeceras habituales:** Host, User-Agent, Content-Type, Content-Length, Accept, Cookie / Set-Cookie, Authorization, Cache-Control.

- **HTTPS:** «es un protocolo de comunicación segura a través de Internet [...] se basa en la comunicación del protocolo HTTP pero con una capa de seguridad adicional cifrando el contenido con TLS o SSL» (Santiago Faci, CC BY-NC-SA 4.0), y proporciona «un mecanismo de autenticación con respecto al sitio web y servidor web que estamos visitando, evitando así ataques como el del Man in the Middle». Añade confidencialidad, integridad y autenticación del servidor mediante un certificado digital. Puerto por defecto 443. El proceso de obtención e instalación de certificados se trabaja en detalle en UD2 (CE2.e, CE2.f); aquí basta con entender qué añade TLS sobre HTTP plano.

- **Sin estado (stateless):** el servidor no recuerda por sí mismo peticiones anteriores del mismo cliente. Para simular estado — una sesión de técnico autenticado en Solvia, por ejemplo — se recurre a mecanismos añadidos: cookies, sesiones de servidor, tokens.

### Ejemplo resuelto

Petición y respuesta HTTP reales (logongas.es, CC BY-SA 4.0) — formato de petición:

```http
GET /index.html HTTP/1.1
Host: www.fpmislata.com
Accept-Language: fr
```

Formato de respuesta:

```http
HTTP/1.1 200 OK
Content-Length: 29769
Content-Type: text/html; charset=utf-8
 
<!DOCTYPE html... (los 29769 bytes de la página)
```

Es exactamente este formato — línea de petición o de estado, cabeceras, línea en blanco y cuerpo opcional — el que se verá con `curl -v` contra Solvia en la sesión de clase de esta UD, cambiando el Host por la dirección del servidor Apache de cada alumno.

Segundo ejemplo, ilustrativo (no una traza capturada): una petición POST de alta de ticket en Solvia, anticipando la funcionalidad que ya se desplegará en esta UD1.

```bash
$ curl -v -X POST http://<servidor-solvia>/api/tickets \
     -H "Content-Type: application/json" \
     -d '{"asunto":"No conecta a la impresora","prioridad":"media"}'
> POST /api/tickets HTTP/1.1
> Content-Type: application/json
>
< HTTP/1.1 201 Created
< Location: /api/tickets/48
< Content-Type: application/json
<
{"id":48,"asunto":"No conecta a la impresora","estado":"abierto"}
```

### Caso práctico

Escalera de diagnóstico cuando «Solvia no funciona»: si el navegador no llega a mostrar ningún código de estado (error de red, DNS, servidor caído), el problema está por debajo de HTTP. Si se recibe un 404 («no encuentra el documento»), el servidor respondió — está vivo y accesible — pero no encuentra el recurso solicitado (ruta mal escrita, archivo no desplegado). Si se recibe un 500, el servidor recibió la petición y llegó a ejecutar código de la aplicación, pero ese código falló. Diferenciar estos tres escalones es el primer paso de cualquier depuración de un despliegue.

### ⚖️ Alternativa y criterio

HTTP/1.1 frente a HTTP/2 y HTTP/3: HTTP/1.1 abre una petición por conexión (salvo keep-alive); HTTP/2 multiplexa varias peticiones sobre una sola conexión TCP y comprime cabeceras, reduciendo notablemente la latencia cuando el cliente pide muchos recursos a la vez; HTTP/3 sustituye TCP por QUIC (sobre UDP), eliminando el bloqueo de cabeza de línea a nivel de transporte y mejorando el rendimiento en redes inestables.

Este módulo trabaja explícitamente con HTTP/1.1 como base, porque expone en texto plano el mecanismo petición-respuesta que hay que entender antes de nada. Apache soporta HTTP/2 por defecto — se puede observar como una mejora añadida, no como sustituto del modelo conceptual.

### ⚠️ Error común

Confundir un 403 («usuario logueado pero con acceso prohibido») con un 404 («no encuentra el documento»). El 403 significa que el servidor entendió la petición y localizó el recurso, pero deniega el acceso — típicamente por permisos de archivo o reglas de configuración. El 404 significa que el servidor no encuentra el recurso en absoluto. Tratar ambos como «no funciona» sin distinguirlos impide saber si hay que revisar permisos, configuración de rutas, o si el recurso simplemente no existe donde se espera.

### 💡 Nota técnica

En 2026 la inmensa mayoría del tráfico web de producción usa HTTPS por defecto: los navegadores marcan como «no seguro» cualquier sitio servido en HTTP plano, y los buscadores penalizan su posicionamiento. La disponibilidad de certificados gratuitos y automatizados (Let's Encrypt, protocolo ACME) ha eliminado prácticamente cualquier justificación operativa para servir HTTP sin cifrar fuera de entornos de desarrollo o de aula aislados de la red pública — como el que se usa en este módulo antes de introducir TLS en UD2.

## 3. Instalación y configuración básica de un servidor web (Apache)

### Definición formal

Instalar y configurar un servidor web consiste en poner en marcha el software que escucha peticiones HTTP en un puerto determinado y las resuelve sirviendo contenido estático o delegando su generación en un intérprete (como PHP), dejando el servicio operativo, seguro por defecto y verificable.

### Explicación desarrollada

- **Paquete y proceso:** en Debian/Ubuntu, apt install apache2 instala el binario apache2, lo registra como servicio systemd (apache2.service) y lo deja escuchando en el puerto 80 nada más terminar la instalación.

- **Estructura de configuración en Debian/Ubuntu:** /etc/apache2/apache2.conf (configuración global), sites-available/ y sites-enabled/ (VirtualHosts, activados mediante enlaces simbólicos con a2ensite), mods-available/ y mods-enabled/ (módulos, activados con a2enmod), ports.conf (puertos de escucha).

- **DocumentRoot:** la carpeta que Apache expone al navegador; todo lo que quede fuera de ella no es alcanzable por HTTP aunque exista en el disco — es el mismo principio ya aplicado en Solvia (DocumentRoot → public/, nunca solvia/ a secas).

- **VirtualHost:** bloque de configuración que asocia un nombre de dominio (o una IP:puerto) con un DocumentRoot y unas opciones propias, permitiendo servir varios sitios distintos desde el mismo Apache.

- **mod_php frente a PHP-FPM:** dos formas de que Apache ejecute PHP. mod_php lo embebe directamente en el proceso de Apache (más simple: cada proceso Apache carga el intérprete PHP). PHP-FPM lo ejecuta como proceso independiente, y Apache le reenvía la petición vía mod_proxy_fcgi (más aislado y eficiente en producción, con más piezas que configurar).

- **Verificación antes de aplicar cambios:** apachectl configtest (o apache2ctl configtest) comprueba que la configuración es sintácticamente correcta sin aplicarla — evita tumbar el servicio por un error de sintaxis.

- **Recarga frente a reinicio:** systemctl reload apache2 aplica cambios de configuración sin cortar las conexiones activas; systemctl restart apache2 detiene y vuelve a levantar el proceso entero.

### Ejemplo resuelto

VirtualHost real para servir Solvia, apuntando el DocumentRoot a public/ tal como exige su README:

```apache
<VirtualHost *:80>
    ServerName solvia.local
    DocumentRoot /var/www/solvia/public

    <Directory /var/www/solvia/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/solvia_error.log
    CustomLog ${APACHE_LOG_DIR}/solvia_access.log combined
</VirtualHost>
```

Para activarlo:

```bash
sudo a2ensite solvia.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

### Caso práctico

Despliegue de Solvia en el servidor de aula (Ubuntu Server, ver estado del módulo): instalar apache2, php, libapache2-mod-php y php-mysql; instalar y arrancar mysql-server; importar db/schema.sql; copiar el código de Solvia fuera de la ruta servida por defecto; crear el VirtualHost de arriba; desactivar el sitio por defecto con a2dissite 000-default para que no compita por el puerto 80; recargar Apache. Es la misma secuencia de la demo de María en Windows con XAMPP, pero con los comandos reales de un sistema Linux de aula.

### ⚖️ Alternativa y criterio

mod_php frente a PHP-FPM: para el tamaño de Solvia en esta UD1 y los recursos limitados de un aula con 9 despliegues simultáneos, mod_php es preferible por su sencillez operativa — menos piezas que puedan fallar en una sesión de clase con poco margen de tiempo. PHP-FPM se preferiría en un entorno de producción real, por su mejor aislamiento y rendimiento bajo carga, pero esa ventaja no compensa aquí la complejidad añadida.

Apache frente a Nginx: la fuente de referencia más reciente del módulo (raul-profesor.github.io/Despliegue, Tema 2) construye su práctica guiada sobre Nginx. Se mantiene Apache porque Solvia ya está construido y probado sobre él, el resto de la planificación de UD1-UD3 lo da por hecho, y no hay ningún motivo curricular que obligue a cambiar de servidor a mitad de curso (ver estado del módulo, decisión del 15 sept 2026).

### ⚠️ Error común

Activar un VirtualHost nuevo sin desactivar el sitio por defecto (000-default), lo que puede provocar que Apache siga sirviendo la página de bienvenida en vez de Solvia si el nombre de servidor no coincide exactamente. Igual de habitual: dejar el DocumentRoot apuntando a la carpeta raíz del proyecto en vez de a public/ — repite el fallo de seguridad ya señalado al hablar de la estructura de Solvia.

### 💡 Nota técnica

En 2026, Apache 2.4.x sigue siendo el servidor web más usado en entornos educativos y de pyme en España, con HTTP/2 disponible de serie activando el módulo mod_http2 — una mejora de rendimiento que no cambia el modelo conceptual petición-respuesta ya visto en el bloque 2.

## 4. Servidor de aplicaciones — introducción conceptual (Tomcat)

### Definición formal

Un servidor de aplicaciones es el software que aloja y ejecuta la lógica de negocio de una aplicación —en este caso, código Java empaquetado como servlets o aplicaciones web— gestionando su ciclo de vida (arranque, hilos, memoria) de forma independiente del servidor web que atiende las peticiones HTTP de entrada.

### Explicación desarrollada

- **Servidor web frente a servidor de aplicaciones:** Apache sirve contenido estático y delega en intérpretes ligeros como PHP; un servidor de aplicaciones como Tomcat ejecuta código compilado (bytecode Java) sobre una JVM, con gestión de sesiones, hilos y recursos propia.

- **Tomcat como contenedor de servlets:** implementa las especificaciones Jakarta Servlet y JSP (herencia de las antiguas javax.servlet, renombradas tras ceder Oracle Java EE a la Eclipse Foundation). No es un servidor Java EE completo — no incluye EJB ni JMS por defecto —, es más ligero, pensado justo para servlets y JSP.

- **Empaquetado y despliegue:** una aplicación Tomcat se empaqueta como fichero .war (Web Application Archive) y se coloca en el directorio webapps/ de Tomcat; este lo descomprime y lo pone en marcha automáticamente sin reiniciar el servidor.

- **Puerto por defecto:** Tomcat escucha en el 8080, para no competir con Apache en el puerto 80.

- **El patrón reverse proxy:** en una arquitectura real, Apache (o Nginx) queda como único punto de entrada en el puerto 80/443 y reenvía —mediante mod_proxy y mod_proxy_http— las peticiones dirigidas a rutas concretas hacia Tomcat en el 8080. El navegador nunca habla directamente con Tomcat.

- **Aplicación a Solvia:** este es exactamente el patrón que Solvia adoptará en UD3 — Apache seguirá sirviendo la mayor parte de la aplicación (PHP), y solo las rutas de un módulo Java nuevo (por ejemplo, informes o estadísticas de tickets) se reenviarán a Tomcat.

### Ejemplo resuelto

Fragmento conceptual de cómo se vería, en UD3, la configuración de Apache que reenvía a Tomcat (no se instala todavía en esta UD1):

```apache
ProxyPass /informes http://localhost:8080/solvia-informes
ProxyPassReverse /informes http://localhost:8080/solvia-informes
```

Requiere tener activados los módulos mod_proxy y mod_proxy_http (a2enmod proxy proxy_http) antes de recargar Apache.

### Caso práctico

En UD3 se empaquetará un módulo Java sencillo de Solvia (por ejemplo, un generador de estadísticas de tickets) como solvia-informes.war, se colocará en webapps/ de Tomcat, y Apache se configurará con el ProxyPass de arriba para que /informes en el mismo dominio de Solvia sea, en realidad, resuelto por Tomcat sin que el usuario lo note. En esta UD1 el contenido se queda en lo conceptual — no hay instalación real de Tomcat todavía.

### ⚖️ Alternativa y criterio

Tomcat frente a un servidor Java EE completo (WildFly, antiguo JBoss): Tomcat es preferible aquí porque Solvia solo necesita servlets y JSP simples, sin EJB ni colas de mensajería, y es más ligero de instalar y mantener con los recursos de un aula. WildFly se justificaría si el proyecto necesitara servicios transaccionales distribuidos o inyección de dependencias Java EE completa, que no es el caso de Solvia.

### ⚠️ Error común

Pensar que "servidor de aplicaciones" significa "servidor más potente". No es una cuestión de potencia sino de responsabilidad: qué lenguaje y tecnología ejecuta, y cómo gestiona el ciclo de vida de ese código — no cuántas peticiones por segundo aguanta.

### 💡 Nota técnica

La migración de javax.* a jakarta.* (tras la cesión de Java EE a la Eclipse Foundation) es la razón por la que Tomcat 10 en adelante ya no acepta directamente código escrito con las antiguas importaciones javax.servlet.* — hay que actualizarlas a jakarta.servlet.*. Conviene tenerlo presente cuando se elija la versión de Tomcat para el módulo Java de UD3.

## 5. Estructura y recursos de una aplicación web (Solvia: PHP + MySQL)

### Definición formal

La estructura de una aplicación web es la organización de sus ficheros y carpetas en el sistema, de forma que se separen responsabilidades —código servido públicamente, lógica de negocio, configuración sensible, plantillas de presentación, recursos de datos— y solo quede expuesto al servidor web lo estrictamente necesario para responder peticiones HTTP.

### Explicación desarrollada

- **config/ — **configuración de conexión a la base de datos (config.php, generado a partir de config.example.php y nunca subido al repositorio). Vive fuera de public/ para que Apache no pueda servirlo aunque alguien pida la ruta directamente.

- **db/schema.sql — **script SQL que crea la base de datos solvia y sus tablas (categorias, tickets), con datos de ejemplo ya cargados.

- **src/ — **lógica de negocio pura, sin HTML: Database.php (conexión PDO única, patrón singleton sencillo), TicketRepository.php (todo el acceso a datos: listar, buscar por código, crear), TicketValidator.php (validación del formulario, separada del controlador).

- **templates/ — **las vistas: HTML con el mínimo PHP de impresión necesario (layout_header.php / layout_footer.php, home.php, ticket_form.php, ticket_lookup.php, partials/ticket_tags.php).

- **bootstrap.php — **autoload de clases: registra una función que, ante un new NombreClase(), busca automáticamente src/NombreClase.php, evitando un require manual por cada clase que se usa.

- **public/ — **la única carpeta que debe apuntar el DocumentRoot de Apache: los controladores (index.php, nuevo_ticket.php, consultar_ticket.php) y los recursos estáticos (assets/css, assets/js).

### Ejemplo resuelto

El controlador de la portada (public/index.php), código real de Solvia:

```php
<?php
require __DIR__ . '/../bootstrap.php';

$repo = new TicketRepository(Database::conexion());

$pageTitle = 'Inicio';
$ultimosTickets = $repo->recientes(10);

require __DIR__ . '/../templates/layout_header.php';
require __DIR__ . '/../templates/home.php';
require __DIR__ . '/../templates/layout_footer.php';
```

El flujo es siempre el mismo: bootstrap.php activa el autoload; se instancia el repositorio con la conexión de Database::conexion(); se piden los datos que hagan falta; y por último se delega el pintado a las plantillas, en orden (cabecera, contenido, pie). El controlador no contiene SQL ni construye HTML directamente — es un "controlador delgado" que solo orquesta.

### Caso práctico

El alta de un ticket (public/nuevo_ticket.php) sigue el mismo patrón, pero con validación: si la petición es POST, TicketValidator revisa los datos recibidos; si hay errores, se vuelve a mostrar el formulario con los datos ya escritos por el usuario (para que no tenga que volver a teclearlo todo) y la lista de errores; si todo es correcto, TicketRepository crea el ticket, genera su código (formato SOLV-000003, siguiente número libre) y se muestra la confirmación. Ningún paso de este flujo mezcla acceso a datos con presentación.

### ⚖️ Alternativa y criterio

Se podría haber escrito toda la lógica dentro de los propios ficheros de public/, mezclando SQL, validación y HTML en el mismo archivo — como hacían muchas aplicaciones PHP clásicas. Se prefiere la separación en config/src/templates/public porque cada pieza puede revisarse, probarse o sustituirse por separado, y es exactamente la práctica que se os va a pedir justificar en la bitácora de decisiones cuando defendáis vuestra propia estructura de despliegue.

### ⚠️ Error común

Colocar config/ o src/ dentro de public/ "porque es más cómodo" durante las pruebas, y olvidarse de moverlo antes de dar el despliegue por terminado — expone el código fuente y las credenciales de base de datos a cualquiera que conozca o adivine la ruta.

### 💡 Nota técnica

Esta separación manual (autoload propio + config/src/templates/public) es una versión simplificada de lo que frameworks PHP modernos como Laravel o Symfony hacen con Composer y un autoload PSR-4. Entender la versión manual ayuda a entender después por qué esos frameworks estructuran los proyectos exactamente así.

## 6. Requerimientos del proceso de implantación de una aplicación web

### Definición formal

Los requerimientos de implantación son el conjunto de condiciones —de software, datos, red y configuración— que deben cumplirse en el entorno de destino antes de poner en marcha una aplicación, de forma que su comportamiento no dependa de casualidades del equipo donde se desarrolló originalmente.

### Explicación desarrollada

- **Requisitos de software:** versión de PHP y extensiones necesarias (para Solvia, en concreto pdo_mysql), versión de MySQL/MariaDB, y los módulos de Apache que la aplicación necesita.

- **Requisitos de datos:** que la base de datos exista y su esquema ya esté creado (schema.sql), con las credenciales correctas configuradas — sin esto la aplicación arranca pero falla en el primer acceso a datos.

- **Requisitos de red y puertos:** que el puerto elegido (80 por defecto) esté libre. En un aula donde cada alumno despliega en su propia máquina esto no suele chocar, pero sí sería un requerimiento a resolver si varias instancias compartieran un mismo servidor.

- **Requisitos del sistema de ficheros:** permisos de lectura para Apache sobre public/ (y de escritura si la aplicación necesitara subir ficheros, que Solvia en RA1 no hace).

- **Configuración específica de cada entorno:** el fichero config.php es exactamente esto — cada entorno (tu portátil, el aula, un servidor real) tiene sus propias credenciales sin tocar una línea de código.

- **Por qué documentarlos antes de implantar:** evita el clásico "en mi máquina funciona" — si nadie deja escrito qué versión de PHP hace falta o qué extensiones, cada despliegue nuevo es un ensayo y error.

### Ejemplo resuelto

Checklist de requerimientos reales de Solvia, tal como los declara su propio README:

- Apache 2.4.x con mod_php activo
- PHP con la extension pdo_mysql instalada
- MySQL/MariaDB en marcha, accesible con las credenciales de config.php
- DocumentRoot del VirtualHost apuntando a solvia/public (nunca a solvia/ a secas)
- config/config.php creado a partir de config.example.php, con credenciales validas
- Base de datos "solvia" importada desde db/schema.sql

### Caso práctico

Si un alumno despliega Solvia sin tener instalada la extensión pdo_mysql de PHP, la aplicación falla con un error fatal en cuanto intenta conectar. Si en cambio la extensión está instalada pero no se ha creado config/config.php, es el propio código de Database.php el que lanza una excepción explícita ("Falta config/config.php. Copia config/config.example.php...") en lugar de un error críptico — un ejemplo de cómo anticipar requerimientos en el propio código ayuda a quien despliega la aplicación.

### ⚖️ Alternativa y criterio

Documentar los requerimientos en prosa libre (como en un correo) frente a un checklist estructurado y verificable: se prefiere el checklist porque permite comprobar antes de desplegar, no solo leer y confiar. En el bloque 8 se retoma esta misma idea aplicada a la documentación completa del proceso.

### ⚠️ Error común

Confundir "requerimientos de implantación" con "requerimientos funcionales". Los primeros son sobre el entorno necesario para que la aplicación llegue a arrancar; los segundos son sobre qué funcionalidades ofrece una vez ya está en marcha. Son preguntas distintas y se responden en momentos distintos del proceso.

### 💡 Nota técnica

En entornos profesionales este checklist se automatiza con herramientas de infraestructura como código (Ansible, Terraform), que comprueban y crean el entorno exacto necesario antes de desplegar. Queda fuera del alcance de esta UD1, pero es el mismo principio que aquí se practica a mano.

## 7. Pruebas de funcionamiento básicas del servidor web desplegado

### Definición formal

Las pruebas de funcionamiento son la verificación sistemática, tras la implantación, de que el servidor web y la aplicación responden como se espera —desde que el proceso está en marcha hasta que cada funcionalidad concreta se comporta correctamente— antes de dar el despliegue por válido.

### Explicación desarrollada

- **Nivel 1 — el proceso está vivo:** systemctl status apache2 confirma que el servicio está activo (active (running)); si no lo está, journalctl -u apache2 o los logs de error indican la causa.

- **Nivel 2 — el puerto escucha:** ss -tlnp | grep :80 muestra si algo está escuchando en el puerto esperado — útil para detectar conflictos si otro servicio ya ocupa ese puerto.

- **Nivel 3 — HTTP responde:** curl -I http://localhost/ (o el navegador) debe devolver una cabecera con código 200. La escalera de diagnóstico por código de estado ya se vio en el bloque 2 de estos apuntes: sin respuesta indica un problema de red o de proceso; 404, que el recurso no se encuentra; 500, que la aplicación falló al ejecutarse.

- **Nivel 4 — la aplicación funciona de verdad:** para Solvia esto significa probar el flujo real: cargar la portada y ver los tickets de ejemplo, dar de alta un ticket nuevo con el formulario, y consultarlo por su código. No basta con que la portada cargue.

- **Registro de la prueba:** cada verificación debería quedar anotada —qué se probó, cuándo, con qué resultado—, que es exactamente lo que se formaliza en el bloque 8 de documentación.

### Ejemplo resuelto

Secuencia real de comprobación tras desplegar Solvia:

```bash
sudo systemctl status apache2
ss -tlnp | grep :80
curl -I http://solvia.local/
curl -I http://solvia.local/nuevo_ticket.php
```

Un 200 OK en las dos peticiones curl confirma que Apache sirve tanto la portada como el controlador de alta de tickets — aunque, como se ve en el caso práctico, un 200 en el formulario no garantiza todavía que el alta funcione de verdad hasta probarlo con datos reales.

### Caso práctico

Un alumno despliega Solvia: la portada carga bien, pero al enviar el formulario de un ticket nuevo recibe un error 500. La escalera de diagnóstico dice que el servidor recibió la petición y ejecutó código, pero ese código falló. El siguiente paso es mirar el log de error configurado en el VirtualHost del bloque 3 (tail -f /var/log/apache2/solvia_error.log) para ver la excepción real de PHP — que, siguiendo el bloque 6, probablemente sea la falta de config/config.php o de la extensión pdo_mysql.

### ⚖️ Alternativa y criterio

Probar manualmente con curl y el navegador frente a un script de pruebas automatizado que repita siempre las mismas comprobaciones: para esta UD1, con una única instancia y un despliegue puntual, las pruebas manuales son suficientes y más rápidas de aprender. Un script automatizado se justifica cuando el despliegue se repite muchas veces o forma parte de un proceso continuo, contenido que llega en UD6 con CI/CD.

### ⚠️ Error común

Dar por buena una implantación solo porque "la portada carga". La portada de Solvia es una página de solo lectura sencilla; el formulario de alta, que escribe en la base de datos, es una prueba mucho más exigente y es donde suelen aparecer los fallos reales de configuración.

### 💡 Nota técnica

Herramientas como Apache JMeter, mencionadas en materiales de referencia más antiguos del módulo, permiten automatizar pruebas de carga —no solo de funcionamiento— para responder no solo "funciona" sino "aguanta el tráfico esperado". Queda fuera del alcance de esta UD1.

## 8. Documentación del proceso de instalación y configuración realizado

### Definición formal

Documentar el proceso de instalación y configuración es dejar constancia escrita, reproducible y verificable de qué se instaló, con qué versiones, qué configuración se aplicó y qué pruebas confirmaron que funcionaba, de forma que cualquier persona —incluido el propio autor, meses después— pueda repetir o auditar el despliegue sin depender de la memoria de quien lo hizo.

### Explicación desarrollada

- **Qué debe incluir como mínimo:** software instalado y versión (Apache, PHP, MySQL/MariaDB), configuración aplicada (VirtualHost, módulos activados), pasos de despliegue de la aplicación (dónde se copió el código, cómo se creó la base de datos, cómo se configuraron las credenciales) y las pruebas de funcionamiento realizadas, con su resultado.

- **Formato:** puede ser un README técnico —como el que ya trae Solvia— o un documento de despliegue independiente. Lo importante no es el formato sino que sea reproducible: quien lo siga desde cero, en una máquina limpia, debe llegar al mismo resultado.

- **Relación con la bitácora de decisiones del módulo:** no son lo mismo. La documentación técnica describe qué se hizo y cómo; la bitácora de decisiones justifica por qué se eligió hacerlo así frente a otras opciones, y es la que se defiende oralmente. Un buen despliegue necesita las dos.

- **Precisión antes que extensión:** un README breve con comandos exactos, versiones concretas y una sección de pruebas realizadas es mucho más útil que documentación extensa pero vaga.

### Ejemplo resuelto

El propio README.md de Solvia (ya en vuestra carpeta) es el modelo a seguir: declara qué es el proyecto, su estructura de carpetas con el motivo de cada separación, los pasos exactos de instalación con los comandos literales, y qué se ha probado antes de entregarlo. Documentar vuestro propio despliegue en el aula es escribir la misma clase de documento, pero con vuestras versiones, vuestra máquina y vuestras pruebas reales.

### Caso práctico

Tras desplegar vuestra propia instancia de Solvia en la sesión 3, completaréis una plantilla de documentación con estos campos exactos: fecha del despliegue; versión instalada de Apache, PHP y MySQL (comprobada con apache2 -v, php -v y mysql --version); configuración del VirtualHost aplicada; y resultado de cada una de las pruebas del bloque 7. Esta plantilla es, en la práctica, la base de la primera auditoría de RA1.

### ⚖️ Alternativa y criterio

Documentación generada automáticamente (capturando la salida de los comandos de instalación con un script) frente a documentación redactada a mano: para esta UD1 se prefiere redactarla a mano, porque el objetivo pedagógico es que entendáis y sepáis explicar cada paso, no solo que quede registrado. La documentación automatizada se retoma como contenido explícito en RA6.

### ⚠️ Error común

Documentar solo el resultado final ("Apache y MySQL instalados, todo funciona") sin documentar el proceso ni las versiones. No es reproducible ni auditable — es precisamente el tipo de informe de aspecto correcto pero sin sustancia que a este grupo ya le costó defender el curso pasado.

### 💡 Nota técnica

Esta misma disciplina de documentación es la que en RA6 se formaliza con control de versiones: cada commit de Git con un mensaje claro es, en el fondo, documentación incremental del proceso. Aquí en UD1 se practica en prosa; más adelante, en historial de Git.

## Glosario

- **Arquitectura cliente-servidor:** paradigma de comunicación en el que un cliente inicia peticiones y un servidor las atiende; no dice nada sobre cuántas capas o máquinas hay detrás del servidor.

- **Monolito:** modelo en el que toda la aplicación (interfaz, lógica de negocio, acceso a datos) se despliega como una única unidad, normalmente en un solo proceso.

- **N-capas (N-tier):** modelo en el que las responsabilidades de la aplicación (presentación, lógica de negocio, datos) se separan en procesos o máquinas distintas que se comunican por red.

- **Microservicios:** modelo en el que la lógica de negocio se descompone en servicios pequeños e independientes, cada uno con responsabilidad única y a menudo su propia base de datos.

- **HTTP:** protocolo de la capa de aplicación que define el formato de los mensajes de petición y respuesta entre cliente y servidor; sin estado (stateless).

- **HTTPS:** HTTP transportado sobre una capa de cifrado (TLS/SSL), que añade confidencialidad, integridad y autenticación del servidor mediante certificado digital.

- **Stateless (sin estado):** característica de un protocolo o servicio en la que cada petición se procesa de forma independiente, sin memoria de peticiones anteriores.

- **Servidor web:** software que escucha peticiones HTTP en un puerto y las resuelve sirviendo contenido estático o delegando su generación en un intérprete.

- **DocumentRoot:** carpeta que un servidor web expone al navegador; lo que queda fuera de ella no es alcanzable por HTTP.

- **VirtualHost:** bloque de configuración de Apache que asocia un dominio (o IP:puerto) con un DocumentRoot y opciones propias, permitiendo servir varios sitios desde el mismo servidor.

- **mod_php / PHP-FPM:** dos formas de que Apache ejecute PHP: embebido en el propio proceso de Apache (mod_php) o como proceso independiente al que Apache reenvía peticiones (PHP-FPM).

- **Servidor de aplicaciones:** software que aloja y ejecuta la lógica de negocio de una aplicación (por ejemplo, código Java) gestionando su ciclo de vida de forma independiente del servidor web.

- **Servlet:** componente Java que procesa peticiones y genera respuestas dentro de un contenedor como Tomcat, siguiendo la especificación Jakarta Servlet.

- **WAR (Web Application Archive):** formato de empaquetado de una aplicación Java web, que un servidor como Tomcat descomprime y pone en marcha automáticamente.

- **Reverse proxy (proxy inverso):** servidor que recibe las peticiones del cliente y las reenvía a otro servidor interno, devolviendo la respuesta como si fuera propia; el cliente nunca contacta directamente con el servidor real.

- **PDO (PHP Data Objects):** extensión de PHP que ofrece una interfaz común para conectar con distintas bases de datos mediante consultas preparadas.

- **Autoload:** mecanismo por el que un lenguaje carga automáticamente el fichero de una clase la primera vez que se usa, sin necesidad de un require manual.

- **Requerimientos de implantación:** condiciones de software, datos, red y configuración que deben cumplirse en el entorno de destino antes de poner en marcha una aplicación.

- **Pruebas de funcionamiento:** verificación, tras la implantación, de que el servicio y la aplicación responden como se espera, en niveles progresivos desde el proceso hasta la funcionalidad completa.

- **Documentación del despliegue:** registro escrito y reproducible de qué se instaló, con qué versiones, qué configuración se aplicó y qué pruebas lo confirmaron.

## Referencias y recursos adicionales

*Las fuentes se citan aquí de forma consolidada, en lugar de repetirlas tras cada bloque. El formato sigue el estilo APA en lo posible; para las páginas web sin fecha de publicación indicada se usa «(s.f.)».*

- **Apache Software Foundation. (s.f.). **Apache HTTP Server documentation. https://httpd.apache.org/docs/ — Documentación oficial consultada para VirtualHost, módulos mod_proxy/mod_proxy_http y registro de logs (usada en los bloques 3, 4 y 7).

- **Apache Software Foundation. (s.f.). **Apache Tomcat documentation. https://tomcat.apache.org/ — Documentación oficial sobre despliegue de aplicaciones WAR y la especificación Jakarta Servlet (usada en el bloque 4).

- **Faci, S. (s.f.). **Despliegue de Aplicaciones Web [apuntes de la asignatura]. https://despliegue.codeandcoke.com — Licencia CC BY-NC-SA 4.0 (uso docente no comercial, citando autoría; usada en los bloques 1 y 2).

- **logongas.es. (s.f.). **El protocolo HTTP. https://logongas.es — Licencia CC BY-SA 4.0, reutilizable citando autoría (usada en el bloque 2).

- **raul-profesor. (s.f.). **Despliegue de Aplicaciones Web, Tema 2: Arquitectura y servidores web. https://raul-profesor.github.io/Despliegue — Sin licencia indicada en el repositorio; usado como referencia de contenido actualizado, no como texto citado literalmente, adaptando su práctica de Nginx al Apache real de Solvia (ver auditoria-materiales-referencia.md, informe 5; usada en el bloque 3).

- **Solvia [proyecto de referencia del módulo]. (2026). **README.md y código fuente (Database.php, TicketRepository.php y otros), en la carpeta del módulo — fuente primaria, estructura y comandos ya probados end-to-end (usada en los bloques 5, 6, 7 y 8).
