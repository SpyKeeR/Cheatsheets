# 📧 Outlook — Aide-mémoire

## 🏗️ Architecture & Concepts

### Types de Comptes
| Type | Protocol | Stockage | Synchronisation | Usage |
|------|----------|----------|-----------------|-------|
| **Exchange** | MAPI/EWS | Serveur + cache local | Bidirectionnelle | Enterprise |
| **IMAP** | IMAP | Serveur + copie locale | Mail + dossiers | Professionnel |
| **POP3** | POP3 | Local uniquement | Téléchargement seul | Personnel basic |
| **Outlook.com** | EAS/Graph | Cloud | Temps réel | Microsoft 365 |

### Fichiers de Données
```
Types de fichiers Outlook :
├── .pst (Personal Storage Table) : Données locales
├── .ost (Offline Storage Table) : Cache Exchange/IMAP
├── .oab (Offline Address Book) : Carnet adresses hors ligne
└── .nk2 (AutoComplete) : Suggestions destinataires
```

## ⚙️ Configuration & Profils

### Gestion Profils
```cmd
# Accès configuration
control mlcfg32.cpl          # Panneau config mail
outlook.exe /profiles        # Sélecteur profils au démarrage
outlook.exe /profile "MonProfil"  # Profil spécifique

# Création profil
Panneau de configuration > Courrier > Afficher les profils
```

### Paramètres Avancés
| Paramètre | Location | Description |
|-----------|----------|-------------|
| **AutoDiscover** | Config compte | Découverte auto serveur Exchange |
| **Mode cache** | Paramètres compte | Synchronisation hors ligne |
| **Taille cache** | Paramètres avancés | Limite fichier .ost |
| **Dossiers partagés** | Paramètres dossier | Boîtes aux lettres délégées |

## 🔧 Raccourcis Clavier Essentiels

### Navigation
- `Ctrl + 1-8` : Basculer entre vues (Mail, Calendrier, Contacts...)
- `Ctrl + Shift + I` : Boîte de réception
- `Ctrl + Shift + O` : Éléments envoyés
- `F3` ou `Ctrl + E` : Rechercher
- `Ctrl + Shift + F` : Recherche avancée

### Composition & Gestion
- `Ctrl + N` : Nouveau message
- `Ctrl + R` : Répondre
- `Ctrl + Shift + R` : Répondre à tous
- `Ctrl + F` : Transférer
- `Ctrl + Enter` : Envoyer
- `F7` : Vérification orthographe

### Organisation
- `Ctrl + Shift + V` : Déplacer vers dossier
- `Delete` : Supprimer (vers éléments supprimés)
- `Shift + Delete` : Suppression définitive
- `Ctrl + U` : Marquer comme non lu
- `Ctrl + Q` : Marquer comme lu

## 🚨 Dépannage & Réparation

### Réparation Fichiers PST
```cmd
# Outil ScanPST (MAPI)
# Localisation selon version Office :
C:\Program Files\Microsoft Office\OfficeXX\scanpst.exe
C:\Program Files (x86)\Microsoft Office\OfficeXX\scanpst.exe

# Utilisation ScanPST :
1. Fermer Outlook complètement
2. Lancer scanpst.exe en tant qu'administrateur
3. Sélectionner fichier .pst à vérifier
4. Cliquer "Démarrer" pour analyse
5. Si erreurs détectées : "Réparer"
6. Sauvegarde automatique créée (.bak)
```

### Modes de Démarrage
```cmd
# Modes diagnostics
outlook.exe /safe            # Mode sans échec (pas de compléments)
outlook.exe /safe:1          # Mode sans échec + extensions
outlook.exe /safe:2          # Mode sans échec + lecture seule
outlook.exe /safe:3          # Mode sans échec + pas de extensions/lecture

# Autres options
outlook.exe /resetnavpane    # Reset volet navigation
outlook.exe /resetformregions  # Reset régions formulaires
outlook.exe /cleanreminders # Nettoyer rappels
outlook.exe /resetfolders    # Reset dossiers par défaut
```

### Reconstruction Profil
```cmd
# Méthode complète
1. Panneau config > Courrier > Afficher profils
2. Supprimer profil existant
3. Ajouter nouveau profil
4. Configurer compte (AutoDiscover recommandé)
5. Tester envoi/réception

# Conservation données
- Exporter .pst avant suppression profil
- Sauvegarder paramètres règles/signatures
- Noter serveurs/ports si config manuelle
```

## 📊 Tests Connectivité

### Tests Intégrés
```cmd
# Test connectivité Exchange
Outlook > Fichier > Paramètres compte > Tester paramètres compte

# Informations connexion
Ctrl + clic droit sur icône Outlook (systray)
> État de la connexion
> Test de configuration automatique
```

### Outils Microsoft
| Outil | Usage | Accès |
|-------|-------|-------|
| **Microsoft Remote Connectivity Analyzer** | Tests Exchange Online | testconnectivity.microsoft.com |
| **Exchange Online PowerShell** | Diagnostics avancés | Connect-ExchangeOnline |
| **Office Config Analyzer** | Analyse configuration | Support Microsoft |

### PowerShell Diagnostics
```powershell
# Test connectivité Exchange Online
Test-NetConnection outlook.office365.com -Port 443
Test-NetConnection smtp.office365.com -Port 587

# Informations compte
Get-ExoMailbox -Identity user@domain.com | fl
Get-MessageTrace -RecipientAddress user@domain.com -StartDate (Get-Date).AddHours(-1)

# Test AutoDiscover
Test-OutlookWebServices -Identity user@domain.com -MailboxCredential $cred
```

## 🔍 Journalisation & Diagnostics

### Activation Logs
```
1. Fichier > Options > Avancé
2. Section "Autre" > Activer la journalisation (dépannage)
3. Redémarrer Outlook
4. Reproduire problème
5. Logs générés dans %TEMP%\Outlook Logging\
```

### Emplacements Logs
| Type Log | Emplacement | Contenu |
|----------|-------------|---------|
| **Outlook Logging** | `%TEMP%\Outlook Logging\` | Activité générale |
| **ETW Logs** | Event Viewer > Applications | Erreurs système |
| **AutoDiscover** | Registre + logs réseau | Découverte serveur |
| **MAPI Logs** | CDO/Redemption | Appels MAPI |

## 🔌 Compléments & Performance

### Gestion Compléments
```
Fichier > Options > Compléments :
├── Compléments actifs : Chargés et fonctionnels
├── Compléments inactifs : Désactivés manuellement  
├── Compléments COM : Extensions système
└── Compléments désactivés : Problématiques (crash/lenteur)

Actions disponibles :
- Activer/Désactiver sélectivement
- Gérer temps démarrage
- Supprimer définitivement
```

### Optimisation Performance
- ✅ **Limiter taille cache** : 1-12 mois selon usage
- ✅ **Désactiver compléments** non essentiels
- ✅ **Archivage automatique** : Fichiers .pst séparés
- ✅ **Nettoyage régulier** : Éléments supprimés, brouillons
- ✅ **Mode cache Exchange** : Synchronisation optimisée

## 📁 Gestion Données

### Sauvegarde & Export
```
Fichier > Ouvrir et exporter > Importer/Exporter :

Types export :
├── Exporter vers un fichier (.pst)
├── Exporter vers .csv (contacts)  
├── Archiver dossiers (par date)
└── Créer règle archivage auto

Éléments à sauvegarder :
├── Messages et dossiers
├── Contacts et groupes
├── Calendrier et tâches
├── Règles de messagerie
├── Signatures
└── Paramètres de compte
```

### Archivage & Nettoyage
```cmd
# Archivage manuel
Fichier > Outils de nettoyage > Archiver

# Archivage automatique
Fichier > Options > Avancé > Paramètres archivage auto

# Nettoyage boîte aux lettres  
Fichier > Outils de nettoyage > Nettoyer la boîte aux lettres
- Recherche éléments volumineux
- Éléments anciens par dossier
- Conflit synchronisation
```

## 🌐 Exchange Online & Microsoft 365

### Fonctionnalités Cloud
| Fonctionnalité | Description | Avantage |
|----------------|-------------|----------|
| **Shared Mailbox** | BAL partagée équipe | Collaboration |
| **In-Place Archive** | Archive illimitée | Conformité |
| **Retention Policies** | Conservation automatique | Légal |
| **eDiscovery** | Recherche légale | Audit |
| **Anti-Spam/Malware** | Protection intégrée | Sécurité |

### Configuration Moderne
```powershell
# Connexion Exchange Online
Install-Module ExchangeOnlineManagement
Connect-ExchangeOnline -UserPrincipalName admin@domain.com

# Gestion utilisateurs
Get-EXOMailbox -Identity user@domain.com
Set-EXOMailbox -Identity user@domain.com -DeliverToMailboxAndForward $true
New-EXOMailbox -UserPrincipalName newuser@domain.com -DisplayName "New User"

# Règles transport
Get-TransportRule | fl Name,State,Description
```

## 🚀 Fonctionnalités Avancées

### Règles et Alertes
```
Règles côté client vs serveur :
├── Client (fichier .rwz) : Outlook ouvert requis
├── Serveur (Exchange) : Toujours actives
└── Hybride : Combinaison des deux

Types actions :
├── Déplacement dossier
├── Transfert/redirection  
├── Marquage/catégorie
├── Réponse automatique
└── Suppression/archivage
```

### Délégation & Partage
```
Niveaux accès délégation :
├── Reviewer : Lecture seule
├── Author : Lire + créer
├── Editor : Lire + créer + modifier
├── Owner : Contrôle total
└── Custom : Permissions spécifiques

Configuration :
Fichier > Paramètres compte > Accès délégué
```

## 💡 Bonnes Pratiques

### Organisation
- ✅ **Structure dossiers** : Hiérarchie logique (projets, années)
- ✅ **Règles automatiques** : Tri automatique par expéditeur/sujet
- ✅ **Catégories couleur** : Classification visuelle
- ✅ **Recherche efficace** : Utiliser critères avancés

### Performance
- ✅ **Nettoyage régulier** : Suppression définitive périodique
- ✅ **Archivage planifié** : Déplacement messages anciens
- ✅ **Compléments minimal** : Garder seulement l'essentiel
- ✅ **Cache optimisé** : Taille adaptée à la connectivité

### Sécurité
- ✅ **Signatures** : Cohérence professionnelle
- ✅ **Chiffrement** : Messages sensibles (S/MIME)
- ✅ **Sauvegarde** : Export PST régulier
- ✅ **Authentification** : MFA Exchange Online

---
**💡 Memo** : ScanPST pour .pst corrompus, mode /safe pour dépannage, archivage auto pour performance !