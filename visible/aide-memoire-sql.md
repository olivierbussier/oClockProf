## SQL et PSQL - Aide mémoire

- Les éléments entre <> sont des éléments à renseigner obligatoires
- les éléments entre [] sont facultatives

### Commandes SQL de base

- Lister les databases existantes :
```sql
SELECT datname FROM pg_database;
```
- Créer une database :
```sql
CREATE DATABASE <database>;
```
- Supprimer une database :
```sql
DROP DATABASE <database>;
```
- Créer une table (pour plus de détail, allez voir le site https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-create-table/)
```sql
CREATE TABLE [if not exists] test (
    id serial primary key,
    nom varchar(255) ,
    prenom varchar(255) ,
    created_at timestamp,
    ...
);
```
- Créer un utilisateur et lui donner les droits sur une database:
```sql
create user <user> [with password 'password'];
GRANT ALL ON DATABASE <database> TO <user>;
```
- pour ajouter un password à un compte existant

```sql
alter user user with password 'password';
```
Toutes les commandes SQL ci-dessus peuvent etre éxécutées dans psql. Les commandes sql peuvent être chainées grace au séparateur ';', exemple :
```sql
create user user_essai ; grant all on ...
```
## Commandes PSQL
psql est un outil de ligne de commande interactif pour `postregsSql`. Avec lui on peut executer n'importe quelle commande SQL.

Pour lancer psql (et éventuellement se connecter a une database précise).

```sql
psql [database]
```
Attention, par défault, le compte utilisé pour se connecter au SGBD est celui du login courant (on peut le voir avec la commande `whoami`); et la database séléctionnée est celle qui porte le même nom que le user.

Liste des commandes psql les plus communes
- `\c [database]` : Séléctionner une database (sans paramètre, indique la dabatase active):

- `\l` : Liste les databases
- `\d table` : Liste les champs de la table `table`
- `\dt` : liste les tables de la database active
- `\du` : Liste les utilisateurs
