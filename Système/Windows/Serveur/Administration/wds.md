# 🚀 Déploiement OS Windows — WDS, MDT & Modern Deployment

## 🏗️ Solutions de Déploiement

### Comparatif Technologies Microsoft
| Solution | Complexité | Scalabilité | Automation | Use Case |
|----------|------------|-------------|------------|----------|
| **WDS** | Faible | Moyenne | Limitée | PXE boot basique |
| **MDT** | Moyenne | Élevée | Complète | Déploiement automatisé |
| **ConfigMgr** | Élevée | Très élevée | Complète | Enterprise lifecycle |
| **Autopilot** | Faible | Élevée | Cloud | Modern workplace |
| **WinGet** | Faible | Moyenne | Apps | Package management |

### Technologies Alternatives
- **FOG Project** : Open source, léger
- **Clonezilla** : Clone disk-to-disk
- **SmartDeploy** : Commercial, user-friendly
- **Acronis Snap Deploy** : Enterprise imaging

## 🖥️ Windows Deployment Services (WDS)

### Architecture & Prérequis
```
Infrastructure Requise:
├── Active Directory (recommandé)
├── DHCP Server (obligatoire)
├── DNS Server (recommandé)
└── Stockage réseau suffisant

Services WDS:
├── Deployment Server (core)
├── Transport Server (PXE/TFTP)
└── PXE Server (boot management)
```

### WDS
- Rôle Windows : PXE boot + hébergement `boot.wim` (WinPE) & `install.wim`.  
- Services requis : DHCP (obligatoire), DNS (recommandé), AD (fortement recommandé).  
- Modes PXE : DHCP+WDS même serveur (options 66/67) OU DHCP séparé + WDS en ProxyDHCP.  
- Install (PowerShell) : `Install-WindowsFeature -Name WDS -IncludeManagementTools`.  
- Config initiale : choix domaine/hors-domaine, dossier REMINST, mode réponse PXE (autoriser/tarterget).  
- Ajout d'images : importer boot.wim (Images de démarrage) puis install.wim (Images d'installation).  
- Pilotes : ajout manuel ou via MDT, organiser par modèle.

### ConfigMgr (si enterprise)
- Déploiement + patching + inventaire + distribution applis (usage grande infra).

### Types d'images
- Image de partition : snapshot complet (rapide, matériel homogène requis).  
- Image WIM (install.wim) : installation dynamique (flexible, multi-matériel).  
- Choix = homogénéité matérielle vs flexibilité.

### Capture d'image (workflow)
1. Préparer poste modèle → `sysprep` (C:\Windows\System32\Sysprep\sysprep.exe).  
2. Créer image de capture liée à une image boot dans WDS.  
3. PXE boot poste → lancer assistant de capture → envoyer .wim au serveur WDS.

### Unattend.xml (fichiers de réponse)
- WinPE (phase 1) → placer `unattend.xml` dans `\\<WDS>\REMINST\WDSClientUnattend`.  
- Install images (phases 2-7) → associer le `unattend.xml` à l'image d'installation (propriétés image).  
- Outils : WSIM (ADK), PowerShell, éditeurs GUI.

### PXE flow résumé (8 étapes)
1. Client PXE -> DHCPDISCOVER  
2. DHCPOFFER (DHCP +/- PXE options)  
3. DHCPREQUEST → DHCPACK  
4. Client récupère option 66/67 (TFTP/boot file)  
5. Téléchargement boot.wim via TFTP/HTTP  
6. WinPE démarre → exécute séquence MDT/WDS  
7. Récupération install.wim et installation  
8. Post-install (drivers, apps, join AD, finalisation via unattend)

### Scénarios de déploiement
- Bare Metal (nouveau) — full OS fresh.  
- Réinstallation (wipe & load) — réparer / conserver données selon méthode.  
- Remplacement (migrate) — capture ancien → restore sur nouveau.  
- In-place Upgrade — upgrade sans formatage.

### WDS / DHCP cohabitation (rappel rapide)
- Même serveur : DHCP fournit 66/67 ; WDS n'écoute pas sur ports DHCP.  
- Serveurs distincts : activer Proxy DHCP sur WDS pour fournir uniquement options PXE.

### Bonnes pratiques (checklist)
- Intégrer AD, utiliser DNS interne.  
- Stockage images sur disque dédié.  
- Organiser drivers par modèle ; utiliser MDT pour injecter drivers.  
- Utiliser unattend.xml pour automatisation complète.  
- Tester capture → déploiement sur modèles représentatifs.  
- Mettre en place inventaire & versions d'image (naming / serial : AAAAMMJJ).  
- Sauvegarder répertoire REMINST / backups WDS.  
- Gestion licences Windows + activation KMS/AAD si nécessaire.  
- Automatiser via PowerShell / MDT pour reproductibilité.

### Commandes utiles (rapides)
- Installer WDS : `Install-WindowsFeature -Name WDS -IncludeManagementTools`  
- Sysprep (local) : lancer `sysprep.exe` → Generalize / Shutdown.  
- Vérifier/importer images : via console WDS / MDT ou scripts PowerShell WDS.

### Installation & Configuration
```powershell
# Installation rôle WDS
Install-WindowsFeature -Name WDS -IncludeManagementTools

# Configuration initiale via PowerShell
wdsutil /Initialize-Server /RemInst:"C:\RemoteInstall"
wdsutil /Set-Server /AnswerClients:All

# Ou via GUI
# Server Manager > Add Roles > Windows Deployment Services
```

### Configuration DHCP Integration
```powershell
# Même serveur DHCP/WDS
wdsutil /Set-Server /UseDhcpPorts:No /DhcpOption60:Yes

# Serveurs séparés - Configuration DHCP
netsh dhcp server \\dhcp-server scope 192.168.1.0 set optionvalue 66 STRING "192.168.1.10"  # IP WDS
netsh dhcp server \\dhcp-server scope 192.168.1.0 set optionvalue 67 STRING "boot\\x64\\wdsnbp.com"  # Boot file
```

### Gestion Images
```powershell
# Importer image de démarrage
wdsutil /Add-Image /ImageFile:"C:\Sources\boot.wim" /ImageType:Boot

# Importer image d'installation
wdsutil /Add-Image /ImageFile:"C:\Sources\install.wim" /ImageType:Install /ImageGroup:"Windows 11"

# Lister images
wdsutil /Get-AllImages /Show:Images

# Supprimer image
wdsutil /Remove-Image /Image:"Windows 11 Pro" /ImageType:Install /ImageGroup:"Windows 11"
```

### Types d'Images
| Type | Description | Avantages | Inconvénients |
|------|-------------|-----------|---------------|
| **Boot Image** | Windows PE pour démarrage | Universelle | Taille importante |
| **Install Image** | OS de base | Flexible, multi-hardware | Installation plus lente |
| **Capture Image** | Image personnalisée | Pré-configurée | Hardware spécifique |
| **Discover Image** | PXE alternatif | Contourne limitations DHCP | Configuration manuelle |


### Structure Deployment Share
```
C:\DeploymentShare\
├── Applications\              # Applications à déployer
├── Operating Systems\         # Images OS
├── Out-of-Box Drivers\       # Pilotes
├── Packages\                 # Updates/Language packs
├── Task Sequences\           # Séquences de déploiement
├── Selection Profiles\       # Profils de sélection
└── Boot Images\             # Images de démarrage générées
```

### Création Task Sequence
```powershell
# Via PowerShell MDT
Import-Module "C:\Program Files\Microsoft Deployment Toolkit\bin\MicrosoftDeploymentToolkit.psd1"

New-PSDrive -Name "DS001" -PSProvider MDTProvider -Root "C:\DeploymentShare"

# Importer OS
Import-MDTOperatingSystem -Path "DS001:\Operating Systems" -SourcePath "C:\Sources\Windows11" -DestinationFolder "Windows 11 22H2"

# Créer Task Sequence
Import-MDTTaskSequence -Path "DS001:\Task Sequences" -Name "Deploy Windows 11" -Template "Client.xml" -ID "WIN11" -OperatingSystemPath "DS001:\Operating Systems\Windows 11 22H2"
```

### CustomSettings.ini Configuration
```ini
[Settings]
Priority=Default
Properties=MyCustomProperty

[Default]
OSInstall=Y
SkipCapture=YES
SkipAdminPassword=NO
SkipProductKey=YES
SkipComputerBackup=YES
SkipBitLocker=YES

; Active Directory Join
JoinDomain=domain.local
DomainAdmin=deployer
DomainAdminDomain=domain.local
DomainAdminPassword=P@ssw0rd!

; Time Zone
TimeZoneName=Romance Standard Time

; Regional Settings
KeyboardLocale=040c:0000040c
UserLocale=fr-FR
UILanguage=fr-FR

; Applications
MandatoryApplications001={GUID-App1}
MandatoryApplications002={GUID-App2}

; Drivers
DriverSelectionProfile=All Drivers

; Logging
SLShare=\\server\logs$
SLShareDynamicLogging=\\server\logs$\%OSDComputerName%
```

### Bootstrap.ini Configuration
```ini
[Settings]
Priority=Default

[Default]
DeployRoot=\\server\DeploymentShare$
SkipBDDWelcome=YES

; Credentials for deployment share access
UserID=deployer
UserPassword=P@ssw0rd!
UserDomain=domain.local

; Skip wizard pages
SkipTaskSequence=NO
SkipComputerName=NO
SkipDomainMembership=YES
SkipUserData=YES
SkipApplications=YES
SkipBitLocker=YES
SkipSummary=YES
SkipRoles=YES
SkipCapture=YES
SkipFinalSummary=NO
```


## 🌐 Modern Deployment (Cloud)

### Windows Autopilot
```powershell
# Installation module Autopilot
Install-Module -Name WindowsAutoPilotIntune -Force

# Récupération Hardware Hash
Get-WindowsAutoPilotInfo -OutputFile "C:\Autopilot\devices.csv"

# Import dans Intune via Graph API
Connect-MSGraph
Import-AutoPilotCSV -csvFile "C:\Autopilot\devices.csv"
```

### Configuration Autopilot Profile
```json
{
    "displayName": "Corporate Deployment",
    "description": "Standard corporate deployment profile",
    "language": "fr-FR",
    "locale": "fr-FR",
    "timezone": "Romance Standard Time",
    "hideEULA": true,
    "hidePrivacySettings": true,
    "hideChangeAccountOpts": true,
    "skipKeyboardSelectionPage": true,
    "deviceNameTemplate": "CORP-%SERIAL%",
    "enableWhiteGlove": true,
    "outOfBoxExperienceSettings": {
        "hidePrivacySettings": true,
        "hideEULA": true,
        "userType": "standard"
    }
}
```

### WinGet Package Management
```yaml
# winget-config.yaml
applications:
  - id: Microsoft.VisualStudioCode
    source: winget
  - id: Google.Chrome
    source: winget
  - id: 7zip.7zip
    source: winget
  - id: Adobe.Acrobat.Reader.64-bit
    source: winget

# Deployment script
winget import -i winget-config.yaml --accept-package-agreements --accept-source-agreements
```

## 📊 Monitoring & Logging

### WDS Event Logs
| Log Location | Event IDs | Description |
|--------------|-----------|-------------|
| **Microsoft-Windows-Deployment-Services-Diagnostics/Operational** | 4097-4115 | Client deployment events |
| **System** | 7001,7002 | WDS service events |
| **Microsoft-Windows-Deployment-Services/Operational** | 513,515 | Image serving events |
| **Application** | Various | MDT task sequence logs |

## 🛠️ Troubleshooting

### Diagnostics PXE Boot
```bash
# Flow PXE debugging
1. DHCP Discovery    -> Vérifier réponse DHCP
2. PXE Options       -> Contrôler options 66/67
3. TFTP Download     -> Tester connectivité TFTP
4. Boot Image Load   -> Vérifier intégrité boot.wim
5. WinPE Start       -> Logs WinPE (X:\Windows\Temp\SMSTSLog)
```

### Commandes Diagnostics
```powershell
# Test connectivité WDS
Test-NetConnection -ComputerName "wds-server" -Port 69   # TFTP
Test-NetConnection -ComputerName "wds-server" -Port 4011 # PXE

# Vérification services WDS
Get-Service WDSServer, WDSUTIL

# Logs détaillés WDS
wdsutil /Set-Server /Verbose:Yes

# Test DHCP options
ipconfig /all  # Vérifier options DHCP reçues
```

### Problèmes Courants
| Problème | Cause Probable | Solution |
|----------|----------------|----------|
| **PXE-E53** | DHCP/PXE config | Vérifier options 66/67 |
| **Boot loop** | Image corrompue | Re-capturer/importer image |
| **Driver missing** | Pilotes absents | Ajouter drivers out-of-box |
| **Domain join fail** | Credentials/DNS | Vérifier compte service |
| **Slow deployment** | Réseau/stockage | Optimiser infra réseau |

## 📋 Best Practices & Security

### Sécurisation Déploiement
```powershell
# Sécuriser WDS
# 1. Authentification requise
wdsutil /Set-Server /AnswerClients:Known

# 2. Approbation manuelle
wdsutil /Set-Server /NewMachineNamingPolicy:%Username%#

# 3. Limitation par groupe AD
# Via WDS Console > Properties > Client > Enable unattended installation
```

### Architecture Recommandée
```
Production Deployment Architecture:
├── WDS Server (Dedicated)
│   ├── RAID 1 System
│   ├── RAID 5/10 Storage (Images)
│   └── 1Gbps Network minimum
├── MDT Server (Can be same as WDS)
├── File Server (Drivers/Apps)
├── DHCP Server (Redundant)
└── Network Infrastructure
    ├── Gigabit switches
    ├── VLAN segmentation
    └── QoS for deployment traffic
```

### Checklist Déploiement
- [ ] Infrastructure réseau validée (DHCP/DNS/AD)
- [ ] Stockage suffisant calculé (images + logs)
- [ ] Sauvegarde REMINST configurée
- [ ] Tests sur matériel représentatif effectués
- [ ] Documentation procédures maintenue
- [ ] Scripts automation versionnés
- [ ] Monitoring et alerting en place
- [ ] Formation équipe effectuée
- [ ] Plan de rollback défini
- [ ] Licences Windows vérifiées
