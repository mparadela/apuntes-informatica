# UD2 — Administración de servidores web · Apuntes

**0614 · Despliegue de Aplicaciones Web · 2º DAW · IES Río Arba · Curso 2026-27**

**RA2 — Implanta aplicaciones web en servidores web, evaluando y aplicando criterios de configuración para su funcionamiento seguro.**

> Estos apuntes no son el punto de partida de la clase. Primero se trabaja cada cosa sobre vuestro Solvia, en vuestra máquina. Aquí está la explicación completa y ordenada, para consolidar lo hecho y repasar.
>
> Todo lo que aparece se ha probado sobre **Ubuntu Server 24.04.5 con Apache 2.4.58, PHP 8.3.6, MariaDB 10.11.14, OpenSSL 3.0.13 y GoAccess 1.8.1**, con el código real de Solvia. Si tu máquina tiene otra versión de Ubuntu, pueden cambiar los números de versión. Los comandos son los mismos.

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

### 1.1 Definición

Los **parámetros de administración** de un servidor web son las directivas de configuración que gobiernan su comportamiento global, independientemente del sitio que sirva: en qué direcciones y puertos escucha, con qué identidad de usuario se ejecutan sus procesos, cuántos procesos mantiene y cuánto espera a un cliente, qué información revela sobre sí mismo y dónde registra lo que ocurre.

En Apache, cada parámetro es una **directiva**: una línea con un nombre y uno o más valores (`Timeout 300`, `Listen 80`). Cada directiva tiene un **contexto** en el que es válida, y escribirla fuera de él es un error de sintaxis.

### 1.2 Dónde vive la configuración en Ubuntu

En UD1 ya visteis las carpetas `sites-*` y `mods-*`. Ahora toca el mapa completo, porque para administrar hay que saber **en qué orden** lee Apache sus ficheros. Lo que se lee después puede cambiar lo que se leyó antes.

```text
/etc/apache2/
├── apache2.conf          ← fichero principal: parámetros globales e inclusiones
├── envvars               ← variables de entorno: usuario, grupo, carpeta de logs...
├── ports.conf            ← en qué puertos escucha (Listen)
├── mods-available/       ← módulos instalados          ┐ a2enmod / a2dismod
├── mods-enabled/         ← enlaces a los activos       ┘
├── conf-available/       ← fragmentos de configuración ┐ a2enconf / a2disconf
├── conf-enabled/         ← enlaces a los activos       ┘
├── sites-available/      ← sitios (VirtualHost)        ┐ a2ensite / a2dissite
└── sites-enabled/        ← enlaces a los activos       ┘
```

El propio `apache2.conf` dice en qué orden se incluye todo. Quitando comentarios, el esqueleto es este:

```apache
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

# ... bloques <Directory> y formatos de log ...

IncludeOptional conf-enabled/*.conf
IncludeOptional sites-enabled/*.conf
```

Tres consecuencias prácticas:

- **Los módulos se cargan antes que todo lo demás**, porque las directivas de un módulo no existen hasta que el módulo está cargado. Por eso `ports.conf` puede decir `<IfModule ssl_module> Listen 443 </IfModule>`: cuando se lee, ya se sabe si `mod_ssl` está activo.
- **`conf-enabled/` se lee antes que `sites-enabled/`**. Lo global va en `conf-*`; lo de cada sitio, en su fichero de `sites-*`.
- **Dentro de cada carpeta, el orden es alfabético.** Esto importa más de lo que parece: volverá a salir en el apartado 3, porque decide cuál es el sitio por defecto.

Los `${...}` son variables definidas en `/etc/apache2/envvars`. En Ubuntu, `APACHE_RUN_USER` y `APACHE_RUN_GROUP` valen `www-data`, y `APACHE_LOG_DIR` vale `/var/log/apache2`.

Para ver la lista exacta de ficheros que Apache ha incluido, en el orden en que los ha leído:

```bash
sudo apache2ctl -t -D DUMP_INCLUDES
```

```text
Included configuration files:
  (*) /etc/apache2/apache2.conf
    (146) /etc/apache2/mods-enabled/access_compat.load
    (146) /etc/apache2/mods-enabled/alias.load
    (146) /etc/apache2/mods-enabled/auth_basic.load
    ...
```

El número entre paréntesis es la línea de `apache2.conf` donde está el `IncludeOptional` que trajo ese fichero.

### 1.3 Contextos: dónde puede ir cada directiva

| Contexto | Dónde se escribe | Afecta a |
|---|---|---|
| **server config** | Fuera de cualquier bloque: `apache2.conf`, `ports.conf`, `conf-enabled/*.conf` | Todo el servidor |
| **virtual host** | Dentro de `<VirtualHost>` | Solo ese sitio |
| **directory** | Dentro de `<Directory>`, `<Location>`, `<Files>` | Solo esa ruta |
| **.htaccess** | En un fichero `.htaccess` dentro de la carpeta servida | Esa carpeta y sus subcarpetas, si `AllowOverride` lo permite |

La documentación oficial de Apache indica el contexto de cada directiva en una línea llamada *Context*. Es lo primero que hay que mirar cuando una directiva «no funciona».

**Ejemplo.** `ServerTokens` solo es válida en *server config*. Si la metes dentro de tu VirtualHost:

```apache
<VirtualHost *:443>
    ServerName solvia.test
    DocumentRoot /var/www/solvia/public
    ServerTokens Full
    ...
</VirtualHost>
```

el chequeo de sintaxis lo rechaza y te dice dónde:

```bash
sudo apache2ctl configtest
```

```text
AH00526: Syntax error on line 12 of /etc/apache2/sites-enabled/solvia.conf:
ServerTokens cannot occur within <VirtualHost> section
```

### 1.4 Los parámetros que hay que conocer

Valores por defecto comprobados en Ubuntu 24.04.

| Directiva | Dónde está | Valor por defecto | Qué controla |
|---|---|---|---|
| `Listen` | `ports.conf` | `80`, y `443` si `mod_ssl` está activo | En qué puerto (y opcionalmente en qué IP) acepta conexiones |
| `User` / `Group` | `apache2.conf` (vía `envvars`) | `www-data` / `www-data` | Con qué usuario se ejecutan los procesos que atienden peticiones. Es el usuario que necesita permiso para leer tus ficheros |
| `ServerName` (global) | no viene puesto | — | Nombre del servidor. Sin él, Apache avisa con AH00558 al arrancar |
| `Timeout` | `apache2.conf` | `300` | Segundos máximos que Apache espera ciertos eventos de E/S de una conexión |
| `KeepAlive` | `apache2.conf` | `On` | Si una misma conexión TCP sirve varias peticiones seguidas |
| `MaxKeepAliveRequests` | `apache2.conf` | `100` | Cuántas peticiones como máximo por conexión persistente |
| `KeepAliveTimeout` | `apache2.conf` | `5` | Cuántos segundos espera la siguiente petición antes de cerrar la conexión |
| `HostnameLookups` | `apache2.conf` | `Off` | Si traduce a nombre la IP de cada cliente para el log (cuesta una consulta DNS por petición) |
| `ErrorLog` / `LogLevel` | `apache2.conf` | `error.log` / `warn` | Dónde y con qué detalle se registran los errores globales (apartado 8) |
| `DirectoryIndex` | `mods-enabled/dir.conf` | `index.html index.cgi index.pl index.php index.xhtml index.htm` | Qué fichero se sirve cuando se pide una carpeta |
| `ServerTokens` | `conf-available/security.conf` | `OS` | Cuánto cuenta la cabecera `Server` de cada respuesta |
| `ServerSignature` | `conf-available/security.conf` | `On` | Si las páginas de error generadas por Apache llevan un pie con versión y nombre |
| `TraceEnable` | `conf-available/security.conf` | `Off` | Si se acepta el método HTTP `TRACE` |

**El MPM.** El *Multi-Processing Module* decide cómo reparte Apache el trabajo entre procesos e hilos. Con `mod_php`, que es lo que usa Solvia, Ubuntu activa **prefork**: un proceso por conexión, sin hilos, porque `mod_php` no es seguro con hilos. Sus parámetros están en `mods-available/mpm_prefork.conf`:

```apache
StartServers            5
MinSpareServers         5
MaxSpareServers         10
MaxRequestWorkers       150
MaxConnectionsPerChild  0
```

`MaxRequestWorkers 150` significa que, como mucho, se atienden 150 conexiones a la vez. La 151 espera. Se ve en la lista de procesos: uno de `root`, que solo vigila y abre puertos, y varios de `www-data`, que son los que atienden.

```bash
ps -eo user,comm | grep apache2 | sort | uniq -c
```

```text
      1 root     apache2
      5 www-data apache2
```

### 1.5 Las herramientas del administrador

| Comando | Para qué |
|---|---|
| `sudo apache2ctl configtest` (o `-t`) | Comprueba la sintaxis **sin** tocar el servicio. Siempre antes de recargar |
| `sudo apache2ctl -S` | Qué sitios hay, en qué puerto, cuál es el sitio por defecto |
| `sudo apache2ctl -M` | Qué módulos están cargados, y si son estáticos o dinámicos |
| `apache2ctl -V` | Versión, MPM, rutas compiladas |
| `sudo apache2ctl -t -D DUMP_INCLUDES` | Qué ficheros se han leído y en qué orden |
| `sudo systemctl reload apache2` | Relee la configuración sin cortar las conexiones en curso |
| `sudo systemctl restart apache2` | Para y arranca. Necesario, por ejemplo, al cambiar de MPM |

```bash
apache2ctl -V | grep -E "Server version|MPM|SERVER_CONFIG_FILE"
```

```text
Server version: Apache/2.4.58 (Ubuntu)
Server MPM:     prefork
 -D SERVER_CONFIG_FILE="apache2.conf"
```

### 1.6 Ejemplo resuelto: lo que tu servidor cuenta de sí mismo

Con la configuración que dejasteis en UD1, cualquiera puede saber qué versión exacta de Apache tenéis y sobre qué sistema corre. Basta con pedir la cabecera de la respuesta o provocar un error:

```bash
curl -sI http://localhost/ | grep -i ^server
curl -s http://localhost/noexiste | grep -i address
```

```text
Server: Apache/2.4.58 (Ubuntu)
<address>Apache/2.4.58 (Ubuntu) Server at localhost Port 80</address>
```

La primera línea la produce `ServerTokens OS`. La segunda, `ServerSignature On`, en el pie de la página de error 404 que genera Apache.

**Solución.** Editar `/etc/apache2/conf-available/security.conf` (ya está activo: viene enlazado en `conf-enabled/`) y cambiar dos líneas:

```apache
ServerTokens Prod
ServerSignature Off
```

Comprobar y recargar:

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -sI http://localhost/ | grep -i ^server
curl -s http://localhost/noexiste | grep -i -c address
```

```text
Syntax OK
Server: Apache
0
```

Con `Prod` la cabecera sigue existiendo, pero solo dice «Apache». El `0` final significa que la página de error ya no tiene la línea `<address>`.

Los valores posibles de `ServerTokens`, de menos a más información: `Prod` (`Apache`), `Major` (`Apache/2`), `Minor` (`Apache/2.4`), `Minimal` (`Apache/2.4.58`), `OS` (`Apache/2.4.58 (Ubuntu)`) y `Full` (además, los módulos que se anuncian).

### 1.7 Ejemplo resuelto: el aviso AH00558

Desde la primera instalación habéis visto esto al recargar:

```text
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
```

Como se dijo en UD1, **es un aviso, no un error**: Apache arranca igual. Dice que no hay un `ServerName` en el contexto global y que ha tenido que deducir uno. El `ServerName` de vuestro VirtualHost no cuenta, porque está dentro de un sitio, no en el contexto global.

**Solución limpia**, sin tocar `apache2.conf`: un fragmento propio en `conf-available/`, activado con `a2enconf`.

```bash
echo "ServerName solvia.test" | sudo tee /etc/apache2/conf-available/servername.conf
sudo a2enconf servername
sudo apache2ctl configtest
sudo systemctl reload apache2
```

```text
Enabling conf servername.
Syntax OK
```

El aviso desaparece.

### 1.8 Caso práctico sobre Solvia: el inventario de tu servidor

Antes de cambiar nada, el administrador sabe qué tiene. En tu máquina, rellena esta tabla mirando los ficheros, no de memoria:

| Directiva | Fichero donde la encontraste | Valor que tenía | Valor que dejas | Por qué |
|---|---|---|---|---|
| `Listen` | | | | |
| `User` | | | | |
| `Timeout` | | | | |
| `KeepAliveTimeout` | | | | |
| `LogLevel` | | | | |
| `ServerTokens` | | | | |
| `ServerSignature` | | | | |
| `ServerName` global | | | | |

Comandos que ayudan:

```bash
grep -rn "^\s*Listen" /etc/apache2/ports.conf
grep -n -E "^(Timeout|KeepAlive|LogLevel|User)" /etc/apache2/apache2.conf
grep -E "APACHE_RUN_USER|APACHE_LOG_DIR" /etc/apache2/envvars
grep -v "^\s*#" /etc/apache2/conf-available/security.conf | grep -v "^$"
```

En la mayoría de filas la respuesta correcta es **dejarlo como está**, y eso también hay que justificarlo. Solo `ServerTokens`, `ServerSignature` y el `ServerName` global tienen motivo claro para cambiar en esta unidad.

### 1.9 ⚖️ Alternativa y criterio

**Editar directamente `apache2.conf` frente a crear un fichero en `conf-available/` y activarlo con `a2enconf`.** Las dos cosas funcionan. Se prefiere la segunda por tres razones:

- **Actualizaciones.** `apache2.conf` es un fichero de configuración del paquete. Si lo has modificado y una actualización trae una versión nueva, el sistema te pregunta qué hacer con él, y es fácil equivocarse. Un fichero tuyo en `conf-available/` el paquete no lo toca.
- **Reversibilidad.** `sudo a2disconf servername` deshace el cambio en un comando, sin buscar qué línea tocaste.
- **Legibilidad.** Quien herede el servidor ve en `conf-enabled/` qué se ha añadido a mano, sin comparar `apache2.conf` línea a línea con el original.

Con `security.conf` pasa algo parecido: también es del paquete, pero existe precisamente para que se edite. Se acepta modificarlo, y es lo que hacemos aquí.

### 1.10 ⚠️ Error común

**Creer que ocultar la versión es protegerse.** `ServerTokens Prod` no arregla ninguna vulnerabilidad: si tu Apache tiene un fallo conocido, lo sigue teniendo. Lo que hace es no regalar la información a quien busca servidores con una versión concreta. Es una medida de **reducción de información**, útil y barata, pero no sustituye a mantener el sistema actualizado (`sudo apt update && sudo apt upgrade`).

**Otro, igual de frecuente:** cambiar una directiva, no recargar Apache, y concluir que «no funciona». Apache solo lee la configuración al arrancar o al recargar. La secuencia es siempre la misma: editar → `configtest` → `reload` → comprobar con `curl`.

### 1.11 💡 Nota técnica

El `Timeout 300` por defecto es generoso: cinco minutos. Un tipo de ataque de denegación de servicio, conocido como *Slowloris*, consiste justamente en abrir muchas conexiones y enviar las cabeceras muy despacio para tener ocupados todos los procesos (en prefork, los 150 de `MaxRequestWorkers`). Ubuntu ya trae activado **`mod_reqtimeout`**, que limita cuánto se puede tardar en enviar las cabeceras y el cuerpo de una petición:

```apache
RequestReadTimeout header=20-40,minrate=500
RequestReadTimeout body=10,minrate=500
```

Es decir: entre 20 y 40 segundos para las cabeceras, siempre que lleguen al menos 500 bytes por segundo. Es un buen ejemplo de lo que se verá en el apartado 2: buena parte de la seguridad de Apache no está en el núcleo, sino en módulos.

Y uno más que ya tenéis bien sin haberlo tocado: `TraceEnable Off`. El método `TRACE` devuelve al cliente la petición tal como llegó, cabeceras incluidas, y se ha usado para robar cookies. Con `Off`, Apache responde `405 Method Not Allowed`:

```bash
curl -s -X TRACE -o /dev/null -w "%{http_code}\n" http://localhost/
```

```text
405
```

---

*Apartados 2 a 9, glosario y referencias: pendientes de redactar.*
