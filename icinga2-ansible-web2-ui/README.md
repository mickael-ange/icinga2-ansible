Role Name
=========

Ansible role to install Icinga2 Web2 Ui

Requirements
------------

Mysql or MariaDB or PostgreSQL - Httpd or Nginx - PHP

Dependencies
------------

Roles: **icinga2-ansible-no-ui**

Example Playbook
----------------

```yaml
---
- hosts: monitoring_servers
  roles:
   # You can use librarian-ansible: role "php", github: "geerlingguy/ansible-role-php"
   - role: php
     become: yes
   - role: icinga2-ansible-no-ui
     become: yes
     tags: icinga2-no-ui
   - role: icinga2-ansible-web2-ui
     become: yes
     tags: icinga2-ansible-web2-ui

```

Then Go to http://IP/icingaweb2 and use icingaadmin/icingaadmin to login.

Role Variables
--------------

See variables:
* [defaults/main.yml](defaults/main.yml)
* [vars/mysql-ido-backend.yml](vars/mysql-ido-backend.yml) for MySQL IDO backend variables
* [vars/mysql-web2-backend.yml](vars/mysql-web2-backend.yml) for MySQL Web2 backend variables
* [vars/pqsql-ido-backend.yml](vars/pqsql-ido-backend.yml) for PgSQL IDO backend variables
* See [vars/pqsql-web2-backend.yml](vars/pqsql-web2-backend.yml) for PgSQL Web2 backend variables

License
-------

GNU General Public License Version 2

Author Information
------------------

Valentino Gagliardi - Icinga Dev Team

Contributors Information
------------------------

* Mickael Ange <akiran28@hotmail.com>
