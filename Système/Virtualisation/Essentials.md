# 🔧 Virtualisation — Aide-mémoire

## 💻 Types d'Hyperviseurs

### Classification Fondamentale
```
Type 1 (Bare Metal)          Type 2 (Hosted)
    ┌─────────────┐             ┌─────────────┐
    │     VMs     │             │     VMs     │
    ├─────────────┤             ├─────────────┤
    │ Hypervisor  │             │ Hypervisor  │
    ├─────────────┤             ├─────────────┤
    │  Hardware   │             │   Host OS   │
    └─────────────┘             ├─────────────┤
                                │  Hardware   │
                                └─────────────┘
```

### Comparaison Solutions
| Solution | Type | Entreprise | Personnel | Coût | OS Support |
|----------|------|------------|-----------|------|------------|
| **VMware ESXi** | 1 | ✅ Excellent | ❌ | Élevé | Complet |
| **Hyper-V** | 1 | ✅ Très bon | ⚠️ Limité | Moyen | Windows focus |
| **Proxmox** | 1 | ✅ Bon | ⚠️ | Gratuit | Linux/Windows |
| **VMware Workstation** | 2 | ⚠️ Dev/Test | ✅ | Moyen | Complet |
| **VirtualBox** | 2 | ⚠️ Basique | ✅ | Gratuit | Complet |
| **Parallels** | 2 | ❌ | ✅ macOS | Élevé | macOS focus |

## 🏗️ Technologies de Base

### Support Matériel
| Technologie | Intel | AMD | Function |
|-------------|-------|-----|----------|
| **Virtualisation CPU** | VT-x | AMD-V | Instructions privilégiées |
| **SLAT** | EPT | NPT/RVI | Translation adresses mémoire |
| **I/O** | VT-d | AMD-Vi | Passthrough périphériques |

### Modes de Virtualisation
- **Full Virtualization** : VM non modifiée, émulation complète
- **Para-virtualization** : OS invité modifié (Xen legacy)
- **Hardware-assisted** : VT-x/AMD-V + SLAT (standard moderne)
- **Container** : Partage kernel OS (Docker, LXC)

## 📁 Formats Fichiers & VM

### Formats Disques Virtuels
| Format | Hypervisor | Taille | Snapshots | Performance |
|--------|------------|--------|-----------|-------------|
| **VMDK** | VMware | Dynamique/Fixe | ✅ | Excellente |
| **VHD/VHDX** | Hyper-V | Dynamique/Fixe | ✅ | Très bonne |
| **QCOW2** | QEMU/KVM | Dynamique | ✅ | Bonne |
| **VDI** | VirtualBox | Dynamique/Fixe | ✅ | Moyenne |

### Structure VM VMware
```bash
# Fichiers VM essentiels
myvm.vmx                    # Configuration VM (texte)
myvm.vmdk                   # Descripteur disque
myvm-flat.vmdk              # Données disque (binaire)
myvm.nvram                  # BIOS/UEFI settings
myvm.vmss                   # Suspended state
myvm.vmsn                   # Snapshot metadata
vmware.log                  # Logs exécution

# Snapshots
myvm-Snapshot1.vmsn         # Metadata snapshot
myvm-Snapshot1.vmem         # État mémoire
myvm-000001.vmdk            # Delta disk snapshot
```

### Import/Export Standards
```bash
# OVF (Open Virtualization Format)
.ovf                        # Descripteur XML
.mf                         # Manifest (checksums)
.cert                       # Certificat signature
.vmdk/.vhd                  # Disques virtuels

# OVA (Archive unique)
.ova                        # Tar archive contenant OVF + disques
```

## 🔧 VMware vSphere

### Architecture vSphere
```
vCenter Server
    ├── Datacenter
    │   ├── Cluster
    │   │   ├── ESXi Host 1
    │   │   ├── ESXi Host 2
    │   │   └── ESXi Host 3
    │   └── Datastore Cluster
    └── Network (vSwitches, dvSwitches)
```

### Fonctionnalités Cluster
| Feature | Description | Pré-requis |
|---------|-------------|------------|
| **vMotion** | Migration live VM | Stockage partagé, réseau compatible |
| **Storage vMotion** | Migration stockage live | Aucun downtime |
| **DRS** | Distribution ressources automatique | Cluster, vMotion |
| **HA** | Haute disponibilité | Stockage partagé, agent réseau |
| **FT** | Tolérance pannes (zero downtime) | CPU compatible, réseau 10Gb+ |

### Configuration vMotion/DRS
```bash
# Pré-requis réseau vMotion
- VLAN dédié recommandé (isolation trafic)
- Bande passante: minimum 1Gb, recommandé 10Gb
- Latence < 5ms entre hôtes
- MTU cohérent (9000 pour jumbo frames)

# Niveaux DRS
Conservative (niveau 1): Manuel
Moderately Conservative (2): Recommandations conservatives  
Moderate (3): Équilibre (défaut)
Moderately Aggressive (4): Recommandations agressives
Fully Automated (5): Migration automatique
```

## 💾 Microsoft Hyper-V

### Éditions Hyper-V
| Édition | Usage | Limitations |
|---------|-------|-------------|
| **Hyper-V Server** | Gratuit, bare-metal | Pas GUI, PowerShell only |
| **Windows Server** | Rôle intégré | Selon licence Windows |
| **Windows 10/11 Pro** | Client, développement | Performance limitée |

### Structure Fichiers Hyper-V
```bash
# Emplacements par défaut
C:\ProgramData\Microsoft\Windows\Hyper-V\   # Configurations
C:\Users\Public\Documents\Hyper-V\          # Disques par défaut

# Fichiers VM
VM_NAME.vmcx                # Configuration VM
VM_NAME.vmrs                # Runtime state  
VM_NAME.vhdx                # Disque virtuel
CheckPoints\                # Snapshots/Checkpoints
```

### PowerShell Hyper-V
```powershell
# Commandes essentielles
Get-VM                      # Lister VMs
New-VM -Name "TestVM" -MemoryStartupBytes 2GB
Start-VM -Name "TestVM"
Stop-VM -Name "TestVM"
Get-VMSwitch               # Switches virtuels
New-VMSwitch -Name "External" -NetAdapterName "Ethernet"

# Checkpoints (Snapshots)
Checkpoint-VM -Name "TestVM" -SnapshotName "Before Update"
Restore-VMSnapshot -Name "Before Update" -VMName "TestVM"
```

## 🌊 Migration & Mobilité

### Types de Migration
| Type | Downtime | Pré-requis | Usage |
|------|----------|------------|-------|
| **Cold Migration** | Complet | Stockage accessible | Maintenance planifiée |
| **Live Migration** | Aucun | Stockage partagé/répliqué | Production continue |
| **Cross-host** | Variable | Compatibilité hyperviseur | Changement infrastructure |

### VMware vMotion Requirements
```bash
# Compatibilité CPU
- Même famille processeur ou masquage EVC
- Fonctionnalités CPU compatibles
- Mode EVC (Enhanced vMotion Compatibility) si mixte

# Réseau
- Accès aux mêmes réseaux/VLANs
- vmkernel port vMotion configuré
- Résolution DNS bidirectionnelle

# Stockage  
- Accès partagé aux datastores VM
- Même chemins datastores
- Permissions cohérentes
```

### Hyper-V Live Migration
```powershell
# Configuration Live Migration
Enable-VMMigration              # Activer fonctionnalité
Set-VMMigrationNetwork 192.168.1.0 # Réseau dédié migration
Add-VMMigrationNetwork 192.168.2.0 # Réseau supplémentaire

# Types stockage
# Shared Nothing: Migration complète (VM + stockage)
# Shared Storage: Migration VM uniquement
# Storage Migration: Migration stockage uniquement
```

## 🔌 Réseaux Virtuels

### Types de Switches Virtuels
| Type | Connectivité | Usage |
|------|--------------|-------|
| **External** | Réseau physique | Production, accès réseau |
| **Internal** | Host + VMs | Isolation partielle |
| **Private** | VMs uniquement | Isolation complète |

### VMware vSphere Networking
```bash
# Standard Switch (vSS)
- Par hôte ESXi
- Configuration manuelle chaque hôte
- Maximum 4096 ports

# Distributed Switch (vDS)  
- Centralisé vCenter
- Configuration cohérente cluster
- Fonctionnalités avancées (QoS, monitoring)
- Licences Enterprise Plus requises
```

### Configuration réseau robuste
```bash
# Bonding/Teaming réseau (ESXi)
Route based on originating port ID    # Load balancing simple
Route based on IP hash               # Meilleur load balancing
Route based on source MAC hash       # Pour certains switches
Use explicit failover order          # Actif/Passif uniquement

# Hyper-V NIC Teaming
LACP (IEEE 802.3ad)                  # Standard, switch managé requis
Static Teaming                       # Configuration manuelle
Switch Independent                    # Pas configuration switch
```

## 💡 Optimisation & Performance

### Dimensionnement Ressources
| Ressource | Règle Générale | Surveillance |
|-----------|----------------|--------------|
| **CPU** | 1:4 ratio vCPU/pCPU max | CPU Ready time < 5% |
| **Mémoire** | Pas overcommit critique | Ballooning < 5% |
| **Stockage** | IOPS selon workload | Latence < 20ms |
| **Réseau** | Bande passante agrégée | Utilisation < 80% |

### VMware Tools & Integration Services
```bash
# VMware Tools fonctions
- Pilotes optimisés (VMXNET3, PVSCSI)
- Synchronisation horloge
- Operations VM (shutdown propre)
- Copier/coller entre host/guest
- Shared folders
- Memory ballooning

# Hyper-V Integration Services
- Synthetic drivers (réseau, stockage)
- Time synchronization
- Heartbeat service  
- VSS integration (backup)
- Guest services (file copy)
```

### Bonnes Pratiques Performance
- ✅ **VMware Tools/Integration Services** toujours à jour
- ✅ **Thin provisioning** disques pour optimisation espace
- ✅ **Réservation mémoire** pour VMs critiques
- ✅ **Anti-affinité** VMs critiques sur hôtes différents
- ✅ **Monitoring** proactif ressources et performances

## 📊 Monitoring & Alertes

### Métriques Critiques
| Métrique | Seuil Attention | Seuil Critique |
|----------|-----------------|----------------|
| **CPU Ready** | > 2% | > 5% |
| **Memory Ballooning** | > 1% | > 5% |
| **Datastore Latency** | > 15ms | > 25ms |
| **Network Dropped Packets** | > 0.1% | > 1% |

### Outils Monitoring
- **vCenter** : Performance charts, alarms
- **ESXTOP** : Monitoring temps réel ESXi
- **Perfmon** : Monitoring Hyper-V Windows
- **PRTG/Zabbix** : Monitoring infrastructure étendu

## 🔒 Sécurité & Isolation

### Niveaux d'Isolation
```
Physical Separation > Network Segmentation > VM Isolation > Process Isolation
        ↑                       ↑                 ↑              ↑
   Air Gap              VLANs/Firewalls    Hypervisor    Container/OS
```

### Sécurisation Hyperviseur
- ✅ **Hardening** selon guides constructeur (VMware/Microsoft)
- ✅ **Network segmentation** : Management, vMotion, VM networks
- ✅ **Access control** : RBAC, comptes dédiés, MFA
- ✅ **Updates** régulières hyperviseur et firmware
- ✅ **Logging/Monitoring** accès et modifications

## 🚀 Alternatives Modernes

### Containers vs VMs
| Aspect | Virtual Machines | Containers |
|--------|------------------|------------|
| **Isolation** | Forte (kernel séparé) | Processus (kernel partagé) |
| **Overhead** | Élevé (OS complet) | Faible (binaires uniquement) |
| **Démarrage** | Minutes | Secondes |
| **Portabilité** | Limitée | Excellente |
| **Use Case** | Legacy, isolation forte | Microservices, DevOps |

### Orchestration Moderne
- **Kubernetes** : Orchestration containers
- **Docker Swarm** : Clustering Docker natif  
- **VMware Tanzu** : Kubernetes sur vSphere
- **Azure Arc** : Gestion hybride multi-cloud

---
**💡 Memo** : VT-x/AMD-V obligatoire, VMware Tools essentiels, monitoring CPU Ready < 5%, isolation réseau critique !
