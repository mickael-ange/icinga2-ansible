Role Name
=========

Ansible role to install Icinga2 Web2 Ui

Requirements
------------

Mysql or MariaDB - Httpd or Nginx - PHP

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
     icinga2_conf_global: |
       include "constants.conf"
       include "zones.conf"
       include <itl>
       include <plugins>
       include "features-enabled/*.conf"
       include_recursive "conf.d"
     check_commands:
       check_nrpe: |
          "-H", "$address$",
              "-c", "$remote_nrpe_command$",
     tags: icinga2-no-ui
   -  role: icinga2-ansible-web2-ui
      become: yes
      tags: icinga2-ansible-web2-ui

```

Then Go to http://IP/icingaweb2 and use icingaadmin/icingaadmin to login.

Role Variables
--------------

See `defaults/main.yml`

License
-------

GNU General Public License Version 2

Author Information
------------------

Valentino Gagliardi - Icinga Dev Team

