---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 4) – Securización"
date: "2020-01-20"
---

## ¿Es necesario securizar mi máquina?

Esto lo tienes que decidir tu.

Si vas a tener tú máquina en el salón de casa y sólo la usas para XBMC o para emular juegos y poco más lo mismo no te merece ni molestarte en hacerlo.

Ahora bien, si vas a tener tu máquina en la wifi de tu piso de estudiantes compartido, si la vas a conectar al exterior y vas a querer conectarte desde fuera por conexión SSH, o si vas a poner esta máquina en el colegio donde das clases lleno de jóvenes hiperhormonados que desean pasar a la historia del centro educativo como el que formateó el servidor del profesor... **pues sí, debes securizarla**.

Esto es como dejar una bicicleta en la puerta de una biblioteca, contra más medidas de seguridad más costará que te la roben, pero ten por seguro que si quieren ir a por ella y robártela, les costará más o menos pero te la robarán.

## Vale, entonces securizamos, creamos usuario

Lo primero que vamos a hacer es **dejar de usar nuestro apreciado usuario root** o administrador para todas las tareas habituales. Con esto consigues dos cosas, por un lado dejas de exponer tu conexión segura con estas credenciales y consigues usar otro usuario con muchos menos privilegios que puede que a la hora de que algún comando de los "críticos" se te escape de los dedos, la cosa no vaya a mayores.

Por tanto, **vamos a crear un nuevo usuario**.

En Linux/Unix los usuarios residen en un fichero de configuración `/etc/passwd`, este fichero tiene estructura de campos separados por el caracter **:** y hay datos tales como su UID o **identificador de usuario**, su GID o **identificador de grupo** PRIMARIO (por que **podemos tener varios grupos secundarios**), datos como nombre, ruta del directorio HOME y el ejecutable de nuestra Shell de usuarios, esto es, nuestro intérprete de comandos.

En ningún caso aparecen claves, ni cifradas ni sin cifrar, estas claves están en otro fichero `/etc/shadow` donde aparecen cifradas y este fichero a su vez no es accesible salvo para los administradores del sistema.

Como os he dicho los usuarios tienen un grupo primario y pueden tener grupos secundarios, los grupos se guardan en el fichero `/etc/passwd` que también tiene esa estructura de campos separados por el carácter :, apareciendo en primer lugar el nombre del grupo, a continuación el GID o identificador de grupo y por último, una lista separada por , con todos los usuarios que forman parte del grupo a nivel secundario.

Creamos el grupo y el usuario con esta secuencia de comandos:
```
[root@Jarvis etc]# groupadd -g 1001 operadores
[root@Jarvis etc]# useradd -m -g 1001 -u 1001 -s /bin/bash -d /home/operador operador
[root@Jarvis ~]# passwd operador

Enter new UNIX password: ***
Retype new UNIX password: ***

passwd: password updated successfully
```
El comando `groupadd` añade un nuevo grupo al fichero `/etc/group`, con el parámetro -g indicamos el número 
identificador de grupo que queremos utilizar, y en último lugar el argumento que se corresponde al nombre del grupo.

El comando `useradd`añade un nuevo usuario al fichero `/etc/passwd`, con el parámetro -g indicamos el número de identificador de su grupo primario, con el parámetro -u indicamos el número de identificador del usuario, con el parámetro -s el ejecutable de nuestro intérprete de comando o shell, con el parámetro -d la ruta del home del usuario y por último el argumento que da nombre a nuestro usuario.

Con estos dos comandos creamos por tanto grupo y usuario, se rellenan los ficheros passwd y group y se coloca a dicho usuario dentro del grupo creado como grupo primario.

¿**Y la clave**?, la clave se la damos después, con el comando `passwd`, en este caso, el comando recibe un argumento que es el nombre del usuario que hemos creado. Por defecto si no indicamos nada el cambio de clave se hace sobre el usuario con el que estamos conectado, y por definición un usuario no puede cambiar más que su propia clave, así que ejecutar el comando `passwd <usuario>`sólo lo podremos hacer con el usuario administrador o root.

## Instalamos sudo

Ahora vamos a instalar `sudo`, esto os suena de cuando lo hemos usado para instalar la imagen Iso en nuestra tarjeta SD, es ese programa que nos permite ejecutar otros programas como si fuésemos otro usuario, esto se hace así:
```
[root@Jarvis ~]# pacman -S sudo
resolving dependencies...
looking for inter-conflicts...
Packages (1): sudo-1.8.6.p8-1
Total Download Size: 0.55 MiB
Total Installed Size: 2.48 MiB
:: Proceed with installation? [Y/n] Y
:: Retrieving packages ...
sudo-1.8.6.p8-1-armv6h 566.1 KiB 717K/s 00:01 [#################################] 100%
(1/1) checking keys in keyring                [#################################] 100%
(1/1) checking package integrity              [#################################] 100%
(1/1) loading package files                   [#################################] 100%
(1/1) checking for file conflicts             [#################################] 100%
(1/1) checking available disk space           [#################################] 100%
(1/1) installing sudo                         [#################################] 100%
```
Utilizamos nuestro gestor de paquetes **pacman con el parámetro -S** y el argumento sudo, esto es, instalame ese paquete.

Y una vez instalado tenemos que configurarlo, la configuración no es más que la edición de un fichero, `/etc/sudoers`, y para tal fin tenemos un programa específico, visudo, y claro, al llevar VI en el nombre ya os podéis imaginar como seusa.

Por supuesto que también podremos modificar el fichero libremente con VI, pero **cualquier error de sintáxis dejará el sistema hecho unos zorros**, visudo, además de bloquear el fichero (nadie más podrá editarlo a la vez) y hacer copia previa, verifica la sintaxis antes de salvarlo, así pues, uso más que recomendado.
```
[root@Jarvis etc]# visudo

## Uncomment to allow members of group wheel to execute any command

# %wheel ALL=(ALL) ALL

%operadores ALL=(ALL) ALL
```
Añadimos la línea arriba indicada, ¿qué significa esa línea?

El pimero ALL= indica **desde donde nos pueden ejecutar**, en este caso desde cualquier máquina.

El segundo (ALL) **indica los usuarios que podrán hacer sudo**, dentro del grupo que vemos ahora.

El tercer ALL **indica los comandos que ese usuario podrá ejecutar**, en este caso todos.

¿Y ese %operadores? pues con eso indicamos que todo esto que hemos explicado arriba de los ALL sólo lo van a poder hacer los usuarios que por perfilado (passwd y group) tengran el grupo operadores como grupo **PRIMARIO**, esto es, como grupo dentro de passwd, si el usuario tiene el grupo como **SECUNDARIO** a través del fichero de grupos no cumplirá esta configuración y no podrá ejecutar sudo.

Cuando ejecutemos por tanto sudo con nuestro usuario operador nos solicitará una clave, que es la nuestra, la del usuario operador.

Ahora con esto instalado ya no necesitamos el acceso directo a la máquina con el usuario root, por tanto, lo mejor es cerrarlo, que no se pueda conectar nadie directamente con un usuario privilegiado.

Esta operación es algo más delicada pues hay que tocar algunos ficheros de configuración que de hacerlo mal nos quedamos sin conexión... así que cuidado, y **ante la duda ESC + :q! para salir de VI sin hacer cambios**.

## Cortamos la conexión directa de root

Para esto, en primer lugar editamos el fichero `/etc/ssh/sshd_config`

`[root@Jarvis etc]# vi /etc/ssh/sshd_config`

Debemos buscar la línea que tiene:

`#PermitRootLogin yes`

Y cambiarla por esta otra:

`PermitRootLogin no`

Para esto bajamos con los cursores por el fichero y una vez puestos sobre el símbolo `#` de la línea pulsamos la tecla `x`, que nos borra ese carácter. Ahora nos movemos con las flechas a la derecha hasta ponernos encima de la y de yes y pulsamos el comando cw (Change Word), nos aparecerá un símbolo `$` sobre la `s` que nos marca el final de la palabra que estamos sustituyendo, y tecleamos no.

Salimos salvando el fichero con `ESC :wq!`

Tenemos que reiniciar el demonio de ssh, **¿demonio?**, si, los procesos residentes en el sistema, que se quedan a la espera de hacer su trabajo ante eventos específicos se llaman demonios (Daemon), en el caso de los procesos de ssh **su demonio se llaman sshd**.

`[root@Jarvis etc]# systemctl restart sshd`

El comando `systemctl` es nuestro comando para controlar el sistema de arranque/ejecución de servicios en la máquina bajo [systemd](https://wiki.archlinux.org/index.php/systemd_(Espa%C3%B1ol))

En este caso le estamos diciendo que reinicie ese servicio/demonio concreto.

En cuanto termine el reinicio del demonio podremos probar a abrir una nueva conexión ssh con root y veremos como nos da un acceso denegado, así pues, a partir de este momento todas las conexiones con el usuario operador y las ejecuciones privilegiadas o bien con sudo o bien haciendo `su -` para cambiar a una shell de usuario root.

## ¿Y podemos usar autenticación en dos pasos?, pues claro

Primero tenemos que instalarnos en nuestro teléfono el programa de Google que nos **genera los "tokens"** por tiempo, para iOs este es el [enlace](https://itunes.apple.com/us/app/google-authenticator/id388497605?mt=8)

Tenemos que descargarnos los fuentes de este librería PAM de la propia Google, y lo tenemos que hacer desde nuestra máquina, vamos a utilizar el comando `wget`

PAM es el acrónimo de **Linux Pluggable Authentication Modules**, esto es, módulo conectables de autenticación, vamos a darle a nuestro sistema por tanto un módulo de autenticación en dos pasos de Google en formato librería.
```
[operador@Jarvis ~]$ su - 
  Password: <aquí la clave de root> 

[root@Jarvis ~]# mkdir /Descargas; cd /Descargas 
[root@Jarvis Descargas]# wget https://google-authenticator.googlecode.com/files/libpam-google-authenticator-1.0-source.tar.bz2
```
Nos conectamos con operador, hacemos un `su -` para cambiar al usuario root (la clave que se nos pide es la de ese usuario, root).

Creamos un directorio con el comando `mkdir` (Make Directory), y nos posicionamos en el con `cd`. Como veis los dos comandos están en la misma línea separados por un `;` ,esto es por que la shell ejecuta un comando y ejecuta el otro.

Si miramos la "doble extensión" del fichero que nos descargamos vemos que por un lado se trata de un fichero comprimido con [Bzip2](https://www.archlinux.org/packages/core/i686/bzip2/) dada la extensión `bz2` y que además se trata de un conjunto de archivos y directorios empaquetados con [tar](http://es.wikipedia.org/wiki/Tar)

Para desempaquetar el **tar** no tenemos problemas pues es algo que siempre vamos a encontrar en la distribución, prácticamente sea la que sea.

En cuanto a **bzip2** ya es otra cosa y la tendremos que instalar, para esto, ya sabéis recurrimos a nuestro gestor de paquetes (seguimos teniendo nuestra conexión y nuestra sesión de operador con la que hemos cambiado a root):

```
[root@Jarvis Descargas]# pacman -S bzip2

resolving dependencies...
```

Tenemos que descomprimir y desempaquetar el fichero de los fuentes de esta librería, con algunas versiones recientes de tar es posible que este haga las dos operaciones, que primero descomprima y luego desempaquete, pero con esta que os pongo yo a continuación os funcionará siempre, sea cual sea la versión de tar y el formato de compresión:

```
[root@Jarvis Descargas]# bzip2 -dc libpam-google-authenticator-1.0-source.tar.bz2 | tar -xv
```

¿Os acordáis que antes os he contado lo de `> <` y `|`?, pues aquí vemos el **uso de | (pipe o tubería)**, esto lo que hace es convertir la salida estandar de un comando en la entrada estándar del siguiente, por tanto toma la descompresión del fichero y se lo entrega al comando tar para que lo desempaquete.

Sólo nos queda compilar, y para esto necesitamos entorno de desarrollo, lo que implica instalar al menos los siguientes paquetes:

```
[root@Jarvis Descargas]# pacman -S make gcc

resolving dependencies...
```

Y estas son las órdenes para poder compilar:

```
[root@Jarvis Descargas]# cd libpam-google-authenticator-1.0
[root@Jarvis libpam-google-authenticator-1.0]# make

gcc --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o google-authenticator.o google-authenticator.c
gcc --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o base32.o base32.c
gcc --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o hmac.o hmac.c
gcc --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o sha1.o sha1.c
gcc -g -o google-authenticator google-authenticator.o base32.o hmac.o sha1.o -ldl
gcc --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o pam_google_authenticator.o pam_google_authenticator.c
gcc -shared -g -o pam_google_authenticator.so pam_google_authenticator.o base32.o hmac.o sha1.o -lpam
gcc --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o demo.o demo.c
gcc -DDEMO --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o pam_google_authenticator_demo.o pam_google_authenticator.c
gcc -g -rdynamic -o demo demo.o pam_google_authenticator_demo.o base32.o hmac.o sha1.o -ldl
gcc -DTESTING --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o pam_google_authenticator_testing.o pam_google_authenticator.c
gcc -shared -g -o pam_google_authenticator_testing.so pam_google_authenticator_testing.o base32.o hmac.o sha1.o -lpam
gcc --std=gnu99 -Wall -O2 -g -fPIC -c -fvisibility=hidden -o pam_google_authenticator_unittest.o pam_google_authenticator_unittest.c
gcc -g -rdynamic -o pam_google_authenticator_unittest pam_google_authenticator_unittest.o base32.o hmac.o sha1.o -lc -ldl
```

Y si la compilación no nos da ningún error, procedemos a instalarlo:
```
[root@Jarvis libpam-google-authenticator-1.0]# make install

cp pam_google_authenticator.so /lib/security
cp google-authenticator /usr/local/bin
```

Esto nos deja por un lado instalada la librería de seguridad, en `/lib/security`, y el programa `google-authenticator` que nos va a servir para genera la entrada en el programa de autenticación de nuestro terminal móvil.

Antes de activar la librería debemos tener generados los códigos de acceso con la aplicación, ¿con que usuario?, puescon el único con el que nos podemos conectar a la máquina, operador, así que necesitamos tener la conexión a nuestra máquina abierta con dicho usuario o abrir una nueva.

Ejectamos el siguiente comando.
```
[operador@Jarvis ~]$ /usr/local/bin/google-authenticator

Do you want authentication tokens to be time-based (y/n) y

https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/root@DomePi%3Fsecret%3DYYMTXZ56APYLY6CE

Your new secret key is: DYYMTXZ56APYLY6CE
Your verification code is 636744

Your emergency scratch codes are:

31868847
79023148
93855998
42788591
48617175

Do you want me to update your "~/.google_authenticator" file (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) n

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
```

Como véis, **se dice a todo que sí salvo a la compensación extra de tiempo**.

Se nos genera una URL y un conjunto de códigos adicionales.

Si copiáis esa URL en un navegador os mostrará un QR que se puede escasear con la aplicación de Google y que directamente os arrancará la aplicación con la generación de códigos para vuestro dispositivo.

Así de fácil, iniciar la app en el móvil, pulsar sobre añadir y escanear ese código QR.

Los otros códigos, pues son código "quemables", esto es, si alguna vez no puedes acceder a la aplicación y te tienes que conectar puedes usar uno de esos códigos, una vez que lo has usado ya no es válido y por tanto decimos que se ha quemado.

Si los usas todos, pues no pasa nada, podrías volver a usar el ejecutable de `google-authenticator` y generar un nuevo conjunto de códigos y una nueva URL de acceso.

Ya tenemos nuestro generador de códigos en sincronía con nuestra máquina, así que vamos a activarlo, para esto en primer lugar tenemos que editar el fichero `/etc/ssh/sshd_config` y modificar lo que se indica a continuación (con el usuario root, si estás con operador deberás hacer el `su -`):

```
[root@Jarvis ~]# vi /etc/ssh/sshd_config

# Change to no to disable s/key passwords

ChallengeResponseAuthentication no
```

Cambiamos esa línea, y **donde dice no ponemos yes**

Ahora editamos el fichero `/etc/pam.d/sshd`, añadiendo una línea al principio del fichero (pulsando la combinación de teclas SHIFT + o, que nos añade una linea por encima de la actual):

```
[root@Jarvis ~]# vi /etc/pam.d/sshd

#%PAM-1.0

#auth required pam_securetty.so #disable remote root

auth required pam_google_authenticator.so

Y ya sólo nos queda reiniciar sshd

[root@Jarvis ~]# systemctl restart sshd
```

Ahora, si hacemos una conexión veremos que no nos pide usuario/clave, sino que nos pide un código de verificación, que es el que nos genera nuestra aplicación móvil (ó alguno de aquellos códigos quemables que habíamos generado)

```
Verification code:
Password:

Last login: Mon Apr 15 19:16:10 2013 from 192.168.1.10

[operador@Jarvis ~]$
```

## ¿Y ya estamos seguros?

Pues no, con esto sólo lo has puesto un poco más difícil, pero no es una máquina segura 100%, para tener unamáquina así, **segura al 100% lo único que puedes hacer es apagarla y desconectarla de la red**...

## ¿Que más se puede hacer?

Pues podemos **activar el firewall** de nuestra máquina, para restringir las conexiones, que sólo nos lo permita desde las direcciones IP de nuestros equipos (p.e. desde nuestro equipo MAC de sobremesa o portátil), o activar la apertura del puerto de conexión sólo ante un evento denominado **PortKnocking**.

Pero esto son cuestiones algo extensas que ya trataré más adelante.
