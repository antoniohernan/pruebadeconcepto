---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 16) – Apéndice II"
date: "2020-03-31"
---

## XBMC con MariaDB para instalación distribuida

Como imagino que no seré el único que se encuentra tarde o temprano desplazado del televisor del salón, os cuento la forma de adaptar vuestros XBMC para que se puedan seguir usando tanto los ficheros de media de vuestros discos duros conectados a la Pi, como la base de datos de contenidos para las marcas de capítulos vistos, punto de reproducción etc.

En principio está todo bastante basado en una guía que publico Arthurbetico en NasyMas, adaptándolo a mi uso y necesidades por que claro, yo no tengo NAS.

Evalúa de todas maneras si esto que he hecho te sirve para algo, yo ahora mismo tengo la versión Frodo 12.3, la versión Gotham 13 está a pocas semanas de salir y anuncian cambios en todo esto de la “compartición” de contenidos, lo mismo luego es mucho más fácil… o no…

De lo que se trata es de instalar un motor de base de datos (MariaDB), configurar XBMC para que use este motor, y además crear los “Scrapers” de una forma determinada para que todo se comporte igual estemos ejecutando XBMC en nuestra Rpi o en nuestro iMac/MacBook o Pc con Windows (arggg).

Debemos tener en cuenta que todos los “clientes” de XBMC que empleemos deberán ser la misma versión, esto es, o todos con Frodo 12.3 o ninguno...

Una implicación que tiene esto es que vais a perder toda la información de vuestra base de datos de XMBC, marcadores de visto/no visto, etc.

He probado las opciones de exportación/importanción de datos pero no he sido capaz de hacerlo funcionar, por otro lado el cambio que os voy a comentar a continuación en los scrapers no es compatible con los datos que tendríais en esas bases de datos actuales basadas en ficheros de texto y nos exigiría realizar tareas de actualización de datos en la base de datos MariaDB a manita… yo ya me he visto en estas cosas miles de veces por trabajo, pero a alguno se le puede fundir el “fusible” así que mejor esto otro…

Por partes entonces, lo primero parar xbmc si lo tenemos funcionando.

[root@Jarvis ~]# systemctl stop xbmc

Ahora instalamos MariaDB

[root@Jarvis ~]# pacman -S mariadb

resolving dependencies...
looking for inter-conflicts...
Packages (1): mariadb-5.5.36-1
Total Download Size: 8.90 MiB
Total Installed Size: 88.27 MiB
:: Proceed with installation? [Y/n] Y
:: Retrieving packages ...
mariadb-5.5.36-1-armv6h 8.9 MiB 2.80M/s 00:03 [##########################################] 100%
(1/1) checking keys in keyring                [##########################################] 100%
(1/1) checking package integrity              [##########################################] 100%
(1/1) loading package files                   [##########################################] 100%
(1/1) checking for file conflicts             [##########################################] 100%
(1/1) checking available disk space           [##########################################] 100%
(1/1) installing mariadb                      [##########################################] 100%

>> If you are migrating from MySQL, don't forget to run 'mysql_upgrade'
after mysqld.service restart.
synchronizing filesystem...

Arrancamos MariaDB y la configuramos:

[root@Jarvis lib]# systemctl start mysqld

[root@Jarvis lib]# systemctl enable mysqld
ln -s '/usr/lib/systemd/system/mysqld.service' '/etc/systemd/system/multi-user.target.wants/mysqld.service'

[root@Jarvis lib]# mysql_secure_installation
/usr/bin/mysql_secure_installation: line 379: find_mysql_client: command not found
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!
In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password:
Re-enter new password:
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

Bien, ahora vamos a crear un usuario dentro de nuestro gestor de base de datos, con privilegios para crear su propia base de datos.

Recordar que la contraseña del usuario root que hemos puesto hace un momento al configurar MariaDB (ó Mysql) es la del usuario root de la base de datos, no del sistema operativo!!.

[root@Jarvis ~]# mysql -u root -p

Enter password:
Welcome to the MariaDB monitor. Commands end with ; or \\g.
Your MariaDB connection id is 141
Server version: 5.5.37-MariaDB-log MariaDB Server
Copyright (c) 2000, 2014, Oracle, Monty Program Ab and others.
Type 'help;' or '\\h' for help. Type '\\c' to clear the current input statement.
MariaDB [(none)]> create user 'xbmc' identified by 'xbmc';
MariaDB [(none)]> grant all on *.* to 'xbmc';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
Bye

Siguiente paso, configurar XBMC para que utilice esa base de datos.

El fichero de configuración será el mismo que empleemos luego en nuestros clientes xbmc en el entorno MAC.

Debemos ejecutar la siguiente cadena de comandos:

[root@Jarvis ~]# vi ~xbmc/.xbmc/userdata/advancedsettings.xml

Introducir este contenido y salvar documento.

<advancedsettings>
 <videodatabase>
  <type>mysql</type>
  <host>192.168.1.20</host>
  <name>xbmc_video</name>
  <port>3306</port>
  <user>xbmc</user>
  <pass>xbmc</pass>
 </videodatabase>
 <musicdatabase>
  <type>mysql</type>
  <host>192.168.1.20</host>
  <name>xbmc_music</name>
  <port>3306</port>
  <user>xbmc</user>
  <pass>xbmc</pass>
 </musicdatabase>
</advancedsettings>

Importante, cambiad la IP de vuestra máquina en vuestra LAN.

Como hemos editado con root, le damos el fichero al usuario xbmc:

[root@Jarvis ~]# chown xbmc:xbmc ~xbmc/.xbmc/userdata/advancedsettings.xml

Y ahora, por último antes de arrancar y empezar a configurar scrapers en XBMC nos faltan dos tareas, la primera es exportar nuestra biblioteca de medios por SAMBA.

¿Por que?, pues porqué así el resto de equipos de la red van a ver los ficheros compartidos, es más, nuestra propia RPi va a acceder a sus propios ficheros de media por SAMBA, si, el local accediendo por SAMBA, por que así es la única manera de mantener un acceso único y uniforme al fichero, todos accediendo por igual.

Sin más líos, SAMBA [ya hemos visto como ponerlo en marcha](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-8-practico-2-gestor-de-descargas/), basta con decir que tenemos que exportar esto:

[root@Jarvis ~]# vi /etc/samba/smb.conf
[global]
workgroup = JarvisGRP
security = user
load printers = no

[Videos]
path = Videos
public = no
writable = yes
browseable = yes
valid users = root

Yo tengo colgando de `/Videos` las carpetas `/Videos/Series` , `/Videos/Pelis` y `/Videos/Documentales` .

Así que el acceso por SAMBA [smb://192.168.1.20/Videos](smb://192.168.1.20/Videos) será nuestro punto de entrada al sistema de medios.

Podéis incluso probar que funciona, desde vuestros OSX, abriendo un nuevo finder (CMD + N) y pulsado sobre conectar a servidor (CMD +K) con ese [smb://vuestraIP/vuestroShare](smb://vuestraIP/vuestroShare) podréis acceder a la Rpi por SAMBA.

Recordad activar samba en el inicio con

[root@Jarvis ~]# systemctl restart smbd nmbd
[root@Jarvis ~]# systemctl enable smbd nmbd

La otra tarea que hace falta hacer es el arranque de MariaDB, ahora está puesto en automático que arranque siempre cuando se inicie el sistema, pero siendo un proceso algo pesado, pues quizás nos sea más conveniente que se inicie siempre solo cuando XMBC esté activado también para arrancar ¿no?, total, van a ir de la mano.

Para esto que parece que es mucho y no es nada, editamos el fichero `/usr/lib/systemd/system/xbmc.service` para añadir en After y Requires el servicio de base de datos:

[Unit]

Description = Starts instance of XBMC using xinit
After = remote-fs.target mysqld.service
Requires = mysqld.service

Y en el fichero `/usr/lib/systemd/system/mysqld.service` añadir esto en Before:

[Unit]

Description=MariaDB database server
After=syslog.target
Before = xbmc.service

Si tenemos los servicios de xbmc y mysqld ya activos en el inicio debemos refrescar esos demonios, ejecutando esto:

[root@Jarvis system]# systemctl daemon-reload

Y listo, vamos a empezar, arrancamos XBMC, se nos arrancará la base de datos, bien, sin problemas.

Ahora BORRAMOS los scrapers y todo lo que tuviésemos de antes (ojo borrar datos, no ficheros) y volvemos a empezar a crear de nuevo.

Cuando entremos a crear los scrapers de nuevo, en la ventana en la que escogemos la ruta debemos desplazarnos hasta que encontremos una que nos indica acceso por Samba o por SMB, esa es la nuestra, tan sólo seguir los pasos de la pantalla, poner usuarios y claves donde nos lo pida y listo.

No es muy complicado y ya lo hemos hecho una vez, a ver si puedo poner unas capturas de pantalla.

Al terminar todo habremos reconstruido nuestra base de datos pero utilizando una base de datos a la que podemos atacar desde el exterior, en nuestra LAN.

La parte de OSX, pues muy fácil, instalamos XBMC (misma versión que en la RPI, recordad) y generamos el fichero advancedsettings.xml tal y como tenemos en nuestra RPI, el mismo contenido.

Este fichero está en `/Users/TUUAURIO/Library/Application Support/XBMC/userdata` y si no existe lo debéis crear.

Cuando os arranque XBMC de vuestro OSX es crear los scrapers tal y como hemos hecho en la RPi (acceso único por samba. genial verdad?!!) y listo, ya veréis como las marcas de visto/no visto se sincronizan y los puntos de reproducción también.

Pendiente dejo lo de las capturas… que ando con lío.
