---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 10) – práctico 4 - Servidor copias de Seguridad"
date: "2020-02-08"
---

## Servidor de Copias de Seguridad

Si, ya sé que con nuestros MACs tenemos la estupenda utilidad de Time Machine, que funciona a las mil maravillas, e incluso si queremos tenemos también SuperDuper que es un muy buen complemento para nuestras copias en Time Machine.

Pero esto que os voy a contar seguramente interese a más de uno, y es que cuando por ejemplo acabas a las mil de editar un podcast, de retocar fotos o de editar video, y quieres hacer copia de seguridad de esas carpetas… seguramente que incluso esas carpetas no las tengas en copia de seguridad de Time Machine por que son grandes y tardan mucho en hacerse copia, te llenan el disco de TM en poco.. así que te toca esperarte a la copia en otro HD ó Pendrive termine para poder irte a dormir tranquilo.

Pues bien, vamos a ver un par de maneras de hacer **copia de seguridad SOBRE nuestra RPi de una parte en concreto de nuestro dispositivos OSX**.

Incluso se dice que si tienes un dispositivo iOS con JailBreak también es posible hacer copia de seguridad de ellos, pero esto es algo que no puedo contar pues no tengo JailBreak en ninguno de mis dispositivos.

## Servidor FTP

El protocolo de transferencia de datos [FTP](http://es.wikipedia.org/wiki/File_Transfer_Protocol)  nos permite, mediante una infraestructura de cliente/servidor, realizar la transferencia de datos en ambas direcciones de una manera bastante rápida, tanto es así que el protocolo está optimizado para que sea rápido, **pero no para que sea seguro**, básicamente porqué la información viaja en claro, no está cifrada.

Para mejorar esto, es decir, renunciar a parte de esa velocidad pero ganar mucho en seguridad tenemos la transmisión [SFTP](http://es.wikipedia.org/wiki/SSH_File_Transfer_Protocol) , que es algo así como montar un FTP pero sobre una capa de cifrado SSL (Secure Socket Layer).

En el caso de las Rpi con ARCH, al estar SSH habilitado por defecto también tenemos acceso al subsistema de sftpserver, de manera que es el propio demonio de SSH (sshd) quien invoca al servidor de ftp seguro cuando así el cliente lo solicita.

Y como en nuestra conexión, en el apartado de securización hemos, por un lado **eliminado el acceso** directo **con el usuario root**, y por otro **activado la autenticación en dos pasos** para el usuario operador, pues aquí nos pasará lo mismo, si abrimos una conexión sftp a nuestra máquina veremos como con el usuario root es imposible que obtengamos una sesión, y que cuando la abrimos con el usuario operador nos solicita lo primero el código de verificación de la autenticación en dos pasos y luego la contraseña del usuario.

Así pues, para esto no tenemos que hacer nada, tan sólo saber que tenemos este tipo de conexión disponible y que podemos pasar, tanto de ida como de vuelta, documentos y ficheros de todo tipo, eso sí, siempre basándonos en el perfilado del usuario con el que nos conectamos.

Podemos incluso utilizar un cliente de FTP para que nos haga el entorno más amigable a la par de gráfico, por ejemplo podemos emplear [Filezilla](https://filezilla-project.org) , configurando la conexión como del tipo SFTP - SSH File Transfer Protocol y el modo de acceso Interactivo, para que veamos el mensaje del sistema en el que se nos solicita el código de verificación en dos pasos que implementamos.

Esto lo comento aunque no es el recurso de copia de seguridad que vamos a emplear para este fin, es “otra” forma más de poder pasar contenidos a nuestra máquina de backup.

## Servidor RSYNC

Sobre Rsync ya hablamos en el capítulo de copias de seguridad, y ahora vamos a ampliar la información por que este software también podemos explotarlo como parte servidor, esto es, activarlo de manera que otros equipos actúen como clientes y puedan hacer copia sobre nuestra RPI.

Para eso tan sólo tenemos que configurarlo adecuadamente y ejecutarlo de manera que se quede funcionando en la máquina en modo “demonio”.

La configuración es sencilla, en primer lugar tenemos el fichero de configuración `/etc/rsyncd.conf`

Aquí un ejemplo a usar:

uid = nobody
gid = nobody
use chroot = no
max connections = 4
syslog facility = local5
pid file = /run/rsyncd.pid

Ahora vamos a poner a correr rsync en modo demonio y a probar a grabar desde nuestro equipo Mac

[root@Jarvis ~]# systemctl start rsyncd

En caso de que lo queramos dejar corriendo siempre que se inicie el sistema operativo de la máquina, este es el comando:

[root@Jarvis ~]# systemctl enable rsyncd

Y la línea por ejemplo para copiar un directorio sobre el home del usuario operador (el único que puede conectar y autenticar por SSH) sería:

Hal9000:~ $ rsync --progress -avhe ssh /Users/admin/Documents/Vehiculos/Octavia/Manuales/ operador@192.168.1.20:CopiaRemota/

Verification code:
Password:
building file list ...
3 files to consider
created directory CopiaRemota
./
A5_Octavia_OwnersManual.pdf
4.05M 100% 6.34MB/s 0:00:00 (xfer#1, to-check=1/3)
A5_Octavia_Swing_CarRadio.pdf
671.79K 100% 831.48kB/s 0:00:00 (xfer#2, to-check=0/3)
sent 4.72M bytes received 70 bytes 219.53K bytes/sec
total size is 4.72M speedup is 1.00

Como vemos la conexión a Rsync se hace sobre SSH, es una copia por tanto segura, es más, se nos está pidiendo autenticación en dos pasos y la copia se deja bajo el usuario operador:

[root@Jarvis CopiaRemota]# pwd
/home/operador/CopiaRemota
[root@Jarvis CopiaRemota]# ls -l

total 4616
-rwxrwxrwx 1 operador operadores 4047332 ene 25 2012 A5_Octavia_OwnersManual.pdf
-rwxrwxrwx 1 operador operadores 671786 ene 25 2012 A5_Octavia_Swing_CarRadio.pdf

Bien, como habíamos pensado hacer ese proceso que nos haga copia de una serie de carpetas de nuestro OSX sobre la Rpi, y luego nos apague la máquina OSX, pues no podemos estar pendientes para poder introducir el famoso código de autenticación en dos pasos ni la contraseña del usuario, vamos a hacer lo que se llama una conexión transparente mediante generando para esto nuestra huella RSA propia.

En nuestro equipo OSX, abrimos un terminal y nos vamos a nuestro home de usuario (recordar, la virgulilla), y dentro del home al directorio .ssh

cd ~/.ssh
ssh-keygen -t rsa

Generating public/private rsa key pair.
Enter file in which to save the key (/Users/admin/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/admin/.ssh/id_rsa.
Your public key has been saved in /Users/admin/.ssh/id_rsa.pub.

The key fingerprint is:
f4:86:f3:50:3b:13:e9:e2:bf:2b:02:0e:a6:b8:3e:f3 admin@Hal9000.local

The key's randomart image is:
+--[ RSA 2048]----+
|                 |
| .               |
| . +             |
| . = o           |
| S B             |
| o . . * o       |
|.o o . . .       |
|oo . . ..        |
|oo+E . .+o       |
+-----------------+

Respondemos a todas las preguntas que nos hace este comando con INTRO.

Ahora tenemos que copiar el fichero id_rsa.pub que hemos generado en nuestro OSX a la máquina RPI, y debe quedar en el directorio /home/operador/.ssh, se debe llamar authorized_keys y los permisos del directorio y del fichero deben excluir a grupo y otros por que de lo contrario todo esto se desactiva por temas relativos a la seguridad.

Para esta copia, podemos usar este comando en nuestro terminal OSX:

Hal9000:.ssh admin$ scp id_rsa.pub operador@192.168.1.20:id_rsa.pub

Verification code:
Password:
id_rsa.pub

Como vemos es una copia con SSL, autenticación en dos pasos, clave, etc.

El fichero se ha quedado en el home del usuario operador en la PI, así que estos son los siguientes pasos a dar:

[operador@Jarvis ~]$ mkdir -p .ssh
[operador@Jarvis ~]$ mv ./id_rsa.pub ./.ssh/authorized_keys
[operador@Jarvis ~]$ chmod go-w . .ssh .ssh/authorized_keys

Que pasa a partir de este punto, pues que las conexiones por SSH entre ese equipo que ha generado su clave RSA y se la ha enviado a nuestra RPI, y la RPI ya no precisan de usuario/clave ni código de verificación en dos pasos.

Y ahora, por ejemplo, si repetimos en nuestro OSX el comando de copia rsync sobre la RPi ya no nos pide nada y lo podemos ejecutar en modo desatendido:

Hal9000: ~$ rsync --progress -avhe ssh /Users/admin/Documents/Vehiculos/Octavia/Manuales/ operador@192.168.1.20:CopiaRemota/
building file list ...
3 files to consider

sent 161 bytes received 20 bytes 362.00 bytes/sec
total size is 4.72M speedup is 26072.48

## Script de copia y apagado del MAC

Y ahora, juntando todas estas piezas que hemos ido viendo podremos implementar lo siguiente, nuestra idea inicial, poder pulsar un “botón” y que se haga copia de lo que quiero y luego se apague el equipo.

Para esto, necesitamos hacer una simple tarea en nuestro OSx con Automator.

Se trata de abrir automator y crear un nuevo flujo de trabajo.

En primer lugar, el flujo ejecutará una secuencia de Shell Scrip, en esta secuencia es donde pegamos nuestra línea de Rsync con la que hacíamos copia de la carpeta.

El siguiente elemento de ese flujo es la ejecución de un script hecho en AppleScript, este ejecuta el comando tell application "Finder" to shut down.

Y listo, ahora guardamos este flujo de Automator como aplicación y la grabamos donde queramos, en al carpeta de aplicaciones, en nuestra carpeta personal, da igual, cuando ejecutemos nos hará en primer lugar la copia por rsync y al finalizar apaga el equipo al completo.
