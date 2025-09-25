# 🏗️ Active Directory — Aide-mémoire

## 🌳 Architecture & Concepts

### Hiérarchie Logique
```
Forêt (Forest)
├── Domaine Racine (Root Domain)
│   ├── Unités d'Organisation (OU)
│   ├── Utilisateurs & Groupes
│   └── Ordinateurs
├── Domaines Enfants (Child Domains)
├── Domaines Externes (External Trusts)
└── Schéma (Forest-wide)
```

### Composants Essentiels
| Composant | Portée | Description | Usage |
|-----------|--------|-------------|-------|
| **Forêt** | Globale | Périmètre sécuritaire ultime | Isolation complète |
| **Domaine** | Administrative | Unité de gestion (GPO, auth) | Segmentation organisationnelle |
| **OU** | Locale | Organisation hiérarchique | Délégation, GPO ciblées |
| **Site** | Physique | Localisation géographique | Optimisation réplication |

### Schéma Active Directory
- **Classes** : Types d'objets (User, Computer, Group)
- **Attributs** : Propriétés objets (sAMAccountName, mail, memberOf)
- **Règles** : Attributs obligatoires/optionnels par classe
- **Extensions** : Personnalisation schéma (Exchange, applications)

## 🔐 Protocoles & Services

### Trinity AD : DNS + LDAP + Kerberos
```
DNS (Port 53)
├── Enregistrements SRV (_ldap._tcp, _kerberos._tcp)
├── Localisation services AD
└── Résolution noms NetBIOS

LDAP/LDAPS (389/636)
├── Annuaire X.500 simplifié
├── Requêtes objet AD
└── Authentification simple/SASL

Kerberos (Port 88)
├── Authentification par tickets
├── SSO (Single Sign-On)
└── Délégation sécurisée
```

### Ports Critiques AD
| Service | Port | Protocol | Usage |
|---------|------|----------|-------|
| **DNS** | 53 | UDP/TCP | Résolution noms |
| **Kerberos** | 88 | UDP/TCP | Authentification |
| **RPC Endpoint** | 135 | TCP | RPC initial |
| **NetLogon** | 389 | UDP | Logon domain |
| **LDAP** | 389 | TCP | Queries AD |
| **LDAPS** | 636 | TCP | LDAP over SSL |
| **Global Catalog** | 3268/3269 | TCP | GC queries |

## 👥 Objets & Conteneurs par Défaut

### Conteneurs Système
| Conteneur | Fonction | Contenu Typique |
|-----------|----------|-----------------|
| **Built-in** 🧱 | Groupes locaux hérités SAM | Administrators, Users, Guests |
| **Users** 👥 | Conteneur utilisateurs défaut | Comptes créés sans OU spécifiée |
| **Computers** 💻 | Ordinateurs joints domaine | Postes de travail, serveurs |
| **System** ⚙️ | Objets système critiques | Configuration AD, politiques |
| **ForeignSecurityPrincipals** 🔒 | Objets domaines externes | Trustees externes dans groupes |

### Comptes Administrateurs par Défaut
- **Administrator** : Compte admin intégré domaine
- **Guest** : Compte invité (désactivé par défaut)
- **krbtgt** : Compte service Kerberos (critique, jamais supprimer)
- **DSRM** : Directory Services Restore Mode (local DC)

## 🏘️ Groupes & Stratégie AGDLP

### Types & Portées Groupes
| Type | Portée | Membres Autorisés | Usage |
|------|--------|-------------------|-------|
| **Sécurité** | Globale | Users/Computers/GG même domaine | Regroupement utilisateurs |
| **Sécurité** | Domaine Local | Tous objets forêt + groupes externes | Attribution permissions |
| **Sécurité** | Universelle | Tous objets forêt | Cross-domain, réplication GC |
| **Distribution** | * | Selon portée | Messagerie uniquement |

### Stratégie AGDLP (Best Practice)
```
A (Accounts) → G (Global Groups) → DL (Domain Local) → P (Permissions)
     ↓                ↓                    ↓               ↓
Utilisateurs → GG_Dept_Finance → DL_Share_RW → ACL Dossier
```

#### Exemple Pratique AGDLP
```
1. Créer groupes métier (Global)
   GG_Finance_Users, GG_RH_Users, GG_IT_Admins

2. Créer groupes ressource (Domain Local)  
   DL_FileServer_Read, DL_FileServer_Modify, DL_Printers_Print

3. Affecter permissions
   DL_FileServer_Modify → NTFS Full Control sur \\srv\shared

4. Imbriquer groupes
   GG_Finance_Users → membre de → DL_FileServer_Modify
```

## 🎭 Rôles FSMO (Single Master Operations)

### Rôles Niveau Forêt (1 seul par forêt)
| Rôle | Fonction | Impact si Indisponible |
|------|----------|------------------------|
| **Schema Master** 🧬 | Modifications schéma AD | Pas d'extensions schéma |
| **Domain Naming Master** 🌐 | Ajout/suppression domaines | Pas nouveaux domaines |

### Rôles Niveau Domaine (1 par domaine)
| Rôle | Fonction | Impact si Indisponible |
|------|----------|------------------------|
| **PDC Emulator** 🕘 | Auth prioritaire, temps, mdp | Auth lente, pb synchro temps |
| **RID Master** 🆔 | Distribution pools RID | Plus création objets |
| **Infrastructure Master** 🔁 | Références inter-domaines | Problèmes groupes cross-domain |

### Vérification & Transfert FSMO
```cmd
# Vérifier détenteurs FSMO
netdom query fsmo

# Via PowerShell
Get-ADForest | Select SchemaMaster, DomainNamingMaster
Get-ADDomain | Select PDCEmulator, RIDMaster, InfrastructureMaster

# Transfert FSMO (gracieux)
Move-ADDirectoryServerOperationMasterRole -Identity DC02 -OperationMasterRole SchemaMaster

# Saisie FSMO (forcé, si détenteur inaccessible)
Move-ADDirectoryServerOperationMasterRole -Identity DC02 -OperationMasterRole SchemaMaster -Force
```

## 🗂️ Profils Utilisateurs & Redirection

### Types de Profils
| Type | Stockage | Avantages | Inconvénients |
|------|----------|-----------|---------------|
| **Local** | Poste local | Rapide | Pas de mobilité |
| **Itinérant (Roaming)** | Serveur | Mobilité complète | Lent, volumineux |
| **Obligatoire** | Serveur (RO) | Standardisation | Pas personnalisation |
| **Temporaire** | RAM/Temp | Sécurisé | Perdu à déconnexion |

### Configuration Profils
```
Propriétés Utilisateur > Profil:
├── Chemin profil: \\srv\profiles\%username%
├── Script ouverture: logon.bat
├── Dossier personnel: \\srv\home\%username%
└── Lecteur: H:
```

### Redirection de Dossiers (GPO)
```
Gestion Politique > Configuration Utilisateur > Stratégies > 
Paramètres Windows > Redirection de dossiers:

├── Documents → \\srv\redirected\%username%\Documents
├── Bureau → \\srv\redirected\%username%\Desktop  
├── Menu Démarrer → \\srv\redirected\%username%\StartMenu
└── Favoris → \\srv\redirected\%username%\Favorites
```

**Avantages Redirection** :
- Performance (seuls dossiers concernés)
- Sauvegarde centralisée
- Accès offline possible (avec synchronisation)

## 🌐 Sites, Liens & Réplication

### Architecture Sites
```
Site Paris (Subnet: 192.168.1.0/24)
├── DC01.domain.com
└── DC02.domain.com

Site Lyon (Subnet: 192.168.2.0/24)  
├── DC03.domain.com
└── Liens Site: Paris-Lyon (Coût: 100)
```

### Configuration Réplication
| Paramètre | Valeur Recommandée | Impact |
|-----------|-------------------|--------|
| **Intervalle réplication** | 180 min (LAN), 15 min (intra-site) | Cohérence vs bande passante |
| **Fenêtre réplication** | 22:00-06:00 (WAN) | Éviter heures ouvrées |
| **Coût lien** | 100 (LAN), 500+ (WAN) | Préférence route réplication |
| **Compression** | >50% WAN | CPU vs bande passante |

### Commandes Réplication
```cmd
# Forcer réplication
repadmin /replicate DC02 DC01 "DC=domain,DC=com"

# Statut réplication
repadmin /showrepl DC01

# Vérifier topologie
repadmin /kcc

# Synchroniser tous DCs
repadmin /syncall /A /e /P /d
```

## 💾 Sauvegarde & Récupération

### Sauvegarde AD Intégrée
```cmd
# Windows Server Backup
wbadmin start systemstatebackup -backuptarget:E: -quiet

# Via PowerShell
Backup-SystemState -BackupLocation E:\ADBackup -Force

# Planification
schtasks /create /tn "AD Backup" /tr "wbadmin start systemstatebackup -backuptarget:E:" /sc daily /st 02:00
```

### Modes Récupération
| Mode | Description | Usage |
|------|-------------|-------|
| **Non-Authoritative** | Restaure et réplique depuis autres DCs | Récupération DC défaillant |
| **Authoritative** | Force propagation objets restaurés | Récupération objets supprimés |
| **Primary Restore** | Premier DC restauré dans domaine | Récupération domaine complet |

### Commandes Récupération
```cmd
# DSRM (Directory Services Restore Mode)
# F8 > DSRM au boot du DC

# Restauration authoritative (après restore non-auth)
ntdsutil "authoritative restore" "restore subtree ou=users,dc=domain,dc=com" quit quit

# Marquer objets comme autoritaires
ntdsutil "authoritative restore" "restore object cn=john,ou=users,dc=domain,dc=com" quit quit
```

## 🔍 Outils Administration & Diagnostic

### Outils MMC Essentiels
| Console | Usage | Raccourci |
|---------|-------|-----------|
| **dsa.msc** | Utilisateurs et ordinateurs AD | - |
| **domain.msc** | Domaines et approbations | - |
| **dssite.msc** | Sites et services AD | - |
| **gpmc.msc** | Gestion stratégies de groupe | - |
| **adsiedit.msc** | Éditeur ADSI (raw LDAP) | Avancé |

### PowerShell Active Directory
```powershell
# Import module
Import-Module ActiveDirectory

# Utilisateurs
Get-ADUser -Filter * -Properties * | Select Name, EmailAddress, LastLogonDate
New-ADUser -Name "John Doe" -SamAccountName "jdoe" -UserPrincipalName "jdoe@domain.com"
Set-ADUser -Identity "jdoe" -EmailAddress "john.doe@company.com"

# Groupes  
Get-ADGroup -Filter * | Where-Object {$_.GroupScope -eq "Global"}
Add-ADGroupMember -Identity "GG_Finance" -Members "jdoe"
Get-ADGroupMember -Identity "Domain Admins"

# Ordinateurs
Get-ADComputer -Filter * -Properties * | Select Name, OperatingSystem, LastLogonDate
```

### Outils Ligne Commande
```cmd
# Net commands
net user /domain                    # Lister utilisateurs domaine
net group "Domain Admins" /domain   # Membres groupe
net accounts /domain               # Politique mots de passe

# DS commands  
dsquery user -name "john*"         # Rechercher utilisateurs
dsget user CN=john,CN=Users,DC=domain,DC=com -display -email
dsmod user CN=john,CN=Users,DC=domain,DC=com -pwd NewPassword123!

# LDAP queries
ldifde -f export.ldf -t 389 -d "DC=domain,DC=com" -r "(objectClass=user)"
csvde -f users.csv -d "CN=Users,DC=domain,DC=com" -r "(objectClass=user)"
```

## 📊 Surveillance & Performance

### Compteurs Performance AD
| Compteur | Objet | Seuil Normal | Description |
|----------|-------|--------------|-------------|
| **LDAP Searches/sec** | NTDS | <50 | Requêtes LDAP |
| **LDAP Binds/sec** | NTDS | <100 | Connexions LDAP |
| **DRA Inbound Objects/sec** | NTDS | Variable | Réplication entrante |
| **DRA Outbound Objects/sec** | NTDS | Variable | Réplication sortante |
| **ATQ Threads LDAP** | NTDS | <50 | Threads LDAP actifs |

### Event IDs Critiques
| Event ID | Log | Description | Action |
|----------|-----|-------------|--------|
| **1644** | Directory Service | Base défragmentée | Information |
| **2042** | Directory Service | Pas de réplication >24h | Vérifier connectivité |
| **4624** | Security | Ouverture session réussie | Audit normal |
| **4625** | Security | Échec authentification | Surveiller attaques |
| **5805** | DNS | Enregistrement échoué | Vérifier DNS |

## 🔐 Sécurité & Bonnes Pratiques

### Comptes de Service Managés
```powershell
# Group Managed Service Account (gMSA)
New-ADServiceAccount -Name "svc-webapp" -DNSHostName "webapp.domain.com" -ManagedPasswordIntervalInDays 30

# Installer sur serveur
Install-ADServiceAccount -Identity "svc-webapp"

# Utilisation dans service
# Compte: DOMAIN\svc-webapp$
# Mot de passe: <géré automatiquement>
```

### Délégation Administrative
```
Délégation Assistant (Utilisateurs et Ordinateurs AD):
├── Sélectionner OU cible
├── Ajouter utilisateurs/groupes délégataires
├── Choisir tâches prédéfinies:
│   ├── Créer, supprimer comptes utilisateurs
│   ├── Réinitialiser mots de passe
│   └── Modifier informations utilisateurs
└── Permissions personnalisées si nécessaire
```

### Audit & Compliance
```cmd
# Activer audit événements AD
auditpol /set /subcategory:"Directory Service Access" /success:enable /failure:enable
auditpol /set /subcategory:"Directory Service Changes" /success:enable /failure:enable

# Via GPO: Configuration ordinateur > Stratégies > Paramètres Windows > 
# Paramètres de sécurité > Stratégies locales > Stratégie d'audit
```

## 💡 Bonnes Pratiques

### Architecture
- ✅ **Un domaine** par défaut (sauf isolation forte requise)
- ✅ **Pas d'utilisateurs** dans conteneur Users par défaut
- ✅ **Structure OU** réfléchie (par fonction, pas par localisation)
- ✅ **Délégation administrative** granulaire via OU
- ✅ **Comptes de service** dédiés (pas admin domaine)

### Sécurité
- ✅ **Politique mots de passe** renforcée (>12 caractères)
- ✅ **Audit** activation/modifications objets sensibles
- ✅ **Sauvegarde régulière** System State
- ✅ **Test restauration** procédures DSRM
- ✅ **Monitoring** réplication et performance

### Operational
- ✅ **Documentation** architecture et procédures
- ✅ **Naming conventions** cohérentes
- ✅ **Lifecycle management** comptes inactifs
- ✅ **Change management** modifications schéma
- ✅ **Disaster recovery** plan testé régulièrement

---
**💡 Memo** : AGDLP pour groupes, FSMO à surveiller, DNS critique, sauvegarde System State régulière !
