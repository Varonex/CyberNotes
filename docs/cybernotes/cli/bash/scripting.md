```bash
#!/bin/bash
# Shebang pour préciser la shell.

#
# VARIABLES
#
monInt=1
maString="bonsoir"
maSlice=${maString: 0:3} # Affiche "bon". Fait une slice du string en mode : dep:length. Attention à l'espace ! Peut se faire de l'arrière avec les nombres négatifs. ": -1:3".
maString+=" bonjour" # Concatène.
monInterpolation="$PATH"
monLitteral='$PATH' # N'interpole pas.

#
# ARRAYS
#
monArr=("a" "b" "c")

elm0=${monArr[0]} # Élément 0.
elm0=$monArr # Aussi l'élément 0.
toutLeTab=${monArr[@]} # Tous les éléments dans une chaîne itérable.

size=${#monArr[@]} # Taille du tableau.
monArr+=("d") # Append de l'array dans l'array originelle.
slice="${monArr[@]: 1:2}" # Slicing, des éléments d'indice [1, 1+2[/

unset "monArr[0]" # Prend le nom ici. Détruit la première case sans faire de shift.
unset "monArr" # Détruit le tableau.

#
# DICTIONNAIRES
#

declare -A monDico
monDico["bonsoir"]="a"
maVal=${monDico["bonsoir"]}
cles=${!monDico[@]}
valeurs=${monDico[@]}
taille=${#monDico[@]}

#
# ARGUMENTS
#
arg0=$0 # Unsafe (null).
arg1="$1" # Safe.
arg10="${10}" # {} pour noms complexes.
nbArg="$#" # NB arguments.
allArgs="$*" # Prend tous les args dans une même chaîne. Pas d'itération dans une boucle.
allArgsSep="$@" # Fait une liste de tous les arguments. Une boucle ferait une itération sur chaque élément.

# Décalage des paramètres ($1 est écrasé par $2 etc... $0 reste intact).
shift

#
# EXPANSIONS DE VARIABLES
#
var=""
echo "${var:-default}" # Affiche default. Si var est vide, lit la 2e partie.
echo "${var:=default}" # Assigne default si var est vide.


#
# FONCTIONS
#
maFonction() {
	a=1 # Assignation globale.
	local b=2 # Assignation locale. Comme en Lua !
	
	echo "$1" # Argument $1 local à la fonction. Affiche le premier arg.
	# Tout marche de la même manière que dans le globalscope.
	
	# Les returns se font via le stdout/stderr, donc utiliser echo/printf.
	return 0 # Code de retour annexe (0-255). N'est PAS la valeur récupérée par le pipe.
}

#
# APPEL DE FONCTIONS
#
maFonction 4 # Affiche 4 car echo de $1.
monResultat=$(maFonction 5) # Stocke 5.
monResultat2=`maFonction 5` # Même chose qu'au dessus. Ancienne syntaxe.
dernierCode="$?" # Récupère le dernier code. Comme maFonction est la dernière commande appelée, renvoie 0.

#
# IF
#
if [ 1 ]; then
	echo "Code"

elif [ 2 ]; then

else

fi

# Vieux comparateur, rustique.
if [ -z "$0" -a -f "$1" ]; then
fi

[ -f -z -a -o ! -lt -gt -eq -ne ... ]

# Plus moderne. Les comparaisons sont axées string. Donc "2" > "10" dans l'ordre lexicographique. On peut toujours utiliser -eq, -lt, -le, ...
if [[ "$0" == "" && -f "$1" || -f *.txt ]]; then
fi

[[ -f ... && || == != >= <= > < * =~ ]]

# Axé uniquement arithmétique. Supporte tout comme en C.
if ((i == 21)); then
fi

((i ++ -- / * + - >= <= > < != == && || & | > < \<< >>))

#
# WHILE
#
i=0
while ((i < 3)); do
	echo "$i"
	((i++))
done

# Lecture ligne à ligne. IFS met le field separator sur " ".
while IFS=" " read -r line; do
	echo "Ligne : $line"
	
	if [[ "$line" == "" ]]; then
		continue
	fi
	
	# Ou plus simplement
	[[ "$line" == "" ]] && continue
done

# Feed du contenu dans le while.
while IFS=" " read -r line; do
	echo "Ligne : $line"
done <<< "Je peux feed ici des choses."

#
# UNTIL
#
until ((i >= 5)); do
	echo "$i"
	((i++))
done

#
# FOR
#
for i in {1..10}; do
done

# C-style.
for ((i=0; i<5; i++)); do
done

for item in *; do
done

#
# CASE
#

val="hi"
case "$val" in
	a)
		# code
		;;
	
	b|c)
		# code
		;;
	
	*)
		echo "Aucune valeur, retour au défaut"
		;;
esac

#
# RANGES
#
{1..10} # 1 à 10
{01..010} # 01 à 010 (padding)
{1..10..2} # 1 à 10 par pas de 2
{10..1} # 10 à 1
{a..e} # a à e

#
# ASSIGNATION TEMPORAIRE
#

# Si une commande intervient à la fin d'une assignation de variables, alors ces variables assignées sont temporaires et uniquement là pour la ligne.
IFS=" " read -r l

k=1 sh -c 'echo $k' # 1. Si on fait direct echo, la variable ne sera pas initialisée à ce moment là.
echo "$k" # vide.

a=1 b=2 c=3 echo "$a $b $c" # On peut chaîner.

#
# BLOC
#

# Bloc de code comme dans n'importe quel langage. Capture et overwrite. Les variables déclarées ne sont pas propres au bloc.
x=3
cd /
{
	cd /home
	x=1
	z=3
	
	pwd # Affiche /home.
	echo "$x $z" # Affiche "1 3".
}
pwd # Affiche /home.
echo "$x $z" # Affiche "1 3". Z existe.

#
# BLOC ARITHMETIQUE/LEGA-C.
#

# Reprend les principes des blocs, mais utilise les choses arithmétiques faisables.
res=$x
((
	res = res & x
	res = res + 1
	res++, res-- # Oneline avec ,
	
	res = (x > 0) ? res : x # Ternaires
))
echo "$res"

#
# SOUS-SHELL
#

# Invoque un shell imbriqué qui permet de redémarrer avec un environnement dédié.
# Capture les variables, mais ne peut pas outsourcer une variable.
# Les syntaxes $() et `` sont des sous-shell qui renvoient le résultat de stdout.
cd /
(
	# Sous-shell.
	cd /home
	x=2
	y=1
	
	pwd # Affiche /home.
	echo "$x $y" # Affiche "2 1".
)
pwd # Affiche /.
echo "$x $y" # Affiche "1  ". Y est vide.

#
# VARIABLES SPECIALES
#

IFS # Internal field separator.
$$ # PID du script actuel.
$! # PID du dernier processus lancé en arrière plan (avec &).
$- # Options actives de la shell.
$_ # Dernier arg de la commande précédente.
```
