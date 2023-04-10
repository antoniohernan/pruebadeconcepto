---
title: "AWS Route53 como DNS dinámico y Certificado SSL LetsEncrypt"
date: "2019-05-27"
categories: 
  - "tecnologia"
tags: 
  - "aws"
  - "ec2"
  - "route-53"
---

En la [segunda entrega](http://pruebadeconcepto.es/?p=140) de como montar tu servidor web en AWS por 0€ os mostraba como tener un dominio gratuito que atendiese a los cambios de IP de nuestro servidor.

Ahora os voy a enseñar una forma un poco más refinada utilizando el [servicio Route 53 de AWS](https://aws.amazon.com/es/route53/), que por desgracia **no forma parte de la free tier.**..

En mi empresa tenemos una cuenta AWS, digamos que para usos múltiples... es la cuenta de cacharreo y de la que han salido unas cuantas pruebas de concepto que a día de hoy podemos decir que son productos vendibles (estos con el tiempo los iré desgranando por aquí).

Hace tiempo que me cansé de pedir registros DNS internos y creé una **zona hospedada en Route53**, con esto consigue tener el **control completo de tu "subdominio" en tu empresa** sin pasar por el staff de sistemas.

Lo único que necesitarás de ellos es que hagan la **delegación de la zona en el DNS,** y a partir de ese momento todos lo hosts que deis de alta en vuestra zona hospedada será automáticamente de vuestra empresa.

Para la delegación de DNS **tan sólo deben apuntar a los NS que Route 53** asigna a la zona para el subdominio que hemos creado, y tardará en función del tiempo de refresco de los DNS.

Dicho y hecho entonces, tenemos nuestra zona hospedada y nuestra delegación de DNS, y ahora vamos a crear una **máquina nueva**, la cual **se va a presentar a nuestro Route53** con su ip pública para generar la entrada si no existe o actualizarla si existe.

Con esto **nos ahorramos Elastic IP,** el tener que estar recordando las Elastic IP que se han quedado sin asignar o por máquina apagada (que cuestan dinero) y **ganamos en la gestión de estancias efímeras como sería la creación de instancias Spot**, que con este método podemos tener con mucho más control.

Al lío, lo **primero es obtener el identificador de nuestra zona** (porque podemos tener más de una:

ubuntu@ip-172-18-10-38:~$ aws route53 list-hosted-zones-by-name | jq --arg name "manoloybenito.es."  -r '.HostedZones | .\[\] | select(.Name=="\\($name)") | .Id'
/hostedzone/Z2NYXJXXXXQOLJSF

En **segundo lugar vamos a obtener la IP pública** del servidor que se va a presentar al Route53, para esto empleamos una consulta con Curl al propio servidor, todas las instancias EC2 tienen esto:

ubuntu@ip-172-18-10-38:~$ curl http://169.254.169.254/latest/meta-data/public-ipv4
52.214.3.X3

En **tercer lugar, tenemos que modificar el valor de esta IP en la plantilla .json** que tenemos para nuestra zona, esto lo podemos hacer en línea de comando con sed, aunque más abajo podréis encontrar un enlace a mi gthub con todo el script.

La **plantilla sería** esta:

{
            "Comment": "CREATE/DELETE/UPSERT a record ",
            "Changes": \[{
            "Action": "CREATE",
                        "ResourceRecordSet": {
                                    "Name": "NOMBREHOST.manoloybenito.es",
                                    "Type": "A",
                                    "TTL": 300,
                                 "ResourceRecords": \[{ "Value": "IP\_PUBLICA"}\]
}}\]
}

Con la IP cambiada en esta plantilla, y el nombre del servidor sólo nos queda invocar el método de creación/cambio en nuestra zona de Route 53

aws route53 change-resource-record-sets --hosted-zone-id /hostedzone/Z2NYXJXXXXJSF --change-batch file://plantilla.json

A los pocos segundos / minutos **tendremos resolución del nombre**.

En [mi github](https://github.com/antoniohernan/route53ssl) tenéis la plantilla y el script completo.

**Segunda parte**, ya que tenemos el control absoluto de nuestro DNS **vamos a generar los certificados SSL** de [LetsEncrypt](https://letsencrypt.org/) aprovechándonos del [plugin para route53](https://certbot-dns-route53.readthedocs.io/en/stable/).

**Instalamos CertBot asi como el plugin** que se instala con pip:

$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository ppa:certbot/certbot
$ sudo apt update
$ sudo apt install certbot

$ sudo apt install python3-pip
$ sudo pip3 install certbot-dns-route53

Para generar y descargar el certificado, la línea de comando sería similar a esta:

certbot certonly -d NOMBREHOST.manoloybenito.es --dns-route53 \\
--logs-dir /home/ubuntu/letsencrypt/log/ --config-dir /home/ubuntu/letsencrypt/config/ \\
--work-dir /home/ubuntu/letsencrypt/work/ -m toor@manoloybenito.es \\
--agree-tos --non-interactive --server https://acme-v02.api.letsencrypt.org/directory

Para las renovaciones de los certificados, la línea de comando sería esta:

certbot renew --dns-route53 --logs-dir /home/ubuntu/letsencrypt/log/ --config-dir /home/ubuntu/letsencrypt/config/ \\
--work-dir /home/ubuntu/letsencrypt/work/ --non-interactive \\
--server https://acme-v02.api.letsencrypt.org/directory

Espero os sea de utilidad.
