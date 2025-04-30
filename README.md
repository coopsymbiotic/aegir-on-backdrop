# Aegir on BackdropCMS

Port of the Aegir hosting system to the BackdropCMS.

The original Aegir version 3.x uses Drupal7 as the web interface. This port of
Aegir runs on BackdropCMS, an actively maintained fork of Drupal7.

Why? Because it was much less work than porting to Drupal10+.
[More background information](https://www.bidon.ca/random/2024-10-02-aegir-drupal10-backdrop/)

License: GPLv2, see LICENSE.txt

## Disclaimer

This is work in progress. Do not expect it to work or to be stable.
Use at your own risk. Aegir is part of the tooling we use for hosting.

## Supported Aegir features

We aim to support the basic features of Aegir:

* Actions: Install a site, enable, disable, verify, backup, restore, clone, migrate, remote import
* CMSes: Drupal7, Drupal10+, WordPress, CiviCRM (as Standalone and with Drupal or WordPress)

Eventually, and somewhat ironically, we do hope to support the installation of
BackdropCMS sites too.

## Unsupported features

* Remote deploy to multiple webservers
* Building platforms using drush make, composer, etc. We assume platforms are deployed manually (you can still use composer or whatever, but when adding the platform, it is assumed that the files are already on the server).

Part of the motivation to drop these features was to make the code easier to maintain. We do not use these features.

## Getting started

Requirements:

* Setup a new VM running Debian stable (bookworm, as of this writing)
* Install php-fpm and MariaDB (apache and mysql are untested)
* Tested on PHP 8.2

The installer assumes that root has passwordless access to MySQL, which is the default since Debian 12 (bookworm). You can test this by typing `mysql` as root, it should not require a password.

To enable:

```
# mysql -u root -p
mysql> grant usage on *.* to 'root'@'localhost' identified via unix_socket;
```

Install the Ansible bits:

```
# apt install ansible-core
# ansible-galaxy collection install community.mysql ansible.posix:1.6.1
# cd /usr/local
# git clone https://github.com/coopsymbiotic/aegir-ansible-playbooks.git
# ln -s /usr/local/aegir-ansible-playbooks/bin/aegir-ansible /usr/local/bin/
```

(forcing ansible.posix:1.6.1 is required when using Ansible 2.14, which is what Debian 12/bookworm ships)

Now run Ansible to do some of the setup:

```
# cd /usr/local/aegir-ansible-playbooks
# ansible-playbook ./aegir/admin/install.yml
```

The above will:

- Install nginx, dehydrated and some other packages (although it does not install php-fpm, since you probably want to configure sury.org and run a specific version)
- Create the aegir unix user
- Download the code necessary for the admin UI (or what Aegir calls "hostmaster")

Enable hosting modules (bee does not seem to enable dependencies, and they are somewhat circular here because of views classes):

```
cd /var/aegir/admin/web
bee en views views_bulk_operations
bee en --no-dependency-checking hosting hosting_platform hosting_package hosting_site hosting_db_server hosting_server hosting_web_server hosting_nginx hosting_clone hosting_alias hosting_migrate hosting_queued hosting_task aegir_ansible_inventory
```

And then you probably want to enable the hosting-queue daemon:

```
cd /var/aegir/admin/web
bee en hosting_queued
```

The `hosting_queued` module provides a small daemon that can be managed by systemd (see: `hosting/queue/hosting-queued.service.example`). It runs the tasks that are created while using the Aegir frontend (ex: install a site, verify, backup, etc). Those tasks are run as background tasks (daemon) because they require more privileges than what the web user usually has, and they might take more than 30-60 seconds to run, so this would clog up web PHP processes and cause timeouts.

You can also run it from the command line as the `aegir` user (not root!):

```
cd /var/aegir/admin/web
bee hosting-queued
```

Now you can login to Aegir:


```
cd /var/aegir/admin/web
bee uli
```

Make sure that link has the correct hostname before copy-pasting in a browser (it might be 'web', todo).

You may find it useful to set the default home page to `hosting/sites` (list of sites), under Configuration > System > Site Information.

Then create the basic resources:

- Servers: create a server called "localhost" for "nginx" and "mysql" (assuming both services run locally, but mysql can also be remote)
- Platform: add a platform that points to a codebase under `/var/aegir/platforms`
- Sites: when the above is done, you are ready to create sites.

You can also manage Aegir with Aegir, by adding the platform, then adding the site (fixme: do we need to set the password?). This is partially automated by running:

```
cd /var/aegir/admin/web
bee hosting-platform-setup-aegir
```

You will still need to edit the "localhost" server, and enable MySQL.

## Support

Commercial support available from Coop Symbiotic: https://www.symbiotic.coop/en/contact

Feel free to open an issue on Github, but please do not expect a quick nor detailed response.
