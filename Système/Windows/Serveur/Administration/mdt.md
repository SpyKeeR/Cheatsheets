# 🚀 Microsoft Deployment Toolkit (MDT) — Cheatsheet

## 🎯 Vue d'ensemble

### Rôle & Positionnement
- **MDT** : Orchestration et automatisation complète du déploiement
- **WDS** : Transport PXE et hébergement des images boot
- **Workflow** : WDS sert le boot.wim généré par MDT → MDT exécute les séquences

### Avantages Clés
✅ **Zero-Touch Deployment** (ZTD) possible  
✅ **Standardisation** des déploiements  
✅ **Monitoring** temps réel intégré  
✅ **Extensibilité** via scripts PowerShell/VBS  
✅ **Base de données** centralisée pour configuration

## 📦 Installation & Prérequis

### Stack Technique Requis
```
1. Windows ADK (Assessment and Deployment Kit)
2. Windows PE add-on pour ADK
3. Microsoft Deployment Toolkit (MDT)
4. PowerShell 5.1+ (inclus Windows Server)
5. .NET Framework 4.6+
```


## 🏗️ Configuration Deployment Share

### Création via PowerShell
```powershell
# Importer module MDT
Import-Module "C:\Program Files\Microsoft Deployment Toolkit\bin\MicrosoftDeploymentToolkit.psd1"

# Créer Deployment Share
$DSPath = "C:\DeploymentShare"
$DSName = "MDT Deployment Share"
$DSShare = "DeploymentShare$"

New-Item -Path $DSPath -ItemType Directory -Force
New-SmbShare -Name $DSShare -Path $DSPath -FullAccess "Domain\MDT-Admins" -ReadAccess "Domain\Domain Users"

# Créer PSDrive MDT
New-PSDrive -Name "DS001" -PSProvider MDTProvider -Root $DSPath -Description $DSName

# Structure automatique créée
Get-ChildItem -Path "DS001:" | Select-Object Name, PSIsContainer
```

### Structure Deployment Share
```
C:\DeploymentShare\
├── Applications\              # Applications à déployer
│   ├── Adobe Reader\
│   ├── Microsoft Office\
│   └── Custom Apps\
├── Operating Systems\         # Images OS importées
│   ├── Windows 11 22H2\
│   └── Windows Server 2022\
├── Out-of-Box Drivers\       # Pilotes par modèle
│   ├── Dell OptiPlex\
│   ├── HP ProBook\
│   └── Lenovo ThinkPad\
├── Packages\                 # Updates & Language Packs
├── Task Sequences\           # Séquences de déploiement
├── Selection Profiles\       # Profils de sélection
├── Control\                  # Fichiers de configuration
│   ├── Bootstrap.ini
│   ├── CustomSettings.ini
│   └── Settings.xml
└── Boot\                     # Images générées
    ├── LiteTouchPE_x64.iso
    └── LiteTouchPE_x64.wim
```

## 🔧 Configuration Avancée

### Bootstrap.ini Complet
```ini
[Settings]
Priority=Default

[Default]
; Deployment Share Access
DeployRoot=\\mdtserver\DeploymentShare$
UserID=mdtservice
UserPassword=P@ssw0rd!
UserDomain=domain.local

; Skip Welcome
SkipBDDWelcome=YES

; Network Configuration
_SMSTSHTTPPort=80
_SMSTSHTTPSPort=443

; Wizard Control
SkipTaskSequence=NO
SkipComputerName=NO
SkipDomainMembership=YES
SkipUserData=YES
SkipTimeZone=YES
SkipApplications=YES
SkipBitLocker=YES
SkipSummary=YES
SkipCapture=YES
SkipAdminPassword=NO
SkipProductKey=YES

; Logging
SLShare=\\mdtserver\logs$
EventService=http://mdtserver:9800
```

### CustomSettings.ini Avancé
```ini
[Settings]
Priority=ByLaptop,ByDesktop,ByVM,ByArchitecture,Default
Properties=IsLaptop,IsDesktop,IsVM,Architecture,AssetTag,SerialNumber

[Default]
; ===== GENERAL =====
OSInstall=YES
SkipCapture=YES
SkipAdminPassword=YES
SkipProductKey=YES
SkipComputerBackup=YES
SkipBitLocker=YES
SkipUserData=YES
HideShell=YES

; ===== COMPUTER NAMING =====
OSDComputerName=#Left("%SerialNumber%",8)#
; Alternative: OSDComputerName=#Mid("%SerialNumber%",4,8)#
; ===== DOMAIN JOIN =====
JoinDomain=domain.local
DomainAdmin=mdtservice
DomainAdminDomain=domain.local
DomainAdminPassword=P@ssw0rd!
MachineObjectOU=OU=Workstations,OU=Computers,DC=domain,DC=local

; ===== REGIONAL SETTINGS =====
TimeZoneName=Romance Standard Time
KeyboardLocale=040c:0000040c
UserLocale=fr-FR
UILanguage=fr-FR
SystemLocale=fr-FR

; ===== APPLICATIONS =====
SkipApplications=YES
; Mandatory apps for all deployments
MandatoryApplications001={12345678-1234-1234-1234-123456789012}
MandatoryApplications002={87654321-4321-4321-4321-210987654321}

; ===== DRIVERS =====
DriverSelectionProfile=Nothing

; ===== USER STATE =====
UserDataLocation=NONE
UDShare=\\fileserver\UserState$
UDDir=%OSDComputerName%

; ===== LOGGING =====
SLShare=\\mdtserver\logs$
SLShareDynamicLogging=\\mdtserver\logs$\%OSDComputerName%

; ===== MONITORING =====
EventService=http://mdtserver:9800

; ===== POWERSHELL EXECUTION =====
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

## 📱 Gestion Applications

### Import Applications PowerShell
```powershell
# Import application standard
function Import-MDTApplication {
    param(
        [string]$Name,
        [string]$Version,
        [string]$Publisher,
        [string]$Language = "fr-FR",
        [string]$SourcePath,
        [string]$CommandLine,
        [string]$WorkingDirectory = ".",
        [string]$DestinationFolder
    )
    
    if (!$DestinationFolder) {
        $DestinationFolder = "$Publisher $Name $Version"
    }
    
    $AppPath = "DS001:\Applications"
    
    Import-MDTApplication -Path $AppPath -Name "$Name $Version" `
        -ShortName $Name -Version $Version -Publisher $Publisher `
        -Language $Language -SourcePath $SourcePath `
        -DestinationFolder $DestinationFolder `
        -CommandLine $CommandLine -WorkingDirectory $WorkingDirectory
}

# Exemples d'import
Import-MDTApplication -Name "Adobe Reader DC" -Version "2023.006.20320" -Publisher "Adobe" -SourcePath "\\fileserver\software\Adobe\Reader" -CommandLine "AcroRdrDC2300620320_fr_FR.exe /sAll /rs"

Import-MDTApplication -Name "Google Chrome" -Version "Latest" -Publisher "Google" -SourcePath "\\fileserver\software\Google\Chrome" -CommandLine "googlechromestandaloneenterprise64.msi /quiet"

Import-MDTApplication -Name "Microsoft Office 365" -Version "2023" -Publisher "Microsoft" -SourcePath "\\fileserver\software\Microsoft\Office365" -CommandLine "setup.exe /configure configuration.xml"
```

### Application Bundle
```powershell
# Créer bundle d'applications
$BundlePath = "DS001:\Applications"
$BundleName = "Standard Corporate Apps"

New-Item -Path "$BundlePath\$BundleName" -ItemType Directory

# Définir applications du bundle
$BundleApps = @(
    "Adobe Reader DC 2023.006.20320",
    "Google Chrome Latest",
    "Microsoft Office 365 2023",
    "7-Zip 23.01"
)

# Configuration bundle dans CustomSettings.ini
# Applications001={GUID-Bundle}
```

## 🖥️ Gestion Pilotes

### Organisation Drivers
```powershell
# Structure recommandée Out-of-Box Drivers
function New-MDTDriverStructure {
    $DriverPath = "DS001:\Out-of-Box Drivers"
    
    $Manufacturers = @("Dell", "HP", "Lenovo", "Microsoft")
    $OSVersions = @("Windows 10", "Windows 11")
    $Architectures = @("x64", "x86")
    
    foreach ($Manufacturer in $Manufacturers) {
        foreach ($OS in $OSVersions) {
            foreach ($Arch in $Architectures) {
                $FolderPath = "$DriverPath\$Manufacturer\$OS\$Arch"
                if (!(Get-Item -Path $FolderPath -ErrorAction SilentlyContinue)) {
                    New-Item -Path $FolderPath -ItemType Directory
                    Write-Host "Créé: $FolderPath" -ForegroundColor Green
                }
            }
        }
    }
}

New-MDTDriverStructure
```

### Import Drivers Automatisé
```powershell
# Import drivers avec detection automatique
function Import-MDTDrivers {
    param(
        [string]$SourcePath,
        [string]$Manufacturer,
        [string]$Model,
        [string]$OSVersion = "Windows 11",
        [string]$Architecture = "x64"
    )
    
    $DestPath = "DS001:\Out-of-Box Drivers\$Manufacturer\$Model\$OSVersion\$Architecture"
    
    # Créer dossier si inexistant
    if (!(Get-Item -Path $DestPath -ErrorAction SilentlyContinue)) {
        New-Item -Path $DestPath -ItemType Directory
    }
    
    # Import récursif
    Import-MDTDriver -Path $DestPath -SourcePath $SourcePath
    
    Write-Host "Drivers importés: $DestPath" -ForegroundColor Green
}

# Exemple utilisation
Import-MDTDrivers -SourcePath "\\fileserver\drivers\Dell\OptiPlex7090" -Manufacturer "Dell" -Model "OptiPlex 7090"
```

### Selection Profiles
```powershell
# Créer profils de sélection pour drivers
New-MDTSelectionProfile -Path "DS001:\Selection Profiles" -Name "Dell Drivers Only" -Comments "Drivers Dell uniquement" -Definition "<SelectionProfile><Include path=`"Out-of-Box Drivers\Dell`" /></SelectionProfile>"

New-MDTSelectionProfile -Path "DS001:\Selection Profiles" -Name "HP Drivers Only" -Comments "Drivers HP uniquement" -Definition "<SelectionProfile><Include path=`"Out-of-Box Drivers\HP`" /></SelectionProfile>"
```

## 🔄 Task Sequences Avancées

### Création Task Sequence PowerShell
```powershell
# Créer Task Sequence standard
function New-MDTTaskSequence {
    param(
        [string]$Name,
        [string]$ID,
        [string]$Comments,
        [string]$Template = "Client.xml",
        [string]$OSPath
    )
    
    Import-MDTTaskSequence -Path "DS001:\Task Sequences" -Name $Name -Template $Template -Comments $Comments -ID $ID -OperatingSystemPath $OSPath -FullName "Administrateur MDT" -OrgName "Entreprise" -HomePage "about:blank"
    
    Write-Host "Task Sequence créée: $Name ($ID)" -ForegroundColor Green
}

# Exemples
New-MDTTaskSequence -Name "Deploy Windows 11 Standard" -ID "WIN11STD" -Comments "Déploiement standard Windows 11" -OSPath "DS001:\Operating Systems\Windows 11 22H2"

New-MDTTaskSequence -Name "Deploy Windows 11 Developer" -ID "WIN11DEV" -Comments "Déploiement développeur Windows 11" -OSPath "DS001:\Operating Systems\Windows 11 22H2"
```

### Customisation Task Sequence
```xml
<!-- Exemple ajout d'étapes personnalisées -->
<!-- Dans Deployment Workbench > Task Sequences > Edit -->

<!-- Étape PowerShell personnalisée -->
<step type="SMS_TaskSequence_RunPowerShellScriptAction">
    <name>Configure Corporate Settings</name>
    <description>Application des paramètres corporate</description>
    <action>powershell.exe -ExecutionPolicy Bypass -File "%SCRIPTROOT%\Configure-Corporate.ps1"</action>
    <condition>
        <expression type="SMS_TaskSequence_VariableConditionExpression">
            <variable name="TaskSequenceID" operator="equals" value="WIN11STD"/>
        </expression>
    </condition>
</step>
```

## 📊 Database Integration

### Configuration Database MDT
```powershell
# Configuration base de données (optionnel mais recommandé en prod)
$DatabaseServer = "sqlserver.domain.local"
$DatabaseName = "MDT"
$DatabaseNetLib = "DBNMPNTW"

# Configuration dans Deployment Workbench
# Advanced Configuration > Database > Configure Database
# Permet stockage centralisé des configurations par machine/utilisateur/rôle
```

### Database Schema Utile
```sql
-- Tables principales MDT Database
ComputerIdentity       -- Informations machines
ComputerSettings       -- Settings par machine
LocationIdentity       -- Informations sites
LocationSettings       -- Settings par site
MakeModelIdentity      -- Marques/modèles
MakeModelSettings      -- Settings par modèle
RoleIdentity          -- Rôles organisationnels  
RoleSettings          -- Settings par rôle
```

## 📡 Monitoring & Logging

### Configuration Monitoring
```powershell
# Activer monitoring MDT
Set-ItemProperty -Path "DS001:" -Name "MonitorHost" -Value $env:COMPUTERNAME
Set-ItemProperty -Path "DS001:" -Name "MonitorEventPort" -Value "9800"

# Service monitoring Windows
New-Service -Name "MDT Monitor" -BinaryPathName "C:\Program Files\Microsoft Deployment Toolkit\Monitor\Microsoft.BDD.MonitorService.exe" -StartupType Automatic
Start-Service "MDT Monitor"
```

### PowerShell Monitoring
```powershell
# Script monitoring temps réel
function Get-MDTDeploymentStatus {
    param([string]$MDTServer = $env:COMPUTERNAME)
    
    try {
        $WebRequest = Invoke-WebRequest -Uri "http://$MDTServer:9800/MDTMonitorData/Computers" -UseBasicParsing
        $Data = $WebRequest.Content | ConvertFrom-Json
        
        foreach ($Computer in $Data) {
            [PSCustomObject]@{
                ComputerName = $Computer.Name
                Status = $Computer.StatusType
                CurrentStep = $Computer.CurrentStep
                TotalSteps = $Computer.TotalSteps
                PercentComplete = $Computer.PercentComplete
                Warnings = $Computer.Warnings
                Errors = $Computer.Errors
                StartTime = $Computer.StartTime
                EndTime = $Computer.EndTime
            }
        }
    }
    catch {
        Write-Error "Impossible de récupérer les données de monitoring: $($_.Exception.Message)"
    }
}

# Utilisation
Get-MDTDeploymentStatus | Format-Table -AutoSize
```

### Logs Locations & Analysis
```powershell
# Emplacements logs selon phase
$LogLocations = @{
    "WinPE_Deployment" = "X:\MININT\SMSOSD\OSDLOGS\"
    "Windows_Setup" = "C:\Windows\Panther\"
    "Post_Install" = "C:\MININT\SMSOSD\OSDLOGS\"
    "MDT_Logs" = "\\mdtserver\logs$\\"
}

# Fonction analyse logs
function Analyze-MDTLogs {
    param([string]$LogPath)
    
    $ImportantLogs = @("BDD.log", "Deploy.log", "SMSTS.log", "setupact.log", "setuperr.log")
    
    foreach ($LogFile in $ImportantLogs) {
        $FullPath = Join-Path $LogPath $LogFile
        if (Test-Path $FullPath) {
            Write-Host "=== Analyse $LogFile ===" -ForegroundColor Yellow
            $Errors = Get-Content $FullPath | Select-String -Pattern "error|fail|exception" -CaseSensitive:$false
            if ($Errors) {
                $Errors | ForEach-Object { Write-Host $_.Line -ForegroundColor Red }
            } else {
                Write-Host "Aucune erreur détectée" -ForegroundColor Green
            }
        }
    }
}
```

## 🛠️ Scripts & Automation

### Hook Scripts MDT
```powershell
# Scripts hooks MDT (dans Scripts\)
# Exécutés automatiquement à différentes phases

# ZTIGather.xml - Variables personnalisées
@'
<Properties>
  <Property Name="IsLaptop" Value="" />
  <Property Name="IsDesktop" Value="" />
  <Property Name="IsVM" Value="" />
</Properties>
'@ | Out-File -FilePath "Scripts\ZTIGather.xml" -Encoding UTF8

# Custom.ini - Détection matériel
@'
[Default]
Priority=Custom

[Custom]
IsLaptop=#IsPortable("True")#
IsDesktop=#Not(IsPortable("True"))#
IsVM=#InStr(UCase(Product), "VIRTUAL") > 0#
'@ | Out-File -FilePath "Scripts\Custom.ini" -Encoding UTF8
```

### Post-Installation Scripts
```powershell
# Configure-Corporate.ps1 - Configuration post-déploiement
@'
# Configuration corporate post-déploiement
Write-Host "=== Configuration Corporate ===" -ForegroundColor Yellow

# Désactiver services non nécessaires
$ServicesToDisable = @("Fax", "Windows Search", "Superfetch")
foreach ($Service in $ServicesToDisable) {
    try {
        Set-Service -Name $Service -StartupType Disabled -ErrorAction SilentlyContinue
        Write-Host "Service $Service désactivé" -ForegroundColor Green
    }
    catch {
        Write-Warning "Impossible de désactiver $Service"
    }
}

# Configuration registre corporate
$RegSettings = @{
    "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" = @{
        "EnableLUA" = 1
        "ConsentPromptBehaviorAdmin" = 2
    }
    "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" = @{
        "UseWUServer" = 1
        "AUOptions" = 4
    }
}

foreach ($RegPath in $RegSettings.Keys) {
    if (!(Test-Path $RegPath)) {
        New-Item -Path $RegPath -Force | Out-Null
    }
    foreach ($Setting in $RegSettings[$RegPath].GetEnumerator()) {
        Set-ItemProperty -Path $RegPath -Name $Setting.Key -Value $Setting.Value
        Write-Host "Registre: $RegPath\$($Setting.Key) = $($Setting.Value)" -ForegroundColor Green
    }
}

# Installation certificats corporate
$CertPath = "\\fileserver\certificates\corporate-root.cer"
if (Test-Path $CertPath) {
    Import-Certificate -FilePath $CertPath -CertStoreLocation "Cert:\LocalMachine\Root"
    Write-Host "Certificat corporate installé" -ForegroundColor Green
}

Write-Host "=== Configuration Corporate Terminée ===" -ForegroundColor Green
'@ | Out-File -FilePath "Scripts\Configure-Corporate.ps1" -Encoding UTF8
```

## 🔍 Troubleshooting

### Diagnostics Courants
```powershell
# Script diagnostic MDT complet
function Test-MDTHealth {
    Write-Host "=== Diagnostic MDT ===" -ForegroundColor Yellow
    
    # Vérification services
    $Services = @("MDT Monitor", "WDSServer", "DHCP Server")
    foreach ($Svc in $Services) {
        $Status = Get-Service $Svc -ErrorAction SilentlyContinue
        if ($Status) {
            $Color = if($Status.Status -eq "Running") {"Green"} else {"Red"}
            Write-Host "Service $Svc : $($Status.Status)" -ForegroundColor $Color
        }
    }
    
    # Vérification Deployment Share
    $DSPath = "C:\DeploymentShare"
    if (Test-Path $DSPath) {
        $DSSize = (Get-ChildItem $DSPath -Recurse | Measure-Object -Property Length -Sum).Sum / 1GB
        Write-Host "Deployment Share: $([math]::Round($DSSize, 2)) GB" -ForegroundColor Green
    } else {
        Write-Host "Deployment Share introuvable !" -ForegroundColor Red
    }
    
    # Vérification permissions
    $SharePerms = Get-SmbShare "DeploymentShare$" -ErrorAction SilentlyContinue
    if ($SharePerms) {
        Write-Host "Partage DeploymentShare$ : Actif" -ForegroundColor Green
    } else {
        Write-Host "Partage DeploymentShare$ : Inactif !" -ForegroundColor Red
    }
    
    # Test connectivité monitoring
    try {
        $MonitorTest = Invoke-WebRequest -Uri "http://localhost:9800" -UseBasicParsing -TimeoutSec 5
        Write-Host "Monitoring MDT : Accessible" -ForegroundColor Green
    }
    catch {
        Write-Host "Monitoring MDT : Inaccessible" -ForegroundColor Red
    }
}

Test-MDTHealth
```

### Problèmes Fréquents
| Problème | Cause | Solution |
|----------|-------|----------|
| **Boot loop WinPE** | Bootstrap.ini incorrect | Vérifier credentials/chemin DeployRoot |
| **Drivers not found** | Profil sélection incorrect | Configurer DriverSelectionProfile |
| **Apps install fail** | Commande silencieuse incorrecte | Tester install manual + logs |
| **Domain join fail** | Permissions/DNS | Vérifier compte service + résolution DNS |
| **Slow deployment** | Réseau/disque | Optimiser infrastructure |

## 🚀 Optimisation & Performance

### Tuning Deployment Share
```powershell
# Optimisations performance
function Optimize-MDTDeploymentShare {
    param([string]$DSPath = "C:\DeploymentShare")
    
    # Désactiver indexation
    $Volume = Get-WmiObject -Class Win32_Volume | Where-Object {$_.DriveLetter -eq ($DSPath.Substring(0,2))}
    if ($Volume.IndexingEnabled) {
        $Volume.IndexingEnabled = $false
        $Volume.Put()
        Write-Host "Indexation désactivée sur $($DSPath.Substring(0,2))" -ForegroundColor Green
    }
    
    # Configuration cache WinPE
    $BootPath = Join-Path $DSPath "Boot"
    if (Test-Path $BootPath) {
        # Optimiser taille boot.wim
        & dism /Mount-Wim /WimFile:"$BootPath\LiteTouchPE_x64.wim" /Index:1 /MountDir:"C:\Mount"
        & dism /Image:"C:\Mount" /Cleanup-Image /StartComponentCleanup /ResetBase
        & dism /Unmount-Wim /MountDir:"C:\Mount" /Commit
        Write-Host "Boot.wim optimisé" -ForegroundColor Green
    }
    
    # Compression aggressive pour économiser espace
    & compact /c /s:$DSPath /i /f
    Write-Host "Compression appliquée sur $DSPath" -ForegroundColor Green
}

Optimize-MDTDeploymentShare
```

### Parallel Deployments
```ini
; CustomSettings.ini - Support déploiements parallèles
[Default]
; ...existing configuration...

; Optimisations réseau
Priority=Default
_SMSTSHTTPPort=80
_SMSTSHTTPSPort=443

; Cache local WinPE
USMTOfflineMigration=TRUE
DoCapture=NO
ComputerBackupLocation=NONE

; Réduction timeout
TimeZoneName=Romance Standard Time
WSUSServer=http://wsus.domain.local:8530
```