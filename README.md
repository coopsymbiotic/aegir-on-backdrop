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

Install the Ansible bits:

```
# apt install ansible-core
# ansible-galaxy collection install community.mysql
# cd /usr/local
# git clone https://github.com/coopsymbiotic/aegir-ansible-playbooks.git
# ln -s /usr/local/aegir-ansible-playbooks/bin/aegir-ansible /usr/local/bin/
```

Now run Ansible to do some of the setup:

```
# cd /usr/local/aegir-ansible-playbooks
# ansible-playbook ./aegir/admin/install.yml
```

The above will:

- Install the nginx, dehydrated and some other packages
- Create the aegir unix user
- Download the code necessary for the admin UI (or what Aegir calls "hostmaster")

Fixme - Lanch the Aegir installation - this will be launched by Ansible, the following will not work, because we do not need provision anymore.

```
$ cd /var/aegir/admin
$ drush @none hostmaster-install --root="/var/aegir/admin/web" \
  --http_service_type=nginx --aegir_db_host=localhost \
  --aegir_db_user=aegir_root --aegir_db_pass=[...] aegir.example.org
```

.. where `aegir_root` is a mysql user with grant permissions, and `aegir.example.org` is the expected frontend URL.

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

Then create the basic resources:

- Servers: create a server called "localhost" for "nginx" and "mysql" (assuming both services run locally, but mysql can also be remote)
- Platform: add a platform that points to a codebase under `/var/aegir/platforms`
- Sites: when the above is done, you are ready to create sites.

## Support

Commercial support available from Coop Symbiotic: https://www.symbiotic.coop/en/contact

Feel free to open an issue on Github, but please do not expect a quick nor detailed response.
