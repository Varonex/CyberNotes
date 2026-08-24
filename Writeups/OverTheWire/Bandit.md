# Niveaux

Débute au niveau 0 jusqu'au niveau 33.

## Niveau 0

De base, `ssh` s'utilise du style `ssh user@host`. L'ajout d'un port peut se faire via l'option `-p` car le port par défaut est 22.

Il faut utiliser la commande
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
password: bandit0
```
## Niveau 0 -> 1

```bash
cd && cat readme
```
Ces commandes donnent le mdp.
Pour se connecter à l'autre compte, on peut quitter la session `ssh` avec `exit` puis écrire
```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
password: put.password.here
```
## Niveau 1 -> 2

Le shortcut `-` sert de référence au flux précédent.
On a par exemple
```bash
echo "Texte" | cat - # "-" fait référence au texte outputté par echo
cat - # pioche dans stdin si aucun pipe précédent
```
Donc on ne peut pas run `cat -` sinon ça va faire de la lecture de stdin. Il faut utiliser le chemin relatif/absolu **complet** e.g `./-` ou `/home/bandit1/-` !
Aussi `cat < -` marche.

```bash
cat ./-
```
## Niveau 2 -> 3

Le nom du fichier est `--spaces in this filename--`.
Faire
```bash
cat "--spaces in this filename--"
```
Ne marche pas. Bash consomme les `--` pour le paramétrage de la commande, même dans une chaîne fermée par `""`.
Il faut préciser à Bash où la liste des paramètres se termine via `--`.

La réponse est
```bash
cat -- "--spaces in this filename--"
```

Comme pour le reste, pour lever l'ambiguïté, on peut aussi faire
```bash
cat "./--spaces in this filename--"
```
## Niveau 3 -> 4

```bash
cd inhere
ls -a
cat ...Hiding-From-You
```
`ls -a` montre les fichiers cachés, qui sont des fichiers préfixés par `.`
## Niveau 4 -> 5

Pour détecter la propriété de lisibilité par les humains, on peut utiliser la commande `file`
```bash
cd inhere
file ./*
```
Cette commande va donner des informations sur les fichiers.
## Niveau 5 -> 6

On va utiliser la commande `file` de nouveau en mode récursif
```bash
file ./*//*
```
Le problème c'est qu'il y a énormément de résultats. Il faudrait filtrer sur la taille.

- Pour obtenir la taille, on peut utiliser la commande `du` qui donne la taille d'un fichier. `ls -l` fait la même chose. `du -b` permet de donner la taille en octets. Sinon on aura la taille en `MiB`.
- Avec `grep`, on peut filtrer sur la taille. On note que `grep -v` cherche les éléments qui ne correspondent pas à l'expression, mais ici ça ne sera pas utile.

- Un autre problème, il faut prendre en compte les fichiers commençant par `.` qui seraient cachés. Pour ce faire, on pourrait utiliser `{.,}` pour préciser que le premier caractère peut être `.`, avec `,` qui veut dire tous les caractères usuels pour des noms de fichier.
Donc on peut écrire par exemple
```bash
cat {.,}*
```
- Avec `*` le wildcard traditionnel, et `{.,}` une addition précisant que le 1er caractère doit être soit `.`, soit un caractère usuel dans les noms de fichier.

- Enfin, la commande `find` prend des arguments comme `-size`.

Ainsi, plusieurs chemins peuvent être utilisés. J'ai choisi
```bash
ls -la * | grep 1033 # La plus simple et accessible. Prend en compte les fichiers cachés via l'option a
du -b ./*//{.,}* | grep 1033
```
La meilleure solution reste `find` via
```bash
find . -type f -size 1033c -readable ! -executable
# -type f : fichier
# -size 1033c : 1033 octets
# -readable : lisible
# ! -executable : PAS exécutable
```

Cette commande est suffisante car un unique fichier a 1033 octets. Il s'agit de `./maybehere07/.file2`
## Niveau 6 -> 7

Pour chercher dans `man`, on peut faire des regexp avec `/`. `n` va au prochain et `N` va à l'ancienne occurrence !!!! `q` pour quitter.

Selon `man find`, on a:
- `-user name` pour le nom ou `-uid id`
- `-group name` pour le groupe ou `-gid id`
- `-size size[c]` pour la taille en octets

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```
## Niveau 7 -> 8

```bash
cat data.txt | grep millionth
```
## Niveau 8 -> 9

- La commande `uniq` factorise les lignes similaires **ADJACENTES !!!**. `uniq -c` permet de mettre en évidence le nombre de fois que cette ligne apparaît **DANS SON VOISINAGE**. `uniq -u` ne garde que les lignes **ADJACENTES** uniques, exactement ce qu'on cherche.
- La commande `sort` fait un tri.

```bash
cat data.txt | sort | uniq -u
```
## Niveau 9 -> 10

- Le fichier contient du contenu binaire. Pour filtrer le contenu afin de ne garder que le texte affichable, on utilise `strings`.
- Ensuite on peut utiliser `grep` pour déterminer les préfixes.

```bash
strings data.txt | grep ===
```
## Niveau 10 -> 11

La commande `base64` permet de traiter ce contenu. Elle encode par défaut mais décode avec le flag `-d`.

```bash
base64 -d data.txt
```
## Niveau 11 -> 12

La commande `tr` a pour but d'appliquer des transformations de **TRANSLATION** (ou suppression, ...) sur des chaînes. On a
```bash
# Suppression
echo "Bonsoir" | tr -d 'o' # Bnsir

# Substitution
echo "Bonsoir" | tr 'o' 'O' # BOnsOir

# Squeezing
echo "Boooonsoiiir" | tr -s 'oi' # Bonsoir

# Shift
echo "Bonsoir" | tr 'a-zA-Z' 'b-zaB-ZA' # Cpotpjs (shift d'un unique caractère)

# Attention, le mapping est vraiment 1-1.
# a-z s'expand abcdefgihklmnopqrstuvwxyz avec 26 caractères.
# b-za s'expand bcdefghijklmnopqrstuvwxyza avec 26 caractères.
# Le mapping fait JUSTE un checkup d'index de la source vers la destination !
```

On obtient :
```bash
cat data.txt | tr 'a-zA-Z' 'n-za-mN-ZA-M'

# Mapping 1-1 par expansion des expressions et correspondance des index
# abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ
# nopqrstuvwyxzabcdefghijklmNOPQRSTUVWXYZABCDEFGHIJKLM
```
## Niveau 12 -> 13

Ce problème est NP-difficile.
On va utiliser la commande xxd pour donner des dumps hexadécimaux du fichier data.txt.
On va se baser sur la partie d'entête des fichiers pour déterminer le type de fichier.

| Bits       | Algo  | Technique             |
| ---------- | ----- | --------------------- |
| `1f 8b`    | Gzip  | `gzip -d archive.gz`  |
| `42 5a 68` | Bzip2 | `bzip2 -d archive.bz` |

Certains longs fichiers mettent en préambule des noms de fichier directement. Ces archives sont probablement des fichiers `tar`, qu'on peut désarchiver via `tar -xvf archive.tar`.

Après avoir fait ça de manière cyclique (utiliser `xxd` sur le fichier pour obtenir l'entête puis utiliser le bon utilitaire pour décompresser/extraire), on trouve le mdp.
## Niveau 13 -> 14

On peut utiliser `sftp` pour get le fichier, comme `scp`.

| Méthode SFTP                                                                                                                                                                                                                                           | Méthode SCP                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| On utilise la commande<br>```sftp -P 2220 bandit13@bandit.labs.overthewire.org```<br>On se connecte avec le mdp déjà obtenu<br>On run `get sshkey.private`<br>On run `exit`<br>=> Le résultat réside dans le workingdir actuel, dès la sortie d'sftp ! | On utilise la commande<br>```scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private .```<br>On se connecte avec le mdp déjà obtenu<br>Tip : l'ordre `remote local` indique un GET, là où un `local remote` indique un PUT<br>Le résultat réside dans le workingdir si `.` est utilisé comme emplacement de dépôt ! |

Une fois `sshkey.private` récupéré, on peut se connecter au 14 avec
```bash
ssh -i sshkey.private bandit13@bandit.labs.overthewire.org -p 2220
```
## Niveau 14 -> 15

Comme on est dans `bandit14`, on peut lire le mdp.

On lance un serveur `netcat` via `nc localhost 30000` et on colle le mdp.
## Niveau 15 -> 16

On lance un serveur supportant TLS via `openssl s_client -connect localhost:30001`
## Niveau 16 -> 17

On scanne le réseau en faisant
```bash
nmap -sV -p 31000-32000 localhost
```
- s : identifie les services
- V : identifie la version
- p : Met une plage de ports à essayer

On obtient plusieurs ports

| Port      | Service     |
| --------- | ----------- |
| 31046/tcp | echo        |
| 31518/tcp | ssl/echo    |
| 31691/tcp | echo        |
| 31790/tcp | ssl/unknown |
| 31960/tcp | echo        |

Les ports associés à `echo` vont juste output le contenu envoyé. Le port `ssl/echo` fait la même chose, mais via encryption `SSL`. Le seul service est `ssl/unknown` sur le port 31790.

On utilise la commande `openssl s_client -connect localhost:31790 -ign_eof`
/!\ le flag `-ign_eof` permet d'éviter le `EOF` qui fait foirer la détection de la bonne réponse

On met le mdp de la room, et on obtient une clé privée RSA.
On la sauvegarde, et on se connecte au prochain niveau via
```bash
ssh -i sshkey.private bandit17@bandit.labs.overthewire.org -p 2220
```
## Niveau 17 -> 18

On utilise
```bash
diff passwords.old passwords.new
```
La différence sur `passwords.new` correspond au mdp.
## Niveau 18 -> 19

La connexion est coupée instantanément via ssh. Mais comme `.bashrc` affecte le terminal en tant que tel, on peut tout-de-même faire un accès sftp et get le fichier.

Une autre méthode est de faire de l'exécution de commande à distance via
```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```
## Niveau 19 -> 20

Le bit `suid` est une permission de fichier (`4000`) qui permet d'overwrite les permissions réelles de l'utilisateur connecté exécutant un exécutable, par les permissions de l'utilisateur qui détient le fichier.

Le fichier est détenu par `bandit20`, et permet d'exécuter des commandes. En l'utilisant, on peut faire
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```
Le fichier sera exécuté comme si `bandit20` le faisait, donnant des permissions extra !
## Niveau 20 -> 21

Il faut créer un serveur qui renvoie le mdp précédent. Pour ce faire, on peut faire
```bash
echo "<mdp>" | nc -l -p 1234 &
```
- l : créé un serveur sur localhost
- p : spécifie le port

L'usage de `&` rend le processus asynchrone la durée d'une lecture
On peut ensuite faire `./suconnect 1234`
## Niveau 21 -> 22

Dans `/etc/cron.d`, on voit un fichier nommé `cronjob_bandit22`. En cattant son contenu, on voit
```bash
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

Pour rappel, la division est :
> minute | hour | day | month | weekday
- minute : borne des minutes (0-59)
- hour : borne des heures (0-23)
- day : borne des jours (1-31, sans contrôle du mois donc la valeur 31/02 est valide syntaxiquement)
- month : mois (1-12)
- weekday : jour de la semaine (0-6 de dimanche à samedi. Des fois le 7 est accepté comme dimanche mais pas toujours)

On obtient les divers syntaxes valables pour toutes les valeurs
> `5 * * * *` => À la minute 5
> `*/5 * * * *` => Toutes les 5 minutes
> `1-5 * * * *` => À toutes les minutes de 1 à 5
> `1,5 * * * *` => À la minute 1 et 5
> `5-10/2 * * * *` => Toutes les 2 minutes de 5 à 10
> `5-10/2,3 * * * *` => À toutes les 2e minutes de 5 à 10 **ET** à la minute 3

Au delà de ce rappel, on fait `cat /usr/bin/cronjob_bandit2.sh` et on voit le mdp.
## Niveau 22 -> 23

Rebelotte. On cat `/etc/cron.d/bandit22`, on cat le script lancé et ce script fait pleins d'opérations :
```bash
#!/bin/bash

myname=$(whoami) # renvoie bandit23 car cron exécute ceci avec le user bandit23
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1) # Le manque de "" ne gêne pas pour le echo a priori. On simule la commande :
# echo "I am user bandit23" | md5sum et on récupère la première partie avant l'espace.

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

# Il copie le contenu du fichier dans /tmp/<hash>, avec <hash> le hash produit au dessus, qu'on cat.
cat /etc/bandit_pass/$myname > /tmp/$mytarget
```
## Niveau 23 -> 24

Rebelotte, le script est :
```bash
#!/bin/bash

# shopt = shell option
# -s : active une option
# nullglob : option reliée au fait qu'aucune erreur n'est produite lors d'un parcours d'un dossier vide
shopt -s nullglob

# Dans le contexte du script, vaut bandit24
myname=$(whoami)

# Met le wd sur /var/spool/bandit24/foo
cd /var/spool/"$myname"/foo || exit
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"

# Parcourt i selon les noms de fichiers présents dans le wd
for i in *.*;
do
	# Évite . et ..
    if [ "$i" != "." ] && [ "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" "./$i")"
        
        # Exécute les fichiers de bandit23 qui sont dans le dossier
        if [ "${owner}" = "bandit23" ] && [ -f "$i" ]; then
            timeout -s 9 60 "./$i"
        fi
        
        # Supprime le fichier
        rm -rf "./$i"
    fi
done
```

Le but est d'écrire un script qui sera exécuté avec les privilèges de bandit24. Je pense faire `cat /etc/bandit_pass/bandit24`.
Je crée un dossier temp dans lequel j'ai écrit
```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/tmp.<id>/result
```
J'ai ensuite exécuté `chmod 777 script.sh` pour assurer l'exécution du script par tout le monde.
ATTENTION, il faut aussi vérifier si les autres ont accès au dossier `/tmp/tmp.<id>` !! Sinon aucune écrite ne se fera ! Donc `chmod 777 .`
Enfin, j'ai copié ce script dans `/var/spool/bandit24/foo` et j'ai attendu une minute.
## Niveau 24 -> 25

On doit faire rentrer dans le serveur toutes les lignes `<mdp> $i` avec `$i` dans `[0, 9999]`. Le serveur voit la réponse et valide le tout.

POUR RAPPEL, `echo "Hey $(seq 1 5)"` donne `Hey 1\n2\n3\n4\n5` et ne régénère pas le début du echo !

On va donc générer la passphrase dans un fichier avec un script:
```bash
for i in {0..9999}; do
	echo "<mdp> $i" >> possibilites.txt # >> pour append
done
```
Et on ouvre un serveur netcat sur le port 30002 en lui donnant le contenu souhaité. On prend aussi le soin d'exclure les messages d'erreur (qui commencent par "Wrong") : `cat possibilites.txt | nc localhost 30002 | grep -v Wrong`
## Niveau 25 -> 26

On a un fichier `bandit26.sshkey` sur le bureau. Via `scp`, on récupère le fichier :
`scp -P 2220 bandit25@bandit.labs.overthewire.org:/home/bandit25/bandit26.sshkey .`. En essayant de se connecter via `ssh -i bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220`, on voit les messages habituels mais on se fait automatiquement kick. Essayer d'appender une commande ne marche pas.

Dans `/etc/passwd`, on voit quel terminal est utilisé pour tous les utilisateurs lors de la connexion. On peut filtrer et voir que bandit26 utilise `/usr/bin/showtext`. Ce fichier contient
```bash
#!/bin/sh

export TERM=linux

exec more ~/text.txt
exit 0
```
On voit que `more` est appelé sur un texte. La commande `more` permet d'afficher le contenu d'un fichier comme pour `man` SI le texte est trop grand relatif à la taille de la fenêtre. Le cas échéant, un simple print est fait et la commande se termine. En réduisant la fenêtre, on devrait pouvoir rentrer dans ce mode avant que le programme quitte.

/!\ il faut le faire sur wsl sinon ça ne marche pas !!!

En ayant établi la connexion et en ayant réduit le terminal, on arrive sur le mode réduit de `more`. En tapant `v`, on peut aller dans vim et changer de fichier via `:e`. Comme on est sur `bandit26`, il suffit juste de faire `:e /etc/bandit\_pass/bandit26` et on obtient le mdp.

De manière générale, sur `vi` et `vim`, on peut faire `:set shell=/bin/bash` puis `:shell`, ou directement `:! /bin/bash`. Cela nous passe dans un shell bash !

Rappel : quitter `vim` c'est `:q`.
## Niveau 26 -> 27

On a un script avec bit `suid`. On fait `./bandit27-do cat /etc/bandit_pass/bandit27`
## Niveau 27 -> 28

On a besoin de faire `git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo` avec le port après le `fqdn`.
## Niveau 28 -> 29

On clone le repo comme au dessus. Les credentials ne sont pas disponibles, mais on peut penser qu'il y a des commits précédents. On fait `git log` et on obtient le hash du commit précédent. On voit un commit qui parle d'une leak. De ce hash on peut faire `git checkout a1487fd098591dfa210ede70ba60f7093f47d20d` histoire d'y retourner, et voir qu'il y a le mdp dedans.
## Niveau 29 -> 30

Rebelotte. Ici on vérifie les branches via `git branch -a`.
On voit que le message est "aucun mdp en prod", donc on peut aller checker la branche dev via `git checkout origin/dev`.
En cattant le fichier, on gagne.
## Niveau 30 -> 31

J'ai essayé de lister les remotes avec `git remote -v`, rien à signaler. `git branch -a` ne donne rien, de même pour `git log`.
Une notion pas utilisée à la fac mais beaucoup utilisée en prod, les **tags** !
On liste les tags via `git tag` et on consulte la valeur d'un tag via `git show <tag>`.
`git tag` donne "secret", et `git show secret` donne le mdp.
## Niveau 31 -> 32

Le but est de push un fichier nommé `key.txt`. `key.txt` est dans le `.gitignore`.
On fait
```bash
echo "May I come in?" > key.txt
git add -f key.txt #-f pour forcer à ne pas écouter le .gitignore
git commit -m "h"
git push
```
## Niveau 32 -> 33

On peut utiliser `$0` qui est une variable d'environnement. Les variables environnement sont écrites en majuscules. De ce fait, `$0` va pointer sur la shell en question car c'est le programme invoqué, et nous laisser y avoir accès. On s'attend à ce que ça répète le problème d'uppercase, mais la relancer le supprime. Cela peut s'expliquer par l'implémentation du programme, qui ne lance l'uppercase qu'au démarrage initial !
On est dans une shell limitée. Faire `ls -l` montre `uppershell` avec bit `suid` qui appartient à `bandit33`. On peut donc essayer `whoami`/`id` et constater que la shell pense qu'on est bandit33. On peut récupérer le mdp via `cat /etc/bandit_pass/bandit33`.
## Niveau 33 -> 34

Terminado !!!