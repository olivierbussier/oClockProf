
# Commandes SQL

Un moteur de bases de données PostegreSQL contient des databases. Chaque database peut contenir plusieurs tables

Les commandes SQL pour gérer les databases et les tables.

- Les databases se créent au moyen de la commande `create database` [lien vers la doc](https://docs.postgresql.fr/9.6/sql-createdatabase.html)
- Les tables sont créées au moyen de la commande `create table` [lien vers la doc](https://docs.postgresql.fr/9.6/sql-createtable.html)

De la même manière, les databases et tables pourront être supprimées (irréversible) au moyen des commandes :

- `drop database` [lien vers la doc](https://docs.postgresql.fr/9.6/sql-dropdatabase.html)
- `drop table`    [lien vers la doc](https://docs.postgresql.fr/9.6/sql-droptable.html)

En SQL, la manipulation des données dans les tables s'effectue au moyen de 4 commandes:
- `select` qui permet de séléctionner et récuperer des lignes de données. [lien vers la doc](https://docs.postgresql.fr/9.6/sql-select.html)
- `insert` qui permet d'insérer des lignes. [lien vers la doc](https://docs.postgresql.fr/9.6/sql-insert.html)
- `update` qui permet de modifier des lignes existantes. [lien vers la doc](https://docs.postgresql.fr/9.6/sql-update.html)
- `delete` qui permet de supprimer des lignes. [lien vers la doc](https://docs.postgresql.fr/9.6/sql-delete.html)

Il existe également beaucoup d'autres commandes pour gérer les utilisateurs, les droits d'accès aux databases et tables, toutes ces commandes sont décrites dans la documentation officielle que l'on trouvera [ici](https://docs.postgresql.fr/9.6/sql-commands.html)

## Travaux pratiques

On considère ici que postgresql est installé, en fonctionnement et que le compte par défault `postgres` est utilisable (voir le fichier [installation](installation.md)).

Avant de commencer les travaux pratiques il faut faire quelques bricoles qui seront utiles pour plus tard, on y reviendra
- Ouvrir une session shell avec votre login.
- se connecter àu compte postgresql en utilisant la commande `sudo` (la commande `sudo` doit normalement vous demander votre mot de passe).

```shell
$ sudo -i -u postgres
```
et ensuite on lance l'interpreteur SQL interactif `psql`

```shell
$ psql

postgres=#
```
A ce stade, si on execute la commande `\du` (display users), il ne devrait y avoir qu'un seul utilisateur

```sql
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

```
On va maintenant créer un utlisateur qui a le même nom que notre login. Moi mon login c'est `olivier`, je vais donc créer un user postgres `olivier`, je vais également lui affecter un mot de passe

```sql
postgres=# create user olivier with password 'test';
CREATE ROLE
```
le système répond `CREATE ROLE`, ce qui indique que le rôle a été créé avec succès.

On va maintenant créer une database qui porte également notre nom de login

```sql
postgres=# create database olivier;
CREATE DATABASE
```
Le systeme répond `CREATE DATABASE`, ce qui indique là encore que la database à été créée avec succès.

On a donc créé une base olivier et un role olivier. Mais cela ne suffira pas pour se connecter à la base `olivier` en tant qu'utilisateur `olivier`. En effet, à ce stade cet utilisateur ne dispose d'aucun droit sur cette base (n'oublions pas que nous sommes connectés à la base en tant qu'utilisateur `postgres` qui est admin). Nous allons maintenant attribuer des droits a l'utilisateur `olivier`:

```sql
postgres=# grant all on database olivier to olivier;
GRANT
```

Yes! Maintenant, il doit être possible de se connecter en tant qu'utilisateur `olivier` au système; allons voir cela de plus près:

Je quitte psql (`exit`)
Je me déconnecte du compte `postgres` avec la commande `exit`.

Je peux utiliser la commande `whoami` (who I'm I) pour vérifier que je suis bien sur mon compte.

```
postgres=# exit

postgres@Olivs-Ubuntu:~$ exit

olivier@Olivs-Ubuntu:~/oClock$ whoami
olivier

olivier@Olivs-Ubuntu:~/oClock$
```
Parfait, je tente maintenant de me connecter à la base avec psql

```
olivier@Olivs-Ubuntu:~/oClock$ psql
sql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

olivier=>
```
Ca à l'air bien , non ?, mais est-ce que je suis bien sur la base `olivier` ? On peut le vérifier avec la commande `\c` sans paramètre qui indique la base sur laquelle on est connecté.

```
olivier=> \c
You are now connected to database "olivier" as user "olivier".
```
Yes again! je suis connecté a la base `olivier` avec mon role `olivier`. Mais pour autant, puis-je créer une table ? Eh bien essayons !

```sql
create table if not exists essai (
    id int
);
CREATE TABLE
```
CREATE TABLE ?, c'est tout bon, on peut le vérifier avec la commande `\dt`
```sql
olivier=> \dt
        List of relations
 Schema | Name  | Type  |  Owner
--------+-------+-------+---------
 public | essai | table | olivier
(1 row)
```
Nickel, la table `sac` existe bien! () noter qu'un synonyme de `table` dans le jargon SQL est `relation`.

Supprimons la maintenant que tout est ok
```sql
drop table essai;
````

Pour importer facilement une bonne quantité de données, J'ai réalisé un fichier sql contenant deux tables qui nous seront utiles pour le travail à venir. On va réaliser l'importation de ce fichier avec la commande.
```shell
$ psql < base-sql/data.sql
```
L'indication `COPY 86` et `COPY 134` dans les lignes affichées indique que la commande à bien fonctionné, et a créé 86 nouvelles ligners dans la table `sac` et 134 dans la table emprunts_nantes.

Revenons dans psql
```sql
olivier:~$ psql
psql (14.7 (Ubuntu 14.7-0ubuntu0.22.04.1))
Type "help" for help.

olivier=> \c
You are now connected to database "olivier" as user "olivier".
olivier=> \dt
             List of relations
 Schema |      Name       | Type  |  Owner
--------+-----------------+-------+---------
 public | emprunts_nantes | table | olivier
 public | sac             | table | olivier
(2 rows)
```
Tout va bien.

Pour vérifier si la table `sac` contient bien des choses, on va réaliser notre première selection de données sur une table:

```postgres
select * from sac; /* affichage de toutes les lignes de la table */
```
Cette commande nous retourne toutes les lignes de la table `sac`.
Résultat :
```sql
id |     categorie     |             materiel             |             type             | dans_sac | poids_en_g |   present   |         commentaire          | pese |     emplacement
----+-------------------+----------------------------------+------------------------------+----------+------------+-------------+------------------------------+------+----------------------
  1 | 0 - Vêtements     | Chaussures rando                 |                              | Non      |        690 | Oui         |                              |      | Hors sac
  2 | 0 - Vêtements     | Polaire                          |                              | Oui      |        318 | Oui         | Au lieu de 380               | Oui  | 2
  3 | 0 - Vêtements     | Sandales                         |                              | Oui      |        300 | Non         | A acheter plus tard          |      | Grande poche
  4 | 0 - Vêtements     | Pantalon                         |                              | Oui      |        213 | Oui         |                              | Oui  | 2
  5 | 0 - Vêtements     | Merinos manches longues          | Odlo vert                    | Oui      |        212 | Oui         |                              | Oui  | 2
...
```
Et bien d'autres encore (il est possible de naviguer dans les données retournées avec les fleches haut et bas, pour sortir, appuyer sur `Q`)

Attention `*` se réfère aux colonnes et non aux lignes, pour le montrer on peut réduire le nombre de colonnes retournées en listant en remplacant `*` par ceux souhaités. Mais comment connaitre le nom des champs de la table `sac` ? avec la commande `\d sac`!

On voit ici 2 types de champs
- integer
- character varying -> Cela signifie que la taille est variable, mais limitée au paramètre; si j'essaie plus grand, alors il y aura une erreur.

Il y en a d'autres
- date : ben une date exprimée au format iso YYYY-MM-DD HH-MM-SS.xxxx+2
- money : valeur monétaire avec des séparateurs entre milliers et le signe currency a la fin
- float : nombre en virgule flottante
- ... voir la doc postgresql pour la liste exhaustive

On voit aussi qu'il y a une colonne `Nullable`. Quand il y est marqué `not null`, cela signifie que la valeur ne peut être nulle ou non définie, sinon erreur.


```sql
olivier=> \d sac
   Column    |          Type          | Collation | Nullable |             Default
-------------+------------------------+-----------+----------+---------------------------------
 id          | integer                |           | not null | nextval('sac_id_seq'::regclass)
 categorie   | character varying(255) |           | not null |
 materiel    | character varying(255) |           | not null |
 type        | character varying(255) |           |          |
 dans_sac    | character varying(32)  |           | not null |
 poids_en_g  | integer                |           |          |
 present     | character varying(32)  |           |          |
 commentaire | character varying(255) |           |          |
 pese        | character varying(32)  |           |          |
 emplacement | character varying(128) |           |          |
Indexes:
    "sac_pkey" PRIMARY KEY, btree (id)
```
Faisons une selection un peu plus 'short' en ne gardant que 3 colonnes

```sql
olivier=> select categorie, materiel, poids_en_g from sac;

     categorie     |             materiel             | poids_en_g
-------------------+----------------------------------+------------
 0 - Vêtements     | Chaussures rando                 |        690
 0 - Vêtements     | Polaire                          |        318
 0 - Vêtements     | Sandales                         |        300
 0 - Vêtements     | Pantalon                         |        213
 0 - Vêtements     | Merinos manches longues          |        212
 0 - Vêtements     | coupe-vent                       |        194
 0 - Vêtements     | Bermuda                          |        233
 ...
```
Il n'y a plus que les colonnes demandées, par contre il y a toujours 86 lignes.

Je peux faire des milliers de choses avec SQL, comme par exemple, lister quelles sont les différentes catégories de matériel de mon sac. Pour cela je tente:

```sql
select distinct(categorie) from sac;
```

Je vois que c'est un peu le foutoir dans cette liste, est-ce que par hasard, je pourrai pas mieux 'ranger' l'affichage des catégories ?

```sql
select distinct(categorie) from sac order by categorie;
```

La je viens de rajouter une clause "order by" qui permet de classer les lignes suivant un ordre précis, ici le champ catégorie. Si je veux trier par ordre descendant (l'ordre est ascendant par défaut), je rajoute `desc` apres order by.

```sql
select distinct categorie from sac order by categorie desc;
```

Du coup, je peux réaliser des opération plus complexes sur cette table, comme par exemple lister les champs matériel, emplacement et poids_en_g, trier par poids décroissant, cela donne:

```sql
select materiel, emplacement, poids_en_g from sac order by poids_en_g desc;
```

Et si maintenant je veux affichier le top 10 des matériels les plus lourds ?

```sql
select materiel, emplacement, poids_en_g from sac order by poids_en_g desc limit 10;
```
Je viens d'ajouter une nouveauté, la clause `limit <nombre [offset départ]>` qui vient après order, du coup, postgres ne me renvoie que les 10 premières lignes.

SQL est également capable de faire des calculs; par exemple, et en dehors de toute table, je peux faire des opérations telles-que :

```sql
select ((3 + 7) * 12) / 22; /* sous forme de résultat entier */
select ((3 + 7) * 12) / 22.0; /* sous forme de résultat en nombres flottants */
select now(); /* renvoie la date et heure actuelle */
select pi(); /*  la liste est longue */
```
Je peux faire des calculs sur les données avant de les afficher, par exemple je vais demander a postgres de multiplier par pi (3.1415) tous les poids ... ca marche, mais bon ...

```sql
select categorie, poids_en_g * pi() from sac limit 10;
```
Plus utile, je peux demander a postgres de faire la somme des poids de toutes les lignes; la du coup, j'aurai le poids total de mon sac.

```sql
select sum(poids_en_g) from sac;
```
Après on peut vraiment aller très loin dans la structuration d'une requete pour affiner les résultats, par exemple, si je veux connaitre le poinds des différentes poches et emplacements de mon sac, j'aurai recours à la requete suivante:

```sql
select
    emplacement, sum(poids_en_g)
from sac
group by emplacement
order by emplacement;
```
Et si je veux savoir les poids sans l'eau et la nourriture du contenu du sac:

```sql
select
    emplacement, sum(poids_en_g)
from sac
    where categorie not like '%8 -%' and dans_sac = 'Oui'
group by emplacement
order by sum desc;

```
Bon, pour la selection de données, vous commencez à voir la puissance du truc. Pour le reste

## Insert
Pour inserer des nouvelles lignes dans la table, c'est très simple, la syntaxe de l commande est :
```sql
insert into <table> (champ1, champ2, ...) values (value1, value2, ...), (...)
```
Voir la documentation complète à ce sujet : [ici](https://www.postgresql.org/docs/current/sql-insert.html)

Par exemple si je veux insérer un ordi portable dans mes affaires :
Il faut faire attention a renseigner à minima les champs marqués `not null` dans la définition de la table.
```sql
insert into sac ()

## Update
Il est également possible de modifier des lignes existantes de la table, pour ce faire, utiliser la commande :
```sql
update <table> set champ1=value1, champ2=value2, ... where <where clause>
```

Par exemple poir modifier le poids de la tente, il faudra executer la requete en ciblant la ligne 'Tente':
```sql
update sac set poids_en_g=1234 where materiel='Tente'
```
Mais cela peut etre imprécis (si on emporte deux tentes par exemple), il est préférable de faire la selection avec l'id (qui est créé pour cela). En l'occurence, celui de la tente est 59, donc, on peut effectuer :
```sql
update sac set poids_en_g=1234 where id=59
```
La on est sur de notre coup.

Attention, là encore, ne **JAMAIS** faire d'update sans cibler une ou plusieurs lignes avec une clause where. La ligne `update sac set poids=1234';` aura pour effet de remplacer tous les poids de toutes les lignes par 1234, oups!


## Delete
On peut supprimer des lignes avec la commande `delete from <table>`, à manipuler avec précaution, et **JAMAIS** sans clause `where`. Pourquoi ?

Parce que si j'execute la commande `delete from sac`, cela aura pour effet de supprimer **toutes les lignes** de la table `sac`; un peu radical ...

## Les transactions

Pour finir notre tour d'horizon du langage SQL, il nous reste à parler des transactions. On a dit plus haut qu'une base de données offrait des mécanismes permettant d'assurer l'intégrité des données.

Prenons un exemple pour expliciter le besoin:
J'ai en charge une application pour une centrale de réservation. ok. Cette application permet de réserver une place dans un train.

Après consulation et choix d'une place, le client confirme sa réservation. Pour cela, il faut effectuer coup sur coup une requete `select` pour vérifier que la place demandée est bien libre dans le train visé, puis tout de suite après, effectuer une requete `update` pour marquer la place comme étant réservée avant de retourner ok au client

Rien de bien compliqué, non ?

Sauf que ...

Immaginons que nous soyons plusieurs a regarder et réserver les places de train (n'oublions pas qu'une BD permet les accès concurrents).
tant que chaque client effectue sa réservation a des moments différents, tout se passe bien. Mais si deux client réservent au meme moment, que risque-t-il de se passer ?

Et bien le risque dans ce cas, est que :
- le select 'place 12 libre ?' du client 1 arrive à la bd
- la bd répond 'place 12 libre'
- le select du client 2 'place 12 libre ?' arrive a la bd
- la encore, la bd retourne 'place 12 libre' (eh oui, rien n'a été réservé pour l'instant)
- Du coup, l'update 'place 12 occupée par client 2' part du 2eme client
- place réservé pour le 2eme client, fin pour le client 2
- l'update 'place 12 occupée par client 1' arrive quelques ms plus tard à la BD (eh oui il ne sait pas qu'entre temps client 2 à été plus rapide!)
- La on ne peut que constater le probleme qui va arriver et mettre le 2eme client dans la misère: la DB update et répond 'place 12 occupée par client 1' -> La résa du client 2 est purement et simplement poubellisée!

Bon tout cela est très simplifié, mais l'idée est de vous faire comprendre que dans ce cas, le select conditionne l'update, les deux requetes doivent être groupées et ininterrompues. Mais comment faire ? depuis le back-end ce n'est pas possible !

Un autre cas de figure difficile est lorsqu'il faut faire 2 updates a la suite - par exemple la gestion d'un virement bancaire.

Un virement c'est :
- un update (solde = solde - 300€) du compte du client emetteur
- un autre update (solde = solde + 300€) du compte du client recepteur

Mais si le 2eme update se passe mal, il ne faudrait pas que le 1er update soit réalisé non plus, mais trop tard !

C'est là qu'intervient la notion de transaction.

En SQL il est possible d'encadrer la séquence select-update par les mots clé :
- `begin` et `commit` si tout se passe bien, l'ensemble des transactions est rendu visible.
- `begin` et `rollback` si il y a une erreur. Dans ce cas, l'ensemble des requetes du bloc est ignoré.

Quand un utilisateur démarre une transaction, l'ensemble des requetes qu'il y effectue est invisible des autres utilisateur. L'ensemble des requetes devient visible aux autres utilisateurs après le commit, ou est définitivement perdu après le rollback.

Voila c'est la dèrnière fonctionalité importante du langage SQL et des bases de données. Maintenant il faut pratiquer, pratiquer et pratiquer ;-)
