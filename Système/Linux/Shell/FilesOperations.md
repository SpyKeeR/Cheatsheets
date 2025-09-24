# 📁 Opérations Fichiers Linux — Aide-mémoire

## 🏗️ Système de Fichiers & Inodes

### Concepts Fondamentaux
- **Inode** : Structure contenant métadonnées fichier (permissions, propriétaire, taille, dates, pointeurs blocs)
- **Chemin** : Route vers fichier/dossier (`/home/user/file.txt`)
- **Liens** : Références vers même données (hard/soft links)
- **VFS** : Virtual File System - abstraction unifiée des différents FS

### Types de Fichiers (ls -l)
| Symbole | Type | Description |
|---------|------|-------------|
| `-` | Fichier régulier | Documents, binaires, données |
| `d` | Directory | Répertoire/dossier |
| `l` | Symbolic link | Lien symbolique (raccourci) |
| `b` | Block device | Périphérique bloc (disques) |
| `c` | Character device | Périphérique caractère (terminaux) |
| `p` | Named pipe | FIFO (communication processus) |
| `s` | Socket | Communication réseau locale |

### Attributs Temporels
```bash
# Dates fichiers
stat fichier.txt                    # Infos complètes inode
ls -l --time=atime fichier.txt      # Dernier accès
ls -l --time=ctime fichier.txt      # Dernière modif métadonnées

# Modification dates
touch fichier.txt                   # MAJ timestamp actuel
touch -t 202401151430 fichier.txt   # Date spécifique (YYYYMMDDhhmm)
touch -r ref.txt nouveau.txt        # Copier dates de référence
```

## 🧭 Navigation & Exploration

### Déplacement
```bash
# Navigation de base
pwd                                 # Répertoire courant
cd /path/to/directory              # Chemin absolu
cd ../parent                       # Chemin relatif
cd ~                              # Répertoire home
cd -                              # Répertoire précédent

# Historique
pushd /path && popd               # Stack de répertoires
dirs -v                          # Afficher stack
```

### Listing (ls)
```bash
# Options essentielles
ls -la                           # Long format + fichiers cachés
ls -lh                           # Tailles human-readable
ls -lt                           # Tri par date modification
ls -ltr                          # Tri par date (ancien → récent)
ls -lS                           # Tri par taille
ls -ld directory/                # Info dossier (pas contenu)
ls -R                            # Récursif sous-dossiers

# Filtrage & sélection
ls *.txt                         # Globbing simple
ls [Ff]ile*                      # Plage caractères
ls -I "*.tmp"                    # Ignorer pattern
ls --color=always                # Forcer couleurs
```

### Exploration Avancée
```bash
# Tree view (si installé)
tree                             # Arborescence
tree -L 2                        # Limiter niveaux
tree -a                          # Inclure fichiers cachés
tree -d                          # Dossiers seulement

# Alternatives natives
find . -type d | head -20        # Dossiers avec find
ls -R | grep "^\./"              # Simulation tree basique
```

## 📋 Manipulation Fichiers

### Copie (cp)
```bash
# Copie basique
cp source.txt destination.txt         # Fichier simple
cp file1 file2 file3 /target/dir/    # Multiple vers dossier
cp -r directory/ /backup/             # Récursif (dossiers)

# Options utiles
cp -p source dest                     # Préserver permissions/dates
cp -u source dest                     # Update (si plus récent)
cp -i source dest                     # Interactif (demande confirmation)
cp -v source dest                     # Verbeux (affiche actions)
cp -n source dest                     # No-clobber (pas d'écrasement)

# Copie avancée
cp -a directory/ backup/              # Archive (équivaut -dpR)
cp --backup=numbered old new          # Backup avec numérotation
rsync -av source/ dest/               # Alternative robuste
```

### Déplacement/Renommage (mv)
```bash
# Usages courants
mv oldname newname                    # Renommer fichier
mv file1 file2 directory/             # Déplacer vers dossier
mv *.txt /documents/                  # Déplacer par pattern

# Options sécurité
mv -i source dest                     # Demander avant écrasement
mv -n source dest                     # Pas d'écrasement
mv -v source dest                     # Mode verbeux
mv --backup=simple old new            # Créer backup avant
```

### Suppression (rm)
```bash
# Suppression standard
rm fichier.txt                       # Fichier simple
rm file1 file2 file3                 # Fichiers multiples
rm -r directory/                     # Récursif (dossiers + contenu)
rm -rf /path/dangerous/              # Force + récursif (DANGER!)

# Options sécurité
rm -i fichier.txt                    # Demander confirmation
rm -v fichier.txt                    # Afficher suppressions
rm -I *.txt                          # Confirm si >3 fichiers ou récursif

# Alternatives sécurisées
trash fichier.txt                    # Vers corbeille (si installé)
mv fichier.txt ~/.trash/             # Simulation corbeille
```

## 🔗 Gestion des Liens

### Liens Physiques (Hard Links)
```bash
# Création & propriétés
ln source.txt hardlink.txt           # Créer lien physique
ls -li                               # Voir numéros inodes (-i)
stat source.txt hardlink.txt         # Vérifier même inode

# Limitations
# ❌ Pas de liens hard vers répertoires
# ❌ Pas de liens hard trans-partition
# ✅ Fichier persiste tant qu'un lien existe
```

### Liens Symboliques (Soft Links)
```bash
# Création
ln -s /path/to/source symlink        # Lien absolu (recommandé)
ln -s ../relative/path symlink       # Lien relatif
ln -sf new_target existing_link     # Force remplacement

# Gestion
readlink symlink                     # Voir cible du lien
readlink -f symlink                  # Résoudre chemin absolu
ls -l                                # Affiche cible (→)
file symlink                         # Type et cible

# Réparation liens cassés
find . -type l -exec test ! -e {} \; -print  # Trouver liens cassés
```

### Comparaison Liens
| Aspect | Hard Link | Symbolic Link |
|--------|-----------|---------------|
| **Inode** | Même que source | Propre inode |
| **Suppression source** | Lien fonctionne | Lien cassé |
| **Répertoires** | Non supporté | Supporté |
| **Cross-filesystem** | Non | Oui |
| **Visibilité** | Identique au fichier | Visible avec ls -l |

## 🔍 Recherche & Localisation

### find (Recherche Temps Réel)
```bash
# Par nom
find /path -name "*.txt"             # Extension
find . -name "*config*"              # Contient mot
find . -iname "*.PDF"                # Insensible casse

# Par type
find . -type f                       # Fichiers seulement
find . -type d                       # Dossiers seulement
find . -type l                       # Liens symboliques

# Par taille
find . -size +100M                   # Plus de 100MB
find . -size -1k                     # Moins de 1KB
find . -size 50c                     # Exactement 50 bytes

# Par date
find . -mtime -7                     # Modifiés derniers 7 jours
find . -atime +30                    # Accédés il y a plus de 30j
find . -ctime -1                     # Métadonnées changées dernière 24h
find . -newer reference.txt          # Plus récents que référence

# Par permissions/propriétaires
find . -perm 755                     # Permissions exactes
find . -perm -644                    # Au moins ces permissions
find . -user john                    # Propriétaire spécifique
find . -group admin                  # Groupe spécifique
```

### find avec Actions
```bash
# Actions simples
find . -name "*.tmp" -delete         # Supprimer fichiers trouvés
find . -name "*.log" -exec gzip {} \; # Compresser chaque fichier
find . -type f -exec chmod 644 {} \; # Fixer permissions fichiers

# Actions interactives
find . -name "*.bak" -ok rm {} \;    # Demander avant suppression
find . -size +100M -ok mv {} /big/ \; # Confirmer déplacements

# Actions groupées (plus efficace)
find . -name "*.txt" -exec mv {} /texts/ +  # Déplacer par lots
find . -type f -print0 | xargs -0 chmod 644 # Via xargs (gros volumes)
```

### locate (Index Pre-construit)
```bash
# Recherche rapide
locate filename.txt                  # Recherche dans index
locate -i config                    # Insensible casse
locate -r '\.conf$'                  # Regex POSIX
locate -c config                     # Compter résultats seulement

# Gestion base de données
sudo updatedb                        # Mettre à jour index
locate -S                           # Statistiques base
```

### Différences find vs locate
| Aspect | find | locate |
|--------|------|--------|
| **Vitesse** | Lent (temps réel) | Rapide (index) |
| **Actualité** | Temps réel | Dépend updatedb |
| **Filtrage** | Très avancé | Basique |
| **Disponibilité** | Toujours | Dépend du système |

## 📖 Lecture & Consultation

### Affichage Complet
```bash
# Fichiers entiers
cat fichier.txt                      # Affichage brut
cat -n fichier.txt                   # Avec numéros lignes
cat -A fichier.txt                   # Caractères non-imprimables
tac fichier.txt                      # Ordre inverse (dernière ligne → première)
```

### Pagination (more/less)
```bash
# more (basique)
more fichier.txt                     # Pagination simple
more -s fichier.txt                  # Squeeze (réduire lignes vides)

# less (avancé, recommandé)
less fichier.txt                     # Navigation complète
less +F fichier.txt                  # Mode follow (comme tail -f)
less +G fichier.txt                  # Aller fin de fichier

# Navigation dans less:
# j/k : ligne suivante/précédente
# espace/b : page suivante/précédente  
# g/G : début/fin fichier
# /pattern : recherche
# n/N : occurrence suivante/précédente
# q : quitter
```

### Début/Fin de Fichier
```bash
# head (début)
head fichier.txt                     # 10 premières lignes (défaut)
head -n 20 fichier.txt               # 20 premières lignes
head -c 100 fichier.txt              # 100 premiers caractères
head -n -5 fichier.txt               # Tout sauf 5 dernières lignes

# tail (fin)
tail fichier.txt                     # 10 dernières lignes
tail -n 20 fichier.txt               # 20 dernières lignes
tail -f fichier.log                  # Suivi temps réel
tail -F fichier.log                  # Suivi + recréation fichier
tail -n +10 fichier.txt              # À partir ligne 10
```

### Suivi de Fichiers (Logs)
```bash
# Surveillance temps réel
tail -f /var/log/syslog              # Suivi standard
tail -f file1.log file2.log          # Multiples fichiers
tail -F /var/log/apache/*.log        # Suivi avec rotation

# Alternatives
less +F fichier.log                  # less en mode follow
watch -n 1 'tail -10 fichier.log'   # Rafraîchissement périodique
multitail fichier1.log fichier2.log # Suivi parallèle (si installé)
```

## 📊 Analyse & Statistiques

### Comptage (wc)
```bash
# Options de base
wc fichier.txt                       # Lignes, mots, caractères
wc -l fichier.txt                    # Lignes seulement
wc -w fichier.txt                    # Mots seulement
wc -c fichier.txt                    # Octets (bytes)
wc -m fichier.txt                    # Caractères (multibyte-aware)

# Utilisation pratique
find . -name "*.txt" | wc -l         # Compter fichiers
ps aux | wc -l                       # Compter processus
history | wc -l                      # Commandes en historique
```

### Informations Détaillées
```bash
# stat (métadonnées complètes)
stat fichier.txt                     # Infos inode complètes
stat -c "%n %s %y" fichier.txt       # Format personnalisé
stat --format="%n: %s bytes" *.txt   # Format sur multiple fichiers

# file (type de contenu)
file fichier.txt                     # Type MIME + description
file -b fichier.txt                  # Description brève
file -i fichier.txt                  # Type MIME seulement
file /dev/sda1                       # Type périphérique
```

## 📁 Création & Suppression Dossiers

### mkdir (Création)
```bash
# Création simple
mkdir nouveau_dossier                # Dossier simple
mkdir dir1 dir2 dir3                 # Multiples dossiers
mkdir -p path/to/nested/dir          # Arborescence complète (-p = parents)

# Options avancées
mkdir -m 755 dossier                 # Avec permissions spécifiques
mkdir -v dossier                     # Mode verbeux
mkdir -p project/{src,docs,tests}    # Expansion accolades
```

### rmdir (Suppression Dossiers Vides)
```bash
# Suppression sécurisée (vides seulement)
rmdir dossier_vide                   # Dossier vide uniquement
rmdir -p path/to/empty/dirs          # Récursif si tous vides
rmdir -v dossier                     # Mode verbeux

# Alternative pour non-vides
rm -rf dossier_avec_contenu          # ATTENTION: Tout supprimé!
```

## 🔧 Outils Avancés

### Gestion Espace Disque
```bash
# du (disk usage)
du -h dossier/                       # Taille dossier human-readable
du -sh dossier/                      # Résumé seulement (-s)
du -ah dossier/                      # Inclure fichiers (-a)
du -h --max-depth=1                  # Limiter profondeur
du -h | sort -rh                     # Tri par taille décroissante

# df (filesystem usage)
df -h                                # Utilisation partitions
df -i                                # Utilisation inodes
df -T                                # Inclure type filesystem
```

### Comparaison Fichiers
```bash
# diff (différences)
diff file1.txt file2.txt             # Différences standard
diff -u file1.txt file2.txt          # Format unifié
diff -r dir1/ dir2/                  # Comparaison dossiers
diff -q file1 file2                  # Quiet (différent ou pas)

# Autres outils
cmp file1 file2                      # Comparaison binaire
md5sum file1 file2                   # Vérifier intégrité
sha256sum fichier.txt                # Hash sécurisé
```

### Synchronisation
```bash
# rsync (synchronisation avancée)
rsync -av source/ destination/       # Archive + verbeux
rsync -av --delete src/ dst/         # Supprimer fichiers en trop
rsync -av --progress src/ dst/       # Barre progression
rsync -av -e ssh src/ user@host:dst/ # Via SSH
```

## 💡 Bonnes Pratiques

### Sécurité
- ✅ **Toujours** utiliser `-i` avec `rm`/`mv` pour fichiers importants
- ✅ **Tester** commandes `find` avec `-print` avant `-delete`
- ✅ **Sauvegarder** avant suppression massive
- ✅ **Utiliser** chemins absolus pour scripts automatisés

### Performance
- ✅ **Préférer** `locate` à `find` pour recherches simples
- ✅ **Utiliser** `rsync` plutôt que `cp` pour gros volumes
- ✅ **Limiter** profondeur avec `find -maxdepth`
- ✅ **Grouper** actions `find` avec `+` plutôt que `\;`

### Organisation
- ✅ **Nommer** fichiers/dossiers explicitement
- ✅ **Éviter** espaces dans noms (ou quoter)
- ✅ **Utiliser** extensions cohérentes
- ✅ **Documenter** arborescence complexe

---
**💡 Memo** : `find` pour recherche avancée, `locate` pour rapidité, toujours `ls -la` pour debug permissions !