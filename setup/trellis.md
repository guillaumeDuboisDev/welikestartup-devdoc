---
description: Comment installer l'application avec Trellis et l'architecture Bedrock
---

# Trellis

## Déploiement en développement

### Configuration de Trellis & Bedrock

D'abord on récupère l'architecture.

```bash
mkdir dev.welikestartup.io && cd dev.welikestartup.io
git clone --depth=1 git@github.com:roots/trellis.git 
git clone --depth=1 git@github.com:roots/bedrock.git site
cd trellis
```

#### On paramètre Trellis.

{% hint style="info" %}
Vous pouvez modifier le nom de domaine utilisé dans les exemples, mais pensez à le faire à chaque fichiers où il apparait.
{% endhint %}

D'abords les paramètres de PHP :

{% code-tabs %}
{% code-tabs-item title="group\_vars/php.yml" %}
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
{% endcode-tabs %}

Ensuite les paramètres d'installation de notre application :

{% code-tabs %}
{% code-tabs-item title="group\_vars/wordpress\_sites.yml" %}
```yaml
wordpress_sites:
  dev.welikestartup.io:
    site_hosts:
      - canonical: dev.welikestartup.io
    local_path: ../site # path targeting local Bedrock site directory (relative to Ansible root)
    site_install: false
    admin_email: wewont@usethis.butrequired
    multisite:
      enabled: false
    ssl:
      enabled: true
      provider: self-signed
      hsts_max_age: 0
    csp:
      enabled: false
    env:
      db_name: wls_dev
      db_user: wls
      db_prefix: app_
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Enfin les valeurs qui doivent rester secrètes : 

{% code-tabs %}
{% code-tabs-item title="group\_vars/vault.yml" %}
```yaml
vault_mysql_root_password: wewontusethisbutrequired

vault_wordpress_sites:
  dev.welikestartup.io:
    admin_password: wewontusethisbutrequired
    env:
      db_password: 1234567890
      acf_pro_key: ACF_PRO_KEY_HERE
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Pensez à mettre la clef d'ACF PRO.

Enfin on ajoute un peu de RAM à la machine virtuelle en modifiant la ligne suivante dans `vagrant.default.yml` :

{% code-tabs %}
{% code-tabs-item title="vagrant.default.yml" %}
```yaml
vagrant_memory: 2048
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### On configure Bedrock

```text
cd ../site
```

D'abord on remplace le composer.json avec celui-ci : [composer.json](https://gist.github.com/treyssatvincent/82fb6062e14aeb1296fb74d4fe37b5e1)

On récupère une sauvegarde de la base de données que l'on va placer dans la racine du site.

Puis on désactive les messages d'erreurs :

{% code-tabs %}
{% code-tabs-item title="app.welikestartup.local/site/config/environments/development.php" %}
```php
define('WP_DEBUG', false);
define('SCRIPT_DEBUG', false);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Démarrage de la machine virtuelle

```bash
cd ../trellis
vagrant box update
vagrant up
```

On se connecte en ssh à la machine virtuelle et on se rends dans le dossier de l'application pour importer la base de données \(Remplacez wls\_dev.sql par le nom de votre ficher sql\).

```bash
vagrant ssh
cd /srv/www/dev.welikestartup.io/current/
mysql -u wls -p wls_dev < wls_dev.sql
wp search-replace 'https://app.welikestartup.io' 'https://dev.welikestartup.io'
```

En local notre certificat n'est pas valide, il faut donc enlever l'HSTS de notre configuration nginx :

```text
nano /etc/nginx/sites-available/dev.welikestartup.io.conf
```

{% code-tabs %}
{% code-tabs-item title="/etc/nginx/sites-available/dev.welikestartup.io.conf" %}
```text
#add_header Strict-Transport-Security "max-age=0; includeSubDomains; ";
#add_header Content-Security-Policy "frame-ancestors 'self'" always;
#add_header X-Frame-Options $x_frame_options always;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

On recharge la configuration de nginx

```text
sudo systemctl reload nginx
```

### Ajout du code de l'application

Déplacer les thèmes \(buddyapp et buddyapp-child\) dans le dossier `dev.welikestartup.io/site/web/app/themes`.

Déplacer les médias \(uploads\) dans `dev.welikestartup.io/site/web/app/uploads`

{% hint style="success" %}
Votre application est disponible à https://dev.welikestartup.io !
{% endhint %}

## Déploiement en production

## Changements dans le code :

* [7.2 Fix: verify the type before counting](https://github.com/treyssatvincent/wp-content/commit/08e812662c87dce323f6c5c7372b6d400de41d72)
* [7.2 fix: Put strings between quotation marks](https://github.com/treyssatvincent/wp-content/commit/fa69f7a0a2477603543bbefb9eccb31ed1e8d43d)

