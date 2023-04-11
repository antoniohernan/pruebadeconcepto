---
title: "Chuleta 04 - Trabajando con el segundo plano"
date: "2019-10-29"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
categories: 
  - "chuletas"
tags: 
  - "linux"
  - "multitarea"
  - "segundo-plano"
  - "unix"
---

Esto **no es nada nuevo**, existe desde que Unix/Linux es Unix/Linux, aunque hoy en día **esté quizás en desuso**. Os recuerdo el fin de este blog/diario, recordarme a mi mismo como se hacían las cosas...

Si necesitamos abrir más "ventanas" de trabajo (_Windows y trabajo, que caprichoso oximorón_) recurrimos a abrir más terminales o conexiones al sistema y listo, pero no queda tan lejos **la época en la que conseguir un terminal al servidor no era tan sencillo**, el número de terminales por usuario estaba limitado (incluso existían los terminales físicos, cable directo al servidor ahí es nada) o bien la conexión era tan lenta que mantener un terminal abierto era todo un logro, como para abrir más de uno. (algún día os contaré en [informática viejuna](http://pruebadeconcepto.es/category/informaticaviejuna/) como conectábamos por una línea "satelital" desde Madrid a Bogotá, pasando por París y medio Estados Unidos, con un lag entre pulsación de tecla y el pintado del carácter en pantalla de 1 segundo mínimo).

Llega el momento de trabajar con varias cosas a la vez, **un único terminal** y te ves obligado a hacer multitarea de la buena.

Imaginad... editando un fichero de virtual host para nuestro apache, y a mitad de edición no recuerdas el nombre exacto de la máquina backend sobre la que hacer el proxy, si era demoesto o estodemo.

Sería algo similar a esto:
```
operador@VASS-TONIO:/tmp$ vi virtualhost\_demo.es.conf 

...
  SSLProxyCheckPeerExpire off
  ProxyPass / "http://demoint.es:8080/" connectiontimeout=5 retry=1 acquire=3000 timeout=600 Keepalive=On
  ProxyPassReverse / "http://demoint.es:8080/"
</VirtualHost>
...
```
En este punto, sin salir del editor pulsamos la combinación de teclas **CTRL + Z** lo que causa que el actual proceso, nuestro editor VI se **quede detenido**.
```
"virtualhost\_demo.es.conf" 33L, 939C                                                                                                                              

^Z

[1]+  Detenido                vi virtualhost\_demo.es.conf
```
Hemos recuperado la shell, tenemos un proceso detenido y podemos consultar los procesos que tenemos en nuestra sesión mediante el comando **jobs**
```
operador@VASS-TONIO:/tmp$ jobs -l
[1]+  9515 Parado                  vi virtualhost\_demo.es.conf
```
Ahora es donde entran en juego dos comandos más son **fg** y **bg**

**fg** lo utilizaremos para traer procesos que están en segundo plano o background al primer plano o foreground. Si **solo tenemos un job** y ejecutamos fg nos traerá de nuevo ese proceso al primer plano. Si **tenemos más de un proceso** debemos indicar el número de job que queremos traer al primer plano, este número de job **se obtiene de la salida del comando jobs** antes descrito.

**bg** lo utilizaremos para poner un trabajo **detenido** a correr en segundo plano, en este caso en el que tenemos un VI corriendo la ejecución de bg no tiene mucho sentido pues el proceso requiere salida a terminal
```
operador@VASS-TONIO:/tmp$ bg
[1]+ vi virtualhost_demo.es.conf &

[1]+  Detenido                vi virtualhost_demo.es.conf
operador@VASS-TONIO:/tmp$ jobs -l
[1]+  9515 Parado (requiere salida por terminal)             vi virtualhost_demo.es.conf
```
Por tanto, para la tarea anterior que os he comentado, estar editando tal fichero y salir de la edición para hacer otra cosa estaríamos en el punto de hacer esa otra cosa y **"rescatar" nuestra edición** del segundo plano, ejecutando **fg**  (ó **fg #job** si tenemos más de uno)

¿Realmente **cuando ponemos un job en segundo plano a correr se sigue ejecutando**?

Pues si, mirad este pequeño script, que muestra un contador, actualizado cada segundo:
```
operador@VASS-TONIO:~/Scripts/Bash$ cat testclock.sh 
#!/bin/bash
for ii in \`seq 1 1000000\`;
  do
    echo "Valor Secuencial: ${ii}"
    sleep 1
  done
```
Lo ejecutamos y pasados unos segundos lo detenemos con **CTRL + Z**:
```
operador@VASS-TONIO:~/Scripts/Bash$ ./testclock.sh > testclock.txt
^Z
[1]+  Detenido                ./testclock.sh > testclock.txt
```
El proceso ha llegado hasta este valor de cuenta, y no avanza más, **se encuentra detenido**/suspendido.
```
operador@VASS-TONIO:~/Scripts/Bash$ tail -1 testclock.txt 
Valor Secuencial: 15
```
Ahora **lo mandamos a segundo plano**, y observamos como **sigue corriendo generando líneas** en el fichero de salida:
```
operador@VASS-TONIO:~/Scripts/Bash$ jobs -l 
[1]+ 28244 Parado                  ./testclock.sh > testclock.txt
operador@VASS-TONIO:~/Scripts/Bash$ bg 1
[1]+ ./testclock.sh > testclock.txt &
operador@VASS-TONIO:~/Scripts/Bash$ tail -f testclock.txt 
Valor Secuencial: 24
Valor Secuencial: 25
Valor Secuencial: 26
Valor Secuencial: 27
Valor Secuencial: 28
Valor Secuencial: 29
Valor Secuencial: 30
Valor Secuencial: 31
Valor Secuencial: 32
Valor Secuencial: 33
Valor Secuencial: 34
Valor Secuencial: 35
Valor Secuencial: 36
```
Si queremos **eliminar el proceso que está corriendo** en segundo plano, o que está detenido, podemos hacer uso del comando kill indicando el job-id
```
operador@VASS-TONIO:~/Scripts/Bash$ jobs -r
[1]+  Ejecutando              ./testclock.sh > testclock.txt &
operador@VASS-TONIO:~/Scripts/Bash$ kill %1
operador@VASS-TONIO:~/Scripts/Bash$ jobs -l
[1]+ 28244 Terminado               ./testclock.sh > testclock.txt
```
Con jobs y el parámetro -r vemos las tareas que tenemos corriendo.

Con `kill %1` detenemos el job-id número 1.

Por ultimo, al listar de nuevo todos los jobs con el parámetros -l vemos que el proceso que hemos matado figura como "Terminado".

Y esto es todo por ahora, no es gran cosa, es algo muy básico de toda la vida, y que con la facilidad de abrir múltiples terminales y calidad de las conexiones ha dejado relegada a, nunca mejor dicho, un segundo plano estas forma de trabajar.
