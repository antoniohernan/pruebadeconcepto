---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 6) – práctico 1 - Wordpress I"
date: "2020-02-08"
---

## Nuestro propio servidor de WordPress

En este, el primer caso práctico de uso real de la Rpi vamos a ver como instalar un servidor completo de WordPress, exactamente igual al que tendríamos si contratamos un hosting en internet.

La utilidad de esto… que cada uno busque su justificación… si estáis intentando montar un blog y no sabéis muy bien como os va a salir la cosa pues… es una estupenda plataforma para las diversas pruebas, y sobre todo barata porqué no os costará nada.

También lo podéis enfocar desde el ámbito lectivo, un servidor de WordPress en local, que cuesta poco (y hoy en día hay que justificar muy mucho los gastos en educación), que no necesita salida a internet por lo tanto no hay esos posibles “peligros” de la chiquillería en clase… etc.

[WordPress](http://es.wikipedia.org/wiki/WordPress) no es más que un software escrito en php que necesita para poderse ejecutar de una base de datos, un intérprete de php y un servidor web.

Para la base de datos, tradicionalmente se ha empleado [MySQL](http://es.wikipedia.org/wiki/Mysql), un motor de base de datos bastante ligero a la par que robusto, si bien, este software no es todo lo “_**libre**_” que debería pues pertenece desde hace un tiempo al todopoderoso Oracle.

Así que en su lugar, en vez de emplear MySQL vamos a utilizar el motor de base de datos [MariaDB](http://es.wikipedia.org/wiki/MariaDB), este software está desarrollado por la propia gente que en su momento realizó MySQL, así que no existen muchas diferencias…

En cuanto al servidor web, lo normal es encontrarse con [Apache](http://es.wikipedia.org/wiki/Servidor_HTTP_Apache), si bien este servidor web en las Raspberry presenta un comportamiento un tanto “gastón” con los recursos, así que en su lugar emplearemos [NginX](http://es.wikipedia.org/wiki/Nginx) el cual se desenvuelve bastante mejor para estos menesteres.

## Necesidades de disco previas

Para generar los directorios donde se guardarán muchos de los ficheros de esta instalación necesitamos más disco del que dispone la tarjeta SD de nuestra máquina.

Puesto que tenemos instalada un pendrive USB vamos a emplearlo para montar sobre el un nuevo FS (FileSystem) que se corresponda con `/srv` .

Por defecto en nuestra instalación ese directorio ya existe y está montado sobre el FS raíz del sistema:

```
[root@Jarvis srv]# df -kh .
S.ficheros Tamaño Usados Disp Uso% Montado en
/dev/root 3,0G 676M 2,2G 24% /

[root@Jarvis srv]# ls -lR
.: total 8
dr-xr-xr-x 2 root ftp 4096 jun 4 2013 ftp
drwxr-xr-x 2 root root 4096 jun 4 2013 http
```

Vamos a crear un nuevo sistema de ficheros en la partición que tenemos libre en el USB, movemos el actual directorio a otro nombre, montamos la unidad y devolvemos el directorio, fácil…

Creamos el sistema de ficheros:

``` 
[root@Jarvis srv]# mkfs -t ext4 /dev/sdb2

mke2fs 1.42.9 (28-Dec-2013)
Etiqueta del sistema de ficheros=
OS type: Linux
Tamaño del bloque=4096 (bitácora=2)
Tamaño del fragmento=4096 (bitácora=2)
Stride=0 blocks, Stripe width=0 blocks
293184 inodes, 1170688 blocks
58534 blocks (5.00%) reserved for the super user
Primer bloque de datos=0
Número máximo de bloques del sistema de ficheros=1199570944
36 bloque de grupos
32768 bloques por grupo, 32768 fragmentos por grupo
8144 nodos-i por grupo
Respaldo del superbloque guardado en los bloques:
32768, 98304, 163840, 229376, 294912, 819200, 884736
Allocating group tables: hecho
Escribiendo las tablas de nodos-i: hecho
Creating journal (32768 blocks):hecho
Escribiendo superbloques y la información contable del sistema de ficheros: hecho
```

Editamos el fichero `/etc/fstab` para configurar el punto de montaje

```
[root@Jarvis srv]# vi /etc/fstab
/dev/sdb1 /BCK ext4 defaults 0 0
/dev/sdb2 /srv ext4 defaults 0 0
```

Añadimos la segunda línea que vemos.

Salvamos el fichero pero aún no montamos la unidad pues el directorio existe y tiene contenido.

Copiamos el directorio srv con otro nombre y lo volvemos a crear para poder hacer el montaje de la unidad USB

```
[root@Jarvis ]# cd /
[root@Jarvis ]# mv /srv /srv2
[root@Jarvis ]# mkdir /srv
```

Ahora podemos montar la partición sin problemas y volver a mover el directorio donde estaba

```
[root@GumPI ]# mount -a
[root@GumPI ]# mv /srv2/* /srv
[root@GumPI ]# rmdir srv2
```

Y para comprobar que lo hemos hecho bien
```
[root@Jarvis ~]# cd /srv
[root@Jarvis srv]# df -kh .
S.ficheros Tamaño Usados Disp Uso% Montado en
/dev/sdb2 4,3G 9,0M 4,1G 1% /srv

[root@Jarvis srv]# ls -lR
.:
total 24
dr-xr-xr-x 2 root ftp 4096 jun 4 2013 ftp
drwxr-xr-x 2 root root 4096 jun 4 2013 http
drwx------ 2 root root 16384 mar 25 19:03 lost+found
```

## Instalamos el servidor WEB

Con nuestro gestor de paquetes, `pacman` vamos a instalar el servidor web

```
[root@Jarvis ~]# pacman -S nginx
resolviendo dependencias...
verificando conflictos...
Paquetes (1): nginx-1.4.7-1
Tamaño Total de Descarga: 0,25 MiB
Tamaño Total Instalado: 0,71 MiB
:: ¿Continuar con la instalación? [S/n] S
:: Recuperando paquetes ...
nginx-1.4.7-1-armv6h 251,9 KiB 913K/s 00:00      [##################################################] 100%
(1/1) verificando llaves en el llavero           [##################################################] 100%
(1/1) verificando la integridad de los paquetes  [##################################################] 100%
(1/1) cargando los archivos del paquete...       [##################################################] 100%
(1/1) verificando conflictos entre archivos      [##################################################] 100%
(1/1) verificando el espacio disponible en disco [##################################################] 100%
(1/1) instalando nginx                           [##################################################] 100%
```

Lo arrancamos para ver si todo funciona correctamente

`[root@Jarvis ~]# systemctl start nginx`

Y comprobamos que se puede acceder al servidor por el puerto web por defecto que es el 80.

Nos conectamos a nuestra Raspberry 192.168.1.20 (_por aquí tu IP_) ([http://192.168.1.20](http://192.168.1.20)) y debemos ver un
mensaje tal que así:

```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.

Commercial support is available at nginx.com.

Thank you for using nginx.
```

Perfecto, ya tenemos el servidor web instalado, ahora instalamos PHP y PHP-FPM (es el FastCGI para PHP) y terminamos de configurarlo.

## Instalamos PHP y PHP-FPM

Como siempre, con `pacman` , solicitamos a los repositorios los paquetes oficiales.

```
[root@Jasrvis ~]# pacman -S php php-fpm
resolviendo dependencias...
verificando conflictos...
Paquetes (3): libxml2-2.9.1-5 php-5.5.10-1 php-fpm-5.5.10-1
Tamaño Total de Descarga: 5,46 MiB
Tamaño Total Instalado: 29,19 MiB
:: ¿Continuar con la instalación? [S/n] S
:: Recuperando paquetes ...
libxml2-2.9.1-5-armv6h 1090,4 KiB 2030K/s 00:01      [##################################################] 100%
php-5.5.10-1-armv6h 2,8 MiB 3,16M/s 00:01            [##################################################] 100%
php-fpm-5.5.10-1-armv6h 1675,1 KiB 4,21M/s 00:00     [##################################################] 100%
(3/3) verificando llaves en el llavero               [##################################################] 100%
(3/3) verificando la integridad de los paquetes      [##################################################] 100%
(3/3) cargando los archivos del paquete...           [##################################################] 100%
(3/3) verificando conflictos entre archivos          [##################################################] 100%
(3/3) verificando el espacio disponible en disco     [##################################################] 100%
(1/3) instalando libxml2                             [##################################################] 100%
(2/3) instalando php                                 [##################################################] 100%
(3/3) instalando php-fpm                             [##################################################] 100%
```

Arrancamos PHP-FPM (FastGCI Process Manager, donde FastCGI es un Gateway para la conexión de procesos web…)
```
[root@Jarvis ~]# systemctl start php-fpm
```

Y configuramos el conjunto NginX PHP y PHP-FPM

Por defecto NginX busca los ficheros en `/usr/share/nginx/html` , vamos a modificar su configuración básica para que los busque en `/srv/http`, ajustamos también algunos parámetros para la gestión de los lobs de errores, compresión de mensajes… etc.

Editamos `/etc/nginx/nginx.conf` , borramos todo su contenido y lo reemplazamos con este

```
#user html;
worker_processes 1;
error_log /var/log/error_nginx.log;
events {
        worker_connections 1024;
       }
http {
      include mime.types;
      default_type application/octet-stream;
      sendfile on;
      keepalive_timeout 15;
      gzip on;
      gzip_comp_level 1;
      server {
            listen 80;
            server_name localhost;
            location ~ \.php {
                              root /srv/http;
                              fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
                              fastcgi_index index.php;
                              fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                              include /etc/nginx/fastcgi_params;
                              }
            location / {
                        root /srv/http;
                        index index.php;
                        }
            error_page 500 502 503 504 /50x.html;
            location = /50x.html {
                                  root /usr/share/nginx/html;
                                  }
      }
}
```

Ahora creamos un fichero de prueban en php para ver si nuestra instalación y configuración son correctas, así que editamos el fichero `/srv/http/index.php` y usamos este contenido:

```
<?php
echo "<p>Prueba</p>";
?>
```

Reiniciamos tanto NginX como PHP-FPM y probamos a ver si se ve esta prueba

```
[root@Jarvis ~]# systemctl restart nginx php-fpm
```

Probamos el acceso nuevamente a nuestro servidor web en la url http://192.168.1.20 (por aquí tu IP) ([http://192.168.1.20](http://192.168.1.20))

Y tendremos que ver como el navegador web nos muestra en texto “**Prueba**” que se genera con ese sencillo `.php` de índice que hemos creado.

Sólo nos falta dejar estos dos servicios activos en los futuros inicios de máquina mediante los comandos:

```
[root@Jarvis ~]# systemctl enable nginx
[root@Jarvis ~]# systemctl enable php-fpm
```

## Instalación y configuración de MariaDB

Vamos con la base de datos.

Nuestro amigo pacman, lo volvemos a emplear para la descarga del software, como vemos las posibles dependencias del mismo, librerías, clientes, etc, se instalan junto con el software.

Prestad especial atención al final de la instalación por que este paquete ofrece una documentación “post-instalación” muy buena, de los pocos…

``` 
[root@ Jarvis ~]# pacman -S mariadb
resolviendo dependencias...
verificando conflictos...
Paquetes (4): libaio-0.3.109-7 libmariadbclient-5.5.36-1 mariadb-clients-5.5.36-1 mariadb-5.5.36-1
Tamaño Total de Descarga: 16,32 MiB
Tamaño Total Instalado: 141,79 MiB
:: ¿Continuar con la instalación? [S/n] S
:: Recuperando paquetes ...
libaio-0.3.109-7-armv6h 4,1 KiB 4,03M/s 00:00            [##################################################] 100%
libmariadbclient-5.5.36-1-armv6h 6,6 MiB 2,46M/s 00:03   [##################################################] 100%
mariadb-clients-5.5.36-1-armv6h 799,6 KiB 1221K/s 00:01  [##################################################] 100%
mariadb-5.5.36-1-armv6h 8,9 MiB 2,04M/s 00:04            [##################################################] 100%
(4/4) verificando llaves en el llavero                   [##################################################] 100%
(4/4) verificando la integridad de los paquetes          [##################################################] 100%
(4/4) cargando los archivos del paquete...               [##################################################] 100%
(4/4) verificando conflictos entre archivos              [##################################################] 100%
(4/4) verificando el espacio disponible en disco         [##################################################] 100%
(1/4) instalando libaio                                  [##################################################] 100%
(2/4) instalando libmariadbclient                        [##################################################] 100%
(3/4) instalando mariadb-clients                         [##################################################] 100%
(4/4) instalando mariadb                                 [##################################################] 100%
Installing MariaDB/MySQL system tables in '/var/lib/mysql' ... OK
Filling help tables... OK
To start mysqld at boot time you have to copy support-files/mysql.server to the right place for your system
PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER !
To do so, start the server, then issue the following commands:
'/usr/bin/mysqladmin' -u root password 'new-password'
'/usr/bin/mysqladmin' -u root -h Jarvis password 'new-password'
Alternatively you can run:
'/usr/bin/mysql_secure_installation'
which will also give you the option of removing the test
databases and anonymous user created by default. This is
strongly recommended for production servers.

See the MariaDB Knowledgebase at http://mariadb.com/kb or the
MySQL manual for more instructions.
You can start the MariaDB daemon with:
cd '/usr' ; /usr/bin/mysqld_safe --datadir='/var/lib/mysql'
You can test the MariaDB daemon with mysql-test-run.pl
cd '/usr/mysql-test' ; perl mysql-test-run.pl
Please report any problems at http://mariadb.org/jira
The latest information about MariaDB is available at http://mariadb.org/.
You can find additional information about the MySQL part at:
http://dev.mysql.com
Support MariaDB development by buying support/new features from
SkySQL Ab. You can contact us about this at sales@skysql.com.

Alternatively consider joining our community based development effort:
http://mariadb.com/kb/en/contributing-to-the-mariadb-project/

>> If you are migrating from MySQL, don't forget to run 'mysql_upgrade'
after mysqld.service restart.
```

Iniciamos el gestor de base de datos, para poder configurarlo, este inicio tarda un poquito, no asustarse y correr a reiniciar el servidor

`[root@Jarvis ~]# systemctl start mysqld`

Y ahora realizamos la configuración inicial de MariaDB empleando el script que esta documentación del paquete nos indica (hay un error en el script pero no afecta a su funcionalidad, lleva ahí desde hace varios meses…):

```
[root@Jarvis ~]# /usr/bin/mysql_secure_installation
/usr/bin/mysql_secure_installation: línea 379: find_mysql_client: no se encontró la orden
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.
Enter current password for root (enter for none): <pulsamos intro>
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.
Set root password? [Y/n] Y
New password: <ponemos clave>
Re-enter new password: <repetimos clave>
Password updated successfully!
Reloading privilege tables..
... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them. This is intended only for testing, and to make the installation
go a bit smoother. You should remove them before moving into a
production environment.
Remove anonymous users? [Y/n] Y
... Success!

Normally, root should only be allowed to connect from 'localhost'. This
ensures that someone cannot guess at the root password from the network.
Disallow root login remotely? [Y/n] Y
... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access. This is also intended only for testing, and should be removed
before moving into a production environment.
Remove test database and access to it? [Y/n] Y
- Dropping test database...
... Success!

- Removing privileges on test database...
... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
Reload privilege tables now? [Y/n] Y
... Success!

Cleaning up...
All done! If you've completed all of the above steps, your MariaDB
installation should now be secure.
Thanks for using MariaDB!
```

Y una vez configurado todo, reincidamos el gestor de base de datos y lo habilitamos en el arranque de máquina

```
[root@Jarvis ~]# systemctl restart mysqld
[root@Jarvis ~]# systemctl enable mysqld
```

## Administración de MariaDB con PHPMyAdmin

Para hacer la vida un poco más fácil, y poder administrar la base de datos de una manera algo más gráfica, instalamos el paquete de administración basado en Php.

Más que nada por que en cualquier Hosting de pago es el paquete que con casi toda seguridad os encontraréis, así que vamos a tratar de poner aquí todo igual.

Recurrimos a pacman una vez más:

```
[root@Jarvis ~]# pacman -S phpmyadmin php-mcrypt
resolviendo dependencias...
verificando conflictos...
Paquetes (3): libmcrypt-2.5.8-3 php-mcrypt-5.5.10-1 phpmyadmin-4.1.11-1
Tamaño Total de Descarga: 4,78 MiB
Tamaño Total Instalado: 26,21 MiB
:: ¿Continuar con la instalación? [S/n] S
:: Recuperando paquetes ...
libmcrypt-2.5.8-3-armv6h 64,9 KiB 319K/s 00:00       [##################################################] 100%
php-mcrypt-5.5.10-1-armv6h 9,9 KiB 236K/s 00:00      [##################################################] 100%
phpmyadmin-4.1.11-1-any 4,7 MiB 2,52M/s 00:02        [##################################################] 100%
(3/3) verificando llaves en el llavero               [##################################################] 100%
(3/3) verificando la integridad de los paquetes      [##################################################] 100%
(3/3) cargando los archivos del paquete...           [##################################################] 100%
(3/3) verificando conflictos entre archivos          [##################################################] 100%
(3/3) verificando el espacio disponible en disco     [##################################################] 100%
(1/3) instalando phpmyadmin                          [##################################################] 100%
Check http://wiki.archlinux.org/index.php/Phpmyadmin for details.
Dependencias opcionales para phpmyadmin
php-mcrypt: to use phpMyAdmin internal authentication [pendiente]
(2/3) instalando libmcrypt                           [##################################################] 100%
(3/3) instalando php-mcrypt                          [##################################################] 100%
```

Por defecto la instalación se realiza dentro de la ruta `/usr/share/webapps/phpMyAdmin` y nuestro servidor web no está configurado para arrancar nada allí.

La solución, pues fácil, creamos un enlace de es ruta en nuestra ruta de servidor http y así podremos ejecutarlo.

Se podría mover/copiar el contenido de esa webapps a la carpeta, pero de cara a actualizaciones del paquete es más recomendable hacerlo por enlace.

```
[root@Jarvis ~]# ln -s /usr/share/webapps/phpMyAdmin /srv/http/phpMyAdmin
```

Y ahora tenemos que volver a modificar el fichero de configuración del servidor web `/etc/nginx/nginx.conf` , este sería el contenido del fichero que debemos tener:

```
#user html;
worker_processes 1;
error_log /var/log/error_nginx.log;
events {
    worker_connections 1024;
    }
http {
    include mime.types;
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 15;
    gzip on;
    gzip_comp_level 1;
    server {
        listen 80;
    server_name localhost;
    location ~ \.php {
            root /srv/http;
            fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include /etc/nginx/fastcgi_params;
            }
    location /phpmyadmin {
            rewrite ^/* /phpMyAdmin last;
            }
    location / {
            root /srv/http;
            index index.php;
            }
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    root /usr/share/nginx/html;
            }
    }
}
```

También debemos modificar el fichero `/etc/php/php.ini` realizando estos cambios que a continuación se detallan:

Localizamos la línea que dice `open_basedir=` y ponemos esto:

`open_basedir= /srv/http/:/home/:/tmp/:/usr/share/pear/:/usr/share/webapps/:/etc/webapps/`

Localizamos estas líneas, y las descomentamos, esto es, quitamos el ; del principio de la línea (pulsando la letra x)

```
extension=mysqli.so
extension=mysql.so
extension=mcrypt.so
mysqli.allow_local_infile = On
```

Como hemos realizado modificaciones que afectan a la configuración del servidor web y del FastCGI de PHP, vamos a reiniciar ambos procesos

```
[root@Jarvis nginx]# systemctl restart nginx php-fpm
```

Y probamos para ver si hemos conseguido instalar todo bien.

Para esto, entramos a la url http://192.168.1.20/phpmyadmin (por aquí tu IP) ([http://192.168.1.20/phpmyadmin](http://192.168.1.20/phpmyadmin))

... [Continuará](rpi_wordpress2.md) ...
