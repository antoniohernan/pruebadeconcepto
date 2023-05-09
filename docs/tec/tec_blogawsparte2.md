---
title: "Monta tu blog en AWS por 0€ – parte 2"
date: "2019-05-29"
categories: 
  - "tecnologia"
tags: 
  - "aws"
  - "dynamicdns"
  - "ec2"
  - "ubuntu"
---

Continuamos con nuestro pequeño servidor, al que deberemos tener acceso si nos han ido bien los pasos de la [primera parte](/tec/tec_blogawsparte1.md)

Antes de seguir con esta máquina **vamos a generar nuestra base de datos**, empleando el servicio gestionado de bases de datos relacionaes, [RDS](https://aws.amazon.com/es/rds/) de AWS.

Como en el caso que vimos **para crear la instancia EC2, las credenciales de AIM** que tengamos debe tener privilegios para poder crear este tipo de objetos, y le debemos ortorgar a ese perfil dentro de IAM [el acceso pleno a RDS](https://aws.amazon.com/es/rds/):

![](images/Selección_454.png)

La creación de la instancia la podemos hacer a través de esta línea de comandos (**el SG será el creado para SG\_RDS**, ver parte 1):

aws rds create-db-instance \\
--allocated-storage 20 --db-instance-class db.t2.micro \\
--db-instance-identifier pruebadeconceptoDB --engine MySQL \\
--master-username "pruebadeconcepto" --master-user-password "TUCLAVEAQUI" \\
--backup-retention-period 1 \\
--vpc-security-group-ids "sg-0729ab6694bef3faa" \\
--availability-zone eu-west-1a

Y listo, pasados un par de minutos ya tendremos operativa la base datos, que consumimos como un servicio y nos despreocupamos de todo.

Con la **base de datos funcionando ya podemos terminar la instalación** de nuestro Wordpress, en primer lugar actualizamos el servidor:

ubuntu@ip-172-31-22-120:~$ sudo apt-get update
Hit:1 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu bionic InRelease
Get:2 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu bionic-updates InRelease \[88.7 kB\]
Get:3 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu bionic-backports InRelease \[74.6 kB\]
Hit:4 http://ppa.launchpad.net/certbot/certbot/ubuntu bionic InRelease
Get:5 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main Sources \[272 kB\]
Get:6 http://security.ubuntu.com/ubuntu bionic-security InRelease \[88.7 kB\]
Get:7 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages \[627 kB\]
Get:8 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main Translation-en \[233 kB\]
Fetched 1384 kB in 1s (2079 kB/s)
Reading package lists... Done

ubuntu@ip-172-31-22-120:~$ sudo apt-get upgrade
Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
python3-software-properties software-properties-common
2 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 33.8 kB of archives.
After this operation, 0 B of additional disk space will be used.
Do you want to continue? \[Y/n\] y
Get:1 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 software-properties-common all 0.96.24.32.9 \[9992 B\]
Get:2 http://eu-west-1.ec2.archive.ubuntu.com/ubuntu bionic-updates/main amd64 python3-software-properties all 0.96.24.32.9 \[23.8 kB\]
Fetched 33.8 kB in 0s (3343 kB/s)
(Reading database ... 114214 files and directories currently installed.)
Preparing to unpack .../software-properties-common\_0.96.24.32.9\_all.deb ...
Unpacking software-properties-common (0.96.24.32.9) over (0.96.24.32.8) ...
Preparing to unpack .../python3-software-properties\_0.96.24.32.9\_all.deb ...
Unpacking python3-software-properties (0.96.24.32.9) over (0.96.24.32.8) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Setting up python3-software-properties (0.96.24.32.9) ...
Processing triggers for dbus (1.12.2-1ubuntu1) ...
Setting up software-properties-common (0.96.24.32.9) ...

Ahora **vamos a crear un dominio, gratuito claro** está, basado en _nsupdate.info_ y con el cliente _ddclient_ para que se refresque cada vez que tengamos un cambios de IP en la máquina.

**Necesitaremos una cuenta** en [https://www.nsupdate.info/](https://www.nsupdate.info/), es gratis, el registro es rápido y daremos de alta el dominio que queramos.

En [https://www.nsupdate.info/host/add/](https://www.nsupdate.info/host/add/) tan solo **tendremos que poner el nombre, elegir el dominio al que va a pertenecer ese nombre y un comentario**. Al aceptar nos saldrá una pantalla en la que **deberemos fijarnos el secreto** que se nos genera, cada vez que entremos a esa pantalla va a generar uno nuevo que tendremos que actualizar, así que cuidado con esto:

![](images/Selección_455-300x149.png)

Y al finalizar **tendremos nuestro dominio creado**, y ahora vamos ver como actualizar la IP a la que debe apuntar.

En el servidor **instalamos ddclient**

ubuntu@ip-172-31-22-120:~$ sudo apt-get install ddclient

El **fichero de configuración** es más o menos como este (bueno, yo lo tengo con dos dominios... pero eso ya os lo contaré en otra ocasión)

ubuntu@ip-172-31-22-120:~$ sudo cat /etc/ddclient.conf
# /etc/ddclient.conf

protocol=dyndns2
use=web, web=http://ipv4.nsupdate.info/myip
ssl=yes # yes = use https for updates
server=ipv4.nsupdate.info
login=pruebadeconcepto.es
password='XXXXXXXXX'
pruebadeconcepto.es

protocol=dyndns2
use=web, web=http://ipv4.nsupdate.info/myip
ssl=yes # yes = use https for updates
server=ipv4.nsupdate.info
login=piscinadeentropia.nsupdate.info
password='XXXXXXXXX'
piscinadeentropia.nsupdate.info

Y es justo donde **pone esos password= donde tenéis que poner el Secret** que hemos visto antes en la página de _nsupdate.info_ que permite sincronizar la IP con el dominio.

**Antes de arrancar**, tenemos que asegurar que la configuración **permite a este proceso correr como demonio**:

ubuntu@ip-172-31-22-120:~$ sudo vi /etc/default/ddclient

# Configuration for ddclient scripts
# generated from debconf on Thu Apr 25 09:08:04 UTC 2019
#
# /etc/default/ddclient

# Set to "true" if ddclient should be run every time DHCP client ('dhclient'
# from package isc-dhcp-client) updates the systems IP address.
run\_dhclient="false"

# Set to "true" if ddclient should be run every time a new ppp connection is
# established. This might be useful, if you are using dial-on-demand.
run\_ipup="false"

# Set to "true" if ddclient should run in daemon mode
# If this is changed to true, run\_ipup and run\_dhclient must be set to false.
run\_daemon="true"

# Set the time interval between the updates of the dynamic DNS name in seconds.
# This option only takes effect if the ddclient runs in daemon mode.
daemon\_interval="300"

**Arrancamos el proceso y lo dejamos habilitado** para los reincios del sistema:

ubuntu@ip-172-31-22-120:~$ sudo systemctl start ddclient
ubuntu@ip-172-31-22-120:~$ sudo systemctl enable ddclient

Puedes **comprobar** que la IP "dinámica" del equipo que tenemos en la consola de AWS **coincide** con la IP de la página de nsupdate.info.

![](images/Selección_456.png)

En la **siguiente entrada veremos como instalar el servidor web, php y el propio wordpress**.

... continuará ...
