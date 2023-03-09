---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 5) – Copias de Seguridad"
date: "2020-02-03"
---

## Copias de seguridad

En este punto tenemos nuestra máquina lista y preparada para empezar a meter contenido en ella.

Instalar programas, activar funcionalidades, en definitiva, para empezar el cacharreo, así que toca pensar en **tener como mínimo un punto de restauración** para no tener que volver a realizar todos los trabajos que hemos hecho hasta ahora.

Además de todo esto, la copia de seguridad nos va a aportar un plus de tranquilidad, y ya no tendremos tanto miedo por tocar este o aquel fichero de configuración y llevarnos la sorpresa de que nuestra máquina no se comporta como queríamos **o que simplemente no arranca**.

Os cuento como hago yo las copias de seguridad, y como siempre digo, esto es sólo una parte, y como yo lo hago, tomadlo como una base si queréis pero no como un dogma.

Por desgracia para nosotros los Maqueros, aquí no disponemos de nada similar al fantástico [TimeMachine](https://web.archive.org/web/20090615053605/http://www.apple.com/es/macosx/what-is-macosx/time-machine.html), así que debemos implementarlo todo con otros programas y mecanismos (programación de tareas), además las copias completas serán algo más manuales y a máquina parada.

Distinguiremos dos tipos de copias de seguridad, las copias en **frío**, o copias completas, donde **la máquina está parada** y no se realizan por tanto modificaciones en su contenido, es más, para estas copias deberemos retirar la tarjeta SD de la máquina.

Y las copias en **caliente**, que realizaremos con **la máquina en funcionamiento**, aunque en ocasiones podamos llegar a sufrir algo de degradación en el rendimiento de nuestra máquina por en consumo de disco/cpu de la operación de copia. Estas copias en caliente **serán complemento de las anteriores**.

Explicado un poco mejor, partimos de una copia en frío, la cual no es más que un espejo o snapshot de la actual información de nuestra tarjeta SD, y que se realiza con no mucha asiduidad, ya que implica la parada de la máquina.

Tenemos por otro lado que a diario, o incluso varias veces a lo largo del dÌa, se realizan copias en caliente, las cuales además son incrementales, esto es, se copia todo y luego se copia únicamente lo que se modifica (realmente se copian los bloques de los ficheros que cambian, es la llamada [tecnología delta](http://es.wikipedia.org/wiki/Delta_encoding)

Para restaurar nuestra máquina en el último momento antes de un error debemos restaurar la tarjeta con la última copia de seguridad completa en frío que tengamos y luego añadirle la información del incremental de que disponemos.

## Copia completa SD (copia en frío)

No se si recordáis la utilidad que os comenté en el momento de pasar la imagen del sistema operativo a la SD, [Pi Filler](http://ivanx.com/raspberrypi/) , como os dije ese fabricante tiene otra utilidad llamada [Pi Copier](http://ivanx.com/raspberrypi/) que hace la operación inversa, pasar el contenido de la tarjeta a un fichero en nuestro disco duro, incluso hace una compresión del fichero de volcado si así se lo indicamos.

Ambos programas, Pi Filler y Pi Copier están **hechos en AppleScript y son fácilmente editables**, para que veáis si queréis lo que realmente hacen.

El uso de Pi Copier es muy sencillo pues sólo hay que seleccionar nuestra tarjeta SD e indicar un fichero de destino, marcando si queremos que Además la copia sea posteriormente comprimida y listo,a esperar el resultado.

Es posible que queramos hacerlo manualmente, total es hacer la operación inversa a la de carga de la SD con la imagen original.

Para esto, volveremos a usar el comando dd cambiando como es lógico los parámetros de entrada y salida de la información.

Este sería el comando para volcar nuestra tarjeta SD en nuestra carpeta Descargas:

``` 
sudo dd bs=1m if=/dev/rdisk1 | gzip > ~/downloads/2014-04-26_Jarvis.img.gz
```

El parámetro `if` indica que nuestro origen será el disco (completo, no particiones) que corresponde a nuestra tarjeta SD (en mi caso es este, `/dev/rdisk1` pues monto con un USB porta SD, recordad como vimos como localizar nuestro dispositivo en el mensaje de instalación de la SD).

Y ¿el parámetro de salida?, no está, se omite, por que la salida del comando, salida estándar o a pantalla se está enviando a un comando mediante el uso de ese `|`, esto es, la salida estándar del comando `dd` se está empleando como entrada estándar del comando `gzip` ( es la versión [GNU](http://es.wikipedia.org/wiki/GNU) de nuestro conocido Zip), y la salida generada por gzip es enviada a un fichero mediante el uso de `>` el cual se encuentra en la carpeta de Descargas (Downloads, recordad que el terminal del sistema operativo lo ve **todo en inglés**...) de la carpeta HOME del usuario, la cual viene indicada por el uso de nuestra amiga la virgulilla `~`.

Importante, el tamaño del fichero de destino, en caso de no comprimirlo es el mismo que el tamaño de la tarjeta SD, esté esta llena o no, si nuestra tarjeta es de 32Gb nuestra imagen en disco será de esos 32Gb, pues la copia es literal, así pues, imprescindible comprimirla a no ser que dispongamos de Terabytes libres para regalar.

## Copias incrementales (copia en caliente)

Bien, pues ahora tenemos un punto de partida para poder restaurar nuestra máquina, a través de esas imágenes completas de tarjeta. Estas copias, por lo que tardan y por que debemos detener la máquina no las haremos con mucha frecuencia, yo en mi caso particular las hago una vez al mes... **mes arriba mes abajo** XD.

Para las copias en caliente yo empleo dos métodos, uso [Rsync](http://es.wikipedia.org/wiki/Rsync) y [Dump](http://en.wikipedia.org/wiki/Dump_(program)) casi indistintamente, si bien rsync es bastante más frecuente y me fuerzo a usarlo bastante más regularmente pues es posible hacerlo incluso volcando sobre equipos que estén en nuestra red (p.e. un NAS)

Os voy a contar como opero con Rsync, para tratar de no hacer este tutorial demasiado extenso y dejamos por ahora el tema de dump para futuras revisiones de este texto. Además, como veremos un poco más adelante, el uso de rsync nos va a venir muy bien para utilizar nuestra máquina como centro de respaldo de nuestros equipos Apple, algo así como una **pequeña Time Capsule**.

Necesitamos un pendrive de al menos 4Gb (fácil y barato hoy en día), el pendrive lo conectamos a uno de los dos puertos USB y será nuestra unidad de respaldo.

Este unidad la vamos a **montar de manera fija**, por tanto no dependeremos de la posibilidad de automontaje que luego más adelante veremos.

En primer lugar, después de conectar el pendrive a la máquina, debemos identificar que dispositivo se corresponde con nuestro pendrive:

Con el siguiente comando vemos como el dispositivo está conectado a la máquina y que es identificado por esta (en mi caso un lector de tarjeta Micro-SD):

```
[root@Jarvis log]# lsusb

Bus 001 Device 004: ID 1058:1103 Western Digital Technologies, Inc. My Book Studio
Bus 001 Device 005: ID 05e3:0715 Genesys Logic, Inc. USB 2.0 microSD Reader
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9512 Standard Microsystems Corp. SMC9512/9514 USB Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Y la mejor manera de ver cual es el dispositivo de nuestro pendrive es guiarnos por la salida del comando fdisk y por las capacidades de los "discos":

```
[root@Jarvis ~]# fdisk -l

Disk /dev/sdb: 3.7 GiB, 3965190144 bytes, 7744512 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000
Device Boot Start End Blocks Id System
/dev/sdb1 8192 7744511 3868160 b W95 FAT32
```

Nuestro pendrive es la unidad `/dev/sdb` y la partición es la 1 (única partición por otro lado), está formateado sistema de ficheros Fat32, así que vamos a hacer lo propio que es crear unas particiones más acordes con el sistema operativo con el que estamos tratando y a formatear estas particiones.

En concreto crearemos dos, una la vamos a emplear ahora en este paso de copias de seguridad y la otra partición la utilizaremos en el siguiente mensaje...

Es posible usar el “disco” en Fat32, y así podremos compartir este pendrive entre sistemas, pero esto lo veremos en otra ocasión ya que necesitaríamos modificar las opciones de montaje del dispositivo (no vale emplear el defaults en fstab) y podemos tener algunos problemas de permisos, por no decir que tendremos la limitación de tamaño máximo de archivo que tiene Fat32 respecto a ext4.

```
[root@Jarvis ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.24.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Orden (m para obtener ayuda): p
Disk /dev/sdb: 7,5 GiB, 8017412096 bytes, 15659008 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xc3072e18
Disposit. Inicio Start Final Blocks Id System
/dev/sdb1 2048 15659007 7828480 83 Linux
Orden (m para obtener ayuda): d
Selected partition 1
Partition 1 has been deleted.

Orden (m para obtener ayuda): n
Partition type:
p primary (0 primary, 0 extended, 4 free)
e extended

Select (default p): p
Número de partición (1-4, default 1): 1
First sector (2048-15659007, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-15659007, default 15659007): +3G
Created a new partition 1 of type 'Linux' and of size 3 GiB.

Orden (m para obtener ayuda): n
Partition type:
p primary (1 primary, 0 extended, 3 free)
e extended

Select (default p): p
Número de partición (2-4, default 2): 2
First sector (6293504-15659007, default 6293504):
Last sector, +sectors or +size{K,M,G,T,P} (6293504-15659007, default 15659007):
Created a new partition 2 of type 'Linux' and of size 4,5 GiB.

Orden (m para obtener ayuda): p
Disk /dev/sdb: 7,5 GiB, 8017412096 bytes, 15659008 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xc3072e18
Disposit. Inicio Start Final Blocks Id System
/dev/sdb1 2048 6293503 3145728 83 Linux
/dev/sdb2 6293504 15659007 4682752 83 Linux

Orden (m para obtener ayuda): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Ahora creamos el FileSystem en esa partición de disco (de tipo [ext4](http://es.wikipedia.org/wiki/Ext4) ), y el directorio donde montaremos el disco:

```
[root@Jarvis ~]# mkfs -t ext4 /dev/sdb1

mke2fs 1.42.8 (20-Jun-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
242400 inodes, 967808 blocks
48390 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=994050048
30 block groups
32768 blocks per group, 32768 fragments per group
8080 inodes per group
Superblock backups stored on blocks:
32768, 98304, 163840, 229376, 294912, 819200, 884736
Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

[root@Jarvis /]# mkdir /BCK
```

Y por último, configuramos el punto de montaje, para lo que debemos editar nuestro fichero `/etc/fstab` (con VI por supuesto) y añadir la siguiente línea:

```
/dev/sdb1 /BCK ext4 defaults 0 0
```

Para añadir esa línea, después de haber tecleado `vi /etc/fstab` pulsamos las teclas SHIFT + G (que nos lleva al final del fichero) y una vez allí pulsamos la tecla o (o minúscula) para añadir una nueva línea a continuación de la actual(última en nuestro caso) y así entramos en modo edición, aquí podemos teclear la información o usar copiar y pegar de la que os he puesto si es que el nombre del dispositivo coincide.

En caso de no realizar un nuevo particionamiento y aprovechar lo que tenemos en el pendrive (p.e. Fat32), la línea que debemos crear en `/etc/fstab` es la siguiente:

```
/dev/sdb1 /BCK vfat user,rw,umask=111,dmask=000 0 0
```

En los siguientes arranques de la máquina ese punto de montaje ya estará presente, y si queremos activarlo ahora, sin tener que reiniciar recurriremos al comando mount el cual nos montará los recursos pendientes de montaje tal y como aparecen en nuestro `/etc/fstab`

```
[root@Jarvis /]# mount -a
[root@Jarvis /]# df -kh

Filesystem Size Used Avail Use% Mounted on
/dev/sdb1 3.6G 7.4M 3.4G 1% /BCK
```

Con la infraestructura necesaria para estas copias incrementales ya operativa vamos a ver los comandos y scripts necesarios para hacerla funcionar.

Lo primero es instalar rsync que por defecto no está incluido en la instalación base, para esto:

```
sudo pacman -S rsync
```

El comando para hacer una copia completa de toda nuestra tarjeta sobre el directorio `/BCK/Copias` del pendrive es este:

```
rsync -aAXv /* /BCK --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/media/*,/lost+found,/BCK/*}
```

Los parámetros empleados, -aAX son para que la copia sea lo más literal posible en cuanto a permisos, ACLs, etc, así como para que el copiado se comporte de manera recursiva.

En las exclusiones ponemos todos los directorios de los que cuelga el sistema “en vuelo”, esto es, en `/proc` por ejemplo tenemos los procesos que están corriendo, en `/tmp` los temporales que se pierden tras cada reinicio, etc.

Y por supuesto, debemos excluir la propia carpeta de copias, que en este caso está montada bajo `/` y sería tomada en cuenta por el `/*`.

Si el comando finaliza bien podemos incorporarlo en un fichero de shell script y programarlo mediante [crontab](http://es.wikipedia.org/wiki/Cron_(Unix)) para su ejecución.

Para esto, con el usuario root ejecutamos:
```
mkdir -p /home/operador/scripts
mkdir -p /home/operador/logs
chown operador:operadores /home/operador/scripts
vi /home/operador/scripts/CopiaSeg.sh
```

El primer comando,`mkdir -p` crea el directorio scripts, el modificador -p lo que indica es que se cree toda la estructura de directorios que haga falta.

El segundo comando,`chown` es de Change Owner, esto es, cambiar de propietario, el directorio que hemos creado es propiedad del usuario root, lo cambiamos para que sea propiedad del usuario operador y del grupo operadores.

El contenido del fichero que estamos editando con VI será este:
```
#!/bin/bash

rsync -aAXv /* /BCK --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/media/*,/lost+found,/BCK/*} > /dev/null 2>/home/operador/logs/CopiaSeg.log

ESC + :wq!
```

Le damos permisos de ejecución al script con el comando chmod:
```
chmod u+x /home/operador/scripts/CopiaSeg.sh
```

Y por último editamos el crontab del usuario root para ponerlo que se ejecute a diario, todos los días de la semana a las 23:00 horas.
```
crontab -e

0 23 * * * /home/operador/scripts/CopiaSeg.sh
```

Esto es **una aproximación** digamos que “decente” de un sistema de copias de seguridad, menos es nada, y por lo menos desde estas copias podréis volver a poner el sistema en marcha.

En cuando que pueda pondré en la sección “Anexos” de este mensaje algún script algo más elaborado respecto a las copias de seguridad, para poder tener copias por semanas, manteniendo al menos un mes, etc… vamos, algo un poco mejor que todo esto.
