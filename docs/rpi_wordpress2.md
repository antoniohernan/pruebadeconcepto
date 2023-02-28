---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 7) – práctico 1 – WordPress II"
date: "2020-03-05"
---

## Y ahora si, por fin, instalamos WordPress!!

Ha costado un poquito pero ya está, ahora lo más rápido, montar WordPress, así que descargamos el paquete directamente de wordpress.org y los descomprimimos:

\[root@Jarvis http\]# cd /srv
\[root@Jarvis http\]# wget http://wordpress.org/latest.tar.gz
--2014-03-25 22:55:41-- http://wordpress.org/latest.tar.gz
Resolviendo wordpress.org (wordpress.org)... 66.155.40.249, 66.155.40.250
Conectando con wordpress.org (wordpress.org)\[66.155.40.249\]:80... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 5869727 (5,6M) \[application/x-gzip\]
Grabando a: “latest.tar.gz”
100%\[======================================================================================================>\] 5.869.727
1,49MB/s en 4,4s
2014-03-25 22:55:46 (1,28 MB/s) - “latest.tar.gz” guardado \[5869727/5869727\]
\[root@Jarvis srv\]# tar -xvf latest.tar.gz

Solo nos resta hacer la configuración de WordPress, podemos echar mano de [esta guía](http://codex.wordpress.org/Installing_WordPress#Famous_5-Minute_Install) relativa a la instalación de un WP en 5 minutos.

Los pasos son estos:

1.- Creamos una base de datos para nuestro WordPress, fácil, tenemos phpMyAdmin, nos conectamos y creamos.

2.- Creamos un usuario para esta base de datos con permisos para crear/modificar objetos (nos aparece como “Otorgar todos los privilegios para la base de datos "WordPress”)

Estos dos primeros pasos se pueden realizar, por supuesto, por línea de comando, las sentencias serían estas:

\[root@Jarvis http\]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor. Commands end with ; or \\g.
Your MariaDB connection id is 8
Server version: 5.5.36-MariaDB-log MariaDB Server
Copyright (c) 2000, 2014, Oracle, Monty Program Ab and others.
Type 'help;' or '\\h' for help. Type '\\c' to clear the current input statement.
MariaDB \[(none)\]> CREATE DATABASE blog;
Query OK, 1 row affected (0.01 sec)
MariaDB \[(none)\]> CREATE USER wordpressuser@localhost;
Query OK, 0 rows affected (0.00 sec)
MariaDB \[(none)\]> SET PASSWORD FOR wordpressuser@localhost=PASSWORD('clave');
Query OK, 0 rows affected (0.01 sec)
MariaDB \[(none)\]> GRANT ALL PRIVILEGES ON blog.\* TO wordpressuser@localhost IDENTIFIED BY 'clave';
Query OK, 0 rows affected (0.00 sec)
MariaDB \[(none)\]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)
MariaDB \[(none)\]> exit
Bye
\[root@Jarvis http\]#

3.- Creamos el fichero wp-config.php a partir del fichero de ejemplos

\[root@Jarvis wordpress\]# cp /srv/wordpress/wp-config-sample.php /srv/wordpress/wp-config.php

4.- Editar este fichero `/srv/wordpress/wp-config.php` para modificar la información de la conexión a la base de datos que hemos creado en el punto 1, nombre de la base de datos, usuario y clave.

define('DB\_NAME', 'blog');
/\*\* MySQL database username \*/
define('DB\_USER', 'wordpressuser');
/\*\* MySQL database password \*/
define('DB\_PASSWORD', 'clave');

5.- Copiamos la instalación/configuración a su lugar bajo `/srv/http`

\[root@Jarvis srv\]# cd /srv/wordpress
\[root@Jarvis wordpress\]# cp -pr \* /srv/http

Por último, la url para arrancar la configuración final de WordPress es esta: [http://192.168.1.20/wp-admin/install.php](http://192.168.1.20/wp-admin/install.php)

Por si lo quieres tener en castellano, puedes hacerlo de dos maneras, bajando el paquete completo de instalación en castellano, que siempre va un poco por detrás en las nuevas versiones que las americanas, e instalando tal y como hemos hecho pero a partir de ese fichero, y la otra opción es poner tan sólo el fichero de lenguaje, que es lo que te cuento a continuación y que hasta ahora no he conseguido que funcione al 100% y siempre termina apareciendo algún que otro literal en otro idioma…

Debes bajar la traducción de WordPress para la versión en concreto que tienes, ahora mismo es la 3.8.1.

La descarga la hacemos así

\[root@Jarvis srv\]# cd /srv
\[root@Jarvis srv\]# wget http://es.wordpress.org/wordpress-3.8.1-es\_ES.zip

Debemos extraer de ese zip el fichero wordpress/wp-content/languages/es\_ES.mo para copiarlo dentro de nuestra instalación en la misma ruta.

\[root@Jarvis srv\]# unzip -jqq wordpress-3.8.1-es\_ES.zip wordpress/wp-content/languages/es\_ES.mo
\[root@Jarvis srv\]# mkdir /srv/http/wp-content/languages
\[root@Jarvis srv\]# mv /srv/es\_ES.mo /srv/http/wp-content/languages

En último lugar debemos modificar el fichero de configuración de WordPress para que indicar el lenguaje que vamos a emplear, editando el fichero /srv/http/wp-config.php y modificando esta línea para que quede así:

define('WPLANG', 'es\_ES');

## Acceso desde el exterior de nuestra red

Si queremos acceder a nuestro servidor desde fuera de nuestra red debemos habilitar el acceso a el en nuestro router.

Pero claro, a no ser que tengas dirección IP fija tendremos problemas cuando el router obtenga una nueva dirección IP.

Para solucionar esto podemos emplear un servicio de [DDNS](http://es.wikipedia.org/wiki/DNS_dinámico) , que en algunos router incluso podremos configurar y activar allí.

En cualquier caso, aquí os pongo las sencillas líneas para poder instalar el cliente de DDNS de [www.no-ip.com](http://www.noip.com)

Primero necesitamos tener un registro en ese servicio antes de continuar.

Luego tendremos que añadir un host (nombre con el que queremos que se conozca nuestro servicio en la red) con un dominio de los gratuitos que nos ofrece.

En este caso hemos registrado gumcampi.no-ip.org

Ahora tenemos que abrir en nuestro router el acceso a nuestra máquina por el puerto que vamos a emplear, en este caso la Ip de la red local 192.168.1.20 que será accedido por el puerto 80. Así que sencillo, entramos a Nat-VirtualServers y añadimos para esta IP y este puerto la entrada correspondiente.

Una nota al margen, los router ya emplean ese puerto, el 80 para la web de administración, normalmente cuando indicamos que vamos a emplear ese puerto para algo el router suele modificar su propio puerto y se lo lleva al 8080.

Y ahora instalamos el cliente de no-ip.

\[root@Jarvis ~\]# pacman -S noip

resolviendo dependencias...
verificando conflictos...
Paquetes (1): noip-2.1.9-5
Tamaño Total de Descarga: 0,02 MiB
Tamaño Total Instalado: 0,07 MiB
:: ¿Continuar con la instalación? \[S/n\] S
:: Recuperando paquetes ...
noip-2.1.9-5-armv6h 15,6 KiB 129K/s 00:00        \[################################################\] 100%
(1/1) verificando llaves en el llavero           \[################################################\] 100%
(1/1) verificando la integridad de los paquetes  \[################################################\] 100%
(1/1) cargando los archivos del paquete...       \[################################################\] 100%
(1/1) verificando conflictos entre archivos      \[################################################\] 100%
(1/1) verificando el espacio disponible en disco \[################################################\] 100%
(1/1) instalando noip                            \[################################################\] 100%

Before running noip2 you must configure it.
To configure noip2 run the command "noip2 -C -Y"

Como vemos el propio instalador nos indica la forma de realizar la configuración así que le hacemos caso:

\[root@Jarvis ~\]# noip2 -C -Y
Auto configuration for Linux client of no-ip.com.
Please enter the login/email string for no-ip.com antonioXXXX@gmail.com
Please enter the password for user 'antonioXXXX@gmail.com' \*\*\*\*\*\*
Only one host \[gumcampi.no-ip.org\] is registered to this account.
It will be used.
Please enter an update interval:\[30\] 30
Do you wish to run something at successful update?\[N\] (y/N) N
New configuration file '/etc/no-ip2.conf' created.

Y estando instalado y configurado tan sólo nos falta ejecutarlo y dejarlo activo para posteriores arranques automáticos del sistema

\[root@Jarvis ~\]# systemctl start noip2
\[root@Jarvis ~\]# systemctl enable noip2
ln -s '/usr/lib/systemd/system/noip2.service' '/etc/systemd/system/multi-user.target.wants/noip2.service'

Ahora podréis conectar desde el exterior siempre bajo esa dirección.

Eso si, os aviso, si probáis esto desde vuestra casa es posible que os falle, ya que la mayoría de los router no os dejarán salir al exterior para volver a entrar a la red local, es lo que se denomina [NAT Loopback](http://opensimulator.org/wiki/NAT_Loopback_Routers) y son pocos los routers que lo soportan.

Y una última cosa, de cara a la configuración de WordPress, si accedéis así, recordad que tenéis que indicar en la página de configuración del site (http://192.168.1.20/wp-admin/options-general.php) el nombre del dominio por DDNS en vez de la dirección ip Local.
