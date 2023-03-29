---
title: "Chuleta 00 - Sumar con AWK"
date: "2019-05-23"
categories: 
  - "chuletas"
tags: 
  - "awk"
---

Inauguramos sección donde iré poniendo comandos más o menos útiles a modo de chuleta. No os penséis que es por vosotros.. es por mí, para que no se me olviden, que ya tengo una edad.

Vamos a contar (sumar) el resultado de un comando con **AWK**, esto es, por ejemplo, los tamaños de MaxMetaspaceSize de las múltiples javas que corren en una máquina.

Empezamos, lo que queremos sumar _\-XX:MaxMetaspaceSize=100m_ ocupa la posición _15_, y viene acompañado por una _m_ a la derecha (megas).

ps -ef | grep java | awk '{print $15}' | cut -d= -f2 | sed 's/m//'

Así que a esta salida de comando añadimos nuestro sumatorio awk:

ps -ef | grep java | awk '{print $15}' | cut -d= -f2 | sed 's/m//' | awk '{total += $NF} END { print total }

¿Qué es lo que hace este churro?

_ps -ef_ nos lista todos los procesos del servidor en modo extendido.

_grep java_ nos filta los procesos que contienen java (mmm nota mental.. afinar más)

_awk '{print $15}'_ nos corta la lista anterior por el campo 15, que es justo ese MaxMetaspaceSize que queremos

_cut -d= -f2_ nos corta la salida usando el = como separador, y nos da la parte de la derecha, que termina en una m (mmm otra nota mental.. ¿y si mezcla m's con g's con k's?? )

_sed 's/m//'_ quita esa m del final y la reemplaza por nada

_awk '{total += $NF} END { print total }_ y esta la que vale, interpreta línea a linea el resultado y suma con ese += los valores de cada línea que se acumulan en la variable Built-IN $NF, que contiene únicamente la última columna de cada registro, nos imprime el total acumulado.

No puedo cerrar esta primera entrada sin recomendar esta lectura necesaria para todos los enamorados de la línea de comandos: la lectura de este [ensayo sobre la línea de comandos](https://es.wikipedia.org/wiki/En_el_principio_fue_la_l%C3%ADnea_de_comandos).
