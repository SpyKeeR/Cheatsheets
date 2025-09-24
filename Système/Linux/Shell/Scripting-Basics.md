# 🚀 Scripting Shell — Aide-mémoire

## 🎯 Démarrage & Structure

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
# Vérifier interpréteur disponible
which bash                    # Localiser bash
which python3                 # Pour autres langages

# Rendre exécutable
chmod +x monscript.sh
chmod 755 monscript.sh        # Permissions explicites

# Options robustesse (mode strict)
set -e                        # Arrêt si commande échoue
set -u                        # Arrêt si variable non définie
set -o pipefail              # Arrêt si erreur dans pipe
set -euo pipefail            # Combinaison recommandée
```

## 🔧 Variables & Portée

### Types de Variables
| Type | Syntaxe | Portée | Usage |
|------|---------|--------|-------|
| **Locale** | `var="valeur"` | Shell courant | Usage interne |
| **Environnement** | `export var="valeur"` | Shell + sous-processus | Configuration globale |
| **Constante** | `declare -r CONST="val"` | Lecture seule | Valeurs fixes |
| **Tableau** | `arr=("a" "b" "c")` | Éléments multiples | Listes de données |

### Variables Spéciales (Réservées)
| Variable | Description | Exemple |
|----------|-------------|---------|
| `$$` | PID processus courant | `echo "PID: $$"` |
| `$?` | Code retour dernière commande | `ls; echo $?` |
| `$!` | PID dernier processus arrière-plan | `sleep 10 & echo $!` |
| `$0` | Nom du script | `basename $0` |
| `$1..$9` | Arguments positionnels | `echo "Premier: $1"` |
| `$#` | Nombre d'arguments | `if [[ $# -lt 1 ]]` |
| `$@` | Tous arguments (séparés) | `for arg in "$@"` |
| `$*` | Tous arguments (chaîne) | `echo "$*"` |

### Gestion Variables
```bash
# Déclaration et assignation
var="valeur"                  # Pas d'espaces autour du =
export PATH="$PATH:/usr/local/bin"  # Modifier variable environnement

# Valeurs par défaut
${var:-défaut}                # Si var vide/non définie
${var:=défaut}                # Assigne défaut si vide
${var:+autre}                 # Si var définie, retourne autre
${var:?erreur}                # Erreur si var vide

# Utilitaires
set                          # Afficher toutes variables
unset var                    # Supprimer variable
declare -r CONST="fixe"      # Variable lecture seule
```

## 📥 Entrées & Arguments

### Lecture Utilisateur
```bash
# Lecture simple
read -p "Entrez votre nom: " nom

# Options utiles
read -s password             # Masquer saisie (mot de passe)
read -t 10 response         # Timeout 10 secondes
read -n 1 key               # Lire 1 caractère seulement

# Lecture multiple
read -p "Nom Prénom Age: " nom prenom age
read nom prenom reste       # 'reste' capture surplus

# Validation simple
while [[ ! "$input" =~ ^[0-9]+$ ]]; do
    read -p "Entrez un nombre: " input
done
```

### Arguments Ligne de Commande
```bash
# Vérification basique
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <fichier>"
    exit 1
fi

# Parsing avec getopts
while getopts "hvf:" opt; do
    case $opt in
        h) echo "Aide"; exit 0 ;;
        v) verbose=true ;;
        f) fichier="$OPTARG" ;;
        *) echo "Option invalide"; exit 1 ;;
    esac
done
shift $((OPTIND-1))          # Retirer options traitées
```

## 🔍 Tests & Conditions

### Types de Tests
| Type | Syntaxe | Compatibilité | Usage |
|------|---------|---------------|-------|
| **POSIX** | `[ condition ]` | Tous shells | Maximum compatibilité |
| **Bash** | `[[ condition ]]` | Bash uniquement | Fonctions étendues |
| **Arithmétique** | `(( expression ))` | Bash/moderne | Calculs numériques |

### Opérateurs de Test
```bash
# Comparaisons numériques
[[ $a -eq $b ]]              # Égal
[[ $a -ne $b ]]              # Différent
[[ $a -gt $b ]]              # Supérieur
[[ $a -ge $b ]]              # Supérieur ou égal
[[ $a -lt $b ]]              # Inférieur
[[ $a -le $b ]]              # Inférieur ou égal

# Tests chaînes
[[ "$str" = "valeur" ]]      # Égalité exacte
[[ "$str" != "autre" ]]      # Différence
[[ "$str" =~ regex ]]        # Expression régulière (bash)
[[ -z "$str" ]]              # Chaîne vide
[[ -n "$str" ]]              # Chaîne non vide

# Tests fichiers/répertoires
[[ -f "$file" ]]             # Fichier existe
[[ -d "$dir" ]]              # Répertoire existe
[[ -r "$file" ]]             # Lisible
[[ -w "$file" ]]             # Écrivable
[[ -x "$file" ]]             # Exécutable
[[ -s "$file" ]]             # Non vide
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
# For - Liste explicite
for item in "élément1" "élément2" "avec espace"; do
    echo "$item"
done

# For - Expansion
for i in {1..10}; do echo "$i"; done
for i in {1..10..2}; do echo "$i"; done    # Pas de 2
for file in *.txt; do echo "$file"; done

# For - Style C
for ((i=0; i<10; i++)); do
    echo "$i"
done

# While (tant que condition vraie)
while [[ condition ]]; do
    actions
done

# Until (jusqu'à ce que condition vraie)
until [[ condition ]]; do
    actions
done

# Lecture fichier robuste
while IFS= read -r ligne; do
    echo "Ligne: $ligne"
done < fichier.txt
```

### Contrôle de Flux
```bash
break                        # Sortir de boucle
continue                     # Itération suivante
break 2                      # Sortir de 2 niveaux de boucles
```

## 🧮 Arithmétique & Calculs

### Méthodes de Calcul
```bash
# Moderne (recommandé)
result=$((5 + 3))            # Expression arithmétique
((counter++))                # Incrémentation
((sum += value))             # Addition assignment

# Test arithmétique
if (( a > b )); then
    echo "$a est plus grand"
fi

# Legacy (éviter)
result=`expr 5 + 3`          # Backticks obsolètes
result=$(expr $a + $b)       # Espaces obligatoires
```

### Opérateurs Arithmétiques
| Opérateur | Description | Exemple |
|-----------|-------------|---------|
| `+` `-` `*` `/` | Basiques | `$((a + b))` |
| `%` | Modulo | `$((a % b))` |
| `**` | Puissance | `$((a ** b))` |
| `++` `--` | Incr/Décr | `$((a++))` |
| `+=` `-=` | Assignment | `$((a += 5))` |

## 🌟 Patterns & Globbing

### Wildcards de Base
| Pattern | Description | Exemple |
|---------|-------------|---------|
| `*` | 0 ou plus caractères | `*.txt` |
| `?` | 1 caractère exact | `file?.txt` |
| `[abc]` | Un des caractères | `file[123].txt` |
| `[a-z]` | Plage caractères | `[A-Z]*` |
| `[^abc]` | Sauf ces caractères | `[^0-9]*` |

### Extended Globbing
```bash
shopt -s extglob             # Activer fonctions étendues

# Patterns avancés
?(pattern)                   # 0 ou 1 occurrence
*(pattern)                   # 0 ou plus occurrences
+(pattern)                   # 1 ou plus occurrences
@(pattern)                   # Exactement 1 occurrence
!(pattern)                   # Tout sauf ce pattern

# Exemples pratiques
ls !(*.tmp)                  # Tous sauf .tmp
ls *.@(jpg|png|gif)         # Images seulement
```

## ⚡ Chaînage & Contrôle d'Exécution

### Opérateurs de Chaînage
```bash
# Séquentiel (toujours exécuté)
cmd1; cmd2; cmd3

# Conditionnel
cmd1 && cmd2                 # cmd2 si cmd1 réussit
cmd1 || cmd2                 # cmd2 si cmd1 échoue
cmd1 && cmd2 || cmd3         # if-then-else compact

# Groupement
(cd /tmp && make)            # Sous-shell (changements isolés)
{ cd /tmp && make; }         # Shell courant (espaces requis!)
```

### Gestion Codes de Sortie
```bash
# Codes standards
exit 0                       # Succès
exit 1                       # Erreur générale
exit 2                       # Usage incorrect
exit 130                     # Interrompu par Ctrl+C

# Vérification succès
if command; then
    echo "Succès"
else
    echo "Échec (code: $?)"
fi
```

## 🔧 Fonctions

### Définition & Usage
```bash
# Syntaxe moderne
ma_fonction() {
    local param1="$1"
    local param2="$2"
    
    # Corps fonction
    echo "Résultat: $param1 $param2"
    return 0                 # Code retour (0-255)
}

# Appel
ma_fonction "arg1" "arg2"
result=$?                    # Récupérer code retour
```

### Variables Locales
```bash
fonction_avec_locales() {
    local var_locale="valeur"     # Portée limitée à fonction
    global_var="modifiée"         # Variable globale modifiée
    
    # Vérifier nombre arguments
    if [[ $# -ne 2 ]]; then
        echo "Usage: fonction arg1 arg2" >&2
        return 1
    fi
}
```

## 📂 Pipes & Redirections

### Redirections de Base
```bash
# Sortie
command > fichier            # Redirection (écrase)
command >> fichier           # Ajout (append)
command 2> erreurs.log       # Erreurs vers fichier
command &> tout.log          # Tout vers fichier
command > /dev/null 2>&1     # Silence complet

# Entrée
command < fichier.txt        # Lire depuis fichier
command <<< "chaîne"         # Here string
command <<EOF                # Here document
ligne 1
ligne 2
EOF
```

### Pipes & Subshells
```bash
# Pipes simples
command1 | command2 | command3

# Éviter subshell avec process substitution
while read -r ligne; do
    # Variables persistent hors boucle
    ((compteur++))
done < <(commande_source)

# vs pipe classique (crée subshell)
commande_source | while read -r ligne; do
    # Variables perdues hors pipe
    ((compteur++))
done
```

## 🐛 Debug & Gestion d'Erreurs

### Techniques Debug
```bash
# Modes debug
bash -x script.sh            # Trace exécution
bash -n script.sh            # Vérif syntaxe uniquement
bash -v script.sh            # Affiche commandes lues

# Dans le script
set -x                       # Activer trace
set +x                       # Désactiver trace

# Debug conditionnel
DEBUG=${DEBUG:-0}
debug() {
    [[ $DEBUG -eq 1 ]] && echo "[DEBUG] $*" >&2
}
```

### Gestion d'Erreurs
```bash
# Vérification existence commande
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

## 🎨 Formatage Sortie

### Couleurs ANSI
```bash
# Codes couleur
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'                 # No Color

echo -e "${RED}Erreur${NC}: message"
echo -e "${GREEN}Succès${NC}: opération réussie"
```

### Options echo
```bash
echo -e "Ligne 1\nLigne 2"      # Nouvelle ligne (\n)
echo -e "Col1\tCol2\tCol3"      # Tabulations (\t)
echo -e "Loading\c"             # Pas de retour ligne (\c)
printf "%-10s %5d\n" "Nom" 42   # Formatage printf
```

## 🔒 Sécurité & Bonnes Pratiques

### Validation Entrées
```bash
# Vérifier arguments
[[ $# -eq 0 ]] && { echo "Usage: $0 <file>"; exit 1; }

# Vérifier fichiers
[[ ! -f "$file" ]] && error_exit "Fichier non trouvé: $file"

# Validation format
if [[ ! "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Email invalide" >&2
    exit 1
fi
```

### Script Robuste - Template
```bash
#!/bin/bash
# Description: Template script robuste
# Author: Nom
# Date: YYYY-MM-DD

# Mode strict
set -euo pipefail

# Constantes
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly TEMP_DIR="/tmp/$(basename "$0").$$"

# Nettoyage automatique
cleanup() {
    [[ -d "$TEMP_DIR" ]] && rm -rf "$TEMP_DIR"
}
trap cleanup EXIT

# Fonction aide
usage() {
    echo "Usage: $0 [options] arguments" >&2
    echo "Options:" >&2
    echo "  -h    Afficher cette aide" >&2
    echo "  -v    Mode verbeux" >&2
}

# Variables
verbose=false

# Parse options
while getopts "hv" opt; do
    case $opt in
        h) usage; exit 0 ;;
        v) verbose=true ;;
        *) usage; exit 1 ;;
    esac
done
shift $((OPTIND-1))

# Fonction principale
main() {
    # Logique script ici
    echo "Script exécuté avec succès"
}

# Exécution
main "$@"
```

### Règles d'Or
- ✅ **Toujours** quoter variables : `"$var"`
- ✅ **Vérifier** existence fichiers avant traitement
- ✅ **Valider** toutes entrées utilisateur
- ✅ **Utiliser** `local` dans toutes les fonctions
- ✅ **Éviter** `eval` sauf cas très spécifiques
- ✅ **Nettoyer** fichiers temporaires avec trap
- ✅ **Tester** scripts avec `shellcheck`

---
**💡 Memo** : Mode strict `set -euo pipefail`, toujours quoter `"$var"`, `local` dans fonctions, trap pour cleanup !


