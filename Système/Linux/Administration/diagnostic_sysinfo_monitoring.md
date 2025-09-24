# 🔍 Diagnostic & Monitoring Linux — Aide-mémoire

## 🖥️ Informations Système

### Identification & Version
```bash
# Distribution & version
cat /etc/os-release           # Info complète distribution
lsb_release -a               # Description standard LSB
cat /etc/debian_version      # Version Debian/Ubuntu
cat /etc/redhat-release      # Version Red Hat/CentOS

# Noyau & architecture
uname -a                     # Tout (kernel, hostname, arch)
uname -r                     # Version kernel seulement
uname -m                     # Architecture (x86_64, aarch64)
cat /proc/version           # Version détaillée + compilateur
```

### Hardware Discovery
```bash
# CPU & processeur
lscpu                       # Informations CPU complètes
nproc                       # Nombre de cœurs
cat /proc/cpuinfo          # Détails processeur bruts
lscpu | grep "^CPU(s)"     # Nombre total CPU

# Mémoire
lsmem                      # Modules mémoire
free -h                    # Utilisation mémoire (human-readable)
cat /proc/meminfo         # Détails mémoire système

# Périphériques
lspci                      # Devices PCI (cartes réseau, GPU)
lspci -v                   # Version verbeuse
lsusb                      # Périphériques USB
lsusb -t                   # Arbre USB
lsblk                      # Périphériques bloc (disques)
lshw                       # Hardware complet (root requis)
```

## 📊 Monitoring Processus & Performance

### Monitoring Interactif
| Outil | Focus | Points Forts |
|-------|-------|--------------|
| **top** | CPU/Mémoire classique | Universel, toujours présent |
| **htop** | Interface améliorée | Couleurs, tri facile, arbre |
| **atop** | Complet (CPU/IO/Net) | Historique, très détaillé |
| **glances** | Vue d'ensemble | Multi-info, interface moderne |
| **bashtop/btop** | Interface graphique | Très visuel, moderne |

```bash
# top - Raccourcis utiles
top                        # Lancer monitoring
# Dans top :
M                         # Trier par utilisation mémoire
P                         # Trier par utilisation CPU  
k                         # Tuer processus (PID)
q                         # Quitter
c                         # Afficher commande complète
1                         # Détail par CPU (multi-core)

# htop améliorations
htop                      # Interface couleur + souris
# F1-F10 : aide, setup, recherche, filtres, arbre, tri, etc.
```

### Analyse Processus
```bash
# Liste processus
ps aux                    # Tous processus, format BSD
ps -ef                    # Tous processus, format System V
ps -eLf                   # Avec threads (LWP)
ps --forest              # Arbre hiérarchique
pstree                   # Arbre visuel processus
pgrep -f pattern         # Trouver PID par motif

# Processus spécifiques  
pidof program            # PID d'un programme
pgrep -u user            # Processus d'un utilisateur
ps -u username           # Processus utilisateur spécifique

# Alternative moderne
procs                    # Remplaçant ps moderne (si installé)
procs --tree            # Arbre processus
procs --watch           # Mode monitoring
```

### Ressources & Performances
```bash
# Mémoire
free -h                  # Usage mémoire global
vmstat 1 5              # Statistiques mémoire virtuelle (5 fois, 1s)
cat /proc/meminfo       # Détails mémoire système
smem                    # Mémoire par processus (si installé)

# CPU & Load Average
uptime                  # Load average + uptime
w                       # Utilisateurs + load average
cat /proc/loadavg       # Load average brut
mpstat                  # Statistiques processeur (sysstat)
iostat                  # CPU + I/O statistics

# Monitoring continu
watch -n 2 'free -h'    # Surveiller mémoire toutes les 2s
watch 'ps aux --sort=-%cpu | head -10'  # Top CPU processes
```

## 💾 Stockage & I/O

### Espace Disque
```bash
# Utilisation espace
df -h                   # Espace disques (human-readable)
df -i                   # Utilisation inodes
du -sh /path/*          # Taille répertoires (summary)
du -h --max-depth=1     # Taille niveau 1 seulement
ncdu /path              # Interface interactive (si installé)

# Analyse détaillée
lsblk                   # Arbre périphériques bloc
lsblk -f                # Avec systèmes fichiers
blkid                   # UUID et labels partitions
findmnt                 # Points montage actifs
mount | column -t       # Points montage formatés
```

### I/O Performance
```bash
# Monitoring I/O processus
iotop                   # I/O par processus (root requis)
iotop -o                # Seulement processus actifs I/O
pidstat -d 1            # I/O par processus (sysstat)

# Statistiques I/O système
iostat -x 1             # I/O étendues toutes les secondes
dstat -d                # I/O disques simple
vmstat 1                # Include I/O dans stats VM

# Tests performance
hdparm -t /dev/sda      # Test lecture disque
dd if=/dev/zero of=/tmp/test bs=1M count=1024  # Test écriture
```

## 🌐 Réseau & Connectivité

### Interfaces & Configuration
```bash
# État interfaces
ip addr show            # Adresses IP (moderne)
ip link show           # Interfaces réseau
ifconfig               # Configuration réseau (legacy)
nmcli device status    # Status NetworkManager

# Statistiques réseau
ip -s link show        # Statistiques interfaces
cat /proc/net/dev      # Statistiques brutes
ethtool eth0           # Détails interface Ethernet
iwconfig               # Configuration Wi-Fi (legacy)
```

### Connexions & Ports
```bash
# Connexions actives
ss -tuln               # Ports TCP/UDP en écoute (moderne)
ss -tulpn              # Avec PID processus
netstat -tuln          # Alternative legacy
netstat -i             # Statistiques interfaces

# Monitoring réseau temps réel
iftop                  # Trafic par connexion
nethogs               # Trafic par processus
nload                  # Monitoring simple bande passante
vnstat -i eth0         # Statistiques historiques
```

## 📋 Logs & Journaux

### Journalisation Systemd
```bash
# Journalctl essentiels
journalctl             # Tous logs depuis boot
journalctl -f          # Suivi temps réel (follow)
journalctl -u service  # Logs service spécifique
journalctl --since "1 hour ago"  # Dernière heure
journalctl --until "2023-12-01"  # Jusqu'à date
journalctl -p err      # Erreurs seulement (priorities)
journalctl -b          # Boot actuel seulement
journalctl -b -1       # Boot précédent
journalctl --disk-usage # Espace utilisé par logs

# Filtres avancés
journalctl -u apache2 --since today
journalctl _PID=1234   # Logs PID spécifique
journalctl /usr/bin/ssh # Logs binaire spécifique
```

### Logs Système Traditionnels
```bash
# Emplacements logs courants
tail -f /var/log/syslog      # Système général (Debian/Ubuntu)
tail -f /var/log/messages    # Système général (Red Hat/CentOS)
tail -f /var/log/auth.log    # Authentifications
tail -f /var/log/kern.log    # Kernel messages
tail -f /var/log/dmesg       # Boot messages

# Rotation et analyse
logrotate -d /etc/logrotate.conf  # Test config rotation
zcat /var/log/syslog.1.gz | grep ERROR  # Logs compressés
multitail /var/log/syslog /var/log/auth.log  # Multiple logs
```

## 🔍 Diagnostic Avancé

### Fichiers Ouverts & Processus
```bash
# lsof - List Open Files
lsof                    # Tous fichiers ouverts
lsof /path/file        # Processus utilisant fichier
lsof -p PID            # Fichiers ouverts par processus
lsof -u user           # Fichiers ouverts par utilisateur
lsof -i                # Connexions réseau
lsof -i :80            # Processus sur port 80
lsof +D /path          # Récursif dans répertoire

# Surveillance continue
lsof -r 2              # Répéter toutes les 2s
watch 'lsof -i'        # Surveiller connexions réseau
```

### Debugging & Tracing
```bash
# strace - System calls
strace -p PID          # Tracer processus existant
strace -f command      # Tracer avec processus fils
strace -e trace=file command  # Seulement appels fichiers
strace -c command      # Résumé statistiques
strace -o trace.log command   # Sauver dans fichier

# Performance profiling
perf top               # Profiling temps réel (si disponible)
perf record command    # Enregistrer session
perf report            # Analyser session

# Autres outils debug
ltrace command         # Tracer appels bibliothèques
gdb -p PID            # Debugger sur processus actif
```

## ⚙️ Services & Systemd

### Gestion Services
```bash
# Status et contrôle
systemctl status service    # État détaillé service
systemctl start service     # Démarrer service
systemctl stop service      # Arrêter service  
systemctl restart service   # Redémarrer service
systemctl reload service    # Recharger config
systemctl enable service    # Activer au boot
systemctl disable service   # Désactiver au boot

# Listing et états
systemctl list-units       # Services actifs
systemctl list-units --all # Tous services
systemctl list-units --failed  # Services en échec
systemctl --failed         # Raccourci services échoués

# Analyse boot
systemd-analyze            # Temps boot total
systemd-analyze blame      # Services lents au boot
systemd-analyze critical-chain  # Chaîne critique
systemd-analyze plot > boot.svg # Graphique timeline
```

## 🛠️ Outils Utilitaires

### Surveillance Continue
```bash
# Watch - Exécution répétée
watch command              # Répéter toutes les 2s
watch -n 1 command         # Intervalle 1 seconde
watch -d command           # Highlighter différences
watch 'ps aux --sort=-%mem | head -10'  # Top mémoire

# Monitoring spécialisé
dstat                      # Statistiques système complètes
dstat -cdngy               # CPU, disque, réseau, système
sar -u 1 5                # CPU utilization (sysstat)
sar -r 1 5                # Memory utilization
```

### Outils Modernes
```bash
# Alternatives modernes (si installées)
exa -la                   # ls amélioré
bat file.txt             # cat avec coloration
fd pattern               # find amélioré  
rg pattern               # grep amélioré (ripgrep)
dust                     # du amélioré
procs                    # ps amélioré
delta                    # diff amélioré
```

## 🚨 Diagnostic Problèmes Courants

### Performance Lente
```bash
# Checklist diagnostic
uptime                   # Load average élevé ?
free -h                  # Mémoire saturée ?
df -h                    # Disques pleins ?
iotop                   # I/O élevées ?
top -d 1                # Processus gourmands ?
```

### Problèmes Réseau
```bash
# Tests connectivité
ping gateway            # Réseau local OK ?
ping 8.8.8.8           # Internet OK ?
dig domain.com         # DNS OK ?
ss -tuln | grep :80    # Service écoute ?
```

### Analyse Espace Disque
```bash
# Trouver gros fichiers
find / -size +100M 2>/dev/null  # Fichiers >100MB
du -sh /var/log/*              # Logs volumineux
lsof +L1                       # Fichiers supprimés mais ouverts
df -i                          # Inodes épuisés ?
```

### Services Non Fonctionnels
```bash
# Diagnostic service
systemctl status service      # État et logs récents
journalctl -u service -f      # Logs temps réel
systemctl cat service         # Voir configuration unit
systemctl show service        # Propriétés complètes
```

## 💡 Bonnes Pratiques

### Monitoring Proactif
- ✅ **Surveillance régulière** : `htop`, `df -h`, `free -h`
- ✅ **Logs centralisés** : `journalctl` pour debugging
- ✅ **Baseline** : Connaître valeurs normales du système
- ✅ **Alertes** : Scripts monitoring + notifications

### Diagnostic Méthodique
1. **Vue d'ensemble** : `htop`, `df -h`, `free -h`
2. **Identification** : `ps aux`, `lsof`, `ss -tuln`
3. **Analyse logs** : `journalctl`, `/var/log/*`
4. **Tests ciblés** : `strace`, `tcpdump`, benchmarks
5. **Documentation** : Noter symptômes et solutions

### Outils par Problème
| Problème | Outils Recommandés |
|----------|-------------------|
| **Performance CPU** | `htop`, `top`, `perf`, `sar -u` |
| **Mémoire** | `free`, `vmstat`, `smem` |
| **I/O Disque** | `iotop`, `iostat`, `dstat -d` |
| **Réseau** | `iftop`, `nethogs`, `ss`, `tcpdump` |
| **Processus** | `ps`, `lsof`, `strace`, `pstree` |
| **Services** | `systemctl`, `journalctl` |
| **Espace** | `df`, `du`, `ncdu`, `lsblk` |

---
**💡 Memo** : `htop` pour overview, `lsof` pour files, `journalctl -f` pour logs temps réel, `watch` pour monitoring !