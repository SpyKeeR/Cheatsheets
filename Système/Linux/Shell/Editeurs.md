# 📝 Éditeurs Linux — Aide-mémoire

## 🧠 Concepts Éditeurs

### Philosophie Unix
- **Vi/Vim** : Modal, puissant, courbe d'apprentissage élevée
- **Nano** : WYSIWYG, simple, débutant-friendly
- **Emacs** : Extensible, environnement complet
- **Ed/Sed** : Ligne de commande, scripts automatisés

### Modes d'Édition
| Type | Exemple | Caractéristique |
|------|---------|-----------------|
| **Modal** | Vim, Vi | Modes séparés (navigation/insertion) |
| **Non-modal** | Nano, Emacs | Insertion directe + raccourcis |
| **Stream** | Sed, Awk | Traitement flux de données |

## 🎯 Vim — Éditeur Modal

### Démarrage & Options
```bash
# Ouverture fichiers
vim fichier.txt              # Édition standard
vim +42 fichier.txt          # Aller ligne 42
vim +/pattern fichier.txt    # Aller première occurrence
vim -R fichier.txt           # Mode lecture seule
vim -o file1 file2           # Split horizontal
vim -O file1 file2           # Split vertical

# Récupération après crash
vim -r fichier.txt           # Récupérer .swp
vim -r                       # Lister fichiers récupérables

# Apprentissage
vimtutor                     # Tutoriel interactif (30min)
```

### Modes Fondamentaux
| Mode | Activation | Usage | Sortie |
|------|------------|-------|--------|
| **Normal** | `Esc` | Navigation, commandes | Mode par défaut |
| **Insertion** | `i`, `a`, `o` | Saisie texte | `Esc` |
| **Visuel** | `v`, `V`, `Ctrl+v` | Sélection | `Esc` |
| **Commande** | `:` | Ex-commandes | `Esc` ou `Enter` |
| **Recherche** | `/`, `?` | Recherche | `Esc` ou `Enter` |

### Navigation Efficace
```vim
" Déplacements de base
h j k l                      " Gauche, bas, haut, droite
0 ^ $                        " Début ligne, premier mot, fin ligne
w b e                        " Mot suivant, précédent, fin mot
W B E                        " Mot large (ignore ponctuation)

" Saut de ligne
gg G                         " Début/fin fichier  
42G :42                      " Ligne 42
42gg                         # Alternative ligne 42

" Navigation écran
Ctrl+f Ctrl+b               " Page suivante/précédente
Ctrl+d Ctrl+u               " Demi-page bas/haut
Ctrl+e Ctrl+y               " Scroll ligne par ligne
H M L                       " Haut/milieu/bas écran
zz zt zb                    " Centrer/haut/bas curseur
```

### Insertion & Édition
```vim
" Modes d'insertion
i a                         " Insérer avant/après curseur
I A                         " Début/fin de ligne
o O                         " Nouvelle ligne bas/haut
s S                         " Substituer caractère/ligne

" Suppression
x X                         " Caractère sous/avant curseur
dw db dd                    " Mot suivant/précédent/ligne
d$ d0                       " Fin/début ligne
5dd                         " 5 lignes

" Modification
r R                         " Remplacer 1 car/mode remplace
cw cb cc                    " Changer mot/mot préc./ligne
C c$                        " Changer jusqu'à fin ligne
~ u U                       " Changer casse/undo/undo ligne
```

### Copier-Coller & Registres
```vim
" Copie standard
yy Y                        " Copier ligne
yw yb                       " Copier mot suivant/précédent
y$ y0                       " Copier jusqu'à fin/début ligne
5yy                         " Copier 5 lignes

" Collage
p P                         " Coller après/avant curseur

" Registres nommés (a-z)
"ayy                        " Copier ligne dans registre a
"ap                         " Coller depuis registre a
"Ayy                        " Ajouter ligne au registre a

" Registres spéciaux
""                          " Registre par défaut
"0                          " Dernière copie (yank)
"1-"9                       " Historique suppressions
"+                          " Presse-papier système
"*                          " Sélection souris (Linux)
```

### Recherche & Remplacement
```vim
" Recherche
/pattern                    " Recherche avant
?pattern                    " Recherche arrière
n N                         " Occurrence suivante/précédente
* #                         " Mot sous curseur avant/arrière

" Options recherche
:set ignorecase             " Ignorer casse
:set smartcase              " Casse intelligente
:set hlsearch               " Surligner résultats
:nohlsearch                 " Supprimer surlignage

" Remplacement
:s/old/new/                 " Première occurrence ligne
:s/old/new/g                " Toutes occurrences ligne
:%s/old/new/g               " Tout le fichier
:%s/old/new/gc              " Avec confirmation
:15,25s/old/new/g           " Lignes 15 à 25
```

### Commandes Globales (:g)
```vim
" Syntaxe: :g/pattern/command
:g/ERROR/d                  " Supprimer lignes avec ERROR
:g/TODO/m$                  " Déplacer TODOs à la fin
:g/DEBUG/s//INFO/g          " Remplacer DEBUG par INFO
:v/pattern/d                " Supprimer lignes SANS pattern

" Avec plages
:1,50g/function/p           " Afficher lignes avec 'function'
:g/class/+2d                " Supprimer ligne + 2 suivantes
```

### Gestion Fichiers & Buffers
```vim
" Sauvegarde & sortie
:w                          " Sauvegarder
:w filename                 " Sauvegarder sous
:wq :x ZZ                   " Sauvegarder et quitter
:q :q!                      " Quitter (avec/sans sauvegarde)

" Buffers multiples
:e filename                 " Ouvrir fichier
:b1 :b2                     " Basculer buffer 1/2
:bn :bp                     " Buffer suivant/précédent
:bd                         " Fermer buffer
:ls :buffers                " Lister buffers

" Splits
:split :sp                  " Split horizontal
:vsplit :vsp                " Split vertical
Ctrl+w h/j/k/l              " Naviguer entre splits
Ctrl+w +/-                  " Redimensionner
Ctrl+w =                    " Égaliser tailles
```

### Configuration & Personnalisation
```vim
" Configuration session (~/.vimrc pour permanent)
:set number                 " Numérotation lignes
:set relativenumber         " Numérotation relative
:set expandtab              " Espaces au lieu de tabs
:set tabstop=4              " Largeur tabulation
:set shiftwidth=4           " Largeur indentation
:set autoindent             " Indentation automatique
:syntax on                  " Coloration syntaxique

" Exemple ~/.vimrc minimal
set nocompatible            " Mode Vim (pas Vi)
set number                  " Numéros de ligne
set expandtab               " Espaces pour tabs
set tabstop=4 shiftwidth=4  " Indentation 4 espaces
set autoindent smartindent  " Indentation intelligente
syntax on                   " Coloration
set hlsearch incsearch      " Recherche améliorée
set mouse=a                 " Support souris
```

### Macros & Automatisation
```vim
" Enregistrement macro
qa                          " Commencer macro dans registre a
[commandes...]              " Séquence d'actions
q                           " Arrêter enregistrement

" Exécution macro
@a                          " Exécuter macro a
5@a                         " Exécuter 5 fois
@@                          " Répéter dernière macro
```

## 🌱 Nano — Éditeur Simple

### Interface & Navigation
```bash
# Démarrage
nano fichier.txt            # Édition standard
nano +42 fichier.txt        # Aller ligne 42
nano -R fichier.txt         # Mode lecture seule
nano -w fichier.txt         # Désactiver word wrap
nano -T 4 fichier.txt       # Tabs = 4 espaces
```

### Raccourcis Essentiels
| Raccourci | Action | Description |
|-----------|--------|-------------|
| `Ctrl+O` | WriteOut | Sauvegarder fichier |
| `Ctrl+X` | Exit | Quitter (demande sauvegarde si modifié) |
| `Ctrl+R` | Read File | Insérer contenu autre fichier |
| `Ctrl+W` | Where Is | Rechercher texte |
| `Ctrl+\` | Replace | Rechercher et remplacer |
| `Ctrl+K` | Cut | Couper ligne entière |
| `Ctrl+U` | Uncut | Coller lignes coupées |
| `Ctrl+G` | Help | Aide contextuelle |

### Navigation Avancée
```bash
# Déplacement
Ctrl+A / Ctrl+E            # Début/fin de ligne
Ctrl+Y / Ctrl+V            # Page précédente/suivante  
Ctrl+_ (Ctrl+Shift+-)      # Aller à ligne X
Alt+\ (Alt+|)              # Aller début/fin fichier
Ctrl+C                     # Position curseur actuelle

# Sélection & édition
Alt+A                      # Marquer début sélection
Alt+6                      # Copier sélection
Ctrl+K                     # Couper ligne/sélection
Ctrl+U                     # Coller
```

### Configuration Nano
```bash
# Fichier ~/.nanorc
set tabsize 4               # Taille tabulations
set tabstospaces            # Convertir tabs en espaces
set autoindent              # Indentation automatique
set linenumbers             # Numéros de ligne
set mouse                   # Support souris
set softwrap                # Retour ligne doux
set smooth                  # Scroll doux

# Coloration syntaxique
include "/usr/share/nano/*.nanorc"
```

## 🔄 Conversion Formats

### Fin de Lignes (CRLF/LF)
| Système | Format | Caractères |
|---------|--------|------------|
| **Unix/Linux** | LF | `\n` |
| **Windows** | CRLF | `\r\n` |  
| **Mac Classic** | CR | `\r` |

### Outils Conversion
```bash
# dos2unix / unix2dos
dos2unix fichier.txt        # Windows → Unix
unix2dos fichier.txt        # Unix → Windows
dos2unix -k fichier.txt     # Garder timestamp

# Avec sed
sed 's/\r$//' file.txt      # Supprimer \r (Win → Unix)
sed 's/$/\r/' file.txt      # Ajouter \r (Unix → Win)

# Vérifier format
file fichier.txt            # Indique type fin de ligne
od -c fichier.txt | head    # Visualiser caractères spéciaux

# Dans Vim
:set fileformat=unix        # Forcer format Unix
:set fileformat=dos         # Forcer format Windows
:set fileformat?            # Voir format actuel
```

### Encodages Caractères
```bash
# Détecter encodage
file -bi fichier.txt        # Avec libmagic
chardet fichier.txt         # Python chardet

# Convertir avec iconv
iconv -f ISO-8859-1 -t UTF-8 input.txt > output.txt
iconv -l                    # Lister encodages disponibles

# Dans Vim
:set encoding=utf-8         # Encodage interne Vim
:set fileencoding=utf-8     # Encodage fichier
:e ++enc=latin1 fichier.txt # Ouvrir avec encodage spécifique
```

## 🚀 Éditeurs Avancés

### Emacs (Concepts)
```bash
# Raccourcis de base (Ctrl = C-, Meta/Alt = M-)
C-x C-f                     # Ouvrir fichier
C-x C-s                     # Sauvegarder
C-x C-c                     # Quitter
C-g                         # Annuler commande
C-x u                       # Undo
C-space                     # Marquer début sélection
```

### Ed (Éditeur Ligne)
```bash
# Éditeur scriptable primitif
ed fichier.txt
a                           # Mode ajout
.                           # Fin mode ajout
1,5p                        # Afficher lignes 1-5
s/old/new/g                 # Remplacer sur ligne courante
w                           # Sauvegarder
q                           # Quitter
```

## 🎯 Comparaison Éditeurs

### Choix selon Contexte
| Situation | Éditeur Recommandé | Raison |
|-----------|-------------------|--------|
| **SSH distant/limité** | Vi/Vim | Toujours disponible |
| **Débutant** | Nano | Interface intuitive |
| **Script/automatisation** | Sed/Awk | Traitement batch |
| **Développement** | Vim/Emacs/VSCode | Fonctions avancées |
| **Configuration rapide** | Nano | Modification simple |
| **Fichiers volumineux** | Vim | Performance |

### Performance & Mémoire
- **Vi/Vim** : Rapide, léger, modal
- **Nano** : Simple, mémoire raisonnable
- **Emacs** : Lourd mais très extensible
- **VSCode/Atom** : Interface moderne, consommation élevée

## 💡 Bonnes Pratiques

### Sécurité Édition
- ✅ **Sauvegarde** avant modification fichiers système
- ✅ **Permissions** : vérifier droits écriture
- ✅ **Sudo** : éditer avec privilèges si nécessaire
- ✅ **Verrouillage** : attention aux .swp (Vim)

### Efficacité
- ✅ **Raccourcis** : apprendre progressivement
- ✅ **Configuration** : personnaliser selon usage
- ✅ **Cohérence** : choisir éditeur principal et s'y tenir
- ✅ **Backup** : copier fichiers critiques avant édition

### Dépannage Vim
```bash
# Problèmes courants
:set paste                  # Désactiver auto-indent pour coller
:set nopaste                # Réactiver auto-indent
:syntax off                 # Désactiver coloration (performance)
:set nocompatible           # Mode Vim complet

# Récupération fichiers
vim -r                      # Lister fichiers récupérables
ls -la .*.swp               # Voir fichiers .swp
rm .filename.swp            # Supprimer si récupération OK
```

---
**💡 Memo** : `vimtutor` pour apprendre, `Ctrl+G` dans nano pour aide, toujours sauvegarder avant gros changements !