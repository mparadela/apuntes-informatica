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

```text
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

- Los comandos van en bloques `bash`; la salida esperada, en bloques `text`, justo debajo.
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

```text
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

La directiva `User` remite a una variable. Para saber su valor hay que mirar `envvars` (apartado 1.5). Si no entiendes qué hace una directiva, el propio fichero lo explica en los comentarios que tiene encima; se pueden ver con `grep -n -B4 "^Timeout" /etc/apache2/apache2.conf`, donde `-B4` muestra además las cuatro líneas anteriores a la que coincide:

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

## 2. Módulos: ampliar la funcionalidad del servidor

**CE 2.b** — Se han ampliado y reducido las funcionalidades del servidor mediante el uso de módulos.

Este apartado explica de qué piezas está hecho Apache, cómo saber cuáles tiene tu servidor, cómo se añaden y se quitan, qué ocurre cuando falta una y cómo decidir cuáles dejar. Se apoya en el ciclo de trabajo del apartado 1.7 (editar, `configtest`, recargar, comprobar) y en el mapa de ficheros del apartado 1.2. Al final del apartado se examina `mod_status`, el módulo que enseña por dentro el estado del servidor, porque es el ejemplo perfecto de una pieza útil que se vuelve peligrosa si nadie decide quién puede mirarla.

### 2.1 Qué es un módulo y por qué Apache se reparte en trozos

Un **módulo** es una pieza de código que añade una capacidad concreta a Apache: entender las reglas `Require`, comprimir las respuestas, ejecutar PHP, cifrar con TLS, escribir cabeceras, mostrar el estado del servidor. El núcleo de Apache (el *core*) es muy pequeño: sabe aceptar una conexión, leer una petición HTTP, decidir qué sitio debe atenderla y devolver una respuesta. Casi todo lo demás lo hacen módulos.

Esta forma de construir el servidor existe por tres razones. La primera es el **tamaño y el rendimiento**: un servidor que solo sirve ficheros estáticos no necesita cargar el código de PHP ni el de TLS. La segunda es la **seguridad**: el código que no está cargado no puede tener fallos que alguien explote, así que cuantas menos piezas, menos superficie de ataque. La tercera es la **extensibilidad**: cualquiera puede escribir un módulo nuevo sin tocar el núcleo, y por eso hay módulos de Apache para PHP, Python, autenticación contra LDAP o cortafuegos de aplicaciones web.

Hay dos formas de que un módulo forme parte del servidor:

- **Estático** (*static*, o *compiled in*): el código está dentro del ejecutable `apache2`. No se puede quitar sin volver a compilar Apache. En Ubuntu solo son estáticos los módulos más básicos, los que el servidor no puede funcionar sin ellos.
- **Compartido** o dinámico (*shared*, también llamado **DSO**, *Dynamic Shared Object*): el código está en un fichero aparte con extensión `.so`, dentro de `/usr/lib/apache2/modules/`, y Apache lo carga al arrancar si una línea de la configuración se lo pide. Esa línea es la directiva `LoadModule`. Es la forma habitual y la que permite activar y desactivar módulos sin compilar nada. Para poder cargar módulos compartidos, Apache necesita a su vez un módulo estático llamado `mod_so`.

### 2.2 Qué módulos tiene cargados tu servidor

La orden que lista los módulos que Apache tiene cargados es `apache2ctl -M` (la `M` mayúscula es de *modules*). Usa `sudo` para evitar problemas de permisos:

```bash
sudo apache2ctl -M
```

```text
Loaded Modules:
 core_module (static)
 so_module (static)
 watchdog_module (static)
 http_module (static)
 log_config_module (static)
 logio_module (static)
 version_module (static)
 unixd_module (static)
 access_compat_module (shared)
 alias_module (shared)
 auth_basic_module (shared)
 authn_core_module (shared)
 authn_file_module (shared)
 authz_core_module (shared)
 authz_host_module (shared)
 authz_user_module (shared)
 autoindex_module (shared)
 deflate_module (shared)
 dir_module (shared)
 env_module (shared)
 filter_module (shared)
 mime_module (shared)
 mpm_prefork_module (shared)
 negotiation_module (shared)
 php_module (shared)
 reqtimeout_module (shared)
 setenvif_module (shared)
 status_module (shared)
```

La salida se lee así: cada línea es un módulo, con su nombre interno terminado en `_module` y, entre paréntesis, si es `static` o `shared`. Los ocho primeros son estáticos; los veinte siguientes son compartidos. Si todavía no has resuelto el aviso `AH00558` (apartado 1.9), verás esa advertencia antes de `Loaded Modules:`; no afecta a la lista. El número exacto de módulos puede diferir en tu máquina según lo que hayas instalado.

Existe una segunda orden, `apache2ctl -l` (la `l` es de *list*), que enseña **solo los estáticos**, los compilados dentro del ejecutable:

```bash
apache2ctl -l
```

```text
Compiled in modules:
  core.c
  mod_so.c
  mod_watchdog.c
  http_core.c
  mod_log_config.c
  mod_logio.c
  mod_version.c
  mod_unixd.c
```

Los ocho nombres coinciden con los ocho `(static)` de la lista anterior. Fíjate en que aquí los módulos se escriben con otra forma de nombre (`mod_so.c`, el nombre del fichero de código fuente) y en `-M` con la forma `so_module`. Son el mismo módulo escrito de dos maneras; en la práctica, la forma de `-M` es la que se usa para comprobar si un módulo está cargado.

Para qué sirve cada uno de los veinte compartidos que trae una instalación de Apache con PHP en Ubuntu, y si hace falta para servir Solvia, se resume en el apartado 2.10.

### 2.3 Dónde viven los módulos: `mods-available` y `mods-enabled`

Los módulos siguen el mismo patrón que los sitios y los fragmentos de configuración, que ya conoces (apartado 1.2): hay una carpeta con **todo lo disponible** y otra con **lo activado**, y la segunda contiene solo enlaces simbólicos a la primera.

```plaintext
/etc/apache2/
├── mods-available/     ← todos los módulos que se pueden activar (un .load por módulo, y a veces un .conf)
│   ├── headers.load
│   ├── ssl.load
│   ├── ssl.conf
│   ├── status.load
│   ├── status.conf
│   └── ...
└── mods-enabled/       ← los activados: enlaces simbólicos a mods-available
    ├── status.load  -> ../mods-available/status.load
    ├── status.conf  -> ../mods-available/status.conf
    └── ...
```

Cada módulo tiene como mínimo un fichero `.load`, que contiene la línea que lo carga, y algunos tienen además un fichero `.conf` con sus ajustes. Cuenta cuántos hay disponibles y cuántos activados:

```bash
ls /etc/apache2/mods-available/*.load | wc -l
ls /etc/apache2/mods-enabled/*.load | wc -l
```

```text
116
20
```

En una instalación de Apache con PHP hay 116 módulos disponibles y 20 activados: los veinte compartidos de la lista del apartado 2.2. El comando `wc -l` cuenta líneas y el comodín `*.load` hace que `ls` imprima un fichero por línea. Tu cifra puede variar ligeramente.

El contenido de un fichero `.load` es una sola línea:

```bash
cat /etc/apache2/mods-available/headers.load
```

```text
LoadModule headers_module /usr/lib/apache2/modules/mod_headers.so
```

La directiva `LoadModule` tiene dos argumentos: el **nombre interno** del módulo (`headers_module`, el mismo que aparece en `apache2ctl -M`) y la **ruta del fichero `.so`** que contiene su código. Activar un módulo consiste en que Apache lea ese `.load`, y eso se consigue con un enlace en `mods-enabled/`:

```bash
ls -l /etc/apache2/mods-enabled/status.load
```

```text
lrwxrwxrwx 1 root root 29 Nov  3 13:31 /etc/apache2/mods-enabled/status.load -> ../mods-available/status.load
```

La `l` del principio de `lrwxrwxrwx` indica que es un enlace simbólico, y la flecha `->` muestra a qué fichero apunta. Es lo mismo que ocurre con `sites-enabled/` y `conf-enabled/`: el fichero real vive en `-available/` y `-enabled/` solo dice qué está en uso.

### 2.4 Activar y desactivar módulos: `a2enmod`, `a2dismod` y `a2query`

No se crean los enlaces a mano. Igual que `a2ensite` y `a2enconf`, hay dos órdenes para módulos: `a2enmod` (*enable module*) y `a2dismod` (*disable module*). Reciben el nombre del módulo **sin** el prefijo `mod_` y **sin** el sufijo `_module`: para `mod_headers` o `headers_module`, el nombre es `headers`.

```bash
sudo a2enmod headers
```

```text
Enabling module headers.
To activate the new configuration, you need to run:
  service apache2 restart
```

La orden ha creado el enlace en `mods-enabled/`, pero **Apache todavía no lo ha cargado**: solo lee su configuración al arrancar o al recargar. El mensaje habla de `restart` (reiniciar). En la práctica, para **añadir** un módulo basta con recargar, que no corta las conexiones en curso; se ha comprobado con Apache 2.4.58 en Ubuntu 24.04. Hay una excepción: cambiar el módulo de multiprocesamiento (el MPM, apartado 2.14) exige reiniciar.

El ciclo completo para un módulo es, por tanto, el mismo de siempre más un paso:

```bash
sudo a2enmod headers
sudo apache2ctl configtest
sudo systemctl reload apache2
apache2ctl -M | grep headers
```

```text
Syntax OK
 headers_module (shared)
```

La última orden es la comprobación: si `headers_module` aparece en la lista, Apache lo tiene cargado. Si no aparece, falta la recarga (o el reinicio, si el módulo lo exige). Con `sudo apache2ctl -M` y un `grep` se comprueba si un módulo concreto está cargado en menos de un segundo.

Si repites la activación de un módulo ya activo, la orden lo avisa y no hace nada; si el módulo no existe, falla con un código de error distinto de cero:

```bash
sudo a2enmod headers
sudo a2enmod nada
```

```text
Module headers already enabled
ERROR: Module nada does not exist!
```

Para desactivar se usa `a2dismod`:

```bash
sudo a2dismod headers
```

```text
Module headers disabled.
To activate the new configuration, you need to run:
  service apache2 restart
```

Para consultar el estado de un módulo sin cambiarlo existe `a2query`. Con `-m` y sin nombre lista los módulos activados con el motivo; con un nombre dice si ese módulo está activo:

```bash
a2query -m headers
a2query -m status
```

```text
No module matches headers
status (enabled by maintainer script)
```

La frase entre paréntesis tiene interés: **«enabled by maintainer script»** significa que el módulo lo activó el propio paquete al instalarse (la persona que mantiene el paquete en Ubuntu decidió que viniera activado), y **«enabled by site administrator»** significa que lo activaste tú con `a2enmod`. Es una pista útil para saber qué has tocado en un servidor y qué es de serie.

### 2.5 Dependencias entre módulos

Algunos módulos no funcionan sin otros. Debian y Ubuntu lo anotan en la primera línea del fichero `.load`, en forma de comentario. Mira el de TLS, `ssl`:

```bash
cat /etc/apache2/mods-available/ssl.load
```

```text
# Depends: mime socache_shmcb
LoadModule ssl_module /usr/lib/apache2/modules/mod_ssl.so
```

La línea `# Depends: mime socache_shmcb` dice que `ssl` necesita `mime` y `socache_shmcb` (este último guarda en memoria compartida las sesiones TLS para no repetir el *handshake* completo en cada conexión). `a2enmod` lee esa línea y **activa las dependencias antes** que el módulo:

```bash
sudo a2enmod ssl
```

```text
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Enabling module socache_shmcb.
Enabling module ssl.
See /usr/share/doc/apache2/README.Debian.gz on how to configure SSL and create self-signed certificates.
To activate the new configuration, you need to run:
  service apache2 restart
```

Una sola orden ha activado dos módulos: `socache_shmcb`, que faltaba, y `ssl`. El paso inverso está protegido. Si intentas desactivar un módulo del que depende otro que sigue activo, `a2dismod` se niega:

```bash
sudo a2dismod socache_shmcb
```

```text
ERROR: The following modules depend on socache_shmcb and need to be disabled first: ssl
```

El orden correcto para deshacer es el inverso: primero `ssl` (el que depende) y después `socache_shmcb`. Desactivar `ssl` no desactiva automáticamente `socache_shmcb`: quedará activo como módulo suelto. Lo mismo ocurre con PHP en tu servidor: el módulo `php8.3` depende de `mpm_prefork`, y `sudo a2dismod mpm_prefork` responde que `php8.3` debe desactivarse antes.

Esta protección existe para que nadie rompa el servidor por descuido, y no hay que esquivarla: si una dependencia te estorba, es una señal de que hay que pensar en qué estás quitando.

### 2.6 Módulos esenciales: `a2dismod` pide confirmación

Hay una lista de módulos que los mantenedores del paquete consideran **esenciales**: `alias`, `auth_basic`, `authn_file`, `authz_host`, `authz_user`, `autoindex`, `deflate`, `dir`, `env`, `filter`, `logio`, `mime`, `negotiation`, `setenvif`, `unixd`, `version` y `watchdog`. Si intentas desactivar uno de ellos, `a2dismod` no lo hace a la primera:

```bash
sudo a2dismod autoindex
```

```text
WARNING: The following essential module will be disabled.
This might result in unexpected behavior and should NOT be done
unless you know exactly what you are doing!
 autoindex

To continue type in the phrase 'Yes, do as I say!' or retry by passing '-f': Aborting
```

La orden pide que escribas la frase literal `Yes, do as I say!` («sí, haz lo que digo»), o que repitas la orden con la opción `-f` (*force*, forzar). Que la lista de esenciales sea larga **no quiere decir que no se puedan desactivar**: significa que desactivarlos puede cambiar el comportamiento del servidor de formas poco evidentes, y que quien lo haga debe saber qué consecuencia tiene. Observa que `status`, `headers` y `ssl` no están en la lista y se desactivan sin avisar.

### 2.7 Qué ocurre cuando falta un módulo

Un módulo ausente no siempre se nota en el momento: depende de qué función cubría. Tres casos reales, probados con Apache 2.4.58, muestran las tres formas típicas de fallar.

**La directiva no se reconoce.** Cada módulo aporta sus propias directivas. `Header` (para escribir cabeceras HTTP) es de `mod_headers`. Si escribes `Header` y el módulo no está cargado, `configtest` falla:

```bash
echo 'Header always set X-Prueba "hola"' | sudo tee /etc/apache2/conf-available/prueba-cabecera.conf
sudo a2enconf prueba-cabecera
sudo apache2ctl configtest
```

```text
AH00526: Syntax error on line 1 of /etc/apache2/conf-enabled/prueba-cabecera.conf:
Invalid command 'Header', perhaps misspelled or defined by a module not included in the server configuration
```

Fíjate en la frase final del error: **«perhaps misspelled or defined by a module not included»** (quizá mal escrita, o definida por un módulo que no está incluido). Es el mismo mensaje que da una errata, como `ServerSignatur` (apartado 1.10), pero ahora la directiva está bien escrita y lo que falta es el módulo. Cuando veas `Invalid command` con una directiva que sabes que existe, la primera pregunta es qué módulo la define. Para dejar el servidor como estaba, deshaz la prueba:

```bash
sudo a2disconf prueba-cabecera
sudo rm /etc/apache2/conf-available/prueba-cabecera.conf
```

**La función simplemente no ocurre.** Con PHP, el módulo `php8.3` es el que convierte los ficheros `.php` en páginas. Si se desactiva, el servidor arranca sin errores, no se queja de nada, y entrega el código fuente como si fuera texto:

```bash
echo '<?php echo "hola ".(2+3);' | sudo tee /var/www/solvia/public/prueba.php
sudo a2dismod php8.3
sudo systemctl reload apache2
curl -s http://localhost/prueba.php
```

```text
<?php echo "hola ".(2+3);
```

El primer comando crea un fichero PHP de una línea en la carpeta que sirve Apache. Con el módulo activo, la respuesta habría sido `hola 5`. Para deshacer la prueba, `sudo a2enmod php8.3`, recarga y `sudo rm /var/www/solvia/public/prueba.php`. Es el síntoma de «PHP servido como texto plano» que ya apareció al desplegar Solvia, y es peligroso por una razón añadida: el servidor está entregando al visitante el código fuente de la aplicación, con las contraseñas que contenga.

**Desaparece una función de administración.** Si se desactiva `status` y se pide `/server-status` (apartado 2.11), el servidor responde `404`: la ruta ya no existe. Esto es lo esperado, y es una forma de «quitar» una función sin escribir ninguna regla.

Dos casos más, de los que conviene saber el efecto. Si desactivas `mime` (con `-f`), el servidor deja de enviar la cabecera `Content-Type` en los ficheros estáticos como una hoja de estilos, y los navegadores pueden dejar de aplicarla. Y si desactivas `authz_core`, que es el que entiende `Require`, `configtest` falla con `Invalid command 'Require'` en los ficheros del paquete que usan esa directiva, como `alias.conf`.

### 2.8 Los nombres de un módulo: tres formas de escribir lo mismo

Es una fuente habitual de confusión, porque un mismo módulo aparece escrito de tres maneras según dónde mires:

| Dónde se usa | Forma | Ejemplo con `headers` | Ejemplo con PHP |
|---|---|---|---|
| `a2enmod`, `a2dismod`, `a2query` | nombre corto | `headers` | `php8.3` |
| `apache2ctl -M` y la directiva `LoadModule` | nombre interno terminado en `_module` | `headers_module` | `php_module` |
| Documentación y nombre del fichero `.so` | `mod_` + nombre | `mod_headers` | `libphp8.3.so` |

La regla general es sencilla: el nombre interno es el nombre corto más `_module`. La excepción que más confunde es **PHP**: se activa como `php8.3` (el nombre incluye la versión) pero en la lista aparece como `php_module`. Por eso, para comprobar que PHP está cargado, se busca `php`, no `php8.3`:

```bash
apache2ctl -M | grep php
```

```text
 php_module (shared)
```

### 2.9 Ejemplo resuelto: activar `mod_headers` y comprobar que funciona

El módulo `headers` permite añadir, cambiar o quitar cabeceras en las respuestas. Se usará más adelante en la unidad para las cabeceras de seguridad. Aquí se activa y se comprueba con una cabecera de prueba. Parte de una máquina con el módulo desactivado.

**Paso 1. Comprueba que el módulo no está cargado.**

```bash
apache2ctl -M | grep headers
```

La orden no imprime nada: el módulo no está cargado.

**Paso 2. Actívalo y comprueba que la sintaxis es correcta.**

```bash
sudo a2enmod headers
sudo apache2ctl configtest
```

```text
Enabling module headers.
To activate the new configuration, you need to run:
  service apache2 restart
Syntax OK
```

**Paso 3. Crea un fragmento de configuración propio con la cabecera de prueba** (mismo patrón que `servername.conf` en el apartado 1.9) **y actívalo:**

```bash
echo 'Header always set X-Servidor-Administrado "UD2"' | sudo tee /etc/apache2/conf-available/cabecera-prueba.conf
sudo a2enconf cabecera-prueba
sudo apache2ctl configtest
sudo systemctl reload apache2
```

La directiva se lee así: `Header` es la directiva del módulo; `always` hace que se añada también en las respuestas de error; `set` fija el valor; `X-Servidor-Administrado` es el nombre de la cabecera (las que empiezan por `X-` son cabeceras inventadas, no estándar) y `"UD2"` el valor.

**Paso 4. Comprueba desde fuera que la cabecera llega.**

```bash
apache2ctl -M | grep headers
curl -sI http://localhost/ | grep -i x-servidor
```

```text
 headers_module (shared)
X-Servidor-Administrado: UD2
```

Se han hecho dos comprobaciones distintas, y las dos hacen falta: la primera dice que el módulo **está cargado**, la segunda que su **efecto es visible** para quien hace una petición.

**Paso 5. Deshaz la prueba** (la regla de oro de un servidor ajeno es dejarlo como estaba):

```bash
sudo a2disconf cabecera-prueba
sudo rm /etc/apache2/conf-available/cabecera-prueba.conf
sudo a2dismod headers
sudo systemctl reload apache2
```

### 2.10 Caso práctico sobre Solvia: ¿qué necesita tu servidor y qué le sobra?

Con la lista del apartado 2.2 delante, la pregunta de un administrador es: **para cada módulo cargado, ¿para qué está aquí?** La tabla resume los veinte módulos compartidos de un Apache con PHP en Ubuntu y su relación con Solvia. La última columna distingue tres situaciones: lo que Solvia necesita para funcionar hoy, lo que se va a usar más adelante en la unidad, y lo que no se usa de forma evidente.

| Módulo | Qué hace | En Solvia |
|---|---|---|
| `mpm_prefork` | Modelo con el que Apache reparte las peticiones entre procesos (apartado 2.14) | Necesario: lo exige `php8.3` |
| `php8.3` | Ejecuta los ficheros `.php` | Necesario: sin él, Solvia no es una aplicación |
| `dir` | Decide qué fichero se sirve cuando se pide una carpeta (`DirectoryIndex`, como `index.php`) | Necesario: la portada se pide como `/` |
| `authz_core` | Entiende las reglas `Require` | Necesario: lo usa el propio `solvia.conf` y otros ficheros del paquete |
| `authz_host` | Permite `Require ip` y `Require local` | Se usa en `mod_status` |
| `mime` | Asocia extensiones con tipos de contenido (`.css` es `text/css`) | Necesario en la práctica: sin él, no sale `Content-Type` |
| `alias` | Permite `Alias` y `Redirect` | Se usará más adelante (redirecciones) |
| `status` | Muestra el estado del servidor en `/server-status` | Se usará (apartado 2.11) |
| `auth_basic`, `authn_file`, `authz_user` | Autenticación con usuario y contraseña desde un fichero | Se usarán para proteger el sitio de informes (CE 2.d) |
| `reqtimeout` | Limita el tiempo para recibir las cabeceras y el cuerpo (apartado 1.14) | Protección útil: se deja |
| `autoindex` | Genera el listado de una carpeta cuando no hay `index` y `Options Indexes` está activo | **Candidato a revisar**: es la causa del listado de `/assets/` |
| `negotiation` | Negociación de contenido (idioma, formato) | No se usa |
| `deflate` | Comprime las respuestas | Mejora el rendimiento; sin consecuencias de seguridad evidentes |
| `env`, `setenvif`, `filter`, `access_compat` | Variables de entorno, reglas condicionales por cabecera, cadenas de filtros y la sintaxis antigua `Order`/`Allow` | De apoyo; no se tocan sin motivo |

Esta tabla es el punto de partida para razonar, no para decidir. La decisión se toma con la regla del apartado siguiente, y la tabla puede variar si instalas otros paquetes.

**Un módulo en el que merece la pena detenerse es `autoindex`.** Cuando una carpeta no tiene fichero `index` y la opción `Options Indexes` está activa en ese sitio, Apache genera una página con la lista de los ficheros de la carpeta. Eso es lo que ocurre en la carpeta `/assets/` de Solvia, y en muchos servidores es una fuga de información. Hay dos formas de corregirlo y están en niveles distintos: **quitar el módulo** (`a2dismod -f autoindex`, que elimina la función en todo el servidor) o **quitar la opción** en el sitio concreto (`Options -Indexes`, que se estudia en el apartado 4, y deja el módulo cargado). Con el módulo desactivado, la misma petición devuelve `404`, y con la opción quitada, `403`; el apartado 2.12 explica cuál se prefiere y por qué.

### 2.11 `mod_status`: lo que el servidor cuenta de sí mismo, y a quién

`mod_status` publica una página con el estado interno del servidor en la ruta `/server-status`. Viene activado en Ubuntu, y su configuración es esta:

```bash
grep -v '^\s*#' /etc/apache2/mods-available/status.conf | grep -v '^\s*$'
```

```text
<Location /server-status>
	SetHandler server-status
	Require local
</Location>
ExtendedStatus On
<IfModule mod_proxy.c>
	ProxyStatus On
</IfModule>
```

La orden quita las líneas de comentario (las que empiezan por `#`) y las vacías para mostrar solo lo que cuenta. El bloque `<Location /server-status>` dice que, para la dirección `/server-status`, el servidor no entregue un fichero sino que el *handler* (el encargado de producir la respuesta) sea el del estado del servidor. La regla `Require local` restringe el acceso: solo se atiende a peticiones que llegan **desde la propia máquina**. Cualquier otro cliente recibe `403 Forbidden`. `ExtendedStatus On` hace que la página incluya datos de cada petición.

Desde la propia máquina se puede ver:

```bash
curl -s "http://localhost/server-status?auto"
```

```text
localhost
ServerVersion: Apache/2.4.58 (Ubuntu)
ServerMPM: prefork
Server Built: 2026-10-08T19:58:01
CurrentTime: Tuesday, 03-Nov-2026 13:31:35 CET
RestartTime: Tuesday, 03-Nov-2026 13:31:33 CET
ServerUptimeSeconds: 1
ServerUptime: 1 second
Total Accesses: 6
BusyWorkers: 1
IdleWorkers: 4
Scoreboard: ...__W__.....
```

El parámetro `?auto` pide la versión para máquinas, una línea `Nombre: valor` por dato; la salida real tiene unas treinta líneas y aquí se muestran las más claras. Informa de la versión exacta, del MPM, de cuánto lleva el servidor en marcha, de cuántos procesos están ocupados (`BusyWorkers`) o libres (`IdleWorkers`) y del `Scoreboard`, un tablero con un carácter por proceso (`_` esperando, `W` enviando respuesta, `.` hueco libre).

La página para humanos, sin `?auto`, añade una tabla con **una fila por petición reciente**, y cada fila incluye la dirección IP del cliente, el sitio virtual que atendió y la petición completa, con su ruta y sus parámetros. Se muestran solo cuatro de las columnas de la tabla real:

```text
Client     Protocol  VHost         Request
127.0.0.1  http/1.1  127.0.0.1:80  GET /nada HTTP/1.1
127.0.0.1  http/1.1  127.0.0.1:80  GET /server-status?auto HTTP/1.1
```

Esa tabla es lo que hace delicado al módulo. Si cualquiera pudiera abrir `/server-status`, vería de qué direcciones llegan los usuarios, qué rutas se consultan y qué parámetros viajan en la petición (a veces, identificadores o *tokens* dentro de la dirección). Además, le habrías dicho a un atacante la versión exacta y la carga del servidor. Es información de reconocimiento de primera calidad.

Por eso la regla `Require local` es una decisión de seguridad de quien empaquetó Apache: el módulo viene activado **y cerrado**. Hay tres cosas que conviene entender sobre esa regla:

- **Un cambio descuidado la abre.** Si alguien, para poder ver la página desde su portátil, sustituye `Require local` por `Require all granted`, el estado del servidor queda público.
- **No hace falta eliminar el módulo para protegerlo.** Se puede dejar cerrado a todos salvo a quien se autentique; es lo que se hace con el sitio de informes en el apartado 5.
- **Si no se usa, se puede quitar.** Con el módulo desactivado, `/server-status` devuelve `404` y la superficie de ataque desaparece. Es la opción correcta en un servidor donde nadie va a consultar esa página.

### 2.12 ⚖️ Alternativa y criterio

**Quitar un módulo o restringir su uso con configuración.** Casi siempre hay dos formas de dejar de ofrecer una función: desactivar el módulo que la implementa (`a2dismod`) o dejarlo cargado y limitar su uso con una directiva (`Require`, `Options -Indexes`). Las dos son válidas, y se elige con este criterio:

- **Si nadie va a necesitar la función nunca, se quita el módulo.** El código que no se carga no puede tener fallos que se exploten, no consume memoria y no puede reactivarse por un descuido en un fichero de configuración. Es el caso de `negotiation` en un sitio que no tiene versiones en varios idiomas.
- **Si la función se necesita en unos sitios y en otros no, o a veces, se deja el módulo y se restringe con configuración.** Por ejemplo, el listado de carpetas de `autoindex` puede ser deseable en un sitio de descargas y no en Solvia; quitar el módulo penalizaría al primero.
- **Si el módulo es esencial o tiene dependientes, se deja.** Y se actúa con configuración. Es la razón de que `a2dismod` pida confirmación en los esenciales.

Hay una tercera alternativa que no se usa en esta unidad: **compilar Apache a mano** con solo los módulos necesarios incorporados. Se obtiene el servidor más pequeño posible, pero se pierden las actualizaciones de seguridad automáticas que da el paquete de Ubuntu, y a cambio de ahorrar unos megabytes se asume la tarea de recompilar cada vez que se publica un parche. Para un servidor de producción gestionado con `apt`, no compensa.

**Qué dejar de serie.** La pregunta «¿por qué está cargado este módulo?» debe poder responderse para cada línea de `apache2ctl -M`. Si la respuesta es «porque venía así», aún no está tomada la decisión: se puede dejar, pero entonces la justificación es «no tengo motivo para quitarlo y quitarlo tiene un riesgo», no «venía de serie».

### 2.13 ⚠️ Errores comunes

**Usar el nombre equivocado.** `sudo a2enmod mod_headers` y `sudo a2enmod headers_module` fallan con `ERROR: Module ... does not exist!`. El nombre correcto es el corto: `headers`. En el caso de PHP, el nombre incluye la versión (`php8.3`) aunque en la lista aparezca `php_module`.

**Activar el módulo y no recargar.** Se ejecuta `a2enmod`, se prueba la función y no ocurre nada. El enlace existe, pero Apache lee la configuración solo al arrancar o recargar. Se reconoce porque `apache2ctl -M` no muestra el módulo.

**Dar por bueno el módulo sin comprobar su efecto.** Que `headers_module` aparezca cargado no significa que se esté escribiendo ninguna cabecera: hace falta además una directiva `Header`. Un módulo cargado solo indica que **la función está disponible**; hay que configurarla y comprobarla desde fuera, con `curl`.

**Desactivar un módulo con `-f` sin mirar lo que depende de él.** La opción `-f` salta el aviso de los módulos esenciales. Después, `configtest` puede fallar en un fichero que parecía ajeno (por ejemplo, con `authz_core` desactivado falla `alias.conf`, porque usa `Require`). Si el servidor deja de arrancar, deshaz el último `a2dismod`.

**Editar a mano `mods-enabled/`.** Es una carpeta de enlaces gestionada con `a2enmod` y `a2dismod`; copiar ficheros dentro o borrarlos con `rm` rompe el vínculo con `mods-available/`.

**Leer `Invalid command` como errata cuando es un módulo que falta.** Con una directiva bien escrita, la causa casi siempre es que su módulo no está cargado (apartado 2.7).

**Dejar `mod_status` abierto para «poder verlo».** Cambiar `Require local` por `Require all granted` publica el estado del servidor (apartado 2.11).

### 2.14 💡 Nota técnica: el módulo que decide cómo se atienden las peticiones

Uno de los módulos de la lista no añade una función, sino que decide **cómo trabaja el servidor por dentro**: `mpm_prefork`. **MPM** significa *Multi-Processing Module* (módulo de multiprocesamiento) y es el que define cómo Apache reparte las conexiones entre procesos o hilos. Solo puede haber uno cargado a la vez. Los tres que se usan en Linux son:

| MPM | Cómo trabaja | Cuándo se usa |
|---|---|---|
| `prefork` | Un proceso independiente por conexión, sin hilos | Obligatorio con `mod_php`, que no es seguro para hilos |
| `worker` | Varios procesos, cada uno con varios hilos | Menos habitual hoy |
| `event` | Como `worker`, pero las conexiones inactivas (*keep-alive*) no ocupan un hilo | El moderno; el que usa Ubuntu por defecto cuando no se instala `mod_php` |

Tu servidor usa `prefork` por una razón concreta: `php8.3` lo exige como dependencia (apartado 2.5). Tiene una consecuencia ya vista en el apartado 1.5: con `prefork`, cada conexión mantenida abierta ocupa un proceso entero, de ahí el interés de un `KeepAliveTimeout` bajo. La alternativa moderna es usar el MPM `event` y ejecutar PHP como un servicio aparte, **PHP-FPM**, al que Apache le pasa las peticiones. Es lo habitual en servidores con mucho tráfico, y queda fuera del alcance de esta unidad.

Dos precisiones técnicas. Cambiar de MPM sí exige **reiniciar** el servicio en vez de recargarlo (es la excepción del apartado 2.4). Y, además de `mod_status`, Apache tiene otros módulos de diagnóstico que no vienen activados, como `info`, que enseña la configuración completa del servidor; sigue el mismo principio que `status`: útil para quien administra, delicado si es público.

### 2.15 Comprueba lo que has entendido

**1. Escribes `Header always set X-Frame-Options "DENY"` en un fichero de `conf-available/`, lo activas con `a2enconf` y `configtest` responde `Invalid command 'Header'`. ¿Qué ocurre y cómo se arregla?**

La directiva `Header` es del módulo `mod_headers` y ese módulo no está cargado, así que Apache no sabe qué es `Header`. No es una errata. Se arregla con `sudo a2enmod headers`, otra vez `configtest` y recarga, y se comprueba con `apache2ctl -M | grep headers` y con `curl -sI`.

**2. Has ejecutado `sudo a2enmod headers` y `curl -sI` no muestra la cabecera que esperas. Da dos causas posibles.**

La primera, que falta recargar: el enlace existe, pero Apache no ha vuelto a leer la configuración, y `apache2ctl -M` no listaría el módulo. La segunda, que el módulo ya está cargado pero no hay ninguna directiva `Header` que escriba la cabecera, o la directiva está en un contexto o fichero que no se aplica a esa respuesta.

**3. ¿Por qué `sudo a2dismod socache_shmcb` falla si `ssl` está activo, y en qué orden se desactivan los dos?**

Porque `ssl` depende de `socache_shmcb` (lo dice la línea `# Depends:` de `ssl.load`) y `a2dismod` protege esa dependencia. Se desactiva primero `ssl` (el que depende) y después, si ya no se necesita, `socache_shmcb`.

**4. `/server-status` responde `200` desde la propia máquina. ¿Qué responderá desde otro equipo de la red y por qué? ¿Qué responderá si se desactiva `status`?**

Desde otro equipo responde `403 Forbidden`, porque la regla `Require local` solo admite peticiones que llegan desde la propia máquina. Con el módulo `status` desactivado responde `404`, porque la ruta deja de existir.

**5. Para impedir el listado de ficheros de `/assets/` puedes quitar el módulo `autoindex` o quitar la opción `Indexes` del sitio. ¿Cuál eliges para Solvia y cuál para un servidor de descargas? Razónalo.**

Para Solvia sirve cualquiera de las dos, y se prefiere **quitar la opción** (`Options -Indexes`) porque `autoindex` es un módulo esencial que `a2dismod` solo quita con `-f`, y porque dejar el módulo cargado permite activarlo después en un sitio concreto. En un servidor de descargas el listado de carpetas puede ser la función deseada, así que se deja el módulo y se activa `Indexes` solo en esas carpetas.

---

*Los bloques 3 a 9, el glosario y las referencias se irán añadiendo uno a uno, en el orden de la unidad.*
