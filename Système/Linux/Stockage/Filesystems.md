# 💾 Filesystems Linux — Aide-mémoire

## 🏗️ Concepts Fondamentaux

### Structure Filesystem
```
Filesystem = Structure organisée pour stockage données
├── Superbloc : Métadonnées vitales FS
├── Inodes : Métadonnées fichiers/dossiers
├── Blocs données : Contenu réel
└── Tables allocation : Gestion espace libre
```

### Types de Données
| Élément | Contenu | Localisation |
|---------|---------|--------------|
| **Superbloc** | Config FS, taille, limites | Début partition + copies |
| **Inode** | Métadonnées fichier | Table inodes |
| **Bloc données** | Contenu fichier/dossier | Zone données |
| **Bloc indirection** | Pointeurs gros fichiers | Zone données |

## 📋 Types de Filesystems

### Comparatif Filesystems Linux
| FS | Journalisation | Taille Max Fichier | Taille Max FS | Usage |
|----|----------------|-------------------|---------------|-------|
| **ext2** | ❌ | 2 TiB | 32 TiB | Legacy, /boot |
| **ext3** | ✅ | 2 TiB | 32 TiB | Transition |
| **ext4** | ✅ | 16 TiB | 1 EiB | Standard serveurs |
| **XFS** | ✅ | 8 EiB | 8 EiB | Gros volumes |
| **Btrfs** | ✅ | 16 EiB | 16 EiB | Snapshots, RAID |
| **ZFS** | ✅ | 16 EiB | 256 ZiB | Intégrité données |

### Évolution EXT
```
EXT2 (1993) → EXT3 (2001) → EXT4 (2008)
     ↓              ↓              ↓
 Pas journal   + Journal    + Extents
 Simple        Robuste     Performance
```

## 🔧 Gestion Filesystems

### Création & Formatage
```bash
# Formatage standard
mkfs.ext4 /dev/sdc1              # EXT4 basique
mkfs.xfs /dev/sdc1               # XFS
mkfs.btrfs /dev/sdc1             # Btrfs

# Options avancées EXT4
mkfs.ext4 -L "MonDisk" /dev/sdc1              # Avec label
mkfs.ext4 -b 4096 -i 16384 /dev/sdc1         # Taille bloc + ratio inodes
mkfs.ext4 -m 1 /dev/sdc1                     # 1% réservé root (vs 5% défaut)
mkfs.ext4 -E lazy_itable_init=0 /dev/sdc1    # Init immédiate (plus lent)

# Vérification avant formatage
lsblk /dev/sdc                   # Vérifier partitions
wipefs -a /dev/sdc1              # Nettoyer signatures FS précédentes
```

### Informations Filesystem
```bash
# Identification
blkid                            # UUID, LABEL, TYPE tous FS
blkid /dev/sdc1                  # Infos partition spécifique
lsblk -f                         # Arbre + infos FS
findmnt                          # Points montage actifs

# Détails EXT2/3/4
tune2fs -l /dev/sdc1             # Infos complètes superbloc
dumpe2fs /dev/sdc1               # Détails techniques
e2fsck -n /dev/sdc1              # Check read-only

# État utilisation
df -h                            # Espace utilisé human-readable
df -i                            # Utilisation inodes
du -sh /path                     # Taille dossier
```

## 🔧 Configuration Filesystem

### Tuning EXT4
```bash
# Labeling
tune2fs -L "NouveauLabel" /dev/sdc1

# Paramètres montage par défaut
tune2fs -o journal_data_writeback /dev/sdc1   # Performance (moins sûr)
tune2fs -o ^journal_data_ordered /dev/sdc1    # Désactiver option

# Vérifications automatiques
tune2fs -c 30 /dev/sdc1          # Vérif tous les 30 montages
tune2fs -i 1m /dev/sdc1          # Vérif tous les mois
tune2fs -c 0 -i 0 /dev/sdc1      # Désactiver vérifs auto

# Réservation espace root
tune2fs -m 1 /dev/sdc1           # 1% réservé (vs 5% défaut)
tune2fs -r 1024 /dev/sdc1        # 1024 blocs réservés
```

### Redimensionnement
```bash
# Agrandir (online pour ext4)
resize2fs /dev/sdc1              # Utilise tout l'espace disponible
resize2fs /dev/sdc1 10G          # Taille spécifique

# Réduire (OFFLINE obligatoire)
umount /mnt/disk
e2fsck -f /dev/sdc1              # Vérification obligatoire
resize2fs /dev/sdc1 5G           # Réduire FS AVANT partition
# Puis réduire partition avec fdisk/parted
```

## 🗂️ Structure Inode

### Métadonnées Inode
```bash
# Contenu inode
stat fichier.txt                 # Afficher métadonnées complètes
ls -li fichier.txt               # Numéro inode + détails

# Composants inode
Type fichier     : - d l b c p s (regular, directory, link, block, char, pipe, socket)
Permissions      : rwxrwxrwx (user, group, other)
Liens hardlinks  : Nombre références vers même inode
UID/GID          : Propriétaire / Groupe
Taille           : Octets
Timestamps       : atime, mtime, ctime
Pointeurs blocs  : Vers données (direct + indirect)
```

### Types de Blocs
| Type | Description | Usage |
|------|-------------|-------|
| **Direct** | Pointeur vers bloc données | Petits fichiers (<48KB) |
| **Indirect simple** | Pointeur vers bloc de pointeurs | Fichiers moyens |
| **Indirect double** | Pointeur vers pointeurs de pointeurs | Gros fichiers |
| **Indirect triple** | 3 niveaux indirection | Très gros fichiers |

## 📁 Montage & fstab

### Montage Manuel
```bash
# Montage basique
mount /dev/sdc1 /mnt             # Type auto-détecté
mount -t ext4 /dev/sdc1 /mnt     # Type explicite
mount -L "MonLabel" /mnt         # Par label
mount UUID="123-456" /mnt        # Par UUID (recommandé)

# Options montage courantes
mount -o ro /dev/sdc1 /mnt       # Lecture seule
mount -o rw,noatime /dev/sdc1 /mnt  # R/W sans mise à jour atime
mount -o remount,ro /mnt         # Changer options sans démonter

# Démontage
umount /mnt                      # Par point montage
umount /dev/sdc1                 # Par périphérique
umount -l /mnt                   # Lazy (démonte quand plus utilisé)
fuser -km /mnt                   # Forcer fermeture processus utilisant
```

### Configuration fstab
```bash
# Structure /etc/fstab
# <source> <mountpoint> <fstype> <options> <dump> <pass>

# Exemples pratiques
UUID=123-456-789 /home ext4 defaults,noatime 0 2
LABEL=DataDisk /data xfs defaults,noatime 0 2
/dev/sdb1 none swap sw 0 0
tmpfs /tmp tmpfs defaults,size=1G 0 0

# Options communes
defaults     # rw,suid,dev,exec,auto,nouser,async
noatime      # Pas MAJ temps accès (performance)
relatime     # MAJ atime si plus ancien que mtime
user         # Utilisateurs peuvent monter
nouser       # Seul root peut monter (défaut)
auto         # Montage automatique au boot
noauto       # Montage manuel uniquement
```

### Options Montage Avancées
```bash
# Performance
noatime,nodiratime              # Pas MAJ temps accès
data=writeback                  # Performance max (moins sûr)
data=ordered                    # Compromis (défaut ext4)
data=journal                    # Sécurité max (plus lent)

# Sécurité
noexec                         # Pas exécution binaires
nosuid                         # Ignorer bit SUID
nodev                          # Pas fichiers périphériques

# Réseau
_netdev                        # Attendre réseau (NFS, CIFS)
```

## 🔍 Diagnostic & Maintenance

### Vérification Filesystem
```bash
# Check filesystem (OFFLINE)
fsck /dev/sdc1                  # Auto-détection type
fsck.ext4 /dev/sdc1            # Type spécifique
e2fsck -f /dev/sdc1            # Forcer vérification complète
e2fsck -p /dev/sdc1            # Réparation automatique
e2fsck -n /dev/sdc1            # Read-only (pas modifications)

# Vérification avancée
badblocks -v /dev/sdc1         # Test blocs défectueux
badblocks -w /dev/sdc1         # Test destructif (reformate!)
```

### Analyse Utilisation
```bash
# Espace disque
df -h                          # Résumé utilisation
df -i                          # Utilisation inodes
du -sh /*                      # Taille répertoires racine
ncdu /                         # Interface interactive (si installé)

# Fichiers volumineux
find / -size +100M -type f     # Fichiers >100MB
find / -size +1G -type f -exec ls -lh {} \;  # Fichiers >1GB avec détails

# Inodes
find / -type f | wc -l         # Compter fichiers totaux
tune2fs -l /dev/sdc1 | grep -i inode  # Stats inodes
```

### Monitoring Temps Réel
```bash
# I/O en cours
iotop                          # Monitoring I/O par processus
iostat -x 1                    # Stats I/O détaillées
watch -n 1 'df -h'             # Surveillance espace disque

# Processus utilisant filesystem
lsof /mnt                      # Fichiers ouverts sur /mnt
fuser -v /mnt                  # Processus utilisant /mnt
```

## 🚨 Recovery & Troubleshooting

### Problèmes Courants
| Problème | Symptôme | Solution |
|----------|----------|----------|
| **FS corrompu** | Erreurs I/O, montage échoue | `e2fsck -f /dev/sdX1` |
| **Inodes pleins** | "No space" avec espace libre | Supprimer fichiers ou agrandir |
| **Blocs défectueux** | Erreurs lecture/écriture | `badblocks` + `e2fsck -c` |
| **Montage échoue** | Erreur montage | Vérifier fstab, fsck |

### Recovery Avancé
```bash
# Superbloc de secours
dumpe2fs /dev/sdc1 | grep -i superblock  # Localiser backups
e2fsck -b 32768 /dev/sdc1      # Utiliser superbloc backup
mke2fs -n /dev/sdc1            # Lister superblocs sans formater

# Recovery fichiers supprimés (ext4)
debugfs /dev/sdc1              # Interface debug
# > lsdel                      # Lister inodes supprimés
# > dump <inode> /tmp/recovered # Récupérer fichier

# Clonage partition
dd if=/dev/sdc1 of=/dev/sdd1 conv=noerror,sync  # Clone avec erreurs
ddrescue /dev/sdc1 /dev/sdd1   # Recovery avancé (si installé)
```

## 🔄 Filesystems Spécialisés

### Filesystems Temporaires
```bash
# tmpfs (RAM)
tmpfs /tmp tmpfs defaults,size=1G,mode=1777 0 0     # Dans fstab
mount -t tmpfs -o size=512M tmpfs /tmp              # Manuel

# ramfs (RAM sans swap)
mount -t ramfs ramfs /mnt       # Pas de limite taille (danger!)
```

### Filesystems Réseau
```bash
# NFS
mount -t nfs server:/export /mnt
# Dans fstab: server:/export /mnt nfs defaults,_netdev 0 0

# CIFS/SMB
mount -t cifs //server/share /mnt -o username=user,password=pass
# Dans fstab: //server/share /mnt cifs credentials=/etc/cifs-creds,_netdev 0 0
```

### Filesystems Overlay
```bash
# OverlayFS (Docker/containers)
mount -t overlay overlay -o lowerdir=/lower,upperdir=/upper,workdir=/work /merged
```

## 💡 Bonnes Pratiques

### Performance
- ✅ **noatime** sur filesystems hautes performances
- ✅ **Taille blocs** adaptée : 4KB standard, 64KB+ gros fichiers
- ✅ **Réservation root** : 1% au lieu de 5% sur gros volumes
- ✅ **XFS** pour très gros volumes/fichiers

### Sécurité & Maintenance
- ✅ **UUID** dans fstab (stabilité)
- ✅ **fsck périodique** sur filesystems critiques
- ✅ **Monitoring** espace disque et inodes
- ✅ **Snapshots** avant modifications importantes (LVM/Btrfs)

### Organisation
- ✅ **Séparer** `/`, `/home`, `/var` sur partitions distinctes
- ✅ **Labels explicites** pour identification rapide
- ✅ **Documentation** des montages non-standard
- ✅ **Tests recovery** réguliers sur systèmes critiques

---
**💡 Memo** : UUID dans fstab, fsck avant resize, noatime pour performances, toujours démonter avant fsck !
