---
title: "Instalando una RaspberryPI todo uso… en 2014 (parte 17) – Apéndice III"
date: "2020-02-08"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
---

## Administrar por consola WEB nuestra Raspberry

He dejado un poco para el final esta parte del tutorial.

Espero que por lo menos lo que hayáis leído hasta ahora os haya dejado algo de poso.

A la respuesta de si se puede administrar la Raspberry de una forma más amigable, hasta ahora os he contestado que NO.

Mentira… pero como dicen los padres… era por vuestro bien.

Existe un software, desde hace muuuchooo tiempo que se llama [Webmin](http://www.webmin.com)

Es una interfaz basada en web que nos permite administrar sistemas Unix (desde grandes a pequeños) sin ni siquiera tocar la línea de comando (os estoy viendo… algunos dais saltos de alegría…)

¿Se puede hacer lo mismo con Webmin que con la línea de comandos?, en principio si, pero no, la línea es mucho más potente, de eso no hay duda, pero digamos que podréis hacer un 90% con Webmin.

Existe otro riesgo potencial y es que los sistemas operativos evolucionan más y mejor que Webmin (que ha mejorado mucho desde la última vez que lo vi, hará como unos 15 años) y podréis encontrar que algo falla o que no se ejecuta bien en la máquina por que el componente Webmin que interactúa con esa parte de la máquina está desactualizado.

Esto en el caso de ARCH, que como os dije es una Rolling Release, pues ya véis, actualizaciones casi a diario del sistema, vosotros decidís.

Eso sí, si alguien os ve alguna vez esta consola y os pregunta, por favor, no le digáis que os lo he contado yo, quiero mantener mi ya de por sí maltrecha reputación intacta.

## Instalación de Webmin

Os recomiendo que antes de instalar Webmin ejecutéis el comando de actualización completa de máquina

[root@Jarvis ~]# pacman -Suy

:: Synchronizing package databases...
core 154.6 KiB 522K/s 00:00     [############################################] 100%
extra 2.0 MiB 2.16M/s 00:01     [############################################] 100%
community 2.2 MiB 2.19M/s 00:01 [############################################] 100%
alarm is up to date
aur is up to date
:: Starting full system upgrade...
resolving dependencies...
looking for inter-conflicts...
Packages (2): mesa-10.1.2-1 mesa-libgl-10.1.2-1
Total Download Size: 1.82 MiB
Total Installed Size: 12.63 MiB
Net Upgrade Size: 0.01 MiB
:: Proceed with installation? [Y/n]

…

[root@Jarvis ~]# sync; reboot

Y después de reiniciar también es una buena idea “apagar” los servicios que tengáis corriendo en la máquina para evitar conflictos de ficheros abiertos.

[root@Jarvis ~]# systemctl stop xbmc mysqld smbd nmbd

Instalamos el paquete de Webmin, que como siempre, gracias a pacman es muy fácil

[root@Jarvis ~]# pacman -S webmin

resolving dependencies...
looking for inter-conflicts...
Packages (2): perl-perl4-corelibs-0.003-2 webmin-1.680-1
Total Download Size: 11.69 MiB
Total Installed Size: 57.01 MiB
:: Proceed with installation? [Y/n] Y
:: Retrieving packages ...
perl-perl4-corelibs-0.003-2-any 36.5 KiB 269K/s 00:00 [############################################] 100%
webmin-1.680-1-armv6h 11.7 MiB 3.56M/s 00:03          [############################################] 100%
(2/2) checking keys in keyring                        [############################################] 100%
(2/2) checking package integrity                      [############################################] 100%
(2/2) loading package files                           [############################################] 100%
(2/2) checking for file conflicts                     [############################################] 100%
(2/2) checking available disk space                   [############################################] 100%
(1/2) installing perl-perl4-corelibs                  [############################################] 100%
(2/2) installing webmin                               [############################################] 100%

Note:
==> It is not allowed to install 3rd party modules, or delete existing modules.
==> Please write your own PKGBUILDS for 3rd party modules and additional themes.

Setup:
==> To make webmin start at boot time, add webmin to rc.conf daemons
==> Point your web browser to http://localhost:10000 to use webmin.
==> The access is restricted to localhost, if you want to connect from other locations
==> change /etc/webmin/miniserv.conf to something like that: allow=127.0.0.1 <your-ip>
==> If you want to have ssl encryption please install 'perl-net-ssleay' additional.

synchronizing filesystem...

## Configuración de Webmin

Como podéis leer en el mensaje post-instalación el acceso a la máquina para administrarla con webmin es por http al puerto 1000, este valor se puede cambiar y poner otro que no sea tan estándar, aunque siendo webmin un tanto “coladero” os dará igual.

Más, la conexión es en plano, http, no hay cifrado a no ser que instaléis las librerías de perl para ssl como os indica este mismo mensaje, pero bueno… como webmin es un “coladero”…

Así que sólo os queda modificar este fichero /etc/webmin/miniserv.conf para añadir la dirección IP desde la que podréis acceder a administrar el equipo, vamos, la dirección o direcciones ip de vuestra red LOCAL (tu mac, tu iphone, tu ipad) que podrán acceder, que nadie abra Webmin al exterior en el router, por si no os lo he dicho, webmin es un “coladero”.

En caso contrario, no añadir vuestras IPs de red local, no podréis hacer nada pues Webmin por defecto solo admite conexiones procedente de 127.0.0.1 (localhost).

En mi caso esa línea se queda tal que así, con la dirección de mi iMac y del iPhone

allow=127.0.0.1 192.168.1.11 192.168.1.18

Lo activamos para que se arranque al inicio del sistema y lo arrancamos para poder entrar

[root@Jarvis ~]# systemctl enable webmin
ln -s '/usr/lib/systemd/system/webmin.service' '/etc/systemd/system/multi-user.target.wants/webmin.service'

[root@Jarvis ~]# systemctl start webmin

Y como es lógico no funciona… (la línea de comandos si… Webmin 0 - Linea de Comandos 1 mu ha ha ha), el log `/var/log/webmin/miniserv.err` nos indica que le falta un paquete del intérprete de Perl para PAM (recordar, los módulos de autenticación, como el google_authenticator que montamos).

Esto pasa por no leer la documentación de instalación ([http://www.webmin.com/udownload.html](http://www.webmin.com/udownload.html)) que lo dice muy clarito en sus requisitos.

Por si acaso, vamos a instalar las perl-net-ssl por si acaso decidimos montar el acceso por SSL.

[root@Jarvis ~]# pacman -S perl-net-ssleay

resolving dependencies...
looking for inter-conflicts...
Packages (1): perl-net-ssleay-1.58-1
Total Download Size: 0.17 MiB
Total Installed Size: 0.67 MiB
:: Proceed with installation? [Y/n] Y
:: Retrieving packages ...
perl-net-ssleay-1.58-1-armv6h 172.1 KiB 486K/s 00:00 [############################################] 100%
(1/1) checking keys in keyring                       [############################################] 100%
(1/1) checking package integrity                     [############################################] 100%
(1/1) loading package files                          [############################################] 100%
(1/1) checking for file conflicts                    [############################################] 100%
(1/1) checking available disk space                  [############################################] 100%
(1/1) installing perl-net-ssleay                     [############################################] 100%

synchronizing filesystem...

Y ahora vamos con ese módulo para PAM en perl que hay que compilarlo (Webmin 0 - Linea de Comandos 2)

Nos creamos un directorio temporal, descargamos los fuentes desde CPAN y compilamos siguiente las instrucciones de su README.

[root@Jarvis ~]# mkdir ~/Builds
[root@Jarvis ~]# cd ~/Builds/

[root@Jarvis Builds]# wget http://search.cpan.org/CPAN/authors/id/N/NI/NIKIP/Authen-PAM-0.16.tar.gz

--2014-05-06 21:18:41-- http://search.cpan.org/CPAN/authors/id/N/NI/NIKIP/Authen-PAM-0.16.tar.gz
Resolving search.cpan.org (search.cpan.org)... 199.15.176.161
Connecting to search.cpan.org (search.cpan.org)|199.15.176.161|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://www.cpan.org/authors/id/N/NI/NIKIP/Authen-PAM-0.16.tar.gz [following]

--2014-05-06 21:18:41-- http://www.cpan.org/authors/id/N/NI/NIKIP/Authen-PAM-0.16.tar.gz
Resolving www.cpan.org (www.cpan.org)... 212.117.177.118, 2a01:608:2:4::2, 2607:f238:3::91:1
Connecting to www.cpan.org (www.cpan.org)|212.117.177.118|:80... connected.
HTTP request sent, awaiting response... 200 OK

Length: 45922 (45K) [application/x-gzip]
Saving to: 'Authen-PAM-0.16.tar.gz'
100%[======================================================================================>] 45,922 --.-K/s in 0.1s
2014-05-06 21:18:42 (341 KB/s) - 'Authen-PAM-0.16.tar.gz' saved [45922/45922]

[root@Jarvis Builds]# tar -xvzf Authen-PAM-0.16.tar.gz

Authen-PAM-0.16/
Authen-PAM-0.16/PAM_config.h.in
Authen-PAM-0.16/META.yml
Authen-PAM-0.16/configure.ac
Authen-PAM-0.16/test.pl
Authen-PAM-0.16/Changes
Authen-PAM-0.16/d/
Authen-PAM-0.16/d/PAM.pm
Authen-PAM-0.16/PAM.xs
Authen-PAM-0.16/MANIFEST
Authen-PAM-0.16/PAM.pm.in
Authen-PAM-0.16/typemap
Authen-PAM-0.16/configure
Authen-PAM-0.16/PAM/
Authen-PAM-0.16/PAM/FAQ.pod
Authen-PAM-0.16/pam.cfg.in
Authen-PAM-0.16/README
Authen-PAM-0.16/Makefile.PL

[root@Jarvis Builds]# cd Authen-PAM-0.16

[root@Jarvis Authen-PAM-0.16]# perl Makefile.PL
Checking if your kit is complete...
Looks good
checking for gcc... cc

…

root@Jarvis Authen-PAM-0.16]# make
cp PAM.pm blib/lib/Authen/PAM.pm
cp PAM/FAQ.pod blib/lib/Authen/PAM/FAQ.pod
/usr/bin/perl /usr/share/perl5/core_perl/ExtUtils

…

[root@Jarvis Authen-PAM-0.16]# make test
PERL_DL_NONLAZY=1 /usr/bin/perl "-Iblib/lib" "-Iblib/arch" test.pl
1..10
ok 1

---- The remaining tests will be run for service 'login', user 'root' and
---- device '/dev/pts/0'.
ok 2
ok 3
ok 4
ok 5
ok 6
ok 7
ok 8
---- Now, you may be prompted to enter the password of 'root'.

Password:
not ok 9 (7 - Authentication failure)
---- The failure of test 9 could be due to your PAM configuration
---- or typing an incorrect password.
ok 10
ok 11

[root@Jarvis Authen-PAM-0.16]# make install
Files found in blib/arch: installing files in blib/lib into architecture dependent library tree
Installing /usr/lib/perl5/site_perl/auto/Authen/PAM/PAM.so
Installing /usr/lib/perl5/site_perl/auto/Authen/PAM/PAM.bs
Installing /usr/lib/perl5/site_perl/Authen/PAM.pm
Installing /usr/lib/perl5/site_perl/Authen/PAM/FAQ.pod
Installing /usr/share/man/man3/Authen::PAM.3pm
Installing /usr/share/man/man3/Authen::PAM::FAQ.3pm
Appending installation info to /usr/lib/perl5/core_perl/perllocal.pod

Reiniciamos webmin para que se cargue con los cambios y librerías nuevas

[root@Jarvis ~]# systemctl restart webmin

Y nueva prueba, por lo menos ahora al arrancar el log nos dice que ha cargado el módulo de PAM, que ya es algo

[06/May/2014:21:26:45 +0200] miniserv.pl started
[06/May/2014:21:26:45 +0200] Using MD5 module Digest::MD5
[06/May/2014:21:26:45 +0200] PAM authentication enabled

Y probamos a entrar a la url de gestión por Webmin, eso si, recordad que ahora estamos con SSL y por tanto será algo así como https://192.168.1.20:10000

Deberéis aceptar el certificado para poder entrar.

## Y así es Webmin

*Nota del 2020: las capturas de pantalla... se perdieron, pero como webmin sigue siendo un coladero, tampoco importa.

La página inicial de login, según la documentación, debemos entrar con usuario UID 0 (root) o con un usuario con privilegios para ejecutar sudo.

(http://i59.tinypic.com/3462qee.png)

Y no te deja entrar si no entras con root directamente, fantástico coladero (Webmin 0 - Linea de Comandos 3)

Una vez accedemos se nos muestra una pantalla de resumen con el estado de nuestra máquina

(http://i57.tinypic.com/2mccdg8.png)

Y este menú en la izquierda podremos seleccionar lo que queremos hacer en nuestra máquina RPi

(http://i62.tinypic.com/qyedkz.png)

Os recomiendo entra en la configuración de webmin y cambiar el idioma

(http://i62.tinypic.com/4revmg.png)

Así como añadir módulos adicionales de servicios que tengáis en marcha, como podría ser SAMBA.

(http://i60.tinypic.com/24o65ax.png)

Otra opción importante a activar es la autenticación en dos pasos, esto ya lo hemos visto y bueno, por si no os lo he dicho, webmin es un “coladero”, así que siempre viene bien un poco más de seguridad.

(http://i62.tinypic.com/ztamg1.png)

Y de aquí en adelante, pues ya es investigar, yo es que no pierdo mucho el tiempo con estas herramientas.
