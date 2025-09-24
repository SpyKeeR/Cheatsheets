# 🎭 Méta-caractères & Expressions Régulières — Aide-mémoire

## 🌟 Méta-caractères Shell (Globbing)

### Wildcards de Base
| Méta-caractère | Description | Exemple |
|----------------|-------------|---------|
| `*` | 0 ou plusieurs caractères | `*.txt` → tous les .txt |
| `?` | Exactement 1 caractère | `file?.txt` → file1.txt, filea.txt |
| `[abc]` | Un des caractères listés | `file[123].txt` → file1.txt, file2.txt |
| `[a-z]` | Plage de caractères | `[A-Z]*` → fichiers commençant par majuscule |
| `[!abc]` ou `[^abc]` | Tout sauf ces caractères | `[!0-9]*` → pas de chiffre initial |

### Extended Globbing (extglob)
```bash
# Activation nécessaire
shopt -s extglob

# Patterns avancés
?(pattern)    # 0 ou 1 occurrence (optionnel)
*(pattern)    # 0 ou plusieurs occurrences  
+(pattern)    # 1 ou plusieurs occurrences
@(pattern)    # Exactement 1 occurrence
!(pattern)    # Tout sauf ce pattern
```

### Exemples Pratiques
```bash
# Globbing de base
ls *.{jpg,png,gif}              # Images communes
ls file[0-9][0-9].txt          # file00.txt à file99.txt
ls [A-Z]*                      # Fichiers majuscule initiale

# Extended globbing
ls !(*.tmp)                    # Tout sauf fichiers .tmp
ls *+([0-9]).txt              # Fichiers avec au moins 1 chiffre
ls @(readme|license|changelog) # Fichiers spécifiques exactement
ls ?(backup-)*.sql            # Avec ou sans préfixe "backup-"
```

## 🔤 Protection & Échappement

### Types de Quotes
| Délimiteur | Interprétation | Usage |
|------------|----------------|-------|
| `"double"` | Variables, commandes interprétées | `"Bonjour $USER"` |
| `'simple'` | Littéral (aucune interprétation) | `'$HOME reste $HOME'` |
| `\` | Échappement caractère suivant | `\$` pour $ littéral |
| `$'...'` | Séquences d'échappement ANSI | `$'Ligne1\nLigne2'` |

### Caractères Spéciaux à Protéger
```bash
# Méta-caractères shell courants
$ * ? [ ] { } ( ) | & ; < > space tab newline

# Exemples d'échappement
echo \$HOME                    # Affiche "$HOME" littéral
echo "Prix: 100\$"            # $ littéral dans double quote
echo 'Coût: $100'            # $ littéral dans simple quote
touch "fichier avec espaces.txt"  # Espaces protégés
```

## 🔍 Expressions Régulières (POSIX)

### Méta-caractères Fondamentaux
| Symbole | Signification | Exemple |
|---------|---------------|---------|
| `.` | N'importe quel caractère (sauf `\n`) | `a.c` → abc, axc, a5c |
| `^` | Début de ligne | `^Hello` → ligne commence par Hello |
| `$` | Fin de ligne | `end$` → ligne finit par end |
| `*` | 0 ou plus du précédent | `ab*c` → ac, abc, abbc |
| `\+` | 1 ou plus du précédent | `ab\+c` → abc, abbc (pas ac) |
| `\?` | 0 ou 1 du précédent | `colou\?r` → color, colour |
| `\|` | OU logique | `cat\|dog` → cat ou dog |

### Classes de Caractères
```bash
# Classes prédéfinies POSIX
[[:alpha:]]     # Lettres a-z, A-Z
[[:alnum:]]     # Lettres + chiffres
[[:digit:]]     # Chiffres 0-9
[[:upper:]]     # Majuscules A-Z
[[:lower:]]     # Minuscules a-z  
[[:space:]]     # Espaces, tabs, retours ligne
[[:punct:]]     # Ponctuation

# Raccourcis modernes (selon outil)
\d              # Chiffres (équivaut [0-9])
\w              # Mot (équivaut [a-zA-Z0-9_])
\s              # Espacement [\t\n\r\f ]
\D \W \S        # Négations des précédents
```

### Quantificateurs Précis
```bash
# Répétitions exactes
{n}             # Exactement n fois
{n,}            # Au moins n fois  
{n,m}           # Entre n et m fois

# Exemples
a{3}            # aaa exactement
a{2,}           # aa, aaa, aaaa...
a{2,4}          # aa, aaa, aaaa
[0-9]{3,5}      # 3 à 5 chiffres consécutifs
```

## 🎯 Groupes & Références

### Groupes de Capture
```bash
# Syntaxe POSIX Basic (BRE)
\(...\)         # Groupe capturant
\1, \2, \3      # Références aux groupes

# Exemples avec sed
echo "John Doe" | sed 's/\([A-Z][a-z]*\) \([A-Z][a-z]*\)/\2, \1/'
# Résultat: "Doe, John"

# Détecter doublons de mots
echo "le le chat" | sed 's/\([a-z]\+\) \1/[\1 répété]/'
# Résultat: "[le répété] chat"
```

### Ancres & Positions
```bash
^               # Début ligne absolue
$               # Fin ligne absolue  
\<              # Début de mot (word boundary)
\>              # Fin de mot
\b              # Frontière de mot (moderne)

# Exemples
^[A-Z]          # Ligne commence par majuscule
[.!?]$          # Ligne finit par ponctuation
\<the\>         # Mot "the" complet (pas "then", "other")
```

## ⚙️ Outils & Applications

### grep (Recherche dans Fichiers)
```bash
# Options regex importantes
grep -E "pattern"           # Extended regex (ERE)
grep -P "pattern"           # Perl-compatible regex (PCRE)
grep "^pattern$"           # Ligne exacte
grep -v "pattern"          # Inverser (lignes sans pattern)
grep -i "pattern"          # Insensible casse
grep -n "pattern"          # Numéros lignes
grep -o "pattern"          # Seulement partie matchée

# Exemples pratiques
grep -E '^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$' # IP basique
grep '^#' /etc/passwd      # Lignes commentées
grep -v '^$' fichier       # Lignes non vides
```

### sed (Stream Editor)
```bash
# Substitutions avec regex
sed 's/ancien/nouveau/'         # Première occurrence ligne
sed 's/ancien/nouveau/g'        # Toutes occurrences
sed 's/^[ \t]*//g'             # Supprimer espaces début
sed 's/[ \t]*$//'              # Supprimer espaces fin
sed '/^$/d'                    # Supprimer lignes vides

# Adressage avec regex
sed '/^#/d'                    # Supprimer lignes commençant par #
sed '/pattern/s/old/new/g'     # Substituer seulement lignes avec pattern
```

### awk (Pattern Scanning)
```bash
# Regex dans conditions
awk '/^[A-Z]/ { print $0 }'   # Lignes commençant majuscule
awk '$1 ~ /^[0-9]+$/ { sum += $1 } END { print sum }'  # Sommer champs numériques
awk 'NF > 0'                  # Lignes non vides (NF = nombre de champs)
```

## 📝 Expressions Courantes

### Validation Formats
```bash
# Email basique
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$

# Numéro téléphone français  
^0[1-9]([0-9]{8})$

# Code postal français
^[0-9]{5}$

# IPv4 (simple)
^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$

# URL HTTP(S)
^https?://[a-zA-Z0-9.-]+(/.*)?$

# Mot de passe fort (8+ chars, maj+min+chiffre)
^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9]).{8,}$
```

### Nettoyage Texte
```bash
# Supprimer lignes vides
sed '/^$/d'

# Supprimer espaces début/fin
sed 's/^[ \t]*//;s/[ \t]*$//'

# Supprimer doublons lignes consécutives
sed '/^$/N;/^\n$/d'

# Extraire adresses email
grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'

# Extraire URLs
grep -oE 'https?://[a-zA-Z0-9.-]+(/[^ ]*)?'
```

## 🔧 Différences BRE vs ERE

### Basic Regular Expression (BRE) - défaut grep, sed
```bash
# Méta-caractères échappés
\+              # 1 ou plus (au lieu de +)
\?              # 0 ou 1 (au lieu de ?)  
\|              # OU (au lieu de |)
\{n,m\}         # Quantificateurs (au lieu de {n,m})
\(...\)         # Groupes (au lieu de (...))

# Usage BRE
grep 'colou\?r'             # 0 ou 1 'u'
sed 's/ab\+/X/'            # 1 ou plus 'b'
```

### Extended Regular Expression (ERE) - grep -E, egrep
```bash
# Méta-caractères directs
+               # 1 ou plus
?               # 0 ou 1
|               # OU  
{n,m}           # Quantificateurs
(...)           # Groupes

# Usage ERE
grep -E 'colou?r'           # 0 ou 1 'u'
grep -E 'cat|dog'           # cat OU dog
grep -E '[0-9]{2,4}'        # 2 à 4 chiffres
```

## 💡 Bonnes Pratiques & Astuces

### Performance & Optimisation
- ✅ **Ancrer** expressions : `^pattern$` plus rapide que `pattern`
- ✅ **Être spécifique** : `[0-9]` plus clair que `[0123456789]`
- ✅ **Éviter** quantificateurs gourmands sur gros textes
- ✅ **Utiliser** classes POSIX pour portabilité : `[[:digit:]]`

### Debugging Regex
```bash
# Tester interactivement
echo "test string" | grep -E "pattern"
echo "test" | sed -n 's/pattern/replacement/p'

# Verbose mode (certains outils)
grep --color=always "pattern"      # Surligner matches
sed -n 'l' fichier                 # Visualiser caractères spéciaux
```

### Échappement Contexte
```bash
# Dans bash (double quotes)
grep "^\$[0-9]"                    # Chercher $123 ($ échappé)

# Dans sed  
sed 's/\$/EUR/g'                   # Remplacer $ par EUR

# Dans fichiers de config (souvent literal)
# Éviter regex accidentelle avec fgrep
fgrep '$HOME' fichier              # Recherche littérale
```

---
**💡 Memo** : `*` = globbing shell, `.*` = regex "tout caractère". Toujours tester regex avec échantillon avant usage massif !