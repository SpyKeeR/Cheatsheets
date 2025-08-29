# Éditeurs (vim / nano) — résumé pratique

## Vim (essentiel)

`vimtutor` pour apprendre VIM.

### Options cmdline
- -R : ouvre le fichier en lecture seule (pratique pour ne pas modifier accidentellement un fichier système).
- +n : ouvre le fichier directement à la ligne n (ex : +42 va à la ligne 42).
- +/motif : ouvre le fichier en positionnant le curseur sur la première occurrence du mot ou motif recherché.
- ouvrir plusieurs fichiers d’un coup (ex : vim fichier1.txt fichier2.txt) et naviguer entre eux avec :n (suivant) ou :prev (précédent).


| Mode              | Fonction principale                       | Pour y accéder                            |
|-------------------|-------------------------------------------|-------------------------------------------|
| Mode normal       | Navigation, suppression, copier/coller…   | Par défaut ou avec Échap                  |
| Mode insertion    | Écriture de texte                         | i, a, o, O                                |
| Mode visuel       | Sélection de texte                        | v (caractères), V (lignes), Ctrl+v (bloc) |
| Ligne de commande | Commandes comme :w, :q, :set number, etc. | Touche deux-points :                      |

### Déplacements
- h, j, k, l : gauche, bas, haut, droite
- 0 : début de la ligne
- ^ : début du premier mot de la ligne
- $ : fin de la ligne (fin de fichier?)
- w : début du mot ou ponctuation suivant
- b : début du mot ou ponctuation précédent
- e : fin du mot ou ponctuation suivant
- gg : tout début du fichier
- G : dernière ligne du fichier
- :n : va à la ligne numéro n


### Enregistrer/quitter
- :w : enregistre les modifications
- :wq : enregistre et quitte
- :wq! : enregistre et quitte, même si normalement c’est en lecture seule
- :x : enregistre si besoin et quitte
- :q : quitte sans enregistrer (si aucun changement)
- :q! : quitte en forçant, sans sauvegarder

### Modifications

#### Supprimer/Couper
- x : supprime un caractère sous le curseur
- dw : supprime un mot (à partir du curseur)
- dd : supprime la ligne entière

#### Copier
- yy : copie la ligne actuelle
- yw : copie un mot (à partir du curseur)

#### Coller
- p : colle après le curseur
- P : colle avant le curseur

#### Remplacer

| Commande | Action                                | Passe en insertion ?       |
|----------|---------------------------------------|----------------------------|
| r        | Remplace 1 caractère                  | ❌ (juste une lettre tapée) |
| R        | Mode remplacement (texte tapé écrase) | ✅                          |
| cl       | Supprime 1 caractère et insertion     | ✅                          |
| cw       | Supprime 1 mot et insertion           | ✅                          |
| cc       | Supprime la ligne et insertion        | ✅                          |

### Actions multiplexés / Registres

5yy : copie 5 lignes à partir de la ligne actuelle
"ayy : copie la ligne dans le tampon a (a-z)
"ap : colle le contenu du tampon a (a-z)
Astuce : Copier plusieurs lignes dans un tampon : 5"ayy

### Rechercher :
`/mot` → `n`/`N`.  

### Pouvoir de G

#### Supprimer
- :g/motif/d → supprime toutes les lignes contenant motif
- :15,17 g/erreur/d → supprime les lignes 15 à 17 contenant "erreur"

#### Déplacer
- :1,20 g/alerte/m 50 → déplace toutes les lignes contenant "alerte" vers la ligne 50

#### Remplacer
- :30,45 g/demi-dieu/s/Héraclès/Achille/g → dans les lignes 30 à 45, si "demi-dieu" est présent, remplace tous les "Héraclès" par "Achille"
- :g/Nantes/s//Rennes/g → sur tout le fichier, dans chaque ligne contenant "Nantes", remplace "Nantes" par "Rennes"
- :g/Niort/s/^#// → supprime les débuts de ligne commençant par # dans les lignes contenant "Niort" (remplacement par rien)

### Options et personnalisation

#### Options via mode commande
- `:set all` > Affiche toutes les options existantes
- `:set` > Affiche les options actives sur la session
- `:set option?` > Affiche la valeur actuelle d’une option précise.
- `:help` ou `:h` ou `:help option` > Affiche l'aide de VIM
- `:set fileformat` > Affiche le format de fin de ligne actuellement utilisé (unix, dos ou mac)
- `:colorscheme` > Voir les thèmes dispo
- `:colorscheme` > desert Appliquer un thème
	- Tu peux aussi télécharger des thèmes personnalisés et les placer dans ~/.vim/colors

#### Personnalisation
Fichier de config à créer ou éditer : vim ~/.vimrc

- `set expandtab` > Transforme les tabulations en espaces
- `set autoindent` > Active l’indentation automatique à chaque nouvelle ligne
- `set nocompatible` > Active les déplacements et fonctions Linux de VIM
- `set showmode` > Affiche dans la barre du bas le mode actif
- `set number` > Affiche les numéros de ligne.
- `set tabstop=4` > Définit la largeur d’une tabulation à 4 espaces
- `syntax on` > Active la coloration syntaxique selon le langage détecté
- `set fileformat` > Affiche le format de fin de ligne actuellement utilisé (unix, dos ou mac)
- `set fileformat={unix|dos}` > Force les fins de ligne au format Linux (LF)
- `colorscheme nomtheme` > Appliquer un thème permanent

#### Config copier/coller via souris post debian 9
Dans /etc/vim/vimrc.local :

```bash
source /usr/share/vim/vimXX/defaults.vim ➜ récupère les réglages par défaut système
let skip_defaults_vim = 1 ➜ empêche le rechargement en double
if has('mouse') set mouse=r endif ➜ évite que la souris bascule en Visual Mode
set paste ➜ désactive l’auto-indent lors du collage (nickel pour les blocs)
```

- Le chemin /usr/share/vim/vimXX/ dépend de ta version de Vim (`vim --version`).

## Nano (rapide)
- `nano fichier` → éditeur simple.  
- Ctrl+O = sauvegarder,
- Ctrl+X = quitter,
- Ctrl+K = couper,
- Ctrl+U = coller,
- Ctrl+W = recherche,
- Ctrl + \ : Remplacer.

## Conversion CRLF/LF
- `dos2unix` / `unix2dos`.
- Via sed : 
	- `sed 's/\r$//'` → Supprimer les caractères \r (passer de Windows à Linux)
	- `sed 's/$/\r/'` → Ajouter des \r en fin de ligne (passer de Linux à Windows)