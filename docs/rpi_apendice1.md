---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 15) – Apéndice I"
date: "2020-02-08"
---

## Reiniciar nuestro router

Me ha pasado alguna vez que me encuentro el router de casa tostado.

Da igual el modelo y el tipo de conectividad porqué me ha pasado siempre, con ADSL, con Fibra, routers buenos y no tan buenos, etc.

Todos se bloquean tarde o temprano, porqué pierden el sincronismo con la línea, porqué se les llena la tabla de NAT... mil cosas.

Os cuento como podemos reiniciar el router usando para ello nuestra RPI, claro está, el router tiene que estar sin conexión al exterior pero debe seguir sirviendo la LAN por ethernet, de otra manera no tenemos nada ya que el router estaría en un estado catatónico y para eso no existe otra solución que la intervención humana y apagar/encender.

Todos los routers, o la mayoría, tiene habilitado el protocolo para poder realizar conexiones por [TELNET](http://es.wikipedia.org/wiki/Telnet) , **son conexiones no segueras ya que la información viaja en "plano",** no hay capa de cifrado.

Pero no pasa nada (o casi nada) ya que esta conexión sólo está disponible en la red interna, en nuestra LAN, no es posible acceder desde fuera.

Lo primero que tenemos que hacer es, por un lado mirar si este protocolo está habilitado en nuestro router, y en segundo lugar mirar cual es el comando de reinicio del mismo, que suele ser un comando al estilo `reboot`

Sería algo más o menos como esto:

[root@Jarvis ~]# telnet 192.168.1.1

Trying 192.168.1.1...
Connected to 192.168.1.1.
Escape character is '^]'.
BCM963268 Broadband Router
Login: USUARIOADMINISTRADOR
Password: CLAVEADMINISTRADOR
> reboot

Como véis, en mi router, un VG8050 de FTTH el comando de reinicio es reboot.

Para adivinar el vuestro, cuando el router os de línea de comando, tendréis que teclear help y mirar los comandos disponibles, no será difícil, reboot, restart o similar.

Bien, pues ahora la parte que hacemos en la RPI, vamos a implementar un mecanismo por el que simulamos esta interacción con el router, de manera que lanzamos el comado de Telnet, y luego vamos esperando cadenas de texto y enviamos más información según se nos pide.

Para esto utilizamos una maravilla llamada [Expect](http://en.wikipedia.org/wiki/Expect) , que nos permite diseñar un script que haga esto que precisamente os cuento.

Lo instalamos como es habitual, con `pacman`

[root@Jarvis ~]# pacman -S expect

resolving dependencies...
looking for inter-conflicts...
Packages (2): tcl-8.6.1-1 expect-5.45-3
Total Download Size: 2.31 MiB
Total Installed Size: 6.54 MiB
:: Proceed with installation? [Y/n] Y
:: Retrieving packages ...
tcl-8.6.1-1-armv6h 2.1 MiB 2.16M/s       00:01 [###############################] 100%
expect-5.45-3-armv6h 162.8 KiB 465K/s    00:00 [###############################] 100%
(2/2) checking keys in keyring                 [################################] 100%
(2/2) checking package integrity               [################################] 100%
(2/2) loading package files                    [################################] 100%
(2/2) checking for file conflicts              [################################] 100%
(2/2) checking available disk space            [################################] 100%
(1/2) installing tcl                           [################################] 100%
(2/2) installing expect                        [################################] 100%

El script, al que yo he llamado `RebotaRouter.sh`tiene este contenido:

#!/usr/bin/expect
spawn telnet 192.168.1.1
expect "Login:"
send USUARIOADMINISTRADOR
send "\\r"
expect "Password:"
send CLAVEADMINISTRADOR
send "\\r"
expect -exact " > "
send "reboot"
send "\\r"
sleep 1
interact
exit

Tenemos que prestar atención a la IP de nuestro router (en mi caso 192.68.1.1), el usuario y clave que usamos para administrar el router por http, y el comando de reinicio (reboot en mi caso).

El código es bastate legible, espera esta cadena (expect) y manda esta otra (send).

Entre una y otra establece un retardo de 1 segunto (sleep X donde X es el número de segundos a esperar).

Tenemos que dar permisos de ejecución a este script

[root@Jarvis ~]# chmod 700 RebotaRouter.sh

El permiso es 700, que es -rwx------, esto es, todos los permisos para el propietario y nada para otros y grupo, más que nada por que en ese fichero hemos puesto las credenciales de nuestro router al completo.

Salida de esta ejecución:

[root@Jarvis ~]# ./rebota_router.sh

spawn telnet 192.168.1.1
Trying 192.168.1.1...
Connected to 192.168.1.1.
Escape character is '^]'.

BCM963268 Broadband Router
Login: USUARIOADMINISTRADOR
Password:
> reboot

The system shell is being reset. Please wait...
> Connection closed by foreign host.

Y ya estaría, el router se reinicia, lo primero es probar que esto es así y que os funciona esta primera parte.

## Validamos nuestra conexión

Ahora vamos a hacer otro script, el cual valida que tenemos conexión a internet, y para esto lo mejor es descargaralgo.

Para la descarga empleamos `wget` y nos bajamos una página que sabemos que existe, activamos el parámetro de reintentos a 5, que por defecto son 20 (muchos) y comprobamos el resultado de este comando.

Los comandos en Unx/Lnx siempre dan un retorno, `0` si todo va bien y `>0` si hay algún problema.

Con la variable de entorno `$?` obtenemos el código de retorno del último comando que hemos ejecutado.

Sin más rollos, este es el script `RevisaRouter.sh`

#!/usr/bin/bash
# Comprobacion del estado de la conexion a internet / router tostado.

wget —tries=5 http://www.yahoo.com > /dev/null 2>&1
if ! [ "$?" -eq 0 ]
then
 # No hay conexion a internet.
 # Reiniciamos el router
 /root/RebotaRouter.sh
 sleep 90 # esperamos 90 segundos al reinicio
 echo "Envio Router Comtrend " > /root/router.txt
 echo "Fecha: " \`date '+%d/%m/%Y %H:%M:%S'\` >> /root/router.txt
 echo "Reinicio de router" >> /root/router.txt
 mutt -s "Router" tumail@gmail.com < /root/router.txt
fi
rm -f /root/index.html > /dev/null 2>&1

Como véis he puesto también una llamada a `mutt` que ya teníamos configurado de haberlo usado antes, para que nos notifique por mail que se ha producido un reinicio de la conexión.

Le damos permisos como antes:

[root@Jarvis ~]# chmod 755 RevisaRouter.sh

Para probar que esto funciona, soltáis el cable de teléfono del router (ADSL) o apagáis la ONT (FTTH) y ejecutáis el scrip, las luces del router os indicarán que el reinicio se está produciendo.

Si queréis programarlo para que se ejecute cada cierto tiempo lo podéis incluir dentro de crontab

[root@Jarvis ~]# crontab -e

0,10,20,30,40,50 * * * * /root/RevisaRouter.sh

Con esa línea nueva en crontab el script se ejecutaría siempre cada 10 minutos.
