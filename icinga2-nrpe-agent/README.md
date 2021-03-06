Role Name
========

Ansible role to install and configure Nrpe Agent and Plugins

Dependencies
------------

None

Requirements
------------

EPEL

Example Playbook
-------------------------

```yaml
---
- hosts: all
  gather_facts: True

  roles:
   - role: icinga2-nrpe-agent
     tags: nrpe-agent

```

Role Variables
--------------

See variables:
* [defaults/main.yml](defaults/main.yml): For default values
* [vars/Debian.yml](vars/Debian.yml): For Debian OS family
* [vars/RedHat.yml](vars/RedHat.yml): For RedHat OS family
* [vars/Gentoo.yml](vars/Gentoo.yml): For Gentoo OS family

### Example of NRPE check commands:

```yaml
---
# Default NRPE check commands
nrpe_agent_check_commands:
  - { name: check_users, command: "{{ nrpe_agent_nagios_plugins_path }}/check_users -w 5 -c 10" }
  - { name: check_load, command: "{{ nrpe_agent_nagios_plugins_path }}/check_load -w 15,10,5 -c 30,25,20" }
  - { name: check_hda1, command: "{{ nrpe_agent_nagios_plugins_path }}/check_disk -w 20% -c 10% -p /dev/hda1" }
  - { name: check_zombie_procs, command: "{{ nrpe_agent_nagios_plugins_path }}/check_procs -w 5 -c 10 -s Z" }
  - { name: check_total_procs, command: "{{ nrpe_agent_nagios_plugins_path }}/check_procs -w 150 -c 200" } 

```


License
-------

GNU General Public License Version 2

Author Information
------------------

Valentino Gagliardi - Icinga Dev Team

Contributors Information
------------------------

* Mickael Ange <akiran28@hotmail.com>
