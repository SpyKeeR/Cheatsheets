# 🖥️ Déploiement Windows — Aide-mémoire

## 🏗️ Concepts de Déploiement

### Types d'Images
| Type | Description | Usage | Avantages |
|------|-------------|-------|-----------|
| **Thick Image** | OS + apps + config complète | Postes fixes standard | Déploiement rapide |
| **Thin Image** | OS minimal + installation apps post-déploiement | Environnements variés | Flexibilité |
| **Hybrid Image** | OS + apps de base | Compromis | Rapidité + flexibilité |

### Méthodes de Déploiement
```
High Touch (Manuel) → Low Touch (Semi-auto) → Zero Touch (Automatique)
        ↓                      ↓                       ↓
   Installation CD        Answer Files           Task Sequences
      + Config               + Scripts             + Full Auto
```

## 🔧 Sysprep (System Preparation)

### Objectifs Sysprep
- **Généraliser** : Supprimer informations uniques (SID, nom machine)
- **OOBE** : Préparer première expérience utilisateur
- **Auditer** : Mode préparation avancée

### Syntaxe & Options
```cmd
# Emplacement
C:\Windows\System32\Sysprep\sysprep.exe

# Syntaxe de base
sysprep.exe /oobe /generalize /shutdown

# Options principales
/oobe          # Out-Of-Box Experience (configuration utilisateur)
/audit         # Mode audit (configuration système)
/generalize    # Supprimer données uniques (SID, pilotes, etc.)
/unattend:fichier.xml  # Fichier de réponses automatiques

# Actions après préparation
/shutdown      # Arrêter le système
/reboot        # Redémarrer
/quit          # Fermer sysprep uniquement
```

### Modes de Fonctionnement
```
Mode Normal → OOBE → Configuration utilisateur
     ↓
Mode Audit (Ctrl+Shift+F3) → Configuration avancée
     ↓
Sysprep → Image maître prête
```

### Limitations Sysprep
- **Maximum 3 exécutions** par installation Windows (sauf réarm)
- **Pilotes** : Conserver avec `PersistAllDeviceInstalls`
- **Applications** : Certaines ne supportent pas la généralisation
- **Domaine** : Doit être retiré avant sysprep

## 📁 Structure Fichiers & Logs

### Emplacements Importants
```
C:\Windows\System32\Sysprep\
├── sysprep.exe                    # Exécutable principal
├── Panther\                       # Logs de préparation
│   ├── setupact.log              # Actions réalisées
│   ├── setuperr.log              # Erreurs rencontrées
│   └── IE\                       # Logs Internet Explorer
└── ActionFiles\                   # Actions personnalisées
```

### Analyse des Logs
```cmd
# Logs principaux
C:\Windows\System32\Sysprep\Panther\setupact.log    # Succès
C:\Windows\System32\Sysprep\Panther\setuperr.log    # Erreurs

# Logs setup Windows
C:\Windows\Panther\setupact.log                     # Installation OS
C:\Windows\Panther\unattend.xml                     # Fichier réponses utilisé
```

## 📋 Answer Files (Unattend.xml)

### Structure Answer File
```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
  <!-- Pass 1 - windowsPE : Phase boot WinPE -->
  <settings pass="windowsPE">
    <component name="Microsoft-Windows-Setup">
      <!-- Configuration disque, produit key -->
    </component>
  </settings>
  
  <!-- Pass 4 - specialize : Personnalisation système -->
  <settings pass="specialize">
    <component name="Microsoft-Windows-Shell-Setup">
      <!-- Nom machine, fuseau, langue -->
    </component>
  </settings>
  
  <!-- Pass 7 - oobeSystem : Configuration utilisateur -->
  <settings pass="oobeSystem">
    <component name="Microsoft-Windows-Shell-Setup">
      <!-- Comptes utilisateurs, OOBE settings -->
    </component>
  </settings>
</unattend>
```

### Passes de Configuration
| Pass | Phase | Description | Usage |
|------|-------|-------------|-------|
| **1 - windowsPE** | Boot WinPE | Config disque, clé produit | Installation |
| **2 - offlineServicing** | Hors ligne | Mise à jour, pilotes | DISM |
| **3 - generalize** | Généralisation | Nettoyage avant image | Sysprep |
| **4 - specialize** | Spécialisation | Config matérielle | Déploiement |
| **5 - auditSystem** | Audit système | Config système mode audit | Préparation |
| **6 - auditUser** | Audit utilisateur | Config utilisateur mode audit | Préparation |
| **7 - oobeSystem** | OOBE | Première expérience | Finalisation |

### Exemple Configuration Courante
```xml
<!-- Nom machine et configuration réseau -->
<ComputerName>PC-001</ComputerName>
<TimeZone>Romance Standard Time</TimeZone>

<!-- Compte administrateur local -->
<UserAccounts>
  <LocalAccounts>
    <LocalAccount>
      <Name>Admin</Name>
      <Group>Administrators</Group>
      <Password>
        <Value>UABhAHMAcwB3AG8AcgBkAA==</Value>
        <PlainText>false</PlainText>
      </Password>
    </LocalAccount>
  </LocalAccounts>
</UserAccounts>

<!-- Ignorer étapes OOBE -->
<OOBE>
  <HideEULAPage>true</HideEULAPage>
  <NetworkLocation>Work</NetworkLocation>
  <ProtectYourPC>1</ProtectYourPC>
</OOBE>
```

## 🛠️ DISM (Deployment Image Servicing)

### Gestion Images Hors Ligne
```cmd
# Monter image pour modification
DISM /Mount-Image /ImageFile:C:\Images\install.wim /Index:1 /MountDir:C:\Mount

# Ajouter pilotes
DISM /Image:C:\Mount /Add-Driver /Driver:C:\Pilotes /Recurse

# Ajouter packages/fonctionnalités
DISM /Image:C:\Mount /Add-Package /PackagePath:C:\Updates\update.msu
DISM /Image:C:\Mount /Enable-Feature /FeatureName:IIS-WebServer

# Appliquer modifications et démonter
DISM /Unmount-Image /MountDir:C:\Mount /Commit

# Annuler modifications
DISM /Unmount-Image /MountDir:C:\Mount /Discard
```

### Capture d'Images
```cmd
# Capturer partition système
DISM /Capture-Image /ImageFile:C:\Images\CustomWindows.wim /CaptureDir:C:\ /Name:"Windows Custom" /Description:"Image personnalisée"

# Ajouter image à WIM existant
DISM /Append-Image /ImageFile:C:\Images\install.wim /CaptureDir:C:\ /Name:"Custom Build"

# Optimiser image (supprime doublons)
DISM /Export-Image /SourceImageFile:C:\Images\install.wim /SourceIndex:1 /DestinationImageFile:C:\Images\optimized.wim
```

### Informations Images
```cmd
# Lister images dans WIM
DISM /Get-WimInfo /WimFile:C:\Images\install.wim

# Détails image spécifique
DISM /Get-ImageInfo /ImageFile:C:\Images\install.wim /Index:1

# Pilotes dans image
DISM /Image:C:\Mount /Get-Drivers
```

## 💾 Technologies de Déploiement

### Windows Deployment Services (WDS)
```
Architecture WDS:
├── PXE Server : Boot réseau (DHCP options 66/67)
├── TFTP Server : Transfert boot files
├── Image Repository : Store des images
└── Multicast : Distribution efficace
```

#### Configuration WDS
```cmd
# Installation rôle
Install-WindowsFeature WDS -IncludeManagementTools

# Configuration initiale
wdsutil /initialize-server /remInst:D:\RemoteInstall

# Ajouter image boot
wdsutil /add-image /imagefile:C:\Images\boot.wim /imagetype:Boot

# Ajouter image install
wdsutil /add-image /imagefile:C:\Images\install.wim /imagetype:Install
```

### Microsoft Deployment Toolkit (MDT)
```
Composants MDT:
├── Deployment Workbench : Interface gestion
├── Task Sequences : Séquences automatisées
├── Selection Profiles : Groupes logiques
└── Boot Images : WinPE personnalisé
```

### System Center Configuration Manager (SCCM)
- **Operating System Deployment** : Déploiement automatisé
- **Software Distribution** : Gestion applications
- **Compliance Management** : Conformité configurations
- **Reporting** : Suivi déploiements

## 🔍 Windows Preinstallation Environment (WinPE)

### Création WinPE
```cmd
# Installer Windows ADK + WinPE addon
# Lancer Deployment and Imaging Tools Environment

# Créer environnement WinPE
copype amd64 C:\WinPE_amd64

# Personnaliser (ajouter pilotes, outils)
DISM /Mount-Image /ImageFile:C:\WinPE_amd64\media\sources\boot.wim /Index:1 /MountDir:C:\WinPE_amd64\mount
DISM /Image:C:\WinPE_amd64\mount /Add-Driver /Driver:C:\Pilotes /Recurse
DISM /Unmount-Image /MountDir:C:\WinPE_amd64\mount /Commit

# Créer ISO bootable
MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64\WinPE_amd64.iso
```

### Scripts WinPE Utiles
```cmd
# Partitionnement automatique (diskpart script)
wpeinit                          # Initialiser WinPE
diskpart /s script.txt          # Exécuter script partitionnement
dism /apply-image               # Appliquer image
bcdboot C:\Windows /s C:        # Configurer boot
```

## 🎯 Stratégies de Déploiement

### Lite Touch Installation (LTI)
- **MDT** : Déploiement semi-automatique
- **Intervention minimale** : Sélections ponctuelles
- **Flexibilité** : Choix images/applications

### Zero Touch Installation (ZTI)
- **SCCM** : Déploiement entièrement automatique
- **Task Sequences** : Logique complexe
- **Targeting** : Déploiement par critères

### User State Migration Tool (USMT)
```cmd
# Capturer profil utilisateur
scanstate.exe C:\Backup\UserState /i:migdocs.xml /i:migapp.xml /v:13 /l:scan.log

# Restaurer profil utilisateur
loadstate.exe C:\Backup\UserState /i:migdocs.xml /i:migapp.xml /v:13 /l:load.log
```

## 🚨 Troubleshooting Déploiement

### Problèmes Courants Sysprep
| Erreur | Cause Probable | Solution |
|--------|---------------|----------|
| **Apps Store** | Applications non supprimées | Remove-AppxPackage |
| **3 exécutions max** | Limite atteinte | Réarm ou réinstallation |
| **Pilotes** | Pilotes personnalisés | PersistAllDeviceInstalls |
| **Profils** | Profils utilisateurs actifs | Nettoyage avant sysprep |

### Registry Tweaks Utiles
```cmd
# Conserver pilotes après sysprep
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\Sysprep\Settings\sppnp" /v PersistAllDeviceInstalls /t REG_DWORD /d 1

# Ignorer erreurs Apps Store
reg add "HKEY_LOCAL_MACHINE\SYSTEM\Setup\Status\SysprepStatus" /v GeneralizationState /t REG_DWORD /d 7
```

### Commandes Debug
```cmd
# Vérifier état sysprep
reg query "HKEY_LOCAL_MACHINE\SYSTEM\Setup\Status\SysprepStatus"

# Forcer nettoyage profils temporaires
rd /s /q C:\Users\Temp*

# Réinitialiser compteur sysprep (attention!)
slmgr /rearm
```

## 🔧 Outils Complémentaires

### Windows System Image Manager (WSIM)
- **Création answer files** : Interface graphique
- **Validation** : Vérification compatibilité
- **Catalogues** : Génération pour chaque image

### ImageX (Legacy) / DISM
```cmd
# ImageX (remplacé par DISM)
imagex /capture C: C:\Images\windows.wim "Windows Custom"
imagex /apply C:\Images\windows.wim 1 C:

# DISM (moderne)
DISM /Capture-Image /ImageFile:C:\Images\windows.wim /CaptureDir:C:\ /Name:"Windows Custom"
DISM /Apply-Image /ImageFile:C:\Images\windows.wim /Index:1 /ApplyDir:C:\
```

### Outils Tiers
- **Ghost** : Symantec (legacy)
- **Acronis** : Solutions backup/restore
- **CloneZilla** : Open source
- **FOG Project** : Déploiement réseau gratuit

## 💡 Bonnes Pratiques

### Préparation Image
- ✅ **Updates** : Installer toutes mises à jour avant capture
- ✅ **Drivers** : Inclure pilotes génériques uniquement
- ✅ **Software** : Applications de base seulement
- ✅ **Profiles** : Supprimer profils utilisateurs temporaires
- ✅ **Registry** : Nettoyer clés inutiles

### Sécurité
- ✅ **Passwords** : Encoder dans answer files
- ✅ **Keys** : Protéger clés produit
- ✅ **Access** : Sécuriser partages images
- ✅ **Audit** : Logger déploiements
- ✅ **Testing** : Valider en environnement isolé

### Performance
- ✅ **WIM Optimization** : Export pour compacter
- ✅ **Multicast** : Pour déploiements multiples
- ✅ **Hardware Detection** : Injection pilotes ciblée
- ✅ **Network** : Optimiser bande passante
- ✅ **Storage** : SSD pour images et caches

### Organisation
- ✅ **Naming** : Convention images cohérente
- ✅ **Versioning** : Gestion versions images
- ✅ **Documentation** : Procédures et configurations
- ✅ **Backup** : Sauvegarder images maîtres
- ✅ **Testing** : Lab dédié aux tests

---
**💡 Memo** : Sysprep max 3 fois, toujours OOBE+Generalize, logs dans Panther, DISM pour modifications offline !