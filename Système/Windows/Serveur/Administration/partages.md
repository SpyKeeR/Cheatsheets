# 📁 Partages Windows (SMB/CIFS) — Aide-mémoire

## 🔐 Concepts de Sécurité

### Double Authentification
```
Accès Partage = Permissions Partage ∩ Permissions NTFS
                      ↓                    ↓
              (Niveau réseau)      (Niveau système fichiers)
```

**Règle d'Or** : L'autorisation la **plus restrictive** s'applique toujours

### Exemple Pratique
```
Utilisateur "John" accède à \\Server\Share\file.txt :

Permissions Partage : John = Lecture
Permissions NTFS    : John = Contrôle total
Résultat final      : Lecture (plus restrictif)

Permissions Partage : Everyone = Contrôle total  
Permissions NTFS    : John = Lecture
Résultat final      : Lecture (plus restrictif)
```

## 📋 Types de Permissions

### Permissions Partage (Share-Level)
| Permission | Description | Droits Accordés |
|------------|-------------|-----------------|
| **Lecture** | Read | Lire fichiers/dossiers, exécuter |
| **Modifier** | Change | Lecture + écriture + suppression |
| **Contrôle total** | Full Control | Modify + change permissions |

### Permissions NTFS (Granulaires)
| Permission | Description | Contenu |
|------------|-------------|---------|
| **Contrôle total** | Full Control | Tous droits + propriété |
| **Modifier** | Modify | RWX + suppression (pas permissions) |
| **Lecture et exécution** | Read & Execute | Lire + naviguer + exécuter |
| **Écriture** | Write | Créer fichiers/dossiers |
| **Lecture** | Read | Lire contenu seulement |
| **Affichage du contenu** | List Folder Contents | Naviguer dossiers (pas fichiers) |

## 🛠️ Gestion via Interface (MMC)

### Création Partage
```
Gestionnaire de serveur > Rôles > Services de fichiers et de stockage
└── Partages > Tâches > Nouveau partage
    ├── Profil : Partage SMB - Rapide/Avancé
    ├── Emplacement : Choisir serveur et volume
    ├── Nom : Nom du partage réseau
    ├── Paramètres : Cache, chiffrement, énumération
    └── Permissions : Share + NTFS
```

### Propriétés Avancées
- **Mise en cache** : Améliore performances (BranchCache)
- **Enumération basée sur l'accès** : Masque fichiers sans droits
- **Chiffrement** : SMB 3.0+ (Windows 8/Server 2012+)
- **Partages administratifs cachés** : C$, ADMIN$, IPC$

## 💻 Gestion via Ligne de Commande

### Commandes NET (Legacy)
```cmd
# Création partage
net share MonPartage=C:\Dossier
net share MonPartage=C:\Dossier /GRANT:"Domain Users",READ
net share MonPartage=C:\Dossier /GRANT:"Administrators",FULL

# Gestion existant
net share MonPartage /DELETE              # Supprimer partage
net share                                 # Lister tous partages
net share MonPartage                      # Détails partage spécifique

# Connexion client
net use Z: \\Server\MonPartage           # Mapper lecteur
net use Z: \\Server\MonPartage /USER:DOMAIN\username  # Avec credentials
net use Z: /DELETE                       # Déconnecter lecteur
net use                                  # Lister connexions actives
```

### PowerShell (Moderne)
```powershell
# Création et gestion partages
New-SmbShare -Name "MonPartage" -Path "C:\Dossier" -FullAccess "Administrators" -ReadAccess "Domain Users"
Get-SmbShare                            # Lister partages
Get-SmbShare -Name "MonPartage"         # Détails partage
Remove-SmbShare -Name "MonPartage" -Force  # Supprimer

# Permissions partage
Grant-SmbShareAccess -Name "MonPartage" -AccountName "Domain\John" -AccessRight Read
Revoke-SmbShareAccess -Name "MonPartage" -AccountName "Domain\John" -Force
Get-SmbShareAccess -Name "MonPartage"   # Voir permissions

# Connexions client  
New-SmbMapping -LocalPath Z: -RemotePath \\Server\MonPartage
New-SmbMapping -LocalPath Z: -RemotePath \\Server\MonPartage -UserName "DOMAIN\user" -Password (ConvertTo-SecureString "password" -AsPlainText -Force)
Get-SmbMapping                          # Lister mappings
Remove-SmbMapping -LocalPath Z: -Force  # Supprimer mapping

# Sessions actives
Get-SmbSession                          # Sessions utilisateurs
Get-SmbOpenFile                         # Fichiers ouverts
```

## 🔍 Monitoring & Diagnostic

### Surveillance Partages
```powershell
# Connexions actives
Get-SmbSession | Select-Object ClientComputerName, ClientUserName, NumOpens
Get-SmbOpenFile | Select-Object ClientComputerName, ClientUserName, Path

# Statistiques usage
Get-SmbConnection                       # Connexions SMB
Get-SmbMultichannelConnection          # Connexions multicanal (SMB 3.0+)

# Événements partages (Event Viewer)
Get-WinEvent -LogName "Microsoft-Windows-SmbServer/Security" -MaxEvents 50
```

### Commandes Diagnostic
```cmd
# Tests connectivité
net view \\ServerName                   # Lister partages serveur
net view \\ServerName\IPC$             # Test connectivité base

# Informations sessions
net session                            # Sessions actives (serveur)
net file                               # Fichiers ouverts
net config server                     # Configuration serveur
```

## 🌐 Versions SMB & Fonctionnalités

### Évolution Protocole SMB
| Version | Windows | Fonctionnalités Clés |
|---------|---------|----------------------|
| **SMB 1.0** | XP/2003 | ⚠️ Obsolète, vulnérabilités |
| **SMB 2.0** | Vista/2008 | Performance améliorée, OpLocks |
| **SMB 2.1** | 7/2008 R2 | BranchCache, Large MTU |
| **SMB 3.0** | 8/2012 | Chiffrement, Multicanal, VSS |
| **SMB 3.02** | 8.1/2012 R2 | Améliorations performance |
| **SMB 3.1.1** | 10/2016 | Authentification, intégrité |

### Configuration SMB
```powershell
# Version SMB actuelle
Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol, EnableSMB2Protocol
Get-SmbClientConfiguration | Select-Object EnableSMB1Protocol, EnableSMB2Protocol

# Désactiver SMB 1.0 (sécurité)
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

# Activer fonctionnalités SMB 3.0+
Set-SmbServerConfiguration -EnableMultiChannel $true
Set-SmbServerConfiguration -EncryptData $true  # Chiffrement global
```

## 🏢 Partages Administratifs

### Partages par Défaut
| Partage | Description | Accès |
|---------|-------------|-------|
| **C$, D$, etc.** | Racines lecteurs | Administrateurs |
| **ADMIN$** | %SystemRoot% (C:\Windows) | Administrateurs |
| **IPC$** | Inter-Process Communication | Authentification |
| **NETLOGON** | Scripts logon domaine | Contrôleurs domaine |
| **SYSVOL** | Réplication AD | Contrôleurs domaine |

### Gestion Partages Cachés
```powershell
# Voir partages cachés (finissent par $)
Get-SmbShare | Where-Object Name -like "*$"

# Créer partage caché
New-SmbShare -Name "Secret$" -Path "C:\Confidential" -FullAccess "Administrators"

# Accès partages cachés
# \\Server\Secret$ (ne s'affiche pas dans net view)
```

## 🔧 Configuration Avancée

### Paramètres Serveur SMB
```powershell
# Configuration optimisée
Set-SmbServerConfiguration -MaxThreadsPerQueue 20
Set-SmbServerConfiguration -MaxChannelPerSession 32
Set-SmbServerConfiguration -MaxSessionPerConnection 16384

# Sécurité renforcée
Set-SmbServerConfiguration -RejectUnencryptedAccess $true
Set-SmbServerConfiguration -RequireSecuritySignature $true

# BranchCache (WAN optimization)
Set-SmbServerConfiguration -EnableHashPublication $true
```

### DFS Integration
```powershell
# DFS avec partages SMB
Install-WindowsFeature FS-DFS-Namespace, FS-DFS-Replication -IncludeManagementTools

# Créer namespace DFS
New-DfsnRoot -TargetPath "\\Server\DFSRoot" -Type DomainV2 -Path "\\domain.com\files"
New-DfsnFolder -Path "\\domain.com\files\shared" -TargetPath "\\Server1\MonPartage"
```

## 🚨 Troubleshooting

### Problèmes Courants
| Symptôme | Cause Probable | Diagnostic | Solution |
|----------|----------------|------------|----------|
| **Accès refusé** | Permissions insuffisantes | Vérifier Share + NTFS | Ajuster permissions |
| **Partage invisible** | Pas de droits lecture | `net view` en admin | Grant permissions |
| **Connexion lente** | SMB 1.0 ou réseau | `Get-SmbConnection` | Upgrade SMB, vérifier réseau |
| **Authentification échec** | Credentials invalides | Event Viewer | Vérifier compte/mot de passe |

### Commandes Diagnostic
```powershell
# Test accès réseau
Test-NetConnection ServerName -Port 445    # Port SMB
telnet ServerName 445                      # Alternative

# Vérifier authentification
Test-ComputerSecureChannel                 # Test relation domaine
nltest /sc_query:domain.com               # Secure channel

# Logs détaillés
Enable-SmbServerAudit                      # Activer audit SMB
Get-WinEvent -LogName "Microsoft-Windows-SMBServer/Audit" -MaxEvents 20
```

### Reset Connexions SMB
```cmd
# Côté Client
net use * /delete /y                      # Supprimer toutes connexions
klist purge                               # Vider cache Kerberos  

# Côté Serveur
net session /delete                       # Fermer sessions
net file /close                          # Fermer fichiers ouverts
```

## 💡 Bonnes Pratiques

### Sécurité
- ✅ **Désactiver SMB 1.0** (vulnérable)
- ✅ **Utiliser groupes AD** plutôt qu'utilisateurs individuels
- ✅ **Permissions NTFS granulaires** + Share permissions larges
- ✅ **Chiffrement SMB 3.0+** pour données sensibles
- ✅ **Audit accès** fichiers critiques

### Performance
- ✅ **SMB Multicanal** pour liaisons redondantes
- ✅ **BranchCache** pour sites distants
- ✅ **Jumbo Frames** si infrastructure supporte
- ✅ **SSD** pour partages haute fréquence
- ✅ **Monitoring** utilisation et performances

### Organisation
- ✅ **Convention nommage** claire et cohérente
- ✅ **Structure hiérarchique** logique
- ✅ **Documentation** permissions et usage
- ✅ **Partages dédiés** par fonction/département
- ✅ **Sauvegarde** partages critiques

### Template Partage Standard
```powershell
# Création partage type entreprise
New-SmbShare -Name "Departement-IT" -Path "D:\Shares\IT" -Description "Partage service IT"
Grant-SmbShareAccess -Name "Departement-IT" -AccountName "Domain\IT-Team" -AccessRight Full -Force
Grant-SmbShareAccess -Name "Departement-IT" -AccountName "Domain\IT-ReadOnly" -AccessRight Read -Force
Revoke-SmbShareAccess -Name "Departement-IT" -AccountName "Everyone" -Force

# Permissions NTFS correspondantes
icacls "D:\Shares\IT" /grant "Domain\IT-Team:(OI)(CI)F" /grant "Domain\IT-ReadOnly:(OI)(CI)R"
```

---
**💡 Memo** : Share permissions + NTFS permissions = le plus restrictif s'applique ! SMB 1.0 = danger, SMB 3.0+ = sécurisé.