---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 13) – práctico 7 - Emulador de Videojuegos"
date: "2020-02-08"
---

## Emulación de Juegos

Pues sí, se trata de convertir nuestra RPI en una especie de moderna/vieja videoconsola o maquina recreativa.

No vamos a obtener el rendimiento de las consolas actuales ni mucho menos, pero si que tenemos potencia para poder hacer correr todo este llamado “AbandonWare”, que son los juegos de los finales de los 80 y principio de los 90 que a muchos nos entretuvieron durante un buen puñado de horas.

Podemos hacerlo de dos maneras, montando los distintos emuladores uno a uno… en algunos casos hay incluso que compilar y entras en una espiral de dependencias/compilaciones que te hace perder mucho tiempo para lo que obtienes… y la otra forma es usando (sin que sirva de precedente) una imagen que ya lo trae todo y es “Ready tu Play”, esta distribución es [PiMame](http://pimame.org)

(*Nota 2020 .. las capturas de este post.. también están perdidas)

## Scumm VM en ARCH para juegos tipo “Lucas”

La [SCUMM](http://es.wikipedia.org/wiki/SCUMM) es una especie de motor gráfico y lenguaje de programación que se creó en 1985 para el juego [Maniac Mansion](http://es.wikipedia.org/wiki/Maniac_Mansion) .

Desde entonces los juegos de LucasFilms Games lo adoptan como “Framework” de desarrollo.

Vamos a instalar una máquina virtual para SCUMM, llamada SCUMMVM, la cual es multiplataforma y corre prácticamente en cualquier hardware, de hecho es una premisa de todo informático de bien, tratar de instalar la ScummVM y hacer correr el Monkey Island en todo aquello que pase por sus manos y tenga una pantalla.

Así que instalamos, como veis, ya existe como paquete “oficial” en los repositorios de ARCH, lo que confirma mis palabras anteriores…

[root@Jarvos ~]# pacman -S scummvm

resolviendo dependencias...
verificando conflictos...
Paquetes (9): faad2-2.7-4 fluidsynth-1.1.6-2 glu-9.0.0-2 jack-0.124.1-1 json-c-0.11-1 libasyncns-0.8-5 libpulse-5.0-1 libtheora-1.1.1-3 scummvm-1.6.0-2

Tamaño Total de Descarga: 7,31 MiB
Tamaño Total Instalado: 27,55 MiB
:: ¿Continuar con la instalación? [S/n]
(6/9) instalando fluidsynth             [########################################################################] 100%

> To use FluidSynth as a daemon copy the service file from:
/usr/lib/systemd/system/fluidsynth.service

> to:
/etc/systemd/system/multi-user.target.wants/

> and then edit accordingly.
> PulseAudio output when running as a daemon is known to be
> problematic. See the following bulletin board post:
https://bbs.archlinux.org/viewtopic.php?id=135092

Estos mensajes tan “feos” que vemos durante la instalación son por la instalación de un sintetizador, y que bueno, es claro, te indica que si quieres tener la posibilidad de arrancarlo y pararlo como el resto del sistema pues que te lo hagas tu.

Pero la realidad es que está todo hecho y con un entable y un start a través de systemctl solucionado, y así lo hacemos:

[root@Jarvis ~]# systemctl enable fluidsynth
ln -s '/usr/lib/systemd/system/fluidsynth.service' '/etc/systemd/system/multi-user.target.wants/fluidsynth.service'

[root@Jarvis ~]# systemctl start fluidsynth

## Cargando Monkey Island

Vamos con un clásico, Monkey Island.

Yo este juego lo tengo original, pero por desgracia a día de hoy imposible jugarlo por que era para MsDOS, está en disquetes y por que hace años que perdí la ruleta de los códigos de bloqueo (si que chula, con sus piratas… snifff)

Para la Rom, hablamos en privado.

[root@Jarvis ~]# scummvm

WARNING: FSNode::createReadStream: '.scummvmrc' does not exist!
Default configuration file missing, creating a new one

Y ya está, ahora solo hay que cargar las ROM y a jugar.

Eso sí, necesitaréis conectar un ratón para poder manejarlo y un teclado.

Para añadir el nuevo juego:

(http://i58.tinypic.com/2a5eps6.jpg)

Dentro navegamos por la estructura de directorios hasta que nos metamos dentro del directorio que tiene nuestro juego:

(http://i57.tinypic.com/iypr8j.jpg)

Y el sistema nos reconocerá el juego sin problemas:

(http://i62.tinypic.com/2qjkepz.jpg)

Y listo, a disfrutarlo, [aquí](http://www.gr-lida.org/tutoriales/ver/5/tutorial-scummvm) tenéis una mini guía de los comandos de teclado de Scumm VM (salvar partida, cargar partida, etc.)

En esa guía anuncian CTRL+F5 como el comando para sacar a mitad de juego el menú que nos interesa… el de cargar/salvar salir, etc. En el caso de los teclados de Apple, por lo menos en el que se conecta por USB la combinación de teclas a utilizar es CTRL+FN+F5, tenedlo en cuenta, que si no te quedas como atrapado dentro del juego ;)

Como podréis comprobar, no hay sonido… **el sonido es algo puñetero en las Rpi**, vamos a ver si lo conseguimos activar.

1.- Acutalizamos/completamos la instalación de Alsa

[root@Jarvis ~]# pacman -S alsa-firmware alsa-lib alsa-plugins alsa-utils

atención: alsa-lib-1.0.27.2-1 está actualizado -- re-instalando
resolviendo dependencias...
verificando conflictos...
Paquetes (4): alsa-firmware-1.0.27-2 alsa-lib-1.0.27.2-1 alsa-plugins-1.0.27-2 alsa-utils-1.0.27.2-1
Tamaño Total de Descarga: 2,98 MiB
Tamaño Total Instalado: 15,56 MiB
Tamaño neto a actualizar: 14,12 MiB
:: ¿Continuar con la instalación? [S/n] S
:: Recuperando paquetes ...

alsa-firmware-1.0.27-2-any 2,0 MiB 1480K/s 00:01     [#########################################################################] 100%
alsa-plugins-1.0.27-2-armv6h 50,4 KiB 245K/s 00:00   [#########################################################################] 100%
alsa-utils-1.0.27.2-1-armv6h 905,8 KiB 1201K/s 00:01 [#########################################################################] 100%
(4/4) verificando llaves en el llavero               [#########################################################################] 100%
(4/4) verificando la integridad de los paquetes      [#########################################################################] 100%
(4/4) cargando los archivos del paquete...           [#########################################################################] 100%
(4/4) verificando conflictos entre archivos          [#########################################################################] 100%
(4/4) verificando el espacio disponible en disco     [#########################################################################] 100%
(1/4) instalando alsa-firmware                       [#########################################################################] 100%
(2/4) reinstalando alsa-lib                          [#########################################################################] 100%
(3/4) instalando alsa-plugins                        [#########################################################################] 100%

Dependencias opcionales para alsa-plugins
libpulse: PulseAudio plugin[instalado]
jack: Jack plugin[instalado]
ffmpeg: libavcodec resampling plugin, a52 plugin
libsamplerate: libsamplerate resampling plugin[instalado]
speex: libspeexdsp resampling plugin
(4/4) instalando alsa-utils

2.- Activamos el módulo de sonido

[root@Jarvis ~]# modprobe snd-bcm2835

3.- Creamos el fichero de configuración del módulo `/etc/modules-load.d/snd-bcm2835.conf`

[root@Jarvis ~]# echo "snd-bcm2835" >> /etc/modules-load.d/snd-bcm2835.conf

4.- Arrancamos el “mezclador” para ver que no esté silencioado

[root@Jarvis ~]# alsamixer

5.- Podemos hacer un test de altavoces

[root@Jarvis ~]# speaker-test -c 2
speaker-test 1.0.27.2
Playback device is default
Stream parameters are 48000Hz, S16_LE, 2 channels
Using 16 octaves of pink noise
Rate set to 48000Hz (requested 48000Hz)
Buffer size range from 256 to 16384
Period size range from 256 to 16384
Using max buffer size 16384
Periods = 4

6.- Y salvamos los cambios

[root@GumPI ~]# alsactl store

Y ahora, si entramos a ScummVM y vamos a opciones podremos activar el sonido por Midi en los ajustes y tener sonido… o no… por que no he podido hacer más pruebas, pero cuando lo consiga aquí os actualizo la guía.

## Instalación de Imagen de PiMAME

Tenemos que pasar la imagen del sistema operativo que nos descargamos a una tarjeta SD.

Recordad que en los primeros mensajes de este hilo se trataba en profundidad esto.

La imagen la obtenemos de la [página del proyecto](http://sourceforge.net/projects/pimame/files/latest/download?source=files)

Y una vez cargada esta distribución es de las “todo hecho”, colocar en la ranura y encender.

Nos aparece un menú para elegir el emulador que queramos y listo, a elegir las Roms y a jugar.

Para pasar las Roms, lo más sencillo es hacerlo con un cliente SFTP y pasar los ficheros a la Raspberry.

El usuario de conexión es pi, y la clave es raspberry.

El lugar donde dejar las Roms es `/home/pi/pimame/roms`

Si queréis más información (de hecho es TODA la información) de esta distribución, aquí está [pimame.org](http://pimame.org)
