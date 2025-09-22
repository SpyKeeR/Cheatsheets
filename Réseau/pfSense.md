# 🛡️ pfSense — Cheatsheet Complète

## 🚀 Installation & Configuration Initiale

### Première Configuration
```bash
# Console pfSense (après install)
1) Assign Interfaces
2) Set interface IP address
8) Shell (pour commandes avancées)

# Interface web: https://IP_LAN (défaut: 192.168.1.1)
# Login: admin / pfsense
```

### Interfaces Réseau
| Interface | Rôle | Configuration Type |
|-----------|------|-------------------|
| **WAN** | Internet | DHCP/Static/PPPoE |
| **LAN** | Réseau interne | Static (192.168.1.1/24) |
| **OPT1/DMZ** | Services exposés | Static |
| **OPT2/GUEST** | Invités | Static isolé |

### Configuration Basique
```
System > General Setup:
├── Hostname: pfsense.domain.local
├── Domain: domain.local
├── DNS: 8.8.8.8, 1.1.1.1
├── Timezone: Europe/Paris
└── NTP: pool.ntp.org
```

## 🌐 Configuration Réseau

### DHCP Server
```
Services > DHCP Server > LAN:
├── Enable: ✓
├── Range: 192.168.1.100 - 192.168.1.200
├── DNS: 192.168.1.1
├── Gateway: 192.168.1.1
└── Lease Time: 7200 (2h)
```

### DNS Resolver (Unbound)
```
Services > DNS Resolver:
├── Enable: ✓
├── Network Interfaces: LAN, Localhost
├── Outgoing Network Interfaces: WAN
├── Register DHCP leases: ✓
└── Register DHCP static mappings: ✓
```

### VLAN Configuration
```bash
# Interfaces > Assignments > VLANs
VLAN 10: LAN (Admin)
VLAN 20: DMZ (Serveurs)
VLAN 30: GUEST (Invités)
VLAN 99: MGMT (Management)
```

## 🔒 Pare-feu & Règles

### Philosophie des Règles
- **Default Deny** : Tout bloqué par défaut
- **Stateful** : Suivi des connexions établies
- **Ordre** : Première règle qui match s'applique
- **Interface** : Règles sur interface d'entrée

### Règles Type par Interface
```
WAN (Restrictives):
├── Block RFC1918 (private networks)
├── Block Bogons
└── Allow specific services (VPN, port forwards)

LAN (Permissives):
├── Allow LAN to any (avec exceptions)
├── Block LAN to DMZ mgmt
└── Allow LAN to DMZ services

DMZ (Restrictives):
├── Allow DMZ to WAN (updates)
├── Block DMZ to LAN
└── Allow specific DMZ services
```

### Alias Utiles
```
Firewall > Aliases:
├── RFC1918: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
├── MGMT_PORTS: 22,23,80,443,3389
├── WEB_PORTS: 80,443
└── MAIL_PORTS: 25,110,143,993,995
```

## 🔄 NAT & Redirections

### Types de NAT
| Type | Usage | Configuration |
|------|-------|---------------|
| **Outbound** | LAN → WAN | Auto/Hybrid/Manual |
| **Port Forward** | WAN → LAN service | Interface + port + destination |
| **1:1 NAT** | IP publique ↔ IP privée | Mapping complet |
| **NPt** | IPv6 prefix translation | IPv6 networks |

### Port Forwarding Exemple
```
Firewall > NAT > Port Forward:
Interface: WAN
Protocol: TCP
Source: any
Destination: WAN address
Destination port: 443
Redirect target IP: 192.168.1.100 (web server)
Redirect target port: 443
Description: HTTPS to Web Server
NAT reflection: Pure NAT
```

### Outbound NAT Modes
- **Automatic** : Règles générées automatiquement
- **Hybrid** : Auto + règles personnalisées
- **Manual** : Contrôle total des règles
- **Disabled** : Pas de NAT sortant

## 🔐 VPN Configuration

### OpenVPN Server (Remote Access)
```
VPN > OpenVPN > Servers > Add:
Server Mode: Remote Access (SSL/TLS)
Protocol: UDP
Port: 1194
Description: Remote Users
TLS Configuration:
├── Peer Certificate Authority: Internal CA
├── Server Certificate: pfSense Server Cert
├── DH Parameter Length: 2048
└── Encryption Algorithm: AES-256-GCM

Tunnel Settings:
├── IPv4 Tunnel Network: 10.8.0.0/24
├── IPv4 Local Network: 192.168.1.0/24
├── IPv4 Remote Network: (leave blank)
├── Concurrent Connections: 10
└── Duplicate Connections: ✓
```

### Client Export Package
```bash
# Installation du package
System > Package Manager > Available Packages
Search: openvpn-client-export
Install: openvpn-client-export

# Export client
VPN > OpenVPN > Client Export:
├── Remote Access Server: Server créé
├── Hostname Resolution: Other (specify)
├── Host Name Resolution: vpn.domain.com
└── Certificate Export Options selon besoin
```

### IPsec Site-to-Site
```
VPN > IPsec > Tunnels > Add P1:
General Information:
├── Disabled: ✗
├── Key Exchange: IKEv2
├── Interface: WAN
├── Remote Gateway: IP_distant
└── Description: Site Branch

Phase 1 Proposal:
├── Authentication Method: Mutual PSK
├── My identifier: My IP address
├── Peer identifier: Peer IP address
├── Pre-Shared Key: [strong key]
├── Encryption Algorithm: AES 256
├── Hash Algorithm: SHA256
└── DH Group: 14 (2048 bit)

Add P2 (depuis P1):
General Information:
├── Mode: Tunnel IPv4
├── Local Network: 192.168.1.0/24
├── Remote Network: 192.168.2.0/24
└── Description: LAN to LAN

Phase 2 Proposal:
├── Protocol: ESP
├── Encryption: AES 256
├── Hash: SHA256
└── PFS key group: 14 (2048 bit)
```

### WireGuard Configuration
```
VPN > WireGuard > Settings:
├── Enable WireGuard: ✓
└── Keep Configuration: ✓

VPN > WireGuard > Tunnels > Add:
Interface Configuration:
├── Description: WG Server
├── Listen Port: 51820
├── Interface Keys: Generate
└── Interface Addresses: 10.10.10.1/24

Peers > Add:
├── Description: Client1
├── Public Key: [client public key]
├── Allowed IPs: 10.10.10.2/32
└── Endpoint: (leave blank pour server)
```

### Comparatif VPN
| Protocol | Port | Performance | Sécurité | Mobile | NAT Traversal |
|----------|------|-------------|----------|--------|---------------|
| **OpenVPN** | 1194 UDP/TCP | Moyenne | Excellente | Bon | Excellent |
| **IPsec IKEv2** | 500/4500 UDP | Élevée | Excellente | Excellent | Bon |
| **WireGuard** | 51820 UDP | Très élevée | Excellente | Excellent | Excellent |
| **L2TP/IPsec** | 500/1701/4500 | Moyenne | Bonne | Bon | Difficile |
| **SSTP** | 443 TCP | Moyenne | Bonne | Windows | Excellent |

## 🛡️ Sécurité & Certificats

### PKI Interne
```
System > Cert. Manager > CAs > Add:
Method: Create an internal Certificate Authority
Descriptive name: Internal CA
Key Type: RSA
Key Length: 2048
Digest Algorithm: SHA256
Lifetime: 3650
Common Name: Internal CA
Country Code: FR
```

### Certificat Serveur
```
System > Cert. Manager > Certificates > Add:
Method: Create an internal Certificate
Descriptive name: pfSense WebGUI
Certificate authority: Internal CA
Key Type: RSA
Key Length: 2048
Certificate Type: Server Certificate
Common Name: pfsense.domain.local
Alternative Names: IP:192.168.1.1, DNS:pfsense
```

### LDAPS Authentication
```
System > User Manager > Authentication Servers > Add:
Descriptive name: Active Directory
Type: LDAP
Hostname: dc.domain.local
Port: 636
Transport: SSL - Encrypted
Peer Certificate Authority: [CA importée]
Protocol version: 3
Server Timeout: 25
Search scope: Entire Subtree
Base DN: DC=domain,DC=local
Authentication containers: CN=Users,DC=domain,DC=local
Bind credentials:
├── Username: pfservice@domain.local
└── Password: [password]
User naming attribute: sAMAccountName
Group naming attribute: cn
Group member attribute: memberOf
```

### Suricata IDS/IPS
```bash
# Installation
System > Package Manager > Suricata

# Configuration
Services > Suricata > Global Settings:
├── Enable Suricata: ✓
├── Enable eve JSON log: ✓
└── Live rule swap: ✓

Interfaces > Add:
├── Interface: LAN
├── Description: LAN Monitor
└── Block Offenders: ✓ (pour IPS)

Rules > Update:
├── Snort VRT: ✓
├── Emerging Threats Open: ✓
└── Update Interval: 12 hours
```

## 📊 Monitoring & Logging

### Logging Configuration
```
Status > System Logs > Settings:
├── Reverse Display: ✓
├── Number of log entries: 500
├── Log file size: 512000
└── Number of archived logs: 5

Log Destinations:
├── Enable syslog: ✓
├── Remote syslog server: 192.168.1.10:514
└── Log everything: ✓
```

### SNMP Monitoring
```
Services > SNMP:
├── Enable: ✓
├── Polling Port: 161
├── System Contact: admin@domain.com
├── System Location: Datacenter
├── Read Community String: [custom]
└── Network(s): 192.168.1.0/24,10.10.10.0/24
```

### Dashboard Widgets Utiles
- **System Information** : Version, uptime, température
- **Interface Statistics** : Trafic par interface
- **Gateway Status** : État connexions WAN
- **Services Status** : État services critiques
- **Traffic Graphs** : Graphiques temps réel

## 🚀 Services Additionnels

### Captive Portal
```
Services > Captive Portal > Add:
Zone Configuration:
├── Zone name: GUEST
├── Zone description: Guest Access
├── Interface: OPT2_GUEST
└── Maximum concurrent connections: 50

Authentication:
├── Authentication method: Local User Manager
├── Idle timeout: 60 minutes
├── Hard timeout: 480 minutes
└── Pass-through MAC automatic: ✓
```

### Proxy Squid
```bash
# Installation package
System > Package Manager > squid

# Configuration
Services > Squid Proxy Server > General:
├── Enable Squid Proxy: ✓
├── Proxy interface: LAN
├── Proxy port: 3128
├── ICP port: (disabled)
└── Visible hostname: proxy.domain.local

Cache Management:
├── Hard disk cache size: 1024 MB
├── Hard disk cache location: /var/squid/cache
├── Level 1 directories: 16
└── Memory cache size: 256 MB
```

### pfBlockerNG
```bash
# Installation
System > Package Manager > pfBlockerNG

# Configuration
Firewall > pfBlockerNG > IP > Add:
├── Alias Name: Malware_Domains
├── Description: Malicious domains
├── Format: Domain/Host format
├── Action: Deny Both
└── Source: Spamhaus DROP, Emerging Threats

DNSBL Configuration:
├── Enable DNSBL: ✓
├── DNSBL Virtual IP: 10.10.10.1
└── Categories: Ads, Malware, Phishing
```

## 🔧 Haute Disponibilité (CARP)

### Configuration CARP
```
System > High Avail. Sync:
Synchronize Config to IP: [IP secondary]
Remote System Username: admin
Remote System Password: [password]
Synchronize:
├── States: ✓
├── Nat: ✓
├── Rules: ✓
├── Schedules: ✓
├── Virtual IPs: ✓
├── Users and Groups: ✓
└── Certificates: ✓

Firewall > Virtual IPs > Add:
Type: CARP
Interface: LAN
Address: 192.168.1.254/24 (VIP)
VHID Group: 1
Advertising Frequency: Base 1, Skew 0 (master)
Password: [strong password]
```

### États CARP
- **MASTER** : Unité active
- **BACKUP** : Unité en standby
- **INIT** : Initialisation

## 💾 Sauvegarde & Restauration

### Configuration Backup
```
Diagnostics > Backup & Restore:
├── Backup area: All
├── Skip packages: ✗
├── Skip RRD data: ✓
└── Encrypt: ✓ (recommandé)

# Automatic backup
System > Package Manager > AutoConfigBackup
```

### Gold Config
```bash
# Sauvegardes critiques
1. Configuration complète (XML)
2. Certificats & clés privées
3. Liste des packages installés
4. Règles personnalisées
5. Documentation réseau
```

## 🔍 Troubleshooting

### Diagnostics Essentiels
```bash
# Console pfSense
pfctl -s rules          # Règles firewall actives
pfctl -s nat            # Règles NAT actives
pfctl -s states         # États connexions
pfctl -s info           # Statistiques firewall
netstat -rn             # Table routage
tcpdump -i em0          # Capture trafic interface

# Interface web
Diagnostics > States:   # Connexions actives
Diagnostics > Routing:  # Table routage
Diagnostics > ARP:      # Table ARP
Status > Traffic Graph: # Trafic temps réel
```

### Logs Importants
| Log | Location | Description |
|-----|----------|-------------|
| **System** | Status > System Logs > System | Démarrages, erreurs système |
| **Firewall** | Status > System Logs > Firewall | Règles bloquées/autorisées |
| **DHCP** | Status > System Logs > DHCP | Baux DHCP |
| **VPN** | Status > System Logs > VPN | Connexions VPN |
| **Gateway** | Status > System Logs > Gateways | Monitoring WAN |

### Commandes Shell Utiles
```bash
# Redémarrage services
/etc/rc.restart_webgui
/etc/rc.reload_all
service pf restart

# Monitoring
top                     # Processus
df -h                   # Espace disque
ifconfig -a             # Interfaces réseau
route -n show           # Routes
netstat -an             # Connexions réseau

# Certificats
openssl x509 -in /tmp/cert.pem -text -noout
```

## 📚 Maintenance & Bonnes Pratiques

### Checklist Sécurité
- [ ] Changer mot de passe admin par défaut
- [ ] Désactiver HTTP (forcer HTTPS)
- [ ] Limiter accès WebGUI par IP
- [ ] Activer 2FA pour admin
- [ ] Utiliser certificats SSL valides
- [ ] Configurer NTP (synchronisation temps)
- [ ] Activer logging approprié
- [ ] Sauvegardes automatiques
- [ ] Monitoring et alertes
- [ ] Mise à jour régulière

### Maintenance Périodique
```bash
# Hebdomadaire
- Vérifier logs système/firewall
- Monitoring utilisation ressources
- Vérifier état CARP (HA)
- Test connectivité VPN

# Mensuel
- Mise à jour pfSense
- Mise à jour packages
- Sauvegarde configuration
- Audit règles firewall
- Vérification certificats

# Trimestriel
- Test restauration backup
- Audit sécurité complet
- Documentation mise à jour
- Formation équipe
```

---
**💡 Ressources** : [pfSense Docs](https://docs.netgate.com/pfsense/) • [Netgate Forum](https://forum.netgate.com/) • [pfSense Book](https://www.netgate.com/resources/pfsense-book)
