# 0614 · Despliegue de Aplicaciones Web

Cómo trabajamos en este módulo: Solvia

Todo lo que vais a encontrar en estos apuntes gira en torno a un mismo proyecto: Solvia, una aplicación de gestión de incidencias IT (un helpdesk, vamos) que cada alumno instala, configura y hace crecer con sus propias manos a lo largo de todo el curso.

¿Por qué no un ejercicio distinto para cada tema? Porque desplegar aplicaciones web no es una colección de recetas sueltas, es un oficio que se construye sobre lo que ya tienes montado. En una empresa de verdad nadie tira el servidor y empieza de cero cada vez que toca aprender algo nuevo, se sigue trabajando sobre el sistema que ya está en producción. Este módulo funciona igual.

Solvia empieza siendo lo más sencillo posible: un servidor Apache sirviendo páginas PHP contra una base de datos MySQL. Con eso ya se puede abrir un ticket y consultarlo, así que ya hay algo real que enseñar y que defender. A partir de ahí, cada bloque del currículo le añade una capa a la misma aplicación, nunca la sustituye por otra: primero se asegura con HTTPS y se le pone encima gestión de logs, después se le añade un servidor de aplicaciones en Java detrás de Apache (el patrón que se usa en cualquier arquitectura real de servidor web más servidor de aplicaciones), más adelante se monta un servidor FTP para gestionar sus archivos, se documenta y se pone bajo control de versiones, se dockeriza junto con un pipeline de integración continua, y termina con autenticación centralizada por LDAP y su propio nombre en el DNS interno del aula.

Nunca se empieza de cero.

Al final del curso, lo que cada uno tiene desplegado no es un cuaderno de ejercicios resueltos. Es un sistema completo, con historial y con decisiones tomadas por él, que sabe explicar y defender porque lo ha construido paso a paso.

Y esto último importa: aquí no se aprueba solo por entregar. Cada etapa de Solvia se defiende oralmente, se audita buscando fallos reales en configuraciones (propias o ajenas) y queda anotada en una bitácora de decisiones que también cuenta para la nota. La prueba escrita existe, pero es la pieza más pequeña de las cuatro. Lo que de verdad pesa es si sabéis explicar por qué vuestro Solvia está montado como está.

## Unidades didácticas

- [UD1 · Arquitecturas web](/ifc303-daw/0614-despliegue-aplicaciones-web/UD1_Arquitecturas_web.md)

# Descargas

- [UD1 · Arquitecturas web](ifc303-daw/0614-despliegue-aplicaciones-web/UD1_Arquitecturas_web.md ':ignore')
