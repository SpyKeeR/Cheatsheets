# 🖱️ Windows GUI — Raccourcis & Outils — Aide-mémoire

## ⌨️ Raccourcis Clavier Essentiels

### Navigation & Fenêtres
| Raccourci | Action | Usage |
|-----------|--------|-------|
| `Win + Tab` | Task View (bureau virtuel) | Navigation moderne |
| `Alt + Tab` | Basculer applications | Commutation classique |
| `Win + Ctrl + D` | Créer bureau virtuel | Multi-desktop |
| `Win + Ctrl + ←/→` | Changer bureau virtuel | Navigation bureaux |
| `Win + D` | Afficher bureau | Minimiser tout |
| `Win + M` | Minimiser toutes fenêtres | Alternative Win+D |
| `Win + ↑/↓/←/→` | Ancrer fenêtre | Snap windows |

### Système & Administration
| Raccourci | Action | Usage |
|-----------|--------|-------|
| `Win + X` | Menu Power User | Accès admin rapide |
| `Win + R` | Exécuter | Lancer commandes |
| `Ctrl + Shift + Esc` | Gestionnaire tâches | Monitoring |
| `Win + I` | Paramètres Windows 10/11 | Configuration moderne |
| `Win + L` | Verrouiller session | Sécurité |
| `Win + Pause` | Propriétés système | Infos système |
| `Win + Shift + S` | Capture écran | Outil capture |

### Raccourcis Avancés
```
Win + Ctrl + Shift + B    # Reset driver graphique (écran noir)
Alt + F4                  # Fermer application/arrêter PC (sur bureau)
Ctrl + Alt + Del          # Écran sécurisé (changement mdp, verrouillage)
Win + G                   # Xbox Game Bar (enregistrement écran)
Win + V                   # Historique presse-papiers (si activé)
Win + .                   # Émojis et symboles
```

## 🚀 Boîte de Dialogue Exécuter (Win+R)

### Outils Système Essentiels
| Commande | Outil | Description |
|----------|-------|-------------|
| `msconfig` | Configuration système | Boot, services, démarrage |
| `regedit` | Éditeur registre | Modifications registre |
| `msinfo32` | Informations système | Hardware, software, config |
| `dxdiag` | Diagnostic DirectX | Graphiques, son, multimédia |
| `perfmon` | Moniteur performance | Compteurs système |
| `eventvwr` | Observateur événements | Journaux système |
| `control` | Panneau configuration | Interface classique |

### Consoles MMC (Microsoft Management Console)
```
mmc                       # Console vide (ajout composants)
services.msc              # Services Windows
compmgmt.msc              # Gestion ordinateur
lusrmgr.msc               # Utilisateurs/groupes locaux
devmgmt.msc               # Gestionnaire périphériques
diskmgmt.msc              # Gestion disques
taskschd.msc              # Planificateur tâches
```

### Sécurité & Certificats
```
secpol.msc                # Stratégies sécurité locale
gpedit.msc                # Éditeur stratégies groupe (Pro+)
certlm.msc                # Certificats ordinateur local
certmgr.msc               # Certificats utilisateur
wf.msc                    # Pare-feu Windows avec fonctions avancées
```

### Réseau & Communication
```
ncpa.cpl                  # Connexions réseau
inetcpl.cpl               # Propriétés Internet
firewall.cpl              # Pare-feu Windows (basique)
telephon.cpl              # Options téléphonie/modem
```

## 📂 Chemins Système Importants

### Variables Environnement
| Variable | Chemin Typique | Usage |
|----------|----------------|-------|
| `%temp%` | `C:\Users\{user}\AppData\Local\Temp` | Fichiers temporaires |
| `%appdata%` | `C:\Users\{user}\AppData\Roaming` | Données applications |
| `%localappdata%` | `C:\Users\{user}\AppData\Local` | Données locales |
| `%programdata%` | `C:\ProgramData` | Données partagées |
| `%systemroot%` | `C:\Windows` | Dossier Windows |
| `%system32%` | `C:\Windows\System32` | Binaires système |
| `%programfiles%` | `C:\Program Files` | Applications 64-bit |

### Dossiers Système Critiques
```
%windir%\Prefetch         # Cache préchargement apps
%windir%\System32\drivers # Pilotes système
%windir%\SysWOW64         # Binaires 32-bit (sur x64)
%windir%\Temp             # Fichiers temp système
%windir%\Logs             # Journaux Windows
%windir%\SoftwareDistribution # Cache Windows Update
```

## 🔧 Outils Ligne Commande depuis GUI

### Accès Élevé (Administrateur)
- **Ctrl+Shift+Entrée** dans Exécuter = Élévation UAC
- **Shift+Clic droit** sur exe = "Exécuter en tant qu'admin"
- **Win+X puis A** = PowerShell Admin (Windows 10)
- **Win+X puis I** = Windows Terminal Admin (Windows 11)

### Outils Diagnostics GUI
```
winver                    # Version Windows détaillée
systeminfo                # Infos système (CLI dans GUI)
driverquery               # Liste pilotes installés
pnputil /enum-drivers     # Énumérer pilotes PnP
sigverif                  # Vérification signatures fichiers
```

### Gestionnaires Spécialisés
```
hdwwiz                    # Assistant ajout matériel
devmgmt.msc               # Gestionnaire périphériques
diskmgmt.msc              # Gestion disques et volumes
compmgmt.msc              # Console gestion complète
```

## 🎛️ Panneaux de Configuration (CPL)

### Principaux Applets
| Commande | Panel | Description |
|----------|-------|-------------|
| `appwiz.cpl` | Programmes et fonctionnalités | Désinstallation |
| `desk.cpl` | Propriétés affichage | Résolution, thèmes |
| `sysdm.cpl` | Propriétés système | Nom PC, domaine, perf |
| `timedate.cpl` | Date et heure | Fuseau, synchro temps |
| `powercfg.cpl` | Options alimentation | Gestion énergie |
| `mmsys.cpl` | Sons et périph. audio | Configuration audio |
| `joy.cpl` | Contrôleurs de jeu | Joysticks, manettes |

### Paramètres Utilisateur
```
userpasswords2            # Comptes utilisateurs avancé
netplwiz                  # Gestion comptes (alternative)
control userpasswords     # Comptes utilisateurs simple
```

## 🖥️ Outils Monitoring & Performance

### Interfaces Temps Réel
```
taskmgr                   # Gestionnaire tâches (moderne)
perfmon                   # Moniteur performances
perfmon /res              # Vue ressources temps réel
resmon                    # Moniteur ressources détaillé
```

### Journaux & Événements
```
eventvwr                  # Observateur événements
eventvwr /c:Application   # Log Application direct
eventvwr /c:System       # Log Système direct
eventvwr /c:Security      # Log Sécurité direct
```

### Informations Matériel
```
dxdiag                    # Diagnostic complet multimédia
msinfo32                  # Informations système complètes
devmgmt.msc               # Périphériques et pilotes
```

## 🔍 Recherche & Indexation

### Windows Search (Cortana/Recherche)
- **Win** puis taper = Recherche universelle
- **Win+S** = Ouvrir recherche explicitement
- **Win+Q** = Cortana (Windows 10)

### Recherche Avancée
```
# Opérateurs recherche Windows
kind:document             # Documents seulement
type:.pdf                 # Fichiers PDF
modified:today            # Modifiés aujourd'hui
size:>1MB                 # Taille supérieure à 1MB
author:john               # Par auteur (métadonnées)
```

## 🎯 Modes d'Ouverture Spéciaux

### Modes Sans Échec & Démarrage
- **F8** (legacy) = Menu démarrage avancé
- **Shift+Redémarrer** = Options avancées (moderne)
- **msconfig > Boot** = Configuration démarrage
- **Win+R > shutdown /r /o /f /t 0** = Redémarrage options avancées

### Modes Application
```
# Navigateurs
iexplore -private         # Internet Explorer mode privé
chrome --incognito        # Chrome incognito
firefox -private-window   # Firefox mode privé

# Office
winword /safe             # Word mode sans échec
excel /safe               # Excel mode sans échec
powerpnt /safe            # PowerPoint mode sans échec
```

## 🛠️ Outils Intégrés Cachés

### Utilitaires Système
```
charmap                   # Table caractères
calc                      # Calculatrice
notepad                   # Bloc-notes
wordpad                   # WordPad
mspaint                   # Paint
snippingtool              # Outil capture (legacy)
```

### Outils Réseau GUI
```
ncpa.cpl                  # Adaptateurs réseau
inetcpl.cpl               # Options Internet
mstsc                     # Connexion bureau distant
```

### Diagnostics Avancés
```
reliability               # Moniteur fiabilité
perfmon /report           # Rapport performance système
```

## 📋 Contexte Clic Droit Avancé

### Extensions Shift+Clic Droit
- **"Ouvrir dans Terminal"** (Windows 11)
- **"Ouvrir fenêtre PowerShell ici"** (Windows 10)
- **"Copier comme chemin"** (obtenir chemin complet)

### Menu Administrateur (Win+X)
```
Applications et fonctionnalités
Options d'alimentation
Observateur d'événements
Gestionnaire de périphériques
Connexions réseau
Gestion des disques
Gestionnaire des tâches
Paramètres (Windows 10+)
```

## 💡 Astuces Productivité GUI

### Personnalisation Interface
- **Épingler** outils fréquents dans barre tâches
- **Raccourcis bureau** vers outils admin (MMC, etc.)
- **Menu Démarrer** : organiser par catégories
- **Barre d'outils** : accès rapide Explorateur

### Gestion Fenêtres Efficace
```
Win + 1,2,3...           # Lancer apps épinglées barre tâches
Win + Shift + 1,2,3...    # Nouvelle instance app
Alt + Espace             # Menu système fenêtre
Win + Home              # Minimiser autres fenêtres
F11                      # Plein écran (applications)
```

### Raccourcis Explorateur
```
Ctrl + Shift + N         # Nouveau dossier
Alt + ↑                  # Dossier parent
Alt + ←/→               # Navigation historique
F2                       # Renommer
F4                       # Barre d'adresse
F5                       # Actualiser
Ctrl + L                 # Sélectionner barre adresse
```

## 🔧 Maintenance Rapide GUI

### Nettoyage Système
```
cleanmgr                 # Nettoyage disque
cleanmgr /sageset:1      # Configuration profil nettoyage
dfrg.msc                 # Défragmenteur (interface)
```

### Gestion Services
```
services.msc             # Console services
msconfig                 # Démarrage et services
```

### Résolution Problèmes
```
msdt                     # Diagnostics Microsoft génériques
control /name Microsoft.Troubleshooting  # Centre résolution problèmes
```

---
**💡 Memo** : Win+X pour menu admin, Ctrl+Shift+Entrée pour élévation UAC, Win+R est votre ami pour tout lancer !