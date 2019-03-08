---
title: Centos 7 hardering
description: 
published: true
date: 2019-03-08T17:36:16.315Z
tags: 
---

#Hardering Centos 7 minimal

##Particionado de disco
* Swap 2G --> swap
* / 8G --> xfs bajo LVM
* /boot 488Mb --> ext4
* /var 1G --> xfs bajo LVM
* /tmp 488Mb --> xfs bajo LVM
* /home 200Mb --> xfs bajo LVM

```
$ vi /etc/fstab
-------
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=11111111-1111-1111-1111-111111111111 /boot ext4    defaults        1 2
/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-tmp  /tmp                    xfs     defaults,nosuid,noexec,nodev        0 0
/dev/mapper/centos-var  /var                    xfs     defaults,nosuid        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
---
```

##Configuración NTP

```
$ yum install ntp ntpdate
$ vi /etc/ntp.conf
---
server IP_NTP_INTERNO iburst
---
$ systemctl enable ntpd
$ ntpdate IP_NTP_INTERNO
$ systemctl start ntpd
```

##Deshabilitar montaje de USB

`$ echo "install usb-storage /bin/false" > /etc/modprobe.d/usb-storage.conf`

##Seguridad de contraseñas
- Habilitar SHA512 por contra de MD5

```
authconfig --passalgo=sha512 --update
```

- Configurar políticas
```
vi /etc/security/pwquality.conf
  difok = 5
  minlen = 8
  dcredit = 1
  ucredit = 1
  lcredit = 1
  ocredit = 1
  minclass = 4
  maxrepeat = 3
  maxclassrepeat = 3
  gecoscheck = 1
vi /etc/login.defs
  PASS_MAX_DAYS   180
  PASS_MIN_DAYS   1
  PASS_MIN_LEN    8
```

- Loguear intentos erroneos de login

```
vi /etc/pam.d/system-auth
  session required pam_lastlog.so showfailed
```

- Reintentos de login

```
vi /etc/pam.d/password-auth
  password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
```

- Bloqueo al superar el máximo de reintentos

```
vi /etc/pam.d/password-auth
  auth [default=die] pam_faillock.so authfail deny=3 unlock_time=604800 fail_interval=900
  auth required pam_faillock.so authsucc deny=3 unlock_time=604800 fail_interval=900
```

- Prevenir login sin password
```
sed -i 's/\<nullok\>//g' /etc/pam.d/system-auth
```

##Configuración SSHD
```
vi /etc/ssh/sshd_config
## Oblicación del protoclo 2 de ssh
   Protocol 2
## Tiempo d'espera de la pantalla de longin antes de cerrarse
  LoginGraceTime 1m
## no permitir aceso como root
  PermitRootLogin no
## Max intentos de login
  MaxAuthTries 3
## Cada cuando enviará un mensaje keepalive (5 min)
  ClientAliveInterval 300
## Cuantos keepalive deja abans de cerrar sesión (15 min)
  ClientAliveCountMax 3
## Deshabilitar el acceso de rshosts
  IgnoreRhosts yes
## Deshabilitar el acceso sin password
 HostbasedAuthentication no
## No permitir contraseñas en blanco
  PermitEmptyPasswords no
## No permitir enviar variables de entorno
PermitUserEnvironment no
## Ciphers
  Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc,aes192-cbc,aes256-cbc

```

##Instalar AIDE
```
yum install aide -y && /usr/sbin/aide --init && cp /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz && /usr/sbin/aide --check
aide --update
```

##Instalar RKHunter
```
yum -y install rkhunter
rkhunter --update
......
modificar configuración al gusto
......
rkhunter --propupd
```

> Después de una modificación de archivo y estar seguro de que esta es correcta, ejecutar lo siguente:
```
rkhunter -c --enable all --disable none --rwo
```
> Listado de tests
```
rkhunter --list
```

##Configurar GRUB
- Verificar permisos
```
chmod 600 /boot/grub2/grub.cfg
  -rw-------. 1 root root 4289 nov  8 12:09 /boot/grub2/grub.cfg
```

- Configurar contraseña
```
cd /etc/grub.d/
grub2-mkpasswd-pbkdf2
  Introduzca la contraseña:
  Reintroduzca la contraseña:
  El hash PBKDF2 de su contraseña es grub.pbkdf2.sha512.10000.E25620C940319CD5B8AD237025E328121951EAFEBE90217DD1B6BE88992BCADDFA856365FC302575BCD58107CE20A39D875EC89EBA6DE16A5958B357C3260F98.69577375971964E38A307A41BC66B7C3F51BE8E94F78681369E6EFF3953E7DD561C261448E654B7B939EF55ACACB255E541B21925D9615E6194ED5D5B34B5BA1
vi 40_custom
  # define superusers
  set superusers="g2r00t"

  #define users
  password_pbkdf2 g2r00t grub.pbkdf2.sha512.10000.E25620C940319CD5B8AD237025E328121951EAFEBE90217DD1B6BE88992BCADDFA856365FC302575BCD58107CE20A39D875EC89EBA6DE16A5958B357C3260F98.69577375971964E38A307A41BC66B7C3F51BE8E94F78681369E6EFF3953E7DD561C261448E654B7B939EF55ACACB255E541B21925D9615E6194ED5D5B34B5BA1
grub2-mkconfig -o /boot/grub2/grub.cfg
```

##Networking
- Deshabilitar IPv6
```
vi /etc/modprobe.d/disabled.conf
  options ipv6 disable=1
vi /etc/sysconfig/network
  NETWORKING_IPV6=no
  IPV6INIT=no
```

- Deshabilitar PIPA (DHCP)
```
echo "NOZEROCONF=yes" >> /etc/sysconfig/network
```

- Otras configuraciones

> http://unaaldia.hispasec.com/2002/11/opciones-de-seguridad-en-linux-traves.html

```
## No envia paquetes entre ifaces
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

## Numero mñaximo de peticiones de conectiones accordadas
net.ipv4.tcp_max_syn_backlog = 1280

## Ignora paquetes ICMP 
net.ipv4.icmp_echo_ignore_broadcasts = 1

## Deshabilitar  el control de rutas 
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

## Deshabilitar la aceptación de redirecciones
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0

## Habilitar registro de actividad sospechosa
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

## Protección contra IPs no válidas
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

## Alguns routers trenquen el RFC1122 i envien falsos respostes per broadcast, evitem que kernel mostri un Warning
net.ipv4.icmp_ignore_bogus_error_responses = 1

## Si el kernel esta compilat amb CONFIG_SYN_COOKIES envia fora les syncookies quan el socket esta sobrepasat
net.ipv4.tcp_syncookies = 1
```

- Configuración estandar
```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPADDR=XXX.XXX.XXX.XXX
NETMASK=255.255.255.0
GATEWAY=XXX.XXX.XXX.XXY
IPV4_FAILURE_FATAL=no
IPV6INIT=no
IPV6_AUTOCONF=no
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens192
UUID=11111111-1111-1111-1111-111111111111
DEVICE=ens192
ONBOOT=yes
ZONE=PublicDMZ
```

- Congiguración DNS
```
vi /etc/resolv.conf
  search tudominio.local
  nameserver IP_DNS_INTERNO_1
  nameserver IP_DNS_INTERNO_2
```

##Firewalld
```
# Configuración Firewall por defecto
  firewall-cmd --permanent --new-zone=PublicDMZ/Local
  firewall-cmd --zone=PublicDMZ --add-interface=ens192
  firewall-cmd --reload  
  firewall-cmd --set-default-zone=PublicDMZ
  firewall-cmd --reload
###### Rules

### OUTPUT
# Connexión establecida
  firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -m state --state ESTABLISHED,RELATED -j ACCEPT
# DNS
  firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p udp -d IP_DNS_INTERNO_1 --dport 53 -j ACCEPT
  firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p udp -d IP_DNS_INTERNO_2 --dport 53 -j ACCEPT
# SMTP 
  firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p tcp -d IP_SMTP_INTERNO --dport 25 -j ACCEPT
# NTP 
  firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p udp -d IP_NTP_INTERNO --dport 123 -j ACCEPT
# Drop all
  firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 2 -j DROP
  
### INPUT
# PING desde servidor de monitoring y LAN
  firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -p icmp -s NETWORL/24 -j ACCEPT
  firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -p icmp -s IP_MONITORING_SERVER -j ACCEPT
# SSH des de LAN
  firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -p tcp -s NETWORL/24 --dport 22 -j ACCEPT
```

##Deshabilitar funciones por defecto
>  Deshabilitar protocolos no usados
- Protocol de contgrol de Datagramas orientat a missatges 
```
echo "install dccp /bin/false" > /etc/modprobe.d/dccp.conf
```
- Stream Control Transmission Protocol Alternativa a TCP/UDP pero permite mensajes sin orden
```
echo "install sctp /bin/false" > /etc/modprobe.d/sctp.conf
```
- Reliable Datagram Sockets envio de datagramas 
```
echo "install rds /bin/false" > /etc/modprobe.d/rds.conf
```
- Transparent Inter-process Communication comunicación intra cluster
```
echo "install tipc /bin/false" > /etc/modprobe.d/tipc.conf
```
> Desinstalar paquetes no usados
- Postfix
```
systemctl stop postfix
systemctl disable postfix
yum remove postfix
rm -rf /etc/postfix/
```

##Otras configuraciones
- Configurar single mode con contraseña
```
vi /etc/sysconfig/init
  SINGLE=/sbin/sulogin
```

- Instalacion de paquetes recomentados
irqbalance --> redistribución de IRQ loads multiples CPUs
psacct --> Herramientas de monitorització de processos
screen
open-vm-tools
```
yum -y install irqbalance psacct screen open-vm-tools
systemctl enable irqbalance
systemctl enable psacct
```

- Habilitar unmask 077
```
vi /etc/bashrc
   umask 077
   umask 077
vi /etc/csh.cshrc
   umask 077
   umask 077
```

- Deshabilitar filesystems infrecuentes
```
echo "install cramfs /bin/false" > /etc/modprobe.d/cramfs.conf
echo "install freevxfs /bin/false" > /etc/modprobe.d/freevxfs.conf
echo "install jffs2 /bin/false" > /etc/modprobe.d/jffs2.conf
echo "install hfs /bin/false" > /etc/modprobe.d/hfs.conf
echo "install hfsplus /bin/false" > /etc/modprobe.d/hfsplus.conf
echo "install squashfs /bin/false" > /etc/modprobe.d/squashfs.conf
echo "install udf /bin/false" > /etc/modprobe.d/udf.conf
```

- Habilitar la protección de desbordamento de buffer
```
echo "kernel.exec-shield = 1" >> /etc/sysctl.conf
sysctl -w kernel.exec-shield=1
```

- Habilitem el Address Space Layuot Randomization (ASLR) prevención de explotación de vulnerabilidades a memoria
```
sysctl -q -n -w kernel.randomize_va_space=2
```
