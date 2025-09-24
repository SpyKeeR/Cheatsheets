# ⚙️ Boot, Installation & Démarrage Linux — Aide-mémoire

## 🔌 Processus de Démarrage Complet

### Pipeline de Boot
1. **POST** (Power-On Self Test) → Vérification hardware
2. **Firmware** (BIOS/UEFI) → Initialisation système
3. **Bootloader** (GRUB/systemd-boot) → Chargement OS  
4. **Kernel** (vmlinuz) → Cœur système
5. **Init System** (systemd/OpenRC) → Services utilisateur

### Firmware : BIOS vs UEFI

| Aspect | BIOS (Legacy) | UEFI (Moderne) |
|--------|---------------|----------------|
| **Architecture** | 16-bit, mode réel | 32/64-bit, mode protégé |
| **Partition** | MBR (4 primaires max) | GPT (128 partitions) |
| **Taille disque** | 2TB maximum | >9 ZB supporté |
| **Boot** | MBR (512 octets) | Partition EFI (FAT32) |
| **Sécurité** | Limitée | Secure Boot |
| **Interface** | Texte basique | Graphique + souris |

```
BIOS Boot:
POST → MBR (446 bytes bootloader) → GRUB Stage 1.5 → GRUB Stage 2

UEFI Boot:  
POST → EFI System Partition → bootx64.efi → OS Boot Manager
```

## 🛠️ Installation & Préparation

### Outils de Création Média
| Outil | Plateforme | Spécialité |
|-------|------------|------------|
| **Ventoy** | Multi-OS | Multi-ISO sur une clé |
| **Rufus** | Windows | Simple et rapide |
| **Balena Etcher** | Cross-platform | Validation automatique |
| **dd** | Linux/Unix | Commande native |

```bash
# Création média USB (Linux)
sudo dd if=debian.iso of=/dev/sdX bs=4M status=progress sync
sudo cp debian.iso /dev/sdX && sync    # Alternative simple

# Vérification intégrité
sha256sum debian.iso                    # Comparer avec somme officielle
```

### Types d'Installation

#### Installation Interactive
- **Standard** : Assistant graphique/texte
- **Expert** : Contrôle granulaire options
- **Rescue** : Mode dépannage système

#### Installation Automatisée
```bash
# Preseed (Debian/Ubuntu)
auto url=http://server/preseed.cfg

# Kickstart (Red Hat/CentOS)
inst.ks=http://server/ks.cfg

# Cloud-init (Cloud)
user-data: |
  #cloud-config
  users:
    - name: admin
      sudo: ALL=(ALL) NOPASSWD:ALL
```

## 💾 Partitionnement & Stockage

### Schémas de Partitionnement

#### BIOS/MBR (Legacy)
```
/dev/sda1    /boot     ext4    500MB   (bootable)
/dev/sda2    /         ext4    20GB    
/dev/sda3    /home     ext4    reste
/dev/sda4    swap      swap    RAM×1.5
```

#### UEFI/GPT (Moderne)
```
/dev/sda1    /boot/efi  vfat    512MB   (EF00 - EFI System)
/dev/sda2    /boot      ext4    1GB     (boot loader files)
/dev/sda3    /          ext4    30GB    (root filesystem)  
/dev/sda4    /home      ext4    reste   (user data)
/dev/sda5    swap       swap    RAM×1   (Linux swap)
```

### Systèmes de Fichiers
| FS | Usage | Avantages | Inconvénients |
|----|-------|-----------|---------------|
| **ext4** | Standard Linux | Stable, éprouvé | Pas de snapshots natifs |
| **xfs** | Gros volumes | Performance, scalabilité | Difficile à réduire |
| **btrfs** | Moderne | Snapshots, compression | Complexité, bugs |
| **zfs** | Enterprise | Intégrité, RAID-Z | Licence, RAM gourmand |
| **fat32** | EFI/Boot | Compatibilité universelle | Taille fichier 4GB max |

## 🥾 GRUB Bootloader

### Architecture GRUB2
```
Stage 1:     boot.img (MBR/PBR) - 446 bytes
Stage 1.5:   core.img (/boot/grub/) - modules essentiels  
Stage 2:     /boot/grub/ - configuration complète
```

### Configuration GRUB
```bash
# Fichiers de configuration (Ne pas éditer grub.cfg directement!)
/etc/default/grub                   # Paramètres principaux
/etc/grub.d/                       # Scripts génération menu
├── 00_header                      # En-tête configuration
├── 10_linux                       # Détection kernels Linux
├── 30_os-prober                   # Détection autres OS
└── 40_custom                      # Entrées personnalisées

# Régénération configuration
sudo update-grub                   # Debian/Ubuntu
sudo grub-mkconfig -o /boot/grub/grub.cfg  # Générique
```

### Paramètres /etc/default/grub
```bash
GRUB_DEFAULT=0                     # Entrée par défaut (numéro)
GRUB_TIMEOUT=5                     # Délai attente (secondes)
GRUB_DISTRIBUTOR="Debian"          # Nom distribution
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"  # Paramètres kernel par défaut
GRUB_CMDLINE_LINUX=""              # Paramètres kernel toujours
GRUB_DISABLE_RECOVERY="true"       # Désactiver mode recovery
```

### Installation/Réparation GRUB
```bash
# Installation GRUB (BIOS)
sudo grub-install /dev/sda

# Installation GRUB (UEFI)
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi

# Réparation depuis Live CD
sudo mount /dev/sda2 /mnt          # Monter système
sudo mount /dev/sda1 /mnt/boot/efi # Monter EFI (si UEFI)
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc  
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
grub-install /dev/sda && update-grub
```

## 🎯 Systemd Init System

### Targets (Niveaux d'Exécution)
| Target | Equivalent SysV | Description |
|--------|-----------------|-------------|
| **poweroff.target** | 0 | Arrêt système |
| **rescue.target** | 1 | Mode maintenance (single-user) |
| **multi-user.target** | 3 | Multi-utilisateur sans graphique |
| **graphical.target** | 5 | Multi-utilisateur avec graphique |
| **reboot.target** | 6 | Redémarrage |
| **emergency.target** | - | Shell d'urgence minimal |

### Gestion Services
```bash
# État et contrôle services
systemctl status apache2           # État détaillé service
systemctl start apache2            # Démarrer service
systemctl stop apache2             # Arrêter service  
systemctl restart apache2          # Redémarrer service
systemctl reload apache2           # Recharger configuration

# Activation/désactivation auto-start
systemctl enable apache2           # Activer au boot
systemctl disable apache2          # Désactiver au boot
systemctl is-enabled apache2       # Vérifier état activation

# Masquage (empêche démarrage)
systemctl mask apache2             # Masquer service
systemctl unmask apache2           # Démasquer service
```

### Gestion Targets
```bash
# Targets système
systemctl get-default              # Target par défaut
systemctl set-default multi-user.target  # Changer target par défaut
systemctl isolate rescue.target    # Passer en mode maintenance
systemctl isolate graphical.target # Passer en mode graphique

# Listing et états
systemctl list-units               # Services actifs
systemctl list-units --all         # Tous services
systemctl list-units --failed      # Services en échec
systemctl list-unit-files          # Tous fichiers unités
```

### Types d'Unités Systemd
| Type | Extension | Description | Exemple |
|------|-----------|-------------|---------|
| **Service** | .service | Démon/processus | apache2.service |
| **Target** | .target | Groupe d'unités | graphical.target |
| **Socket** | .socket | Communication IPC/réseau | ssh.socket |
| **Mount** | .mount | Point de montage | home.mount |
| **Device** | .device | Périphérique | dev-sda1.device |
| **Timer** | .timer | Tâche planifiée | backup.timer |

## 🆘 Mode Rescue & Dépannage

### Accès Mode Maintenance
```bash
# Depuis GRUB (éditer entrée avec 'e')
# Ajouter à la ligne kernel:
init=/bin/bash                     # Shell root direct (pas de services)
systemd.unit=rescue.target         # Mode rescue avec services minimaux  
systemd.unit=emergency.target      # Mode urgence (FS en lecture seule)
single                             # Single-user mode traditionnel
```

### Réparations Courantes
```bash
# Remonter système en écriture
mount -o remount,rw /

# Réinitialiser mot de passe root
passwd                             # Depuis mode rescue

# Vérifier/réparer systèmes de fichiers
fsck /dev/sda2                     # Vérification/réparation ext4
fsck -y /dev/sda2                  # Réparation automatique

# Reconstruire initramfs
update-initramfs -u                # Debian/Ubuntu
dracut --force                     # Red Hat/CentOS

# Synchroniser et arrêt propre
sync                               # Écrire buffers sur disque
umount -a                          # Démonter tous systèmes
```

### Boot depuis Live CD/USB
```bash
# Monter système pour réparation
sudo mount /dev/sda2 /mnt          # Système principal
sudo mount /dev/sda1 /mnt/boot     # Partition boot (si séparée)
sudo mount --bind /dev /mnt/dev    # Périphériques
sudo mount --bind /proc /mnt/proc  # Processus
sudo mount --bind /sys /mnt/sys    # Système
sudo chroot /mnt                   # Changer racine

# Dans l'environnement chrooté
update-grub                        # Reconstruire GRUB
systemctl enable service           # Réactiver services
```

## 🔄 Gestion Arrêt/Redémarrage

### Commandes Système
```bash
# Arrêt immédiat
shutdown -h now                    # Arrêt immédiat
poweroff                          # Arrêt via systemd
halt                              # Arrêt simple

# Redémarrage
shutdown -r now                    # Redémarrage immédiat  
reboot                            # Redémarrage via systemd

# Arrêt programmé
shutdown -h +10 "Maintenance in 10 min"  # Arrêt dans 10 min avec message
shutdown -r 16:30 "Reboot scheduled"     # Redémarrage à 16:30

# Annulation
shutdown -c                        # Annuler arrêt programmé
```

### Niveaux d'Arrêt Système
```bash
# Commandes systemd (modernes)
systemctl poweroff                 # Arrêt propre
systemctl reboot                   # Redémarrage propre
systemctl suspend                  # Mise en veille
systemctl hibernate               # Hibernation
systemctl hybrid-sleep            # Veille hybride

# Signals processus
kill -TERM $(pidof process)        # Arrêt propre processus
kill -KILL $(pidof process)        # Arrêt forcé processus
killall process_name               # Arrêt tous processus du nom
```

## 🏗️ Clonage & Déploiement

### Outils de Clonage
| Outil | Type | Usage | Avantages |
|-------|------|-------|-----------|
| **Clonezilla** | Open Source | Clonage disque/partition | Gratuit, supports multiples FS |
| **FOG** | Réseau | Déploiement PXE | Automatisation, gestion centralisée |
| **dd** | Commande | Copie bit-à-bit | Simple, intégré |
| **rsync** | Fichiers | Synchronisation | Incrémental, réseau |

```bash
# Clonage avec dd
dd if=/dev/sda of=/dev/sdb bs=64K conv=noerror,sync status=progress

# Clonage partition avec compression
dd if=/dev/sda1 | gzip > backup.img.gz
gunzip -c backup.img.gz | dd of=/dev/sdb1

# Synchronisation avec rsync
rsync -avH --progress /source/ /destination/
```

### PXE Boot (Network Install)
```bash
# Serveur DHCP (configuration PXE)
next-server 192.168.1.100;        # IP serveur TFTP
filename "pxelinux.0";             # Bootloader réseau

# Structure TFTP
/tftpboot/
├── pxelinux.0                     # Bootloader PXE
├── pxelinux.cfg/
│   └── default                    # Configuration boot
└── images/
    ├── vmlinuz                    # Kernel réseau
    └── initrd.img                 # Initramfs réseau
```

## 💡 Modèles d'Administration

### Gestion Utilisateur Root
```bash
# Modèle traditionnel (root activé)
su -                               # Devenir root avec environnement
su -c "command"                    # Exécuter commande en tant que root

# Modèle sudo (root désactivé)
sudo command                       # Exécuter avec privilèges
sudo -i                           # Shell root interactif
sudo -s                           # Shell avec environnement actuel
sudo -u user command              # Exécuter en tant qu'autre utilisateur

# Gestion sudoers
visudo                            # Éditer /etc/sudoers (avec vérification)
%admin ALL=(ALL) NOPASSWD:ALL     # Groupe admin sans mot de passe
user ALL=(ALL) ALL                # Utilisateur avec tous droits
```

### Sécurisation Boot
```bash
# Protection GRUB par mot de passe
grub-mkpasswd-pbkdf2              # Générer hash mot de passe

# Dans /etc/grub.d/40_custom:
set superusers="admin"
password_pbkdf2 admin [hash_généré]

# Secure Boot (UEFI)
mokutil --sb-state                # Vérifier état Secure Boot
mokutil --import certificate.der  # Importer certificat
```

## 🔍 Diagnostic & Logs Boot

### Journaux Système
```bash
# Logs boot avec systemd
journalctl -b                     # Boot actuel
journalctl -b -1                  # Boot précédent
journalctl -u service.name        # Logs service spécifique
journalctl --since yesterday      # Depuis hier
journalctl -f                     # Suivi temps réel

# Logs traditionnels
dmesg                             # Messages kernel
dmesg | grep -i error             # Erreurs kernel
tail -f /var/log/syslog           # Log système temps réel
```

### Analyse Performance Boot
```bash
# Temps de boot systemd
systemd-analyze                   # Temps total boot
systemd-analyze blame             # Services plus lents
systemd-analyze critical-chain    # Chaîne critique
systemd-analyze plot > boot.svg   # Graphique timeline
```

## ⚠️ Troubleshooting Courant

### Problèmes Boot Fréquents
| Problème | Symptôme | Solution |
|----------|----------|----------|
| **GRUB corrompu** | "grub rescue>" | Réinstaller GRUB depuis Live CD |
| **Kernel panic** | Freeze au boot | Boot kernel précédent, vérifier hardware |
| **FS corrompu** | "fsck required" | Boot rescue, fsck -y partition |
| **fstab invalide** | Échec montage | Éditer fstab en mode rescue |
| **Services failing** | Boot lent/incomplet | journalctl -b, désactiver services problématiques |

### Commandes de Diagnostic
```bash
# Matériel
lshw -short                       # Liste hardware
lscpu                            # Informations CPU
lsblk                            # Périphériques bloc
lsusb && lspci                   # Périphériques USB/PCI

# Système
uname -a                         # Informations kernel
cat /proc/version                # Version détaillée
systemctl --failed               # Services en échec
mount | column -t                # Points de montage
```

---
**💡 Memo** : Mode rescue = `init=/bin/bash` • GRUB = `update-grub` après modif • Systemd = `systemctl` pour tout !