---
title: "Monta tu blog en AWS por 0€ - parte 1"
date: "2019-05-08"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
categories:
  - "tecnologia"
tags:
  - "aws"
  - "command-line"
  - "ec2"
---

Monté este blog animado por la [capa gratuita](https://aws.amazon.com/es/free) de Amazón en su AWS.

Antes de esto había experimentado con [otras opciones](http://pruebadeconcepto.es/?p=1) de lo más variopintas, tuve **algún blog funcionando con una pequeña RaspberryPi sujeta con velcro** detrás de la televisión cercana al router... todo un ejercicio de optimización de recursos aquello... no puedes desperdiciar un byte de la escasa ram y optimizas el gasto de CPU como cuando de "mozo" mirabas cada línea de código pensando en que alguien vendría con los dichosos MIPS... (esto ya os lo contaré)

**AWS te permite tener los siguientes elementos** en su denominada free-tier:

- Instancia EC2 tipo t2.micro (750 horas mes)
- Instancia RDS db2.micro (750 horas mes)
- S3 5GB gratis 20k gets y 2k puts

Tenemos que en un mes ( de los largos de 31 días ) son unas 744 horas, así que nos regalan un mes + unas pocas horas, **durante un año**.

Y está aquí el truco, un año, vas a necesitar por tanto registrar año tras año una cuenta a la que mover tu servicio (tu servidor, tu base de datos, tus ficheros), a cambio de tenerlo todo gratis y **con una calidad que no tiene que envidiar a servicios dedicados** (para un blog personal de poco calado como este).

Así que.. empezamos, es posible que AWS no te deje crear nada hasta que no validen tu tarjeta de crédito, te suelen hacer un cargo de 1€, no recuerdo si luego lo devuelven, pero vamos, que por 1€ un año de alojamiento no está mal.

Y **no solo de alojamiento**, que **tendrás un "servidor" en Internet** para usos múltiples, tu VPN, máquina de salto, lanzar algunos "tests" (escaneo de puertos, pruebas de vulnerabilidades, etc.) y **todo lo que se te pueda ocurrir**.

Empezamos creado dos Security Groups.

**Web-SG** en el que vamos a dar acceso a todo el mundo para HTTP y HTTPS (80 y 443), y acceso SSH (22) para nuestras ips personales (la de casa, la del trabajo, etc.)

**RDS-SG** en el que vamos a dar acceso al servicio MySql con el origen del Security Group Web-SG, esto es, las peticiones a la base de datos podrán llegar únicamente desde el frontal web.

Esto lo **podéis hacer por consola**, o **ponerle un poco más de emoción** al asunto y hacerlo por línea de comando usando el [cliente](https://aws.amazon.com/es/cli/), al que tendréis que configurar unas credenciales de [IAM](https://aws.amazon.com/es/iam/) (más que contado en internet).

Creamos el Security Group WEB-SG, dentro de nuestra VPC por defecto (id vpc-b689b1XX) y capturamos la salida del comando para filtrar el identificador que nos genera AWS.
```
root@WOPR:~# aws ec2 create-security-group \\
--group-name "WEB-SG" \\
--description "Acceso WEB" \\
--vpc-id "vpc-b689b1XX" --output json | /usr/bin/jq '.GroupId' | tr -d '"'
sg-0022389898cf38226
```
Ahora nombramos a nuestro SG:

```
root@WOPR:~# aws ec2 create-tags \\
--resources "sg-0022389898cf38226" \\
--tags Key=Name,Value="WEB-SG"
```

Y por último abrimos las comunicaciones de los puertos indicados antes:

```
root@WOPR:~# aws ec2 authorize-security-group-ingress \\
--group-id "sg-0022389898cf38226" \\
--protocol tcp --port 22 \\
--cidr "XXX.XXX.XXX.XXX/32"

root@WOPR:~# aws ec2 authorize-security-group-ingress \\
--group-id "sg-0022389898cf38226" \\
--protocol tcp --port 80 \\
--cidr "0.0.0.0/0"

root@WOPR:~# aws ec2 authorize-security-group-ingress \\
--group-id "sg-0022389898cf38226" \\
--protocol tcp --port 443 \\
--cidr "0.0.0.0/0"
```

Creamos el Security Group RDS-SG:
```
root@WOPR:~# aws ec2 create-security-group \\
--group-name "RDS-SG" \\
--description "Acceso RDS" \\
--vpc-id "vpc-b689b1XX" --output json | /usr/bin/jq '.GroupId' | tr -d '"'
sg-0729ab6694bef3faa
```
Con ese identificador nombramos el SG:
```
root@WOPR:~# aws ec2 create-tags \\
--resources "sg-0729ab6694bef3faa" \\
--tags Key=Name,Value="RDS-SG"
```
Y abrimos las comunicaciones para MySql en el 3306 para el origen SG WEB-SG:
```
root@WOPR:~# aws ec2 authorize-security-group-ingress \\
--group-id "sg-0729ab6694bef3faa" \\
--protocol tcp --port 3306 \\
--source-group "sg-0022389898cf38226"
```
El resultado de estos comando en la consola será similar a esto:

![](../images/Selección_438.png)

Para poder crear (por fin) nuestra máquina en EC2 debemos tener:

1.- **KeyPair** para poder acceder a ella por SSH. La generamos así (guarda bien esta información!!)
```
pi@WOPR:~ $ aws ec2 create-key-pair --key-name "KP_MIEC2"
{
"KeyFingerprint": "4b:2e:5a:94:cb:6b:8a:ab:59:73:77:3f:4a:37:38:6c:d3:d2:5a:d7",
"KeyName": "KP_MIEC2",
"KeyMaterial": "-----BEGIN RSA PRIVATE KEY-----\\n TEXTO ELIMINADO INTENCIONADAMENTE \\n-----END RSA PRIVATE KEY-----"
}
```
2.- Conocer cuales son nuestras **subredes** y las **zonas de disponibilidad**:
```
pi@WOPR:~ $ aws ec2 describe-subnets --output text | awk '{print $3" "$12}'
eu-west-1a subnet-c1f2449b
eu-west-1c subnet-4630740e
eu-west-1b subnet-88a8ddee
```
(\* _notamental_... manejar mejor las salidas usando formato Json y filtrar con query jq...)
(\* otra _notamental_... cambiar todo este código disperso por Cloudformation o por un poco de Terraform... algún día...)

3.- Conocer el id de AMI que vamos a utilizar, para Ubuntu 18.04 es esta:
```
pi@WOPR:~ $ aws ec2 describe-images --owners 099720109477 --filters 'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-??????????' 'Name=state,Values=available' --output json | jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'
ami-0b2a4d260c54e8d3d
```
Con todo esto montamos un fichero (p.e. specification.json) que emplearemos ahora después como plantilla:  
**specifications.json**
```
{
"ImageId": "ami-0b2a4d260c54e8d3d",
"InstanceType": "t2.micro",
"KeyName": "KP_MIEC2",
"Placement": { "AvailabilityZone": "eu-west-1a" },
"SecurityGroupIds": [ "sg-072526f914e7202f8" ],
"SubnetId": "subnet-c1f2449b",
"DryRun": true
}
```
(\* _DryRun_ está a true para probar que todo va bien)

Y ahora si, por fin, vamos a generar nuestra máquina:
```
pi@WOPR:~ $ aws ec2 run-instances --cli-input-json file://specifications.json

An error occurred (DryRunOperation) when calling the RunInstances operation: Request would have succeeded, but DryRun flag is set.
```

Si el mensaje **es como este** podemos editar el fichero de especificaciones, poner _DryRun_ a false y volver a ejecutar para crear nuestra instancia.

**Para terminar esta primer parte**, vamos a conectarnos, empleando la clave SSH que tenemos descargada.  
Primero obtenemos **los datos de nuestra instancia**:
```
pi@WOPR:~ $ aws ec2 describe-instances --filter Name="instance-type",Values="t2.micro" Name="instance-state-name",Values="running" --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name,PrivateIpAddress,PublicIpAddress'] --output text
i-0594d4acc5XX1c465 t2.micro running 172.31.22.120 52.17.XX.126
```
Y nos conectamos a ella por SSH:
```
pi@WOPR:~ $ ssh -i KP_MIEC2.pem ubuntu@52.17.XX.126
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-1039-aws x86_64)
```
... [continuará](/tec/tec_blogawsparte2) ...
