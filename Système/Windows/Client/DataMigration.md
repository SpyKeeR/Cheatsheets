# 💾 Migration de Données Windows — Aide-mémoire

## 🎯 Concepts Migration

### Types de Migration
| Type | Description | Usage | Complexité |
|------|-------------|-------|------------|
| **Side-by-side** | Ancien + nouveau système simultanés | Migration progressive | Faible |
| **Wipe-and-load** | Sauvegarde → formatage → restauration | Remplacement matériel | Moyenne |
| **In-place** | Mise à niveau sur même système | Windows 7→10, 10→11 | Élevée |
| **Refresh** | Réinstallation avec conservation données | Problèmes système | Moyenne |

### Éléments à Migrer
```
Données Utilisateur:
├── Profils utilisateurs (%USERPROFILE%)
├── Documents, Bureau, Images, Vidéos
├── Favoris navigateurs
├── Emails (PST, OST)
└── Certificats personnels

Configurations Système:
├── Paramètres applications
├── Registre Windows (branches utilisateur)
├── Polices personnalisées
├── Pilotes spécifiques
└── Licences logiciels

Paramètres Réseau:
├── Profils Wi-Fi
├── Imprimantes configurées
├── Lecteurs réseau mappés
└── Certificats d'authentification
```

## 🛠️ USMT (User State Migration Tool)

### Composants USMT
| Outil | Fonction | Usage |
|-------|----------|-------|
| **ScanState** | Capture données | Machine source |
| **LoadState** | Restaure données | Machine destination |
| **UsmtUtils** | Gestion store | Dépannage |

### Workflow USMT
```
1. Préparation
   ├── Installer Windows ADK sur machine admin
   ├── Identifier données à migrer
   └── Préparer espace stockage (réseau/local)

2. Capture (Machine Source)
   ├── Fermer applications utilisateur
   ├── Exécuter ScanState
   └── Vérifier intégrité store

3. Restauration (Machine Destination)
   ├── Installer OS + applications
   ├── Créer comptes utilisateurs
   ├── Exécuter LoadState
   └── Vérifier migration
```

### Commandes USMT Essentielles
```cmd
# Capture données utilisateur
scanstate.exe C:\MigrationStore /i:migdocs.xml /i:migapp.xml /v:13 /l:scanstate.log

# Capture avec compression et chiffrement
scanstate.exe C:\MigrationStore /i:migdocs.xml /i:migapp.xml /encrypt /compress /v:13

# Restauration données
loadstate.exe C:\MigrationStore /i:migdocs.xml /i:migapp.xml /v:13 /l:loadstate.log

# Restauration avec création comptes
loadstate.exe C:\MigrationStore /i:migdocs.xml /i:migapp.xml /lac /lae /v:13

# Validation store
usmtutils.exe /verify C:\MigrationStore

# Extraction contenu (dépannage)
usmtutils.exe /extract C:\MigrationStore C:\ExtractedData
```

### Paramètres USMT Importants
| Paramètre | Description | Exemple |
|-----------|-------------|---------|
| `/i:fichier.xml` | Fichier inclusion | `/i:migdocs.xml` |
| `/e:fichier.xml` | Fichier exclusion | `/e:migexclusions.xml` |
| `/v:niveau` | Verbosité logs (0-13) | `/v:13` (maximum) |
| `/l:logfile` | Fichier log | `/l:migration.log` |
| `/lac` | Créer comptes locaux | Loadstate uniquement |
| `/lae` | Activer comptes locaux | Loadstate uniquement |
| `/c` | Continuer malgré erreurs | Mode robuste |

### Fichiers XML USMT
```xml
<!-- Exemple migdocs.xml personnalisé -->
<migration urlid="http://www.microsoft.com/migration/1.0/migxmlext/migdocs">
  <component type="Documents" context="User">
    <displayName>Custom Documents</displayName>
    <role role="Data">
      <rules>
        <include>
          <objectSet>
            <pattern type="File">%CSIDL_PERSONAL%\*.docx</pattern>
            <pattern type="File">%CSIDL_PERSONAL%\*.xlsx</pattern>
            <pattern type="File">%CSIDL_DESKTOP%\*.pdf</pattern>
          </objectSet>
        </include>
        <exclude>
          <objectSet>
            <pattern type="File">%CSIDL_PERSONAL%\temp\*</pattern>
          </objectSet>
        </exclude>
      </rules>
    </role>
  </component>
</migration>
```

## 🗂️ Migration Manuelle

### Profils Utilisateurs
```cmd
# Sauvegarder profil utilisateur
robocopy "C:\Users\%USERNAME%" "\\server\backup\%USERNAME%" /MIR /XD AppData\Local\Temp /R:3

# Restaurer profil utilisateur
robocopy "\\server\backup\%USERNAME%" "C:\Users\%USERNAME%" /MIR /R:3

# Copier uniquement documents
robocopy "C:\Users\%USERNAME%\Documents" "\\server\backup\Documents" /MIR
robocopy "C:\Users\%USERNAME%\Desktop" "\\server\backup\Desktop" /MIR
```

### Registre Windows
```cmd
# Exporter branches registre utilisateur
reg export HKCU "C:\Backup\HKCU.reg"
reg export "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" "C:\Backup\Software.reg"

# Importer registre (attention!)
reg import "C:\Backup\HKCU.reg"

# Exporter clés spécifiques
reg export "HKCU\Software\Microsoft\Office" "C:\Backup\Office.reg"
reg export "HKCU\Software\Adobe" "C:\Backup\Adobe.reg"
```

### Données Applications
| Application | Localisation | Méthode Migration |
|-------------|--------------|-------------------|
| **Outlook** | `%APPDATA%\Microsoft\Outlook\*.pst` | Export/Import PST |
| **Chrome** | `%LOCALAPPDATA%\Google\Chrome\User Data` | Sync Google ou copie |
| **Firefox** | `%APPDATA%\Mozilla\Firefox\Profiles` | Firefox Sync ou copie |
| **Thunderbird** | `%APPDATA%\Thunderbird\Profiles` | Copie profil complet |
| **Office** | Registre + `%APPDATA%\Microsoft\Office` | Export reg + copie |

## 📦 Images Windows

### Types d'Images
```
Images de Déploiement:
├── install.wim/install.esd : Éditions Windows
├── boot.wim : Environnement préinstallation (WinPE)
├── winre.wim : Environnement récupération (WinRE)
└── Custom WIM : Image personnalisée (Sysprep)

Formats Fichiers:
├── WIM : Windows Imaging Format (non compressé)
├── ESD : Electronic Software Distribution (ultra-compressé)
└── VHD/VHDX : Virtual Hard Disk (bootable)
```

### Gestion Images DISM
```cmd
# Informations images
dism /get-wiminfo /wimfile:install.wim

# Monter image pour modification
dism /mount-wim /wimfile:install.wim /index:1 /mountdir:C:\Mount

# Ajouter pilotes à image
dism /image:C:\Mount /add-driver /driver:C:\Drivers /recurse

# Installer mises à jour
dism /image:C:\Mount /add-package /packagepath:C:\Updates\update.msu

# Valider et démonter
dism /unmount-wim /mountdir:C:\Mount /commit

# Capturer image personnalisée
dism /capture-image /imagefile:C:\CustomImage.wim /capturedir:C:\ /name:"Custom Windows"
```

## 💼 Migration Entreprise

### Stratégies Déploiement
```
Méthodes Déploiement:
├── Lite Touch (MDT) : Semi-automatique
├── Zero Touch (SCCM) : Entièrement automatique  
├── Modern Deployment : Autopilot + Intune
└── Cloud-based : Azure Virtual Desktop
```

### Active Directory & Domaine
```powershell
# Informations domaine utilisateur
Get-ADUser $env:USERNAME -Properties *

# Export utilisateurs domaine
Get-ADUser -Filter * -Properties * | Export-Csv "C:\ADUsers.csv"

# Migration certificats utilisateur
certlm.msc                    # Certificats machine locale
certmgr.msc                   # Certificats utilisateur courant

# Export certificat personnel
Get-ChildItem Cert:\CurrentUser\My | Export-Certificate -FilePath "C:\UserCerts\"
```

### Migration Exchange/Office 365
```powershell
# Export PST Outlook
New-MailboxExportRequest -Mailbox "user@domain.com" -FilePath "\\server\pst\user.pst"

# Migration vers Office 365
Connect-ExchangeOnline
New-MigrationBatch -Name "UserMigration" -SourceEndpoint $Endpoint -TargetDeliveryDomain "domain.mail.onmicrosoft.com"

# Synchronisation OneDrive
# Via OneDrive client sync ou SharePoint Migration Tool
```

## 🔧 Outils Migration Tiers

### Solutions Commerciales
| Outil | Éditeur | Forces | Usage |
|-------|---------|--------|-------|
| **PCmover** | Laplink | Interface simple | Particuliers/PME |
| **Zinstall** | Zinstall | Migration complète | Migration OS différents |
| **Transwiz** | ForensIT | Profils utilisateurs | Domaines AD |
| **User Profile Wizard** | ForensIT | Gestion profils | Entreprise |

### Solutions Gratuites
- **Windows Easy Transfer** : Legacy (Windows 7/8)
- **Robocopy** : Copie fichiers robuste (intégré Windows)
- **7-Zip** : Compression/archivage
- **Clonezilla** : Clonage disque complet

## 🚨 Problèmes Courants & Solutions

### Erreurs USMT Fréquentes
| Erreur | Cause | Solution |
|--------|-------|----------|
| **0x80070005** | Permissions insuffisantes | Exécuter en admin |
| **0x8007000E** | Mémoire insuffisante | Libérer RAM, /compress |
| **0x80070070** | Espace disque | Vérifier espace destination |
| **Migration partielle** | Profil non standard | Personnaliser XML |

### Validation Post-Migration
```cmd
# Vérifier intégrité profils
whoami /groups                # Groupes utilisateur
gpresult /r                   # Stratégies appliquées

# Test applications critiques
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall"

# Vérifier services
sc query state=all | find "RUNNING"

# Test connectivité réseau
ipconfig /all
ping domain.com
nslookup domain.com
```

### Recovery & Rollback
```cmd
# Sauvegarde avant migration
wbadmin start backup -backupTarget:E: -include:C: -allCritical -quiet

# Point de restauration
wmic.exe /Namespace:\\root\default Path SystemRestore Call CreateRestorePoint "Pre-Migration", 100, 12

# Restauration USMT (si problème)
loadstate.exe C:\MigrationStore /i:migdocs.xml /i:migapp.xml /lac /v:13 /c
```

## 📋 Checklist Migration

### Pré-Migration
- [ ] Inventaire matériel/logiciel source
- [ ] Vérification compatibilité destination
- [ ] Sauvegarde complète système source
- [ ] Test plan migration (environnement pilote)
- [ ] Communication utilisateurs
- [ ] Planification fenêtre maintenance

### Migration
- [ ] Fermeture applications utilisateur
- [ ] Capture données (USMT/manuel)
- [ ] Vérification intégrité store
- [ ] Installation OS destination
- [ ] Installation applications requises
- [ ] Restauration données utilisateur
- [ ] Configuration réseau/domaine

### Post-Migration
- [ ] Test fonctionnalités critiques
- [ ] Validation accès ressources réseau
- [ ] Formation utilisateur si nécessaire
- [ ] Documentation changements
- [ ] Nettoyage fichiers temporaires
- [ ] Archivage données migration

## 💡 Bonnes Pratiques

### Planification
- ✅ **Inventaire complet** : Hardware, software, données
- ✅ **Tests pilotes** : Validation sur échantillon
- ✅ **Documentation** : Procédures détaillées
- ✅ **Formation équipe** : Outils et procédures
- ✅ **Communication** : Planning utilisateurs

### Exécution
- ✅ **Sauvegarde préalable** : Point retour garanti
- ✅ **Validation étapes** : Vérification avant suite
- ✅ **Logs détaillés** : Traçabilité complète
- ✅ **Tests post-migration** : Fonctionnalités critiques
- ✅ **Support utilisateur** : Accompagnement changement

### Sécurité
- ✅ **Données sensibles** : Chiffrement migration store
- ✅ **Accès restreint** : Permissions minimales
- ✅ **Audit trail** : Journalisation accès
- ✅ **Nettoyage** : Suppression données temporaires
- ✅ **Conformité** : Respect réglementations données

---
**💡 Memo** : USMT = ScanState (capture) + LoadState (restore), toujours sauvegarder avant migration, tester sur pilote !
