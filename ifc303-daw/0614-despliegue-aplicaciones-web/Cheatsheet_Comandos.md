# Cheatsheet de comandos · 0614 Despliegue de Aplicaciones Web

**2º DAW · IES Río Arba · Curso 2026-27** · Referencia para todo el módulo (UD1 a UD6)

## Cómo se usa esta hoja

Está pensada para **consultar, no para leer de arriba abajo**. Cada comando lleva su sintaxis, sus opciones explicadas una a una (qué hace y cuándo la usas) y, cuando hay una trampa típica, el error que da. Antes de preguntar «¿qué hace este comando?», busca aquí; si no está, usa `man` ([apartado 0](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b0)).

**Convenciones**

- `<algo>` es un hueco que **tú sustituyes** (sin los símbolos `< >`). `[algo]` es opcional.
- Una etiqueta como **(UD2)** indica en qué unidad se trabaja. Lo marcado **(UD3-UD6)** corresponde a unidades que aún no se han dado.
- **[por confirmar]** marca lo que depende de decisiones aún no tomadas en la programación (versión, paquete o herramienta concreta). Los comandos de ese bloque son los estándar de la herramienta, pero compruébalos contra la versión que se instale en clase antes de darlos por válidos.
- Los comandos que **se teclean** van en un recuadro marcado como «comando»; lo que **sale por pantalla**, en uno marcado como «salida». Los ficheros que escribes llevan en la esquina su tipo (`apache`, `yaml`, `ldif`…).
- **Para moverte**: en el [índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice), el número y el nombre de cada bloque te llevan a él; al final de cada bloque, «↑ Volver al índice» te devuelve aquí. Las referencias del tipo «apartado 13» también son enlaces.
- Se asume **Ubuntu Server 24.04** (lo comprobado con Apache 2.4.58, PHP 8.3, OpenSSL 3.0 y GoAccess 1.8).

**La regla de oro**: casi todo en Linux es un fichero y casi cada comando hace **una** cosa. Si no recuerdas la opción exacta, sabes igualmente qué pieza toca: el servicio, el puerto, el fichero o los permisos. Con eso ya sabes dónde mirar.

## Índice :id=indice

| Bloque | Contenido |
|---|---|
| [0](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b0) | [Cómo averiguar qué hace un comando](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b0) |
| [1](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b1) | [Moverse y listar](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b1) |
| [2](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b2) | [Ver el contenido de ficheros](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b2) |
| [3](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b3) | [Crear, copiar, mover y borrar](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b3) |
| [4](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b4) | [Buscar y filtrar texto](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b4) |
| [5](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b5) | [Redirecciones, tuberías y atajos](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b5) |
| [6](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b6) | [Permisos y propietarios](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b6) |
| [7](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b7) | [Usuarios, grupos y `sudo`](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b7) |
| [8](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b8) | [Paquetes con `apt`](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b8) |
| [9](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b9) | [Servicios con `systemctl` y registros con `journalctl`](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b9) |
| [10](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b10) | [Procesos y recursos](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b10) |
| [11](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b11) | [Red](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b11) |
| [12](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b12) | [Comprimir y empaquetar](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b12) |
| [13](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b13) | [Editar con `nano` y comparar con `diff`](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b13) |
| [14](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b14) | [Apache (UD1-UD2)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b14) |
| [15](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b15) | [`curl` y códigos HTTP (UD1-UD2)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b15) |
| [16](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b16) | [Certificados y HTTPS con `openssl` (UD2)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b16) |
| [17](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b17) | [Autenticación Basic con `htpasswd` (UD2)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b17) |
| [18](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b18) | [Logs: leer, filtrar, rotar y analizar (UD2)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b18) |
| [19](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b19) | [MariaDB y PHP (UD1)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b19) |
| [20](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b20) | [Tomcat y proxy inverso (UD3)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b20) |
| [21](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b21) | [FTP, FTPS y SFTP (UD4)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b21) |
| [22](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b22) | [DNS (UD6 · RA5)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b22) |
| [23](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b23) | [LDAP (UD5)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b23) |
| [24](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b24) | [Git y GitHub (UD6)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b24) |
| [25](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b25) | [Docker y Docker Compose (UD6)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b25) |
| [26](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b26) | [Integración continua con GitHub Actions (UD6)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b26) |
| [27](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b27) | [Generación de documentación (UD6)](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b27) |
| [28](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b28) | [Mapa rápido: «tengo este problema, ¿qué miro?»](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b28) |

---

## 0. Cómo averiguar qué hace un comando :id=b0

Un administrador consulta la documentación decenas de veces al día. No saberse una opción de memoria no es un fallo; no saber dónde mirarla, sí.

| Comando | Qué hace | Cuándo lo usas |
|---|---|---|
| `man <comando>` | Abre el manual completo del comando | Quieres todas las opciones. Se navega con **espacio** (avanzar), **b** (retroceder), **/texto** (buscar), **n** (siguiente coincidencia) y **q** (salir) |
| `<comando> --help` | Imprime un resumen corto de uso y opciones | Quieres recordar una opción sin abrir el manual. Si es largo: `<comando> --help \| less` |
| `man -k <palabra>` (o `apropos <palabra>`) | Busca comandos cuya descripción contiene la palabra | No sabes cómo se llama el comando que buscas |
| `type <comando>` | Dice si es un programa, un alias o una función del shell | Un comando no se comporta como esperabas |
| `which <comando>` | Muestra la ruta del programa | Quieres saber qué binario se ejecuta |
| `apt show <paquete>` | Descripción de un paquete | Quieres saber qué es un paquete antes de instalarlo |

El manual está dividido en **secciones**: `1` comandos, `5` formatos de ficheros de configuración, `8` administración del sistema. Si un nombre existe en varias, se elige con `man 5 <nombre>`.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 1. Moverse y listar :id=b1

### `pwd` y `cd`

```bash
pwd                  # ¿dónde estoy?
cd /var/www          # ir a una ruta absoluta
cd solvia            # ir a una subcarpeta (ruta relativa)
cd ..                # subir un nivel
cd -                 # volver a la carpeta anterior
cd                   # ir a mi carpeta personal (igual que cd ~)
```

| Símbolo | Significa |
|---|---|
| `/` | Raíz del sistema. Una ruta que empieza por `/` es **absoluta**; si no, es **relativa** a donde estás |
| `~` | Tu carpeta personal (`/home/<usuario>`) |
| `.` | La carpeta actual |
| `..` | La carpeta de arriba |
| `*` | Cualquier cosa, cualquier longitud (`ls *.conf`) |
| `?` | Un único carácter cualquiera |
| `[abc]` | Uno de esos caracteres |

### `ls`

```bash
ls -la /etc/apache2
ls -lt /var/log/apache2      # el más reciente, arriba
ls -ld /var/www/solvia       # datos de la propia carpeta, no de su contenido
```

| Opción | Qué hace | Cuándo la usas |
|---|---|---|
| `-l` | Formato largo: permisos, dueño, grupo, tamaño, fecha | Casi siempre |
| `-a` | Incluye ficheros ocultos (empiezan por punto) | Buscar `.htaccess`, `.env` o `.git` |
| `-h` | Tamaños legibles (K, M, G). Se combina con `-l` | Ver cuánto ocupa algo |
| `-d` | Muestra la carpeta en sí, no su contenido | Comprobar permisos de una carpeta |
| `-t` | Ordena por fecha de modificación (lo más reciente primero) | Saber qué log se ha escrito hace un minuto |
| `-S` | Ordena por tamaño | Encontrar el fichero que llena el disco |
| `-r` | Invierte el orden | Combinado con `-t` o `-S` |
| `-R` | Recursivo: entra en las subcarpetas | Ver un árbol sin `tree` |
| `-i` | Muestra el número de inodo | Casi nunca |

En un listado `-l`, una `l` al principio de la línea (`lrwxrwxrwx`) indica un **enlace simbólico**, y se ve a dónde apunta con una flecha `->`.

### `tree` (hay que instalarlo: `sudo apt install tree`)

| Opción | Qué hace |
|---|---|
| `tree <ruta>` | Árbol de carpetas y ficheros |
| `-L <n>` | Limita la profundidad a *n* niveles |
| `-d` | Solo carpetas |
| `-a` | Incluye ocultos |

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 2. Ver el contenido de ficheros :id=b2

| Comando | Qué hace | Opciones útiles |
|---|---|---|
| `cat <fichero>` | Vuelca el fichero entero por pantalla | `-n` numera las líneas |
| `less <fichero>` | Lo muestra paginado. Salir con **q**; buscar con **/texto**; ir al final con **G** y al principio con **g**; **F** sigue el fichero como `tail -f` (Ctrl+C para parar) | `-N` numera; `-S` no parte las líneas largas |
| `head <fichero>` | Primeras líneas (10 por defecto) | `-n 20` las 20 primeras |
| `tail <fichero>` | Últimas líneas (10 por defecto) | `-n 20` las 20 últimas; `-f` se queda enganchado y muestra lo que se vaya escribiendo (Ctrl+C para salir); `-n0 -f` solo lo que llegue desde ahora |
| `wc <fichero>` | Cuenta | `-l` líneas; `-w` palabras; `-c` bytes |
| `file <fichero>` | Dice qué tipo de fichero es | |
| `stat <fichero>` | Datos completos: permisos, dueño, fechas | `-c '%a %U:%G %n'` formato corto: permisos en octal, dueño:grupo, nombre |

Un log crece **por abajo**: lo último que ha pasado está al final. Abrirlo entero con `nano` es un error de principiante; usa `tail`.

### Tamaño y espacio

| Comando | Qué hace |
|---|---|
| `df -h` | Espacio libre y usado en cada disco/partición |
| `du -sh <ruta>` | Cuánto ocupa una carpeta (`-s` solo el total, `-h` legible) |
| `du -h --max-depth=1 <ruta>` | Cuánto ocupa cada subcarpeta de primer nivel |
| `free -h` | Memoria RAM libre y usada |

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 3. Crear, copiar, mover y borrar :id=b3

| Comando | Qué hace | Opciones útiles |
|---|---|---|
| `mkdir <carpeta>` | Crea una carpeta | `-p` crea también las intermedias y no da error si ya existe |
| `touch <fichero>` | Crea un fichero vacío (o actualiza su fecha) | |
| `cp <origen> <destino>` | Copia | `-r` carpetas enteras; `-a` conserva permisos, dueño y fechas; `-p` conserva permisos y fechas; `-i` pregunta antes de sobrescribir; `-v` muestra lo que hace |
| `mv <origen> <destino>` | Mueve **o renombra** | `-i` pregunta antes de sobrescribir; `-n` no sobrescribe nunca; `-v` muestra lo que hace |
| `rm <fichero>` | Borra. **No hay papelera** | `-r` carpetas; `-i` pregunta; `-f` no pregunta ni protesta. `rm -rf` no se deshace |
| `rmdir <carpeta>` | Borra una carpeta **vacía** | |
| `ln -s <destino> <enlace>` | Crea un enlace simbólico (un atajo) | `ls -l` muestra a dónde apunta |

**Trampa del asterisco** (UD1): `cp -r solvia/* /var/www/solvia/` copia el **contenido** de la carpeta; `cp -r solvia /var/www/solvia/` copia la carpeta **dentro** y acabas con `/var/www/solvia/solvia/public`, y el `DocumentRoot` apunta a un sitio vacío (404).

**Antes de editar un fichero de configuración, haz copia**: `sudo cp -a fichero fichero.orig`. Luego `diff` te dice qué has cambiado ([apartado 13](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b13)).

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 4. Buscar y filtrar texto :id=b4

### `find`: buscar ficheros por sus características

```bash
find /var/www -name "*.php"
find /var/www/solvia/public -name .htaccess
find /var/log -type f -mtime -1
find . -type f -name "*.conf" -exec grep -l "ServerName" {} \;
```

| Opción | Qué hace |
|---|---|
| `-name "<patrón>"` | Por nombre (distingue mayúsculas). `-iname` no las distingue |
| `-type f` / `-type d` / `-type l` | Solo ficheros / carpetas / enlaces simbólicos |
| `-mtime -1` / `-mtime +7` | Modificados hace menos de 1 día / hace más de 7 |
| `-size +10M` | Mayores de 10 MB |
| `-user <usuario>` | Cuyo dueño es ese usuario |
| `-perm <modo>` | Con esos permisos (`-perm 777`) |
| `-maxdepth <n>` | No baja más de *n* niveles |
| `-exec <comando> {} \;` | Ejecuta el comando sobre cada resultado (`{}` es el fichero) |

### `grep`: buscar texto dentro de ficheros

```bash
grep -n "ServerName" /etc/apache2/sites-available/solvia.conf
grep -rn "DocumentRoot" /etc/apache2/sites-available/
grep -v '^#' /etc/apache2/conf-available/security.conf
```

| Opción | Qué hace | Cuándo la usas |
|---|---|---|
| `-i` | Ignora mayúsculas | Buscar `address` sin importar cómo esté escrito |
| `-n` | Muestra el número de línea | Para saber dónde editar |
| `-v` | **Invierte**: líneas que NO contienen el patrón | Quitar comentarios (`grep -v '^#'`) |
| `-c` | Cuenta las líneas que coinciden | «¿Cuántas veces ocurre?» |
| `-l` | Solo el nombre de los ficheros que coinciden | Buscar en qué fichero está |
| `-r` | Recursivo en una carpeta. **No sigue los enlaces simbólicos que encuentra dentro** | Buscar en un árbol |
| `-R` | Como `-r`, pero **sí** sigue los enlaces simbólicos | Buscar en `sites-enabled/` (todo son enlaces) |
| `-E` | Expresiones regulares extendidas (`\|`, `+`, `?`, `()`) | `grep -E "error\|warn"` |
| `-o` | Imprime solo la parte que coincide | Extraer un valor |
| `-w` | Palabra completa | Que `80` no coincida con `8080` |
| `-F` | Texto literal, sin interpretar símbolos | Buscar algo con puntos o corchetes |
| `-A <n>` / `-B <n>` / `-C <n>` | *n* líneas **después** / **antes** / alrededor de cada coincidencia | Ver el contexto de un error |
| `--line-buffered` | No acumula la salida | Imprescindible si encadenas con `tail -f` |

**Trampa de `sites-enabled/`** (UD2): sus ficheros son enlaces simbólicos. `grep -r` los salta y «no encuentra nada». Usa `grep -R`, o un patrón: `grep DocumentRoot /etc/apache2/sites-enabled/*.conf`.

**Expresiones regulares mínimas**: `^` principio de línea · `$` final de línea · `.` un carácter cualquiera · `*` cero o más del anterior · `[0-9]` un dígito · `\.` un punto literal.

### Herramientas para trabajar la salida

| Comando | Qué hace | Ejemplo |
|---|---|---|
| `sort` | Ordena líneas. `-n` numérico, `-r` inverso, `-u` sin repetidos, `-k <n>` por la columna *n*, `-t <c>` separador | `sort -rn` |
| `uniq` | Colapsa líneas **consecutivas** repetidas. `-c` cuenta. **Hay que ordenar antes con `sort`** | `sort \| uniq -c` |
| `cut` | Extrae columnas. `-d <c>` separador, `-f <n>` campo | `cut -d: -f1 /etc/passwd` |
| `tr` | Cambia o borra caracteres. `-d '\r'` borra | `tr -d '\r'` |
| `awk` | Trabaja por columnas ([apartado 18](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b18)) | `awk '{print $1}'` |
| `sed` | Edita texto al vuelo. `-n 'Np'` imprime la línea *N*; `s/a/b/` sustituye; `-i` edita **el fichero** en el sitio; `-E` regex extendida; `/patrón/d` borra | `sed -n 1,4p fichero` |
| `tee` | Escribe a la vez en pantalla y en un fichero. `-a` añade | `sudo tee fichero` |
| `jq` | Lee JSON (hay que instalarlo). `jq -c '.campo'` | `jq '.general' informe.json` |

**El patrón que más se repite**: contar cuántas veces aparece cada cosa.

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -rn
```

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 5. Redirecciones, tuberías y atajos :id=b5

| Símbolo | Qué hace | Ejemplo |
|---|---|---|
| `\|` | Pasa la salida de un comando como entrada del siguiente | `ls \| wc -l` |
| `>` | Guarda la salida en un fichero y **lo sobrescribe** | `ls > lista.txt` |
| `>>` | **Añade** al final del fichero | `echo hola >> lista.txt` |
| `<` | Lee la entrada desde un fichero | `mysql solvia < schema.sql` |
| `2>` | Redirige solo los **errores** | `comando 2> errores.txt` |
| `2>&1` | Une los errores con la salida normal | `comando 2>&1 \| grep AH` |
| `2>/dev/null` | Tira los errores a la basura | `apache2ctl -S 2>/dev/null` |
| `$( )` | Sustituye por el resultado del comando | `echo "hoy es $(date)"` |
| `&&` | Ejecuta el siguiente **solo si** el anterior ha ido bien | `configtest && reload` |
| `\|\|` | Ejecuta el siguiente **solo si** el anterior ha fallado | |
| `;` | Ejecuta uno tras otro, falle o no | |
| `&` | Lanza el comando en segundo plano | `sleep 100 &` |

**El ciclo seguro de Apache**: `sudo apache2ctl configtest && sudo systemctl reload apache2` (solo recarga si la sintaxis es válida).

**Escribir en un fichero de `/etc` con `sudo`**: `sudo echo "x" > /etc/fichero` **no funciona**, porque la redirección `>` la hace tu shell, que no tiene permisos. Usa `tee`:

```bash
echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/servername.conf
```

**Escribir varias líneas de una vez** (documento *heredoc*): todo hasta la palabra final `EOF` va al fichero. Con comillas simples en `'EOF'`, el shell no interpreta `$` ni `` ` `` en el interior.

```bash
sudo tee /etc/apache2/sites-available/defecto.conf > /dev/null <<'EOF'
<VirtualHost *:80>
    ServerName defecto.test
</VirtualHost>
EOF
```

### Procesos en primer y segundo plano

| Tecla / comando | Qué hace |
|---|---|
| **Ctrl+C** | Corta el comando que está corriendo |
| **Ctrl+Z** | Lo suspende |
| `jobs` | Lista los comandos en segundo plano de esta terminal |
| `fg %1` / `bg %1` | Trae el trabajo 1 al primer plano / lo reanuda en segundo |
| `kill %1` | Lo mata |

### Atajos del teclado en la terminal

| Atajo | Qué hace |
|---|---|
| **Tab** | Autocompleta nombres. Dos veces: lista las opciones |
| **↑ / ↓** | Recorre el historial |
| **Ctrl+R** | Búsqueda inversa en el historial: escribes parte del comando y aparece |
| `history` | Lista los últimos comandos (con número). `history \| grep openssl` filtra |
| `!!` | Repite el comando anterior. Típico: `sudo !!` (el anterior, con `sudo`) |
| `!<n>` | Repite el comando número *n* del historial |
| **Ctrl+A / Ctrl+E** | Principio / final de la línea |
| **Ctrl+U / Ctrl+W** | Borra hasta el principio de la línea / la última palabra |
| **Ctrl+L** (o `clear`) | Limpia la pantalla |
| **Ctrl+D** | Cierra la sesión (o el intérprete en el que estés) |

El historial permite documentar lo que hiciste: `history | tail -n 30` te da los últimos treinta comandos para pegarlos en tu documentación o bitácora (revisa que no incluyan contraseñas).

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 6. Permisos y propietarios :id=b6

Cada fichero tiene un **dueño**, un **grupo** y tres juegos de permisos: para el dueño (**u**), para el grupo (**g**) y para el resto (**o**).

```plaintext
-rw-r----- 1 root www-data 689 nov  9 10:21 config.php
│└┬┘└┬┘└┬┘   │    │
│ │  │  │    │    └─── grupo
│ │  │  │    └──────── dueño
│ │  │  └───────────── permisos del resto (o):   ---  nada
│ │  └──────────────── permisos del grupo (g):   r--  leer
│ └─────────────────── permisos del dueño (u):   rw-  leer y escribir
└───────────────────── tipo: - fichero · d carpeta · l enlace simbólico
```

| Permiso | Letra | Valor | En un fichero | En una carpeta |
|---|---|---|---|---|
| Lectura | `r` | 4 | Leerlo | Listar su contenido |
| Escritura | `w` | 2 | Modificarlo | Crear o borrar cosas dentro |
| Ejecución | `x` | 1 | Ejecutarlo | **Atravesarla** (entrar y llegar a lo que hay debajo) |

Sumas los valores de cada juego y obtienes el modo octal: `rw-` = 4+2 = **6**, `r-x` = 4+1 = **5**.

| Modo | Significa | Se usa para |
|---|---|---|
| `644` | dueño lee/escribe, resto solo lee | Ficheros públicos |
| `755` | dueño todo, resto lee y entra | Carpetas públicas y programas |
| `600` | solo el dueño | **Claves privadas** |
| `640` | dueño lee/escribe, grupo lee | Ficheros con secretos que Apache debe leer (`config.php`, `.htpasswd`) |
| `750` | dueño todo, grupo lee y entra | Carpetas de secretos |

### `chmod`, `chown`, `stat`, `namei`

```bash
sudo chmod 600 /etc/apache2/ssl/solvia.key
sudo chmod u+x script.sh
sudo chmod -R o+rX /var/www/solvia
sudo chown root:www-data /var/www/solvia/config/config.php
sudo chown -R www-data:www-data /var/www/solvia/uploads
namei -l /var/www/solvia/public/index.php
```

| Comando / opción | Qué hace | Cuándo lo usas |
|---|---|---|
| `chmod <modo> <ruta>` | Cambia los permisos. El modo puede ser octal (`640`) o simbólico (`u+x`, `g-w`, `o=r`: quién `u`/`g`/`o`/`a` + `+`/`-`/`=` + qué `r`/`w`/`x`) | |
| `chmod -R` | Recursivo: también lo que hay dentro | Una carpeta entera |
| `X` mayúscula (`o+rX`) | Da `x` **solo a las carpetas** (y a lo que ya era ejecutable) | Abrir una carpeta sin hacer ejecutables los ficheros normales |
| `chown <dueño>:<grupo> <ruta>` | Cambia el dueño y el grupo | `root:www-data` = solo root escribe, Apache lee |
| `chown -R` | Recursivo | |
| `stat -c '%a %U:%G %n' <ruta>` | Permisos octales, dueño:grupo y nombre en una línea | Comprobar rápido varias rutas |
| `namei -l <ruta>` | Muestra los permisos de **cada carpeta del camino** hasta el fichero | Diagnosticar un **403** |

**Por qué da 403 un proyecto en `/home`** (UD1): para leer un fichero, Apache (usuario `www-data`) tiene que poder **atravesar todas las carpetas del camino**. `/home/<usuario>` no deja pasar a otros usuarios. Por eso el proyecto vive en `/var/www`. `namei -l` lo enseña: busca la carpeta del camino donde falta la `x` para `www-data`.

**Probar como si fueras otro usuario**: `sudo runuser -u www-data -- cat /var/www/solvia/config/config.php` ejecuta el comando como `www-data` (el usuario con el que corre Apache). Si da «Permission denied», Apache tampoco podrá leerlo.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 7. Usuarios, grupos y `sudo` :id=b7

| Comando | Qué hace | Opciones útiles |
|---|---|---|
| `whoami` | Quién soy | |
| `id [usuario]` | UID, GID y grupos | |
| `groups [usuario]` | Grupos a los que pertenece | |
| `sudo <comando>` | Ejecuta **un** comando como administrador, pide tu contraseña y queda registrado | `-u <usuario>` lo ejecuta como ese usuario; `-l` lista lo que puedes hacer con sudo; `-i` abre una sesión de root (evítalo) |
| `su - <usuario>` | Cambia de usuario con su entorno | |
| `sudo adduser <usuario>` | Crea un usuario con su carpeta personal | |
| `sudo usermod -aG <grupo> <usuario>` | **Añade** un usuario a un grupo. Sin `-a` le **quita** de los demás | |
| `sudo passwd <usuario>` | Cambia la contraseña | |
| `sudo deluser <usuario>` | Borra un usuario | `--remove-home` también su carpeta |
| `w` / `who` | Quién está conectado | |

**Por qué `sudo` y no trabajar como root**: como root un comando mal escrito se ejecuta sin red de seguridad y sin rastro. Si el prompt acaba en `#` (`root@servidor:~#`) estás como root; si acaba en `$`, como usuario normal. Para salir, `exit`.

**`www-data`** no es una persona: es el usuario con el que corren Apache y PHP. Lo que él no pueda leer, Apache no lo sirve, aunque tú sí puedas verlo.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 8. Paquetes con `apt` :id=b8

| Comando | Qué hace |
|---|---|
| `sudo apt update` | Descarga la **lista** de paquetes disponibles. No instala ni actualiza nada |
| `sudo apt upgrade` | Actualiza los paquetes ya instalados a la última versión |
| `sudo apt install <paquete>` | Instala el paquete y sus dependencias. `-y` responde «sí» a todo |
| `sudo apt remove <paquete>` | Desinstala, **conserva** la configuración |
| `sudo apt purge <paquete>` | Desinstala y **borra** la configuración |
| `sudo apt autoremove` | Quita dependencias que ya nadie usa |
| `apt search <palabra>` | Busca paquetes |
| `apt show <paquete>` | Descripción, dependencias y tamaño |
| `apt list --installed` | Paquetes instalados (filtra con `\| grep`) |
| `apt-cache policy <paquete>` | Versión instalada, versión disponible y de qué repositorio. **Sirve para comprobar si un paquete existe** |
| `dpkg -L <paquete>` | Lista los ficheros que instaló el paquete (¿dónde puso su configuración?) |
| `dpkg -S <ruta>` | ¿De qué paquete es este fichero? |
| `lsb_release -a` | Versión de Ubuntu |

**Orden correcto**: `apt update` primero, o trabajas con un catálogo viejo y `apt` puede decir que un paquete no existe cuando sí existe.

**Paquetes de UD1 y para qué sirve cada uno**:

| Paquete | Qué es |
|---|---|
| `apache2` | El servidor web |
| `libapache2-mod-php` | El **puente**: permite a Apache ejecutar PHP en vez de servirlo como texto |
| `php-mysql` | El **conector**: permite que el código PHP hable con MariaDB |
| `mariadb-server` | El **motor** de base de datos |

`mariadb-server` y `php-mysql` son cosas distintas: puede haber motor sin conector, y entonces la web da error 500 con `could not find driver`.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 9. Servicios con `systemctl` y registros con `journalctl` :id=b9

Un **servicio** es un programa que corre en segundo plano sin ventana (Apache, MariaDB, Tomcat). `systemctl` es su mando a distancia.

| Comando | Qué hace |
|---|---|
| `systemctl status <servicio> --no-pager` | ¿Está vivo, desde cuándo, y qué dicen sus últimas líneas de log? Busca `active (running)`. `--no-pager` evita el visor del que hay que salir con **q** |
| `sudo systemctl start <servicio>` | Lo arranca |
| `sudo systemctl stop <servicio>` | Lo para |
| `sudo systemctl restart <servicio>` | Lo para y lo arranca (**corta las conexiones**) |
| `sudo systemctl reload <servicio>` | Relee la configuración **sin cortar** conexiones |
| `sudo systemctl enable <servicio>` | Hace que arranque solo al encender |
| `sudo systemctl enable --now <servicio>` | Lo activa en el arranque **y** lo arranca ahora |
| `sudo systemctl disable <servicio>` | Quita el arranque automático |
| `systemctl is-active <servicio>` | Responde `active` o `inactive` (sirve en scripts) |
| `systemctl is-enabled <servicio>` | Responde si arranca solo |
| `systemctl list-units --type=service --state=running` | Servicios en marcha |
| `sudo systemctl daemon-reload` | Relee los ficheros de definición de servicios (si has creado o cambiado uno) |

**¿`reload` o `restart`?** Regla de bolsillo: **configuración → `reload`; software o módulo nuevo → `restart`**. Con `restart` en un servidor real cortas a los usuarios conectados.

### `journalctl`: los registros de systemd

| Comando / opción | Qué hace |
|---|---|
| `journalctl -u <servicio>` | Registros de ese servicio |
| `-n <N>` | Solo las últimas *N* líneas |
| `-f` | Sigue en vivo (Ctrl+C para salir) |
| `--since "10 min ago"` | Desde hace 10 minutos (también `"today"`, `"2026-11-10 09:00"`) |
| `-p err` | Solo errores o más graves |
| `-b` | Solo desde el último arranque |
| `--no-pager` | Imprime sin visor |

Cuando un servicio **no arranca**, la primera pista es: `systemctl status <servicio> --no-pager` y después `journalctl -u <servicio> -n 30 --no-pager`.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 10. Procesos y recursos :id=b10

| Comando | Qué hace | Opciones útiles |
|---|---|---|
| `ps aux` | Todos los procesos, con usuario, CPU y memoria | Filtra con `\| grep <nombre>` |
| `ps -eo user,comm` | Usuario y nombre de cada proceso | `ps -eo user,comm \| grep apache2 \| sort \| uniq -c` muestra qué usuarios ejecutan Apache |
| `pgrep -a <nombre>` | PID y comando de los procesos con ese nombre | |
| `top` | Procesos en tiempo real. **q** para salir | **M** ordena por memoria, **P** por CPU |
| `kill <PID>` | Pide al proceso que termine | `kill -9 <PID>` lo mata sin contemplaciones (último recurso) |
| `uptime` | Cuánto lleva encendido y carga media | |
| `free -h` | Memoria | |
| `df -h` | Disco | |

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 11. Red :id=b11

| Comando | Qué hace | Opciones útiles |
|---|---|---|
| `ip a` | Interfaces y direcciones IP | `ip -br a` formato resumido |
| `ip r` | Tabla de rutas (la puerta de enlace es la línea `default`) | |
| `hostname -I` | Solo las IP de esta máquina | |
| `ping <host>` | ¿Llega el tráfico? | `-c 4` envía 4 y para (sin `-c` hay que cortar con Ctrl+C) |
| `ss -tlnp` | **Qué escucha en qué puerto** | `-t` TCP · `-l` solo los que escuchan · `-n` números en vez de nombres · `-p` qué proceso (necesita `sudo` para ver los de otros) · `-u` UDP |
| `nc -zv <host> <puerto>` | ¿Está abierto ese puerto? | `-z` solo comprueba, `-v` explica |
| `wget <url>` | Descarga un fichero | `-O <nombre>` lo guarda con ese nombre; `-q` silencioso |
| `curl` | Cliente HTTP ([apartado 15](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b15)) | |
| `sudo tcpdump` | Captura tráfico ([apartado 14.9](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b14-9)) | |
| `resolvectl status` | Qué servidores DNS usa la máquina | |

`ss -tlnp | grep :80` responde a «¿hay alguien escuchando en el 80?». Que un proceso esté vivo **no** significa que escuche donde crees.

### Puertos que hay que reconocer

| Puerto | Servicio |
|---|---|
| 22 | SSH / SFTP |
| 21 | FTP (control) |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 389 | LDAP |
| 443 | HTTPS |
| 636 | LDAPS (LDAP sobre TLS) |
| 3306 | MariaDB / MySQL |
| 8009 | AJP de Tomcat |
| 8080 | Tomcat (HTTP) |
| 8443 | Tomcat (HTTPS) |

### Nombres de prueba: `/etc/hosts`

El fichero `/etc/hosts` asocia nombres a direcciones **solo en esta máquina**, antes de consultar ningún DNS. Una línea por entrada: `<IP> <nombre> [alias...]`.

```bash
echo "127.0.0.1 solvia.test informes.solvia.test" | sudo tee -a /etc/hosts
```

Se usa el dominio `.test` (reservado para pruebas) y **no** `.local`, que el sistema reserva para mDNS.

### La red de VirtualBox

| Dirección | Qué es |
|---|---|
| `10.0.2.2` | Desde la VM, **el anfitrión (Windows)** vía NAT. Sirve para descargar con `wget` un fichero servido desde Windows (`python -m http.server 8000`) |
| Reenvío de puertos | Para llegar **de Windows a la VM**: VirtualBox → Configuración → Red → Adaptador 1 → Avanzadas → Reenvío de puertos (p. ej. anfitrión 8080 → invitado 80); luego `http://localhost:8080` en el navegador de Windows |

En un log de Apache, `client 10.0.2.2` es una petición que viene del navegador de Windows; `::1` o `127.0.0.1` viene de dentro de la propia VM.

### Cortafuegos `ufw`

| Comando | Qué hace |
|---|---|
| `sudo ufw status` | Estado y reglas |
| `sudo ufw allow <puerto>/tcp` | Permite entrada a ese puerto |
| `sudo ufw deny <puerto>/tcp` | La bloquea |
| `sudo ufw delete allow <puerto>/tcp` | Borra una regla |
| `sudo ufw enable` / `disable` | Activa / desactiva el cortafuegos |

**Antes de `ufw enable` por SSH, permite el 22** (`sudo ufw allow 22/tcp`) o te quedas fuera.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 12. Comprimir y empaquetar :id=b12

| Comando | Qué hace |
|---|---|
| `tar -czvf <nombre.tar.gz> <ruta>` | **C**rea un paquete comprimido: `c` crear · `z` comprimir con gzip · `v` mostrar lo que hace · `f` nombre del fichero (la `f` va **la última**, pegada al nombre) |
| `tar -xzvf <nombre.tar.gz>` | E**x**trae. `-C <carpeta>` extrae en esa carpeta |
| `tar -tzf <nombre.tar.gz>` | Lista el contenido **sin extraer** (`t`) |
| `tar -czpf <n.tar.gz> -C /etc apache2` | `-p` conserva permisos; `-C /etc` cambia a esa carpeta antes de empaquetar (así dentro queda `apache2/...` y no `/etc/apache2/...`) |
| `zip -r <nombre.zip> <carpeta>` | Crea un zip de una carpeta |
| `unzip <nombre.zip>` | Descomprime. `-l` solo lista; `-d <carpeta>` destino |
| `zcat <fichero.gz>` | Lee un fichero comprimido `.gz` sin descomprimirlo (los logs rotados lo están) |
| `zgrep <patrón> <fichero.gz>` | `grep` dentro de un `.gz` |

**Copia de seguridad de la configuración antes de tocar nada**: `sudo tar -czpf /root/apache2-conf-$(date +%F).tar.gz -C /etc apache2`.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 13. Editar con `nano` y comparar con `diff` :id=b13

```bash
sudo nano /etc/apache2/sites-available/solvia.conf
```

| Atajo | Qué hace |
|---|---|
| **Ctrl+O**, **Enter** | Guardar (Enter confirma el nombre) |
| **Ctrl+X** | Salir. Si hay cambios sin guardar, pregunta: `S` guardar, `N` descartar |
| **Ctrl+W** | Buscar (Ctrl+W otra vez, siguiente) |
| **Ctrl+K** / **Ctrl+U** | Cortar la línea / pegar |
| **Alt+U** | Deshacer |
| **Ctrl+_** | Ir a una línea (te pide el número) |
| **Ctrl+C** | Muestra en qué línea estás |

Los atajos aparecen en la barra de abajo con `^` en lugar de Ctrl y `M-` en lugar de Alt. `nano -l <fichero>` muestra los números de línea.

**Si no puedes guardar**, lo primero que hay que mirar es si abriste con `sudo`: los ficheros de `/etc` son de root y sin `sudo` nano se abre en modo solo lectura.

### `diff`

```bash
sudo cp -a /etc/apache2/conf-available/security.conf ~/security.conf.orig
# ... editas ...
diff ~/security.conf.orig /etc/apache2/conf-available/security.conf
diff -u ~/solvia.conf.antes /etc/apache2/sites-available/solvia.conf
```

| Opción | Qué hace |
|---|---|
| `diff <A> <B>` | Líneas que difieren: `<` es de A, `>` es de B |
| `-u` | Formato unificado: `-` línea eliminada, `+` línea añadida, con contexto. Es el que usa Git |
| `-r` | Compara carpetas enteras |
| `-q` | Solo dice si difieren |

`diff` sin diferencias **no imprime nada**: eso significa «iguales».

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 14. Apache (UD1-UD2) :id=b14

### 14.1 Dónde está cada cosa en Ubuntu

| Ruta | Qué contiene |
|---|---|
| `/etc/apache2/apache2.conf` | Configuración principal. Incluye al resto |
| `/etc/apache2/ports.conf` | Puertos en los que escucha (`Listen 80`; el 443 se añade al activar `ssl`) |
| `/etc/apache2/envvars` | Variables de entorno (usuario `www-data`, ruta de logs) |
| `/etc/apache2/sites-available/` | Sitios **definidos** |
| `/etc/apache2/sites-enabled/` | Sitios **activos**: son **enlaces simbólicos** a los de `sites-available` |
| `/etc/apache2/mods-available/` y `mods-enabled/` | Igual, para módulos |
| `/etc/apache2/conf-available/` y `conf-enabled/` | Igual, para fragmentos de configuración global (`security.conf`, `servername.conf`) |
| `/var/www/` | Donde viven los sitios (`/var/www/solvia/public` es el `DocumentRoot` de Solvia) |
| `/var/log/apache2/` | `error.log`, `access.log` y los de cada sitio (`solvia_error.log`, `solvia_access.log`) |
| `/etc/logrotate.d/apache2` | Cómo se rotan los logs |

**El patrón *available / enabled***: se **define** en `*-available` y se **activa** con un enlace en `*-enabled`. Permite tener algo escrito y desactivado sin borrarlo. `a2ensite`, `a2enmod` y `a2enconf` solo crean ese enlace.

### 14.2 El ciclo de trabajo (la regla de oro de Apache)

```plaintext
EDITAR  →  CONFIGTEST  →  RELOAD  →  COMPROBAR
```

```bash
sudo nano /etc/apache2/sites-available/solvia.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -sI http://solvia.test/
```

- **Nunca recargues sin `configtest`.** Con una errata, `reload` se niega y Apache sigue sirviendo la configuración anterior: parece que «no pasa nada» pero tu cambio no está aplicado.
- **`Syntax OK` solo comprueba la sintaxis.** No detecta, por ejemplo, un `DocumentRoot` que no existe (da un aviso `AH00112` pero sigue en `Syntax OK`) ni una clave TLS que no casa con su certificado (falla al arrancar).
- **Un cambio de configuración → `reload`.** Un módulo nuevo o un cambio de puertos → `restart`.
- **Si tocas algo, comprueba que ha cambiado** (con `curl`, con `apache2ctl -S`...). «Ha dado `Syntax OK`» no significa «hace lo que quería».
- **Antes de tocar, copia**: `sudo cp -a <fichero> <fichero>.orig`, y luego `diff` ([apartado 13](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b13)).

### 14.3 `apache2ctl`

| Comando | Qué hace | Cuándo lo usas |
|---|---|---|
| `sudo apache2ctl configtest` (o `-t`) | Comprueba la sintaxis de **toda** la configuración sin aplicarla. La última línea debe ser `Syntax OK` | Siempre antes de recargar |
| `sudo apache2ctl -S` | Muestra los sitios virtuales **realmente cargados**, a qué puerto y nombre responde cada uno, cuál es el **por defecto**, y el usuario/grupo de Apache | Saber qué sitio contesta a qué. Añade `2>/dev/null` para quitar el aviso AH00558 |
| `sudo apache2ctl -M` | Lista los módulos cargados | `apache2ctl -M \| grep headers` |
| `apache2ctl -V` | Versión, MPM, rutas compiladas | `apache2ctl -V \| grep -E "Server version\|Server MPM"` |
| `apache2ctl -l` | Módulos compilados dentro del binario | |
| `sudo apache2ctl -t -D DUMP_INCLUDES` | Muestra **en orden de lectura** todos los ficheros que Apache incluye | Ver quién pisa a quién |
| `sudo apache2ctl graceful` | Reinicio suave: termina las peticiones en curso antes de recargar | Equivale en efecto a `reload` |

**Mensajes que hay que saber leer:**

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| `Syntax OK` | La sintaxis es válida | Puedes recargar |
| `AH00558: Could not reliably determine the server's fully qualified domain name` | **Es un aviso, no un error.** Apache no tiene un `ServerName` global y usa `127.0.0.1` | Opcional: `echo "ServerName localhost" \| sudo tee /etc/apache2/conf-available/servername.conf && sudo a2enconf servername` |
| `AH00526: Syntax error on line N of <fichero>: Invalid command 'X', perhaps misspelled or defined by a module not included` | Directiva mal escrita, **o** falta el módulo que la define | Revisa la errata, o activa el módulo (`a2enmod`) |
| `AH00112: Warning: DocumentRoot [...] does not exist` | La ruta del `DocumentRoot` no existe. Es un aviso: la sintaxis sigue siendo `OK` | `ls` la ruta; ¿error de escritura? |
| `AH01276: Cannot serve directory ...: No matching DirectoryIndex ... server-generated directory index forbidden by Options directive` | Piden una carpeta sin `index` y `-Indexes` impide listarla | Es el comportamiento esperado con `Options -Indexes` |

### 14.4 Activar y desactivar: `a2*`

| Comando | Qué hace |
|---|---|
| `sudo a2ensite <sitio>` / `a2dissite <sitio>` | Activa / desactiva un sitio (crea / borra el enlace en `sites-enabled`). `<sitio>` es el nombre con o sin `.conf` |
| `sudo a2enmod <módulo>` / `a2dismod <módulo>` | Activa / desactiva un módulo. Activa también sus dependencias. `a2dismod` pide confirmación para módulos esenciales |
| `sudo a2enconf <conf>` / `a2disconf <conf>` | Activa / desactiva un fragmento de `conf-available` |
| `a2query -m` | Lista los módulos activos. `a2query -m <módulo>` comprueba si uno concreto lo está |
| `a2query -s` | Lista los sitios activos (`a2query -s <sitio>`: uno concreto) |
| `a2query -c` | Lista las configuraciones activas |

Después de cualquiera de ellos: `configtest` y `reload` (o `restart` si es un módulo). **El comando por sí solo no cambia el Apache que está corriendo.**

### 14.5 Directivas de uso habitual

**Identidad y arranque**

| Directiva | Qué hace |
|---|---|
| `ServerName <nombre>` | El nombre con el que se identifica un sitio. Apache lo compara con la cabecera `Host` de la petición para **elegir el sitio** |
| `ServerAlias <nombre> ...` | Otros nombres que también atiende ese sitio |
| `DocumentRoot <ruta>` | La carpeta del disco que se sirve como raíz. **Todo lo que cuelga de ella es público** |
| `Listen <puerto>` | En qué puerto escucha (en `ports.conf`) |
| `ServerTokens Prod` | Cuánta información da la cabecera `Server`: `Prod` solo «Apache»; el valor de serie (`OS`) añade versión y sistema operativo |
| `ServerSignature Off` | Quita la firma con la versión en las páginas de error |
| `TraceEnable Off` | Desactiva el método `TRACE` |
| `LogLevel <nivel>` | Nivel de detalle del log de errores (`warn`, `info`, `debug`...) |

**Carpetas y acceso**

| Directiva | Qué hace |
|---|---|
| `<Directory /ruta> ... </Directory>` | Reglas para una carpeta del disco |
| `<Location /url> ... </Location>` | Reglas para una URL (útil para `/server-status`) |
| `<Files "x"> ... </Files>` / `<FilesMatch "regex">` | Reglas para ficheros por nombre |
| `Require all granted` / `Require all denied` | Permite / prohíbe a todos |
| `Require ip <IP o rango>` / `Require local` | Solo desde esas IP / solo desde la propia máquina |
| `Require valid-user` / `Require user <nombre>` | Cualquier usuario autenticado / solo ese |
| `Options -Indexes` | Quita el listado automático de carpetas. El `-` resta a lo heredado. **O todas las opciones llevan `+`/`-`, o ninguna** |
| `Options -ExecCGI` | Sin ejecución de CGI (peligroso en carpetas de subidas) |
| `AllowOverride None` | Ignora los `.htaccess` (más seguro y más rápido). `All` los permite |
| `Alias /url /ruta` | Publica una carpeta del disco bajo una URL (cuidado: expone lo que haya dentro) |
| `ErrorDocument 404 /404.html` | Página de error propia |

**Registros** (`LogFormat` define formatos; `CustomLog` y `ErrorLog` eligen dónde se escribe)

| Directiva | Qué hace |
|---|---|
| `ErrorLog ${APACHE_LOG_DIR}/solvia_error.log` | Fichero de errores de este sitio |
| `CustomLog ${APACHE_LOG_DIR}/solvia_access.log combined` | Fichero de accesos con el formato `combined` |

**HTTPS** (requiere `a2enmod ssl`)

| Directiva | Qué hace |
|---|---|
| `SSLEngine on` | Activa TLS en el sitio |
| `SSLCertificateFile <ruta>` | Certificado (público) |
| `SSLCertificateKeyFile <ruta>` | Clave privada (nunca sale del servidor) |
| `SSLProtocol -all +TLSv1.2 +TLSv1.3` | Versiones de TLS aceptadas |
| `Redirect permanent / https://solvia.test/` | Redirección 301 de HTTP a HTTPS (`Redirect 308 ...` conserva el `POST`) |
| `Header always set <Nombre> "<valor>"` | Añade una cabecera a todas las respuestas, errores incluidos (requiere `a2enmod headers`). `always` es el que hace que salga también en errores |

**Autenticación Basic**

| Directiva | Qué hace |
|---|---|
| `AuthType Basic` | Método de autenticación |
| `AuthName "<texto>"` | Texto de la ventana de acceso (el *realm*) |
| `AuthUserFile <ruta>` | Fichero de usuarios (ruta absoluta, **fuera del `DocumentRoot`**) |
| `Require valid-user` | Quién entra |

**Cabeceras de seguridad habituales** (en el sitio HTTPS): `X-Content-Type-Options "nosniff"`, `Referrer-Policy "strict-origin-when-cross-origin"`, `Content-Security-Policy "..."` (se escribe a la medida de la aplicación), `Strict-Transport-Security "max-age=300"` (HSTS: **empieza con un `max-age` corto**, porque con HSTS el navegador no deja saltarse un aviso de certificado).

**Orden de las directivas globales**: en las directivas de contexto global gana **la última que Apache lee**. Con `sudo apache2ctl -t -D DUMP_INCLUDES` ves el orden real.

### 14.6 Plantilla de sitio HTTP (sitio de Solvia, UD1-UD2)

```apache
<VirtualHost *:80>
        ServerName solvia.test
        DocumentRoot /var/www/solvia/public

        <Directory /var/www/solvia/public>
                Options -Indexes
                AllowOverride None
                Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/solvia_error.log
        CustomLog ${APACHE_LOG_DIR}/solvia_access.log combined
</VirtualHost>
```

**La decisión de seguridad central**: el `DocumentRoot` apunta a `public/`, **no** a la raíz del proyecto. Todo lo que cuelga del `DocumentRoot` es público: si apuntara a `/var/www/solvia`, cualquiera podría pedir `/config/config.php` y llevarse las credenciales de la base de datos.

### 14.7 Plantilla de sitio HTTPS con redirección (UD2)

```apache
<VirtualHost *:80>
        ServerName solvia.test
        Redirect permanent / https://solvia.test/
</VirtualHost>

<VirtualHost *:443>
        ServerName solvia.test
        DocumentRoot /var/www/solvia/public

        SSLEngine on
        SSLProtocol -all +TLSv1.2 +TLSv1.3
        SSLCertificateFile    /etc/apache2/ssl/solvia.crt
        SSLCertificateKeyFile /etc/apache2/ssl/solvia.key

        Header always set X-Content-Type-Options "nosniff"
        Header always set Strict-Transport-Security "max-age=300"

        ErrorLog ${APACHE_LOG_DIR}/solvia_error.log
        CustomLog ${APACHE_LOG_DIR}/solvia_access.log combined
</VirtualHost>
```

### 14.8 Plantilla de zona protegida con usuario y contraseña (UD2)

```apache
<Directory /var/www/informes>
        AuthType Basic
        AuthName "Informes de Solvia"
        AuthUserFile /etc/apache2/.htpasswd-informes
        Require valid-user
</Directory>
```

### 14.9 `tcpdump`: ver qué viaja por la red :id=b14-9

Sirve para **demostrar** la diferencia entre HTTP y HTTPS: por el puerto 80 se lee el contenido entero; por el 443 no se ve nada legible.

```bash
sudo tcpdump -l -i lo -A -s0 'tcp port 80' | grep -a --line-buffered "Authorization"
sudo tcpdump -i lo -n -w /tmp/c443.pcap 'tcp port 443' &
sudo tcpdump -nr /tmp/c443.pcap -A | grep -a -c "Authorization"
```

| Opción | Qué hace |
|---|---|
| `-i <interfaz>` | Qué interfaz escucha. `lo` es la propia máquina; `any` todas |
| `-n` | No resuelve nombres (más rápido y sin ruido) |
| `-A` | Muestra el contenido de cada paquete como texto ASCII |
| `-s0` | Captura el paquete entero (sin recortarlo) |
| `-w <fichero>` | Guarda la captura en un fichero |
| `-r <fichero>` | Lee una captura guardada |
| `-l` | Salida línea a línea (necesaria para encadenar con `grep`) |
| `'tcp port 80'` | **Filtro**: solo el tráfico TCP del puerto 80 |
| `sudo` | Obligatorio: capturar tráfico requiere permisos de administrador |

Con `&` se lanza la captura en segundo plano; para pararla, `sudo kill %1`.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 15. `curl` y códigos HTTP (UD1-UD2) :id=b15

`curl` es un navegador sin ventana: lanza una petición HTTP y te enseña la respuesta en crudo. Es la herramienta de comprobación del módulo.

```bash
curl -I http://localhost/                                    # solo cabeceras
curl -s http://solvia.test/ | head -n 20                     # el HTML, sin barra de progreso
curl -s -o /dev/null -w "%{http_code}\n" http://localhost/  # solo el código de estado
```

| Opción | Qué hace | Cuándo la usas |
|---|---|---|
| `-s` | Silencioso: sin barra de progreso ni errores | Casi siempre, junto a `-o` o `\| head` |
| `-S` | Con `-s`, muestra igualmente los errores | `-sS` |
| `-I` | Pide solo las cabeceras (petición `HEAD`) | Ver el código y cabeceras sin el HTML |
| `-i` | Incluye las cabeceras de la respuesta **además** del cuerpo | Ver cabeceras y página |
| `-v` | Detallado: muestra la petición (`>`) y la respuesta (`<`), y el saludo TLS. **No muestra el cuerpo** de un `POST` | Depurar. Para ver el cuerpo: `--trace-ascii -` |
| `-o <fichero>` | Guarda la respuesta en un fichero. `-o /dev/null` la descarta | Con `-w` para quedarte solo con el código |
| `-O` | Guarda con el nombre que tiene en el servidor | Descargas |
| `-w "<formato>"` | Imprime datos al final. `%{http_code}` código de estado; `%{time_total}` tiempo total; `%{url_effective}` URL final | Pruebas automáticas |
| `-L` | Sigue las redirecciones (301/302/308). Sin `-L`, `curl` se para en la primera | Con HTTP→HTTPS |
| `-H "<Cabecera: valor>"` | Añade una cabecera. `-H "Host: informes.solvia.test"` | Probar un sitio virtual sin tocar `/etc/hosts` |
| `--resolve <nombre>:<puerto>:<IP>` | Obliga a que ese nombre resuelva a esa IP, sin DNS ni `/etc/hosts` | `--resolve otro.test:443:127.0.0.1` |
| `-u <usuario>:<contraseña>` | Autenticación Basic | Zona protegida |
| `-d "<datos>"` | Envía un `POST` con esos datos (`campo=valor&otro=valor`) | Probar un formulario |
| `--data-urlencode "<campo=valor>"` | Igual, codificando caracteres especiales | Valores con espacios o tildes |
| `-X <MÉTODO>` | Fuerza el método (`-X TRACE`, `-X DELETE`) | Comprobar que `TRACE` da 405 |
| `-k` | **No verifica** el certificado | Solo para ver si algo responde; nunca como solución |
| `--cacert <fichero>` | Verifica el certificado contra esa CA | HTTPS con la CA del aula |
| `--tlsv1.2 --tls-max 1.2` | Fija la versión mínima y máxima de TLS | Comprobar qué versiones acepta el servidor |
| `-c <fichero>` / `-b <fichero>` | Guarda / envía las cookies | Probar sesiones |
| `-m <segundos>` | Tiempo máximo de toda la operación | Evitar cuelgues |
| `--trace-ascii -` | Vuelca todo lo enviado y recibido, **cuerpo incluido** | Demostrar que un `POST` por HTTP viaja en claro |

**Errores típicos de `curl`:**

| Salida | Significa |
|---|---|
| `curl: (7) Failed to connect ... Connection refused` | Nadie escucha: Apache parado o en otro puerto. Vuelve a `systemctl status` y `ss -tlnp` |
| `curl: (6) Could not resolve host` | El nombre no resuelve: falta la entrada en `/etc/hosts` (o DNS) |
| `curl: (60) SSL certificate problem` | El certificado no es de confianza: usa `--cacert` con la CA correcta |
| `curl: (35)` / `(56)` | Fallo en el saludo TLS (protocolo, certificado, cifrado) |

### Códigos de estado HTTP: dónde mirar

| Código | Significa | Dónde está el problema |
|---|---|---|
| Sin código, «Connection refused» | No ha habido conversación | El servicio no corre o no escucha: `systemctl status`, `ss -tlnp` |
| **200** OK | Responde | Nivel HTTP superado. No dice nada de si la **aplicación** funciona |
| **301** / **302** / **308** | Redirección (`Location:` dice a dónde) | Usa `curl -IL` para seguirla |
| **401** Unauthorized | Falta autenticación o es incorrecta | Usuario/contraseña, `.htpasswd` |
| **403** Forbidden | Prohibido | **Permisos** del sistema (¿`www-data` puede atravesar el camino?: `namei -l`), o `Require`, o `-Indexes` |
| **404** Not Found | No existe esa ruta | `DocumentRoot` mal apuntado, fichero en otro sitio |
| **405** Method Not Allowed | Método no permitido | Esperado con `TRACE` desactivado |
| **500** Internal Server Error | Fallo del servidor/código | **Siempre está en el log**, nunca en el navegador: `tail` del `*_error.log` |
| **502** / **503** | Bad Gateway / servicio no disponible | El proxy no alcanza a Tomcat (UD3) o el servicio está caído |

**La escalera de diagnóstico (cuatro niveles; se sube en orden):**

| Nivel | Pregunta | Comando |
|---|---|---|
| 1 | ¿El proceso vive? | `systemctl status apache2 --no-pager` → `active (running)` |
| 2 | ¿Escucha en el puerto? | `ss -tlnp \| grep :80` |
| 3 | ¿Responde por HTTP y con qué código? | `curl -I http://localhost/` |
| 4 | ¿La aplicación hace su trabajo? | `curl -s http://localhost/ \| head -n 20` (¿salen los tickets?) |

Que la portada cargue demuestra que Apache sirve un fichero, no que la aplicación funcione.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 16. Certificados y HTTPS con `openssl` (UD2) :id=b16

Conceptos mínimos: un certificado es una **clave pública** firmada por una **CA** (autoridad certificadora). El servidor guarda la **clave privada** (`.key`, nunca sale de la máquina) y enseña el **certificado** (`.crt`, público). Una **CSR** (`.csr`) es la petición de firma que se envía a la CA. El **SAN** (*Subject Alternative Name*) lista los nombres para los que vale el certificado: **hoy los navegadores solo miran el SAN, no el `CN`**.

### Crear

```bash
# Certificado autofirmado (clave + certificado de una vez)
openssl req -x509 -newkey rsa:2048 -nodes -keyout solvia-auto.key -out solvia-auto.crt -days 365 \
  -subj "/CN=solvia.test" -addext "subjectAltName=DNS:solvia.test,DNS:informes.solvia.test"

# CA propia (la «CA del aula»)
openssl req -x509 -newkey rsa:4096 -nodes -keyout ca-aula.key -out ca-aula.crt -days 1825 \
  -subj "/O=IES Rio Arba/CN=CA del aula" \
  -addext "basicConstraints=critical,CA:TRUE" -addext "keyUsage=critical,keyCertSign,cRLSign"

# Clave + petición de firma (CSR) para que la CA firme
openssl req -new -newkey rsa:2048 -nodes -keyout solvia.key -out solvia.csr \
  -subj "/CN=solvia.test" -addext "subjectAltName=DNS:solvia.test,DNS:informes.solvia.test"

# Firmar la CSR con la CA (solvia.ext lleva los SAN y los usos de la clave)
openssl x509 -req -in solvia.csr -CA ca-aula.crt -CAkey ca-aula.key -CAcreateserial \
  -out solvia.crt -days 365 -sha256 -extfile solvia.ext
```

| Opción | Qué hace |
|---|---|
| `req` | Trabaja con peticiones de certificado (CSR) |
| `-x509` | En vez de una CSR, genera directamente un **certificado autofirmado** |
| `-new` | Nueva petición |
| `-newkey rsa:2048` | Genera una clave RSA nueva de ese tamaño (4096 para una CA) |
| `-nodes` | La clave privada **no se cifra con contraseña**. Necesario para que Apache arranque solo (a cambio, la clave se protege con permisos `600`) |
| `-keyout <f>` / `-out <f>` | Dónde se escribe la clave / el resultado |
| `-days <n>` | Validez en días |
| `-subj "/CN=nombre"` | Datos del titular sin preguntas interactivas |
| `-addext "subjectAltName=DNS:a,DNS:b"` | Añade los SAN (lista de nombres válidos) |
| `x509 -req` | Firma una CSR (modo «CA») |
| `-CA <f> -CAkey <f>` | Certificado y clave de la CA que firma |
| `-CAcreateserial` | Crea el fichero de número de serie de la CA |
| `-extfile <f>` | Fichero con las extensiones que se añaden al firmar |
| `-sha256` | Algoritmo de la firma |

### Inspeccionar y verificar

```bash
openssl x509 -in solvia.crt -noout -subject -issuer -dates -ext subjectAltName
openssl x509 -in solvia.crt -noout -text
openssl req -in solvia.csr -noout -verify -subject
openssl verify -CAfile ca-aula.crt solvia.crt
openssl s_client -connect 127.0.0.1:443 -servername solvia.test -CAfile ca-aula.crt </dev/null
```

| Comando / opción | Qué hace |
|---|---|
| `x509 -in <crt> -noout ...` | Lee un certificado. `-noout` evita volcar el certificado codificado |
| `-subject` / `-issuer` | Titular / quién lo firmó. En un autofirmado coinciden |
| `-dates` | Desde cuándo y hasta cuándo vale (`notBefore` / `notAfter`) |
| `-enddate` | Solo la fecha de caducidad |
| `-ext subjectAltName` | Muestra los SAN |
| `-text` | Todo el contenido legible |
| `openssl verify -CAfile <ca> <crt>` | ¿Está este certificado firmado por esa CA? Responde `OK` |
| `s_client -connect host:puerto` | Se conecta como lo haría un navegador y enseña el certificado que presenta el servidor y el resultado de la verificación |
| `-servername <nombre>` | Envía el **SNI**: el nombre del sitio al que quieres hablar (imprescindible con varios sitios virtuales en el 443) |
| `-CAfile <ca>` | Verifica contra esa CA |
| `</dev/null` | Cierra la entrada para que `s_client` termine en vez de quedarse esperando |
| `-tls1_2` / `-tls1_3` | Fuerza una versión de TLS |

### ¿Casan la clave y el certificado?

**`apache2ctl configtest` NO lo detecta**: da `Syntax OK` aunque la clave sea de otro certificado. El fallo aparece al **arrancar o recargar**, y se lee en el registro de errores:

```text
AH02565: Certificate and private key solvia.test:443:0 from /etc/apache2/ssl/solvia.crt and /etc/apache2/ssl/otra.key do not match
AH00016: Configuration Failed
```

Para comprobarlo antes, compara el *módulo* de la clave y del certificado:

```bash
sudo openssl rsa -noout -modulus -in /etc/apache2/ssl/solvia.key | md5sum
openssl x509 -noout -modulus -in /etc/apache2/ssl/solvia.crt | md5sum
```

Las dos sumas tienen que ser **idénticas**.

Otro mensaje que puede aparecer con un certificado autofirmado hecho con `openssl req -x509`: `AH01906: ... server certificate is a CA certificate (BasicConstraints: CA == TRUE !?)`. Es un **aviso**: Apache arranca igualmente. Se debe a que ese certificado se declara a sí mismo como CA; el certificado de servidor firmado por la CA del aula no tiene ese aviso.

### Instalar los ficheros y protegerlos

```bash
sudo mkdir -p /etc/apache2/ssl
sudo cp solvia.crt solvia.key /etc/apache2/ssl/
sudo chown root:root /etc/apache2/ssl/solvia.crt /etc/apache2/ssl/solvia.key
sudo chmod 750 /etc/apache2/ssl
sudo chmod 644 /etc/apache2/ssl/solvia.crt
sudo chmod 600 /etc/apache2/ssl/solvia.key
```

La clave privada, `600` y de `root`. Si alguien la copia, puede hacerse pasar por tu servidor.

### Que el sistema se fíe de la CA del aula

```bash
sudo cp ca-aula.crt /usr/local/share/ca-certificates/ca-aula.crt
sudo update-ca-certificates
```

**Let's Encrypt** (`sudo certbot --apache`) emite certificados gratuitos de confianza pública, pero valida que controlas el nombre mediante un reto **HTTP-01** que necesita ser alcanzable desde Internet: en una VM detrás de NAT no funciona. Por eso en el módulo solo se explica.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 17. Autenticación Basic con `htpasswd` (UD2) :id=b17

```bash
sudo htpasswd -B -c /etc/apache2/.htpasswd-informes maria     # crea el fichero y el primer usuario
sudo htpasswd -B /etc/apache2/.htpasswd-informes luis        # añade otro usuario
sudo chown root:www-data /etc/apache2/.htpasswd-informes
sudo chmod 640 /etc/apache2/.htpasswd-informes
```

| Opción | Qué hace | Trampa |
|---|---|---|
| `-c` | **Crea** el fichero | **Solo en el primer usuario**: con un fichero que ya existe, lo vacía y pierdes a los demás |
| `-B` | Guarda la contraseña con **bcrypt** (lento y resistente a fuerza bruta) | Sin `-B` usa MD5 (`$apr1$`), mucho más débil |
| `-D <usuario>` | Borra un usuario | |
| `-v <usuario>` | Verifica una contraseña (la pregunta) | |
| `-n` | No toca el fichero: imprime el resultado por pantalla | Para generar una línea y pegarla |
| `-b` | La contraseña va **en la propia línea de comandos** | **No lo uses**: queda en el historial y en la lista de procesos |

El fichero se guarda **fuera del `DocumentRoot`** (por eso en `/etc/apache2/`). Su nombre empieza por `.ht`, y `apache2.conf` ya impide servir cualquier fichero `.ht*`.

**Basic no cifra**: `usuario:contraseña` viaja en Base64, que se descodifica en un instante. Una zona protegida con Basic **solo es segura sobre HTTPS**.

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://informes.solvia.test/                         # 401 sin credenciales
curl -s -o /dev/null -w "%{http_code}\n" -u 'maria:miClave' http://informes.solvia.test/      # 200 con credenciales
```

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 18. Logs: leer, filtrar, rotar y analizar (UD2) :id=b18

### 18.1 Dónde están y cuál mirar

| Fichero | Contiene |
|---|---|
| `/var/log/apache2/error.log` | Errores y avisos generales de Apache (arranque, avisos) |
| `/var/log/apache2/<sitio>_error.log` | Errores de **ese sitio**: aquí suele estar la respuesta a un 500 |
| `/var/log/apache2/<sitio>_access.log` | Una línea por petición de ese sitio |

Si el log general solo trae avisos de arranque, la respuesta está en el log **del sitio**. `ls -lt /var/log/apache2/` pone arriba el que se ha escrito más recientemente.

### 18.2 Seguir y mirar el final

```bash
sudo tail -n 20 /var/log/apache2/solvia_error.log
sudo tail -f /var/log/apache2/solvia_access.log
sudo tail -n0 -f /var/log/apache2/solvia_access.log | grep --line-buffered '" 404 '
```

### 18.3 Las columnas del formato `combined`

```text
127.0.0.66 - - [09/Nov/2026:09:21:39 +0000] "GET /.env HTTP/1.1" 404 2667 "-" "zgrab/0.x"
```

| `awk` | Campo | Ejemplo en la línea |
|---|---|---|
| `$1` | IP del cliente | `127.0.0.66` |
| `$3` | Usuario (si hubo autenticación) | `-` |
| `$4` | Fecha y hora | `[09/Nov/2026:09:21:39` |
| `$6` | Método (con comilla) | `"GET` |
| `$7` | Ruta pedida | `/.env` |
| `$9` | **Código de estado** | `404` |
| `$10` | Bytes enviados | `2667` |
| `-F'"' ... $6` | *User-Agent* (separando por comillas) | `zgrab/0.x` |

### 18.4 Analizar con la línea de comandos

```bash
awk '{print $9}' solvia_access.log | sort | uniq -c | sort -rn                  # peticiones por código de estado
awk '$9 == 404 {print $1}' solvia_access.log | sort | uniq -c | sort -rn        # IP que más 404 genera
awk '$9 == 401 {print $1, $3}' informes_access.log | sort | uniq -c            # intentos fallidos de acceso
awk -F'"' '{print $6}' solvia_access.log | sort | uniq -c | sort -rn           # User-Agents
awk '$1 == "127.0.0.66" && $9 == 404 {print $7}' solvia_access.log             # qué rutas pidió esa IP
```

| Pieza de `awk` | Qué hace |
|---|---|
| `awk '<condición> {<acción>}' fichero` | Para cada línea que cumple la condición, hace la acción |
| `$1`, `$9`, `$NF` | Primera columna, novena, la **última** |
| `-F'"'` | Cambia el separador de columnas (aquí, la comilla) |
| `$9 == 404` | Condición: la columna 9 vale 404 |
| `&&` | Y lógico entre condiciones |
| `{print $1, $3}` | Imprime esas columnas |
| `END {print n}` | Acción que se ejecuta al terminar de leer |

El patrón `... | sort | uniq -c | sort -rn` es **contar y ordenar de mayor a menor**: el `sort` primero es obligatorio porque `uniq` solo junta líneas consecutivas.

### 18.5 Rotación con `logrotate`

Los logs crecen sin parar. `logrotate` los renombra, comprime y borra los antiguos según `/etc/logrotate.d/apache2` (por defecto en Ubuntu: diario, 14 copias, `delaycompress`).

```bash
sudo logrotate -d /etc/logrotate.d/apache2     # simulación: dice qué haría sin hacerlo
sudo logrotate -f /etc/logrotate.d/apache2     # rota ahora mismo, aunque no toque
ls -l /var/log/apache2/                        # aparecen solvia_access.log.1, .2.gz ...
```

| Opción | Qué hace |
|---|---|
| `-d` | *Debug*: simula y no cambia nada |
| `-f` | Fuerza la rotación |
| `-v` | Detalla lo que hace |

Los logs rotados con `.gz` se leen con `zcat` o `zgrep` ([apartado 12](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b12)).

### 18.6 GoAccess: informes y panel en tiempo real

```bash
goaccess solvia_access.log --log-format=COMBINED                                       # panel en la terminal
goaccess solvia_access.log --log-format=COMBINED --no-progress -o informe.html         # informe HTML
goaccess solvia_access.log --log-format=COMBINED --no-progress -o informe.json         # lo mismo en JSON
goaccess solvia_access.log --log-format=COMBINED --real-time-html -o informe.html --port=7890
cat solvia_access.log informes_access.log | goaccess - --log-format=COMBINED -o todo.html
```

| Opción | Qué hace |
|---|---|
| `--log-format=COMBINED` | Indica el formato del log (el nuestro es `combined`) |
| `-o <fichero>` | Fichero de salida. La extensión elige el formato: `.html`, `.json`, `.csv` |
| `--no-progress` | Sin barra de progreso (para scripts) |
| `--real-time-html` | El HTML se actualiza en vivo, mediante un WebSocket |
| `--port=<n>` | Puerto del WebSocket (conviene indicarlo) |
| `-` | Leer del flujo de entrada en vez de un fichero: **consolida varios logs** con `cat a b \| goaccess -` |
| `--version` | Versión instalada |

Los registros contienen **datos personales** (IP, usuario): trátalos como tal (no los subas ni los compartas sin anonimizar).

### 18.7 Consolidación entre servidores

Reunir los logs de **varios servidores** en un punto (rsyslog, pila Elastic, Graylog, Loki) **se explica, no se practica** en el módulo.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 19. MariaDB y PHP (UD1) :id=b19

### 19.1 Entrar y ejecutar consultas

```bash
sudo mysql                                      # entrar como administrador (autenticación por socket)
sudo mysql -e "SHOW DATABASES;"                 # una consulta y salir
sudo mysql -e "USE solvia; SHOW TABLES;"
mysql -u solvia_app -p solvia                   # entrar como el usuario de la aplicación
mysql -u solvia_app -p solvia -e "SELECT COUNT(*) FROM tickets;"
```

| Opción | Qué hace |
|---|---|
| `-u <usuario>` | Con qué usuario de MariaDB te conectas |
| `-p` | Pide la contraseña **sin que se vea**. No la pegues pegada a la `-p` (`-pClave`): queda en el historial |
| `<base>` al final | Base de datos con la que trabajas |
| `-e "<SQL>"` | Ejecuta esa sentencia y sale. No hay que acordarse de `EXIT` |
| `-h <host>` | A qué servidor te conectas (por defecto, el local) |

**Por qué `sudo mysql` y no `mysql -u root -p`**: en Ubuntu el `root` de MariaDB se identifica **por socket**: confía en que ya eres `root` del sistema, así que no hay contraseña. Por eso `mysql -u root -p` falla recién instalado; no está roto, está así a propósito.

### 19.2 SQL que se usa en el módulo

Dentro del cliente `mysql>` (cada sentencia **acaba en `;`**):

| Sentencia | Qué hace |
|---|---|
| `SHOW DATABASES;` | Lista las bases de datos |
| `USE solvia;` | Selecciona la base |
| `SHOW TABLES;` | Lista las tablas de la base seleccionada |
| `DESCRIBE tickets;` | Columnas y tipos de una tabla |
| `SELECT * FROM tickets LIMIT 5;` | Las 5 primeras filas |
| `SELECT COUNT(*) FROM tickets;` | Cuántas filas hay |
| `SOURCE /var/www/solvia/db/schema.sql;` | Ejecuta un fichero SQL (crea la base, las tablas y los datos de ejemplo) |
| `CREATE DATABASE solvia;` | Crea una base |
| `CREATE USER 'solvia_app'@'localhost' IDENTIFIED BY '<contraseña>';` | Crea un usuario |
| `GRANT ALL PRIVILEGES ON solvia.* TO 'solvia_app'@'localhost';` | Le da permisos **solo sobre la base `solvia`** |
| `SHOW GRANTS FOR 'solvia_app'@'localhost';` | Qué puede hacer ese usuario |
| `ALTER USER 'solvia_app'@'localhost' IDENTIFIED BY '<nueva>';` | Cambia su contraseña |
| `DROP USER 'solvia_app'@'localhost';` | Borra el usuario |
| `FLUSH PRIVILEGES;` | Recarga la tabla de permisos (no siempre hace falta; no estorba) |
| `EXIT;` | Salir |

- `'solvia_app'@'localhost'` es **el nombre completo**: usuario + desde dónde se conecta. `solvia_app@localhost` y `solvia_app@%` son usuarios distintos.
- **Las comillas son obligatorias** alrededor del usuario, el host y la contraseña. Sin ellas: `ERROR 1064`.
- **`<contraseña>` es un hueco**: la eliges tú, sin `ñ`, tildes ni espacios. Si copias el marcador literalmente, la contraseña la sabrá toda la clase.
- **Mínimo privilegio**: la aplicación usa `solvia_app`, que solo toca `solvia.*`, no `root`. Si el código tiene un fallo, el daño queda acotado.

### 19.3 Copias de seguridad

```bash
sudo mysqldump solvia > solvia-copia.sql             # exporta la base entera a un fichero SQL
sudo mysql solvia < solvia-copia.sql                 # la restaura
```

### 19.4 Errores de MariaDB que ya han salido

| Error | Significa | Qué mirar |
|---|---|---|
| `ERROR 1064 (42000): You have an error in your SQL syntax` | **Error de sintaxis de SQL**, no de permisos | Comillas, `;` y la errata |
| `ERROR 1045 (28000): Access denied for user ...` | Usuario o contraseña incorrectos | Probar a mano: `mysql -u solvia_app -p` |
| `ERROR 1049 (42000): Unknown database` | Esa base no existe | `SHOW DATABASES;` |
| `ERROR 2002 (HY000): Can't connect to local MySQL server through socket` | El servicio no está corriendo | `systemctl status mariadb` |
| `Failed to open file '...', error: 2` (al hacer `SOURCE`) | El `2` significa «no existe el fichero»: es la **ruta** | `ls` la ruta; ¿carpeta `solvia` o `solvia-ra1`? |
| En la web: `500` con `PDOException: could not find driver` | Falta el **conector** `php-mysql` aunque `mariadb-server` esté instalado | `sudo apt install php-mysql` y `sudo systemctl restart apache2` |
| En la web: `500` con `Access denied for user 'solvia_app'` | La contraseña de `config.php` no coincide con la del usuario | Aislar: probar la credencial a mano antes de tocar nada |

### 19.5 PHP desde la terminal

| Comando | Qué hace |
|---|---|
| `php -v` | Versión de PHP |
| `php -m` | Módulos cargados (`php -m \| grep pdo_mysql`: ¿tiene el conector?) |
| `php -l <fichero.php>` | **Comprueba la sintaxis** de un fichero sin ejecutarlo |
| `php --ini` | Qué ficheros `php.ini` carga |
| `php -i \| grep -E "display_errors\|expose_php"` | Valor actual de una opción |
| `php -r 'echo phpversion();'` | Ejecuta una línea de PHP |
| `php -S 127.0.0.1:8000 -t public` | Servidor de desarrollo integrado (solo para pruebas) |

El `php.ini` que usa **Apache** es distinto del de la terminal (CLI): `/etc/php/8.3/apache2/php.ini` frente a `/etc/php/8.3/cli/php.ini`. Cambiar uno no cambia el otro. En producción: `display_errors = Off` y `expose_php = Off`. **Un `phpinfo()` de prueba se borra al terminar**: publica la configuración entera.

### 19.6 Comprobar versiones (útil en la documentación)

```bash
apache2 -v          # o apache2ctl -v
php -v
mysql --version     # o mariadb --version
openssl version
goaccess --version
lsb_release -a
```

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 20. Tomcat y proxy inverso (UD3) · **[por confirmar]** :id=b20

> Esta unidad aún no se ha dado. Lo que sigue son los comandos y rutas **estándar del paquete de Ubuntu 24.04** (`tomcat10`, comprobado que existe en el repositorio y las rutas que instala). Confirma la versión que se use en clase y que las rutas coinciden con `apt-cache policy tomcat10` y `dpkg -L tomcat10`.

**Qué es**: un servidor de aplicaciones Java (servlets y JSP). Escucha por defecto en el **8080** (HTTP). Apache se pone delante como **proxy inverso**: recibe en el 80/443 y reenvía a Tomcat.

### 20.1 Instalar y gestionar

```bash
sudo apt install tomcat10                     # el servidor (ya trae Java como dependencia)
sudo apt install tomcat10-admin               # (opcional) las aplicaciones Manager y Host Manager
sudo systemctl status tomcat10 --no-pager
sudo systemctl restart tomcat10
sudo journalctl -u tomcat10 -n 50 --no-pager
ss -tlnp | grep -E ":8080|:8005|:8009"
curl -I http://127.0.0.1:8080/
java -version                                 # qué Java se está usando
```

### 20.2 Dónde está cada cosa (paquete de Ubuntu)

| Ruta | Qué contiene |
|---|---|
| `/etc/tomcat10/` | **Configuración**: `server.xml`, `web.xml`, `context.xml`, `tomcat-users.xml`, `catalina.properties`, `logging.properties` |
| `/var/lib/tomcat10/webapps/` | **Aplicaciones desplegadas** (carpetas y `.war`) |
| `/var/log/tomcat10/` | Registros (`/var/lib/tomcat10/logs` es un enlace a esta carpeta) |
| `/etc/default/tomcat10` | Opciones de arranque (memoria de Java, `JAVA_OPTS`...) |
| `/usr/share/tomcat10/bin/` | Scripts: `version.sh`, `configtest.sh`, `digest.sh`, `catalina.sh` |
| `/usr/share/tomcat10/` (`CATALINA_HOME`) | Programa (los `.jar`) |
| `/var/lib/tomcat10/` (`CATALINA_BASE`) | Datos y configuración **de esta instancia** |

| Fichero | Para qué sirve |
|---|---|
| `server.xml` | Puertos y **conectores** (`8080` HTTP, `8009` AJP, `8005` parada), *hosts* y su carpeta `appBase` |
| `web.xml` | Configuración **por defecto** de todas las aplicaciones web (servlets, sesiones, errores) |
| `context.xml` | Configuración de contexto (recursos, p. ej. conexión a base de datos) |
| `tomcat-users.xml` | **Usuarios y roles** (para acceder al Manager). Las contraseñas se guardan en claro salvo que se configure un resumen (`digest.sh`) |
| `WEB-INF/web.xml` (dentro de cada aplicación) | Descriptor de despliegue de **esa** aplicación |

Tomcat corre con su propio usuario, **`tomcat`**, no con `www-data`. Los ficheros de `/etc/tomcat10/` pertenecen a `root:tomcat` con modo `640`: Tomcat puede leerlos pero **no modificarlos** (así, una aplicación comprometida no puede reescribir la configuración).

| Comando | Qué hace |
|---|---|
| `/usr/share/tomcat10/bin/version.sh` | Versión de Tomcat y de Java |
| `/usr/share/tomcat10/bin/configtest.sh` | Comprueba la configuración (equivale a `apache2ctl configtest`) |
| `/usr/share/tomcat10/bin/digest.sh -a sha-256 <contraseña>` | Genera un resumen para `tomcat-users.xml` |

### 20.3 Desplegar una aplicación (WAR)

Un **WAR** es un `.zip` con una aplicación web Java (carpeta `WEB-INF/`, clases, JSP...). Tomcat despliega solo el que aparezca en `webapps/`.

```bash
unzip -l informes.war                                           # ver qué contiene
sudo cp informes.war /var/lib/tomcat10/webapps/                 # desplegar
sudo chown tomcat:tomcat /var/lib/tomcat10/webapps/informes.war
ls /var/lib/tomcat10/webapps/                                   # aparece la carpeta informes/
curl -I http://127.0.0.1:8080/informes/
```

| Comando | Qué hace |
|---|---|
| `jar -cvf <app>.war -C <carpeta> .` | **Construye** un WAR a partir de una carpeta (`c` crear, `v` detalle, `f` fichero) |
| `jar -tf <app>.war` | Lista el contenido de un WAR |
| `javac -version` | ¿Hay compilador de Java (JDK)? (`sudo apt install default-jdk`) |

El nombre del WAR **decide la URL**: `informes.war` se publica en `/informes/`. `ROOT.war` se publica en `/`.

### 20.4 Apache delante de Tomcat (proxy inverso)

```bash
sudo a2enmod proxy proxy_http                 # reenvío por HTTP
sudo systemctl restart apache2
```

```apache
<VirtualHost *:80>
        ServerName solvia.test
        ProxyPreserveHost On
        ProxyPass        /informes http://127.0.0.1:8080/informes
        ProxyPassReverse /informes http://127.0.0.1:8080/informes
</VirtualHost>
```

| Directiva | Qué hace |
|---|---|
| `ProxyPass <ruta> <destino>` | Las peticiones a `<ruta>` se reenvían al `<destino>` |
| `ProxyPassReverse <ruta> <destino>` | Reescribe las redirecciones y cabeceras de vuelta para que apunten a Apache y no a Tomcat |
| `ProxyPreserveHost On` | Mantiene la cabecera `Host` original (para que Tomcat vea el nombre que pidió el cliente) |

**AJP** (`ajp://127.0.0.1:8009/...` con `a2enmod proxy_ajp`) es el protocolo binario alternativo a HTTP entre Apache y Tomcat. En las versiones actuales de Tomcat el conector AJP viene **desactivado en `server.xml`** y, si se activa, exige un `secret` y debe escuchar solo en `127.0.0.1`. Comprueba qué trae tu versión con `grep -n -i ajp /etc/tomcat10/server.xml`. **Nunca expongas el 8009 fuera de la máquina.**

### 20.5 Errores típicos

| Síntoma | Causa probable |
|---|---|
| `502 Bad Gateway` en Apache | Tomcat parado o en otro puerto (`systemctl status tomcat10`, `ss -tlnp`) |
| `404` con página de Tomcat («HTTP Status 404») | Tomcat responde, pero esa ruta/aplicación no existe: revisa `webapps/` y el nombre del WAR |
| La aplicación no se despliega | Propietario o permisos del WAR; mira `/var/log/tomcat10/` |
| El Manager da 403 | Por defecto solo admite conexiones locales y exige un usuario con rol en `tomcat-users.xml` |
| Tomcat no arranca | `journalctl -u tomcat10 -n 50`: puerto ocupado, XML mal formado o Java ausente |

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 21. FTP, FTPS y SFTP (UD4) · **[por confirmar]** :id=b21

> Unidad aún no dada. La programación fija **ProFTPD** como servidor. Los paquetes y rutas se han comprobado en el repositorio de Ubuntu 24.04; confirma los nombres de directivas contra la versión que se instale (`man proftpd.conf` y `/usr/share/doc/proftpd-doc` si se instala `proftpd-doc`).

### 21.1 Conceptos que se preguntan siempre

| Concepto | Qué es |
|---|---|
| **FTP** | Protocolo de transferencia de ficheros. Usa **dos conexiones**: la de **control** (puerto 21: comandos) y la de **datos** (los ficheros). **No cifra nada**, ni siquiera la contraseña |
| **Modo activo** | El **servidor** abre la conexión de datos hacia el **cliente** (desde su puerto 20). Falla si el cliente está detrás de un cortafuegos o NAT |
| **Modo pasivo** | El **cliente** abre la conexión de datos hacia un puerto alto del servidor. Es el habitual hoy; exige abrir en el cortafuegos el **rango de puertos pasivos** |
| **FTPS** | FTP + TLS. *Explícito*: se conecta al 21 y se pide cifrar (`AUTH TLS`). *Implícito*: cifrado desde el principio, en el 990 (en desuso) |
| **SFTP** | **Es otro protocolo**: transferencia de ficheros **sobre SSH** (puerto 22). No tiene nada que ver con FTP salvo el nombre |

### 21.2 Instalar y gestionar ProFTPD

```bash
sudo apt install proftpd-core                    # en Ubuntu 24.04 «proftpd-basic» es un nombre virtual que lo instala
sudo apt install proftpd-mod-crypto              # (para FTPS y SFTP) módulos mod_tls y mod_sftp
sudo proftpd -t                                  # comprueba la sintaxis de la configuración
sudo systemctl restart proftpd
sudo systemctl status proftpd --no-pager
ss -tlnp | grep :21
```

| Ruta / comando | Qué es |
|---|---|
| `/etc/proftpd/proftpd.conf` | Configuración principal |
| `/etc/proftpd/modules.conf` | Qué módulos se cargan |
| `/etc/proftpd/tls.conf` | Plantilla de configuración TLS (se incluye desde `proftpd.conf`) |
| `/etc/proftpd/conf.d/` | Ficheros de configuración adicionales |
| `/etc/ftpusers` | Usuarios a los que **se prohíbe** el acceso FTP (root, etc.) |
| `proftpd -t` | *Test*: comprueba la sintaxis (el equivalente de `apache2ctl configtest`) |
| `ftpwho` | Quién está conectado ahora |
| `ftpcount` | Cuántos usuarios hay conectados |
| `ftptop` | Panel en tiempo real de sesiones |
| `ftpasswd` | Crea usuarios virtuales (con su propio fichero de contraseñas, aparte de los del sistema) |

### 21.3 Directivas habituales de `proftpd.conf`

| Directiva | Qué hace |
|---|---|
| `ServerName "<texto>"` | Nombre que se muestra |
| `Port 21` | Puerto de control |
| `DefaultRoot ~` | **Enjaula** a cada usuario en su carpeta personal: no puede subir a `/` |
| `RequireValidShell off` | Permite usuarios sin shell válida (típico del usuario de solo FTP) |
| `PassivePorts 49152 49200` | Rango de puertos de datos para el modo pasivo (**ábrelo en el cortafuegos**) |
| `MasqueradeAddress <IP>` | IP que se anuncia en modo pasivo si el servidor está detrás de NAT |
| `AuthUserFile <ruta>` | Fichero de usuarios virtuales |
| `TLSEngine on` | Activa TLS (FTPS) |
| `TLSRSACertificateFile` / `TLSRSACertificateKeyFile` | Certificado y clave (los mismos de `openssl`, [apartado 16](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b16)) |
| `TLSRequired on` | Obliga a cifrar las conexiones |
| `<Limit ...> ... </Limit>` | Reglas por operación (leer, escribir, borrar) |
| `Umask 022` | Permisos por defecto de lo que se sube |

Tras cambiar algo: `sudo proftpd -t && sudo systemctl restart proftpd`.

**Crear un usuario solo de FTP** (sin acceso por consola):

```bash
sudo adduser --shell /usr/sbin/nologin --home /srv/ftp/adjuntos tecnico1
```

(La shell `/usr/sbin/nologin` impide el login por SSH; con `RequireValidShell off` ProFTPD le deja entrar por FTP. `/usr/sbin/nologin` debe aparecer en `/etc/shells` si no se activa esa directiva.)

### 21.4 Clientes de línea de comandos

```bash
ftp 127.0.0.1                          # cliente FTP clásico
lftp -u tecnico1 127.0.0.1             # lftp: mejor cliente de terminal (pide la contraseña)
sftp tecnico1@127.0.0.1                # SFTP (sobre SSH)
sftp -P 2222 tecnico1@127.0.0.1        # SFTP en otro puerto (la P es MAYÚSCULA)
scp -P 2222 fichero.txt usuario@host:/ruta/    # copiar por SSH (también P mayúscula)
```

**Comandos dentro de la sesión** (valen en `ftp`, `lftp` y, casi todos, en `sftp`):

| Comando | Qué hace |
|---|---|
| `ls` / `dir` | Lista la carpeta **remota** |
| `cd <ruta>` | Cambia de carpeta **remota** |
| `pwd` | Carpeta remota actual |
| `lcd <ruta>` | Cambia de carpeta **local** |
| `lpwd` | Carpeta local actual (en `sftp`; en `lftp` es `!pwd`) |
| `!<comando>` | Ejecuta un comando **en tu máquina** (p. ej. `!ls`) |
| `get <fichero>` | **Descarga** (remoto → local) |
| `put <fichero>` | **Sube** (local → remoto) |
| `mget <patrón>` / `mput <patrón>` | Descarga / sube varios (`mput *.txt`) |
| `mkdir <carpeta>` / `rm <fichero>` | Crea / borra en remoto |
| `passive` | (`ftp`) alterna modo pasivo / activo |
| `binary` | (`ftp`) transferencia binaria (para todo salvo texto plano) |
| `bye` / `quit` / `exit` | Salir |

Configuración útil de **`lftp`** (se escribe dentro de la sesión, o con `lftp -e "<comando>"`):

| Comando de `lftp` | Qué hace |
|---|---|
| `set ftp:passive-mode off` | Prueba el modo **activo** (por defecto es pasivo) |
| `set ftp:ssl-force true` | Obliga a usar TLS (FTPS) |
| `set ssl:verify-certificate no` | No verifica el certificado (**solo para pruebas** con uno autofirmado) |
| `debug 5` | Muestra el diálogo de comandos y respuestas FTP |

**Clientes gráficos**: FileZilla es el habitual **[por confirmar]**. Se configura con servidor, puerto, protocolo (FTP, FTP con TLS explícito o SFTP) y usuario.

### 21.5 SSH (para SFTP y administración remota)

| Comando | Qué hace |
|---|---|
| `ssh <usuario>@<host>` | Abre una sesión remota (`-p <puerto>` si no es el 22; **p minúscula**) |
| `ssh -i <clave> <usuario>@<host>` | Con una clave privada concreta |
| `ssh-keygen -t ed25519 -C "<comentario>"` | Genera un par de claves (`~/.ssh/id_ed25519` privada y `.pub` pública) |
| `ssh-copy-id <usuario>@<host>` | Instala tu clave pública en el servidor para entrar sin contraseña |

La clave **privada** nunca se comparte ni se sube a un repositorio.

### 21.6 Códigos de respuesta FTP

| Código | Significa |
|---|---|
| `220` | Servidor listo (el saludo) |
| `230` | Sesión iniciada |
| `331` | Usuario correcto, falta la contraseña |
| `530` | **Login incorrecto** (usuario/clave, o usuario en `/etc/ftpusers`, o shell no válida) |
| `227` | Entrando en modo pasivo (el servidor da IP y puerto) |
| `150` | Abriendo la conexión de datos |
| `425` | **No se puede abrir la conexión de datos**: casi siempre cortafuegos/NAT o puertos pasivos sin abrir |
| `226` | Transferencia completada |
| `550` | Fichero o carpeta no existe, o **permiso denegado** (revisa propietario y permisos de la carpeta) |
| `421` | Servicio no disponible (servidor parado o sobrecargado) |

**Síntoma típico**: «me conecto, hago login, pero `ls` se queda colgado» = falla la **conexión de datos** (425): modo pasivo, rango `PassivePorts` y cortafuegos (`sudo ufw allow 21/tcp` y `sudo ufw allow 49152:49200/tcp`).

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 22. DNS (UD6 · RA5) · **[por confirmar]** :id=b22

> El DNS se trabaja en la parte teórica de RA5 durante la FEOE. La programación no fija todavía qué servidor se instalará, así que aquí están (a) las **herramientas de consulta**, que valen siempre, y (b) un esquema de **BIND9** como opción habitual.

### 22.1 Cómo se resuelve un nombre en Linux

Orden: `/etc/hosts` → servidor DNS configurado (`/etc/resolv.conf`, gestionado por `systemd-resolved`). El orden lo marca la línea `hosts:` de `/etc/nsswitch.conf`. **`dig` y `nslookup` consultan al servidor DNS directamente e ignoran `/etc/hosts`**; para ver lo que ve una aplicación, usa `getent hosts <nombre>`.

### 22.2 Herramientas de consulta

```bash
dig solvia.test                      # consulta completa
dig +short solvia.test               # solo la respuesta
dig MX gmail.com +short              # un tipo de registro concreto
dig @8.8.8.8 ejemplo.org A           # preguntando a un servidor concreto
dig -x 8.8.8.8 +short                # consulta inversa (IP → nombre)
dig +trace ejemplo.org               # recorre la jerarquía desde la raíz
getent hosts solvia.test             # lo que resuelve el sistema (incluye /etc/hosts)
```

| Opción de `dig` | Qué hace |
|---|---|
| `@<servidor>` | Pregunta a ese servidor DNS |
| `<TIPO>` (`A`, `AAAA`, `MX`...) o `-t <tipo>` | Qué tipo de registro pides |
| `+short` | Solo el dato de la respuesta |
| `+noall +answer` | Solo la sección de respuesta, con formato |
| `+trace` | Sigue la delegación desde los servidores raíz |
| `-x <IP>` | Consulta inversa |
| `+norecurse` | No pide al servidor que resuelva por ti: solo lo que sabe |

Otras herramientas: `nslookup <nombre> [servidor]`, `host <nombre>`, `resolvectl query <nombre>`, `resolvectl status`.

**Cómo leer una respuesta de `dig`**: `status: NOERROR` = bien; `NXDOMAIN` = ese nombre **no existe**; `SERVFAIL` = el servidor falló al resolver; `ANSWER: 0` con `NOERROR` = el nombre existe pero no tiene ese tipo de registro. `flags: aa` = respuesta **autoritativa** (viene del servidor que gestiona la zona); sin `aa`, viene de una caché.

### 22.3 Tipos de registro

| Tipo | Qué guarda | Ejemplo |
|---|---|---|
| `A` | Nombre → IPv4 | `solvia IN A 192.168.1.10` |
| `AAAA` | Nombre → IPv6 | |
| `CNAME` | Alias de otro nombre | `www IN CNAME solvia.test.` |
| `MX` | Servidor de correo (con prioridad) | `@ IN MX 10 mail.solvia.test.` |
| `NS` | Servidores de nombres de la zona | `@ IN NS ns1.solvia.test.` |
| `TXT` | Texto libre (verificaciones, SPF) | |
| `PTR` | IP → nombre (zona inversa) | |
| `SOA` | Datos de la zona: servidor primario, contacto, **serial** y tiempos | |

### 22.4 Servidor BIND9 (esquema)

```bash
sudo apt install bind9 bind9-utils
sudo named-checkconf                                          # revisa named.conf*
sudo named-checkzone solvia.test /etc/bind/db.solvia.test     # revisa un fichero de zona
sudo rndc reload                                              # recarga zonas sin parar el servicio
sudo systemctl restart bind9
dig @127.0.0.1 solvia.test +short
```

| Fichero | Qué contiene |
|---|---|
| `/etc/bind/named.conf.options` | Opciones globales (reenviadores, quién puede consultar) |
| `/etc/bind/named.conf.local` | Aquí **declaras tus zonas** |
| `/etc/bind/db.<zona>` | El fichero de zona con los registros |

Declarar una zona en `named.conf.local`:

```bind
zone "solvia.test" {
        type master;
        file "/etc/bind/db.solvia.test";
};
```

Fichero de zona mínimo (`/etc/bind/db.solvia.test`):

```bind
$TTL    604800
@       IN      SOA     ns1.solvia.test. admin.solvia.test. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.solvia.test.
ns1     IN      A       127.0.0.1
solvia  IN      A       127.0.0.1
informes IN     A       127.0.0.1
```

Tres reglas que se olvidan: (1) los nombres **completos acaban en punto** (`ns1.solvia.test.`); sin punto, BIND les añade el nombre de la zona; (2) al cambiar la zona hay que **subir el `Serial`**; (3) pasa siempre `named-checkzone` antes de recargar.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 23. LDAP (UD5) · **[por confirmar]** :id=b23

> Unidad aún no dada. La programación fija **OpenLDAP** y su despliegue como contenedor en el Compose de UD6. Las herramientas cliente (`ldap-utils`) y la instalación de `slapd` son las estándar; comprueba nombres de contenedor e imagen cuando se decida.

### 23.1 Conceptos

Un directorio LDAP es un **árbol** de entradas. Cada entrada se identifica por su **DN** (*Distinguished Name*), que se lee de derecha a izquierda desde la raíz:

```plaintext
uid=maria,ou=tecnicos,dc=solvia,dc=test
│         │           └─────────────── dominio: solvia.test
│         └─────────────────────────── unidad organizativa «tecnicos»
└───────────────────────────────────── la entrada: usuario «maria»
```

| Abreviatura | Significa |
|---|---|
| `dc` | *Domain Component*: una parte del dominio (`solvia.test` → `dc=solvia,dc=test`) |
| `ou` | *Organizational Unit*: una «carpeta» |
| `uid`, `cn` | Identificador / nombre común de una entrada |
| `objectClass` | Qué **tipo** de entrada es; fija los atributos permitidos (`inetOrgPerson`, `posixAccount`, `organizationalUnit`, `groupOfNames`...) |
| **Bind** | Autenticarse contra el directorio (con un DN y una contraseña) |

La autenticación centralizada funciona así: la aplicación hace un **bind** con el DN del usuario y la contraseña que éste teclea. Si el bind tiene éxito, está autenticado.

### 23.2 Instalar el servidor (si es en la VM y no en contenedor)

```bash
sudo apt install slapd ldap-utils
sudo dpkg-reconfigure slapd                         # configura dominio, organización y contraseña del admin
sudo slapcat                                        # vuelca todo el directorio en formato LDIF
sudo systemctl status slapd --no-pager
ss -tlnp | grep :389
```

### 23.3 Consultar: `ldapsearch`

```bash
ldapsearch -x -H ldap://localhost -b "dc=solvia,dc=test" "(uid=maria)"
ldapsearch -x -LLL -H ldap://localhost -b "dc=solvia,dc=test" "(objectClass=inetOrgPerson)" uid cn mail
ldapsearch -x -D "cn=admin,dc=solvia,dc=test" -W -b "dc=solvia,dc=test" -s one "(objectClass=*)" dn
```

| Opción | Qué hace |
|---|---|
| `-x` | Autenticación **simple** (DN + contraseña). Sin `-x` intenta SASL y suele fallar |
| `-H <URI>` | A qué servidor: `ldap://host`, `ldaps://host` (TLS), `ldapi:///` (socket local) |
| `-b "<base>"` | **DN de partida** de la búsqueda |
| `-s base\|one\|sub` | Alcance: solo la base / un nivel / todo el subárbol (por defecto) |
| `-D "<DN>"` | DN con el que haces el *bind* |
| `-W` | Pide la contraseña por teclado (**no uses `-w <clave>`**: queda en el historial) |
| `-LLL` | Salida limpia, sin comentarios |
| `"(filtro)"` | Qué entradas buscas |
| `uid cn mail` (al final) | Qué atributos mostrar (si no, todos) |

**Filtros**: `(uid=maria)` igualdad · `(uid=m*)` comodín · `(&(objectClass=inetOrgPerson)(uid=m*))` **y** · `(|(uid=maria)(uid=luis))` **o** · `(!(uid=maria))` **no**.

### 23.4 Modificar: `ldapadd`, `ldapmodify`, `ldapdelete`, `ldappasswd`

Los cambios se describen en ficheros **LDIF** (una entrada = un bloque separado por línea en blanco):

```ldif
dn: ou=tecnicos,dc=solvia,dc=test
objectClass: organizationalUnit
ou: tecnicos

dn: uid=maria,ou=tecnicos,dc=solvia,dc=test
objectClass: inetOrgPerson
cn: Maria
sn: Tecnica
uid: maria
mail: maria@solvia.test
```

```bash
ldapadd -x -D "cn=admin,dc=solvia,dc=test" -W -f tecnicos.ldif                           # añadir
ldapmodify -x -D "cn=admin,dc=solvia,dc=test" -W -f cambio.ldif                          # modificar
ldapdelete -x -D "cn=admin,dc=solvia,dc=test" -W "uid=maria,ou=tecnicos,dc=solvia,dc=test"   # borrar
ldappasswd -x -D "cn=admin,dc=solvia,dc=test" -W -S "uid=maria,ou=tecnicos,dc=solvia,dc=test"   # cambiar contraseña
ldapwhoami -x -D "uid=maria,ou=tecnicos,dc=solvia,dc=test" -W                            # ¿funciona el bind de este usuario?
```

`ldapwhoami` con el DN del usuario es **la comprobación directa** de que ese usuario puede autenticarse: es lo que hará la aplicación.

Cambio en LDIF (`cambio.ldif`):

```ldif
dn: uid=maria,ou=tecnicos,dc=solvia,dc=test
changetype: modify
replace: mail
mail: maria.nueva@solvia.test
```

**Configuración del propio servidor** (`cn=config`): se lee y se modifica como `root` por el socket local, sin contraseña:

```bash
sudo ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config dn
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f ajuste.ldif
```

(`-Y EXTERNAL` = «autentícame con mi usuario del sistema».)

### 23.5 Errores de LDAP frecuentes

| Mensaje | Significa |
|---|---|
| `ldap_bind: Invalid credentials (49)` | DN o contraseña incorrectos |
| `No such object (32)` | El DN base (`-b`) o el DN que buscas no existe |
| `Can't contact LDAP server (-1)` | Servidor parado, URI o puerto incorrectos, o cortafuegos |
| `Insufficient access (50)` | Ese usuario no tiene permiso para esa operación |
| `Already exists (68)` | Estás añadiendo una entrada que ya existe |
| `Naming violation` / `Object class violation` | Falta un atributo obligatorio de su `objectClass` |

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 24. Git y GitHub (UD6) · **[por confirmar]** :id=b24

> Unidad aún no dada. La programación fija **Git** y **GitHub**. Los comandos son los estándar y se han contrastado con la versión de Ubuntu 24.04 (Git 2.43).

### 24.1 Primeros pasos

```bash
sudo apt install git
git config --global user.name "Nombre Apellido"
git config --global user.email "tu@correo.es"
git config --global init.defaultBranch main
git config --list                                  # ¿qué tengo configurado?
```

### 24.2 El flujo básico

Tres zonas: tu **carpeta de trabajo** → el **área de preparación** (*staging*, lo que irá en el próximo *commit*) → el **repositorio** (el historial).

```bash
git init                           # empieza un repositorio en la carpeta actual
git status                         # ¿qué ha cambiado? (úsalo constantemente)
git add src/Database.php           # prepara ese fichero
git add .                          # prepara todo lo cambiado (¡revisa antes con git status!)
git commit -m "Añade validación de tickets"
git log --oneline                  # el historial resumido
git push                           # sube los commits al remoto
```

| Comando / opción | Qué hace |
|---|---|
| `git status` (`-s` resumen corto) | Qué está modificado, preparado y sin seguimiento |
| `git add <ruta>` | Pasa cambios al área de preparación. `-p` te deja elegir trozo a trozo |
| `git commit -m "<mensaje>"` | Guarda lo preparado en el historial. El mensaje explica **por qué** |
| `git commit -am "<mensaje>"` | Prepara y guarda los ficheros **ya seguidos** (no incluye los nuevos) |
| `git commit --amend` | Rehace el último commit (**solo si aún no lo has subido**) |
| `git log` | Historial. `--oneline` una línea por commit · `--graph --decorate --all` con ramas · `-n 5` los últimos 5 · `-p` con los cambios · `--stat` resumen de ficheros |
| `git diff` | Cambios **sin preparar**. `--staged` los ya preparados. `git diff <A> <B>` entre commits o ramas |
| `git show <commit>` | Detalle de un commit |
| `git restore <fichero>` | **Descarta** los cambios sin guardar de ese fichero (¡no se recuperan!) |
| `git restore --staged <fichero>` | Saca un fichero del área de preparación sin perder los cambios |
| `git blame <fichero>` | Quién cambió cada línea y en qué commit |

### 24.3 Ramas

| Comando | Qué hace |
|---|---|
| `git branch` | Lista las ramas (`-a` incluye las remotas). `git branch <nombre>` crea una |
| `git switch <rama>` | Cambia de rama. `git switch -c <rama>` la **crea y cambia** |
| `git merge <rama>` | Fusiona esa rama en la actual. `--no-ff` fuerza un commit de fusión. `--abort` cancela si hay conflicto |
| `git branch -d <rama>` | Borra una rama ya fusionada (`-D` la borra aunque no lo esté) |
| `git stash` / `git stash pop` | Aparca cambios sin guardar / los recupera |

**Conflicto de fusión**: Git marca los ficheros con `<<<<<<<`, `=======` y `>>>>>>>`. Edita el fichero, deja la versión correcta, quita las marcas, `git add <fichero>` y `git commit`.

### 24.4 Remotos (GitHub)

```bash
git clone https://github.com/<usuario>/<repo>.git       # copia un repositorio remoto
git remote add origin https://github.com/<usuario>/<repo>.git
git remote -v                                           # ¿a qué remoto apunto?
git push -u origin main                                 # primera subida (-u recuerda el destino)
git pull                                                # trae y fusiona lo nuevo del remoto
git fetch                                               # trae lo nuevo sin fusionar
```

| Comando | Qué hace |
|---|---|
| `git clone <url>` | Copia todo el repositorio y su historial |
| `git remote -v` | Lista los remotos y sus URL |
| `git push [-u] origin <rama>` | Sube los commits de esa rama |
| `git pull` | `fetch` + `merge`. `git pull --rebase` reaplica tus commits encima |
| `git fetch` | Descarga sin tocar tu trabajo; luego decides |

**Autenticación en GitHub**: por HTTPS con **token de acceso personal** (ya no se admite la contraseña de la cuenta), o por **SSH** con clave (`ssh-keygen -t ed25519`, pegar la `.pub` en GitHub y comprobar con `ssh -T git@github.com`).

### 24.5 Deshacer, con cuidado

| Comando | Qué hace | Peligro |
|---|---|---|
| `git revert <commit>` | Crea un commit nuevo que **deshace** otro | Seguro: no reescribe el historial |
| `git reset --soft HEAD~1` | Deshace el último commit y deja los cambios **preparados** | Local |
| `git reset HEAD~1` (`--mixed`) | Lo deshace y deja los cambios **sin preparar** | Local |
| `git reset --hard HEAD~1` | Lo deshace y **borra los cambios** | **Pierdes trabajo**. Nunca sobre commits ya subidos |
| `git clean -n` | Muestra qué ficheros sin seguimiento borraría (`-f` los borra) | `-n` primero, siempre |

### 24.6 `.gitignore` y secretos

Un fichero `.gitignore` en la raíz lista lo que Git **no** debe seguir. En Solvia, la configuración con credenciales no se sube:

```gitignore
config/config.php
.env
*.log
*.key
```

**Un secreto subido a Git, aunque lo borres después, queda en el historial.** Si pasa, hay que cambiar la contraseña, no solo borrar el fichero. Se sube `config.example.php` (con valores de ejemplo) y se ignora `config.php` (con los reales).

`git ls-files` lista lo que Git está siguiendo; `git check-ignore -v <ruta>` dice qué regla ignora un fichero.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 25. Docker y Docker Compose (UD6) · **[por confirmar]** :id=b25

> Unidad aún no dada. La programación fija **Docker y Docker Compose** para el stack completo de Solvia (Apache y PHP, Tomcat, base de datos y LDAP). Los comandos son los estándar de Docker; los de Compose son los de la versión 2 (`docker compose`, con espacio).

### 25.1 Instalar

```bash
sudo apt install docker.io docker-compose-v2        # paquetes del repositorio de Ubuntu (existen en 24.04)
sudo systemctl enable --now docker
docker --version
docker compose version
sudo usermod -aG docker $USER                       # usar docker sin sudo (cierra sesión y vuelve a entrar)
```

El grupo `docker` equivale a tener **root** en la máquina: añade solo a quien deba. (Docker también tiene su propio repositorio oficial, con paquete `docker-ce`; cuál se usa en clase está **por confirmar**.)

### 25.2 Conceptos

| Concepto | Qué es |
|---|---|
| **Imagen** | La «plantilla» de solo lectura (sistema + programa + configuración) |
| **Contenedor** | Una **instancia en marcha** de una imagen. Aislado, efímero |
| **Dockerfile** | Receta para construir una imagen |
| **Volumen** | Almacenamiento que **sobrevive** al contenedor (la base de datos va en un volumen, o se pierde al borrarlo) |
| **Red** | Red virtual entre contenedores; en ella se alcanzan **por el nombre del servicio** |
| **Compose** | Un fichero (`compose.yaml`) que define **varios** contenedores, redes y volúmenes y los levanta juntos |

### 25.3 Comandos de Docker

```bash
docker run -d --name web -p 8080:80 php:8.3-apache
docker ps
docker logs -f web
docker exec -it web bash
docker stop web && docker rm web
```

| Comando | Qué hace |
|---|---|
| `docker run <imagen>` | Crea y arranca un contenedor |
| `-d` | En segundo plano (*detached*) |
| `--name <nombre>` | Nombre del contenedor |
| `-p <host>:<contenedor>` | **Publica un puerto**: `-p 8080:80` = el 8080 de la máquina llega al 80 del contenedor |
| `-e VAR=valor` | Variable de entorno (así se pasan contraseñas y ajustes) |
| `-v <origen>:<destino>` | Monta una carpeta o volumen dentro del contenedor |
| `--rm` | Lo borra al parar |
| `-it` | Interactivo con terminal (para entrar a una shell) |
| `docker ps` | Contenedores en marcha. `-a` incluye los parados |
| `docker images` | Imágenes descargadas |
| `docker pull <imagen>` | Descarga una imagen |
| `docker build -t <nombre> .` | **Construye** una imagen con el `Dockerfile` de la carpeta (`.`) |
| `docker logs <contenedor>` | Lo que ha escrito. `-f` en vivo · `--tail 50` las últimas 50 |
| `docker exec -it <contenedor> <cmd>` | Ejecuta un comando **dentro** de un contenedor en marcha (`bash`, `sh`) |
| `docker stop` / `start` / `restart <contenedor>` | Parar / arrancar / reiniciar |
| `docker rm <contenedor>` (`-f` fuerza) | Borra un contenedor |
| `docker rmi <imagen>` | Borra una imagen |
| `docker inspect <contenedor>` | Toda su configuración (IP, montajes...) |
| `docker network ls` / `docker volume ls` | Redes / volúmenes |
| `docker system prune` | Limpia lo que no se usa. **Lee lo que va a borrar antes de confirmar** |

### 25.4 `Dockerfile`: las instrucciones

```dockerfile
FROM php:8.3-apache
WORKDIR /var/www/html
COPY ./public/ /var/www/html/
EXPOSE 80
```

| Instrucción | Qué hace |
|---|---|
| `FROM <imagen>` | Imagen de partida |
| `RUN <comando>` | Ejecuta un comando **al construir** (instalar paquetes) |
| `COPY <origen> <destino>` | Copia ficheros del proyecto a la imagen |
| `WORKDIR <ruta>` | Carpeta de trabajo |
| `ENV VAR=valor` | Variable de entorno |
| `EXPOSE <puerto>` | Documenta qué puerto usa (**no lo publica**; eso lo hace `-p`) |
| `CMD [...]` | Comando que se ejecuta **al arrancar** el contenedor |
| `USER <usuario>` | Con qué usuario corre (mejor no `root`) |

### 25.5 Docker Compose

```yaml
services:
  web:
    image: php:8.3-apache
    ports:
      - "8080:80"
    volumes:
      - ./solvia/public:/var/www/html:ro
    depends_on:
      - db
  db:
    image: mariadb:10.11
    environment:
      MARIADB_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MARIADB_DATABASE: solvia
    volumes:
      - datos-db:/var/lib/mysql
volumes:
  datos-db:
```

*(Esquema **ilustrativo** de la estructura de un `compose.yaml`; no es la solución del proyecto. Las contraseñas se leen de un fichero `.env` que **no se sube a Git**.)*

| Comando | Qué hace |
|---|---|
| `docker compose up -d` | Crea y arranca todo lo del fichero en segundo plano. `--build` reconstruye antes las imágenes |
| `docker compose down` | Para y borra los contenedores y la red. `-v` **también borra los volúmenes (y los datos)** |
| `docker compose ps` | Estado de los servicios |
| `docker compose logs -f [servicio]` | Registros en vivo |
| `docker compose exec <servicio> <cmd>` | Ejecuta un comando dentro de un servicio (`exec db mariadb -u root -p`) |
| `docker compose config` | **Valida** el fichero y muestra el resultado ya interpolado: úsalo antes de `up` |
| `docker compose restart [servicio]` | Reinicia |
| `docker compose pull` | Descarga las imágenes nuevas |
| `docker compose -f <fichero> up` | Usa otro fichero distinto de `compose.yaml` |

**Reglas que evitan problemas**: (1) en Compose, un servicio alcanza a otro **por su nombre** (`db`), no por `localhost`; (2) los datos que importan van en un **volumen**; (3) `depends_on` espera a que el otro contenedor *arranque*, no a que esté *listo*; (4) los puertos publicados en una VM se tienen que reenviar en VirtualBox para llegar desde Windows.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 26. Integración continua con GitHub Actions (UD6) · **[por confirmar]** :id=b26

Una *workflow* es un fichero YAML en `.github/workflows/` del repositorio. GitHub lo ejecuta en sus máquinas cada vez que ocurre el evento indicado.

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  comprobar:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Comprobar sintaxis PHP
        run: find src public -name "*.php" -print0 | xargs -0 -n1 php -l
```

| Elemento | Qué es |
|---|---|
| `name` | Nombre de la workflow |
| `on` | **Cuándo** se ejecuta (`push`, `pull_request`, `schedule`, `workflow_dispatch` para lanzarla a mano) |
| `jobs` | Trabajos. Cada uno corre en una máquina limpia |
| `runs-on` | Sistema donde corre (`ubuntu-latest`) |
| `steps` | Pasos en orden |
| `uses: <acción>@<versión>` | Reutiliza una acción hecha (`actions/checkout` descarga tu código). **Comprueba cuál es la versión vigente** antes de fijar la etiqueta |
| `run:` | Ejecuta un comando de shell |
| `${{ secrets.NOMBRE }}` | Lee un **secreto** guardado en GitHub (Settings → Secrets). Nunca escribas contraseñas en el YAML |
| `needs: <job>` | Este trabajo espera a otro |

Probar antes de subir lo que hará la CI: ejecuta en local los mismos comandos del `run` (`php -l`, `docker build`, `docker compose config`...). En la pestaña **Actions** del repositorio se ven los registros de cada ejecución: la respuesta a un fallo está ahí, paso por paso.

La YAML es sensible a la **indentación** (espacios, nunca tabuladores) y a los `:` sin espacio detrás.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 27. Generación de documentación (UD6) · **[por confirmar]** :id=b27

> La programación nombra **phpDocumentor o Doxygen** para el código PHP y **Javadoc** para el módulo Java. Cuál se usa finalmente está por decidir; aquí están los patrones de uso.

| Herramienta | Para qué | Uso básico |
|---|---|---|
| **Javadoc** (viene con el JDK) | Documentación HTML del código Java a partir de sus comentarios `/** ... */` | `javadoc -d docs/api -sourcepath src -subpackages <paquete>` |
| **Doxygen** | Documentación de varios lenguajes (incluye PHP) | `doxygen -g` crea un `Doxyfile` · `doxygen Doxyfile` genera la documentación |
| **phpDocumentor** | Documentación específica de PHP a partir de comentarios `/** ... */` | `phpdoc run -d src -t docs/api` (se distribuye como `.phar`: `php phpDocumentor.phar run -d src -t docs/api`) |
| **Pandoc** | Convierte entre formatos de documento | `pandoc entrada.md -o salida.docx` · `pandoc entrada.md -o salida.html` |

| Opción de Javadoc | Qué hace |
|---|---|
| `-d <carpeta>` | Carpeta de salida |
| `-sourcepath <ruta>` | Dónde está el código fuente |
| `-subpackages <paquete>` | Documenta ese paquete y sus subpaquetes |
| `-author` / `-version` | Incluye las etiquetas `@author` / `@version` |
| `-private` | Incluye también los miembros privados |

| Ajuste de `Doxyfile` | Qué hace |
|---|---|
| `PROJECT_NAME` | Título |
| `INPUT` | Carpetas o ficheros de entrada |
| `OUTPUT_DIRECTORY` | Carpeta de salida |
| `RECURSIVE = YES` | Entra en subcarpetas |
| `EXTRACT_ALL = YES` | Documenta también lo que no tiene comentario |
| `GENERATE_LATEX = NO` | No genera la salida LaTeX |

**Formatos y trabajo colaborativo**: la documentación de texto del módulo se escribe en **Markdown** (`.md`), que se versiona en Git junto al código. Los comentarios de código con etiquetas (`@param`, `@return`, `@throws`) son los que leen estas herramientas.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)

---

## 28. Mapa rápido: «tengo este problema, ¿qué miro?» :id=b28

| Síntoma | Primero comprueba | Comando | Más en |
|---|---|---|---|
| **La web no carga** | ¿El servicio vive? | `systemctl status apache2 --no-pager` | [9](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b9) |
| «Connection refused» | ¿Escucha en el puerto? | `ss -tlnp \| grep :80` | [11](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b11) |
| **403 Forbidden** | ¿Puede `www-data` atravesar el camino y leer? | `namei -l <ruta>` · `sudo runuser -u www-data -- cat <fichero>` | [6](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b6) |
| **404 Not Found** | ¿A dónde apunta el `DocumentRoot`? ¿Carpeta anidada? | `sudo apache2ctl -S` · `grep DocumentRoot /etc/apache2/sites-enabled/*.conf` | [14](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b14) |
| **500** | Siempre está en el log, no en el navegador | `sudo tail -n 30 /var/log/apache2/<sitio>_error.log` | [18](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b18) |
| 500 con `could not find driver` | Falta `php-mysql` | `php -m \| grep pdo_mysql` | [19](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b19) |
| Cambié la configuración y no se nota | ¿Pasé `configtest` y recargué? ¿Hay otra directiva posterior que lo pisa? | `sudo apache2ctl configtest` · `sudo apache2ctl -t -D DUMP_INCLUDES` | [14](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b14) |
| `reload` se niega | Errata en la configuración | `sudo apache2ctl configtest` (lee `AH00526` y la línea que indica) | [14](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b14) |
| Contesta el sitio equivocado | ¿Qué sitio es el predeterminado y qué `Host` envías? | `sudo apache2ctl -S` · `curl -H "Host: <nombre>" http://localhost/` | [14](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b14) |
| «Could not resolve host» | ¿Está el nombre en `/etc/hosts`? | `getent hosts <nombre>` | [11](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b11) |
| Aviso del navegador / `curl: (60)` | ¿Es de confianza la CA? ¿Cubre el SAN este nombre? | `openssl x509 -in <crt> -noout -ext subjectAltName` · `openssl verify -CAfile <ca> <crt>` | [16](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b16) |
| Apache no arranca tras instalar el certificado | ¿Casan clave y certificado? ¿Permisos? | `openssl x509 -noout -modulus -in <crt> \| md5sum` (y lo mismo con la clave) · `ls -l /etc/apache2/ssl/` | [16](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b16) |
| No puedo guardar un fichero en `nano` | ¿Lo abrí con `sudo`? | | [13](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b13) |
| Un servicio no arranca | ¿Qué dice su registro? | `systemctl status <srv> --no-pager` · `journalctl -u <srv> -n 50 --no-pager` | [9](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b9) |
| Se llena el disco | ¿Qué ocupa? ¿Logs sin rotar? | `df -h` · `sudo du -h --max-depth=1 /var \| sort -h` | [2](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b2) |
| Un puerto «está ocupado» | ¿Quién lo tiene? | `sudo ss -tlnp \| grep :<puerto>` | [11](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b11) |
| MariaDB: `ERROR 1064` | Error de **sintaxis** SQL (comillas, `;`) | | [19](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b19) |
| MariaDB: `ERROR 1045` | Credencial incorrecta | `mysql -u <usuario> -p` a mano | [19](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b19) |
| 502 Bad Gateway | Apache no alcanza a Tomcat | `systemctl status tomcat10` · `curl -I http://127.0.0.1:8080/` | [20](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b20) |
| FTP: login bien pero `ls` se cuelga | Conexión de datos (modo pasivo / cortafuegos) | `PassivePorts` · `ufw status` | [21](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b21) |
| LDAP: `Invalid credentials (49)` | DN o contraseña del *bind* | `ldapwhoami -x -D "<DN>" -W` | [23](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b23) |
| Docker: el servicio «no ve» a otro | ¿Usas su nombre de servicio, no `localhost`? | `docker compose ps` · `docker compose logs <servicio>` | [25](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b25) |
| Git: «no sé qué he cambiado» | | `git status` · `git diff` | [24](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b24) |
| No sé qué hace un comando | | `man <comando>` · `<comando> --help` | [0](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=b0) |

**La escalera de diagnóstico** (se sube en orden, cada nivel comprueba **una** cosa): proceso vivo → puerto escuchando → responde por HTTP (y con qué código) → la aplicación hace su trabajo. Saltarse niveles es lo que hace pasar media hora mirando el fichero equivocado.

**Si no recuerdas un comando o una opción**, no la adivines: `man <comando>` o `<comando> --help`. Consultar la documentación es parte del trabajo de un administrador.

[↑ Volver al índice](/ifc303-daw/0614-despliegue-aplicaciones-web/Cheatsheet_Comandos.md?id=indice)
