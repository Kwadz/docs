* PHP can be configured with the following [environment variables](https://github.com/wodby/drupal-php#environment-variables)
* Available [php extensions](https://github.com/wodby/php#php-extensions)
* Composer pre-installed with a default global package `hirak/prestissimo:^0.3` to download dependencies in parallel 

### Environment variables

!!! info "Variables availability" 
    Environment variables provided by Wodby are always available in PHP even if `PHP_FPM_CLEAR_ENV` set to `no`. 

In addition to [global environment variables](/infrastructure/env-vars.md), we provide the following variables in PHP container that you can use in your [post-deployment scripts](/apps/post-deployment-scripts.md) or settings files:

| Variable              | Description                                   |
| --------------------- | --------------------------------------------  |
| `$APP_ROOT`           | `/var/www/html` by default                    |
| `$HTTP_ROOT`          | e.g. `/var/www/html/web`                      |
| `$CONF_DIR`           | `/var/www/conf` by default                    |
| `$WODBY_APP_NAME`     | My app                                        |
| `$WODBY_HOST_PRIMARY` | example.com                                   |
| `$WODBY_URL_PRIMARY`  | http://example.com                            |
| `$WODBY_HOSTS`        | `[ "example.com", "dev.example.org.wod.by" ]` |

Deprecated variables:

| Variable             | Instead use    |
| -------------------- | -------------- |
| `$WODBY_APP_ROOT`    | `$APP_ROOT`    |
| `$WODBY_APP_DOCROOT` | `$HTTP_ROOT`   |
| `$WODBY_CONF`        | `$CONF_DIR`    |
| `$WODBY_DIR_CONF`    | `$CONF_DIR`    |