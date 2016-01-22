Role Name
========

Ansible role to install and configure PNP4Nagios graphing for Icinga2 Web2:

* install pnp4nagios package
* download and install icinga2-pnp-module
* configure pnp4nagios including npcd, working directory permissions, and icinga2 templates (i.e. `pnp-hst` and `pnp-svc`)
* create htpasswd for a list of users (see `icinga2_htpasswd_users`)
* configure httpd (`pnp4nagios.conf`)
* enable icinga2-pnp-module

You can also take a look at [icinga2-ansible-custom-config-example](../icinga2-ansible-custom-config-example) role which presents a possible way to customize Icinga2 configuration.


Dependencies
------------

None

Requirements
------------

Roles: **icinga2-ansible-web2-ui**

You need to enable `perfdata` feature:

    icinga2_features: 
      - { name: perfdata, enabled: true }
      - others managed features..

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


