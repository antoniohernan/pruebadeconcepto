---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 3) – Instalación básica (II)"
date: "2019-12-23"
---

## … Continuamos con la personalización

Y ya, para terminar de personalizar nuestra máquina vamos a cambiarle el nombre, ¿alarmpi? a quien se le ese nombre, con la de cosas bonitas y frikis que hay para usar, toda la saga de ESDLA está llena de nombres máquinas, el mundo del comic, de las novelas de Sci-Fi, o puedes usar los socorridos nombres de familiares mascotas... **el nombrar o bautizar máquinas es un arte** ;)

```
[root@alarmpi etc]# vi /etc/hostame
Jarvis
```

Con esto editamos el fichero donde está el nombre de nuestra máquina. Sobre el editor VI os podría contar mucho, pero para este caso os doy un pequeño guión y si os pica la dudéis consultar esta [página](http://www.lagmonster.org/docs/vi.html) o esta [otra](http://www.viemu.com/a_vi_vim_graphical_cheat_sheet_tutorial.html). Después de pulsar intro al final del comando `vi /etc/hostname` tendreis el editor abierto con una línea que dice `_alarmpi_`. Borramos esta línea, pulsando dos veces la tecla d `dd`, la línea desaparece. Ahora insertamos el nuevo texto, pulsmos la tecla i `i`, y teclamos el nombre de nuestra máquina. Termina la edición volvemos al modo de comando del editor pulsando la tecla ESC. Y ejecutamos el comando del editor que nos salva el fichero y finaliza la edición `:wq!`

Como hemos cambiado el nombre de la máquina, debemos cambiar también la configuración del fichero de direcciones locales, de manera que cuando hagamos referencia a nuestro nuevo nombre de máquina, el sistema sea capaz de encontrarse a si mismo.

```
[root@alarmpi etc]# vi hosts
#
# /etc/hosts: static lookup table for host names
#
#<ip-address> <hostname.domain.org> <hostname>
127.0.0.1 localhost.localdomain localhost Jarvis
::1 localhost.localdomain localhost Jarvis
# End of file
```

Nuevamente recurrimos a VI. Nos desplazamos con los cursores hasta el espacio que hay detrás de localhost, vamos a borrar todo lo que existe en esa línea desde ese punto hasta el final. Para eso, la combinación de teclas es `SHIFT + d`. Y ahora añadimos al final de la línea, el comando que nos habilita la edición al final de línea es `SHIFT + a`. Estando en modo edicion tecleamos el nombre nuevo de nuestra máquina y para volver a modo de comando pulsamos ESC. Nos vamos a la otra línea y repetimos la misma operación. Y para salvar, como os he dicho antes, ESC y luego `:wq!`

Un reinicio de la máquina para que se refresquen estos cambios y **ahora entramos en palabras mayores**, que vamos a tocar la **tabla de particiones de la tarjeta**.

```
[root@alarmpi ~]# sync
[root@alarmpi ~]# reboot
```

## Particionamiento y redimensionamiento de SD

Por defecto, las imágenes de las distribuciones (no se si con NOOB es igual) vienen para tarjetas de 4Gb, nosotros es muy posible que tengamos una tarjeta más grande, 8, 16 o 32Gb. Vamos a cambiar la tabla de particiones para que se pueda aprovechar todo es espacio, y de paso vamos a hacer una cosa muy importante que es crear una **zona de Swap**. Para el particionamiento yo soy muy maniático, me gusta crear cada file system crítico en su lugar, para evitar que te crezcan descontroladamente si no los creas así, como file system y los dejas como directorios. Por ejemplo, si creas un FS (file system) para el raiz de la máquina `/` y en este existe el directorio `/var/log` (que es donde suelen ir todos los logs del sistema) no sabrás de una manera rápida y clara si el contenido de ese directorio crece de manera inusual, tampoco tendrás posibilidades de maniobrar ampliando si se da el caso. En cambio, si creas un FS para `/` y otro FS para `/var/log` ambos puntos de montaje son independientes y puedes ampliar llegado el caso el de log y controlarlo de un simple vistazo. De todas maneras, todo esto es mucho, pero que mucho más manejable si en vez de hacerlo así empleamos LVM (Logical Volumen Manager), el cual nos permite agrupar el almacenamiento y crear volúmenes lógicos, si veo que no me sale una guía excesivamente grande no descarto contar algo de LVM de cara sobre todo al almacenamiento externo. Para este caso y casi todo lo que voy a contar en la guia, con un particionamiento más o menos rudimenatario puede valer, esto es, dando todo el espacio que nos sea posible a la raiz del sistema (`/`) y generando una pequeña área para la zona de swap que ahora después os comentaré. La modificación de la tabla de particiones **es una tarea crítica, un error y no arrancáis**, así que si has pensado en hacer un clonado de tu SD, este es el momento, recordad aquel programa, Pi Copier que os comenté.

Vamos a particionar, para esto, conectados a nuestra Pi por SSH con el usuario root utilizamos el comando `fdisk`(Fixed Disk Setup), y tenemos que indicarle a este programa cual es el dispositivo que apunta a nuestra tarjeta SD. Con el comando `df -kh` que os indiqué antes podréis ver en que dispositivo están montadas las particiones y sus FileSystem, ya os lo digo yo, es **/dev/mmcblk0** Todo lo que os pongo a continuación es específico de un ARCH Linux, de una imagen reciente (a partir de Septiembre de 2013).

```
fdisk /dev/mmcblk0
Welcome to fdisk (util-linux 2.23.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help):
```

¿Que? ¿intuitivo verdad?... En primer lugar, listamos las particiones actuales, para esto, el comando es **p** (de print).

```
Disk /dev/mmcblk0: 31.7 GB, 31674335232 bytes, 61863936 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00057540
Device Boot Start End Blocks Id System
/dev/mmcblk0p1 2048 186367 92160 c W95 FAT32 (LBA)
/dev/mmcblk0p2 186368 3667967 1740800 5 Extended
/dev/mmcblk0p5 188416 3667967 1739776 83 Linux
``` 
Hay que fijarse, importante, el punto de inicio de las particiones (Start). Hay tres particiones, la **p1** que es donde está la partición de arranque o boot del sistema. La **p2** que es extendida y la **p5** que es una partición lógica y está contenida dentro de **p2**. Empezamos por eliminar **p2** (y por tanto p5).
```
Command (m for help): d
Partition number (1,2,5, default 5): 2
Partition 2 is deleted
```
Con el comando **d** (delete) le decimos que queremos borrar una partición y luego le damos el número de partición. Si listamos las particiones que nos quedan ahora, el resultado es este:

Command (m for help): p
Disk /dev/mmcblk0: 31.7 GB, 31674335232 bytes, 61863936 sectors
Units = sectors of 1 \* 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00057540
Device Boot Start End Blocks Id System
/dev/mmcblk0p1 2048 186367 92160 c W95 FAT32 (LBA)

Nos hemos quedado con la partición de arranque, pero tranquilos, que no hemos perdido nada de nuestra tarjeta (todavía!! XD). Añadimos de nuevo la partición 2, extendida, dándole más espacio en la tarjeta pero con el mismo punto de inicio, no hace falta que nos acordemos de cual, el sistema nos va a decir cual es ese punto, importante, dejad un poco de hueco para un área de Swap que en breve os voy a contar.

Command (m for help): n
Partition type:
p primary (1 primary, 0 extended, 3 free)
e extended
Select (default p): e
Partition number (2-4, default 2): 2
First sector (186368-61863935, default 186368):
Using default value 186368
Last sector, +sectors or +size{K,M,G} (186368-7698431, default 7698431): +28G
Partition 2 of type Linux and of size 28 GiB is set

Pulsamos comando **n** (new) e indicamos que es una partición de tipo **e** (extendida), el número que vamos a usar es el 2 y el número del sector de inicio que nos marca es el 186368, que es el mismo que teníamos en el primero listado con el comando **p** (este dato puede variar en tu caso), y ahora le indicamos que a partir de ese sector nos cree una partición de 28Gb con el +28G. Ahora vamos a crear una nueva partición, la **5**, de tipo lógico, lo hacemos así:

Command (m for help): n
Partition type:
p primary (1 primary, 1 extended, 2 free)
l logical (numbered from 5)
Select (default p): l
Adding logical partition 5
First sector (188416-61863935, default 188416):
Using default value 188416
Last sector, +sectors or +size{K,M,G} (188416-61863935, default 61863935): +28G
Partition 5 of type Linux and of size 28 GiB is set

Una vez más, vemos como el sector de inicio coincide con el que teníamos en el primer listado (esto es realmente lo importante, que eso no cambie). Hemos puesto el comando n para una nueva partición, el tipo indicado es l de logical, y el tamaño el mismo que hemos dado para la extendida (una partición extendida puede contener varias lógicas). Si queremos ver si tenemos todo bien, hasta ahora, volvemos a imprimir la tabla actual de particiones:

Command (m for help): p
Disk /dev/mmcblk0: 31.7 GB, 31674335232 bytes, 61863936 sectors
Units = sectors of 1 \* 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00057540
Device Boot Start End Blocks Id System
/dev/mmcblk0p1 2048 186367 92160 c W95 FAT32 (LBA)
/dev/mmcblk0p2 186368 61863935 30838784 5 Extended
/dev/mmcblk0p5 188416 61863935 30837760 83 Linux

¿Ya hemos terminado?, pues no. Tenemos 29Gb más o menos particionados en nuestra tarjeta, vamos a **crear la partición para Swap**, que veremos más adelante como manejar, y así dejamos esto listo para el reinicio.

Command (m for help): n
Partition type:
p primary (2 primary, 0 extended, 2 free)
e extended
Select (default p): p
Partition number (1-4, default 3): 3
First sector (6477824-7698431, default 6477824):
Using default value 6477824
Last sector, +sectors or +size{K,M,G} (6477824-7698431, default 7698431):
Using default value 7698431
Partition 3 of type Linux and of size 1.1 GiB is set
Command (m for help): t
Partition number (1-4): 3
Hex code (type L to list codes): 82
Changed system type of partition 3 to 82 (Linux swap / Solaris)

Empezamos con el comando **n** para añadir una nueva partición de tipo **p** (primary), y el número de partición a usar es el 3. No tenemos que preocuparnos por sectores de inicio y sectores de fin, el sistema nos ofrece el primer sector libre como sector de inicio, y el último sector disponible como sector de fin, dándonos así todo el espacio que queda disponible en la tarjeta. Y luego damos el comando **t** (type), porque por defecto el tipo es el **83**, una partición Linux, pero nosotros queremos que sea del tipo **82 Linux Swap**, así que después de indicar el número de partición, la 3, ponemos ese código, el 82 y ya está.

Bien, tarjeta particionada, solo tenemos que hacer dos cosas, salvar la nueva tabla de particiones y reiniciar (y como tercera cosa, cruzad los dedos). Para salvar:

Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.

Y ahora reiniciamos (recordad, cruzad los dedos, es fundamental) ....

\[root@Jarvis ~\]# sync
\[root@Jarvis ~\]# reboot

Cuando el sistema vuelva a ofrecernos conexión (y podamos volver a respirar.. XD) tendremos la nueva particiones cargada, pero los File System ocupan lo mismo, así que **debemos redimensionarlos para que se extiendan** hasta ocupar todo el espacio disponible en las particiones, esto es así:

\[root@Jarvis ~\]$ sudo resize2fs /dev/mmcblk0p5
resize2fs 1.42.7 (21-Jan-2013)
Filesystem at /dev/mmcblk0p5 is mounted on /; on-line resizing required
old\_desc\_blocks = 1, new\_desc\_blocks = 2
The filesystem on /dev/mmcblk0p5 is now 7709440 blocks long.

El comando `resize2fs`si no le indicamos ningún parámetro adicional extiende el File System hasta la capacidad disponible en el dispositivo, en nuestro caso /dev/mmcblk0p5, siendo p5 la partición 5, esto es, nuestra partición lógica.

Este comando también se puede usar para extender parcialmente el FS y dejar así algo de espacio no disponible a modo de reserva, y también se puede hacer para reducir el espacio disponible de un FS, siempre que no esté en uso claro. Ya está, nuestra SD está siendo utilizada al máximo.

## Área de Swap

¿Que es esto el área de Swap? Pues sencillo, así, a _grosso modo_ es un área de intercambio. ¿Como el archivo de paginación de Windows (arggg)?, si, como eso, pero BIEN hecho. La gestión de la memoria física (RAM) se hace mediante bloques, llamados páginas. Cuando el sistema necesita liberar páginas de memoria física las vuelca sobre esta memoria virtual. Por lógica, el acceso a la memoria física es mucho más rápido que a la virtual, estamos hablando de accesos a placas de memoria que se miden en nonasegundos respecto a accesos a discos duros que se miden en milisengundos. Pero también es cierto que el almacenamiento en disco es mucho más barato que la ram física, por no hablar de que la mayoría de las máquinas tienen una cantidad de memoria RAM con la que pueden trabajar, si necesitamos más o bien cambiamos de máquina o bien usamos Swap. Hay una expresión muy común en esto de la informática que es, la máquina está paginando, esto viene a indicar que la máquina está casi sin memoria física y empieza a volcar en la swap todo lo que puede para evitar quedarse tiesa, durante ese tiempo sigue dando servicio pero se ve claramente degradado. El encontrar ese equilibro entre la cantidad de RAM física, la cantidad de Swap a usar, y el momento en el que paginares uno de los "unicornios" en la optimización de servidores...

Para nuestro caso, la rPi, modelo B tiene 512 Mb de RAM, yo le voy a dar 1Gb adicional para Swap. Posteriormente veremos como podemos hacer ciertos ajustes en el comportamiento de esta Swap que vamos a crear para precisamente definir eso que comentaba. Hace un momento os he dicho como crear una partición en nuestra SD específica de Swap, vamos a darle uso.

\[root@Jarvis ~\]# mkswap /dev/mmcblk0p3
Setting up swapspace version 1, size = 610300 KiB
no label, UUID=97bae743-01b0-4307-baae-fc41b40bc4c6

Con ese comando `mkswap`(make swap) creamos un área de swap en el dispositivo que le indiquemos, en este caso la partición p3 de nuestra SD que es la que hemos habilitado para ello. Ahora debemos configurar el sistema para que en cada arranque este área de swap se ponga en uso:

\[root@Jarvis ~\]# vi /etc/fstab
#
# /etc/fstab: static file system information
#
# <file system> <dir> <type> <options> <dump> <pass>
/dev/mmcblk0p1 /boot vfat defaults 0 0
/dev/mmcblk0p3 none swap defaults 0 0

Editamos con nuestro nuevo amigo, el editor Vi el fichero fstab (file system table), y añadimos una línea al final (pulsando `SHIFT + G`para ir al final, y una vez allí pulsando **i** de insertar).

La línea indica el dispositivo, el punto de montaje (que en el caso de swap no aplica), el [tipo a usar](https://wiki.archlinux.org/index.php/Fstab_(Espa%C3%B1ol)#Definiciones_de_los_campos) y luego las opciones por defecto, default y 0 0. Sobre las opciones, usamos defaults. Estas opciones si miráis en enlace que os he puesto, veréis de todo, incluso una **opción discard muy apropiada para los discos SSD** tan de moda últimamente. En el caso de un FS basado en ext4 sería similar a poner las opciones rw, suid, dev, exec, auto, nouser y async. Y los 0 0 del final, el primero es para dump, los posibles valores son 0 o 1, en caso de 1 se hace dump de este punto de montaje, si es que dump está instalado, que no es más que una herramienta que nos permite hacer volcados a un fichero de un punto de montaje, luego lo vemos en el apartado de backup.

El segundo 0 es para el proceso fsck o File System Check, los posibles valores son 0,1 y 2. El valor 0 es que no se tiene en cuenta esta FS en caso de hacer un chequeo automático, los valores 1 y 2 si se tiene en cuenta y especificas la prioridad de validación, siendo siempre 1 para el punto de arranque o boot que debe tener prioridad sobre el resto, que en caso de que queramos chequearlos tendrán que llevar un 2. Activamos la swap, con el siguiente comando `[root@Jarvis ~]# swapon -a`

Esto le dice al sistema que mire en el fichero /etc/fstab y ponga en marcha todos los dispositivos de tipo swap. Para ver que se está usando, pues utilizamos el mismo comando pero con este otro parámetro:

\[root@Jarvis ~\]# swapon -s
Filename Type Size Used Priority
/dev/mmcblk0p3 partition 610300 0 -1

Y si queremos asegurarnos que está activa, las primeras líneas del comando top nos dirán la cantidad de swap disponible:

KiB Mem: 473124 total, 48240 used, 424884 free, 7280 buffers
KiB Swap: 610300 total, 0 used, 610300 free, 20072 cached

Más cosas sobre la swap, **podemos modificar su comportamiento**, en el fichero /etc/sysctl.conf:

\# Swapping too much or not enough? Disks spinning up when you'd
# rather they didn't? Tweak these.
#vm.vfs\_cache\_pressure = 100
#vm.laptop\_mode = 0
#vm.swappiness = 60

Modificando el parámetro **swappiness** le indicamos al sistema su tendencia usar o no la swap, contra más cerca del 99 más paginará en este área.

## ¿Y si quiero usar mi rPI por Wifi?

Por defecto en ARCH no tenemos soporte para los adaptares WIFI, en otras distribuciones si que bien integrado pero en esta debemos instalarlo y configurarlo, no es muy difícil, estos son los pasos. En primer lugar instalamos el paquete necesario:

\[root@Jarvis ~\]# pacman -S wireless\_tools
resolving dependencies...
looking for inter-conflicts...
Packages (1): wireless\_tools-29-8
Total Download Size: 0.08 MiB
Total Installed Size: 0.24 MiB
:: Proceed with installation? \[Y/n\] Y
:: Retrieving packages ...
wireless\_tools-29-8-armv6h 82.4 KiB 231K/s 00:00
\[#############################################\] 100%
(1/1) checking keys in keyring
\[#############################################\] 100%
(1/1) checking package integrity
\[#############################################\] 100%
(1/1) loading package files
\[#############################################\] 100%
(1/1) checking for file conflicts
\[#############################################\] 100%
(1/1) checking available disk space
\[#############################################\] 100%
(1/1) installing wireless\_tools
\[#############################################\] 100%

Ahora conectamos nuestro adaptador WIFI al puerto USB libre y reinicarmos el sistema para que se carguen los drivers del mismo `[root@Jarvis ~]# sync; reboot`

Una vez que ha arrancado debemos mirar si nuestro adaptador USB aparece listado, para esto:

\[root@Jarvis ~\]# lsusb
Bus 001 Device 004: ID 148f:5370 Ralink Technology, Corp. RT5370 Wireless Adapter
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9512 Standard Microsystems Corp. SMC9512/9514 USB Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

En el bus 001, el dispositivo 4 se correspondea nuestro adaptador Wifi. Y ahora lo vamos a configurar, para después activarlo. El comando de configuración es este, y os aparece un menu realtivamente intuitivo `[root@Jarvis ~]# wifi-menu`

En este menú debemos ver nuestra BSSID listada, esto es, nuestra red Wifi, podremos introducir un nombre que le vamos a dar a la conexión y teclear la clave de la Wifi, ojo, que esta clave se muestra en plano en pantalla, no aparecen los \* para taparla. Ahora vamos a comprobar que se ha activado bien el interfaz de red. El comando con el que manejamos los interfaces de red es ifconfig, con el parámetro -a que nos los lista todos. En la salida de este comando deberemos localizar algo así:

\[root@Jarvis ~\]# ifconfig -a
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 192.168.1.20 netmask 255.255.255.0 broadcast 192.168.1.255
ether b8:27:eb:30:f0:8f txqueuelen 1000 (Ethernet)
RX packets 440 bytes 37247 (36.3 KiB)
RX errors 0 dropped 1 overruns 0 frame 0
TX packets 264 bytes 43962 (42.9 KiB)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
...
wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 192.168.1.23 netmask 255.255.255.0 broadcast 192.168.1.255
ether 00:87:32:91:02:39 txqueuelen 1000 (Ethernet)
RX packets 7 bytes 1346 (1.3 KiB)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 26 bytes 5325 (5.2 KiB)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

Ese eth0 se corresponde con nuestra tarjeta de red por Ethernet, y el wlan0 se corresponde con nuestro adaptador Wifi. En ambos casos vemos las Ips asigandas a esas direcciones MAC por nuestro DHCP en el router. Para esta configuración de wlan0 se ha generado un fichero de perfil en la máquina. El fichero de perfil está guardado aquí: `/etc/netctl/wlan0-JAZZTEL_B1B9`

Tenemos que hacer dos cosas, la primera es activarlo para ver que todo marcha bien:

\[root@Jarvis netctl\]# netctl start wlan0-JAZZTEL\_B1B9

Si ejecutamos un `ifconfig -a`a continuación veremos como nuestro interfaz de red wlan0 ya tiene dirección IP asignada:

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 192.168.1.23 netmask 255.255.255.0 broadcast 192.168.1.255

Y ahora lo dejamos habilitado para que se cargue en el arranque de máquina.

\[root@Jarvis netctl\]# netctl enable wlan0-JAZZTEL\_B1B9
ln -s '/etc/systemd/system/netctl@wlan0\\x2dJAZZTEL\_B1B9.service' '/etc/systemd/system/multiuser.
target.wants/netctl@wlan0\\x2dJAZZTEL\_B1B9.service'

Con esto, en los **sucesivos arranques se activará el dispositivo Wifi**. Y listo, ya tenemos nuestra Pi para poder empezar a darle caña.
