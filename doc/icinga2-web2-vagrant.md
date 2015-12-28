# Icinga2 with Web2 and Vagrant

Let's get Icinga2 with Web2 ready to test.

## Pre-requisites

* [Ansible](http://docs.ansible.com/ansible/intro_installation.html#installing-the-control-machine) v1.9.4+
* [Librarian-Ansible](https://github.com/bcoe/librarian-ansible) v1.0.6+ to manage Ansible's dependencies
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads) v5.0+
* [Vagrant](https://www.vagrantup.com/downloads.html) v1.7.4+

## Install librarian-ansible

    gem install librarian-ansible

## Install Dependencies

To install dependencies locally, run:

    librarian-ansible install

This command downloads roles into `librarian_roles/`. This directory has been added to `roles_path` in [ansible.cfg](../ansible.cfg).

## Create and Provision a VM with Vagrant

    vagrant provision icinga2-web2-mysql
    vagrant provision icinga2-web2-postgres

## Enjoy Icinga2 Web2

* Username: icingaadmin
* Password: icingaadmin

* [http://monitoring/icingaweb2-mysql](http://172.16.1.2/icingaweb2)
* [http://monitoring/icingaweb2-postgres](http://172.16.1.3/icingaweb2)

