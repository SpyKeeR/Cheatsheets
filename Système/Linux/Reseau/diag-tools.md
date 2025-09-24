# 🔍 Diagnostic Réseau Linux — Aide-mémoire

## 🌐 Tests de Connectivité

### Ping & Traceroute
```bash
# Tests connectivité de base
ping -c 4 8.8.8.8                    # 4 pings vers Google DNS
ping -i 0.2 -c 10 target             # Interval 200ms, 10 pings
ping6 2001:4860:4860::8888           # IPv6

# Traceroute (chemin réseau)
traceroute google.com                # Chemin IPv4
traceroute6 google.com               # Chemin IPv6
mtr google.com                       # Traceroute continu + stats
mtr --report -c 10 target            # Rapport 10 cycles

# Options utiles
ping -f target                       # Flood ping (root)
ping -s 1472 target                  # Taille paquet personnalisée
ping -M do -s 1500 target            # Test MTU (Don't Fragment)
```

### Tests Avancés
```bash
# Hping3 - Tests TCP/UDP
hping3 -c 3 -S -p 80 target         # SYN flood sur port 80
hping3 -c 3 -2 -p 53 target         # UDP vers port 53

# Ncat - Test ports
nc -zv google.com 80                 # Test port TCP
nc -u -zv google.com 53              # Test port UDP
nc -l 1234                           # Écouter port 1234

# Telnet - Test applicatif
telnet smtp.gmail.com 587            # Test SMTP
```

## 🔍 Analyse Ports & Connexions

### ss (Socket Statistics) - Moderne
```bash
# Vue d'ensemble connexions
ss -tuln                             # TCP/UDP listening ports
ss -tulpn                            # Avec PID processus
ss -i                                # Infos interfaces TCP
ss -s                                # Statistiques résumées

# Filtrage spécifique
ss -t state established              # Connexions TCP établies
ss -u                                # Connexions UDP seulement
ss dst 192.168.1.1                  # Connexions vers IP
ss sport 80                          # Connexions depuis port 80
ss dport 443                         # Connexions vers port 443

# Filtres avancés
ss -o state fin-wait-1               # États TCP spécifiques
ss -A inet                           # IPv4 seulement
ss -f inet6                          # IPv6 seulement
```

### netstat (Legacy mais Utile)
```bash
# Équivalences avec ss
netstat -tuln                        # Ports en écoute
netstat -tulpn                       # Avec PID
netstat -i                           # Statistiques interfaces
netstat -r                           # Table routage
netstat -s                           # Statistiques protocoles

# Filtrage
netstat -an | grep :80               # Connexions port 80
netstat -an | grep ESTABLISHED       # Connexions établies
```

### lsof (List Open Files) - Réseau
```bash
# Processus et ports
lsof -i                              # Toutes connexions réseau
lsof -i :80                          # Port 80 spécifiquement
lsof -i tcp:80                       # TCP port 80
lsof -i udp:53                       # UDP port 53

# Par processus/utilisateur
lsof -i -P | grep httpd              # Connexions Apache
lsof -i -u www-data                  # Connexions utilisateur
lsof -p 1234                         # Fichiers ouverts par PID

# Combinaisons utiles
lsof -i -n                           # Pas de résolution DNS
lsof -i -P                           # Pas de résolution ports
```

## 📊 Monitoring Trafic Temps Réel

### iftop - Trafic par Connexion
```bash
# Utilisation basique
iftop                                # Interface par défaut
iftop -i eth0                        # Interface spécifique
iftop -n                             # Pas de résolution DNS
iftop -P                             # Afficher ports
iftop -B                             # Affichage en bytes (pas bits)

# Dans iftop (touches interactives)
# p : toggle ports
# n : toggle DNS resolution
# t : toggle text interface
# q : quit
```

### nethogs - Trafic par Processus
```bash
# Monitoring par processus
nethogs                              # Interface par défaut
nethogs eth0                         # Interface spécifique
nethogs -d 2                         # Refresh toutes les 2s
nethogs -v 3                         # Mode verbose

# Utile pour identifier processus gourmands
```

### nload - Bande Passante Simple
```bash
# Graphique simple
nload                                # Interface par défaut
nload eth0                           # Interface spécifique
nload -u M                           # Unités en MB/s
nload -t 500                         # Intervalle 500ms

# Navigation : ← → pour changer interface
```

### bmon - Monitoring Avancé
```bash
# Interface complète
bmon                                 # Vue d'ensemble interfaces
bmon -p eth0                         # Interface spécifique
bmon -o ascii                        # Sortie ASCII (pour scripts)
```

## 🔧 Analyse Interfaces & Matériel

### ethtool - Interface Ethernet
```bash
# Informations interface
ethtool eth0                         # État liaison (link, speed, duplex)
ethtool -i eth0                      # Infos driver
ethtool -S eth0                      # Statistiques détaillées
ethtool -k eth0                      # Features/offloads

# Tests et modifications
ethtool -t eth0                      # Self-test interface
ethtool -s eth0 speed 100 duplex full  # Forcer vitesse/duplex
ethtool -r eth0                      # Restart auto-negotiation

# Wake-on-LAN
ethtool -s eth0 wol g                # Activer WoL
```

### iwconfig/iw - Wi-Fi (Legacy/Moderne)
```bash
# Legacy (iwconfig)
iwconfig                             # Interfaces Wi-Fi
iwconfig wlan0                       # Interface spécifique
iwscan wlan0                         # Scan réseaux disponibles

# Moderne (iw)
iw dev                               # Interfaces Wi-Fi
iw dev wlan0 info                    # Infos interface
iw dev wlan0 scan                    # Scan réseaux
iw dev wlan0 link                    # Infos connexion courante
```

## 🌍 Résolution DNS & Tests

### dig (Domain Information Groper)
```bash
# Requêtes de base
dig google.com                       # Requête A (IPv4)
dig google.com AAAA                  # Requête AAAA (IPv6)
dig google.com MX                    # Serveurs mail
dig google.com NS                    # Serveurs DNS

# Serveur DNS spécifique
dig @8.8.8.8 google.com             # Via Google DNS
dig @1.1.1.1 google.com AAAA        # Via Cloudflare

# Options utiles
dig +short google.com                # Réponse courte
dig +trace google.com                # Trace résolution complète
dig -x 8.8.8.8                      # Reverse DNS (PTR)
dig +tcp google.com                  # Forcer TCP

# Debugging DNS
dig +norecurse @ns1.google.com google.com  # Requête non-récursive
dig +dnssec google.com               # Vérifier DNSSEC
```

### host & nslookup
```bash
# host (simple et rapide)
host google.com                      # Résolution basique
host -t MX google.com                # Type d'enregistrement
host -a google.com                   # Tous enregistrements
host 8.8.8.8                         # Reverse lookup

# nslookup (interactif)
nslookup google.com                  # Requête simple
nslookup google.com 8.8.8.8         # Serveur spécifique

# Mode interactif nslookup
nslookup
> set type=MX
> google.com
> exit
```

## 📈 Statistiques & Performance

### sar (System Activity Reporter)
```bash
# Statistiques réseau
sar -n DEV 1 10                      # Stats interfaces (1s, 10 fois)
sar -n EDEV 1 5                      # Erreurs interfaces
sar -n TCP 1 5                       # Statistiques TCP
sar -n SOCK 1 5                      # Statistiques sockets

# Analyse historique
sar -n DEV -f /var/log/sa/sa15       # Fichier historique jour 15
```

### vnstat - Statistiques Historiques
```bash
# Installation et configuration
vnstat -u -i eth0                   # Initialiser interface
vnstat -i eth0                      # Statistiques eth0
vnstat -i eth0 -d                   # Par jour
vnstat -i eth0 -m                   # Par mois
vnstat -i eth0 -h                   # Par heure

# Sortie formats
vnstat -i eth0 --json               # Format JSON
vnstat -i eth0 --xml                # Format XML
```

## 🧮 Calcul & Planification Réseau

### ipcalc - Calculs Subnetting
```bash
# Informations réseau
ipcalc 192.168.1.0/24               # Infos réseau complet
ipcalc 192.168.1.100/24             # IP dans réseau
ipcalc 192.168.1.0 255.255.255.0   # Avec masque décimal

# Planification subnets
ipcalc 10.0.0.0/8 -s 1000 500 200  # Diviser en sous-réseaux
```

### sipcalc - Alternative Avancée
```bash
# Plus d'options que ipcalc
sipcalc 192.168.1.0/24              # Calculs complets
sipcalc -a 24 192.168.1.0           # Toutes infos
sipcalc -s 26 192.168.1.0/24        # Subdiviser en /26
```

## 🔍 Capture & Analyse Paquets

### tcpdump - Capture Ligne de Commande
```bash
# Captures basiques
tcpdump -i eth0                      # Tout trafic interface
tcpdump -i any                      # Toutes interfaces
tcpdump -n                          # Pas de résolution DNS
tcpdump -c 100                      # Capturer 100 paquets

# Filtres courants
tcpdump host 192.168.1.1             # Trafic vers/depuis IP
tcpdump port 80                      # Port 80 seulement
tcpdump tcp and port 22              # SSH seulement
tcpdump icmp                         # Ping seulement
tcpdump 'port 53 or port 67'        # DNS ou DHCP

# Sauvegarde et lecture
tcpdump -w capture.pcap              # Sauver capture
tcpdump -r capture.pcap              # Lire capture
tcpdump -r capture.pcap 'host 1.1.1.1'  # Filtrer lecture

# Options avancées
tcpdump -s 0                         # Capturer paquets complets
tcpdump -X                           # Affichage hex + ASCII
tcpdump -vv                          # Très verbose
```

### tshark - Wireshark CLI
```bash
# Équivalent Wireshark en ligne de commande
tshark -i eth0                       # Capture temps réel
tshark -r capture.pcap               # Lecture fichier
tshark -i eth0 -f "port 80"          # Filtre capture
tshark -r file.pcap -Y "http"        # Filtre affichage

# Statistiques
tshark -r capture.pcap -q -z io,stat,1  # Statistiques I/O
tshark -r capture.pcap -q -z conv,ip     # Conversations IP
```

## 🌐 Tests Applicatifs

### curl - Tests HTTP/HTTPS
```bash
# Tests web basiques
curl -I http://google.com            # Headers seulement
curl -v https://google.com           # Verbose (détails SSL/TLS)
curl -w "@curl-format.txt" https://site.com  # Métriques timing

# Tests performance
curl -o /dev/null -s -w "Time: %{time_total}s\n" https://site.com
curl --connect-timeout 5 --max-time 10 https://site.com

# Tests SSL/TLS
curl -k https://site.com             # Ignorer erreurs SSL
curl --tlsv1.2 https://site.com      # Forcer TLS 1.2
```

### Tests Autres Protocoles
```bash
# SMTP
telnet smtp.gmail.com 587
# Puis: EHLO test.com

# POP3/IMAP
telnet pop.gmail.com 995

# FTP
ftp ftp.example.com
# ou: ncftp ftp.example.com

# SSH (connexion test)
ssh -o ConnectTimeout=5 user@host 'echo "SSH OK"'
```

## 🛠️ Troubleshooting Méthodique

### Approche par Couches OSI
```bash
# Couche 1-2 (Physique/Liaison)
ethtool eth0                         # État liaison
ip link show                        # Interfaces up/down
dmesg | grep -i network             # Messages kernel

# Couche 3 (Réseau)
ip route show                       # Table routage
ping -c 1 gateway                   # Test passerelle
ping -c 1 8.8.8.8                  # Test Internet

# Couche 4 (Transport)  
ss -tuln | grep :80                 # Service écoute ?
nc -zv target 80                    # Port accessible ?

# Couche 7 (Application)
curl -I http://target               # Service répond ?
dig target                          # DNS résout ?
```

### Checklist Diagnostic Réseau
1. **Interface** : `ip link show` - Interface UP ?
2. **Adressage** : `ip addr show` - IP configurée ?
3. **Routage** : `ip route show` - Route par défaut ?
4. **DNS** : `dig google.com` - Résolution OK ?
5. **Connectivité** : `ping gateway` puis `ping 8.8.8.8`
6. **Services** : `ss -tuln` - Service écoute ?
7. **Firewall** : `iptables -L` - Règles bloquantes ?

### Tests Performance
```bash
# Bande passante (iperf3)
iperf3 -s                           # Serveur (sur target)
iperf3 -c target                    # Client (test download)
iperf3 -c target -R                 # Test upload
iperf3 -c target -t 30              # Test 30 secondes

# Latence réseau détaillée
mtr --report-cycles 100 target      # 100 cycles MTR
```

---
**💡 Memo** : `ss -tuln` pour ports, `dig` pour DNS, `mtr` pour latence, `tcpdump` pour debug avancé !