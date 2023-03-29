---
title: "Chuleta 05 – Journal"
date: "2020-02-11"
categories: 
  - "chuletas"
---

A continuación una lista de opciones para ejecutar jornalctl y consultar el sistema de registro de systemd que pensamos que os será de utilidad.

### Seguir los mensajes del sistema en tiempo real

Empleamos el parámetro `-f`que nos dará salida contínua de las entradas del journal según de generen.

\[root@CENTOS7JEEP ~\]# journalctl -f
-- Logs begin at mar 2020-01-28 23:20:47 CET. --
ene 28 23:21:32 CENTOS7JEEP systemd\[1\]: Started Docker Application Container Engine.
ene 28 23:21:32 CENTOS7JEEP systemd\[1\]: Reached target Multi-User System.
ene 28 23:21:32 CENTOS7JEEP systemd\[1\]: Starting Update UTMP about System Runlevel Changes...
ene 28 23:21:32 CENTOS7JEEP systemd\[1\]: Started Update UTMP about System Runlevel Changes.

### Ver la lista de los arranques de la máquina

Empleamos el parámetro `--list-boots`que nos dará un listado de los inicios de máquina.

\[root@CENTOS7JEEP ~\]# journalctl --list-boots
-1 c6c999671b0945edac95628aa60ee53d mar 2020-01-28 23:33:45 CET—mar 2020-01-28 23:34:14 CET
 0 0fcba56bf217445f80bfe2dc2f5851d5 mar 2020-01-28 23:34:23 CET—mar 2020-01-28 23:34:40 CET

### Ver los logs del boot actual

Empleamos el parámetro `-b`que nos dará la información del último boot del equipo (o el ID de boot 0 si lo listamos).

\[root@CENTOS7JEEP ~\]# journalctl -b
-- Logs begin at mar 2020-01-28 23:20:47 CET, end at mar 2020-01-28 23:22:49 CET. --
ene 28 23:20:47 CENTOS7JEEP systemd-journal\[97\]: Runtime journal is using 8.0M (max allowed 189.4M, trying to leave 284.2M free o
ene 28 23:20:47 CENTOS7JEEP kernel: Initializing cgroup subsys cpuset
ene 28 23:20:47 CENTOS7JEEP kernel: Initializing cgroup subsys cpu
ene 28 23:20:47 CENTOS7JEEP kernel: Initializing cgroup subsys cpuacct

### Ver los logs de boot anteriores

Si necesitamos ver alguno de los anteriores tenemos dos opciones: Utilizamos simplemente **una cuenta regresiva**. Por ejemplo para ver al anterior emplearmos los parámetros `-b -1`

\[root@CENTOS7JEEP ~\]# journalctl -b -1
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at mar 2020-01-28 23:34:40 CET. --
ene 28 23:33:45 CENTOS7JEEP systemd-journal\[95\]: Runtime journal is using 8.0M (max allowed 189.4M, trying to leave 284.2M free of 1
ene 28 23:33:45 CENTOS7JEEP kernel: Initializing cgroup subsys cpuset
ene 28 23:33:45 CENTOS7JEEP kernel: Initializing cgroup subsys cpu
ene 28 23:33:45 CENTOS7JEEP kernel: Initializing cgroup subsys cpuacct

O elegimos **usar la ID del boot**, que nos aparecio al listar los procesos de arranque con `journal –list-boots` y emplenado la variable `_BOOT_ID=`

\[root@CENTOS7JEEP ~\]# journalctl \_BOOT\_ID=c6c999671b0945edac95628aa60ee53d
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at mar 2020-01-28 23:35:41 CET. --
ene 28 23:33:45 CENTOS7JEEP systemd-journal\[95\]: Runtime journal is using 8.0M (max allowed 189.4M, trying to leave 284.2M free of 1
ene 28 23:33:45 CENTOS7JEEP kernel: Initializing cgroup subsys cpuset

## Ver los mensajes del kernel

Utilizaremos el parámetro `-k`

\[root@CENTOS7JEEP ~\]# journalctl -k
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 22:27:31 CET. --
feb 08 22:26:58 CENTOS7JEEP kernel: Initializing cgroup subsys cpuset
feb 08 22:26:58 CENTOS7JEEP kernel: Initializing cgroup subsys cpu
feb 08 22:26:58 CENTOS7JEEP kernel: Initializing cgroup subsys cpuacct

Como suele pasar en la linea de comandos podemos combinar varios parámetros para afinar la búsqueda. Aquí vemos los mensajes referidos al kernel durante el **boot actual**:

\[root@CENTOS7JEEP ~\]# journalctl -b -k
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 22:35:30 CET. --
feb 08 22:34:06 CENTOS7JEEP kernel: Initializing cgroup subsys cpuset
feb 08 22:34:06 CENTOS7JEEP kernel: Initializing cgroup subsys cpu
feb 08 22:34:06 CENTOS7JEEP kernel: Initializing cgroup subsys cpuacct

## Filtrar por número de entradas en el registro de logs

La opción predeterminada es ejecutar usando el parámetro `-n`

\[root@CENTOS7JEEP ~\]# journalctl -n
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 22:35:30 CET. --
feb 08 22:34:16 CENTOS7JEEP systemd\[1\]: Startup finished in 541ms (kernel) + 2.086s (initrd) + 7.459s (userspace) = 10.087s.
feb 08 22:34:16 CENTOS7JEEP chronyd\[795\]: Selected source 162.159.200.1
feb 08 22:34:16 CENTOS7JEEP chronyd\[795\]: System clock wrong by 2.344175 seconds, adjustment started
feb 08 22:34:18 CENTOS7JEEP systemd\[1\]: Time has been changed
feb 08 22:34:18 CENTOS7JEEP chronyd\[795\]: System clock was stepped by 2.344175 seconds
feb 08 22:35:29 CENTOS7JEEP sshd\[1886\]: Accepted password for root from 10.0.2.2 port 54974 ssh2
feb 08 22:35:30 CENTOS7JEEP systemd\[1\]: Created slice User Slice of root.
feb 08 22:35:30 CENTOS7JEEP systemd\[1\]: Started Session 1 of user root.
feb 08 22:35:30 CENTOS7JEEP systemd-logind\[787\]: New session 1 of user root.
feb 08 22:35:30 CENTOS7JEEP sshd\[1886\]: pam\_unix(sshd:session): session opened for user root by (uid=0)

La salida que nos proporciona es de los últimos 10 mensajes.

Si indicamos un valor a continuación del parámetro podemos obtener esas N líneas de log

\[root@CENTOS7JEEP ~\]# journalctl -n 5
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 22:35:30 CET. --
feb 08 22:35:29 CENTOS7JEEP sshd\[1886\]: Accepted password for root from 10.0.2.2 port 54974 ssh2
feb 08 22:35:30 CENTOS7JEEP systemd\[1\]: Created slice User Slice of root.
feb 08 22:35:30 CENTOS7JEEP systemd\[1\]: Started Session 1 of user root.
feb 08 22:35:30 CENTOS7JEEP systemd-logind\[787\]: New session 1 of user root.
feb 08 22:35:30 CENTOS7JEEP sshd\[1886\]: pam\_unix(sshd:session): session opened for user root by (uid=0)

## Filtras los logs por ejecutables o programas

En este caso también tenemos varias formas de hacerlo, bien directamente con el ejecutable, usando `_COMM=<valor>`

\[root@CENTOS7JEEP ~\]# journalctl \_COMM=containerd
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 23:01:02 CET. --
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.313813235+01:00" level=info msg="starting containerd" revisi
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.316016691+01:00" level=info msg="loading plugin "io.containe
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.317083348+01:00" level=info msg="loading plugin "io.containe
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.322496150+01:00" level=warning msg="failed to load plugin io
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.322539469+01:00" level=info msg="loading plugin "io.containe
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.333122662+01:00" level=warning msg="failed to load plugin io
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.333729429+01:00" level=info msg="loading plugin "io.containe

o especificando la ruta

\[root@CENTOS7JEEP ~\]# journalctl /usr/bin/containerd
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 23:01:02 CET. --
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.313813235+01:00" level=info msg="starting containerd" revisi
ene 28 23:33:53 CENTOS7JEEP containerd\[1057\]: time="2020-01-28T23:33:53.316016691+01:00" level=info msg="loading plugin "io.containe

## Mostrar la salida por PID

Filtramos mediante el número identificador del proceso (algo que podemos consultar con `top` ), en esta ocasión como veis en el ejemplo 1053 corresponde a Docker, y usamos `_PID=<pid>`

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1053 root      20   0  520876  70768  26204 S   0,3  1,8   0:00.80 dockerd

\[root@CENTOS7JEEP ~\]# journalctl \_PID=1053
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 22:35:30 CET. --
ene 29 21:57:28 CENTOS7JEEP kdumpctl\[1053\]: kexec: loaded kdump kernel
ene 29 21:57:28 CENTOS7JEEP kdumpctl\[1053\]: Starting kdump: \[OK\]
-- Reboot --
feb 01 18:36:29 CENTOS7JEEP sshd\[1053\]: Server listening on 0.0.0.0 port 22.
feb 01 18:36:29 CENTOS7JEEP sshd\[1053\]: Server listening on :: port 22.
feb 01 19:46:37 CENTOS7JEEP sshd\[1053\]: Received signal 15; terminating.
-- Reboot --
feb 02 16:31:24 CENTOS7JEEP containerd\[1053\]: time="2020-02-02T16:31:24.198649259+01:00" level=info msg="starting containerd" revisi

## Especificar la salida por usuarios

Filtramos mediante el identificador del usuario y usamos `_UID=<uid>`

\[root@CENTOS7JEEP ~\]# journalctl \_UID=0
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 22:49:42 CET. --
ene 28 23:33:45 CENTOS7JEEP systemd-journal\[95\]: Runtime journal is using 8.0M (max allowed 189.4M, trying to leave 284.2M free of 1
ene 28 23:33:45 CENTOS7JEEP systemd-journal\[95\]: Journal started
ene 28 23:33:45 CENTOS7JEEP systemd\[1\]: Started Create Static Device Nodes in /dev.

## Filtrar la salida por servicios de systemd

Podemos ver los servicios que dependen de **systemd**, ejecutando con el parámetro `list-units -t service --all`

\[root@CENTOS7JEEP ~\]# systemctl list-units -t service --all
  UNIT                                                 LOAD      ACTIVE   SUB     DESCRIPTION
  auditd.service                                       loaded    active   running Security Auditing Service
  chronyd.service                                      loaded    active   running NTP client/server
  containerd.service                                   loaded    active   running containerd container runtime
  cpupower.service                                     loaded    inactive dead    Configure CPU power related settings
  crond.service                                        loaded    active   running Command Scheduler
  dbus.service                                         loaded    active   running D-Bus System Message Bus
● display-manager.service                              not-found inactive dead    display-manager.service
  dm-event.service                                     loaded    inactive dead    Device-mapper event daemon
  docker.service                                       loaded    active   running Docker Application Container Engine
  dracut-shutdown.service                              loaded    inactive dead    Restore /run/initramfs
  ebtables.service                                     loaded    inactive dead    Ethernet Bridge Filtering tables
  emergency.service                                    loaded    inactive dead    Emergency Shell
  firewalld.service                                    loaded    inactive dead    firewalld - dynamic firewall daemon

Si nos interesa uno en particular, estudiamos sus mensajes añadiendo el parámetro `-u` y el nombre del service, como en este ejemplo

\[root@CENTOS7JEEP ~\]# journalctl -u dbus.service
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 23:01:02 CET. --
ene 28 23:33:50 CENTOS7JEEP systemd\[1\]: Started D-Bus System Message Bus.
ene 28 23:33:51 CENTOS7JEEP dbus\[789\]: \[system\] Activating via systemd: service name='org.freedesktop.hostname1' unit='dbus-org.free
ene 28 23:33:51 CENTOS7JEEP dbus\[789\]: \[system\] Successfully activated service 'org.freedesktop.hostname1'

## Filtrar por fechas

Se utilizan los parámetros `–since` y `–until`, así como expresiones tipo `yesterday` `ago` ó `today`.

El formato de tiempo es habitualmente **YYYY-MM-DONDE HH:MM:SS**.

\[root@CENTOS7JEEP ~\]# journalctl --since 'today' --until '23:30'
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 23:01:02 CET. --
feb 08 22:26:58 CENTOS7JEEP systemd-journal\[97\]: Runtime journal is using 8.0M (max allowed 189.4M, trying to leave 284.2M free of 1
feb 08 22:26:58 CENTOS7JEEP kernel: Initializing cgroup subsys cpuset
feb 08 22:26:58 CENTOS7JEEP kernel: Initializing cgroup subsys cpu

\[root@CENTOS7JEEP ~\]# journalctl --since='2020-01-28 00:00' --until='2020-01-29 23:59'
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 23:01:02 CET. --
ene 28 23:33:45 CENTOS7JEEP systemd-journal\[95\]: Runtime journal is using 8.0M (max allowed 189.4M, trying to leave 284.2M free of 1
ene 28 23:33:45 CENTOS7JEEP kernel: Initializing cgroup subsys cpuset

Se pueden ir "**concatenando**" condiciones de manera que veamos la actividad de un programa concreto durante unas fechas concretas:

\[root@CENTOS7JEEP ~\]# journalctl \_COMM=containerd --since='2020-02-01 00:00' --until='2020-02-08 00:00'
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 23:36:39 CET. --
feb 01 18:36:38 CENTOS7JEEP containerd\[1060\]: time="2020-02-01T18:36:38.664444543+01:00" level=info msg="starting containerd" revisi
feb 01 18:36:38 CENTOS7JEEP containerd\[1060\]: time="2020-02-01T18:36:38.667845640+01:00" level=info msg="loading plugin "io.containe
feb 01 18:36:38 CENTOS7JEEP containerd\[1060\]: time="2020-02-01T18:36:38.687200426+01:00" level=info msg="loading plugin "io.containe

Un recurso muy utilizado, ver la actividad del sistema de los últimos 10 minutos:

\[root@CENTOS7JEEP ~\]# journalctl --since='10 min ago'
-- Logs begin at mar 2020-01-28 23:33:45 CET, end at sáb 2020-02-08 23:36:39 CET. --
feb 08 22:49:42 CENTOS7JEEP systemd\[1\]: Starting Cleanup of Temporary Directories...
feb 08 22:49:42 CENTOS7JEEP systemd\[1\]: Started Cleanup of Temporary Directories.
feb 08 23:01:02 CENTOS7JEEP systemd\[1\]: Started Session 2 of user root.

## Filtrar por la prioridad del mensaje

Existen 7 niveles de clasificación de los mensajes según prioridad:

<table style="border-collapse: collapse; width: 9.87821%; height: 199px;"><tbody><tr><td style="width: 50%;">0</td><td style="width: 50%;">emerg</td></tr><tr><td style="width: 50%;">1</td><td style="width: 50%;">alert</td></tr><tr><td style="width: 50%;">2</td><td style="width: 50%;">critical</td></tr><tr><td style="width: 50%;">3</td><td style="width: 50%;">error</td></tr><tr><td style="width: 50%;">4</td><td style="width: 50%;">notice</td></tr><tr><td style="width: 50%;">5</td><td style="width: 50%;">notice</td></tr><tr><td style="width: 50%;">6</td><td style="width: 50%;">info</td></tr><tr><td style="width: 50%;">7</td><td style="width: 50%;">debug</td></tr></tbody></table>

Para filtrarlos utilizamos el parámetro `-p` seguido del número correspondiente

\`journalctl -p 2\`

## Espacio ocupado por los logs

Para poder ver lo que ocupa nuestro journal en la máquina empleamos el parámetros `--disk-usage`

\[cloud\_user@ahernan1c ~\]$ journalctl --disk-usage
Archived and active journals take up 24.0M on disk.

## Purgado de logs

Si los logs ocupan mucho podemos limitar su crecimiento editando el fichero `/etc/systemd/journald.conf`

\[cloud\_user@ahernan1c ~\]$ sudo vi /etc/systemd/journald.conf

\[Journal\]
Storage=persistent
SystemMaxFileSize=10M

\[cloud\_user@ahernan1c ~\]$ sudo systemctl restart systemd-journald

Si queremos vaciar journal lo haremos con la siguiente orden

\[cloud\_user@ahernan1c ~\]$ sudo journalctl --vacuum-size=5M
Vacuuming done, freed 0B of archived journals on disk.

Por último, una nota sobre **JournalD** bajo distribución **CentOS**.

Este SO por defecto eliminar en la parada del servidor toda la información, de manera que perdemos todo lo que sea anterior al último boot.

Es posible que queramos esta información, así que tenemos que editar el fichero `journalctl.conf` y cambiar el parámetro de tipo de almancenamiento

\[Journal\]
Storage=persistent
