# 📦 Gestion Paquets Linux — Aide-mémoire

## 🏗️ Concepts Fondamentaux

### Architecture Gestionnaires de Paquets
```
Package Manager (Frontend)
    ↓
Dependency Resolution
    ↓
Package Database
    ↓
Package Installation/Removal
```

### Types de Paquets
| Format | Distribution | Outil | Description |
|--------|--------------|-------|-------------|
| **.deb** | Debian/Ubuntu | dpkg, apt | Debian package |
| **.rpm** | RHEL/CentOS/SUSE | rpm, yum, dnf | RedHat package |
| **.pkg.tar.xz** | Arch Linux | pacman | Arch package |
| **.snap** | Universal | snap | Containerized apps |
| **.flatpak** | Universal | flatpak | Sandboxed apps |
| **.appimage** | Universal | - | Portable apps |

## 🔧 APT (Debian/Ubuntu)

### Commandes Essentielles
```bash
# Gestion du cache
apt update                     # Mettre à jour index paquets
apt upgrade                    # Mettre à jour paquets installés
apt full-upgrade              # Upgrade + gestion dépendances
apt dist-upgrade              # Mise à jour distribution (legacy)

# Installation/Suppression
apt install package           # Installer paquet
apt install package=version   # Version spécifique
apt install -y package        # Sans confirmation
apt remove package           # Supprimer (garde configs)
apt purge package            # Supprimer + configurations
apt autoremove               # Supprimer dépendances orphelines
apt autopurge                # autoremove + purge
```

### Recherche & Information
```bash
# Recherche
apt search "motif"           # Rechercher dans nom/description
apt list --installed         # Paquets installés
apt list --upgradable       # Paquets à mettre à jour
apt list package            # Versions disponibles

# Informations détaillées
apt show package            # Détails paquet
apt depends package         # Dépendances
apt rdepends package        # Dépendances inverses
```

### Maintenance & Nettoyage
```bash
# Cache et nettoyage
apt clean                   # Vider cache téléchargement
apt autoclean              # Nettoyer anciens paquets cache
apt autoremove             # Supprimer dépendances inutiles

# Vérification système
apt check                  # Vérifier intégrité dépendances
apt -f install            # Réparer dépendances cassées
```

## 🗂️ Sources de Paquets

### Format Sources.list (Legacy)
```bash
# /etc/apt/sources.list
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

# Format: deb [options] URI distribution composants
# Types: deb (binaires), deb-src (sources)
# Composants Debian:
# - main: logiciels libres
# - contrib: libres mais dépendent de non-free
# - non-free: logiciels propriétaires
# - non-free-firmware: microcodes propriétaires
```

### Format DEB822 (Moderne - Debian 12+)
```bash
# /etc/apt/sources.list.d/debian.sources
Types: deb deb-src
URIs: http://deb.debian.org/debian
Suites: bookworm bookworm-updates
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb deb-src
URIs: http://security.debian.org/debian-security
Suites: bookworm-security
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

### Gestion Sources
```bash
# Migration vers DEB822
apt modernize-sources        # Convertir sources.list (Debian 13+)

# Ajout PPA (Ubuntu)
add-apt-repository ppa:user/ppa-name
add-apt-repository --remove ppa:user/ppa-name

# Sources tierces
echo "deb [signed-by=/usr/share/keyrings/example.gpg] https://repo.example.com stable main" \
    > /etc/apt/sources.list.d/example.list
```

### Gestion Clés GPG
```bash
# Importer clé repository
wget -qO- https://repo.example.com/key.gpg | gpg --dearmor > /usr/share/keyrings/example.gpg

# Lister clés
apt-key list                # Legacy (deprecated)
gpg --list-keys            # Moderne

# Mise à jour clés
apt-key update             # Legacy
```

## 🔍 DPKG (Niveau Bas)

### Installation & Gestion Locale
```bash
# Installation paquets .deb
dpkg -i package.deb        # Installer paquet local
dpkg --force-depends -i package.deb  # Forcer installation
apt --fix-broken install   # Réparer après dpkg -i

# Suppression
dpkg -r package           # Supprimer paquet
dpkg -P package           # Purger (supprimer + configs)
```

### Information & Diagnostic
```bash
# Lister paquets
dpkg -l                   # Tous paquets installés
dpkg -l | grep pattern    # Filtrer par motif
dpkg -l package           # État paquet spécifique

# Contenu paquets
dpkg -L package           # Fichiers installés par paquet
dpkg -S /path/to/file     # Quel paquet contient ce fichier
dpkg -c package.deb       # Contenu fichier .deb

# Informations détaillées
dpkg -s package           # Statut paquet installé
dpkg -I package.deb       # Infos fichier .deb
dpkg --get-selections     # Sélections paquets
```

### Reconfiguration
```bash
# Reconfigurer paquet
dpkg-reconfigure package  # Interface configuration
dpkg-reconfigure -p low package  # Toutes questions
dpkg-reconfigure locales  # Exemple: locales système
dpkg-reconfigure keyboard-configuration  # Clavier
```

## 🎯 Autres Gestionnaires

### YUM/DNF (Red Hat/CentOS/Fedora)
```bash
# DNF (moderne)
dnf update                # Mettre à jour
dnf install package       # Installer
dnf remove package        # Supprimer
dnf search pattern        # Rechercher
dnf info package          # Informations
dnf list installed        # Paquets installés

# YUM (legacy)
yum update
yum install package
yum remove package
```

### Pacman (Arch Linux)
```bash
# Synchronisation & installation
pacman -Sy                # Sync databases
pacman -Syu               # Full system upgrade
pacman -S package         # Installer paquet
pacman -R package         # Supprimer paquet
pacman -Rs package        # Supprimer + dépendances

# Recherche & information
pacman -Ss pattern        # Rechercher
pacman -Si package        # Info paquet
pacman -Q                 # Paquets installés
pacman -Ql package        # Fichiers paquet
```

### Zypper (openSUSE)
```bash
zypper refresh            # Actualiser repos
zypper update             # Mettre à jour
zypper install package    # Installer
zypper remove package     # Supprimer
zypper search pattern     # Rechercher
```

## 🌐 Paquets Universels

### Snap (Canonical)
```bash
# Gestion snaps
snap list                 # Paquets installés
snap find pattern         # Rechercher
snap install package      # Installer
snap remove package       # Supprimer
snap refresh              # Mettre à jour tous
snap refresh package      # Mettre à jour spécifique

# Informations
snap info package         # Détails paquet
snap connections package  # Interfaces connectées
```

### Flatpak
```bash
# Configuration
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Gestion applications
flatpak install flathub com.example.App
flatpak run com.example.App
flatpak update
flatpak uninstall com.example.App
flatpak list              # Applications installées
```

### AppImage
```bash
# Utilisation (pas de gestionnaire)
chmod +x application.appimage
./application.appimage

# Intégration système (optionnel)
# AppImageLauncher pour intégration menu
```

## 🔨 Compilation Source

### Processus Standard
```bash
# Préparation
sudo apt install build-essential  # Outils compilation
sudo apt build-dep package        # Dépendances build

# Configuration & compilation
./configure                   # Vérifier dépendances
./configure --prefix=/usr/local  # Préfixe personnalisé
make                         # Compiler
make -j$(nproc)              # Paralléliser (nb CPU)

# Installation
sudo make install            # Installer système
sudo make uninstall          # Désinstaller (si supporté)
```

### Alternatives Modernes
```bash
# CMake
mkdir build && cd build
cmake ..
make
sudo make install

# Meson
meson build
cd build
ninja
sudo ninja install

# Autotools
autoreconf -i               # Si configure.ac seulement
./configure
make && sudo make install
```

## 🚨 Troubleshooting

### Problèmes Courants APT
```bash
# Dépendances cassées
apt --fix-broken install
apt -f install              # Alias

# Paquets bloqués
apt-mark hold package       # Bloquer mise à jour
apt-mark unhold package     # Débloquer
apt-mark showhold          # Lister bloqués

# Cache corrompu
rm -rf /var/lib/apt/lists/*
apt update

# Verrous
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/apt/lists/lock
sudo dpkg --configure -a
```

### DPKG Issues
```bash
# Paquet dans état incohérent
dpkg --configure -a         # Configurer paquets en attente
dpkg --audit                # Vérifier inconsistances

# Forcer opérations (dangereux)
dpkg --force-depends        # Ignorer dépendances
dpkg --force-conflicts      # Ignorer conflits
dpkg --force-overwrite      # Écraser fichiers

# Base de données corrompue
cd /var/lib/dpkg
sudo cp status status.backup
sudo cp available available.backup
```

### Diagnostic Système
```bash
# Vérification intégrité
debsums -c                  # Vérifier checksums paquets
debsums -l                  # Lister paquets sans checksums

# Orphelins et recommandations
apt autoremove --purge      # Supprimer orphelins + configs
apt-mark showauto          # Paquets auto-installés
apt-mark showmanual        # Paquets manuels

# Statistiques
apt list --installed | wc -l    # Nombre paquets installés
dpkg-query -W -f='${Installed-Size}\t${Package}\n' | sort -n  # Taille paquets
```

## 📋 Configuration Avancée

### Préférences APT
```bash
# /etc/apt/preferences (APT pinning)
Package: *
Pin: release a=stable
Pin-Priority: 700

Package: *
Pin: release a=testing
Pin-Priority: 650

# Exemple: préférer stable mais autoriser testing
```

### Configuration APT
```bash
# /etc/apt/apt.conf.d/
# Exemple: 10local
APT::Default-Release "stable";
APT::Install-Recommends "false";
APT::Install-Suggests "false";
Acquire::http::Proxy "http://proxy:8080";
```

### Hooks APT
```bash
# /etc/apt/apt.conf.d/80custom
DPkg::Pre-Install-Pkgs {"/usr/local/bin/pre-install-hook";};
DPkg::Post-Invoke {"/usr/local/bin/post-install-hook";};
```

## 💡 Bonnes Pratiques

### Sécurité
- ✅ **Mettre à jour régulièrement** : `apt update && apt upgrade`
- ✅ **Vérifier signatures** : Ne pas ignorer erreurs GPG
- ✅ **Sources fiables** : Éviter repositories non officiels suspects
- ✅ **Backups** : Sauvegarder avant mises à jour majeures

### Performance
- ✅ **Mirrors locaux** : Utiliser serveurs proches géographiquement
- ✅ **Nettoyage régulier** : `apt autoremove && apt clean`
- ✅ **Parallélisation** : `make -j$(nproc)` pour compilation

### Gestion
- ✅ **Documentation** : Noter sources personnalisées ajoutées
- ✅ **Tests** : Tester sur machine de dev avant production
- ✅ **Snapshots** : LVM/Btrfs snapshots avant changements majeurs

---
**💡 Memo** : `apt update` avant tout, `apt autoremove` pour nettoyer, `dpkg -l` pour lister installés !