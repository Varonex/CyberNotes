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