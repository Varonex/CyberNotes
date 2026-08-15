# Introduction au "path traversal"

Le "path traversal" (ou parcours de chemins) consiste à pouvoir accéder à des ressources protégées (ou du moins qui ne sont pas conçues pour être disponibles en ligne) via des points d'entrée.
Le but est d'exfiltrer des données via divers techniques. Dans certains cas, il est possible de faire du defacing (écriture) selon le contexte.
# Lecture de fichiers arbitraires

Lorsqu'il y a du code ressemblant à
```html
<img src="/loadImage?filename=218.png"/>
```
Cela veut dire que le serveur s'occupe de récupérer l'image par une certaine méthode utilisant l'entrée utilisateur via le GET.
De ce fait, il est envisageable d'entrer d'autres chemins, permettant d'accéder à l'arborescence du serveur selon l'url `/loadImage?filename=../../../etc/passwd`. Ici, le but a été de récupérer les utilisateurs linux.

Ceci fonctionne car le code ressemble sans-doute à
```php
// /loadImage
<= file_get_contents("/var/www/images/" . $_GET["filename"]); =>
```
Donc le chemin généré est `/var/www/images/../../../etc/passwd` équivalent à `/etc/passwd`. 
# Techniques supplémentaires

L'usage de chemins relatifs peut être bloqué, cependant certaines applications peuvent permettre l'usage de chemins absolus en fonction de l'implémentation.

Sur windows, `/` et `\` sont valides.

Il est possible de bypasser le tronquage en doublant et échappant des caractères. Si une appli transforme `..` en `.` et camoufle les `/`, on peut essayer
- `../` => `....//` : dédoubler tout
- `.....\/` => dédoubler les points et "échapper" le `/`

Certaines fois, comme par exemple pour les formulaires en mode `multipart/form-data`, le système peut stripper les chemins avant d'envoyer le contenu à l'application. Dans ce cas, il est possible d'url-encoder `../` une voire deux fois, donnant `%2e%2e%2f` une fois, `%252e%252e%252f` deux fois. Les séquences `..%c0%af` et `..%ef%bc%8f` peuvet également marcher. Dans la solution du lab, on utilise `..%252f` comme flag de remplacement.

Parfois, inclure le chemin de base peut être utile si l'application vérifie que le début du chemin matche. On peut donc passer le chemin complet depuis la source : `/var/www/images/../../../etc/passwd` avec la source étant `/var/www/images` évidemment.

Enfin, la dernière technique consiste à donner une extension de fichier. Évidemment, faire ceci bloquerait la bonne interprétation du fichier. Mais pour rappel, les strings en C se terminant par `\0`, on peut introduire un tel caractère avant l'extension, ralliant la présence d'une extension lors de la vérification métier à l'absence de parsing lors de l'inteprétation du chemin. On obtient `../../../etc/passwd%00.png` !
# Comment régler le problème

À partir d'un chemin, il faut vérifier si le chemin ne sort pas des bounds routées.
```php
$cheminOriginel = "/var/www/images";
$entreeUtilisateur = $_GET["filename"];

// Recréation d'un chemin permettant l'interprétation et l'obtention du chemin canonoique
$chemin = recreerChemin($cheminOriginal, $entreeUtilisateur);

if ($chemin.getCanonicalPath().startsWith($cheminOriginel))
{
	// Logique
}
```
On construit le chemin, on interprète pour produire un chemin sans redirection (`/var/www/images/../../../etc/passwd` devient `/etc/passwd`) et on vérifie si la racine correspond bien à l'endroit où on veut piocher les images.