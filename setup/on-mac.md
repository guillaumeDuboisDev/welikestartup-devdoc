# Composer

## Prérequis

* PHP &gt;= 5.6

## Installation SSH

Vérifier la version de PHP de disponible \(5.6 minimum\) :

```bash
php --version
```

Si ce n’est pas la bonne version, configurer un alias :

```bash
alias php='/usr/local/php5.6/bin/php'
```

Il est conseillé de rester au sein du dossier racine afin de ne pas rendre accessible publiquement les fichiers de “Composer”. Il faut donc exécuter cette commande :

```bash
curl -sS https://getcomposer.org/installer | php
```

Félicitations, “Composer” est désormais disponible !

## Installation Linux / Unix / OSX

