# 🛡️ Cisco IOS Security — Aide-mémoire

## 🔐 Authentification & Mots de Passe

### Sécurisation Mode Privilégié
```cisco
# Mots de passe enable
enable password cisco123      # Stocké en clair (éviter)
enable secret cisco123        # Haché MD5 (recommandé)

# Chiffrement mots de passe existants
service password-encryption   # Type 7 (faible mais mieux que rien)

# Vérification
show running-config | include enable
```

### Configuration SSH
```cisco
# Prérequis SSH (ordre important)
hostname SWITCH-01
ip domain-name entreprise.local
crypto key generate rsa modulus 2048

# Configuration SSH
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3

# Utilisateur local
username admin privilege 15 secret AdminPass123

# Ligne VTY sécurisée
line vty 0 15
 login local                  # Auth par utilisateur local
 transport input ssh          # SSH seulement (pas Telnet)
 exec-timeout 10 0           # Timeout 10 min
 logging synchronous          # Évite mélange messages/saisie
```

### Sécurisation Console
```cisco
line console 0
 password ConsolePass123
 login                       # Demander mot de passe
 exec-timeout 5 0           # Timeout 5 min
 logging synchronous
```

### Banner de Sécurité
```cisco
banner motd #
************************************************
*  ACCES AUTORISE UNIQUEMENT                    *
*  Toute activité est surveillée et journalisée *
*  Déconnectez-vous immédiatement si non        *
*  autorisé - Violations = poursuites           *
************************************************
#
```

## 🔌 Port Security (Switch)

### Configuration de Base
```cisco
# Sur interface access
interface fastethernet0/1
 switchport mode access
 switchport port-security                    # Activer port security
 switchport port-security maximum 2          # Max 2 MAC addresses
 switchport port-security mac-address sticky # Apprendre MAC dynamiquement
 switchport port-security violation shutdown # Action si violation
```

### Modes de Violation
| Mode | Action Violation | Trafic | Log | Compteur |
|------|------------------|--------|-----|----------|
| **Protect** | Drop silencieux | ✓ | ✗ | ✗ |
| **Restrict** | Drop + log | ✓ | ✓ | ✓ |
| **Shutdown** | Interface err-disabled | ✗ | ✓ | ✓ |

### MAC Address Types
```cisco
# MAC statique (permanente)
switchport port-security mac-address 1234.5678.9abc

# MAC sticky (sauvée en running-config)
switchport port-security mac-address sticky

# Combinaison
switchport port-security mac-address sticky 1234.5678.9abc
```

### Monitoring & Recovery
```cisco
# Vérifications
show port-security                          # Vue globale
show port-security interface fa0/1          # Interface spécifique  
show port-security address                  # MAC addresses apprises

# Recovery depuis err-disabled
interface fastethernet0/1
 shutdown
 no shutdown

# Recovery automatique (optionnel)
errdisable recovery cause psecure-violation
errdisable recovery interval 300            # 5 minutes
```

## 🌉 Spanning Tree Security

### PortFast & Protection
```cisco
# PortFast (ports terminaux uniquement)
interface fastethernet0/1
 spanning-tree portfast              # Transition immédiate à forwarding

# BPDU Guard (shutdown si BPDU reçu)
interface fastethernet0/1
 spanning-tree bpduguard enable     # Sécurise PortFast

# Global PortFast + BPDU Guard
spanning-tree portfast default      # Tous ports access
spanning-tree portfast bpduguard default
```

### Root Guard & Loop Guard
```cisco
# Root Guard (empêche devenir root bridge)
interface gigabitethernet0/1
 spanning-tree guard root

# Loop Guard (détecte perte BPDU)
interface gigabitethernet0/2  
 spanning-tree guard loop

# Global Loop Guard
spanning-tree loopguard default
```

## 🛡️ DHCP Snooping & DAI

### DHCP Snooping Configuration
```cisco
# Activation globale
ip dhcp snooping
ip dhcp snooping vlan 10,20,30

# Ports de confiance (uplinks/serveurs DHCP)
interface gigabitethernet0/1
 ip dhcp snooping trust

# Rate limiting (protection DoS)
interface range fastethernet0/1-24
 ip dhcp snooping limit rate 10     # 10 paquets DHCP/sec max

# Options avancées
ip dhcp snooping information option # Option 82
ip dhcp snooping database flash:dhcp_snooping.db
```

### Dynamic ARP Inspection (DAI)
```cisco
# Prérequis : DHCP Snooping actif
ip arp inspection vlan 10,20,30

# Ports de confiance (pas de validation ARP)
interface gigabitethernet0/1
 ip arp inspection trust

# Rate limiting DAI
interface range fastethernet0/1-24  
 ip arp inspection limit rate 15    # 15 paquets ARP/sec

# Validation supplémentaire
ip arp inspection validate src-mac dst-mac ip
```

### Monitoring DHCP/ARP Security
```cisco
show ip dhcp snooping                # Configuration globale
show ip dhcp snooping binding        # Table bindings IP-MAC
show ip arp inspection interfaces    # Interfaces DAI
show ip arp inspection statistics    # Statistiques violations
```

## 🚫 Access Control Lists (ACLs)

### ACL Standard (1-99, 1300-1999)
```cisco
# Numérotée (proche destination)
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 deny any

# Nommée (plus lisible)
ip access-list standard ADMIN_HOSTS
 permit host 192.168.1.100
 permit 192.168.1.0 0.0.0.15
 deny any

# Application
interface gigabitethernet0/1
 ip access-group ADMIN_HOSTS in
```

### ACL Étendue (100-199, 2000-2699)
```cisco
# Contrôle granulaire (proche source)
ip access-list extended FIREWALL_RULES
 permit tcp 192.168.1.0 0.0.0.255 any eq www
 permit tcp 192.168.1.0 0.0.0.255 any eq https
 permit udp 192.168.1.0 0.0.0.255 any eq domain
 permit icmp any any echo-reply
 deny ip any any log

# Application
interface serial0/0/0
 ip access-group FIREWALL_RULES out
```

### Wildcards & Keywords
```cisco
# Wildcards (inverse du masque)
# /24 (255.255.255.0) → wildcard 0.0.0.255
# /30 (255.255.255.252) → wildcard 0.0.0.3

# Keywords pratiques
permit ip host 192.168.1.1 any        # Une seule IP source
permit tcp any host 10.1.1.100 eq 80  # Une seule IP destination  
deny ip any any                        # Tout bloquer
```

### ACL pour VTY (Telnet/SSH)
```cisco
# Restreindre accès management
ip access-list standard VTY_ACCESS
 permit 192.168.1.0 0.0.0.255    # Réseau admin
 permit host 10.1.1.50           # Poste admin
 deny any log

line vty 0 15
 access-class VTY_ACCESS in       # Appliquer sur VTY
```

## 🔄 Network Address Translation (NAT)

### NAT Dynamique avec PAT/Overload
```cisco
# Définir interfaces
interface gigabitethernet0/0
 ip nat outside                   # Interface publique

interface gigabitethernet0/1  
 ip nat inside                    # Interface privée

# ACL pour trafic à traduire
access-list 1 permit 192.168.1.0 0.0.0.255

# NAT avec surcharge (PAT)
ip nat inside source list 1 interface gigabitethernet0/0 overload
```

### NAT Statique (Port Forwarding)
```cisco
# Serveur web interne
ip nat inside source static tcp 192.168.1.100 80 interface gigabitethernet0/0 80

# Serveur SSH
ip nat inside source static tcp 192.168.1.100 22 203.0.113.1 22

# Pool d'adresses publiques
ip nat pool PUBLIC_POOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0
ip nat inside source list 1 pool PUBLIC_POOL overload
```

### Monitoring NAT
```cisco
show ip nat translations         # Table translations active
show ip nat translations verbose # Détails temporisation
show ip nat statistics          # Compteurs NAT
clear ip nat translation *      # Vider table (debug)

# Debug (attention production)
debug ip nat                    # Traces NAT temps réel
```

## 📡 Wireless Security

### Standards Sécurité WiFi
| Standard | Chiffrement | Sécurité | Usage |
|----------|-------------|----------|--------|
| **WEP** | RC4 | ❌ Très faible | Obsolète |
| **WPA** | TKIP | ⚠️ Faible | Legacy |
| **WPA2** | AES-CCMP | ✅ Bon | Standard actuel |
| **WPA3** | AES-GCMP | ✅ Excellent | Moderne |

### Configuration WPA2/WPA3
```cisco
# SSID sécurisé entreprise
dot11 ssid ENTREPRISE_SECURE
 authentication open
 authentication key-management wpa version 2
 wpa-psk ascii 0 MotDePasseComplexe2024!
 mbssid guest-mode

# Désactiver WPS (vulnérable)
no dot11 extension aironet
```

### Bonnes Pratiques WiFi
- ✅ **WPA3** si supporté, sinon **WPA2-AES uniquement**
- ✅ **SSID non-révélateur** (éviter nom entreprise)
- ✅ **Mots de passe >15 caractères** complexes
- ✅ **VLAN séparé** pour invités
- ✅ **Isolation client** activée sur SSID public
- ❌ **Désactiver WPS** (faille PIN)
- ❌ **Masquer SSID** n'apporte pas de sécurité

## 🔧 Hardening Avancé

### Désactivation Services Inutiles
```cisco
# Services à désactiver généralement
no ip http-server              # Serveur web HTTP
no ip http-secure-server       # Serveur web HTTPS (sauf si nécessaire)
no cdp run                     # Cisco Discovery Protocol
no lldp run                    # Link Layer Discovery Protocol
no ip source-route             # Source routing
no ip finger                   # Service finger
no service pad                 # Packet Assembler/Disassembler

# Services utiles à conserver
service timestamps debug datetime msec  # Timestamps précis
service timestamps log datetime msec
service sequence-numbers              # Numéros séquence logs
logging buffered 16384                # Buffer logs local
```

### Control Plane Protection
```cisco
# Rate limiting management
control-plane
 service-policy input CONTROL_PLANE_POLICY

# Exemple policy-map
policy-map CONTROL_PLANE_POLICY
 class MANAGEMENT_TRAFFIC
  police 1000000 conform-action transmit exceed-action drop
```

### Authentication, Authorization, Accounting (AAA)
```cisco
# AAA avec serveur RADIUS
aaa new-model
aaa authentication login default group radius local
aaa authorization exec default group radius local  
aaa accounting exec default start-stop group radius

# Serveur RADIUS
radius server ISE-SERVER
 address ipv4 192.168.10.100 auth-port 1812 acct-port 1813
 key 7 SharedSecretKey123
```

## 📊 Monitoring & Logging

### Configuration Logging
```cisco
# Niveaux de log (0=emergencies à 7=debugging)
logging buffered 16384 informational     # Buffer local
logging host 192.168.1.50               # Syslog server
logging trap warnings                    # Niveau minimum syslog
logging facility local0                  # Facility code

# Synchronisation temps
ntp server 192.168.1.10
clock timezone CET 1
clock summer-time CEST recurring
```

### SNMP Sécurisé
```cisco
# SNMPv3 (sécurisé)
snmp-server group ADMIN_GROUP v3 priv
snmp-server user admin ADMIN_GROUP v3 auth sha AuthPass123 priv aes 128 PrivPass456

# SNMPv2c (si nécessaire)
snmp-server community ReadOnlyCommunity RO
snmp-server location "Datacenter - Rack A12"
snmp-server contact "admin@entreprise.com"
```

## 🚨 Détection & Réponse

### Event Monitoring
```cisco
# EEM (Embedded Event Manager) exemples
event manager applet HIGH_CPU
 event snmp oid 1.3.6.1.4.1.9.2.1.56.0 get-type exact entry-op ge entry-val 80
 action 1.0 syslog msg "High CPU detected"
 action 2.0 mail server "mail.company.com" to "admin@company.com" subject "Router High CPU"

# Interface monitoring
interface gigabitethernet0/1
 carrier-delay msec 0         # Détection rapide perte liaison
 load-interval 30             # Statistiques toutes les 30s
```

### Security Checklist
- [ ] **Mots de passe** : enable secret configuré, service password-encryption
- [ ] **SSH** : v2 activé, Telnet désactivé, clés RSA >2048 bits
- [ ] **Utilisateurs** : comptes nominatifs avec privilèges appropriés
- [ ] **Banners** : avertissement légal configuré
- [ ] **Services** : inutiles désactivés, essentiels sécurisés
- [ ] **ACLs** : trafic management restreint aux IPs autorisées
- [ ] **Logging** : serveur syslog configuré, NTP synchronisé
- [ ] **SNMP** : v3 si possible, communautés complexes
- [ ] **Interfaces** : non utilisées shutdown
- [ ] **Updates** : IOS à jour avec patches sécurité

---
**💡 Memo** : Sécurité = Defense in Depth → Multiple couches de protection + Principle of Least Privilege !