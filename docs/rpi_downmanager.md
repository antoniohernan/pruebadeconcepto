---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 8) – práctico 2 - Gestor de Descargas"
date: "2020-02-08"
---

## Gestor de descargas

No debemos perder de vista que el gasto energético de estas máquinas es mínimo, se puede situar en casi todos los casos **por debajo de los 5W**.

Esto lo hace que sean ideales para tener como máquinas que estén permanentemente conectadas a la red con el propósito de descargar y compartir contenidos, bien sea por redes P2P como por servicio de descarga directa.

En cualquier caso, necesitaremos conectar a la máquina una unidad de disco externo por los puertos USB, y este disco que se conecte no puede tener un consumo elevado pues la máquina no podrá con ello.

Todo lo que sea más grande de un pendrive es necesario que esté autoalimentado (con su propio adaptador de corriente) o bien que este conectado a un HUB o regleta USB que esté a su vez autoalimentada.

Configuraciones para la descarga y los discos hay muchas, podemos emplear los discos de un NAS para dejar allí los datos, de otra unidad remota en red, etc. pero la configuración de un disco USB para este fin quizás sea la más sencilla.

## Infraestructura previa

Como he dicho necesitamos un disco externo para guardar todo lo que descarguemos, las descargas en curso, etc.

El proceso para particionar ese disco que conectemos (que con suerte nos vendrá de tienda con formato NTFS o HFS+) es igual que el que hemos seguido para particionar las tarjetas SD.

Localizamos la unidad/dispositivo.

Ejecutamos el comando `fdisk` para ese `/dev/sd_loquesea`

Y borramos las particiones que tenga para crear otra nueva, dándole todo el espacio disponible en el disco o bien creando varias particiones por, por ejemplo, tamaño en Gb.

Terminado el particionamiento, como hemos visto antes, creamos el sistema de ficheros ext4 y lo montamos modificando el fichero `/etc/fstab` .

Como leéis he pasado un poco de puntillas por este paso inicial, pero es algo que ya llevo contadas al menos un par de veces… mirad atrás que si no la guía saldrá kilométrica.

En mi caso, para hacer estas pruebas he seguido usando el pendrive de pasos anteriores, el que está montado sobre `/BCK` y `/srv`.

Lo único que yo creo es una estructura de directorios para tenerlo todo organizado, pero como siempre digo, esto es como yo lo hago, que no es ni la mejor ni la única manera.

```
[root@Jarvis ~]# mkdir -p /srv/downloads/Torrent/Finalizadas
[root@Jarvis ~]# mkdir -p /srv/downloads/Torrent/EnCurso
[root@Jarvis ~]# mkdir -p /srv/downloads/Directas
```

## Descarga P2P

Para la gestión de descargas de la red Torrent (¿Aun existe algo en las redes Emule??) el cliente que por lo menos a mi me resulta más fácil de instalar, configurar y que menos recursos consume en el sistema es [Transmission](http://www.transmissionbt.com).

Además, este cliente también está disponible en el mismo “formato” para nuestros Mac OSX por lo que es posible que a muchos les suene.

Los pasos de instalación son extremadamente simples, no así la configuración que tiene un punto delicado que ahora os cuento.

Para instalar, lo de siempre, `pacman` …

```
[root@Jarvis srv]# pacman -S transmission-cli
resolviendo dependencias...
verificando conflictos...
Paquetes (2): libevent-2.0.21-3 transmission-cli-2.82-1
Tamaño Total de Descarga: 0,66 MiB
Tamaño Total Instalado: 3,92 MiB
:: ¿Continuar con la instalación? [S/n] S
:: Recuperando paquetes ...
libevent-2.0.21-3-armv6h 183,7 KiB 536K/s 00:00 [################] 100%
transmission-cli-2.82-1-armv6h 491,8 KiB 700K/s 00:01 [################] 100%
(2/2) verificando llaves en el llavero [################] 100%
(2/2) verificando la integridad de los paquetes [################] 100%
(2/2) cargando los archivos del paquete... [################] 100%
(2/2) verificando conflictos entre archivos [################] 100%
(2/2) verificando el espacio disponible en disco [################] 100%
(1/2) instalando libevent [################] 100%
Dependencias opcionales para libevent
python2: to use event_rpcgen.py
(2/2) instalando transmission-cli [################] 100%
```

Este paquete, además de instalar el producto nos crea un usuario:
 
```
[root@Jarvis srv]# tail -1 /etc/passwd
transmission:x:169:169:Transmission BitTorrent Client:/var/lib/transmission:/bin/false
```

Nos ha creado un usuario transmission, incluso un grupo transmission y su home de usuario en `/var/lib/transmission`

Es precisamente en esa ruta donde se guarda el fichero de configuración de Transmission y el que debemos modificar.

El “problema” de este fichero es que se carga en memoria cuando se arranca el cliente, y si realizamos alguna modificación sobre el los cambios no se verán reflejados en los siguientes arranques… y esto puede mosquear un poco y volverte algo loco, haces cambios y luego no los ves ahí donde habías tocado… XD

Arrancamos el cliente, porqué hasta que no se hace el primer arranque no se genera el fichero de configuración, y acto seguido lo paramos para poder modificarlo sin problemas.

```
[root@Jarvis srv]# systemctl start transmission
[root@Jarvis srv]# systemctl stop transmission
```

Ahora en el home del usuario transmission (recordar nuestra amiga la virgulilla `~transmission` ) se ha generado un directorio `.config`
```
[root@Jarvis srv]# cd ~transmission/.config/transmission-daemon/
[root@Jarvis srv]# vi settings.json
```

Los parámetros que podemos configurar son muchos y no tienen mucha pérdida pero os cuento más o menos lo que suelo modificar yo (atención con las **,** al final de cada línea!!):

`"download-dir": “/srv/downloads/Torrent/Finalizadas”,`  Directorio donde se quedan los ficheros cuando se ha terminado la descarga.

`"download-queue-enabled": true,`  Habilitamos la cola de descargas.

`"download-queue-size": 10,`  Y el tamaño de la cola de descargas, este es el valor de las descargas simultáneas.

`"incomplete-dir": "/Descargas/Torrent/EnCurso",`  Directorio para las descargas en curso.

`"incomplete-dir-enabled": true,`  Y activación para este directorio (¿un poco ridículo no?).

Como sabréis, y si no os lo cuento yo ahora mismo, el cliente de Transmission es por web, y estos son los parámetros que yo modifico para poder tener acceso a esa web con usuario/clave

`"rpc-authentication-required": true,`  Lógico, lo activamos.

`"rpc-enabled": true,` Y lo volvemos a activar…

`"rpc-password": "{4c5248273881de992229dca1f72c531b0129db34pRnyy6th",` Es la clave de acceso, **la primera vez** se pone en texto en “claro” entre las ”.

`"rpc-port": 9091,` Puerto de escucha del cliente web

`"rpc-url": "/transmission/",`Prefijo para componer la URL de acceso a este cliente web.

`"rpc-username": "operador",` Y el usuario con el que nos vamos a conectar.

`"rpc-whitelist-enabled": false,` para que no valide la lista “blanca” de direcciones IP desde las que podemos administrar.

Así, con todos estos parámetros vemos por ejemplo que la url de administración de ese transmission será http://IP_De_Tu_Rpi:9091/transmission

En cuando a la clave, se pone en claro la primera vez, pero cuando arranquemos luego transmission, se cifra y aparecerá un chorro de números y letras.

Y el resto de parámetros, pues si queremos esos ya podemos cambiarlos por el cliente web, por que haciendo estos cuatro cambios que os digo ya tendríais todo salvo la configuración de velocidades máxima/mínima, el scheduler para las zonas horarias en las que poder tener distinta velocidad, etc.

Arrancamos el cliente y lo fijamos para que en los sucesivos rearranques del servidor se active.
```
[root@Jarvis srv]# systemctl start transmission
[root@Jarvis srv]# systemctl enable transmission
```

## Descarga directa mega.co.nz

Para la descarga desde servicios directos podemos emplear software como Jdownloader2, y ahí tendríamos todas las posibilidades de hostings disponibles.

A mi, personalmente no me gusta mucho, el tema de la resolución de Capchas no termina de ir bien o debes buscar algún servicio de resolución de capuchas que también te llevan un tiempo y al final terminas haciéndolo todo a mano o buscado los enlaces en mega.co.nz que no tiene capchas, y encima da una velocidad de descarga de escándalo.

No tenemos cliente nativo de Mega para las Rpi, así que toca buscar alguna solución que ya lo haya implementado. Hay pocas por no decir que sólo hay una [megatools](http://megatools.megous.com) .

Este software no es más que una capa sobre la propia Api de Mega que nos permite realizar casi todas las tareas que queramos con nuestra cuenta de Mega, aunque la función que nos interesa es la de descarga de enlaces públicos.

Por otro lado, este software se encuentra en estos momentos “congelado” por las cláusulas relativas a la privacidad que la Api de Mega te obliga a “acatar” si las quieres usar.

En cualquier caso, la parte de descargas de enlaces públicos que es lo que vamos a usar funcionan correctamente. Este software no lo vamos a encontrar en los repositorios oficiales y por tanto esta vez no vamos a emplear pacman.

Debemos **descargar los fuentes del software y compilarlo**.

No os asustéis por que esto es de lo más normal y no todos los productos deben existir en formato paquete en repositorio para poder ser instalado.

Necesitamos por tanto los fuentes del software que descargamos así:

```
[root@Jarvis ~]# cd /root/Descargas
[root@Jarvis Descargas]# wget http://megatools.megous.com/builds/megatools-1.9.91.tar.gz
--2014-03-28 18:58:33-- http://megatools.megous.com/builds/megatools-1.9.91.tar.gz
Resolviendo megatools.megous.com (megatools.megous.com)... 77.93.202.41
Conectando con megatools.megous.com (megatools.megous.com)[77.93.202.41]:80... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 450518 (440K) [application/x-gzip]
Grabando a: “megatools-1.9.91.tar.gz”
100%[=============================================>] 450.518 830KB/s en 0,5s
2014-03-28 18:58:34 (830 KB/s) - “megatools-1.9.91.tar.gz” guardado [450518/450518]
```

Y los compiladores para generar el programa ejecutable, que se obtienen así:
```
[root@Jarvis Descargas]# pacman -S make gcc glib-networking curl fuse pkg-config
atención: make-4.0-2 está actualizado -- re-instalando
atención: gcc-4.8.2-7 está actualizado -- re-instalando
resolviendo dependencias...
verificando conflictos...
Paquetes (9): gcc-4.8.2-7 make-4.0-2 gnutls-3.2.12.1-1 gsettings-desktop-schemas-3.10.1-1 libproxy-0.4.11-2
libtasn1-3.4-1 nettle-2.7.1-1 p11-kit-0.20.2-1 glib-networking-2.38.2-1 curl-7.35.0-1 fuse-2.9.3-2
pkg-config-0.28-1
Tamaño Total Instalado: 65,97 MiB
Tamaño neto a actualizar: 0,00 MiB
:: ¿Continuar con la instalación? [S/n] S
(2/2) verificando llaves en el llavero [################] 100%
(2/2) verificando la integridad de los paquetes [################] 100%
(2/2) cargando los archivos del paquete... [################] 100%
(2/2) verificando conflictos entre archivos [################] 100%
(2/2) verificando el espacio disponible en disco [################] 100%
(1/2) reinstalando make [################] 100%
(2/2) reinstalando gcc [################] 100%
…
```

Descomprimimos los fuentes:
```
[root@Jarvis Descargas]# tar -xvzf megatools-1.9.91.tar.gz
megatools-1.9.91/
megatools-1.9.91/Makefile.in

…

megatools-1.9.91/m4/lt~obsolete.m4
megatools-1.9.91/install-sh
```

Y ahora la secuencia de comandos de configuración, compilación e instalación son los siguientes:
```
[root@Jarvis ~]# cd ~/Descargas/megatools-1.9.91
[root@Jarvis megatools-1.9.91]# ./configure
[root@Jarvis megatools-1.9.91]# make
[root@Jarvis megatools-1.9.91]# make install
```
Llegados a este último punto, si no hemos encontrado errores (que aparecen en grande en la pantalla…) tendremos

en la ruta `/usr/local/bin`instalados los programas.

Si queremos podemos eliminar la descarga de fuentes de megatools que no nos harán falta más:
```
[root@Jarvis megatools-1.9.91]# cd ~/Descargas
[root@Jarvis Descargas]# rm -fr megatools-1.9.91*
```
Ahora hacemos una prueba de descarga, necesitamos un enlace válido a [mega.co.nz](http://mega.co.nz) para probarlo.

Por ejemplo este: [https://mega.co.nz/#!fNQkWTDR!vXyDZ1EbBVvYzBLVCqCzUfVi2CVW-uhfmKevIxn9ZWU](https://mega.co.nz/#!fNQkWTDR!vXyDZ1EbBVvYzBLVCqCzUfVi2CVW-uhfmKevIxn9ZWU)

El comando que ejecutaríamos es:
```
[root@Jarvis Descargas]# megadl 'https://mega.co.nz/#!fNQkWTDR!vXyDZ1EbBVvYzBLVCqCzUfVi2CVW-uhfmKevIxn9ZWU'

True Detective 1x01 [josele].avi: 2% - 15,2 MiB of 622,0 MiB
```

## Automatización de descargas

Llegados a este punto podemos descargar tanto de la red Torrent como de las descargas directas de Mega.

Ahora queremos automatizar las descargas, para la parte Torrent lo vemos ahora después.

Para la parte de Mega la forma de automatizarlo exige un poco de programación.

Esto que os pongo a continuación es un pequeño script, hecho de una manera un tanto improvisada pero que cumple con su cometido.

Lee de un fichero los enlaces que le digamos uno a uno, los descarga y una vez que esta descarga ha finalizado los borra del fichero y sigue con el siguiente.

```
#!/usr/bin/perl
# Gestion de fichero de
# Variables de entorno

$FICENTRADA="~/enlaces.txt";
$FICTMP="/tmp/enlaces.tmp";
$EXEMEGA="/usr/local/bin/megadl";

# Bucle de ejecucion
# Filtrado previo

system("cat ${FICENTRADA} 2>/dev/null | grep '^http' > ${FICTMP}; mv ${FICTMP} ${FICENTRADA} 2>/dev/null");
@ENLACES=\`cat ${FICENTRADA} 2>/dev/null | grep '^http'\`;
if (@ENLACES)
 {
  foreach(@ENLACES)
  {
  $ENLACE=$_; chomp($ENLACE);

  # Ejecutamos la descarga y esperamos a que termine
  @SALIDA=\`${EXEMEGA} --no-progress '${ENLACE}' 2>/dev/null\`;
  system("grep -v '${ENLACE}' ${FICENTRADA} > ${FICTMP}; mv ${FICTMP} ${FICENTRADA}");
  }
 }
else
 { print "\\n No existen enlaces por descargar\\n"; }
```

Por supuesto que esto se puede superar, no es difícil…

El proceso se programar para que se inicie su ejecución cada X horas, sólo algunos días de la semana, etc. Recordar que antes os conté como funciona crontab y que es muy fácil establecer un patrón de comportamiento de un programa, es darle vueltas a como queremos que nos funcione.

Imaginad que sólo queréis que esté descargando de lunes a viernes por la noche, y que los ficheros de descarga se los habéis dejado a última hora del día… No se, es un ejemplo, yo de hecho lo hago así, se ejecuta sólo por las noches y coge los enlaces que le he ido dejando a lo largo del día.

El fichero de enlaces se modifica (editándolo) para añadir nuevas Urls de nuevos ficheros a descargar.

Y para hacerlo, desde nuestro MacOSX de una manera bastate cómoda y rápida, sin tener que andar con terminales, usaremos la conexión a través de **SAMBA**, os lo cuento un poco más adelante.

## Uso de Feeds RSS para gestión de series

Una de las opciones de descarga más interesantes es la gestión desasistida de nuevos contenidos, en concreto de las series, no tener que preocuparnos cuando aparece un nuevo episodio, etc.

Para esto, el montaje que tenemos hecho con trasmission es perfectamente válido, pero además tenemos que añadir algún que otro componente más, en este caso se llama [FlexGet](http://flexget.com) , y por supuesto necesitaremos los correspondientes Feeds de los que obtener la información, como por ejemplo [showrss.info](http://showrss.info) .

Empezamos, paramos transmission si es que no lo habíamos hecho ya:
```
[root@Jarvis ~]# systemctl stop transmission
```
Instalamos FlexGet, recurrimos a pacmac y no os asustéis por la cantidad de dependencias que encuentra pacman por satisfacer y cosas a instalar.

Hay una dependencia que en el listado os saldría al final como opcional pero que aquí va a ser necesaria, para poder comunicar con python al rpc de transmission, así que esta a la añadimos a mano en la línea de comando de pacman.
```
[root@Jarvis ~]# pacman -S flexget python2-transmissionrpc
resolviendo dependencias...
verificando conflictos...
Paquetes (23): libyaml-0.1.5-1 python2-2.7.6-3 python2-beautifulsoup4-4.3.2-1 python2-certifi-1.0.0-1 python2-dateutil-2.1-2.1-8
python2-feedparser-5.1.3-2 python2-html5lib-0.999-3 python2-jinja-2.7.2-1 python2-jsonschema-2.3.0-1
python2-markupsafe-0.19-1
python2-progressbar-2.3-4 python2-pynzb-0.1.0-2 python2-pyrss2gen-1.1-2 python2-requests-2.2.1-1
python2-rpyc-3.2.3-1
python2-setuptools-3.3-1 python2-six-1.6.1-1 python2-sqlalchemy-0.9.3-1 python2-tmdb3-0.6.17-1 python2-
tvrage-0.3.0-1
python2-yaml-3.10-3 sqlite-3.8.4.1-1 flexget-1.2.58-1
Tamaño Total de Descarga: 12,67 MiB
Tamaño Total Instalado: 82,94 MiB
:: ¿Continuar con la instalación? [S/n] s
:: Recuperando paquetes ...
```
Ahora nos vamos a registrar en la página de los rss, que es algo sencillo, sin correos ni nada de por medio.

Allí pues puedes configurar si quieres un RSS para todas tus series filtrados por HD o no HD, si quieres un RSS para una serie en concreto… es darle vueltas por que no es nada complicado, tan sólo hay que echarle unos minutillos de tiempo.

## Vamos a editar la configuración de FlexGet.

El fichero de configuración es bastante crítico, en el sentido de que cada línea de texto que aparece en el debe estar convenientemente escrita en cuanto a posición de espacios se refiere.

Ahora en cuanto veáis un ejemplo lo veréis más claro.

Configuramos…
```
[root@Jarvis ~]# mkdir ~/.flexget
[root@Jarvis .flexget]# vi config.yml

tasks:
rss:
rss: http://showrss.info/feeds/699.rss
all_series: yes
quality: 720p+
transmission:
host: 192.128.1.20
port: 9091
username: 'operador'
password: 'operador'
ratio: -1
path: /srv/downloads/Torrent/EnCurso/
sort-files:
find:
path: /srv/downloads/Torrent/EnCurso/
regexp: '.*\\.(avi|mkv|mp4)$'
recursive: yes
accept_all: yes
seen: local
thetvdb_lookup: yes
all_series:
parse_only: yes
move:
to: /srv/downloads/Torrent/Finalizadas/{{ tvdb_series_name }}/
filename: '{{ tvdb_series_name }} - {{ series_id }} - {{ tvdb_ep_name }}{{ location | pathext }}'
```
El fichero básico no tiene mucha complejidad, la URL de nuestro FEED, las claves y url de nuestro transmission, rutas donde llegan los ficheros, la conexión al repositorio de thetvdb para que ponga el nombre a las series y por último que mueva el contenido descargado a la ruta definitiva.

Ahora sólo nos faltan dos cosas por hacer.

La primera es arrancar y dejar arrancando transmission:
```
[root@Jarvis ~]# systemctl start transmission
[root@Jarvis ~]# systemctl enable transmission
```
Y automatizar flexget para que se ejecute cada cierto tiempo y mire lo que hay para descargar, lo que se está descargando y lo que se ha finalizado de descargar…

Y para esto lo mejor es emplear `crontab -e` del cual ya hemos hablado.

Una última nota, si queremos probar si nuestra configuración es correcta antes de poner todo esto en marcha:
```
[root@Jarvis .flexget]# flexget --test --logfile /var/log/flexget.log execute
2014-03-29 10:15 WARNING details sort-files Task didn't produce any entries. This is likely due to a misconfigured or non-functional input.
2014-03-29 10:15 VERBOSE details sort-files Summary - Accepted: 0 (Rejected: 0 Undecided: 0 Failed: 0)
2014-03-29 10:15 INFO transmission rss Trying to connect to transmission...
2014-03-29 10:15 INFO transmission rss Successfully connected to transmission.
2014-03-29 10:15 VERBOSE details rss Produced 14 entries.
2014-03-29 10:15 VERBOSE task rss REJECTED: \`True Detective 1x08 Form and Void\` by quality plugin because hdtv h264 does not match quality requirement [<Requirements(720p+)>]
2014-03-29 10:15 VERBOSE task rss REJECTED: \`True Detective 1x07 After You've Gone\` by quality plugin because hdtv h264 does not match quality requirement [<Requirements(720p+)>]
2014-03-29 10:15 VERBOSE task rss REJECTED: \`True Detective 1x06 Haunted Houses\` by quality plugin because hdtv h264 does not match quality requirement [<Requirements(720p+)>]
2014-03-29 10:15 VERBOSE task rss REJECTED: \`True Detective 1x05 The Secret Fate Of All Life\` by quality plugin because hdtv h264 does not match quality requirement [<Requirements(720p+)>]
2014-03-29 10:15 VERBOSE task rss REJECTED: \`True Detective 1x04 Who Goes There?\` by quality plugin because hdtv h264 does not match quality requirement [<Requirements(720p+)>]
2014-03-29 10:15 VERBOSE task rss REJECTED: \`True Detective 1x03 The Locked Room\` by quality plugin because hdtv h264 does not match quality requirement [<Requirements(720p+)>]
2014-03-29 10:15 VERBOSE task rss REJECTED: \`True Detective 1x02\` by quality plugin because hdtv h264 does not match quality requirement [<Requirements(720p+)>]
2014-03-29 10:15 VERBOSE task rss ACCEPTED: \`True Detective 1x02 720p\` by series plugin because choosing best available quality
2014-03-29 10:15 VERBOSE task rss ACCEPTED: \`True Detective 1x05 The Secret Fate Of All Life 720p\` by series plugin because choosing best available quality
2014-03-29 10:15 VERBOSE task rss ACCEPTED: \`True Detective 1x07 After You've Gone 720p\` by series plugin because choosing best available quality
2014-03-29 10:15 VERBOSE task rss ACCEPTED: \`True Detective 1x08 Form and Void 720p\` by series plugin because choosing best available quality
2014-03-29 10:15 VERBOSE task rss ACCEPTED: \`True Detective 1x04 Who Goes There? 720p\` by series plugin because choosing best available quality
2014-03-29 10:15 VERBOSE task rss ACCEPTED: \`True Detective 1x06 Haunted Houses 720p\` by series plugin because choosing best available quality
2014-03-29 10:15 VERBOSE task rss ACCEPTED: \`True Detective 1x03 The Locked Room 720p\` by series plugin because choosing best available quality
2014-03-29 10:15 VERBOSE details rss Summary - Accepted: 7 (Rejected: 7 Undecided: 0 Failed: 0)
2014-03-29 10:15 INFO transmission rss Would add magnet:?xt=urn:btih:2DB3E1CAA8CDE3D9651C7B539CA887F714322E35&dn=True+Detective+S01E08+720p+HDTV+x264+2HD&tr=udp://tracker.openbittorrent.com:80&tr=udp://tracker.to transmission
2014-03-29 10:15 INFO transmission rss Would add magnet:?xt=urn:btih:41AFF51BA027C488280E248511A9C13C11A9FA11&dn=True+Detective+S01E07+720p+HDTV+x264+KILLERS&tr=udp://tracker.openbittorrent.com:80&tr=udp://tracker.to transmission
2014-03-29 10:15 INFO transmission rss Would add magnet:?xt=urn:btih:5D7AF918D4CA00709DB6F6E924217E57A6143BD0&dn=True+Detective+S01E06+720p+HDTV+x264+2HD&tr=udp://tracker.openbittorrent.com:80&tr=udp://tracker.to transmission
2014-03-29 10:15 INFO transmission rss Would add magnet:?xt=urn:btih:7D1795E2E575AA68F57E674F6DD594E847A6A851&dn=True+Detective+S01E05+720p+HDTV+x264+KILLERS&tr=udp://tracker.openbittorrent.com:80&tr=udp://tracker.to transmission
2014-03-29 10:15 INFO transmission rss Would add magnet:?xt=urn:btih:ABE95E4C2BF49C1FD99EBA0DBE0591B965439042&dn=True+Detective+S01E04+720p+HDTV+x264+KILLERS&tr=udp://tracker.openbittorrent.com:80&tr=udp://tracker.to transmission
2014-03-29 10:15 INFO transmission rss Would add magnet:?xt=urn:btih:A7D60019E259A95921B1310509783DE7AC80ECD9&dn=True+Detective+S01E03+720p+HDTV+x264+KILLERS&tr=udp://tracker.openbittorrent.com:80&tr=udp://tracker.to transmission
2014-03-29 10:15 INFO transmission rss Would add magnet:?xt=urn:btih:C6545AC26191F13DF03B06FC42DC485BA92384C4&dn=True+Detective+S01E02+720p+HDTV+x264+KILLERS&tr=udp://tracker.openbittorrent.com:80&tr=udp://tracker.to transmission
2014-03-29 10:15 INFO manager Removed test database
```
## Exportación SAMBA

Ahora vamos a exportar el contenido de algunas de nuestras carpetas para hacerlos visibles en nuestros escritorios MAC Osx.

Hay otras alternativas de conexión, como [NFS](http://es.wikipedia.org/wiki/NFS) , depende de para que algunas son más robustas y otras my fáciles de configurar.

Si bien a día de hoy parece que Samba (http://es.wikipedia.org/wiki/Samba_(programa)) ha cogido mucha fuerza y es difícil encontrar un dispositivo que no pueda realizar una conexión con este protocolo.

Vamos a exportar una serie de directorios de nuestra RPI para montarlos luego en el MAC y poder hacer operativas tales como el uso de Hazel, Automator, o simplemente visualizar los ficheros e incluso reproducirlos en nuestro MAC Osx.

Primero, tenemos que instalar Samba
```
[root@Jarvis ~]# pacman -S samba

resolviendo dependencias...
verificando conflictos...

Paquetes (16): cifs-utils-6.2-1 gamin-0.1.10-8 iniparser-3.1-4 ldb-1.1.16-1 libbsd-0.6.0-2 libcap-ng-0.7.3-1 libcups-1.7.1-4 libjpeg-turbo-1.3.0-4 libpng-1.6.10-1 libtiff-4.0.3-4 libwbclient-4.1.6-1
smbclient-4.1.6-1 talloc-2.1.0-1 tdb-1.2.12-1 tevent-0.9.21-2 samba-4.1.6-1
Tamaño Total de Descarga: 10,25 MiB
Tamaño Total Instalado: 53,20 MiB
:: ¿Continuar con la instalación? [S/n]
:: Recuperando paquetes ...
libjpeg-turbo-1.3.0-4-armv6h 240,1 KiB 416K/s   00:01 [################] 100%
…

(15/16) instalando smbclient                          [################] 100%
(16/16) instalando samba                              [################] 100%
```
Y lo configuramos, el fichero de configuración de ejemplo de samba es una locura… recurrimos a una plantilla:
```
[root@Jarvis ~]# vi /etc/samba/smb.conf
[global]
workgroup = JarvisGRP
security = user
load printers = no

[DescargasDirectas]
path = /srv/downloads/Directas
public = no
writeable = yes
browseable = yes
valid users = root
```
Arrancamos samba y lo dejamos corriendo como proceso en arranque de máquina:
```
[root@Jarvis ~]# systemctl start smbd nmbd
[root@Jarvis ~]# systemctl enable smbd nmbd
ln -s '/usr/lib/systemd/system/samba.service' '/etc/systemd/system/multi-user.target.wants/samba.service'
```
Y como tenemos la seguridad por usuario, debemos crear el usuario en las tablas de usuarios propias de samba, así que:
```
[root@Jarvis ~]# smbpasswd -a root
New SMB password:
Retype new SMB password:
Added user root.
```
Para conectarnos desde nuestros Mac Osx a esta carpeta compartida, tenemos que abrir una nueva ventana de Finder CMD+N y a continuación conectarnos al servidor CMD+K

En la conexión del servidor pondremos:

`smb://192.168.1.20/DescargasDirectas`

Nos pedirá usuario y clave, que serán el usuario que hemos configurados en `smb.conf` y la clave la que hemos generado con `smbpasswd` y a partir de ese momento se comportará como una carpeta local al sistema Osx.
