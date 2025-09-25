# 💻 Command Prompt (cmd.exe) — Aide-mémoire

## 🏗️ Concepts Fondamentaux

### Philosophy & Limitations
- **Héritage DOS** : Compatibilité rétrograde avec MS-DOS
- **Shells modernes** : PowerShell recommandé pour tâches avancées
- **Encoding** : CP-850/CP-1252 par défaut (problèmes UTF-8)
- **Limitations** : Pas d'objets, manipulation texte basique

### Syntaxe & Conventions
```cmd
commande [/option] [paramètre] [cible]

# Conventions aide
[] = Optionnel
{} = Requis  
| = OU exclusif
... = Répétition possible
```

### Élévation Privilèges
| Méthode | Usage | UAC Bypass |
|---------|-------|------------|
| **Clic droit > Exécuter en tant qu'admin** | GUI | ✅ Prompt UAC |
| **runas** | `runas /user:DOMAIN\admin cmd` | ❌ Change utilisateur |
| **psexec** | `psexec -s cmd` | ⚠️ Système (dangereux) |

## 📁 Navigation & Fichiers

### Déplacement & Exploration
```cmd
# Navigation
cd [chemin]                 # Changer répertoire
cd ..                       # Remonter niveau parent
cd \                        # Racine lecteur courant
cd /d D:\path               # Changer lecteur + répertoire
pushd \\server\share        # Empiler + changer (UNC supporté)
popd                        # Revenir répertoire empilé

# Exploration
dir                         # Listing basique
dir /a                      # Tous fichiers (cachés inclus)
dir /s                      # Récursif sous-dossiers
dir /b                      # Format simple (noms seulement)
dir /o:d                    # Trier par date (:n=nom, :s=taille)
dir *.txt /s | find /c /v ""  # Compter fichiers .txt récursif
tree /f                     # Arborescence avec fichiers
```

### Manipulation Fichiers/Dossiers
```cmd
# Création
mkdir dossier              # Créer répertoire
mkdir "avec espaces"       # Guillemets si espaces
echo. > fichier.txt        # Créer fichier vide
type nul > fichier.txt     # Alternative fichier vide

# Copie simple
copy source.txt dest.txt   # Fichier unique
copy *.log D:\backup\      # Plusieurs fichiers (même dossier)
copy /y source dest        # Écraser sans confirmation

# Déplacement/Renommage
move ancien.txt nouveau.txt # Renommer
move *.bak D:\archive\     # Déplacer plusieurs fichiers
ren ancien.ext nouveau.ext # Renommer (syntaxe courte)

# Suppression
del fichier.txt            # Supprimer fichier
del /f /q *.tmp            # Force + silencieux
rmdir /s /q dossier        # Supprimer dossier + contenu (⚠️)
```

## 🔄 Copie Robuste & Synchronisation

### XCOPY (Legacy mais utile)
```cmd
# Syntaxe complète recommandée
xcopy source dest /E /I /H /K /Y /C
```

| Option | Description | Usage |
|--------|-------------|-------|
| `/E` | Copie sous-dossiers (y compris vides) | vs `/S` (ignore vides) |
| `/I` | Destination = dossier (pas d'ambiguïté) | Évite questions |
| `/H` | Copie fichiers cachés/système | Sauvegarde complète |
| `/K` | Conserve attributs lecture seule | Préservation métadonnées |
| `/Y` | Écrase sans confirmation | Automation |
| `/C` | Continue malgré erreurs | Robustesse |

### ROBOCOPY (Moderne & Recommandé)
```cmd
# Syntaxe de base
robocopy source dest [fichiers] [options]

# Mirror (synchronisation exacte) ⚠️
robocopy C:\Source D:\Backup /MIR /Z /R:3 /W:5 /MT:8 /LOG+:backup.log

# Copie sélective (recommandée)
robocopy C:\Data D:\Backup *.* /E /COPY:DATS /R:3 /W:10 /LOG+:copy.log
```

#### Options ROBOCOPY Essentielles
| Option | Description | Recommandé Pour |
|--------|-------------|-----------------|
| `/MIR` | Mirror (⚠️ supprime différences) | Synchronisation exacte |
| `/E` | Copie sous-dossiers (sans suppression) | Sauvegarde sécurisée |
| `/Z` | Mode restartable (gros fichiers) | Réseau instable |
| `/MT[:n]` | Multi-thread (défaut 8) | Performance |
| `/R:n` | Tentatives si échec (défaut 1M) | Robustesse |
| `/W:n` | Attente entre tentatives (défaut 30s) | Réseau surchargé |
| `/COPY:flags` | DATS=Données+Attributs+Timestamps+Sécurité | Préservation complète |
| `/XO` | Exclude Older | Sync incrémentale |
| `/FFT` | Fat File Times (NAS) | Compatibilité |

## 🔧 Diagnostics Système

### Vérification Intégrité
```cmd
# Vérification disques
chkdsk C:                   # Lecture seule
chkdsk C: /f               # Correction erreurs (redémarrage requis)
chkdsk C: /r               # Récupération secteurs défectueux (long)
chkdsk C: /x               # Démonte volume au préalable

# Intégrité Windows
sfc /scannow               # System File Checker (30min~)
sfc /verifyonly            # Vérification sans réparation
sfc /scanfile=C:\Windows\System32\kernel32.dll  # Fichier spécifique

# Réparation image Windows
DISM /Online /Cleanup-Image /CheckHealth      # Vérification rapide
DISM /Online /Cleanup-Image /ScanHealth       # Analyse approfondie
DISM /Online /Cleanup-Image /RestoreHealth    # Réparation (Internet requis)
```

### Informations Système
```cmd
# Vue d'ensemble
systeminfo                 # Résumé complet système
systeminfo | find "Total Physical Memory"  # Info spécifique

# Composants détaillés
wmic computersystem get manufacturer,model,systemtype
wmic bios get serialnumber,version
wmic logicaldisk get caption,size,freespace,filesystem
wmic os get caption,version,buildnumber,lastbootuptime

# Performance & utilisation
tasklist /svc              # Processus + services associés
tasklist /m                # Modules DLL chargés
wmic process get name,processid,commandline  # Ligne commande processus
```

## 🛠️ Services & Processus

### Gestion Processus
```cmd
# Consultation
tasklist                   # Tous processus
tasklist /fi "imagename eq notepad.exe"  # Filtrage
tasklist /svc              # Services par processus
tasklist /m kernel32.dll   # Processus utilisant DLL spécifique

# Terminaison
taskkill /pid 1234 /f      # Par PID (force)
taskkill /im notepad.exe /f # Par nom image
taskkill /fi "memusage gt 500000" /f  # Par critère (>500MB)
```

### Gestion Services
```cmd
# Service Controller (sc)
sc query                   # Tous services
sc query spooler           # Service spécifique
sc qc spooler              # Configuration service
sc start/stop/pause spooler # Actions de base

# Net commands (plus simple)
net start                  # Services démarrés
net start "Print Spooler"  # Démarrer service (nom complet)
net stop spooler           # Arrêter service (nom court)

# Informations avancées
sc queryex spooler         # État détaillé + PID
sc config spooler start=disabled  # Modifier type démarrage
```

### Planificateur de Tâches (schtasks)
```cmd
# Création tâche
schtasks /create /tn "BackupDaily" /tr "C:\scripts\backup.bat" /sc daily /st 02:00 /ru SYSTEM

# Gestion
schtasks /query /tn "BackupDaily" /fo table /v  # Détails tâche
schtasks /run /tn "BackupDaily"                 # Exécution immédiate
schtasks /delete /tn "BackupDaily" /f           # Suppression
```

#### Options Planificateur
| Option | Description | Exemples |
|--------|-------------|----------|
| `/SC` | Planification | DAILY, WEEKLY, MONTHLY, ONCE, ONSTART |
| `/ST` | Heure début | 14:30 (format 24h) |
| `/RU` | Compte exécution | SYSTEM, DOMAIN\user |
| `/RL` | Niveau privilèges | LIMITED, HIGHEST |
| `/F` | Force écrasement | Remplace tâche existante |

## 🌐 Réseau & Connectivité

### Configuration IP
```cmd
# Consultation
ipconfig                   # IP basique interfaces actives
ipconfig /all              # Configuration complète
ipconfig /displaydns       # Cache DNS local

# DHCP
ipconfig /release          # Libérer bail DHCP
ipconfig /renew            # Renouveler bail DHCP
ipconfig /registerdns      # Renouveler enregistrements DNS

# DNS
ipconfig /flushdns         # Vider cache DNS
nslookup google.com        # Requête DNS simple
nslookup google.com 8.8.8.8 # Serveur DNS spécifique
```

### Tests Connectivité
```cmd
# Ping avancé
ping google.com            # Test basique
ping -n 10 google.com      # Nombre paquets spécifié
ping -l 1500 google.com    # Taille paquet (MTU test)
ping -t google.com         # Continu (Ctrl+C pour arrêter)

# Traceroute
tracert google.com         # Route complète
tracert -h 15 google.com   # Maximum 15 sauts

# Connexions réseau
netstat -an               # Toutes connexions (numérique)
netstat -ano              # Avec PID processus
netstat -r                # Table routage
netstat -e                # Statistiques Ethernet
```

### Gestion Routes & ARP
```cmd
# Table routage
route print               # Afficher routes
route add 10.0.0.0 mask 255.0.0.0 192.168.1.1  # Ajouter route
route delete 10.0.0.0    # Supprimer route

# ARP (Address Resolution)
arp -a                    # Table ARP complète
arp -a 192.168.1.1        # Entrée spécifique
arp -d 192.168.1.1        # Supprimer entrée
arp -s 192.168.1.1 aa-bb-cc-dd-ee-ff  # Entrée statique
```

## 🔐 Sécurité & Permissions

### Gestion Comptes Locaux
```cmd
# Consultation
net user                  # Tous comptes locaux
net user administrator    # Détails compte spécifique
net localgroup administrators # Membres groupe admin

# Modification
net user newuser password123 /add  # Créer compte
net user testuser /active:no       # Désactiver compte
net user testuser /delete          # Supprimer compte
net localgroup administrators newuser /add  # Ajouter au groupe
```

### Permissions Fichiers (NTFS)
```cmd
# Prise possession
takeown /f "C:\folder" /r /d y    # Récursif sur dossier

# ICACLS (remplacement cacls)
icacls "C:\folder"                # Afficher permissions actuelles
icacls "C:\folder" /grant "Users:(R)"    # Accorder lecture
icacls "C:\folder" /grant "Admins:(F)"   # Accès complet
icacls "C:\folder" /remove "Users"       # Supprimer permissions
icacls "C:\folder" /reset /t             # Réinitialiser (héritage)
```

#### Codes Permissions ICACLS
| Code | Permission | Description |
|------|------------|-------------|
| `F` | Full control | Contrôle total |
| `M` | Modify | Modification |
| `RX` | Read & execute | Lecture + exécution |
| `R` | Read | Lecture seule |
| `W` | Write | Écriture |
| `D` | Delete | Suppression |

## 💾 Redirections & Pipes

### Flux Standard
| Stream | Numéro | Description | Redirection |
|--------|--------|-------------|-------------|
| **STDIN** | 0 | Entrée standard | `< fichier` |
| **STDOUT** | 1 | Sortie standard | `> fichier` |
| **STDERR** | 2 | Erreur standard | `2> fichier` |

### Redirections Courantes
```cmd
# Sortie
commande > fichier.txt     # Redirection (écrase)
commande >> fichier.txt    # Ajout au fichier
commande > nul             # Suppression sortie

# Erreurs
commande 2> errors.txt     # Erreurs vers fichier
commande 2>&1             # Erreurs vers sortie standard
commande > output.txt 2>&1 # Tout vers même fichier

# Pipes & filtres
dir | find ".txt"         # Filtrer résultats
tasklist | findstr notepad # Rechercher processus
type fichier.log | more    # Pagination
```

### Outils de Filtrage
```cmd
# FIND (recherche simple)
find "error" logfile.txt   # Recherche chaîne
find /c "warning" *.log    # Compter occurrences
find /v "debug" app.log    # Inverser (exclure lignes)

# FINDSTR (regex basique)
findstr /i "error" *.log   # Insensible casse
findstr /r "^[0-9]" data.txt # Regex: lignes commençant par chiffre
findstr /b "INFO" app.log  # Début de ligne

# SORT & MORE
sort data.txt             # Tri alphabétique
sort /r data.txt          # Tri inverse
type bigfile.txt | more   # Pagination
```

## 📋 Variables & Environnement

### Variables Session
```cmd
# Définition temporaire
set VAR=valeur            # Session courante
set PATH=%PATH%;C:\tools  # Ajouter au PATH

# Consultation
set                       # Toutes variables
set PATH                  # Variable spécifique
echo %VAR%                # Utiliser variable

# Variables persistantes (utilisateur)
setx VAR "valeur"         # Nécessite nouvelle session
setx PATH "%PATH%;C:\tools" /M  # Machine (admin requis)
```

### Variables Système Importantes
| Variable | Description | Exemple |
|----------|-------------|---------|
| `%USERPROFILE%` | Profil utilisateur | `C:\Users\john` |
| `%APPDATA%` | Données application | `C:\Users\john\AppData\Roaming` |
| `%TEMP%` | Dossier temporaire | `C:\Users\john\AppData\Local\Temp` |
| `%WINDIR%` | Dossier Windows | `C:\Windows` |
| `%PROGRAMFILES%` | Program Files | `C:\Program Files` |
| `%COMPUTERNAME%` | Nom ordinateur | `PC-JOHN` |

## 🔧 Outils Système Avancés

### Chiffrement & Sécurité
```cmd
# EFS (Encrypting File System)
cipher /e /s:C:\folder    # Chiffrer dossier
cipher /d /s:C:\folder    # Déchiffrer
cipher /w:C:\             # Effacement sécurisé espace libre

# Checksums & hash
certutil -hashfile file.exe SHA256  # Hash SHA-256
certutil -hashfile file.exe MD5     # Hash MD5 (legacy)
fc /b file1.exe file2.exe          # Comparaison binaire fichiers
```

### Registre Windows
```cmd
# Sauvegarde/restauration
reg export HKLM\SOFTWARE\MyApp backup.reg  # Exporter clé
reg import backup.reg                       # Importer sauvegarde

# Modification
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion /v ProgramFilesDir
reg add HKLM\SOFTWARE\MyApp /v Setting /t REG_DWORD /d 1
reg delete HKLM\SOFTWARE\MyApp /v OldSetting /f
```

### Partages Réseau
```cmd
# Montage/démontage
net use Z: \\server\share /user:domain\user  # Monter lecteur
net use Z: /delete                           # Démonter
net use * /delete /y                         # Tout démonter

# Gestion partages locaux
net share                          # Lister partages
net share MyShare=C:\folder /grant:everyone,read  # Créer partage
net share MyShare /delete          # Supprimer partage
```

## 🔄 Maintenance & Nettoyage

### Forfiles (Gestion fichiers par date)
```cmd
# Syntaxe de base
forfiles /p C:\Logs /s /m *.log /d -30 /c "cmd /c del @path"

# Exemples pratiques
forfiles /p C:\Temp /m *.tmp /d -7 /c "cmd /c echo Suppression: @path & del @path"
forfiles /p C:\Backup /m *.bak /d -90 /c "cmd /c move @path C:\Archive\"
```

#### Variables Forfiles
| Variable | Description | Exemple |
|----------|-------------|---------|
| `@path` | Chemin complet | `C:\logs\app.log` |
| `@file` | Nom avec extension | `app.log` |
| `@fname` | Nom sans extension | `app` |
| `@ext` | Extension | `.log` |
| `@fsize` | Taille fichier | `1024` |
| `@fdate` | Date modification | `01/15/2024` |

### Tâches Maintenance
```cmd
# Nettoyage système
cleanmgr /sagerun:1               # Nettoyage disque (profil 1)
sfc /scannow                      # Vérification intégrité
DISM /Online /Cleanup-Image /StartComponentCleanup  # Nettoyage composants

# Défragmentation (HDD uniquement)
defrag C: /a                      # Analyser fragmentation
defrag C: /o                      # Optimiser (défragmenter)
defrag C: /x                      # Consolidation espace libre
```

## 💡 Astuces & Raccourcis

### Raccourcis Clavier CMD
| Raccourci | Action | Usage |
|-----------|--------|-------|
| `↑` `↓` | Historique commandes | Navigation rapide |
| `Tab` | Auto-complétion | Chemins, fichiers |
| `Ctrl+C` | Interruption | Arrêt commande en cours |
| `F7` | Historique popup | Sélection visuelle |
| `F8` | Recherche historique | Frappe partielle + F8 |
| `Alt+Enter` | Plein écran | Mode console étendu |

### Trucs & Astuces
```cmd
# Temps d'exécution commande
echo %time% & ping google.com -n 1 > nul & echo %time%

# Boucle simple
for /L %i in (1,1,10) do echo Iteration %i

# Test existence avant action
if exist "C:\file.txt" del "C:\file.txt" & echo File deleted

# Pause avec message
echo Appuyez sur une touche pour continuer... & pause > nul

# Variables dans noms fichiers
set backupname=backup_%date:~-4,4%%date:~-7,2%%date:~-10,2%
echo %backupname%
```

## ⚠️ Limitations & Considérations

### Quand Éviter CMD
- ✅ **PowerShell** : Objets, remoting, gestion moderne
- ✅ **PowerShell Core** : Cross-platform, performance
- ✅ **WSL/Bash** : Outils Unix/Linux natifs
- ⚠️ **CMD** : Compatibilité legacy uniquement

### Pièges Courants
```cmd
# Problème: Espaces dans chemins
copy C:\Program Files\file.txt D:\  # ❌ ERREUR
copy "C:\Program Files\file.txt" D:\ # ✅ CORRECT

# Problème: Variables dans boucles
for %f in (*.txt) do set count=%count%+1  # ❌ Pas incrémenté
setlocal EnableDelayedExpansion            # ✅ Solution
for %f in (*.txt) do set /a count+=1

# Problème: Encoding
type utf8file.txt  # ❌ Caractères corrompus
powershell Get-Content utf8file.txt -Encoding UTF8  # ✅ Solution
```

---
**💡 Memo** : CMD = compatibilité legacy, PowerShell = moderne ! Toujours tester commandes destructives (`/MIR`, `rmdir /s`) !

