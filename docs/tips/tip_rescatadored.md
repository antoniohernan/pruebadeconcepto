---
title: "Chuleta 02 - Rescantando de la red"
date: "2019-06-13"
categories: 
  - "chuletas"
tags: 
  - "mirror"
  - "perl"
  - "scanner"
---

En esta entrada os traigo unas cuantas cosillas que tenía **publicadas por otros blogs**.

**Empezamos por un scaner de puertos abiertos** en nuestra conexión a internet.

[Porquiz.net](http://portquiz.net) es un equipo que está escuchando por **TODOS los puertos** (salvo por el 445, que según su administrador no consigue mapear por restricciones de su ISP).

Con este sencillo comando nmap **podemos ver que puertos de salida tenemos abiertos en nuestra infraestructura** (aka, los puertos que nos permiten salir en nuestro proxy/firewall):

operador@VASS-TONIO:~$ nmap -p 1-65535 portquiz.net

Starting Nmap 7.01 ( https://nmap.org ) at 2017-10-03 08:25 CEST
Nmap scan report for portquiz.net (178.33.250.62)
Host is up (0.0049s latency).
rDNS record for 178.33.250.62: electron.positon.org
Not shown: 65518 filtered ports
PORT STATE SERVICE
80/tcp open http
110/tcp open pop3
143/tcp open imap
389/tcp open ldap
443/tcp open https
1723/tcp open pptp
1883/tcp open unknown
3389/tcp open ms-wbt-server
3998/tcp open dnx
5938/tcp open teamviewer
7777/tcp open cbt
8443/tcp open https-alt
10038/tcp open unknown
10039/tcp open unknown
10040/tcp open unknown
10041/tcp open unknown
43440/tcp open unknown

Nmap done: 1 IP address (1 host up) scanned in 104.11 seconds

Es muy util para ver que restricciones de salida tenemos en el trabajo... para conectarnos según que servicios.

**Seguimos con la forma de cambiar editor** de /etc/sudoers Ubuntu

Alguien ha decidido que lo mejor para editar el fichero /etc/sudoers en Ubuntu es la basura infecta del editor nano… Es de risa.. invocas el comando visudo porque se supone que quieres editar con VI el fichero sudoers de sudo.. (vi sudo) y van y te cargan Nano... me entra una mala leche.

Para **solventar esta terrible decisión que alguien de Canonical que debería estar vendiendo microondas en un WallMart** podemos hacer esto:

operador@VASS-TONIO:~$ sudo update-alternatives --config editor \[sudo\] password for operador:

operador@VASS-TONIO:~$ sudo update-alternatives --config editor
\[sudo\] password for operador:

Existen 4 opciones para la alternativa editor (que provee /usr/bin/editor).

Selección Ruta Prioridad Estado
------------------------------------------------------------
\* 0 /bin/nano 40 modo automático
1 /bin/ed -100 modo manual
2 /bin/nano 40 modo manual
3 /usr/bin/vim.basic 30 modo manual
4 /usr/bin/vim.tiny 10 modo manual

Press <enter> to keep the current choice\[\*\], or type selection number: 3

Y usar [VI](https://es.wikipedia.org/wiki/Vi), como debe ser, para editar ficheros.

(\* _notamental_… tan sencillo como que la segunda línea de la instalación de una máquina sea eliminar ese editor..)

**Por último, vamos a ver como sacar el hilo de ejecición de una JVM que más consume**

Los pasos para hacerlo "a mano" son estos:

1. Lanzamos un top y pulsamos May+h
2. Con esto veremos en el top la lista de threads que mas consumen
3. En paralelo vamos lanzando un kill -3 al proceso de java que esta consumiendo mucha cpu
4. Del top sacamos el pid que esta consumiendo mas cpu
5. Convertimos el PID a hexadecimal
6. Buscamos en el thread dump nid=0x\[PID en exadecimal\]

Yo en su momento lo pase a una utilidad para ir desplegando en todas las máquinas, aquí te lo pongo de regalo:

#!/usr/bin/perl

# Solo para usuario (ps -f en vez de ps -ef)
$PROG=$0; chomp($PROG);
$CUANTAS=\`ps -f | grep ${PROG} | egrep -v 'grep' | wc -l 2>/dev/null\`; chomp($CUANTAS);
( $CUANTAS ) or $CUANTAS=0;
( $CUANTAS > 1 ) and die ("\\n\\n Existe un proceso en ejecucion de ${PROG}\\n\\n");
undef $CUANTAS;

# Variables de entorno
$JSTACKCMD='/usr/java/jdk1.7.0\_40/bin/jstack'; chomp($JSTACKCMD);

# Variables por parametros
(@ARGV\[0\]) and $PID=@ARGV\[0\]; chomp($PID);
(@ARGV\[0\]) or $PID=\`ps -ef | grep java | grep liferay | grep -v grep | awk '{print \\$2}'\`; chomp($PID);

$ESJAVA=\`ps -p ${PID} | tail -1 | grep java | wc -l\`; chomp($ESJAVA);
($ESJAVA) or die("\\n\\n El PID: ${PID} no es un proceso java\\n\\n");

# Hilos de ese Java
@TOP=\`top -bH -n 1 -p ${PID}\`;

# El hilo que más consume
$MASTOP=@TOP\[7\]; chomp($MASTOP);
($DECPID,$resto,$resto,$resto,$resto,$resto,$resto,$resto,$CPU) = split(" ",$MASTOP,10);

$HEXPID= sprintf("0x%X",$DECPID);
$CADHEXPID="nid=".lc($HEXPID);

@THREADDUMP=\`${JSTACKCMD} ${PID}\`;

$IMPRIME=0;

print "\\n\\n El Hilo que mas consume (${CADHEXPID}) del proceso ${PID} (${CPU} %CPU):\\n\\n";

foreach(@THREADDUMP) {
$LIN=$\_; chomp($LIN);
($LIN =~ $CADHEXPID) and $IMPRIME=1;
($IMPRIME > 0 ) and print "${LIN}\\n";
($LIN =~ /^$/) and $IMPRIME=0;
}

... continuará ...
