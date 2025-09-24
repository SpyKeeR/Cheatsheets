# 🌐 IPv6 — Aide-mémoire

## 📋 Structure & Format

### Format Adresse IPv6
```
128 bits = 8 groupes de 16 bits (4 chiffres hexa chacun)
Format : xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx

Exemple complet : 2001:0db8:85a3:0000:0000:8a2e:0370:7334
Exemple compressé : 2001:db8:85a3::8a2e:370:7334
```

### Règles de Compression
1. **Supprimer zéros initiaux** : `0db8` → `db8`
2. **Remplacer séquence zéros par `::`** (une seule fois par adresse)
3. **Préférer compresser la plus longue séquence** de zéros consécutifs

### Notation CIDR
```
2001:db8::/32          # Réseau (32 premiers bits)
2001:db8:85a3::/64     # Sous-réseau typique
::1/128                # Hôte unique (loopback)
```

## 🏷️ Types d'Adresses

### Catégories Principales
| Type | Préfixe | Portée | Usage |
|------|---------|--------|-------|
| **Loopback** | `::1/128` | Local | Test interface locale |
| **Lien-local** | `fe80::/10` | Lien | Communication locale automatique |
| **Unique Local (ULA)** | `fc00::/7` `fd00::/8` | Site | Adressage privé |
| **Global Unicast** | `2000::/3` | Internet | Adressage public routé |
| **Multicast** | `ff00::/8` | Groupe | Communication un-vers-plusieurs |

### Adresses Spéciales
```
::                     # Adresse non spécifiée (0.0.0.0 en IPv4)
::1                    # Loopback (127.0.0.1 en IPv4)
fe80::                 # Lien-local (169.254.x.x en IPv4)
::ffff:192.168.1.1     # IPv4-mapped (transition)
2002::/16              # 6to4 tunneling
```

## 📡 Multicast IPv6

### Adresses Multicast Essentielles
| Adresse | Description | Équivalent IPv4 |
|---------|-------------|-----------------|
| `ff02::1` | Tous nœuds lien-local | 224.0.0.1 |
| `ff02::2` | Tous routeurs lien-local | 224.0.0.2 |
| `ff02::5` | Routeurs OSPF | 224.0.0.5 |
| `ff02::6` | Routeurs OSPF désignés | 224.0.0.6 |
| `ff02::9` | Routeurs RIP | 224.0.0.9 |
| `ff02::1:2` | Agents DHCP | 224.0.0.251 |

### Format Multicast
```
ff02::1:ffxx:xxxx      # Solicited-node multicast
└─┘ └┘    └──────┘
 │  │        └─ 24 derniers bits de l'adresse cible
 │  └─ Scope (02 = lien-local)
 └─ Préfixe multicast
```

### Mapping Ethernet
```
IPv6 Multicast : ff02::1:ff12:3456
MAC Multicast  : 33:33:ff:12:34:56
                 └─────┘ └──────┘
                 Préf.   32 derniers bits IPv6
```

## 🔍 Neighbor Discovery Protocol (NDP)

### Processus NDP vs ARP
| Phase | IPv4 (ARP) | IPv6 (NDP) |
|-------|------------|------------|
| **Requête** | ARP Request (broadcast) | Neighbor Solicitation (multicast) |
| **Réponse** | ARP Reply (unicast) | Neighbor Advertisement (unicast) |
| **Cible** | Broadcast (ff:ff:ff:ff:ff:ff) | Solicited-node multicast |

### Messages NDP
```
1. Router Solicitation (RS)     → Découverte routeurs
2. Router Advertisement (RA)    → Annonce routeur + préfixes  
3. Neighbor Solicitation (NS)   → Résolution adresse (comme ARP)
4. Neighbor Advertisement (NA)  → Réponse résolution
5. Redirect                     → Routage optimal
```

### Workflow NDP Typique
```
1. Host génère adresse lien-local : fe80::mac-derived
2. DAD (Duplicate Address Detection) via NS
3. RS envoyé pour découvrir routeurs  
4. RA reçu avec préfixe global
5. SLAAC : génération adresse globale
6. Communication normale avec NS/NA pour résolution
```

## ⚙️ Auto-Configuration (SLAAC)

### Méthodes d'Attribution
| Méthode | Description | Avantages | Inconvénients |
|---------|-------------|-----------|---------------|
| **SLAAC** | Auto-config via RA | Simple, automatique | Peu de contrôle |
| **DHCPv6** | Serveur centralisé | Contrôle total | Infrastructure requise |
| **Statique** | Configuration manuelle | Prévisible | Gestion lourde |

### Drapeaux Router Advertisement
```
M-Flag (Managed) = 1    → Utiliser DHCPv6 pour adresses
O-Flag (Other) = 1      → Utiliser DHCPv6 pour options (DNS, etc.)

Combinaisons courantes :
M=0, O=0 → SLAAC pur
M=0, O=1 → SLAAC + DHCPv6 pour options  
M=1, O=1 → DHCPv6 complet
```

### Formation Adresse SLAAC
```
Préfixe réseau (64 bits) + Interface ID (64 bits)

Interface ID peut être :
- EUI-64 modifié (MAC + ff:fe insertion)
- Aléatoire (privacy extensions)
- Tokenisé (configuration manuelle)

Exemple EUI-64 :
MAC : 00:1b:63:84:45:e6
EUI-64 : 021b:63ff:fe84:45e6 (bit U/L inversé)
```

## 🌍 Transition IPv4/IPv6

### Mécanismes de Coexistence
| Mécanisme | Description | Usage |
|-----------|-------------|-------|
| **Dual Stack** | IPv4 + IPv6 simultané | Standard moderne |
| **Tunneling** | IPv6 over IPv4 (ou inverse) | Migration progressive |
| **Translation** | NAT64/DNS64 | Interconnexion |

### Types de Tunnel
```
6in4     → IPv6 dans IPv4 (proto 41)
6to4     → Auto-tunnel (2002::/16)  
6rd      → IPv6 Rapid Deployment
Teredo   → Traversal NAT IPv4
ISATAP   → Intra-Site Automatic Tunnel
```

## 🔧 Commandes Pratiques

### Windows
```cmd
# Configuration interfaces
netsh interface ipv6 show config
netsh interface ipv6 set address "Interface" 2001:db8::1

# Table de routage
netsh interface ipv6 show route

# Cache NDP  
netsh interface ipv6 show neighbors

# Test connectivité
ping -6 2001:db8::1
ping -6 ::1
```

### Linux
```bash
# Configuration interfaces
ip -6 addr show
ip -6 addr add 2001:db8::1/64 dev eth0

# Routage IPv6
ip -6 route show
ip -6 route add default via fe80::1 dev eth0

# Table NDP
ip -6 neigh show

# Test connectivité  
ping6 2001:db8::1
ping6 ::1
```

### Cisco IOS
```cisco
# Activation IPv6
ipv6 unicast-routing

# Configuration interface
interface fastethernet0/0
 ipv6 enable
 ipv6 address 2001:db8:1::1/64
 ipv6 address fe80::1 link-local

# Routage statique
ipv6 route ::/0 2001:db8:1::1

# Vérifications
show ipv6 interface brief
show ipv6 neighbors
show ipv6 route
```

## 📊 Comparaison IPv4 vs IPv6

### Caractéristiques Techniques
| Aspect | IPv4 | IPv6 |
|--------|------|------|
| **Taille adresse** | 32 bits | 128 bits |
| **Notation** | Décimale pointée | Hexadécimale |
| **En-tête** | Variable (20-60 bytes) | Fixe (40 bytes) |
| **Fragmentation** | Routeurs + hôtes | Hôtes seulement |
| **Checksum** | En-tête | Aucun (délégué couches sup.) |
| **Broadcast** | Oui | Non (multicast seulement) |
| **Configuration** | Manuelle/DHCP | Auto/DHCPv6/manuelle |

### Résolution d'Adresses
| Protocole | IPv4 | IPv6 |
|-----------|------|------|
| **Résolution L2** | ARP | NDP |
| **Requête** | Broadcast | Multicast solicited-node |
| **Table** | ARP table | Neighbor cache |
| **Détection doublons** | Gratuitous ARP | DAD (Duplicate Address Detection) |

## 🛠️ Troubleshooting IPv6

### Tests de Connectivité
```bash
# Tests basiques
ping6 ::1                    # Loopback local
ping6 fe80::1%eth0          # Lien-local (% = interface)
ping6 2001:db8::1           # Global unicast

# Trace route IPv6
traceroute6 2001:db8::1
tracert -6 2001:db8::1      # Windows

# Tests DNS IPv6
nslookup -type=AAAA google.com
dig AAAA google.com
```

### Vérifications Configuration
```bash
# Vérifier adresses configurées
ip -6 addr show | grep -E "(inet6|scope)"

# Vérifier routage par défaut
ip -6 route show | grep default

# Vérifier NDP/cache voisins
ip -6 neigh show | grep -v FAILED

# Vérifier Router Advertisement
rdisc6 eth0                 # Linux (rdisc6 package)
```

### Problèmes Courants
| Symptôme | Cause Probable | Solution |
|----------|----------------|----------|
| Pas d'adresse globale | Pas de RA reçu | Vérifier routeur/SLAAC |
| Échec ping lien-local | Interface non activée | `ipv6 enable` |
| Résolution DNS lente | Préférence IPv4 | Configurer préférences |
| Tunnel ne fonctionne pas | MTU/fragmentation | Ajuster MTU (1280 min) |

## 💡 Bonnes Pratiques

### Planification Adressage
- ✅ **Utiliser /64 pour sous-réseaux** (standard SLAAC)
- ✅ **Hiérarchie logique** : /48 site, /56 département, /64 VLAN
- ✅ **Documenter plan d'adressage** avant déploiement
- ✅ **Prévoir expansion** (IPv6 = espace quasi-illimité)

### Sécurité IPv6
- ✅ **Filtrer trafic ICMPv6** nécessaire uniquement
- ✅ **Surveillance NDP** (détection attaques)
- ✅ **Extension headers** → attention fragmentation
- ✅ **RA Guard** sur ports access switch

### Migration
- ✅ **Dual-stack** comme étape intermédiaire
- ✅ **Tester applications** avec IPv6
- ✅ **Former équipes** avant mise en production
- ✅ **Monitoring** spécifique IPv6