# UD2 — Administración de servidores web · Apuntes

**0614 · Despliegue de Aplicaciones Web · 2º DAW · IES Río Arba · Curso 2026-27**

**RA2 — Implanta aplicaciones web en servidores web, evaluando y aplicando criterios de configuración para su funcionamiento seguro.**

## Cómo usar estos apuntes

Estos apuntes sirven para poder aprender la unidad incluso **sin haber asistido a las clases**. Todo lo que hace falta para seguir cada apartado está dentro del texto: las explicaciones, los comandos con su salida esperada y los ejemplos completos. Si tienes un servidor Ubuntu con Apache, puedes reproducir cada ejemplo y comparar lo que ves con lo que aquí se muestra.

Cada apartado sigue el mismo orden: qué es la cosa y para qué sirve, cómo funciona, un ejemplo resuelto, un caso práctico sobre Solvia, la alternativa que se descarta y por qué, el error más frecuente, una nota técnica y, al final, unas preguntas para que compruebes que lo has entendido.

Todo lo que aparece se ha probado sobre **Ubuntu Server 24.04 con Apache 2.4.58, PHP 8.3.6, MariaDB 10.11, OpenSSL 3.0.13 y GoAccess 1.8.1**, con el código real de Solvia. Si tu máquina tiene otras versiones, pueden cambiar los números de versión que ves en las salidas. Los comandos son los mismos.

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

El bloque `<Directory>` fija las reglas de esa carpeta: `AllowOverride All` permite que ficheros `.htaccess` modifiquen la configuración, y `Require all granted` permite el acceso a cualquier cliente. En los apartados 1 y 2 se usa `solvia.local`, el nombre del sitio en UD1; si el tuyo es otro, sustitúyelo. En el apartado 3 el sitio pasa a llamarse `solvia.test`, por una razón que allí se explica, y ese es el nombre en todos los apartados siguientes.

## Convenciones

- Los comandos que tienes que escribir van en un recuadro marcado como «comando». Justo debajo, en otro marcado como «salida», está lo que debe aparecer en pantalla.
- Los ficheros de configuración o de código que tienes que escribir o editar van en recuadros con su propio lenguaje (por ejemplo, Apache o HTML).
- Los comandos que cambian algo en el sistema llevan `sudo` delante. Sin él, Linux responde con un error de permisos.
- Los números de línea de las salidas (`12:`, `23:`) pueden ser distintos en tu máquina. Lo importante es el texto.
- Las salidas con fechas, números de proceso o tamaños son de un servidor de pruebas: en el tuyo cambiarán. Lo importante es la forma.

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

La orden no imprime ningún resultado: el módulo no está cargado. Si todavía no has resuelto el aviso `AH00558` (apartado 1.9), lo verás en pantalla igualmente: Apache lo escribe por un canal que `grep` no filtra, y no forma parte del resultado de la búsqueda.

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

## 3. Sitios virtuales

**CE 2.c** — Se han creado y configurado sitios virtuales.

Este apartado explica cómo consigue un único Apache servir varias webs distintas, cómo decide cuál de ellas atiende cada petición, y qué ocurre cuando llega una petición que no corresponde a ninguna. Es la base de todo lo que viene después: la protección con contraseña, el HTTPS y los logs de esta unidad se aplican **a un sitio concreto**, así que hay que saber qué es un sitio y cómo se elige. Se apoya en el ciclo de trabajo del apartado 1.7 (editar, `configtest`, recargar, comprobar) y en el reparto de ficheros de `sites-available/` y `sites-enabled/` del apartado 1.2.

Al terminar el apartado, el servidor tendrá **dos sitios** (`solvia.test`, la aplicación, e `informes.solvia.test`, un sitio nuevo que más adelante se protegerá con contraseña) y un tercero cuyo único trabajo es contestar cuando llega un nombre que no es de nadie.

### 3.1 Qué es un sitio virtual y por qué existe

Un servidor web es un programa que escucha en un puerto de una máquina. Una máquina tiene una o pocas direcciones IP, pero una organización tiene muchas webs: la aplicación, el blog, el panel de informes, la web de pruebas. Si cada web exigiera una máquina, o al menos una dirección IP propia, el coste sería enorme. Un **sitio virtual** (*virtual host*) resuelve el problema: es un bloque de configuración que describe **una web concreta** (con qué nombre responde, qué carpeta sirve, qué reglas aplica y dónde anota sus registros) de modo que un único proceso de Apache pueda alojar muchas webs a la vez, cada una con su propia configuración.

En Apache, cada sitio virtual se escribe en un bloque `<VirtualHost>`:

```apache
<VirtualHost *:80>
        ServerName informes.solvia.test
        DocumentRoot /var/www/informes
</VirtualHost>
```

El `*:80` de la primera línea significa «cualquier dirección IP de esta máquina, puerto 80». Dentro del bloque, las directivas valen **solo para ese sitio**: el `DocumentRoot` de este bloque no afecta a ningún otro.

Hay tres formas de distinguir un sitio de otro cuando llega una petición:

| Forma | El sitio se elige según… | Cuándo se usa |
|---|---|---|
| **Por nombre** (*name-based*) | El nombre que escribió el cliente (`informes.solvia.test`) | Es la habitual: muchos sitios, una sola dirección IP y un solo puerto |
| **Por dirección IP** (*IP-based*) | La dirección IP a la que llegó la conexión | Cuando cada sitio tiene su propia IP: el servidor tiene varias tarjetas o direcciones |
| **Por puerto** | El puerto al que llegó la conexión (`80`, `8080`…) | Pruebas, o servicios internos en un puerto distinto |

Esta unidad usa la **primera**, la de nombre. Es la que permite que `solvia.test` e `informes.solvia.test` convivan en la misma máquina, en la misma dirección y en el mismo puerto, y es la que se encuentra en casi cualquier servidor real.

### 3.2 Cómo elige Apache el sitio: el nombre que viaja en la petición

Para que Apache pueda elegir por nombre, el cliente tiene que decirle **qué nombre ha escrito el usuario**. La dirección IP no sirve: `solvia.test` y `informes.solvia.test` pueden apuntar a la misma IP, y a la conexión solo llega la IP. La solución está en el protocolo HTTP. Toda petición HTTP/1.1 incluye una cabecera llamada **`Host`** con el nombre (y, si no es el estándar, el puerto) que el usuario escribió en la barra de direcciones. Si escribes `http://informes.solvia.test/`, el navegador envía:

```plaintext
GET / HTTP/1.1
Host: informes.solvia.test
```

Con esa cabecera, Apache sigue siempre la misma regla, que conviene aprender de memoria porque explica todo lo que ocurre en este apartado:

```plaintext
Llega una petición
│
├─ Apache mira la cabecera Host
│
├─ ¿Coincide con el ServerName o con un ServerAlias de algún sitio?
│     ├─ Sí → contesta ese sitio
│     └─ No → contesta el PRIMER sitio que Apache leyó para esa IP y puerto
│            (el «sitio por defecto»)
```

Dos directivas dan los nombres de un sitio:

- **`ServerName`** es el nombre principal del sitio. Cada sitio debe tener uno. Es el que Apache usa cuando tiene que escribir una dirección completa (en una redirección, por ejemplo).
- **`ServerAlias`** añade más nombres al mismo sitio. Puede repetirse o llevar varios nombres separados por espacios: `ServerAlias www.solvia.test localhost`. Un nombre, por tanto, puede ser el `ServerName` de un sitio o el `ServerAlias` de otro, pero no de dos a la vez.

La comparación no distingue mayúsculas de minúsculas (`Solvia.TEST` y `solvia.test` son el mismo nombre).

Lo más importante de la regla es su última línea. **Si ningún sitio coincide, Apache no da un error: contesta el primero.** Con un solo sitio configurado, ese sitio contesta a todo lo que llegue, sea cual sea el nombre pedido, porque no hay nada entre lo que elegir. Eso fue lo que ocurrió desde el primer día con Solvia: se podía entrar por `localhost`, por `127.0.0.1` o por cualquier nombre que apuntara a la máquina, y siempre salía Solvia. Con dos sitios, el comportamiento sigue siendo el mismo, pero ahora hay que saber **cuál es el primero**, y ahí está la trampa que se examina en el apartado 3.7.

### 3.3 Nombres de prueba: el dominio `.test` y el fichero `hosts`

Para tener dos sitios hacen falta dos nombres. Y un nombre solo sirve si el cliente sabe a qué dirección IP corresponde. En Internet eso lo resuelve el DNS, pero en un servidor de pruebas no se va a registrar un dominio real. Hay dos decisiones que tomar.

**Qué terminación usar.** Existe una lista de terminaciones de dominio reservadas para pruebas, definida en el RFC 2606: `.test`, `.example`, `.invalid` y `.localhost`. Nunca se asignarán en Internet, así que un nombre como `solvia.test` no puede chocar con una web real. Esta unidad usa **`.test`**. En UD1 el sitio se llamaba `solvia.local`; `.local` conviene evitarlo porque se usa para el descubrimiento automático de equipos en la red local (mDNS), de modo que algunos sistemas intentan resolverlo de una forma distinta a la normal y pueden dar retrasos o resultados raros.

**Cómo se enseña al sistema a qué dirección apunta el nombre.** Antes de consultar al DNS, Linux mira un fichero local, **`/etc/hosts`**, que es la libreta de direcciones de la máquina: una línea por dirección, con la IP primero y uno o varios nombres detrás.

```bash
echo "127.0.0.1 solvia.test informes.solvia.test" | sudo tee -a /etc/hosts
tail -n 3 /etc/hosts
```

```text
127.0.0.1 localhost
127.0.1.1 ubuntu-server
127.0.0.1 solvia.test informes.solvia.test
```

`127.0.0.1` es la dirección de la propia máquina (*loopback*): los dos nombres apuntan al propio servidor. La opción `-a` de `tee` es esencial: significa *append* (añadir al final). Sin ella, `tee` **reemplaza** el contenido del fichero y se pierde la línea de `localhost`. (El nombre de tu máquina en la segunda línea y el resto de líneas pueden ser distintos; lo importante es que tu línea aparezca al final.)

Si pruebas desde el navegador del equipo anfitrión (el Windows en el que corre VirtualBox) en vez de desde dentro de la máquina virtual, el equivalente es el fichero `C:\Windows\System32\drivers\etc\hosts`, que solo se puede editar con permisos de administrador. Para estos apuntes todas las comprobaciones se hacen desde dentro de la máquina virtual con `curl`, porque no depende de nada que no controles.

**Cómo comprobar sin tocar ningún fichero.** `curl` permite decir a mano qué nombre se pide, con la opción `-H` (añade una cabecera). Esto es lo que usan los ejemplos de este apartado:

```bash
curl -s -H "Host: informes.solvia.test" http://localhost/
```

`-s` (*silent*) quita los mensajes de progreso de `curl`. La orden se conecta a `localhost` (la propia máquina, puerto 80) pero **afirma** que el usuario escribió `informes.solvia.test`. Para Apache, que solo mira la cabecera `Host`, es indistinguible de una petición normal. Es la forma de **probar la regla de elección sin depender de la resolución de nombres**, y la de provocar a propósito peticiones con un nombre que no existe.

Hay una tercera opción, `--resolve nombre:puerto:IP`, que le dice a `curl` a qué IP debe conectar cuando vea ese nombre, sin editar `/etc/hosts`:

```bash
curl -s -o /dev/null -w "%{http_code}\n" --resolve otro.test:80:127.0.0.1 http://otro.test/
```

```text
403
```

Aquí `-o /dev/null` descarta la página recibida y `-w "%{http_code}\n"` escribe solo el código de estado de la respuesta. El `403` es el sitio por defecto del apartado 3.8.

### 3.4 Dónde viven los sitios y cómo se activan

Cada sitio es un fichero en `/etc/apache2/sites-available/` (los disponibles). Para que Apache lo use, debe haber un **enlace simbólico** a ese fichero en `/etc/apache2/sites-enabled/` (los activados). Es el mismo patrón `available`/`enabled` de los módulos (apartado 2.3) y de los fragmentos de `conf-available/` (apartado 1.9), con sus propias órdenes:

| Orden | Qué hace |
|---|---|
| `sudo a2ensite nombre` | Crea el enlace en `sites-enabled/`: activa el sitio. El nombre es el del fichero sin `.conf` |
| `sudo a2dissite nombre` | Quita el enlace: desactiva el sitio sin borrar el fichero |
| `a2query -s` | Lista los sitios activados |

Tras activar o desactivar un sitio hay que **recargar** Apache (`sudo systemctl reload apache2`), igual que con cualquier cambio.

Una consecuencia importante del orden en que Apache lee los ficheros: `apache2.conf` incluye todo `sites-enabled/*.conf` con la expansión de un comodín, y el comodín devuelve los ficheros **por orden alfabético de nombre**. El sitio cuyo fichero va antes en orden alfabético es el primero que Apache lee. Es la clave del sitio por defecto.

La herramienta para comprobar cómo ha interpretado Apache los sitios es la que ya se vio en el apartado 1.6:

```bash
sudo apache2ctl -S
```

Cuando hay un sitio solo, la parte importante de la salida es:

```text
VirtualHost configuration:
*:80                   is a NameVirtualHost
         default server solvia.test (/etc/apache2/sites-enabled/solvia.conf:1)
         port 80 namevhost solvia.test (/etc/apache2/sites-enabled/solvia.conf:1)
```

Se lee así: «en cualquier IP, puerto 80 (`*:80`), Apache elige el sitio por nombre. El sitio por defecto (*default server*) es `solvia.test`, definido en la línea 1 de `sites-enabled/solvia.conf`; y hay un sitio por nombre, también ese». La palabra **`default server`** señala el sitio que contestará a lo que no coincida con nada. Es la línea más importante de la salida y la que más se pasa por alto. Las líneas siguientes de `apache2ctl -S` (`ServerRoot`, `Main ErrorLog`…) ya se explicaron en el apartado 1.6.

Dato de utilidad: `apache2ctl -S` muestra lo que Apache **ha cargado de verdad**, mientras que `ls /etc/apache2/sites-enabled/` solo muestra qué ficheros hay. Si un fichero tiene un error o un enlace roto, `ls` lo enseña igualmente y `-S` no. Por eso `-S` es la comprobación fiable.

### 3.5 Anatomía de un sitio

Conviene leer con calma el fichero de un sitio, porque se va a ampliar a lo largo de toda la unidad. El de Solvia, tras cambiarle el nombre, es:

```apache
<VirtualHost *:80>
        ServerName solvia.test
        DocumentRoot /var/www/solvia/public

        <Directory /var/www/solvia/public>
                AllowOverride All
                Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/solvia_error.log
        CustomLog ${APACHE_LOG_DIR}/solvia_access.log combined
</VirtualHost>
```

- **`<VirtualHost *:80>`** abre el sitio: atiende conexiones a cualquier IP de la máquina en el puerto 80. El puerto debe estar entre los que Apache escucha (`Listen` en `ports.conf`, apartado 1.5).
- **`ServerName`** es el nombre del sitio (apartado 3.2).
- **`DocumentRoot`** es la carpeta que se sirve como raíz del sitio. En Solvia es `public/`, y no la carpeta de la aplicación entera, para que el código, la configuración y la base de datos no sean accesibles desde el navegador (apartado 4.1).
- **`<Directory ruta>`** agrupa reglas que valen para una carpeta del disco. `Require all granted` permite el acceso a cualquier cliente, y `AllowOverride All` permite que los ficheros `.htaccess` modifiquen la configuración (apartado 4.4).
- **`ErrorLog` y `CustomLog`** dicen dónde anota este sitio sus errores y sus accesos. `${APACHE_LOG_DIR}` es una variable que vale `/var/log/apache2` (apartado 1.5). Al escribir en ficheros propios, los registros de cada sitio quedan separados (apartado 8).

Un sitio nuevo es siempre una variación de este esquema: cambian el nombre, la carpeta y los registros.

### 3.6 Ejemplo resuelto: un segundo sitio, `informes`

Se parte de la máquina con un único sitio, `solvia`, y se quiere añadir `informes.solvia.test`, que servirá una carpeta distinta. Más adelante (apartado 5) se protegerá con contraseña, y más adelante todavía (apartado 8) publicará un informe de los registros. Por ahora solo tendrá una página de prueba.

**Paso 1. Comprueba el punto de partida.**

```bash
sudo apache2ctl -S | head -n 6
```

```text
VirtualHost configuration:
*:80                   is a NameVirtualHost
         default server solvia.test (/etc/apache2/sites-enabled/solvia.conf:1)
         port 80 namevhost solvia.test (/etc/apache2/sites-enabled/solvia.conf:1)
ServerRoot: "/etc/apache2"
Main DocumentRoot: "/var/www/html"
```

Un solo sitio. `head -n 6` se queda con las seis primeras líneas.

**Paso 2. Haz una copia de `solvia.conf` y cambia su nombre a `solvia.test`.** Es el mismo patrón de seguridad que con `security.conf` (apartado 1.7): copia antes de tocar, y `diff` después para ver exactamente qué ha cambiado.

```bash
cp /etc/apache2/sites-available/solvia.conf ~/solvia.conf.orig
sudo sed -i -E 's/^(\s*ServerName\s+).*/\1solvia.test/' /etc/apache2/sites-available/solvia.conf
diff ~/solvia.conf.orig /etc/apache2/sites-available/solvia.conf
```

```text
2c2
<         ServerName solvia.local
---
>         ServerName solvia.test
```

La orden `sed` cambia en el fichero la línea que empieza por `ServerName` (con la sangría que tenga) y deja el nombre nuevo; `-i` modifica el fichero en su sitio, `-E` activa las expresiones regulares ampliadas, y `\1` repite lo que casó entre paréntesis, es decir, la sangría y la palabra `ServerName`. Puedes hacer el mismo cambio con `sudo nano` si lo prefieres. La salida de `diff` se lee así: `2c2` significa «la línea 2 ha cambiado»; las líneas con `<` son del fichero antiguo y las de `>` del nuevo.

**Paso 3. Crea la carpeta y la página de `informes`.**

```bash
sudo mkdir -p /var/www/informes
echo "<h1>INFORMES</h1>" | sudo tee /var/www/informes/index.html
```

```text
<h1>INFORMES</h1>
```

`mkdir -p` crea la carpeta (y sus padres si faltaran, sin quejarse si ya existe). `tee` escribe el texto en el fichero y también lo repite por pantalla.

**Paso 4. Crea el fichero del sitio.**

```bash
sudo tee /etc/apache2/sites-available/informes.conf <<'EOF'
<VirtualHost *:80>
        ServerName informes.solvia.test
        DocumentRoot /var/www/informes

        <Directory /var/www/informes>
                Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/informes_error.log
        CustomLog ${APACHE_LOG_DIR}/informes_access.log combined
</VirtualHost>
EOF
```

La construcción `<<'EOF' … EOF` es un *here-document*: todo lo que hay entre las dos marcas `EOF` se envía como entrada de `tee`. Las comillas simples alrededor de la primera `EOF` hacen que la terminal no sustituya `${APACHE_LOG_DIR}` por su valor (que no existe en tu terminal): el texto debe llegar al fichero tal cual, y es Apache quien resuelve la variable al leerlo. El `Directory` solo lleva `Require all granted` porque no hace falta más para servir una página estática. `tee` repetirá el contenido del fichero por pantalla.

**Paso 5. Activa el sitio y comprueba.**

```bash
sudo a2ensite informes
sudo apache2ctl configtest
sudo systemctl reload apache2
sudo apache2ctl -S | head -n 8
```

```text
Enabling site informes.
To activate the new configuration, you need to run:
  service apache2 reload
Syntax OK
VirtualHost configuration:
*:80                   is a NameVirtualHost
         default server informes.solvia.test (/etc/apache2/sites-enabled/informes.conf:1)
         port 80 namevhost informes.solvia.test (/etc/apache2/sites-enabled/informes.conf:1)
         port 80 namevhost solvia.test (/etc/apache2/sites-enabled/solvia.conf:1)
ServerRoot: "/etc/apache2"
Main DocumentRoot: "/var/www/html"
```

Hay dos sitios. Y la línea `default server` ya no es `solvia.test`, sino **`informes.solvia.test`**. Nadie ha cambiado el sitio por defecto a propósito: ha cambiado solo porque `informes.conf` va antes que `solvia.conf` en orden alfabético (la `i` va antes que la `s`). Es la clave del apartado 3.7.

**Paso 6. Pide cada sitio por su nombre.**

```bash
curl -s http://solvia.test/ | head -n 3
curl -s http://informes.solvia.test/
```

```text
<!DOCTYPE html>
<html lang="es">
<head>
<h1>INFORMES</h1>
```

(La primera orden enseña el principio del HTML de Solvia, que será distinto en cada versión; lo importante es que no es la página de informes. La segunda muestra la página de prueba.) Cada nombre sirve su sitio.

### 3.7 La trampa: lo que contesta Apache cuando el nombre no es de nadie

Con los dos sitios funcionando, se repite el experimento de la regla: se pide un nombre que no es de ninguno.

```bash
curl -s -H "Host: nada.test" http://localhost/
curl -s http://localhost/
curl -s http://127.0.0.1/
```

```text
<h1>INFORMES</h1>
<h1>INFORMES</h1>
<h1>INFORMES</h1>
```

Las tres peticiones tienen un nombre (`nada.test`, `localhost`, y la IP escrita como dirección) que no es ni el `ServerName` ni un `ServerAlias` de ningún sitio. Apache aplica la última línea de la regla y contesta el primero que leyó: **`informes`**. Todo funciona «bien»: no hay ningún error, ningún mensaje y el servidor responde `200 OK`. Y sin embargo hay dos problemas reales:

- **Antes se entraba a Solvia por `localhost`; ahora ya no.** `http://localhost/` (y `http://localhost:8080/` desde el navegador del equipo anfitrión, con el reenvío de puertos de VirtualBox) servía Solvia cuando había un sitio solo. Ahora sirve informes, aunque nada se haya tocado en el sitio de Solvia.
- **Informes pasa a ser el sitio que recibe lo que nadie pidió.** Cualquiera que llegue por la IP, o con un nombre equivocado, o con un nombre que alguien ha apuntado hacia el servidor sin que tú lo sepas, ve el contenido de `informes`. Es el sitio que más adelante contendrá información interna. Que sea el «cajón de sastre» del servidor es mala idea.

La causa de fondo es que **el sitio por defecto lo ha decidido el orden alfabético de los nombres de fichero**, y eso es una propiedad accidental de cómo se llaman los ficheros. Basta con crear un sitio nuevo cuyo nombre empiece por una letra anterior para que cambie sin avisar. En un servidor con diez sitios mantenidos por varias personas, nadie sabría decir cuál es el que contesta a lo desconocido.

### 3.8 El arreglo: un sitio por defecto puesto a propósito

La solución es no depender del orden: crear un sitio cuyo único trabajo es contestar cuando llega un nombre que no es de nadie, y asegurarse de que es el primero que Apache lee. Se le da un nombre de fichero que empieza por `00-`, para que quede el primero en orden alfabético, y lo que contesta es un **`403 Forbidden`**: ni una página, ni una redirección, ni información.

```bash
sudo mkdir -p /var/www/defecto
sudo tee /etc/apache2/sites-available/00-defecto.conf <<'EOF'
<VirtualHost *:80>
        ServerName defecto.test
        DocumentRoot /var/www/defecto

        <Directory /var/www/defecto>
                Require all denied
        </Directory>
</VirtualHost>
EOF
sudo a2ensite 00-defecto
sudo apache2ctl configtest
sudo systemctl reload apache2
sudo apache2ctl -S | head -n 9
```

```text
Enabling site 00-defecto.
To activate the new configuration, you need to run:
  service apache2 reload
Syntax OK
VirtualHost configuration:
*:80                   is a NameVirtualHost
         default server defecto.test (/etc/apache2/sites-enabled/00-defecto.conf:1)
         port 80 namevhost defecto.test (/etc/apache2/sites-enabled/00-defecto.conf:1)
         port 80 namevhost informes.solvia.test (/etc/apache2/sites-enabled/informes.conf:1)
         port 80 namevhost solvia.test (/etc/apache2/sites-enabled/solvia.conf:1)
ServerRoot: "/etc/apache2"
```

**`Require all denied`** es lo contrario de `Require all granted`: deniega el acceso a cualquier cliente. La carpeta `/var/www/defecto` existe pero no contiene nada; ni siquiera hace falta, porque no se llega a leer, pero Apache se queja en `configtest` si el `DocumentRoot` no existe. El `default server` es ahora `defecto.test`. El nombre `defecto.test` no importa: nadie lo va a pedir.

Se comprueba qué contesta cada petición. Esta vez se descarta la página (`-o /dev/null`) y solo se escribe el código de estado (`-w "%{http_code}\n"`):

```bash
curl -s http://solvia.test/ | head -n 3
curl -s http://informes.solvia.test/
curl -s -o /dev/null -w "%{http_code}\n" -H "Host: nada.test" http://localhost/
curl -s -o /dev/null -w "%{http_code}\n" http://localhost/
```

```text
<!DOCTYPE html>
<html lang="es">
<head>
<h1>INFORMES</h1>
403
403
```

Los dos sitios siguen igual, y lo que no es de nadie recibe un `403`. El inconveniente es que **`http://localhost/` también da `403`**: `localhost` no es el nombre de ningún sitio, así que recibe la respuesta de lo desconocido. Si quieres seguir entrando a Solvia por `localhost` (por ejemplo, desde el navegador del equipo anfitrión a través del puerto 8080), hay que decirle a Solvia que también responde a ese nombre. Para eso sirve **`ServerAlias`**, que añade un nombre más al mismo sitio. Se coloca justo debajo del `ServerName`:

```bash
sudo sed -i -E 's/^(\s*ServerName\s+solvia.test)$/\1\n        ServerAlias localhost/' /etc/apache2/sites-available/solvia.conf
sed -n 1,4p /etc/apache2/sites-available/solvia.conf
```

```text
<VirtualHost *:80>
        ServerName solvia.test
        ServerAlias localhost
        DocumentRoot /var/www/solvia/public
```

En la orden `sed`, el `\n` inserta un salto de línea y los ocho espacios mantienen la sangría. Si el nombre de tu sitio no es exactamente `solvia.test`, `sed` no cambia nada: mira primero con `sed -n 1,4p` cómo está tu fichero, o añade la línea con `nano`. Ahora recarga y comprueba:

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -s http://localhost/ | head -n 3
curl -s -o /dev/null -w "%{http_code}\n" -H "Host: nada.test" http://localhost/
sudo apache2ctl -S | sed -n 3,8p
```

```text
Syntax OK
<!DOCTYPE html>
<html lang="es">
<head>
403
         default server defecto.test (/etc/apache2/sites-enabled/00-defecto.conf:1)
         port 80 namevhost defecto.test (/etc/apache2/sites-enabled/00-defecto.conf:1)
         port 80 namevhost informes.solvia.test (/etc/apache2/sites-enabled/informes.conf:1)
         port 80 namevhost solvia.test (/etc/apache2/sites-enabled/solvia.conf:1)
                 alias localhost
ServerRoot: "/etc/apache2"
```

Solvia responde por `localhost` y lo desconocido sigue dando `403`. En `apache2ctl -S` aparece ahora la línea `alias localhost` bajo `solvia.test`: es como se ve un `ServerAlias`.

### 3.9 Caso práctico sobre Solvia: qué contesta cada petición, y por qué

Con tres sitios activos (`00-defecto`, `informes` y `solvia`, este último con el alias `localhost`), esto es lo que contesta el servidor ante cada petición. La columna importante es la última: no basta con saber qué contesta, hay que saber **qué regla lo decide**.

| Petición | Contesta | Por qué |
|---|---|---|
| `http://solvia.test/` | `solvia` | El nombre coincide con su `ServerName` |
| `http://informes.solvia.test/` | `informes` | El nombre coincide con su `ServerName` |
| `http://localhost/` | `solvia` | El nombre coincide con su `ServerAlias` |
| `Host: nada.test` | `defecto` (403) | Ningún nombre coincide: contesta el primero que Apache lee, que ahora es `00-defecto` |
| `http://127.0.0.1/` (la IP) | `defecto` (403) | Una dirección IP no es el nombre de ningún sitio: igual que el caso anterior |

Esta tabla es una buena comprobación de que has entendido el apartado. Si alguna fila te sorprende, repasa la regla del apartado 3.2.

Para comprobar el estado final, ejecuta `sudo apache2ctl -S`, que debe mostrar tres sitios (el `default server` es `defecto.test`) y el alias de Solvia, y los cuatro `curl` del apartado 3.8.

### 3.10 ⚖️ Alternativa y criterio

**Sitios por nombre, por IP o por puerto.** Los tres métodos del apartado 3.1 sirven para distinguir sitios, y se elige con este criterio:

- **Por nombre**, cuando todos los sitios comparten IP y puerto. Es la opción por defecto porque no consume direcciones IP (que son escasas y, en la nube, de pago) y funciona con cualquier navegador moderno. Es la que se enseña aquí.
- **Por IP**, cuando cada sitio necesita su propia dirección: por ejemplo, para exponer unos sitios en una red interna y otros en la pública a través de dos tarjetas distintas, o cuando hay software antiguo que no envía la cabecera `Host`.
- **Por puerto**, para servicios que no son la web principal: un panel de administración en el `8080`, o varias versiones de una aplicación en pruebas. Tiene el inconveniente de que hay que escribir el puerto en cada dirección y que algunos cortafuegos solo dejan pasar el 80 y el 443.

**Un sitio por defecto explícito o dejar que lo decida el orden.** Dejarlo al orden alfabético funciona hasta el día en que alguien añade un fichero con otro nombre, y entonces cambia el sitio que recibe lo desconocido sin que nada avise. Un sitio por defecto explícito (`00-defecto.conf`) tiene un coste mínimo (un fichero de seis líneas) y convierte un comportamiento accidental en una decisión. Es la práctica que se debe seguir en cualquier servidor con más de un sitio.

**Qué debe contestar el sitio por defecto.** Un `403` es la opción más simple y la que menos información da. Hay otras: una página en blanco, una redirección al sitio principal o cerrar la conexión sin respuesta (con la directiva `Redirect` o con un módulo de reescritura). Un `403` es preferible a servir una página, porque no sugiere que haya nada detrás.

**Un fichero por sitio o varios sitios en un fichero.** Apache admite varios `<VirtualHost>` en un mismo fichero. Se prefiere **un fichero por sitio** porque `a2ensite` y `a2dissite` trabajan con ficheros: con uno por sitio, se puede activar o desactivar cada web sin tocar las demás, y un error de sintaxis en una no obliga a editar el fichero de las otras.

### 3.11 ⚠️ Errores comunes

**Olvidar la `-a` en `tee`.** `echo "..." | sudo tee /etc/hosts` reemplaza el fichero entero y se pierde `127.0.0.1 localhost`. Se arregla con `echo "127.0.0.1 localhost" | sudo tee -a /etc/hosts` y volviendo a añadir tu línea.

**Crear el enlace en `sites-enabled/` a mano.** La carpeta de enlaces la gestiona `a2ensite`; copiar un fichero dentro, en vez de enlazarlo, deja dos copias que se desincronizan al editar la de `sites-available/`.

**Activar el sitio y no recargar.** `apache2ctl -S` sigue mostrando los sitios antiguos. Se arregla con `sudo systemctl reload apache2`.

**Esperar que `a2ensite` valide el sitio.** Solo crea el enlace. Los errores de sintaxis salen en `configtest`, y las carpetas que no existen también.

**Nombre del fichero sin `00-` en el sitio por defecto.** Si el fichero se llama `defecto.conf`, va después de `informes.conf` en orden alfabético y el sitio por defecto vuelve a ser `informes`. Se detecta con `apache2ctl -S`: la línea `default server` no es la que esperabas.

**Poner dos sitios con el mismo `ServerName`.** Apache 2.4 **no avisa**: `configtest` responde `Syntax OK` y atiende solo el que lee primero (el de nombre de fichero anterior en orden alfabético). Se reconoce porque el segundo sitio nunca contesta, y `apache2ctl -S` lo delata: el mismo nombre aparece en dos líneas `namevhost`.

**Probar con el nombre y no resolverlo.** `curl http://informes.solvia.test/` falla con `Could not resolve host` si falta la línea de `/etc/hosts`. Si lo que se quiere es comprobar solo la elección de sitio, `curl -H "Host: ..." http://localhost/` no depende de la resolución.

**Poner el puerto en un lugar equivocado.** Un `<VirtualHost *:80>` solo atiende el puerto 80. Si más adelante añades un sitio en el 443, necesita su propio bloque `<VirtualHost *:443>` (apartado 7).

### 3.12 💡 Nota técnica: el nombre también viaja en HTTPS

Todo el mecanismo de este apartado depende de que el servidor conozca el nombre pedido **antes de decidir qué sitio contesta**. En HTTP eso ocurre al leer la cabecera `Host`. Con HTTPS aparece un problema: la conexión se cifra antes de enviar la petición, y el servidor necesita presentar un **certificado** (apartado 6) antes de poder leer la cabecera `Host`. ¿Cómo sabe cuál certificado mostrar si aún no sabe qué sitio se ha pedido?

La solución es una extensión del protocolo TLS llamada **SNI** (*Server Name Indication*): el cliente incluye el nombre en el primer mensaje del *handshake* (apertura de la conexión TLS), sin cifrar, y el servidor elige con él el sitio y el certificado. Es lo que permite tener varios sitios HTTPS por nombre en una sola IP, y también tiene una consecuencia de privacidad que se verá en el apartado 7: quien observa la red no puede leer lo que envías por HTTPS, pero sí puede leer **a qué nombre** te conectas.

Una precisión sobre el protocolo: la cabecera `Host` solo es obligatoria en HTTP/1.1. Un cliente muy antiguo que hable HTTP/1.0 puede no enviarla; en ese caso, Apache no tiene nombre contra el que comparar y contesta el sitio por defecto. Es una razón más para que ese sitio sea una decisión tuya y no un accidente.

Otra: si escribes `http://solvia.test:8080/`, el navegador envía `Host: solvia.test:8080`. Apache ignora el puerto al comparar con `ServerName`, así que el sitio se elige igual.

### 3.13 Comprueba lo que has entendido

**1. Un Apache tiene un solo sitio, con `ServerName solvia.test`. Alguien pide `http://nada.test/` apuntando ese nombre a la máquina. ¿Qué contesta el servidor y por qué?**

Contesta el sitio de Solvia, con un `200`. Con un solo sitio no hay nada entre lo que elegir: la última línea de la regla («si ningún nombre coincide, contesta el primero») deja ese sitio como respuesta a cualquier nombre.

**2. Añades un segundo sitio, `informes`, y de pronto `http://localhost/` deja de mostrar Solvia y muestra informes. Nadie ha tocado `solvia.conf`. ¿Qué ha pasado y cómo lo compruebas?**

Ningún sitio tiene `localhost` como nombre, así que Apache contesta el primero que leyó. Lee `sites-enabled/` por orden alfabético y `informes.conf` va antes que `solvia.conf`, por eso es el `default server`. Se comprueba con `sudo apache2ctl -S`, mirando la línea `default server`.

**3. ¿Cómo evitas que el sitio por defecto dependa del orden alfabético, y qué debe contestar?**

Creando un sitio cuyo fichero empiece por `00-` (por ejemplo `00-defecto.conf`), para que sea el primero, y haciendo que responda lo mínimo posible: un `403` con `Require all denied`. Ya no depende de cómo se llamen los demás ficheros.

**4. Pides `curl -H "Host: informes.solvia.test" http://127.0.0.1/` sin haber añadido nada a `/etc/hosts`. ¿Funciona? ¿Qué demuestra?**

Funciona. La opción `-H` fija la cabecera `Host` a mano y `curl` se conecta a la IP indicada sin resolver ningún nombre. Demuestra que Apache elige el sitio por la cabecera `Host` y no por la IP ni por cómo se resolvió el nombre.

**5. ¿Por qué es mejor un fichero por sitio que dos `<VirtualHost>` en el mismo fichero?**

Porque `a2ensite` y `a2dissite` activan y desactivan ficheros enteros: con un fichero por sitio puedes desactivar una web sin tocar la otra, y un error en una no obliga a editar la otra.

---

## 4. Ajustes para la implantación de aplicaciones

**CE 2.h** — Se han realizado los ajustes necesarios para la implantación de aplicaciones en el servidor web.

Hasta ahora el servidor ha servido Solvia correctamente. Pero «funciona» no es lo mismo que «está bien implantada». Entre una aplicación que arranca en la máquina de quien la desarrolla y una aplicación lista para recibir usuarios reales hay una serie de **ajustes** que no cambian lo que la aplicación hace, sino **qué puede ver, qué puede hacer y qué cuenta de sí misma cuando algo falla**. Este apartado recorre los cinco ajustes más importantes para una aplicación PHP sobre Apache: qué carpeta se sirve, quién puede leer los ficheros, qué opciones tiene la carpeta, si se permiten los `.htaccess` y cómo se comporta PHP y las páginas de error ante un fallo.

Se apoya en los contextos de directivas del apartado 1.4 (un bloque `<Directory>` agrupa reglas para una carpeta), en la anatomía de un sitio del apartado 3.5 y en el ciclo de trabajo del apartado 1.7. Al final se aplican todos los ajustes al sitio de Solvia, con las comprobaciones que demuestran que cada uno hace lo que debe.

### 4.1 Qué se sirve y qué no: el `DocumentRoot` y la carpeta `public/`

El primer ajuste, y el más importante, decide **qué ficheros del proyecto pueden llegar al navegador**. Apache sirve todo lo que hay dentro del `DocumentRoot`: si un fichero está dentro de esa carpeta y los permisos lo permiten, cualquiera puede pedirlo escribiendo su ruta en la barra de direcciones. Una aplicación web, además de las páginas que el usuario debe poder abrir, tiene ficheros que **nunca** debe ver: la configuración con la contraseña de la base de datos, el código fuente, el esquema de la base de datos, el historial del control de versiones.

Por eso Solvia está organizada en dos zonas:

```plaintext
/var/www/solvia/
├── config/         ← configuración y contraseñas: NO debe llegar al navegador
├── db/             ← esquema de la base de datos: NO
├── src/            ← código de la aplicación: NO
├── templates/      ← plantillas: NO
├── bootstrap.php   ← arranque de la aplicación: NO
├── README.md       ← documentación: NO
└── public/         ← lo único que el navegador puede pedir (DocumentRoot)
    ├── index.php
    ├── nuevo_ticket.php
    ├── consultar_ticket.php
    └── assets/
```

Las páginas de `public/` son «puertas»: reciben la petición, cargan el código de `src/` y las plantillas de `templates/` desde fuera de la zona servida, y devuelven el resultado. El navegador solo ve las puertas; el resto no tiene dirección web.

**Qué ocurre si se sirve la carpeta equivocada.** Se puede comprobar con un experimento controlado: un sitio temporal cuyo `DocumentRoot` sea la raíz del proyecto en vez de `public/`. Se crea un nombre de prueba, un sitio y se pide cada fichero:

```bash
echo "127.0.0.1 malo.test" | sudo tee -a /etc/hosts
sudo tee /etc/apache2/sites-available/malo.conf <<'EOF'
<VirtualHost *:80>
        ServerName malo.test
        DocumentRoot /var/www/solvia

        <Directory /var/www/solvia>
                Require all granted
        </Directory>
</VirtualHost>
EOF
sudo a2ensite malo
sudo apache2ctl configtest
sudo systemctl reload apache2
```

Y ahora se pide, desde fuera, lo que **no** debería poder pedirse:

```bash
for r in db/schema.sql README.md config/config.php src/Database.php; do printf "%-22s " $r; curl -s -o /dev/null -w "%{http_code}\n" http://malo.test/$r; done
curl -s http://malo.test/db/schema.sql | head -n 4
```

```text
db/schema.sql          200
README.md              200
config/config.php      200
src/Database.php       200
-- Solvia — esquema de base de datos (RA1)
-- Sistema interno de gestión de incidencias/tickets IT
-- Módulo 0614 Despliegue de Aplicaciones Web — 2º DAW — IES Río Arba (2026-27)

```

El bucle `for` repite la orden para cada ruta de la lista; `printf` escribe la ruta alineada y `curl` imprime solo el código de estado. Los cuatro ficheros responden `200`. El esquema completo de la base de datos y la documentación se descargan; cualquiera sabe ya qué tablas y columnas tiene la aplicación. Los dos ficheros `.php` (`config.php`, `Database.php`) devuelven la página vacía, porque Apache los **ejecuta** en vez de enviarlos, y eso es lo único que impide ver su contenido. La protección es frágil. Se puede ver qué pasa si el módulo de PHP deja de cargarse (un fallo de actualización, una configuración errónea, un `a2dismod` accidental):

```bash
sudo a2dismod php8.3
sudo systemctl restart apache2
curl -s http://malo.test/config/config.php
```

```text
<?php
return [
    'db' => [
        'host' => 'localhost',
        'name' => 'solvia',
        'user' => 'solvia_app',
        'pass' => '(la contraseña real de tu base de datos)',
    ],
];
```

Sin el módulo, Apache no sabe que `.php` es código, lo trata como texto y **envía el fichero tal cual, con la contraseña dentro**. Es la razón por la que se dice que `config.php` debe estar fuera del `DocumentRoot`: si no está servido, no puede filtrarse, funcione PHP o no. Haz este experimento solo en una máquina de pruebas: lo que verás es el contenido de tu fichero de configuración real.

Después, deshaz todo (sitio, nombre y módulo):

```bash
sudo a2enmod php8.3
sudo a2dissite malo
sudo rm /etc/apache2/sites-available/malo.conf
sudo sed -i '/malo.test/d' /etc/hosts
sudo systemctl restart apache2
```

(`a2enmod php8.3` y `restart` son necesarios porque, al ser un módulo, cambiar su estado exige reiniciar y no solo recargar, como se vio en el apartado 2.4. `sed -i '/malo.test/d'` borra de `/etc/hosts` la línea que contiene ese nombre.)

**Otros ficheros que no deben servirse.** Aunque el `DocumentRoot` sea el correcto, es fácil que en `public/` acaben ficheros que no deberían estar: un `.env` con claves, una carpeta `.git/` con todo el historial del código, copias de seguridad como `config.php.bak` o `volcado.sql`. Apache protege por defecto solo los ficheros que empiezan por `.ht` (como `.htaccess` y `.htpasswd`), con esta regla de `apache2.conf`:

```apache
<FilesMatch "^\.ht">
        Require all denied
</FilesMatch>
```

Cualquier otro fichero oculto se sirve normalmente. Se puede ampliar la protección con un fragmento propio, que deniega los ficheros y carpetas cuyo nombre empieza por punto, salvo `.well-known` (una carpeta que algunos servicios, como la emisión de certificados, necesitan que sea pública):

```apache
<DirectoryMatch "/\.git">
        Require all denied
</DirectoryMatch>
<FilesMatch "^\.(?!well-known)">
        Require all denied
</FilesMatch>
```

`<FilesMatch>` y `<DirectoryMatch>` son como `<Directory>`, pero en lugar de una ruta reciben una **expresión regular**: la primera casa con nombres de fichero y la segunda con rutas de carpeta. `^\.` significa «empieza por un punto» y `(?!well-known)` «y lo que sigue no es `well-known`». Con un `.env` y un `.git/config` colocados a propósito dentro de `public/`, la prueba antes y después del fragmento es:

```text
.env 200
.git/config 200
(...se activa el fragmento con a2enconf y se recarga...)
.env 403
.git/config 403
index 200
```

Es una red de seguridad, no un sustituto: lo correcto sigue siendo **no poner esos ficheros dentro de `public/`**.

### 4.2 Permisos: quién puede leer y quién puede escribir

El segundo ajuste decide **qué puede hacer el usuario con el que corre Apache**. Como se vio en el apartado 1.5, Apache atiende las peticiones con el usuario sin privilegios `www-data`. Eso significa que cualquier cosa que haga la aplicación PHP la hace con los permisos de `www-data`, y que cualquier fallo de seguridad en la aplicación le da a un atacante, como máximo, los permisos de `www-data`. Cuanto menos pueda hacer `www-data`, menos daño puede causar.

**Cómo leer los permisos.** La orden `ls -l` muestra el tipo, los permisos, el propietario y el grupo de cada fichero:

```bash
ls -l /var/www/solvia/config
```

```text
total 4
-rw-r----- 1 root www-data 23 Nov  9 09:53 config.php
```

La cadena `-rw-r-----` se lee en cuatro partes. El primer carácter es el tipo (`-` fichero, `d` carpeta). Los nueve siguientes son tres grupos de tres: permisos del **propietario** (`rw-`: leer y escribir), del **grupo** (`r--`: solo leer) y de **los demás** (`---`: nada). Cada grupo usa `r` (leer), `w` (escribir) y `x` (ejecutar, y en una carpeta, atravesarla). Tras el número de enlaces vienen el propietario (`root`) y el grupo (`www-data`). Con estos permisos: `root` puede leer y modificar el fichero, `www-data` solo puede leerlo (por ser del grupo), y nadie más puede ni mirarlo.

**Qué necesita Apache.** Para servir un fichero estático, `www-data` necesita permiso de **lectura** sobre el fichero y de **atravesar** (`x`) todas las carpetas del camino hasta él. Para ejecutar un `.php` solo necesita leerlo: el código lo ejecuta el módulo de PHP dentro del proceso de Apache. **No necesita escribir** en ninguna carpeta de código. Solvia guarda sus datos en la base de datos, no en ficheros, así que `www-data` no tiene motivo para modificar nada dentro de `/var/www/solvia/`.

Esto lleva a la regla de oro del ajuste de permisos:

- **El código pertenece a `root` (o a quien despliega) y `www-data` solo puede leerlo.** Si una vulnerabilidad permitiera escribir ficheros desde la aplicación, no podría modificar el propio código ni colocar nuevos `.php` en la carpeta servida.
- **Los ficheros con secretos, como `config.php`, no los lee nadie más que `root` y `www-data`.** Se consigue con propietario `root`, grupo `www-data` y modo `640`:

```bash
sudo chown root:www-data /var/www/solvia/config/config.php
sudo chmod 640 /var/www/solvia/config/config.php
ls -l /var/www/solvia/config/config.php
```

```text
-rw-r----- 1 root www-data 23 Nov  9 09:53 /var/www/solvia/config/config.php
```

`chown root:www-data` cambia propietario y grupo (`propietario:grupo`); `chmod 640` fija los permisos en notación numérica. Cada cifra es la suma de `r` = 4, `w` = 2 y `x` = 1, y las tres cifras son propietario, grupo y demás: `6` = 4+2 (leer y escribir), `4` = leer, `0` = nada. Cualquier otro usuario del sistema ya no puede leer las credenciales. Se puede comprobar actuando como cada usuario con `runuser`:

```bash
sudo runuser -u www-data -- cat /var/www/solvia/config/config.php
sudo runuser -u nobody -- cat /var/www/solvia/config/config.php
```

```text
<?php
return [ ... el contenido del fichero ... ]
cat: /var/www/solvia/config/config.php: Permission denied
```

`runuser -u usuario -- orden` ejecuta la orden como ese usuario. `www-data` puede leerlo (la aplicación lo necesita); `nobody`, un usuario cualquiera, recibe `Permission denied`.

**El error del `DocumentRoot` en la carpeta personal.** Al revés, el exceso de restricción también rompe el sitio. Si el `DocumentRoot` apunta a una carpeta que `www-data` no puede atravesar, Apache responde `403 Forbidden` por mucho que `Require all granted` esté bien escrito. Es la razón por la que Solvia se sirve desde `/var/www/` y no desde la carpeta personal de un usuario (cuya carpeta tiene permisos que impiden entrar a otros). El registro de errores lo dice con claridad:

```text
[core:error] (13)Permission denied: [client 127.0.0.1:33920] AH00035: access to / denied (filesystem path '/root/priv') because search permissions are missing on a component of the path
```

El mensaje `AH00035` con «search permissions are missing on a component of the path» significa: «falta el permiso `x` (atravesar) en alguna carpeta del camino». Al encontrarlo, se revisa cada carpeta del camino con `namei -l ruta`, que muestra los permisos de cada componente.

### 4.3 La directiva `Options`: qué le está permitido a la carpeta

La directiva **`Options`** activa o desactiva funciones de Apache **para una carpeta**. Las que importan aquí:

| Opción | Qué hace | Riesgo |
|---|---|---|
| `Indexes` | Si se pide una carpeta que no tiene `index.html`, `index.php`… (véase `DirectoryIndex`, apartado 1.5), Apache **muestra un listado de su contenido** | Revela qué ficheros hay, incluso los que no están enlazados desde ninguna página |
| `FollowSymLinks` | Apache sigue los enlaces simbólicos de la carpeta | Un enlace mal puesto puede sacar a la web ficheros de otra zona del disco |
| `ExecCGI` | Permite ejecutar programas CGI | Ejecución de código en el servidor. Sobre todo peligroso en carpetas donde se suben ficheros |
| `MultiViews` | Elige un fichero según el tipo o idioma que pide el cliente | Complica lo que se sirve y no se necesita |

El valor de partida no lo decide el sitio de Solvia: viene de `apache2.conf`, que en Ubuntu tiene un bloque para todo `/var/www/`:

```apache
<Directory /var/www/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
```

Lo que significa que **`Indexes` está activado por defecto** en cualquier sitio de `/var/www/`, y que el `<Directory>` de Solvia, que no menciona `Options`, lo hereda. Se comprueba pidiendo una carpeta de Solvia sin página de inicio:

```bash
curl -s http://solvia.test/assets/ | sed -n 1,12p
```

```text
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /assets</title>
 </head>
 <body>
<h1>Index of /assets</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th>...
```

Es el listado automático, un «Index of /assets» con el nombre, la fecha y el tamaño de cada fichero de la carpeta. Para quien visita la web es información gratuita sobre cómo está organizado el proyecto. Se quita con **`Options -Indexes`**. El signo `-` resta esa opción a las que se heredan. Se añade dentro del `<Directory>` del sitio:

```apache
<Directory /var/www/solvia/public>
        Options -Indexes
        Require all granted
</Directory>
```

Con la regla activa, la misma petición responde:

```text
HTTP/1.1 403 Forbidden
```

Y el registro de errores del sitio deja constancia, con un mensaje que conviene saber leer:

```text
[autoindex:error] [client 127.0.0.1:52326] AH01276: Cannot serve directory /var/www/solvia/public/assets/: No matching DirectoryIndex (index.html,index.cgi,index.pl,index.php,index.xhtml,index.htm) found, and server-generated directory index forbidden by Options directive
```

Dice que no existe ninguno de los ficheros de `DirectoryIndex` y que el listado está prohibido por `Options`. Lo importante es lo que **no** ha cambiado: `http://solvia.test/assets/estilo.css` sigue respondiendo `200`. Quitar `Indexes` no oculta los ficheros, solo impide que se **listen**. Quien conozca la ruta exacta los sigue obteniendo, que es lo que se espera de los ficheros públicos de una web.

**Una regla de sintaxis de `Options`.** O todas las opciones llevan `+` o `-` delante, o ninguna. Mezclarlas produce un error de configuración:

```text
AH00526: Syntax error on line 7 of /etc/apache2/sites-enabled/solvia.conf:
Either all Options must start with + or -, or no Option may.
```

La diferencia importa: `Options -Indexes` (con signo) **modifica** las opciones heredadas, quitando solo `Indexes`; `Options None` o `Options FollowSymLinks` (sin signo) **sustituyen** todo lo heredado por exactamente lo que se escribe. Con signos, se sabe qué cambia; sin ellos, se pierde lo que venía de arriba sin darse cuenta.

### 4.4 `AllowOverride` y los ficheros `.htaccess`

Hay una segunda forma de configurar Apache, además de los ficheros de `/etc/apache2/`: un fichero llamado **`.htaccess`** colocado dentro de una carpeta del sitio. Apache lo lee **en cada petición** que llega a esa carpeta y aplica las directivas que contiene. Existe para que quien aloja una web en un servidor compartido, sin acceso a la configuración principal, pueda cambiar ajustes de su carpeta (redirecciones, `Options`, `php_flag`…).

Que Apache lea o no estos ficheros, y qué directivas les permite usar, lo decide **`AllowOverride`**:

| Valor | Qué permite en un `.htaccess` |
|---|---|
| `None` | Nada: Apache **ni siquiera lo busca** |
| `All` | Casi cualquier directiva |
| `Options`, `AuthConfig`, `FileInfo`, `Limit`, `Indexes` | Solo directivas de ese grupo (se pueden combinar) |

Se puede ver el efecto con una prueba. Con `AllowOverride All` (el valor del sitio de Solvia desde UD1) se coloca un `.htaccess` en `assets/` que prohíbe el listado:

```bash
echo "Options -Indexes" | sudo tee /var/www/solvia/public/assets/.htaccess
curl -s -o /dev/null -w "assets/: %{http_code}\n" http://solvia.test/assets/
```

```text
assets/: 403
```

Con `AllowOverride None`, el mismo fichero es ignorado:

```text
assets/: 200
```

Y es todavía más claro con una directiva de PHP: un `.htaccess` con `php_flag display_errors on` **cambia el comportamiento de PHP** en esa carpeta (apartado 4.5). Con `AllowOverride All`, cualquiera que pueda escribir un fichero `.htaccess` en `public/` (un subidor de ficheros defectuoso, un descuido de permisos) puede cambiar la configuración del servidor para esa carpeta. Con `None`, ese fichero no tiene efecto.

Hay un coste adicional: con `AllowOverride` distinto de `None`, Apache **busca un `.htaccess` en cada carpeta del camino de cada petición** (en `/var/www/`, en `/var/www/solvia/`, en `public/`…), por si existe. Es un acceso a disco extra por petición, que con `None` desaparece.

Solvia, tal como se entrega, **no contiene ningún `.htaccess`**. No lo necesita: su configuración está en el fichero del sitio. Por tanto, `AllowOverride None` es el valor adecuado. Dejar `All` no es un error grave (el sitio funciona igual, y por eso no se considera un fallo crítico en una auditoría), pero es un permiso concedido sin utilizarlo, y la regla de administración es **conceder solo lo que se usa**. La documentación de Apache recomienda directamente evitar `.htaccess` cuando se tiene acceso a la configuración principal.

### 4.5 PHP en modo producción

El módulo de PHP tiene su propia configuración, el fichero **`php.ini`**, independiente de la de Apache. Aquí es fácil equivocarse porque PHP se instala con **un `php.ini` distinto para cada forma de ejecutarlo**:

```bash
php --ini | head -n 3
php -r 'echo php_ini_loaded_file(), "\n";'
ls /etc/php/8.3/
```

```text
Configuration File (php.ini) Path: /etc/php/8.3/cli
Loaded Configuration File:         /etc/php/8.3/cli/php.ini
Scan for additional .ini files in: /etc/php/8.3/cli/conf.d
/etc/php/8.3/cli/php.ini
apache2  cli  mods-available
```

Lo que cambia con la orden `php` de la terminal es el de **`cli`** (línea de comandos). El que usa Apache con `mod_php` es el de la carpeta **`apache2`**: `/etc/php/8.3/apache2/php.ini`. Editar el equivocado no produce ningún efecto sobre el sitio, y `php -i` desde la terminal **no muestra** lo que ve Solvia. Para saber qué valor tiene PHP **dentro de Apache** hay que preguntárselo desde un script servido por Apache (como el de los ejemplos siguientes), no desde la terminal.

Los valores de ese fichero que importan en producción:

| Directiva | Valor en Ubuntu (Apache) | Qué hace y por qué importa |
|---|---|---|
| `display_errors` | `Off` | Si es `On`, los errores de PHP se escriben **en la página que ve el usuario**, con rutas del servidor y fragmentos de código |
| `log_errors` | `On` | Los errores se anotan en el registro de errores de Apache, que es donde los lee el administrador |
| `expose_php` | `Off` | Si es `On`, PHP añade una cabecera `X-Powered-By: PHP/8.3.6` a cada respuesta con su versión exacta |
| `memory_limit` | `128M` | Memoria máxima que puede usar un script |
| `max_execution_time` | `30` | Segundos máximos que puede tardar un script |
| `upload_max_filesize` / `post_max_size` | `2M` / `8M` | Tamaño máximo de un fichero subido / de un envío de formulario completo |
| `allow_url_include` | `Off` | Impide incluir código desde una dirección web |

Los valores de Ubuntu para Apache ya están en modo producción. Se comprueba con:

```bash
grep -nE '^(display_errors|log_errors|expose_php) ' /etc/php/8.3/apache2/php.ini
```

```text
410:expose_php = Off
518:display_errors = Off
537:log_errors = On
```

**Qué ve el visitante de un error, con `display_errors` apagado y encendido.** Un script de prueba con dos fallos (una variable que no existe y una función que no existe):

```bash
sudo mkdir -p /var/www/solvia/public/t
sudo tee /var/www/solvia/public/t/error.php <<'EOF'
<?php
echo "antes\n";
echo $noexiste['x'];
nofunciona();
EOF
curl -s -i http://solvia.test/t/error.php | head -n 1
```

```text
HTTP/1.1 500 Internal Server Error
```

El visitante recibe un `500` **sin ningún detalle**: ni qué ha fallado ni dónde. El administrador lo encuentra en el registro de errores del sitio:

```bash
sudo tail -n 3 /var/log/apache2/solvia_error.log
```

```text
[php:warn] [client 127.0.0.1:58096] PHP Warning:  Undefined variable $noexiste in /var/www/solvia/public/t/error.php on line 3
[php:warn] [client 127.0.0.1:58096] PHP Warning:  Trying to access array offset on null in /var/www/solvia/public/t/error.php on line 3
[php:error] [client 127.0.0.1:58096] PHP Fatal error:  Uncaught Error: Call to undefined function nofunciona() in /var/www/solvia/public/t/error.php:4
```

(Se han abreviado las marcas de hora y de proceso, que son distintas en tu máquina.) Si se pone `display_errors` en `On` solo para esa carpeta, con un `.htaccess` que contenga `php_flag display_errors on` y `AllowOverride All` en el sitio, la respuesta cambia:

```text
antes
<br />
<b>Warning</b>:  Undefined variable $noexiste in <b>/var/www/solvia/public/t/error.php</b> on line <b>3</b><br />
<br />
<b>Fatal error</b>:  Uncaught Error: Call to undefined function nofunciona() in /var/www/solvia/public/t/error.php:4
```

La página muestra la **ruta real del fichero en el servidor** y el tipo de fallo. Es información valiosa para quien quiera atacar: sabe cómo se organiza el disco, qué hay en el código y dónde falla. Por eso en producción los errores van al registro y el visitante solo ve un mensaje genérico. En desarrollo se hace al revés, y por eso PHP trae dos plantillas de partida, `php.ini-development` y `php.ini-production`.

**Cómo cambiar un valor sin tocar `php.ini`.** Editar `php.ini` directamente funciona, pero un paquete nuevo de PHP puede sobrescribirlo. La alternativa es un **fichero adicional** en `/etc/php/8.3/apache2/conf.d/`: PHP lee todos los `.ini` de esa carpeta, por orden alfabético, y el último gana. Por ejemplo, para fijar la zona horaria (que por defecto no está definida y PHP usa `UTC`):

```bash
sudo tee /etc/php/8.3/apache2/conf.d/99-solvia.ini <<'EOF'
; Ajustes de Solvia
date.timezone = Europe/Madrid
EOF
sudo tee /var/www/solvia/public/t/ini.php <<'EOF'
<?php
echo ini_get('date.timezone'), "\n", php_ini_loaded_file(), "\n";
EOF
curl -s http://solvia.test/t/ini.php
sudo systemctl reload apache2
curl -s http://solvia.test/t/ini.php
```

```text
UTC
/etc/php/8.3/apache2/php.ini
Europe/Madrid
/etc/php/8.3/apache2/php.ini
```

La primera respuesta es la de antes de recargar (`UTC`, el valor por defecto) y la segunda la de después (`Europe/Madrid`): PHP, igual que Apache, lee su configuración al arrancar o recargar, y un cambio no se aplica hasta entonces. El `99-` del nombre garantiza que se lee el último. El script `ini.php` pregunta a PHP desde dentro de Apache, que es lo que hay que hacer para saber qué ve la aplicación.

Cuando hayas terminado las pruebas, **borra los ficheros de prueba** (`sudo rm -r /var/www/solvia/public/t`). Un script de diagnóstico olvidado en una web pública es uno de los fallos más habituales; es la misma razón por la que `phpinfo()` se borra después de usarlo.

### 4.6 Páginas de error propias

Cuando una petición falla, Apache responde con una página de error genérica. Es la que devuelve Apache, con su configuración de fábrica, cuando se le pide un nombre que no es de ningún sitio (el `403` del apartado 3.8):

```bash
curl -s -H "Host: nada.test" http://localhost/
```

```text
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.58 (Ubuntu) Server at nada.test Port 80</address>
</body></html>
```

La última línea, la «firma» del servidor, es lo que controla `ServerSignature` y `ServerTokens` (apartados 1.5 y 1.8). Con `ServerSignature Off`, que es lo que ya tienes aplicado si has hecho el ajuste de esa unidad, desaparece: tu servidor mostrará la página sin la línea `<address>`. Además, un usuario que llega a una página de error del sistema, en inglés y sin el aspecto de la aplicación, no sabe a dónde volver. La directiva **`ErrorDocument`** sustituye la página de Apache por una propia para un código de error concreto. Se crea la página (dentro de `public/`, porque debe poder servirse) y se asocia al `404`:

```bash
sudo mkdir -p /var/www/solvia/public/errores
sudo tee /var/www/solvia/public/errores/404.html <<'EOF'
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>No encontrado</title>
</head>
<body>
<h1>Página no encontrada</h1>
<p>Vuelve a la <a href="/">portada de Solvia</a>.</p>
</body>
</html>
EOF
```

En el fichero del sitio, dentro del `<VirtualHost>`, se añade una línea:

```apache
        ErrorDocument 404 /errores/404.html
```

La sintaxis es `ErrorDocument código ruta`. Si la ruta empieza por `/`, es una página del propio sitio. Tras `configtest` y la recarga:

```bash
curl -s -i http://solvia.test/no-existe | tr -d '\r' | sed -n '1p;/^$/,$p'
```

```text
HTTP/1.1 404 Not Found

<!DOCTYPE html>
<html lang="es">
<head>
...
<h1>Página no encontrada</h1>
```

Lo importante es que el **código de estado sigue siendo `404`** (se ve en la primera línea): `ErrorDocument` cambia el cuerpo, no el código. (`tr -d '\r'` quita los retornos de carro que `curl` deja al final de las cabeceras, para que `sed` reconozca la línea vacía que separa cabeceras y cuerpo.) Los demás errores, como el `403` o el `500`, siguen con la página estándar mientras no se les asocie una propia.

### 4.7 Ejemplo resuelto: aplicar los ajustes al sitio de Solvia

Se parte del sitio de Solvia tal como está después del apartado 3.8, con `AllowOverride All` y sin `Options`. Se hace copia, se escribe el fichero con los ajustes y se compara.

**Paso 1. Copia de seguridad.**

```bash
sudo cp /etc/apache2/sites-available/solvia.conf ~/solvia.conf.antes-de-ajustes
```

**Paso 2. Fichero del sitio con los ajustes.** Si el tuyo tiene un `ServerName` o un `ServerAlias` distintos, respétalos:

```bash
sudo tee /etc/apache2/sites-available/solvia.conf <<'EOF'
<VirtualHost *:80>
        ServerName solvia.test
        ServerAlias localhost
        DocumentRoot /var/www/solvia/public

        <Directory /var/www/solvia/public>
                Options -Indexes
                AllowOverride None
                Require all granted
        </Directory>

        ErrorDocument 404 /errores/404.html
        ErrorLog ${APACHE_LOG_DIR}/solvia_error.log
        CustomLog ${APACHE_LOG_DIR}/solvia_access.log combined
</VirtualHost>
EOF
diff ~/solvia.conf.antes-de-ajustes /etc/apache2/sites-available/solvia.conf
```

```text
7c7,8
<                 AllowOverride All
---
>                 Options -Indexes
>                 AllowOverride None
10a12
>         ErrorDocument 404 /errores/404.html
```

(Los números de línea dependen de tu fichero.) El `diff` es la prueba de que solo han cambiado tres cosas: se añade `Options -Indexes`, `AllowOverride All` pasa a `None` (las líneas `7c7,8` dicen que la línea 7 se sustituye por dos) y se añade `ErrorDocument` tras la línea 10 (`10a12`). Es lo que querías cambiar y nada más.

**Paso 3. Permisos de la configuración (si no están ya).**

```bash
sudo chown root:www-data /var/www/solvia/config/config.php
sudo chmod 640 /var/www/solvia/config/config.php
```

**Paso 4. Comprobar la sintaxis, recargar y probar cada ajuste.**

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -s -o /dev/null -w "portada:        %{http_code}\n" http://solvia.test/
curl -s -o /dev/null -w "listado assets: %{http_code}\n" http://solvia.test/assets/
curl -s -o /dev/null -w "fichero assets: %{http_code}\n" http://solvia.test/assets/estilo.css
curl -s -o /dev/null -w "config.php:     %{http_code}\n" http://solvia.test/config/config.php
curl -s -o /dev/null -w "inexistente:    %{http_code}\n" http://solvia.test/no-existe
curl -s http://solvia.test/no-existe | grep -o "<h1>.*</h1>"
```

```text
Syntax OK
portada:        200
listado assets: 403
fichero assets: 200
config.php:     404
inexistente:    404
<h1>Página no encontrada</h1>
```

(En la prueba de arriba `assets/estilo.css` es el nombre de un fichero que existe en tu carpeta `assets/`; usa uno que exista.) Cada línea comprueba un ajuste: la portada sigue funcionando, el listado de la carpeta está prohibido, un fichero concreto se sigue sirviendo, el fichero de configuración no es accesible desde la web (`404` porque, al estar fuera de `public/`, la ruta no existe para Apache) y la página que no existe muestra la página propia. Una comprobación por ajuste es la forma de dejar claro que **cada cambio hace lo que se pretendía**.

**Paso 5. Limpia lo de prueba.**

```bash
sudo rm -r /var/www/solvia/public/t
ls /var/www/solvia/public
```

### 4.8 Caso práctico sobre Solvia: la lista de comprobación de implantación

Los ajustes de este apartado se resumen en una tabla que sirve como lista de comprobación para cualquier aplicación PHP que se implante:

| Ajuste | Qué se comprueba | Cómo | Resultado esperado en Solvia |
|---|---|---|---|
| Carpeta servida | Solo `public/` es accesible | `curl` a `/config/config.php`, `/db/schema.sql`, `/README.md` | `404` los tres |
| Ficheros ocultos | Ningún `.env`, `.git` ni copia de seguridad en `public/` | `ls -la /var/www/solvia/public` | Sin ficheros extraños |
| Permisos de la configuración | Solo `root` y `www-data` pueden leer `config.php` | `ls -l config/config.php` | `-rw-r----- root www-data` |
| Listado de carpetas | `Indexes` desactivado | `curl` a `/assets/` | `403` |
| `.htaccess` | `AllowOverride None` si la aplicación no usa ninguno | `find public -name .htaccess` | Ninguno; `None` en el sitio |
| Errores de PHP | `display_errors` apagado, `log_errors` encendido | `grep` en `php.ini` y script de prueba | `500` sin detalle; el error, en el registro |
| Versión de PHP expuesta | `expose_php` apagado | `curl -sI` y buscar `X-Powered-By` | Ausente |
| Páginas de error | Una página propia en el `404` | `curl` a una ruta inexistente | Página propia con código `404` |

La búsqueda de `.htaccess` se hace con:

```bash
find /var/www/solvia/public -name .htaccess
```

Sin ningún resultado, ninguna carpeta del sitio necesita `AllowOverride`.

### 4.9 ⚖️ Alternativa y criterio

**Configurar en el fichero del sitio o con `.htaccess`.** Las dos formas permiten, por ejemplo, desactivar el listado de una carpeta. Se prefiere el **fichero del sitio** (`<Directory>` en `sites-available/`) por tres razones: es más rápido (Apache lo lee una vez al arrancar, no en cada petición), es más seguro (solo lo puede cambiar quien tiene permisos de administración, no quien pueda escribir en la carpeta de la web) y es más fácil de auditar (todo está en un lugar y `apache2ctl -S` y `configtest` lo comprueban). El `.htaccess` solo es la opción razonable cuando **no se tiene acceso** a la configuración del servidor, como ocurre en un alojamiento compartido: es el caso para el que se inventó.

**Restringir con `chmod` o con `Require`.** Impedir que se lea un fichero puede hacerse con permisos del sistema (`chmod 640`) o con reglas de Apache (`Require all denied` en un `<Files>`). Son capas distintas y se **complementan**: los permisos protegen frente a cualquier usuario del sistema; `Require` frente a cualquier visitante web. El fichero de configuración se protege con las dos: está fuera de `public/` (no tiene dirección web) y con permisos `640` (solo lo lee `www-data`).

**Editar `php.ini` o añadir un fichero en `conf.d/`.** Editar el `php.ini` es lo más directo, pero un paquete actualizado puede reemplazarlo, y un cambio mezclado con el resto es difícil de distinguir de lo que venía de serie. El fichero adicional (`99-solvia.ini`) deja constancia de **qué se ha cambiado y por qué** en un fichero de pocas líneas, que se puede copiar a otra máquina. Se prefiere para ajustes propios.

**Una página de error propia o la estándar.** Una página propia mejora lo que ve el usuario y no revela la versión del servidor, pero es una pieza más que mantener y que puede fallar. Para errores que el usuario ve a menudo, como el `404`, compensa; para el `500`, lo importante es **no revelar nada**, y la estándar de Apache con `ServerSignature Off` ya lo cumple.

**`mod_php` o PHP-FPM.** Esta unidad usa `mod_php`, que ejecuta PHP dentro de Apache. La alternativa más habitual en producción es **PHP-FPM**, un servicio independiente al que Apache le pasa las peticiones. Tiene su propio usuario por aplicación (aísla un sitio de otro) y permite el MPM `event` (apartado 2.14). Queda fuera de esta unidad, pero es donde acaban la mayoría de despliegues PHP en producción.

### 4.10 ⚠️ Errores comunes

**Editar el `php.ini` equivocado.** El de `cli` no afecta a Apache. Se detecta porque el cambio «no hace nada» en la web aunque `php -i` lo muestre. El de Apache está en `/etc/php/8.3/apache2/`.

**Cambiar `php.ini` y no recargar Apache.** PHP lee su configuración al arrancar o recargar. Se arregla con `sudo systemctl reload apache2`.

**Olvidar `ErrorDocument` fuera de `public/`.** Si la página de error está en una carpeta no servida, o el nombre no existe, el visitante recibe una página de error del propio `ErrorDocument` fallido. Se prueba con `curl` a una ruta inexistente.

**Mezclar `Options` con y sin signo.** `Options -Indexes FollowSymLinks` da `AH00526`. O todas con signo, o ninguna.

**Dejar `Options` sin signo cuando se quiere restar.** `Options FollowSymLinks` quita `Indexes` pero también todo lo demás que se heredara. Con signos (`-Indexes`) solo se resta lo que se escribe.

**Quitar `Indexes` y creer que los ficheros ya están ocultos.** `-Indexes` impide el listado, no el acceso: `assets/estilo.css` sigue sirviéndose. Lo que no deba servirse no debe estar en `public/`.

**Dar permiso `777` «para que funcione».** Ante un `403` o un `Permission denied`, la tentación es `chmod -R 777`. Funciona, y deja el código y la configuración escribibles por cualquier usuario del sistema. La solución correcta es leer el registro (`AH00035` dice qué carpeta falta por atravesar) y dar el permiso mínimo.

**Dejar scripts de prueba en la web.** Un `info.php`, un `phpinfo()` o una carpeta `t/` quedan como puerta de información. Se borran al terminar.

**Dar por bueno un ajuste sin comprobarlo desde fuera.** `configtest` solo dice que la sintaxis es correcta. Que `-Indexes` esté escrito no prueba que `/assets/` dé `403`: eso solo lo prueba `curl`.

### 4.11 💡 Nota técnica: cuando `Options` y un manejador convierten una carpeta en un ejecutor de código

Las opciones de una carpeta interactúan con otras directivas. Un caso clásico: la directiva `AddHandler cgi-script .php` asocia una extensión a un programa que Apache **ejecuta**, y `Options +ExecCGI` permite esa ejecución en la carpeta. Si ambas coinciden en una carpeta donde los usuarios pueden **subir ficheros** (`uploads/`, `adjuntos/`), cualquier fichero subido con esa extensión se ejecuta en el servidor con los permisos de `www-data`. Es una de las formas más graves de comprometer un servidor, y es difícil de ver porque cada directiva, por separado, parece inofensiva. Un administrador revisa una configuración **buscando combinaciones**, no líneas aisladas.

Dos ajustes más que se encuentran en la práctica y no se desarrollan aquí. **`open_basedir`** en `php.ini` limita las carpetas del disco a las que puede acceder PHP, de modo que un fallo de la aplicación no permite leer ficheros ajenos al sitio. **`disable_functions`** desactiva funciones de PHP peligrosas (las que ejecutan órdenes del sistema), que una aplicación como Solvia no necesita; en Ubuntu viene vacía.

También hay que saber que, en HTTP, el contenido de una carpeta puede llegar al visitante por vías que no pasan por `Options` ni `Require` de la carpeta: un `Alias` que publica otra ruta del disco, un enlace simbólico con `FollowSymLinks`, una redirección. Un `DocumentRoot` bien elegido no protege lo que se publique con un `Alias` mal puesto, y por eso se audita **todo el sitio**, no solo la carpeta principal.

### 4.12 Comprueba lo que has entendido

**1. Un compañero sirve Solvia con `DocumentRoot /var/www/solvia`. La web funciona y nadie se ha quejado. ¿Qué le dices y cómo se lo demuestras?**

Que funciona, pero publica toda la carpeta del proyecto: `db/schema.sql`, `README.md` y todo el código, y que `config.php` solo se protege porque PHP lo ejecuta en vez de enviarlo. Se demuestra pidiendo `http://sitio/db/schema.sql` con `curl` (responde `200` y descarga el esquema) y, en una máquina de pruebas, desactivando el módulo de PHP: `config.php` se envía como texto con la contraseña de la base de datos. El arreglo es poner el `DocumentRoot` en `public/`.

**2. Después de `chmod 777 /var/www/solvia/config/config.php` el sitio vuelve a funcionar. ¿Por qué es mala solución y cuál es la correcta?**

Porque `777` da permiso de lectura y escritura a cualquier usuario del sistema, y el fichero contiene la contraseña de la base de datos. La correcta es descubrir por qué fallaba (con el registro de errores y `ls -l`/`namei -l`) y dar el mínimo: propietario `root`, grupo `www-data`, modo `640`.

**3. `AllowOverride None` en el sitio y un `.htaccess` con `Options -Indexes` en `assets/`. ¿Se prohíbe el listado? ¿Y con `AllowOverride All`?**

Con `None`, no: Apache ni siquiera busca el `.htaccess`, y `assets/` se sigue listando (`200`). Con `All` sí lo lee y la carpeta responde `403`. Por eso se pone `None` si la aplicación no usa `.htaccess`.

**4. Cambias `display_errors` en `/etc/php/8.3/cli/php.ini` y la web sigue mostrando errores. ¿Qué ha pasado?**

Que has editado el `php.ini` de la línea de comandos, y Apache usa el de `/etc/php/8.3/apache2/`. Además, hay que recargar Apache para que lea el cambio. Se comprueba con un script `ini_get('display_errors')` servido por Apache.

**5. Escribes `Options -Indexes FollowSymLinks` y `configtest` falla con `AH00526`. ¿Qué regla has roto y cómo se escribe bien según lo que quieras conseguir?**

La regla de que todas las opciones llevan `+` o `-`, o ninguna. Si solo quieres quitar el listado conservando lo heredado, `Options -Indexes`. Si quieres fijar exactamente las opciones, sin nada heredado, `Options FollowSymLinks` (sin signo).

---

## 5. Autenticación y control de acceso

**CE 2.d** — Se han configurado los mecanismos de autenticación y control de acceso del servidor.

Un servidor web que sirve información interna (los informes de uso de un servicio, el estado del servidor, un panel de administración) no puede dejarla abierta a quien tenga la dirección. Este apartado explica cómo se decide **quién puede pedir qué** a un Apache: primero con reglas que no necesitan saber quién es el visitante (su dirección de red), después con un mecanismo que le pide un nombre y una contraseña (autenticación Basic) y, por último, combinando ambos. Se aplica al sitio `informes.solvia.test` creado en el apartado 3, que a partir de aquí exige usuario y contraseña.

Se apoya en el ciclo de trabajo del apartado 1.7, en los contextos de directivas del apartado 1.4 (`<Directory>`, `<Location>`) y en la regla de `Require all granted` que ya aparece en el fichero de cada sitio (apartados 3.5 y 4.1). Termina con una limitación que conviene entender antes de seguir: la autenticación Basic **solo es segura si la conexión va cifrada**, y por eso el apartado siguiente de la unidad (certificados y HTTPS) no es un añadido opcional, sino la segunda mitad de este.

### 5.1 Tres preguntas distintas: quién eres, qué puedes hacer, desde dónde vienes

Bajo el nombre genérico de «control de acceso» se mezclan tres cuestiones que conviene separar:

- **Identificación y autenticación**: ¿quién eres, y cómo lo demuestras? El servidor recibe un nombre de usuario y una prueba de que quien escribe es de verdad ese usuario (una contraseña).
- **Autorización**: una vez sabido quién eres, ¿tienes permiso para pedir **esto**? El usuario `maria` puede entrar a los informes; el usuario `luis` puede que no.
- **Control de acceso por origen**: ¿desde qué lugar llega la petición? Es una regla que no pregunta quién eres, sino de dónde vienes: la dirección IP del cliente. «Solo se puede ver desde la red de la oficina».

Apache resuelve las tres con una sola directiva, **`Require`**, que pertenece al módulo `authz_core`, y con módulos distintos que aportan cada tipo de regla. Este es el reparto:

| Tipo | Ejemplo de regla | Módulo | Qué comprueba |
|---|---|---|---|
| Sin condición | `Require all granted` / `Require all denied` | `authz_core` | Nada: deja pasar a todos o a nadie |
| Por origen | `Require ip 192.0.2.0/24`, `Require local` | `authz_host` | La dirección IP del cliente |
| Por usuario | `Require valid-user`, `Require user maria` | `authz_user` | El usuario que se ha autenticado |
| Por grupo | `Require group admins` | `authz_groupfile` | Si el usuario pertenece a un grupo |

La palabra clave es que `Require` **autoriza**; para tener un usuario al que autorizar, primero hay que **autenticarlo**, y eso se configura con otras directivas (`AuthType`, `AuthName`, `AuthUserFile`) que se ven en el apartado 5.4. Si no hay autenticación, las reglas por usuario no pueden cumplirse nunca.

### 5.2 Control de acceso sin identidad: `Require` y el origen de la petición

Se empieza por lo más simple, porque ya se ha usado dos veces. `Require all granted` (acceso para todos) está en el fichero de Solvia y en el de informes; `Require all denied` (acceso para nadie) en el sitio por defecto del apartado 3.8. Entre ambos extremos, `Require` admite condiciones sobre la dirección del cliente:

```apache
Require ip 192.0.2.0/24
```

La expresión `192.0.2.0/24` es una **red** en notación CIDR: la dirección 192.0.2.0 y los 24 primeros bits fijos, es decir, las 256 direcciones de 192.0.2.0 a 192.0.2.255. También se admite una dirección sola (`Require ip 127.0.0.1`), varias separadas por espacios, y direcciones IPv6. Otras condiciones útiles de este grupo son `Require local` (la propia máquina) y `Require host dominio` (el nombre del cliente, que obliga a una consulta de DNS inversa y por eso se evita).

Se comprueba con una regla que **no** incluye al cliente, que es la propia máquina (`127.0.0.1`). En el fichero del sitio `informes.conf`:

```apache
<Directory /var/www/informes>
        Require ip 192.0.2.0/24
</Directory>
```

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://informes.solvia.test/
sudo tail -n 1 /var/log/apache2/informes_error.log
```

```text
403
[authz_core:error] [client 127.0.0.1:34882] AH01630: client denied by server configuration: /var/www/informes/
```

Aparece el código **`403 Forbidden`** (prohibido) y en el registro de errores el mensaje `AH01630: client denied by server configuration`: «el cliente ha sido denegado por la configuración». Es el mensaje que hay que reconocer cuando una regla de origen o `Require all denied` bloquea una petición. Este bloqueo no pregunta nada al cliente: no pide contraseña, solo le cierra la puerta.

Dos cosas más. Primero, se pueden **combinar** condiciones. Por defecto, varias líneas `Require` seguidas se interpretan como «**o**»: basta con cumplir una. Para exigir que se cumplan todas se agrupan en un bloque `<RequireAll>`, y para dejarlo explícito se puede usar `<RequireAny>` (cualquiera). Segundo, la lógica es la de una puerta: **solo pasa quien cumple las condiciones**, y si no hay ninguna condición que se cumpla, no pasa nadie.

### 5.3 Cómo funciona la autenticación Basic

Para saber quién llama, el servidor tiene que pedirle un nombre y una prueba. El protocolo HTTP define cómo se hace con una conversación en tres pasos, que se aprecia mejor mirándola:

```bash
curl -s -i http://informes.solvia.test/ | tr -d '\r' | head -n 6
```

```text
HTTP/1.1 401 Unauthorized
Date: Mon, 09 Nov 2026 09:01:01 GMT
Server: Apache
WWW-Authenticate: Basic realm="Informes de Solvia"
Content-Length: 381
Content-Type: text/html; charset=iso-8859-1
```

(La línea `Server: Apache` es la que queda después de aplicar `ServerTokens Prod` en el apartado 1.8. Si todavía no lo has hecho, verás `Apache/2.4.58 (Ubuntu)`; no afecta a lo que se explica aquí.)

1. **El cliente pide la página** sin identificarse.
2. **El servidor responde `401 Unauthorized`** y añade la cabecera **`WWW-Authenticate: Basic realm="..."`**. Significa: «para entrar aquí necesito que te identifiques con el método *Basic*, y la zona protegida se llama `Informes de Solvia`». El *realm* («dominio» o «ámbito») es el nombre de esa zona; el navegador lo muestra en la ventana que pide usuario y contraseña.
3. **El cliente repite la petición con una cabecera `Authorization`** que lleva el usuario y la contraseña. Si son correctos, el servidor responde `200`; si no, otra vez `401`.

La cabecera del paso 3 es lo más importante de entender. Con `curl -u usuario:contraseña` (o con la ventana del navegador) se envía, y `curl -v` deja verla:

```bash
curl -sv -u 'maria:ClaveDeP1' -o /dev/null http://informes.solvia.test/ 2>&1 | grep -E '^> (GET|Host|Authorization)|^< HTTP'
```

```text
> GET / HTTP/1.1
> Host: informes.solvia.test
> Authorization: Basic bWFyaWE6Q2xhdmVEZVAx
< HTTP/1.1 200 OK
```

El valor `bWFyaWE6Q2xhdmVEZVAx` no parece una contraseña, pero es la contraseña **sin cifrar**. Basic construye esa cadena uniendo el usuario y la contraseña con dos puntos (`maria:ClaveDeP1`) y aplicando una codificación llamada **Base64**, que convierte bytes en letras y números. Base64 **no es cifrado**: no usa ninguna clave y cualquiera la deshace en un segundo.

```bash
echo 'bWFyaWE6Q2xhdmVEZVAx' | base64 -d; echo
```

```text
maria:ClaveDeP1
```

(`base64 -d` decodifica; el `echo` final solo añade el salto de línea.) Esto tiene una consecuencia que decide cómo se usa Basic. Cualquiera que pueda **ver el tráfico** entre el navegador y el servidor (otro equipo de la misma red, un punto de acceso Wi-Fi comprometido, un administrador de red) lee el usuario y la contraseña **en cada petición**, no solo al iniciar sesión. Se comprueba capturando el tráfico de la propia máquina con `tcpdump`, la herramienta que ya se usó en el apartado de arranque para ver el formulario de Solvia viajar en claro:

```bash
sudo tcpdump -l -i lo -A -s0 'tcp port 80' | grep -a --line-buffered "Authorization"
```

En otra terminal, se hace una petición autenticada:

```bash
curl -s -o /dev/null -u 'maria:ClaveDeP1' http://informes.solvia.test/
```

```text
Authorization: Basic bWFyaWE6Q2xhdmVEZVAx
```

Lo que `tcpdump` captura de la red es la cabecera completa, con la contraseña a la vista tras un `base64 -d`. Las opciones son las mismas de aquel apartado: `-i lo` escucha en la interfaz de bucle local (la que usa la propia máquina para hablar consigo misma; para tráfico que llega desde otro equipo es `-i any`), `-A` muestra el contenido como texto, `-s0` captura el paquete entero, y `'tcp port 80'` filtra el puerto de HTTP. `grep -a` trata la entrada como texto aunque contenga bytes raros y `--line-buffered` evita que se retrasen las líneas. `sudo` es necesario porque capturar tráfico exige permisos de administrador.

La conclusión es una regla que se repetirá al llegar a HTTPS: **la autenticación Basic solo es aceptable sobre una conexión cifrada**. Sin cifrado protege del visitante curioso, pero no de quien mira la red. Con TLS (apartado 7) la cabecera `Authorization` viaja dentro del canal cifrado y `tcpdump` ya no la ve.

### 5.4 Cómo se configura: usuarios, contraseñas y las directivas de autenticación

Una autenticación Basic en Apache tiene tres piezas: un **fichero de usuarios y contraseñas**, un **bloque de configuración** que le dice a Apache que lo use, y la **regla `Require`** que decide quién entra.

**El fichero de usuarios.** Se crea y se mantiene con la herramienta `htpasswd`, que viene en el paquete `apache2-utils` (se instala como dependencia de `apache2`). Cada línea del fichero es `usuario:contraseña-cifrada`.

```bash
sudo htpasswd -B -c /etc/apache2/.htpasswd-informes maria
sudo htpasswd -B /etc/apache2/.htpasswd-informes luis
```

`htpasswd` pregunta la contraseña dos veces (no se ve mientras se escribe) y responde `Adding password for user maria`. Las opciones:

- **`-c`** (*create*) **crea el fichero**. Solo se usa para el primer usuario. En los siguientes se omite, porque `-c` sobre un fichero que ya existe **lo vacía y empieza de cero**: se perderían los demás usuarios.
- **`-B`** guarda la contraseña con **bcrypt**, un algoritmo de cifrado de contraseñas diseñado para ser lento y resistir los ataques de fuerza bruta. Sin esta opción, la herramienta usa un formato antiguo basado en MD5 (`$apr1$`), que se calcula mucho más deprisa y por tanto es más fácil de atacar.
- El **nombre del fichero** `.htpasswd-informes` empieza por `.ht`, así que la regla `<FilesMatch "^\.ht">` de `apache2.conf` (apartado 4.1) ya impediría servirlo aunque estuviera dentro de una carpeta web. Aun así, **debe guardarse fuera del `DocumentRoot`**, y aquí está en `/etc/apache2/`.

El contenido del fichero muestra qué se guarda:

```bash
sudo cat /etc/apache2/.htpasswd-informes
```

```text
maria:$2y$05$ts7Uumnl6A8.2yrhtDaVLOKP13Nob14KVxB.nrCZbPe0g8.Iy1ZgS
luis:$2y$05$CZRkRlex3beH5g5w.uqET.eTV.InaTn3sSFFKRT32iNv8UawbtEqW
```

(Los valores serán distintos en tu máquina, porque bcrypt añade un valor aleatorio, la *sal*.) Cada línea tiene el usuario, dos puntos y un *hash*: `$2y$` indica bcrypt y `05` es su coste de cálculo. Un *hash* no es la contraseña ni puede deshacerse: cuando alguien se autentica, Apache calcula el hash de la contraseña recibida y lo compara con el guardado. Por eso el fichero no revela las contraseñas, aunque sí conviene protegerlo, porque con él se pueden intentar adivinar.

Los permisos del fichero siguen la regla del apartado 4.2: lo escribe `root` y lo lee `www-data`, el usuario con el que Apache atiende las peticiones.

```bash
sudo chown root:www-data /etc/apache2/.htpasswd-informes
sudo chmod 640 /etc/apache2/.htpasswd-informes
ls -l /etc/apache2/.htpasswd-informes
```

```text
-rw-r----- 1 root www-data 133 Nov  9 10:03 /etc/apache2/.htpasswd-informes
```

**El bloque de configuración.** En el fichero del sitio, dentro del `<Directory>` que se quiere proteger:

```apache
<Directory /var/www/informes>
        AuthType Basic
        AuthName "Informes de Solvia"
        AuthUserFile /etc/apache2/.htpasswd-informes
        Require valid-user
</Directory>
```

Cada línea cumple un papel:

- **`AuthType Basic`** elige el método de autenticación. Apache incluye `Basic` y `Digest` (apartado 5.8). Lo implementa el módulo `auth_basic`.
- **`AuthName "Informes de Solvia"`** es el nombre de la zona (el *realm* del apartado anterior). Es el texto que verá el usuario en la ventana de acceso.
- **`AuthUserFile`** indica el fichero de usuarios. Lo lee el módulo `authn_file`. Debe ser una ruta absoluta.
- **`Require valid-user`** es la regla de **autorización**: basta con ser cualquier usuario del fichero que haya acertado con su contraseña. Para limitarlo a algunos se escribe `Require user maria` (o varios nombres separados por espacios).

Los tres módulos que intervienen (`auth_basic`, `authn_file` y `authz_user`) vienen activados de serie en Ubuntu. Se comprueba con `apache2ctl -M | grep -E 'auth'`.

### 5.5 Ejemplo resuelto: proteger `informes` con usuario y contraseña

Se parte del sitio `informes` del apartado 3.6, que responde `200` a cualquiera.

**Paso 1. Comprueba el punto de partida.**

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://informes.solvia.test/
```

```text
200
```

**Paso 2. Crea el fichero de usuarios** con dos usuarios y asegura sus permisos (usa contraseñas tuyas, de al menos doce caracteres):

```bash
sudo htpasswd -B -c /etc/apache2/.htpasswd-informes maria
sudo htpasswd -B /etc/apache2/.htpasswd-informes luis
sudo chown root:www-data /etc/apache2/.htpasswd-informes
sudo chmod 640 /etc/apache2/.htpasswd-informes
```

Existe también `-b`, que permite escribir la contraseña en la propia orden (`htpasswd -b fichero usuario contraseña`) y por tanto es cómodo para automatizar. **No se usa** en un servidor real, porque la contraseña queda escrita en el historial de órdenes (`~/.bash_history`) y es visible en la lista de procesos mientras la orden se ejecuta. Sin `-b`, `htpasswd` la pide por teclado sin mostrarla.

**Paso 3. Edita el fichero del sitio**, copia de seguridad antes:

```bash
sudo cp /etc/apache2/sites-available/informes.conf ~/informes.conf.antes-de-auth
sudo tee /etc/apache2/sites-available/informes.conf <<'EOF'
<VirtualHost *:80>
        ServerName informes.solvia.test
        DocumentRoot /var/www/informes

        <Directory /var/www/informes>
                AuthType Basic
                AuthName "Informes de Solvia"
                AuthUserFile /etc/apache2/.htpasswd-informes
                Require valid-user
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/informes_error.log
        CustomLog ${APACHE_LOG_DIR}/informes_access.log combined
</VirtualHost>
EOF
sudo apache2ctl configtest
sudo systemctl reload apache2
```

```text
Syntax OK
```

**Paso 4. Comprueba los cuatro casos.** Una protección no está probada hasta que se han probado **los casos que deben fallar** además del que debe funcionar:

```bash
curl -s -o /dev/null -w "sin usuario:         %{http_code}\n" http://informes.solvia.test/
curl -s -o /dev/null -w "usuario y clave bien: %{http_code}\n" -u 'maria:TuClave' http://informes.solvia.test/
curl -s -o /dev/null -w "clave mala:          %{http_code}\n" -u 'maria:otra' http://informes.solvia.test/
curl -s -o /dev/null -w "usuario inexistente: %{http_code}\n" -u 'pepe:x' http://informes.solvia.test/
```

```text
sin usuario:         401
usuario y clave bien: 200
clave mala:          401
usuario inexistente: 401
```

(Sustituye `TuClave` por la contraseña que escribiste; las comillas simples evitan que la terminal interprete caracteres especiales.) Las tres peticiones fallidas devuelven el mismo `401`: el visitante no puede deducir, por la respuesta, si el usuario existe o si solo era mala la contraseña. El administrador sí lo distingue, en el registro de errores:

```bash
sudo tail -n 2 /var/log/apache2/informes_error.log
```

```text
[auth_basic:error] [client 127.0.0.1:60188] AH01617: user maria: authentication failure for "/": Password Mismatch
[auth_basic:error] [client 127.0.0.1:60192] AH01618: user pepe not found: /
```

`AH01617` es «contraseña incorrecta» (`Password Mismatch`) y `AH01618` es «usuario no encontrado» (`user ... not found`). Son los dos mensajes que hay que reconocer ante un fallo de acceso. Y el registro de accesos refleja el usuario en la tercera columna (el guion `-` cuando no hay):

```bash
sudo tail -n 4 /var/log/apache2/informes_access.log
```

```text
127.0.0.1 - - [09/Nov/2026:10:03:58 +0100] "GET / HTTP/1.1" 401 581 "-" "curl/8.5.0"
127.0.0.1 - maria [09/Nov/2026:10:03:58 +0100] "GET / HTTP/1.1" 200 229 "-" "curl/8.5.0"
127.0.0.1 - maria [09/Nov/2026:10:03:58 +0100] "GET / HTTP/1.1" 401 581 "-" "curl/8.5.0"
```

Que el nombre del usuario quede en el registro es útil para saber quién ha consultado qué, y se aprovechará en el apartado de logs. Si la prueba se hace desde el navegador, la ventana de acceso muestra el texto `Informes de Solvia` (el `AuthName`), y el navegador **recuerda las credenciales mientras la ventana siga abierta**: para probar de nuevo desde cero hay que cerrarla (o abrir una ventana privada).

### 5.6 Combinar autenticación y origen, y limitar a ciertos usuarios

Las reglas se pueden combinar. Dos ejemplos que se encuentran con frecuencia.

**Solo ciertos usuarios.** `Require valid-user` deja pasar a todos los del fichero. Para dejar pasar solo a algunos, se nombra a quien entra:

```apache
Require user maria
```

Con esa línea, `maria` accede y `luis`, aunque su contraseña sea correcta, recibe `401`, y el registro lo explica con otro mensaje distinto, `AH01631: user luis: authorization failure`. Fíjate en la diferencia entre `AH01617`/`AH01618` (**falló la autenticación**: no se pudo comprobar quién es) y `AH01631` (**falló la autorización**: se sabe quién es, y no tiene permiso). Para muchos usuarios con los mismos permisos se definen grupos con un fichero (`AuthGroupFile`) y se escribe `Require group nombre`.

**Usuario y además origen.** Para exigir las dos cosas (que el usuario se identifique **y** que venga de ciertas redes) se usa `<RequireAll>`:

```apache
<Directory /var/www/informes>
        AuthType Basic
        AuthName "Informes de Solvia"
        AuthUserFile /etc/apache2/.htpasswd-informes
        <RequireAll>
                Require valid-user
                Require ip 127.0.0.1 192.0.2.0/24
        </RequireAll>
</Directory>
```

Una petición sin usuario recibe `401` y una autenticada desde una dirección permitida recibe `200`. Si se escribieran las dos líneas `Require` sueltas, sin `<RequireAll>`, bastaría con cumplir **una**, y cualquier usuario válido entraría desde cualquier red: el efecto contrario al buscado. Esta confusión entre «o» (el valor por defecto) y «y» (con `<RequireAll>`) es la causa de muchos accesos abiertos por error.

### 5.7 Caso práctico sobre Solvia: `/server-status` detrás de usuario y contraseña

Solvia tiene que dejar de ser el único sitio con información interesante. El módulo `status` (apartado 2.11) publica `/server-status` con el estado del servidor, y viene limitado a la propia máquina con `Require local`. El sitio `informes` es el lugar natural para exponerlo a quien administra, **si está protegido**. Se añade al fichero de `informes.conf`, dentro del `<VirtualHost>`, un bloque `<Location>` con su propia autenticación:

```apache
        <Location "/server-status">
                SetHandler server-status
                AuthType Basic
                AuthName "Informes de Solvia"
                AuthUserFile /etc/apache2/.htpasswd-informes
                Require valid-user
        </Location>
```

`SetHandler server-status` le dice a Apache que esa ruta no es un fichero sino la página generada por el módulo `status`. `<Location>` es como `<Directory>`, pero se refiere a una **dirección web** (`/server-status`) y no a una carpeta del disco. Las reglas de un `<Location>` del sitio sustituyen a las del `<Location /server-status>` global de `mods-enabled/status.conf` (que decía `Require local`). Tras `configtest` y la recarga:

```bash
curl -s -o /dev/null -w "sin usuario: %{http_code}\n" http://informes.solvia.test/server-status
curl -s -o /dev/null -w "con usuario: %{http_code}\n" -u 'maria:TuClave' http://informes.solvia.test/server-status
curl -s -u 'maria:TuClave' "http://informes.solvia.test/server-status?auto" | head -n 4
```

```text
sin usuario: 401
con usuario: 200
informes.solvia.test
ServerVersion: Apache/2.4.58 (Ubuntu)
ServerMPM: prefork
Server Built: 2026-10-08T19:58:01
```

(`?auto` pide la versión para máquinas, una línea por dato.) Fíjate en la segunda línea de datos: aunque `ServerTokens Prod` ya oculta la versión en las cabeceras y en las páginas de error, la página de estado **sigue mostrando la versión completa de Apache y del sistema**. Es una razón más para que esta página solo la vea quien administra. El resultado cumple lo que se quería: la información sensible sigue disponible para quien administra, y ya no lo está para quien tenga la dirección. Lo que no cambia es que `/server-status` sigue respondiendo en los **demás** sitios para quien llega desde la propia máquina, porque la regla global (`Require local`) sigue vigente allí. Es una exposición menor (solo desde dentro), pero conviene saberlo.

### 5.8 ⚖️ Alternativa y criterio

**Basic o Digest.** Apache ofrece otro método, `AuthType Digest` (módulo `auth_digest`), diseñado para que la contraseña no viaje en claro: el cliente envía un resumen calculado con ella. Parece resolver el problema del apartado 5.3, y por eso se enseñaba en el pasado. La propia documentación del módulo lo descarta: dice que Digest **no ofrece una ventaja de seguridad significativa** sobre Basic, que el almacenamiento de contraseñas en el servidor es **mucho menos seguro** que con Basic (el contenido de un fichero `htdigest` robado sirve directamente para autenticarse), que un atacante que se interponga en la comunicación puede **forzar al navegador a bajar a Basic**, y que el algoritmo de resumen es **demasiado rápido**, de modo que quien escuche la red puede averiguar la contraseña por fuerza bruta con una tarjeta gráfica actual. Su conclusión es que **usar Basic y cifrar toda la conexión con `mod_ssl` es una alternativa mucho mejor**. Con TLS, Basic es suficiente, y sin TLS ninguno de los dos es aceptable; por eso la decisión es **Basic + TLS**, y no Digest. (La página está en las referencias del final de la unidad.)

**Fichero de usuarios, grupos o un directorio.** Un fichero `htpasswd` es adecuado para **pocos usuarios** (un equipo de administración de tres o cuatro personas): es simple y no depende de nada más. Cuando son decenas, cuando hay que dar y quitar altas continuamente o cuando la misma cuenta debe valer para varios servicios, se pasa a un **servicio de directorio** como LDAP, donde las cuentas se gestionan en un solo sitio. Apache puede consultarlo con módulos como `authnz_ldap`. Es lo que llega en una unidad posterior del módulo, y es la razón de que `htpasswd` no se plantee como solución definitiva sino como la adecuada **para este tamaño**.

**Autenticar en el servidor o en la aplicación.** Basic protege **una zona del servidor** sin que la aplicación sepa nada. Es perfecto para un panel interno, y no para una aplicación de usuarios: no hay un formulario de acceso con diseño propio, no hay cierre de sesión (el navegador recuerda las credenciales hasta cerrar), no hay recuperación de contraseña. Para una aplicación con usuarios reales, la autenticación se hace en la propia aplicación, con sesiones y formularios; el servidor se limita a proteger lo que no es de la aplicación (informes, paneles de estado).

**Qué hash usar.** `-B` (bcrypt) es la opción recomendada hoy. El formato por defecto de `htpasswd` es MD5 adaptado (`$apr1$`): sigue siendo compatible, pero se calcula rápido y por eso se rompe más deprisa si el fichero se filtra. Existen otros (`-5` y `-2` para SHA-256 y SHA-512, `-d` para `crypt`); el criterio es **bcrypt salvo que haya un motivo de compatibilidad**.

### 5.9 ⚠️ Errores comunes

**Configurar la autenticación pero no poner `Require`: el sitio queda abierto.** Es el error más grave del apartado y no da ningún aviso. Con este bloque:

```apache
<Directory /var/www/informes>
        AuthType Basic
        AuthName "Informes de Solvia"
        AuthUserFile /etc/apache2/.htpasswd-informes
</Directory>
```

`configtest` responde `Syntax OK` y la petición sin usuario devuelve **`200`**, con la página a la vista. La razón es la herencia: el bloque `<Directory /var/www/>` de `apache2.conf` (apartado 4.3) ya dice `Require all granted`, y sin un `Require` propio el bloque lo hereda. `AuthType` y compañía solo **preparan** la autenticación; quien **exige** algo es `Require`. La comprobación de los cuatro casos (apartado 5.5) lo detecta; mirar solo que «pide contraseña en el navegador» puede no hacerlo, porque con una sesión abierta no se ve.

**Usar `-c` en el segundo usuario.** `htpasswd -c fichero tercero` vacía el fichero y lo deja con un solo usuario. `-c` es solo para crear.

**Escribir la contraseña en la orden (`-b`).** Queda en el historial. Si ocurrió, se borra la línea del historial y se cambia la contraseña.

**Dejar el fichero de contraseñas dentro del `DocumentRoot`.** La regla de `.ht` lo protege solo si el nombre empieza por `.ht`, y un nombre como `usuarios.txt` en `public/` se descarga. Debe estar siempre fuera de la zona servida.

**Ruta relativa o inexistente en `AuthUserFile`.** Apache responde `500 Internal Server Error` y el registro dice `AH01620: Could not open password file: /etc/apache2/noexiste`. Se corrige la ruta; no es un problema del usuario.

**Confundir `401` y `403`.** `401` significa «no sé quién eres» y viene con `WWW-Authenticate`: invita a identificarse. `403` significa «sé lo que pides y no te lo doy»: no sirve de nada volver a intentarlo con otra contraseña. Un `403` en una zona protegida por usuario suele ser una regla `Require ip` o `Require all denied` que se cruza con la autenticación.

**Poner dos líneas `Require` y esperar que se cumplan las dos.** Por defecto basta con una (apartado 5.6). Para exigir ambas, `<RequireAll>`.

**Dar por segura la autenticación Basic sobre HTTP.** Funciona y protege de un visitante casual, pero cada petición envía las credenciales legibles (apartado 5.3). Hasta que el sitio sea solo HTTPS, la protección está incompleta.

**No cerrar el navegador al probar.** Tras una autenticación correcta el navegador reenvía las credenciales sin preguntar, así que una prueba de «sin usuario» puede dar `200` y parecer un fallo. Se prueba con `curl` o con una ventana privada.

### 5.10 💡 Nota técnica: el orden de las secciones y la fuerza bruta

**Directory, Files y Location no se aplican en cualquier orden.** Apache combina las secciones de configuración en un orden fijo: primero los `<Directory>` (y los `.htaccess`), luego los `<Files>` y por último los `<Location>`. Cuando dos secciones dan reglas de `Require` contradictorias para la misma petición, **gana la que se aplica después**, que es el `<Location>`. Por eso la regla `Require valid-user` del `<Location "/server-status">` del apartado 5.7 sustituye al `Require local` global de `status.conf`. Dentro de una misma sección, y para la configuración de autorización, las reglas de una sección posterior **reemplazan** a las anteriores salvo que se pida lo contrario con la directiva `AuthMerging`.

**Basic no limita los intentos.** Nada impide probar mil contraseñas por minuto contra un usuario: Apache responde `401` cada vez y lo anota en el registro (`AH01617`). Defenderse de esa fuerza bruta es tarea de otra capa: herramientas como **fail2ban**, que leen el registro y bloquean la dirección que acumula fallos, o un cortafuegos con límite de tasa. La mejor defensa del lado de la configuración son contraseñas largas, y por eso `bcrypt` (lento a propósito) hace caro cada intento si se filtra el fichero.

**La contraseña viaja en cada petición.** A diferencia de un formulario de acceso con sesión (donde la contraseña se envía una vez y después se usa una cookie), en Basic el navegador reenvía `Authorization` en **todas** las peticiones a esa zona. Más ocasiones de que alguien la capture si la conexión no está cifrada, y razón de más para el TLS.

### 5.11 Comprueba lo que has entendido

**1. Has añadido `AuthType Basic`, `AuthName` y `AuthUserFile` al `<Directory>` de `informes`, pero no `Require`. ¿Qué responde el servidor a una petición sin usuario y por qué?**

Responde `200`: la protección no funciona. El bloque `<Directory /var/www/>` de `apache2.conf` tiene `Require all granted` y el de `informes` lo hereda. Las directivas `Auth*` solo preparan el mecanismo; la que exige es `Require`. Se arregla añadiendo `Require valid-user`, y se comprueba con `curl` sin usuario (debe dar `401`).

**2. En el registro de errores aparece `AH01618: user pepe not found` y, más abajo, `AH01631: user luis: authorization failure`. ¿Qué diferencia hay entre uno y otro?**

`AH01618` es un fallo de **autenticación**: el usuario `pepe` no está en el fichero, así que no se ha podido comprobar quién es. `AH01631` es un fallo de **autorización**: `luis` sí se identificó correctamente, pero la regla `Require` (por ejemplo `Require user maria`) no le permite entrar.

**3. Capturas con `tcpdump` una petición autenticada por HTTP y ves `Authorization: Basic bWFyaWE6Q2xhdmVEZVAx`. ¿Qué información tienes y por qué tiene eso consecuencias para el diseño?**

Tienes el usuario y la contraseña, porque Base64 es una codificación y no un cifrado: `base64 -d` devuelve `maria:ClaveDeP1`. Como Basic reenvía esa cabecera en cada petición, quien vea el tráfico obtiene las credenciales sin esfuerzo. Por eso Basic solo es aceptable dentro de una conexión cifrada con TLS (HTTPS).

**4. Quieres que a `informes` solo se entre con usuario válido **y** desde la red `192.0.2.0/24`. Un compañero escribe las dos líneas `Require valid-user` y `Require ip 192.0.2.0/24` seguidas. ¿Funciona como quiere?**

No. Varias líneas `Require` sueltas se combinan con «o»: basta con cumplir una, y cualquier usuario válido entraría desde cualquier red, o cualquier cliente de esa red sin usuario. Para exigir las dos se agrupan en `<RequireAll> … </RequireAll>`.

**5. Creas el segundo usuario con `sudo htpasswd -B -c /etc/apache2/.htpasswd-informes luis` y `maria` deja de poder entrar. ¿Qué ha pasado?**

La opción `-c` crea el fichero de nuevo y lo deja con un único usuario: ha borrado a `maria`. `-c` solo se usa con el primer usuario; los siguientes se añaden sin ella. Se restaura volviendo a añadir a `maria` (sin `-c`).

---

## 6. Certificados digitales

**CE 2.e** — Se han obtenido e instalado certificados digitales.

El apartado anterior termina con un problema que no tiene solución dentro de HTTP: la contraseña de la autenticación Basic viaja legible por la red, igual que el formulario de Solvia. La solución es cifrar la conexión, y eso se hace con **TLS**, el protocolo que convierte HTTP en HTTPS. Pero antes de cifrar nada, el servidor necesita algo que la conexión HTTP nunca ha tenido: **una forma de demostrar quién es**. Ese algo es el certificado digital.

Este apartado explica qué es un certificado, qué contiene, por qué un navegador se fía de unos y desconfía de otros, y cómo se obtiene e instala uno. Se hacen dos prácticas completas: un certificado **autofirmado** (el más sencillo, y el que provoca el aviso del navegador) y un certificado **firmado por una autoridad propia**, la «CA del aula», que reproduce el proceso real. Al final se explica cómo funciona Let's Encrypt, que es lo que se usa en producción, y por qué no puede usarse en una máquina virtual de prácticas. El apartado 7 usa lo que aquí se prepara para activar HTTPS en Solvia.

No hace falta ninguna instalación nueva: la herramienta es **OpenSSL**, que viene con Ubuntu. Se usa la versión 3.0 (`openssl version`).

### 6.1 Qué problema resuelven los certificados

Un canal cifrado no basta. Si el navegador cifra la conexión con *alguien*, pero no sabe *con quién*, el cifrado puede estar protegiendo la comunicación con un atacante que se hace pasar por el servidor. Para que HTTPS sea útil, TLS ofrece tres propiedades a la vez:

- **Confidencialidad**: lo que viaja está cifrado, y quien observa la red no puede leerlo.
- **Integridad**: lo que llega es lo que se envió; si alguien lo modifica por el camino, se detecta.
- **Autenticación del servidor**: el cliente comprueba que está hablando con el servidor correcto, y no con otro que se hace pasar por él.

La tercera es la que más se olvida y la que justifica el certificado. Un certificado es la respuesta a la pregunta «¿cómo sé que `solvia.test` es de verdad `solvia.test`?»: un documento que une un **nombre** (o varios) con una **clave pública**, y que está **firmado** por una entidad en la que el cliente confía. Si el servidor es capaz de demostrar que posee la clave privada correspondiente a la clave pública del certificado, el cliente sabe que habla con quien dice ser.

### 6.2 Claves, firmas y el hecho de que sean un par

Todo lo que sigue se apoya en la **criptografía de clave pública** (o asimétrica). Se genera un **par de claves** matemáticamente unidas:

- La **clave privada** se queda en el servidor y **no se comparte con nadie, nunca**.
- La **clave pública** se entrega a quien la pida: va dentro del certificado.

Lo que se cifra o firma con una clave solo se puede comprobar o descifrar con la otra. De ahí salen dos usos: **cifrar** para un destinatario (con su clave pública, solo él lo descifra con su privada) y **firmar** (con la clave privada se produce una firma que cualquiera puede verificar con la pública, y que demuestra quién la hizo y que el contenido no ha cambiado). Los certificados usan la **firma**: la entidad que garantiza el certificado lo firma con su clave privada, y cualquiera con su clave pública puede comprobar que esa firma es auténtica.

En la práctica, durante una conexión HTTPS el servidor envía su certificado, el cliente lo valida (apartado 6.4) y después los dos acuerdan una clave de sesión con la que cifran el resto de la conversación (con un algoritmo simétrico, mucho más rápido). El certificado interviene en el primer paso; el cifrado del tráfico lo hace la clave de sesión.

Los dos tipos de clave más usados son **RSA** (con 2048 bits como mínimo habitual; los 4096 son más lentos y rara vez necesarios) y **ECDSA** (curvas elípticas; claves mucho más cortas con una seguridad equivalente). En esta unidad se usa RSA de 2048 bits, que es compatible con todo.

### 6.3 Qué contiene un certificado

El formato es **X.509**. Un certificado es un fichero de texto (codificado en **PEM**: un bloque entre las líneas `-----BEGIN CERTIFICATE-----` y `-----END CERTIFICATE-----`) con una estructura que `openssl` sabe leer. Este es el contenido del certificado que se creará en el apartado 6.6, abreviado:

```bash
openssl x509 -in solvia.crt -noout -text
```

```text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            [...]
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: O = IES Rio Arba, CN = CA del aula
        Validity
            Not Before: Nov  9 09:06:34 2026 GMT
            Not After : Nov  9 09:06:34 2027 GMT
        Subject: CN = solvia.test
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                [...]
        X509v3 extensions:
            X509v3 Subject Alternative Name: 
                DNS:solvia.test, DNS:informes.solvia.test
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        [...]
```

`openssl x509` trabaja con certificados; `-in` indica el fichero; `-noout` evita que repita el certificado codificado; `-text` lo muestra completo. Los campos que importan:

| Campo | Qué dice |
|---|---|
| **Subject** (sujeto) | A quién pertenece el certificado. Hoy el nombre aquí es informativo: lo que se comprueba es el SAN |
| **Issuer** (emisor) | Quién lo ha firmado. Si es igual que el sujeto, el certificado está **autofirmado** |
| **Validity** (validez) | Entre qué fechas es válido. Fuera de ese intervalo, el cliente lo rechaza |
| **Subject Alternative Name** (SAN) | La lista de **nombres para los que vale**. Es el campo que el navegador compara con el nombre que ha escrito el usuario |
| **Subject Public Key Info** | La clave pública del titular |
| **Basic Constraints** | Si el titular es una **autoridad** que puede firmar otros certificados (`CA:TRUE`) o un certificado final (`CA:FALSE`) |
| **Key Usage / Extended Key Usage** | Para qué puede usarse: aquí, autenticar a un servidor web |
| **Signature** | La firma del emisor sobre todo lo anterior |

**El SAN es lo que cuenta, no el nombre del sujeto.** Antes se identificaba al servidor con el *Common Name* (`CN = solvia.test`, dentro del sujeto). Los navegadores actuales **ignoran el CN** y exigen que el nombre esté en el SAN. Un certificado con `CN = solvia.test` y sin SAN falla en el navegador aunque el nombre parezca correcto. Por eso en los ejemplos siguientes el SAN se declara siempre.

Se puede extraer solo lo esencial, que es lo que hay que ser capaz de responder cuando se pregunta por un certificado: quién es, quién lo ha emitido, hasta cuándo vale y para qué nombres.

```bash
openssl x509 -in solvia.crt -noout -subject -issuer -dates -ext subjectAltName
```

```text
subject=CN = solvia.test
issuer=O = IES Rio Arba, CN = CA del aula
notBefore=Nov  9 09:06:34 2026 GMT
notAfter=Nov  9 09:06:34 2027 GMT
X509v3 Subject Alternative Name: 
    DNS:solvia.test, DNS:informes.solvia.test
```

### 6.4 Por qué un navegador se fía de un certificado

El certificado lo firma una **autoridad de certificación** (*Certificate Authority*, **CA**). Una CA es una organización, o simplemente una clave privada guardada con cuidado, cuya firma sirve de garantía. El cliente no puede confiar en todas las firmas del mundo, así que lleva una lista de CA en las que confía por defecto, el **almacén de confianza** (*trust store*): lo mantienen el sistema operativo (Windows, Ubuntu), y el navegador, a veces con el suyo propio (Firefox). Contiene los certificados de unas cuantas decenas de CA, llamadas **raíz**.

La confianza se transmite por una **cadena**. Una CA raíz rara vez firma directamente certificados de servidores; firma el certificado de una **CA intermedia**, y esa firma los de los servidores. El servidor entrega su certificado y los intermedios; el cliente solo necesita tener la raíz:

```plaintext
CA raíz              (en el almacén de confianza del cliente)
  └─ firma →  CA intermedia     (la entrega el servidor)
                └─ firma →  Certificado del servidor  (el que se presenta en la conexión)
```

Cuando el navegador recibe un certificado, hace estas comprobaciones, y **si falla cualquiera, muestra un aviso**:

1. **La firma**: ¿está firmado por alguien, y la cadena de firmas llega a una CA raíz que el almacén contiene?
2. **El nombre**: ¿el nombre escrito en la barra de direcciones está en el SAN?
3. **Las fechas**: ¿está dentro de su periodo de validez?
4. **La revocación**: ¿ha sido retirado antes de caducar (por ejemplo, porque se filtró la clave)? Se consulta con mecanismos específicos (listas de revocación y servicios en línea).
5. **El uso**: ¿vale para autenticar un servidor web?
6. Y, durante la conexión, que el servidor **demuestra poseer la clave privada** correspondiente.

El efecto práctico es que **el navegador no se fía del certificado en sí, sino de quien lo firmó**. Un certificado perfectamente formado pero firmado por una CA que el cliente no conoce provoca un aviso, y es justamente lo que ocurre con un certificado autofirmado. Esta idea es la que organiza el resto del apartado.

### 6.5 Ejemplo resuelto: un certificado autofirmado

Un certificado **autofirmado** (*self-signed*) es uno que lo firma su propio titular con su propia clave: el emisor y el sujeto son el mismo. Es la forma más rápida de tener un certificado y no necesita nadie más. Su límite es que nadie lo tiene en su almacén de confianza, así que ningún cliente se fiará de él por defecto.

**Paso 1. Crea la clave privada y el certificado en un solo comando.** Se trabaja en una carpeta de pruebas:

```bash
mkdir -p ~/pki && cd ~/pki
openssl req -x509 -newkey rsa:2048 -nodes -keyout solvia-auto.key -out solvia-auto.crt -days 365 -subj "/CN=solvia.test" -addext "subjectAltName=DNS:solvia.test,DNS:informes.solvia.test"
```

OpenSSL imprime unos puntos y signos `+` y `*` mientras busca números primos para la clave, y termina sin más mensajes. Lo que hace cada opción:

- **`req`** es el subcomando para peticiones de certificado y, con `-x509`, para crear uno directamente.
- **`-x509`** produce un certificado autofirmado en vez de una petición (el apartado 6.6 usa la otra forma).
- **`-newkey rsa:2048`** genera una clave nueva RSA de 2048 bits.
- **`-nodes`** (*no DES*) guarda la clave **sin contraseña**. Es necesario para que Apache pueda arrancar solo, sin que alguien escriba la contraseña de la clave. El precio es que quien copie el fichero tiene la clave, y por eso se protege con permisos (apartado 6.7). En OpenSSL 3 se escribe también `-noenc`.
- **`-keyout`** y **`-out`** son los ficheros de salida: la clave privada y el certificado.
- **`-days 365`** es la validez, un año.
- **`-subj "/CN=solvia.test"`** fija el sujeto. Sin esta opción, OpenSSL preguntaría interactivamente país, organización, etc.
- **`-addext "subjectAltName=DNS:..."`** añade la extensión SAN con los dos nombres del servidor. Es la parte esencial (apartado 6.3).

**Paso 2. Comprueba los ficheros y sus permisos.**

```bash
ls -l
```

```text
total 8
-rw-r--r-- 1 root root 1184 Nov  9 10:06 solvia-auto.crt
-rw------- 1 root root 1704 Nov  9 10:06 solvia-auto.key
```

La clave privada se ha creado con permisos `600` (solo el propietario puede leerla): OpenSSL la protege por defecto. El certificado, que es público, es legible por todos.

**Paso 3. Léelo.**

```bash
openssl x509 -in solvia-auto.crt -noout -subject -issuer -dates -ext subjectAltName
```

```text
subject=CN = solvia.test
issuer=CN = solvia.test
notBefore=Nov  9 09:06:09 2026 GMT
notAfter=Nov  9 09:06:09 2027 GMT
X509v3 Subject Alternative Name: 
    DNS:solvia.test, DNS:informes.solvia.test
```

El `subject` y el `issuer` son idénticos: es lo que significa «autofirmado». (Las horas están en GMT; si tu máquina tiene otra zona, lo verás distinto en el registro de Apache.)

**Paso 4. Comprueba qué opina el sistema de él.** `openssl verify` ejecuta la comprobación de la cadena (apartado 6.4, punto 1) contra el almacén de confianza del sistema:

```bash
openssl verify solvia-auto.crt
```

```text
CN = solvia.test
error 18 at 0 depth lookup: self-signed certificate
error solvia-auto.crt: verification failed
```

El error 18, `self-signed certificate`, dice exactamente lo esperado: el certificado está firmado por sí mismo y nadie en el almacén de confianza lo respalda. Un navegador que recibiera este certificado mostraría una pantalla de advertencia (Chrome la identifica como `NET::ERR_CERT_AUTHORITY_INVALID` y Firefox como `SEC_ERROR_UNKNOWN_ISSUER`).

**Cómo debe interpretarse ese aviso.** El aviso no significa que la conexión no esté cifrada: con este certificado el tráfico sí iría cifrado. Significa que el cliente **no puede comprobar con quién habla**. El navegador está haciendo bien su trabajo: es el sistema funcionando, no un fallo. En un servidor de pruebas que controlas tú, puedes aceptar la excepción sabiendo lo que haces. En un servidor público, ese aviso es la señal de que algo va mal, y quien se acostumbra a pulsar «continuar» pierde la protección frente a un atacante que se interponga con otro certificado.

### 6.6 Ejemplo resuelto: una CA propia y un certificado firmado por ella

El autofirmado resuelve el cifrado pero no la confianza. Lo que sí resuelve la confianza es reproducir en pequeño lo que hace Internet: tener una **CA**, hacer que los clientes confíen en ella **una sola vez**, y que firme todos los certificados que haga falta. En el aula, la CA es la del profesorado (la **«CA del aula»**); aquí se crea una propia para poder practicar el proceso entero.

El proceso real de obtener un certificado tiene tres pasos, y son los mismos con una CA de juguete que con una pública:

1. El titular genera su **clave privada** y una **petición de firma de certificado** (*Certificate Signing Request*, **CSR**): un fichero que contiene la clave pública y los nombres que quiere, firmado con la clave privada. Nunca contiene la clave privada.
2. La CA recibe el CSR, **verifica** que el solicitante puede tener esos nombres y lo **firma**, produciendo el certificado.
3. El titular **instala** el certificado junto a su clave privada en el servidor.

**Paso 1. Crea la CA (clave y certificado raíz).** Se hace una sola vez y la clave de la CA es **lo más delicado de todo el sistema**: quien la tenga puede emitir certificados de lo que quiera que los clientes que confían en la CA aceptarán.

```bash
cd ~/pki
openssl req -x509 -newkey rsa:4096 -nodes -keyout ca-aula.key -out ca-aula.crt -days 1825 -subj "/O=IES Rio Arba/CN=CA del aula" -addext "basicConstraints=critical,CA:TRUE" -addext "keyUsage=critical,keyCertSign,cRLSign"
openssl x509 -in ca-aula.crt -noout -subject -issuer -dates -ext basicConstraints,keyUsage
```

```text
subject=O = IES Rio Arba, CN = CA del aula
issuer=O = IES Rio Arba, CN = CA del aula
notBefore=Nov  9 09:06:20 2026 GMT
notAfter=Nov  8 09:06:20 2031 GMT
X509v3 Basic Constraints: critical
    CA:TRUE
X509v3 Key Usage: critical
    Certificate Sign, CRL Sign
```

Es un certificado autofirmado (emisor y sujeto iguales), como el anterior, pero con dos diferencias que definen lo que es una CA: `CA:TRUE` en las restricciones básicas (puede firmar otros certificados) y el uso `Certificate Sign` (para qué sirve su clave). Dura cinco años, `-days 1825`, porque una CA vive mucho más que los certificados que firma.

**Paso 2. El titular genera su clave y su CSR.** Esto lo haría cada alumno en su máquina; el CSR es lo único que sale de ella:

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout solvia.key -out solvia.csr -subj "/CN=solvia.test" -addext "subjectAltName=DNS:solvia.test,DNS:informes.solvia.test"
openssl req -in solvia.csr -noout -verify -subject
```

```text
Certificate request self-signature verify OK
subject=CN = solvia.test
```

`-new` crea una petición (y no un certificado, que sería `-x509`). La orden `verify` comprueba que la petición está bien formada y que su autofirma es válida, es decir, que quien la envió tiene la clave privada.

**Paso 3. La CA firma el CSR.** Aquí la CA **decide qué certificado emite**, y no se limita a copiar lo que le piden: esas decisiones van en un pequeño fichero de extensiones. Fíjate en que el nombre se declara también aquí, porque la CA no tiene por qué fiarse de lo que el solicitante pidió.

```bash
cat > solvia.ext <<'EOF'
subjectAltName = DNS:solvia.test, DNS:informes.solvia.test
basicConstraints = critical, CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
EOF
openssl x509 -req -in solvia.csr -CA ca-aula.crt -CAkey ca-aula.key -CAcreateserial -out solvia.crt -days 365 -sha256 -extfile solvia.ext
```

```text
Certificate request self-signature ok
subject=CN = solvia.test
```

Cada opción: **`x509 -req`** firma una petición; **`-CA`** y **`-CAkey`** son el certificado y la clave de la autoridad que firma; **`-CAcreateserial`** crea un fichero de números de serie (`ca-aula.srl`) para que cada certificado emitido tenga uno distinto; **`-days 365`** es la validez del certificado emitido; **`-sha256`** el algoritmo de la firma; **`-extfile`** las extensiones que la CA pone. En el fichero, `CA:FALSE` declara que este certificado **no puede firmar otros**, y `serverAuth` que sirve para identificar a un servidor.

**Paso 4. Comprueba el resultado.**

```bash
openssl x509 -in solvia.crt -noout -subject -issuer -dates -ext subjectAltName
openssl verify -CAfile ca-aula.crt solvia.crt
openssl verify solvia.crt
```

```text
subject=CN = solvia.test
issuer=O = IES Rio Arba, CN = CA del aula
notBefore=Nov  9 09:06:34 2026 GMT
notAfter=Nov  9 09:06:34 2027 GMT
X509v3 Subject Alternative Name: 
    DNS:solvia.test, DNS:informes.solvia.test
solvia.crt: OK
error 20 at 0 depth lookup: unable to get local issuer certificate
error solvia.crt: verification failed
```

El emisor ahora es la CA y no el propio titular. `verify -CAfile ca-aula.crt` da **OK** porque se le ha dicho explícitamente en qué CA confiar. Sin esa indicación, `verify solvia.crt` falla con el error 20, `unable to get local issuer certificate`: «no encuentro en mi almacén el certificado de quien lo firmó». Es un error distinto del 18 de antes (que decía que estaba autofirmado) y se lee igual de bien: el sistema sabe quién lo firmó, pero **no confía en esa CA**.

**Paso 5. Haz que el sistema confíe en la CA, una sola vez.** En Ubuntu, los certificados de CA adicionales se copian a `/usr/local/share/ca-certificates/` (con extensión `.crt`) y se ejecuta `update-ca-certificates`, que reconstruye el almacén del sistema:

```bash
sudo cp ca-aula.crt /usr/local/share/ca-certificates/ca-aula.crt
sudo update-ca-certificates
openssl verify solvia.crt
```

```text
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
solvia.crt: OK
```

Ahora `solvia.crt` se verifica con `OK` sin más indicaciones. Esta es la idea clave del apartado: **no se ha vuelto a tocar el certificado del servidor; se ha cambiado en quién confía el cliente**. A partir de aquí, cualquier certificado firmado por la CA del aula —el de `solvia.test`, el de `informes.solvia.test`, el de cualquier otro— se acepta sin avisos. Y es exactamente lo que hace el navegador con las CA públicas: las que están en su almacén. Para dejar la máquina como estaba, se borra el fichero y se reconstruye el almacén:

```bash
sudo rm /usr/local/share/ca-certificates/ca-aula.crt
sudo update-ca-certificates --fresh
```

Para que el **navegador del equipo anfitrión** (el Windows en el que corre VirtualBox) confíe en la CA, hay que importar el certificado de la CA en su almacén: en Windows, abrir `ca-aula.crt` e instalarlo en «Entidades de certificación raíz de confianza»; Firefox usa un almacén propio, con su propio procedimiento de importación en los ajustes de certificados. Solo debe importarse **el certificado de la CA** (`ca-aula.crt`), **nunca su clave privada** (`ca-aula.key`), y conviene quitarlo cuando termine la práctica, porque lo que se importa es una autoridad de confianza total en ese equipo.

### 6.7 Instalar el certificado y proteger la clave

Un certificado instalado en un servidor son **dos ficheros**: el certificado (público) y la clave privada (secreta). Ubuntu tiene carpetas del sistema para certificados (`/etc/ssl/certs/`) y claves (`/etc/ssl/private/`), pero conviene **no** dejar ahí el certificado del servidor, por una razón que ya conoces del apartado 6.6: `/etc/ssl/certs/` es el **almacén de confianza del sistema**, y `update-ca-certificates` reconstruye sus índices con todo lo que hay dentro. Un certificado de servidor puesto ahí puede acabar siendo aceptado por las herramientas del propio equipo (por ejemplo `curl`) sin que ninguna CA lo respalde, y se perdería justo la comprobación que se quiere practicar. Por eso se usa una carpeta propia de Apache, `/etc/apache2/ssl/`, con esta regla de permisos:

| Qué | Permisos | Por qué |
|---|---|---|
| Carpeta `/etc/apache2/ssl/` | `750`, propietario `root` | Solo `root` entra a leerla |
| Certificado (`solvia.crt`) | `644` | Es público: se envía a cada cliente que conecta |
| Clave privada (`solvia.key`) | `600`, propietario `root` | Es el secreto del servidor: solo `root` la lee |

Ubuntu normalmente trae ya un certificado de ejemplo creado en la instalación (viene con el paquete `ssl-cert`, que `apache2` recomienda), el llamado *snakeoil* (`/etc/ssl/certs/ssl-cert-snakeoil.pem` y su clave en `/etc/ssl/private/`), que es autofirmado y para el nombre de la máquina. Lo usa el sitio de ejemplo `default-ssl.conf`. No sirve para nada real, y se sustituye por uno propio.

```bash
sudo mkdir -p /etc/apache2/ssl
sudo chmod 750 /etc/apache2/ssl
sudo cp ~/pki/solvia.crt /etc/apache2/ssl/solvia.crt
sudo cp ~/pki/solvia.key /etc/apache2/ssl/solvia.key
sudo chown root:root /etc/apache2/ssl/solvia.crt /etc/apache2/ssl/solvia.key
sudo chmod 644 /etc/apache2/ssl/solvia.crt
sudo chmod 600 /etc/apache2/ssl/solvia.key
sudo ls -l /etc/apache2/ssl/
```

```text
total 8
-rw-r--r-- 1 root root 1606 Nov  9 10:15 solvia.crt
-rw------- 1 root root 1704 Nov  9 10:15 solvia.key
```

Los comandos nuevos: `mkdir -p` crea la carpeta (y no falla si ya existe); `chown root:root` fija propietario y grupo; `chmod` cambia los permisos con los números del apartado 4.2 (`644` = el propietario lee y escribe, el resto solo lee; `600` = solo el propietario lee y escribe). La clave queda legible únicamente por `root`, que es quien arranca Apache y la carga en ese momento: los procesos de `www-data` que atienden las peticiones **no necesitan** leerla. Es la misma regla de mínimo privilegio del apartado 4.2, aplicada a la pieza más sensible.

Antes de usarlos, se comprueba que el certificado y la clave son un par (si no lo son, Apache no arrancará con HTTPS). La forma de comprobarlo es comparar el *módulo*, un número que comparten la clave y el certificado:

```bash
openssl x509 -noout -modulus -in /etc/apache2/ssl/solvia.crt | md5sum
sudo openssl rsa -noout -modulus -in /etc/apache2/ssl/solvia.key | md5sum
```

```text
4c2d3591fb5330a3752cb008fe55fcc1  -
4c2d3591fb5330a3752cb008fe55fcc1  -
```

(`md5sum` solo se usa aquí como forma de comparar dos números largos de un vistazo; no es una operación de seguridad.) Las dos sumas son iguales: el certificado y la clave son pareja. Si hubieras mezclado ficheros de pruebas distintas, serían distintas. En el apartado 7 se configuran estos dos ficheros en el sitio.

### 6.8 Caso práctico sobre Solvia: ¿qué certificado tiene tu servidor y por qué el navegador se fía o no?

Con lo anterior hay tres certificados posibles para `solvia.test`, y cada uno responde de forma distinta a las comprobaciones del apartado 6.4:

| | Autofirmado | Firmado por la CA del aula | Firmado por una CA pública (Let's Encrypt) |
|---|---|---|---|
| **Emisor** | El propio servidor | La CA del aula | Let's Encrypt (con una cadena hasta una raíz pública) |
| **¿El cliente se fía?** | No: aviso en el navegador | Sí, **si** tiene instalada la CA del aula | Sí, sin hacer nada |
| **Nombres que puede tener** | Los que quieras | Los que quieras (`.test` incluido) | Solo dominios públicos reales que controles |
| **Esfuerzo** | Un comando | Montar la CA y firmar | Un dominio público y un cliente de ACME |
| **Para qué sirve** | Una prueba rápida | Entornos internos y prácticas | Cualquier servicio público |

Cuando se te pregunte por el certificado de tu servidor, tienes que poder contestar con cuatro datos, que salen de una sola orden y una verificación:

```bash
openssl x509 -in /etc/apache2/ssl/solvia.crt -noout -subject -issuer -dates -ext subjectAltName
openssl verify /etc/apache2/ssl/solvia.crt
```

- **Quién emite**: el campo `issuer`.
- **Para qué nombres vale**: el SAN.
- **Hasta cuándo**: `notAfter`.
- **Por qué el navegador se fía o no**: porque la CA del `issuer` está o no en su almacén. Si lo está, `verify` da `OK`; si no, error 18 (autofirmado) o 20 (CA desconocida).

### 6.9 Let's Encrypt y el protocolo ACME

En producción nadie crea la CA a mano ni compra el certificado a mano. Se usa una CA pública gratuita y automática: **Let's Encrypt**. Funciona con un protocolo estándar llamado **ACME** (*Automatic Certificate Management Environment*). La idea: en vez de que una persona demuestre ante la CA que controla un dominio, lo hace un programa instalado en el servidor (el más usado es **Certbot**), de forma automática.

El problema que ACME debe resolver es el del paso 2 del proceso del apartado 6.6, la verificación previa a la firma: ¿cómo sabe la CA que quien pide un certificado para `ejemplo.org` es de verdad quien controla `ejemplo.org`? Lo comprueba con un **desafío** (*challenge*): le pide al solicitante que haga algo que solo puede hacer quien controla el dominio. Los dos más comunes:

- **HTTP-01**: la CA da al cliente un código que el cliente debe publicar en una dirección concreta de su web (`http://ejemplo.org/.well-known/acme-challenge/...`). La CA, desde Internet, la visita y comprueba que el código está allí.
- **DNS-01**: el cliente publica el código en un registro DNS del dominio. Se usa para certificados que cubren muchos subdominios (comodín) o cuando el servidor no es accesible desde fuera.

Eso explica por qué **no se usa Let's Encrypt en una máquina virtual de prácticas**: HTTP-01 exige un dominio de Internet real y que la CA llegue al puerto 80 del servidor desde fuera, y una máquina virtual detrás de la red privada de VirtualBox no cumple ninguna de las dos cosas. Tampoco hay dominios `.test` en Internet: no existen para ninguna CA pública. Este apartado explica el mecanismo; la práctica se hace con la CA del aula, que reproduce el mismo flujo (clave → CSR → firma de una CA → instalación) con la diferencia de quién firma.

En un servidor público, el flujo con Certbot y Apache es una sola orden que hace todo (pide el certificado, lo instala y configura el sitio) y deja un temporizador del sistema para renovarlo:

```bash
sudo apt install certbot python3-certbot-apache
sudo certbot --apache
```

(Se muestra solo como referencia: necesita un dominio público y no puede ejecutarse en la máquina virtual de prácticas.)

**La vida de un certificado es cada vez más corta.** Let's Encrypt emite hoy por defecto certificados de **90 días**, y su documentación oficial anuncia que reducirá el máximo a **45 días** para febrero de 2028; también ofrece certificados opcionales de 6 días. Por su parte, el sector que fija las normas de las CA públicas (el *CA/Browser Forum*) ha ido bajando el máximo permitido: de 398 días a 200 días en 2026, con previsión de 100 días a partir del 15 de marzo de 2027 y de **47 días a partir del 15 de marzo de 2029**, según el calendario que recogen las fuentes citadas al final de la unidad. La razón que da Let's Encrypt es doble: un certificado robado o emitido por error es válido durante menos tiempo, y se obliga a **automatizar** la renovación, porque renovar a mano cada pocas semanas no es viable. La consecuencia práctica: **un certificado público se gestiona con un programa, no con un calendario**.

Dentro de una organización es habitual lo contrario: una CA interna emite certificados de vida más larga (un año, como el de este apartado) para servicios que no son públicos. Es el caso de la CA del aula.

### 6.10 ⚖️ Alternativa y criterio

**Autofirmado, CA propia o CA pública.** La elección depende de **quién tiene que fiarse** del servidor:

- **Autofirmado**: solo cuando el único cliente eres tú y puedes aceptar la excepción conscientemente (una prueba rápida de cifrado). Su defecto es que acostumbra a pulsar «continuar» ante avisos, el peor hábito posible.
- **CA propia (interna)**: cuando los clientes son una población controlada (los equipos de una organización, un aula) en la que se puede instalar un certificado de CA una vez. Es la solución estándar de las redes internas, y escala: se confía una vez en la CA y se emiten tantos certificados como haga falta.
- **CA pública (Let's Encrypt, o comercial)**: cuando los clientes son cualquiera y el dominio es público. No hay que instalar nada en ningún cliente, porque las CA públicas ya están en todos los almacenes de confianza.

**Un certificado por nombre, con varios nombres o comodín.** Se puede emitir un certificado por sitio (como en `solvia.test` e `informes.solvia.test` por separado), uno con **varios nombres en el SAN** (el de este apartado) o uno **comodín** (`*.solvia.test`). Varios nombres en el SAN es sencillo pero, si se añade un sitio nuevo, hay que reemitir. Un certificado por sitio aísla los riesgos: si se compromete la clave de uno, los demás siguen siendo válidos. El comodín cubre todos los subdominios de un nivel, y a cambio, quien robe esa clave puede suplantar a **cualquiera** de ellos. En una unidad de prácticas se usa un SAN múltiple por simplicidad; en producción se elige según cuántos sitios haya y quién gestiona cada uno.

**RSA o ECDSA.** RSA es compatible con todo; ECDSA da claves más cortas y operaciones más rápidas. Para un servicio nuevo sin clientes antiguos, ECDSA es razonable; RSA de 2048 bits sigue siendo la opción segura por compatibilidad, y es la que se usa aquí.

### 6.11 ⚠️ Errores comunes

**Certificado sin SAN.** El nombre solo está en el `CN`. El navegador responde que el nombre no coincide, aunque el nombre sea el correcto. Se comprueba con `openssl x509 -in cert.crt -noout -ext subjectAltName`: si no sale nada (`No extensions in certificate`), falta. Se arregla reemitiendo con `-addext` (o `-extfile`).

**El nombre del certificado no es el que escribe el usuario.** Si el SAN dice `solvia.test` y se entra por la dirección IP, o por `localhost`, el certificado no vale para ese nombre y el navegador avisa. El certificado vale para los nombres que lista, y solo para ellos.

**Perder o compartir la clave privada.** La clave no sale del servidor, no se envía por correo, no se sube a un repositorio de código y no se copia a una carpeta servida por la web. Quien tiene la clave de un certificado puede suplantar al servidor. Si se filtra, el certificado se **revoca** y se emite uno nuevo con una clave nueva.

**Mezclar la clave y el certificado de dos pruebas.** Apache no arranca (o el apartado 7 da un error de que la clave no corresponde). Se detecta comparando el módulo, como en el apartado 6.7.

**Confundir la clave de la CA con la del servidor.** La clave de la CA (`ca-aula.key`) **nunca** se instala en un servidor ni se reparte. Del lado del cliente solo se importa el certificado de la CA (`ca-aula.crt`).

**Fechas.** Un certificado caducado se rechaza, y también uno que todavía no es válido: si el reloj de la máquina cliente va mal, un certificado recién creado puede parecer del futuro. Se mira con `date`.

**Olvidar `-nodes` y que el servidor no arranque.** Una clave protegida con contraseña obliga a escribirla cada vez que arranca Apache (que no está en una terminal para preguntarla). En un servidor, la clave va sin contraseña y protegida por permisos.

**Dejar el aviso del autofirmado y decir «no pasa nada».** El aviso significa que el cliente no puede comprobar con quién habla. Si se debe aceptar, se acepta sabiendo por qué; la solución correcta para un entorno interno es instalar la CA, no acostumbrarse a la excepción.

### 6.12 💡 Nota técnica: transparencia, revocación y lo que un certificado no dice

**Transparencia de certificados.** Los navegadores principales exigen que los certificados de las CA públicas estén anotados en registros públicos y verificables (*Certificate Transparency*), de modo que, en la práctica, cada certificado público que se emite queda publicado. Sirve para detectar certificados emitidos por error o de forma fraudulenta, y tiene un efecto que conviene conocer: **cualquier nombre para el que se pida un certificado público queda publicado**. Un administrador que obtiene un certificado para `panel-interno.empresa.com` está anunciando que ese nombre existe. Por eso los servicios que no deben ser conocidos no deben tener nombres delatores, y por eso las redes internas usan CA propias: nada de lo que emite la CA del aula aparece en ningún registro.

**Revocación.** Un certificado puede retirarse antes de caducar, y los clientes pueden comprobarlo con listas de revocación o servicios en línea (OCSP). En la práctica, la revocación es un mecanismo con limitaciones, y la tendencia es acortar la vida de los certificados (apartado 6.9) para que el problema pese menos: un certificado de 47 días necesita menos revocación que uno de dos años.

**Qué dice y qué no dice un certificado.** El certificado confirma que quien responde en `solvia.test` posee la clave privada del certificado emitido para ese nombre. No dice nada sobre si el servidor está bien configurado, si la aplicación tiene fallos, o si la organización es honrada: el candado del navegador significa «la conexión va cifrada y con el servidor que dice ser», no «este sitio es de fiar». Los certificados de validación de dominio (los de Let's Encrypt) solo prueban el control del dominio.

### 6.13 Comprueba lo que has entendido

**1. Un compañero dice: «Con un certificado autofirmado la conexión no va cifrada, por eso el navegador avisa». ¿Es correcto?**

No. Con un certificado autofirmado la conexión sí va cifrada. El aviso aparece porque el navegador **no puede comprobar con quién habla**: nadie en su almacén de confianza ha firmado ese certificado. Se pierde la propiedad de autenticación del servidor, no la de confidencialidad.

**2. `openssl verify solvia.crt` responde `error 18 ... self-signed certificate` y, para otro certificado, `error 20 ... unable to get local issuer certificate`. ¿Qué diferencia hay y cómo se arregla cada uno?**

El 18 indica que el certificado está firmado por sí mismo: emisor y sujeto son iguales, y no hay ninguna CA que lo respalde; solo se arregla sustituyéndolo por uno firmado por una CA, o aceptándolo como excepción. El 20 indica que sí lo firmó otra entidad, pero el sistema no tiene el certificado de esa CA en su almacén; se arregla instalando el certificado de la CA (por ejemplo `ca-aula.crt` con `update-ca-certificates`).

**3. Has generado un certificado con `-subj "/CN=solvia.test"` y sin `-addext subjectAltName`. Chrome dice que el nombre no coincide. ¿Por qué?**

Porque los navegadores actuales ignoran el `CN` y exigen que el nombre esté en la extensión SAN. Se arregla reemitiendo el certificado con `-addext "subjectAltName=DNS:solvia.test"` (o con el SAN en el fichero de extensiones al firmar el CSR).

**4. ¿Qué viaja en un CSR y qué no? ¿Por qué esa separación es importante?**

Viaja la clave pública, los nombres solicitados y una autofirma que prueba que el solicitante posee la clave privada. La clave privada **no** viaja nunca. Es importante porque la clave privada es lo que permite suplantar al servidor: si la CA (o quien intercepte el CSR) no la tiene, no puede hacerse pasar por él.

**5. Importas `ca-aula.crt` en el navegador y desde entonces `solvia.test` e `informes.solvia.test` se abren sin aviso, aunque son dos certificados distintos. ¿Por qué no hace falta importar cada uno?**

Porque el navegador no confía en cada certificado, sino en quien los firmó. Al confiar en la CA, acepta todo lo que ella firme: valida la cadena de cada certificado hasta la CA importada. Y por la misma razón, la clave privada de la CA hay que guardarla con el máximo cuidado: quien la tenga puede emitir certificados que ese navegador aceptará.

---

## 7. Asegurar las comunicaciones entre cliente y servidor

**CE 2.f** — Se han implantado sitios virtuales seguros y se han asegurado las comunicaciones (cifrado, redirección a HTTPS, cabeceras de seguridad).

Hasta aquí el servidor de Solvia atiende por el puerto 80 con **HTTP**, un protocolo que viaja en texto claro. En el apartado 5 se vio con `tcpdump` que la cabecera `Authorization` (usuario y contraseña en base64) se lee a simple vista en la red. En el apartado 6 se prepararon las piezas para arreglarlo: un certificado firmado por la CA del aula y su clave privada, guardados en `/etc/apache2/ssl/`. Este bloque las pone a trabajar: se activa **HTTPS** en Solvia, se obliga a usarlo y se añaden las cabeceras con las que el servidor le pide al navegador que se comporte de forma segura.

### 7.1 Qué aporta HTTPS (y qué no)

**HTTPS** es el protocolo HTTP de siempre, pero dentro de un canal protegido por **TLS** (*Transport Layer Security*, el sucesor de SSL; el nombre SSL sigue en muchos sitios por herencia, como en `mod_ssl`). TLS aporta tres garantías que HTTP no tiene:

- **Confidencialidad.** Lo que viaja está cifrado: quien capture el tráfico ve bytes ilegibles.
- **Integridad.** Si alguien modifica los datos por el camino, el receptor lo detecta.
- **Autenticidad del servidor.** El cliente comprueba el certificado (apartado 6.4) y sabe que habla con quien dice ser, no con un intermediario.

Conviene tener claro también lo que **no** hace. No protege el servidor de un ataque a la propia aplicación (una inyección SQL llega igual por HTTPS), ni protege los datos una vez que están en el servidor (en la base de datos, en un fichero de log). Y no lo oculta todo: una persona que capture el tráfico sigue viendo **entre qué direcciones IP** hay comunicación, **cuánto** se transfiere y **cuándo**, y normalmente el **nombre del sitio** al que se conecta el cliente, porque ese nombre viaja sin cifrar en el primer mensaje del saludo TLS (se llama *SNI*, *Server Name Indication*, y es lo que permite al servidor elegir el certificado correcto antes de que empiece el cifrado; es la versión TLS del problema de los sitios virtuales del apartado 3). Lo que queda protegido es la **dirección completa de la página (la ruta y los parámetros), las cabeceras, las cookies y el cuerpo** de la petición y de la respuesta.

### 7.2 Activar HTTPS en un sitio virtual

Apache gestiona TLS con el módulo `mod_ssl`. Los módulos de Apache se activan con `a2enmod` (apartado 2.4) y, como `mod_ssl` y `mod_headers` (que se usará más abajo) vienen instalados pero desactivados, se activan:

```bash
sudo a2enmod ssl
sudo a2enmod headers
sudo apache2ctl configtest
sudo systemctl restart apache2
```

```text
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Module socache_shmcb already enabled
Enabling module ssl.
See /usr/share/doc/apache2/README.Debian.gz on how to configure SSL and create self-signed certificates.
To activate the new configuration, you need to run:
  service apache2 restart
Enabling module headers.
To activate the new configuration, you need to run:
  service apache2 restart
Syntax OK
```

`apache2ctl configtest` comprueba la sintaxis sin tocar el servidor; con `Syntax OK` se puede reiniciar. (En una máquina donde Apache ya esté en marcha, `systemctl restart` también es válido.) Al activar `ssl`, Ubuntu empieza a hacer que Apache escuche además en el **puerto 443**, el puerto estándar de HTTPS: lo hace la directiva `Listen 443` que ya está en `/etc/apache2/ports.conf` dentro de un bloque `<IfModule ssl_module>`, es decir, «solo si el módulo ssl está cargado». No hay que escribirla.

Un sitio virtual HTTPS es un `<VirtualHost *:443>` con tres directivas nuevas:

```apache
<VirtualHost *:443>
        ServerName solvia.test
        DocumentRoot /var/www/solvia/public

        SSLEngine on
        SSLProtocol -all +TLSv1.2 +TLSv1.3
        SSLCertificateFile    /etc/apache2/ssl/solvia.crt
        SSLCertificateKeyFile /etc/apache2/ssl/solvia.key
</VirtualHost>
```

- `SSLEngine on` activa TLS en este sitio virtual. Sin ella, Apache respondería en HTTP por el puerto 443 y el navegador (que espera TLS) fallaría con un error de protocolo.
- `SSLCertificateFile` apunta al certificado, el fichero **público** que el servidor enseña a cada cliente. `SSLCertificateKeyFile` apunta a la **clave privada**, que Apache usa en el arranque y en el saludo TLS pero nunca envía. Las dos rutas son las del apartado 6.7; si el certificado y la clave no son pareja, Apache se niega a arrancar (por eso se comparó el módulo de ambos allí).
- `SSLProtocol -all +TLSv1.2 +TLSv1.3` se explica en el apartado 7.5: lista exactamente qué versiones de TLS se aceptan.

Como en cada sitio virtual, el nombre que decide a qué sitio se atiende es el `ServerName` (apartado 3), y debe coincidir con alguno de los nombres del certificado (el SAN del apartado 6.2). Un detalle importante: **un certificado vale para un nombre, no para una IP**, salvo que la IP figure expresamente en el SAN. Por eso el certificado de Solvia solo se acepta si se entra por `solvia.test` y no por `127.0.0.1`, como se comprueba en el apartado 7.4.

### 7.3 Redirigir de HTTP a HTTPS

Activar el 443 no basta: quien escriba `http://solvia.test` o siga un enlace antiguo seguirá llegando por el puerto 80, sin cifrar. Lo normal es que el sitio del puerto 80 no sirva nada propio y solo **redirija** al equivalente en HTTPS. Una redirección es una respuesta HTTP con código `3xx` y una cabecera `Location` que indica a dónde ir; el navegador la sigue sin intervención de la persona.

```apache
<VirtualHost *:80>
        ServerName solvia.test
        ServerAlias localhost
        Redirect permanent / https://solvia.test/
</VirtualHost>
```

`Redirect` (del módulo `mod_alias`, que viene activado por defecto) tiene la forma `Redirect [estado] ruta-origen destino`. Aquí, `permanent` equivale al código **301** (*Moved Permanently*: «esto se ha mudado para siempre», que los navegadores recuerdan), `/` es la ruta de origen (todo el sitio) y lo que sigue es el destino. Apache **añade al destino la parte de la ruta que sobra**: una petición a `http://solvia.test/nuevo_ticket.php` se redirige a `https://solvia.test/nuevo_ticket.php`. Se comprueba con `curl -I` (la `-I` pide solo las cabeceras de respuesta):

```bash
curl -sI http://solvia.test/
```

```text
HTTP/1.1 301 Moved Permanently
Date: Mon, 09 Nov 2026 09:19:42 GMT
Server: Apache
Location: https://solvia.test/
Content-Type: text/html; charset=iso-8859-1
```

(El `Server: Apache` sin número de versión viene de `ServerTokens Prod`, ajustado en el apartado 1.5.)

**301 o 308: qué pasa con un formulario.** Un 301 tiene una particularidad histórica: ante un 301, los clientes convierten una petición `POST` (la que envía los datos de un formulario) en un `GET` y **pierden el cuerpo**. El código **308** (*Permanent Redirect*) significa lo mismo, pero obliga a repetir el método y el cuerpo. La diferencia se ve en el registro de acceso, que apunta el método con el que llegó la petición definitiva. Se envía un `POST` con `curl -d` a la dirección HTTP y se deja que `curl` siga la redirección (`-L`); con `Redirect permanent`, el servidor registra:

```text
"GET /nuevo_ticket.php HTTP/1.1" 200
```

y con `Redirect 308 / https://solvia.test/`, registra:

```text
"POST /nuevo_ticket.php HTTP/1.1" 200
```

En la práctica esto importa poco, porque la redirección sirve para que las personas **lleguen** a HTTPS, y las personas llegan con `GET` (escribir una dirección o seguir un enlace). Un formulario de Solvia que se envíe por HTTPS no pasa por la redirección. El 308 es un seguro para clientes automáticos que envían datos a una URL antigua. Si los dos funcionan, elige el que prefieras; lo que no vale es redirigir sin que se vea claro el motivo.


### 7.4 Comprobar que HTTPS funciona

La primera comprobación es que Apache reconoce el nuevo sitio. `apache2ctl -S` lista los sitios por puerto:

```bash
sudo apache2ctl -S
```

```text
*:443                  is a NameVirtualHost
         default server defecto.test (/etc/apache2/sites-enabled/00-defecto.conf:10)
         port 443 namevhost defecto.test (/etc/apache2/sites-enabled/00-defecto.conf:10)
         port 443 namevhost informes.solvia.test (/etc/apache2/sites-enabled/informes.conf:6)
         port 443 namevhost solvia.test (/etc/apache2/sites-enabled/solvia.conf:7)
```

(La salida completa de `apache2ctl -S` es más larga; aquí se muestran las líneas del puerto 443, con los tres sitios que se configuran en este bloque.) La segunda es hacer una petición. El cliente tiene que **confiar en la CA que firmó el certificado**: si no, `curl` se detiene, exactamente como lo haría un navegador con un aviso a pantalla completa. Con el sistema donde **no** se ha instalado la CA del aula:

```bash
curl https://solvia.test/
```

```text
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

Es el error 20 del apartado 6 visto desde `curl`. Hay tres maneras de seguir adelante, y la diferencia entre ellas es lo importante:

- **`curl --cacert ca-aula.crt URL`** — le dice a `curl` en qué CA confiar solo para esta petición. Es la forma correcta cuando se confía en esa CA pero no se ha instalado en el sistema.
- **Instalar la CA en el almacén del sistema** (paso 5 del apartado 6.6). Es lo que hay que hacer si se va a usar a menudo.
- **`curl -k URL`** (`--insecure`) — **desactiva la verificación del certificado**. La conexión sigue cifrada, pero no se comprueba con quién se habla, así que no protege contra un intermediario. Solo sirve para diagnosticar («¿el problema es el certificado o es otra cosa?») y nunca debe quedar como solución.

```bash
curl -s --cacert ca-aula.crt -o /dev/null -w "%{http_code}\n" https://solvia.test/
```

```text
200
```

(`-s` silencia el progreso, `-o /dev/null` descarta el contenido y `-w "%{http_code}\n"` imprime solo el código de estado.) Con `-k` en una dirección que el certificado **no** cubre, se ve la diferencia entre «cifrado» y «verificado». Si se entra por la IP:

```bash
curl --cacert ca-aula.crt https://127.0.0.1/
```

```text
curl: (60) SSL: no alternative certificate subject name matches target host name '127.0.0.1'
More details here: https://curl.se/docs/sslcerts.html
```

La CA es de confianza, pero el certificado es para `solvia.test`, no para `127.0.0.1`, y el cliente lo rechaza. Es una protección útil: impide que alguien que se haga pasar por otro nombre use el certificado de un tercero. Para probar un nombre que no está en DNS ni en `/etc/hosts`, se usa `--resolve nombre:443:IP`, que le dice a `curl` «cuando necesites `solvia.test` en el 443, usa esta IP» sin cambiar la dirección escrita (ni, por tanto, el nombre que se comprueba en el certificado).

Con `openssl s_client` se ve el saludo TLS completo, que es lo que hace un navegador antes de mostrar nada:

```bash
openssl s_client -connect 127.0.0.1:443 -servername solvia.test -CAfile ca-aula.crt </dev/null
```

La salida es larga; estas son las líneas que importan:

```text
subject=CN = solvia.test
issuer=O = IES Rio Arba, CN = CA del aula
Verification: OK
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Verify return code: 0 (ok)
```

`-connect` indica a dónde conectar; `-servername` es el SNI (el nombre que el servidor usa para elegir sitio y certificado, apartado 7.1); `-CAfile` es la CA de confianza; y `</dev/null` cierra la entrada para que la orden termine sola. Se lee así: el certificado es el de `solvia.test`, emitido por la CA del aula, la verificación es correcta, y la conexión usa TLS 1.3 con la suite de cifrado `TLS_AES_256_GCM_SHA384`.

### 7.5 Versiones de TLS y suites de cifrado

TLS ha tenido varias versiones y las antiguas tienen debilidades conocidas. SSL 2 y 3 y TLS 1.0 y 1.1 se consideran obsoletos; las dos versiones vigentes son **TLS 1.2** y **TLS 1.3**. La directiva `SSLProtocol` (la que ya aparece en el sitio) lo expresa con una lista: `-all` quita todas las versiones, y `+TLSv1.2 +TLSv1.3` añade solo esas dos. Se escribe explícita aunque Ubuntu ya haga mucho por su cuenta: el valor por defecto que trae `mods-enabled/ssl.conf` es `SSLProtocol all -SSLv3`, y la política de OpenSSL en Ubuntu 24.04 rechaza TLS 1.0 y 1.1 en la práctica. Tenerlo escrito en el sitio hace la intención visible y la protege de que cambie el valor por defecto.

Para comprobar que las versiones antiguas se rechazan, se fuerza al cliente a ofrecer solo una versión. El cliente de OpenSSL también tiene sus propias restricciones de seguridad, que hay que bajar (`-cipher 'DEFAULT:@SECLEVEL=0'`) para que ofrezca una versión tan antigua y la prueba llegue realmente al servidor:

```bash
openssl s_client -connect 127.0.0.1:443 -servername solvia.test -tls1_1 -cipher 'DEFAULT:@SECLEVEL=0' </dev/null
```

```text
4067DFC3277F0000:error:0A00042E:SSL routines:ssl3_read_bytes:tlsv1 alert protocol version:../ssl/record/rec_layer_s3.c:1599:SSL alert number 70
```

El **servidor** responde con la alerta TLS número 70 (*protocol version*): «no acepto esa versión». Si se usa `-tls1_2` o `-tls1_3`, la conexión se establece (con la CA aún sin instalar, aparece además el aviso de verificación del apartado 6). Con `curl` se puede comprobar la parte positiva, que cada una de las versiones vigentes funciona:

```bash
for v in 1.2 1.3; do curl -s --cacert ca-aula.crt --tlsv$v --tls-max $v -o /dev/null -w "tls$v %{http_code}\n" https://solvia.test/; done
```

```text
tls1.2 200
tls1.3 200
```

(`--tlsv$v` fija la versión mínima y `--tls-max $v` la máxima; con las dos iguales, `curl` ofrece exactamente esa versión.)

Las **suites de cifrado** (*cipher suites*) son las combinaciones de algoritmos que se negocian dentro de una versión. Con TLS 1.3 la lista es corta y todas son seguras; con TLS 1.2 es más larga y es donde puede haber opciones débiles. El valor por defecto de Ubuntu (`SSLCipherSuite HIGH:!aNULL`) es razonable para una práctica. Ajustar las suites a mano es un tema delicado que depende de qué clientes viejos se quiera seguir admitiendo; en una instalación real se parte de las recomendaciones de una fuente actualizada (el generador de configuraciones de Mozilla es una referencia conocida; hay que consultar su versión vigente porque las recomendaciones cambian) y no de listas copiadas de internet.

### 7.6 HTTPS para el sitio de informes: la autenticación Basic, por fin cifrada

En el apartado 5 se concluyó que `AuthType Basic` solo es aceptable **sobre HTTPS**, porque envía la contraseña en base64, que no es cifrado. Ahora se puede hacer realidad. El sitio `informes.solvia.test` pasa a tener el mismo esquema que Solvia: el puerto 80 redirige y el 443 sirve el contenido y pide la contraseña.

```apache
<VirtualHost *:80>
        ServerName informes.solvia.test
        Redirect permanent / https://informes.solvia.test/
</VirtualHost>

<VirtualHost *:443>
        ServerName informes.solvia.test
        DocumentRoot /var/www/informes

        SSLEngine on
        SSLCertificateFile    /etc/apache2/ssl/solvia.crt
        SSLCertificateKeyFile /etc/apache2/ssl/solvia.key

        <Directory /var/www/informes>
                AuthType Basic
                AuthName "Informes de Solvia"
                AuthUserFile /etc/apache2/.htpasswd-informes
                Require valid-user
        </Directory>

        <Location "/server-status">
                SetHandler server-status
                AuthType Basic
                AuthName "Informes de Solvia"
                AuthUserFile /etc/apache2/.htpasswd-informes
                Require valid-user
        </Location>

        ErrorLog ${APACHE_LOG_DIR}/informes_error.log
        CustomLog ${APACHE_LOG_DIR}/informes_access.log combined
</VirtualHost>
```

Se reutiliza el mismo certificado porque el que se creó en el apartado 6 incluye en su SAN los dos nombres, `solvia.test` e `informes.solvia.test`. Si no fuera así, `curl` o el navegador rechazarían `informes.solvia.test` por el motivo de la IP del apartado 7.4: el nombre pedido no está en el certificado. Se comprueban los tres casos de siempre, ahora por TLS:

```bash
curl -sI http://informes.solvia.test/ | sed -n '1p;/Location/p'
curl -s --cacert ca-aula.crt -o /dev/null -w "%{http_code}\n" https://informes.solvia.test/
curl -s --cacert ca-aula.crt -u maria:ClaveDeP1 -o /dev/null -w "%{http_code}\n" https://informes.solvia.test/
```

```text
HTTP/1.1 301 Moved Permanently
Location: https://informes.solvia.test/
401
200
```

La primera redirige; la segunda, sin credenciales, recibe `401` (hace falta autenticarse); la tercera, con usuario y contraseña (`-u usuario:contraseña`), recibe `200`. El `sed -n '1p;/Location/p'` solo filtra la salida para mostrar la línea de estado y la de `Location`.

**Qué ve alguien que captura el tráfico.** Con `tcpdump` (apartado 5.3) se capturan por separado los puertos 80 y 443 mientras se hace una petición autenticada, y se busca el texto de la contraseña:

```bash
sudo tcpdump -i lo -n -w /tmp/c443.pcap 'tcp port 443' &
curl -s --cacert ca-aula.crt -u maria:ClaveDeP1 -d "asunto=prueba-secreta&descripcion=clave-visible" -o /dev/null https://informes.solvia.test/
sudo kill %1
sudo tcpdump -nr /tmp/c443.pcap -A | grep -a -c -E "ClaveDeP1|prueba-secreta|Authorization|bWFyaWE6Q2xhdmVEZVAx"
sudo tcpdump -nr /tmp/c443.pcap -A | grep -a -o "informes.solvia.test" | head -1
```

```text
0
informes.solvia.test
```

(`-i lo` captura en la interfaz de bucle local, que es por donde van estas pruebas; `-w` guarda la captura en un fichero; `-nr` lo lee sin resolver nombres; `-A` muestra el contenido de los paquetes como texto; y `grep -a -c` cuenta cuántas líneas contienen alguno de los textos, tratando el binario como texto.) El resultado: **cero coincidencias** para la contraseña, su versión en base64, la cabecera `Authorization` y el contenido del formulario, y **una** para el nombre del sitio, que es el SNI del apartado 7.1: lo único legible del saludo. Para compararlo, la misma captura sobre el puerto 80 contiene:

```text
Authorization: Basic bWFyaWE6Q2xhdmVEZVAx
```

Esa línea es la contraseña, a la vista, aunque el servidor del puerto 80 solo fuera a redirigir (`curl -u` envía las credenciales desde el primer mensaje; un navegador espera a que el servidor las pida con un 401, pero si lo hace por HTTP, el resultado es el mismo). Es la razón de que, en el sitio de informes, **el puerto 80 no pida contraseña ni sirva contenido: solo redirige**.

### 7.7 El sitio por defecto también existe en HTTPS

En el apartado 3 se vio que, si ningún sitio coincide con el nombre pedido, Apache responde con **el primero que leyó**, y se creó `00-defecto.conf` (que se lee primero por el orden alfabético) para que ese primero devuelva `403`. Todo eso era para el puerto 80. Cada puerto tiene su propio sitio por defecto, así que ahora hace falta uno para el 443. Como el sitio por defecto no tiene un nombre propio que cubrir, no tiene un certificado «correcto»; se usa el certificado de ejemplo de Ubuntu (el *snakeoil*, apartado 6.7), que existe justo para esto, y el visitante verá un aviso porque no es de confianza, que es lo que debe ocurrir ante un nombre desconocido.

```apache
<VirtualHost *:443>
        ServerName defecto.test
        DocumentRoot /var/www/defecto

        SSLEngine on
        SSLCertificateFile    /etc/ssl/certs/ssl-cert-snakeoil.pem
        SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key

        <Directory /var/www/defecto>
                Require all denied
        </Directory>
</VirtualHost>
```

Este bloque se añade a `00-defecto.conf` debajo del `<VirtualHost *:80>` que ya tiene. Se comprueba pidiendo un nombre que no corresponde a ningún sitio, con `--resolve` (apartado 7.4) y `-k` (porque ese certificado no es de confianza):

```bash
curl -sk --resolve otro.test:443:127.0.0.1 -o /dev/null -w "%{http_code}\n" https://otro.test/
openssl s_client -connect 127.0.0.1:443 -servername otro.test </dev/null 2>/dev/null | grep -E "subject=|issuer="
openssl s_client -connect 127.0.0.1:443 -servername solvia.test </dev/null 2>/dev/null | grep -E "subject=|issuer="
```

```text
403
subject=CN = localhost
issuer=CN = localhost
subject=CN = solvia.test
issuer=O = IES Rio Arba, CN = CA del aula
```

Para un nombre desconocido, el servidor devuelve `403` y presenta el certificado de ejemplo (un certificado autofirmado para `localhost`); para `solvia.test`, presenta el de la CA del aula. **Sin este sitio**, el primero que Apache leyese en el 443 (en el orden alfabético, `informes.conf`) contestaría a nombres desconocidos, con su contraseña y su certificado: es el mismo fallo que la trampa del apartado 3.

### 7.8 Cabeceras de seguridad HTTP

Las **cabeceras de seguridad** son cabeceras de la respuesta HTTP con las que el servidor le da instrucciones al navegador sobre cómo comportarse con esa página. Son una segunda línea de defensa: si la aplicación tiene un fallo (por ejemplo, permite que se inyecte código en una página), estas cabeceras pueden impedir que el navegador lo aproveche. Se añaden con la directiva `Header` del módulo `mod_headers` (que se activó en el apartado 7.2; el apartado 2.9 lo explica con más detalle):

```apache
Header always set X-Content-Type-Options "nosniff"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Content-Security-Policy "default-src 'self'; style-src 'self' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; script-src 'self'; img-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'"
Header always set Strict-Transport-Security "max-age=300"
```

La forma es `Header [always] set Nombre "valor"`. `set` pone la cabecera (sustituyendo una anterior con el mismo nombre). `always` hace que se envíe también en las respuestas de error (404, 403, 500…); sin `always`, solo se añade a las respuestas con éxito. Interesa en todas, y se comprueba (apartado 7.9).

**`X-Content-Type-Options: nosniff`.** Cada respuesta declara su tipo (`Content-Type`: `text/html`, `text/css`…). Por compatibilidad, los navegadores a veces ignoran lo declarado y «adivinan» el tipo mirando el contenido (*sniffing*). Eso permite, por ejemplo, que un fichero subido por un usuario con aspecto de imagen, pero con código dentro, se ejecute como script. `nosniff` indica al navegador que haga caso a lo declarado.

**`Referrer-Policy: strict-origin-when-cross-origin`.** Cuando un usuario pasa de una página a otra (con un enlace, por ejemplo), el navegador envía en la cabecera `Referer` la dirección de la página de origen. Esa dirección puede contener datos que no deben salir (un identificador de ticket, por ejemplo: `https://solvia.test/consultar_ticket.php?id=1234`). Con este valor, dentro del mismo sitio se envía la dirección completa, hacia otros sitios solo el origen (`https://solvia.test/`, sin ruta ni parámetros), y no se envía nada si se pasa de HTTPS a HTTP.

**`Content-Security-Policy` (CSP).** Es la más potente y la más fácil de configurar mal. Define una lista blanca de **de dónde** puede cargar la página cada tipo de recurso. Cada parte de la política es una directiva:

| Directiva | Qué controla | Valor en Solvia |
|---|---|---|
| `default-src 'self'` | Valor por defecto para todo lo que no tenga directiva propia: solo desde el propio sitio (`'self'`) | solo el propio sitio |
| `script-src 'self'` | Scripts | solo ficheros del propio sitio: **bloquea los scripts escritos dentro de la página** (*inline*) y los de otros orígenes |
| `style-src 'self' https://fonts.googleapis.com` | Hojas de estilo | el sitio y Google Fonts (Solvia carga su tipografía de ahí) |
| `font-src https://fonts.gstatic.com` | Ficheros de tipografía | el servidor donde Google Fonts guarda los ficheros |
| `img-src 'self'` | Imágenes | solo del sitio |
| `frame-ancestors 'none'` | Quién puede incrustar esta página en un `<iframe>` | nadie; protege contra el *clickjacking*, que consiste en incrustar una página invisible bajo un botón engañoso |
| `base-uri 'self'` | La etiqueta `<base>` | solo el propio sitio (si no, un atacante podría cambiar hacia dónde apuntan los enlaces relativos) |
| `form-action 'self'` | A dónde pueden enviar datos los formularios | solo al propio sitio |

La política de arriba no es genérica: está **escrita a la medida de Solvia**, a partir de lo que Solvia realmente carga. Se midió que las páginas de Solvia solo usan una hoja de estilo (`assets/css/style.css`) y un script (`assets/js/main.js`) del propio sitio, no llevan ningún `<script>` ni atributo `style` escrito dentro del HTML, no tienen imágenes, y cargan su tipografía de Google Fonts. Cada directiva se corresponde con un hecho. Si la política hubiera sido más estricta que lo que la aplicación necesita, el navegador bloquearía recursos legítimos y la página se vería rota; si fuera más laxa (por ejemplo `script-src *`), no protegería.


Un modo seguro de introducir una CSP se describe en el apartado 7.12.

Hay otras cabeceras habituales que este bloque no configura: `X-Frame-Options` (el antecesor de `frame-ancestors`, útil en navegadores antiguos) y `Permissions-Policy` (qué funciones del navegador, como cámara o geolocalización, puede usar la página). En la práctica se ve en listas de comprobación de seguridad, y conviene reconocerlas aunque no se pidan aquí.


### 7.9 HSTS: no volver a pasar por HTTP

La redirección del apartado 7.3 tiene un punto débil: la **primera** petición sigue yendo por HTTP, y en esa petición alguien que controle la red (un Wi-Fi público, por ejemplo) puede interceptar y no dejar que llegue la redirección. **HSTS** (*HTTP Strict Transport Security*) lo cierra: la cabecera `Strict-Transport-Security: max-age=300` le dice al navegador «durante los próximos 300 segundos, para este sitio **no uses nunca HTTP**; convierte tú mismo cualquier `http://` en `https://` antes de enviar nada». `max-age` es el tiempo que el navegador recuerda la orden, en segundos.

Dos reglas para usarlo bien:

1. **Solo en las respuestas HTTPS.** Un navegador ignora la cabecera si llega por HTTP (no tendría sentido fiarse de una orden que ha podido manipular cualquiera). Se pone en el `<VirtualHost *:443>`, no en el 80. Se comprueba que está en el 443, también en errores (`always`), y que no está en el 80:

```bash
curl -sI --cacert ca-aula.crt https://solvia.test/noexiste | sed -n '1p;/Strict/p;/nosniff/p'
curl -sI http://solvia.test/ | grep -ci strict
```

```text
HTTP/1.1 404 Not Found
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=300
0
```

   (La primera orden pide una página inexistente para comprobar que las cabeceras salen también en una respuesta de error, gracias a `always`. La segunda cuenta las líneas que contienen «strict» en la respuesta del puerto 80: 0.)

2. **Se empieza con un `max-age` pequeño.** Una vez que un navegador recibe HSTS, **no hay vuelta atrás hasta que caduque**: si después el certificado caduca, o se instala mal, o se usa el sitio con un nombre que el certificado no cubre, el navegador **no permite pasar** (con HSTS no hay botón «continuar de todos modos» ante un certificado no válido) y la gente se queda sin acceso al sitio hasta que el `max-age` termine. Por eso `300` (cinco minutos) en la práctica. En producción, y cuando todo funciona de forma estable, se sube por fases (un día, una semana, un año…). La documentación del propio sitio de listas de precarga (*preload*) de HSTS indica requisitos más exigentes (como `includeSubDomains` y un `max-age` de un año como mínimo) para entrar en la lista que los navegadores traen de fábrica; conviene leer sus condiciones actuales antes de intentarlo, porque salir de esa lista es lento.

### 7.10 Caso práctico sobre Solvia: el sitio final

Con todo lo anterior, el sitio de Solvia queda así. Es el fichero `/etc/apache2/sites-available/solvia.conf` completo, que integra los cambios del apartado 4 (`Options -Indexes`, `AllowOverride None`, `ErrorDocument 404`) con los de este bloque:

```apache
<VirtualHost *:80>
        ServerName solvia.test
        ServerAlias localhost
        Redirect permanent / https://solvia.test/
</VirtualHost>

<VirtualHost *:443>
        ServerName solvia.test
        DocumentRoot /var/www/solvia/public

        SSLEngine on
        SSLProtocol -all +TLSv1.2 +TLSv1.3
        SSLCertificateFile    /etc/apache2/ssl/solvia.crt
        SSLCertificateKeyFile /etc/apache2/ssl/solvia.key

        <Directory /var/www/solvia/public>
                Options -Indexes
                AllowOverride None
                Require all granted
        </Directory>

        Header always set X-Content-Type-Options "nosniff"
        Header always set Referrer-Policy "strict-origin-when-cross-origin"
        Header always set Content-Security-Policy "default-src 'self'; style-src 'self' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; script-src 'self'; img-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'"
        Header always set Strict-Transport-Security "max-age=300"

        ErrorDocument 404 /errores/404.html
        ErrorLog ${APACHE_LOG_DIR}/solvia_error.log
        CustomLog ${APACHE_LOG_DIR}/solvia_access.log combined
</VirtualHost>
```

Después de editarlo: `sudo apache2ctl configtest`, y `sudo systemctl reload apache2` (un `reload` basta cuando solo cambia la configuración de un sitio; los cambios de módulos, como al activar `ssl`, exigen `restart`). Se verifica con una sola petición que enseña todo lo anterior a la vez:

```bash
curl -sI --cacert ca-aula.crt https://solvia.test/
```

```text
HTTP/1.1 200 OK
Date: Mon, 09 Nov 2026 09:16:26 GMT
Server: Apache
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'self'; style-src 'self' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; script-src 'self'; img-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
Strict-Transport-Security: max-age=300
Content-Type: text/html; charset=UTF-8
```

Y, para quien tiene un navegador delante, la comprobación que no se puede hacer con `curl`: abrir `https://solvia.test/` (con la CA del aula instalada en el navegador, apartado 6.6), abrir las herramientas del desarrollador (F12) y mirar la pestaña **Consola**. Si la CSP bloquea algo, aparece ahí un mensaje con la directiva infringida. Las pruebas de este apartado se hicieron con un navegador automatizado sobre la página de inicio de Solvia y no registró ningún bloqueo por la política; sin embargo, la aplicación completa tiene más páginas (el formulario, la consulta de tickets) y **solo se puede dar por buena la política recorriendo todas**. Hay que comprobarlo en el entorno de cada uno.

**Una advertencia sobre VirtualBox.** Si la máquina virtual está en red NAT con **reenvío de puertos** (por ejemplo, el puerto 8443 del equipo anfitrión hacia el 443 de la máquina virtual), la redirección `https://solvia.test/` envía al navegador al puerto 443 **del anfitrión**, no al 8443, y la conexión falla. Es una consecuencia lógica de cómo funciona `Location`: la dirección de destino se escribe tal cual. En esa configuración, la redirección tiene que incluir el puerto (`https://solvia.test:8443/`). No se ha probado en un entorno VirtualBox concreto, así que conviene verificarlo con el reenvío que se haya configurado. Con una red en modo puente (*bridged*), o accediendo desde la propia máquina virtual, no hay problema.

### 7.11 ⚖️ Alternativa y criterio

**Redirigir: `Redirect` frente a `mod_rewrite`.** Existe otra forma de redirigir, con el módulo de reescritura (`mod_rewrite`: `RewriteEngine On`, `RewriteCond`, `RewriteRule`). Es más potente porque permite condiciones (redirigir solo si la petición no es HTTPS, solo para ciertas rutas…), pero también más difícil de leer y de depurar. Cuando, como aquí, el sitio del puerto 80 **no sirve nada excepto la redirección**, `Redirect` es preferible porque hace exactamente una cosa, se entiende de un vistazo y no se puede equivocar. `mod_rewrite` compensa cuando hay condiciones que `Redirect` no sabe expresar.

**Cabeceras: Apache frente a la aplicación.** Las cabeceras también se podrían enviar desde la propia aplicación, con la función `header()` de PHP. La alternativa tiene un defecto: solo afecta a lo que genera PHP. Las hojas de estilo, los scripts, las páginas de error de Apache (404, 403) y cualquier fichero estático **no pasan por PHP** y saldrían sin cabeceras. Configurarlas en Apache las aplica a todo lo que el sitio sirve, y las deja en un solo sitio, bajo la responsabilidad de quien administra el servidor y no de quien programa la aplicación. Cuando una cabecera dependa de lo que hace una página concreta (una CSP distinta para una sola página), se complementa desde la aplicación.

### 7.12 ⚠️ Errores comunes

- **`SSLEngine on` olvidado**: Apache arranca sin errores, pero el navegador muestra un error de protocolo al entrar por HTTPS (el servidor habla HTTP por un puerto donde el cliente espera TLS). Se corrige añadiendo la directiva.
- **Certificado y clave que no son pareja**, o ruta mal escrita: Apache no arranca y el motivo está en `/var/log/apache2/error.log` (`journalctl -xeu apache2` o `systemctl status apache2` también orientan). Se compara el módulo como en el apartado 6.7.
- **Bucle de redirecciones**: si el sitio del 443 también contiene un `Redirect` hacia HTTPS, el navegador avisa de «demasiadas redirecciones». Solo el puerto 80 redirige.
- **Contenido mixto** (*mixed content*): una página HTTPS que carga un recurso por `http://` (una imagen, un script). El navegador lo bloquea o lo marca como inseguro. Todo recurso debe cargarse por HTTPS o con una ruta relativa.
- **HSTS con un `max-age` largo antes de tiempo**, o `-k` dejado en un script «para que funcione»: los dos son el modo de convertir un problema visible en uno invisible.
- **Copiar una CSP de internet y pegarla**: funciona o rompe la aplicación según la suerte, y casi nunca se ajusta bien a lo que la aplicación carga. Un modo seguro de introducir una CSP es la cabecera `Content-Security-Policy-Report-Only` (con la misma política): el navegador **no bloquea nada**, solo informa en la consola del navegador de lo que habría bloqueado, para corregir la política antes de aplicarla. Los bloqueos aparecen en la consola de las herramientas del desarrollador (F12) con un mensaje que nombra la directiva infringida.

### 7.13 💡 Nota técnica

En muchas instalaciones reales no es Apache quien habla TLS con los navegadores, sino un **proxy inverso** delante (Nginx, HAProxy o un balanceador del proveedor de la nube) que termina el cifrado y reenvía al servidor de aplicaciones por la red interna. Las ideas son las mismas —certificado, redirección, cabeceras, HSTS—, pero se configuran en ese componente. El certificado también se renueva solo en esos entornos (ACME, apartado 6.9). Lo aprendido aquí sirve igual: lo único que cambia es dónde está escrito.

### 7.14 Comprueba lo que has entendido

**1. Un compañero captura el tráfico de la red mientras entras en `https://informes.solvia.test/` con tu usuario y contraseña. ¿Qué puede ver y qué no?**

No puede ver la contraseña, ni la cabecera `Authorization`, ni la ruta de la página, ni las cookies, ni el contenido de la respuesta: todo eso viaja cifrado dentro de TLS. Sí puede ver las direcciones IP de ambos extremos, el tamaño y el momento de la comunicación y, en el saludo TLS, el nombre del sitio (el SNI): `informes.solvia.test`.

**2. `curl https://solvia.test/` falla con el error 60 «unable to get local issuer certificate». ¿Qué significa y cuál es la forma correcta de seguir?**

Significa que el certificado lo firmó una CA que el sistema desconoce (la CA del aula no está en su almacén de confianza). Lo correcto es indicar en qué CA confiar: `--cacert ca-aula.crt` para una vez, o instalar la CA en el sistema con `update-ca-certificates` si se va a usar a menudo. `-k` solo se admite para diagnosticar, porque desactiva la comprobación y deja de proteger contra un intermediario.

**3. ¿Por qué el sitio del puerto 80 de Solvia solo contiene una redirección, y por qué la cabecera HSTS se pone únicamente en el 443?**

El puerto 80 no cifra, así que no debe servir contenido ni pedir contraseñas: solo sirve para llevar al cliente a HTTPS. HSTS se pone en el 443 porque el navegador ignora esa cabecera cuando llega por HTTP (podría haberla manipulado cualquiera), y solo es fiable en una respuesta cifrada y con certificado verificado.

**4. Alguien configura `Header set X-Content-Type-Options "nosniff"` (sin `always`) y comprueba que sale en la página de inicio, pero no en una dirección que da 404. ¿Por qué?**

Sin `always`, `Header` solo añade la cabecera a las respuestas con éxito (código 2xx, como el 200). Las demás respuestas, como el 404 o las redirecciones, salen sin ella. Con `Header always set …` se envía en todas.

**5. Pones `Strict-Transport-Security: max-age=31536000` y, una semana después, el certificado caduca. ¿Qué ocurre y qué se debería haber hecho?**

Los navegadores que ya habían visitado el sitio recordarán la orden durante un año y, ante un certificado caducado, **no ofrecen la opción de continuar**: los usuarios se quedan sin acceso hasta renovar el certificado. Se debe empezar con un `max-age` pequeño (como `300`) y subirlo por fases cuando la renovación del certificado esté probada y sea fiable.

---

## 8. Gestión de logs: monitorización, consolidación y análisis

**CE 2.j** — Se han instalado, configurado y utilizado conjuntos de herramientas de gestión de logs, permitiendo su monitorización, consolidación y análisis en tiempo real.

Un servidor web atiende cientos o miles de peticiones sin que nadie las mire. El **registro** (en inglés *log*) es la memoria de lo que ha ocurrido: quién pidió qué, cuándo, qué se le respondió y qué falló por dentro. Sin él, un administrador solo puede adivinar. Este bloque enseña a leer los registros de Apache, a vigilarlos mientras se escriben, a analizarlos con las herramientas de la línea de comandos y con **GoAccess**, a evitar que llenen el disco con **logrotate** y a reunir en un solo sitio los de varios sitios virtuales.

### 8.1 Para qué sirve un registro

Los registros de un servidor web responden a cuatro tipos de preguntas:

- **Diagnóstico**: «la página da error 500, ¿qué ha pasado dentro del servidor?».
- **Seguridad**: «¿alguien está probando contraseñas?», «¿alguien busca ficheros que no debería existir?».
- **Uso y rendimiento**: «¿cuántas visitas hay?», «¿qué páginas son lentas?», «¿qué porcentaje de peticiones falla?».
- **Auditoría**: «¿quién accedió al informe del mes pasado, y desde dónde?». Esta es la razón por la que los registros se conservan durante un tiempo y no se alteran.

Apache escribe, por separado, dos registros:

| Registro | Qué contiene | Directiva | Fichero en Ubuntu |
|---|---|---|---|
| **De accesos** (*access log*) | Una línea por **cada petición atendida**, con su resultado | `CustomLog` | el que se indique; el de ejemplo del sitio de Solvia es `solvia_access.log` |
| **De errores** (*error log*) | Los **problemas** y avisos del servidor, de los módulos y de la aplicación | `ErrorLog` | `error.log` el global; el de cada sitio, el que se indique (`solvia_error.log`) |

En Ubuntu, todos viven en `/var/log/apache2/`. El sitio de Solvia ya los tiene configurados desde el principio, con estas dos líneas dentro del `VirtualHost`:

```apache
ErrorLog ${APACHE_LOG_DIR}/solvia_error.log
CustomLog ${APACHE_LOG_DIR}/solvia_access.log combined
```

`${APACHE_LOG_DIR}` es una variable que Ubuntu define como `/var/log/apache2`; y `combined` es el nombre de un **formato de registro**, que se explica a continuación.

### 8.2 El registro de accesos: leer una línea

Cada petición queda registrada como una línea de texto. Esta es una real del servidor de Solvia:

```text
127.0.0.66 - - [09/Nov/2026:09:21:39 +0000] "GET /.env HTTP/1.1" 404 2667 "-" "zgrab/0.x"
```

Cada trozo de la línea es un campo, y el formato `combined` define el orden. En la configuración global de Ubuntu (`/etc/apache2/apache2.conf`) está definido así:

```apache
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

`LogFormat` da nombre a un formato; cada `%x` es un **código** que Apache sustituye por un dato de la petición. Descifrada, la línea anterior dice:

| Código | Campo | En la línea de ejemplo | Qué significa |
|---|---|---|---|
| `%h` | Host | `127.0.0.66` | Dirección IP del cliente |
| `%l` | Identidad (*ident*) | `-` | Casi siempre vacío (un guion): es un mecanismo antiguo que ya nadie usa |
| `%u` | Usuario | `-` | Nombre de usuario si se autenticó con Basic; `-` si no hubo autenticación |
| `%t` | Hora (*time*) | `[09/Nov/2026:09:21:39 +0000]` | Fecha, hora y desfase respecto a UTC |
| `\"%r\"` | Petición (*request*) | `"GET /.env HTTP/1.1"` | Método, ruta pedida y versión de HTTP. Las comillas se escriben `\"` dentro del formato para que formen parte de la línea |
| `%>s` | Estado final | `404` | Código de respuesta (200 correcto, 404 no encontrado…). El `>` significa «el definitivo», después de redirecciones internas |
| `%O` | Bytes enviados | `2667` | Total de bytes de la respuesta, **incluyendo las cabeceras** |
| `\"%{Referer}i\"` | Origen (*referer*) | `"-"` | Página desde la que llegó el cliente (`-` si no hay) |
| `\"%{User-Agent}i\"` | Agente de usuario | `"zgrab/0.x"` | El programa que hizo la petición: navegador, `curl`, un escáner… |

Dos matices que dan problemas al principio. El primero: el formato `common`, que muchos manuales usan, es igual pero sin los dos últimos campos. El segundo: el formato estándar de Apache usa `%b` (bytes del **cuerpo**, sin cabeceras) para el tamaño; el de Ubuntu usa `%O` (con cabeceras, y es `0` si no se envió nada), de modo que la cifra no coincide con el `Content-Length` de la respuesta. Si se comparan cifras entre servidores, hay que mirar qué código usa cada uno.

> Los campos `%h`, `%u` y `%{User-Agent}i` son datos de personas: la IP y el nombre de usuario identifican a alguien. Un registro de accesos es, por tanto, un fichero con datos personales, y se trata como tal (apartado 8.12).

### 8.3 El registro de errores y el nivel de detalle

El registro de errores tiene otra forma: no es una línea por petición, sino una línea por **suceso**. Esta es de un intento fallido de contraseña en el sitio `informes` (apartado 5):

```text
[Mon Nov 09 09:21:40.066398 2026] [auth_basic:error] [pid 10493] [client 127.0.0.66:58684] AH01617: user maria: authentication failure for "/": Password Mismatch
```

Se lee de izquierda a derecha: la **hora** con microsegundos; entre corchetes el **módulo y la gravedad** (`auth_basic:error`: el módulo de autenticación Basic avisa de un error); el **proceso** de Apache que lo atendió (`pid`); el **cliente** (IP y puerto de origen); el **código del mensaje** (`AH01617`, que empieza siempre por `AH` y se puede buscar en la documentación de Apache o en internet) y el texto. Este mensaje dice: el usuario `maria` ha fallado la contraseña al pedir `/`. Es el mensaje que ya se vio en el apartado 5.

La directiva `LogLevel` fija la **gravedad mínima** que se registra. Los niveles, de más a menos graves, son `emerg`, `alert`, `crit`, `error`, `warn`, `notice`, `info` y `debug` (y `trace1` a `trace8` para depuración muy detallada). Si se pide `warn`, se escribe `warn` y todo lo más grave; no `notice`, `info` ni `debug`. El valor por defecto de Ubuntu está en `/etc/apache2/apache2.conf`:

```apache
LogLevel warn
```

Se puede subir el detalle **solo para un módulo**, lo que es muy útil: no hace falta inundar el registro con todo para ver un problema de TLS. La forma es `LogLevel nivel-general modulo:nivel`. Por ejemplo, en el sitio de Solvia, antes de cerrar la `</VirtualHost>`:

```apache
LogLevel warn ssl:info
```

Con eso, el módulo `ssl` registra también los mensajes informativos. Hecha una petición correcta y otra en la que el cliente corta porque no se fía del certificado (la que da el error 60 de `curl` del apartado 7.4), el registro `solvia_error.log` muestra:

```text
[Mon Nov 09 09:21:54.415915 2026] [ssl:info] [pid 10606] [client 127.0.0.1:49998] AH01998: Connection closed to child 1 with abortive shutdown (server solvia.test:443)
```

Es un mensaje que a nivel `warn` no se habría visto: el cliente cerró la conexión TLS de forma brusca (*abortive shutdown*). Es una pista típica de «un cliente no acepta el certificado». Cuando se termina de investigar, **se vuelve al nivel normal**: `info` y `debug` generan mucho volumen y pueden incluir datos que no deberían guardarse.

### 8.4 Monitorización en tiempo real

**Monitorizar** es mirar el registro **a la vez que se escribe**, en lugar de abrirlo después. La herramienta básica es `tail -f`:

```bash
sudo tail -f /var/log/apache2/solvia_access.log
```

`tail` muestra el final de un fichero; con `-f` (*follow*) no termina: sigue el fichero y escribe cada línea nueva según se añade. Se corta con `Ctrl+C`. Para ver **solo lo que interesa**, se combina con `grep` mediante una tubería (`|`, que pasa la salida de un comando como entrada del siguiente):

```bash
sudo tail -n0 -f /var/log/apache2/solvia_access.log | grep --line-buffered '" 404 '
```

`-n0` no muestra las líneas anteriores (empieza desde ahora); `grep '" 404 '` deja pasar solo las líneas que contienen ` 404 ` precedido de comillas (el código de estado, que va justo después de la petición entrecomillada, y así no se confunde con un `404` que aparezca en otro campo); y `--line-buffered` obliga a `grep` a escribir cada línea nada más recibirla, porque si no, al ir en una tubería, esperaría a juntar un montón de ellas y la monitorización parecería congelada. Se hace una petición a una página que no existe y, en cuanto se envía, aparece:

```text
127.0.0.1 - - [09/Nov/2026:09:21:49 +0000] "GET /no-existe.php HTTP/1.1" 404 2667 "-" "curl/8.5.0"
```

Dos fuentes más de información en vivo. La primera es la página `/server-status` (que se protegió con contraseña en el apartado 5.7), que muestra las peticiones que Apache está atendiendo **ahora mismo**. La segunda es el registro del **servicio** (no de las peticiones): con `sudo journalctl -u apache2 -f` se ven los arranques, recargas y fallos de arranque de Apache, que es lo que hay que mirar cuando el servidor no se levanta (por ejemplo, tras equivocarse en un certificado).

### 8.5 Analizar un registro con la línea de comandos

Mirar líneas sueltas no sirve cuando hay miles. Con unas pocas herramientas combinadas se obtienen resúmenes. Las que se usan:

- `awk '{print $9}' fichero`: `awk` procesa el fichero línea a línea, partiendo cada una en campos separados por espacios (`$1` es el primero, `$9` el noveno…). Aquí imprime el noveno campo. En el formato `combined`, `$1` es la IP, `$7` la ruta pedida y `$9` el código de estado.
- `sort`: ordena las líneas, lo que deja juntas las repetidas.
- `uniq -c`: cuenta las líneas consecutivas iguales y escribe el número delante. Por eso necesita que antes se haya ordenado con `sort`.
- `sort -rn`: ordena de forma numérica (`-n`) y de mayor a menor (`-r`).
- `head`: muestra solo las primeras líneas.

Las cinco preguntas habituales, sobre un registro del sitio de Solvia con 31 peticiones:

```bash
awk '{print $9}' solvia_access.log | sort | uniq -c | sort -rn
```

```text
     24 200
      7 404
```

*Cuántas peticiones de cada tipo de resultado.* De las 31 peticiones, 24 han ido bien y 7 han dado «no encontrado». Un 404 es normal en pequeñas cantidades; interesa **quién** los provoca.

```bash
cut -d' ' -f1 solvia_access.log | sort | uniq -c | sort -rn
```

```text
     12 127.0.0.11
      9 127.0.0.12
      6 127.0.0.66
      4 127.0.0.1
```

*Qué direcciones han hecho más peticiones.* `cut -d' ' -f1` corta cada línea por los espacios (`-d' '`) y se queda con el primer campo (`-f1`); es una forma alternativa a `awk '{print $1}'`.

```bash
awk '$9 == 404 {print $1}' solvia_access.log | sort | uniq -c | sort -rn
```

```text
      6 127.0.0.66
      1 127.0.0.1
```

*Quién provoca los 404.* `awk '$9 == 404 {print $1}'` imprime la IP **solo** de las líneas cuyo noveno campo es 404. La dirección `127.0.0.66` provoca seis de los siete. Las otras direcciones, que visitan las páginas, casi no fallan.

```bash
awk '$1 == "127.0.0.66" && $9 == 404 {print $7}' solvia_access.log
```

```text
/.env
/wp-login.php
/admin
/phpmyadmin
/config/config.php
/.git/config
```

*Qué buscaba.* La lista dice lo que es: ficheros de configuración con secretos (`.env`, `config/config.php`, `.git/config`) y paneles de administración típicos de otras aplicaciones (`wp-login.php` es de WordPress, `phpmyadmin` de una herramienta de bases de datos). Quien lo hace no busca Solvia: **recorre servidores de internet con una lista fija de rutas conocidas**, a ver si alguno tiene algo expuesto. Se llama *escaneo* y es completamente habitual. Que todas den 404 es buena noticia: no hay nada ahí. En concreto, que `/.env` y `/config/config.php` den 404 prueba que el `DocumentRoot` apunta a `public/` (apartado 4.1), de modo que la configuración, que está fuera, ni se ve.

```bash
awk -F'"' '{print $6}' solvia_access.log | sort | uniq -c | sort -rn
```

```text
     12 Mozilla/5.0 (X11; Linux x86_64; rv:132.0) Gecko/20100101 Firefox/132.0
      9 Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36
      6 zgrab/0.x
      4 curl/8.5.0
```

*Qué programas piden.* Aquí `awk -F'"'` cambia el separador de campos a las comillas, y `$6` es el contenido de la última pareja, el agente de usuario. Sirve para reconocer navegadores reales y herramientas automáticas. Hay que tomarlo como pista y no como prueba: el agente de usuario lo escribe el cliente y puede mentir.

### 8.6 Caso práctico sobre Solvia: ¿qué ha pasado en el servidor?

El ejercicio habitual de un administrador es reconstruir un incidente a partir de los registros. Con lo anterior se puede contar esta historia. Se parte de la dirección sospechosa, `127.0.0.66` (en un servidor real sería una IP pública), y se pregunta qué hizo en cada sitio. En Solvia: seis peticiones a rutas inexistentes (arriba). En `informes`, el sitio con contraseña, se cuentan los intentos fallidos (código 401) por IP y por usuario:

```bash
awk '$9 == 401 {print $1, $3}' informes_access.log | sort | uniq -c
```

```text
      5 127.0.0.66 maria
```

El tercer campo (`$3`) es el nombre de usuario, el `%u` del apartado 8.2: cinco intentos con el usuario `maria`, todos fallidos. Se confirma con el registro de errores, que cuenta los fallos con su motivo:

```bash
grep -c AH01617 informes_error.log
grep AH01617 informes_error.log | head -1
```

```text
5
[Mon Nov 09 09:21:40.066398 2026] [auth_basic:error] [pid 10493] [client 127.0.0.66:58684] AH01617: user maria: authentication failure for "/": Password Mismatch
```

(`grep -c` cuenta las líneas que contienen el texto.) La lectura profesional: una sola IP, en menos de un segundo, ha escaneado rutas sospechosas en Solvia y ha probado cinco contraseñas contra un usuario real de `informes`. Es un ataque de **fuerza bruta** (probar contraseñas hasta acertar). Ninguna petición entró, porque no acertó ninguna contraseña. Lo que se haría con esta información: comprobar que la contraseña de `maria` es fuerte, **bloquear esa IP** (con `Require not ip 127.0.0.66` dentro de un `RequireAll`, apartado 5.6, o en el cortafuegos), y valorar herramientas que lo automaticen. Una de las más conocidas es `fail2ban`, que vigila los registros y bloquea las IP que fallan demasiadas veces; aquí solo se menciona como herramienta que existe, sin configurarla.

Lo importante es el método: **de un dato anómalo (muchos 404 de una IP) a tres comprobaciones (qué buscó, qué más hizo, si tuvo éxito)**, siempre apoyándose en registros y no en suposiciones. El registro de accesos dice *qué se pidió y qué se respondió*; el de errores dice *por qué falló*. Los dos juntos cuentan la historia.

### 8.7 Un formato propio: tiempos de respuesta y sitio

El formato `combined` no dice cuánto tarda el servidor en responder. Para analizar rendimiento se define un formato propio con `LogFormat` (fuera del `VirtualHost`, o en el fichero de configuración global) y se usa en un segundo `CustomLog`. Se añaden estas dos líneas (la primera, al principio de `solvia.conf`; la segunda, dentro del `VirtualHost *:443`):

```apache
LogFormat "%h %t \"%r\" %>s %b %D" tiempos
```

```apache
CustomLog ${APACHE_LOG_DIR}/solvia_tiempos.log tiempos
```

`%D` es el tiempo, en **microsegundos**, que Apache tardó en atender la petición; `%b` son los bytes del cuerpo. Un `CustomLog` más escribe el mismo suceso en otro fichero con otro formato. Con `apache2ctl configtest` y `systemctl reload apache2` queda activo, y una línea del nuevo registro tiene esta forma:

```text
127.0.0.11 [09/Nov/2026:09:21:39 +0000] "GET / HTTP/1.1" 200 188 1801
```

El último número es el tiempo: 1801 microsegundos, es decir, unos 1,8 milisegundos. Con `awk` se obtiene la media (campo `$NF`, que es siempre el último de la línea), y la petición más lenta:

```bash
awk '{s+=$NF; n++} END {printf "%d peticiones, media %.0f us\n", n, s/n}' solvia_tiempos.log
```

```text
30 peticiones, media 465 us
```

(`END {…}` se ejecuta al terminar de leer todas las líneas: `s` acumula la suma de los tiempos, `n` cuenta las líneas, y `printf` imprime con formato.) Las cifras de este entorno de pruebas son muy pequeñas porque las peticiones son triviales y locales; lo que importa es el método: con este registro se puede buscar la petición más lenta, o comparar páginas, antes y después de un cambio.

### 8.8 Rotación de logs: que no llenen el disco

Un registro solo crece. Un servidor con mucho tráfico puede llenar el disco en semanas, y un disco lleno es una caída del servicio (Apache no puede escribir en sus propios ficheros, la base de datos no puede guardar…). La solución es la **rotación**: cada cierto tiempo se «cierra» el fichero actual, se guarda con otro nombre, se empieza uno nuevo y se borran los más antiguos.

En Ubuntu lo hace **logrotate**, un programa que el sistema ejecuta a diario y que lee su configuración de `/etc/logrotate.d/`. La de Apache ya viene instalada con el paquete:

```bash
cat /etc/logrotate.d/apache2
```

```text
/var/log/apache2/*.log {
	daily
	missingok
	rotate 14
	compress
	delaycompress
	notifempty
	create 640 root adm
	sharedscripts
	prerotate
		if [ -d /etc/logrotate.d/httpd-prerotate ]; then
			run-parts /etc/logrotate.d/httpd-prerotate
		fi
	endscript
	postrotate
		if pgrep -f ^/usr/sbin/apache2 > /dev/null; then
			invoke-rc.d apache2 reload 2>&1 | logger -t apache2.logrotate
		fi
	endscript
}
```

La primera línea es **a qué ficheros se aplica**: todos los `*.log` de `/var/log/apache2/`, de modo que **los registros de los sitios nuevos (`solvia_access.log`, `informes_error.log`…) se rotan sin hacer nada más**. Cada directiva:

| Directiva | Qué hace |
|---|---|
| `daily` | Rota una vez al día (alternativas: `weekly`, `monthly`) |
| `rotate 14` | Conserva 14 ficheros rotados; al rotar el 15.º, borra el más antiguo. Con `daily`, son dos semanas de historia |
| `compress` | Comprime con `gzip` los rotados (ocupan mucho menos, y son texto repetitivo) |
| `delaycompress` | No comprime el más reciente (`.1`) hasta la siguiente rotación, para poder leerlo sin descomprimir y por si el servidor aún escribe en él |
| `missingok` | Si falta el fichero, no da error |
| `notifempty` | No rota un fichero vacío |
| `create 640 root adm` | Crea el fichero nuevo vacío con permisos `640`, propietario `root` y grupo `adm` |
| `sharedscripts` | Ejecuta `prerotate` y `postrotate` una sola vez para todos los ficheros del patrón, y no una vez por fichero |
| `postrotate … reload` | Después de rotar, **recarga Apache** |

Esta última directiva es la más importante, y su razón se puede comprobar a mano. Rotar es **cambiar el nombre** del fichero (`solvia_access.log` pasa a ser `solvia_access.log.1`). Apache, sin embargo, tiene el fichero **abierto**, y un fichero abierto sigue siendo el mismo aunque cambie de nombre: si no se le avisa, Apache continúa escribiendo en el fichero renombrado. Se simula con un renombrado manual y una petición:

```bash
cd /var/log/apache2
sudo mv solvia_access.log solvia_access.log.prueba
curl -s --cacert ca-aula.crt -o /dev/null https://solvia.test/
ls -l solvia_access.log*
```

```text
-rw-r--r-- 1 root root 4581 Nov  9 10:22 solvia_access.log.1
-rw-r--r-- 1 root root  172 Nov  9 10:24 solvia_access.log.prueba
```

(`ls -l solvia_access.log*` lista todos los ficheros cuyo nombre empieza así; el `.1` es un fichero rotado de antes.) El fichero `solvia_access.log` **ya no existe** y la petición que se acaba de hacer se ha escrito en el renombrado (`.prueba`), que sigue creciendo: Apache lo tiene abierto. Se corrige pidiéndole a Apache que reabra sus registros, con una recarga «amable» (`graceful`: los procesos de Apache terminan las peticiones en curso, reabren sus ficheros y siguen, sin cortar a nadie):

```bash
sudo apache2ctl graceful
ls -l solvia_access.log*
curl -s --cacert ca-aula.crt -o /dev/null https://solvia.test/
ls -l solvia_access.log*
```

```text
-rw-r--r-- 1 root root    0 Nov  9 10:24 solvia_access.log
-rw-r--r-- 1 root root 4581 Nov  9 10:22 solvia_access.log.1
-rw-r--r-- 1 root root  172 Nov  9 10:24 solvia_access.log.prueba
-rw-r--r-- 1 root root   86 Nov  9 10:24 solvia_access.log
-rw-r--r-- 1 root root 4581 Nov  9 10:22 solvia_access.log.1
-rw-r--r-- 1 root root  172 Nov  9 10:24 solvia_access.log.prueba
```

Tras la recarga, Apache **crea un `solvia_access.log` nuevo** (en la primera lista, con 0 bytes) y la siguiente petición se escribe ahí (86 bytes en la segunda lista); `.prueba` deja de crecer. Es exactamente lo que hace la directiva `postrotate` de logrotate después de cada rotación, y por eso está ahí. Si la recarga falta, el síntoma es un registro vacío y otro, ya «antiguo», que crece y se comprime o se borra con datos nuevos dentro.

Se puede ver también una rotación real, forzándola con `-f` (aunque no toque todavía):

```bash
sudo logrotate -f /etc/logrotate.d/apache2
ls -l /var/log/apache2/solvia_access.log*
```

```text
-rw-r----- 1 root adm    0 Nov  9 10:22 /var/log/apache2/solvia_access.log
-rw-r--r-- 1 root root 4495 Nov  9 10:21 /var/log/apache2/solvia_access.log.1
```

(Los tamaños y horas dependen de cada servidor; lo importante son los **permisos**.) Fíjate en ellos: el fichero **nuevo**, creado por logrotate, tiene `640 root adm` (la directiva `create`), pero el **rotado** (`.1`), que creó Apache, conserva `644`, es decir, legible por cualquier usuario del sistema; y los registros de los sitios (`solvia_access.log`, `informes_access.log`…), mientras no se rotan por primera vez, son también `644`, como muestran las listas anteriores. Contienen IP y nombres de usuario. Los registros que Ubuntu crea por su cuenta (`error.log`, `access.log`, `other_vhosts_access.log`) ya nacen con `640` y grupo `adm`. Para que los de un sitio nuevo nazcan igual, se crean de antemano: `sudo touch fichero`, `sudo chown root:adm fichero` y `sudo chmod 640 fichero`.

Antes de tocar nada, la opción `-d` (*debug*) hace un ensayo: muestra qué haría `logrotate` sin hacerlo.

```bash
sudo logrotate -d /etc/logrotate.d/apache2
```

La salida lista, fichero a fichero (`considering log …`), si necesita rotarse y por qué. Es la forma segura de comprobar una configuración nueva.

### 8.9 Consolidación: reunir los registros

**Consolidar** es juntar los registros de varias fuentes en un solo sitio, para poder buscar y comparar con una sola consulta en lugar de abrir diez ficheros. Hay dos niveles.

**Varios sitios virtuales en un mismo servidor.** Hasta ahora cada sitio escribe su propio registro (`solvia_access.log`, `informes_access.log`). Para tener también una vista conjunta se añade a cada sitio un segundo `CustomLog` con el formato `vhost_combined`, que Ubuntu ya trae definido y que empieza por `%v:%p`, el nombre del sitio y el puerto:

```apache
LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
```

Ubuntu ya usa este formato en el fichero `/etc/apache2/conf-enabled/other-vhosts-access-log.conf`, con una línea que dice que escriba en `other_vhosts_access.log` **los accesos de los sitios que no tienen su propio `CustomLog`**. Por eso el sitio por defecto de `00-defecto.conf`, que no define ninguno, deja ahí sus líneas, que empiezan por el nombre del sitio y el puerto:

```text
defecto.test:80 127.0.0.1 - - [09/Nov/2026:09:53:57 +0100] "GET / HTTP/1.1" 403 435 "-" "curl/8.5.0"
```

Si se quisiera una vista conjunta de todos los sitios, bastaría con añadir en cada uno `CustomLog ${APACHE_LOG_DIR}/todos_access.log vhost_combined`. Con el nombre del sitio al principio de cada línea, `grep '^solvia.test'` o `awk '{print $1}'` separan el tráfico por sitio.

**Varios servidores.** Cuando hay muchos servidores (o contenedores, en una unidad posterior), nadie se conecta a cada uno a leer ficheros. Se instala en cada máquina un agente que **envía** sus registros a un servidor central, donde se almacenan e indexan para buscar en todos a la vez. El mecanismo clásico de Linux es **rsyslog**, que puede reenviar registros por red; y las herramientas dedicadas conocidas son la pila **Elastic** (Elasticsearch para almacenar y buscar, Kibana para ver, y agentes como Filebeat para enviar), **Graylog** y **Grafana Loki**. Estas herramientas existen y se usan mucho en producción, pero **no se instalan en esta unidad**: la práctica se hace con un solo servidor y GoAccess. Para trabajar con ellas habría que consultar su documentación actual, porque sus versiones y su configuración cambian con frecuencia.

### 8.10 GoAccess: un análisis visual y en tiempo real

**GoAccess** es un analizador de registros de acceso de servidores web que muestra un panel de resultados (visitantes, páginas más pedidas, códigos de estado, navegadores, horas…) a partir del propio fichero. Se instala del repositorio de Ubuntu:

```bash
sudo apt install goaccess
goaccess --version
```

```text
GoAccess - 1.8.1.
```

GoAccess tiene que saber **qué formato** tiene el registro, para saber qué es cada campo. Para el formato `combined` de Apache existe un nombre predefinido, `COMBINED`:

```bash
goaccess solvia_access.log --log-format=COMBINED --no-progress -o informe.html
```

`--log-format=COMBINED` le dice el formato; `-o informe.html` pide la salida en HTML (la extensión elige el tipo de fichero); y `--no-progress` oculta la barra de progreso. El resultado es **una página web de un solo fichero** que se abre con el navegador, con tablas y gráficas de todo lo anterior. Si se pide la extensión `.json`, la salida es un fichero con los mismos datos en formato estructurado, útil para extraer cifras concretas con `jq`, un programa que consulta ficheros JSON:

```bash
goaccess solvia_access.log --log-format=COMBINED --no-progress -o informe.json
jq -c '.general | {total_requests, valid_requests, failed_requests, unique_visitors, bandwidth}' informe.json
jq -c '.status_codes.data[] | {data, hits: .hits.count}' informe.json
jq -c '.hosts.data[] | {data, hits: .hits.count}' informe.json
```

```text
{"total_requests":31,"valid_requests":31,"failed_requests":0,"unique_visitors":3,"bandwidth":79394}
{"data":"2xx Success","hits":24}
{"data":"4xx Client Errors","hits":7}
{"data":"127.0.0.11","hits":12}
{"data":"127.0.0.12","hits":9}
{"data":"127.0.0.66","hits":6}
{"data":"127.0.0.1","hits":4}
```

Coinciden con lo que se calculó a mano en el apartado 8.5 (24 correctas, 7 con error 404, las mismas direcciones con las mismas cifras): **GoAccess no inventa nada, hace lo mismo que `awk | sort | uniq`, pero ya hecho y presentado**. Los «visitantes únicos» (3) los calcula GoAccess con su propio criterio —agrupa por dirección, día y navegador—, así que no tiene por qué coincidir con el número de direcciones distintas (4); la definición exacta está en su documentación. Si `failed_requests` fuera distinto de cero, indicaría líneas que GoAccess no ha podido interpretar con el formato elegido, y la primera cosa que se revisa es `--log-format`.

**Tiempo real.** La opción `--real-time-html` hace que GoAccess genere el HTML y, además, abra un servidor propio que **empuja los datos nuevos al navegador** según se escriben en el registro, por una conexión llamada *WebSocket*. La página se actualiza sola, sin recargar:

```bash
goaccess solvia_access.log --log-format=COMBINED --real-time-html -o informe.html --port=7890
```

```text
WebSocket server ready to accept new client connections
```

GoAccess se queda en ejecución (se corta con `Ctrl+C`); mientras tanto, el navegador que tenga abierto `informe.html` recibe los datos nuevos por el puerto 7890 (`--port`). Es el sentido de «análisis en tiempo real» del criterio. Hay una variante en el terminal, sin HTML: `tail -f solvia_access.log | goaccess - --log-format=COMBINED` (el guion `-` significa «lee de la entrada estándar») muestra un panel en la propia consola que se va actualizando (no se muestra aquí su salida, porque es interactiva y no se ha probado).

Dos precauciones. La primera: el navegador tiene que **poder llegar** al puerto del WebSocket del equipo donde corre GoAccess; si GoAccess está en una máquina virtual y el navegador en el equipo anfitrión, hace falta que el puerto sea accesible (por ejemplo con el reenvío de puertos de VirtualBox) y, si la dirección que GoAccess incrusta en la página no es alcanzable, se corrige con la opción `--ws-url`. La segunda: ese servicio **no lleva autenticación ni cifrado por sí solo**, así que se abre solo mientras se mira, nunca se deja expuesto a una red que no sea de confianza, y los datos que muestra (IP, rutas, usuarios) son los del registro. La opción `--addr` permite indicar en qué dirección escucha; conviene consultar la documentación de GoAccess para el valor por defecto antes de usarla en un servidor real.

> ⚖️ **Alternativa y criterio.** Hay tres niveles de herramienta, y la elección depende del tamaño del problema. **`grep`, `awk`, `sort` y `uniq`** no necesitan instalar nada, funcionan en cualquier servidor y son lo mejor para una pregunta concreta y puntual («¿cuántos 404 ha dado esta IP?»); pero cada pregunta nueva es otro comando y no dan vista de conjunto. **GoAccess** es lo mejor cuando se quiere una visión general rápida de uno o pocos ficheros, sin infraestructura: un único programa, ligero, que lee el fichero tal cual. **Una pila centralizada (Elastic, Graylog, Loki)** compensa cuando hay muchos servidores, mucho volumen, necesidad de buscar a la vez en todos o de guardar el histórico durante mucho tiempo, o de crear alertas; a cambio, hay que instalarla, mantenerla, dimensionarla y asegurarla, y es mucha más máquina de la que necesita un servidor con un puñado de sitios. Para un servidor como el de Solvia, GoAccess es la herramienta proporcionada; la pila centralizada sería excesiva.

### 8.11 ⚠️ Errores comunes

- **No recargar tras rotar**: el síntoma es un registro nuevo vacío y otro «antiguo» que sigue creciendo (apartado 8.8). Se corrige con `postrotate` o un `apache2ctl graceful`.
- **Mirar solo el registro de accesos**: un 500 en el de accesos no dice por qué; el motivo está en el de errores. Se miran siempre los dos.
- **Dejar `LogLevel debug` activado**: llena el disco y registra información delicada. Se sube solo para el módulo que se investiga y se baja al terminar.
- **`grep` sin `--line-buffered` en una tubería con `tail -f`**: parece que no pasa nada, porque `grep` acumula las líneas. Se arregla añadiendo la opción.
- **`uniq -c` sin `sort` antes**: `uniq` solo junta líneas **consecutivas** iguales, así que el recuento sale fragmentado. Siempre `sort | uniq -c`.
- **Contar con el campo equivocado**: si el formato cambia (por ejemplo, con un `LogFormat` propio), `$9` deja de ser el código de estado. Antes de usar `awk`, se mira una línea y se cuentan los campos.
- **Confiar en el agente de usuario**: lo escribe el cliente. Sirve de pista, no de prueba.
- **Exponer el panel en tiempo real de GoAccess** en una red no confiable (apartado 8.10).

### 8.12 💡 Nota técnica

Un registro de accesos contiene **datos personales**: la dirección IP de cada visitante y, con autenticación, su nombre de usuario. En la Unión Europea, el criterio habitual es que una dirección IP se considera dato personal a efectos del RGPD (conviene consultar la normativa y la política de la organización, que decide la base legal y el plazo). En la práctica esto se traduce en cuatro hábitos: limitar quién puede leer los registros (permisos `640`, grupo `adm`, apartado 8.8), **conservarlos solo el tiempo necesario** (de ahí el `rotate 14`: no son infinitos), no copiarlos a sitios sin protección y, si se publican capturas o ejemplos como los de estos apuntes, usar direcciones de prueba y no reales. Los registros también son una prueba en un incidente: por eso se protegen para que no los modifique quien ha entrado en el servidor, y en producción se envían de inmediato a otra máquina (apartado 8.9), de modo que un atacante que comprometa el servidor no pueda borrar sus huellas.

### 8.13 Comprueba lo que has entendido

**1. En el registro de accesos hay muchas líneas con el código `404` de una misma dirección, que pide `/.env`, `/wp-login.php` y `/phpmyadmin`. ¿Qué indica y qué dato del registro de accesos demuestra que no ha tenido éxito?**

Indica un escaneo: una herramienta que prueba una lista fija de rutas conocidas en muchos servidores, sin interés particular en Solvia. Que no ha tenido éxito lo demuestra el código de estado: `404` quiere decir que la ruta no existe, así que no se ha servido nada. (Si alguna hubiera dado `200`, habría que mirar qué se entregó.)

**2. ¿Por qué hace falta `sort` antes de `uniq -c`, y qué comando cuenta cuántas peticiones ha hecho cada IP?**

`uniq` solo cuenta líneas **consecutivas** iguales; `sort` las deja juntas. El comando es `cut -d' ' -f1 solvia_access.log | sort | uniq -c | sort -rn`, que corta el primer campo (la IP), ordena, cuenta y ordena de mayor a menor.

**3. Rotas los registros con `mv solvia_access.log solvia_access.log.1` y creas uno nuevo vacío, pero el nuevo sigue a cero bytes y el antiguo crece. ¿Por qué y cómo se arregla?**

Apache tiene abierto el fichero original y lo sigue escribiendo aunque se renombre. Hay que decirle que reabra sus registros, con `sudo apache2ctl graceful` (o `systemctl reload apache2`). La directiva `postrotate` de logrotate lo hace automáticamente.

**4. ¿Qué diferencia hay entre el registro de accesos y el de errores, y cuál usarías para investigar por qué un usuario ve un error 500?**

El de accesos tiene una línea por petición con su resultado (aquí se vería el `500`, pero no el motivo). El de errores contiene los mensajes internos del servidor y de los módulos, con la causa. Se empieza por el de accesos para localizar la petición (hora, ruta, IP) y se pasa al de errores, en esa misma hora, para ver el motivo.

**5. Quieres ver en directo solo las peticiones que dan error 404 mientras pruebas. ¿Qué comando usas y para qué sirve cada parte?**

`sudo tail -n0 -f /var/log/apache2/solvia_access.log | grep --line-buffered '" 404 '`. `tail -n0 -f` sigue el fichero mostrando solo lo nuevo; la tubería pasa cada línea a `grep`; `grep` deja pasar solo las que contienen el 404 como código de estado; y `--line-buffered` hace que las muestre al momento y no las acumule.

---

## 9. Documentación, administración segura y recomendaciones de uso

**CE 2.g** — Se ha elaborado documentación relativa a la configuración, administración segura y recomendaciones de uso del servidor.

Un servidor bien configurado que solo entiende quien lo montó es un problema en espera: el día que esa persona se ausenta, se va o simplemente olvida por qué cambió algo seis meses atrás, nadie sabe qué se puede tocar. La **documentación** es la parte del trabajo que convierte una configuración en algo que otra persona (o uno mismo, dentro de un año) puede mantener. Este bloque cierra la unidad recogiendo lo que se ha hecho en los anteriores en tres documentos con tres destinatarios distintos: quien **configura** el servidor, quien lo **administra** y quien lo **usa**.

### 9.1 Qué hay que documentar, y para quién

No existe «la documentación del servidor» como un único texto. Hay al menos tres, porque responden a preguntas distintas:

| Documento | Responde a | Lo lee | Contiene |
|---|---|---|---|
| **Configuración** | «¿Cómo está montado y por qué?» | Quien tiene que mantenerlo o reconstruirlo | Versiones, rutas, sitios, módulos, certificados y, sobre todo, las **decisiones** con su motivo |
| **Administración segura** | «¿Qué hay que hacer, y cada cuánto, para que siga siendo seguro?» | Quien lo administra día a día | Tareas periódicas, procedimiento de cambios, copias de seguridad, qué hacer ante un incidente |
| **Recomendaciones de uso** | «¿Cómo uso este servidor sin romperlo ni exponerlo?» | Quien despliega aplicaciones en él (desarrolladores) y quien lo usa | Dónde se sube cada cosa, qué no se debe hacer, a quién se avisa |

Mezclarlos es el error habitual: un único documento largo donde el desarrollador que solo quiere saber dónde subir un fichero tiene que atravesar los parámetros del módulo SSL. Cada documento es corto y está escrito para su lector.

Una regla común a los tres: **la documentación describe cómo está el servidor y por qué, no repite lo que ya dice el propio fichero de configuración**. Copiar un `solvia.conf` entero dentro de un documento lo deja desactualizado el primer día que alguien lo cambie. Se enlaza o se nombra el fichero, y se escribe lo que el fichero no puede decir: el motivo.

### 9.2 Documentar la configuración

La documentación de configuración tiene dos partes. La primera es el **inventario**: los datos objetivos del servidor, que se pueden obtener con comandos (y por eso se pueden comprobar, en lugar de escribirlos de memoria). La segunda son las **decisiones**.

**El inventario.** Estos son los comandos que lo rellenan; los que se conocen de bloques anteriores, y uno nuevo (`lsb_release`) que muestra la versión de Ubuntu:

```bash
lsb_release -ds
apache2ctl -V | sed -n '1p;/MPM/p'
php -v | head -1
a2query -s
a2query -m | wc -l
openssl x509 -in /etc/apache2/ssl/solvia.crt -noout -subject -issuer -enddate
```

```text
Ubuntu 24.04.5 LTS
Server version: Apache/2.4.58 (Ubuntu)
Server MPM:     prefork
PHP 8.3.6 (cli) (built: Oct 19 2026 15:16:58) (NTS)
informes (enabled by site administrator)
solvia (enabled by site administrator)
00-defecto (enabled by site administrator)
23
subject=CN = solvia.test
issuer=O = IES Rio Arba, CN = CA del aula
notAfter=Nov  9 09:06:34 2027 GMT
```

Cada orden contesta una pregunta del inventario: qué sistema (`lsb_release -ds`), qué versión de Apache y con qué modelo de procesos (`apache2ctl -V`, filtrado con `sed -n '1p;/MPM/p'` para quedarse con la primera línea y la que contiene «MPM»), qué PHP (`php -v`), qué sitios están activos (`a2query -s`), cuántos módulos hay cargados (`a2query -m | wc -l`, donde `wc -l` cuenta líneas) y de quién es el certificado y cuándo caduca. El inventario incluye siempre la **fecha de caducidad del certificado**: es el dato que más a menudo se olvida y cuyo olvido tumba un sitio un día concreto, sin aviso (apartado 6.9).

**Las decisiones.** Es lo que no se puede obtener con ningún comando, y por eso es lo más valioso. Para cada decisión importante se anota lo mismo, en una fila:

| Decisión | Qué se hizo | Alternativa descartada | Motivo | Cómo comprobarlo | Cómo deshacerlo |
|---|---|---|---|---|---|
| Servir solo `public/` | `DocumentRoot /var/www/solvia/public` | `DocumentRoot /var/www/solvia` | Que `config/` (con la contraseña de la base de datos) y `src/` no sean accesibles desde la red | `curl -sI https://solvia.test/config/config.php` da 404 | Cambiar el `DocumentRoot` (no se recomienda) |
| Contraseña en `informes` | `AuthType Basic` + `Require valid-user`, solo por HTTPS | `mod_auth_digest` | Digest no aporta ventaja de seguridad real frente a Basic sobre HTTPS (lo dice la propia documentación de Apache, apartado 5.8) | Sin credenciales, `401`; con ellas, `200` | Quitar las directivas `Auth*` del `<Directory>` |
| Certificado de la CA del aula | Certificado firmado por una CA propia, instalado en `/etc/apache2/ssl/` | Autofirmado; Let's Encrypt | Una CA propia permite varios nombres con una sola confianza instalada; Let's Encrypt exige un dominio público (apartado 6.8) | `openssl s_client … Verification: OK` | Sustituir los dos ficheros y recargar |
| HSTS con `max-age=300` | Cabecera en el sitio del 443 | `max-age` de un año | Un error de certificado con HSTS largo deja a los usuarios sin acceso (apartado 7.9) | `curl -sI` muestra la cabecera | Quitar la línea `Header … Strict-Transport-Security` |

Observa que la columna «Cómo comprobarlo» es un **comando o una observación concreta**, no una opinión («queda bien»). Una decisión documentada sin forma de comprobarla no se puede auditar. Y que la columna «Cómo deshacerlo» obliga a pensar, al hacerlo, que habrá que revertirlo.


### 9.3 Administración segura

Administrar de forma segura es sobre todo **tener hábitos**: lo que se hace siempre, de la misma manera, para que un error no se convierta en una caída ni en una brecha. Lo que sigue son los hábitos mínimos, cada uno con el motivo y la comprobación. Varios son aplicaciones de lo visto en bloques anteriores.

**1. Mínimo privilegio.** Cada pieza tiene solo los permisos que necesita. En el servidor de Solvia: la aplicación se sirve desde `public/` (4.1), los ficheros ocultos no se sirven (4.1), el código y la configuración se leen pero no se escriben desde la web (4.2), el fichero de contraseñas de `informes` es `640 root:www-data` (5.4) y la clave privada es `600 root:root` (6.7). La comprobación periódica es mirar los permisos:

```bash
stat -c '%a %U:%G %n' /etc/apache2/ssl/solvia.key /etc/apache2/.htpasswd-informes
```

```text
600 root:root /etc/apache2/ssl/solvia.key
640 root:www-data /etc/apache2/.htpasswd-informes
```

(`stat -c` imprime los datos de un fichero con el formato indicado: `%a` permisos en octal, `%U:%G` propietario y grupo, `%n` nombre.)

**2. Superficie mínima.** Cada módulo activo, cada sitio y cada puerto abierto es algo que puede tener un fallo. Se activa solo lo que se usa (apartado 2) y se deja constancia en la ficha de por qué está cada cosa. Ubuntu ya trae una configuración razonable en `/etc/apache2/conf-enabled/security.conf`: oculta versión y firma, y desactiva el método `TRACE`. Se comprueba qué valores tiene:

```bash
grep -v -E '^\s*#|^\s*$' /etc/apache2/conf-enabled/security.conf
```

```text
ServerTokens Prod
ServerSignature Off
TraceEnable Off
```

(`grep -v -E '^\s*#|^\s*$'` descarta, `-v`, las líneas que empiezan por `#` —comentarios— y las vacías, para ver solo las directivas activas. Los valores `ServerTokens Prod` y `ServerSignature Off` son los del apartado 1.5; `TraceEnable Off` desactiva el método `TRACE` de HTTP, que no tiene uso en una aplicación normal y puede ayudar a ciertos ataques.)

**3. Mantener el software actualizado.** Los fallos de seguridad de Apache, PHP y OpenSSL se corrigen con actualizaciones del sistema. Con el gestor de paquetes:

```bash
sudo apt update
apt list --upgradable
```

`apt update` descarga la lista de versiones disponibles; `apt list --upgradable` muestra qué paquetes tienen una versión más nueva (y `sudo apt upgrade` las instala). En Ubuntu, las actualizaciones **de seguridad** pueden aplicarse automáticamente con el paquete `unattended-upgrades`; que sea una opción es una decisión de la organización (aplicarlas solas evita olvidos, pero puede reiniciar un servicio en un mal momento), y por eso se anota en la ficha. Tras actualizar Apache, se comprueba que el servicio está activo y que los sitios responden.

**4. Un procedimiento de cambios.** Casi todas las caídas de un servidor web las causa la propia persona que lo administra, al cambiar algo. Un procedimiento de cuatro pasos las evita:

1. **Copia de seguridad** del fichero que se va a tocar, conservando permisos: `sudo cp -a solvia.conf solvia.conf.antes` (`-a` conserva propietario, permisos y fechas).
2. **Cambio**, con anotación de qué y por qué.
3. **Comprobación de sintaxis** antes de aplicar: `sudo apache2ctl configtest`. Si no dice `Syntax OK`, no se aplica.
4. **Aplicar y verificar**: `sudo systemctl reload apache2` y una petición real; si falla, se restaura la copia.

La diferencia entre la copia y el fichero nuevo se ve con `diff -u`, que marca con `-` lo que se quita y con `+` lo que se añade. Por ejemplo, subir el HSTS de 300 a 86400 segundos:

```bash
sudo diff -u solvia.conf.antes solvia.conf
```

```text
--- solvia.conf.antes	2026-11-09 10:22:01.538865675 +0100
+++ solvia.conf	2026-11-09 10:25:59.909380963 +0100
@@ -22,7 +22,7 @@
         Header always set X-Content-Type-Options "nosniff"
         Header always set Referrer-Policy "strict-origin-when-cross-origin"
         Header always set Content-Security-Policy "default-src 'self'; style-src 'self' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; script-src 'self'; img-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'"
-        Header always set Strict-Transport-Security "max-age=300"
+        Header always set Strict-Transport-Security "max-age=86400"
 
         ErrorDocument 404 /errores/404.html
         ErrorLog ${APACHE_LOG_DIR}/solvia_error.log
```

Esta salida es exactamente lo que se pega en la bitácora de cambios: **una línea quitada, una añadida y la fecha**. (Se muestra el contexto de tres líneas arriba y abajo que `diff -u` añade por defecto.)

**5. Copia de seguridad de la configuración.** Todo el ajuste del servidor está en `/etc/apache2/`. Se guarda en un único fichero comprimido y se **comprueba que se puede restaurar**: una copia que no se ha probado no es una copia, es una esperanza.

```bash
sudo tar -czpf /root/apache2-conf-2026-11-09.tar.gz -C /etc apache2
sudo tar -tzf /root/apache2-conf-2026-11-09.tar.gz | grep -E 'sites-available/|ssl/|htpasswd'
```

```text
apache2/.htpasswd-informes
apache2/sites-available/
apache2/sites-available/informes.conf
apache2/sites-available/000-default.conf
apache2/sites-available/solvia.conf
apache2/sites-available/00-defecto.conf
apache2/sites-available/default-ssl.conf
apache2/ssl/
apache2/ssl/solvia.crt
apache2/ssl/solvia.key
```

`tar` empaqueta: `-c` crea, `-z` comprime con gzip, `-p` conserva permisos, `-f` indica el nombre del fichero, y `-C /etc apache2` dice «ve a `/etc` y empaqueta la carpeta `apache2`». Con `-t` (en lugar de `-c`) se lista el contenido sin extraerlo, que es la comprobación de que la copia contiene lo esperado. **La copia incluye la clave privada y el fichero de contraseñas** (se ve en la lista), así que es tan delicada como ellos: se guarda con permisos `600` y propietario `root`, y fuera del servidor, en un lugar al que no tenga acceso quien pueda comprometerlo. Para probar la restauración sin tocar el servidor, se extrae en una carpeta temporal y se compara con el original: `diff -r /etc/apache2 carpeta/apache2` no muestra nada si son idénticos. Los datos de la aplicación (la base de datos MariaDB, los ficheros que suban los usuarios) tienen su propia copia, que es otro procedimiento y otro documento.

**6. Cortafuegos.** Un servidor web solo necesita abiertos los puertos 80 y 443 (más el de administración remota, el 22 de SSH, si se administra por red). Ubuntu incluye `ufw`, un cortafuegos sencillo, y el paquete de Apache trae perfiles ya hechos: `Apache` (el 80), `Apache Secure` (el 443) y `Apache Full` (los dos). Se muestran con:

```bash
cat /etc/ufw/applications.d/apache2-utils.ufw.profile
```

```text
[Apache]
title=Web Server
description=Apache v2 is the next generation of the omnipresent Apache web server.
ports=80/tcp

[Apache Secure]
title=Web Server (HTTPS)
description=Apache v2 is the next generation of the omnipresent Apache web server.
ports=443/tcp

[Apache Full]
title=Web Server (HTTP,HTTPS)
description=Apache v2 is the next generation of the omnipresent Apache web server.
ports=80,443/tcp
```

La configuración del cortafuegos no se practica en esta unidad. Se deja constancia de que existe como medida y de cómo se aplicaría (`sudo ufw allow 'Apache Full'`, y `sudo ufw enable` **después** de haber permitido el acceso por SSH, porque activar el cortafuegos sin haberlo hecho corta la conexión con la propia máquina cuando se administra por red). No se ha probado en este entorno, así que antes de usarlo en un servidor real conviene consultar la documentación de `ufw`.

**7. Vigilancia.** Los registros solo sirven si alguien los mira (apartado 8). Se decide cada cuánto y con qué comprobaciones: por ejemplo, semanalmente, revisar el número de 5xx, las IP con muchos 404 o 401, y que el certificado no esté cerca de caducar.

**8. Saber qué hacer ante un incidente.** El peor momento para decidir qué hacer es durante el incidente. Un guion mínimo, escrito de antemano: **contener** (bloquear la IP, desactivar el sitio con `a2dissite` y recargar), **conservar las pruebas** (copiar los registros antes de que se rote su contenido), **investigar** (registros de accesos y errores, apartado 8.6), **recuperar** (restaurar la copia de seguridad, cambiar las contraseñas y claves que puedan estar comprometidas) y **anotar** qué pasó y qué se cambia para que no se repita.

### 9.4 Recomendaciones de uso

El tercer documento es para quien **despliega aplicaciones o usa el servidor** sin administrarlo. Debe poder leerse en cinco minutos y responder a lo que esa persona necesita saber. Para el servidor de Solvia, un contenido razonable es:

- **Dónde se sube cada cosa.** Solo lo que debe verse desde la red (páginas, estilos, scripts, imágenes) va en `public/`; el código de la aplicación, la configuración y los secretos van fuera, en `src/`, `config/`, etc. Nada que contenga una contraseña se deja dentro de `public/`.
- **Siempre HTTPS.** Las direcciones se escriben con `https://`; el puerto 80 solo redirige. Un formulario o un enlace con `http://` en una página HTTPS es contenido mixto y el navegador lo bloquea.
- **Los recursos externos están restringidos.** La política de seguridad de contenidos (apartado 7.8) solo permite cargar scripts y estilos del propio sitio y las tipografías de Google Fonts. Si una página nueva necesita cargar otra cosa (un script, una imagen de otro sitio), **se pide** que se añada a la política; no se pega código en línea en el HTML, porque la política lo bloquea.
- **Errores.** Si la aplicación falla, el usuario ve una página de error genérica: los detalles no se muestran en pantalla (apartado 4.5) y se consultan en el registro de errores. Para investigar, quien tenga acceso mira `solvia_error.log`.
- **Qué no se hace.** No se cambian permisos «para que funcione» (un `chmod 777` abre la puerta a cualquiera); no se activa `display_errors`; no se sube un `.env`, una copia de seguridad ni un volcado de la base de datos a una carpeta pública.
- **A quién se pide un cambio y cómo.** Un cambio en el servidor lo hace la persona que lo administra, con el procedimiento del apartado 9.3, apuntándolo en el registro de cambios. Quien despliega no edita `/etc/apache2/`.
- **Contraseñas.** Las de los usuarios de `informes` se asignan y cambian por quien administra el servidor (`htpasswd`), no se comparten ni se envían por correo o mensajería.

### 9.5 La ficha del servidor de Solvia

Con las tres piezas se escribe un único documento corto por cada lector. Este es un ejemplo del primero, la **ficha de configuración**, hecho con los datos reales de los bloques anteriores. Es un documento de muestra, para tener un modelo; los valores concretos (fechas, versiones exactas) se sustituyen por los del servidor de cada uno, rellenados con los comandos del apartado 9.2.

```plaintext
FICHA DE CONFIGURACIÓN — Servidor web de Solvia
Última revisión: 9 de noviembre de 2026   Responsable: (nombre)

1. INVENTARIO (obtenido con los comandos de la sección 9.2)
   Sistema ........ Ubuntu 24.04 LTS
   Servidor web ... Apache 2.4.58, MPM prefork, PHP 8.3 como módulo (mod_php)
   Base de datos .. MariaDB 10.11 (instalada en la misma máquina)
   Sitios activos . solvia.test (aplicación), informes.solvia.test (informes con contraseña),
                    00-defecto (sitio por defecto: 403 a todo lo que no sea un sitio conocido)
   Puertos ........ 80 (solo redirige a HTTPS), 443 (HTTPS)
   Módulos activados por el administrador: ssl, headers (el resto, los de serie)

2. FICHEROS Y UBICACIONES
   Aplicación ..... /var/www/solvia/ (se sirve solo public/)
   Sitios Apache .. /etc/apache2/sites-available/ (solvia.conf, informes.conf, 00-defecto.conf)
   Certificados ... /etc/apache2/ssl/ (solvia.crt público; solvia.key privada, 600 root)
   Contraseñas .... /etc/apache2/.htpasswd-informes (640 root:www-data)
   Registros ...... /var/log/apache2/ (rotación diaria, 14 copias, con logrotate)

3. CERTIFICADO
   Emitido por .... CA del aula (CA propia, de prácticas)
   Nombres ........ solvia.test, informes.solvia.test
   Caduca ......... (fecha: poner la salida de openssl x509 -enddate)   ← REVISAR 30 DÍAS ANTES

4. DECISIONES (tabla del apartado 9.2: qué, alternativa descartada, motivo, cómo comprobar, cómo deshacer)
   D1 Servir solo public/
   D2 Basic sobre HTTPS para informes
   D3 Certificado de la CA del aula
   D4 HSTS con max-age corto (300 s)
   D5 CSP a medida de Solvia

5. REGISTRO DE CAMBIOS
   2026-11-09  HSTS max-age 300 → 86400 (probado: curl -sI muestra la nueva cabecera)
   (fecha)     (qué se cambió, por qué y cómo se comprobó)
```

Esto cabe en una página, y quien lo lee puede reconstruir el servidor y sabe qué comprobar y cuándo. Ese es el objetivo. Se evalúa con una prueba sencilla: **dárselo a otra persona** y pedirle que responda a tres preguntas sin usar nada más: ¿qué sitios sirve este servidor?, ¿cuándo caduca el certificado?, ¿qué pasaría si se cambiara el `DocumentRoot` de Solvia a `/var/www/solvia`? Si no puede contestar, la ficha está incompleta.

### 9.6 ⚖️ Alternativa y criterio

La documentación se puede guardar de tres formas: en un documento aparte (un texto, una wiki), como **comentarios dentro de los propios ficheros de configuración**, o en un fichero `README` junto al código. La regla que decide es: *lo que cambia a la vez debe vivir junto*. Un motivo breve («HSTS corto a propósito: ver nota») va como comentario en la línea de configuración (Apache admite comentarios con `#`), porque quien edite esa línea lo verá. El inventario y las decisiones largas van en un documento aparte, versionado, porque tratan del servidor en su conjunto. Lo que no se debe hacer es repetir lo mismo en dos sitios, porque tarde o temprano uno de los dos queda desactualizado y ya no se sabe cuál es el correcto.

### 9.7 ⚠️ Errores comunes

- **Documentar de memoria, no del servidor**: se escribe «Apache 2.4.5x» sin comprobarlo. El inventario sale de los comandos, no del recuerdo.
- **Poner secretos en la documentación**: contraseñas, claves privadas o el contenido del `.htpasswd` en un documento que se comparte. La documentación dice **dónde** está un secreto, nunca **cuál** es.
- **Copiar ficheros enteros al documento**: queda desactualizado; mejor nombrar el fichero y documentar el motivo.
- **No anotar la fecha de caducidad del certificado**: es el origen de muchas caídas «misteriosas».
- **Cambiar sin copia ni `configtest`**: un error de sintaxis impide que Apache arranque tras reiniciar.
- **Copias de seguridad sin probar la restauración**, o guardadas con permisos que permiten leerlas a cualquiera (contienen la clave privada).
- **Un documento único para todos los lectores**: nadie lo lee entero.

### 9.8 💡 Nota técnica

Cuando se administran muchos servidores, las configuraciones se escriben como **código** que una herramienta aplica (Ansible, Puppet, o plantillas de contenedor): es la forma más completa de documentación, porque describe el estado deseado en un fichero que se puede ejecutar, versionar y revisar, y que deja el servidor igual cada vez que se aplica. No se practica en esta unidad. En el módulo se vuelve a ella al llegar a contenedores. Una buena documentación escrita a mano sigue siendo necesaria en esos casos: el código dice *qué*, y el *porqué* solo lo dice una persona.

### 9.9 Comprueba lo que has entendido

**1. ¿Por qué la documentación de configuración incluye el motivo de cada decisión y no solo el valor?**

Porque el valor se puede leer en el fichero de configuración, pero el motivo no. Sin el motivo, quien mantenga el servidor no sabe si un ajuste («HSTS con 300 segundos», «Basic y no Digest») es esencial o accidental, y se arriesga a cambiarlo o a mantenerlo sin entender qué protege. Con el motivo, y con la forma de comprobarlo y de deshacerlo, se puede decidir con criterio.

**2. Vas a cambiar una directiva de `solvia.conf`. ¿Qué cuatro pasos sigues y qué comando evita dejar el servidor caído por un error de escritura?**

Copia de seguridad del fichero (`cp -a`), cambio, comprobación de sintaxis y recarga con verificación. El comando que evita dejar el servidor caído es `sudo apache2ctl configtest`: si no dice `Syntax OK`, no se aplica el cambio. Si tras aplicarlo algo falla, se restaura la copia.

**3. Has hecho una copia de seguridad de `/etc/apache2/` en un `.tar.gz`. ¿Qué dos comprobaciones faltan antes de poder fiarte de ella y por qué hay que protegerla como a un secreto?**

Falta comprobar su contenido (`tar -tzf`) y probar que se puede restaurar (extraerla en una carpeta temporal y comparar con `diff -r`). Hay que protegerla porque incluye la clave privada del certificado y el fichero de contraseñas: quien la lea puede suplantar al servidor y atacar las contraseñas.

**4. Un desarrollador te pide «abrir los permisos de `config/` para que la aplicación funcione». ¿Qué le respondes según las recomendaciones de uso?**

Que no se cambian permisos para que algo funcione: se averigua la causa. El código y la configuración se leen pero no se escriben desde la web, y `chmod 777` los dejaría escribibles por cualquier proceso del sistema. Si hay un fallo, se mira el registro de errores (por ejemplo, un `AH00035` de permiso denegado) y se corrige el propietario o el grupo con el mínimo permiso necesario (apartado 4.2).

**5. ¿Qué tres preguntas debe poder responder otra persona, solo con la ficha del servidor, para considerarla suficiente?**

Cualquiera de este estilo: qué sitios sirve el servidor, dónde están los ficheros importantes (configuración, certificados, registros, aplicación), cuándo caduca el certificado, qué decisiones de seguridad se tomaron y por qué, y cómo se comprueba cada una. La prueba es que quien la lea pueda mantener o reconstruir el servidor sin preguntar a quien lo montó.

---

## Glosario

Los términos están ordenados alfabéticamente. Entre paréntesis, el apartado donde se explican con detalle.

**`a2ensite` / `a2dissite` / `a2enmod` / `a2dismod` / `a2query`.** Órdenes de Ubuntu para activar o desactivar sitios y módulos de Apache (crean o borran enlaces simbólicos en `sites-enabled` y `mods-enabled`) y para consultar qué hay activo (2.4, 3.4).

**ACME** (*Automatic Certificate Management Environment*). Protocolo estándar con el que un programa del servidor pide, demuestra que controla el dominio y renueva certificados de una CA sin intervención humana. Es el que usa Let's Encrypt (6.9).

**`AH00558`, `AH01617`…** Los mensajes de Apache llevan un código que empieza por `AH` seguido de un número. Sirve para buscar el motivo exacto en la documentación (1.9, 8.3).

**`AllowOverride`.** Directiva que dice qué ajustes puede cambiar un fichero `.htaccess` dentro de una carpeta. Con `None` Apache ni siquiera lo busca (4.4).

**Anfitrión (equipo).** El equipo físico donde corre VirtualBox, distinto de la máquina virtual donde está Apache; su navegador tiene su propio almacén de certificados (6.6, 7.10).

**Autenticación.** Comprobar **quién es** el cliente (por ejemplo, con usuario y contraseña). No debe confundirse con la autorización, que decide **qué puede hacer** (5.1).

**Autorización / control de acceso.** Decidir si un cliente ya identificado, o un origen determinado, puede acceder a un recurso. En Apache se expresa con `Require` (5.2).

**Bitácora de cambios.** Registro fechado de qué se ha cambiado en un servidor, por qué y cómo se comprobó (9.3).

**CA (autoridad de certificación).** Entidad en la que los clientes confían y que firma los certificados de otros. Puede ser pública (Let's Encrypt) o propia (la «CA del aula») (6.4, 6.6).

**Cadena de confianza.** Secuencia de certificados, desde el del servidor hasta una CA raíz que el cliente ya conoce, cada uno firmado por el siguiente (6.4).

**Certificado digital (X.509).** Documento firmado que une un nombre (o varios) con una clave pública, e indica quién lo emitió y hasta cuándo es válido (6.3).

**Certificado autofirmado.** Certificado firmado con su propia clave: el emisor y el titular son el mismo. No hay ninguna CA que lo respalde, por lo que los clientes no se fían de él por defecto (6.5).

**Clave privada / clave pública.** Par de números ligados matemáticamente: lo que firma la privada lo verifica la pública. La privada nunca se comparte (6.2, 6.7).

**Confidencialidad.** Garantía de que solo emisor y receptor pueden leer los datos; la aporta el cifrado de TLS (7.1).

**Consolidación de logs.** Reunir en un solo lugar los registros de varios sitios o servidores para consultarlos juntos (8.9).

**Contenido mixto** (*mixed content*). Página cargada por HTTPS que incluye recursos por HTTP. Los navegadores lo bloquean o lo marcan como inseguro (7.12).

**Contexto.** Zona de la configuración donde puede escribirse una directiva: global, `<VirtualHost>`, `<Directory>`, `<Location>`, `.htaccess`… (1.4).

**CSP** (*Content-Security-Policy*). Cabecera HTTP que lista de dónde puede cargar la página cada tipo de recurso, para limitar el daño de una inyección de código (7.8).

**CSR** (*Certificate Signing Request*). Petición de certificado: contiene la clave pública y el nombre solicitado, firmada con la clave privada, y se envía a una CA para que la firme (6.6).

**Directiva.** Instrucción de configuración de Apache: un nombre seguido de sus valores (`ServerName solvia.test`) (1.1).

**`DocumentRoot`.** Carpeta del disco que un sitio sirve como raíz de su web (3.5, 4.1).

**Fuerza bruta.** Ataque que prueba muchas contraseñas hasta acertar. En los registros se ve como muchos intentos fallidos desde la misma dirección (8.6).

**GoAccess.** Programa que analiza registros de accesos y presenta un panel (en terminal o en una página HTML que se actualiza en directo) (8.10).

**Header** (cabecera HTTP). Línea `Nombre: valor` que acompaña a una petición o a una respuesta (5.3, 7.8).

**`Host` (cabecera).** Cabecera de la petición HTTP con el nombre del sitio al que se dirige. Es lo que Apache compara con `ServerName` y `ServerAlias` para elegir el sitio virtual (3.2).

**HSTS** (*HTTP Strict Transport Security*). Cabecera que ordena al navegador no usar nunca HTTP con ese sitio durante un tiempo (`max-age`). Solo es fiable en respuestas HTTPS (7.9).

**`.htaccess`.** Fichero de configuración que Apache lee dentro de una carpeta, en cada petición, si `AllowOverride` lo permite. Es más lento y menos seguro que escribir la regla en el `VirtualHost` (4.4).

**`htpasswd`.** Programa que crea y modifica el fichero con usuarios y contraseñas (con su resumen criptográfico) para la autenticación Basic (5.4).

**HTTP / HTTPS.** Protocolo de la web y su versión sobre un canal cifrado con TLS (7.1).

**Integridad.** Garantía de que los datos no han sido modificados por el camino (7.1).

**`LogLevel`.** Directiva que fija la gravedad mínima de los mensajes del registro de errores; admite un nivel por módulo (8.3).

**Log / registro.** Fichero donde un programa anota lo que ocurre. Apache escribe uno de accesos y otro de errores (8.1).

**logrotate.** Programa de Ubuntu que rota los registros: guarda los antiguos con otro nombre, comprime y borra los más viejos (8.8).

**`mod_status` / `/server-status`.** Módulo y página que muestran en vivo el estado del servidor; por eso se restringe por origen y/o se protege con contraseña (2.11, 5.7).

**Módulo.** Pieza que añade una funcionalidad a Apache y se carga en el arranque (`ssl`, `headers`, `auth_basic`…) (apartado 2).

**MPM** (*Multi-Processing Module*). Módulo que decide cómo Apache atiende peticiones simultáneas: con procesos (`prefork`), con hilos (`worker`, `event`) (1.5, 2.14).

**`Options`.** Directiva que activa o desactiva funciones de una carpeta (como el listado de ficheros con `Indexes`) (4.3).

**Principio de mínimo privilegio.** Cada usuario, proceso o fichero recibe solo los permisos que necesita (4.2, 9.3).

**Proxy inverso.** Servidor que recibe las peticiones de los clientes y las reenvía a otro servidor; suele encargarse del cifrado TLS (7.13).

**Redirección 301 / 308.** Respuestas que indican que un recurso se ha movido para siempre. Con 301 los clientes convierten un `POST` en `GET`; con 308 repiten el método y el cuerpo (7.3).

**`Redirect`.** Directiva que responde con un código 3xx y una cabecera `Location` para enviar al cliente a otra dirección (7.3).

**Referer** (en la cabecera, escrito así por un error histórico). Dirección de la página desde la que llegó el cliente (7.8, 8.2).

**`Require`.** Directiva de autorización: `Require all granted`, `Require ip`, `Require valid-user`… (5.2).

**RGPD.** Reglamento europeo de protección de datos personales. La dirección IP se considera dato personal según el criterio habitual (8.12).

**Rotación de logs.** Ver *logrotate* (8.8).

**SAN** (*Subject Alternative Name*). Campo del certificado con la lista de nombres para los que es válido. Es el que comprueban los navegadores (6.3).

**`ServerAlias` / `ServerName`.** Nombres con los que responde un sitio virtual (3.2, 3.5).

**Sitio virtual** (*VirtualHost*). Bloque de configuración que define un sitio web: nombre, carpeta y reglas. Permite alojar varios sitios en un mismo Apache (apartado 3).

**SNI** (*Server Name Indication*). Extensión de TLS por la que el cliente indica, en claro y al principio del saludo, a qué nombre se quiere conectar, para que el servidor elija el certificado (7.1, 3.12).

**Suite de cifrado** (*cipher suite*). Combinación de algoritmos que cliente y servidor negocian para una conexión TLS (7.5).

**Superficie de ataque.** Conjunto de puntos por los que un sistema puede ser atacado (módulos activos, sitios, puertos abiertos…). Se reduce activando solo lo necesario (9.3).

**`tcpdump`.** Programa que captura el tráfico de red; muestra qué es legible y qué no (5.3, 7.6).

**TLS** (*Transport Layer Security*). Protocolo que cifra y autentica una conexión. Las versiones vigentes son TLS 1.2 y 1.3 (7.5).

**WebSocket.** Conexión persistente entre navegador y servidor por la que este puede enviar datos sin que se los pidan. GoAccess la usa para actualizar su informe en tiempo real (8.10).

**`X-Content-Type-Options: nosniff`.** Cabecera que impide que el navegador «adivine» el tipo de un fichero distinto del declarado (7.8).

---

## Referencias

Se citan solo fuentes cuya existencia se conoce. Las páginas web cambian de dirección y de contenido: si un enlace no funciona, se busca el título en la página principal de la organización. Los apartados donde se usa cada fuente están indicados entre paréntesis.

**Apache HTTP Server 2.4 (documentación oficial).**

- Página principal de la documentación: <https://httpd.apache.org/docs/2.4/>
- Sitios virtuales: <https://httpd.apache.org/docs/2.4/vhosts/> (3).
- Control de acceso, autenticación y autorización: <https://httpd.apache.org/docs/2.4/howto/auth.html> (5).
- Documentación del módulo `mod_auth_digest`, con la advertencia sobre su seguridad: <https://httpd.apache.org/docs/2.4/mod/mod_auth_digest.html> (5.8).
- Módulo `mod_ssl`: <https://httpd.apache.org/docs/2.4/mod/mod_ssl.html> (7).
- Módulo `mod_headers`: <https://httpd.apache.org/docs/2.4/mod/mod_headers.html> (2.9, 7.8).
- Registros (`logs`): <https://httpd.apache.org/docs/2.4/logs.html> y formato de registro, `mod_log_config`: <https://httpd.apache.org/docs/2.4/mod/mod_log_config.html> (8).

**Especificaciones (RFC, publicadas por el IETF en <https://www.rfc-editor.org/>).** Se citan por número; las más relevantes para esta unidad:

- RFC 2606, nombres de dominio reservados para pruebas (`.test`) (3.3).
- RFC 7617, autenticación HTTP «Basic» (5.3).
- RFC 5280, perfil de certificados X.509 (6.3).
- RFC 8555, protocolo ACME (6.9).
- RFC 8446, TLS 1.3 (7.5).
- RFC 6797, HSTS (7.9).
- RFC 9110, semántica de HTTP, que incluye los códigos 301 y 308 (7.3).

**Certificados.**

- Documentación de Let's Encrypt: <https://letsencrypt.org/docs/> (6.9). Los datos sobre la duración de los certificados (90 días por defecto, con planes de acortarla) se tomaron de sus anuncios oficiales a fecha de redacción; **conviene consultarlos de nuevo, porque cambian con el tiempo**.
- Foro CA/Browser (organismo donde las CA públicas y los navegadores fijan las reglas de emisión): <https://cabforum.org/> (6.9). La reducción progresiva de la vida máxima de los certificados públicos está recogida en su votación SC-081v3; se aconseja leer la versión vigente.
- Documentación de OpenSSL: <https://www.openssl.org/docs/> (6).

**Cabeceras de seguridad.**

- MDN Web Docs, referencia de cabeceras HTTP: <https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers> (7.8).
- MDN Web Docs, Content-Security-Policy: <https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP> (7.8).
- Lista de precarga de HSTS y sus requisitos: <https://hstspreload.org/> (7.9).
- Proyecto OWASP Secure Headers: <https://owasp.org/www-project-secure-headers/> (7.8).
- Generador de configuraciones SSL de Mozilla: <https://ssl-config.mozilla.org/> (7.5). Se consulta siempre su versión vigente.

**Registros y análisis.**

- GoAccess, sitio oficial y manual: <https://goaccess.io/> (8.10).
- Manual de `logrotate` en Ubuntu: `man logrotate` (8.8).

**Ubuntu.**

- Documentación oficial de Ubuntu Server: <https://documentation.ubuntu.com/server/> (el sitio se ha reorganizado en ocasiones; si la dirección falla, se busca «Ubuntu Server documentation» en <https://ubuntu.com/>).
- Página `README.Debian` de Apache, instalada con el paquete (`/usr/share/doc/apache2/README.Debian.gz`), que explica la disposición de ficheros de Debian y Ubuntu (1.2).

**Normativa del módulo.** Los resultados de aprendizaje y criterios de evaluación del módulo (RA2, CE a–j) proceden del currículo oficial del ciclo en Aragón (Orden ECD/843/2024).
