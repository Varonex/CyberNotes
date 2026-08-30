# Notation de base
- `&` : asynchrone
- `;` : fin de définition OU lancement séquentiel
- `-` : STDIN-as-file.
- `0` : STDIN.
- `1` : STDOUT.
- `2` : STDERR.
- `>` : Redirection.
- `1>` : Redirection STDOUT.
- `2>` : Redirection STDERR.
- `>&1` : Redirection STDERR dans STDOUT.
- `>&2` : Redirection STDOUT dans STDERR.
- `>>` : Redirection sans écrasement préalable du flux.
- `<` : Apport d'un fichier.
- `<< EOL ... EOL` : Heredoc.
- `<< 'EOL' ... EOL` : Heredoc sans interprétation des variables.
- `<<<` : Herestring. Équivalent à `echo ... |`.
- `--` : Fin interprétation des arguments.
# Manipulation d'un flux
## Lecture du flux
| Commande | Description                                                        | Options                                                                         | Descriptions Options                                                                                                                                                             |
| -------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cat`    | Montrer tout le contenu<br>d'un flux.                              | `-A`<br>`-e`<br>`-E`<br>`-t`<br>`-T`<br>`-v`                                    | -`-vET`<br>-`-vE`<br>-Met $ à chaque fin de ligne.<br>-`-vT`<br>-Affiche les tabs.<br>-Affiche les chars spéciaux.                                                               |
| `head`   | Affiche les premières lignes<br>d'un flux.                         | `-n [n]`                                                                        | -Affiche les n premières lignes.                                                                                                                                                 |
| `less`   | Montre le contenu du flux<br>en pages. Plus élaboré<br>que `more`. | `esp`<br>`down`<br>`b`<br>`up`<br><br>`/[q]`<br>`?[q]`<br>`n`<br>`N`<br><br>`q` | -Page d'après.<br>-Ligne d'après.<br>-Page d'avant.<br>-Ligne d'avant.<br><br>-Recherche avant.<br>-Recherche arrière.<br>-Prochain terme.<br>-Précédent terme.<br><br>-Quitter. |
| `more`   | Montre le contenu du flux<br>en pages.                             |                                                                                 |                                                                                                                                                                                  |
| `tail`   | Affiche les dernières lignes<br>d'un flux.                         | `-n [n]`                                                                        | -Affiche les n dernières lignes.                                                                                                                                                 |
## Traitement du flux
| Commande       | Description                                                                                       | Options                                                  | Descriptions Options                                                                                                                                                                                    |
| -------------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `base64`       | Manipule la base 64.                                                                              | `-d`<br>`-i`                                             | -Décode.<br>-Ignore le non alphabétique.                                                                                                                                                                |
| `cut`          | Coupe les sections d'un flux.                                                                     | `-d [d]`<br>`-f [i]`                                     | -Délimiteur de split.<br>- Indexe du split.                                                                                                                                                             |
| `diff [o]`     | Différences entre deux flux.                                                                      | `-a`<br>`-q`                                             | -Traiter les fichiers comme du texte.<br>-Bref.                                                                                                                                                         |
| `echo [s]`     | Ressort le flux.                                                                                  | `-e`<br>`-E`                                             | -Pretty display.<br>-Désactive l'interprétation des<br>caractères spéciaux.                                                                                                                             |
| `egrep [p]`    | Alias à `grep -E`                                                                                 |                                                          |                                                                                                                                                                                                         |
| `grep [p]`     | Lit chaque ligne du flux <br>et  matche selon le<br>string p.                                     | `-c`<br>`-E`<br>`-i`<br>`-m [n]`<br>`-n`<br>`-s`<br>`-v` | -Compte le nb de matches.<br>-Active le regex.<br>-Insensible à la casse.<br>-Nb max de résultats.<br>-Numéro de ligne.<br>-Supprime les erreurs.<br>-Fait l'inverse.                                   |
| `sed [p]`      | Filtre et transforme le flux.<br>`s/find/replace/g`                                               | `-e`<br>`-i`                                             | -Enchaîne plusieurs expressions.<br>-Modifie directement le flux.                                                                                                                                       |
| `sort`         | Trie le flux.                                                                                     | `-c`<br>`-f`<br>`-h`<br>`-M`<br>`-n`<br>`-r`<br>`-u`     | -Vérifie si c'est trié au lieu de trier.<br>-Insensible à la casse.<br>-Tri naturel avec notation (k, G, ...)<br>-Tri comme les mois.<br>-Tri numérique.<br>-Reverse de tri.<br>-Unicité (voir `uniq`). |
| `strings [s]`  | Ne conserve que les<br>caractères lisibles.                                                       | `-a`                                                     | -Scan de tout le fichier.                                                                                                                                                                               |
| `tr [a] [[b]]` | Translation de l'alphabet `a`<br>dans l'alphabet `b`.<br>Peut s'utiliser sans `b`<br>ssi options. | `-d`<br>`-s`<br>`-t`                                     | -Supprime.<br>-Réduit les répétitions.<br>-Tronque a pour que ça matche la longueur de b.                                                                                                               |
| `uniq`         | Supprime les doublons<br>adjacents d'un flux.                                                     | `-c`<br>`-d`<br>`-f [n]`<br>`-i`<br>`-u`                 | -Compte les occurrences.<br>-Fait l'inverse.<br>-Skip les n premières lignes.<br>-Ignore la casse.<br>-Supprime les lignes avec duplicatas.                                                             |
| `wc`           | Comptage de symboles<br>d'un flux.                                                                | `-c`<br>`-m`<br>`-l`<br>`-w`                             | -Octets.<br>-Caractères.<br>-Lignes.<br>-Mots.                                                                                                                                                          |
## Awk

Traitement de plusieurs fichiers et division de chaque ligne en colonnes selon un séparateur.
- `-F` : changer le séparateur.
- `-f` : lire le code awk depuis un fichier.
- `-v` : injecter une variable (`-v var=val)`.

```bash
awk -F ";" '
	# Commentaire.
	
	# Création de fonctions.
	function carre(x) { return x*x }
	
	# Code lu dès le tout début.
	BEGIN
	{
		# Aucun accès aux variables d\'environnement relatives à la lecture du fichier.
		
		# Variable qui gère le séparateur mis via -F. Overridable ici.
		print FS
		
		# Variable de sortie via print. Par défaut, c\'est l\'espace.
		print OFS
	}
	
	# Code lu pour chaque ligne.
	{
		# $0 = toute la ligne.
		print $0
		
		# Affiche $0.
		print
		
		# $i = ième elm. On peut indexer via littéralement $i (boucle pour etc...)
		print $1
		
		# Donne le nombre de champs sur la ligne, et donne la valeur de la dernière ligne.
		print NF, $NF
		
		# Donne le numéro de la ligne globale, et le numéro de la ligne par fichier donné.
		print NR, FNR
	}
	
	# Code lu si le premier élément est "Hi". Bloc conditionnel.
	$1 == "Hi"
	{
		# Boucle pour.
		for (i = 1; i <= NF; i++)
		{
			# $i permet d\'accéder au champ i.
			print "Colonne " i " : " $i
		}
		
		# Boucle while.
		i = 1
		while (i <= NF)
		{
			print "Colonne " i " : " $i
			i++
			
			if (i == 0)
			{
				break
			}
		}
		
		i = 1
		do
		{
			print "Colonne " i " : " $i
			i++
			continue
		}
		while (i <= NF)
	}
	
	# Code lu si le premier élément suit le regex. Bloc conditionnel.
	$1 ~ /aeiouy/
	{
		# Skip la ligne.
		next
	}
	
	# Bloc lu à la fin.
	END
	{
		# Tableaux.
		t[0] = 0
		
		# Dico.
		t["bonsoir"] = "hey"
		
		# Test de valeur.
		if ("bonsoir" in t)
		{
			print "Trouvé"
		}
		
		# Boucle.
		for (cle in t)
		{...}
		
		# Suppression de la clé.
		delete t[0]
		
		# Supprime tout le tableau.
		delete t
		
		# Multidimension.
		SUBSEP = ":" # préfixe de séparation.
		
		i = 1
		j = 3
		t[i, j] = "a"
		
		# L\'accès est équivalent.
		print t[i, j], t[i SUBSEP j]
		
		# Call de fonction.
		print carre(2)
	}
' ...fichiers
```
> Attention, indexes de strings à 1.

- `length(x)` : Longueur d'un string.
- `substr(x, pos, len)` : Pioche dans x de pos à pos + len. Renvoie le nombre d'éléments placés dans `out`.
- `tolower(x)`, `toupper(x)` : Gère la capitalisation.
- `split(x, out, sep)` : Retourne dans `out` le contenu de `x` splitté par `sep`.
# Système de fichiers
| Commande             | Description                                                            | Options                                                                                                                                                                                                                                | Descriptions Options                                                                                                                                                                                                                                                                                                                                                             |
| -------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `bzip2`              | BZip2 pour la<br>compression.                                          | `-d`<br>`-v`                                                                                                                                                                                                                           | -`bunzip2`<br>-Verbosité.                                                                                                                                                                                                                                                                                                                                                        |
| `bunzip2`            | `bzip2 -d`                                                             | `-v`                                                                                                                                                                                                                                   | -Verbosité.                                                                                                                                                                                                                                                                                                                                                                      |
| `cd [p]`             | Change dir.                                                            |                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                  |
| `chmod [b]`          | Change les permissions.                                                | `u`<br>`g`<br>`o`<br>`a`<br><br>`..+/-r`<br>`..+/-w`<br>`..+/-x`<br>`..+/-s`<br><br>`-R`                                                                                                                                               | -User.<br>-Group.<br>-Other.<br>-All.<br><br>-Lecture.<br>-Écriture.<br>-Exécution.<br>-SUID.<br><br>-Récursif.                                                                                                                                                                                                                                                                  |
| `chown [u]`<br>`[b]` | Change le propriétaire.                                                | TODO                                                                                                                                                                                                                                   | TODO                                                                                                                                                                                                                                                                                                                                                                             |
| `cp [a] [b]`         | Copie a sur b.                                                         |                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                  |
| `df`                 | Donne des informations<br>d'espace libre du disque.                    | `-a`<br>`-h`<br>`-i`<br>`-l`                                                                                                                                                                                                           | -Prend tout (all).<br>-Lisible par humains.<br>-Prend les inodes.<br>-Uniquement le filesystem local.                                                                                                                                                                                                                                                                            |
| `du [f]`             | Donne des informations<br>de taille sur le disque.                     | `-a`<br>`-b`<br>`-c`<br>`-h`                                                                                                                                                                                                           | -Prend tout (all).<br>-Octets.<br>-Total.<br>-Lisible par humains.                                                                                                                                                                                                                                                                                                               |
| `file [f]`           | Donne le type de <br>contenu d'un fichier.                             |                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                  |
| `find [p]`           | Recherche dans le<br>système de fichiers.                              | `-and`<br>`-executable`<br>`-group [g]`<br>`-gid [g]`<br>`-iname [n]`<br>`-iregex [p]`<br>`-name [n]`<br>`-not`\|`!`<br>`-perm`<br>`-readable`<br>`-regex`<br>`-size [n]`<br>`-type [fd]`<br>`-user [u]`<br>`-uid [u]`<br>`-writeable` | -Et logique.<br>-Si bit e.<br>-Par nom de groupe.<br>-Par ID de groupe.<br>-Par nom sans casse.<br>-Par regex sans casse.<br>-Par nom.<br>-Non logique.<br>-Permission par bit (`-perm -4000`<br>pour approx, `-perm /4000` pour<br>exact).<br>-Si bit r.<br>-Par regex.<br>-Taille (bcwkMG).<br>-Filtre sur le type.<br>-Par utilisateur.<br>-Par ID utilisateur.<br>-Si bit w. |
| `gzip`               | GNU Zip pour la<br>compression.                                        | `-d`<br>`-r`<br>`-v`                                                                                                                                                                                                                   | -`gunzip`<br>-Récursif.<br>-Verbosité.                                                                                                                                                                                                                                                                                                                                           |
| `gunzip`             | `gzip -d`                                                              | `-r`<br>`-v`                                                                                                                                                                                                                           | -Récursif.<br>-Verbosité.                                                                                                                                                                                                                                                                                                                                                        |
| `ls [p]`             | Lister le contenu.                                                     | `-a`<br>`-A`<br>`-l`                                                                                                                                                                                                                   | -All (fichiers cachés inclus).<br>-`-a` sans `.` et `..`.<br>-Listing détaillé.                                                                                                                                                                                                                                                                                                  |
| `mkdir [d]`          | Crée un dossier vide.                                                  | `-m [b]`<br>`-p`                                                                                                                                                                                                                       | -Fixe les bits (chmod).<br>-Pas err si le(s) dossier(s)<br>existe(nt).                                                                                                                                                                                                                                                                                                           |
| `mv [a] [b]`         | Bouge/Renomme a sur b.                                                 |                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                  |
| `pwd`                | Donne le working dir.                                                  |                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                  |
| `rm [f]`             | Supprime du disque.                                                    | `-d`<br>`-f`<br>`-r`                                                                                                                                                                                                                   | -Supprime un dossier vide.<br>-Force.<br>-Récursif.                                                                                                                                                                                                                                                                                                                              |
| `rmdir [d]`          | Supprime un dossier vide.                                              |                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                  |
| `tar`                | Archives tarball. Peut<br>prendre la compression<br>via les flags.     | `-c`<br>`-j`<br>`-v`<br>`-x`<br>`-z`                                                                                                                                                                                                   | -Création.<br>-Usage de bzip2 pour l'algo de<br>compression.<br>-Verbosité.<br>-Extraction.<br>-Usage de gzip pour l'algo de<br>compression.                                                                                                                                                                                                                                     |
| `touch [f]`          | Crée un fichier vide, ou<br>modifie sa dernière<br>date d'utilisation. | `-a`<br>`-m`<br>`-t`                                                                                                                                                                                                                   | -Change seulement la date<br>du dernier accès.<br>-Change seulement la date<br>modif.<br>-Change les deux<br>(`YYYYMMDDhhmm.ss`)                                                                                                                                                                                                                                                 |
| `xxd`                | Lecteur hex.                                                           |                                                                                                                                                                                                                                        |                                                                                                                                                                                                                                                                                                                                                                                  |
# Gestion des credentials
## Utilisateurs
| Commande      | Description                                | Options                                                                                                                                                   | Descriptions Options                                                                                                                                                                                                                                                                                                                                                              |
| ------------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `adduser [u]` | \[DEB] Ajout interactif d'un<br>user.      |                                                                                                                                                           | Interactif.                                                                                                                                                                                                                                                                                                                                                                       |
| `chage [u]`   | Gère l'expiration des mdp pour<br>un user. | `-d`<br>`-E`<br>`-l`<br>`-m`<br>`-M`<br>`-W`                                                                                                              | -Fixe la dernière date de<br>modification.<br>-Fixe la date d'expiration.<br>-Liste les infos par<br>utilisateur.<br>-nb jour minimal.<br>-nb jour maximal.<br>-nb jours pour émettre un warning.                                                                                                                                                                                 |
| `passwd [u]`  | Change le mdp d'un<br>user existant.       | `-d`<br>`-e [d]`<br>`-i [d]`<br>`-l`<br>`-u`                                                                                                              | -Supprime le mdp.<br>-Expiration.<br>-Inactivité du mdp.<br>-Lock le compte.<br>-Unlock le compte.                                                                                                                                                                                                                                                                                |
| `useradd [u]` | Ajoute un user.                            | `-d [p]`<br>`-e [d]`<br>`-f [d]`<br>`-g [g]`<br>`-G [g]`<br>`-m`<br>`-M`<br>`-p [p]`<br>`-s [s]`<br>`-u [i]`<br>`-U`                                      | -Précise le chemin du homedir.<br>-Expiration.<br>-Inactivité du mdp.<br>-GID principal.<br>-GID additionnels.<br>-Création du homedir.<br>-Ne créé pas de homedir.<br>-Mdp assigné.<br>-Shell.<br>-UID du user.<br>-Créé groupe avec le même nom<br>que le user.                                                                                                                 |
| `userdel [u]` | Supprime un user existant.                 | `-f`<br>`-r`                                                                                                                                              | -Force.<br>-Supprime homedir et autre.                                                                                                                                                                                                                                                                                                                                            |
| `usermod [u]` | Modifie un user existant.                  | `-a(G)`<br>`-d [p]`<br>`-e [d]`<br>`-f [d]`<br>`-g [g]`<br>`-G [g]`<br>`-l [n]`<br>`-L`<br>`-m(d)`<br>`-p [p]`<br>`-r(G)`<br>`-s [s]`<br>`-u [i]`<br>`-U` | -Append le groupe de l'option G<br>sans override.<br>-Change le homedir.<br>-Expiration.<br>-Inactivité du mdp.<br>-GID principal.<br>-GID additionnels.<br>-Renomme le user.<br>-Lock.<br>-Déplace le contenu du homedir<br>(avec `-d`).<br>-Mdp assigné.<br>-Supprime le groupe de l'option G<br>du groupe sans enlever des autres.<br>-Shell.<br>-UID du user.<br>-Unlock.<br> |
| `whoami`      | `id -un`                                   |                                                                                                                                                           |                                                                                                                                                                                                                                                                                                                                                                                   |
## Groupes
| Commande       | Description                                                                                                        | Options                                                      | Descriptions Options                                                                                                                                          |
| -------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `addgroup [g]` | \[DEB] Ajout interactif d'un<br>groupe.                                                                            |                                                              | Interactif.                                                                                                                                                   |
| `gpasswd [g]`  | Gère la liste des utilisateurs d'un<br>groupe. Permet aussi de redéfinir<br>le mdp du groupe dont on est<br>admin. | `-a [u]`<br>`-A [u]`<br>`-d [u]`<br>`-M [u]`<br>`-r`<br>`-R` | -Ajoute le user.<br>-Ajoute user comme admin.<br>-Supprime le user.<br>-Définit la liste des users.<br>-Supprime le mdp.<br>-Restreint l'accès au groupe.<br> |
| `groupadd [g]` | Ajoute un groupe.                                                                                                  | `-f`<br>`-g [i]`<br>`-p [p]`<br>`-U [u]`                     | -Force.<br>-GID du groupe.<br>-Mdp assigné.<br>-Users du groupe.                                                                                              |
| `groupdel [g]` | Supprime un groupe existant.                                                                                       | `-f`                                                         | -Force.                                                                                                                                                       |
| `groupmod [g]` | Modifie un groupe existant.                                                                                        | `-a(U)`<br>`-g [i]`<br>`-n [n]`<br>`-U [u]`                  | -Append le user de l'option U<br>sans override.<br>-Renomme le groupe.<br>-GID du groupe.<br>-Users du groupe.                                                |
| `groups [u]`   | Liste les groupes d'un utilisateur.                                                                                |                                                              |                                                                                                                                                               |
## Autres manipulations

| Commande     | Description                                                                 | Options                                                                                      | Descriptions Options                                                                                                                                                                                                 |
| ------------ | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `getent [f]` | Voir les entrées de fichiers admins.<br>`passwd`, `shadow`, `services`, ... |                                                                                              |                                                                                                                                                                                                                      |
| `id [u]`     | Donne les informations sur un user.                                         | `-g`<br>`-G`<br>`-n(.)`<br>`-r`<br>`-u`                                                      | -Groupe principal.<br>-Tous les groupes.<br>-Nom associé à l'option désirée.<br>-Donne l'ID réel et non pas<br>l'effectif.<br>-User.                                                                                 |
| `su`         | Change d'utilisateur.                                                       | `-c [c]`<br>`-g [i]`<br>`-G [i]`<br>`-, -l`<br>`-p`<br>`-w [v]`                              | -Commande unitaire à exécuter.<br>-Groupe primaire à cibler.<br>-Groupe secondaire à cibler.<br>-Login.<br>-Préserve les variables<br>d'envrionnement.<br>-Préserve la liste des variables<br>d'environnement.       |
| `sudo`       | Obtient les privilèges admin le<br>temps d'exécution d'une<br>commande.     | `-A`<br>`-b`<br>`-e`<br>`-E`<br>`-g [i]`<br>`-H`<br>`-l`<br>`-p [p]`<br>`-s [s]`<br>`-u [u]` | -Prompt le mdp.<br>-Envoie le proc. en tâche de<br>fond.<br>-Mode édition de fichier.<br>-Préserve l'environnement.<br>-GID/Groupname.<br>-Homedir.<br>-Liste les sudoers.<br>-Mdp.<br>-Shell.<br>-UID/Username.<br> |
# Misc
type, which, whereis, ...
history \[-c], `!`, clear, ...