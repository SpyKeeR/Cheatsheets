# 🌐 DHCP Server — Aide-mémoire

## 🔧 Concepts Fondamentaux

### Protocole DHCP
- **Port UDP 67** : Serveur DHCP
- **Port UDP 68** : Client DHCP
- **Broadcast** : Communication initiale (255.255.255.255)
- **Unicast** : Renouvellements après attribution

### Processus DORA
```
Client                    Serveur
  │                         │
  ├─── DISCOVER ──────────→ │  (Broadcast UDP:67)
  │                         │
  │ ←────── OFFER ────────── │  (Unicast UDP:68)
  │                         │
  ├─── REQUEST ──────────→  │  (Broadcast UDP:67)
  │                         │
  │ ←── ACKNOWLEDGE ─────── │  (Unicast UDP:68)
  │                         │
```

#### Détails des Messages
| Phase | Direction | Description | Contenu |
|-------|-----------|-------------|---------|
| **Discover** | Client → Broadcast | Recherche serveur DHCP | MAC client, options demandées |
| **Offer** | Serveur → Client | Proposition IP | IP proposée, durée bail, options |
| **Request** | Client → Broadcast | Acceptation offre | IP demandée, serveur choisi |
| **Acknowledge** | Serveur → Client | Confirmation attribution | Configuration complète |

## ⚙️ Configuration Windows Server

### Autorisation Active Directory
```cmd
# Autoriser serveur DHCP dans AD (requis domaine)
netsh dhcp add server DHCPServer.domain.com 192.168.1.10
dhcpcmd /cmd EnumServers  # Lister serveurs autorisés

# Via PowerShell
Add-DhcpServerInDC -DnsName "DHCP01.domain.com" -IPAddress 192.168.1.10
Get-DhcpServerInDC        # Vérifier autorisation
```

### Étendues (Scopes)
```powershell
# Créer étendue
Add-DhcpServerv4Scope -Name "LAN Principale" -StartRange 192.168.1.100 -EndRange 192.168.1.200 -SubnetMask 255.255.255.0 -LeaseDuration 8:00:00

# Configurer exclusions
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.1 -EndRange 192.168.1.50

# Activer étendue
Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -State Active

# Lister étendues
Get-DhcpServerv4Scope
```

### Options DHCP Standard
| Option | Nom | Valeur Exemple | Description |
|--------|-----|----------------|-------------|
| **3** | Router | 192.168.1.1 | Passerelle par défaut |
| **6** | DNS Servers | 192.168.1.10,8.8.8.8 | Serveurs DNS |
| **15** | DNS Domain Name | domain.local | Domaine DNS |
| **44** | WINS/NetBIOS Name Servers | 192.168.1.10 | Serveur WINS (legacy) |
| **46** | WINS/NetBT Node Type | 0x8 (H-node) | Type résolution NetBIOS |
| **51** | Lease Time | 691200 (8 jours) | Durée bail en secondes |

### Options Avancées
| Option | Usage | Configuration |
|--------|-------|---------------|
| **66** | Boot Server Host Name | PXE/TFTP server | |
| **67** | Bootfile Name | Fichier boot PXE | |
| **119** | Domain Search | Suffixes DNS multiples | Encodage spécial requis |
| **121** | Classless Static Routes | Routes spécifiques | Format binaire |
| **252** | Web Proxy Auto-Discovery | WPAD URL | http://wpad/wpad.dat |

#### Configuration Options PowerShell
```powershell
# Options globales serveur
Set-DhcpServerv4OptionValue -OptionId 6 -Value "192.168.1.10","8.8.8.8"
Set-DhcpServerv4OptionValue -OptionId 15 -Value "domain.local"

# Options spécifiques étendue
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -OptionId 3 -Value "192.168.1.1"

# Option 119 (Domain Search) - Exemple
Set-DhcpServerv4OptionValue -OptionId 119 -Value ([byte[]](7,100,111,109,97,105,110,49,5,108,111,99,97,108,0))
```

## 🔄 Gestion des Baux

### Cycle de Vie du Bail
```
Attribution → T1 (50%) → T2 (87.5%) → Expiration
     │          │           │           │
  8 jours    4 jours     7 jours    Fin bail
     │          │           │           │
 Utilisation Renouveau  Renouveau   Libération
            Unicast    Broadcast
```

### Commandes Client
```cmd
# Windows
ipconfig /release              # Libérer bail
ipconfig /renew               # Renouveler bail
ipconfig /displaydns          # Cache DNS
ipconfig /all                 # Configuration complète

# Linux
sudo dhclient -r              # Libérer bail
sudo dhclient eth0           # Renouveler bail
sudo dhclient -v eth0        # Mode verbeux
```

### Gestion Serveur
```powershell
# Réservations (IP fixes)
Add-DhcpServerv4Reservation -ScopeId 192.168.1.0 -IPAddress 192.168.1.50 -ClientId "00-11-22-33-44-55" -Name "Serveur-Print"

# Consulter baux actifs
Get-DhcpServerv4Lease -ScopeId 192.168.1.0

# Supprimer bail
Remove-DhcpServerv4Lease -IPAddress 192.168.1.150 -ScopeId 192.168.1.0

# Statistiques étendues
Get-DhcpServerv4ScopeStatistics
```

## 🏗️ Haute Disponibilité

### DHCP Failover (Windows 2012+)
```powershell
# Configuration Load Balance (50/50)
Add-DhcpServerv4Failover -ComputerName "DHCP01" -PartnerServer "DHCP02" -Name "Failover-LAN" -ScopeId 192.168.1.0 -LoadBalancePercent 50 -SharedSecret "SecretKey123"

# Configuration Hot Standby (Actif/Passif)
Add-DhcpServerv4Failover -ComputerName "DHCP01" -PartnerServer "DHCP02" -Name "Failover-Standby" -ScopeId 192.168.1.0 -ServerRole Active -ReservePercent 10 -SharedSecret "SecretKey123"

# Vérifier état failover
Get-DhcpServerv4Failover
```

### Split-Scope (Legacy)
```
Serveur DHCP1: 192.168.1.100 - 192.168.1.150 (80%)
Serveur DHCP2: 192.168.1.151 - 192.168.1.200 (20%)

Avantages:
- Simple à configurer
- Compatible anciennes versions

Inconvénients:
- Gestion manuelle
- Pas de synchronisation automatique
```

## 🌉 DHCP Relay Agent

### Concepts Multi-VLAN
```
VLAN 10 (192.168.10.0/24) ←→ Router/L3 Switch ←→ VLAN 20 (Serveurs)
        │                          │                    │
   Clients DHCP              DHCP Relay Agent      Serveur DHCP
                                   │
                            ip helper-address
```

### Configuration Cisco
```cisco
# Sur interface VLAN clients
interface vlan 10
 ip helper-address 192.168.20.10    # IP serveur DHCP
 ip helper-address 192.168.20.11    # Serveur DHCP secondaire

# Ports forwarded par helper-address
# UDP 53 (DNS), 67 (DHCP), 68 (DHCP), 69 (TFTP), 137 (NetBIOS)
```

### Configuration Windows Server
```
Roles > Remote Access > Routing
├── IPv4 > General > New Routing Protocol
├── Ajouter DHCP Relay Agent
└── Propriétés > Ajouter serveurs DHCP
```

## 📊 Surveillance & Logs

### Journalisation Windows
```
Emplacements logs:
├── Event Viewer > Applications and Services Logs > Microsoft-Windows-Dhcp-Server
├── %SystemRoot%\System32\Dhcp\DhcpSrvLog-*.log
└── Performance Monitor > DHCP Server counters

Événements importants:
├── Event ID 1000-1999: Démarrage/arrêt service
├── Event ID 11004: Attribution bail
├── Event ID 11005: Renouvellement bail
├── Event ID 11013: Adresse IP en conflit
└── Event ID 11031: Base de données corrompue
```

### Monitoring PowerShell
```powershell
# Statistiques étendues
Get-DhcpServerv4ScopeStatistics | Format-Table ScopeId, InUse, Available, Percentage

# Baux par état
Get-DhcpServerv4Lease | Group-Object AddressState

# Détection conflits
Get-DhcpServerv4Lease | Where-Object {$_.AddressState -eq "BadAddress"}

# Utilisation par étendue
Get-DhcpServerv4Scope | ForEach-Object {
    $stats = Get-DhcpServerv4ScopeStatistics -ScopeId $_.ScopeId
    [PSCustomObject]@{
        Scope = $_.Name
        Used = $stats.InUse
        Available = $stats.Available  
        Percentage = $stats.PercentageInUse
    }
}
```

## 🔧 Sauvegarde & Maintenance

### Sauvegarde DHCP
```cmd
# Export configuration complète
netsh dhcp server export "C:\Backup\dhcp-config.txt" all

# Import configuration
netsh dhcp server import "C:\Backup\dhcp-config.txt" all

# Via PowerShell
Export-DhcpServer -ComputerName "DHCP01" -File "C:\Backup\dhcp-backup.xml"
Import-DhcpServer -ComputerName "DHCP02" -File "C:\Backup\dhcp-backup.xml"
```

### Base de Données DHCP
```
Emplacement: %SystemRoot%\System32\Dhcp\
├── dhcp.mdb (base principale)
├── dhcp.pat (fichier patch)
├── J50.log, J51.log (logs transactions)
└── tmp.edb (temporaire)

Maintenance automatique:
- Nettoyage baux expirés: par défaut chaque nuit
- Compactage base: automatique si nécessaire
- Sauvegarde synchrone: toutes les heures
```

### Maintenance Manuelle
```powershell
# Réconcilier étendues (après problème)
Invoke-DhcpServerv4DnsRegistration -ScopeId 192.168.1.0

# Nettoyer baux expirés
Remove-DhcpServerv4Lease -ScopeId 192.168.1.0 -BadPings 2

# Compacter base de données
Stop-Service DHCPServer
esentutl /d "%SystemRoot%\System32\Dhcp\dhcp.mdb"
Start-Service DHCPServer
```

## 🎯 Cas d'Usage Spécialisés

### PXE Boot Configuration
```powershell
# Options PXE standard
Set-DhcpServerv4OptionValue -OptionId 66 -Value "pxe.domain.com"    # Boot server
Set-DhcpServerv4OptionValue -OptionId 67 -Value "pxelinux.0"        # Boot filename

# Classe vendeur pour différencier architectures
Add-DhcpServerv4Class -Name "PXEClient (BIOS)" -Type Vendor -Data "PXEClient:Arch:00000:UNDI:002001"
Add-DhcpServerv4Class -Name "PXEClient (UEFI)" -Type Vendor -Data "PXEClient:Arch:00007:UNDI:003016"
```

### VoIP Configuration
```powershell
# Option 120 pour téléphones SIP
Set-DhcpServerv4OptionValue -OptionId 120 -Value ([byte[]](192,168,1,100))

# VLAN Voice via option 132 (Cisco)
Set-DhcpServerv4OptionValue -OptionId 132 -Value ([byte[]](0,0,0,100))  # VLAN 100
```

### IPv6 DHCP (DHCPv6)
```powershell
# Configuration étendue IPv6
Add-DhcpServerv6Scope -Name "IPv6 LAN" -Prefix 2001:db8:1::/64

# Options DHCPv6 courantes
Set-DhcpServerv6OptionValue -OptionId 23 -Value "2001:db8:1::1"      # DNS Server
Set-DhcpServerv6OptionValue -OptionId 24 -Value "domain.local"       # Domain Name
```

## 🚨 Troubleshooting

### Problèmes Courants
| Symptôme | Cause Probable | Diagnostic | Solution |
|----------|---------------|------------|----------|
| **Pas d'IP attribuée** | Service arrêté | `Get-Service DHCPServer` | Démarrer service |
| **IP 169.254.x.x (APIPA)** | Pas de serveur DHCP | `ipconfig /all` | Vérifier connectivité |
| **Conflit d'adresses** | IP statique dans plage | Event Viewer | Ajuster étendue/exclusions |
| **Renouvellement échoue** | Serveur surchargé | Logs DHCP | Augmenter plage/durée bail |

### Commandes Diagnostics
```cmd
# Tests connectivité
ping 192.168.1.10              # Test serveur DHCP
telnet 192.168.1.10 67         # Test port DHCP (si activé)

# Traces réseau
netsh trace start capture=yes provider=Microsoft-Windows-Dhcp-Server
# Reproduire problème
netsh trace stop

# Event Viewer ciblé
wevtutil qe Microsoft-Windows-Dhcp-Server/Operational /c:50 /rd:true /f:text
```

### Outils Tiers
- **DHCP Explorer** : Monitoring graphique
- **SolarWinds IPAM** : Gestion IP avancée  
- **Wireshark** : Analyse paquets DHCP
- **Advanced IP Scanner** : Scan réseau

## 💡 Bonnes Pratiques

### Sécurité
- ✅ **Autorisation AD** obligatoire en domaine
- ✅ **Segmentation VLAN** par type utilisateur
- ✅ **Réservations** pour serveurs critiques
- ✅ **Monitoring** détection d'intrusion (serveurs DHCP non autorisés)
- ✅ **Audit** attributions IP régulier

### Performance  
- ✅ **Durée bail** adaptée à la mobilité (8h bureaux, 2h Wi-Fi guest)
- ✅ **Plages dimensionnées** : 30% marge sur pic utilisateurs
- ✅ **Exclusions** pour IP statiques/management
- ✅ **Failover** ou split-scope pour redondance
- ✅ **Maintenance** base données mensuelle

### Opérationnel
- ✅ **Documentation** plan adressage complet
- ✅ **Sauvegarde** configuration avant modifications
- ✅ **Nommage** cohérent étendues et réservations
- ✅ **Monitoring** utilisation étendues (<90%)
- ✅ **Tests** après modifications (lab isolé)

---
**💡 Memo** : DORA (Discover-Offer-Request-Acknowledge), options 3/6/15 essentielles, failover > split-scope !