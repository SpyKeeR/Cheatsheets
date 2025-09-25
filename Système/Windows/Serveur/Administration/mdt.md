# 🚀 Microsoft Deployment Toolkit (MDT) — Aide-mémoire

## 🎯 Vue d'Ensemble & Concepts

### Écosystème Déploiement Microsoft
```
WDS (Windows Deployment Services)
├── Rôle : Transport PXE + Boot Images
├── Port : 67 DHCP, 69 TFTP, 4011 ProxyDHCP
└── Fonction : Servir boot.wim vers clients PXE

MDT (Microsoft Deployment Toolkit)
├── Rôle : Orchestration + Séquences + Rules
├── Mode : LiteTouch (Manuel) / Zero-Touch (Automatisé)
└── Fonction : Logique déploiement + Customisation

ADK (Assessment & Deployment Kit)
├── DISM : Gestion images Windows
├── Windows PE : Environnement pré-installation
└── WSIM : Windows System Image Manager
```

### Types de Déploiement
| Type | Description | Usage | Automatisation |
|------|-------------|-------|----------------|
| **New Computer** | Installation propre | Nouveau matériel | Complète |
| **Refresh** | Réinstall + sauvegarde données | Mise à niveau OS | Partielle |
| **Replace** | Migration vers nouveau PC | Remplacement matériel | Avec USMT |

## 📦 Installation & Prérequis

### Stack Technique Requis
1. **Windows ADK** (Assessment and Deployment Kit)
2. **Windows PE Add-on** pour ADK (séparé depuis ADK 1809)
3. **MDT** (Microsoft Deployment Toolkit)
4. **PowerShell 5.1+** (inclus Windows Server)
5. **.NET Framework 4.6+**

### Vérification Installation
```powershell
# Vérifier composants ADK
Get-ItemProperty "HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows Kits\Installed Roots" -Name KitsRoot10 -ErrorAction SilentlyContinue

# Vérifier MDT
Test-Path "C:\Program Files\Microsoft Deployment Toolkit\bin\MicrosoftDeploymentToolkit.psd1"

# Importer module MDT
Import-Module "C:\Program Files\Microsoft Deployment Toolkit\bin\MicrosoftDeploymentToolkit.psd1"
```

## 🏗️ Architecture Deployment Share

### Structure Hiérarchique
```
C:\DeploymentShare\
├── Applications\              # Logiciels à déployer
│   ├── Microsoft Office 365\
│   ├── Adobe Reader DC\
│   └── Corporate Bundle\
├── Operating Systems\         # Images OS (WIM/ISO)
├── Out-of-Box Drivers\       # Pilotes par constructeur
│   ├── Dell\OptiPlex\
│   └── HP\EliteBook\
├── Packages\                 # Updates/Language Packs
├── Task Sequences\           # Séquences déploiement
├── Selection Profiles\       # Filtres contenu
├── Control\                  # Configuration files
│   ├── Bootstrap.ini         # Config WinPE
│   ├── CustomSettings.ini    # Règles déploiement
│   └── Settings.xml          # Paramètres DS
└── Boot\                     # Images générées
    ├── LiteTouchPE_x64.iso   # ISO bootable
    └── LiteTouchPE_x64.wim   # Image PXE
```

### Création Deployment Share
```powershell
# Créer Deployment Share
$DSPath = "C:\DeploymentShare"
New-Item -Path $DSPath -ItemType Directory -Force

# Créer partage SMB
New-SmbShare -Name "DeploymentShare$" -Path $DSPath -FullAccess "Domain\MDT-Admins"

# Monter comme PSDrive MDT
New-PSDrive -Name "DS001" -PSProvider MDTProvider -Root $DSPath -Description "Production Deployment Share"
```

## ⚙️ Configuration Files

### Bootstrap.ini (Configuration WinPE)
```ini
[Settings]
Priority=Default

[Default]
; === Accès Deployment Share ===
DeployRoot=\\mdtserver\DeploymentShare$
UserID=MDTService
UserPassword=P@ssw0rd!
UserDomain=DOMAIN

; === UI Control ===
SkipBDDWelcome=YES
SkipTaskSequence=NO
SkipComputerName=YES
SkipDomainMembership=YES
SkipUserData=YES
SkipApplications=YES
SkipBitLocker=YES
SkipSummary=YES
SkipCapture=YES

; === Monitoring ===
EventService=http://mdtserver:9800
SLShare=\\mdtserver\Logs$
```

### CustomSettings.ini (Règles Déploiement)
```ini
[Settings]
Priority=ByLaptop,ByDesktop,ByVM,Default
Properties=IsLaptop,IsDesktop,IsVM,SerialNumber,AssetTag

[Default]
; === Installation ===
OSInstall=YES
SkipCapture=YES
SkipAdminPassword=YES
SkipProductKey=YES

; === Naming Convention ===
OSDComputerName=#Left("%SerialNumber%",8)#
; Alternative: OSDComputerName=#Mid("%SerialNumber%",4,8)#
; === Domain Join ===
JoinDomain=DOMAIN.LOCAL
DomainAdmin=MDTService
DomainAdminDomain=DOMAIN
DomainAdminPassword=P@ssw0rd!
MachineObjectOU=OU=Workstations,DC=domain,DC=local

; === Regional ===
TimeZoneName=Romance Standard Time
KeyboardLocale=040c:0000040c
UserLocale=fr-FR
UILanguage=fr-FR
SystemLocale=fr-FR

; === Applications ===
SkipApplications=YES
; Mandatory apps for all deployments
MandatoryApplications001={12345678-1234-1234-1234-123456789012}
MandatoryApplications002={87654321-4321-4321-4321-210987654321}

; === Drivers ===
DriverSelectionProfile=Nothing

; === USER STATE ===
UserDataLocation=NONE
UDShare=\\fileserver\UserState$
UDDir=%OSDComputerName%

; === LOGGING ===
SLShare=\\mdtserver\logs$
SLShareDynamicLogging=\\mdtserver\logs$\%OSDComputerName%

; === MONITORING ===
EventService=http://mdtserver:9800

; === POWERSHELL EXECUTION ===
PowerShellExecutionPolicy=Bypass

[ByLaptop]
; Détection automatique laptop
Subsection=Laptop-%Make%

[ByDesktop]
; Détection automatique desktop
Subsection=Desktop-%Make%

[ByVM]
; Détection VM
Subsection=VirtualMachine

[Laptop-Dell]
; Configuration spécifique laptops Dell
DriverSelectionProfile=Dell Latitude Drivers
Applications001={GUID-Dell-Command-Update}

[Desktop-HP]
; Configuration spécifique desktops HP
DriverSelectionProfile=HP Elite Drivers
Applications001={GUID-HP-Support-Assistant}

[VirtualMachine]
; Configuration VMs
DriverSelectionProfile=Nothing
SkipBitLocker=YES
```

## 🔧 Gestion Composants

### Import Operating Systems
```powershell
# Import depuis ISO
Import-MDTOperatingSystem -Path "DS001:\Operating Systems" -SourcePath "D:\" -DestinationFolder "Windows 11 22H2 Pro"

# Import depuis dossier WIM
Import-MDTOperatingSystem -Path "DS001:\Operating Systems" -SourceFile "C:\Images\install.wim" -DestinationFolder "Windows 11 Custom" -Index 1
```

### Gestion Applications
```powershell
# Application avec source
Import-MDTApplication -Path "DS001:\Applications" -Name "Adobe Reader DC" -ShortName "Reader" -CommandLine "setup.exe /S" -SourcePath "\\server\software\Adobe"

# Application sans source (MSI réseau)
Import-MDTApplication -Path "DS001:\Applications" -Name "Google Chrome" -CommandLine "msiexec /i \\server\msi\chrome.msi /quiet" -NoSource
```

### Organisation Pilotes
```
Out-of-Box Drivers\
├── Dell\
│   ├── OptiPlex 7090\Windows 11\x64\
│   └── Latitude 5520\Windows 11\x64\
├── HP\  
│   ├── EliteBook 840\Windows 11\x64\
│   └── ProDesk 600\Windows 11\x64\
└── Lenovo\
    └── ThinkPad T14\Windows 11\x64\
```

### Selection Profiles
```powershell
# Profil drivers par constructeur
New-MDTSelectionProfile -Path "DS001:\Selection Profiles" -Name "Dell Drivers" -Comments "Pilotes Dell uniquement"

# Dans CustomSettings.ini, référencer par :
# DriverSelectionProfile=Dell Drivers
```

## 📋 Task Sequences

### Types de Task Sequences
| Template | Usage | Description |
|----------|--------|-------------|
| **Standard Client** | Workstation | Déploiement poste utilisateur |
| **Standard Server** | Serveur | Installation Windows Server |
| **Sysprep and Capture** | Capture | Créer image personnalisée |
| **Litetouch OEM** | OEM | Déploiement constructeur |

### Structure Task Sequence
```
Task Sequence Steps:
├── Initialization
│   ├── Gather local info
│   └── Validate network
├── Preinstall  
│   ├── Apply drivers
│   └── Configure disk
├── Install
│   ├── Install OS
│   └── Apply updates
├── Postinstall
│   ├── Install applications
│   ├── Configure settings
│   └── Join domain
└── State Restore
    ├── Restore user data
    └── Cleanup
```

### Conditions & Variables
```xml
<!-- Exemples conditions courantes -->
<!-- Installation conditionnelle par type machine -->
<condition>
  <expression type="SMS_TaskSequence_VariableConditionExpression">
    <variable name="IsLaptop" operator="equals" value="True"/>
  </expression>
</condition>

<!-- Installation selon OU destination -->
<condition>
  <expression type="SMS_TaskSequence_VariableConditionExpression">
    <variable name="MachineObjectOU" operator="like" value="*Developers*"/>
  </expression>
</condition>
```

## 📊 Monitoring & Database

### MDT Monitoring Service
```
Port : 9800 (HTTP)
URL : http://mdtserver:9800/MDTMonitorData/Computers
Données : Temps réel, étapes, erreurs, warnings
Format : JSON/XML REST API
```

### Surveillance PowerShell
```powershell
# État déploiements actifs
$Computers = Invoke-RestMethod -Uri "http://mdtserver:9800/MDTMonitorData/Computers"
$Computers | Select-Object Name, CurrentStep, PercentComplete, Status | Format-Table

# Filtrer déploiements en erreur  
$Computers | Where-Object {$_.Errors -gt 0} | Select-Object Name, Errors, Warnings
```

### Database Integration (Optionnel)
```
Avantages Database :
├── Configuration centralisée par machine
├── Gestion par rôles/localisations  
├── Historique déploiements
└── Intégration CMDB/Asset Management

Tables principales :
├── ComputerSettings (config par machine)
├── LocationSettings (config par site)
├── RoleSettings (config par rôle)
└── MakeModelSettings (config par modèle)
```

## 🔍 Variables & Detection Rules

### Variables Courantes MDT
| Variable | Description | Exemple |
|----------|-------------|---------|
| `%SerialNumber%` | Numéro série | PC123456 |
| `%AssetTag%` | Tag inventaire | AST001234 |
| `%Make%` | Constructeur | Dell Inc. |
| `%Model%` | Modèle | OptiPlex 7090 |
| `%IsLaptop%` | Détection portable | True/False |
| `%IsDesktop%` | Détection fixe | True/False |
| `%IsVM%` | Machine virtuelle | True/False |
| `%MacAddress001%` | Adresse MAC | 00:50:56:C0:00:08 |

### Custom Detection Scripts
```vbscript
' ZTIGather.xml - Variables personnalisées
Function UserIsLaptop
    UserIsLaptop = False
    Set objWMI = GetObject("winmgmts:")
    Set colItems = objWMI.ExecQuery("SELECT * FROM Win32_Battery")
    If colItems.Count > 0 Then UserIsLaptop = True
End Function

' Dans CustomSettings.ini
IsLaptop=#UserIsLaptop()#
```

## 🚨 Troubleshooting

### Logs Principaux
| Phase | Localisation | Fichier Clé |
|-------|--------------|-------------|
| **WinPE** | `X:\MININT\SMSOSD\OSDLOGS\` | BDD.log, SMSTS.log |
| **Windows Setup** | `C:\Windows\Panther\` | setupact.log, setuperr.log |  
| **Post-Install** | `C:\MININT\SMSOSD\OSDLOGS\` | Deploy.log |
| **Centralisé** | `\\server\Logs$\ComputerName\` | Tous logs copiés |

### Problèmes Fréquents
| Symptôme | Cause Probable | Diagnostic |
|----------|----------------|------------|
| **Boot PXE échoue** | WDS non configuré | Vérifier service WDS + DHCP options |
| **Credentials rejected** | Bootstrap.ini incorrect | Tester compte service manuellement |
| **No task sequences** | Permissions DS | Vérifier ACL Deployment Share |
| **Drivers not found** | Selection Profile | Vérifier DriverSelectionProfile setting |
| **Domain join fail** | DNS/Permissions | Test résolution + droits compte service |

### Commandes Diagnostics
```cmd
# Dans WinPE, diagnostics réseau
ipconfig /all
nslookup domain.local
telnet mdtserver 9800

# Vérifier variables MDT
cscript X:\Deploy\Scripts\ZTIGather.wsf
type X:\MININT\SMSOSD\OSDLOGS\ZTIGather.log

# Test accès Deployment Share
net use Z: \\mdtserver\DeploymentShare$ /user:domain\mdtservice
```

## 🔄 Intégration WDS

### Configuration WDS pour MDT
```powershell
# Ajouter images boot MDT dans WDS
Import-WdsBootImage -Path "C:\DeploymentShare\Boot\LiteTouchPE_x64.wim" -NewImageName "MDT Boot Image x64"

# Configuration PXE Response
Set-WdsServer -NewMachineNamingPolicy -PolicyType UserDefined -TemplateString "PC%SerialNumber%"

# Options DHCP (si WDS et DHCP sur même serveur)
# Option 60 = "PXEClient"  
# Option 66 = IP du serveur WDS
# Option 67 = "boot\x64\pxeboot.n12"
```

### Boot Options Advanced
```
F12 PXE Boot Menu:
├── Continue normal boot
├── MDT Production (LiteTouchPE_x64.wim)
├── MDT Capture (Custom capture sequence)
└── Windows PE Debug (Troubleshooting)
```

## 💡 Bonnes Pratiques

### Architecture
- ✅ **Deployment Share** dédié sur disque rapide (SSD)
- ✅ **Réplication** DS sur sites distants (DFS-R)
- ✅ **Separation** environnements (Dev/Test/Prod)
- ✅ **Selection Profiles** pour optimiser contenu
- ✅ **Monitoring** activé pour suivi temps réel

### Sécurité  
- ✅ **Compte service** dédié MDT (pas admin domaine)
- ✅ **Permissions** restrictives sur Deployment Share
- ✅ **Chiffrement** Bootstrap.ini (si passwords)
- ✅ **Audit** déploiements via logs centralisés
- ✅ **Isolation** réseau pour PXE (VLAN séparé)

### Performance
- ✅ **Drivers** organisés par constructeur/modèle
- ✅ **Applications** bundlées par profil utilisateur
- ✅ **WinPE** optimisé (drivers réseau uniquement)
- ✅ **Multicast** WDS pour déploiements simultanés
- ✅ **Cache local** pour gros packages (Office, etc.)

### Maintenance
- ✅ **Versionning** Task Sequences (backup before changes)
- ✅ **Test** systematic sur VM avant production
- ✅ **Documentation** naming conventions
- ✅ **Cleanup** régulier logs et anciennes images
- ✅ **Updates** mensuels ADK, MDT, drivers

---
**💡 Memo** : Bootstrap.ini (WinPE config) + CustomSettings.ini (deployment rules) + PXE boot = déploiement automatisé !