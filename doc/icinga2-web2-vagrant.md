# Icinga2 with Web2 and Vagrant

Let's get Icinga2 with Web2 ready to test.

Tested only on CentOS 7 for now.

Icinga Web2 credentials:
* Username: icingaadmin
* Password: icingaadmin

## Pre-requisites

* [Ansible](http://docs.ansible.com/ansible/intro_installation.html#installing-the-control-machine) v2.0+
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

The following Vagrant VMs are provisioned with their respective database backend. Icinga web2 Wizard has been skipped and the setup been automated.

See [Vagrantfile](../Vagrantfile) for Vagrant configuration.

### Provision Icinga Web 2 with MySQL as backend database

    vagrant up icinga2-web2-mysql

This VM plays [playbooks/icinga2-web2-mysql-setup.yml](../playbooks/icinga2-web2-mysql-setup.yml).

If you modify the play you can re-provision with:

    vagrant provision icinga2-web2-mysql

You can remove the VM with:

    vagrant destroy icinga2-web2-mysql

Enjoy Icinga2 Web2 with MySQL as backend database

* [http://monitoring/icingaweb2-mysql](http://172.16.1.2/icingaweb2)

### Provision Icinga Web 2 with PosgreSQL as backend database

See [host_vars/icinga2-web2-postgres.yml](../host_vars/icinga2-web2-postgres.yml) for PosgreSQL server configuration.

    vagrant up icinga2-web2-postgres

If you modify the play you can re-provision with:

    vagrant provision icinga2-web2-postgres

You can remove the VM with:

    vagrant destroy icinga2-web2-postgres

Enjoy Icinga2 Web2 with PosgreSQL as backend database

* [http://monitoring/icingaweb2-postgres](http://172.16.1.3/icingaweb2)


### Known Issues

* [SELinux: notification scripts](#selinux-notification-scripts)

### Fixed Issues

* [IDO: No Historical Data in Icinga2 Web2 and DB](#ido-no-historical-data-in-icinga2-web2-and-db)

## SELinux: notification scripts

SELinux and notification scripts trigger `terminated with exit code 126, output: /bin/sh: /etc/icinga2/scripts/mail-host-notification.sh: Permission denied`. We can see the error in Icinga2 logs but in `/var/log/audit/audit.log`. However, if we configure SELinux in permissive mode: `sudo setenforce 0` then notifications are sent out and causes of the problem appear in `/var/log/audit/audit.log`.

To reproduce the problem with `icinga2-web2-postgres` VM:
  1. Provision Icinga2 web2
    `vagrant up icinga2-web2-postgres`
  2. Login into the VM
    `vagrant ssh icinga2-web2-postgres`
  3. Watch logs files
    `sudo tail -f -n 20 /var/log/audit/audit.log /var/log/icinga2/icinga2.log`
  4. Login to Icinga web2 (Username: icingaadmin, Password: icingaadmin)
    [http://monitoring/icingaweb2-postgres](http://172.16.1.3/icingaweb2) 
  5. [Send a custom Host notification](http://172.16.1.3/icingaweb2/monitoring/list/hosts#!/icingaweb2/monitoring/host/send-custom-notification?host=icinga2-web2-postgres). You should see the following warning in `icinga2.log` but nothing in `audit.log`:

        ==> /var/log/icinga2/icinga2.log <==
        [2016-01-19 03:29:55 +0000] information/ExternalCommandListener: Executing external command: [1453174195] SEND_CUSTOM_HOST_NOTIFICATION;icinga2-web2-postgres;3;icingaadmin;test
        [2016-01-19 03:29:55 +0000] information/Checkable: Checking for configured notifications for object 'icinga2-web2-postgres'
        [2016-01-19 03:29:55 +0000] information/Notification: Sending notification 'icinga2-web2-postgres!mail-icingaadmin' for user 'icingaadmin'
        [2016-01-19 03:29:55 +0000] information/Notification: Completed sending notification 'icinga2-web2-postgres!mail-icingaadmin' for checkable 'icinga2-web2-postgres'
        [2016-01-19 03:29:55 +0000] warning/PluginNotificationTask: Notification command for object 'icinga2-web2-postgres' (PID: 10353, arguments: '/etc/icinga2/scripts/mail-host-notification.sh') terminated with exit code 126, output: /bin/sh: /etc/icinga2/scripts/mail-host-notification.sh: Permission denied

To work around the problem:
  1. Now configure SELinux in permissive mode:
    `sudo setenforce 0`
  2. Watch logs files: 
    `sudo tail -f -n 20 /var/log/audit/audit.log /var/log/icinga2/icinga2.log`
  3. [Send a custom Host notification](http://172.16.1.3/icingaweb2/monitoring/list/hosts#!/icingaweb2/monitoring/host/send-custom-notification?host=icinga2-web2-postgres). No more warning in `icinga2.log` but interesting messages in `audit.log`:

        ==> /var/log/audit/audit.log <==
        type=AVC msg=audit(1453175275.716:3232): avc:  denied  { ioctl } for  pid=10985 comm="mail" path="pipe:[60961]" dev="pipefs" ino=60961 scontext=system_u:system_r:system_mail_t:s0 tcontext=system_u:system_r:icinga2_t:s0 tclass=fifo_file
        type=SYSCALL msg=audit(1453175275.716:3232): arch=c000003e syscall=16 success=no exit=-25 a0=1 a1=5401 a2=7ffd06039010 a3=7ffd06038e20 items=0 ppid=10981 pid=10985 auid=4294967295 uid=995 gid=992 euid=995 suid=995 fsuid=995 egid=992 sgid=992 fsgid=992 tty=(none) ses=4294967295 comm="mail" exe="/usr/bin/mailx" subj=system_u:system_r:system_mail_t:s0 key=(null)
        type=AVC msg=audit(1453175275.742:3233): avc:  denied  { getattr } for  pid=10987 comm="sendmail" path="pipe:[60961]" dev="pipefs" ino=60961 scontext=system_u:system_r:system_mail_t:s0 tcontext=system_u:system_r:icinga2_t:s0 tclass=fifo_file
        type=SYSCALL msg=audit(1453175275.742:3233): arch=c000003e syscall=5 success=yes exit=0 a0=1 a1=7ffd6e7a27b0 a2=7ffd6e7a27b0 a3=2 items=0 ppid=1 pid=10987 auid=4294967295 uid=995 gid=992 euid=995 suid=995 fsuid=995 egid=992 sgid=992 fsgid=992 tty=(none) ses=4294967295 comm="sendmail" exe="/usr/sbin/sendmail.postfix" subj=system_u:system_r:system_mail_t:s0 key=(null)
        type=AVC msg=audit(1453175275.744:3234): avc:  denied  { write } for  pid=10988 comm="postdrop" path="pipe:[60961]" dev="pipefs" ino=60961 scontext=system_u:system_r:postfix_postdrop_t:s0 tcontext=system_u:system_r:icinga2_t:s0 tclass=fifo_file
        type=SYSCALL msg=audit(1453175275.744:3234): arch=c000003e syscall=59 success=yes exit=0 a0=2b456486c2d0 a1=2b456486c300 a2=2b456486b400 a3=2 items=0 ppid=10987 pid=10988 auid=4294967295 uid=995 gid=992 euid=995 suid=995 fsuid=995 egid=90 sgid=90 fsgid=90 tty=(none) ses=4294967295 comm="postdrop" exe="/usr/sbin/postdrop" subj=system_u:system_r:postfix_postdrop_t:s0 key=(null)
        type=AVC msg=audit(1453175275.748:3235): avc:  denied  { getattr } for  pid=10988 comm="postdrop" path="pipe:[60961]" dev="pipefs" ino=60961 scontext=system_u:system_r:postfix_postdrop_t:s0 tcontext=system_u:system_r:icinga2_t:s0 tclass=fifo_file
        type=SYSCALL msg=audit(1453175275.748:3235): arch=c000003e syscall=5 success=yes exit=0 a0=2 a1=7ffdd3092f90 a2=7ffdd3092f90 a3=2 items=0 ppid=10987 pid=10988 auid=4294967295 uid=995 gid=992 euid=995 suid=995 fsuid=995 egid=90 sgid=90 fsgid=90 tty=(none) ses=4294967295 comm="postdrop" exe="/usr/sbin/postdrop" subj=system_u:system_r:postfix_postdrop_t:s0 key=(null)

        ==> /var/log/icinga2/icinga2.log <==
        [2016-01-19 03:47:55 +0000] information/ExternalCommandListener: Executing external command: [1453175275] SEND_CUSTOM_HOST_NOTIFICATION;icinga2-web2-postgres;0;icingaadmin;test
        [2016-01-19 03:47:55 +0000] information/Checkable: Checking for configured notifications for object 'icinga2-web2-postgres'
        [2016-01-19 03:47:55 +0000] information/Notification: Sending notification 'icinga2-web2-postgres!mail-icingaadmin' for user 'icingaadmin'
        [2016-01-19 03:47:55 +0000] information/Notification: Completed sending notification 'icinga2-web2-postgres!mail-icingaadmin' for checkable 'icinga2-web2-postgres'

Strangely now the causes of problem are visible in `/var/log/audit/audit.log`. Permission problem with `mail`, `sendmail`, and `postdrop`.

## IDO: No Historical Data in Icinga2 Web2 and DB

I cannot figure out how to get data in:
* Comments and Downtimes from Overview menu
* Event Grid, Event Overview, Notifications, Timeline from History menu

When I was using Icinga 1.x, I had no problem with retrieving host's and service's comments.

To reproduce the problems with `icinga2-web2-postgres` VM with SELinux in permissive mode:
  1. Provision Icinga2 web2
    `vagrant up icinga2-web2-postgres`
  2. Login into the VM
    `vagrant ssh icinga2-web2-postgres`
  3. Now configure SELinux in permissive mode:
    `sudo setenforce 0`
  4. Enable debuglog feature
    `icinga2 feature enable debuglog`
    `service icinga2 restart`
  5. Watch logs files
    `sudo tail -f -n 20 /var/log/audit/audit.log /var/log/icinga2/debug.log`
  6. Login to Icinga web2 (Username: icingaadmin, Password: icingaadmin)
    [http://monitoring/icingaweb2-postgres](http://172.16.1.3/icingaweb2) 

IDO PGSQL config: `/etc/icinga2/features-enabled/ido-pgsql.conf`

    library "db_ido_pgsql"
    
    object IdoPgsqlConnection "ido-pgsql" {
      user = "icinga"
      password = "icinga"
      host = "localhost"
      database = "icinga"
      table_prefix = "icinga_"
      instance_name = "icinga2"
      instance_description = "icinga2 instance"
    
      cleanup = {
        downtimehistory_age = 48h
        logentries_age = 31d
      }
    
      categories = DbCatConfig | DbCatState
    }


### Add Host Comment

* [Add Host Comment](http://172.16.1.3/icingaweb2/monitoring/list/hosts#!/icingaweb2/monitoring/host/add-comment?host=icinga2-web2-postgres) :

        ==> /var/log/icinga2/debug.log <==
        [2016-01-19 05:47:32 +0000] debug/IdoPgsqlConnection: Query: COMMIT
        [2016-01-19 05:47:32 +0000] information/ExternalCommandListener: Executing external command: [1453182452] ADD_HOST_COMMENT;icinga2-web2-postgres;1;icingaadmin;test
        [2016-01-19 05:47:32 +0000] debug/DbEvents: add external command history
        [2016-01-19 05:47:32 +0000] notice/ExternalCommandProcessor: Creating comment for host icinga2-web2-postgres
        [2016-01-19 05:47:32 +0000] information/ConfigCompiler: Compiling config file: /var/lib/icinga2/api/packages/_api/icinga2-web2-postgres-1451534356-1/conf.d/comments/icinga2-web2-postgres!icinga2-web2-postgres-1453182452-1.conf
        [2016-01-19 05:47:32 +0000] information/ConfigItem: Committing config items
        [2016-01-19 05:47:32 +0000] warning/ApplyRule: Apply rule 'satellite-host' (in /etc/icinga2/conf.d/satellite.conf: 29:1-29:41) for type 'Dependency' does not match anywhere!
        [2016-01-19 05:47:32 +0000] warning/ApplyRule: Apply rule '' (in /etc/icinga2/conf.d/services.conf: 57:1-57:65) for type 'Service' does not match anywhere!
        [2016-01-19 05:47:32 +0000] warning/ApplyRule: Apply rule '' (in /etc/icinga2/conf.d/services.conf: 65:1-65:53) for type 'Service' does not match anywhere!
        [2016-01-19 05:47:32 +0000] information/ConfigItem: Instantiated 1 Comment.
        [2016-01-19 05:47:32 +0000] information/ConfigItem: Triggering Start signal for config items
        [2016-01-19 05:47:32 +0000] information/ConfigItem: Activated all objects.
        [2016-01-19 05:47:32 +0000] notice/Comment: Added comment 'icinga2-web2-postgres!icinga2-web2-postgres-1453182452-1'.

**Note**: a file (`/var/lib/icinga2/api/packages/_api/icinga2-web2-postgres-1451534356-1/conf.d/comments/icinga2-web2-postgres!icinga2-web2-postgres-1453182452-1.conf`) is created with the following object: 

    object Comment "icinga2-web2-postgres-1453181648-21" ignore_on_error {
        author = "icingaadmin"
        entry_type = 1.000000
        expire_time = 0.000000
        host_name = "icinga2-web2-postgres"
        text = "test"
        version = 1453181648.110585
    }


* Get Comments from database:

        $ PGPASSWORD=icinga psql -h 127.0.0.1 icinga icinga -x --command="SELECT * FROM icinga_comments,icinga_commenthistory;"
        (No rows)

### Schedule Host Downtime

* [Schedule Host Downtime](http://172.16.1.3/icingaweb2/monitoring/list/hosts#!/icingaweb2/monitoring/host/schedule-downtime?host=icinga2-web2-postgres) :

        ==> /var/log/icinga2/debug.log <==
        [2016-01-19 05:53:38 +0000] debug/IdoPgsqlConnection: Query: COMMIT
        [2016-01-19 05:53:38 +0000] debug/IdoPgsqlConnection: Query: BEGIN
        [2016-01-19 05:53:39 +0000] information/ExternalCommandListener: Executing external command: [1453182819] SCHEDULE_HOST_DOWNTIME;icinga2-web2-postgres;1453182813;1453186413;1;0;0;icingaadmin;test
        [2016-01-19 05:53:39 +0000] debug/DbEvents: add external command history
        [2016-01-19 05:53:39 +0000] notice/ExternalCommandProcessor: Creating downtime for host icinga2-web2-postgres
        [2016-01-19 05:53:39 +0000] information/ConfigCompiler: Compiling config file: /var/lib/icinga2/api/packages/_api/icinga2-web2-postgres-1451534356-1/conf.d/downtimes/icinga2-web2-postgres!icinga2-web2-postgres-1453182819-2.conf
        [2016-01-19 05:53:39 +0000] information/ConfigItem: Committing config items
        [2016-01-19 05:53:39 +0000] warning/ApplyRule: Apply rule 'satellite-host' (in /etc/icinga2/conf.d/satellite.conf: 29:1-29:41) for type 'Dependency' does not match anywhere!
        [2016-01-19 05:53:39 +0000] warning/ApplyRule: Apply rule '' (in /etc/icinga2/conf.d/services.conf: 57:1-57:65) for type 'Service' does not match anywhere!
        [2016-01-19 05:53:39 +0000] warning/ApplyRule: Apply rule '' (in /etc/icinga2/conf.d/services.conf: 65:1-65:53) for type 'Service' does not match anywhere!
        [2016-01-19 05:53:39 +0000] information/ConfigItem: Instantiated 1 Downtime.
        [2016-01-19 05:53:39 +0000] information/ConfigItem: Triggering Start signal for config items
        [2016-01-19 05:53:39 +0000] debug/IdoPgsqlConnection: Query: UPDATE icinga_hoststatus SET scheduled_downtime_depth = E'4' WHERE host_object_id = 68 AND instance_id = 1
        [2016-01-19 05:53:39 +0000] information/ConfigItem: Activated all objects.
        [2016-01-19 05:53:39 +0000] notice/Downtime: Added downtime 'icinga2-web2-postgres!icinga2-web2-postgres-1453182819-2' between '2016-01-19 05:53:33' and '2016-01-19 06:53:33'.
        [2016-01-19 05:53:39 +0000] debug/IdoPgsqlConnection: Query: COMMIT
        [2016-01-19 05:53:39 +0000] debug/IdoPgsqlConnection: Query: BEGIN
        [2016-01-19 05:53:39 +0000] notice/CheckerComponent: Pending checkables: 0; Idle checkables: 9; Checks/s: 0.8
        [2016-01-19 05:53:39 +0000] notice/DbConnection: Updating programstatus table.
        [2016-01-19 05:53:39 +0000] notice/WorkQueue: #2 tasks: 7
        [2016-01-19 05:53:40 +0000] notice/DbConnection: Cleanup (downtimehistory): 172800 now: 1453182820 old: 1.45301e+09
        [2016-01-19 05:53:40 +0000] notice/DbConnection: Cleanup (logentries): 2.6784e+06 now: 1453182820 old: 1.4505e+09
        [2016-01-19 05:53:40 +0000] debug/IdoPgsqlConnection: Query: DELETE FROM icinga_downtimehistory WHERE instance_id = 1 AND entry_time < TO_TIMESTAMP(1453010020)
        [2016-01-19 05:53:40 +0000] debug/IdoPgsqlConnection: Query: DELETE FROM icinga_logentries WHERE instance_id = 1 AND logentry_time < TO_TIMESTAMP(1450504420)
        [2016-01-19 05:53:40 +0000] debug/IdoPgsqlConnection: Query: SELECT 1
        [2016-01-19 05:53:40 +0000] debug/IdoPgsqlConnection: Query: COMMIT

* Get Downtimes from database:

        $ PGPASSWORD=icinga psql -h 127.0.0.1 icinga icinga -x --command="SELECT * FROM icinga_downtimehistory, icinga_scheduleddowntime;"
        (No rows)

**Note 1**: The downtime is scheduled correctly but we cannot retrieve it from Icinga 2 Web 2 interface: [Downtimes Menu](http://172.16.1.3/icingaweb2/monitoring/list/downtimes).

**Note 2**: a file (`/var/lib/icinga2/api/packages/_api/icinga2-web2-postgres-1451534356-1/conf.d/downtimes/icinga2-web2-postgres!icinga2-web2-postgres-1453182819-2.conf`) is created with the following object: 

    object Downtime "icinga2-web2-postgres-1453181136-20" ignore_on_error {
        author = "icingaadmin"
        comment = "test"
        config_owner = ""
        duration = 0.000000
        end_time = 1453184728.000000
        fixed = true
        host_name = "icinga2-web2-postgres"
        scheduled_by = ""
        start_time = 1453181128.000000
        triggered_by = ""
        version = 1453181136.076452
    }

I have same kind of problems with notifications.

**Question**:

Is there any IDO or Icinga2 configuration to get the database populate with Comments, Downtimes, etc..? 

**Answer**: 

As explained in the documentation:

> categories Optional. The types of information that should be written to the database.

So when I removed categories from my `ido-pgsql.conf` configuration. I restart Icinga2 then the data are written in the DB as expected. `Comments`, `Downtimes`, well everything I wanted is listed the UI now.

I also removed the `cleanup` section as it is probably not what I want too.

The final ido-pgsql.conf configuration:

    library "db_ido_pgsql"
    
    object IdoPgsqlConnection "ido-pgsql" {
      user = "icinga"
      password = "icinga"
      host = "localhost"
      database = "icinga"
      table_prefix = "icinga_"
      instance_name = "icinga2"
      instance_description = "icinga2 instance"
    }

