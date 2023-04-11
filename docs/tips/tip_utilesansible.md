---
title: "Chuleta 01 - Comandos útiles con ansible"
date: "2019-05-24"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
categories: 
  - "chuletas"
tags: 
  - "ansible"
---

Después de media vida lanzado comandos con más o menos grado de automatización / orquestación en servidores te cruzas con [Ansible](https://www.ansible.com) y te das cuenta de la cantidad de tiempo que te hace ganar, el control que tienes sobre tu entorno desde un nodo de bastionado, etc. son todo ventajas, pese a la complejidad que puedan tomar algunos playbooks / tasks si eres un poco maniático como yo.

Aquí van unas cuantas líneas de comando a modo de habituales que me gusta tener a mano.

Comprobar que tenemos conexión con todos los nodos:

`ansible all -i inventario.yml -m ping -o`

Reiniciar todas las máquinas de nuestro inventario:

`ansible all -i inventario.yml -b -B 1 -P 0 -m shell -a "sleep 5 && reboot"`

Parar/arrancar un servicio:

`ansible all -i inventario.yml -m systemd -a "service=docker state=\[stopped|started\]"`

Copiar ficheros:

`ansible all -i inventario.yml -m copy -a "src=/<fichero\_origen> dest=/<fichero\_destino>"`

Borrar ficheros:

`ansible all -i inventario.yml -m file -a "dest=/etc/sysconfig/docker-storage state=absent"`

Conocer la configuración (soft/hard) de los nodos, y filtrado de resultados:

`ansible localhost -m setup`

`ansible localhost -m setup -a 'filter=ansible\_processor\*'`

Listar todos los nodos del inventario:

`ansible all --list-hosts`

Instalar paquetes con Yum:

`ansible webservers -m yum -a "name=httpd state=present" -b -o`

Desinstalar paquetes con Yum:

`ansible webservers -m yum -a "name=httpd state=absent" -b -o`

(\*) en estos dos últimos ejemplos usamos -b (--become) para que se utilize la configuración de ansible.cfg relativa al [escalado de privilegios](https://docs.ansible.com/ansible/2.4/intro_configuration.html#become) con el que se ejecutarán los comandos en destino.

Actualizar todos los paquetes de las maquinas:

`ansible all -i inventario.yml -m yum -a "name=\* state=latest"`

Chequear sintaxis de PlayBook:

`ansible-playbook --syntax-check install\_apache.yml`

Continuará...
