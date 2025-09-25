# 🖥️ Windows Server Essentials — Aide-mémoire

## 🏗️ Éditions & Licencing

### Éditions Principales
| Édition | Usage | Limitations | Fonctionnalités Clés |
|---------|-------|-------------|---------------------|
| **Standard** | PME, virtualization limitée | 2 VM Hyper-V | Rôles standard, clustering |
| **Datacenter** | Virtualisation intensive | VM illimitées | Storage Spaces Direct, SDN |
| **Essentials** | TPE <25 utilisateurs | 1 serveur, pas de virtualisation | Domaine simplifié, backup |

### Licensing (CAL)
- CALs requises pour accès clients (licences).

## Mode Core
- Server Core = sans GUI → administration via PowerShell / remote tools.
- Installer rôles : `Install-WindowsFeature -Name <NomDuRole> -IncludeManagementTools`.

## Rôles & services courants
- AD (Active Directory) → annuaire central.  
- DNS → résolution noms ↔ IP.  
- DHCP → distribution IP (DORA).  
- WSUS → gestion centralisée des updates.  
- Hyper-V → Hyperviseur Type 1 (Host OS devient Guest OS) 

## Outils MMC utiles
- services.msc, compmgmt.msc, gpmc.msc, dhcpmgmt.msc, dns.msc.

## PowerShell & Core
- PowerShell indispensable en Core.  
- Installer/retirer rôles via `Install-WindowsFeature` / `Remove-WindowsFeature`.
	- Option `-IncludeManagementTools` : Pour les outils de gestion associés.


# 🖥️ Windows Server Essentials — Aide-mémoire

## 🏗️ Éditions & Licencing

### Éditions Principales
| Édition | Usage | Limitations | Fonctionnalités Clés |
|---------|-------|-------------|---------------------|
| **Standard** | PME, virtualization limitée | 2 VM Hyper-V | Rôles standard, clustering |
| **Datacenter** | Virtualisation intensive | VM illimitées | Storage Spaces Direct, SDN |
| **Essentials** | TPE <25 utilisateurs | 1 serveur, pas de virtualisation | Domaine simplifié, backup |

### Licensing (CAL)
```
Types CAL :
├── Device CAL : Par appareil (postes partagés)
├── User CAL : Par utilisateur (multiples appareils)
└── RDS CAL : Accès Terminal Server/RemoteApp

CAL requises pour :
├── Accès fichiers/impression
├── Authentification Active Directory
├── Services Exchange, SharePoint, SQL
└── Sessions RDS (en plus Windows CAL)
```

## 🖥️ Server Core vs Desktop Experience

### Comparaison Interfaces
| Aspect | **Server Core** | **Desktop Experience** |
|--------|-----------------|------------------------|
| **Interface** | CLI PowerShell uniquement | GUI complète |
| **Empreinte** | ~4GB (minimal) | ~12GB+ |
| **Performance** | Optimale (moins overhead) | Standard |
| **Sécurité** | Surface d'attaque réduite | Plus de composants |
| **Maintenance** | Moins updates/reboots | Updates GUI fréquents |
| **Administration** | Remote + PowerShell | Locale + Remote |

```powershell
# Basculer entre modes (Windows Server 2012+)
Get-WindowsFeature *gui*                    # Vérifier état GUI
Install-WindowsFeature Server-Gui-Shell     # Installer GUI
Uninstall-WindowsFeature Server-Gui-Shell -Remove  # Retirer GUI
```

### Outils Administration Server Core
- **sconfig** : Configuration rapide (réseau, domaine, updates)
- **PowerShell ISE** : Via `Install-WindowsFeature PowerShell-ISE`
- **Remote tools** : RSAT depuis poste Windows 10/11
- **Windows Admin Center** : Interface web moderne

## 🔧 Gestion Rôles & Fonctionnalités

### Commandes PowerShell Essentielles
```powershell
# Consultation
Get-WindowsFeature                          # Lister toutes fonctionnalités
Get-WindowsFeature | Where-Object InstallState -eq "Installed"
Get-WindowsFeature *hyper*                  # Recherche pattern

# Installation/Suppression
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
Install-WindowsFeature DNS,DHCP -IncludeManagementTools
Uninstall-WindowsFeature -Name Web-Server -Remove -Restart

# Batch installation depuis fichier
Install-WindowsFeature -ConfigurationFilePath C:\roles.xml
```

### Rôles Windows Server Principaux
| Rôle | Nom PowerShell | Description | Prérequis |
|------|----------------|-------------|-----------|
| **Active Directory** | `AD-Domain-Services` | Annuaire entreprise | DNS recommandé |
| **DNS Server** | `DNS` | Résolution noms | - |
| **DHCP Server** | `DHCP` | Attribution IP automatique | - |
| **File Services** | `FS-FileServer` | Partage fichiers | - |
| **Hyper-V** | `Hyper-V` | Virtualisation | Hardware VT/AMD-V |
| **IIS** | `IIS-WebServer` | Serveur web | - |
| **Remote Desktop** | `RDS-RD-Server` | Terminal Server | RDS CAL |
| **WSUS** | `UpdateServices` | Gestion mises à jour | SQL/WID |

## 🔐 Active Directory Domain Services

### Architecture AD
```
Forest (Forêt)
├── Domain (Domaine) - Unité administrative
│   ├── OU (Organizational Unit) - Organisation
│   ├── Users - Comptes utilisateurs
│   ├── Computers - Comptes ordinateurs
│   └── Groups - Groupes sécurité/distribution
├── Sites - Localisation physique
└── Domain Controllers - Serveurs AD
```

### Installation & Configuration
```powershell
# Installer rôle AD DS
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promouvoir en Domain Controller (nouvelle forêt)
Install-ADDSForest -DomainName "domain.local" -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force)

# Ajouter DC à domaine existant
Install-ADDSDomainController -DomainName "domain.local" -Credential (Get-Credential)

# Vérifications post-installation
Get-ADDomain                                # Info domaine
Get-ADForest                               # Info forêt
Get-ADDomainController                     # Liste DC
```

### Objets AD Courants
```powershell
# Utilisateurs
New-ADUser -Name "John Doe" -SamAccountName "jdoe" -UserPrincipalName "jdoe@domain.local" -Path "OU=Users,DC=domain,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) -Enabled $true

# Groupes
New-ADGroup -Name "IT-Team" -GroupScope Global -GroupCategory Security -Path "OU=Groups,DC=domain,DC=local"
Add-ADGroupMember -Identity "IT-Team" -Members "jdoe"

# Ordinateurs
New-ADComputer -Name "PC-001" -Path "OU=Computers,DC=domain,DC=local"
```

## 🌐 Services Réseau Essentiels

### DNS Server
```powershell
# Gestion zones DNS
Add-DnsServerPrimaryZone -Name "domain.local" -ZoneFile "domain.local.dns"
Add-DnsServerResourceRecordA -ZoneName "domain.local" -Name "server1" -IPv4Address "192.168.1.10"
Add-DnsServerResourceRecordCName -ZoneName "domain.local" -Name "web" -HostNameAlias "server1.domain.local"

# Forwarders (résolution externe)
Add-DnsServerForwarder -IPAddress 8.8.8.8,1.1.1.1
Set-DnsServerForwarder -IPAddress 8.8.8.8,1.1.1.1

# Vérifications
Get-DnsServerZone                          # Lister zones
Resolve-DnsName server1.domain.local       # Test résolution
```

### DHCP Server
```powershell
# Configuration DHCP
Add-DhcpServerv4Scope -Name "LAN" -StartRange 192.168.1.100 -EndRange 192.168.1.200 -SubnetMask 255.255.255.0 -State Active
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -Router 192.168.1.1 -DnsServer 192.168.1.10 -DnsDomain "domain.local"

# Réservations
Add-DhcpServerv4Reservation -ScopeId 192.168.1.0 -IPAddress 192.168.1.50 -ClientId "00-15-5D-00-00-01" -Name "Server1"

# Autorisation DHCP dans AD
Add-DhcpServerInDC -DnsName "dhcp.domain.local" -IPAddress 192.168.1.10
```

## 💻 Hyper-V Virtualisation

### Configuration Hyper-V
```powershell
# Installation
Install-WindowsFeature Hyper-V -IncludeManagementTools -Restart

# Commutateurs virtuels
New-VMSwitch -Name "External" -NetAdapterName "Ethernet" -AllowManagementOS $true
New-VMSwitch -Name "Internal" -SwitchType Internal
New-VMSwitch -Name "Private" -SwitchType Private

# Gestion VMs
New-VM -Name "VM-TEST" -MemoryStartupBytes 2GB -NewVHDPath "C:\VMs\VM-TEST.vhdx" -NewVHDSizeBytes 60GB -SwitchName "External"
Start-VM -Name "VM-TEST"
Stop-VM -Name "VM-TEST"
Get-VM | Select-Object Name, State, Status
```

### Types Commutateurs Virtuels
| Type | Connectivité | Usage |
|------|--------------|-------|
| **External** | VM ↔ Réseau physique | Production, accès réseau |
| **Internal** | VM ↔ Host uniquement | Test, lab isolé avec host |
| **Private** | VM ↔ VM uniquement | Test, sécurité maximale |

## 🔄 Windows Server Update Services (WSUS)

### Installation & Configuration
```powershell
# Installation WSUS
Install-WindowsFeature UpdateServices -IncludeManagementTools
& "$env:ProgramFiles\Update Services\Tools\wsusutil.exe" postinstall CONTENT_DIR=C:\WSUS

# Configuration via PowerShell
Get-WsusServer                             # Info serveur WSUS
Get-WsusProduct | Where-Object Title -like "*Windows 10*" | Set-WsusProduct
Invoke-WsusServerCleanup -CleanupObsoleteUpdates -CleanupUnneededContentFiles
```

### Gestion Clients WSUS
```
Group Policy Configuration:
Computer Configuration > Administrative Templates > Windows Components > Windows Update:
├── Configure Automatic Updates: Enabled
├── Intranet Microsoft update service location: http://wsus.domain.local:8530
├── Automatic Updates detection frequency: 4 hours
└── No auto-restart with logged on users: Enabled
```

## 🛠️ Outils Administration (MMC)

### Consoles Essentielles
| Outil | Commande | Description |
|-------|----------|-------------|
| **Services** | `services.msc` | Gestion services Windows |
| **Event Viewer** | `eventvwr.msc` | Journaux événements |
| **Computer Management** | `compmgmt.msc` | Gestion système complète |
| **Group Policy Management** | `gpmc.msc` | GPO Active Directory |
| **Active Directory Users** | `dsa.msc` | Gestion objets AD |
| **DNS Manager** | `dnsmgmt.msc` | Gestion zones DNS |
| **DHCP Manager** | `dhcpmgmt.msc` | Configuration DHCP |
| **Hyper-V Manager** | `virtmgmt.msc` | Gestion machines virtuelles |

### Server Manager
```
Fonctionnalités Server Manager:
├── Dashboard: Vue d'ensemble serveurs
├── Local Server: Configuration serveur local
├── All Servers: Gestion multi-serveurs
├── File and Storage Services: Partages et stockage
└── Tools: Accès rapide outils MMC
```

## 📋 Group Policy Basics

### Structure GPO
```
Group Policy Objects:
├── Computer Configuration (s'applique aux ordinateurs)
│   ├── Software Settings
│   ├── Windows Settings (sécurité, scripts)
│   └── Administrative Templates (registry)
└── User Configuration (s'applique aux utilisateurs)
    ├── Software Settings  
    ├── Windows Settings (dossiers, sécurité)
    └── Administrative Templates (registry)
```

### Commandes GPO Utiles
```cmd
# Forcer application GPO
gpupdate /force                    # Appliquer GPO immédiatement
gpresult /r                       # Résultat GPO appliquées
gpresult /h report.html           # Rapport détaillé HTML
```

## 🔍 Monitoring & Maintenance

### Performance Counters Clés
```
Compteurs importants:
├── Processor(_Total)\% Processor Time < 80%
├── Memory\Available MBytes > 10% RAM totale
├── LogicalDisk(_Total)\% Disk Time < 50%
├── LogicalDisk(_Total)\Avg. Disk Queue Length < 2
└── Network Interface(*)\Bytes Total/sec (selon besoin)
```

### Event Logs Critiques
| Log | Événements Clés | Description |
|-----|-----------------|-------------|
| **System** | 6005,6006,6008 | Démarrage/arrêt système |
| **Application** | 1000,1001 | Erreurs applications |
| **Security** | 4624,4625 | Connexions réussies/échec |
| **Directory Service** | 1311,1644 | Erreurs réplication AD |
| **DNS Server** | 4000-4015 | Problèmes résolution DNS |

### Commandes Diagnostic
```powershell
# État général système
Get-ComputerInfo | Select-Object WindowsProductName, TotalPhysicalMemory, CsProcessors
Get-EventLog -LogName System -Newest 10 -EntryType Error
Get-Service | Where-Object Status -eq "Stopped" | Where-Object StartType -eq "Automatic"

# Tests réseau
Test-NetConnection -ComputerName "dc.domain.local" -Port 389  # LDAP
Test-NetConnection -ComputerName "8.8.8.8" -Port 53          # DNS externe
```

## 💡 Best Practices

### Sécurité
- ✅ **Comptes service** dédiés (pas administrateur)
- ✅ **Groupes sécurité** plutôt qu'utilisateurs individuels
- ✅ **Passwords policy** complexe via GPO
- ✅ **Audit** événements sécurité critiques
- ✅ **Updates** automatiques via WSUS

### Performance
- ✅ **Server Core** pour serveurs sans GUI
- ✅ **Rôles séparés** sur serveurs dédiés
- ✅ **Monitoring** proactif ressources
- ✅ **Antivirus** exclusions dossiers AD/Exchange
- ✅ **Fragmentation** périodique bases AD

### Haute Disponibilité
- ✅ **Domain Controllers** multiples (min 2)
- ✅ **DNS** redondant (primaire/secondaire)
- ✅ **DHCP** failover ou split-scope
- ✅ **Backup** régulier (System State + données)
- ✅ **Documentation** architecture et procédures

### Maintenance
- ✅ **Patches** mensuels coordonnés
- ✅ **Logs** rotation et archivage
- ✅ **Performances** baseline et trending
- ✅ **Disaster Recovery** plan testé
- ✅ **Formation** équipe administrative

---
**💡 Memo** : Server Core = performance optimale • AD DS = cœur infrastructure • CAL requises pour accès • RSAT pour admin à distance !