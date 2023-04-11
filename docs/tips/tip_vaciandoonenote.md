---
title: "Chuleta 03 - Vaciando OneNote"
date: "2019-06-27"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
categories: 
  - "chuletas"
---

Continuamos con la "serie" de **chuletas rescatadas** de distintos soportes, un día de estos conseguiré unificarlo todo... un día de estos...

Ahora es el turno de las que tengo en [OneDrive](https://onedrive.live.com/about/es-es/) que **voy a ir vaciando en favor del uso de otros métodos más ligeros** y rápidos que la suite de Microsoft.

#### Empezamos por la forma de descomprimir un fichero [tar](https://es.wikipedia.org/wiki/Tar)

Ó tar+gz .tgz y directamente volcarlo eliminado los directorios superiores, así dicho es un lío, pero en el caso práctico es esto, si tenemos un tar que contiene por ejemplo:
```
solr-4.3.1/example/solr/collection1/conf/velocity/suggest.vm
solr-4.3.1/example/solr/collection1/conf/velocity/tabs.vm
solr-4.3.1/example/solr/collection1/conf/xslt/example.xsl
solr-4.3.1/example/solr/collection1/conf/xslt/example\_atom.xsl
solr-4.3.1/example/solr/collection1/conf/xslt/example\_rss.xsl
solr-4.3.1/example/solr/collection1/conf/xslt/luke.xsl
solr-4.3.1/example/solr/collection1/conf/xslt/updateXml.xsl
solr-4.3.1/example/solr/solr.xml
solr-4.3.1/example/solr/zoo.cfg
solr-4.3.1/example/start.jar
solr-4.3.1/example/webapps/solr.war
solr-4.3.1/example/solr/bin/
solr-4.3.1/example/cloud-scripts/zkcli.sh
solr-4.3.1/example/etc/create-solrtest.keystore.sh
solr-4.3.1/example/exampledocs/post.sh
solr-4.3.1/example/exampledocs/test\_utf8.sh
```
Y queremos volcarlo omitiendo los niveles superiores ( `solr-4.3.1/example` ) directamente en la ruta que queramos.
```
mkdir -p /opt/solr\_liferay

cd /opt/solr\_liferay

tar -xvzf /tmp/solr-4.3.1.tgz solr-4.3.1/example -C . --strip-components=2
```
Donde el parámetro **\--strip-components=N** **es el número de niveles** a eliminar al descomprimir y **\-C es la ruta** en la que vamos a realizar la extracción.

#### Seguimos con una chuleta que ha salvado más de una vez.. **pasar ficheros de una máquina a otra usando** [NetCat](https://es.wikipedia.org/wiki/Netcat)

En la **máquina de destino**, la que va a recibir el fichero ejecutaremos esto:
``` 
operador@VASS-TONIO:/tmp$ nc -l -p 9999 > Fichero_Destino.txt
``` 

Y en la **máquina de origen**, la que envía el fichero, ejecutaremos esto:

``` 
operador@VASS-TONIO:/tmp$ nc localhost 9999 < Fichero_Origen.txt
``` 

El puerto elegido, el 9999, pues a gusto del consumidor o el que encontremos abierto (si tenemos un FW cortando comunicaciones podemos utilizar [PortQuiz](http://pruebadeconcepto.es/chuleta-02-rescantando-de-la-red/) como os dije entradas atrás).

En esta caso el origen ataca a localhost, pero es obvio que aquí va la máquina de destino...

#### Ahora vamos a ver **la forma de eliminar los ficheros de un directorio**

Pero no unos ficheros cualquiera, los **típicos millones de ficheros** que aparecen en un directorio cuando un proceso enloquece y no podemos ni hacer un ls dentro del directorio sin morir.

La primera forma de hacerlo, con un find y usando el parámetro `-print0` que devuelve el nombre de cada elemento sin realizar un retorno de carro para poder enviarlo a xargs que lo interpreta y evalúa línea a línea (y elimina...):

```
find . -name "*.log" -type f -print0 | xargs -0 rm -f
```

Otra forma de hacerlo, empleando rsync para machacar el directorio "infestado" de ficheros con otro vacío:

```
mkdir empty_dir; rsync -a --delete empty_dir/ directorio_hastalostopes/
```

Y la última forma, usando el todopoderoso Perl, haciendo un one-line del exterminio:

```
perl -e 'for(<*>){((stat)[9]<(unlink))}'
```

#### Por último, vamos a ver **un par de método para conocer nuestra IP pública**

Con la que nos presentamos en internet después de una suerte de conexiones, proxy y firewall.

```
wget --user-agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:21.0) Gecko/20100101 Firefox/21.0" ipinfo.io/ip; cat ip; rm ip*
```

Y la otra forma es:

```
wget -qO- http://ipecho.net/plain; echo
```

En función de si **atacamos por http o por https** nos llevaremos la sorpresa de si nuestra salida a internet encamina el tráfico en plano o seguro por distintas conexiones (algo que puede ser muy muy divertido con páginas que generan el token en plano y continúan con el login por seguro... da para una entrada de Informática Viejuna..)

... continuará ...
