# 🖥️ Remote Desktop Protocol (RDP) — Aide-mémoire

## 💻 Concepts Fondamentaux

### Qu'est-ce que RDP ?
- **Protocole propriétaire Microsoft** pour accès distant graphique
- **Port par défaut** : 3389 (TCP)
- **Chiffrement** : RC4, AES (selon version)
- **Authentification** : NLA (Network Level Authentication) recommandée
- **Versions** : RDP 6.0+ (Windows Vista+), RDP 10+ (Windows 10+)

### Architecture RDP
```
Client RDP ←→ Réseau ←→ Serveur RDP ←→ Session Windows
     ↓                      ↓              ↓
  mstsc.exe            Terminal Services  Desktop/Apps
```

## 🚀 Client RDP (mstsc)

### Connexions de Base
```cmd
# Connexion simple
mstsc                               # Interface graphique
mstsc /v:192.168.1.100             # IP directe
mstsc /v:server.domain.com         # FQDN
mstsc /v:server.domain.com:3390    # Port personnalisé

# Options courantes
mstsc /admin                       # Session console (Windows Server)
mstsc /f                          # Plein écran
mstsc /w:1920 /h:1080             # Résolution spécifique
mstsc /multimon                   # Support multi-moniteurs
```

### Fichiers de Connexion (.rdp)
```cmd
# Utilisation fichiers RDP
mstsc connection.rdp              # Charger profil sauvegardé
mstsc /edit connection.rdp        # Éditer profil existant

# Création via ligne commande
mstsc /v:server /save:connection.rdp
```

### Exemple Fichier .rdp
```ini
screen mode id:i:2                # Mode écran (1=fenêtre, 2=plein écran)
use multimon:i:1                  # Multi-moniteurs activé
desktopwidth:i:1920              # Largeur bureau
desktopheight:i:1080             # Hauteur bureau
session bpp:i:32                 # Profondeur couleur (bits par pixel)
winposstr:s:0,1,0,0,800,600     # Position/taille fenêtre
compression:i:1                   # Compression activée
keyboardhook:i:2                 # Redirection clavier (2=plein écran uniquement)
audiocapturemode:i:1             # Capture audio microphone
videoplaybackmode:i:1            # Accélération vidéo
connection type:i:7              # Type connexion (7=LAN haute vitesse)
networkautodetect:i:1            # Auto-détection réseau
bandwidthautodetect:i:1          # Auto-détection bande passante
drivestoredirect:s:C:\;D:\       # Redirection lecteurs locaux
```

## ⚙️ Configuration Serveur (Cible)

### Activation RDP Windows
```cmd
# Via Registre (nécessite redémarrage)
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

# Via PowerShell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

# Via Interface
# Paramètres > Système > Bureau à distance > Activer
```

### Configuration Pare-feu Windows
```cmd
# Règle firewall standard
netsh advfirewall firewall add rule name="Remote Desktop" dir=in action=allow protocol=TCP localport=3389

# Port personnalisé (sécurité)
netsh advfirewall firewall add rule name="RDP Custom Port" dir=in action=allow protocol=TCP localport=3390

# Supprimer règle
netsh advfirewall firewall delete rule name="Remote Desktop"
```

### Modification Port RDP
```cmd
# Changer port dans registre
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 3390 /f

# Redémarrer service Terminal Services
net stop "Remote Desktop Services"
net start "Remote Desktop Services"

# Ou via PowerShell
Restart-Service -Name "TermService" -Force
```

## 🔐 Sécurité RDP

### Network Level Authentication (NLA)
```cmd
# Activer NLA (recommandé)
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f

# Vérifier état NLA
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication
```

### Gestion Utilisateurs RDP
```cmd
# Ajouter utilisateur au groupe RDP
net localgroup "Remote Desktop Users" username /add
net localgroup "Utilisateurs du Bureau à distance" username /add  # Version française

# Lister membres groupe RDP  
net localgroup "Remote Desktop Users"

# Supprimer utilisateur
net localgroup "Remote Desktop Users" username /delete
```

### Bonnes Pratiques Sécurité
- ✅ **Changer port par défaut** (3389 → autre)
- ✅ **Activer NLA** pour pré-authentification
- ✅ **Comptes forts** + MFA si possible
- ✅ **VPN** pour accès externe (pas RDP direct Internet)
- ✅ **Groupe spécifique** pour utilisateurs RDP
- ✅ **Monitoring connexions** dans Event Viewer
- ✅ **Certificats SSL** pour chiffrement renforcé

## 📊 Sessions & Gestion

### Commandes Terminal Services
```cmd
# Lister sessions actives
quser                            # Sessions utilisateurs
qwinsta                         # Sessions détaillées avec ID
query session                   # Alternative qwinsta

# Informations session courante  
whoami                          # Utilisateur courant
echo %SESSIONNAME%              # Nom session (Console, RDP-Tcp#X)
quser %USERNAME%                # Détails session utilisateur

# Gestion sessions
logoff                          # Déconnecter session courante
logoff 2                        # Déconnecter session ID 2
logoff username                 # Déconnecter utilisateur spécifique
rwinsta 2                      # Réinitialiser session ID 2
```

### Sessions Multiples
```cmd
# Autoriser sessions multiples (Windows Server)
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fSingleSessionPerUser /t REG_DWORD /d 0 /f

# Limite sessions simultanées
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v MaxInstanceCount /t REG_DWORD /d 5 /f
```

## 🛠️ Optimisation & Performance

### Paramètres Performance
| Paramètre | Description | Impact |
|-----------|-------------|---------|
| **Profondeur couleur** | 15/16/24/32 bits | Bande passante |
| **Compression** | Algorithme compression | CPU vs réseau |
| **Cache bitmap** | Cache images localement | Mémoire vs réseau |
| **Thèmes visuels** | Effets Windows | CPU/GPU |

### Configuration Performance (.rdp)
```ini
# Performance optimisée
compression:i:1                   # Compression activée
bitmapcachepersistedisk:i:1      # Cache bitmap sur disque
full window drag:i:0             # Pas d'aperçu fenêtre
menu anims:i:0                   # Pas d'animations menu
themes:i:0                       # Thèmes désactivés
wallpaper:i:0                    # Pas de fond d'écran
disable cursor setting:i:1        # Curseur standard
disable full window drag:i:1      # Contour fenêtre seulement

# Qualité élevée (LAN rapide)
session bpp:i:32                 # 32 bits couleur
connection type:i:7              # LAN haute vitesse
videoplaybackmode:i:1            # Accélération vidéo
```

### RemoteApp (Applications Publiées)
```cmd
# Lancer application spécifique
mstsc /v:server /remoteapplicationmode:1 /remoteapplicationprogram:calc.exe

# Via fichier .rdp
remoteapplicationmode:i:1
remoteapplicationprogram:s:calc.exe
remoteapplicationname:s:Calculator
```

## 🔧 Redirection Ressources

### Redirection Lecteurs
```ini
# Dans fichier .rdp
drivestoredirect:s:*             # Tous lecteurs
drivestoredirect:s:C:\;D:\       # Lecteurs spécifiques
drivestoredirect:s:              # Aucune redirection
```

### Redirection Périphériques
```ini
# Audio
audiomode:i:0                    # Lecture sur serveur
audiomode:i:1                    # Lecture sur client
audiomode:i:2                    # Pas d'audio
audiocapturemode:i:1             # Capture microphone

# Imprimantes
redirectprinters:i:1             # Redirection activée
redirectcomports:i:1             # Ports COM
redirectsmartcards:i:1           # Cartes à puces

# Presse-papier
redirectclipboard:i:1            # Partage presse-papier
```

## 📋 Troubleshooting

### Problèmes Courants
| Symptôme | Cause Probable | Solution |
|----------|---------------|----------|
| **Connexion refusée** | RDP désactivé | Activer RDP + firewall |
| **Authentification échoue** | NLA/credentials | Vérifier NLA + comptes |
| **Session lente** | Bande passante | Optimiser paramètres |
| **Déconnexions fréquentes** | Timeout/réseau | Ajuster keep-alive |
| **Écran noir** | Pilotes graphiques | Mode sans échec |

### Event Viewer (Logs RDP)
```
Emplacements logs importants :
├── Applications and Services Logs
│   ├── Microsoft > Windows > TerminalServices-LocalSessionManager
│   ├── Microsoft > Windows > TerminalServices-RemoteConnectionManager  
│   └── Microsoft > Windows > RemoteDesktopServices-RdpCoreTS
└── Windows Logs > Security (Event IDs 4624, 4625, 4634, 4647)
```

### Tests Connectivité
```cmd
# Test port RDP
telnet server.domain.com 3389
Test-NetConnection server.domain.com -Port 3389  # PowerShell

# Vérifier service
sc query "TermService"
Get-Service -Name "TermService"  # PowerShell

# Test résolution DNS
nslookup server.domain.com
ping server.domain.com
```

## 🌐 Alternatives & Outils

### Clients RDP Tiers
| Client | Plateforme | Avantages |
|--------|------------|-----------|
| **mstsc** | Windows | Natif, complet |
| **Remote Desktop** | macOS/iOS | Microsoft officiel |
| **FreeRDP** | Linux | Open source |
| **Remmina** | Linux | Multi-protocoles |
| **Royal TSX** | macOS | Professionnel |

### RDP vs Autres Protocoles
| Protocole | Port | Plateforme | Usage |
|-----------|------|------------|-------|
| **RDP** | 3389 | Windows | Bureau Windows |
| **VNC** | 5900+ | Multi-plateforme | Bureau distant générique |
| **SSH** | 22 | Unix/Linux | Shell + tunneling |
| **TeamViewer** | 5938 | Multi-plateforme | Support/assistance |

## 💡 Conseils d'Utilisation

### Raccourcis Clavier RDP
- `Ctrl+Alt+End` : Équivalent Ctrl+Alt+Del
- `Alt+Page Up/Down` : Basculer entre applications
- `Ctrl+Alt+Break` : Basculer plein écran/fenêtre
- `Ctrl+Alt+Plus` : Capture écran serveur
- `Ctrl+Alt+Home` : Barre connexion RDP

### Scripts Utiles
```batch
@echo off
REM Connexion RDP rapide avec paramètres
mstsc /v:%1 /f /multimon
if %errorlevel% neq 0 echo Erreur connexion RDP

REM Usage: rdp.bat server.domain.com
```

```powershell
# Test connectivité RDP PowerShell
function Test-RDP {
    param([string]$Server, [int]$Port = 3389)
    Test-NetConnection -ComputerName $Server -Port $Port
}

# Usage: Test-RDP "server.domain.com"
```

---
**💡 Memo** : Port 3389 par défaut, NLA pour sécurité, /admin pour console, fichiers .rdp pour automatisation !