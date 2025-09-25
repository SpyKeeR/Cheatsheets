# 🌐 Virtualisation — Réseau & Stockage — Aide-mémoire

## 🔌 Modes Réseau Virtualisation

### Architectures Réseau VM
```
Bridge (Bridged) ←→ Switch Physique ←→ LAN
NAT ←→ Hypervisor NAT ←→ WAN
Host-Only ←→ Virtual Switch ←→ Host seulement
Private/Isolated ←→ VM ←→ VM (aucun accès externe)
```

### Comparatif Modes Réseau
| Mode | Accès Internet | Accès LAN | Visible depuis LAN | Usage Type |
|------|---------------|-----------|-------------------|------------|
| **Bridge** | ✅ | ✅ | ✅ | Production, services |
| **NAT** | ✅ | ❌ | ❌ | Client, développement |
| **Host-Only** | ❌ | ❌ | ❌ | Tests isolés |
| **Internal/Private** | ❌ | ❌ | ❌ | Lab inter-VM |

### Implémentations par Hyperviseur
#### VMware Workstation/Player
- **VMnet0** : Bridge (réseau physique)
- **VMnet1** : Host-Only (192.168.137.x par défaut)
- **VMnet8** : NAT (192.168.6.x par défaut)
- **VMnet2-VMnet19** : LAN Segments personnalisés

#### Hyper-V
- **External** : Bridge vers adaptateur physique
- **Internal** : VM ↔ Host (avec IP virtuelle host)
- **Private** : VM ↔ VM uniquement (isolation complète)

#### VirtualBox
- **NAT** : Translation adresses (10.0.2.x/24)
- **NAT Network** : NAT partagé entre VMs
- **Bridged** : Accès direct réseau physique
- **Host-only** : Réseau privé host-VM

## 🏗️ Architecture Réseau vSphere

### Composants vSphere Networking
```
Physical Network
    ↓
Physical NICs (vmnic0, vmnic1...)
    ↓
Virtual Switch (vSS/vDS)
    ↓
Port Groups
    ↓
Virtual NICs (VM) + VMkernel Ports (ESXi)
```

### Types d'Interfaces
| Interface | Description | Usage | Configuration |
|-----------|-------------|-------|---------------|
| **pNIC/vmnic** | Interface physique ESXi | Uplink vers réseau | Automatique |
| **vNIC** | Interface virtuelle VM | Connectivité VM | Par VM |
| **VMkernel** | Interface service ESXi | Management, vMotion, stockage | Par service |

### Standard Switch (vSS) vs Distributed Switch (vDS)
| Aspect | vSS (Standard) | vDS (Distributed) |
|--------|---------------|-------------------|
| **Gestion** | Par hôte ESXi | Centralisée vCenter |
| **Configuration** | Manuelle chaque hôte | Cohérente sur cluster |
| **Mobilité VM** | Configuration manuelle | Automatique |
| **Fonctions avancées** | Limitées | QoS, monitoring, LACP |
| **Licence** | Incluse | Enterprise Plus |

## 🌐 VLAN & Segmentation

### Configuration VLAN
```
Port Group Configuration:
├── VLAN ID: 0 (No tagging)
├── VLAN ID: 1-4094 (Tagged)
├── VLAN Trunking: 4095 (All VLANs)
└── Private VLAN: Isolation secondaire
```

### Tagging 802.1Q
- **Access Port** : Un VLAN, pas de tag (VLAN natif)
- **Trunk Port** : Multiples VLANs avec tags 802.1Q
- **Native VLAN** : VLAN non-taggé sur trunk (défaut VLAN 1)

### Bonnes Pratiques VLAN
- ✅ **VLAN Management** : Isoler trafic administration
- ✅ **VLAN vMotion** : Réseau dédié migration live
- ✅ **VLAN Storage** : Trafic iSCSI/NFS séparé
- ✅ **VLAN Production** : VMs par fonction métier
- ✅ **Native VLAN** : Changer VLAN 1 par défaut (sécurité)

## 🔄 Load Balancing & Teaming

### Modes Load Balancing (vSphere)
| Mode | Description | Distribution | Failover |
|------|-------------|--------------|----------|
| **Route based on port ID** | Par port virtuel | Équitable | ✅ |
| **Route based on MAC hash** | Par adresse MAC | Par VM | ✅ |
| **Route based on IP hash** | Par IP source/dest | Par flux | ✅ |
| **Use explicit failover order** | Actif/Passif | Pas équilibrage | ✅ |

### Teaming NIC Physiques
```bash
# Configuration team exemple
Primary: vmnic0 (Active)
Secondary: vmnic1 (Active/Standby)
Policy: Load balancing
Failback: Yes/No
Detection: Link status + Beacon probing
```

### Algorithmes Teaming Windows/Linux
- **Round Robin** : Distribution séquentielle
- **Active Backup** : Un actif, autres en standby
- **XOR/Hash** : Basé sur hash MAC/IP
- **Broadcast** : Toutes interfaces (redondance)
- **802.3ad (LACP)** : Agrégation dynamique

## 💾 Technologies Stockage

### Modes d'Accès Stockage
```
Block Storage (SAN):
├── Fibre Channel (FC)
├── FCoE (FC over Ethernet)
├── iSCSI (SCSI over IP)
└── NVMe over Fabrics

File Storage (NAS):
├── NFS (Network File System)
├── SMB/CIFS (Windows shares)
└── Object Storage (S3, Azure Blob)
```

### Comparaison SAN vs NAS
| Aspect | SAN (Block) | NAS (File) |
|--------|-------------|------------|
| **Protocole** | FC, iSCSI, FCoE | NFS, SMB/CIFS |
| **Performance** | Très élevée | Bonne |
| **Latence** | Très faible | Moyenne |
| **Partage** | Raw device | Système fichiers |
| **Coût** | Élevé | Modéré |
| **Complexité** | Élevée | Faible |

### iSCSI Configuration
```bash
# Composants iSCSI
Initiator (Client) ←→ Target (Storage)
    ↓                    ↓
IQN Client          IQN Storage
    ↓                    ↓
Discovery           Portal/LUN
```

#### iSCSI Naming (IQN)
```
Format: iqn.yyyy-mm.reverse.domain:identifier
Exemple: iqn.2024-01.com.company:storage.server01
```

#### Bonnes Pratiques iSCSI
- ✅ **Réseau dédié** : VLAN séparé pour trafic iSCSI
- ✅ **Jumbo Frames** : MTU 9000 (si supporté bout-en-bout)
- ✅ **Multipath** : Chemins redondants vers stockage
- ✅ **CHAP Authentication** : Sécuriser connexions
- ✅ **QoS** : Prioriser trafic stockage

## 🗄️ Datastores & Provisioning

### Types Datastores vSphere
| Type | Protocole | Format | Partage | Performance |
|------|-----------|--------|---------|-------------|
| **VMFS** | Block (FC/iSCSI) | VMFS6 | Multi-host | Excellente |
| **NFS** | File (NFS) | NFS | Native | Bonne |
| **vSAN** | Object | vSAN | Cluster | Très bonne |
| **Raw Device** | Block | RDM | Limited | Excellente |

### Provisioning Types
```
Thick Provision Eager Zeroed:
└── Espace alloué + écrit immédiatement (sécurité max)

Thick Provision Lazy Zeroed:
└── Espace alloué, écriture à la demande (performance)

Thin Provision:
└── Allocation dynamique (économie espace, risque overcommit)
```

### VMDK vs RDM
| Aspect | VMDK | RDM |
|--------|------|-----|
| **Encapsulation** | Fichier sur datastore | Mapping direct LUN |
| **Snapshots** | ✅ | ❌ (RDM Physical) / ✅ (RDM Virtual) |
| **Clonage** | ✅ | Limité |
| **Performance** | Très bonne | Native |
| **Gestion** | Simple | Complexe |

## 📦 Templates & Formats

### OVF vs OVA vs VMTX
| Format | Description | Portabilité | Modification |
|--------|-------------|-------------|--------------|
| **OVF** | Open Virtualization Format | ✅ Inter-plateformes | ❌ |
| **OVA** | OVF Archive (tar) | ✅ Inter-plateformes | ❌ |
| **VMTX** | VMware Template | ❌ vSphere uniquement | ✅ |

### Déploiement VM depuis Template
```
Template → Deploy VM:
├── Linked Clone (delta disk)
├── Full Clone (copie complète)
└── Instant Clone (mémoire partagée)
```

## 🔄 vMotion & Migration Live

### Prérequis vMotion
```
Réseau:
├── VMkernel port dédié vMotion
├── Même subnet ou routage L3
├── Bande passante ≥ 1Gbps (recommandé 10Gbps)
└── Latence < 5ms

Stockage:
├── Accès partagé datastores (SAN/NAS)
├── Même version VMFS
└── Permissions cohérentes

Compute:
├── CPU compatible (ou EVC)
├── Même version ESXi (ou compatible)
└── Ressources disponibles cible
```

### Enhanced vMotion Compatibility (EVC)
```
EVC Modes (masquage fonctionnalités CPU):
├── Intel: Merom, Penryn, Nehalem, Westmere...
├── AMD: Rev E, Rev F, Greyhound, Barcelona...
└── Activation: Mode cluster (pas VM individuelle)
```

### Types Migration
| Type | Downtime | Pré-requis | Usage |
|------|----------|------------|-------|
| **vMotion** | Aucun | Stockage partagé | Migration compute |
| **Storage vMotion** | Aucun | Aucun | Migration stockage |
| **Cross vCenter** | Aucun | vCenter ≥ 6.0 | Migration datacenter |
| **Cold Migration** | Complet | Basique | Maintenance planifiée |

## 🛡️ Sécurité Réseau Virtuel

### Micro-segmentation
```
Traditional Network:
VLAN 10 (DMZ) ←→ VLAN 20 (APP) ←→ VLAN 30 (DB)

Micro-segmentation:
VM-to-VM policies (NSX/vSphere Network I/O Control)
└── Contrôle granulaire par VM/application
```

### vSphere Network I/O Control
```bash
# QoS par type trafic
Management Traffic: High priority
vMotion: High priority  
Virtual Machine: Normal priority
iSCSI/NFS: High priority
vSphere Replication: Low priority 
```

### Distributed Firewall (NSX)
- **Stateful** : Suivi connexions établies
- **Application-aware** : Contrôle par application
- **VM-centric** : Politiques suivent la VM
- **Zero-trust** : Contrôle East-West traffic

## 📡 VMkernel Services

### Services VMkernel Essentiels
| Service | Port TCP | Description | Réseau Dédié |
|---------|----------|-------------|--------------|
| **Management** | 443, 902 | vCenter communication | Recommandé |
| **vMotion** | 8000 | Migration live VMs | ✅ Obligatoire |
| **Storage (NFS)** | 2049 | Trafic NFS | Recommandé |
| **Storage (iSCSI)** | 3260 | Trafic iSCSI | ✅ Obligatoire |
| **Fault Tolerance** | 8100-8200 | FT logging traffic | ✅ Obligatoire |
| **vSphere Replication** | 31031 | Réplication VM | Optionnel |

### Configuration VMkernel
```bash
# Exemple configuration VMkernel
VMkernel Port: vmk1
IP: 192.168.100.10/24
Service: vMotion
MTU: 9000 (Jumbo Frames)
VLAN: 100
```

## 🚦 Troubleshooting Réseau/Stockage

### Commandes Diagnostic ESXi
```bash
# Réseau
esxcli network ip interface list          # Interfaces VMkernel
esxcli network vswitch standard list      # vSwitches standard
esxcli network ip route ipv4 list         # Table routage
vmkping -I vmk1 192.168.100.20           # Test connectivité VMkernel

# Stockage
esxcli storage core device list           # Périphériques stockage
esxcli storage vmfs extent list           # Extents VMFS
esxcli storage nmp path list              # Chemins multipath
esxcli iscsi session list                 # Sessions iSCSI actives
```

### Tests Performance
```bash
# Test bande passante réseau
iperf3 -c target_ip -t 60 -P 4           # Test multi-stream

# Test latence stockage
esxtop → d (disk view) → latence par device

# Test MTU (Jumbo Frames)
vmkping -I vmk1 -s 8972 -d target_ip     # Test MTU 9000
```

## 💡 Bonnes Pratiques

### Architecture Réseau
- ✅ **Séparation des flux** : Management, vMotion, Storage, Production
- ✅ **Redondance** : Minimum 2 pNIC par vSwitch critique
- ✅ **VLAN dédiés** : Un VLAN par type de trafic
- ✅ **QoS** : Priorisation trafic critique (Storage, vMotion)
- ✅ **Monitoring** : Supervision utilisation et performance

### Stockage
- ✅ **Multipathing** : Redondance chemins vers stockage
- ✅ **Provisioning** : Thick pour performance, Thin pour économie
- ✅ **Snapshots** : Politique rétention et nettoyage
- ✅ **Backup datastores** : Surveillance espace libre
- ✅ **Performance** : Monitoring IOPS et latence

### Migration & Mobilité
- ✅ **EVC** : Activation dès création cluster
- ✅ **Tests migration** : Validation avant production
- ✅ **Bande passante** : Réseau dédié haute performance
- ✅ **Planification** : Migration hors heures pointe
- ✅ **Rollback** : Plan retour en cas de problème

---
**💡 Memo** : Séparer les flux (Mgmt/vMotion/Storage), VMkernel dédié par service, EVC dès le cluster, thin = économie/thick = performance !