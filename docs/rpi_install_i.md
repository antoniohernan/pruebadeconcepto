---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 2) - Instalación básica (I)"
date: "2019-12-16"
---

## ¿Por donde empiezo?

Lo primero que deberías pensar es **¿que voy a hacer con esto?**. No es lo mismo tratar de montar un servidor de WordPress, que un centro multimedia, que un gestor de descargas u otra cosa. También tienes que pensar en la cantidad de tiempo que le quieres dedicar y lo que quieres profundizar en la tarea. Lo mismo que antes, no es lo mismo querer poner una imagen de las del tipo "todo hecho" y no perder más que 5 minutos configurando, que hacer una instalación desde base e ir creando tu, tú propio entorno. Si quieres, para empezar puedes probar las distribuciones ya hechas y ver que por lo menos esto funciona, y luego ya puedes seguir leyendo el resto donde usaré una distribución algo más compleja.

## ¿Alternativas?

Un montón!!!, y cada día más. Tienes distribuciones basadas en BSD, alguna versión de Android y otra muchas basadas en diferentes distribuciones de Linux, como son Debian, Arch Linux y Fedora (algo me dejo seguro...) No busques la famosa Ubuntu por que como tal no existe, Cannonical hace tiempo que estableció su mínimo de hardware para las distribuciones de Ubuntu basadas en [procesadores ARM](https://wiki.ubuntu.com/ARM) en los micros ARM v7, nuestra Pi es un ARM v6, no hay mucho que hacer.

Cada distribución tiene sus pros y sus contras, e incluso con el tiempo **te vuelves un poco "fan" de alguna** en concreto que seguro que no es la mejor en su campo, pero que es con la que mejor te  desenvuelves. En mi caso, y la que vais a ver a lo largo de todo este texto es [ARCH Linux](http://archlinuxarm.org/platforms/armv6/raspberry-pi), en su distribución para microprocesadores ARM (el de nuestra Pi, un v6). Para la descarga de la imagen ISO original, nada mejor que la [página de la Fundación](http://www.raspberrypi.org/downloads).

## ¿Y eso de NOOBS?

Pues esto es relativamente reciente, de Septiembre-Octubre de 2013. (**recordad**... esto se escribió en 2014!!). Consiste en que te descargas una imagen muy pequeña, y una vez puesta en la Pi, conectada a red y conectada a un teclado y video por HDMI, arranca y se empieza a descargar la imagen definitiva. Algo así como los ejecutables de instalación que se emplean en Windows (argggg), que ocupa muy poco y que cuando los ejecutas te das cuenta que se pone a descargar el software real y se pega la tira de tiempo descargando y venga a bajar megas y megas. También tienes la opción de utilizar una imagen NOOB que tiene las distribuciones casi completas y no necesita de esta descarga. Es una facilidad, muy acertada, que nos dan ahora por si no quieres complicaciones de ningún tipo. No es que no recomiende su uso, pero a mi estas cosas no me gustan, las cosas que vienen ya hechas o que te lo hacen todo ellos y yo no somos muy amigos. No digo que no funcione, por que lo he probado y funciona bastante bien, pero así no aprendes nada.

## ¿Entonces ARCH Linux?¿Por qué?

Bien, ARCH es una distribución que con su sistema de liberación de paquetes me tiene encantado, es lo que se llama un [Rolling Release](https://es.wikipedia.org/wiki/Liberación_continua), esto es, según sale un nuevo parche para un componente este se libera y te lo individualmente. No tienes por tanto que aguantarte con un problema en un componente como pasa con otras distribuciones, llamadas Point Release, que sacan versiones estables cada cierto tiempo (en el caso de Debia hablamos de años...). Por otro lado, esas versiones estables suelen implicar la reinstalación completa de la tarjeta, mientras que con una rolling release instalas la base y luego a base de actualizaciones te mantienes al día, bueno, casi, esto es como diría un físico teórico, en condiciones ideales de temperatura, presión y en vacío... Tampoco penséis que ARCH es un mundo ideal, son famosas sus meteduras de pata, como cuando dieron el paso definitivo para abandonar Init de BSD por systemd, o cuando cambiaron en verano de 2013 el punto de montaje de binarios... y más recientemente con los cambios en la distribución de las particiones del disco (tarjeta SD más bien). En cualquiera de esos casos, si no andas un poco pendiente te puedes encontrar con un sistema inútil que no arranca. Pero tranquilos, que el **apartado de backup / restauración** está un poco más abajo para que durmáis mejor.

## Previos a la instalación

Nos descargamos la última imagen, desde [aquí](http://downloads.raspberrypi.org/arch_latest) a nuestro equipo. Tenemos nuestra Pi desconectada del adaptador de corriente, conectada a nuestro router por un cable ethernet, y nada más, no necesitamos video por el momento, ni siquiera teclado ratón, vamos a hacerlo todo por [SSH](http://es.wikipedia.org/wiki/Secure_Shell)

Vas a necesitar acceso a la URL de administración de tu router, por que necesitas saber que dirección IP ha asignado tu DHCP a la MAC de la Pi, eso si no eres como yo, que tengo muy restringida la asignación de IPs en mi DHCP y soy de los que asigna las IPs a mano, para tal MAC tal IP y ni una más. Necesitas también un lector de tarjetas SD, si es que tu equipo no tiene uno integrado, como pasa en muchos portátiles Mac y en los iMac. Yo he usado normalmente un adaptador USB que es como un pendrive, al que se le coloca la SD en un extremo, es quizás algo más lento que un lector de tarjetas en condiciones, pero es mucho más cómodo de usar por sus dimensiones. Y necesitas un software para pasar esa imagen Iso que te descargas a la tarjeta SD, yo uso [Pi Filler](http://ivanx.com/raspberrypi/), aunque si no lo quieres usar no te preocupes por que ahora te voy a contar como hacerlo **usando el Terminal**, que no dista mucho a como este programilla lo hace.

De este mismo desarrollador tenéis [Pi Copier](http://ivanx.com/raspberrypi/), que te vuelca (y comprime si lo quieres) el contenido de la tarjeta SD a tu disco duro. Una buena opción a estudiar, sobre todo al principio cuando es muy fácil entre prueba y prueba cargarte toda la instalación y tener que volver a empezar.

## Ahora si que sí, empezamos

Vamos a pasar la imagen de nuestro sistema operativo, descargado en nuestro disco duro a la tarjeta SD. Antes de continuar, descomprime el .zip del fichero de la imagen, que sin eso no hacemos nada. Ahora, **SIN haber puesto la tarjeta SD** en el equipo ejectutamos Pi Filler, escogemos la imagen a instalar y nos indica que debemos introducir la tarjeta SD, nos la reconocerá y nos pregunta cual es (sólo te puedes fiar por el nombre, cuidado con esto que puedes borrar algo que no es...), una vez aceptadas las pantallas se pone a volcar la imagen en la tarjeta y después de un buen rato, mínimo media hora, ya estará lista para su uso.

Si quieres hacer este mismo proceso por ti mismo, con el terminal de OSX los pasos a dar son estos:

- En primer lugar, abre el terminal, bien por que lo ejecutas a partir de tus aplicaciones o bien por que lo inicias desde una búsqueda de Spotligth (CMD + Espacio, teclear terminal y aceptar con intro)
- Conectas la tarjeta SD al sistema.
- Identificamos cual es la tarjeta SD que hemos puesto

```
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *1.0 TB     disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                 Apple_APFS Container disk1         1000.0 GB  disk0s2

/dev/disk1
#: TYPE NAME
0: FDisk_partition_scheme *32.0 GB
1: DOS_FAT 32.0 GB
```

- Como véis, en mi caso mi SD es el disco que está en ```/dev/disk1```. El tamaño de la tarjeta SD es una muy buena pista para identificarla.
- Ahora bien, por si tenéis alguna duda, ejecuntado este comando podréis ver donde está montada la tarjeta en el sistema de ficheros:

```
df -kh
Filesystem Size Used Avail Capacity Mounted on 
/dev/disk0s2 233Gi 208Gi 25Gi 90% /
...
/dev/disk1s2 32Gi 17Mi 31Mi 1% /Volumes/Untitled
```

Todas las unidades externas en OSX se montan en ```/Volumes```, y ese Untitled que aparece ahí es el nombre de nuestra tarjeta, que seguramente si está nueva sea ese. El disco es ```/dev/disk1``` y ```s2``` es el número de la partición del disco (la s es de slice), por tanto nuestro disco es ```/dev/disk1```, de la partición nos olvidamos por que al volcar la imagen de la distribución se crea su propia tabla de particiones (que luego vamos a modificar, por supuesto).

- Ahora tenemos que desmontar la tarjeta SD, pero no expulsarla!!. Esto es, vamos a ver cómo desaparece la unidad de nuestro escritorio, pero el dispositivo sigue conectado al equipo.
```
sudo diskutil unmountDisk /dev/disk1
```

**sudo** es el comando para la ejecución de comandos bajo otro usuario, su es de switch user y do de hacer, esto es, hacer con otro usuario. Aquí en este comando que os he puesto es ejecutar con el usuario administrador el comando diskutil. Os va a pedir una clave antes de continuar, es la clave del usuario administrador de vuestro equipo.

- El comando para grabar la imagen en ese dispositvo es este:
```
sudo dd bs=1m if=<ficheroimagen.img> of=/dev/disk1
```

El comando **dd** toma su nombre de Dataset Definition, y sirve para la manipulación de la información a bajo nivel, para transferir datos, manipular raw data/devices (si, este RAW significa lo mismo que el RAW de las cámaras de fotos tan de moda últimamente, acceso en "crudo") y realizar algunas conversiones predefinidas. Los parámetros que pasamos al comando son bs=1m, que indica que use bloques de 1m para la lectura y escritura, if que le indica al comando el fichero (con ruta completa si es necesario) de origen, y of que le indica el destino de los datos, en nuestro caso el dispositivo de la tarjeta SD. Debéis reemplazar <ficheroimagen.img> por el nombre y ruta de vuestro fichero de imagen, si por ejemplo lo tenéis en vuestras descargas de usuario tendréis dos opciones para hacerlo.

La primera es cambiar el directorio actual de trabajo del terminal, en los sistemas OSX, aunque en el navegador de carpetas lo veáis todo en castellano, en realidad todo eso "por debajo" no son más que carpetas del sistema operativo BSD, que están en inglés. Vuestra carpeta de descargas de usuario estará en ```~/Downloads``` El símbolo ```~``` (si, el gorrinillo de encima de las Ñ o virgulilla) se corresponde al caracter ASCII 126 y es un carácter reservado que viene a indicar o contener la ruta del directorio HOME del usuario. En los equipos con Windows (arggg) se obtiene manteniendo pulsada la tecla ALT y tecleando 126 en el teclado numérico. En los equipos con OSX se obtiene manteniendo pulsada la tecla ALT y pulsando la tecla Ñ. En mi caso, que mi usuario se llama Nono, mi carpeta de descargas estará en la ruta ```/Users/Nono/Downloads``` ó bien en la ruta ```~/Downloads```.

- Así que, ejecutamos esto para cambiar al directorio
```
cd ~/Downloads
```

El comando **cd** es Change Dir, cambia de directorio, y nos lleva al mismo sitio donde tenemos descargada nuestra imagen, si este fichero se llama, por ejemplo ```archlinux-hf-2013-07-22.img```, el comando dd que tenemos que ejecutar es este:

```
sudo dd bs=1m if=archlinux-hf-2013-07-22.img of=/dev/disk1
```

Si no queremos hacer cambios de directorios y ejecutar el comando dd buscando el fichero de imagen **como ruta absoluta** a nuestras descargas, estemos posicionados donde estemos en el terminal, el comando dd a ejecutar es:

```
sudo dd bs=1m if=~/Downloads/archlinux-hf-2013-07-22.img of=/dev/disk1
```

En OSX tenemos posibilidad de acceder al dispositivo también por un puntero al mismo en modo RAW, que ya que estamos usando dd, que como os he contado es experto en estos desempeños, podemos hacer uso de el.

Hay quien dice que con este tipo de acceso se pueden llegar a tener un 20% más de rendimiento, yo no aprecié tanto, aunque si que se nota algo.. Para esto, en vez de utilizar el **dispositivo /dev/disk1** usamos el de acceso raw **/dev/rdisk1**, esa r ese raw. Por tanto, el comando dd "definitivo" sería este:

```
sudo dd bs=1m if=archlinux-hf-2013-07-22.img of=/dev/rdisk1
```

- Tan sólo resta esperar a la finalización del comando dd y expulsar la tarjeta SD del sistema, para la expulsión el comando es este:
    ```
    sudo diskutil eject /dev/disk1
    ```
    

## Encendemos nuestra Pi por primera vez

Quitamos la SD del equipo y/o adaptador y la ponemos en la ranura de la Pi, conectamos el cable ethernet y le ponemos corriente. La máquina empieza a arrancar, si la tuviésemos conectada por HDMI a un monitor o a nuestra televisión (el arranque de un linux en un plasma de 50" es espectacular XD) veríamos todo el proceso de arranque, carga de drivers, montaje de file system, etc. Tenemos que identificar la dirección IP que nos ha dado el router, será alguna de nuestra red local, algo en plan 192.168.1.XXX

Vamos a realizar una conexión por el protocolo SSH, esto es, una conexión que lleva una capa de cifrado y se comunica al puerto 22 de nuestra Pi, que por defecto en esta distribución viene abierto, y que un poco más adelante podréis ver como securizar convenientemente en caso de que lo consideréis oportuno. Para realizar esta conexión podemos emplear nuestro propio terminal en OSX, pues disponemos de cliente nativo de SSH, o bien podemos emplear algún tipo de software, de los denominados de terminal, que nos aporte algún valor añadido, como un historial de comandos o una gestión de conexiones.

Yo uso habitualmente [iTerm2](http://www.iterm2.com), pero podéis usar el terminal del propio Osx para esto, el comando a ejecutar es este:

```
ssh root@192.168.1.20
```

Una anotación sobre las conexiones SSH, cuando realizamos una conexión por primera vez a un sistema nos suele aparecer un mensaje en el terminal tal que así:

```
The authenticity of host '192.168.1.20 (192.168.1.20)' can't be established.
RSA key fingerprint is 91:2d:b5:60:8a:17:60:4e:62:37:d2:ce:ea:94:7f:65.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.20' (RSA) to the list of known hosts.
```

Tenemos que decir **yes**, que aceptamos que se guarde la huella RSA para ese servidor en nuestra lista de equipos conocidos. Y esta información se guarda en el fichero **known\_hosts** que tenemos en el directorio `.ssh` de nuestro home de usuario `~/.ssh`. **¿Por que os cuento esto?**, pues por que esa IP, nombre de máquina y huella de clave RSA se almacenan ahí donde os digo y en caso de hacer una reinstalación de la tarjeta SD (lo harás tarde o temprano.. al tiempo) esta información cambia. Cuando tratéis de hacer una conexión SSH a la nueva/recién formateada Pi se hará un lio tremendo entre lo antiguo y lo nuevo y veréis como directamente la conexión SSH se cierra. Es más, es posible que os llegue a salir un aviso en pantalla acerca de un posible ataque del tipo “[Man in the Middle](http://es.wikipedia.org/wiki/Ataque_Man-in-the-middle)”, como esto:

```
@@@@@@@@@@@
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
32:e5:d9:77:59:79:f3:83:3f:d6:19:76:b7:00:f4:c1.
Please contact your system administrator.
Add correct host key in /Users/Nono/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/Nono/.ssh/known_hosts:1
RSA host key for 192.168.1.20 has changed and you have requested strict checking.
Host key verification failed.
```

¿Solución?, modificar o eliminar este fichero, si eliminar, no pasa nada, se volverá a generar y llenar de contenido en las siguientes conexiones. Así que, si reinstalas tu Pi y ves que no puedes conectar con ella por SSH, y que esta conexión se cierra al instante 
prueba esto:

```
rm ~/.ssh/known_hosts
```

El comando `rm` toma su nombre de Remove, elimina ese fichero, es un comando PELIGROSO ya que no tiene marcha atrás, **no existe un unrm**... lo que borres borrado está, además este comando puede tomar a través de sus parámetros un montón de opciones que hagan que su comportamiento sea letal, como `-fr` (f = force r=recursive, borra recursivamente sin confirmación), todo un HitParade entre los becarios de administración de sistemas, y que tire la primera piedra el que nunca se le haya escapado uno y la haya liado parda... Si te da miendo usar este comando puedes tratar de mover el fichero, si mover, por que en Unix/Linux no existe un comando que renombre elementos, existe un comando que los mueve.

```
mv ~./ssh/known_hosts ~./ssh/known_hosts.old
```

El comando **mv es de Move**, y lo que hace es que mueve el fichero known_hosts a otro destino, que es el fichero known_hosts.old. Retomamos nuestra instalación, perdón por la dispersión, pero creo que es bueno ir de vez en cuando soltando cosas de estas en plan culturilla general, que son las que más te hacen aprender. Hemos abierto una conexión SSH, hemos autorizado a que se almacene en local la huella de la clave RSA de nuestra PI y nos está pidiendo la clave del usuario root, que es el que hemos configurador en la conexión. ¿Que clave?, la de por defecto, luego la cambiamos, **es root**

## Pantalla negra letras blancas, tu tienes el poder
 
```
[root@alarmpi ~]#
```

Con estos caracteres tu **Pi te da la bienvenida al mundo** del cacharreo. ¿Decepcionado verdad?, espera, que te explico la cantidad de información que te está dando con esa línea aparentemente tan simple. **root** = nos informa del usuario con el que estamos interactuando en esta sesión, en breve vemos como crear más usuarios y esta información te será de utilidad, así sabes que estas ejecutando y con que usuario. **alarmpi** = el nombre de nuestra máquina, así sabes a que máquina te has conectado. ¿Pero si sólo tengo una?, si, tu si, pero yo a día de hoy administro administraba entorno a los 200 servidores, vital saber el cual estás. En breve vemos como cambiar el nombre a nuestra máquina, ante todo personalidad propia!!. **~** = nuestra amiga la virgulilla aparece de nuevo, nos indica que estamos en el directorio HOME de este usuario. Si hacemos referencia en un cd (change directory, cambio de directorio) a ~ nos llevará a este directorio o bien lo empleará para componer la ruta relativa usándolo como base, es lo mismo que emplear la varible de entorno $HOME (las variables se referencia así, con un $ por delante). Esta información es muy util pues en cuanto que cambiemos de directorio aquí nos aparecerá el nombre del directorio (sin ruta) en el que nos encontramos (que sería lo mismo que consultar la variable de entorno $PWD, ó Process Working Directory, o ejecutar el comando pwd que nos da esta misma información). **#** = esto nos indica que tenemos la shell como root, nuevamente en entornos monousuario que sólo tenemos root no nos aporta nada, pero cuando tenemos varios usuarios es importante porque, con usuarios que no son UID 0 (administradores, como nuestro root) el símbolo que veremos es un $.

¿Que diferencia hay con la shell de root y la de otro usuario?, pensad en aquel `-fr` del comando `rm`, y ahora lo juntáis con una shell privilegiada, si... la aniquilación de todo fichero de tu instalación... ¿A que mola tener el poder?? mu ha ha ha... Toda esta información que se nos muestra es completamente personalizable, para ello podemos modificar en nuestro fichero de perfil de shell (`.profile`, `.bash_profile` o en los esqueletos de usuarios para hacerlo general a todos los nuevos usuarios) el valor de las variables de entorno `$PS1` y `$PS2`, las llamadas Prompt String.

No quiero hacer esta guía interminable así que voy a omitir estas configuraciones, si alguien está interesado en ello que me lo pregunte, que con mucho gusto le cuento como modificarlo.

## Nos ponemos al día

Antes de empezar a hacer más cosas, vamos a actualizar nuestro sistema para ponerlo a la última. Recordad, estamos en una distribución Rolling Release, así que vamos a decirle al sistema que mire que es lo que tiene instalado, busque en los repositosio las actualizaciones y nos las instale.

```
[root@alarmpi ~]# pacman -Syu
```

**pacman** es el comando para manejar y administrar paquetes de software, con el podrás instalar, actualizar, eliminar y listar el software que tienes instalado. Con los parámetros **\-Syu** le indicamos la forma de comportarse, en este caso es S = sincroniza paquetes, y = refresca la descarga del paquete desde el repositorio, es lo mismo que usar --refresh, y u = actualizar todos los paquetes que están fuera de fecha, es lo mismo que usar el parámetro `--sysupgrade` Veremos como el sistema comprueba todos los paquetes y nos solicita confirmación para realizar la actualización. La primera vez lleva un tiempo. Finalizada la actualización debemos reiniciar la máquina para que se carguen todos los paquetes nuevos/actualizados correctamente.

```
[root@alarmpi ~]# sync
[root@alarmpi ~]# reboot
```

El comando **sync** lo que hace es volcar a disco toda la información que puedan contener los buffers de memoria y que aún no se han fijado en fichero. Es por decirlo en lenguaje más plano, salvar a disco lo que está en vuelo o en memoria de la máquina. El comando **reboot** lo que hace es un apagado y posterior inicio de nuestra máquina. Podemos hacerlo de más maneras, usando por ejemplo el comando `shutdown -r` o el propio systemd nos permite reiniciar con el comando `systemctl reboot`.

Alguna vez he visto diferencias entre ambos, aparte de la evidente que shutdown tiene por defecto un periodo de tiempo de gracia durante el cual puedes interrumpir la parada (con `shutdown -c`), pero son cosas relativamente específicas al sistema operativo, tal como esto que os decía de sync, que pueda ser invocado por el comando de reinicio, desconexiones y parada de servicios, etc. No quiero entretenerme demasiado.

## Un toque personal

Nuestra máquina ya está casi lista para empezar a hacer tareas de las "divertidas", antes de seguir, vamos a darle un poco de toque personal, hay unas cuantas cosas que no están configuradas o que si lo están es con los valores por defecto y esto no me gusta.

- En primer lugar, la clave del **administrador**:

```
[root@alarmpi ~]# passwd root
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

El comando **passwd** nos permite cambiar la clave, se nos solicita dos veces y no se muestra en pantalla. No recuerdo bien si esta clave tiene política de caracteres, en plan mínimo 8 caracteres, al menos una mayúscula, un dígito numérico, un carácter raruno como #$%& y que no se haya repetido la clave en los últimos cambios.. etc etc.. La verdad es que suelo usar una clave más o menos compleja que harto de estas exigencias las cumple todas y una parte de la clave es variable para evitar repetir en futuros cambios. Lo dicho, ponéis la clave nueva que queráis.

- Cambiamos la configuración de zona horaria

```
[root@alarmpi etc]# echo "Europe/Madrid" > /etc/timezone
```

Os explico. El comando **echo** lo que hace es imprimir en la salida estandar lo que le digamos, hace un "eco". ¿Salida estandar?, bien os amplio la información, sin nervios. Estáis tratando con un sistema operativo de verdad, no aquellos juguetes de Ms-DOS (argggg), aquí tenéis una entrada estandar (por defecto teclado) y DOS salidas, la estandar (ó salida 1) y la de errores (ó salida 2), ambas se imprimen en pantalla por defecto. Con los símbolos > < | podéis manejar y redireccionar las entradas/salidas a vuestro antojo, entrada, salida y pipe o tubería. En este caso, el del comando echo, lo que hacemos es que la salida estandar, que es la pantalla (ó salida 1, solo que el 1 siempre se omite) es redireccionada a algo, en este caso a un fichero, por tanto TODO lo que ese comando echo vaya a mostrar la pantalla se va a grabar en ese fichero, salvo lo que vaya dirigido a pantalla y que esté por la salida de errores. Existe otro formato de redireción, que es >> , con este lo que indicamos es que si el destino de nuestra salida existe no lo machaque, sino que añada contenido. ¿Y la salida de errores?, pues bien, los comandos en Unix/Lnx muestran información por las dos salidas, por la estandar y por la de errores, ambas por defecto aparecen en pantalla, pero nosotros podemos diferenciarlos y reconducir una, otra o ambas a donde queramos. Por ejemplo si un comando lo tratamos para que su salida sea así `> /var/log/ejecucion.log 2> /var/log/errores.log` tendremos que en pantalla no vemos nada (se puede solucionar con tee) pero se generan dos ficheros, uno, ejecución.log tiene la salida estandar, y el otro, errores.log tiene los errores del proceso.

De esta manera podemos consultar el fichero de errores, que puede tener pocas o ninguna línea sin necesidad de validar un único archivo de miles de líneas. También podemo decidir unificar ambas salidas, para tener tanto la salida como el error en el mismo fichero, para esto, la redirección sería así `> /var/log/ejecucion.log 2>&1`, con ese &1 indicamos que todo lo que vaya a la salida 2 (errores) sea enviado junto con la salida 1 (estandar), y por tanto al mismo fichero. Y otra cosa, que se suele usar mucho, en muchos casos queremos que se ejecute un proceso/programa, pero que nonos deje rastro del posible error.

Imaginad que lanzáis un borrado de ficheros residuales, si existen el comando de borrado los elimina, salida de errores 0, pero si no existe saca error por la salida de errores, aunque a nosotros no nos importa, si hay borras, si no hay pues vale. Para eso, enviamos la salida de errores a un dispositivo muy especial y entrañable /dev/null, siendo el comando algo así `rm *.kk 2 > /dev/null` . Me borras todo lo que sea .kk y en caso de error (salida 2) me envías toda esa información al dispositivo /dev/null, que para que os hagáis una idea es como la alfombra que se levanta para meter lo que barres del suelo y allí se queda, desaparece sólo.

- Continuamos con la personalización, perdón por el tostón, a veces me cuesta contenerme.
```
[root@alarmpi ~]# rm /etc/localtime
[root@alarmpi ~\# ln -s /usr/share/zoneinfo/Europe/Madrid /etc/localtime
```

Borramos el enlace `/etc/localime` y lo volvemos a crear. ¿Que es un enlace?, pues un puntero a otro fichero/directorio en nuestro sistema. Si lo borramos estamos borrando solamente el enlace, no el destino del puntero, y si borramos el destino del puntero nuestro enlace se queda "colgado" (según el tipo de terminal y su configuración de visualización, podrás llegar a ver los enlaces colgados parpadeando en pantalla). Con el comando ln creamos un link o enlace, con el modificador -s le indicamos que es un enlace de los denominados suaves, damos el destino del enlace y el nombre del mismo.

- Ahora modificamos las “locales”

Más bien las generamos para poder tener soporte completo a todo nuestros juego de caracteres, incluidas las Ñs y todas las vocales acentuadas. Para hacer esto, los pasos son tres: 

1.- Editar el fichero `/etc/locale.gen` para eliminar el comentario (`#`) de las líneas que contienen los juegos de caracteres en español (`#es_ES@euro ISO-8859-15` y `#es_ES.UTF-8 UTF-8`)

```
[root@alarmpi ~]# vi /etc/locale.gen
(Pulsar la tecla / shift + 7, y teclear la cadena a buscar, en este caso `es_ES` no lleva a la primera línea que lo
contiene)
#es_ES.UTF-8 UTF-8
#es_ES@euro ISO-8859-15
(Sobre la # de estas dos líneas pulsamos la tecla x (x minúscula) para eliminar ese carácter)
ESC + :wq!
```

2.- Generar las locales, mediante el comando locale-gen
```
[root@alarmpi ~]# locale-gen
Generating locales...
es_ES.UTF-8
es_ES.ISO-8859-15
Generation complete.
```

3.- Configurar nuestro sistema para usar las locales generadas, editando y/o creando el fichero /etc/locale.conf
```
[root@alarmpi ~]# vi /etc/locale.conf
LANG=es_ES.UTF-8
LC_MESSAGES=es_ES.UTF-8
ESC + :wq!
```

Si queremos estar seguro que tenemos las locales y el juego de caracteres correcto, en el siguiente reinicio de 
máquina podemos emplear el comando locale para ver una salida del comando similar a esto:

```
[root@ alarmpi ~]# localectl status
System Locale: LANG=es_ES.UTF-8
VC Keymap: n/a
X11 Layout: n/a
```

4.- Y configuramos la distribución de las teclas para el posible uso de un teclado USB conectado a la máquina.
```
[root@alarmpi ~]# echo “KEYMAP=mac-euro2” > /etc/vconsole.conf [root@alarmpi ~]# localectl set-keymap mac-euro2
[root@alarmpi ~]# localectl status
System Locale: LANG=es_ES.UTF-8 
    VC Keymap: mac-euro2
   X11 Layout: es X11 Model: pc105
  X11 Options: terminate:ctrl_alt_bksp
```

... Seguimos en el [siguiente mensaje](rpi_install_ii.md) ...
