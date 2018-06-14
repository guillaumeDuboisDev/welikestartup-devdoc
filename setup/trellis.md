---
description: Comment installer l'application avec Trellis et l'architecture Bedrock
---

# Trellis

## Installation de Trellis

```text
mkdir dev.welikestartup && cd dev.welikestartup
git clone --depth=1 git@github.com:roots/trellis.git && rm -rf trellis/.git
git clone --depth=1 git@github.com:roots/bedrock.git site && rm -rf site/.git
```

Migration partielle

{% hint style="warning" %}
Ces consignes ne permettent pas de profiter de l'infrastructure de Bedrock correctement.
{% endhint %}

### Changements dans le code :

* [7.2 Fix: verify the type before counting](https://github.com/treyssatvincent/wp-content/commit/08e812662c87dce323f6c5c7372b6d400de41d72)
* [7.2 fix: Put strings between quotation marks](https://github.com/treyssatvincent/wp-content/commit/fa69f7a0a2477603543bbefb9eccb31ed1e8d43d)

### Changements dans bedrock :

{% code-tabs %}
{% code-tabs-item title="app.welikestartup.local/site/config/environments/development.php" %}
```php
define('WP_DEBUG', false);
define('SCRIPT_DEBUG', false);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Configuration de trellis :

On augmente la RAM disponible pour la VM :

{% code-tabs %}
{% code-tabs-item title="vagrant.default.yml" %}
```yaml
vagrant_cpus: 1
vagrant_memory: 4096 # in MB
```
{% endcode-tabs-item %}
{% endcode-tabs %}

On configure notre installation :

{% code-tabs %}
{% code-tabs-item title="group\_vars/development/php.yml" %}
```yaml
php_error_reporting: 'E_ERROR'
php_display_errors: 'Off'
php_display_startup_errors: 'Off'
php_track_errors: 'Off'
php_mysqlnd_collect_memory_statistics: 'Off'
php_opcache_enable: 1

xdebug_remote_enable: 0
xdebug_remote_connect_back: 0
xdebug_remote_autostart: 0
```
{% endcode-tabs-item %}

{% code-tabs-item title="group\_vars/development/wordpress\_sites.yml" %}
```yaml
wordpress_sites: dev.welikestartup:
  site_hosts:
    canonical: dev.welikestartup
  local_path: ../site # path targeting local Bedrock site directory (relative to Ansible root)
  site_install: false
  admin_email: local@welikestartup.local
  multisite:
    enabled: false
  ssl:
    enabled: true
    provider: self-signed
  hsts_max_age: 0
  csp:
    enabled: false
#  db_import: ../site/app.sql
```
{% endcode-tabs-item %}

{% code-tabs-item title="group\_vars/development/vault.yml" %}
```yaml
vault_mysql_root_password: devpw

vault_wordpress_sites:
  dev.welikestartup:
    db_user_host: root
    admin_password: admin
    env:
      db_password: 123
      db_prefix: "app_"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Lancement de la VM :

On peux maintenant créer la VM, puis rentrer dedans pour la configurer.

```bash
vagrant up
vagrant ssh
cd /srv/www/dev.welikestartup/current/
mysql -u dev_welikestartup -p dev_welikestartup_development < app.sql
wp search-replace 'https://app.welikestartup.io' 'https://dev.welikestartup'
```

En local notre certificat n'est pas valide, il faut donc enlever l'HSTS de notre configuration nginx :

{% code-tabs %}
{% code-tabs-item title="/etc/nginx/sites-available/app.welikestartup.local.conf" %}
```text
#add_header Strict-Transport-Security "max-age=0; includeSubDomains; ";
#add_header Content-Security-Policy "frame-ancestors 'self'" always;
#add_header X-Frame-Options $x_frame_options always;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Ajout du code de l'application

Déplacer les dossiers de `wp-content` dans les dossiers correspondant dans `dev.welikestartup/site/web/app/`.

