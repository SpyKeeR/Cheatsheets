# 🌐 Configuration Réseau Linux — Aide-mémoire

## 🏗️ Architecture Réseau Linux

### Nommage Interfaces (systemd)
| Type | Format | Exemple | Description |
|------|--------|---------|-------------|
| **Ethernet** | `enp<bus>s<slot>` | `enp0s3` | PCI slot |
| **Ethernet** | `ens<slot>` | `ens33` | Slot numérique |
| **Wi-Fi** | `wlp<bus>s<slot>` | `wlp3s0` | Wireless PCI |
| **Loopback** | `lo` | `lo` | Interface locale |
| **Virtuel** | `veth*`, `br*` | `veth0`, `br0` | Containers, bridges |

```bash
# Découverte interfaces
ip link show                  # Toutes interfaces
lshw -class network          # Hardware réseau
dmesg | grep -E "(eth\|ens\|enp)" # Messages kernel interfaces
```

## ⚙️ Configuration avec ip (iproute2)

### Gestion Interfaces
```bash
# État interfaces
ip link show                  # Lister interfaces
ip addr show                  # Adresses IP
ip addr show eth0            # Interface spécifique

# Activation/Désactivation
ip link set eth0 up          # Activer interface
ip link set eth0 down        # Désactiver interface

# Configuration IP
ip addr add 192.168.1.100/24 dev eth0    # Ajouter IP
ip addr del 192.168.1.100/24 dev eth0    # Supprimer IP
ip addr flush eth0           # Vider toutes IP

# MAC address
ip link set eth0 address aa:bb:cc:dd:ee:ff  # Changer MAC
```

### Routage
```bash
# Consultation routes
ip route show                # Table routage
ip route show table local    # Table locale
ip -6 route show             # Routes IPv6

# Ajout/Suppression routes
ip route add default via 192.168.1.1        # Route par défaut
ip route add 10.0.0.0/8 via 192.168.1.254  # Route spécifique
ip route add 192.168.2.0/24 dev eth1        # Route directe
ip route del 10.0.0.0/8                     # Supprimer route

# Diagnostic routage
ip route get 8.8.8.8         # Chemin vers destination
```

### Tables de Routage Avancées
```bash
# Tables multiples (Policy Based Routing)
ip rule list                 # Lister règles
ip route show table main     # Table principale
ip route add table 100 default via 192.168.2.1  # Route dans table custom

# Exemple multi-homing
ip rule add from 192.168.1.0/24 table 100
ip route add table 100 default via 192.168.1.1
```

## 🔧 NetworkManager (nmcli)

### Gestion Connexions
```bash
# Vue d'ensemble
nmcli device status          # État périphériques
nmcli connection show        # Connexions configurées
nmcli general status         # État général NetworkManager

# Connexions actives
nmcli connection up "Wired connection 1"     # Activer
nmcli connection down "Wired connection 1"   # Désactiver
nmcli device disconnect eth0                 # Déconnecter périphérique
```

### Configuration Statique
```bash
# Créer connexion Ethernet statique
nmcli connection add \
    type ethernet \
    con-name "Static-eth0" \
    ifname eth0 \
    ipv4.method manual \
    ipv4.addresses "192.168.1.100/24" \
    ipv4.gateway "192.168.1.1" \
    ipv4.dns "8.8.8.8,1.1.1.1"

# Modifier connexion existante
nmcli connection modify "Static-eth0" \
    ipv4.addresses "192.168.1.150/24"
nmcli connection modify "Static-eth0" \
    +ipv4.dns "8.8.4.4"              # Ajouter DNS

# Configuration DHCP
nmcli connection modify "Wired connection 1" \
    ipv4.method auto
```

### Configuration Wi-Fi
```bash
# Scanner réseaux
nmcli device wifi list

# Connexion WPA/WPA2
nmcli device wifi connect "SSID" password "motdepasse"

# Connexion avec paramètres avancés
nmcli connection add \
    type wifi \
    con-name "MonWiFi" \
    ifname wlan0 \
    ssid "MonSSID" \
    wifi-sec.key-mgmt wpa-psk \
    wifi-sec.psk "motdepasse"
```

## 📁 Fichiers Configuration

### Netplan (Ubuntu 18.04+)
```yaml
# /etc/netplan/01-network-config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
        search: [domain.local]
```

```bash
# Application configuration
netplan try              # Test avec timeout
netplan apply           # Appliquer définitivement
netplan generate        # Générer config systemd-networkd
```

### Interfaces Debian (/etc/network/interfaces)
```bash
# Configuration statique
auto eth0
iface eth0 inet static
    address 192.168.1.100/24
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1
    dns-search domain.local

# Configuration DHCP
auto eth0
iface eth0 inet dhcp

# Interface virtuelle
auto eth0:0
iface eth0:0 inet static
    address 192.168.1.101/24

# VLAN
auto eth0.100
iface eth0.100 inet static
    address 10.0.100.10/24
    vlan-raw-device eth0
```

### Red Hat/CentOS (/etc/sysconfig/network-scripts/)
```bash
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=static
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=1.1.1.1
```

## 🔄 Redémarrage Services Réseau

### Systemd (Moderne)
```bash
# NetworkManager
systemctl status NetworkManager
systemctl restart NetworkManager
systemctl enable NetworkManager

# systemd-networkd (Ubuntu/netplan)
systemctl status systemd-networkd
systemctl restart systemd-networkd

# Interfaces traditionnelles
systemctl status networking
systemctl restart networking
```

### Commandes Legacy
```bash
# Debian/Ubuntu
service networking restart
/etc/init.d/networking restart

# Red Hat/CentOS
service network restart
systemctl restart network

# Interface spécifique
ifdown eth0 && ifup eth0
```

## 🏷️ Résolution Noms & DNS

### Configuration DNS
```bash
# /etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1
search domain.local company.com
options timeout:2 attempts:3 rotate
```

### systemd-resolved
```bash
# État et configuration
resolvectl status           # Configuration DNS globale
resolvectl status eth0      # DNS interface spécifique
resolvectl query domain.com # Résoudre nom
resolvectl flush-caches     # Vider cache DNS

# Configuration par interface
resolvectl dns eth0 8.8.8.8 1.1.1.1
resolvectl domain eth0 domain.local
```

### Fichiers Hosts & NSSwitch
```bash
# /etc/hosts
127.0.0.1       localhost
127.0.1.1       hostname.domain.local hostname
192.168.1.10    server.domain.local server

# /etc/nsswitch.conf (ordre résolution)
hosts: files mdns4_minimal [NOTFOUND=return] dns myhostname
```

### Hostname
```bash
# Configuration hostname
hostnamectl set-hostname server.domain.local
hostnamectl status

# Fichiers
echo "server.domain.local" > /etc/hostname
echo "127.0.1.1 server.domain.local server" >> /etc/hosts
```

## 🌉 Réseaux Virtuels & Avancés

### VLAN (802.1Q)
```bash
# Création VLAN avec ip
ip link add link eth0 name eth0.100 type vlan id 100
ip addr add 10.0.100.10/24 dev eth0.100
ip link set eth0.100 up

# Suppression VLAN
ip link delete eth0.100
```

### Bridges
```bash
# Créer bridge
ip link add br0 type bridge
ip link set br0 up

# Ajouter interfaces au bridge
ip link set eth0 master br0
ip link set eth1 master br0

# Configuration IP sur bridge
ip addr add 192.168.1.1/24 dev br0

# Supprimer du bridge
ip link set eth0 nomaster
```

### Tunnels
```bash
# Tunnel GRE
ip tunnel add gre1 mode gre remote 203.0.113.10 local 203.0.113.20
ip addr add 10.0.0.1/30 dev gre1
ip link set gre1 up

# Tunnel VXLAN
ip link add vxlan1 type vxlan id 100 remote 203.0.113.10 dstport 4789
```

## 🔥 Firewall & Forwarding

### IP Forwarding
```bash
# Activation temporaire
echo 1 > /proc/sys/net/ipv4/ip_forward
sysctl net.ipv4.ip_forward=1

# Activation permanente (/etc/sysctl.conf)
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1

# Application
sysctl -p
```

### NAT avec iptables/nftables
```bash
# iptables (legacy)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

# nftables (moderne)
nft add table ip nat
nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }
nft add rule ip nat postrouting oif "eth0" masquerade
```

## 🔍 Diagnostic & Monitoring

### Connectivité Réseau
```bash
# Tests ping
ping 8.8.8.8                # Connectivité IPv4
ping6 2001:4860:4860::8888  # Connectivité IPv6
ping -c 4 -i 0.5 target     # 4 pings, interval 0.5s

# Traceroute
traceroute 8.8.8.8          # Chemin réseau IPv4
traceroute6 2001:4860:4860::8888  # IPv6
mtr 8.8.8.8                 # Traceroute continu + stats
```

### Ports & Connexions
```bash
# Écoute et connexions
ss -tuln                    # Ports en écoute (moderne)
ss -tulpn                   # Avec PID processus
ss -i                       # Infos interfaces
netstat -tuln               # Legacy alternative
lsof -i :80                 # Processus sur port 80

# Connexions actives
ss -t state established     # Connexions TCP établies
ss -u                       # Connexions UDP
ss dst 192.168.1.1         # Connexions vers IP
```

### Analyse Trafic
```bash
# Monitoring temps réel
iftop -i eth0               # Trafic par connexion
nethogs eth0                # Trafic par processus
nload eth0                  # Bande passante interface
vnstat -i eth0              # Statistiques historiques

# Capture paquets
tcpdump -i eth0 -n host 192.168.1.1     # Trafic vers/depuis IP
tcpdump -i any -n port 80                # Port 80 toutes interfaces
tcpdump -i eth0 -w capture.pcap          # Sauver capture
```

### DNS & Résolution
```bash
# Tests résolution
dig @8.8.8.8 domain.com A   # Query DNS spécifique
dig +short domain.com       # Réponse courte
dig -x 8.8.8.8              # Reverse DNS
host domain.com             # Résolution simple
nslookup domain.com         # Interface interactive

# Cache DNS
dig domain.com +trace       # Trace résolution complète
systemd-resolve --flush-caches  # Vider cache systemd
```

## 🛠️ Outils Spécialisés

### Analyse Réseau
```bash
# Informations matériel
ethtool eth0                # État interface Ethernet
ethtool -i eth0             # Infos driver
ethtool -S eth0             # Statistiques détaillées
iwconfig wlan0              # Configuration Wi-Fi (legacy)
iw dev wlan0 info           # Infos Wi-Fi (moderne)

# Calculs réseau
ipcalc 192.168.1.0/24       # Calculs subnet
sipcalc 192.168.1.0/24      # Alternative avec plus d'infos
```

### Surveillance
```bash
# Monitoring continu
watch -n 1 'ss -tuln | grep :80'         # Surveiller port
watch -n 2 'ip -s link show eth0'        # Stats interface
sar -n DEV 1 10                          # Stats réseau sar
```

## 💡 Bonnes Pratiques

### Configuration Résistante
- ✅ **Tester avant production** : `ping`, `traceroute` après changements
- ✅ **Sauvegarder configs** avant modifications
- ✅ **Utiliser connexions multiples** pour éviter coupures
- ✅ **Documenter changements** avec dates et raisons

### Sécurité Réseau
- ✅ **Désactiver interfaces** non utilisées
- ✅ **Configurer firewall** approprié (iptables/nftables)
- ✅ **Surveiller connexions** anormales avec `ss` et `netstat`
- ✅ **Utiliser VLANs** pour segmentation

### Diagnostic Méthodique
1. **Couche physique** : Câbles, LED interfaces
2. **Couche liaison** : `ip link show`, `ethtool`
3. **Couche réseau** : `ip addr`, `ip route`
4. **Couche transport** : `ss`, `netstat`
5. **Couche application** : Tests spécifiques (HTTP, DNS)

---
**💡 Memo** : `ip` pour config temporaire, fichiers config pour persistance, `systemctl restart` pour appliquer !