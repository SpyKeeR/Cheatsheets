# 👥 Gestion Utilisateurs & Groupes Locaux — Aide-mémoire

## 🏗️ Concepts Fondamentaux

### Identité & Authentification
- **SID** (Security Identifier) : Identifiant unique utilisateur/groupe
- **Jeton d'accès** : Identité + appartenances groupes + privilèges
- **SAM** (Security Account Manager) : Base locale comptes Windows
- **LSA** (Local Security Authority) : Gestion authentification locale

### Architecture Sécurité
```
Utilisateur → Authentification → Jeton d'accès → Contrôle accès
     ↓              ↓                ↓              ↓
   Login      Vérif mdp        SID + groupes   ACL + droits
```

## 🔐 Types de Comptes

### Comptes Utilisateurs
| Type | Description | Usage | Localisation |
|------|-------------|-------|--------------|
| **Administrateur** | Compte admin intégré | Administration système | Toujours présent |
| **Invité** | Accès limité (désactivé par défaut) | Accès temporaire | Contrôlé par GPO |
| **Utilisateur standard** | Droits limités | Usage quotidien | Créé manuellement |
| **Compte de service** | Applications/services | Processus automatiques | Local ou domaine |

### États de Compte
```
Actif    : Peut se connecter
Désactivé : Connexion interdite (compte existe)
Verrouillé : Trop de tentatives échouées
Expiré   : Date d'expiration dépassée
```

## 👥 Groupes Locaux Windows

### Groupes Intégrés (Built-in)
| Groupe | SID | Description | Permissions Typiques |
|--------|-----|-------------|---------------------|
| **Administrators** | S-1-5-32-544 | Contrôle total système | Tout |
| **Users** | S-1-5-32-545 | Utilisateurs standards | Lecture, exécution |
| **Guests** | S-1-5-32-546 | Accès invité limité | Très restrictif |
| **Power Users** | S-1-5-32-547 | Legacy (Vista+) | Obsolète |
| **Backup Operators** | S-1-5-32-551 | Sauvegarde/restauration | SeBackupPrivilege |
| **Remote Desktop Users** | S-1-5-32-555 | Connexion RDP | Droit de connexion distante |

### Groupes Spécialisés
```
Event Log Readers (S-1-5-32-573)     # Lecture journaux événements
Performance Log Users (S-1-5-32-559)  # Compteurs performance
IIS_IUSRS (S-1-5-32-568)             # IIS worker processes
Network Configuration Operators       # Configuration réseau
```

### Identités Spéciales
| Identité | Description | Usage |
|----------|-------------|-------|
| **Everyone** | Tous utilisateurs authentifiés | ACL universelles |
| **Authenticated Users** | Utilisateurs authentifiés (pas Anonymous) | Sécurité renforcée |
| **Interactive** | Connexion locale | Distinguer local/réseau |
| **Network** | Connexion réseau | Accès partagés |
| **System** | Compte système local | Services Windows |

## 🛠️ Gestion via Interface Graphique

### MMC Users & Groups (lusrmgr.msc)
```
Accès :
├── Win+R → lusrmgr.msc
├── Computer Management → Local Users and Groups
└── Server Manager → Tools → Computer Management

Fonctionnalités :
├── Créer/modifier/supprimer utilisateurs
├── Gérer appartenances groupes
├── Définir propriétés avancées
└── Configurer profils utilisateurs
```

### Propriétés Utilisateur Avancées
- **Général** : Nom complet, description, mot de passe
- **Membre de** : Appartenances groupes
- **Profil** : Chemin profil, script ouverture, dossier personnel
- **Sessions** : Restrictions horaires, déconnexion automatique
- **Contrôle d'accès** : Connexion à distance, services

## 💻 Gestion Ligne de Commande

### Commandes NET (Legacy)
```cmd
# Utilisateurs
net user                              # Lister utilisateurs
net user john                         # Détails utilisateur
net user john Password123 /add        # Créer utilisateur
net user john /delete                 # Supprimer utilisateur
net user john /active:no              # Désactiver compte
net user john * /add                  # Mot de passe interactif

# Groupes
net localgroup                        # Lister groupes
net localgroup Administrators         # Membres groupe
net localgroup Administrators john /add    # Ajouter à groupe
net localgroup Administrators john /delete # Retirer du groupe
net localgroup "Backup Operators" john /add # Nom avec espaces
```

### Options Utilisateur NET
```cmd
# Options courantes
/add                    # Créer compte
/delete                 # Supprimer compte
/active:yes|no         # Activer/désactiver
/expires:mm/dd/yyyy    # Date d'expiration
/passwordchg:yes|no    # Peut changer mot de passe
/passwordreq:yes|no    # Mot de passe requis
/fullname:"Nom Complet" # Nom complet
/comment:"Description"  # Description
/homedir:C:\Users\john # Dossier personnel
```

## ⚡ PowerShell (Moderne)

### Gestion Utilisateurs
```powershell
# Consultation
Get-LocalUser                         # Tous utilisateurs
Get-LocalUser -Name "john"           # Utilisateur spécifique
Get-LocalUser | Where-Object Enabled -eq $false # Comptes désactivés

# Création
$password = ConvertTo-SecureString "Password123" -AsPlainText -Force
New-LocalUser -Name "john" -Password $password -FullName "John Doe" -Description "Utilisateur standard"

# Modification
Set-LocalUser -Name "john" -Description "Nouvelle description"
Set-LocalUser -Name "john" -Password $newPassword
Set-LocalUser -Name "john" -AccountExpires (Get-Date).AddDays(30)

# Suppression
Remove-LocalUser -Name "john"
```

### Gestion Groupes
```powershell
# Consultation
Get-LocalGroup                        # Tous groupes
Get-LocalGroupMember -Group "Administrators" # Membres groupe
Get-LocalUser -Name "john" | Get-LocalGroupMembership # Groupes d'un utilisateur

# Gestion appartenances
Add-LocalGroupMember -Group "Administrators" -Member "john"
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "john","jane" # Plusieurs
Remove-LocalGroupMember -Group "Users" -Member "john"

# Création groupe personnalisé
New-LocalGroup -Name "AppAdmins" -Description "Administrateurs application"
```

### Requêtes Avancées
```powershell
# Utilisateurs avec propriétés étendues
Get-LocalUser | Select-Object Name, Enabled, LastLogon, PasswordLastSet, PasswordExpires

# Comptes inactifs (pas de connexion récente)
Get-LocalUser | Where-Object { $_.LastLogon -lt (Get-Date).AddDays(-30) -and $_.Enabled -eq $true }

# Mots de passe expirant bientôt
Get-LocalUser | Where-Object { $_.PasswordExpires -lt (Get-Date).AddDays(7) -and $_.PasswordExpires -ne $null }

# Appartenance multiple
Get-LocalUser | ForEach-Object {
    [PSCustomObject]@{
        User = $_.Name
        Groups = (Get-LocalGroupMembership -User $_.Name).Name -join ", "
    }
}
```

## 🗂️ Profils Utilisateurs

### Types de Profils
| Type | Description | Emplacement | Usage |
|------|-------------|-------------|-------|
| **Local** | Stocké sur machine | `C:\Users\username` | Poste fixe |
| **Itinérant** | Stocké sur serveur | Réseau (UNC) | Mobilité |
| **Obligatoire** | Lecture seule | `.man` extension | Standardisation |
| **Temporaire** | RAM/temporaire | Supprimé à déconnexion | Invités |

### Gestion Profils
```powershell
# Lister profils
Get-WmiObject -Class Win32_UserProfile | Select-Object LocalPath, SID, Loaded

# Supprimer profil utilisateur
Get-WmiObject -Class Win32_UserProfile | Where-Object { $_.LocalPath -like "*username*" } | Remove-WmiObject

# Profil par défaut (modèle nouveaux utilisateurs)
# Localisation : C:\Users\Default
# Modifications appliquées aux nouveaux comptes
```

### Dossiers Profil Importants
```
C:\Users\username\
├── Desktop\              # Bureau utilisateur
├── Documents\            # Documents personnels
├── Downloads\            # Téléchargements
├── AppData\
│   ├── Local\           # Données applications locales
│   ├── LocalLow\        # Données basse intégrité
│   └── Roaming\         # Données itinérantes
└── NTUSER.DAT           # Ruche registre utilisateur
```

## 🔑 Droits & Privilèges

### User Rights Assignment
```
Privilèges système importants :
├── SeServiceLogonRight        # Ouvrir session en tant que service
├── SeBatchLogonRight         # Ouvrir session en lot
├── SeNetworkLogonRight       # Accès réseau
├── SeRemoteInteractiveLogonRight # Bureau à distance
├── SeBackupPrivilege         # Sauvegarder fichiers/répertoires
├── SeRestorePrivilege        # Restaurer fichiers/répertoires
├── SeShutdownPrivilege       # Arrêter système
└── SeSystemtimePrivilege     # Modifier heure système
```

### Vérification Droits
```powershell
# Droits utilisateur courant
whoami /priv                          # Privilèges
whoami /groups                        # Groupes
whoami /user                          # SID utilisateur

# Via PowerShell
[System.Security.Principal.WindowsIdentity]::GetCurrent().Groups
```

## 🔐 Sécurité & Bonnes Pratiques

### Politique Mots de Passe
```cmd
# Consultation politique locale
net accounts                          # Politique mots de passe locale

# Paramètres typiques :
# - Longueur minimale : 8 caractères minimum
# - Complexité : Majuscules + minuscules + chiffres + symboles
# - Durée maximale : 90 jours maximum
# - Historique : 12 derniers mots de passe
# - Verrouillage : 3 tentatives, 30 min verrouillage
```

### Audit & Monitoring
```powershell
# Événements sécurité importants
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624} | Select-Object -First 10 # Connexions réussies
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} | Select-Object -First 10 # Échecs connexion
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4732} | Select-Object -First 10 # Ajout membre groupe

# Dernières connexions
Get-LocalUser | Select-Object Name, LastLogon, PasswordLastSet | Sort-Object LastLogon -Descending
```

### Nettoyage & Maintenance
```powershell
# Comptes inactifs
Get-LocalUser | Where-Object { 
    $_.LastLogon -lt (Get-Date).AddDays(-90) -and 
    $_.Enabled -eq $true -and 
    $_.Name -notin @('Administrator', 'DefaultAccount', 'Guest')
}

# Profils orphelins (utilisateur supprimé mais profil reste)
Get-WmiObject Win32_UserProfile | Where-Object { 
    $_.Special -eq $false -and 
    !(Get-LocalUser -SID $_.SID -ErrorAction SilentlyContinue)
}
```

## 🛡️ Comptes de Service

### Types de Comptes Service
| Type | Description | Mot de Passe | Usage |
|------|-------------|--------------|-------|
| **LocalSystem** | Système local (privilèges élevés) | N/A | Services système |
| **NetworkService** | Service réseau | N/A | Services IIS, SQL |
| **LocalService** | Service local | N/A | Services locaux |
| **Utilisateur dédié** | Compte personnalisé | Géré | Applications métier |
| **MSA/gMSA** | Managed Service Account | Auto-géré | Recommandé domaine |

### Configuration Service
```powershell
# Lister services et comptes
Get-WmiObject Win32_Service | Select-Object Name, StartName, State | Format-Table

# Modifier compte service
Set-Service -Name "MonService" -Credential (Get-Credential)

# Service avec compte système
sc.exe config "MonService" obj= "LocalSystem"
```

## 💡 Bonnes Pratiques

### Sécurité
- ✅ **Principe moindre privilège** : Droits minimaux nécessaires
- ✅ **Comptes dédiés** : Un compte par fonction/application
- ✅ **Désactiver** comptes inutilisés plutôt que supprimer
- ✅ **Audit régulier** : Vérifier appartenances groupes
- ✅ **Rotation mots de passe** : Comptes de service critiques

### Administration
- ✅ **Documentation** : Qui a quoi et pourquoi
- ✅ **Naming convention** : Noms cohérents et explicites
- ✅ **Séparation** : Admin du quotidien ≠ admin système
- ✅ **Monitoring** : Surveiller connexions et modifications
- ✅ **Sauvegarde** : Export configuration avant changements

### Automatisation
```powershell
# Template création utilisateur standard
function New-StandardUser {
    param($Username, $FullName, $Password)
    
    $SecurePassword = ConvertTo-SecureString $Password -AsPlainText -Force
    New-LocalUser -Name $Username -Password $SecurePassword -FullName $FullName
    Add-LocalGroupMember -Group "Users" -Member $Username
    Add-LocalGroupMember -Group "Remote Desktop Users" -Member $Username
}

# Rapport utilisateurs/groupes
Get-LocalUser | ForEach-Object {
    [PSCustomObject]@{
        Name = $_.Name
        Enabled = $_.Enabled
        LastLogon = $_.LastLogon
        Groups = (Get-LocalGroupMembership -User $_.Name -ErrorAction SilentlyContinue).Name -join ", "
    }
} | Export-Csv "UsersReport.csv" -NoTypeInformation
```

---
**💡 Memo** : `lusrmgr.msc` pour GUI, PowerShell Get/Set-LocalUser pour moderne, toujours principe moindre privilège !