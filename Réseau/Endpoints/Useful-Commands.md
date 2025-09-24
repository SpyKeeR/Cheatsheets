# 🛠️ Commandes Réseau & Diagnostics — Aide-mémoire

## 🔌 Configuration Interfaces

### Windows (ipconfig)
```cmd
# Configuration basique
ipconfig                    # Résumé IP des interfaces actives
ipconfig /all              # Détails complets (MAC, DHCP, DNS, passerelle)
ipconfig /release          # Libérer bail DHCP
ipconfig /renew            # Renouveler bail DHCP
ipconfig /renew6           # Renouveler IPv6

# DNS
ipconfig /flushdns         # Vider cache DNS
ipconfig /displaydns       # Afficher cache DNS
ipconfig /registerdns      # Renouveler enregistrements DNS dynamiques

# Interface spécifique
ipconfig /release "Ethernet"
ipconfig /renew "Wi-Fi"
```

### Linux (ip command)
```bash
# Interfaces & liens
ip link show                    # Lister interfaces physiques
ip link set eth0 up            # Activer interface
ip link set eth0 down          # Désactiver interface
ip link set eth0 mtu 1400      # Modifier MTU

# Adressage IPv4/IPv6
ip addr show                   # Toutes adresses configurées
ip addr show eth0             # Adresses interface spécifique
ip addr add 192.168.1.100/24 dev eth0    # Ajouter IP
ip addr del 192.168.1.100/24 dev eth0    # Supprimer IP
ip -6 addr show               # Adresses IPv6 uniquement

# Routage
ip route show                 # Table routage IPv4
ip route show table all       # Toutes tables routage
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0    # Ajouter route
ip route del 10.0.0.0/8      # Supprimer route
ip route get 8.8.8.8         # Route vers destination spécifique
```

### Linux Legacy (ifconfig/route)
```bash
# ifconfig (si installé)
ifconfig                      # Toutes interfaces
ifconfig eth0                 # Interface spécifique
ifconfig eth0 192.168.1.100 netmask 255.255.255.0    # Config IP
ifconfig eth0:1 192.168.1.101 # Alias IP

# route (legacy)
route -n                      # Table routage (numérique)
route add -net 10.0.0.0/8 gw 192.168.1.1    # Ajouter route
route del -net 10.0.0.0/8    # Supprimer route
```

## 🔍 Table ARP/Voisinage

### Gestion ARP
```bash
# Affichage
arp -a                        # Table ARP complète (Windows/Linux)
arp -n                        # Sans résolution DNS (Linux)
ip neigh show                 # Table voisins (Linux moderne)

# Manipulation
arp -s 192.168.1.1 aa:bb:cc:dd:ee:ff    # Entrée statique (Windows)
arp -d 192.168.1.1           # Supprimer entrée (Windows)
sudo ip neigh flush all       # Vider table (Linux)
sudo ip neigh del 192.168.1.1 dev eth0  # Supprimer entrée (Linux)

# Monitoring
arp -a | grep incomplete      # Entrées incomplètes
ip neigh show nud failed      # Entrées échouées (Linux)
```

## 🌐 Connectivité & Tests

### Ping Avancé
```bash
# Options courantes
ping -c 4 8.8.8.8            # 4 paquets seulement (Linux)
ping -n 4 8.8.8.8            # 4 paquets seulement (Windows)
ping -t 8.8.8.8              # Continu (Windows)
ping -s 1472 8.8.8.8         # Taille paquet (Linux)
ping -l 1472 8.8.8.8         # Taille paquet (Windows)
ping -i 0.5 8.8.8.8          # Intervalle 0.5s (Linux)
ping -W 1000 8.8.8.8         # Timeout 1s (Linux)
ping -w 1000 8.8.8.8         # Timeout 1s (Windows)

# Tests spécialisés
ping -f 8.8.8.8              # Flood ping (root, Linux)
ping -D 8.8.8.8              # Don't fragment (Linux)
ping -q 8.8.8.8              # Quiet (stats seulement)
ping6 ::1                    # IPv6 loopback
```

### Interprétation Réponses Ping
| Message | Signification | Cause Probable |
|---------|---------------|----------------|
| **Reply** | Succès | Connectivité OK |
| **Request timeout** | Pas de réponse | Firewall, hôte éteint, réseau down |
| **Destination unreachable** | Routage impossible | Pas de route, interface down |
| **TTL expired** | Boucle routage | Problème routage, distance trop élevée |
| **Fragmentation needed** | MTU trop petit | Problème MTU path discovery |

### Traceroute/Tracert
```bash
# Usage basique
traceroute google.com         # Linux
tracert google.com           # Windows
traceroute -n google.com     # Sans résolution DNS (Linux)
tracert -d google.com        # Sans résolution DNS (Windows)

# Options avancées
traceroute -m 15 google.com  # TTL max 15 (Linux)
tracert -h 15 google.com     # TTL max 15 (Windows)
traceroute -w 2 google.com   # Timeout 2s (Linux)
tracert -w 2000 google.com   # Timeout 2s (Windows)
traceroute -I google.com     # ICMP mode (Linux)
traceroute -T -p 80 google.com # TCP mode port 80 (Linux)

# Analyse IPv6
traceroute6 google.com       # IPv6 traceroute (Linux)
tracert -6 google.com        # IPv6 traceroute (Windows)
```

## 🔗 Connexions & Ports

### Netstat (Legacy mais universel)
```bash
# Connexions actives
netstat -a                   # Toutes connexions + ports écoute
netstat -an                  # Sans résolution DNS
netstat -at                  # TCP seulement
netstat -au                  # UDP seulement
netstat -al                  # Ports en écoute seulement
netstat -ap                  # Avec PID processus (Linux)
netstat -ano                 # Avec PID processus (Windows)

# Statistiques
netstat -s                   # Statistiques protocoles
netstat -i                   # Statistiques interfaces
netstat -r                   # Table routage
```

### ss (Linux moderne)
```bash
# Remplaçant moderne de netstat
ss -tulpn                    # TCP+UDP, écoute, processus, numérique
ss -ta                       # Toutes connexions TCP
ss -ua                       # Toutes connexions UDP
ss -ln                       # Ports écoute sans résolution
ss -o                        # Avec infos timer

# Filtrage avancé
ss -t state established      # Connexions TCP établies
ss -t dport :80             # Connexions vers port 80
ss src 192.168.1.0/24       # Connexions depuis subnet
ss -K dst 192.168.1.100     # Tuer connexions vers IP
```

### PowerShell (Windows moderne)
```powershell
# Connexions TCP/UDP
Get-NetTCPConnection         # Connexions TCP
Get-NetUDPEndpoint          # Points terminaison UDP
Get-NetTCPConnection -State Established    # TCP établies
Get-NetTCPConnection -LocalPort 80         # Port local 80

# Avec processus
Get-NetTCPConnection | Select LocalAddress,LocalPort,RemoteAddress,RemotePort,State,@{n='Process';e={(Get-Process -Id $_.OwningProcess).Name}}
```

## 📊 États TCP & Diagnostics

### États Connexion TCP
| État | Description | Signification |
|------|-------------|---------------|
| **LISTEN** | Écoute | Port ouvert, attend connexions |
| **SYN_SENT** | SYN envoyé | Demande connexion en cours |
| **SYN_RECV** | SYN reçu | Serveur traite demande |
| **ESTABLISHED** | Établie | Connexion active |
| **FIN_WAIT_1** | FIN envoyé | Fermeture initiée |
| **FIN_WAIT_2** | ACK reçu | Attente FIN distant |
| **CLOSE_WAIT** | FIN reçu | Attente fermeture locale |
| **CLOSING** | FIN mutuel | Fermeture simultanée |
| **LAST_ACK** | Dernier ACK | Attente ACK final |
| **TIME_WAIT** | Attente sécurité | Connexion fermée, timeout |

### Analyse Performance Réseau
```bash
# Bande passante
iperf3 -c server             # Test client (Linux)
iperf3 -s                    # Mode serveur (Linux)

# Monitoring temps réel
iftop                        # Monitoring interface (Linux)
nload eth0                   # Monitoring simple (Linux)
nethogs                     # Usage par processus (Linux)

# Windows - Performance counters
typeperf "\\Network Interface(*)\\Bytes Total/sec"
```

## 🌍 DNS & Résolution

### nslookup (Universel)
```bash
# Requêtes basiques
nslookup google.com          # A record (IPv4)
nslookup google.com 8.8.8.8  # Serveur DNS spécifique

# Types d'enregistrements
nslookup -type=AAAA google.com    # IPv6
nslookup -type=MX google.com      # Mail servers
nslookup -type=NS google.com      # Name servers
nslookup -type=TXT google.com     # Text records
nslookup -type=SOA google.com     # Start of Authority

# Résolution inverse
nslookup 8.8.8.8             # PTR record
```

### dig (Linux avancé)
```bash
# Requêtes DNS détaillées
dig google.com               # A record avec détails
dig @8.8.8.8 google.com     # Serveur DNS spécifique
dig google.com MX            # Mail exchangers
dig google.com NS            # Name servers
dig google.com TXT           # Text records
dig -x 8.8.8.8              # Résolution inverse

# Options avancées
dig +trace google.com        # Trace complet DNS
dig +short google.com        # Réponse courte seulement
dig +noall +answer google.com # Section réponse seulement
```

### host (Linux simple)
```bash
host google.com              # Résolution simple
host -t MX google.com        # Type spécifique
host 8.8.8.8                # Résolution inverse
```

## 🔧 Outils Diagnostics Avancés

### Analyse Paquets
```bash
# tcpdump (Linux)
sudo tcpdump -i eth0         # Capture interface
sudo tcpdump host 192.168.1.1    # Trafic vers/depuis host
sudo tcpdump port 80         # Port spécifique
sudo tcpdump -w capture.pcap # Sauvegarder capture
sudo tcpdump -r capture.pcap # Lire capture

# Wireshark (GUI)
wireshark                    # Interface graphique
tshark -i eth0              # Version ligne commande
```

### Tests Connectivité Avancés
```bash
# telnet (test port TCP)
telnet google.com 80         # Test port 80
telnet 192.168.1.1 22       # Test SSH

# nc (netcat) - Swiss Army Knife
nc -z google.com 80         # Test port (zero-I/O)
nc -zv google.com 80-90     # Scan ports 80-90
nc -l 1234                  # Écouter port 1234
echo "test" | nc host 1234  # Envoyer données

# curl/wget (HTTP)
curl -I http://google.com    # Headers HTTP seulement
curl -o /dev/null -s -w "%{time_total}\n" http://google.com  # Temps réponse
wget --spider --server-response http://google.com  # Test sans télécharger
```

### Monitoring Interfaces
```bash
# Linux
cat /proc/net/dev           # Statistiques interfaces
ethtool eth0                # Détails interface Ethernet
iwconfig wlan0              # Configuration Wi-Fi
ip -s link show eth0        # Statistiques interface

# Windows
netsh interface show interface      # État interfaces
netsh wlan show profiles           # Profils Wi-Fi
netsh interface ip show config     # Configuration IP
```

## 🚨 Troubleshooting Courant

### Checklist Connectivité
1. **Interface** : `ip link show` / `ipconfig`
2. **Adressage** : `ip addr show` / `ipconfig /all`
3. **Passerelle** : `ip route show` / `route print`
4. **DNS** : `nslookup` / `dig`
5. **Connectivité locale** : `ping gateway`
6. **Connectivité externe** : `ping 8.8.8.8`
7. **Résolution DNS** : `ping google.com`

### Problèmes Fréquents
| Symptôme | Diagnostic | Commandes Utiles |
|----------|------------|------------------|
| **Pas d'IP** | DHCP/Config statique | `ipconfig /renew`, `dhclient` |
| **Pas de résolution DNS** | Serveurs DNS | `nslookup`, `cat /etc/resolv.conf` |
| **Connectivité intermittente** | Perte paquets | `ping -t`, `mtr` |
| **Lenteur** | Bande passante/Latence | `iperf3`, `traceroute` |
| **Port fermé** | Firewall/Service | `nc -z`, `telnet` |

### Commandes de Réinitialisation
```bash
# Windows - Reset réseau complet
netsh winsock reset
netsh int ip reset
ipconfig /flushdns
ipconfig /release
ipconfig /renew

# Linux - Redémarrage services réseau
sudo systemctl restart networking     # Debian/Ubuntu
sudo systemctl restart NetworkManager # RHEL/CentOS
sudo service networking restart       # SysV init
```