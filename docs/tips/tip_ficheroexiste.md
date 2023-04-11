---
title: "Chuleta 06 - ¿El fichero existe?"
date: "2021-06-25"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
categories: 
  - "chuletas"
---

Cuando empiezas a tirar código Shell con cierta soltura empiezas a meter validaciones digamos que básicas, ¿existe el fichero de configuración de este proceso?, ¿y la ruta del fichero de log que voy a grabar?, ¿el fichero de datos a procesar existe? ¿lo puedo leer? ¿y además tiene datos o está vacio?

A todas estas preguntas y a bastantes más podemos dar respuesta con un simple **IF** y la correspondiente **evaluación del fichero/directorio**.

Utilizaremos la expresión de test, la cual podemos expresar con estas  tres sintaxis:
```
test EXPRESION
```
```
[ EXPRESION ] 
```
```
[[ EXPRESION ]]
```
Por cuestiones de portabilidad, si quieres que el script que vas a hacer sea utilizable en el mayor número de sistemas posible, te recomiendo que uses la sintaxis de "test" pues parece estar disponible en todas las shelll basada en norma [POSIX](https://es.wikipedia.org/wiki/POSIX).

La primera comprobación, la más básica que podemos hacer es comprobar si el elemento existe, sea un fichero, un directorio o un enlace, para essto emplearemos la opción **\-e**
```
ahernan:/tmp$ FIC=/tmp/pp
ahernan:/tmp$ if test -e "$FIC"; then echo "$FIC existe"; fi
/tmp/pp existe
```
Si queremos afinar un poco más y preguntar si además de existir, sí se trata de un fichero, empleraremos la opción **\-f**
```
ahernan:/tmp$ FIC=/tmp/pp
ahernan:/tmp$ if test -f "$FIC"; then echo "$FIC existe y es un fichero regular"; fi
/tmp/pp existe y es un fichero regular
```
Utilizando los otros formataos de expresar el "test", serían algo así como esto que os pongo a continuación, donde se ejecuta la parte a la derecha de los `&&` cuando el resultado de la evaluación del test es verdadero, o a la derecha de los `||` cuando la evaluación del test es falso.
```
ahernan:/tmp$ FIC=/tmp/pp
ahernan:/tmp$ [[ -f $FIC ]] && echo "$FIC existe y es un fichero regular" || echo "$FIC no existe o no es un fichero regular"
/tmp/pp existe y es un fichero regular
```
Como veis, la potencia del comando reside en el parámetro que utilicemos. A continuación os enumero bastantes de sus opciones, no todas, pero si las que me parecen más útiles:

Tipo del descriptor:

<table style="border-collapse: collapse; width: 100%;"><tbody><tr><td style="width: 4.47838%;">-b</td><td style="width: 95.5216%;">Devuelve TRUE si existe y se trata de un Block device o special block file</td></tr><tr><td style="width: 4.47838%;">-c</td><td style="width: 95.5216%;">Devuelve TRUE si exsite y se trata de un fichero de caracteres</td></tr><tr><td style="width: 4.47838%;">-d</td><td style="width: 95.5216%;">Devuelve TRUE si existe y es un directorio</td></tr><tr><td style="width: 4.47838%;">-f</td><td style="width: 95.5216%;">Devuelve TRUE si existe y es un fichero regular.</td></tr><tr><td style="width: 4.47838%;">-S</td><td style="width: 95.5216%;">Devuelve TRUE si existe y es un socket.</td></tr><tr><td style="width: 4.47838%;">-p</td><td style="width: 95.5216%;">Devuelve TRUE si existe y es un pipe</td></tr><tr><td style="width: 4.47838%;">-L</td><td style="width: 95.5216%;">Devuelve TRUE si existe y es un enlace simbólico</td></tr><tr><td style="width: 4.47838%;">-h</td><td style="width: 95.5216%;">Devuelve TRUE si existe y es un enlace simbólico</td></tr></tbody></table>

Permisos del descriptor:

<table style="border-collapse: collapse; width: 100%; height: 116px;"><tbody><tr style="height: 23px;"><td style="width: 5.73013%; height: 23px;">-u</td><td style="width: 148.224%; height: 23px;">Devuelve TRUE si existe y tiene el flag de set-user-id (suid) activo</td></tr><tr style="height: 23px;"><td style="width: 5.73013%; height: 23px;">-g</td><td style="width: 148.224%; height: 23px;">Devuelve TRUE si existe y tiene el flag de set-group-id (sgid) activo</td></tr><tr style="height: 23px;"><td style="width: 5.73013%; height: 23px;">-k</td><td style="width: 148.224%; height: 23px;">Devuelve TRUE si existe y activo el flag de sticky bit</td></tr><tr style="height: 24px;"><td style="width: 5.73013%; height: 22px;">-w</td><td style="width: 148.224%; height: 22px;">Devuelve TRUE si existe y es escribible</td></tr><tr style="height: 24px;"><td style="width: 5.73013%; height: 24px;">-x</td><td style="width: 148.224%; height: 24px;">Devuelve TRUE si existe y es ejecutable</td></tr><tr style="height: 24px;"><td style="width: 5.73013%; height: 24px;">-r</td><td style="width: 148.224%; height: 24px;">Devuelve TRUE si existe y es leible</td></tr></tbody></table>

Otros acerca del descriptor:

<table style="border-collapse: collapse; width: 100%;"><tbody><tr><td>-s</td><td>Devuelve TRUE si existe y tienen tamaño distinto de 0</td></tr><tr><td>-G</td><td>Devuelve TRUE si existe y el descriptor tiene el mismo grupo que el usuario que ejecuta el comando</td></tr><tr><td>-O</td><td>Devuelve TRUE si exsite y el descriptor tiene el mismo propietario que el usuario que ejecuta el comando</td></tr></tbody></table>
