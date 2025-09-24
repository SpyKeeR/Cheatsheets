# 🐚 Bash Scripting — Aide-mémoire

## 🚀 Démarrage Script

### Shebang & Permissions
```bash
#!/bin/bash                    # Shebang obligatoire (1ère ligne)
#!/usr/bin/env bash           # Version portable (trouve bash dans PATH)

# En-tête recommandé
# FILE: monscript.sh
# AUTHOR: John Doe
# CREATED: 2024-01-15
# PURPOSE: Description du script
```

```bash
# Vérifier chemin interpréteur
which bash                    # Localiser bash
which python3                 # Pour autres langages

# Rendre exécutable
chmod +x monscript.sh
chmod 755 monscript.sh        # Permissions explicites
```

### Options Robustesse
```bash
set -e                        # Arrêt si commande échoue
set -u                        # Arrêt si variable non définie
set -o pipefail              # Arrêt si erreur dans pipe
set -euo pipefail            # Combinaison recommandée (mode strict)

# Alternative par ligne
#!/bin/bash -euo pipefail
```

## 🔧 Variables & Types

### Déclaration & Portée
| Type | Syntaxe | Portée |
|------|---------|--------|
| **Locale** | `var="valeur"` | Shell courant uniquement |
| **Environnement** | `export var="valeur"` | Shell + sous-processus |
| **Constante** | `declare -r CONST="val"` | Lecture seule |
| **Tableau** | `arr=("a" "b" "c")` | Éléments multiples |

```bash
# Bonnes pratiques
local var="valeur"            # Dans fonctions (portée locale)
readonly CONST="valeur"       # Alternative declare -r
unset var                     # Supprimer variable

# Arrays
arr=("élément1" "élément2")
echo "${arr[0]}"              # Premier élément
echo "${arr[@]}"              # Tous éléments
echo "${#arr[@]}"             # Nombre d'éléments
```

### Variables Spéciales
| Variable | Description | Exemple |
|----------|-------------|---------|
| `$$` | PID processus courant | `echo $$` |
| `$?` | Code retour dernière commande | `ls; echo $?` |
| `$!` | PID dernier processus arrière-plan | `sleep 10 & echo $!` |
| `$0` | Nom du script | `basename $0` |
| `$1..$9` | Arguments positionnels | `echo "Premier: $1"` |
| `$#` | Nombre d'arguments | `if [[ $# -lt 1 ]]` |
| `$@` | Tous arguments (séparés) | `for arg in "$@"` |
| `$*` | Tous arguments (chaîne) | `echo "$*"` |

### Expansion Variables
```bash
# Substitution
echo "Bonjour $USER"          # Simple
echo "Bonjour ${USER}"         # Explicite (recommandé)
echo "File: ${1}.txt"          # Concaténation

# Valeurs par défaut
${var:-défaut}                 # Si var vide/non définie
${var:=défaut}                 # Assigne défaut si vide
${var:+autre}                  # Si var définie, retourne autre
${var:?erreur}                 # Erreur si var vide

# Manipulation chaînes
${var#pattern}                 # Supprime début (shortest match)
${var##pattern}                # Supprime début (longest match)
${var%pattern}                 # Supprime fin (shortest match)
${var%%pattern}                # Supprime fin (longest match)
${var/old/new}                 # Remplace première occurrence
${var//old/new}                # Remplace toutes occurrences
```

## 📥 Entrées Utilisateur

### Lecture Interactive
```bash
# Lecture simple
read -p "Entrez votre nom: " nom

# Options utiles
read -s password               # Masquer saisie (mot de passe)
read -t 10 response           # Timeout 10 secondes
read -n 1 key                 # Lire 1 caractère seulement
read -a array                 # Lire dans tableau

# Lecture multiple
read -p "Nom Prénom Age: " nom prenom age
read nom prenom reste         # 'reste' capture surplus

# Validation
while [[ ! "$input" =~ ^[0-9]+$ ]]; do
    read -p "Entrez un nombre: " input
done
```

### Arguments Ligne de Commande
```bash
# Parsing simple
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <fichier>"
    exit 1
fi

# Parsing avec options
while getopts "hvf:" opt; do
    case $opt in
        h) echo "Aide"; exit 0 ;;
        v) verbose=true ;;
        f) fichier="$OPTARG" ;;
        *) echo "Option invalide"; exit 1 ;;
    esac
done
shift $((OPTIND-1))            # Retirer options traitées
```

## 🔍 Tests & Conditions

### Tests de Base
| Type | Syntaxe | Usage |
|------|---------|-------|
| **POSIX** | `[ condition ]` | Compatible tous shells |
| **Bash** | `[[ condition ]]` | Fonctions étendues |
| **Arithmétique** | `(( expression ))` | Calculs numériques |

### Comparaisons Numériques
```bash
# Dans [[ ]] ou [ ]
[[ $a -eq $b ]]               # Égal
[[ $a -ne $b ]]               # Différent
[[ $a -gt $b ]]               # Supérieur
[[ $a -ge $b ]]               # Supérieur ou égal
[[ $a -lt $b ]]               # Inférieur
[[ $a -le $b ]]               # Inférieur ou égal

# Dans (( ))
(( a == b ))                  # Égal
(( a > b ))                   # Supérieur
(( a >= b ))                  # Supérieur ou égal
```

### Tests Chaînes & Fichiers
```bash
# Chaînes
[[ "$str" = "valeur" ]]       # Égalité exacte
[[ "$str" != "autre" ]]       # Différence
[[ "$str" =~ regex ]]         # Expression régulière (bash)
[[ -z "$str" ]]               # Chaîne vide
[[ -n "$str" ]]               # Chaîne non vide

# Fichiers/Répertoires
[[ -f "$file" ]]              # Fichier existe
[[ -d "$dir" ]]               # Répertoire existe
[[ -r "$file" ]]              # Lisible
[[ -w "$file" ]]              # Écrivable
[[ -x "$file" ]]              # Exécutable
[[ -s "$file" ]]              # Non vide
[[ "$f1" -nt "$f2" ]]         # Plus récent que
[[ "$f1" -ot "$f2" ]]         # Plus ancien que
```

### Structures Conditionnelles
```bash
# If standard
if [[ condition ]]; then
    actions
elif [[ autre_condition ]]; then
    autres_actions
else
    actions_par_défaut
fi

# If compact
[[ condition ]] && echo "Vrai" || echo "Faux"

# Case/Switch
case "$variable" in
    pattern1) actions1 ;;
    pattern2|pattern3) actions23 ;;
    [0-9]*) actions_numérique ;;
    *) actions_défaut ;;
esac
```

## 🔄 Boucles & Itération

### Types de Boucles
```bash
# For - Liste
for item in liste élément "avec espace"; do
    echo "$item"
done

# For - Range
for i in {1..10}; do echo "$i"; done
for i in {1..10..2}; do echo "$i"; done    # Pas de 2

# For - C-style
for ((i=0; i<10; i++)); do
    echo "$i"
done

# While
while [[ condition ]]; do
    actions
done

# Until (inverse de while)
until [[ condition ]]; do
    actions
done

# Boucle infinie
while true; do
    actions
    [[ condition ]] && break
done
```

### Contrôle Boucles
```bash
break                         # Sortir boucle
continue                      # Itération suivante
break 2                       # Sortir 2 niveaux de boucles

# Lecture fichier robuste
while IFS= read -r ligne; do
    echo "Ligne: $ligne"
done < fichier.txt
```

## 🧮 Arithmétique

### Méthodes Calcul
```bash
# Moderne (recommandé)
result=$((5 + 3))             # Expression arithmétique
((counter++))                 # Incrémentation
((sum += value))              # Addition assignment

# Legacy (éviter si possible)
result=`expr 5 + 3`           # Espaces obligatoires
result=$(expr $a + $b)        # Version $()

# Avec let
let "result = 5 + 3"
let result++                  # Incrémentation
```

### Opérateurs Arithmétiques
| Opérateur | Description | Exemple |
|-----------|-------------|---------|
| `+` `-` `*` `/` | Basiques | `$((a + b))` |
| `%` | Modulo | `$((a % b))` |
| `**` | Puissance | `$((a ** b))` |
| `++` `--` | Incr/Décr | `$((a++))` |
| `+=` `-=` | Assignment | `$((a += 5))` |

## 🔧 Fonctions

### Définition & Appel
```bash
# Syntaxe moderne
ma_fonction() {
    local param1="$1"
    local param2="$2"
    
    # Corps fonction
    echo "Résultat: $param1 $param2"
    return 0                  # Code retour (0-255)
}

# Appel
ma_fonction "arg1" "arg2"
result=$?                     # Récupérer code retour
```

### Variables Locales
```bash
fonction_avec_locales() {
    local var_locale="valeur"      # Portée limitée à fonction
    global_var="modifiée"          # Variable globale modifiée
    
    # Vérifier nombre arguments
    if [[ $# -ne 2 ]]; then
        echo "Usage: fonction arg1 arg2" >&2
        return 1
    fi
}
```

### Sourcing & Bibliothèques
```bash
# Charger fonctions externes
source ~/mes_fonctions.sh     # Méthode moderne
. ~/mes_fonctions.sh          # Méthode POSIX

# Structure bibliothèque
# ~/lib/utils.sh
is_number() {
    [[ $1 =~ ^[0-9]+$ ]]
}

log_info() {
    echo "[INFO] $(date): $*" >&2
}
```

## 📂 Gestion Fichiers & Flux

### Redirections
```bash
# Sortie standard
command > fichier             # Redirection (écrase)
command >> fichier            # Ajout (append)
command 2> erreurs.log        # Erreurs vers fichier
command &> tout.log           # Tout vers fichier
command > /dev/null 2>&1      # Silence complet

# Entrée
command < fichier.txt         # Lire depuis fichier
command <<< "chaîne"          # Here string
command <<EOF                 # Here document
ligne 1
ligne 2
EOF
```

### Pipes & Process Substitution
```bash
# Pipes simples
command1 | command2 | command3

# Éviter sous-shell avec process substitution
while read -r ligne; do
    # Variables persistent hors boucle
    ((compteur++))
done < <(commande_source)

# vs pipe classique (crée sous-shell)
commande_source | while read -r ligne; do
    # Variables perdues hors pipe
    ((compteur++))
done
```

## 🎨 Formatage Sortie

### Couleurs ANSI
```bash
# Codes couleur
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
PURPLE='\033[0;35m'
CYAN='\033[0;36m'
WHITE='\033[1;37m'
NC='\033[0m'                  # No Color

echo -e "${RED}Erreur${NC}: message"
echo -e "${GREEN}Succès${NC}: opération réussie"
```

### Caractères Spéciaux
```bash
echo -e "Ligne 1\nLigne 2"           # Nouvelle ligne
echo -e "Col1\tCol2\tCol3"           # Tabulations
echo -e "Loading\c"; sleep 1         # Pas de retour ligne
printf "%-10s %5d\n" "Nom" 42        # Formatage printf
```

## 🌐 Glob & Patterns

### Wildcards Basiques
| Pattern | Description | Exemple |
|---------|-------------|---------|
| `*` | 0 ou plus caractères | `*.txt` |
| `?` | 1 caractère exact | `file?.txt` |
| `[abc]` | Un parmi les caractères | `file[123].txt` |
| `[a-z]` | Plage caractères | `[A-Z]*.txt` |
| `[!abc]` | Sauf ces caractères | `[!0-9]*` |

### Extended Glob (extglob)
```bash
shopt -s extglob              # Activer extended globbing

# Patterns avancés
?(pattern)                    # 0 ou 1 fois
*(pattern)                    # 0 ou plus fois
+(pattern)                    # 1 ou plus fois
@(pattern)                    # Exactement 1 fois
!(pattern)                    # Tout sauf pattern

# Exemples
ls !(*.tmp)                   # Tous sauf .tmp
ls *.@(jpg|png|gif)          # Images seulement
```

## 🐛 Debug & Gestion Erreurs

### Techniques Debug
```bash
# Modes debug
bash -x script.sh             # Trace exécution
bash -n script.sh             # Vérif syntaxe uniquement
bash -v script.sh             # Affiche commandes lues

# Dans le script
set -x                        # Activer trace
set +x                        # Désactiver trace

# Debug conditionnel
DEBUG=${DEBUG:-0}
debug() {
    [[ $DEBUG -eq 1 ]] && echo "[DEBUG] $*" >&2
}
```

### Gestion Erreurs
```bash
# Vérification commandes
if ! command -v git >/dev/null 2>&1; then
    echo "Git n'est pas installé" >&2
    exit 1
fi

# Fonction error handling
error_exit() {
    echo "ERREUR: $1" >&2
    exit "${2:-1}"
}

# Trap pour nettoyage
cleanup() {
    rm -f "$TEMP_FILE"
    echo "Nettoyage effectué" >&2
}
trap cleanup EXIT INT TERM
```

## ⚡ Chaînage Commandes

### Opérateurs Logiques
```bash
# Exécution conditionnelle
cmd1 && cmd2                  # cmd2 si cmd1 réussit
cmd1 || cmd2                  # cmd2 si cmd1 échoue
cmd1 && cmd2 || cmd3          # if-then-else compact

# Séquentiel
cmd1; cmd2; cmd3              # Toujours toutes

# Groupement
(cd /tmp && make)             # Sous-shell (changements isolés)
{ cd /tmp && make; }          # Shell courant (espaces requis!)
```

## 🛠️ Outils Système

### Information Système
```bash
# Variables environnement
echo "$USER $HOME $PATH"
echo "$HOSTNAME $PWD"
printenv                      # Toutes variables

# Informations processus
ps aux | grep bash
pgrep -f mon_script
kill -TERM $PID

# Système
uname -a                      # Info système complète
df -h                         # Espace disques
free -h                       # Mémoire
uptime                        # Charge système
```

### Utilitaires Texte Courants
```bash
# Dans pipes/scripts
grep "pattern" fichier        # Recherche
sed 's/old/new/g' fichier     # Remplacement
awk '{print $1}' fichier      # Extraction colonnes
sort fichier                  # Tri
uniq fichier                  # Doublons
wc -l fichier                 # Compter lignes
head -n 10 fichier           # Premières lignes
tail -f fichier              # Suivre fichier
```

## ✅ Bonnes Pratiques

### Structure Script
```bash
#!/bin/bash
# Description, auteur, date

set -euo pipefail            # Mode strict

# Constantes
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly TEMP_DIR="/tmp/monscript.$$"

# Fonctions
usage() {
    echo "Usage: $0 [options] arguments" >&2
}

main() {
    # Logic principale
    return 0
}

# Trap cleanup
trap cleanup EXIT

# Parse arguments
while getopts "hv" opt; do
    case $opt in
        h) usage; exit 0 ;;
        v) set -x ;;
        *) usage; exit 1 ;;
    esac
done

# Exécution
main "$@"
```

### Règles de Sécurité
- ✅ **Toujours** quoter variables : `"$var"`
- ✅ **Vérifier** existence fichiers avant usage
- ✅ **Valider** entrées utilisateur
- ✅ **Utiliser** `local` dans fonctions
- ✅ **Éviter** `eval` sauf cas spéciaux
- ✅ **Nettoyer** fichiers temporaires (trap)
- ✅ **Tester** avec ShellCheck pour qualité code

### Validation & Tests
```bash
# Validation arguments
[[ $# -eq 0 ]] && { usage; exit 1; }
[[ ! -f "$fichier" ]] && error_exit "Fichier non trouvé: $fichier"

# Tests syntaxe
bash -n monscript.sh         # Vérification syntaxe
shellcheck monscript.sh      # Analyse qualité code
```
