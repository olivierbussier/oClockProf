# Installation des composants du cours

Ce tuto est basé sur une installation standart de ubuntu, php et postgres. Adaptez ce tuto aux spécificités de votre installation.

Vérifier que php est bien installé avec les bons modules

Pour cela executer la commande suivante

```bash
php -v
```
Le résultat doit être

```bash
PHP 8.1.2-1ubuntu2.11 (cli) (built: Feb 22 2023 22:56:18) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.2, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.2-1ubuntu2.11, Copyright (c), by Zend Technologies
    with Xdebug v3.1.2, Copyright (c) 2002-2021, by Derick Rethans
```

Php7.4 minimum avec opcache par défaut, xdebug est optionnel (utilise pour faire du debug pas à pas avec php). Le module php php-pgsql doit etre installé

Pour s'en assurer on execute la commande php -m qui liste les modules disponibles (ils peuvent ne pas être activés):

```bash
php -m | grep "pgsql"

pdo_pgsql
pgsql

```

Le module qui nous intéresse est ici pdo_pgsql. Si ce module n'apparait pas dans la liste, c'est qu'il n'est pas installé. Il est alors possible de le faire avec les commandes:

```bash
sudo apt install php-pgsql
sudo phpenmod pgsql
```
Maintenant, on peut passer à la base de données, et vérifier qu'elle est bien installé et en fonctionnement

```
dpkg --get-selections | grep postgres

postgresql					    install
postgresql-14					install
postgresql-client-14			install
postgresql-client-common		install
postgresql-common				install
```
Et pour vérifier que le service est bien en fonctionnement:

```
systemctl status postgresql
```

Doit donner un résultat similaire à celui ci-dessous, loaded and active

```
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Thu 2023-05-04 10:24:34 CEST; 49min ago
    Process: 1370 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 1370 (code=exited, status=0/SUCCESS)
        CPU: 1ms

mai 04 10:24:33 Olivs-Ubuntu systemd[1]: Starting PostgreSQL RDBMS...
mai 04 10:24:34 Olivs-Ubuntu systemd[1]: Finished PostgreSQL RDBMS.
```

Si postgresql n'est pas installé, il est possible de le faire avec la commande :

```
sudo apt install postgresql
```
Lors de son installation, prostgressql crée un utilisateur 'postgres' et qui dispose des droits nécessaires pour administrer la database.
Ce compte est sans mot de passe pour l'instant...

On peut maintenant vérifier que la database est en fonctionnement, pour cela il faut se loguer en tant qu'utilisateur 'postgres' avec la commande
```
sudo -i -u postgres
```
L'invite de commande doit contenir maintenant ``postgres@<nom de machine>:#$``
On peut maintenant lancer le client psql qui permet d'executer des commandes en interactif avec la database

```sql
psql
```
On doit alors avoir un message du genre, suivi par une invite de commande (postgres=#).
```
psql (13.1-1)
Type "help" for help.

postgres=#
```
Si vous n'obtenez pas la bannière ci-dessus, reportez vous au site : https://doc.ubuntu-fr.org/postgresql qui vous aidera a corriger cela

Pour faciliter le travail avec PHP qui n'aime pas bien les connexions sans mot de passe, nous allons en ajouter un au compte 'postgres' de la base de données. Pour ce faire, toujours depuis psql, il faut executer la commande (toujours dans psql):

```sql
ALTER USER postgres WITH PASSWORD 'test';
```
C'est notre première commande SQL.

A partir de la, tout semble correct au niveau des installations, nous allons maintenant pouvoir tester l'accès à la database via le module PDO (PHP Data Object) de PHP. Pour cela vous allons écrire un petit bout de code dans un fichier nommé 'testdb.php' :

```php
<?php

    $pdo = new PDO("pgsql:host=127.0.0.1;dbname=test", 'postgres', 'test');

    // List all databases on the system
    $stmt = $pdo->query('SELECT datname FROM pg_database');

    // Fetch the results as an associative array
    $results = $stmt->fetchAll(PDO::FETCH_ASSOC);

    // Output the results
    echo '<pre>';
    print_r($results);
    echo '</pre>';
```
Il n'y a plus qu'à essayer avec la commande suivante
```shell
$ php testdb.php
Array
(
    [0] => postgres
    [1] => template1
    [2] => template0
)
```

Voila, tout est prêt pour la suite du cours