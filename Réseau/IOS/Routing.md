# 🗺️ Cisco IOS Routage — Aide-mémoire

## 🧭 Concepts Fondamentaux

### Administrative Distance (AD)
| Source | AD | Fiabilité |
|--------|-----|-----------|
| **Interface connectée** | 0 | Maximum |
| **Route statique** | 1 | Très élevée |
| **EIGRP interne** | 90 | Élevée |
| **OSPF** | 110 | Bonne |
| **RIP** | 120 | Faible |
| **EIGRP externe** | 170 | Très faible |
| **Route inconnue** | 255 | Non routable |

### Codes Table de Routage
| Code | Source | Description |
|------|--------|-------------|
| **C** | Connected | Interface directement connectée |
| **L** | Local | Adresse locale de l'interface |
| **S** | Static | Route configurée manuellement |
| **R** | RIP | Routing Information Protocol |
| **D** | EIGRP | Enhanced Interior Gateway Routing Protocol |
| **O** | OSPF | Open Shortest Path First |
| ***** | Default | Route candidate par défaut |

### Processus de Routage
```
1. Réception paquet → Désencapsulation L2
2. Vérification TTL → Décrémentation (évite boucles)
3. Consultation table routage → Longest prefix match
4. Détermination next-hop/interface sortie
5. Réencapsulation L2 → Envoi sur interface
```

## 🛣️ Routes Statiques

### Configuration de Base
```cisco
# Route statique standard
ip route 192.168.2.0 255.255.255.0 192.168.1.1
ip route 10.0.0.0 255.0.0.0 Serial0/0/0

# Route par défaut
ip route 0.0.0.0 0.0.0.0 192.168.1.1
ip route 0.0.0.0 0.0.0.0 Serial0/0/0

# Route avec distance administrative personnalisée
ip route 192.168.3.0 255.255.255.0 192.168.1.2 150    # AD=150 (backup)

# Route statique flottante (backup automatique)
ip route 192.168.4.0 255.255.255.0 10.1.1.1 1         # Route principale
ip route 192.168.4.0 255.255.255.0 10.1.1.2 50        # Route backup
```

### Types de Routes Statiques
- **Directly Attached** : `ip route 0.0.0.0 0.0.0.0 Serial0/0/0`
- **Recursive** : `ip route 192.168.1.0 255.255.255.0 10.1.1.1`
- **Fully Specified** : `ip route 192.168.1.0 255.255.255.0 Serial0/0/0 10.1.1.1`

## 🔄 Protocoles de Routage Dynamique

### RIP (Routing Information Protocol)

#### Caractéristiques RIP
| Version | Type | Métrique | Max Hops | Transport | Multicast |
|---------|------|----------|----------|-----------|-----------|
| **RIPv1** | Classful | Hop Count | 15 | UDP:520 | Broadcast |
| **RIPv2** | Classless | Hop Count | 15 | UDP:520 | 224.0.0.9 |
| **RIPng** | IPv6 | Hop Count | 15 | UDP:521 | FF02::9 |

#### Configuration RIP
```cisco
# Configuration basique RIPv2
router rip
 version 2
 network 10.0.0.0
 network 192.168.1.0
 no auto-summary                    # Désactive résumé automatique
 
# Options avancées
router rip
 passive-interface FastEthernet0/1  # Pas d'updates sur cette interface
 default-information originate      # Propager route par défaut
 maximum-paths 4                    # Load balancing (1-6 chemins)
 
# Authentification RIPv2
interface FastEthernet0/0
 ip rip authentication mode md5
 ip rip authentication key-chain RIP_KEYS
```

### EIGRP (Enhanced Interior Gateway Routing Protocol)

#### Caractéristiques EIGRP
- **Métrique composite** : Bande passante + Délai (par défaut)
- **Convergence rapide** : Algorithme DUAL
- **Load balancing** : Inégal supporté
- **Transport** : Protocole 88, Multicast 224.0.0.10

#### Configuration EIGRP
```cisco
# Configuration basique
router eigrp 100                    # AS (Autonomous System) number
 network 10.0.0.0
 network 192.168.1.0 0.0.0.255     # Avec wildcard mask
 no auto-summary

# Optimisations
router eigrp 100
 passive-interface default          # Par défaut passif
 no passive-interface Serial0/0/0   # Activer seulement où nécessaire
 variance 2                         # Load balancing inégal (ratio 1:2)
 
# Métriques personnalisées
interface Serial0/0/0
 bandwidth 1544                     # Définir bande passante (Kbps)
 delay 20000                        # Définir délai (microseconds)
```

### OSPF (Open Shortest Path First)

#### Caractéristiques OSPF
- **Métrique** : Coût basé sur bande passante
- **Algorithme** : Shortest Path First (Dijkstra)
- **Zones** : Hiérarchie pour scalabilité
- **Transport** : Protocole 89, Multicast 224.0.0.5/224.0.0.6

#### Configuration OSPF
```cisco
# Configuration basique
router ospf 1                       # Process ID (local)
 network 192.168.1.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.3 area 0
 
# Configuration avancée
router ospf 1
 router-id 1.1.1.1                 # Router ID manuel
 passive-interface FastEthernet0/1
 default-information originate      # Route par défaut
 area 1 stub                        # Zone stub
 
# Coût interface
interface Serial0/0/0
 bandwidth 1544                     # Pour calcul coût automatique
 ip ospf cost 64                    # Coût manuel
 ip ospf hello-interval 10          # Intervalle Hello
 ip ospf dead-interval 40           # Intervalle Dead
```

## 🌐 Routage Inter-VLAN

### Router-on-a-Stick (ROAS)
```cisco
# Interface principale
interface GigabitEthernet0/0
 no shutdown
 
# Sous-interfaces par VLAN
interface GigabitEthernet0/0.10
 description "VLAN 10 - Admin"
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 
interface GigabitEthernet0/0.20
 description "VLAN 20 - Users"
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 
interface GigabitEthernet0/0.99
 description "VLAN 99 - Management"
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
```

### Switch L3 (SVI)
```cisco
# Activation routage IP
ip routing

# Création SVIs
interface Vlan10
 description "VLAN 10 - Admin"
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 
interface Vlan20
 description "VLAN 20 - Users"
 ip address 192.168.20.1 255.255.255.0
 no shutdown

# Configuration ports
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
```

### Routeur Multi-Interface
```cisco
# Une interface physique par VLAN
interface FastEthernet0/0
 description "VLAN 10 - Admin"
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 
interface FastEthernet0/1
 description "VLAN 20 - Users"
 ip address 192.168.20.1 255.255.255.0
 no shutdown
```

## 📊 Monitoring & Diagnostics

### Commandes de Vérification
```cisco
# Table de routage
show ip route                       # Table complète
show ip route summary               # Résumé par protocole
show ip route 192.168.1.0          # Route spécifique
show ip route connected             # Routes connectées seulement
show ip route static                # Routes statiques seulement

# Protocoles de routage
show ip protocols                   # Protocoles actifs
show ip rip database               # Base RIP
show ip eigrp neighbors            # Voisins EIGRP
show ip eigrp topology             # Topologie EIGRP
show ip ospf neighbor              # Voisins OSPF
show ip ospf database              # Base OSPF
```

### Diagnostics Connectivité
```cisco
# Tests réseau
ping 192.168.1.1                   # Test connectivité
traceroute 192.168.1.1             # Trace route
show ip arp                        # Table ARP
show cdp neighbors                 # Découverte voisins

# Debug (attention production!)
debug ip routing                   # Changements table routage
debug ip rip                       # Messages RIP
debug ip eigrp                     # Messages EIGRP
debug ip ospf hello                # Messages Hello OSPF
undebug all                        # Arrêter tous debug
```

### Informations Interfaces
```cisco
# État interfaces
show ip interface brief            # Résumé IP
show interfaces                    # Détails complets
show interfaces summary            # Résumé utilisation
show interfaces trunk              # Ports trunk (switch)

# Statistiques
show interfaces FastEthernet0/0 | include packets
show ip traffic                    # Statistiques IP globales
```

## 🔧 Optimisation & Tuning

### Load Balancing
```cisco
# EIGRP - Load balancing inégal
router eigrp 100
 variance 2                        # Ratio max 1:2
 maximum-paths 6                   # Max 6 chemins

# OSPF - Load balancing égal seulement
router ospf 1
 maximum-paths 4                   # Max 4 chemins égaux
```

### Convergence & Timers
```cisco
# RIP - Accélération convergence
router rip
 timers basic 10 30 30 60          # Update/Invalid/Hold/Flush

# EIGRP - Timers Hello/Hold
interface Serial0/0/0
 ip hello-interval eigrp 100 5     # Hello toutes les 5s
 ip hold-time eigrp 100 15         # Hold time 15s

# OSPF - Timers SPF
router ospf 1
 timers throttle spf 1 5 10        # Initial/Min/Max delay (ms)
```

### Authentification
```cisco
# RIP - Authentification MD5
key chain RIP_AUTH
 key 1
  key-string cisco123
  
interface FastEthernet0/0
 ip rip authentication mode md5
 ip rip authentication key-chain RIP_AUTH

# EIGRP - Authentification MD5
key chain EIGRP_AUTH
 key 1
  key-string eigrp123
  
interface Serial0/0/0
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP_AUTH

# OSPF - Authentification MD5
interface Serial0/0/0
 ip ospf message-digest-key 1 md5 ospf123
 
router ospf 1
 area 0 authentication message-digest
```

## 🚨 Troubleshooting Courant

### Checklist Inter-VLAN
1. **VLANs créés** : `show vlan brief`
2. **Trunk configuré** : `show interfaces trunk`
3. **Sous-interfaces/SVI up** : `show ip interface brief`
4. **Adresses IP correctes** : `show running-config`
5. **Routage activé** : `show ip route` (switch L3)
6. **Ping inter-VLAN** : Test connectivité

### Problèmes Fréquents
| Symptôme | Cause Probable | Vérification |
|----------|----------------|--------------|
| **Routes manquantes** | Réseau non annoncé | `show ip protocols` |
| **Convergence lente** | Timers par défaut | `show ip eigrp interfaces detail` |
| **Load balancing absent** | Métriques inégales | `show ip route` |
| **Voisinage instable** | Mismatch paramètres | `show ip ospf neighbor` |

### Commandes de Dépannage
```cisco
# Réinitialisation processus
clear ip route *                   # Vider table routage
clear ip eigrp neighbors           # Reset voisins EIGRP
clear ip ospf process              # Restart processus OSPF

# Vérifications réseau
show arp                          # Table ARP
show mac address-table            # Table MAC (switch)
show spanning-tree                # État STP
```

## 💡 Bonnes Pratiques

### Conception Réseau
- ✅ **Hiérarchie à 3 niveaux** : Core/Distribution/Access
- ✅ **Redondance** : Chemins multiples + protocoles
- ✅ **Segmentation** : VLANs par fonction/sécurité
- ✅ **Adressage logique** : Schéma cohérent et scalable

### Configuration
- ✅ **Router ID explicite** : OSPF/EIGRP avec loopback
- ✅ **Interfaces passives** : Sécurité + performance
- ✅ **Authentification** : Sécuriser protocoles routage
- ✅ **Summarisation** : Réduire taille tables routage

### Monitoring
- ✅ **Documentation** : Diagrammes à jour
- ✅ **Baseline** : Métriques normales connues
- ✅ **Logging** : Activation logs appropriés
- ✅ **Sauvegarde** : Configurations et topologies

---
**💡 Memo** : AD détermine fiabilité, métrique détermine meilleur chemin, longest prefix match détermine la route utilisée !