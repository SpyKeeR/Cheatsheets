# 🖥️ Remote Desktop Services (RDS) — Cheatsheet

## 🏗️ Architecture & Composants

### Rôles RDS Essentiels
| Rôle | Fonction | Requis | Recommandations |
|------|----------|--------|-----------------|
| **RD Session Host** | Héberge sessions/apps | ✅ | Multi-serveurs pour HA |
| **RD Connection Broker** | Load balancing & reconnexion | ✅ | Redondant en prod |
| **RD Web Access** | Portail web RemoteApp | ✅ | Certificat SSL requis |
| **RD Licensing** | Gestion CAL | ✅ | Un par forêt AD |
| **RD Gateway** | Accès HTTPS externe | ⚠️ | Pour accès Internet |

### Modes de Déploiement
```
🔹 Quick Deployment (Test/Dev)
   └── Tous les rôles sur 1 serveur

🔹 Standard Deployment (Production)
   ├── RD Session Host (2+ serveurs)
   ├── RD Connection Broker (HA)
   ├── RD Web Access (Load balanced)
   ├── RD Gateway (DMZ)
   └── RD Licensing (Dédié)
```

### Types d'Accès
- **Session-based Desktop** : Bureau complet partagé
- **RemoteApp** : Applications publiées individuellement
- **VDI (Virtual Desktop)** : VM dédiée par utilisateur
- **Personal Desktop** : Session dédiée persistante

## 📦 Installation & Déploiement

### Déploiement Standard PowerShell
```powershell
# Installation rôles RDS
Install-WindowsFeature RDS-RD-Server, RDS-Connection-Broker, RDS-Web-Access, RDS-Licensing, RDS-Gateway -IncludeManagementTools

# Créer déploiement RDS
New-RDSessionDeployment -ConnectionBroker "broker.domain.com" -WebAccessServer "web.domain.com" -SessionHost "host01.domain.com","host02.domain.com"

# Ajouter serveur de licences
Add-RDServer -Server "license.domain.com" -Role RDS-LICENSING -ConnectionBroker "broker.domain.com"

# Configurer mode de licence
Set-RDLicenseConfiguration -LicenseServer "license.domain.com" -Mode PerUser -ConnectionBroker "broker.domain.com"
```

### Configuration High Availability
```powershell
# Broker redondant (nécessite SQL)
Add-RDServer -Server "broker02.domain.com" -Role RDS-CONNECTION-BROKER -ConnectionBroker "broker.domain.com"

# Base de données centralisée
Set-RDDatabaseConnectionString -DatabaseConnectionString "Server=sql.domain.com;Database=RDS;Integrated Security=true" -ConnectionBroker "broker.domain.com"

# DNS Round Robin pour Web Access
# Créer enregistrement DNS pour web.domain.com pointant vers multiples IPs
```

## 🔧 Configuration Collections

### Création Collection Session
```powershell
# Nouvelle collection
New-RDSessionCollection -CollectionName "Office Apps" -SessionHost "host01.domain.com","host02.domain.com" -ConnectionBroker "broker.domain.com"

# Configuration collection
Set-RDSessionCollectionConfiguration -CollectionName "Office Apps" -MaxRedirectedMonitors 2 -AuthenticateUsingNLA $true -ConnectionBroker "broker.domain.com"

# Propriétés sessions
Set-RDSessionCollectionConfiguration -CollectionName "Office Apps" -DisconnectedSessionLimitMin 60 -IdleSessionLimitMin 120 -ConnectionBroker "broker.domain.com"
```

### Configuration Load Balancing
```powershell
# Poids relatifs serveurs
Set-RDSessionHost -SessionHost "host01.domain.com" -NewConnectionAllowed Yes -MaxSessions 50 -ConnectionBroker "broker.domain.com"
Set-RDSessionHost -SessionHost "host02.domain.com" -NewConnectionAllowed Yes -MaxSessions 75 -ConnectionBroker "broker.domain.com"

# Mode drain (maintenance)
Set-RDSessionHost -SessionHost "host01.domain.com" -NewConnectionAllowed NotUntilReboot -ConnectionBroker "broker.domain.com"
```

## 📱 RemoteApp Configuration

### Publication Applications
```powershell
# Lister apps disponibles
Get-RDAvailableApp -ConnectionBroker "broker.domain.com" -CollectionName "Office Apps"

# Publier application
New-RDRemoteApp -ConnectionBroker "broker.domain.com" -CollectionName "Office Apps" -DisplayName "Microsoft Word" -FilePath "C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE"

# Application personnalisée
New-RDRemoteApp -ConnectionBroker "broker.domain.com" -CollectionName "Office Apps" -DisplayName "Custom App" -FilePath "C:\Apps\MyApp.exe" -FileVirtualPath "C:\Program Files\MyApp\MyApp.exe" -VPath "C:\Apps\MyApp.exe" -FolderName "Custom Apps"
```

### Configuration Avancée RemoteApp
```powershell
# Paramètres application
Set-RDRemoteApp -ConnectionBroker "broker.domain.com" -CollectionName "Office Apps" -DisplayName "Microsoft Word" -CommandLineSetting Require -RequiredCommandLine "/safe"

# Associations fichiers
Set-RDRemoteApp -ConnectionBroker "broker.domain.com" -CollectionName "Office Apps" -DisplayName "Microsoft Word" -FileTypeAssociation @(".docx",".doc")

# Icône personnalisée
Set-RDRemoteApp -ConnectionBroker "broker.domain.com" -CollectionName "Office Apps" -DisplayName "Custom App" -IconPath "C:\Icons\app.ico"
```

## 🔒 Sécurité & Authentification

### RD Gateway Configuration
```powershell
# Installation certificat
Import-PfxCertificate -FilePath "C:\Certs\rdgateway.pfx" -CertStoreLocation "Cert:\LocalMachine\My" -Password (ConvertTo-SecureString "password" -AsPlainText -Force)

# Configuration Gateway
$cert = Get-ChildItem -Path "Cert:\LocalMachine\My" | Where-Object {$_.Subject -like "*rdgateway*"}
Set-RDDeploymentGatewayConfiguration -GatewayMode Custom -GatewayExternalFqdn "rdgateway.domain.com" -LogonMethod Password -UseCachedCredentials $true -BypassLocal $true -ConnectionBroker "broker.domain.com"
```

### SSL/TLS Configuration
```powershell
# Certificat RD Web Access
Set-RDCertificate -Role RDWebAccess -ImportPath "C:\Certs\rdweb.pfx" -Password (ConvertTo-SecureString "password" -AsPlainText -Force) -ConnectionBroker "broker.domain.com"

# Certificat RD Gateway
Set-RDCertificate -Role RDGateway -ImportPath "C:\Certs\rdgateway.pfx" -Password (ConvertTo-SecureString "password" -AsPlainText -Force) -ConnectionBroker "broker.domain.com"

# Certificat RemoteApp Signing
Set-RDCertificate -Role RDRedirector -ImportPath "C:\Certs\rdapp.pfx" -Password (ConvertTo-SecureString "password" -AsPlainText -Force) -ConnectionBroker "broker.domain.com"
```

### Authentification Multi-Factor
```powershell
# NPS avec RADIUS
Install-WindowsFeature NPAS-Policy-Server -IncludeManagementTools

# Configuration RD Gateway pour NPS
# Interface graphique : RD Gateway Manager > Policies > Connection Authorization Policies
```

### Group Policy Configuration
```
Configuration ordinateur\Stratégies\Modèles d'administration\Composants Windows\Services Bureau à distance\

Paramètres recommandés :
├── Hôte de session Bureau à distance
│   ├── Limite de temps des sessions déconnectées : 1 heure
│   ├── Limite de temps des sessions actives : 8 heures
│   ├── Limite de temps des sessions inactives : 2 heures
│   └── Redirection des lecteurs : Désactivée (sécurité)
├── Client Connexion Bureau à distance
│   ├── Authentification au niveau du réseau : Activée
│   └── Autoriser la reconnexion automatique : Activée
└── Gestionnaire de licences Bureau à distance
    └── Serveur de licences : license.domain.com
```

## 👥 Gestion Profils Utilisateurs

### FSLogix Profile Containers
```powershell
# Installation FSLogix
# Télécharger depuis Microsoft et installer FSLogixAppsSetup.exe

# Configuration via GPO
$regPath = "HKLM:\SOFTWARE\FSLogix\Profiles"
New-ItemProperty -Path $regPath -Name "Enabled" -Value 1 -PropertyType DWORD -Force
New-ItemProperty -Path $regPath -Name "VHDLocations" -Value "\\fileserver\profiles$" -PropertyType String -Force
New-ItemProperty -Path $regPath -Name "SizeInMBs" -Value 30720 -PropertyType DWORD -Force  # 30GB
New-ItemProperty -Path $regPath -Name "IsDynamic" -Value 1 -PropertyType DWORD -Force
```

### User Profile Disks (Alternative)
```powershell
# Configuration UPD
Set-RDSessionCollectionConfiguration -CollectionName "Office Apps" -EnableUserProfileDisk -MaxUserProfileDiskSizeGB 20 -DiskPath "\\fileserver\upd$" -ConnectionBroker "broker.domain.com"

# Exclusions UPD
Set-RDSessionCollectionConfiguration -CollectionName "Office Apps" -ExcludeFolderPath @("Downloads","AppData\Local\Temp") -ConnectionBroker "broker.domain.com"
```

## 📊 Monitoring & Performance

### PowerShell Monitoring
```powershell
# Sessions actives
Get-RDUserSession -ConnectionBroker "broker.domain.com" | Select-Object UserName, SessionState, IdleTime, HostServer

# Performances serveurs
Get-RDSessionHost -ConnectionBroker "broker.domain.com" | ForEach-Object {
    $sessions = Get-RDUserSession -ConnectionBroker "broker.domain.com" -HostServer $_.SessionHost
    [PSCustomObject]@{
        Server = $_.SessionHost
        ActiveSessions = ($sessions | Where-Object {$_.SessionState -eq "Active"}).Count
        DisconnectedSessions = ($sessions | Where-Object {$_.SessionState -eq "Disconnected"}).Count
        MaxSessions = $_.MaxSessions
        NewConnectionAllowed = $_.NewConnectionAllowed
    }
}

# État collections
Get-RDSessionCollection -ConnectionBroker "broker.domain.com" | Select-Object CollectionName, @{Name="SessionHosts";Expression={$_.SessionHost -join ","}}
```

### Performance Counters Clés
```
Surveillance recommandée :
├── \Terminal Services Sessions\Active Sessions
├── \Terminal Services Sessions\Inactive Sessions
├── \RAS Total\Total Connections
├── \Processor(_Total)\% Processor Time
├── \Memory\Available MBytes
├── \PhysicalDisk(_Total)\Avg. Disk Queue Length
└── \Network Interface(*)\Bytes Total/sec
```

### Event Logs Importants
| Log | Event IDs | Description |
|-----|-----------|-------------|
| **Microsoft-Windows-TerminalServices-LocalSessionManager/Operational** | 21,23,24,25 | Connexions/déconnexions |
| **Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational** | 1149 | Échecs auth |
| **System** | 1074,6005,6006 | Redémarrages système |
| **Application** | 1000,1001 | Erreurs applications |

## 🎮 GPU Virtualization

### RemoteFX vGPU (Legacy)
```powershell
# Vérification support
Get-WmiObject -Class Win32_VideoController | Select-Object Name, AdapterCompatibility

# Configuration via Hyper-V Manager pour VDI
# Note: RemoteFX déprécié dans Windows Server 2019+
```

### GPU Passthrough Moderne
```powershell
# NVIDIA GRID/Tesla configuration
# Configuration nécessaire au niveau hyperviseur (VMware vSphere, Hyper-V Gen2)

# Vérification support DDA (Windows Server 2016+)
$gpu = Get-PnpDevice | Where-Object {$_.Class -eq "Display"}
Dismount-VMHostAssignableDevice -LocationPath $gpu.LocationPath -Force
```

## 🔧 Optimisation Performances

### Session Host Tuning
```powershell
# Désactiver services non critiques
$services = @("Fax", "Windows Search", "Superfetch", "Themes")
foreach ($service in $services) {
    Set-Service -Name $service -StartupType Disabled -ErrorAction SilentlyContinue
    Stop-Service -Name $service -Force -ErrorAction SilentlyContinue
}

# Configuration mémoire virtuelle
$pagefile = Get-WmiObject -Class Win32_ComputerSystem -EnableAllPrivileges
$pagefile.AutomaticManagedPagefile = $false
$pagefile.Put()

# Désactiver indexation sur volumes systèmes
Get-WmiObject -Class Win32_Volume | Where-Object {$_.IndexingEnabled -eq $true -and $_.DriveLetter -ne $null} | ForEach-Object {
    $_.IndexingEnabled = $false
    $_.Put()
}
```

### Registry Optimizations
```powershell
# Performance RDP
$rdpPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp"
Set-ItemProperty -Path $rdpPath -Name "fEnableWinStation" -Value 1
Set-ItemProperty -Path $rdpPath -Name "MaxInstanceCount" -Value 0xffffffff
Set-ItemProperty -Path $rdpPath -Name "MaxConnectionTime" -Value 0

# Optimisations réseau
$tcpPath = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
Set-ItemProperty -Path $tcpPath -Name "TcpWindowSize" -Value 65536
Set-ItemProperty -Path $tcpPath -Name "TcpAckFrequency" -Value 1
Set-ItemProperty -Path $tcpPath -Name "TCPNoDelay" -Value 1
```

## 🛠️ Troubleshooting

### Diagnostics Sessions
```powershell
# Sessions bloquées
Get-RDUserSession -ConnectionBroker "broker.domain.com" | Where-Object {$_.SessionState -eq "Disconnected" -and $_.IdleTime -gt "02:00:00"}

# Forcer déconnexion
Disconnect-RDUser -HostServer "host01.domain.com" -UserName "domain\user" -Force

# Reset session host
Reset-RDSessionHost -SessionHost "host01.domain.com" -ConnectionBroker "broker.domain.com"
```

### Diagnostics Réseau
```powershell
# Test connectivité RD Gateway
Test-NetConnection -ComputerName "rdgateway.domain.com" -Port 443

# Test RDP direct
Test-NetConnection -ComputerName "host01.domain.com" -Port 3389

# Vérification certificats
Get-ChildItem -Path "Cert:\LocalMachine\My" | Where-Object {$_.Subject -like "*rd*"} | Select-Object Subject, NotAfter, Thumbprint
```

### Commandes Utiles
```cmd
:: Services RDS
net start TermService
net start SessionEnv
net start UmRdpService

:: Informations sessions
qwinsta
quser
rwinsta <session_id>

:: Logs détaillés RDP
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v "LoggingEnabled" /t REG_DWORD /d 1
```

## 📋 Licensing & Compliance

### Types de Licences
| Type CAL | Description | Usage |
|----------|-------------|-------|
| **RDS User CAL** | Par utilisateur nommé | Utilisateurs fixes |
| **RDS Device CAL** | Par appareil | Appareils partagés |
| **Windows Server CAL** | Licence OS de base | Requis en plus RDS CAL |

### Configuration Licensing
```powershell
# Vérification licences
Get-RDLicenseConfiguration -ConnectionBroker "broker.domain.com"

# Installation licences
Install-RDLicense -LicenseServer "license.domain.com" -LicenseCode "XXXXX-XXXXX-XXXXX-XXXXX-XXXXX"

# Rapport utilisation
Get-RDLicenseConfiguration -ConnectionBroker "broker.domain.com" | Format-List
```

## 🚀 Automatisation & Scripts

### Script Déploiement Complet
```powershell
# deploy-rds.ps1
param(
    [Parameter(Mandatory=$true)]
    [string]$ConnectionBroker,
    
    [Parameter(Mandatory=$true)]
    [string[]]$SessionHosts,
    
    [string]$WebAccessServer = $ConnectionBroker,
    [string]$LicenseServer = $ConnectionBroker
)

# Installation des rôles
$servers = @($ConnectionBroker) + $SessionHosts + @($WebAccessServer, $LicenseServer) | Select-Object -Unique

foreach ($server in $servers) {
    Invoke-Command -ComputerName $server -ScriptBlock {
        Install-WindowsFeature RDS-RD-Server, RDS-Connection-Broker, RDS-Web-Access, RDS-Licensing -IncludeManagementTools
    }
}

# Création du déploiement
New-RDSessionDeployment -ConnectionBroker $ConnectionBroker -WebAccessServer $WebAccessServer -SessionHost $SessionHosts

# Configuration licensing
Add-RDServer -Server $LicenseServer -Role RDS-LICENSING -ConnectionBroker $ConnectionBroker
Set-RDLicenseConfiguration -LicenseServer $LicenseServer -Mode PerUser -ConnectionBroker $ConnectionBroker

Write-Host "Déploiement RDS terminé !" -ForegroundColor Green
```

### Maintenance Automatisée
```powershell
# maintenance-rds.ps1
# Nettoyage sessions déconnectées > 4h
Get-RDUserSession -ConnectionBroker "broker.domain.com" | 
    Where-Object {$_.SessionState -eq "Disconnected" -and $_.IdleTime -gt "04:00:00"} |
    ForEach-Object {
        Disconnect-RDUser -HostServer $_.HostServer -UserName $_.UserName -Force
        Write-Host "Déconnexion forcée: $($_.UserName) sur $($_.HostServer)"
    }

# Vérification santé serveurs
Get-RDSessionHost -ConnectionBroker "broker.domain.com" | ForEach-Object {
    $health = Test-NetConnection -ComputerName $_.SessionHost -Port 3389 -InformationLevel Quiet
    if (-not $health) {
        Send-MailMessage -To "admin@domain.com" -Subject "RDS Alert" -Body "Serveur $($_.SessionHost) inaccessible"
    }
}
```