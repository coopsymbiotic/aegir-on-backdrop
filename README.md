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
* Install nginx, php-fpm and MariaDB (apache and mysql are untested)
* Install composer in a global path (https://getcomposer.org/)
* Tested on PHP 8.2
* Create a unix user called "aegir"

```
$ sudo adduser --system --group --home /var/aegir aegir
$ sudo adduser aegir www-data  #make aegir a user of group www-data
```

See: https://docs.aegirproject.org/install/#2-install-system-requirements

Then install the 'bee' CLI utility somewhere global, ex:

```
# wget https://github.com/backdrop-contrib/bee/releases/download/1.x-1.1.0/bee.phar -O /usr/local/bin/bee
# chmod 0755 /usr/local/bin/bee
```

Now deploy Aegir on Backdrop:

```
$ sudo -i -u aegir
$ cd /var/aegir
# Clone our fork of Provision
$ mkdir .drush
$ cd .drush
$ git clone https://github.com/mlutfy/provision.git
# Now clone the admin frontend (formerly hostmaster)
$ cd /var/aegir
$ git clone https://github.com/coopsymbiotic/aegir-on-backdrop.git admin
$ cd admin
$ bee dl-core web
$ composer install
```

Lanch the Aegir installation:

```
$ cd /var/aegir/admin
$ drush @none hostmaster-install --root="/var/aegir/admin/web" \
  --http_service_type=nginx --aegir_db_host=localhost \
  --aegir_db_user=aegir_root --aegir_db_pass=[...] aegir.example.org
```

.. where `aegir_root` is a mysql user with grant permissions, and `aegir.example.org` is the expected frontend URL.

## Support

Commercial support available from Coop Symbiotic: https://www.symbiotic.coop/en/contact

Feel free to open an issue on Github, but please do not expect a quick nor detailed response.
