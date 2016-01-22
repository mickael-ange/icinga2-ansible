Role Name
========

Ansible role to show a way to provision custom Icinga2 config:


Dependencies
------------

None

Requirements
------------

Roles: **icinga2-ansible-web2-ui**
Roles: **icinga2-pnp4nagios**

Example Playbook
-------------------------

```yaml
---
- hosts: monitoring_servers
  roles:
   - role: icinga2-ansible-web2-ui
     become: yes
     tags: icinga2-ansible-web2-ui
   - role: icinga2-pnp4nagios
     tags: icinga2-pnp4nagios
   - role: icinga2-ansible-custom-config-example
     tags: icinga2-ansible-custom-config-example

```

Role Variables
--------------

See variables:
* [defaults/main.yml](defaults/main.yml): For default values



License
-------

GNU General Public License Version 2

Author Information
------------------

Mickael Ange <akiran28@hotmail.com>

Contributors Information
------------------------


