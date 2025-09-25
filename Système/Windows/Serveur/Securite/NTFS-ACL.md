# 🔐 NTFS Permissions & ACL — Aide-mémoire

## 🏗️ Architecture Sécurité NTFS

### Modèle de Sécurité Windows
```
Objet Sécurisé (Fichier/Dossier)
├── Security Descriptor (SD)
    ├── Owner SID (Propriétaire)
    ├── Primary Group SID
    ├── DACL (Discretionary Access Control List)
    │   └── ACE (Access Control Entries) × N
    └── SACL (System Access Control List) - Audit
```

### Types d'ACE (Access Control Entry)
| Type | Description | Usage |
|------|-------------|-------|
| **Allow ACE** | Autorise accès | Permissions positives |
| **Deny ACE** | Refuse accès | Restrictions explicites |
| **Audit ACE** | Enregistre accès | Journalisation (SACL) |

## 🎯 Permissions NTFS Standard

### Permissions de Base
| Permission | Description | Droits Inclus |
|------------|-------------|---------------|
| **Lecture** | Lire contenu fichier | R |
| **Écriture** | Modifier contenu | W |
| **Lecture et exécution** | Lire + naviguer + exécuter | RX |
| **Liste du contenu** | Naviguer dossiers (pas fichiers) | Liste uniquement |
| **Modification** | Lecture + écriture + suppression | RWX + Delete |
| **Contrôle total** | Tous droits + changement permissions | Full + Change Permissions |

### Permissions Granulaires (Avancées)
```
Permissions détaillées :
├── Parcourir le dossier / Exécuter le fichier
├── Afficher le contenu du dossier / Lire les données
├── Lire les attributs
├── Lire les attributs étendus
├── Créer des fichiers / Écrire les données
├── Créer des dossiers / Ajouter les données
├── Écrire les attributs
├── Écrire les attributs étendus
├── Supprimer les sous-dossiers et les fichiers
├── Supprimer
├── Lire les autorisations
├── Modifier les autorisations
└── Appropriation (Take Ownership)
```

## ⚖️ Règles de Précédence

### Ordre d'Évaluation
1. **Deny explicite** (priorité absolue)
2. **Allow explicite**
3. **Deny hérité**
4. **Allow hérité**
5. **Refus par défaut** (si aucune permission)

### Principe d'Accumulation
```
Permissions effectives = SOMME des permissions Allow - Permissions Deny

Exemple utilisateur "John" membre de :
├── Groupe A : Lecture (Allow)
├── Groupe B : Écriture (Allow)  
├── Groupe C : Modification (Deny)
└── Résultat : Pas d'accès (Deny prioritaire)
```

## 🔄 Héritage & Propagation

### Mécanisme Héritage
- **Héritage activé** : Sous-dossiers/fichiers héritent permissions parent
- **Propagation** : Modifications remontent l'arborescence
- **Rupture héritage** : Permissions explicites remplacent héritées

### Règles Déplacement/Copie
| Opération | Même Volume | Entre Volumes |
|-----------|-------------|---------------|
| **Déplacement** | Conserve permissions | Hérite destination |
| **Copie** | Hérite destination | Hérite destination |
| **Création** | Hérite parent | Hérite parent |

### Types Héritage
```
Héritage Options :
├── Ce dossier seulement
├── Ce dossier, sous-dossiers et fichiers
├── Ce dossier et sous-dossiers
├── Ce dossier et fichiers
├── Sous-dossiers et fichiers seulement
├── Sous-dossiers seulement
└── Fichiers seulement
```

## 🛠️ Gestion via Interface

### Propriétés Sécurité
```
Clic droit > Propriétés > Sécurité :
├── Noms groupes/utilisateurs
├── Permissions pour [sélection]
├── Modifier... (permissions de base)
├── Avancé... (permissions granulaires)
└── Propriétaire actuel
```

### Onglet Avancé
- **Propriétaire** : Changer propriétaire
- **Permissions effectives** : Calculer droits réels utilisateur
- **Audit** : Configurer journalisation accès
- **Héritage** : Gérer propagation permissions

## 💻 Gestion via Commandes

### ICACLS (Moderne)
```cmd
# Afficher permissions
icacls "C:\dossier"                # Permissions actuelles
icacls "C:\dossier" /t             # Récursif sous-arborescence

# Accorder permissions
icacls "C:\dossier" /grant "Utilisateurs:(R)"     # Lecture
icacls "C:\dossier" /grant "Admins:(F)"           # Contrôle total
icacls "C:\dossier" /grant "Users:(M)"            # Modification
icacls "C:\dossier" /grant "Users:(RX)"           # Lecture+Exécution

# Permissions spécifiques dossiers/fichiers
icacls "C:\dossier" /grant "Users:(OI)(CI)(R)"    # Héritage objets/conteneurs
icacls "C:\dossier" /grant "Users:(IO)(R)"        # Héritage seulement

# Supprimer permissions
icacls "C:\dossier" /remove "Users"               # Supprimer utilisateur
icacls "C:\dossier" /deny "Guest:(F)"             # Refus explicite

# Réinitialiser permissions
icacls "C:\dossier" /reset /t                     # Reset + héritage
icacls "C:\dossier" /restore "backup.acl"         # Restaurer sauvegarde
```

### Codes Permissions ICACLS
| Code | Permission | Description |
|------|------------|-------------|
| `F` | Full control | Contrôle total |
| `M` | Modify | Modification |
| `RX` | Read & execute | Lecture + exécution |
| `R` | Read | Lecture seule |
| `W` | Write | Écriture seule |
| `D` | Delete | Suppression |

### Codes Héritage ICACLS
| Code | Description | Application |
|------|-------------|-------------|
| `(OI)` | Object Inherit | Fichiers héritent |
| `(CI)` | Container Inherit | Dossiers héritent |
| `(IO)` | Inherit Only | Pas d'effet direct, héritage seulement |
| `(NP)` | No Propagate | Pas de propagation aux sous-niveaux |
| `(I)` | Permission héritée | Lecture seule (information) |

### PowerShell ACL
```powershell
# Consultation
Get-Acl "C:\dossier" | Format-List
(Get-Acl "C:\dossier").Access

# Création règle
$acl = Get-Acl "C:\dossier"
$AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("Users","ReadAndExecute","Allow")
$acl.SetAccessRule($AccessRule)
Set-Acl "C:\dossier" $acl

# Copier permissions
$acl = Get-Acl "C:\source"
Set-Acl "C:\destination" $acl
```

## 👤 Prise de Possession

### TAKEOWN Command
```cmd
# Prendre possession
takeown /f "C:\dossier"               # Fichier/dossier
takeown /f "C:\dossier" /r /d y       # Récursif avec confirmation
takeown /f "C:\dossier" /a            # Attribuer aux Administrateurs

# Via ICACLS après takeown
icacls "C:\dossier" /grant "Administrators:(F)" /t
```

### Situations Courantes
- **Fichiers orphelins** : Compte supprimé, SID invalide
- **Migration** : Déplacement entre domaines
- **Corruption** : Permissions incohérentes
- **Maintenance** : Accès administrateur requis

## 🔍 Audit & Journalisation

### Configuration Audit
```
Local Security Policy > Audit Policy :
├── Audit object access : Success/Failure
├── Audit privilege use : Failure (minimum)
└── Advanced Audit Policy Configuration (Server 2008+)

Via GPO :
Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy Configuration
```

### SACL Configuration
```cmd
# Activer audit sur objet
icacls "C:\sensitive" /audit "Everyone:(F)"        # Audit accès complet
icacls "C:\sensitive" /audit "Users:(R)"           # Audit lecture
icacls "C:\sensitive" /audit "Admins:(M)"          # Audit modifications
```

### Event Viewer Logs
| Event ID | Description | Log |
|----------|-------------|-----|
| **4656** | Handle to object requested | Security |
| **4658** | Handle to object closed | Security |
| **4663** | Object access attempted | Security |
| **4670** | Permissions changed | Security |

## 📊 Permissions Effectives

### Calcul Permissions
```
Permissions effectives pour utilisateur "John" :
├── Permissions directes sur objet
├── + Permissions groupes (Domain Users, Marketing, etc.)
├── - Permissions Deny (directes et groupes)
└── = Résultat final
```

### Outils Analyse
- **GUI** : Propriétés > Sécurité > Avancé > Permissions effectives
- **PowerShell** : `Get-Acl | Select-Object -ExpandProperty Access`
- **ICACLS** : `/t` pour analyse récursive

## 🚨 Problèmes Courants

### Symptômes Fréquents
| Problème | Cause Probable | Solution |
|----------|----------------|----------|
| **Access Denied** | Permissions insuffisantes | Vérifier permissions effectives |
| **Deny prioritaire** | Règle Deny explicite | Supprimer ou modifier Deny |
| **Héritage cassé** | Permissions explicites | Reset ou reconfigurer héritage |
| **SID invalide** | Compte supprimé/migré | Nettoyer ou remapper SID |

### Commandes Diagnostic
```cmd
# Analyser permissions utilisateur
whoami /all                          # SID et groupes utilisateur
icacls "C:\dossier" | findstr "John"  # Permissions spécifiques

# Vérifier propriétaire
dir /q "C:\dossier"                  # Afficher propriétaire
icacls "C:\dossier" | findstr ":"    # Détail permissions

# Test d'accès
cacls "C:\dossier"                   # Legacy mais parfois utile
```

## 💡 Bonnes Pratiques

### Organisation
- ✅ **Groupes plutôt qu'utilisateurs** individuels
- ✅ **Permissions minimales** (principe moindre privilège)
- ✅ **Héritage** pour simplifier gestion
- ✅ **Structure logique** dossiers par fonction/département

### Sécurité
- ✅ **Éviter Deny** sauf cas spéciaux (complexifie dépannage)
- ✅ **Documenter** permissions spéciales
- ✅ **Audit** dossiers sensibles
- ✅ **Tests** réguliers permissions effectives

### Maintenance
- ✅ **Sauvegarde ACL** avant modifications importantes
- ✅ **Nettoyage SID** obsolètes
- ✅ **Vérification** cohérence après migrations
- ✅ **Formation** utilisateurs sur bonnes pratiques

### Template Permissions Standard
```
Structure recommandée :
C:\Data\
├── Public\ (Everyone: R, Authenticated Users: RW)
├── Departments\
│   ├── IT\ (IT-Team: F, IT-ReadOnly: R)
│   ├── HR\ (HR-Team: F, HR-ReadOnly: R)
│   └── Finance\ (Finance-Team: F, Finance-ReadOnly: R)
└── Restricted\ (Administrators: F, Backup-Operators: R)
```

---
**💡 Memo** : Deny > Allow explicite > Allow hérité • Héritage simplifie gestion • Groupes > utilisateurs individuels !

## ACL & DACL (essentiel)
- DACL = liste ACE (Access Control Entries) → qui peut quoi.
- Refus explicite > Autorisation explicite > Hérité.
- Héritage : utile pour simplifier ; attention aux copies entre volumes (perms changent).
- Éviter `Deny` inutilement.
- Autorisations de base : Lecture, Liste de dossier, Lecture/Exécution, Écriture, Modification, Contrôle total
- Autorisations avancées : Création de fichiers/dossiers, Suppression, Lecture des autorisations, Appropriation, Écriture d’attributs étendus…
- Déplacement sur même volume conserve permissions ; copie entre volumes hérite du dossier cible.
