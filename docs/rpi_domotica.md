---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 11) – práctico 5 - Domótica"
date: "2020-02-08"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
---

## Pequeña central domótica

Aunque la máquina por hardware es pequeña, no lo es más que otras centralitas de domótica que actualmente se encuentran en el mercado.

Por ejemplo una Vera Lite del fabricante Mi Casa Verde apenas tiene 100 Mb de memoria y está basada es una placa que más que un “ordenador” es un router al que se le han añadido algunas funcionalidad más, de hecho el fabricante anuncia como que corre con Linux 2.6 (el último kernel 2.6 que se lanzó es de octubre de 2010… muy al día tampoco están).

En el caso de los Home Center de Fibaro, el modelo Lite está basado al igual que nuestras Rpi en arquitectural ARM, en concreto un Cortext A8, que es un procesador ligeramente superior al que incorporan las Rpi.

Con las RPI también tenemos que tener en cuenta las posibilidades de ampliación a través del puerto GPIO, y la expansión hacia sistemas bien consolidados como son los Arduinos.

En este sentido, en la “domotización” de los elementos de la vida cotidiana, como puede por ejemplo un simple acuario, Arduido barre, haced la prueba y buscar en google acuario + arduino y ya veréis lo que sale, auténticos sistemas domóticos para el control hasta del más mínimo detalle del sistema.

Estos sistemas basados en Arduino están empezando a pasarse a Raspberry, dejando la "lógica" del proceso en manos de la RPI y el trabajo más "mecánico" o "eléctrico" en manos de las placas Arduino.

En este sentido tenemos una placa, que se incorpora a nuestra Rpi en el puerto GPIO llamada [RaZberry](http://razberry.z-wave.me).

Esta placa en su última versión de software, la 1.5 permite incluso la ejecución de código Java a través de sus APIs para la gestión de la red y componentes Z-wave que tengamos establecida.

Pero primero, antes de ir a esto que es algo más complejo, vamos a probar con cosas más sencillas

## Gestión de cámaras IP

Podemos gestionar una o varias cámaras IP que tengamos en nuestra red mediante el uso de llamadas http.

Los códigos Http a emplear, por lo menos en el caso de las cámaras Foscam, es relativamente estándar, los códigos los vais a encontrar en el documento que os adjunto a este mensaje.

Vamos a hacer un pequeño script que nos maneje una o varias cámaras, nos modifique el punto o preset en el que la cámara está situado, tome una instantánea de ese punto, vuelva al origen y nos mande todas esas fotos a nuestro correo electrónico.

Esto no es por capricho, a mi me surgía la necesidad de poder saber si mi cámara estaba operativa en casa, si estaba bien enfocada donde le había dicho, y como estaba el resto de la sala, no podía cada X tiempo andar entrado en aplicaciones como CamViewer para mirar con el consumo de datos tan grande que tiene esa aplicación.

Así que me implementé este proceso. Creo que a través de un NAS y la Surveillance Station se puede hacer algo similar, pero en cuanto que hay más de una cámara hay que andar pagando licencias, en este caso es gratis…

Lo primero, para poder recibir correos electrónicos en nuestro móvil por ejemplo, es instalar [Mutt](http://www.mutt.org) , que es un cliente de correo, que nos permite en modo texto ó línea de comandos enviar correos interactuando con el smtp, incluso con documentos adjuntos.

```
[root@Jarvis ~]# pacman -S mutt

resolviendo dependencias...
verificando conflictos...
Paquetes (2): mime-types-9-1 mutt-1.5.23-1
Tamaño Total de Descarga: 1,15 MiB
Tamaño Total Instalado: 6,40 MiB
:: ¿Continuar con la instalación? [S/n] S
:: Recuperando paquetes ...
mime-types-9-1-any 14,6 KiB 176K/s 00:00        [######################################################] 100%
mutt-1.5.23-1-armv6h 1166,8 KiB 1321K/s 00:01   [######################################################] 100%
(2/2) verificando llaves en el llavero          [######################################################] 100%
(2/2) verificando la integridad de los paquetes [######################################################] 100%
(2/2) cargando los archivos del paquete...      [######################################################] 100%
(2/2) verificando conflictos entre archivos     [######################################################] 100%
(2/2) verificando el espacio disponible en disco[######################################################] 100%
(1/2) instalando mime-types                     [######################################################] 100%
(2/2) instalando mutt                           [######################################################] 100%
==> For GPG support, add the following to your muttrc:
==> source /etc/Muttrc.gpg.dist

Dependencias opcionales para mutt
smtp-forwarder: to send mail
```

Ahora vamos a configurarlo, la configuración se hace a nivel de usuario, y básicamente lo que vamos a configurar es el uso de un servicio smtp, y nada más.

Si tenemos una cuenta de correo en Gmail por ejemplo, de las que usan autenticación en dos pasos recordad que la contraseña del servicio smtp debemos generarla para poder usarla, ya que de otra manera nada de nada, nos dará error de conexión todo el tiempo.

Así pues, para configurar editamos el fichero `~/.muttrc`

```
[root@Jarvis ~]# vi .muttrc
set from = "antonio.hernan@gmail.com"
set realname = "Antonio J. Hernan"
set imap_user = "antonio.hernan@gmail.com"
set imap_pass = "mcabbahyqm"
set folder = "imaps://imap.gmail.com:993"
set spoolfile = "+INBOX"
set postponed ="+[Gmail]/Drafts"
set header_cache =~/.mutt/cache/headers
set message_cachedir =~/.mutt/cache/bodies
set certificate_file =~/.mutt/certificates
set smtp_url = "smtp://antonio.hernan@smtp.gmail.com:587/"
set smtp_pass = "mcadahyqm"
set move = no
set imap_keepalive = 900
```
Si ejecutamos el comando:
```
[root@Jarvis ~]# mutt -s "Prueba Jarvis” antonio.hernan@gmail.com
```
Entramos en el "cliente en modo texto" de envío de mail, y el resultado final es que nos llegará un mail a nuestra cuenta.

Bien pues ahora que podemos enviar correos fácilmente por la línea de comandos, vamos con el script de vigilancia, que como veréis es muy sencillo pero que podemos ir complicando todo lo que queremos según vayamos incorporando nuevas cámaras o queramos establecer más puntos de control sobre las cámaras actuales.
```
[root@Jarvis ~]# cat patrol.sh
#!/usr/bin/bash

# Mandamos a Preset 1, esperamos y tomamos imagen.
wget -O - http://192.168.1.27:6159/decoder_control.cgi?command=31\\&user=operador\\&pwd=CLAVE
sleep 10
wget -O /root/preset1.jpg http://192.168.1.27:6159/snapshot.cgi?resolution=32\\&user=operador\\&pwd=CLAVE

# Mandamos a Preset 2, esperamos y tomamos imagen.
wget -O - http://192.168.1.27:6159/decoder_control.cgi?command=33\\&user=operador\\&pwd=CLAVE
sleep 10
wget -O /root/preset2.jpg http://192.168.1.27:6159/snapshot.cgi?resolution=32\\&user=operador\\&pwd=CLAVE

# Mandamos a Preset 3, esperamos y tomamos imagen.
wget -O - http://192.168.1.27:6159/decoder_control.cgi?command=35\\&user=operador\\&pwd=CLAVE
sleep 10
wget -O /root/preset3.jpg http://192.168.1.27:6159/snapshot.cgi?resolution=32\\&user=operador\\&pwd=CLAVE

# Mandamos a Preset 1, sin esperar
wget -O - http://192.168.1.27:6159/decoder_control.cgi?command=31\\&user=operador\\&pwd=CLAVE

# Componemos el envio de correo con las imagenes tomadas
echo "Envio Camara Foscam" > /root/patrol.txt
echo "Fecha: " \`date '+%d/%m/%Y %H:%M:%S'\` >> /root/patrol.txt
echo "" >> /root/patrol.txt
mutt -s "Foscam" antonio.hernan@gmail.com < /root/patrol.txt -a /root/preset*.jpg
rm -f /root/patrol.txt > /dev/null 2>&1
rm -f /root/preset*.jpg > /dev/null 2>&1
```
Y ahora tan sólo nos falta dar permisos de ejecución a dicho script
``` 
[root@Jarvis ~]# chmod 700 patrol.sh
```
Y si queremos que se ejecute siguiendo alguna condición de hora, programarlo con el correspondiente `crontab -e`.

Como veis es muy sencillo, el comando 31 lleva al preset 1, el 33 al preset 2, el 35 al preset 3, etc.

Todo está explicado en el manual que os adjunto.

Hay mucho más comandos que se pueden emplear, como los que activa y desactiva en sensor de movimiento, el sensor de voz, la luz infrarroja, la grabación sobre ftp y un largo etc, esto que os he puesto tan sólo es un esqueleto básico.

## RaZberry y módulos Z-Wave

RaZberry es una placa hija para conectar en nuestras RaspBerryPI, en su puerto GPIO.

Una vez conectado y configurados nos permite crear nuestra red mallada con nuestros dispositivos Z-Wave.

![](images/RaZberry-300x282.jpg)

Una vez que tengamos funcionando nuestra Rpi+RaZberry podremos establecer comunicación con los módulos a través de sus APIs y Cgis, de manera que será muy similar a lo que hemos visto hace un momento con la cámara IP, o podremos instalar algunos de los productos que hoy en día están en desarrollo para tener una central de domótica algo más completa, como son Z-Way y todo lo que se está moviendo a través de OpenRemote.

Para poder en marcha esta RazBerry, por desgracia nuestra, todo lo que el fabricante ha puesto a disposición de su público está basado en Debian, por tanto si no tenemos una RaspBerryPi que corra con raspbmc, raspbian y openelec no podremos usar los instaladores que nos da el fabricante y tendremos que hacerlo mediante este método que os cuento a continuación.

1.- Crear los directorios donde vamos a instalar y el fichero de testigo de la versión

[root@Jarvis ~]# mkdir -p /opt/z-way-server
[root@Jarvis ~]# mkdir -p /etc/z-way
[root@Jarvis ~]# echo "v1.5.0" > /etc/z-way/VERSION

2.- Instalamos las librerías de Yajl (otro gestor más Json, vamos intercambio de datos entre procesos basados en ficheros…)

[root@Jarvis ~]# pacman -S yajl

resolviendo dependencias...
verificando conflictos...
Paquetes (1): yajl-2.0.4-2
Tamaño Total de Descarga: 0,03 MiB
Tamaño Total Instalado: 0,18 MiB
:: ¿Continuar con la instalación? [S/n] S
:: Recuperando paquetes ...
yajl-2.0.4-2-armv6h 30,6 KiB 205K/s 00:00        [######################################################] 100%
(1/1) verificando llaves en el llavero           [######################################################] 100%
(1/1) verificando la integridad de los paquetes  [######################################################] 100%
(1/1) cargando los archivos del paquete...       [######################################################] 100%
(1/1) verificando conflictos entre archivos      [######################################################] 100%
(1/1) verificando el espacio disponible en disco [######################################################] 100%
(1/1) instalando yajl                            [######################################################] 100%

3.- Descargamos los drivers y programas para RaZberry/ZWay

[root@Jarvis ~]# cd ~Descargas
[root@Jarvis Descargas]# wget http://razberry.z-wave.me/z-way-server-RaspberryPiXTools-v1.5.0.tgz
--2014-03-30 18:24:25-- http://razberry.z-wave.me/z-way-server-RaspberryPiXTools-v1.5.0.tgz
Resolviendo razberry.z-wave.me (razberry.z-wave.me)... 46.20.244.36
Conectando con razberry.z-wave.me (razberry.z-wave.me)[46.20.244.36]:80... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 8552999 (8,2M) [application/x-gzip]
Grabando a: “z-way-server-RaspberryPiXTools-v1.5.0.tgz”
100%[=================================================================================================================>]
8.552.999 2,63MB/s en 3,1s
2014-03-30 18:24:28 (2,63 MB/s) - “z-way-server-RaspberryPiXTools-v1.5.0.tgz” guardado [8552999/8552999]

4.- Descompresión de los ficheros

[root@Jarvis Descargas]# cd /opt
[root@Jarvis opt]# tar -xvzf ~/Descargas/z-way-server-RaspberryPiXTools-v1.5.0.tgz

5.- Ahora vamos a “engañar” a Z-Way por que espera una versión un tanto antiguo de una librería y le vamos a engañar con la nueva…

[root@Jarvis ~]# cd /usr/lib
[root@Jarvis lib]# ln -s /usr/lib/libarchive.so.13.1.2 /usr/lib/libarchive.so.12

Con esto, omitimos el error que nos daría en el arranque, algo así como:

/opt/z-way-server/z-way-server: error while loading shared libraries: libarchive.so.12: cannot open shared object file:
No such file or directory

6.- Eliminamos la carga de ttyAMA0

Debemos editar el fichero de arranque del sistema /boot/cmdline.txt, importante tener prevista una copia de seguridad por si lo hacemos mal, el sistema es posible que no arranque.

Lo que vamos a hacer es evitar que el sistema operativo trate de hacer uso de la UART, esto es, trate de activar para su uso propio el puerto GPIO, [aquí](http://elinux.org/RPi_Serial_Connection#Preventing_Linux_using_the_serial_port) os cuentan con más detalle lo que es esto.

Tenemos que eliminar de este fichero el texto que dice:

console=ttyAMA0,115200 kgdboc=ttyAMA0,115200

El fichero quedaría tal que así una vez modificado:

[operador@Jarvis boot]$ cat cmdline.txt
ipv6.disable=1 avoid_safe_mode=1 selinux=0 plymouth.enable=0 smsc95xx.turbo_mode=N dwc_otg.lpm_enable=0 console=tty1
root=/dev/mmcblk0p5 rootfstype=ext4 elevator=noop rootwait

[operador@Jarvis boot]$

7.- Preparamos el proceso de arranque y parada de ZWay.

Como está pensado para distribuciones Debian, los scripts de arranque y parada de todo esto están bajo el init de SystemV, deberíamos transformar esto para que puedan operar bajo SystemD que es lo que emplea ArchLinux.

En primer lugar modificamos el proceso de arranque/parada manual, de manera que desde el home del usuario root podamos arrancar y parar el proceso a petición:

Editamos el fichero (nuevo) /root/Z-Way

[root@Jarvis ~]# vi ~/Z-Way
#!/bin/sh

PATH=/bin:/usr/bin:/sbin:/usr/sbin
NAME=z-way-server
DAEMON_PATH=/opt/z-way-server
PIDFILE=/run/$NAME.pid
LOGFILE=/var/log/$NAME.log

# adding z-way libs to library path
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/z-way-server/libs
case "$1" in
 start)
  echo -n "Starting Z-Way: "
  cd $DAEMON_PATH  
  nohup $DAEMON_PATH/z-way-server & > /dev/null 2>&1
  echo "done."
  cd ~
  ps -e | grep 'z-way-server' | awk '{print $1}' > $PIDFILE
  ;;
 stop)
  echo -n "Stopping Z-Way: "
  kill -9 \`cat $PIDFILE\`
  rm $PIDFILE
  echo "done."
  ;;
 *)
  echo "Usage: ~/Z-Way {start|stop}"
  exit 1
  ;;
esac
exit 0

Ahora vamos a crearnos una unidad para carga en SystemD que simulará lo que hace un tiempo teníamos como los rc.local, que era un script donde el usuario podía libremente meter todo lo que quisiese que se ejecutase al finalizar el arranque de la máquina.

Esto nos pude venir muy bien para otras cosas… ;)

Editamos el fichero `/usr/lib/systemd/system/rc-local.service` y le ponemos este contenido:

[root@Jarvis ~]# vi /usr/lib/systemd/system/rc-local.service

[Unit]
Description=/etc/rc.local Compatibility
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/etc/rc.local start
ExecStop=/etc/rc.local stop

[Install]
WantedBy=multi-user.target

Creamos el fichero `/etc/rc.local` , para poner la ejecución del arranque de Z-Way

[root@Jarvis ~]# vi /etc/rc.local

#!/usr/bin/bash
/root/Z-Way $1 >> /var/log/local.log 2>&1

Y damos permisos de ejecución a los scripts creados, así como habilitar el arranque del servicio rc.local.

[root@Jarvis ~]# chmod a+x /etc/rc.local
[root@Jarvis ~]# chmod a+x /usr/lib/systemd/system/rc-local.service
[root@Jarvis ~]# systemctl enable rc-local
ln -s '/usr/lib/systemd/system/rc-local.service' '/etc/systemd/system/multi-user.target.wants/rc-local.service'

Y si reincidamos el equipo veremos como Z-Way se carga al arrancar a través de estos scripts, tenemos acceso al servidor web, tenemos en /var/log los ficheros de lo correspondientes, etc.

## Jugando un poco con Z-Wave

*Nota del 2020... las capturas de imágenes ya no están disponibles

Si, digo jugando por que no me ha dado tiempo a hacer mucho más.

Tan sólo tengo un módulo prestado, un Wall Plug de Fibaro con el que he podido hacer alguna prueba y ver hasta donde podemos llegar.

Una vez iniciado el sistema podemos entrar en la URL de administración de ese Z-Way ([http://192.168.1.20:8083/expert](http://192.168.1.20:8083/expert)).

Aquí veremos que tenemos un dispositivo funcionando, es nuestra RazBerry, y yendo a la pestaña de red, y aquí en el elemento administración de red podemos incluir nuevos módulos.

Yo como os decía he probado con un Wall Plug de Fibaro, y no conectaba ni a tiros, y parece ser que todo es por lo mismo… el módulo estaba dado de alta en otra red y hasta que no lo resetee al completo no enlazó (en el caso de este módulo, pulsar el botón de enlace hasta que el anillo de color cambia a amarillo, soltar el botón y hacer un click rápido, el anillo se pone rojo y ya está, como salió de fábrica).

Una vez respetado, el enlace es casi instantáneo y se ve que se detecta perfectamente.

(http://i58.tinypic.com/16bwun7.png)

Este módulo lo podemos usar como Switch (enciede/apaga) y como Sensor/Medidor (consumo).

Así que si entramos a las opciones de dispositivo tipo Interruptor vemos esto:

(http://i62.tinypic.com/2wd0wfs.png)

Y si entramos a las opciones de dispositivo tipo medidores veremos el resto de la información que nos aporta:

(http://i60.tinypic.com/2s6rn0l.png)

Para los que estén acostumbrados o hayan visto la interfaz de usuario de otros productos como los de Vera Lite, o los nuevos Home Center de Fibaro… pues este “UI” es una castaña de mucho cuidado, pero quizás podamos enfocar todo esto desde otro punto de vista, como hemos hecho con las cámaras y pensar en montar nuestro propio sistema domótico en “scracth”, tirando nosotros el código por que al fin y al cabo no es más que una API a la que le puedes hacer peticiones según el tipo de módulo, y esperar respuesta de esas peticiones (igual que hacemos con las cámaras).

Aquí (http://docs.zwayhomeautomation.apiary.io) encontraréis ejemplos de esa API por si alguien se anima.

Yo no descarto controlar las dos o tres cosas que inicialmente pienso domotizar con un sistema como este, y luego ya veremos si doy el salto a una centralita más potente, que parece que es el camino a seguir.
