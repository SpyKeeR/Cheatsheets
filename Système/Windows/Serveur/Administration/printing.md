# 🖨️ Serveur d'Impression Windows — Aide-mémoire

## 🏗️ Architecture d'Impression

### Composants Principaux
```
Client → Spooler → Imprimante → Port → Périphérique
  ↓        ↓          ↓         ↓         ↓
App     Service    Objet     Interface  Matériel
       Windows    logique   transport   physique
```

| Composant | Rôle | Localisation |
|-----------|------|--------------|
| **Périphérique d'impression** | Matériel physique | Réseau/USB/parallèle |
| **Imprimante** | Objet logique Windows | Serveur d'impression |
| **Port d'imprimante** | Interface communication | IP, USB, LPT, FILE |
| **Pilote d'imprimante** | Translation données | Serveur + clients |
| **File d'attente** | Jobs en attente | Spooler Windows |

### Service Print Spooler
- **Service** : `Print Spooler` (spoolsv.exe)
- **Dossier spool** : `%SystemRoot%\System32\spool\PRINTERS`
- **Formats jobs** : RAW, EMF, XPS
- **Processus** : Application → GDI → Spooler → Pilote → Port

## 🛠️ Configuration Serveur d'Impression

### Installation Rôle
```powershell
# Via PowerShell
Install-WindowsFeature Print-Services -IncludeManagementTools
Install-WindowsFeature Print-Server, Print-Internet, Print-LPD-Service

# Composants disponibles
Print-Server                  # Serveur d'impression de base
Print-Internet               # Impression via Internet (IPP)
Print-LPD-Service           # Line Printer Daemon
Print-Scan-Server           # Serveur de numérisation
```

### Ajout Imprimante Réseau
```powershell
# Via PowerShell
Add-Printer -Name "HP-Bureau" -DriverName "HP LaserJet PCL 6" -PortName "IP_192.168.1.100"
Add-PrinterPort -Name "IP_192.168.1.100" -PrinterHostAddress "192.168.1.100" -Protocol RAW -PortNumber 9100

# Via interface
# Panneau de configuration > Périphériques et imprimantes > Ajouter une imprimante
# → Imprimante réseau → TCP/IP → Adresse IP
```

### Pilotes d'Imprimante
```powershell
# Gestion pilotes
Get-PrinterDriver                     # Lister pilotes installés
Add-PrinterDriver -Name "HP Universal Printing PCL 6"
Remove-PrinterDriver -Name "OldDriver"

# Pilotes additionnels (architectures)
# Interface graphique : Propriétés imprimante > Partage > Pilotes supplémentaires
# Installer x86 ET x64 pour compatibilité clients
```

## 🔐 Permissions & Sécurité

### Niveaux de Permissions
| Permission | Description | Capacités |
|------------|-------------|-----------|
| **Imprimer** | Utilisation basique | Envoyer jobs, voir sa file |
| **Gérer les documents** | Gestion files | Annuler/suspendre tous jobs |
| **Gérer cette imprimante** | Administration complète | Config, permissions, partage |

### Attribution Permissions
```powershell
# Via PowerShell
$printer = Get-Printer -Name "HP-Bureau"
Set-PrinterPermission -PrinterName "HP-Bureau" -UserName "Domain\PrintUsers" -Permission Print
Set-PrinterPermission -PrinterName "HP-Bureau" -UserName "Domain\PrintAdmins" -Permission ManagePrinter

# Groupes recommandés
# Domain\Print Users → Permission "Imprimer"
# Domain\Print Operators → Permission "Gérer les documents"  
# Domain\Print Admins → Permission "Gérer cette imprimante"
```

### Audit d'Impression
```
Configuration audit :
├── GPO > Stratégies locales > Audit des événements d'ouverture de session d'objet
├── Propriétés imprimante > Sécurité > Avancé > Audit
└── Event Viewer > Windows Logs > Security

Événements critiques :
├── ID 4624 : Connexion réussie
├── ID 4625 : Échec connexion
└── ID 307 : Job d'impression (si audit activé)
```

## 📤 Déploiement Client

### Méthodes de Déploiement
| Méthode | Avantages | Inconvénients | Usage |
|---------|-----------|---------------|--------|
| **Manuel** | Simple, direct | Non automatisé | Tests, cas ponctuels |
| **Script logon** | Automatisé, flexible | Maintenance scripts | Moyennes structures |
| **GPO** | Centralisé, contrôlé | Complexité GPO | Grandes structures |
| **Print Management** | Interface graphique | Windows seulement | Environnements homogènes |

### Connexion Manuelle
```cmd
# Ligne de commande
rundll32 printui.dll,PrintUIEntry /in /n "\\PrintServer\HP-Bureau"
net use LPT1: \\PrintServer\HP-Bureau

# Interface utilisateur
\\PrintServer → Double-clic imprimante → Installation automatique
```

### Déploiement GPO
```
Group Policy Management :
├── Computer Configuration > Policies > Windows Settings > Deployed Printers
├── User Configuration > Policies > Windows Settings > Deployed Printers
└── Configuration :
    ├── Action : Deploy (computer) / Replace (user)
    ├── Path : \\PrintServer\PrinterName
    └── Default : Oui/Non (imprimante par défaut)
```

### Script PowerShell Déploiement
```powershell
# Exemple script logon
$PrintServer = "\\PrintServer"
$Printers = @("HP-Bureau", "Canon-Couleur", "Zebra-Etiquettes")

foreach ($Printer in $Printers) {
    $PrinterPath = "$PrintServer\$Printer"
    if (!(Get-Printer -Name $Printer -ErrorAction SilentlyContinue)) {
        Add-Printer -ConnectionName $PrinterPath
        Write-Host "Imprimante $Printer ajoutée"
    }
}

# Définir imprimante par défaut
$DefaultPrinter = Get-WmiObject -Class Win32_Printer | Where-Object {$_.Name -like "*HP-Bureau*"}
if ($DefaultPrinter) {
    $DefaultPrinter.SetDefaultPrinter()
}
```

## 🌐 Impression Internet (IPP)

### Configuration IPP
```
IIS Manager > Sites > Default Web Site :
├── Ajouter répertoire virtuel "printers" → %SystemRoot%\System32\spool\drivers\w32x86\3
├── Authentication : Windows Authentication
└── Permissions : Authenticated Users (Read)

URL d'accès : http://printserver/printers/
Configuration client : Ajouter imprimante > URL Internet
```

### Avantages IPP
- ✅ **Accès web** : Interface browser pour gestion
- ✅ **Traversée firewall** : Port 80/443 standard  
- ✅ **Authentification** : Intégration AD
- ✅ **SSL** : Chiffrement communications

## 📊 Monitoring & Maintenance

### Outils de Surveillance
```powershell
# État files d'attente
Get-PrintJob -PrinterName "HP-Bureau"
Get-PrintJob | Where-Object {$_.JobStatus -eq "Error"}

# Statistiques impression
Get-Counter "\Print Queue(*)\Jobs"
Get-Counter "\Print Queue(*)\Total Jobs Printed"

# Espace disque spool
Get-ChildItem "$env:SystemRoot\System32\spool\PRINTERS" | Measure-Object -Property Length -Sum
```

### Event Logs
| Log | Événement | Description |
|-----|-----------|-------------|
| **System** | 10 | Service Print Spooler démarré |
| **System** | 37 | Pilote installé |
| **System** | 372 | Imprimante supprimée |
| **Microsoft-Windows-PrintService/Operational** | 307 | Job imprimé |
| **Microsoft-Windows-PrintService/Admin** | 800 | Erreur spooler |

### Maintenance Préventive
```powershell
# Nettoyage files spool
Stop-Service Spooler
Remove-Item "$env:SystemRoot\System32\spool\PRINTERS\*" -Force
Start-Service Spooler

# Vérification intégrité pilotes
pnputil /enum-drivers | findstr "Print"
Get-PrinterDriver | Where-Object {$_.MajorVersion -lt 3}  # Pilotes obsolètes
```

## 🔧 Résolution Problèmes

### Problèmes Courants
| Symptôme | Cause Probable | Diagnostic | Solution |
|----------|----------------|------------|----------|
| **Jobs bloqués** | Spooler/pilote | Event Viewer, file spool | Restart spooler, purge |
| **Installation échoue** | Permissions/pilote | Droits admin, pilote compatible | Élever privilèges |
| **Impression lente** | Réseau/pilote | Ping, format job | Optimiser réseau/pilote |
| **Pas d'impression** | Connectivité/service | Telnet port 9100, services | Vérifier réseau/services |

### Commandes Diagnostic
```cmd
# Test connectivité imprimante
ping 192.168.1.100
telnet 192.168.1.100 9100     # Port RAW standard
nslookup printer.domain.com    # Résolution DNS

# État services
sc query spooler
net start spooler
sc qc spooler                  # Configuration service

# Informations système
wmic printer list brief
wmic printer get name,portname,drivername
printmanagement.msc           # Console graphique
```

### Reset Complet Spooler
```powershell
# Procédure reset complet
Stop-Service Spooler -Force
Stop-Service "Fax" -ErrorAction SilentlyContinue

# Nettoyer registre spooler
Remove-Item "HKLM:\SYSTEM\CurrentControlSet\Control\Print\Printers\*" -Recurse -Force
Remove-Item "HKLM:\SYSTEM\CurrentControlSet\Control\Print\Monitors\*" -Recurse -Force -ErrorAction SilentlyContinue

# Vider dossiers spool
Remove-Item "$env:SystemRoot\System32\spool\PRINTERS\*" -Force
Remove-Item "$env:SystemRoot\System32\spool\drivers\*" -Recurse -Force -ErrorAction SilentlyContinue

Start-Service Spooler
```

## 🎯 Cas d'Usage Spécialisés

### Impression Follow-Me
- **Principe** : Job retenu jusqu'à authentification physique
- **Solutions** : PaperCut, FollowMe, SafeQ
- **Avantages** : Sécurité, réduction gaspillage
- **Intégration** : Badge RFID, PIN code

### Impression Mobile
```
Protocoles supportés :
├── AirPrint (iOS) : Bonjour + IPP
├── Google Cloud Print (obsolète)
├── Mopria (Android) : IPP over USB/Wi-Fi
└── Windows Mobile : WSD (Web Services for Devices)

Configuration :
├── Activer IPP sur serveur impression
├── Certificat SSL recommandé
└── VLAN invités avec routage contrôlé
```

### Impression Sécurisée
```
Mesures de sécurité :
├── Chiffrement IPP over HTTPS
├── Authentification utilisateur (Pull Printing)
├── Watermarking automatique
├── Audit complet (qui, quoi, quand)
├── Quota par utilisateur/département
└── DLP (Data Loss Prevention) integration
```

## 💡 Bonnes Pratiques

### Architecture
- ✅ **Serveurs dédiés** : Séparer impression des autres rôles
- ✅ **Redondance** : Cluster ou Load Balancing
- ✅ **Segmentation** : VLAN dédié imprimantes
- ✅ **Nommage** : Convention claire (LOC-TYPE-MODEL-NNN)

### Sécurité
- ✅ **Groupes AD** : Permissions par groupes, pas utilisateurs
- ✅ **Audit** : Traçabilité impressions sensibles
- ✅ **SSL/TLS** : Chiffrer communications IPP
- ✅ **Segmentation réseau** : Isoler imprimantes
- ✅ **Firmware** : Maintenir imprimantes à jour

### Gestion
- ✅ **Standardisation** : Modèles/pilotes limités
- ✅ **Documentation** : Localisation, contacts, spécifications
- ✅ **Monitoring** : Alertes proactives (papier, toner)
- ✅ **Backup** : Exporter configuration serveur impression
- ✅ **Tests** : Valider déploiements avant production

### Performance
- ✅ **Pilotes universels** : Réduire variété pilotes
- ✅ **Impression directe** : Éviter serveur pour gros volumes
- ✅ **Compression** : Optimiser transferts réseau
- ✅ **Planning** : Gros jobs hors heures ouvrées
- ✅ **Cache pilotes** : Distribution locale via GPO

---
**💡 Memo** : Imprimante = objet logique, périphérique = matériel • Permissions par groupes AD • GPO pour déploiement large !