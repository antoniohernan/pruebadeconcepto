---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 12) – práctico 6 - Media Center"
date: "2020-02-08"
---

## Media Center - XBMC

Este uso sea quizás el más extendido para las Rpi.

Es raro el blog/foro en el que no se cuenta como montar un servidor de medios empleando [XBMC](http://es.wikipedia.org/wiki/XBMC)

Así que, aquí no vamos a ser menos y os cuento las cuatro cosas que hay que hacer para tener el media center funcionando.

De cara a la infraestructura necesaria para poder hacer funcionar Rpi con XBMC, pues la Pi, la SD y un disco duro conectado por USB es más que suficiente.

Yo trabajo con un disco de 1Tb y por ahora no tengo problemas ni de espacio ni de rendimiento, claro que yo borro cosas, no soy ningún Diógenes digital.

Vamos a instalar entonces..

## Instalación paquete XMBC

La instalación es… según algunos blogs superdifícil y le tienes que dar opciones y paquetes adicionales y las opciones de la cpu y… es un comando de una línea y tres palabras…

\[root@Jarvis ~\]# pacman -S xbmc
:: Existen 2 proveedores disponibles para xbmc:
:: Repositorio alarm

1) xbmc-rbp 2) xbmc-rbp-git
Introduzca un número (por defecto=1): 1
resolviendo dependencias...
verificando conflictos...
Paquetes (49): afpfs-ng-0.8.1-8 alsa-lib-1.0.27.2-1 bluez-libs-5.16-1 compositeproto-0.4.2-2 dmxproto-2.3.1-2 enca-1.15-1 flac-1.3.0-1 fribidi-0.19.6-1 inputproto-2.3-1 libao-1.2.0-1 libass-0.11.1-1 libbluray-0.5.0-1 libcddb-1.3.2-4 libcdio-0.92-1 libcec-2.1.4-1.1 libdmx-1.1.3-1 libmad-0.15.1b-7 libmicrohttpd-0.9.33-1 libmodplug-0.8.8.5-1 libmpeg2-0.5.1-4 libnfs-1.9.3-1 libogg-1.3.1-2 libplist-1.10-1 libsamplerate-0.1.8-3 libshairport-1.2.1.20121215-3 libsndfile-1.0.25-2 libssh-0.5.5-3 libva-1.3.0-1 libvorbis-1.3.4-1 libxcomposite-0.4.4-1 libxi-1.7.2-1 libxinerama-1.1.3-2 libxtst-1.2.2-1 libxxf86dga-1.1.4-1 lockdev-1.0.3\_1.5-4 raspberrypi-firmware-20140324-1 raspberrypifirmware-examples-20140324-1 recode-3.6-8 recordproto-1.14.2-1 rtmpdump-20131205-1 sdl-1.2.15-5 sdl\_image-1.2.12-3 swig-3.0.0-1 taglib-1.9.1-1 tinyxml-2.6.2-3 xf86dgaproto-2.1-2 xineramaproto-1.2.1-2 xorg-xdpyinfo-1.3.1-1 xbmc-rbp-12.3-1

Tamaño Total de Descarga: 59,24 MiB
Tamaño Total Instalado: 118,71 MiB

Y ya… le decimos a pacman que nos instale el paquete xbmc y ya está, no tiene más, el valida las dependencias e instala lo que falte.

Como veis, hay dos paquetes xbmc, que son 1) xbmc-rbp y 2) xbmc-rbp-git, **el primero es el paquete ESTABLE**, el probado y liberado en condiciones para nuestra plataforma, el otro, el GIT es el paquete en desarrollo y como tal podemos dar con BETAS muy avanzadas (como la Beta 3 de Gotham) y con **otras que sean un desastre**.

Así que, a no ser que sufras de insomnio por las noches pensando en esta o aquella nueva funcionalidad que con la Beta podrías estar disfrutando… yo pondría la estable… en tu mano lo dejo.

Otra curiosidad, si miráis la lista de paquetes de los que depende xbmc, tenemos en primer lugar [afpfs-ng](http://sourceforge.net/projects/afpfs-ng/), que es un protocolo para poder conectarnos a una unidad Mac OSX exportada por [AFP](http://es.wikipedia.org/wiki/Apple_Filing_Protocol)  sobre nuestra red TCP.

Termina la instalación y esto es importante, nos aparece este mensaje:

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

If xbmc systemd service does not start, try adding a udev rule:
echo 'SUBSYSTEM=="vchiq",GROUP="video",MODE="0660"' > /etc/udev/rules.d/10-vchiq-permissions.rules

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Dependencias opcionales para xbmc-rbp

lirc: remote controller support
udisks: automount external drives
upower: used to trigger power management functionality
unrar: access compressed files without unpacking them

La primera parte nos dice que si nos encontramos problemas para arrancar con systemd, que ejecutemos ese comando (ese echo...)

Pues mira, como que no hay que esperar a tener problemas para arrancar para ejecutar eso, que sólo modifica permisos…

\[root@Jarvis ~\]# echo 'SUBSYSTEM=="vchiq",GROUP="video",MODE="0660"' > /etc/udev/rules.d/10-vchiq-permissions.rules

Las otras dependencias opcionales que nos marca, pues a gusto del consumidor… yo no he tenido que instalarlas para hacer funcionar mi XBMC.

## Configuración de CPU

Aunque parezca un poco increíble a nuestra placa le podemos alterar ligeramente los parámetros de frecuencia del procesador y alguna que otra cosilla.

Con esto ¿conseguiremos que vaya todo mejor, más fluido y pueda correr más cosas a la vez?, pues no.

Va a ir un poco más ligera, puede que no se enganche tanto, pero de ahí a que se note… ya os digo.. no…

Estos cambios los podemos hacer editando el fichero `config.txt` que está en el directorio /boot, ojo con tocar ahí que es el arranque de la máquina…

\[root@Jarvis ~\]# vi /boot/config.txt

En la parte final del fichero encontráis grupos de configuración según la frecuencia del procesador.

Por defecto, este procesador arm v6 funciona a 700Mhz, el límite teórico estaría en 1000Mhz (1Ghz), que por lo visto degrada bastante el componente y le acorta bastante su vida útil.

Así que, nuevamente, a gusto del consumidor, eso sí, si optas por las frecuencias más altas no estaría de más que comprases un kit de disipadores pasivos para refrigerar un poco esa placa, por lo que pueda pasar.

Si os sirve de guía, yo uso la recomendación Medium, y para esto, tan sólo tenemos que eliminar el carácter # del principio de la línea, moviéndonos con los cursores en VI y pulsando la tecla x (x minúscula) cuando estemos sobre ello.

##Medium

#arm\_freq=900
#core\_freq=333
#sdram\_freq=450
#over\_voltage=2

Quedaría así

##Medium

arm\_freq=900
core\_freq=333
sdram\_freq=450
over\_voltage=2

Del resto de parámetros, he visto ajuste de memoria de la gpu (cuanta ram le damos a la gráfica, así en palabras más llanas), pero a mi no me han supuesto gran cambio.

Reiniciamos la máquina después de estos cambios.

\[root@Jarvis ~\]# sync; reboot

Y ahora, cuando haya terminado vamos a arrancar XBMC de manera manual para luego fijarlo y dejarlo en arranque automático.

\[root@Jarvis ~\]# systemctl start xbmc
\[root@Jarvis ~\]# systemctl enable xbmc

Si todo sale bien veremos en nuestra televisión por HDMI el logotipo y posteriormente el menú de nuestro Media Center.

Y una última cosa, para la unidad USB a conectar, por ejemplo, en mi caso un pendrive USB, tenemos que saber que dispositivo es, que tipo de partición y debemos montarlo.

Para un pendrive con formato Fat32 en el dispositivo`/dev/sdb` la línea a incluir en `/etc/fstab` es la siguiente:

/dev/sdb1 /media vfat defaults 0 0

Y una vez ejecutado el comando de montaje tendríamos el contenido del Pendrive:

\[root@Jarvis /\]# mount -a
\[root@Jarvis /\]# cd media
\[root@Jarvis media\]# ls

Documentales Peliculas Series

## Multimedia y Configuración Scrapers

Para manejar el Media Center, se puede usar el mando de la televisión si esta es más o menos moderna y dispone de HDMI CEC, pero lo mejor, con diferencia… instalar la aplicación disponible para iOS, la de iPad es una maravilla, se manera todo perfectamente y sincroniza datos con la Rpi en un instante, aportándonos información acerca de lo que estamos viendo procedente de los repositorios de películas/series/documentales en la red, en múltiples idiomas.

Antes de poder usar esta aplicación para manejar nuestro equipo, debemos habilitar el acceso por servicio web que viene desactivado, son 2 minutos.

\[root@Jarvis ~\]# vi /var/lib/xbmc/.xbmc/userdata/guisettings.xml

<webserver>false</webserver>

Por

<webserver>true</webserver>

Salvamos el fichero y reincidamos XBMC

\[root@Jarvis ~\]# systemctl restart xbmc

Los ajustes de configuración de la App para iOS (en este caso para iPhone) son muy sencillos, y una vez que conecte, el icono cambie a verde en vez de rojo, podremos movernos por los menús libremente usado la opción de control remoto.

Ahora vamos a añadir una fuente de contenido multimedia de tipo película, basada en un directorio de nuestra unidad externa.

(\*Nota en 2020... las capturas de pantalla, se han perdido, obvio, es lo que pasa con los enlaces efímeros en los foros.)

En el menú principal nos vamos a Videos, y pulsando flecha abajo entramos a Files.

(http://i59.tinypic.com/r8tfdt.jpg)

En esta nueva página bajamos hasta donde dice Add Videos…

(http://i59.tinypic.com/2w3z7r9.jpg)

En esta pantalla vamos a elegir el directorio a partir del cual se van a buscar los elementos, pulsamos flecha a la derecha hasta que estamos sobre el botón de Browse, pulsamos botón central y se nos abre un navegador de carpetas.

Tendremos que ir hasta la carpeta donde están nuestras películas (/media/Peliculas en mi caso), así que pulsamos sobre Root Filesystem, luego buscamos media y luego buscamos Peliculas, aquí pulsamos botón central y para poder aceptar y que se nos active el botón de OK otra vez tendremos que pulsar la flecha hacia la izquierda.

(http://i62.tinypic.com/24qnxg7.jpg)

Ahora, bajamos con las flechas hasta que se nos vuelva a activar el botón de OK y pulsamos en el centro.

En esta nueva ventana es donde se configura el llamado Scraper, que se escoge en función del contenido para que se nos complete la información del “medio” con la que existe en los diferentes repositorios.

Con las flechitas de la pantalla, pulsando botón central iremos pasando entre los scrapers disponibles hasta que nos aparezca themoviedb.org.

Una vez que nos sale, no tenemos que tocar nada más en esta pantalla, tan sólo ir con las flechas (las del remote, lo mejor es ponerte sobre The Movie Database y pulsar botón central, y luego flecha abajo) hasta el botón que dice Settings.

(http://i57.tinypic.com/28b8nsy.jpg)

Y en la pantalla de setting hay que bajar hasta los valores de lenguaje, que fijaremos en ES, así como el posible bloqueo por calificación de edad, que marcaremos también como ES.

(http://i59.tinypic.com/2ufudfo.jpg)

Finalizamos con OK, que nos devuelve a la pantalla anterior, botón a la derecha y de nuevo OK.

Nos aparece una pantalla en la que se nos pregunta si queremos actualizar con al información del Path, y le decimos que si (yes) y nos aparecerán nuestras películas ya en condiciones. Por si acaso, si no os reconoce alguna vez una película, revisad que el nombre del fichero está bien escrito, siempre es eso…

(http://i62.tinypic.com/b9hqp4.jpg)

En la app de iOs, si entramos en “Movies” y refrescamos (desplazar hacia abajo) veremos como esta información también nos aparece.

Ahora vamos con el Scraper de Series y también para documentales.

El alta del scrapper es el mismo, añadir vídeos, elegir ruta, elegir el scrapper, que es lo que cambia, y el resto todo igual, el idioma.. etc.

Este es el scrapper para las series y documentales.

En mi caso, he añadido dos carpetas a este scraper /media/Series y /media/Documentales.

Esta es la pantalla de elección del scrapper para series.

(http://i61.tinypic.com/50nlud.jpg)

Estos son los settings de ese scrapper

(http://i59.tinypic.com/f3dw0i.jpg)

Y este es el resultado final

(http://i58.tinypic.com/x0obic.jpg)

## Configuración AirPlay

La configuración es la más sencilla de todas.

En el menú principal vamos a sistema SYSTEM.

En este menú vamos a servicios SERVICE.

Bajamos hasta AirPlay, pulsamos botón a la derecha y se nos ilumina “Allow XBMC to receive AirPlay content”, pulsamos con el botón central, se ilumina en azul la “bolita” y listo, podemos salir al menú principal con el botón de salir/volver de nuestro mando remoto.

Tan sólo tendremos que reiniciar para que se activen los cambios.

Para usar Airplay, pues por ejemplo, desde el iPhone / iPad en la App de fotos, pulsamos sobre el botón de compartir (el cuadradito con la flecha hacia arriba) y aquí escogemos “AirPlay”, y nos saldrá un listado de máquinas que hacen de servidores de AirPlay, entre ellas esta nuestra.

Y en el caso del vídeo es igual, compartir por airplay y seleccionar nuestra máquina, no debería dar ningún problema, ni con video ni con audio.

## Plug-INS

Sobre plugins para XBMC se puede hablar mucho, bueno, no tanto, hay cosas que no se pueden decir en público e imagino que todos sabéis de “que” y el “porqué”.

Os cuento como instalar un plugin bastante interesante, el de [búsqueda automática de subtítulos](http://wiki.xbmc.org/index.php?title=Add-on:XBMC_Subtitles) .

El fichero a descargar es [este](http://www.mediafire.com/?bb3u85ogfdvqetu) , y una vez descargado el zip en nuestro equipo, lo pasamos a la RPI, el método a utilizar el que queramos, podemos usar Samba que más atrás hemos configurado, una conexión Sftp, la cuestión es hacer llegar ese fichero zip.

Hal9000:Downloads admin$ sftp operador@192.168.1.20
Connected to 192.168.1.20.
sftp> put script.xbmc.subtitles.zip
Uploading script.xbmc.subtitles.zip to /home/operador/script.xbmc.subtitles.zip
script.xbmc.subtitles.zip 100% 1519KB 1.5MB/s
00:01
sftp> bye

Ahora, con nuestra app de gestión de XBMC vamos a “Remote Control” y nos movemos con las flechas por el menú hasta que aparece SYSTEM, pulsamos el centro y en el nuevo menú que se nos abre bajamos hasta Add-ons.

Pulsamos de nuevo en el centro, y en la nueva pantalla escogemos Install from zip file.

Termina sin decirnos gran cosa… parece que incluso no ha ido bien por que no haya hecho nada pero si, está ahí.

Entramos en la opción (seguimos dentro de System -> Add-ons) Enable Add-ons.

Buscamos nuestro “Subtitles” y entramos.

Nuevamente pulsamos para entrar a configurar este add-on.

(http://i58.tinypic.com/2rrxg28.jpg)

Y ahora si, por fin lo vamos a configurar, entramos en “Configure” y ya podemos escoger los idiomas por orden, salimos como “Spanish”.

(http://i57.tinypic.com/2mmxnr4.jpg)

Terminado esto, botón a la derecha para marchar los servicios de subtítulos que emplearemos, yo tengo puestos subtitulos.es, subdivx.es y openSubtitles.

Volvemos a pulsar a la derecha para ir a la tercera pestaña de opciones, donde podemos da preferencia a un motor de búsqueda de subtítulos en concreto para películas y para series.

(http://i58.tinypic.com/2h4f728.jpg)

Y ya está, salimos pulsando OK y a partir de ese momento, cuando estemos reproduciendo contenido, si pulsamos sobre la opción de subtítulos veremos si están disponibles y el PlugIn directamente lo descarga.

(http://i57.tinypic.com/x52sdk.jpg)

Y aquí podremos escoger entre todos los subtítulos disponibles cual es que mejor se adapta a nuestras necesidades

(http://i61.tinypic.com/30m62kh.jpg)

Y por último, así se verá

(http://i61.tinypic.com/3521ana.jpg)

Listo, no tiene mucho más.

Tampoco os rompáis mucho la cabeza con este plug-in, por que para la versión 13 de XBMC cambia totalmente la gestión de los subtítulos y con ello este plug-in.
