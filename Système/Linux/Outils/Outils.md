# 🧰 Outils Linux Essentiels — Aide-mémoire

## 🔍 Recherche & Filtrage

### Grep & Variants
```bash
# grep classique
grep "pattern" fichier          # Recherche basique
grep -i "pattern" fichier       # Insensible à la casse
grep -r "pattern" /path         # Récursif dans répertoires
grep -v "pattern" fichier       # Inverser (tout sauf pattern)
grep -n "pattern" fichier       # Avec numéros de ligne
grep -l "pattern" *.txt         # Noms fichiers seulement
grep -A 3 -B 3 "pattern" file   # 3 lignes avant/après contexte

# Patterns multiples
grep -e "pat1" -e "pat2" file   # OU logique
grep -E "(pat1|pat2)" file      # Regex étendue
grep -F "literal.string" file   # Chaîne littérale (pas regex)

# ripgrep (moderne, rapide)
rg "pattern"                    # Recherche récursive intelligente
rg -i "pattern"                 # Insensible casse
rg -t py "pattern"              # Filtrer par type fichier
rg "pattern" --type-not log     # Exclure types
rg -A 3 -B 3 "pattern"          # Contexte avant/après
```

### Find & Locate
```bash
# find standard
find /path -name "*.txt"        # Par nom fichier
find /path -type f -size +100M  # Fichiers > 100MB
find /path -mtime -7            # Modifiés < 7 jours
find /path -exec ls -l {} \;    # Exécuter commande sur résultats

# fd (moderne, simple)
fd "pattern"                    # Recherche nom fichier
fd -t f "pattern"               # Files seulement
fd -t d "pattern"               # Directories seulement
fd -e txt                       # Extension spécifique
fd -H "pattern"                 # Inclure fichiers cachés

# locate (base données)
locate filename                 # Recherche rapide nom
updatedb                        # Mettre à jour base (root)
```

## ✂️ Traitement Texte

### Cut, Sort, Uniq
```bash
# cut - Extraction colonnes
cut -d: -f1 /etc/passwd         # 1ère colonne, délimiteur :
cut -c1-10 file                 # Caractères 1 à 10
cut -d, -f2,4 file.csv          # Colonnes 2 et 4

# sort - Tri
sort file                       # Tri alphabétique
sort -n file                    # Tri numérique
sort -r file                    # Ordre inverse
sort -k2 file                   # Trier par 2ème colonne
sort -u file                    # Tri + suppression doublons

# uniq - Doublons (fichier doit être trié)
uniq file                       # Supprimer doublons adjacents
uniq -c file                    # Compter occurrences
uniq -d file                    # Doublons seulement
```

### Sed & Awk
```bash
# sed - Éditeur flux
sed 's/old/new/' file           # Remplacer 1ère occurrence
sed 's/old/new/g' file          # Remplacer toutes occurrences
sed -i 's/old/new/g' file       # Modifier fichier en place
sed '/pattern/d' file           # Supprimer lignes contenant pattern
sed -n '1,5p' file              # Afficher lignes 1 à 5

# awk - Traitement structuré
awk '{print $1}' file           # Première colonne
awk -F: '{print $1}' /etc/passwd # Délimiteur personnalisé
awk 'NR>1 {print}' file         # Ignorer première ligne
awk '$3 > 100' file             # Lignes où 3ème colonne > 100
awk '{sum+=$1} END {print sum}' # Somme première colonne
```

## 📦 Archives & Compression

### Tar (Standard)
```bash
# Création archives
tar cf archive.tar files/       # Créer archive
tar czf archive.tar.gz files/   # Avec compression gzip
tar cjf archive.tar.bz2 files/  # Avec compression bzip2
tar cJf archive.tar.xz files/   # Avec compression xz

# Extraction
tar xf archive.tar              # Extraire
tar xzf archive.tar.gz          # Extraire gzip
tar xf archive.tar -C /dest/    # Extraire vers destination

# Consultation
tar tf archive.tar              # Lister contenu
tar tzf archive.tar.gz          # Lister contenu gzip
```

### Autres Formats
```bash
# ZIP
zip -r archive.zip directory/   # Créer ZIP
unzip archive.zip               # Extraire ZIP
unzip -l archive.zip            # Lister contenu

# 7zip (si installé)
7z a archive.7z files/          # Créer 7z
7z x archive.7z                 # Extraire 7z

# Outil universel
unp archive.*                   # Extraire selon extension
```

## 🔧 Outils Modernes

### Navigation & Exploration
```bash
# exa - ls amélioré
exa -la                         # Liste détaillée
exa --tree                      # Vue arbre
exa --git                       # Status git intégré
exa -T -L 2                     # Arbre niveau 2

# zoxide - cd intelligent
z directory                     # Jump vers directory
zi                              # Interactive selector
zoxide add /path                # Ajouter path manuellement

# ranger - Explorateur fichiers
ranger                          # Interface TUI
ranger --choosedir=/tmp/cwd     # Changer répertoire shell
```

### Monitoring & Système
```bash
# duf - df moderne
duf                             # Usage disques coloré
duf /home /var                  # Paths spécifiques

# ncdu - du interactif
ncdu /path                      # Analyse espace disque
ncdu -x                         # Ne pas traverser filesystems

# htop/btop - top amélioré
htop                            # Processus interactif
btop                            # Interface moderne graphique

# procs - ps moderne
procs                           # Processus avec couleurs
procs --tree                    # Vue arborescence
procs firefox                   # Filtrer par nom

# dust - du rapide
dust                            # Taille répertoires
dust -n 10                      # Top 10 seulement
```

### Réseau & Transfer
```bash
# mosh - SSH robuste
mosh user@server                # SSH avec reconnexion auto

# magic-wormhole - Transfer P2P
wormhole send file.txt          # Envoyer fichier
wormhole receive 1-code-word    # Recevoir avec code

# termshark - Wireshark TUI
termshark -i eth0               # Capture interface
termshark -r capture.pcap       # Lire fichier capture

# iftop - Trafic réseau temps réel
iftop -i eth0                   # Interface spécifique
iftop -n                        # Pas de résolution DNS
```

## 🎛️ Utilitaires Pipeline

### Watch & Repeat
```bash
# watch - Répéter commande
watch "df -h"                   # Répéter toutes les 2s
watch -n 1 "ps aux"             # Intervalle 1 seconde
watch -d "ls -la"               # Highlight différences

# Alternatives modernes
viddy "command"                 # watch amélioré (si installé)
```

### FZF - Fuzzy Finder
```bash
# Utilisation standalone
ls | fzf                        # Sélection interactive
find . -type f | fzf            # Chercher fichier

# Intégration shell (si configuré)
**<TAB>                         # Completion fuzzy
Ctrl+R                          # Historique fuzzy
Ctrl+T                          # Fichiers fuzzy
Alt+C                           # cd fuzzy

# Dans pipes
ps aux | fzf                    # Sélectionner processus
docker ps | fzf                # Sélectionner container
```

### Autres Utilitaires Pipeline
```bash
# ts - Timestamps
command | ts                    # Ajouter horodatage
command | ts '[%Y-%m-%d %H:%M:%S]' # Format personnalisé

# vipe - Éditer dans pipeline
command | vipe | other_command  # Éditer résultat intermédiaire

# pv - Progress viewer
pv bigfile | command            # Barre progression
dd if=/dev/zero bs=1M count=100 | pv | dd of=/tmp/test

# parallel - Parallélisation
parallel echo {} ::: 1 2 3 4    # Exécuter en parallèle
ls *.jpg | parallel convert {} {.}.png # Conversion batch
```

## 🗂️ Productivité & Organisation

### Taskwarrior - Gestion Tâches
```bash
# Tâches de base
task add "Faire rapport"        # Ajouter tâche
task list                       # Lister tâches
task 1 done                     # Marquer terminée
task 1 delete                   # Supprimer tâche

# Organisation
task add "Réunion" due:tomorrow # Avec échéance
task add "Bug fix" project:web  # Avec projet
task 1 modify priority:H        # Modifier priorité
```

### Terminal Recording
```bash
# asciinema - Enregistrer terminal
asciinema rec session.cast      # Enregistrer session
asciinema play session.cast     # Rejouer session
asciinema upload session.cast   # Uploader (asciinema.org)

# script - Alternative basique
script session.log              # Enregistrer session
exit                            # Arrêter enregistrement
```

## 🐳 Docker & Containers

### Lazydocker - Interface TUI
```bash
lazydocker                      # Interface graphique containers
# Navigation: hjkl ou flèches
# Actions: contextuelles selon sélection
```

### Portainer Alternative CLI
```bash
# dive - Analyser images
dive image:tag                  # Explorer layers image

# ctop - top pour containers
ctop                            # Monitoring containers temps réel
```

## 🔐 Sécurité & Nettoyage

### Suppression Sécurisée
```bash
# shred - Suppression multiple passes
shred -v -n 3 -z fichier        # 3 passes + zéros final
shred -vfz /dev/sdX             # Disque entier (DANGER!)

# wipe - Alternative (si installé)
wipe -rf directory/             # Suppression récursive sécurisée
```

### Informations Système
```bash
# lshw - Hardware détaillé
lshw                            # Liste complète (root)
lshw -short                     # Format court
lshw -class network             # Catégorie spécifique

# inxi - Info système
inxi -Fxz                       # Infos complètes anonymisées
inxi -b                         # Infos basiques
inxi -N                         # Réseau seulement
```

## 💡 JSON & APIs

### jq - Processeur JSON
```bash
# Basique
echo '{"name":"John","age":30}' | jq '.name'           # Extraire champ
curl api.url | jq '.[0].field'                        # Premier élément tableau

# Filtrage
curl api.url | jq '.[] | select(.status == "active")' # Filtrer par condition
curl api.url | jq 'map(.name)'                        # Extraire noms seulement

# Transformation
echo '[{"a":1},{"a":2}]' | jq 'map(.a) | add'        # Somme valeurs
```

## 🎯 Installation & Gestion

### Gestionnaires Paquets Modernes
```bash
# Flatpak - Applications sandbox
flatpak install app.name        # Installer app
flatpak run app.name            # Lancer app
flatpak update                  # Mettre à jour toutes

# Snap - Packages universels
snap install app                # Installer app
snap list                       # Lister installées
snap refresh                    # Mettre à jour

# AppImage - Portables
chmod +x app.AppImage           # Rendre exécutable
./app.AppImage                  # Lancer directement
```

### Outils Installation
```bash
# curl/wget - Téléchargement
curl -fsSL url | sh             # Installation script sécurisée
wget -qO- url | bash            # Alternative wget

# git - Sources
git clone --depth 1 repo.git   # Clone shallow (plus rapide)
```

## 📚 Aide & Documentation

### Commandes d'Aide
```bash
# Standards
man command                     # Manuel complet
command --help                  # Aide courte
info command                    # Documentation GNU

# Modernes
tldr command                    # Exemples pratiques (si installé)
cheat command                   # Cheatsheets (si installé)
```

### Exploration Commandes
```bash
# which/type - Localiser commandes
which command                   # Chemin exécutable
type command                    # Type commande (builtin/alias/file)

# apropos - Recherche manuels
apropos keyword                 # Chercher dans descriptions manuels
man -k keyword                  # Équivalent apropos
```

---
**💡 Memo** : Les outils modernes (rg, fd, exa, zoxide) améliorent l'expérience mais les classiques restent universels !