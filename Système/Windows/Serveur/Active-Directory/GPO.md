# 📋 Group Policy Objects (GPO) — Aide-mémoire

## 🏗️ Architecture & Concepts

### Hiérarchie Application GPO
```
Ordre d'application (LSDOU) :
1. Local Policy (machine locale)
2. Site Policy (site AD)
3. Domain Policy (domaine)
4. Organizational Unit Policy (OU)
   └── OU Parent → OU Enfant → OU Petit-enfant
```

**Règle** : La dernière appliquée l'emporte (OU la plus spécifique)

### Structure GPO
| Composant | Description | Stockage |
|-----------|-------------|----------|
| **Group Policy Container (GPC)** | Métadonnées, version | Active Directory |
| **Group Policy Template (GPT)** | Fichiers paramètres | SYSVOL (`\\domain\SYSVOL\domain\Policies\{GUID}`) |
| **GUID** | Identifiant unique | `{12345678-1234-5678-9ABC-123456789012}` |

### Sections Configuration
```
GPO Structure:
├── Computer Configuration (Ordinateur)
│   ├── Policies/Stratégies
│   └── Preferences/Préférences
└── User Configuration (Utilisateur)
    ├── Policies/Stratégies
    └── Preferences/Préférences
```

## 🎯 Liaison & Ciblage

### Niveaux de Liaison
| Niveau | Portée | Usage | Exemples |
|--------|--------|-------|---------|
| **Site** | Sites AD (géographique) | Paramètres réseau, proxy | Site Paris, Site Lyon |
| **Domain** | Domaine complet | Politiques globales | Sécurité, audit |
| **OU** | Unité organisationnelle | Ciblage précis | Département, fonction |

### Mécanismes de Contrôle

#### Block Inheritance (Bloquer Héritage)
- **Usage** : Empêche application GPO niveau supérieur
- **Attention** : Peut casser configurations critiques
- **Recommandation** : Utiliser avec parcimonie

#### Enforced/No Override (Appliqué)
- **Usage** : Force application même si héritage bloqué
- **Priorité** : Passe outre Block Inheritance
- **Cas d'usage** : GPO sécurité critique

#### Security Filtering (Filtrage Sécurité)
```
Permissions requises pour application GPO :
├── Read (Lecture) : Lire paramètres GPO
├── Apply Group Policy : Appliquer réellement
└── Deny : Exclure explicitement (priorité absolue)

Groupes par défaut :
- Authenticated Users : Tous utilisateurs/ordinateurs domaine
- Domain Computers : Tous ordinateurs domaine
- Domain Users : Tous utilisateurs domaine
```

#### WMI Filtering (Filtrage WMI)
```powershell
# Exemples requêtes WMI courantes
# OS spécifique
SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.0%"

# RAM minimum
SELECT * FROM Win32_ComputerSystem WHERE TotalPhysicalMemory > 4294967296

# Type ordinateur
SELECT * FROM Win32_ComputerSystem WHERE PCSystemType = 1  # Desktop
SELECT * FROM Win32_ComputerSystem WHERE PCSystemType = 2  # Laptop
```

## ⚙️ Gestion & Administration

### GPMC (Group Policy Management Console)
```cmd
# Lancer GPMC
gpmc.msc

# Commandes PowerShell GPO
Import-Module GroupPolicy
Get-GPO -All                     # Lister toutes GPO
Get-GPOReport -All -ReportType Html -Path "C:\GPOReport.html"
New-GPO -Name "TestGPO" -Domain domain.com
Set-GPLink -Name "TestGPO" -Target "OU=IT,DC=domain,DC=com"
```

### Commandes Essentielles
```cmd
# Force actualisation
gpupdate /force                  # Client actuel
gpupdate /target:computer /force # Partie ordinateur uniquement
gpupdate /target:user /force     # Partie utilisateur uniquement

# Informations appliquées
gpresult /r                      # Résumé
gpresult /h report.html          # Rapport HTML détaillé
gpresult /z                      # Verbeux (toutes infos)
gpresult /user username /h user_report.html  # Utilisateur spécifique

# Test application
rsop.msc                         # Resultant Set of Policy (GUI)
```

### Actualisation & Cycle
| Type | Intervalle | Variation | Exceptions |
|------|------------|-----------|------------|
| **Client** | 90 minutes | ±30 minutes | Sécurité : 16h max |
| **Serveur** | 5 minutes | 0 | Immediate |
| **DC** | 5 minutes | 0 | Immediate |

## 🏛️ GPO par Défaut

### Default Domain Policy (DDP)
```
Emplacement: Racine domaine
Fonction: Politiques sécurité domaine
Contenu type:
├── Account Policies (mots de passe, verrouillage)
├── Kerberos Policy
├── Audit Policy
└── User Rights Assignment (droits utilisateurs)

⚠️ Attention: Ne pas modifier directement, créer GPO dédiées
```

### Default Domain Controllers Policy (DDCP)
```
Emplacement: OU Domain Controllers
Fonction: Sécurité spécifique DCs
Contenu type:
├── User Rights (Allow log on locally, etc.)
├── Audit policies avancées
├── Security settings DC
└── Services configuration

⚠️ Attention: Critique pour sécurité AD
```

## 📁 Modèles d'Administration (ADMX/ADML)

### Structure Templates
```
PolicyDefinitions/
├── *.admx (définitions policies - langue neutre)
├── fr-FR/ (ou autre langue)
│   └── *.adml (labels traduits)
├── en-US/
│   └── *.adml
└── Application-specific/
    ├── chrome.admx
    ├── office.admx
    └── etc.
```

### Magasin Central ADMX
```
Création magasin central:
1. Créer: \\domain\SYSVOL\domain\Policies\PolicyDefinitions\
2. Copier: C:\Windows\PolicyDefinitions\* → Magasin central
3. Structure:
   \\domain\SYSVOL\domain\Policies\PolicyDefinitions\
   ├── *.admx
   └── [language]\*.adml

Avantages:
- Cohérence entre admins
- Templates centralisés
- Pas de conflits versions
```

### Templates Tiers Courants
| Application | Source | Usage |
|-------------|--------|--------|
| **Google Chrome** | Google | Policies navigateur |
| **Microsoft Office** | Microsoft | Configuration Office |
| **Adobe Reader** | Adobe | Paramètres PDF |
| **Mozilla Firefox** | Mozilla | Policies Firefox |
| **Windows 10/11** | Microsoft | Nouvelles fonctionnalités |

## 📊 Types de Paramètres

### Computer Configuration
```
Administrative Templates:
├── Control Panel (Panneau configuration)
├── Network (Réseau)
├── Printers (Imprimantes)  
├── System (Système)
├── Windows Components
└── MSI Package Management

Security Settings:
├── Account Policies
├── Local Policies
├── Windows Firewall
├── Registry & File System
└── Services
```

### User Configuration
```
Administrative Templates:
├── Control Panel
├── Desktop
├── Start Menu and Taskbar
├── Windows Components
└── System

Security Settings:
├── Public Key Policies
├── Software Restriction Policies
└── IP Security Policies
```

### Preferences vs Policies
| Type | Comportement | Modification Utilisateur | Persistance |
|------|-------------|-------------------------|-------------|
| **Policies** | Gris (disabled) | ❌ Impossible | Supprimé si GPO retirée |
| **Preferences** | Appliqué une fois | ✅ Possible | Persiste même si GPO retirée |

## 🔍 Diagnostics & Troubleshooting

### Logs Événements GPO
| Log | Event IDs | Description |
|-----|-----------|-------------|
| **System** | 1500-1502 | Erreurs application GPO |
| **Application** | 1000-1007 | Processing GPO |
| **Group Policy** | 1085, 1125, 1127 | Détails application |

### Outils Debug
```cmd
# Mode debug GPO (Windows 10+)
# Registry: HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Diagnostics
# UserGPVerboseStatus = 1
# GPVerboseStatus = 1

# Journalisation avancée
gpedit.msc → Computer Config → Admin Templates → System → Group Policy
Enable: "Configure Group Policy processing"
```

### Problèmes Courants
| Symptôme | Cause Probable | Solution |
|----------|---------------|----------|
| **GPO non appliquée** | Permissions, filtrage | Vérifier Security Filtering, WMI |
| **Application lente** | Réseau, DC distant | Optimiser liens sites AD |
| **Conflit paramètres** | Ordre application | Revoir hiérarchie OU |
| **Préférences échouent** | Permissions cibles | Droits sur registre/fichiers |

### Commandes Debug
```cmd
# Test connectivité DC
nltest /dsgetdc:domain.com

# Vérifier réplication SYSVOL
dfsrdiag pollad
dfsrdiag backlog /rgname:"Domain System Volume" /rfname:"SYSVOL Share" /rmem:DC02

# Cache GPO local
gpotool /checkacl
gpotool /verbose
```

## 🚀 Stratégies Déploiement

### Méthode Progressive
```
1. Développement
   ├── Créer GPO test
   ├── Lier à OU pilote
   └── Valider fonctionnement

2. Pré-production
   ├── Dupliquer GPO
   ├── Élargir périmètre test
   └── Monitoring erreurs

3. Production
   ├── Déploiement par phases
   ├── Communication utilisateurs
   └── Support niveau 1 préparé
```

### Bonnes Pratiques Nommage
```
Convention recommandée:
[TYPE]_[CIBLE]_[FONCTION]_[VERSION]

Exemples:
- COMP_Servers_Security_v1.0
- USER_Finance_OfficeSettings_v2.1
- BOTH_AllUsers_Proxy_v1.5
- PREF_IT_SoftwareDeployment_v1.0
```

## 📋 Modèle GPO Type

### Structure Organisée
```
Domain Root
├── Default Domain Policy (ne pas modifier)
├── Security_Password_Policy
├── Security_Audit_Policy
└── OUs
    ├── Servers
    │   ├── COMP_Servers_Security
    │   ├── COMP_Servers_WindowsUpdate
    │   └── Domain Controllers
    │       └── Default Domain Controllers Policy
    ├── Workstations
    │   ├── COMP_Workstations_Security
    │   ├── COMP_Workstations_SoftwareInstall
    │   └── USER_All_DesktopSettings
    └── Users
        ├── USER_Finance_Applications
        ├── USER_IT_AdminRights
        └── USER_All_NetworkDrives
```

## 💡 Bonnes Pratiques

### Architecture
- ✅ **Séparer** Computer et User configurations
- ✅ **Une GPO = Une fonction** (pas monolithique)
- ✅ **Minimiser profondeur** OU (max 3-4 niveaux)
- ✅ **Documenter** chaque GPO (commentaires)
- ✅ **Versionning** GPO importantes

### Sécurité
- ✅ **Ne jamais modifier** Default Domain Policy
- ✅ **Tester** toujours en environnement pilote
- ✅ **Sauvegarder** GPO avant modifications
- ✅ **Principe moindre privilège** (Security Filtering)
- ✅ **Audit** modifications GPO

### Performance
- ✅ **Désactiver** sections non utilisées
- ✅ **Éviter** Block Inheritance sauf nécessité
- ✅ **Optimiser** requêtes WMI (coûteuses)
- ✅ **Regrouper** paramètres similaires
- ✅ **Supprimer** GPO obsolètes

### Maintenance
- ✅ **Audit régulier** GPO appliquées
- ✅ **Nettoyage** GPO non liées
- ✅ **Monitoring** logs événements
- ✅ **Formation** équipe administration
- ✅ **Documentation** à jour

---
**💡 Memo** : LSDOU pour ordre application, `gpupdate /force` pour test, documenter et tester avant production !