# UD2 — Administración de servidores web · Apuntes

**0614 · Despliegue de Aplicaciones Web · 2º DAW · IES Río Arba · Curso 2026-27**

**RA2 — Implanta aplicaciones web en servidores web, evaluando y aplicando criterios de configuración para su funcionamiento seguro.**

## Cómo usar estos apuntes

Estos apuntes están escritos para poder aprender la unidad **sin haber asistido a las clases**. Todo lo que hace falta para seguir cada apartado está dentro del texto: las explicaciones, los comandos con su salida esperada y los ejemplos completos. Si tienes un servidor Ubuntu con Apache, puedes reproducir cada ejemplo y comparar lo que ves con lo que aquí se muestra.

Cada apartado sigue el mismo orden: qué es la cosa y para qué sirve, cómo funciona, un ejemplo resuelto, un caso práctico sobre Solvia, la alternativa que se descarta y por qué, el error más frecuente, una nota técnica y, al final, unas preguntas para que compruebes que lo has entendido.

Todo lo que aparece se ha probado sobre **Ubuntu Server 24.04 con Apache 2.4.58, PHP 8.3.6, MariaDB 10.11 y OpenSSL 3.0.13**, con el código real de Solvia. Si tu máquina tiene otras versiones, pueden cambiar los números de versión que ves en las salidas. Los comandos son los mismos.

## Qué necesitas saber antes de empezar

Esta unidad parte de lo que se trabajó en la unidad anterior (UD1, arquitecturas web). Si no lo tienes claro, repásalo aquí antes de seguir.

**Línea de comandos de Linux.** Se usan `cat` (mostrar un fichero), `grep` (buscar texto dentro de ficheros), `cp` (copiar), `ls` (listar), `nano` (editor de texto en la terminal) y `sudo` (ejecutar un comando con permisos de administrador). Cada vez que aparece un comando con opciones, estos apuntes explican qué hace cada una.

**HTTP.** Una conversación web consiste en una **petición** del cliente (un método como `GET` o `POST`, una dirección y unas cabeceras) y una **respuesta** del servidor (un código de estado como `200`, `404` o `500`, unas cabeceras y, normalmente, un cuerpo con la página). Las **cabeceras** son líneas `Nombre: valor` que acompañan a la petición y a la respuesta y llevan metadatos: tipo de contenido, fecha, y también información sobre el propio servidor.

**Apache y los sitios.** Apache HTTP Server es un programa servidor web. En Ubuntu se instala con el paquete `apache2`, se controla con `systemctl` y escucha por defecto en el puerto 80. Un **sitio virtual** (*VirtualHost*) es un bloque de configuración que define un sitio web concreto: con qué nombre responde, qué carpeta sirve y con qué opciones. El **DocumentRoot** es la carpeta del disco que Apache sirve como raíz de ese sitio.

**Solvia, el proyecto del módulo.** Solvia es una aplicación web interna de gestión de incidencias (tickets de soporte informático), escrita en PHP clásico con base de datos MariaDB. En la máquina del alumnado está instalada en `/var/www/solvia/` con esta estructura:

```plaintext
/var/www/solvia/
├── bootstrap.php   ← arranque común de la aplicación
├── config/         ← configuración, incluida la contraseña de la base de datos
├── db/             ← esquema de la base de datos
├── public/         ← la única carpeta que Apache sirve al navegador (DocumentRoot)
├── README.md
├── src/            ← código PHP de la aplicación
└── templates/      ← plantillas de las páginas
```

Y se sirve con un sitio virtual como este, que es el punto de partida de toda la unidad:

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

El bloque `<Directory>` fija las reglas de esa carpeta: `AllowOverride All` permite que ficheros `.htaccess` modifiquen la configuración, y `Require all granted` permite el acceso a cualquier cliente. En los ejemplos se usa `solvia.local`, el nombre del sitio en UD1; si el tuyo es otro, sustitúyelo.

## Convenciones

- Los comandos que tienes que escribir van en un recuadro marcado como «comando». Justo debajo, en otro marcado como «salida», está lo que debe aparecer en pantalla.
- Los comandos que cambian algo en el sistema llevan `sudo` delante. Sin él, Linux responde con un error de permisos.
- Los números de línea de las salidas (`12:`, `23:`) pueden ser distintos en tu máquina. Lo importante es el texto.

## Índice

1. Parámetros de administración del servidor web *(CE 2.a)*
2. Módulos: ampliar la funcionalidad del servidor *(CE 2.b)*
3. Sitios virtuales *(CE 2.c)*
4. Ajustes para la implantación de aplicaciones *(CE 2.h)*
5. Autenticación y control de acceso *(CE 2.d)*
6. Certificados digitales *(CE 2.e)*
7. Asegurar las comunicaciones entre cliente y servidor *(CE 2.f)*
8. Gestión de logs: monitorización, consolidación y análisis *(CE 2.j)*
9. Documentación, administración segura y recomendaciones de uso *(CE 2.g)*

Glosario · Referencias

*La CE 2.i (virtualización en la nube y en contenedores) se desarrolla en la unidad de Docker.*

---

## 1. Parámetros de administración del servidor web

### 1.1 Qué son y por qué importan

Un **parámetro de administración** es una opción de configuración que gobierna el comportamiento global del servidor, es decir, el de Apache como programa, con independencia del sitio que esté sirviendo. Responde a preguntas como: ¿en qué puertos escucha?, ¿con qué usuario del sistema se ejecutan los procesos que atienden a los clientes?, ¿cuánto espera a un cliente lento?, ¿cuánta información cuenta de sí mismo en cada respuesta?, ¿dónde anota lo que ocurre?

En Apache cada parámetro es una **directiva**: una línea de texto con el nombre de la opción y uno o varios valores, por ejemplo `Timeout 300` o `Listen 80`. Las líneas que empiezan por `#` son comentarios y Apache las ignora. Cada directiva solo es válida en determinados **contextos** (apartado 1.4); escribirla en un sitio donde no se admite produce un error de sintaxis.

Un administrador tiene que conocer estos parámetros por tres razones. La primera es **funcional**: son los que hacen que el servidor atienda, o deje de atender, a quien debe. La segunda es de **seguridad**: los valores por defecto de Apache y de Ubuntu están pensados para que todo funcione al instalar, no para que el servidor revele lo mínimo; de serie, por ejemplo, cada respuesta anuncia la versión exacta del servidor. La tercera es de **mantenimiento**: quien herede un servidor necesita poder distinguir lo que se ha tocado de lo que viene de fábrica. Por eso en este apartado se aprende primero a leer la configuración, después a cambiarla con un método fijo y, por último, a justificar cada decisión.

### 1.2 Dónde vive la configuración de Apache en Ubuntu

Toda la configuración está en `/etc/apache2/`. Ubuntu (como Debian) reparte la configuración de Apache en muchos ficheros pequeños en lugar de uno grande, y los organiza así:

```plaintext
/etc/apache2/
├── apache2.conf          ← fichero principal: parámetros globales e inclusiones
├── envvars               ← variables de entorno: usuario, grupo, carpeta de logs
├── ports.conf            ← en qué puertos escucha (Listen)
├── mods-available/       ← módulos instalados          ┐ a2enmod / a2dismod
├── mods-enabled/         ← enlaces a los activos       ┘
├── conf-available/       ← fragmentos de configuración ┐ a2enconf / a2disconf
├── conf-enabled/         ← enlaces a los activos       ┘
├── sites-available/      ← sitios (VirtualHost)        ┐ a2ensite / a2dissite
└── sites-enabled/        ← enlaces a los activos       ┘
```

Las tres parejas `*-available` / `*-enabled` funcionan igual. En `*-available/` están **todos** los ficheros disponibles; en `*-enabled/` solo hay **enlaces simbólicos** a los que están activados. Un enlace simbólico es un fichero especial que apunta a otro: el sistema lo trata como si fuera el original, pero borrar el enlace no borra el original. Se ve con `ls -l`, donde la `l` inicial indica que es un enlace y la flecha `->` muestra su destino:

```bash
ls -l /etc/apache2/conf-enabled/
```

```text
total 0
lrwxrwxrwx 1 root root 30 Oct 18 18:17 charset.conf -> ../conf-available/charset.conf
lrwxrwxrwx 1 root root 44 Oct 18 18:17 localized-error-pages.conf -> ../conf-available/localized-error-pages.conf
lrwxrwxrwx 1 root root 46 Oct 18 18:17 other-vhosts-access-log.conf -> ../conf-available/other-vhosts-access-log.conf
lrwxrwxrwx 1 root root 31 Oct 18 18:17 security.conf -> ../conf-available/security.conf
lrwxrwxrwx 1 root root 36 Oct 18 18:17 serve-cgi-bin.conf -> ../conf-available/serve-cgi-bin.conf
```

Los comandos `a2enconf`, `a2disconf`, `a2ensite`, `a2dissite`, `a2enmod` y `a2dismod` (el nombre significa *apache2 enable / disable*) crean y borran esos enlaces. Así activar o desactivar un fragmento, un sitio o un módulo es una operación reversible, que no obliga a borrar ni a editar nada. Hay una consecuencia práctica que conviene tener presente: **el fichero real está siempre en `*-available/`**; los mensajes de error de Apache citan la ruta del enlace en `*-enabled/` porque es la que ha leído, pero el fichero que hay que editar es el de `*-available/`.

El fichero `envvars` define variables de entorno que Apache usa al arrancar y que `apache2.conf` consulta escribiéndolas entre `${…}`. En Ubuntu, `APACHE_RUN_USER` y `APACHE_RUN_GROUP` valen `www-data` y `APACHE_LOG_DIR` vale `/var/log/apache2`.

### 1.3 En qué orden lee Apache los ficheros

Apache empieza siempre por `apache2.conf` y desde ahí va incluyendo el resto con las directivas `Include` e `IncludeOptional` (esta última no da error si no encuentra ficheros que coincidan). Quitando comentarios y líneas vacías, el contenido completo de `apache2.conf` en Ubuntu 24.04 es este:

```bash
grep -v '^\s*#' /etc/apache2/apache2.conf | grep -v '^\s*$'
```

La orden `grep -v` muestra las líneas que **no** coinciden con el patrón: el primero quita las líneas que empiezan por `#` (comentarios) y el segundo las vacías.

```text
DefaultRuntimeDir ${APACHE_RUN_DIR}
PidFile ${APACHE_PID_FILE}
Timeout 300
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}
HostnameLookups Off
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf
Include ports.conf
<Directory />
	Options FollowSymLinks
	AllowOverride None
	Require all denied
</Directory>
<Directory /usr/share>
	AllowOverride None
	Require all granted
</Directory>
<Directory /var/www/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
AccessFileName .htaccess
<FilesMatch "^\.ht">
	Require all denied
</FilesMatch>
LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent
IncludeOptional conf-enabled/*.conf
IncludeOptional sites-enabled/*.conf
```

De ahí salen el **orden de lectura** y tres consecuencias prácticas:

1. **Primero los módulos** (`mods-enabled`), porque las directivas que aporta un módulo no existen hasta que el módulo está cargado. Por eso `ports.conf` puede contener un `Listen 443` dentro de un bloque `<IfModule ssl_module>`: cuando Apache llega a ese fichero ya sabe si el módulo SSL está cargado.
2. **Después `ports.conf`, luego `conf-enabled/` y al final `sites-enabled/`.** Lo global va en `conf-*`; lo de cada sitio, en su fichero de `sites-*`.
3. **Dentro de cada carpeta, en orden alfabético.** Importa más de lo que parece, como se ve a continuación.

Para ver la lista exacta de ficheros que Apache ha leído, en el orden en que los ha leído, se usa:

```bash
sudo apache2ctl -t -D DUMP_INCLUDES | grep -v mods-enabled
```

`apache2ctl` es la herramienta de control de Apache; `-t` comprueba la sintaxis, `-D DUMP_INCLUDES` define una opción interna que hace que vuelque la lista de ficheros incluidos, y el `grep -v mods-enabled` oculta las decenas de ficheros de módulos para que se lea mejor:

```text
Included configuration files:
  (*) /etc/apache2/apache2.conf
    (150) /etc/apache2/ports.conf
    (222) /etc/apache2/conf-enabled/charset.conf
    (222) /etc/apache2/conf-enabled/localized-error-pages.conf
    (222) /etc/apache2/conf-enabled/other-vhosts-access-log.conf
    (222) /etc/apache2/conf-enabled/security.conf
    (222) /etc/apache2/conf-enabled/serve-cgi-bin.conf
    (225) /etc/apache2/sites-enabled/solvia.conf
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
```

El número entre paréntesis es la línea de `apache2.conf` donde está el `Include` que trajo ese fichero (la 150 es `Include ports.conf`; la 222 es `IncludeOptional conf-enabled/*.conf`). La última línea, que empieza por `AH00558`, es un aviso del que se habla en el apartado 1.9.

**Qué ocurre cuando dos ficheros dicen cosas distintas.** Para una directiva de un solo valor como `ServerTokens`, **gana la última que Apache lee**. Se puede comprobar creando un fichero cuyo nombre se ordene después de `security.conf`:

```bash
echo "ServerTokens Full" | sudo tee /etc/apache2/conf-available/zz-prueba.conf
sudo a2enconf zz-prueba
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -sI http://localhost/ | grep -i ^server
```

```text
Server: Apache/2.4.58 (Ubuntu) OpenSSL/3.0.13
```

Con `security.conf` en `ServerTokens Prod`, la cabecera saldría como `Server: Apache`. Aquí sale con más información porque `zz-prueba.conf` se lee después y su valor (`Full`) manda. Si el fichero se llamara `aa-prueba.conf`, se leería antes que `security.conf` y la cabecera seguiría siendo `Server: Apache`: `security.conf` lo pisaría. Esto mismo ocurre si se pone `ServerTokens Full` en la primera línea de `apache2.conf`: el `security.conf` que se incluye más abajo manda sobre ella. Para deshacer la prueba:

```bash
sudo a2disconf zz-prueba
sudo rm /etc/apache2/conf-available/zz-prueba.conf
sudo systemctl reload apache2
```

La regla práctica es que, **cuando un valor «no hace caso»**, hay que buscar si otro fichero lo define más tarde. `grep -n ServerTokens /etc/apache2/conf-enabled/*.conf` lo encuentra en un segundo.

### 1.4 Contextos: dónde puede ir cada directiva

El **contexto** de una directiva es el lugar de la configuración donde se admite. Hay cuatro que importan ahora:

| Contexto | Dónde se escribe | A qué afecta |
|---|---|---|
| **server config** | Fuera de cualquier bloque: `apache2.conf`, `ports.conf`, `conf-enabled/*.conf` | A todo el servidor |
| **virtual host** | Dentro de `<VirtualHost>` | Solo a ese sitio |
| **directory** | Dentro de `<Directory>`, `<Location>` o `<Files>` | Solo a esa ruta |
| **.htaccess** | En un fichero `.htaccess` dentro de la carpeta servida | A esa carpeta y sus subcarpetas, si `AllowOverride` lo permite |

La documentación oficial de Apache indica el contexto de cada directiva en una línea llamada *Context*. Es lo primero que hay que mirar cuando una directiva «no funciona». Por ejemplo, `ServerTokens` solo es válida en *server config*: describe al servidor entero, no a un sitio. Si se escribe dentro de un `<VirtualHost>`, la comprobación de sintaxis la rechaza y dice dónde está el problema:

```apache
<VirtualHost *:80>
        ServerName solvia.local
        ServerTokens Full
        DocumentRoot /var/www/solvia/public
</VirtualHost>
```

```bash
sudo apache2ctl configtest
```

```text
AH00526: Syntax error on line 3 of /etc/apache2/sites-enabled/solvia.conf:
ServerTokens cannot occur within <VirtualHost> section
```

### 1.5 Los parámetros que hay que conocer

Estos son los parámetros que se estudian en la unidad. Los valores por defecto corresponden a Ubuntu 24.04.

**Puertos: `Listen`.** Define en qué puerto, y opcionalmente en qué dirección IP, acepta conexiones Apache. Está en `ports.conf`:

```bash
grep -n -E "^\s*Listen" /etc/apache2/ports.conf
```

```text
5:Listen 80
8:	Listen 443
12:	Listen 443
```

El primero, `Listen 80`, es el puerto de HTTP. Los dos `Listen 443` (HTTPS) están dentro de bloques `<IfModule ssl_module>` y `<IfModule mod_gnutls.c>`: valen solo si el módulo correspondiente está cargado. Si no hay HTTPS configurado, el puerto 443 no escucha aunque la línea esté en el fichero. La opción `-E` de `grep` permite usar expresiones regulares ampliadas y `\s*` significa «cero o más espacios», para que encuentre las líneas con sangría.

**Identidad: `User` y `Group`.** Son el usuario y el grupo del sistema con los que se ejecutan los procesos de Apache que atienden peticiones. En `apache2.conf` aparecen como `User ${APACHE_RUN_USER}`: eso no es un nombre de usuario sino una variable, que se define en `/etc/apache2/envvars`:

```bash
grep -E "APACHE_RUN_USER|APACHE_LOG_DIR" /etc/apache2/envvars
sudo apache2ctl -S | grep -E "^(User|Group)"
```

```text
export APACHE_RUN_USER=www-data
export APACHE_LOG_DIR=/var/log/apache2$SUFFIX
User: name="www-data" id=33
Group: name="www-data" id=33
```

La primera orden muestra la definición en el fichero; la segunda, lo que Apache tiene realmente en marcha (`apache2ctl -S` vuelca el estado de la configuración y sus líneas `User:` y `Group:` indican la identidad en uso). `$SUFFIX` está vacío en una instalación normal; existe para poder ejecutar varias instancias de Apache en la misma máquina. Lo importante es que el usuario es **`www-data`**, una cuenta sin privilegios. Es el usuario que necesita permiso para leer los ficheros del sitio: por eso el `DocumentRoot` de Solvia está en `/var/www/` y no en la carpeta personal de un usuario, a la que `www-data` no puede entrar.

Puede parecer extraño que Apache arranque como `root` si sus peticiones las atiende `www-data`. Es así porque en Linux solo `root` puede abrir puertos por debajo del 1024, como el 80. El proceso principal arranca como `root`, abre los puertos y lanza procesos hijo que cambian al usuario sin privilegios. Se ve en la lista de procesos:

```bash
ps -eo user,comm | grep apache2 | sort | uniq -c
```

`ps -e` lista todos los procesos y `-o user,comm` pide dos columnas, el usuario y el nombre del programa; `grep apache2` se queda con los de Apache, `sort` los ordena y `uniq -c` cuenta cuántas líneas idénticas hay:

```text
      1 root     apache2
      5 www-data apache2
```

**Tiempos y conexiones: `Timeout`, `KeepAlive`, `MaxKeepAliveRequests`, `KeepAliveTimeout`.** `Timeout` (300 por defecto, en segundos) es el tiempo máximo que Apache espera ciertos eventos de entrada y salida de una conexión: recibir la petición, el intervalo entre paquetes al recibir un cuerpo `POST` o `PUT` y el intervalo entre confirmaciones al enviar la respuesta. Tres cuestiones lo hacen relevante: un valor muy alto deja conexiones lentas ocupando recursos; un valor muy bajo corta a clientes con redes lentas; y el módulo `mod_reqtimeout` ya recorta por su cuenta el tiempo permitido para recibir las cabeceras (apartado 1.14).

Una **conexión persistente** (*keep-alive*) permite que el navegador reutilice la misma conexión TCP para pedir varios recursos seguidos (la página, el CSS, el JavaScript) en lugar de abrir una conexión nueva para cada uno. `KeepAlive On` la activa, `MaxKeepAliveRequests 100` limita cuántas peticiones se atienden por conexión y `KeepAliveTimeout 5` fija cuántos segundos espera Apache la siguiente petición antes de cerrarla. El valor de 5 segundos es corto a propósito: en el modo de trabajo de este servidor (el MPM *prefork*, que se explica más abajo) cada conexión ocupa un proceso entero mientras dura, de modo que una conexión parada esperando ocupa un proceso que otro cliente no puede usar.

**Nombre del servidor: `ServerName` global.** Es el nombre con el que se identifica el servidor en su conjunto, escrito fuera de cualquier `<VirtualHost>`. En una instalación nueva no existe, y Apache lo avisa al arrancar con el mensaje `AH00558` (apartado 1.9). El `ServerName` que va dentro de cada `<VirtualHost>` es otra cosa: identifica a un sitio, no al servidor.

**Resolución de nombres: `HostnameLookups`.** En `Off` por defecto: Apache registra la dirección IP de cada cliente tal cual. Activarlo haría una consulta DNS por cada petición para anotar el nombre, con el coste de rendimiento que eso supone; si hace falta el nombre, se resuelve más tarde al analizar los registros.

**Registros: `ErrorLog` y `LogLevel`.** `ErrorLog ${APACHE_LOG_DIR}/error.log` indica dónde se anotan los errores del servidor, y `LogLevel warn` cuánto detalle se registra. Los niveles, de menos a más detalle, son `emerg`, `alert`, `crit`, `error`, `warn`, `notice`, `info`, `debug` y `trace1` a `trace8`; `warn` anota avisos y todo lo más grave. Un `<VirtualHost>` puede tener su propio `ErrorLog`, como el de Solvia, que escribe en `solvia_error.log`; los errores de ese sitio van allí y no al general. El bloque 8 de estos apuntes desarrolla los registros.

**Lo que el servidor cuenta de sí mismo: `ServerTokens`, `ServerSignature` y `TraceEnable`.** Están en `/etc/apache2/conf-available/security.conf`, un fichero que Ubuntu incluye precisamente para ajustar la seguridad:

```bash
grep -n -E "^(ServerTokens|ServerSignature|TraceEnable)" /etc/apache2/conf-available/security.conf
```

```text
12:ServerTokens OS
23:ServerSignature On
32:TraceEnable Off
```

- **`ServerTokens`** controla cuánto dice la cabecera `Server` de cada respuesta. Valores, de menos a más información: `Prod` (`Apache`), `Major` (`Apache/2`), `Minor` (`Apache/2.4`), `Minimal` (`Apache/2.4.58`), `OS` (`Apache/2.4.58 (Ubuntu)`, el valor por defecto de Ubuntu) y `Full` (lo anterior más los módulos que se anuncian, como `OpenSSL/3.0.13`).
- **`ServerSignature`** controla si las páginas de error que genera Apache terminan con una línea (`<address>`) con el nombre y la versión del servidor y el puerto. Por defecto está en `On`; con `Off` desaparece esa línea.
- **`TraceEnable`** controla el método HTTP `TRACE`, que hace que el servidor devuelva al cliente la petición tal como la recibió. Es una herramienta de diagnóstico que apenas se usa y que en el pasado se aprovechó para atacar a los navegadores. Con `Off`, Apache responde `405 Method Not Allowed`. Ubuntu lo trae ya desactivado: es un valor correcto y no hay que cambiarlo.

**El MPM.** El *Multi-Processing Module* es el módulo que decide cómo reparte Apache el trabajo entre procesos e hilos. Con PHP instalado como módulo de Apache (`libapache2-mod-php`, que es como se sirve Solvia), se usa el MPM **prefork**: un proceso por conexión, sin hilos, porque ese módulo de PHP no es seguro con hilos. Se comprueba con:

```bash
apache2ctl -V | grep -E "Server version|Server MPM|SERVER_CONFIG_FILE"
```

```text
Server version: Apache/2.4.58 (Ubuntu)
Server MPM:     prefork
 -D SERVER_CONFIG_FILE="apache2.conf"
```

Sus parámetros están en `mods-available/mpm_prefork.conf`:

```bash
grep -v '^\s*#' /etc/apache2/mods-available/mpm_prefork.conf | grep -v '^\s*$'
```

```text
StartServers            5
MinSpareServers         5
MaxSpareServers         10
MaxRequestWorkers       150
MaxConnectionsPerChild  0
```

`StartServers 5` son los procesos que se lanzan al arrancar (los cinco `www-data` de la lista anterior) y `MinSpareServers` / `MaxSpareServers` el margen de procesos libres que Apache procura mantener. `MaxRequestWorkers 150` es el máximo de conexiones atendidas a la vez: la 151 espera a que se libere un proceso. `MaxConnectionsPerChild 0` significa que ningún proceso se recicla por número de peticiones.

**Qué se sirve al pedir una carpeta: `DirectoryIndex`.** Está en `mods-available/dir.conf` y vale `index.html index.cgi index.pl index.php index.xhtml index.htm`: cuando se pide una carpeta, Apache busca esos ficheros en ese orden y sirve el primero que exista. Si no hay ninguno y las opciones de la carpeta incluyen `Indexes`, muestra un listado de su contenido (esto se trata en el bloque 4).

### 1.6 Las herramientas del administrador

| Comando | Para qué sirve |
|---|---|
| `sudo apache2ctl configtest` (o `-t`) | Comprueba la sintaxis de la configuración **sin tocar el servicio en marcha**. Se usa siempre antes de recargar |
| `sudo apache2ctl -S` | Muestra los sitios configurados, en qué puerto, cuál es el sitio por defecto, y el usuario y grupo en uso |
| `sudo apache2ctl -M` | Lista los módulos cargados y si son estáticos o dinámicos |
| `apache2ctl -V` | Versión, MPM y rutas compiladas |
| `sudo apache2ctl -t -D DUMP_INCLUDES` | Lista los ficheros leídos y en qué orden |
| `sudo systemctl reload apache2` | Relee la configuración **sin cortar** las conexiones en curso |
| `sudo systemctl restart apache2` | Para y vuelve a arrancar el servicio |

La diferencia entre `reload` y `restart` es importante. Con `reload`, Apache deja que los procesos terminen la petición que están atendiendo y arranca otros con la configuración nueva: los clientes no notan nada. Con `restart` se detiene y se vuelve a arrancar el servicio, y se cortan las conexiones que hubiera; hace falta en pocos casos, por ejemplo al cambiar de MPM. Con una configuración errónea, la recarga **no se aplica** y Apache sigue sirviendo con la configuración anterior (se ha comprobado en Ubuntu 24.04); un reinicio con configuración errónea puede dejar el servicio parado, porque al volver a arrancar Apache se encuentra el error. Por eso la regla es siempre la misma: **primero `configtest`**.

### 1.7 Cómo se cambia un parámetro: el ciclo de trabajo

Cada cambio de configuración sigue estos cuatro pasos, siempre en este orden:

1. **Editar** el fichero.
2. **`configtest`**: comprobar la sintaxis.
3. **`reload`**: aplicar el cambio.
4. **Comprobar desde fuera**: pedir al servidor lo que se ha cambiado y ver que responde como se espera.

Antes del paso 1 hay un paso 0, que consiste en **guardar una copia del fichero original**, para saber cuál era el valor de partida y poder volver atrás:

```bash
cp /etc/apache2/conf-available/security.conf ~/security.conf.orig
```

`cp origen destino` copia un fichero; `~` es la carpeta personal del usuario. Para el paso 1 hay dos formas. La habitual es el editor `nano`, que se abre con `sudo` porque los ficheros de `/etc/apache2/` solo los puede modificar el administrador (sin `sudo`, nano avisa de que no puede escribir y no guarda nada):

```bash
grep -n -E "^(ServerTokens|ServerSignature)" /etc/apache2/conf-available/security.conf
sudo nano +12 /etc/apache2/conf-available/security.conf
```

El `grep -n` da el número de línea donde está la directiva; `nano +12` abre el fichero directamente en la línea 12. Dentro de nano: **Ctrl+W** busca texto, **Ctrl+O** y Enter guardan, **Ctrl+X** sale. La otra forma es automatizar el cambio con `sed`, que edita ficheros desde la línea de comandos:

```bash
sudo sed -i -E 's/^ServerTokens .*/ServerTokens Prod/; s/^ServerSignature .*/ServerSignature Off/' /etc/apache2/conf-available/security.conf
```

`-i` modifica el fichero en su sitio; `-E` activa las expresiones regulares ampliadas; cada `s/patrón/reemplazo/` sustituye el texto que coincide con el patrón: `^` es el principio de línea y `.*` cualquier resto de la línea, de modo que cada orden cambia la línea completa. Los dos cambios se separan con `;`. Un riesgo de `sed` es que, si el patrón no coincide con nada, no avisa y no cambia nada; por eso se comprueba siempre el resultado con `diff`:

```bash
diff ~/security.conf.orig /etc/apache2/conf-available/security.conf
```

`diff` compara dos ficheros y enseña solo las diferencias. En su salida, `12c12` significa «la línea 12 ha cambiado (*changed*)», las líneas que empiezan por `<` son del primer fichero (el original) y las que empiezan por `>` del segundo (el modificado):

```text
12c12
< ServerTokens OS
---
> ServerTokens Prod
23c23
< ServerSignature On
---
> ServerSignature Off
```

Estas dos columnas, lo que había y lo que queda, son exactamente lo que hay que anotar al documentar un cambio.

### 1.8 Ejemplo resuelto: lo que tu servidor cuenta de sí mismo

**El problema.** Con la configuración de serie, cualquiera que pida una página a Solvia puede saber qué versión exacta de Apache corre y sobre qué sistema operativo. Basta con mirar las cabeceras de una respuesta o provocar un error. Estos son los dos comandos:

```bash
curl -sI http://localhost/ | grep -i ^server
curl -s http://localhost/noexiste | grep -i address
```

`curl` es un cliente HTTP de línea de comandos. `-s` (*silent*) oculta la barra de progreso; `-I` hace una petición `HEAD`, que pide solo las cabeceras y no la página. `grep -i ^server` se queda con las líneas que empiezan por «server» sin distinguir mayúsculas de minúsculas (`-i`). En el segundo comando se pide una página que no existe y `grep -i address` busca el pie de la página de error:

```text
Server: Apache/2.4.58 (Ubuntu)
<address>Apache/2.4.58 (Ubuntu) Server at localhost Port 80</address>
```

La primera línea la produce `ServerTokens OS`; la segunda, `ServerSignature On`. Con esa información, alguien que prepare un ataque sabe qué fallos conocidos buscar para esa versión y para ese sistema. A esto se le llama **reconocimiento** y es la primera fase de casi cualquier ataque.

**La solución.** Se aplica el ciclo del apartado 1.7. Con el fichero editado (`ServerTokens Prod` y `ServerSignature Off`, como en el `diff` del apartado anterior):

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -sI http://localhost/ | grep -i ^server
curl -s http://localhost/noexiste | grep -i -c address
curl -s -o /dev/null -w "%{http_code}\n" http://localhost/noexiste
```

```text
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
Syntax OK
Server: Apache
0
404
```

El `grep -c` cuenta las líneas que coinciden: el `0` significa que la página de error ya no lleva la línea `<address>`. En el último comando, `-o /dev/null` descarta el cuerpo de la respuesta y `-w "%{http_code}\n"` imprime solo el código de estado. El `404` demuestra que **la página de error sigue existiendo**: se ha quitado su pie, no el error. Con `Prod` la cabecera `Server` sigue presente, pero solo dice `Apache`.

### 1.9 Ejemplo resuelto: el aviso AH00558

Cada vez que se comprueba la configuración o se recarga Apache en una instalación nueva aparece este mensaje:

```text
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
```

**Es un aviso, no un error**: Apache arranca y funciona igual (`Syntax OK` aparece justo debajo). Significa que no hay ninguna directiva `ServerName` en el contexto global, así que Apache ha tenido que deducir un nombre y ha usado una dirección IP (127.0.0.1 o 127.0.1.1, según el equipo). El `ServerName` del `<VirtualHost>` de Solvia **no cuenta**, porque está dentro de un sitio y no en el contexto global. Se puede comprobar dónde hay `ServerName`:

```bash
grep -nE "ServerName|</?VirtualHost" /etc/apache2/apache2.conf /etc/apache2/conf-enabled/*.conf /etc/apache2/sites-enabled/*.conf
```

`-nE` es lo mismo que `-n -E` escrito junto. En `</?VirtualHost` el interrogante hace opcional la barra, así que el patrón encuentra tanto la etiqueta de apertura (`<VirtualHost`) como la de cierre (`</VirtualHost`). Con ello se ve no solo dónde hay un `ServerName`, sino entre qué etiquetas está.

```text
/etc/apache2/apache2.conf:129:# If you do not specify an ErrorLog directive within a <VirtualHost>
/etc/apache2/apache2.conf:131:# logged here.  If you *do* define an error logfile for a <VirtualHost>
/etc/apache2/sites-enabled/solvia.conf:1:<VirtualHost *:80>
/etc/apache2/sites-enabled/solvia.conf:2:        ServerName solvia.local
/etc/apache2/sites-enabled/solvia.conf:9:</VirtualHost>
```

(Los números de línea y el contenido exacto de las etiquetas pueden variar de una máquina a otra; los de `apache2.conf` son los de Ubuntu 24.04 en la máquina de pruebas, y el resto es un ejemplo con la forma que tendrá la salida.) Se lee así:

- El único `ServerName` que aparece, el de `solvia.conf`, tiene un número de línea **entre** los de `<VirtualHost>` y `</VirtualHost>`: está dentro de un bloque y solo vale para ese sitio.
- Las dos líneas de `apache2.conf` son comentarios que mencionan `<VirtualHost>` de pasada; no son directivas.
- En `apache2.conf` y en `conf-enabled` no sale ninguna línea con `ServerName`, es decir, no hay ninguno fuera de un bloque. Por eso Apache avisa con AH00558: ha leído toda su configuración y no ha encontrado un nombre global.

Dos precisiones sobre este comando: se escribe `*.conf` y no `-r` sobre la carpeta porque los ficheros de `sites-enabled` son enlaces simbólicos y `grep -r` **no los sigue** (no encontraría nada); y si el sitio por defecto `000-default.conf` sigue activo, saldrá también su línea `#ServerName www.example.com`, que está comentada (empieza por `#`) y por tanto no vale.

**La solución limpia** consiste en crear un fragmento propio en `conf-available/` y activarlo con `a2enconf`, sin tocar `apache2.conf`:

```bash
echo "ServerName solvia.local" | sudo tee /etc/apache2/conf-available/servername.conf
sudo a2enconf servername
sudo apache2ctl configtest
sudo systemctl reload apache2
```

El `echo` escribe el texto y `tee` lo guarda en el fichero. Se usa `tee` y no una redirección (`>`) porque con `sudo echo "..." > fichero` la redirección la hace tu usuario, no `root`, y falla por permisos; con `| sudo tee` es `tee` quien escribe, con permisos de administrador. Pon como nombre el de tu propio sitio. La salida es:

```text
ServerName solvia.local
Enabling conf servername.
To activate the new configuration, you need to run:
  service apache2 reload
Syntax OK
```

(La última línea del mensaje de `a2enconf` indica el comando de recarga de tu sistema: puede decir `systemctl reload apache2`.) Ahora `configtest` muestra solo `Syntax OK`, sin el aviso. El `ServerName` global solo da nombre a la máquina y no cambia qué sitio responde a cada petición.

### 1.10 Ejemplo resuelto: leer un error

Cuando `configtest` falla, el mensaje dice tres cosas: **qué fichero, qué línea y qué ha encontrado**. Hay que leerlo entero antes de hacer nada. Se provoca un error a propósito, escribiendo mal el nombre de una directiva:

```bash
sudo sed -i 's/^ServerSignature Off/ServerSignatur Off/' /etc/apache2/conf-available/security.conf
sudo apache2ctl configtest
```

```text
AH00526: Syntax error on line 23 of /etc/apache2/conf-enabled/security.conf:
Invalid command 'ServerSignatur', perhaps misspelled or defined by a module not included in the server configuration
```

El mensaje cita `conf-enabled/security.conf` aunque el fichero que se editó está en `conf-available/`: es el mismo, porque el primero es un enlace al segundo (apartado 1.2). Dice la línea (23) y la directiva exacta que Apache no reconoce (`ServerSignatur`), y propone dos causas: un error al teclear, o una directiva que pertenece a un módulo que no está cargado. Aquí es lo primero. Se deshace y se comprueba:

```bash
sudo sed -i 's/^ServerSignatur Off/ServerSignature Off/' /etc/apache2/conf-available/security.conf
sudo apache2ctl configtest
```

```text
Syntax OK
```

Hay que tener claro lo que habría pasado si se hubiera hecho `reload` con el error puesto: nada. Apache no habría aplicado la configuración nueva y habría seguido con la anterior. Por eso `configtest` se ejecuta antes y se lee su salida. El otro error del apartado 1.4 (una directiva en un contexto que no la admite) se interpreta igual: fichero, línea y causa.

### 1.11 Caso práctico sobre Solvia: el inventario de tu servidor

Antes de cambiar nada, un administrador sabe qué tiene. El caso práctico consiste en rellenar esta tabla **mirando los ficheros, no de memoria**, para las ocho directivas de este apartado:

| Directiva | Fichero donde la encontraste | Qué hace (con tus palabras) | Valor que tenía | Valor que dejas | Por qué |
|---|---|---|---|---|---|
| `Listen` | | | | | |
| `User` | | | | | |
| `Timeout` | | | | | |
| `KeepAliveTimeout` | | | | | |
| `LogLevel` | | | | | |
| `ServerTokens` | | | | | |
| `ServerSignature` | | | | | |
| `ServerName` (global) | | | | | |

Los comandos que dan la información de cada fila son estos. Los resultados esperados se muestran debajo de cada uno:

```bash
grep -n -E "^\s*Listen" /etc/apache2/ports.conf
```

```text
5:Listen 80
8:	Listen 443
12:	Listen 443
```

```bash
grep -n -E "^(Timeout|KeepAlive|LogLevel|User)" /etc/apache2/apache2.conf
```

Este patrón coincide con todas las líneas que empiezan por `Timeout`, `KeepAlive` (incluida `KeepAliveTimeout`), `LogLevel` o `User`:

```text
92:Timeout 300
98:KeepAlive On
111:KeepAliveTimeout 5
115:User ${APACHE_RUN_USER}
143:LogLevel warn
```

La directiva `User` remite a una variable. Para saber su valor hay que mirar `envvars` (apartado 1.5). Si no entiendes qué hace una directiva, el propio fichero lo explica en los comentarios que tiene encima; se pueden ver con este comando, donde `-B4` muestra además las cuatro líneas anteriores a la que coincide:

```bash
grep -n -B4 "^Timeout" /etc/apache2/apache2.conf
```

```text
88-
89-#
90-# Timeout: The number of seconds before receives and sends time out.
91-#
92:Timeout 300
```

Las filas de `ServerTokens` y `ServerSignature` se completan con el comando del apartado 1.5 sobre `security.conf`, y la de `ServerName` con el `grep` del apartado 1.9, que demuestra que no hay ninguno en el contexto global.

**Qué se deja y qué se cambia.** La decisión razonada para las ocho directivas es la siguiente. Se **cambian tres**: `ServerTokens` pasa de `OS` a `Prod`, `ServerSignature` de `On` a `Off`, y se **añade** un `ServerName` global (que antes no existía). Las otras cinco se **dejan como están, pero con una razón**:

- **`Listen 80`** se deja porque el servidor solo sirve HTTP; el puerto 443 se activa cuando se configure HTTPS en esta misma unidad, junto con el módulo SSL.
- **`User www-data`** se deja porque es un usuario sin privilegios; cambiarlo a `root` permitiría que un fallo en la aplicación tuviera control total de la máquina.
- **`Timeout 300`** se deja porque no hay ningún problema medido que justifique cambiarlo y `mod_reqtimeout` ya limita las cabeceras lentas. **Bajarlo (por ejemplo a 60) también es defendible**, pero entonces hay que explicar qué se gana (menos tiempo de ocupación de procesos por clientes lentos) y qué se arriesga (cortar a clientes legítimos con redes lentas).
- **`KeepAliveTimeout 5`** se deja porque ya es un valor corto, adecuado para prefork: permite reutilizar la conexión para el CSS y el JavaScript sin tener procesos parados esperando.
- **`LogLevel warn`** se deja porque es suficiente para operar; subirlo a `info` o `debug` llena el disco y se hace solo de forma temporal para investigar un problema concreto.

Observa que «lo dejo porque es el valor de serie» **no** es una razón: una razón habla del servidor, de la aplicación o del riesgo. La tabla completa con valores y razones es lo que se llama el **inventario** del servidor, y es el punto de partida para la documentación y las recomendaciones que se redactan al cerrar la unidad (bloque 9).

### 1.12 ⚖️ Alternativa y criterio

**Crear un fichero propio en `conf-available/` y activarlo con `a2enconf`, o escribir la directiva directamente en `apache2.conf`.** Las dos formas funcionan y producen el mismo resultado. Se prefiere la primera por tres razones:

- **Actualizaciones.** `apache2.conf` es un fichero de configuración que pertenece al paquete `apache2`. Si lo has modificado y una actualización del paquete trae una versión nueva, el sistema te pregunta qué hacer con tu versión y la nueva, y es fácil equivocarse. Un fichero propio en `conf-available/` el paquete no lo toca.
- **Reversibilidad.** `sudo a2disconf servername` desactiva el cambio con un comando, sin tener que buscar qué línea tocaste dentro de un fichero de doscientas.
- **Legibilidad.** Quien herede el servidor ve en `conf-enabled/` qué se ha añadido a mano y puede leerlo aislado, sin comparar `apache2.conf` línea a línea con el original.

Con `security.conf` ocurre algo parecido: también es un fichero del paquete, pero Ubuntu lo incluye precisamente para que se ajusten ahí los parámetros de seguridad. Por eso se acepta editarlo directamente, guardando antes una copia (apartado 1.7).

### 1.13 ⚠️ Errores comunes

**Creer que ocultar la versión es protegerse.** `ServerTokens Prod` no corrige ninguna vulnerabilidad: si tu Apache tiene un fallo conocido, lo sigue teniendo. Lo que se consigue es no regalar la información a quien busca servidores con una versión concreta. Es una medida de **reducción de información**, útil y barata, pero no sustituye a mantener el sistema actualizado (`sudo apt update && sudo apt upgrade`). Cómo reconocerlo: quien dice «ya es seguro porque ya no sale la versión» ha confundido las dos cosas.

**Cambiar una directiva y no recargar.** Se edita el fichero, se hace `curl`, sigue saliendo lo de antes y se concluye que «no funciona». Apache solo lee la configuración al arrancar o al recargar. La secuencia es siempre editar, `configtest`, `reload`, comprobar.

**Editar sin `sudo`.** `nano` abre el fichero pero avisa de que no se puede escribir, y no guarda. Se cierra y se vuelve a abrir con `sudo nano`.

**Buscar con `grep -r` en las carpetas `*-enabled`.** No encuentra nada porque lo que hay dentro son enlaces simbólicos y `-r` no los sigue. Se usa `*.conf`.

**Cambiar un valor y que no haga efecto.** Puede haber otro fichero que lo define más tarde y que, por el orden de lectura, gana (apartado 1.3). Se busca con `grep -n NombreDeLaDirectiva /etc/apache2/conf-enabled/*.conf /etc/apache2/sites-enabled/*.conf`.

**Confundir un aviso con un error.** `AH00558` va acompañado de `Syntax OK`: Apache arranca. Un error de sintaxis (`AH00526`) impide que la configuración se aplique.

**Justificar con «es el valor de serie».** Una decisión de configuración se justifica por lo que ocurre en el servidor, no por el origen del valor.

### 1.14 💡 Nota técnica

El valor por defecto `Timeout 300` es generoso: cinco minutos. Un tipo de ataque de denegación de servicio, conocido como **Slowloris**, aprovecha eso: abre muchas conexiones y envía las cabeceras de la petición muy despacio, byte a byte, para mantener ocupados todos los procesos del servidor (en prefork, los 150 de `MaxRequestWorkers`) y que no quede ninguno libre para clientes reales. Ubuntu ya trae activado el módulo **`mod_reqtimeout`**, que limita cuánto tiempo se concede para recibir las cabeceras y el cuerpo de una petición:

```bash
grep -v '^\s*#' /etc/apache2/mods-available/reqtimeout.conf | grep -v '^\s*$'
```

```text
RequestReadTimeout header=20-40,minrate=500
RequestReadTimeout body=10,minrate=500
```

Se lee así: para las cabeceras, Apache concede 20 segundos y, mientras lleguen al menos 500 bytes por segundo, va ampliando el plazo hasta un máximo de 40; para el cuerpo, 10 segundos con la misma velocidad mínima. Un cliente que envía los datos más despacio que eso es desconectado. Es un buen ejemplo de lo que se verá en el bloque 2: buena parte de la seguridad de Apache no está en el núcleo, sino en módulos.

### 1.15 Comprueba lo que has entendido

**1. Has puesto `ServerTokens Prod` en la primera línea de `apache2.conf`, pero `curl -sI` sigue mostrando `Apache/2.4.58 (Ubuntu)`. ¿Qué puede estar pasando y cómo lo compruebas?**

Apache lee `apache2.conf` primero y después incluye `conf-enabled/*.conf`. Si `security.conf` (o cualquier otro fichero de `conf-enabled/` que se lea después) define `ServerTokens OS`, ese valor gana, porque con una directiva de un solo valor manda la última que se lee. Se comprueba con `grep -n ServerTokens /etc/apache2/apache2.conf /etc/apache2/conf-enabled/*.conf` y con `sudo apache2ctl -t -D DUMP_INCLUDES` para ver el orden. También podría faltar el `reload`.

**2. ¿Por qué se ejecuta `configtest` antes de `reload`?**

Porque `configtest` solo comprueba la sintaxis y no toca el servicio, así que un error se descubre sin afectar a nadie. Con una configuración errónea, la recarga no se aplica y Apache sigue con la anterior; un reinicio podría dejar el servicio parado.

**3. `grep -rn "ServerName" /etc/apache2/sites-enabled/` no imprime nada, pero sabes que tu sitio tiene un `ServerName`. ¿Por qué?**

Porque los ficheros de `sites-enabled` son enlaces simbólicos a `sites-available` y la opción `-r` de `grep` no sigue los enlaces que encuentra al recorrer una carpeta. Con `grep -n "ServerName" /etc/apache2/sites-enabled/*.conf` sí funciona, porque el asterisco hace que cada enlace llegue al comando como una ruta explícita.

**4. `ServerTokens Prod` hace que la cabecera diga solo `Apache`. ¿Se ha corregido alguna vulnerabilidad del servidor? Razona la respuesta.**

No. Las vulnerabilidades dependen del software que está instalado y siguen igual. Lo único que se ha reducido es la información que se ofrece a quien hace un reconocimiento. Para corregir vulnerabilidades hay que actualizar el sistema.

**5. Tu `solvia.conf` ya contiene `ServerName solvia.local` y aun así aparece el aviso `AH00558`. ¿Por qué, y cómo se quita sin tocar `apache2.conf`?**

Porque ese `ServerName` está dentro de un `<VirtualHost>` y el aviso reclama uno en el contexto global. Se quita creando `/etc/apache2/conf-available/servername.conf` con `ServerName solvia.local`, activándolo con `sudo a2enconf servername`, y ejecutando `configtest` y `reload`.

---

*El resto de los bloques, el glosario y las referencias se irán añadiendo uno a uno, en el orden de la unidad.*
