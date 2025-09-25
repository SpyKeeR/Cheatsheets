# 🛠️ Maintenance Windows — Aide-mémoire

## 🚨 WinRE & Environnement de Récupération

### Accès WinRE (Windows Recovery Environment)
```cmd
# Depuis Windows fonctionnel
shutdown /r /o /f /t 0        # Redémarrage options avancées
# Ou : Paramètres > Mise à jour > Récupération > Redémarrer maintenant

# Au démarrage
Shift + Redémarrer            # Maintenir Shift pendant clic Redémarrer
F8 (legacy)                   # Options démarrage avancées (BIOS)

# Automatique
# WinRE se lance après 2 échecs consécutifs ou 2 arrêts < 2min
```

### Options WinRE Disponibles
| Option | Description | Usage |
|--------|-------------|-------|
| **Continuer** | Démarrage Windows normal | Test après réparation |
| **Dépannage** | Outils réparation avancés | ↓ Voir section suivante |
| **Éteindre** | Arrêt complet | Intervention matérielle |
| **Utiliser périph.** | Boot sur USB/DVD | Installation/récupération |

### Outils Dépannage Avancé
```
Dépannage > Options avancées :
├── Réparation du démarrage (Startup Repair)
├── Désinstaller mises à jour (Rollback updates)  
├── Restauration système (System Restore)
├── Récupération image système (System Image Recovery)
├── Réparation démarrage UEFI (UEFI Firmware Settings)
├── Paramètres de démarrage (Startup Settings)
└── Invite commandes (Command Prompt)
```

## 🔧 Réparation Démarrage

### Réparation Automatique Démarrage
```cmd
# Via WinRE
bootrec /fixmbr              # Réparer Master Boot Record
bootrec /fixboot             # Réécrire secteur boot
bootrec /scanos              # Scanner installations Windows
bootrec /rebuildbcd          # Reconstruire BCD (Boot Configuration Data)

# Séquence complète réparation
bootrec /fixmbr && bootrec /fixboot && bootrec /scanos && bootrec /rebuildbcd
```

### Réparation BCD (Boot Configuration Data)
```cmd
# Sauvegarder BCD existant
bcdedit /export C:\BCD_Backup

# Reconstruire BCD depuis zéro
attrib bcd -s -h -r          # Retirer attributs fichier BCD
ren c:\boot\bcd bcd.old      # Renommer ancien BCD
bootrec /rebuildbcd          # Reconstruire

# Réparation manuelle BCD
bcdboot C:\Windows /s C: /f ALL    # Recréer fichiers boot
```

### Réparation UEFI/GPT
```cmd
# Monter partition EFI
diskpart
list disk                    # Identifier disque système
select disk 0                # Sélectionner disque
list volume                  # Trouver partition EFI (FAT32)
select volume X              # Sélectionner partition EFI
assign letter=Z              # Assigner lettre
exit

# Reconstruire boot UEFI
bcdboot C:\Windows /s Z: /f UEFI
```

## 💻 SFC & DISM (Réparation Système)

### SFC (System File Checker)
```cmd
# Vérification intégrité fichiers système
sfc /scannow                 # Scan complet + réparation auto
sfc /verifyonly              # Scan uniquement (pas réparation)
sfc /scanfile=C:\Windows\System32\kernel32.dll  # Fichier spécifique

# SFC offline (depuis WinRE)
sfc /scannow /offbootdir=C:\ /offwindir=C:\Windows
```

### DISM (Deployment Image Servicing Management)
```cmd
# Vérification santé image Windows
DISM /Online /Cleanup-Image /CheckHealth      # Vérification rapide
DISM /Online /Cleanup-Image /ScanHealth       # Scan approfondi
DISM /Online /Cleanup-Image /RestoreHealth    # Réparation (nécessite internet)

# DISM offline
DISM /Image:C:\ /Cleanup-Image /RestoreHealth /Source:D:\Sources\install.wim

# Nettoyage composants
DISM /Online /Cleanup-Image /StartComponentCleanup
DISM /Online /Cleanup-Image /StartComponentCleanup /ResetBase  # Permanent
```

## 🔄 Restauration Système

### Points de Restauration
```cmd
# Créer point restauration
wmic.exe /Namespace:\\root\default Path SystemRestore Call CreateRestorePoint "Mon Point", 100, 12

# Gérer restauration système
rstrui.exe                   # Interface graphique restauration
powercfg /h off             # Désactiver hibernation (libère espace)

# Via PowerShell
Enable-ComputerRestore "C:\"
Checkpoint-Computer -Description "Avant modification"
Get-ComputerRestorePoint    # Lister points disponibles
```

### Configuration Restauration
```cmd
# Vérifier état restauration système
vssadmin list shadows       # Lister clichés instantanés
vssadmin list providers     # Lister fournisseurs VSS

# Paramétrage via registre
# HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SystemRestore
# DisableSR = 0 (activé) / 1 (désactivé)
```

## 🧹 Nettoyage & Optimisation

### Disk Cleanup Avancé
```cmd
# Nettoyage disque classique
cleanmgr /sagerun:1         # Profil prédéfini
cleanmgr /setup             # Configuration profils

# Storage Sense (Windows 10/11)
# Paramètres > Système > Stockage > Assistant stockage

# Nettoyage Windows Update
DISM /Online /Cleanup-Image /StartComponentCleanup /ResetBase
```

### Nettoyage Manuel Important
| Répertoire | Contenu | Sécurité Suppression |
|------------|---------|---------------------|
| `C:\Windows\Temp\` | Fichiers temporaires | ✅ Sécurisé |
| `C:\Users\%USERNAME%\AppData\Local\Temp\` | Temp utilisateur | ✅ Sécurisé |
| `C:\Windows\SoftwareDistribution\Download\` | Cache Windows Update | ⚠️ Arrêter service Windows Update |
| `C:\Windows\Logs\` | Logs système | ⚠️ Garder récents pour debug |
| `C:\ProgramData\Microsoft\Windows Defender\Scans\History\` | Scans antivirus | ✅ Sécurisé |

### Outils Intégrés Nettoyage
```cmd
# Analyseur utilisation disque
StorageSense.exe            # Interface moderne (Win10+)
cleanmgr                    # Nettoyage disque classique

# Défragmentation
defrag C: /A                # Analyse fragmentation
defrag C: /O                # Défragmentation complète
defrag C: /X                # Consolidation espace libre

# Pour SSD (pas de défragmentation)
sfc /scannow               # Vérification intégrité uniquement
```

## ⚙️ Registre & Services

### Sauvegarde Registre
```cmd
# Exporter branches registre importantes
reg export HKLM\SOFTWARE C:\Backup\SOFTWARE.reg
reg export HKLM\SYSTEM C:\Backup\SYSTEM.reg
reg export HKCU C:\Backup\HKCU.reg

# Sauvegarde complète registre (admin requis)
reg save HKLM\SOFTWARE C:\Backup\SOFTWARE.hiv
reg save HKLM\SYSTEM C:\Backup\SYSTEM.hiv
```

### Gestion Services Problématiques
```cmd
# Identifier services problématiques
sc queryex type=service state=all | find "STOPPED"
net start                   # Services démarrés
services.msc               # Interface graphique

# Réparation services critiques
sfc /scannow               # Vérifie fichiers services
DISM /Online /Cleanup-Image /RestoreHealth
```

### Services Critiques à Surveiller
| Service | Nom | Criticité | Symptôme si Arrêté |
|---------|-----|-----------|-------------------|
| **wuauserv** | Windows Update | Élevée | Pas de mises à jour |
| **winmgmt** | WMI | Critique | Erreurs système, gestion impossible |
| **spooler** | Spouleur impression | Moyenne | Impossible imprimer |
| **themes** | Thèmes | Faible | Interface basique |
| **audiosrv** | Audio Windows | Moyenne | Pas de son |

## 🔍 Diagnostic Matériel

### Tests Mémoire RAM
```cmd
# Test mémoire Windows (nécessite redémarrage)
mdsched.exe                 # Planifier test mémoire au redémarrage

# Vérifier résultats test mémoire
eventvwr.msc                # Event Viewer
# Journaux Windows > Système > Rechercher "MemoryDiagnostics-Results"
```

### Tests Disque Dur
```cmd
# Check Disk (CHKDSK)
chkdsk C: /f               # Vérification + correction (nécessite redémarrage)
chkdsk C: /r               # Vérification + récupération secteurs défectueux
chkdsk C: /x               # Force démontage volume

# État santé disques
wmic diskdrive get status   # État général disques
fsutil dirty query C:       # Vérifier si volume marqué "dirty"
```

### Informations Système Détaillées
```cmd
# Informations matériel complètes
msinfo32                   # System Information (GUI)
systeminfo                # Command line system info
dxdiag                     # DirectX Diagnostic Tool

# Température et santé composants
wmic /namespace:\\root\wmi PATH MSAcpi_ThermalZoneTemperature get CurrentTemperature
```

## 🔐 Mode Sans Échec & Démarrage

### Accès Mode Sans Échec
```cmd
# Depuis Windows fonctionnel
msconfig                   # Configuration système > Boot > Safe Mode
bcdedit /set {default} safeboot minimal    # Mode sans échec via BCD

# Depuis WinRE
Startup Settings > Restart > F4 (Safe Mode) / F5 (Safe Mode with Networking)

# Désactiver mode sans échec
bcdedit /deletevalue {default} safeboot
```

### Paramètres Démarrage Avancés
```cmd
# Configuration démarrage
bcdedit /enum              # Lister configurations boot
bcdedit /set {default} bootmenupolicy legacy    # Activer F8 (Windows 8+)
bcdedit /timeout 30        # Délai sélection OS (secondes)

# Boot logging
bcdedit /set {default} bootlog yes    # Activer log démarrage
# Log généré : C:\Windows\ntbtlog.txt
```

## 🛡️ Outils Diagnostic Avancés

### Windows Performance Toolkit
```cmd
# Performance Monitor
perfmon.exe                # Monitoring performances temps réel
resmon.exe                 # Resource Monitor détaillé

# Event Viewer analysis
eventvwr.msc              # Visionneuse événements
# Journaux importants : Système, Application, Sécurité
```

### Outils Réseau & Connectivité
```cmd
# Tests réseau de base
ipconfig /all             # Configuration réseau complète
nslookup google.com       # Test DNS
ping -t 8.8.8.8          # Test connectivité continue
netsh winsock reset       # Reset stack réseau
netsh int ip reset        # Reset configuration IP
```

## 🔧 Commandes Maintenance Préventive

### Routine Maintenance Hebdomadaire
```cmd
# Séquence maintenance recommandée
sfc /scannow               # 1. Vérifier intégrité fichiers
DISM /Online /Cleanup-Image /RestoreHealth    # 2. Réparer image Windows
chkdsk C: /f              # 3. Vérifier disques (si nécessaire)
defrag C: /O              # 4. Défragmenter (HDD uniquement)
cleanmgr /sagerun:1       # 5. Nettoyage disques
```

### Monitoring Proactif
```cmd
# Vérifications régulières à scripter
wmic diskdrive get status,size,model    # État disques
systeminfo | find "Total Physical Memory"    # Mémoire système
tasklist /svc             # Services et processus
netstat -an | find "LISTENING"    # Ports ouverts
```

## 💡 Bonnes Pratiques

### Avant Intervention
- ✅ **Point de restauration** système obligatoire
- ✅ **Sauvegarde données** critiques utilisateur  
- ✅ **Export registre** sections modifiées
- ✅ **Liste programmes** installés (`wmic product get name,version`)
- ✅ **Documentation problème** (Event Viewer, symptômes)

### Après Réparation
- ✅ **Test fonctionnalités** essentielles
- ✅ **Vérification services** critiques
- ✅ **Windows Update** mise à jour
- ✅ **Scan antivirus** complet
- ✅ **Nettoyage fichiers** temporaires générés

### Maintenance Préventive
- ✅ **Mises à jour** automatiques activées
- ✅ **Points restauration** automatiques configurés
- ✅ **Nettoyage disque** mensuel planifié
- ✅ **Vérification intégrité** (`sfc /scannow`) mensuelle
- ✅ **Monitoring espace disque** (< 15% libre = problème)

---
**💡 Memo** : `sfc + DISM` pour corruption, `bootrec` pour démarrage, toujours sauvegarder avant intervention !