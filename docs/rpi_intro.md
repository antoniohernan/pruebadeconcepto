---
title: "Instalando una RaspberryPI todo uso... en 2014"
date: "2019-12-13"
---

Pues sí, **en 2014**, en el momento de escribir este mensaje son algo más de 6 años, pero claro, **6 años en tecnología son mucho tiempo**.

Por aquella época tenía una **RPi modelo B de 512Mb** con la que trasteaba bastante, y también formaba parte del GUM de la Comunidad de Madrid (**Grupo de usuarios MAC**), una cosa lleva a la otra y terminé ofreciendo un taller donde contar el uso que podíamos darles a estas placas, haciendo una instalación y alguno de **sus usos orientados al usuario de MAC**.

Y... **se me fue un poco de las manos**... allí me planté con la intención contar como montar casi de todo, un servidor web, un servidor de archivos, un servidor para copias de seguridad, un servidor de AirPrint/Cups, un media center con XBMC y AirPlay, gestión domótica... todo con lo que había estado trasteando meses atrás

Así que cargué con mi Pi, mi viejo MacBook, impresora multifunción, enchufes domóticos Z-Wave, un router, cámaras IP, etc. y monté el "tenderete".

A los 5 minutos de empezar la charla ya me di cuenta que **aquel foro no se estaba enterando de nada** y que en el fondo lo que no fuesen pantallas gráficas no tenían cabida allí, por mucho que algunos se pusieran el título de "cacharreros" ([geek](https://es.wikipedia.org/wiki/Geek) que queda mejor... fontanero-geek, chapucero-geek, telepizzero-geek...), pocos eran los que cazaba algo, y muchos los que ya estaban sacando sus iPads y empezaban a garabatear dibujos...

Está claro que **lo hice todo mal desde el principio**, y lo único con lo que hacerte fué con la publicación durante la noche anterior en el foro privado de la asociación una serie de mensajes con todo lo que pretendía contar.

Lógicamente esto **a día de hoy no vale para mucho**, en 6 años hay nuevas versiones de todo (XBMC ya es Kodi, y de AirPlay te puedes despedir...), pero recordad que el motivo de existir de este blog es que pueda guardar mis cosas.

Aquí iré actualizando un índice de las distintas partes de este grupo de mensajes por si queréis ir saltando a alguna parte que os interese más.

- [Introducción](rpi_intro.md)
- [Instalación básica (I)](rpi_install_i.md)
- [Instalación básica (II)](rpi_install_ii.md)
- [Securización](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014-parte-4-securizacion)
- [Copias de Seguridad](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014-parte-5-copiasseguridad/)
- [Práctico 1 - Servidor de WordPress (I)](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014-parte-6-practico1-wordpress1/)
- [Práctico 1 - Servidor de WordPress (II)](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014-parte-7-practico1-wordpress2/)
- [Práctico 2 - Gestor de Descargas](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-8-practico-2-gestor-de-descargas/)
- [Práctico 3 - Servidor Impresión/AirPrint](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-9-practico-3-servidor-airprint/)
- [Práctico 4 - Servidor Copias de Seguridad](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-10-practico-4-servidor-copias-de-seguridad/)
- [Práctico 5 - Domótica, Cámaras IP y RaZberry](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-11-practico-5-domotica/)
- [Práctico 6 - Media Center](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014-parte-12-practico-6-media-center/)
- [Práctico 7 - Emulación de juegos](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-13-practico-7-emulador-de-videojuegos/)
- [Alternativas a RPi](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-14-alternativas-a-raspberrypi/)
- [Apéndices (I) - Reinicio de router](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-15-apendice-i/)
- [Apéndices (II) - XBMC con MariaDB para BBDD distribuida](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-16-apendice-ii/)
- [Apéndices (III) - Administración con Webmin](http://pruebadeconcepto.es/instalando-una-raspberrypi-todo-uso-en-2014/instalando-una-raspberrypi-todo-uso-en-2014-parte-17-apendice-iii/)
