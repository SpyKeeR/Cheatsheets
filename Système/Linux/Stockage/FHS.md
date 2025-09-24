# 📁 Filesystem Hierarchy Standard — Aide-mémoire

## 🏗️ Structure Racine

### Répertoires Essentiels
```
/                    # Racine de l'arborescence (point de montage principal)
├── bin/             # Binaires essentiels utilisateur (ls, cat, cp)
├── sbin/            # Binaires système/admin (mount, iptables, reboot)
├── boot/            # Noyau + bootloader (GRUB, initrd)
├── dev/             # Fichiers périphériques (sda, tty, null)
├── etc/             # Configuration système
├── home/            # Répertoires utilisateurs
├── lib/             # Bibliothèques partagées essentielles
├── media/           # Points montage amovibles
├── mnt/             # Points montage temporaires
├── opt/             # Logiciels tiers
├── proc/            # Filesystem virtuel (infos kernel)
├── root/            # Répertoire personnel root
├── run/             # Données runtime (PID, sockets)
├── srv/             # Données services (web, ftp)
├── sys/             # Interface kernel (sysfs)
├── tmp/             # Fichiers temporaires
├── usr/             # Programmes et données utilisateur
└── var/             # Données variables (logs, cache)
```

## 👤 Répertoires Utilisateurs

### Structure Home
| Répertoire | Usage | Exemples |
|------------|-------|----------|
| `/home/user/` | Répertoire personnel utilisateur | Documents, config personnelles |
| `/root/` | Répertoire personnel root | Séparé pour sécurité |
| `~/.config/` | Configuration applications | `~/.config/git/config` |
| `~/.local/` | Données locales utilisateur | `~/.local/bin/`, `~/.local/share/` |
| `~/.cache/` | Cache applications | Données temporaires apps |

### Fichiers Cachés Courants
```bash
~/.bashrc            # Configuration bash
~/.profile           # Variables environnement
~/.ssh/              # Clés SSH
~/.vimrc             # Configuration vim
~/.gitconfig         # Configuration git globale
~/.history           # Historique commandes
```

## ⚙️ Configuration Système (/etc)

### Fichiers Critiques
| Fichier | Fonction | Exemple |
|---------|----------|---------|
| `/etc/passwd` | Comptes utilisateurs | `user:x:1000:1000:User:/home/user:/bin/bash` |
| `/etc/shadow` | Mots de passe chiffrés | Accessible root uniquement |
| `/etc/group` | Groupes système | `sudo:x:27:user1,user2` |
| `/etc/sudoers` | Configuration sudo | Éditer avec `visudo` |
| `/etc/fstab` | Points de montage automatiques | UUID + options montage |
| `/etc/hosts` | Résolution noms locaux | `127.0.0.1 localhost` |
| `/etc/resolv.conf` | Serveurs DNS | `nameserver 8.8.8.8` |
| `/etc/hostname` | Nom machine | `monserveur.domain.local` |

### Répertoires Configuration
```bash
/etc/network/        # Configuration réseau (Debian)
 /etc/systemd/        # Services systemd
/etc/cron.d/         # Tâches cron système
/etc/ssh/            # Configuration SSH
/etc/ssl/            # Certificats SSL
/etc/apt/            # Configuration APT (Debian/Ubuntu)
 /etc/yum.repos.d/    # Dépôts YUM (RHEL/CentOS)
```

## 🔧 Binaires & Bibliothèques

### Hiérarchie Binaires
| Répertoire | Contenu | Quand Utilisé |
|------------|---------|---------------|
| `/bin/` | Commandes de base | Toujours disponible (rescue mode) |
| `/sbin/` | Commandes système | Administration, démarrage |
| `/usr/bin/` | Programmes utilisateur | Applications standard |
| `/usr/sbin/` | Utilitaires système | Administration avancée |
| `/usr/local/bin/` | Programmes compilés localement | Logiciels personnalisés |

### Bibliothèques
```bash
/lib/                # Bibliothèques essentielles système
/lib64/              # Bibliothèques 64-bit (si système mixte)
 /usr/lib/            # Bibliothèques programmes utilisateur
/usr/local/lib/      # Bibliothèques compilées localement

# Commandes utiles
ldd /bin/ls          # Voir dépendances bibliothèques
ldconfig -p          # Lister bibliothèques disponibles
```

## 💾 Stockage & Montage

### Points de Montage Standards
| Point | Usage | Type Typique |
|-------|-------|--------------|
| `/media/` | Supports amovibles | USB, CD/DVD (auto-montage) |
| `/mnt/` | Montage temporaire manuel | Disques réseau, maintenance |
| `/srv/` | Données services | Sites web, bases données |
| `/var/` | Données variables | Logs, cache, spools |

### Exemples Montage
```bash
# Montage manuel
mount /dev/sdb1 /mnt/usb
mount -t nfs server:/share /mnt/nfs

# Montage automatique (/etc/fstab)
UUID=12345-6789 /home ext4 defaults 0 2
//server/share /mnt/smb cifs credentials=/etc/samba/creds 0 0
```

## 📊 Données Variables (/var)

### Structure /var
```bash
/var/log/            # Journaux système et applications
├── syslog           # Journal système principal
├── auth.log         # Authentifications
├── kern.log         # Messages noyau
└── apache2/         # Logs Apache

/var/cache/          # Cache applications
├── apt/             # Cache paquets APT
└── man/             # Pages manuel mises en cache

/var/lib/            # Données persistantes applications
├── mysql/           # Bases de données MySQL
├── docker/          # Images et conteneurs Docker
└── dpkg/            # État paquets installés

/var/spool/          # Files d'attente
├── mail/            # Mails en attente
├── cron/            # Tâches cron utilisateurs
└── cups/            # Jobs impression

/var/tmp/            # Fichiers temporaires persistants
/var/run/ -> /run/   # Lien symbolique (legacy)
```

## 🖥️ Systèmes Virtuels

### /proc (Process Filesystem)
```bash
# Informations système temps réel
/proc/cpuinfo        # Informations processeur
/proc/meminfo        # Utilisation mémoire
/proc/version        # Version kernel
/proc/uptime         # Temps fonctionnement
/proc/loadavg        # Charge système

# Informations processus
/proc/PID/           # Répertoire par processus
/proc/PID/cmdline    # Ligne commande processus
/proc/PID/environ    # Variables environnement
/proc/PID/fd/        # Descripteurs fichiers ouverts
```

### /sys (Sysfs)
```bash
# Interface avec le kernel et pilotes
/sys/class/          # Classes périphériques
/sys/block/          # Périphériques bloc (disques)
 /sys/bus/            # Bus système (PCI, USB)
 /sys/devices/        # Arbre périphériques physiques
/sys/kernel/         # Paramètres kernel modifiables
/sys/power/          # Gestion énergie

# Exemples pratiques
cat /sys/class/thermal/thermal_zone0/temp  # Température CPU
echo 1 > /sys/block/sda/queue/rotational   # Marquer SSD
```

## 🔧 Répertoire /dev

### Types de Périphériques
```bash
# Disques et partitions
/dev/sda             # Premier disque SATA/SCSI
/dev/sda1            # Première partition
/dev/nvme0n1         # Disque NVMe
/dev/loop0           # Périphérique loop

# Terminaux
/dev/tty1            # Console virtuelle 1
/dev/pts/0           # Terminal pseudo (SSH)
/dev/console         # Console système

# Spéciaux
/dev/null            # "Trou noir" (ignore données)
/dev/zero            # Source de zéros infinis
/dev/random          # Générateur aléatoire bloquant
/dev/urandom         # Générateur aléatoire non-bloquant
```

## 🏛️ Hiérarchie /usr

### Structure Complète
```bash
/usr/                # "Unix System Resources"
├── bin/             # Binaires utilisateur principaux
├── sbin/            # Binaires système utilisateur
├── lib/             # Bibliothèques pour /usr/bin
├── local/           # Hiérarchie locale (compilé manuellement)
│   ├── bin/         # Binaires locaux
│   ├── lib/         # Bibliothèques locales
│   └── share/       # Données locales
├── share/           # Données indépendantes architecture
│   ├── doc/         # Documentation
│   ├── man/         # Pages de manuel
│   ├── locale/      # Traductions
│   └── applications/ # Fichiers .desktop
└── src/             # Code source (optionnel)
```

## 📝 Répertoire /opt

### Logiciels Tiers
```bash
/opt/                # Applications tierces auto-contenues
├── google/          # Google Chrome, Earth
├── teamviewer/      # TeamViewer
├── lampp/           # XAMPP
└── custom-app/      # Application personnalisée
    ├── bin/         # Binaires
    ├── lib/         # Bibliothèques
    └── share/       # Données
```

## 💡 Bonnes Pratiques FHS

### Conventions de Nommage
- ✅ **Jamais d'espaces** dans noms fichiers système
- ✅ **Respecter la casse** (sensible)
- ✅ **Préfixer par point** fichiers cachés : `.bashrc`
- ✅ **Extensions explicites** : `.conf`, `.log`, `.service`

### Gestion Permissions
```bash
# Permissions typiques par répertoire
/                    # 755 (rwxr-xr-x)
 /root/               # 700 (rwx------)
/home/user/          # 755 mais /home/user/private/ → 700
/etc/passwd          # 644 (rw-r--r--)
/etc/shadow          # 640 (rw-r-----)
/tmp/                # 1777 (rwxrwxrwt) sticky bit
/var/log/            # 755, logs → 644 ou 640
```

### Navigation Efficace
```bash
# Variables utiles
$HOME ou ~           # Répertoire personnel
$PWD                 # Répertoire courant
$OLDPWD              # Répertoire précédent

# Raccourcis navigation
cd -                 # Répertoire précédent
cd                   # Retour $HOME
cd ~user             # Répertoire utilisateur
pushd /path && popd  # Stack répertoires
```

## 🔍 Commandes d'Exploration

### Analyse Arborescence
```bash
# Visualisation structure
tree /etc -L 2       # Arborescence limitée à 2 niveaux
find / -maxdepth 2 -type d  # Répertoires racine + 1 niveau
ls -la /             # Contenu racine avec permissions

# Utilisation disque
du -sh /*            # Taille répertoires racine
df -h                # Espace filesystems montés
lsblk                # Arbre périphériques bloc

# Recherche fichiers
locate filename      # Base indexée (updatedb)
find / -name "*.conf" -path "/etc/*"  # Configs dans /etc
which command        # Localiser binaire dans PATH
whereis command      # Binaire + sources + man
```

### Informations Système
```bash
# Montages actifs
mount | column -t    # Points montage formatés
findmnt /            # Arbre montages depuis racine
lsof /path           # Fichiers ouverts dans path

# Processus et fichiers
lsof -p PID          # Fichiers ouverts par processus
fuser -v /path       # Processus utilisant path
ls -l /proc/PID/fd/  # Descripteurs fichiers processus
```

## 📋 Distribution-Specific

### Debian/Ubuntu Specifics
```bash
/etc/apt/sources.list     # Dépôts APT
/var/lib/dpkg/status      # État paquets
/usr/share/doc/           # Documentation paquets
/etc/default/             # Configurations par défaut services
```

### RHEL/CentOS Specifics
```bash
/etc/yum.repos.d/         # Dépôts YUM/DNF
/var/lib/rpm/             # Base RPM
/etc/sysconfig/           # Configuration services système
/usr/share/doc/           # Documentation paquets
```

---
**💡 Memo** : `/etc` = config, `/var` = données variables, `/usr` = programmes utilisateur, `/opt` = tiers !