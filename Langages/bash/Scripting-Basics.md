# Scripting — condensé essentiel

## Shebang & permissions
- Shebang en 1ʳᵉ ligne : `#!chemin/interpréteur` (ex: `#!/bin/bash`, `#!/usr/bin/python3`).  
- Vérifier chemin : `which bash` / `which python3`.  
- Rendre exécutable : `chmod +x monscript.sh`.  
- En-tête conseillé (meta) : `# FILE: ...`, `# AUTHOR: ...`, `# CREATED: ...`.

## Debug / logs rapides
- Trace complète : lancer `bash -x monscript.sh`.  
- Debug manuel : `echo` pour variables ; `read` pour pauses (`read -p "msg" var`).  
- Codes de sortie : `exit N` (0 = succès). Vérifier `$?`.

## Variables & types
- Locales → `var="val"` : valables uniquement dans le Shell courant
- D’environnement → exportées vers les sous-shells
	- Depuis une locale : `logdir="/var/log"` puis `export logdir`
	- Directement : `export logdir="/var/log"`
- Constantes (lecture seule) → créées avec `declare -r CONST="val"`
- Réservées → gérées par le Shell, servent à des infos systèmes
	`$$` → PID du processus actuel
	`$?` → Code retour de la dernière commande (0 si OK)
	`$!` → PID du dernier processus lancé en arrière-plan
	`$#` → Nombre de paramètres passés au script
	`$1` à `$9` → Les paramètres eux-mêmes
	`$0` → Nom du script exécuté
	`$@` ou `$*` → Tous les paramètres passés (enchaînés ou séparés)
	`set`→ affiche toutes les variables du Shell
	`set "$LOGNAME" $(uname -n)` → $1=$LOGNAME, $2=$(uname -n)
- Supprimer : `unset var`.

## Lecture utilisateur
- Simple : `read -p "Entrez: " nom`.  
- Plusieurs : `read -p "Nom prénom: " nom prenom tampon` puis `unset tampon`.  
- Lire dans boucle fichier : `while read a b reste; do ...; done < fichier` (préférer `while read` à `for` pour lignes).

## Contrôle d’exécution (chaînage)
- Inconditionnel : `cmd1 ; cmd2`.  
- Si succès : `cmd1 && cmd2`.  
- Si échec : `cmd1 || cmd2`.  
- Subshell (effets non persistants) : `( cmd1; cmd2 )`.  
- Bloc dans shell courant (effets persistants) : `{ cmd1; cmd2; }` (espaces obligatoires, `;` après la dernière).

## Tests & conditions
- Test POSIX : `[ ... ]` ou `test ...`.  
- Bash étendu : `[[ ... ]]`.  
- Arithmétique : `(( ... ))`.  
- Exemples entiers : `-eq, -ne, -gt, -lt`, `-ge`, `-le`  
- Chaînes : `=`, `!=`, `-z` (vide), `-n` (non vide).  
- Fichiers : `-d` répertoire, `-f` fichier, `-r` readable, `-w` writable, `-x` executable, `-s` size.  
- If compact : `if condition ; then action ; else autre ; fi`.  
- If standard : 
	```bash
	if [[ "$LOGNAME" = root ]] ; then  
		echo "Ne pas se connecter en root"  
	else  
		echo "Bienvenue sur la machine $HOSTNAME"  
	fi
	```
- Case : 
	```bash
	case $var in # ouverture du bloc
		valeur1) # test d’un motif/valeur
			action ;; # instructions exécutées si correspondance - ;; obligatoire en fin de bloc action
		valeur2) etc. # autre cas
		*) # valeur par défaut (fallback, « sinon »)
	esac # fin du bloc case
	```
	- `p)` → mot-clé exact 
	- `[0-9])` → motif : n'importe quel chiffre
	- `a|b)` → OU logique entre plusieurs valeurs

## Glob & motifs
- Basique : 
- `*` ➜ 0 à n caractères
- `?` ➜ 1 caractère
- `[abc]` ➜ un caractère parmi
- `[^abc]` ➜ un caractère autre que
- extglob (activer si besoin) : `shopt -s extglob`
	- `?(motif) ➜ 0 ou 1 fois
	- `*(motif)` ➜ 0 à n fois
	- `+(motif)` ➜ 1 à n fois
	- `@(motif)` ➜ exactement 1 fois
	- `!(motif)` ➜ tout sauf ce motif

## Boucles
- For : `for v in liste; do ...; done`. 
	- "val1" "val2" ou $(commande) ou encore {1..10} ou même un chemin de fichier plat à parcourir.
- While : `while CONDITION; do ...; done`.  (Tant que)
- Until : `until CONDITION; do ...; done`.  (Jusqu'à ce que)
- Boucle infinie : `while :; do ...; done` ou `while true; do ...; done`.  
- Compteurs / arithmétique : `(( i++ ))`, `let i=i+1`, `(( i < 5 ))`. Historique : `expr` (Espace autour de l'opération)

## Pipes, redirections & stdin
- Pipe crée un sous-shell (variables internes perdues hors pipe) ➜ `Command | while ....` 
- Lire stdin coexistence (ex: interaction dans traitement) : `read -p "Msg" var </dev/tty`.  
- Substitution de process : `while read ...; do ...; done < <(commande)` (évite pipe subshell).  
- Rediriger erreurs : `2>/dev/null` ; append : `>> fichier`.

## Stdout
- Echo avec codes ANSI : `echo -e "voici \033[1;32mvert\033[0m"` (`\033[` ou `\e[` ; terminer par `\033[0m`).
	- Attributs : 1 = gras, 4 = souligné, 7 = inversé
	- Texte : 30 à 37 (noir à gris clair)
	- Fond : 40 à 47
	- m : fin de déclaration
- Options `echo -e "\n\ttexte"`
	- `\n` → saut de ligne
	- `\t` → tabulation
	- `\c` → supprime le retour à la ligne final (reste sur la même ligne)

## Fonctions
- Déclaration :
	```bash
  nom_fonction() {
	commandes
  }
	```
- Appel : `nom_fonction arg1 arg2`.  
- Arguments internes : `$1`, `$@` etc.  
- Variables globales par défaut (modifier globales depuis fonctions).  
- Sourcing pour réutiliser fonctions : `source ~/mesfonctions` ou `. ~/mesfonctions`.

## Lecture fichiers & robustesse
- Préférer `while read` pour lignes complètes.  
- Gérer IFS si champs séparateurs différents : `OLDIFS="$IFS"; IFS=":"; ...; IFS="$OLDIFS"`.  
- Vérifier existence avant actions : `[[ -f "$f" ]] || { echo "no file"; exit 1; }`.

## Bonnes pratiques & checklist rapide
- Toujours shebang + chmod + tests de syntaxe.  
- Utiliser `set -euo pipefail` dans scripts robustes (optionnel, connaître effets).  
- Valider entrées utilisateurs / existence fichiers.  
- Éviter commandes dans des blocs non testés ; tester avec `bash -n` (syntax-only) ou `shellcheck`.  
- Versionner scripts / documenter en tête.

### Exemples ultra-rapides
- Shebang : `#!/bin/bash`  
- Rendre exécutable : `chmod +x script.sh`  
- Debug : `bash -x script.sh`  
- If compact : `if [[ -f "$1" ]]; then echo "OK"; else echo "KO"; fi`  
- While read safe : `while IFS= read -r line; do echo "$line"; done < fichier.txt`  
- Fonction : `greet(){ echo "Hello $1"; } ; greet Marc`


