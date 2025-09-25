# 📝 Registre Windows — Aide-mémoire

## 🏗️ Architecture du Registre

### Ruches Principales (Hives)
```
Base de Registre Windows:
├── HKEY_CLASSES_ROOT (HKCR)
├── HKEY_CURRENT_USER (HKCU)  
├── HKEY_LOCAL_MACHINE (HKLM)
├── HKEY_USERS (HKU)
└── HKEY_CURRENT_CONFIG (HKCC)
```

| Ruche | Abréviation | Contenu | Usage |
|-------|-------------|---------|--------|
| **HKEY_CLASSES_ROOT** | HKCR | Extensions fichiers, COM | Associations fichiers/applications |
| **HKEY_CURRENT_USER** | HKCU | Profil utilisateur courant | Paramètres personnels |
| **HKEY_LOCAL_MACHINE** | HKLM | Configuration système | Paramètres globaux machine |
| **HKEY_USERS** | HKU | Tous profils utilisateurs | Administration multi-utilisateurs |
| **HKEY_CURRENT_CONFIG** | HKCC | Config matérielle active | Profils matériels |

### Hiérarchie & Relations
```
Liens logiques:
HKCR → Vue combinée de HKLM\SOFTWARE\Classes + HKCU\SOFTWARE\Classes
HKCC → Alias vers HKLM\SYSTEM\CurrentControlSet\Hardware Profiles\Current
HKCU → Alias vers HKU\{SID utilisateur courant}
```

## 🗂️ Structure HKEY_LOCAL_MACHINE

### Sous-clés Critiques
```
HKLM\
├── HARDWARE          # Matériel détecté au boot
├── SAM               # Security Account Manager (mots de passe)
├── SECURITY          # Politiques sécurité locale
├── SOFTWARE          # Applications système
├── SYSTEM           # Configuration système
└── BCD00000000      # Boot Configuration Data (UEFI)
```

#### HKLM\SOFTWARE (Applications)
```
SOFTWARE\
├── Classes\          # Types fichiers système
├── Microsoft\        # Logiciels Microsoft
│   ├── Windows\      # Configuration Windows
│   ├── Windows NT\   # Paramètres NT
│   └── Office\       # Suite Office
├── Policies\         # GPO appliquées
└── WOW6432Node\      # Apps 32-bit sur système 64-bit
```

#### HKLM\SYSTEM (Configuration)
```
SYSTEM\
├── CurrentControlSet\    # Configuration active (alias)
├── ControlSet001\       # Jeu configuration 1
├── ControlSet002\       # Jeu configuration 2 (backup)
├── Select\             # Pointeur vers ControlSet actif
└── Setup\              # Infos installation Windows
```

### Current Control Set
```
CurrentControlSet\
├── Control\            # Paramètres système globaux
├── Enum\              # Périphériques énumérés
├── Hardware Profiles\ # Profils matériels
├── Policies\          # Politiques locales
└── Services\          # Services Windows
```

## 📋 Types de Données

### Types de Valeurs
| Type | Description | Exemple | Usage |
|------|-------------|---------|--------|
| **REG_SZ** | Chaîne texte | `"C:\Program Files"` | Chemins, noms |
| **REG_DWORD** | Entier 32-bit | `0x00000001` | Flags, compteurs |
| **REG_QWORD** | Entier 64-bit | `0x0000000000000001` | Grandes valeurs |
| **REG_BINARY** | Données binaires | `01 FF A3 2C` | Config binaire |
| **REG_MULTI_SZ** | Multi-chaînes | `"ligne1\0ligne2\0"` | Listes |
| **REG_EXPAND_SZ** | Chaîne avec variables | `"%SystemRoot%\system32"` | Chemins dynamiques |

### Conventions Valeurs
- **0** = Désactivé, **1** = Activé (booléens)
- **DWORD** en hexadécimal : `0x00000001`
- **Chaînes vides** vs **valeurs absentes** : comportements différents
- **Variables d'environnement** : `%SystemRoot%`, `%ProgramFiles%`

## 🔧 Outils d'Accès

### Interface Graphique
```cmd
regedit.exe              # Éditeur registre standard
regedt32.exe             # Éditeur avancé (Windows NT legacy)
```

#### Navigation regedit
- **Ctrl+F** : Rechercher clé/valeur/données
- **F3** : Recherche suivante
- **F5** : Actualiser
- **Suppr** : Supprimer clé/valeur
- **F2** : Renommer

### Ligne de Commande
```cmd
# Requêtes
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion /v ProductName
reg query "HKCU\Control Panel\Desktop" /s    # Récursif

# Ajout/Modification
reg add HKCU\Software\MonApp /v Version /t REG_SZ /d "1.0" /f
reg add HKLM\SOFTWARE\MonApp /v Enabled /t REG_DWORD /d 1

# Suppression
reg delete HKCU\Software\MonApp /v Version /f
reg delete HKCU\Software\MonApp /f          # Clé entière

# Import/Export
reg export HKCU\Software\MonApp backup.reg
reg import backup.reg
```

### PowerShell (Moderne)
```powershell
# Lecture
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion" -Name ProductName
Get-ChildItem "HKCU:\Software" | Select-Object Name

# Écriture
Set-ItemProperty "HKCU:\Software\MonApp" -Name "Version" -Value "1.0"
New-ItemProperty "HKCU:\Software\MonApp" -Name "Enabled" -PropertyType DWord -Value 1

# Suppression
Remove-ItemProperty "HKCU:\Software\MonApp" -Name "Version"
Remove-Item "HKCU:\Software\MonApp" -Recurse

# Test existence
Test-Path "HKCU:\Software\MonApp"
```

## 📍 Emplacements Clés Importantes

### Configuration Système
| Chemin | Description | Usage Typique |
|--------|-------------|---------------|
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion` | Version Windows | ProductName, BuildNumber |
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion` | Infos détaillées NT | ReleaseId, UBR |
| `HKLM\SYSTEM\CurrentControlSet\Control\ComputerName` | Nom ordinateur | ComputerName |
| `HKLM\SOFTWARE\Policies` | GPO machine | Politiques appliquées |
| `HKCU\SOFTWARE\Policies` | GPO utilisateur | Politiques utilisateur |

### Applications & Démarrage
```
# Applications installées
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\   # User-only

# Démarrage automatique
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\         # Tous utilisateurs
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\         # Utilisateur courant
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\     # Une seule fois

# Services
HKLM\SYSTEM\CurrentControlSet\Services\
```

### Interface Utilisateur
```
# Bureau utilisateur
HKCU\Control Panel\Desktop\
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\

# Associations fichiers
HKCR\.*                      # Extensions (.txt, .pdf, etc.)
HKCR\Applications\           # Applications déclarées
```

## 🔐 Sécurité & Permissions

### Permissions Registre
```
Permissions standard:
├── Full Control    # Lecture, écriture, suppression complète
├── Read           # Lecture clés et valeurs
├── Special        # Permissions personnalisées
└── Owner          # Propriétaire (contrôle total)

Héritage:
├── Conteneur parent → sous-clés enfants
├── Possibilité bloquer héritage
└── Permissions explicites > héritées
```

### Accès Sécurisé
```cmd
# Prendre possession (ownership)
takeown /f "HKLM\SOFTWARE\MonApp" /r /d y

# Modifier permissions via icacls (fichiers reg exportés)
icacls backup.reg /grant "Administrators:F"

# Via PowerShell (ACL)
$acl = Get-Acl "HKLM:\SOFTWARE\MonApp"
$acl.SetOwner([System.Security.Principal.NTAccount]"Administrators")
Set-Acl "HKLM:\SOFTWARE\MonApp" $acl
```

## 💾 Sauvegarde & Restauration

### Export/Import Complet
```cmd
# Export ruche complète
reg export HKCU C:\Backup\HKCU_backup.reg
reg export HKLM C:\Backup\HKLM_backup.reg

# Export sélectif
reg export "HKCU\Software\Microsoft\Office" C:\Backup\Office.reg

# Import (attention : écrase existant)
reg import C:\Backup\HKCU_backup.reg

# Import silencieux
regedit /s C:\Backup\Office.reg
```

### Points de Restauration
- **Création** : Automatique avant modifications importantes
- **Localisation** : Inclut état registre
- **Restauration** : Via `rstrui.exe` ou WinRE

### Sauvegarde Système
```cmd
# Sauvegarde manuelle ruches
copy C:\Windows\System32\config\SOFTWARE C:\Backup\SOFTWARE.bak
copy C:\Windows\System32\config\SYSTEM C:\Backup\SYSTEM.bak
# (nécessite mode hors ligne ou PE)

# Via wbadmin (sauvegarde système complète)
wbadmin start backup -backupTarget:E: -include:C: -allCritical -quiet
```

## 🔍 Diagnostic & Dépannage

### Problèmes Courants
| Symptôme | Cause Probable | Solution |
|----------|---------------|----------|
| **Associations fichiers** | HKCR corrompu | `sfc /scannow`, rebuild associations |
| **Services ne démarrent pas** | Registre Services corrompu | Vérifier `HKLM\SYSTEM\...\Services` |
| **Profil utilisateur** | HKCU inaccessible | Nouveau profil, migration données |
| **Erreurs permissions** | ACL registre restrictives | `takeown`, modifier permissions |

### Commandes Diagnostic
```cmd
# Vérifier intégrité registre
sfc /scannow                  # System File Checker
DISM /Online /Cleanup-Image /RestoreHealth

# Audit registre
auditpol /get /category:"Object Access"
eventvwr.msc                  # Event Viewer pour erreurs registre

# Monitoring modifications temps réel
procmon.exe                   # Process Monitor (filter Registry)
```

### Réparation Registre
```cmd
# Mode sans échec + réparation
# F8 > Safe Mode > cmd
chkdsk C: /f                  # Vérifier système fichiers
sfc /scannow                  # Réparer fichiers système

# Restauration via PE/WinRE
copy C:\Backup\SYSTEM C:\Windows\System32\config\SYSTEM
# (nécessite arrêt système)
```

## 🎯 Cas d'Usage Fréquents

### Optimisation Système
```cmd
# Désactiver services inutiles
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Fax" /v Start /t REG_DWORD /d 4

# Modifier délai démarrage
reg add "HKLM\SYSTEM\CurrentControlSet\Control" /v WaitToKillServiceTimeout /t REG_SZ /d "5000"

# Optimiser menu Démarrer
reg add "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced" /v StartMenuAdminTools /t REG_DWORD /d 1
```

### Configuration Réseau
```cmd
# Configuration TCP/IP via registre
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" /v TcpWindowSize /t REG_DWORD /d 65535

# Paramètres DNS
reg query "HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" /v NameServer
```

### Gestion Applications
```cmd
# Désactiver notifications Windows 10
reg add "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\PushNotifications" /v ToastEnabled /t REG_DWORD /d 0

# Configuration Windows Update
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1
```

## 💡 Bonnes Pratiques

### Sécurité
- ✅ **Toujours sauvegarder** avant modification
- ✅ **Tester** modifications en environnement non-critique
- ✅ **Utiliser PowerShell** pour scripts (gestion erreurs)
- ✅ **Vérifier permissions** avant modifications système
- ✅ **Documenter** changements pour rollback

### Performance
- ✅ **Éviter modifications** registre en production directement
- ✅ **Utiliser GPO** quand possible (centralisé, auditable)  
- ✅ **Nettoyer** entrées obsolètes périodiquement
- ✅ **Monitoring** modifications via auditpol

### Maintenance
- ✅ **Points restauration** avant changements importants
- ✅ **Export registre** applications critiques
- ✅ **Vérification intégrité** (`sfc /scannow`) mensuelle
- ✅ **Scripts déploiement** pour changements répétitifs
- ✅ **Documentation** structure registre personnalisée

---
**💡 Memo** : Toujours sauvegarder (`reg export`) avant modification, utiliser PowerShell pour gestion erreurs, GPO > registre manuel !