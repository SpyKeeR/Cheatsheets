# 💾 Stockage Linux — Aide-mémoire

## 🏗️ Architecture Stockage Linux

### Stack de Stockage
```
Application
    ↓
Filesystem (ext4, xfs, btrfs)
    ↓
Block Device (/dev/sdX, /dev/mapper/*)
    ↓
Volume Manager (LVM, mdadm)
    ↓
Partition Table (MBR, GPT)
    ↓
Physical Device (HDD, SSD, NVMe)
```

### Types de Périphériques
| Type | Convention | Exemple | Description |
|------|------------|---------|-------------|
| **IDE/PATA** | `/dev/hdX` | `/dev/hda1` | Legacy (rare) |
| **SCSI/SATA/USB** | `/dev/sdX` | `/dev/sda1` | Standard actuel |
| **NVMe** | `/dev/nvmeXnYpZ` | `/dev/nvme0n1p1` | SSD haute performance |
| **Virtual** | `/dev/vdX` | `/dev/vda1` | VM (KVM/Xen) |
| **Loop** | `/dev/loopX` | `/dev/loop0` | Fichiers montés |
| **Device Mapper** | `/dev/mapper/*` | `/dev/mapper/vg-lv` | LVM, LUKS, RAID |

## 🗂️ Tables de Partitions

### Comparaison MBR vs GPT
| Aspect | MBR (Legacy) | GPT (Moderne) |
|--------|-------------|---------------|
| **Taille max disque** | 2.2 TiB | 9.4 ZiB |
| **Partitions primaires** | 4 max | 128 par défaut |
| **Partitions étendues** | 1 + logiques | Non nécessaire |
| **Boot** | BIOS | UEFI (+ BIOS legacy) |
| **Redondance** | Aucune | Header + backup |
| **Identifiants** | Types numériques | UUID + noms |

### Structure MBR
```
Secteur 0 (512 octets):
├── Boot Code (446 octets)
├── Partition Table (64 octets = 4×16 octets)
└── Signature (2 octets: 0x55AA)

Types de partitions courantes:
├── 0x83: Linux native
├── 0x8E: Linux LVM
├── 0x82: Linux swap
└── 0xEF: EFI System Partition
```

### Structure GPT
```
Secteur 0: Protective MBR (compatibilité)
Secteur 1: GPT Header
Secteurs 2-33: Partition Entries (128 entrées)
[Données partitions]
Secteurs -33 à -2: Backup Partition Entries
Secteur -1: Backup GPT Header
```

## 🔧 Gestion Partitions

### Outils de Partitionnement
```bash
# fdisk (MBR/GPT, interface menu)
fdisk -l                     # Lister toutes partitions
fdisk /dev/sdb               # Éditer table partitions
   p  # Print table
   n  # New partition
   d  # Delete partition
   t  # Change partition type
   a  # Toggle bootable flag
   w  # Write changes
   q  # Quit without saving

# parted (GPT/MBR, ligne commande)
parted /dev/sdb print        # Afficher table
parted /dev/sdb mklabel gpt  # Créer table GPT
parted /dev/sdb mkpart primary ext4 1MiB 100%  # Créer partition
parted /dev/sdb rm 1         # Supprimer partition 1

# gdisk (GPT spécialisé)
gdisk /dev/sdb               # Interface similaire fdisk
```

### Repartitionnement en Ligne
```bash
# Redimensionner partition sans redémarrage
growpart /dev/sda 2          # Étendre partition 2 (cloud-utils)
resize2fs /dev/sda2          # Étendre filesystem ext4
xfs_growfs /mount/point      # Étendre filesystem XFS

# Vérifier changements
lsblk                        # Arbre périphériques blocs
cat /proc/partitions         # Kernel partition table
partprobe /dev/sdb           # Relire table partitions
```

## 📦 LVM (Logical Volume Manager)

### Concepts LVM
```
Physical Volume (PV) → Volume Group (VG) → Logical Volume (LV)
       ↓                      ↓                     ↓
   Disques/Parts         Pool storage        Volumes finaux
```

### Workflow Complet
```bash
# 1. Préparation physique
fdisk /dev/sdb               # Créer partition type 8E (LVM)
# ou utiliser disque entier sans partitionnement

# 2. Initialisation Physical Volumes
pvcreate /dev/sdb1 /dev/sdc1 # Créer PV sur partitions
pvcreate /dev/sdd            # Ou sur disque entier

# 3. Création Volume Group
vgcreate vg-data /dev/sdb1 /dev/sdc1  # Créer VG
vgextend vg-data /dev/sdd    # Ajouter PV au VG existant

# 4. Création Logical Volumes
lvcreate -n lv-home -L 20G vg-data     # Taille fixe
lvcreate -n lv-var -l 50%VG vg-data    # Pourcentage VG
lvcreate -n lv-tmp -l 100%FREE vg-data # Tout l'espace restant

# 5. Formatage et montage
mkfs.ext4 /dev/vg-data/lv-home
mount /dev/vg-data/lv-home /home
```

### Gestion LVM Avancée
```bash
# Informations et monitoring
pvs, vgs, lvs               # Vue résumée
pvdisplay, vgdisplay, lvdisplay  # Infos détaillées
vgdisplay -v vg-data        # Avec détails PV/LV

# Redimensionnement
lvextend -L +5G /dev/vg-data/lv-home      # Ajouter 5GB
lvextend -l +100%FREE /dev/vg-data/lv-var # Utiliser tout libre
lvreduce -L 10G /dev/vg-data/lv-tmp       # Réduire (ATTENTION!)

# Avec redimensionnement filesystem automatique
lvextend -r -L +5G /dev/vg-data/lv-home   # -r = resize fs

# Snapshots (sauvegarde cohérente)
lvcreate -s -L 1G -n lv-home-snap /dev/vg-data/lv-home
mount /dev/vg-data/lv-home-snap /mnt/snapshot
lvremove /dev/vg-data/lv-home-snap        # Supprimer snapshot
```

### Suppression LVM (Ordre Important!)
```bash
# 1. Démonter filesystems
umount /home
umount /var

# 2. Supprimer LV
lvremove /dev/vg-data/lv-home
lvremove /dev/vg-data/lv-var

# 3. Supprimer VG
vgremove vg-data

# 4. Supprimer PV
pvremove /dev/sdb1 /dev/sdc1
```

## 🔍 Détection & Information

### Exploration Hardware
```bash
# Périphériques blocs
lsblk                        # Arbre périphériques (recommandé)
lsblk -f                     # Avec filesystems et UUID
fdisk -l                     # Liste détaillée partitions
cat /proc/partitions         # Vue kernel des partitions

# Informations disques
lshw -class disk             # Détails hardware disques
hdparm -I /dev/sda           # Informations techniques ATA
smartctl -a /dev/sda         # État SMART du disque
```

### UUID et Labels
```bash
# Identifier uniquement
blkid                        # UUID et labels tous périphériques
blkid /dev/sda1              # Infos partition spécifique
ls -l /dev/disk/by-uuid/     # Liens par UUID
ls -l /dev/disk/by-label/    # Liens par label

# Attribuer labels
e2label /dev/sda1 "MyData"   # Label ext2/3/4
xfs_admin -L "MyData" /dev/sda1  # Label XFS
```

### Monitoring Utilisation
```bash
# Espace disque
df -h                        # Utilisation filesystems
df -i                        # Utilisation inodes
du -sh /var/log              # Taille dossier
ncdu /                       # Interface interactive (si installé)

# I/O en temps réel
iotop                        # Top des processus I/O
iostat -x 1                  # Statistiques I/O détaillées
```

## 💿 Périphériques Virtuels

### Loop Devices
```bash
# Créer et monter image disque
dd if=/dev/zero of=disk.img bs=1M count=100  # Créer image 100MB
mkfs.ext4 disk.img           # Formater image
losetup /dev/loop0 disk.img  # Associer loop device
mount /dev/loop0 /mnt        # Monter

# Gestion automatique
mount -o loop disk.img /mnt  # Mount avec loop automatique
losetup -a                   # Lister loop devices actifs
losetup -d /dev/loop0        # Détacher loop device
```

### Device Mapper
```bash
# Lister mappings actifs
dmsetup ls                   # Liste devices mappés
dmsetup info vg-data-lv-home # Infos device spécifique
dmsetup table vg-data-lv-home # Table de mapping

# États devices
dmsetup status               # État tous devices
ls -l /dev/mapper/           # Devices disponibles
```

## 🔄 Migration & Sauvegarde

### Clonage Disques
```bash
# Clonage complet (secteur par secteur)
dd if=/dev/sda of=/dev/sdb bs=4M status=progress conv=fsync

# Clonage avec récupération erreurs
ddrescue /dev/sda /dev/sdb /tmp/rescue.log

# Image de sauvegarde
dd if=/dev/sda1 of=/backup/sda1.img bs=4M
gzip /backup/sda1.img        # Compression

# Restauration
gunzip -c /backup/sda1.img.gz | dd of=/dev/sda1 bs=4M
```

### Sauvegarde LVM
```bash
# Snapshot pour backup cohérent
lvcreate -s -L 2G -n backup-snap /dev/vg-data/lv-home
dd if=/dev/vg-data/backup-snap of=/backup/home.img bs=4M
lvremove /dev/vg-data/backup-snap

# Export/Import VG (migration)
vgexport vg-data             # Désactiver VG
vgimport vg-data             # Réactiver VG (autre machine)
```

## ⚡ Détection Hardware Dynamique

### Hot-plug Detection
```bash
# Forcer scan nouveaux disques SCSI
echo "- - -" > /sys/class/scsi_host/host0/scan
echo "- - -" > /sys/class/scsi_host/host1/scan

# Rescan partition table
partprobe /dev/sdb           # Relire partitions
udevadm settle               # Attendre fin traitement udev

# Informations udev
udevadm info --query=all --name=/dev/sda
udevadm monitor              # Monitoring événements temps réel
```

### USB & Removable Media
```bash
# Informations USB
lsusb                        # Lister périphériques USB
usb-devices                  # Détails périphériques USB
dmesg | grep -i usb          # Messages kernel USB

# Montage sécurisé amovibles
udisksctl mount -b /dev/sdb1 # Montage utilisateur (sans sudo)
udisksctl unmount -b /dev/sdb1  # Démontage sécurisé
```

## 🚨 Récupération & Dépannage

### Récupération Partitions
```bash
# Tenter récupération table partitions
testdisk /dev/sdb            # Outil interactif récupération
photorec /dev/sdb            # Récupération fichiers par signatures

# Vérification intégrité
badblocks -v /dev/sdb        # Test blocs défectueux (lecture)
badblocks -w -v /dev/sdb     # Test destructif (écriture)
```

### Debug LVM
```bash
# Mode debug LVM
vgchange -ay --verbose vg-data  # Activer VG avec debug
lvchange -ay --verbose /dev/vg-data/lv-home

# Reconstruction métadonnées
vgcfgrestore -l vg-data      # Lister sauvegardes config
vgcfgrestore vg-data         # Restaurer dernière config
pvscan --cache               # Reconstruire cache PV
```

### Situations d'Urgence
```bash
# Activation forcée LVM
vgchange -ay --partial vg-data  # Activer même si PV manquant
lvchange -ay --partial /dev/vg-data/lv-home

# Mode read-only filesystem
mount -o remount,ro /         # Basculer en lecture seule
fsck.ext4 -f /dev/sda1       # Vérifier filesystem
mount -o remount,rw /        # Rebascule en écriture
```

## 💡 Bonnes Pratiques

### Planification Stockage
- ✅ **Utiliser GPT** pour nouveaux systèmes
- ✅ **LVM par défaut** pour flexibilité future
- ✅ **Séparer** `/`, `/home`, `/var` sur LV différents
- ✅ **Prévoir snapshots** LVM pour sauvegardes cohérentes
- ✅ **Monitoring** SMART des disques

### Sécurité & Performance
- ✅ **UUID dans fstab** pour stabilité
- ✅ **Alignment** partitions (début sur 1MiB)
- ✅ **Réserver espace** LVM pour snapshots (10-20%)
- ✅ **Labels explicites** pour identification
- ✅ **Documentation** architecture de stockage

### Maintenance Régulière
- ✅ **Vérifier** état SMART mensuel
- ✅ **Tester** procédures restauration
- ✅ **Nettoyer** snapshots anciens
- ✅ **Surveiller** utilisation espace
- ✅ **Planifier** extension avant saturation

---
**💡 Memo** : LVM = PV→VG→LV, toujours UUID dans fstab, snapshots pour backup cohérents !



