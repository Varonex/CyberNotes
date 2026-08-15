*Ce fichier compile le premier cours de PortSwigger ayant des informations substantielles non exhaustives.*
# Traversée des répertoires

Lorsqu'un serveur demande une ressource (image, ...), il va passer par une URI.
Certaines URI pointent vers le serveur lui-même, indiquant qu'il host les ressources.

Un serveur à une url `url.com` peut accéder à ses ressources via du développement d'URI. Par exemple, `url.com/images/image1.png` est une image nommée `image1.png` qui se trouverait dans le dossier `images`, situé dans la racine du serveur.
Le serveur peut, depuis la page `url.com`, requêter l'image via `/images/image1.png` par concaténation implicite à l'URI de base.
### Accès aux ressources interdites

Sur un site contenant des images (ou même, on peut essayer de requêter directement en GET dans l'url), on peut avoir

```html
<img src="/image?name=15.png"/>
```

Ce code suggère un accès GET où le nom de l'image est passé en paramètre à la ressource `image` du serveur. Admettons que cette image soit stockée dans `/var/www`, la requête chercherait l'image `/var/www/15.png`. Si aucun contrôle du chemin n'est fait, on peut query `/image?name=../../../etc/passwd` pour récupérer le fichier `password` du système.
Le chemin n'étant pas vérifié, et étant relatif, on peut récupérer le contenu par requête, et le lire sans aucun problème.
# Contrôle d'accès

Certaines personnes pensent qu'attribuer une url customisée est la solution. Bien que ça puisse temporairement aider, ça ne va pas permettre de garantir une authentification quelconque.
## Élévation verticale des privilèges

**Élévation verticale des privilèges :** Dès qu'un utilisateur gagne l'accès à des ressources protégées par un rôle spécialisé. (= l'utilisateur a monté dans la hiérarchie)
#### Fichier robots.txt

Un fichier nommé `robots.txt` permet de désactiver le crawling pour des url données. Parfois, ce fichier est accessible et peut être lu via `url.com/robots.txt`. Dans ce cas précis, au dépend du contenu du fichier, des url cachées peuvent leak et permettre à des attaquants d'accéder à la page web.
#### Fonctionnalités dans la page

Ce design touche au domaine de "Obscurité comme sécurité".
Le code javascript peut aussi leak ces url s'il y a du code comme :
```html
<script>
	var isAdmin = false;
	
	if (isAdmin)
	{
		...
		var adminTag = document.createElement('a');
		adminTag.setAttribute('href', 'url.com/secret_page');
		adminTag.innerText = 'Admin Panel';
		...
	}
</script>
```
Ce code va directement donner les url cachées car il sera partagé pour tous les clients.

La logique ci-dessus peut s'appliquer à des cookies. Quand on se connecte, il existe peut-être un cookie additionnel déterminant si l'utilisateur est administrateur. Cette erreur permet à l'attaquant de modifier la valeur du cookie, et de faire apparaître des éléments visuels directement.
#### Sur la base de paramètres

Ce design touche au domaine de "Obscurité comme sécurité".
Une autre approche consiste à exploiter les paramètres d'une requête, souvent en GET.
Un exemple reste les forwardeurs comme en PHP, du style
```php
<?php
	$page = $_GET["p"];
	
	$pages = [
		"home" => "pages/home.php",
		"secret-admin-6556d" => "pages/admin.php",
	];
	
	if (isset($pages[$page]))
		include($pages[$page]);
	else
		include("pages/404.php");
?>
```
Ce type de forwardeur, bien que sympa, peut poser problème sans aucune autre vérification de requête. Quelqu'un de malicieux peut donc faire `url.com/index?p=secret-admin-6556d` pour accéder à la page. Le tout est de trouver la chaîne de caractères à rentrer.
## Élévation horizontale des privilèges

**Élévation horizontale des privilèges :** Dès qu'un utilisateur peut accéder à ses ressources et aux ressources d'autres utilisateurs de même rang en simultané. (= l'utilisateur a accès aux ressources bloquées du même niveau de hiérarchie)
#### Sur la base de paramètres

Certaines url peuvent donner accès à des paramètres permettant de parser l'id de la session en cours. Ceci est un gros nono, mais cela donne lieu à des url comme `url.com/myaccount?id=123`. Évidemment, cet id est manipulable. Si la DB utilise des IDs incrémentants, alors le pattern est tout dessiné. L'usage de GUID rendrait cette attaque plus difficile.
Peu importe le type d'id, ce genre de technique n'est absolument pas sécurisé.

Ainsi, si on se connecte et on obtient `url.com/myaccount?id=123` et qu'on obtient les infos de notre compte, puis qu'on se balade sur le site, et on arrive sur `url.com/profile?id=456`, on peut essayer `url.com/myaccount?id=456` pour voir si ça marche.
Le tout est de regarder les ids dans l'url, comme pour Roblox !

Évidemment, si cet accès permet d'avoir un compte administrateur, il s'agit d'une élévation verticale.
# Vulnérabilités d'authentification

**Authentification :** Processus de vérification qu'un utilisateur est bien celui qu'il prétend être.
**Autorisation :** Vérification de la faisabilité d'une action que fait un utilisateur (= credentials).
### Attaques par bruteforce

Cette technique implique d'essayer toutes les combinaisons réalisables pour cracker une sécurité. Souvent, un dictionnaire est utilisé (comme rockyou.txt) voire grâce à des rainbow tables. Ces devinettes ne se font pas forcément au hasard, mais aussi par des biais statistiques. Le processus est automatisé afin de faire le plus d'essais en un temps minime.
Souvent, les plateformes mettent en place des temps d'attente afin d'éviter le spam. Les sites ne mettant pas un tel mécanisme en place sont vulnérables à ce type d'attaque.

Les pseudonymes sont souvent faciles à déterminer, surtout si une logique de formatage apparaît (`firstname.lastname@company.com` permet de deviner un email potentiel). Les credentials suffisent parfois, comme avec les pseudos `admin`, `administrator`, ...
Certains sites (forums, ...) peuvent mettre à disposition des pseudonymes sur l'interface. Certains sites cachent l'information, mais elle subsiste dans les headers (via l'entête `From`) ou dans l'html (au avec `hidden="hidden"` ou via CSS).

Les mots-de-passe suivent une logique similaire, cependant la notion de "force" du mot-de-passe peut rendre un bruteforce difficile. Beaucoup de sites mettent en place des règles de validation de mdps, rendant la tâche plus difficile. On note tout-de-même que ces règles étant uniformes, certaines stratégies et patterns communs existent.
Généralement, les gens prennent un mdp simple qu'ils connaissent et y ajoutent les éléments manquants. Ainsi, `motdepasse` pourrait devenir `Motdepasse!1`, ou `M0tdePa$$e`. Si le mdp doit être souvent renouvelé, les gens auront tendance à reprendre l'ancien, et le modifier légèrement.
### Énumération des pseudonymes

L'énumération des pseudonymes consiste à vérifier si un nom d'utilisateur est effectivement déjà pris. Souvent, cette stratégie se remarque sur la page de login (textes d'erreur qui spécifient que le mdp n'est pas bon, mais que le username l'est), de reset de mdp (confirmation d'envoi de mail si mail valide ET erreur si mail invalide), ... qui donnent trop d'information.
Une énumération avec longueur variable de réponse permet d'identifier 
### Contournement d'authentification à deux facteurs

Parfois, l'authentification à deux facteurs peut être complètement contournée, en fonction de l'implémentation.
Quand l'utilisateur rentre un mdp, et qu'il est redirigé sur une autre page pour obtenir un code de vérification, *alors* l'utilisateur est dans un état intermédiaire. Au dépend de comment l'implémentation est faite, cet état intermédiaire peut correspondre à un état de login. Revenir à une page login-only dans cet état peut donc fonctionner.
# Contrefaçons de requêtes côté serveur

Correspond à une vulnérabilité permettant aux attaquants de dire au serveur de faire des requêtes à des ressources imprévues. Par exemple, un serveur pourrait établir une connexion à un serveur interne, ou à un serveur externe malicieux.
### Attaques contre le serveur

Cette attaque consiste à faire une requête à l'origine via son interface `127.0.0.1`/`localhost` (appelée interface `loopback`).
Si une requête est structurée du type :
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-urlencoded
Content-Length: 47

stockApi=http://some.url.com/api/v1/stock?key=1
```
Alors on peut essayer de faire :
```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-urlencoded
Content-Length: 47

stockApi=http://localhost/admin
```
Le serveur va donc potentiellement query le `localhost`.
On note que comme l'accès à la dashboard admin se fait via le serveur (qui fait office de machine hôte dans ce contexte), il y a des chances que l'accès à la page donne des informations intéressantes car les contrôles d'accès sont potentiellement désactivés, ne demandant aucune authentification ni autorisation.

Le simple fait que les applications croient le serveur en tant qu'exécutant peut s'expliquer car
- Les checks sont à un autre endroit dans le code,
- Les accès admin peuvent être donnés sans login depuis la machine hôte, afin de pouvoir opérer sans les logins dans le cas où ils seraient perdus
- L'interface admin écoute un autre port que le reste de l'app, n'étant pas normalement joignable depuis le monde extérieur
Ces raisons rendent probable le fait que la machine hôte (= serveur) puisse avoir des accès privilégiés.

Dans certains cas, le serveur a accès à des ressources non visibles sur internet. Généralement, aucun contrôle d'accès existe entre le serveur et cette ressource privée. Ainsi, il est plausible que le serveur puisse exécuter une API qui ressemble à `http://192.168.0.45/` avec l'IP `192.168.0.45` qui correspond à la ressource privée.

Pour faire un scan primaire, il est possible d'itérer sur les adresses IP pour demander un service spécifique, si tenté qu'il soit connu.
Pour le reste, la logique d'exécution reste la même !
# Vulnérabilités d'upload de fichiers

Lors de l'upload d'un fichier sur un serveur distant, un utilisateur peut essayer de truander en manipulant le nom, le type, le contenu ou la taille du fichier. Si le serveur ne valide pas suffisamment, alors des fichiers malicieux peuvent être uploadés sur l'origine.

Le manque de restriction complet est rare. Cependant, des restrictions faites maison existent, et sont remplies de failles, voire qui sont contournables. Certains sites peuvent regarder les métadonnées, ces dernières peuvent malheureusement être modifiées à la guise du client au préalable.

### Notion de shell en ligne

Une shell en ligne ("web shell") correspond aux techniques qui permettent d'exécuter du code arbitraire à distance via des techniques.
Les fichiers permettant de faire cela sont les fichiers exécutables. Le pire est si le serveur exécute ce fichier malicieux, en faisant par exemple :
```php
<= file_get_contents('/path/to/file'); =>
```
Après exécution, le contenu du fichier est entièrement disponible.

L'injection par paramètres peut aussi être utilisée dans le fichier malicieux, permettant d'exécuter des commandes système, par exemple :
```php
<= system($_GET['command']); =>
```
Ainsi, si ce code se lance à l'url `url.com/ressource.php`, alors il est possible d'exécuter `url.com/ressource.php?command=id HTTP/1.1`, exécutant la commande `id HTTP/1.1` et renvoyant la réponse.

### Exploitation des restrictions

Les formulaires html fonctionnent majoritairement en `POST` via le type de contenu `application/x-www-form-urlencoded`, qui est correct pour quelques champs. Dans le cas d'un envoi de fichier volumineux, il vaut mieux privilégier le type `multipart/form-data` qui gère mieux les grands volumes.

Voici l'extrait d'un tel formulaire :
```
POST /images HTTP/1.1
Host: normal-website.com
Content-Length: 12345
Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="image"; filename="example.jpg"
Content-Type: image/jpeg

[...binary content of example.jpg...]
---------------------------012345678901234567890123456
Content-Disposition: form-data; name="description" This is an interesting 

description of my image.
---------------------------012345678901234567890123456
Content-Disposition: form-data; name="username"

wiener
---------------------------012345678901234567890123456--
```
Tout est séparé en chunks dans le type `multipart/form-data`. Le délimiteur (`boundary`) est fixé à `---------------------------012345678901234567890123456`, et à chaque fois que ce délimiteur est rencontré, on passe à une prochaine section.
Chaque `Content-Disposition` correspond à une variable du formulaire. La valeur du champ est isolée entre une ligne vide après les entêtes locales à chaque valeur et la prochaine occurrence du délimiteur. Le dernier délimiteur est suffixé de `--`.

Les descripteurs `Content-Type` affinent le type attendu. Ici, `image/jpeg` attend un `jpeg`. La valeur aurait aussi pu être `image/png`. Ces types ne font pas office de vérification, et si le serveur ne vérifie pas plus loin, alors le contenu peut être autre chose que le serveur traitera.
# Injections de commandes OS

Le but de cette manœuvre est d'exécuter des commandes système sur l'origine. L'upload de fichiers est un point d'entrée (cf Vulnérabilités d'upload de fichiers), mais là le but est de les injecter directement.

Il est recommandé d'exécuter des commandes de base au préalable, dès qu'une faille à injection a été trouvée (comme `whoami`, `uname`, `ifconfig`, ...).
### Par injection de paramètres

Admettons qu'un site web lance la commande `cat hi` via l'url `url.com?file=hi`. Le paramètre `file` de l'url est directement injecté dans la commande. Ainsi, on peut détourner l'usage de la commande avec `url.com?file=hi & echo hey & hi`. Par commande asynchrone, ce moyen permet de voir si l'exécution se produit comme voulu.
**Le symbole `&` étant utilisé pour l'enchaînement de variables, il faut songer à le remplacer.**

Le résultat donne
> - Contenu de "hi"
> - hey
> - hi: commande inconnue

L'ajout du 3e membre (`& hi`) est utile pour bien délimiter la commande du milieu (`echo hey`). Des fois, le système peut vouloir append des paramètres qui pourraient rentrer dans le `echo` si cette technique n'était pas utilisée.
# Introduction aux injections SQL

Les injections SQL consistent à bypass le stade d'une requête normalisée et non contrôlée.

```sql
select *
from `table`
where col = $name; -- fhsuhsi
```
La variable `name` vient d'une entrée utilisateur. Si cette entrée n'est pas sanitized, alors si l'utilisateur fournit `$name = ' or 1=1; --`, la requête devient :
```sql
select *
from `table`
where col = '' or 1=1; --';
```
ce qui renvoie l'intégralité de toutes les lignes de la table.
La valeur d'entrée peut être changée pour vérifier la validité de cette technique. Comparer les résultats de `or 1=1` et `or 1=2` peut aider pour savoir si la faille se produit.

On peut imaginer la provenance de `name` depuis un formulaire, voire directement depuis des données `GET` et/ou `POST`. L'url `url.com?category=Gifts` peut vite se transformer en `url.com?category=' or 1=1; --`. Le point-virgule n'est pas toujours nécessaire.

La partie `or 1=1` bypass l'intégralité du `where`, ne pas le rajouter bypass le reste de la requête mais garde le début de la clause `where`.

D'autres clauses peuvent être imaginées, tout sql est à la portée de main !