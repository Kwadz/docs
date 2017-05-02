# Drupal 6 stack

StackHub URL: https://cloud.wodby.com/#/hub/0ade1c32-7b75-41ad-b796-4de97fb7dc4e/detail

* [Overview](#overview)
* [Containers](#containers)
  * [MariaDB](#mariadb)
  * [Memcached](#memcached)
* [Mail Transfer Agent](#mail-transfer-agent)
* [Local environment](#local-environment)
* [Drupal settings](#drupal-settings)
* [SSH](#ssh)
* [Cron](#cron)
* [Drush](#drush)
* [Redirects](#redirects)
* [Environment variables](#environment-variables)
* [Import](#import)

## Overview

[wodby/drupal-nginx]: https://github.com/wodby/drupal-nginx
[wodby/drupal-php]: https://github.com/wodby/drupal-php
[wodby/mariadb]: https://github.com/wodby/mariadb
[wodby/memcached]: https://github.com/wodby/memcached
[wodby/drupal-varnish]: https://github.com/wodby/drupal-varnish
[wodby/adminer]: https://github.com/wodby/adminer
[phpmyadmin/phpmyadmin]: https://hub.docker.com/r/phpmyadmin/phpmyadmin
[mailhog/mailhog]: https://hub.docker.com/r/mailhog/mailhog

| Container | Image |
| --------- | ----------- |
| Nginx | [wodby/drupal-nginx] |
| PHP | [wodby/drupal-php] |
| MariaDB | [wodby/mariadb] |
| [Memcached](#memcached) | [wodby/memcached] |
| [Varnish](#varnish) | [wodby/drupal-varnish] |
| Solr | [wodby/drupal-solr] |
| Mailhog | [mailhog/mailhog] |
| PhpMyAdmin | [phpmyadmin/phpmyadmin] |
| Adminer | [wodby/adminer] |

## Containers

### MariaDB

If you want to access the database outside of the Wodby infrastructure you will have to use SSH tunnel via the main container:

1. Set up SSH tunnel on port `53306` (you can change it). You can find <SSH Port> <Node IP> on the main container page. For MySQL (port `3306` by default) use the following command:
```
$ ssh -L 53306:services:3306 -p <SSH Port> wodby@<Node IP> -N
```

2. Connect to the database (mysql) via the tunnel on port `53306`:
```bash
$ mysql --protocol=TCP -P53306 -uwodby -p<MySQL password> wodby
```

### Memcached

Just install and enable [Memcache API and Integration module](https://docs.wodby.com/bundles/containers/memcached.html) to use it as a default cache storage.

## Mail Transfer Agent

MTA server (OpenSMTPD) included to the stack by default. Additionally, you can catch all your emails by adding a mail catcher service (Mailhog).

### Guaranteed delivery of transaction emails

To make sure your transaction emails will be guaranteed delivered we recommend to use Transaction email services such as:

* <a href="http://sendgrid.com/" target="_blank">SendGrid</a> (has a free version). Read <a href="http://atendesigngroup.com/blog/send-mail-drupal-7-deliver-email-reliably-avoid-spam-folder" target="_blank">this article</a> on how to integration SendGrid with Drupal
* <a href="https://aws.amazon.com/ses/" target="_blank">AWS SES</a>
* <a href="http://mailchimp.com/" target="_blank">Mailchimp</a>

## Local environment

For local development please see https://github.com/wodby/docker4drupal

## Drupal settings

### settings.php

Wodby automatically adds include of `wodby.settings.php` to `settings.php` file inside of `sites/[SITE NAME]`. The value of `[SITE NAME]` is `default` unless you specify it distinctly on a new application deployment form. If directory doesn't exist Wodby will create it automatically.

**! Do not edit wodby.settings.php**, all changes to this file will be reset.

The `wodby.settings.php` file contains configuration settings for integration with Wodby services such as Database, Cache storage and Reverse Caching Proxy. You can override settings specified in wodby.settings.php in your `sites/*/settings.php` file after the include of wodby.settings.php

### Files

Files for Drupal located in `/mnt/files` and symlinked to `sites/default/files`.

### Base URL

The domain marked with primary flag will be used as a `$base_url` in settings.php file and as an `-l` parameter for cron.

## SSH

The copy of PHP container runs with SSHd. You can find access information on `[Instance] > Stack > SSH`

## Cron

The copy of PHP container runs with cron and runs every hour via `drush cron -l {{BASE_URL}}`.

## Drush

PHP container comes with installed drush. You can execute drush commands remotely via drush aliases. Download drush aliases from `[Instance] > Settings > Drush` page and place them to `~/.drush`. Execute commands (replace `[tokens]` with the real values) like this:

```bash
$  drush @[organization].[application].[instance] [drush command]
```

## Redirects

If you need to make a redirect from one domain to another you can do it by customizing configuration files of nginx or by adding the snippets below to your `settings.php` file.

### Redirect from one domain to another

```php
if (isset($_SERVER['WODBY_ENVIRONMENT_TYPE']) && $_SERVER['WODBY_ENVIRONMENT_TYPE'] == 'prod' && php_sapi_name() != "cli") {
  if ($_SERVER['HTTP_HOST'] == 'redirect-from-domain.com') {
    header('HTTP/1.0 301 Moved Permanently');
    header('Location: http://redirect-to-domain.com' . $_SERVER['REQUEST_URI']);
    exit();
  }
}
```

### Redirect from multiple domains.

```php
if (isset($_SERVER['WODBY_ENVIRONMENT_TYPE']) && $_SERVER['WODBY_ENVIRONMENT_TYPE'] == 'prod' && php_sapi_name() != "cli") {
  $redirect_from = array(
    'redirect-from-domain-1.com',
    'redirect-from-domain-2.com',
  );

  if (in_array($_SERVER['HTTP_HOST'], $redirect_from)) {
    header('HTTP/1.0 301 Moved Permanently');
    header('Location: http://redirect-to-domain.com' . $_SERVER['REQUEST_URI']);
    exit();
  }
}
```

### Redirect from HTTP to HTTPS

You can enable this redirect by checking the corresponding option on a domain edit page from the dashboard.

## Environment variables

| Variable  | Description |
| --------- | ----------- |
| `$APP_ROOT`               | `/var/www/html` by default |
| `$HTTP_ROOT`              | `/var/www/html` by default |
| `$WODBY_ENVIRONMENT_NAME` | |
| `$WODBY_ENVIRONMENT_TYPE` | |

## Import

### From drush archive

First, make sure you have <a href="http://www.drush.org/en/master/install/" target="_blank">Drush installed</a>, go to your Drupal website docroot and execute a command:

```bash
$ drush archive-dump
```

or

```bash
$ drush ard
```

You should see output like:

```bash
$ drush ard
Archive saved to /Users/johndoe/drush-backups/archive-dump/20150604001227/drupalapp.20150604_001228.tar.gz
/Users/johndoe/drush-backups/archive-dump/20150604001227/drupalapp.20150604_001228.tar.gz
```

Now navigate to `Apps > Deploy` and choose drush archive on the 2nd step

### From separate archives

Alternatively, you can import Drupal via separate archives for code, database and files. We support `.zip`, `.gz`, `.tar.gz`, `.tgz` and `.tar` archives.

> For big archives we recommend importing it manually after you deploy an app

### Manual

In case your Drupal website is huge it makes sense to import your database/files manually from the server. Follow these steps:

1. Deploy your Drupal website from a git repository without upload database and files
2. Once the app is deployed, go to `Stack > nginx-php` and copy SSH command
3. Connect the container by SSH and navigate to Drupal docroot (normally it's `/srv/app`)
4. Copy your database archive here using wget or scp, unpack the archive
5. Import unpacked database dump using `drush sql-cli < my-db-dump.sql`
6. Now let's import your files, cd to `/srv/files`
7. Copy your files archive here using wget or scp and unpack the archive
8. That's it! Clear app cache from the dashboard and don't forget to remove archives and extracted db dump