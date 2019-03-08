---
title: Vagrant
description: 
published: true
date: 2019-03-08T20:12:32.403Z
tags: 
---

> https://Vagrantup.com

Recursos:
- https://vagrantcloud.com/boxes/search
- https://www.vagrantup.com/docs/virtualbox/networking.html
- https://www.vagrantup.com/intro/getting-started/

# Instalación

> https://www.vagrantup.com/downloads.html
`$ apt install vagrant`


# Comandos básicos

- Crear Vagrantfile inicial

`$ vagrant init`

- Ver listado de VMs activas

`$ vagrant box list`

- Ver puertos de una VM

`$ vagrant port $boxname`

- Interactuar con VMs

`$ vagrant shutdown/suspend/resume/destroy/ssh-config...`

# Tu primer Vagrant

```
$ cd 
$ mkdir -p ~/MyVagrants/example1
$ cd ~/MyVagrants/example1
$ vagrant init hashicorp/precise64
$ vagrant up
```

# Ejemplos

### Crea un vagrant basado en stretch64 que ejecute apache + php ( libapache2-mod-php*) que sirva el "index.php" detallado abajo. 

Mapea el puerto 8080 del anfitrión al 80 de la VM de Vagrant.

```
$ vi Vagrantfile
---
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/precise64"
  config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.network :forwarded_port, guest: 80, host: 8880
  #config.vm.box_url = "https://vagrantcloud.com/hashicorp/precise64"
end

$ vi bootstrap.sh
---
apt-get update
apt-get -y install apache2 libapache2-mod-php5
mount --bind /vagrant /var/www/html

$ vi index.php
---
&lt;?php
	printf ("Hola Mundo!\n" );
?>
```
`$ vagrant up`


### Cómo levantaremos 2 máquinas en red?

```
$ vi Vagrantfile
---
$mi_script = &lt;&lt;SCRIPT
apt -y update
apt -y install xxx
rm -rf xxx
xxx
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.define "apache" do |config|
    config.vm.box = "loquesea/imagen1"
    config.vm.hostname = "apachefrontal"
    config.vm.network "private_network", ip: "10.0.7.11"
    config.vm.provision "shell", inline: $mi_script
    config.vm.synced_folder ".", "/vagrant", disabled: true
  end

  config.vm.define "mysql" do |config|
    config.vm.box = "loquesea/imagen1"
    config.vm.hostname = "mysqlbackend"
    config.vm.network "private_network", ip: "10.0.7.12"
    config.vm.provision "shell", inline: $otro_script
    config.vm.synced_folder ".", "/vagrant", disabled: true
  end
end
```

### Levantar un Docker Swarm

> Faltaría poner las guest additions para que se comparta el "jointoken"

```
$ vi Vagrantfile
---
$docker = &lt;&lt;SCRIPT
apt-get -y update
apt-get -y install curl apt-transport-https
curl -s https://get.docker.com | bash
usermod -aG docker vagrant
SCRIPT

$swarminit = &lt;&lt;SCRIPT
docker swarm init --advertise-addr 10.0.7.11
docker swarm join-token manager | grep swarm | tail -1 > /vagrant/jointoken.txt
SCRIPT

$swarmjoin = &lt;&lt;SCRIPT
bash /vagrant/jointoken.txt
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.define "swarm1" do |config|
    config.vm.box = "debian/stretch64"
    config.vm.hostname = "swarm1"
    config.vm.network "private_network", ip: "10.0.7.11"
    config.vm.provision "shell", inline: $docker
    config.vm.provision "shell", inline: $swarminit
		#config.vm.synced_folder ".", "/vagrant", disabled: true
  end

  config.vm.define "swarm2" do |config|
    config.vm.box = "debian/stretch64"
    config.vm.hostname = "swarm2"
    config.vm.network "private_network", ip: "10.0.7.12"
    config.vm.provision "shell", inline: $docker
    config.vm.provision "shell", inline: $swarmjoin
		#config.vm.synced_folder ".", "/vagrant", disabled: true
  end

  config.vm.define "swarm3" do |config|
    config.vm.box = "debian/stretch64"
    config.vm.hostname = "swarm3"
    config.vm.network "private_network", ip: "10.0.7.13"
    config.vm.provision "shell", inline: $docker
    config.vm.provision "shell", inline: $swarmjoin
		#config.vm.synced_folder ".", "/vagrant", disabled: true
  end

end
```

### Laboratorio Ansible:

- Creamos las máquinas
> Create una key ssh `ed25519` o utiliza la que prefieras

```
$ mkdir laboratorio-ansible
$ cd laboratorio-ansible
$ cp ~/.ssh/id_ed25519.pub .
$ vi Vagrantfile
---
$SSH = &lt;&lt;SCRIPT
        mkdir -m 0700 /root/.ssh
        cp /vagrant/id_ed25519.pub /root/.ssh/authorized_keys
        cat /vagrant/id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
        chmod 600 /root/.ssh/authorized_keys
SCRIPT

$InstallPythonAndSSH = &lt;&lt;SCRIPT
        apt -y install python
        cp /vagrant/id_ed25519.pub /root/.ssh/authorized_keys
        cat /vagrant/id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
        chmod 600 /root/.ssh/authorized_keys
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.define "centos01" do |config|
    config.vm.box = "centos/7"
    config.vm.hostname = "centos01"
    config.vm.provision "shell", inline: $SSH
    config.vm.network :forwarded_port, guest: 22, host: 64000
  end

  config.vm.define "debian01" do |config|
    config.vm.box = "debian/jessie64"
    config.vm.hostname = "debian01"
    config.vm.provision "shell", inline: $SSH
    config.vm.network :forwarded_port, guest: 22, host: 64001
  end

  config.vm.define "debian02" do |config|
    config.vm.box = "debian/jessie64"
    config.vm.hostname = "debian02"
    config.vm.provision "shell", inline: $SSH
    config.vm.network :forwarded_port, guest: 22, host: 64002
  end

  config.vm.define "ubuntu01" do |config|
    config.vm.box = "ubuntu/xenial64"
    config.vm.hostname = "ubuntu01"
    config.vm.provision "shell", inline: $InstallPythonAndSSH
    config.vm.network :forwarded_port, guest: 22, host: 64003
  end

end
```
- Modificamos el archivo hosts de Ansible
> Modifica los puertos SSH correspondientes
```
$ vi /etc/ansible/hosts
---
RBA: ejemplo 
[debian]
debian01        ansible_host=127.0.0.1 ansible_ssh_port=64001 ansible_ssh_user=root
debian02        ansible_host=127.0.0.1 ansible_ssh_port=64002 ansible_ssh_user=root

[centos]
centos01        ansible_host=127.0.0.1 ansible_ssh_port=64000 ansible_ssh_user=root

[ubuntu]
ubuntu01        ansible_host=127.0.0.1 ansible_ssh_port=64003 ansible_ssh_user=root

[laboratorio:children]
centos
debian
ubuntu
```