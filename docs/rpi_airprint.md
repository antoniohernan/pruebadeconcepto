---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 9) – práctico 3 - Servidor AirPrint"
date: "2020-02-08"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
---

## Servidor de Impresión

Vamos a instalar en nuestra RPI el software necesario para convertirla en una estación de impresión en red, así como en un servidor de impresión compatible con AirPrint.

De esta manera, nuestra impresora que no tiene ni Wifi ni es compatible con AirPrint pasaría a serlo.

Para todo esto vamos a emplear un software llamado [CUPS](http://es.wikipedia.org/wiki/Common_Unix_Printing_System), que fue inicialmente desarrollado por Easy Software Products, para ser comprado en 2002 por Apple, la cual licencio el software como GNU GPL, y empezó a implementarlo en sus sistemas Mac OSX a partir de la versión 10.2.

Así que cuando estamos imprimiendo en nuestros Mac OSX y pensamos en complejos controladores y demás… todo se reduce a lo mismo.. a CUPS corriendo por debajo muy bien presentado esos sí…

## Instalación

Vamos a instalar los paquetes que se corresponden a Cups, Cups-pdf (que nos permite realizar impresiones sobre pdf) y [Gutenprint](http://en.wikipedia.org/wiki/Gutenprint) (que son los drivers para casi todas las impresoras).

Para poder hacer AirPrint necesitamos también instalar [Avahi](http://avahi.org) , que no es más que un [Bonjour](http://www.apple.com/es/support/bonjour/) (esto es sonará más), esto es, sirve para anunciar los servicios que tu equipo ofrece al resto de equipos de la red.

Instalamos todo entonces, con ayuda de pacman (se instala python2 aunque ya lo teníamos instalado de antes, por sisólo quieres probar esto de la impresión…)

[root@Jarvis ~]# pacman -Sy cups cups-pdf gutenprint avahi pycups python2

:: Sincronizando las bases de datos de paquetes...
core 153,6 KiB 464K/s 00:00    [###############] 100%
extra 2031,8 KiB 1628K/s 00:01 [###############] 100%
community está actualizado
alarm está actualizado
aur está actualizado
atención: avahi-0.6.31-11 está actualizado -- re-instalando
resolviendo dependencias...
verificando conflictos...
Paquetes (56): bc-1.06.95-1.1 cairo-1.12.16-1 colord-1.0.6-1 cups-filters-1.0.49-1 damageproto-1.2.1-2 dconf-0.18.0-1 elfutils-0.158-1 fixesproto-5.0-2 fontconfig-2.11.0-1 freetype2-2.5.3-2 ghostscript-9.10-3 graphite-1:1.2.4-1 harfbuzz-0.9.26-1 hicolor-icon-theme-0.13-1 jasper-1.900.1-10 js-17.0.0-1 kbproto-1.0.6-1 lcms2-2.5-2 libdrm-2.4.52-1 libgusb-0.1.6-1 libice-1.0.8-2 libpaper-1.1.24-7 libpciaccess-0.13.2-2 libsm-1.2.2-2 libvdpau-0.7-1 libx11-1.6.2-1 libxau-1.0.8-2 libxcb-1.10-1 libxdamage-1.1.4-1 libxdmcp-1.1.1-1 libxext-1.3.2-1 libxfixes-5.0.1-1 libxrender-0.9.8-1 libxshmfence-1.1-1 libxt-1.1.4-1 libxxf86vm-1.1.3-1 llvm-libs-3.4-1 mesa-10.1.0-4 mesa-libgl-10.1.0-4 nspr-4.10.4-1 openjpeg-1.5.1-2 pixman-0.32.4-1 polkit-0.112-1 poppler-0.24.5-1 qpdf-5.1.1-1 renderproto-0.11.1-2 shared-colorprofiles-0.1.5-1 wayland-1.4.0-1 xcb-proto-1.10-2 xextproto-.3.0-1 xf86vidmodeproto-2.3.1-2 xproto-7.0.25-1 avahi-0.6.31-11 cups-1.7.1-4 cups-pdf-2.6.1-2 gutenprint-5.2.9-2

Tamaño Total de Descarga: 52,90 MiB
Tamaño Total Instalado: 358,88 MiB
Tamaño neto a actualizar: 356,75 MiB
:: ¿Continuar con la instalación? [S/n] S

La instalación es un poco larga por la cantidad de paquetes que añade o actualiza en el sistema, pero no debe presentar problema alguno.

## Configuración

El fichero de configuración de cups está en /etc/cups/cupsd.conf

Lo editamos para configurar el rango de direcciones IP que podrán hacer uso de su servicio WEB (y por tanto poder configurarlo por web desde otro equipo de la red), así como para dar permisos a la administración.

[root@Jarvis ~]# vi /etc/cups/cupsd.conf

Donde dice

\# Only listen for connections from the local machine.
Listen localhost:631

Por

\# Only listen for connections from the local machine.
Listen 0.0.0.0:631

Y también añadimos el acceso desde la red local mediante la modificación de estas otras entradas:

\# Restrict access to the server...

<Location />
Order allow,deny
</Location>

# Restrict access to the admin pages...

<Location /admin>
Order allow,deny
</Location>

Por estas

\# Restrict access to the server...

<Location />
Order allow,deny
Allow @LOCAL
</Location>

# Restrict access to the admin pages...

<Location /admin>
Order allow,deny
Allow @LOCAL
</Location>

## Iniciamos el servicio de impresión

[root@Jarvis ~]# systemctl start cups

Y probamos si tenemos accesos la IP de nuestra máquina y el puerto 631 para la consola (http://192.168.1.20:631)

## Configuración impresora

Conectamos nuestra impresora por USB a la RPI y entramos a la url antes mencionada.

Aquí tenemos que ir a la pestaña Administración.

Una vez entramos vemos Impresoras y el botón que dice Añadir Impresora.

Y directamente, en impresoras locales veremos dos, una posible impresora virtual basada en PDF (cups-pdf que hemos instalado), y nuestra impresora que es bastante probable que sea reconocida automáticamente.

En la siguiente página podemos cambiar si queremos el nombre al servidor de impresión para esta impresora, pero lo más importante es marcar la casilla inferior que dice **Compartir esta impresora**.

En la siguiente pantalla debemos escoger entre todos los modelos disponibles en nuestro en concreto, y pulsar el botón que está en la parte inferior de la pantalla que dice **Añadir Impresora**.

Y ahora, ya por fin, la última pantalla de configuración, donde escogemos el comportamiento por defecto de la impresora. Cambiamos el tipo de papel de “Carta/Letter” a A4 que es más común entre nosotros, y le indicamos que no modifique por norma las dimensiones del elemento a imprimir; pulsamos sobre Cambiar opciones predeterminadas.

Si queremos probar que funciona, desde la pestaña que dice Impresoras encontraremos nuestra impresora y un desplegable que dice Mantenimiento.

En ese desplegable escogemos Imprimir página de prueba y listo, veremos como la pantalla se va refrescando y nos muestra el estado de la cola de impresión según tenga esta actividad alguna.

## Compartiendo esta impresora para OSX

Si, lo sé, con OSX se puede instalar la impresora y compartirla al resto de la red Mac sin andarse con estos experimentos… Pero eso no es divertido…

Pero… espera… que aquí también es fácil, sólo tenemos que entrar en nuestro OSX en las Preferencias del Sistema, ahí en las Impresoras, y una vez que estemos allí, cuando pulsemos sobre el símbolo + encontraremos que nuestra **impresora conectada a la RPI se está anunciando**, la escogemos y dejamos que OSX termine de hacer el resto de cosas.

Es otra forma de convertir una impresora “no wifi” en una estación de impresión sin necesidad de tener un ordenador OSX encendido de continuo, con el gasto eléctrico que eso lleva (más el de la impresora en Stand-by…)

## AirPrint

Y aquí lo divertido de todo esto, poder utilizarla como impresora AirPrint para poder imprimir cosas desde nuestros iPhones e iPads.

La verdad es que no se me ocurren muchos ejemplos de porqué podamos necesitar esta funcionalidad, tan sólo se me viene a la cabeza la impresión de entradas que compres en la web y cosas similares, o imprimir una receta que tengas en pdf para no tener el iPad en la cocina al lado de los fogones.. etc.

Bien, vamos paso a paso, **arrancamos el demonio de Avahi**

[root@Jarvis ~]# systemctl start avahi-daemon

Creamos el directorio para la instalación de los programas Python y descargamos el que nos genera la configuración para AirPrint:

[root@Jarvis /]# mkdir /opt/airprint
[root@Jarvis /]# cd /opt/airprint
[root@Jarvis airprint]# wget -O airprint-generate.py --no-check-certificate https://raw.github.com/tjfontaine/airprintgenerate/master/airprint-generate.py

--2014-03-29 15:11:17-- https://raw.github.com/tjfontaine/airprint-generate/master/airprint-generate.py

Resolviendo raw.github.com (raw.github.com)... 185.31.16.133
Conectando con raw.github.com (raw.github.com)[185.31.16.133]:443... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 10093 (9,9K) [text/plain]
Grabando a: “airprint-generate.py” 100% [============================================================================================================>]
10.093 --.-K/s en 0,003s
2014-03-29 15:11:18 (3,57 MB/s) - “airprint-generate.py” guardado [10093/10093]

Como no usamos Python sino que usamos Python2 tenemos que modificar este fichero que nos hemos descargado.

Para esto, lo editamos con vi y modificamos la primera línea, añadiendo un 2 al final para que quede así:

#!/usr/bin/env python2

Vamos a ejecutar por tanto este script en Pyhton para ver que información no genera:

[root@Jarvis airprint]# chmod 755 airprint-generate.py
[root@Jarvis airprint]# ./airprint-generate.py
[root@Jarvis airprint]# mv AirPrint-Canon_MP140_series.service /etc/avahi/services

En la primera línea le hemos dado permisos de ejecución.

En la segunda lo ejecutamos y en la tercera vemos como se nos ha generado el servicio de nuestra impresora conectada (la Canon MP140 ;)), así que movemos este fichero de configuración del servicio a la ruta adecuada.

En caso de conectar posteriormente otra impresora pues sólo hay que repetir este paso para poder tener otra impresora sirviendo AirPrint.

Sólo nos queda reiniciar el demonio de Avahi y ver que reconozca nuestra impresora:

[root@Jarvis airprint]# systemctl restart avahi-daemon
[root@Jarvis airprint]# systemctl status avahi-daemon

avahi-daemon.service - Avahi mDNS/DNS-SD Stack
Loaded: loaded (/usr/lib/systemd/system/avahi-daemon.service; disabled)
Active: active (running) since sáb 2014-03-29 15:18:49 CET; 9s ago
mar 29 15:18:51 Jarvis avahi-daemon[1239]: Service "AirPrint Canon_MP140_series @ Jarvis (/services/AirPrint-
Canon_MP140_series.service) successfully established.

Y desde por ejemplo un iPad, podemos imprimir un PDF, usando el icono de compartir de Adobe Reader, veremos en **Imprimir** como sale nuestra “impresora AirPrint”.

O mientras estamos navegando con Safari, podemos usar el botón de compartir, escoger la opción de imprimir y ahí veremos nuestra impresora.

No es un sistema rápido, por que hay muchos intermediarios… mucha red, mucha conversiones, pero si tienes una impresora y no quieres invertir en una nueva con su Wifi, su AirPrint y demás, pues con la Rpi puedes además de hacer todo lo “otro” que hagas con ella, usarla como centro de impresión.
