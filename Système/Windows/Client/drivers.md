# 🖥️ Pilotes Windows & RDP — Aide-mémoire

## 🔧 Architecture Pilotes Windows

### Types de Pilotes
| Type | Description | Emplacement | Usage |
|------|-------------|-------------|-------|
| **Kernel Mode** | Accès direct hardware | `\System32\drivers\` | Périphériques critiques |
| **User Mode** | API Windows | `\System32\` | Applications, services |
| **WDM** | Windows Driver Model | Géré par PnP | Périphériques USB, PCI |
| **Legacy** | Anciens pilotes | Manuel | Hardware legacy |

### Structure Fichiers Pilote
```
Package Pilote:
├── .inf     # Description installation (texte)
├── .sys     # Binaire pilote kernel
├── .cat     # Signature numérique (certificat)
├── .dll     # Bibliothèques user-mode (optionnel)
└── .exe     # Setup personnalisé (optionnel)
```

#### Fichier INF (Information)
```ini
[Version]
Signature="$WINDOWS NT$"
Class=Display
ClassGuid={...}
Provider=Manufacturer
DriverVer=01/01/2024,1.0.0.0

[Manufacturer]
%ManufacturerName%=Standard,NTamd64

[SourceDisksNames]
1 = %DiskName%,,,""

[SourceDisksFiles]
driver.sys = 1,,

[DestinationDirs]
DefaultDestDir = 12  # System32\drivers
```

## 📁 Emplacements Système

### Répertoires Pilotes
```
Pilotes actifs:
├── C:\Windows\System32\drivers\        # Pilotes kernel (.sys)
├── C:\Windows\System32\                # DLL user-mode
└── C:\Windows\SysWOW64\               # DLL 32-bit sur x64

Packages pilotes:
├── C:\Windows\INF\                     # Fichiers .inf système
├── C:\Windows\System32\DriverStore\   # Magasin pilotes
│   └── FileRepository\                 # Packages indexés
└── C:\Windows\Temp\                   # Installation temporaire
```

### Driver Store
```
C:\Windows\System32\DriverStore\FileRepository\
└── manufacturer_device_version_hash\
    ├── driver.inf
    ├── driver.sys
    ├── driver.cat
    └── autres fichiers...

Avantages:
- Intégrité garantie
- Signature vérifiée  
- Installation rapide
- Rollback possible
```

## ⚙️ Gestion Pilotes

### Device Manager (devmgmt.msc)
```
Actions courantes:
├── Clic droit périphérique > Properties
├── Driver tab > Update Driver
├── Driver tab > Roll Back Driver
├── Driver tab > Uninstall Device
└── View > Show hidden devices
```

### Commandes PowerShell
```powershell
# Lister pilotes installés
Get-WmiObject Win32_PnPSignedDriver | Select DeviceName, DriverVersion, DriverDate

# Informations périphériques
Get-PnpDevice | Where-Object {$_.Status -eq "Error"}
Get-PnpDevice -FriendlyName "*Audio*"

# Gérer pilotes
Add-WindowsDriver -Online -Driver "C:\Drivers\INF_File.inf"
Remove-WindowsDriver -Online -Driver "oem42.inf"

# Driver packages
Get-WindowsDriver -Online
Export-WindowsDriver -Online -Destination "C:\Backup\Drivers"
```

### Commandes CMD Classiques
```cmd
# PnPUtil - Gestion packages pilotes
pnputil /enum-drivers              # Lister packages installés
pnputil /add-driver driver.inf     # Ajouter package
pnputil /delete-driver oem42.inf  # Supprimer package
pnputil /export-driver * C:\Backup # Exporter tous pilotes

# Device Manager CLI
devcon listclass *                 # Lister classes périphériques
devcon status *                    # État tous périphériques  
devcon update driver.inf "Hardware ID"  # Mettre à jour pilote
devcon remove "Hardware ID"        # Supprimer périphérique
```

### DISM - Pilotes Images
```cmd
# Gestion pilotes hors ligne
DISM /Image:C:\Mount /Get-Drivers          # Lister pilotes image
DISM /Image:C:\Mount /Add-Driver /Driver:C:\Drivers /Recurse
DISM /Image:C:\Mount /Remove-Driver /Driver:oem42.inf

# En ligne (système actuel)
DISM /Online /Get-Drivers /Format:Table
DISM /Online /Add-Driver /Driver:"C:\Drivers\mydriver.inf"
```

## 🔍 Diagnostic Pilotes

### Outils Windows Intégrés
```
Device Manager:
├── Icônes d'état (!, ?, ↓)
├── Properties > Details > Hardware IDs
├── Properties > Events (historique)
└── Properties > Resources (conflits)

System Information (msinfo32):
├── Components > Problem Devices
├── Hardware Resources > Conflicts/Sharing
└── Software Environment > Drivers
```

### Event Viewer
```
Logs pertinents:
├── Windows Logs > System
│   └── Source: Service Control Manager, Kernel-PnP
├── Applications and Services Logs > Microsoft > Windows
│   ├── Kernel-PnP/Configuration
│   ├── DriverFrameworks-UserMode/Operational  
│   └── PnP-DeviceInstall/Operational
```

### Commandes Diagnostic
```cmd
# Vérifier signature pilotes
sigverif.exe                       # GUI vérification signatures

# Informations système détaillées
systeminfo | findstr /i driver    # Pilotes chargés
wmic path win32_systemdriver get name,pathname,state

# Test périphériques
devcon hwids *                     # Hardware IDs tous périphériques
devcon drivernodes *               # Pilotes compatibles
```

## 🚨 Résolution Problèmes

### Problèmes Courants
| Symptôme | Cause Probable | Solution |
|----------|---------------|----------|
| **Périphérique non reconnu** | Pilote manquant | Download pilote fabricant |
| **Code d'erreur 10** | Pilote défaillant | Update/reinstall pilote |
| **Code d'erreur 28** | Pilote non installé | Device Manager > Update |
| **BSOD au démarrage** | Pilote corrompu | Mode sans échec, rollback |
| **Performance dégradée** | Pilote générique | Pilote spécialisé fabricant |

### Codes d'Erreur Device Manager
| Code | Description | Action |
|------|-------------|--------|
| **1** | Configuration incorrecte | Réinstaller pilote |
| **10** | Périphérique ne démarre pas | Update pilote |
| **28** | Pilotes non installés | Installer pilotes |
| **43** | Problème signalé par Windows | Diagnostic approfondi |
| **52** | Signature numérique | Pilote non signé |

### Récupération Pilotes
```cmd
# Sauvegarde pilotes actuels
mkdir C:\BackupDrivers
pnputil /export-driver * C:\BackupDrivers

# Mode sans échec - Rollback
# F8 au démarrage > Mode sans échec
# Device Manager > Pilote défaillant > Roll Back

# Restauration système
rstrui.exe                 # Interface restauration
wmic.exe /Namespace:\\root\default Path SystemRestore Call CreateRestorePoint "Avant modification pilotes", 100, 12
```

## 🌐 Bureau à Distance (RDP)

### Activation RDP
```cmd
# Via Registry
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

# Via PowerShell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

# Via GUI: Paramètres > Système > Bureau à distance
```

### Client RDP (mstsc)
```cmd
# Connexion basique
mstsc /v:192.168.1.100                    # IP directe
mstsc /v:server.domain.com                # FQDN
mstsc /v:server.domain.com:3390           # Port personnalisé

# Options avancées
mstsc /admin                              # Session console
mstsc /f                                  # Plein écran
mstsc /w:1920 /h:1080                     # Résolution spécifique
mstsc /multimon                           # Multi-moniteurs

# Avec fichier RDP
mstsc connection.rdp                      # Charger profil
mstsc /edit connection.rdp                # Éditer profil
```

### Configuration Serveur RDP
```cmd
# Port RDP (défaut: 3389)
netsh advfirewall firewall add rule name="RDP Custom" dir=in action=allow protocol=TCP localport=3390

# Registry port change
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 3390 /f

# Redémarrer service
net stop "Remote Desktop Services"
net start "Remote Desktop Services"
```

### Sécurité RDP
```
Bonnes pratiques:
├── Changer port par défaut (3389)
├── Activer authentification réseau (NLA)
├── Limiter utilisateurs autorisés
├── Utiliser comptes forts/MFA
├── VPN pour accès externe
└── Monitoring connexions

Registry NLA:
HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp
UserAuthentication = 1
```

### Fichiers RDP
```ini
# Exemple connection.rdp
screen mode id:i:2                    # Plein écran
use multimon:i:1                      # Multi-moniteurs  
desktopwidth:i:1920                   # Largeur
desktopheight:i:1080                  # Hauteur
session bpp:i:32                      # Profondeur couleur
winposstr:s:0,1,0,0,800,600          # Position fenêtre
compression:i:1                       # Compression
keyboardhook:i:2                      # Redirection clavier
audiocapturemode:i:1                  # Capture audio
videoplaybackmode:i:1                 # Lecture vidéo
connection type:i:7                   # Type connexion (LAN)
networkautodetect:i:1                 # Auto-détection réseau
bandwidthautodetect:i:1               # Auto-détection bande passante
enableworkspacereconnect:i:0          # Reconnexion workspace
```

## 🔒 Signature & Sécurité Pilotes

### Windows Driver Signing
```
Niveaux sécurité (depuis Windows 10):
├── Kernel Mode Code Signing (KMCS) - Obligatoire
├── Windows Hardware Quality Labs (WHQL) - Recommandé  
├── Authenticode - Acceptable
└── Non signé - Bloqué par défaut
```

### Contournement Signature (Test)
```cmd
# Mode test (redémarrage requis)
bcdedit /set testsigning on           # Activer mode test
bcdedit /set testsigning off          # Désactiver

# F8 Advanced Boot > Disable driver signature enforcement (temporaire)

# Forcer installation pilote non signé (PowerShell admin)
pnputil /add-driver driver.inf /install /force
```

### Vérification Signatures
```cmd
# Sigverif - Vérificateur signatures fichiers
sigverif.exe

# Via PowerShell
Get-AuthenticodeSignature "C:\Windows\System32\drivers\*.sys" | Where-Object {$_.Status -ne "Valid"}

# Détails certificat
certlm.msc                            # Certificats machine locale
```

## 🛠️ Outils Tiers Utiles

### Utilitaires Pilotes
- **Driver Booster** : Mise à jour automatique (IObit)
- **Snappy Driver Installer** : Open source, offline
- **Double Driver** : Sauvegarde/restauration pilotes
- **DriverMax** : Gestion complète pilotes

### Outils Diagnostic
- **BlueScreenView** : Analyse BSOD (NirSoft)
- **DriverView** : Liste pilotes chargés (NirSoft) 
- **WhoCrashed** : Analyse plantages système
- **Process Monitor** : Monitoring activité fichiers/registry

## 💡 Bonnes Pratiques

### Maintenance Pilotes
- ✅ **Sauvegarde** pilotes avant mise à jour
- ✅ **Point restauration** avant modifications
- ✅ **Pilotes constructeur** vs génériques Windows
- ✅ **Updates critiques** seulement (sauf problèmes)
- ✅ **Test** en environnement non-critique

### Déploiement Entreprise
- ✅ **WSUS/SCCM** pour distribution centralisée
- ✅ **Images WIM** avec pilotes intégrés
- ✅ **Answer files** (unattend.xml) pour automation
- ✅ **Driver packages** standardisés par modèle
- ✅ **Testing** approfondi avant déploiement

### Sécurité
- ✅ **Sources fiables** uniquement (constructeur, Windows Update)
- ✅ **Vérification signatures** systématique
- ✅ **Isolation test** pour pilotes beta/non-WHQL
- ✅ **Monitoring** comportement post-installation
- ✅ **Politique** signature pilotes en entreprise

---
**💡 Memo** : `pnputil` pour gestion packages, `devcon` pour debug, toujours sauvegarder avant modification !