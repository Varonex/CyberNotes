Le Cross Site Scripting (XSS) est une technique qui consiste à injecter d'une quelconque manière du code javascript sur une page web. Ce code javascript sera exécuté lorsqu'une victime visitera la page infectée, ou après qu'elle ait réalisé une action provoquant l'attaque.
La portée du XSS est vaste, cette attaque permet de voler des données sensibles, se faire passer pour la victime et lui faire réaliser des actions sans même qu'elle s'en rende compte. Si la victime a des privilèges élevés, le XSS permettra à l'attaquant d'accéder à des ressources protégées.
# Fonctionnement de base

Le XSS fonctionne par l'injection d'une balise `<script>` au sein de la page. Par exemple, un site comportant une barre de recherche pourrait être victime d'attaque XSS si, lorsqu'on tape `<script>alert(0);</script>` dans la barre de recherche, une boîte d'alerte apparaît sur l'écran.

On ne va pas s'amuser à aller taper les scripts sur les machines hôtes dans des boîtes de recherche. Si l'url de la recherche comporte un champ de recherche, on peut altérer directement ce champ dans l'url et la faire transiter : `https://mywebsite.com/search?q=<script>alert(0);</script>`. Il faut évidemment rendre l'uri web-friendly.

Les balises `<script>` prennent un attribut `src` permettant de spécifier l'url du script désiré. Ainsi, un long script peut être récupéré via `<script src="https://malicious.script"</script>` directement, limitant la visibilité de l'attaque.

> `alert()` peut être bloqué dans certains types d'attaques comprenant des iframes. Pour ce faire, l'usage de `print()` permet d'invoquer une modale d'impression de page, donnant exactement la même confirmation visuelle qu'`alert()` donne.
# Les différentes approches

Le XSS peut se dérouler sous différentes approches fondamentales. On note les approches :
- Self-XSS : on provoque la victime à s'infecter elle-même.
- XSS réfléchi : dès que le serveur web reçoit une requête HTTP, il retranscrit le payload dans le HTML retourné.
- XSS stocké : dès que le script malveillant est stocké en DB.
- XSS depuis le DOM : la vulnérabilité existe dans la page HTML directement, et aucun requêtage web n'est nécessaire pour la prouver (`innerHTML`, `document.cookie`, ...).
### XSS Réfléchi

Le XSS réfléchi se fait dès que le serveur web donne la réponse à une requête donnée.
#### Exécuter un XSS réfléchi

Plus haut, il a été fait mention d'une barre de recherche. Ce composant va provoquer une requête au serveur, qui va retourner un résultat et afficher le contenu HTML, ainsi que le contenu de la recherche. Le contenu de la recherche étant malicieux, le script est interprété.

```
https://amazon.com/search?q=<script>print();</script>
```
> Le site renvoie le résultat html de la page, avec la barre de recherche / un titre qui contient
```html
<h4><script>print();</script></h4>
```
Ainsi, le script est exécuté et l'affichage montre la page d'impression. Mais évidemment, le script peut venir d'ailleurs et comporter du code malicieux additionnel.

Comme le script malicieux s'exécute sur la page web de la victime, le code malicieux peut exécuter des tâches, et les credentials de la victime seront automatiquement utilisés.
#### Tester la présence d'un XSS réfléchi

Pour automatiser le processus, on peut utiliser différentes stratégies comme :
- Récupérer tous les points d'entrée du site web, et y injecter un numéro au hasard par point d'entrée. Ensuite, on récupère la réponse et on regarde si les numéros sont dans la page. Chaque numéro va permettre d'identifier un point d'entrée susceptible d'être utilisé, et on peut ensuite essayer la méthode du `alert()`.
- Essayer de manipuler toutes les valeurs d'une requête HTTP

> Il faut essayer plusieurs payloads différents, et surtout si un XSS est trouvé dans Burp Suite, il faut essayer la PoC sur un vrai navigateur pour être certain qu'aucun blocage n'est réalisé de ce côté.
### XSS stocké

Le XSS stocké consiste à faire persister le XSS dans la DB du site, ce qui va permettre d'atteindre plusieurs victimes. Cette technique fonctionne lorsqu'il y a un moyen de stocker des entrées utilisateurs non échappées (comme un forum, un journal, et cetera).
#### Exécuter un XSS stocké

Par exemple, j'écris un commentaire :
```
Bonjour, je suis Carlos de Carglass
<script>alert();</script>
```
J'appuye sur le bouton "envoyer le commentaire", formant ce genre de requête
```http
POST /commentaire HTTP/1.0
Host: amazon.com
Content-Length: xxx

comment=Bonjour,+je+suis+Carlos+de+Carglass+%3Cscript%3Ealert();%3C%2Fscript%3E
```
et le serveur stocke cette entrée en DB.
Une fois qu'un autre utilisateur va sur la page où ce commentaire doit apparaître, le contenu va être mis sur la page, comprenant la balise script qui sera exécutée.

Certains sites webs demandent à ce que les utilisateurs soient connectés pour voir les commentaires, rendant cette attaque utile car nous savons que les victimes seront connectées au moment où elles subissent l'attaque.
#### Tester la présence d'un XSS stocké

Les XSS stockés peuvent se trouver dans :
- les paramètres de requêtes HTTP qui engendreraient une persistance
- l'url
- les points d'entrée classiques qui engendreraient une persistance

Dans l'ensemble, comme pour le XSS réfléchi, on peut envoyer une série de nombres aléatoires et voir s'ils apparaissent sur la page, et tester la persistance.
La persistance peut être à durée limitée. Par exemple un système de suggestions de recherches pourrait être vulnérable, mais à une durée limitée.
### XSS depuis le DOM

Le XSS depuis le DOM se déroule dès que le javascript de la page client va exécuter du code unsafe. Il ne s'agit pas d'un serveur qui aurait rendu la page web avec une balise script malicieuse (comme un serveur php duquel un script est injecté), il s'agit bien d'une insertion au niveau **local** : le client directement provoque l'interprétation de code js malicieux.
#### Exécuter un XSS depuis le DOM

On peut par exemple via l'usage de :
```js
eval(...); // Comme loadstring() en lua
document.getElementById("hi").innerHTML = ...; // Load des éléments malicieux
window.location = ...; // Envoyer sur une page malicieuse
document.write(...);
```
Toutes les méthodes d'évaluation unsafe de javascript vont devenir un vecteur XSS.

On peut aussi imaginer un XSS depuis l'url, notamment avec les pages 404, qui lorsqu'elles sont localement rendues, peuvent être amenées à *automatiquement* mettre le nom de la page invalide via un code javascript, sans l'intermédiaire d'un serveur.
#### Tester la présence d'un XSS depuis le DOM

*Les tests de XSS depuis le DOM pouvant se faire intégralement localement, des outils existent pour scanner une page web agressivement dès la réception de son contenu.*

- On compte divers manières qui consistent à essayer de détecter si des éléments entrés apparaissent sur la page. Par exemple, on peut introduire `localtion.search` dans la page html puis recherche sur la page. Si url-encodage il y a, il y a des chances pour que ça ne marche pas.
> `location.search` est une variable js pour obtenir les variables GET de l'url. Par exemple
```js
// Sur url "https://www.url.com?a=1&b=2"
console.log(location.search);
// Met "?a=1&b=2"
```
Par exemple, si j'ai une page qui fait
```html
<!-- url = "https://www.url.com?s=hi" -->
<img src="url.com?img=hi"/>
```
On peut tester l'url `"/><svg onload="alert(1)"/>` pour provoquer
```js
<img src="url.com?img="/><svg onload="alert(1)"/>
```
et provoquer une alert.

On peut aussi avoir ça comme trigger
```js
<img src="<nonexistent" onerror="alert(1)"/>
```

- Pour détecter des portes d'entrée dans le code javascript directement, c'est toujours plus compliqué. Il y a pas mal de techniques qui consistent à utiliser le debugger et/ou faire de la recherche de string sur la page.
- Usage de DOM-invader (des programmes autonomes vérifiant la présence d'XSS depuis le DOM)
#### Exploit de l'XSS depuis le DOM

