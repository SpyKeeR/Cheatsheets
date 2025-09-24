# 🌐 Infrastructures Réseaux — Aide-mémoire

## 📚 Modèles de Référence

### Modèle OSI (7 Couches)
| Couche | Nom | Fonction | PDU | Équipements | Protocoles |
|--------|-----|----------|-----|-------------|------------|
| **7** | Application | Interface utilisateur | Données | - | HTTP, FTP, SMTP |
| **6** | Présentation | Traduction, chiffrement | Données | - | SSL/TLS, JPEG |
| **5** | Session | Gestion sessions | Données | - | NetBIOS, RPC |
| **4** | Transport | Fiabilité bout-en-bout | Segment | Pare-feu L4 | TCP, UDP |
| **3** | Réseau | Routage inter-réseaux | Paquet | Routeur, L3 Switch | IP, ICMP, OSPF |
| **2** | Liaison | Communication directe | Trame | Switch, Bridge | Ethernet, PPP |
| **1** | Physique | Transmission bits | Bit | Hub, Répéteur | Câbles, Wi-Fi |

### Modèle TCP/IP (4 Couches)
```
Application     ← OSI 5,6,7
Transport       ← OSI 4
Internet        ← OSI 3
Accès Réseau    ← OSI 1,2
```

### Encapsulation & PDU
```
Application    → Données
Transport      → Segment (TCP) / Datagramme (UDP)
Réseau         → Paquet (IP)
Liaison        → Trame (Ethernet)
Physique       → Bits (Signal électrique/optique/radio)
```

**Terminologie** :
- **PDU** : Protocol Data Unit (unité complète de chaque couche)
- **SDU** : Service Data Unit (données utiles de la couche supérieure)
- **PCI** : Protocol Control Information (en-têtes/métadonnées ajoutées)

## 🔗 Couche Liaison de Données

### Trame Ethernet (IEEE 802.3)
```
┌─────────────┬─────┬──────────┬──────────┬───────────┬─────────┬─────┐
│ Préambule   │ SFD │ MAC Dest │ MAC Src  │ EtherType │ Payload │ FCS │
│ 7 octets    │ 1   │ 6        │ 6        │ 2         │ 46-1500 │ 4   │
└─────────────┴─────┴──────────┴──────────┴───────────┴─────────┴─────┘
```

#### Champs Principaux
- **Préambule** : `10101010` × 7 (synchronisation)
- **SFD** : `10101011` (Start Frame Delimiter)
- **MAC Dest/Src** : Adresses physiques (6 octets chacune)
- **EtherType** : Type protocole encapsulé
  - `0x0800` : IPv4
  - `0x86DD` : IPv6
  - `0x8100` : VLAN (802.1Q)
  - `0x0806` : ARP
- **FCS** : Frame Check Sequence (CRC-32)

### VLAN Tagging (802.1Q)
```
┌──────────┬──────────┬──────┬──────────┬───────────┬─────────┬─────┐
│ MAC Dest │ MAC Src  │ TPID │ TCI      │ EtherType │ Payload │ FCS │
│ 6        │ 6        │ 2    │ 2        │ 2         │ 46-1500 │ 4   │
└──────────┴──────────┴──────┴──────────┴───────────┴─────────┴─────┘
```

#### Tag Control Information (TCI)
```
┌─────────┬─────┬──────────────┐
│ PCP(3)  │ DEI │ VLAN ID(12)  │
└─────────┴─────┴──────────────┘
```
- **PCP** : Priority Code Point (CoS 0-7)
- **DEI** : Drop Eligible Indicator
- **VLAN ID** : 1-4094 (0 et 4095 réservés)

## 🌍 Couche Réseau (IPv4)

### En-tête IPv4
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌───┬─────┬───────────┬─────────────────────────────────────────────┐
│Ver│ IHL │    ToS    │              Longueur Totale               │
├───┴─────┼───────────┼─────────────────────────────────────────────┤
│         Identification        │Flags│      Fragment Offset       │
├───────────────────────────────┼─────┼─────────────────────────────┤
│      TTL      │   Protocole   │         Checksum En-tête        │
├───────────────┼───────────────┼─────────────────────────────────┤
│                        Adresse Source                            │
├───────────────────────────────────────────────────────────────────┤
│                     Adresse Destination                          │
├───────────────────────────────────────────────────────────────────┤
│                    Options (facultatif)                          │
└───────────────────────────────────────────────────────────────────┘
```

#### Champs Critiques
| Champ | Taille | Description | Valeurs Courantes |
|-------|--------|-------------|-------------------|
| **Version** | 4 bits | Version IP | 4 (IPv4) |
| **IHL** | 4 bits | Longueur en-tête ÷ 4 | 5 (20 octets min) |
| **ToS/DSCP** | 8 bits | Qualité de service | 0 (best effort) |
| **Longueur** | 16 bits | Taille totale paquet | 20-65535 octets |
| **TTL** | 8 bits | Durée de vie | 64, 128, 255 |
| **Protocole** | 8 bits | Protocole encapsulé | 1=ICMP, 6=TCP, 17=UDP |

### Fragmentation IPv4
- **DF** (Don't Fragment) : Interdit fragmentation
- **MF** (More Fragments) : D'autres fragments suivent
- **Fragment Offset** : Position fragment (× 8 octets)

## 🚦 Architectures Réseau

### Architecture 3-Tiers
```
┌────────────────────────────────────────────────────────────┐
│                        CORE                                │
│  • Routage haute vitesse                                   │
│  • Redondance maximale                                     │
│  • Pas de manipulation trafic                              │
└─────────────────┬───────────────────┬──────────────────────┘
                  │                   │
┌─────────────────▼───────────────────▼──────────────────────┐
│                    DISTRIBUTION                            │
│  • Agrégation liens access                                 │
│  • Routage inter-VLAN                                      │
│  • Politiques sécurité (ACL)                               │
│  • QoS                                                     │
└─────┬─────────┬─────────┬─────────┬─────────┬──────────────┘
      │         │         │         │         │
┌─────▼─────────▼─────────▼─────────▼─────────▼───────────┐
│                       ACCESS                            │
│  • Connexion utilisateurs finaux                        │
│  • PoE (téléphones, APs)                                │
│  • Port security                                        │
│  • VLAN assignment                                      │
└─────────────────────────────────────────────────────────┘
```

### Spine-Leaf (Datacenter)
```
┌─────────┬─────────┬─────────┬─────────┐
│ Spine 1 │ Spine 2 │ Spine 3 │ Spine 4 │  ← Couche Spine
└────┬────┴────┬────┴────┬────┴────┬────┘
     │         │         │         │
   ╔═╧═════════════════════════════╧═╗     ← Full Mesh
   ║                                 ║
┌──▼───┬──▼───┬──▼──┬───▼───┬──▼──┬──▼──┐   ← Couche Leaf
│Leaf1 │Leaf2 │Leaf3│ Leaf4 │Leaf5│Leaf6│
└──────┴──────┴─────┴───────┴─────┴─────┘
```

**Avantages** :
- Latence prévisible
- Scalabilité horizontale
- Bande passante élevée
- Pas de boucles STP

## 🗺️ Protocoles de Routage

### Classification
```
Protocoles de Routage
├── Static Routing (Configuration manuelle)
└── Dynamic Routing
    ├── IGP (Interior Gateway Protocol)
    │   ├── Distance Vector
    │   │   ├── RIP (Routing Information Protocol)
    │   │   └── EIGRP (Enhanced IGRP - Cisco)
    │   └── Link State
    │       ├── OSPF (Open Shortest Path First)
    │       └── IS-IS (Intermediate System)
    └── EGP (Exterior Gateway Protocol)
        └── BGP (Border Gateway Protocol)
```

### Comparaison Protocoles IGP
| Protocole | Type | Métrique | Convergence | Scalabilité | Standard |
|-----------|------|----------|-------------|-------------|----------|
| **RIP** | Distance Vector | Hop count (max 15) | Lente | Faible | Ouvert |
| **EIGRP** | Hybride | Composite | Rapide | Bonne | Cisco |
| **OSPF** | Link State | Coût/Bande passante | Rapide | Excellente | Ouvert |
| **IS-IS** | Link State | Métrique | Rapide | Excellente | Ouvert |

### Administrative Distance (AD)
| Route Source | AD | Fiabilité |
|--------------|-----|-----------|
| Interface connectée | 0 | Maximale |
| Route statique | 1 | Très élevée |
| EIGRP (interne) | 90 | Élevée |
| OSPF | 110 | Bonne |
| RIP | 120 | Faible |
| EIGRP (externe) | 170 | Très faible |

## 📶 Technologies Sans Fil

### Wi-Fi Standards (IEEE 802.11)
| Standard | Nom | Fréquence | Débit Max | Portée | Année |
|----------|-----|-----------|-----------|--------|-------|
| **802.11b** | - | 2.4 GHz | 11 Mbps | 35m | 1999 |
| **802.11g** | - | 2.4 GHz | 54 Mbps | 35m | 2003 |
| **802.11n** | Wi-Fi 4 | 2.4/5 GHz | 600 Mbps | 50m | 2009 |
| **802.11ac** | Wi-Fi 5 | 5 GHz | 6.9 Gbps | 35m | 2013 |
| **802.11ax** | Wi-Fi 6 | 2.4/5 GHz | 9.6 Gbps | 30m | 2019 |

### Architecture Wi-Fi
```
Internet ←→ Router ←→ WLC ←→ AP ←→ Client
                      │
                      ├─→ AP ←→ Client  
                      └─→ AP ←→ Client
```

**Composants** :
- **STA** : Station (client Wi-Fi)
- **AP** : Access Point
- **WLC** : Wireless LAN Controller
- **BSS** : Basic Service Set (1 AP + clients)
- **ESS** : Extended Service Set (plusieurs AP, même SSID)

### Sécurité Wi-Fi
| Standard | Chiffrement | Clé | Sécurité | Usage |
|----------|-------------|-----|----------|--------|
| **WEP** | RC4 | Statique | ❌ Très faible | Obsolète |
| **WPA** | TKIP | Dynamique | ⚠️ Faible | Legacy |
| **WPA2** | AES-CCMP | Dynamique | ✅ Bonne | Standard |
| **WPA3** | AES-GCMP | Dynamique | ✅ Excellente | Moderne |

### Canaux Wi-Fi
#### 2.4 GHz (Non-overlapping)
```
Canal 1  : 2412 MHz
Canal 6  : 2437 MHz  
Canal 11 : 2462 MHz
```

#### 5 GHz (Exemples)
```
Canal 36  : 5180 MHz
Canal 40  : 5200 MHz
Canal 44  : 5220 MHz
Canal 149 : 5745 MHz
```

## 🎯 Qualité de Service (QoS)

### Paramètres QoS
| Paramètre | Description | Applications Sensibles |
|-----------|-------------|----------------------|
| **Bande passante** | Débit disponible | Streaming, backup |
| **Latence** | Délai transmission | VoIP, gaming |
| **Gigue** | Variation délais | VoIP, vidéo |
| **Perte paquets** | Taux perte | Toutes applications |

### Classification Trafic
```
┌─────────────────────────────────────────────────────────┐
│                    Voice (Voix)                         │
│  • Latence < 150ms, Gigue < 30ms, Perte < 1%            │
│  • DSCP: EF (46), CoS: 5                                │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│               Video (Vidéo Interactive)                 │
│  • Latence < 400ms, Perte < 1%                          │
│  • DSCP: AF41 (34), CoS: 4                              │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│                 Data (Données Critiques)                │
│  • Applications métier                                  │
│  • DSCP: AF31 (26), CoS: 3                              │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│               Best Effort (Par défaut)                  │
│  • Email, web, FTP                                      │
│  • DSCP: 0, CoS: 0                                      │
└─────────────────────────────────────────────────────────┘
```

### Mécanismes QoS
- **Classification** : Identifier le trafic
- **Marking** : Marquer paquets (DSCP, CoS)
- **Queuing** : Files d'attente prioritaires
- **Shaping** : Limiter débit
- **Policing** : Contrôler conformité

## 🔐 Sécurité Réseau

### Zones de Sécurité
```
Internet (Untrusted)
    │
┌───▼────┐
│  DMZ   │ ← Serveurs exposés (Web, Mail, DNS)
└───┬────┘
    │
┌───▼────┐
│  LAN   │ ← Réseau interne sécurisé
└────────┘
```

### Types de Pare-feu
| Type | Couche OSI | Inspection | Performance |
|------|------------|------------|-------------|
| **Paquet** | 3-4 | Headers IP/TCP | Très élevée |
| **Circuit** | 4 | État connexions | Élevée |
| **Application** | 7 | Contenu applicatif | Moyenne |
| **NGFW** | 3-7 | Deep packet inspection | Variable |

### Contrôle d'Accès
- **802.1X** : Authentification réseau (RADIUS)
- **NAC** : Network Access Control
- **VPN** : Tunnel chiffré (IPsec, SSL)
- **RBAC** : Role-Based Access Control

## 📡 Protocoles de Communication

### Couche Transport
| Protocole | Type | Fiabilité | Usage | Port Range |
|-----------|------|-----------|--------|------------|
| **TCP** | Connection-oriented | Garantie | HTTP, SMTP, SSH | 1-65535 |
| **UDP** | Connectionless | Best effort | DNS, DHCP, Streaming | 1-65535 |

### Ports Bien Connus
| Service | TCP | UDP | Description |
|---------|-----|-----|-------------|
| **HTTP** | 80 | - | Web non-sécurisé |
| **HTTPS** | 443 | - | Web sécurisé |
| **SSH** | 22 | - | Shell sécurisé |
| **DNS** | 53 | 53 | Résolution noms |
| **DHCP** | - | 67/68 | Attribution IP |
| **SNMP** | - | 161 | Monitoring |

## 🔧 Outils & Commandes

### Diagnostic Réseau
```bash
# Connectivité
ping 8.8.8.8                  # Test connectivité
traceroute google.com         # Route paquets
nslookup google.com           # Résolution DNS

# Interface & routage  
ip addr show                  # Adresses IP (Linux)
ip route show                 # Table routage (Linux)
ipconfig /all                 # Configuration IP (Windows)
route print                   # Table routage (Windows)

# Connexions
netstat -tuln                 # Ports ouverts
ss -tuln                      # Alternative moderne (Linux)
```

### Analyse Trafic
```bash
# Capture paquets
tcpdump -i eth0               # Capture interface
wireshark                     # Analyseur graphique
tshark -i eth0 -f "port 80"   # Capture CLI avec filtre

# Monitoring
iftop                         # Trafic temps réel
nload                         # Monitoring simple
vnstat -i eth0                # Statistiques interface
```

## 💡 Bonnes Pratiques

### Conception Réseau
- ✅ **Redondance** : Pas de SPOF (Single Point of Failure)
- ✅ **Segmentation** : VLANs par fonction/sécurité
- ✅ **Scalabilité** : Prévoir croissance
- ✅ **Standardisation** : Conventions nommage cohérentes

### Sécurité
- ✅ **Defense in Depth** : Sécurité multi-couches
- ✅ **Principe moindre privilège** : Accès minimal nécessaire
- ✅ **Monitoring** : Supervision continue
- ✅ **Documentation** : Topologies et configurations à jour

### Performance
- ✅ **QoS** : Prioriser trafic critique
- ✅ **Load Balancing** : Répartir charge
- ✅ **Caching** : Réduire latence
- ✅ **Optimisation** : Tuning selon usage

---
**💡 Memo** : OSI = Please Do Not Throw Sausage Pizza Away • TCP fiable, UDP rapide • VLAN = segmentation L2 • QoS essentiel pour temps réel !