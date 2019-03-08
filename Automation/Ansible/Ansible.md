---
title: Ansible
description: 
published: true
date: 2019-03-08T20:26:54.980Z
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
