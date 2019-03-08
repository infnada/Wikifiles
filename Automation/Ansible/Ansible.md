---
title: Ansible
description: 
published: true
date: 2019-03-08T20:26:24.016Z
tags: 
---

> https://www.ansible.com/

Recursos:
- https://docs.ansible.com/ansible/latest/modules/modules_by_category.html
- https://galaxy.ansible.com/

# Instalar Ansible
https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

`$ pip install ansible`

## Ansible Inventory ( ansible/hosts !!!)

> https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html

# Ejemplos

> Basados en https://wiki.isartnavarro.io/Automation/Vagrant/Vagrant

- Lanzar comandos simples

```
$ ansible -m ping all
$ ansible -m ping debian
$ ansible -m apt -a update_cache=true debian
$ ansible -m apt -a update_cache=true ubuntu
$ ansible -m apt -a "package=pwgen state=latest update_cache=true" debian
$ ansible -m apt -a "package=pwgen state=latest update_cache=true" ubuntu
```

- Utilización de Playbook

```
$ vi debian-install-apache2.yaml
---
- hosts: all
  #serial: 10
  tasks:
    - name: Install apache httpd but avoid starting it immediately (state=present is optional)
      apt:
        name: apache2
        state: present
```

`$ ansible-playbook -l debian debian-install-apache2.yaml`

## Ejemplos de Playbooks

- Package updates
```
- hosts: all
  #serial: 10
  tasks:
    - name: Upgrade Debian-Family
      apt: upgrade=yes update_cache=yes
      when: ansible_os_family == 'Debian'
      become: yes
      
    - name: Upgrade Centos-Family
      yum: name='*' update_only=yes update_cache=yes
      when: ansible_os_family == 'RedHat'
      become: yes
```

- Package updates + Apache + PHP
```
- hosts: all
  tasks:
    - name: Update all packages to the latest version
      apt:
         name: "*"
         state: latest
         force_apt_get: yes
      when: ansible_distribution == "Debian" or ansible_distribution == "Ubuntu"
    - name: Install Apache+PHP
      apt:
         name: "{{ packages }}"
      vars:
         packages:
         - apache2
         - libapache2-mod-php
      when: ansible_distribution == "Debian" or ansible_distribution == "Ubuntu"
    - name: Update all packages to the latest version
      yum:
         name: '*'
         state: latest
         exclude: kernel*
      when: ansible_distribution == "CentOS"
    - name: Install Apache+PHP
      yum:
         name: "{{ packages }}"
      vars:
         packages:
         - httpd
         - php
         - php-mysql
      when: ansible_distribution == "CentOS"
```

- Creación de usuarios

> Hay que crear el directorio `files` en local para copiar las claves ssh

```
- hosts: "all"
  #sudo: true
  vars:
    users:
    - "operador1"
    - "operador2"
    - "operador3"
  tasks:
  - name: "crear grupo operador"
    become: yes 
    group:
      name: operador
      state: present

  - name: "Create user accounts"
    user:
      name: "{{ item }}"
      groups: "operador"
    with_items: "{{ users }}"

  - name: "Add authorized keys"
    authorized_key:
      user: "{{ item }}"
      key: "{{ lookup('file', 'files/'+ item + '.pub') }}"    #   files/operador1.pub files/operador2.pub ...
    with_items: "{{ users }}"

  - name: "Allow admin users to sudo without a password"
    lineinfile:
      dest: "/etc/sudoers"
      state: "present"
      regexp: "^%operador"
      line: "%operador ALL=(ALL) NOPASSWD: ALL"

  - name: Install apache httpd but avoid starting it immediately (state=present is optional)
     package:
      name: httpd
     when: ansible_os_family == 'RedHat'
     become: yes
```

