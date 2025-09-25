# 🖥️ Remote Desktop Services (RDS) — Aide-mémoire

## 🏗️ Architecture & Concepts

### Composants RDS Essentiels
```
┌─────────────────────────────────────────────────────────────┐
│                    Internet/WAN                             │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌─────────────────────▼──────────────────────────────────────┐
│              RD Gateway (DMZ)                              │
│  • HTTPS/SSL tunnel (port 443)                             │
│  • Authentification & autorisation                         │
└─────────────────────┬──────────────────────────────────────┘
                      │
┌─────────────────────▼─────────────────────────────────────┐
│            RD Connection Broker                           │
│  • Load balancing sessions                                │
│  • Reconnexion automatique                                │
│  • Base de données centralisée                            │
└─────────┬───────────────────────────────────┬─────────────┘
          │                                   │
┌─────────▼──────────┐               ┌───────▼─────────────┐
│  RD Web Access     │               │  RD Licensing       │
│  • Portail web     │               │  • CAL management   │
│  • RemoteApp feed  │               │  • Per User/Device  │
└────────────────────┘               └─────────────────────┘
                      │
        ┌─────────────▼─────────────────────────┐
        │         RD Session Host               │
        │  • Sessions utilisateurs              │
        │  • Applications publiées              │
        │  • Ressources partagées               │
        └───────────────────────────────────────┘
```

### Types d'Accès RDS
| Type | Description | Cas d'Usage | Ressources |
|------|-------------|-------------|------------|
| **Session-based Desktop** | Bureau complet partagé | Utilisateurs standard | CPU/RAM partagés |
| **RemoteApp** | Applications individuelles | Usage spécialisé | Optimisé par app |
| **VDI (Virtual Desktop)** | VM dédiée par utilisateur | Haute performance | Ressources dédiées |
| **Personal Desktop** | Session persistante | Personnalisation requise | État préservé |

## 📦 Installation & Configuration

### Rôles Windows Server
```powershell
# Installation complète (serveur unique)
Install-WindowsFeature RDS-RD-Server, RDS-Connection-Broker, RDS-Web-Access, RDS-Licensing, RDS-Gateway -IncludeManagementTools

# Déploiement distribué (production)
New-RDSessionDeployment -ConnectionBroker "broker.domain.com" -WebAccessServer "web.domain.com" -SessionHost "host01.domain.com","host02.domain.com"

# Ajout serveur licences
Add-RDServer -Server "license.domain.com" -Role RDS-LICENSING -ConnectionBroker "broker.domain.com"
```

### Configuration Haute Disponibilité
```powershell
# Connection Broker redondant (nécessite SQL Server)
Set-RDDatabaseConnectionString -DatabaseConnectionString "Server=sql.domain.com;Database=RDS;Integrated Security=true" -ConnectionBroker "broker.domain.com"
Add-RDServer -Server "broker02.domain.com" -Role RDS-CONNECTION-BROKER -ConnectionBroker "broker.domain.com"

# Load balancing Web Access (DNS Round Robin)
# Créer multiple A records pour web.domain.com
```

## 🔧 Gestion Collections & Sessions

### Configuration Collection
```powershell
# Créer collection
New-RDSessionCollection -CollectionName "OfficeApps" -SessionHost "host01.domain.com","host02.domain.com" -ConnectionBroker "broker.domain.com"

# Paramètres sessions
Set-RDSessionCollectionConfiguration -CollectionName "OfficeApps" `
    -DisconnectedSessionLimitMin 60 `
    -IdleSessionLimitMin 120 `
    -MaxRedirectedMonitors 2 `
    -AuthenticateUsingNLA $true `
    -ConnectionBroker "broker.domain.com"

# Load balancing
Set-RDSessionHost -SessionHost "host01.domain.com" -NewConnectionAllowed Yes -MaxSessions 50 -ConnectionBroker "broker.domain.com"
```

### RemoteApp Publication
```powershell
# Publier application
New-RDRemoteApp -ConnectionBroker "broker.domain.com" -CollectionName "OfficeApps" -DisplayName "Microsoft Word" -FilePath "C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE"

# Application avec paramètres
New-RDRemoteApp -ConnectionBroker "broker.domain.com" -CollectionName "OfficeApps" -DisplayName "Excel Safe Mode" -FilePath "C:\Program Files\Microsoft Office\root\Office16\EXCEL.EXE" -CommandLineSetting Require -RequiredCommandLine "/safe"

# Association fichiers
Set-RDRemoteApp -ConnectionBroker "broker.domain.com" -CollectionName "OfficeApps" -DisplayName "Microsoft Word" -FileTypeAssociation @(".docx",".doc")
```

## 🔒 Sécurité & Certificats

### Configuration SSL/TLS
```powershell
# Import certificats
Set-RDCertificate -Role RDWebAccess -ImportPath "C:\Certs\rdweb.pfx" -Password (ConvertTo-SecureString "password" -AsPlainText -Force) -ConnectionBroker "broker.domain.com"
Set-RDCertificate -Role RDGateway -ImportPath "C:\Certs\rdgateway.pfx" -Password (ConvertTo-SecureString "password" -AsPlainText -Force) -ConnectionBroker "broker.domain.com"
Set-RDCertificate -Role RDRedirector -ImportPath "C:\Certs\rdapp.pfx" -Password (ConvertTo-SecureString "password" -AsPlainText -Force) -ConnectionBroker "broker.domain.com"
```

### RD Gateway Configuration
```powershell
# Configuration Gateway
Set-RDDeploymentGatewayConfiguration -GatewayMode Custom -GatewayExternalFqdn "rdgateway.domain.com" -LogonMethod Password -UseCachedCredentials $true -BypassLocal $true -ConnectionBroker "broker.domain.com"

# Politique d'accès (via GUI RD Gateway Manager)
# Connection Authorization Policies : Qui peut se connecter
# Resource Authorization Policies : À quoi ils peuvent accéder
```

### Authentification & NLA
- **Network Level Authentication (NLA)** : Authentification avant établissement session RDP
- **Avantages** : Sécurité renforcée, protection contre attaques DoS
- **Prérequis** : Windows Vista/Server 2008+ côté client

## 👥 Gestion Profils Utilisateurs

### FSLogix Profile Containers (Recommandé)
```
Avantages FSLogix vs UPD :
├── Performance améliorée (montage VHD)
├── Support Office 365 optimisé
├── Compatibilité multiplateforme
├── Fonctionnalités avancées (Cloud Cache)
└── Supporté nativement par Microsoft

Configuration Registry :
├── HKLM\SOFTWARE\FSLogix\Profiles
│   ├── Enabled = 1
│   ├── VHDLocations = \\fileserver\profiles$
│   ├── SizeInMBs = 30720 (30GB)
│   └── IsDynamic = 1
```

### User Profile Disks (Legacy)
```powershell
# Configuration UPD (alternative FSLogix)
Set-RDSessionCollectionConfiguration -CollectionName "OfficeApps" -EnableUserProfileDisk -MaxUserProfileDiskSizeGB 20 -DiskPath "\\fileserver\upd$" -ConnectionBroker "broker.domain.com"

# Exclusions recommandées
Set-RDSessionCollectionConfiguration -CollectionName "OfficeApps" -ExcludeFolderPath @("Downloads","AppData\Local\Temp","AppData\Local\Microsoft\Windows\Temporary Internet Files") -ConnectionBroker "broker.domain.com"
```

## 📊 Licensing & Compliance

### Types de Licences
| CAL Type | Modèle | Usage Optimal | Exemple |
|----------|--------|---------------|---------|
| **RDS User CAL** | Par utilisateur nommé | Utilisateurs multiples appareils | Employé avec PC + mobile |
| **RDS Device CAL** | Par appareil | Postes partagés | Terminal point de vente |
| **Windows Server CAL** | Base OS | Toujours requis | Accès serveur Windows |
| **Office 365** | Subscription | Apps Microsoft | Word, Excel via RemoteApp |

### Configuration Licensing
```powershell
# Mode licensing
Set-RDLicenseConfiguration -LicenseServer "license.domain.com" -Mode PerUser -ConnectionBroker "broker.domain.com"  # ou PerDevice

# Installation licences
Install-RDLicense -LicenseServer "license.domain.com" -LicenseCode "XXXXX-XXXXX-XXXXX-XXXXX-XXXXX"

# Vérification licences
Get-RDLicenseConfiguration -ConnectionBroker "broker.domain.com"
```

### Période de Grâce
- **120 jours** sans serveur de licences configuré
- **Connexion refusée** après expiration
- **Monitoring** essentiel pour éviter interruptions

## 📈 Monitoring & Performance

### Métriques Clés
```powershell
# Sessions actives
Get-RDUserSession -ConnectionBroker "broker.domain.com" | Group-Object SessionState | Select-Object Name, Count

# Charge serveurs
Get-RDSessionHost -ConnectionBroker "broker.domain.com" | ForEach-Object {
    $sessions = Get-RDUserSession -ConnectionBroker "broker.domain.com" -HostServer $_.SessionHost
    [PSCustomObject]@{
        Server = $_.SessionHost
        ActiveSessions = ($sessions | Where-Object {$_.SessionState -eq "Active"}).Count
        MaxSessions = $_.MaxSessions
        UtilizationPct = [math]::Round((($sessions | Where-Object {$_.SessionState -eq "Active"}).Count / $_.MaxSessions) * 100, 1)
    }
}
```

### Performance Counters Essentiels
| Compteur | Seuil Recommandé | Description |
|----------|------------------|-------------|
| **\Terminal Services Sessions\Active Sessions** | < 80% MaxSessions | Utilisation sessions |
| **\Processor(_Total)\% Processor Time** | < 80% | Charge CPU |
| **\Memory\Available MBytes** | > 1GB | Mémoire disponible |
| **\LogicalDisk(C:)\% Free Space** | > 15% | Espace disque |
| **\Network Interface(*)\Bytes Total/sec** | Variable | Bande passante |

### Event Logs Critiques
```
Event Viewer Locations :
├── Microsoft-Windows-TerminalServices-LocalSessionManager/Operational
│   ├── Event ID 21 : Session logon succeeded
│   ├── Event ID 23 : Session logoff succeeded  
│   └── Event ID 24 : Session disconnected
├── Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational
│   └── Event ID 1149 : Authentication failed
└── System
    ├── Event ID 1074 : System shutdown
    └── Event ID 6005/6006 : Event log start/stop
```

## 🎮 GPU & Multimédia

### GPU Virtualization
| Technology | Support | Usage | Limitations |
|------------|---------|-------|-------------|
| **RemoteFX vGPU** | Legacy (déprécié 2019+) | VDI graphique | Windows Server 2016 max |
| **DDA (Discrete Device Assignment)** | Server 2016+ | GPU passthrough | 1 GPU = 1 VM |
| **GPU Partitioning** | Server 2022+ | Partage GPU | Support fabricant requis |
| **Software Rendering** | Toujours | Applications basiques | Performance limitée |

### Configuration Audio/Vidéo
```powershell
# Redirection audio
Set-RDSessionCollectionConfiguration -CollectionName "OfficeApps" -AudioRedirectionMode Enabled -ConnectionBroker "broker.domain.com"

# Qualité vidéo (GPO recommandé)
# Computer Configuration > Policies > Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Session Host > Remote Session Environment
```

## ⚙️ Optimisation & Performance

### Session Host Tuning
```powershell
# Services non critiques à désactiver
$nonEssentialServices = @("Fax", "Windows Search", "Superfetch", "Themes", "TabletInputService")
foreach ($service in $nonEssentialServices) {
    Set-Service -Name $service -StartupType Disabled -ErrorAction SilentlyContinue
}

# Optimisations registry
$rdpOptimizations = @{
    "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" = @{
        "MaxInstanceCount" = [UInt32]0xffffffff
        "fEnableWinStation" = [UInt32]1
    }
}
```

### Group Policy Optimizations
```
Configuration recommandée :
├── Hôte de session Bureau à distance
│   ├── Limites temps sessions déconnectées : 1 heure
│   ├── Limites temps sessions actives : 8 heures  
│   ├── Limites temps sessions inactives : 2 heures
│   └── Redirection lecteurs : Désactivée (sécurité)
├── Client Connexion Bureau à distance
│   ├── Authentification niveau réseau : Activée
│   ├── Qualité expérience : Optimisée pour WAN
│   └── Reconnexion automatique : Activée
└── Sécurité
    ├── Chiffrement RDP : Niveau élevé
    ├── Authentification serveur : Requise
    └── NLA : Obligatoire
```

## 🛠️ Troubleshooting

### Diagnostics Sessions
```powershell
# Sessions bloquées/déconnectées
Get-RDUserSession -ConnectionBroker "broker.domain.com" | Where-Object {$_.SessionState -eq "Disconnected" -and $_.IdleTime -gt "02:00:00"}

# Forcer déconnexion
Disconnect-RDUser -HostServer "host01.domain.com" -UserName "domain\user" -Force

# Reset session host (maintenance)
Set-RDSessionHost -SessionHost "host01.domain.com" -NewConnectionAllowed NotUntilReboot -ConnectionBroker "broker.domain.com"
```

### Problèmes Courants
| Symptôme | Cause Probable | Diagnostic | Solution |
|----------|----------------|------------|----------|
| **Connexion lente** | Bande passante/latence | Test réseau, GPO qualité | Optimiser paramètres RDP |
| **Apps ne se lancent pas** | Profil utilisateur corrompu | Event Viewer, FSLogix logs | Recréer profil utilisateur |
| **Déconnexions fréquentes** | Timeout/réseau instable | Logs RDP, ping tests | Ajuster timeout, stabiliser réseau |
| **"Trop de connexions"** | Limite sessions atteinte | Vérifier MaxSessions | Augmenter limite ou ajouter serveurs |
| **Erreur licensing** | CAL insuffisantes/expirées | RD Licensing Manager | Acheter/installer CAL |

### Outils Diagnostic
```cmd
# Commandes système utiles
qwinsta                          # Lister sessions locales
quser                           # Utilisateurs connectés
rwinsta <session_id>            # Fermer session
query session /server:hostname  # Sessions serveur distant

# Tests connectivité
Test-NetConnection -ComputerName "rdgateway.domain.com" -Port 443
Test-NetConnection -ComputerName "host01.domain.com" -Port 3389

# Logs RDP détaillés (debugging)
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v "LoggingEnabled" /t REG_DWORD /d 1
```

## 💡 Bonnes Pratiques

### Architecture
- ✅ **Séparer les rôles** : Connection Broker ≠ Session Host
- ✅ **Redondance** : Minimum 2 Session Hosts, HA pour Broker
- ✅ **Localisation** : Session Hosts près des utilisateurs
- ✅ **Sizing** : 2-4 vCPU et 4-8GB RAM par serveur + utilisateurs
- ✅ **Storage** : SSD pour OS et profils, HDD pour données

### Sécurité
- ✅ **NLA obligatoire** : Authentification avant session
- ✅ **Certificats SSL** : Tous les rôles avec certificats valides
- ✅ **RD Gateway** : Accès externe sécurisé uniquement
- ✅ **Groupe policies** : Restrictions appropriées
- ✅ **Monitoring** : Surveillance continue des connexions

### Performance
- ✅ **FSLogix** : Préférer aux UPD pour profils
- ✅ **Optimization GPO** : Tuning spécifique RDS
- ✅ **Resource allocation** : Pas de surallocation critique
- ✅ **Application optimization** : Apps compatibles Terminal Server
- ✅ **Network tuning** : QoS pour trafic RDP

### Maintenance
- ✅ **Patching** : Windows Updates coordonnés
- ✅ **Backup** : Configuration + profils utilisateurs  
- ✅ **Documentation** : Architecture et procédures
- ✅ **Tests DR** : Plan de continuité validé
- ✅ **Monitoring** : Alertes proactives sur métriques clés

---
**💡 Memo** : NLA + certificats SSL + FSLogix + monitoring = RDS production ready !