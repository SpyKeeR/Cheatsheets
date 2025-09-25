# 💾 Stockage Windows — Aide-mémoire

## 📁 Systèmes de Fichiers

### Comparaison Formats
| Système | Taille Max Fichier | Taille Max Volume | Fonctionnalités | Usage |
|---------|-------------------|-------------------|-----------------|--------|
| **NTFS** | 256 TB | 256 TB | Permissions, compression, chiffrement | Windows moderne |
| **ReFS** | 35 PB | 35 PB | Intégrité, auto-réparation | Windows Server |
| **FAT32** | 4 GB | 8 TB | Simple, compatible | Supports amovibles |
| **exFAT** | 16 EB | 64 ZB | Pas de limite pratique | Gros supports amovibles |

### Fonctionnalités NTFS Avancées
```
Sécurité:
├── ACL granulaires (utilisateur/groupe)
├── Audit accès fichiers
├── Chiffrement EFS (Encrypting File System)
└── Quotas utilisateur

Performance:
├── Compression transparente
├── Liens symboliques et jonctions
├── Sparse files (fichiers épars)
└── Journalisation ($LogFile)

Gestion:
├── Volume Shadow Copy (VSS)
├── Indexation Windows Search
├── Déduplication (Windows Server)
└── Tiering automatique (SSD + HDD)
```

## 🗂️ Tables de Partition

### MBR vs GPT
| Critère | MBR (Master Boot Record) | GPT (GUID Partition Table) |
|---------|--------------------------|----------------------------|
| **Taille max disque** | 2.2 TB | 9.4 ZB |
| **Partitions primaires** | 4 max | 128 par défaut Windows |
| **Firmware** | BIOS Legacy | UEFI (recommandé) |
| **Protection** | Secteur unique | Sauvegarde en fin de disque |
| **Compatibilité** | Anciens OS | Windows 7+ / Linux moderne |

### Conversion MBR ↔ GPT
```cmd
# Avec Diskpart (destructif, perte données)
diskpart
list disk
select disk 1
clean
convert gpt

# Avec mbr2gpt (non-destructif, Windows 10+)
mbr2gpt /validate /disk:1
mbr2gpt /convert /disk:1
```

## 🛠️ Outils de Gestion

### Disk Management (diskmgmt.msc)
```
Interface graphique pour:
├── Initialiser nouveaux disques
├── Créer/supprimer/redimensionner partitions
├── Attribuer lettres de lecteur
├── Configurer volumes dynamiques
└── Vérifier santé disques
```

### Diskpart (CLI Avancé)
```cmd
# Navigation et sélection
diskpart
list disk                    # Lister disques physiques
list volume                  # Lister tous volumes
select disk 1               # Sélectionner disque
select volume E:            # Sélectionner volume par lettre
detail disk                 # Détails disque sélectionné

# Initialisation
clean                       # Effacer table partitions (ATTENTION!)
convert gpt                 # Convertir en GPT
convert mbr                 # Convertir en MBR

# Gestion partitions
create partition primary size=50000    # Créer partition 50GB
create partition extended             # Partition étendue (MBR)
create partition logical size=10000   # Partition logique dans étendue
delete partition                      # Supprimer partition sélectionnée
active                               # Rendre partition bootable (MBR)

# Formatage et lettres
format fs=ntfs quick label="DATA"    # Formatage rapide
assign letter=E                      # Attribuer lettre
assign mount="C:\MountPoint"         # Monter dans dossier
remove                               # Supprimer lettre lecteur
```

### PowerShell Storage (Moderne)
```powershell
# Gestion disques
Get-Disk                            # Lister disques
Get-Disk | Where-Object PartitionStyle -eq "RAW"  # Disques non initialisés
Initialize-Disk -Number 1 -PartitionStyle GPT     # Initialiser en GPT
Set-Disk -Number 1 -IsOffline $false             # Mettre en ligne

# Partitions et volumes
Get-Partition                       # Toutes partitions
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
New-Partition -DiskNumber 1 -Size 50GB -DriveLetter F
Resize-Partition -DriveLetter E -Size 100GB         # Redimensionner

# Formatage
Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel "DATA" -Confirm:$false
Format-Volume -DriveLetter F -FileSystem ReFS -AllocationUnitSize 4096

# Points de montage
New-Volume -DiskNumber 1 -FriendlyName "Backup" -FileSystem NTFS
Add-PartitionAccessPath -DriveLetter E -AccessPath "C:\Mount\Backup"
```

## 🔧 Types de Volumes

### Volumes de Base (Recommandé)
```
Simple Volume:
├── Une partition sur un disque
├── Compatible avec tous OS
├── Sauvegarde/restauration simple
└── Maintenance facilitée

Extended Volume (MBR seulement):
├── Contient partitions logiques
├── Dépasse limite 4 partitions primaires
└── Plus complexe à gérer
```

### Volumes Dynamiques (Legacy)
| Type | Description | Disques Min | Tolérance Panne |
|------|-------------|-------------|-----------------|
| **Simple** | Volume standard | 1 | ❌ |
| **Spanned** | Agrégation capacité | 2-32 | ❌ |
| **Striped** | RAID 0 (performance) | 2-32 | ❌ |
| **Mirrored** | RAID 1 (redondance) | 2 | ✅ 1 disque |
| **RAID-5** | Parité distribuée | 3-32 | ✅ 1 disque |

```powershell
# Conversion en dynamique (ATTENTION: irréversible facilement)
Convert-Disk -Number 1 -PartitionStyle Dynamic

# Gestion volumes dynamiques (exemple miroir)
New-Volume -StoragePoolFriendlyName "Storage Pool" -FriendlyName "Mirror" -ResiliencySettingName Mirror -Size 100GB
```

## ⚡ RAID & Espaces de Stockage

### RAID Matériel vs Logiciel
```
RAID Matériel:
├── Contrôleur dédié (carte PCIe/intégré)
├── Performance optimale
├── Transparent pour l'OS
├── Batterie backup pour cache
└── Coût plus élevé

RAID Logiciel:
├── Géré par l'OS
├── Utilise CPU système
├── Plus flexible (différents types disques)
├── Moins cher
└── Espaces de stockage Windows (moderne)
```

### Espaces de Stockage (Storage Spaces)
```powershell
# Créer pool de stockage
New-StoragePool -FriendlyName "MonPool" -StorageSubsystems (Get-StorageSubsystem -FriendlyName "*Windows*") -PhysicalDisks (Get-PhysicalDisk -CanPool $true)

# Types de résilience
New-VirtualDisk -StoragePoolFriendlyName "MonPool" -FriendlyName "Simple" -ResiliencySettingName Simple -Size 100GB
New-VirtualDisk -StoragePoolFriendlyName "MonPool" -FriendlyName "Mirror" -ResiliencySettingName Mirror -Size 100GB
New-VirtualDisk -StoragePoolFriendlyName "MonPool" -FriendlyName "Parity" -ResiliencySettingName Parity -Size 100GB

# Gestion pool
Get-StoragePool
Get-VirtualDisk
Repair-VirtualDisk -FriendlyName "Mirror"  # Réparer si disque défaillant
```

### Comparaison Types RAID
| RAID | Disques Min | Capacité Utile | Performance | Tolérance | Usage |
|------|-------------|----------------|-------------|-----------|--------|
| **0** | 2 | 100% | Lecture/écriture élevée | Aucune | Performance pure |
| **1** | 2 | 50% | Lecture élevée | 1 disque | Données critiques |
| **5** | 3 | ~83% (n-1/n) | Lecture élevée | 1 disque | Compromis coût/perf |
| **6** | 4 | ~75% (n-2/n) | Lecture élevée | 2 disques | Haute sécurité |
| **10** | 4 | 50% | Très élevée | 50% disques | Performance + sécurité |

## 🔍 Diagnostic & Maintenance

### Vérification État Disques
```powershell
# Santé physique
Get-PhysicalDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus
Get-Disk | Where-Object HealthStatus -ne "Healthy"

# SMART data
Get-StorageReliabilityCounter -PhysicalDisk (Get-PhysicalDisk)[0]

# Espaces de stockage
Get-VirtualDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus
```

### Outils Intégrés Windows
```cmd
# Check Disk (CHKDSK)
chkdsk C: /f /r              # Vérifier et réparer (redémarrage requis)
chkdsk E: /f                 # Réparer erreurs seulement
chkdsk E: /r                 # Localiser secteurs défectueux

# System File Checker
sfc /scannow                 # Vérifier intégrité fichiers système

# Défragmentation
defrag C: /a                 # Analyser fragmentation
defrag C: /o                 # Défragmenter complet
defrag C: /x                 # Trim SSD (si applicable)
```

### PowerShell Maintenance
```powershell
# Défragmentation moderne
Optimize-Volume -DriveLetter C -Analyze           # Analyser
Optimize-Volume -DriveLetter C -Defrag            # Défragmenter HDD
Optimize-Volume -DriveLetter C -ReTrim            # Optimiser SSD

# Vérification intégrité
Repair-Volume -DriveLetter E -Scan               # Scanner erreurs
Repair-Volume -DriveLetter E -OfflineScanAndFix  # Réparation hors ligne
```

## 💡 Fonctionnalités Avancées

### Déduplication (Windows Server)
```powershell
# Activation sur volume
Enable-DedupVolume -Volume "D:" -UsageType Default

# Types d'usage optimisés
Enable-DedupVolume -Volume "E:" -UsageType FileServer    # Serveur fichiers
Enable-DedupVolume -Volume "F:" -UsageType HVDI          # VDI Hyper-V

# Monitoring déduplication
Get-DedupStatus -Volume D:
Get-DedupVolume
```

### Tiering Automatique
```powershell
# Création tier SSD + HDD
New-StorageTier -StoragePoolFriendlyName "MonPool" -FriendlyName "SSD" -MediaType SSD
New-StorageTier -StoragePoolFriendlyName "MonPool" -FriendlyName "HDD" -MediaType HDD

# Volume avec tiering
$ssdTier = Get-StorageTier -FriendlyName "SSD"  
$hddTier = Get-StorageTier -FriendlyName "HDD"
New-VirtualDisk -StoragePoolFriendlyName "MonPool" -FriendlyName "Tiered" -StorageTiers $ssdTier,$hddTier -StorageTierSizes 20GB,80GB
```

### VSS (Volume Shadow Copy)
```cmd
# Configuration copies instantanées
vssadmin list shadows          # Lister snapshots existants
vssadmin list providers        # Fournisseurs VSS
vssadmin create shadow /for=C: # Créer snapshot manuel

# Gestion via interface
# Propriétés lecteur > Versions précédentes > Paramètres
```

## 🚨 Récupération & Troubleshooting

### Problèmes Courants
| Symptôme | Cause Probable | Diagnostic | Solution |
|----------|----------------|------------|----------|
| **Disque non reconnu** | Driver/câble/alimentation | Device Manager, BIOS | Vérifier connexions |
| **RAW file system** | Corruption table partition | `chkdsk`, `testdisk` | Récupération données |
| **Lenteur extrême** | Disque défaillant | SMART, Event Viewer | Remplacer disque |
| **Erreurs I/O** | Secteurs défectueux | `chkdsk /r`, SMART | Sauvegarder puis remplacer |

### Commandes Récupération
```cmd
# Boot depuis média Windows
bootrec /fixmbr              # Réparer MBR
bootrec /fixboot             # Réparer secteur boot
bootrec /rebuildbcd          # Reconstruire BCD
bootrec /scanos              # Scanner OS installés

# Récupération GPT
gdisk /dev/sda               # Linux (GPT fdisk)
# Windows: diskpart + convert gpt après sauvegarde
```

### Sauvegarde Configuration
```powershell
# Exporter configuration disques
Get-Disk | Export-Clixml "C:\Backup\DisksConfig.xml"
Get-Partition | Export-Clixml "C:\Backup\PartitionsConfig.xml"

# Documentation volumes
Get-Volume | Select-Object DriveLetter, FileSystemLabel, Size, SizeRemaining, FileSystem | Export-Csv "C:\Backup\VolumesInfo.csv"
```

## 💡 Bonnes Pratiques

### Planification
- ✅ **Prévoir croissance** : 20-30% espace libre minimum
- ✅ **Séparer système/données** : OS sur SSD, données sur HDD
- ✅ **Documenter** : Configuration RAID, points montage
- ✅ **Tester récupération** : Validation procédures backup

### Sécurité
- ✅ **RAID ≠ Backup** : Redondance locale seulement
- ✅ **Chiffrement** : BitLocker pour données sensibles
- ✅ **Permissions** : Principe moindre privilège
- ✅ **Audit** : Traçabilité accès fichiers critiques

### Performance
- ✅ **SSD pour OS** : Système et applications sur SSD
- ✅ **Alignement 4K** : Partitions alignées sur 4096 bytes
- ✅ **Défragmentation** : HDD périodique, SSD trim seulement
- ✅ **Monitoring** : Surveillance SMART et performance

### Maintenance
- ✅ **Surveillance proactive** : Alertes SMART, espace disque
- ✅ **Sauvegarde régulière** : 3-2-1 rule (3 copies, 2 supports, 1 offsite)
- ✅ **Tests périodiques** : Vérification intégrité mensuelle
- ✅ **Plan de remplacement** : Cycle vie disques (3-5 ans)

---
**💡 Memo** : GPT > MBR pour nouveaux disques • Espaces de stockage > volumes dynamiques • RAID matériel > logiciel en prod !