# 🔀 Cisco IOS Switching — Aide-mémoire

## 🌐 Concepts Fondamentaux

### Fonctionnement Switch
- **MAC Learning** : Table MAC associe adresses MAC ↔ ports
- **Forwarding** : 
  - Destination connue → Unicast sur port spécifique
  - Inconnue → Flood sur tous ports du VLAN (sauf source)
  - Broadcast/Multicast → Diffusion à tous ports VLAN

### Modes de Commutation
| Mode | Description | Latence | Vérification |
|------|-------------|---------|--------------|
| **Store-and-Forward** | Stocke trame complète | Élevée | FCS + taille |
| **Cut-Through** | Forward dès MAC dest | Faible | Aucune |
| **Fragment-Free** | Lit 64 premiers octets | Moyenne | Collisions |

## 🔌 Configuration Interfaces

### Paramètres Physiques
```cisco
# Configuration duplex/vitesse
interface fastethernet0/1
 duplex full                  # auto, full, half
 speed 100                    # auto, 10, 100, 1000
 description "Serveur Web"
 no shutdown

# Auto-négociation (recommandé)
interface gigabitethernet0/1
 duplex auto
 speed auto
```

### Types de Duplex
- **Full-Duplex** : Émission/réception simultanées, pas de CSMA/CD
- **Half-Duplex** : Un sens à la fois, CSMA/CD actif
- **Auto** : Négociation automatique (recommandé)

## 🏷️ VLANs (Virtual LANs)

### Gestion VLANs
```cisco
# Créer VLAN
vlan 10
 name ADMIN
vlan 20
 name USERS
vlan 30
 name SERVERS

# Voir VLANs
show vlan brief
show vlan id 10
```

### VLANs par Défaut
- **VLAN 1** : Management (ne pas utiliser pour données)
- **VLAN 1002-1005** : Token Ring/FDDI (legacy)
- **Plage normale** : 1-1005 (stockés dans vlan.dat)
- **Plage étendue** : 1006-4094 (stockés dans running-config)

## 🔗 Configuration Ports

### Port Access (Un VLAN)
```cisco
interface fastethernet0/5
 switchport mode access       # Force mode access
 switchport access vlan 10    # Assigner au VLAN 10
 switchport nonegotiate       # Désactiver DTP
 description "PC Bureau 1"
```

### Port Trunk (Plusieurs VLANs)
```cisco
interface gigabitethernet0/1
 switchport mode trunk                    # Force mode trunk
 switchport trunk native vlan 99         # VLAN natif (non-taggé)
 switchport trunk allowed vlan 10,20,30  # VLANs autorisés
 switchport nonegotiate                   # Désactiver DTP
 description "Liaison vers Switch Core"
```

### Voice VLAN (Téléphonie IP)
```cisco
interface fastethernet0/10
 switchport mode access
 switchport access vlan 20        # VLAN données
 switchport voice vlan 110        # VLAN voix
 mls qos trust cos               # Trust CoS pour QoS
 spanning-tree portfast          # Transition rapide
 description "Téléphone IP + PC"
```

## 📊 Dynamic Trunking Protocol (DTP)

### Modes DTP
| Mode | Envoie DTP | Forme Trunk Si |
|------|------------|----------------|
| **trunk** | Oui | Toujours |
| **dynamic desirable** | Oui | Autre côté trunk/desirable/auto |
| **dynamic auto** | Oui | Autre côté trunk/desirable |
| **access** | Non | Jamais |
| **nonegotiate** | Non | Si configuré trunk |

### Bonnes Pratiques DTP
```cisco
# Désactiver DTP (sécurité)
interface range fastethernet0/1-24
 switchport nonegotiate

# Ports vers utilisateurs finaux
switchport mode access
switchport nonegotiate
```

## 🌉 Spanning Tree Protocol (STP)

### États des Ports STP
| État | Écoute BPDU | Apprend MAC | Forward Données | Durée |
|------|-------------|-------------|-----------------|-------|
| **Disabled** | Non | Non | Non | — |
| **Blocking** | Oui | Non | Non | 20s |
| **Listening** | Oui | Non | Non | 15s |
| **Learning** | Oui | Oui | Non | 15s |
| **Forwarding** | Oui | Oui | Oui | — |

### Configuration STP Basique
```cisco
# Modifier priorité (devenir root)
spanning-tree vlan 1 priority 4096    # Multiples de 4096

# PortFast (ports terminaux)
interface fastethernet0/1
 spanning-tree portfast              # Transition directe forwarding

# BPDU Guard (sécurité PortFast)
interface fastethernet0/1
 spanning-tree bpduguard enable     # Shutdown si BPDU reçu

# Global PortFast + BPDU Guard
spanning-tree portfast default
spanning-tree portfast bpduguard default
```

### Variantes STP
- **STP** : 802.1D (50s convergence)
- **RSTP** : 802.1w (Rapid, <6s convergence)
- **PVST+** : Per-VLAN STP (Cisco)
- **RPVST+** : Rapid Per-VLAN STP (Cisco, défaut)

## 🔧 Interface Virtuelle Switch (SVI)

### Management VLAN
```cisco
# Configuration SVI pour management
interface vlan 1
 ip address 192.168.1.100 255.255.255.0
 description "Management Interface"
 no shutdown

# Passerelle par défaut (Switch L2)
ip default-gateway 192.168.1.1

# Vérification
show ip interface brief
ping 192.168.1.1
```

## 📋 EtherChannel (Agrégation Liens)

### Protocoles d'Agrégation
| Protocole | Type | Négociation |
|-----------|------|-------------|
| **LACP** | 802.3ad (standard) | Dynamique |
| **PAgP** | Cisco propriétaire | Dynamique |
| **Static** | Pas de protocole | Statique |

### Configuration EtherChannel
```cisco
# LACP (recommandé)
interface range gigabitethernet0/1-2
 channel-group 1 mode active    # active/passive
 
interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30

# Vérification
show etherchannel summary
show etherchannel 1 detail
```

## 📊 Monitoring & Diagnostic

### Commandes Show Essentielles
```cisco
# VLANs
show vlan brief                 # Résumé VLANs
show vlan id 10                # VLAN spécifique
show interfaces switchport     # Config switchport

# MAC Address Table
show mac address-table          # Table MAC complète
show mac address-table vlan 10 # Par VLAN
show mac address-table interface fa0/1  # Par interface
show mac address-table count   # Compteurs

# STP
show spanning-tree             # Toutes instances
show spanning-tree vlan 1      # VLAN spécifique
show spanning-tree interface fa0/1  # Port spécifique
show spanning-tree summary     # Résumé

# Interfaces
show interfaces status         # État compact
show interfaces trunk         # Ports trunk
show interfaces fa0/1 switchport  # Config port spécifique
```

### Diagnostic Connectivité
```cisco
# Détection collision/erreurs
show interfaces fa0/1 | include error
show interfaces counters errors

# Utilisation ports
show interfaces | include line protocol
show mac address-table aging-time
```

## 🛡️ Sécurité Switching

### Port Security
```cisco
interface fastethernet0/5
 switchport port-security                    # Activer
 switchport port-security maximum 2          # Max 2 MAC
 switchport port-security mac-address sticky # Apprendre dynamiquement
 switchport port-security violation shutdown # Action si violation

# Monitoring
show port-security interface fa0/5
show port-security address

# Recovery port err-disabled
interface fa0/5
 shutdown
 no shutdown
```

### DHCP Snooping
```cisco
# Configuration globale
ip dhcp snooping
ip dhcp snooping vlan 10,20

# Ports de confiance (serveurs DHCP/uplinks)
interface gigabitethernet0/1
 ip dhcp snooping trust

# Rate limiting
interface range fastethernet0/1-24
 ip dhcp snooping limit rate 10

# Vérification
show ip dhcp snooping
show ip dhcp snooping binding
```

## ⚙️ QoS Basique (VoIP)

### Configuration Voice
```cisco
# Global QoS
mls qos

# Port téléphone IP
interface fastethernet0/10
 switchport access vlan 20         # VLAN données
 switchport voice vlan 110         # VLAN voix
 mls qos trust cos                 # Trust CoS markings
 auto qos voip cisco-phone         # Template QoS Cisco

# Vérification
show mls qos interface fa0/10
show auto qos
```

## 🔧 Commandes Utilitaires

### Gestion Configuration
```cisco
# Sauvegarde
copy running-config startup-config
copy running-config tftp:

# VLAN Database
show vlan brief
delete flash:vlan.dat          # Reset VLAN database
```

### Dépannage Courant
```cisco
# Reset interface
interface fa0/1
 shutdown
 no shutdown

# Clear MAC table
clear mac address-table dynamic
clear mac address-table dynamic interface fa0/1

# STP troubleshooting
debug spanning-tree events
show spanning-tree blockedports
```

## 💡 Bonnes Pratiques

### Conception
- ✅ **VLAN 1** : Réservé management uniquement
- ✅ **Native VLAN** : Changer par défaut (sécurité)
- ✅ **Voice VLAN** : Séparé du VLAN données
- ✅ **Trunk** : Limiter VLANs autorisés

### Sécurité
- ✅ **DTP** : Désactiver (`nonegotiate`)
- ✅ **Ports inutilisés** : Shutdown + VLAN parking
- ✅ **PortFast** : Uniquement ports terminaux
- ✅ **BPDU Guard** : Activer sur PortFast

### Performance
- ✅ **EtherChannel** : Agrégation uplinks critiques
- ✅ **QoS** : Configurer pour VoIP
- ✅ **STP** : Rapid-PVST+ par défaut
- ✅ **MAC aging** : Ajuster selon environnement

### Maintenance
```cisco
# Documentation automatique
show running-config | redirect tftp://server/config.txt
show vlan brief | redirect tftp://server/vlans.txt
show mac address-table | redirect tftp://server/mac-table.txt

# Vérifications périodiques
show logging | include %LINK
show interfaces counters errors
show spanning-tree root
```

---
**💡 Memo** : `nonegotiate` sur tous ports, PortFast + BPDU Guard sur access, trunk limité aux VLANs nécessaires !