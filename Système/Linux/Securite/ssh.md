# 🔐 SSH — Aide-mémoire

## 🏗️ Architecture SSH

### Protocole & Sécurité
- **Port par défaut** : 22 (TCP)
- **Chiffrement** : Asymétrique (échange clés) + Symétrique (données)
- **Authentification** : Mot de passe, clés publiques, certificats
- **Intégrité** : HMAC (Hash-based Message Authentication Code)

### Composants
```
SSH Client ←→ SSH Server (sshd)
    ↓              ↓
~/.ssh/        /etc/ssh/
├── id_rsa     ├── sshd_config
├── id_rsa.pub ├── ssh_host_*_key
├── config     └── authorized_keys
└── known_hosts
```

## 🔑 Gestion Clés SSH

### Génération Clés
```bash
# RSA (legacy, compatible)
ssh-keygen -t rsa -b 4096 -C "email@domain.com"

# Ed25519 (moderne, recommandée)
ssh-keygen -t ed25519 -C "email@domain.com"

# ECDSA (équilibre sécurité/performance)
ssh-keygen -t ecdsa -b 521 -C "email@domain.com"

# Options utiles
ssh-keygen -t ed25519 -f ~/.ssh/id_server -N "passphrase"
```

### Types de Clés & Sécurité
| Type | Taille | Sécurité | Performance | Compatibilité |
|------|--------|----------|-------------|---------------|
| **Ed25519** | - | Excellente | Très rapide | OpenSSH 6.5+ |
| **ECDSA** | P-256/384/521 | Très bonne | Rapide | Moderne |
| **RSA** | 2048/4096 | Bonne | Lente | Universelle |
| **DSA** | 1024 | Faible | Lente | Legacy (éviter) |

### Déploiement Clés
```bash
# Méthode automatique (recommandée)
ssh-copy-id user@hostname
ssh-copy-id -i ~/.ssh/id_custom.pub user@hostname

# Méthode manuelle
cat ~/.ssh/id_rsa.pub | ssh user@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Via SCP (si déjà connecté)
scp ~/.ssh/id_rsa.pub user@hostname:~/.ssh/authorized_keys
```

## ⚙️ Configuration Client

### Fichier ~/.ssh/config
```ssh
# Configuration globale
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    TCPKeepAlive yes
    
# Serveur spécifique
Host prod-server
    HostName 192.168.1.100
    User admin
    Port 2222
    IdentityFile ~/.ssh/id_prod
    ProxyJump bastion.company.com
    
# Bastion/Jump host
Host bastion
    HostName bastion.company.com
    User jumpuser
    Port 22
    IdentitiesOnly yes
    
# Wildcard pour sous-domaines
Host *.company.com
    User myuser
    IdentityFile ~/.ssh/id_company
    
# Connexion via proxy
Host internal-server
    HostName 10.0.1.50
    ProxyCommand ssh -W %h:%p bastion
```

### Options Client Utiles
```bash
# Connexion avec options
ssh -p 2222 user@hostname                    # Port personnalisé
ssh -i ~/.ssh/custom_key user@hostname       # Clé spécifique
ssh -L 8080:localhost:80 user@hostname       # Port forwarding local
ssh -R 9000:localhost:8000 user@hostname     # Port forwarding distant
ssh -D 1080 user@hostname                    # SOCKS proxy
ssh -X user@hostname                         # X11 forwarding
ssh -A user@hostname                         # Agent forwarding

# Options de sécurité
ssh -o StrictHostKeyChecking=no user@hostname    # Skip host verification (dangereux)
ssh -o UserKnownHostsFile=/dev/null user@hostname  # Pas de known_hosts
ssh -o PasswordAuthentication=no user@hostname   # Clés seulement
```

## 🖥️ Configuration Serveur

### Fichier /etc/ssh/sshd_config
```ssh
# Paramètres de base
Port 22
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Authentification
PermitRootLogin prohibit-password    # no | yes | prohibit-password | forced-commands-only
PubkeyAuthentication yes
PasswordAuthentication no            # Désactiver mots de passe
PermitEmptyPasswords no
AuthenticationMethods publickey      # ou "publickey,password"

# Limitations
MaxAuthTries 3                       # Tentatives max
MaxSessions 10                       # Sessions simultanées max
ClientAliveInterval 300              # Keepalive client (secondes)
ClientAliveCountMax 2                # Tentatives keepalive

# Restrictions utilisateurs
AllowUsers user1 user2
DenyUsers baduser
AllowGroups ssh-users
DenyGroups no-ssh

# Sécurité réseau
ListenAddress 0.0.0.0               # Toutes interfaces (ou IP spécifique)
AddressFamily any                    # IPv4 + IPv6
```

### Durcissement Serveur SSH
```ssh
# /etc/ssh/sshd_config - Configuration sécurisée
Port 2222                           # Port non-standard
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key

# Authentification renforcée
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
UsePAM yes

# Limitations strictes
MaxAuthTries 2
MaxSessions 5
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 0

# Algorithmes sécurisés
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group16-sha512
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com

# Restrictions
AllowUsers deployuser adminuser
PermitTunnel no
AllowAgentForwarding yes
AllowTcpForwarding yes
GatewayPorts no
X11Forwarding no
```

## 🔧 Gestion Service SSH

### Commandes Système
```bash
# Gestion service
systemctl status sshd
systemctl start sshd
systemctl stop sshd
systemctl restart sshd
systemctl reload sshd               # Recharge config sans couper connexions
systemctl enable sshd               # Démarrage automatique

# Test configuration
sshd -t                            # Vérifier syntaxe config
sshd -T                            # Afficher config effective

# Monitoring
ss -tuln | grep :22               # Vérifier écoute port
journalctl -u sshd -f             # Logs en temps réel
tail -f /var/log/auth.log          # Logs authentification (Debian)
```

### Permissions Fichiers Critiques
```bash
# Permissions correctes
chmod 700 ~/.ssh/                          # Répertoire SSH utilisateur
chmod 600 ~/.ssh/authorized_keys           # Clés autorisées
chmod 600 ~/.ssh/id_rsa                    # Clé privée
chmod 644 ~/.ssh/id_rsa.pub               # Clé publique
chmod 644 ~/.ssh/known_hosts              # Hôtes connus
chmod 600 ~/.ssh/config                   # Config client

# Permissions serveur
chmod 600 /etc/ssh/ssh_host_*_key         # Clés privées serveur
chmod 644 /etc/ssh/ssh_host_*_key.pub     # Clés publiques serveur
chmod 644 /etc/ssh/sshd_config            # Config serveur
```

## 🌐 Tunnels & Port Forwarding

### Local Port Forwarding
```bash
# Rediriger port local vers distant
ssh -L 8080:target:80 user@gateway
# Accès: localhost:8080 → gateway → target:80

ssh -L 3306:db.internal:3306 user@bastion
# Accès: localhost:3306 → bastion → db.internal:3306

# Bind sur interface spécifique
ssh -L 192.168.1.10:8080:target:80 user@gateway
```

### Remote Port Forwarding
```bash
# Exposer port local sur serveur distant
ssh -R 9000:localhost:8000 user@server
# server:9000 → localhost:8000

# Exemple: exposer serveur de dev
ssh -R 80:localhost:3000 user@public-server
# public-server:80 → localhost:3000 (dev server)
```

### Dynamic Port Forwarding (SOCKS)
```bash
# Proxy SOCKS
ssh -D 1080 user@proxy-server
# Configurer navigateur: SOCKS proxy localhost:1080

# Avec compression et keepalive
ssh -D 1080 -C -N -f user@proxy-server
# -C: compression, -N: pas de commande, -f: background
```

### Tunnels Persistants
```bash
# AutoSSH pour tunnels robustes
autossh -M 20000 -L 8080:target:80 user@gateway
# -M: port monitoring pour reconnexion auto

# Systemd service pour tunnel permanent
# /etc/systemd/system/ssh-tunnel.service
[Unit]
Description=SSH Tunnel
After=network.target

[Service]
ExecStart=/usr/bin/ssh -N -L 8080:target:80 user@gateway
Restart=always
User=tunneluser

[Install]
WantedBy=multi-user.target
```

## 📁 Transfert Fichiers

### SCP (Secure Copy)
```bash
# Fichier local → distant
scp file.txt user@hostname:/path/destination/
scp -r directory/ user@hostname:/path/

# Distant → local  
scp user@hostname:/path/file.txt ./
scp -r user@hostname:/path/dir/ ./local-dir/

# Options utiles
scp -P 2222 file.txt user@hostname:/path/    # Port personnalisé
scp -i ~/.ssh/key file.txt user@hostname:/path/  # Clé spécifique
scp -C file.txt user@hostname:/path/         # Compression
scp -v file.txt user@hostname:/path/         # Verbose
```

### SFTP (SSH File Transfer Protocol)
```bash
# Session interactive
sftp user@hostname
sftp -P 2222 user@hostname                   # Port personnalisé

# Commandes SFTP
put file.txt                                 # Upload
get remote-file.txt                          # Download  
put -r local-dir/                           # Upload récursif
get -r remote-dir/                          # Download récursif
ls                                          # Lister distant
lls                                         # Lister local
cd /path                                    # Changer répertoire distant
lcd /local/path                             # Changer répertoire local
mkdir newdir                                # Créer répertoire
rm file.txt                                 # Supprimer fichier
```

### Rsync via SSH
```bash
# Synchronisation avec SSH
rsync -avz -e ssh /local/path/ user@hostname:/remote/path/
rsync -avz -e "ssh -p 2222" source/ user@host:/dest/

# Options courantes
-a: archive (préserve permissions, dates)
-v: verbose
-z: compression
-P: progress + partial
--delete: supprimer fichiers destination absents source
```

## 🖥️ Clients SSH Windows

### OpenSSH Intégré Windows
```powershell
# Installation (Windows 10 1809+)
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Utilisation
ssh user@hostname
scp file.txt user@hostname:/path/
sftp user@hostname
```

### Clients Graphiques Populaires
| Client | Type | Points Forts |
|--------|------|--------------|
| **PuTTY** | SSH/Telnet | Léger, configuration avancée |
| **WinSCP** | SCP/SFTP | Interface graphique transferts |
| **MobaXterm** | Suite complète | X11, tunnels, sessions multiples |
| **mRemoteNG** | Multi-protocoles | Gestion sessions centralisée |
| **Termius** | Moderne | Mobile + desktop, sync cloud |

### Configuration PuTTY
```
Session:
├── Host Name: server.domain.com
├── Port: 22
├── Connection type: SSH
└── Saved Sessions: [nom]

Connection > SSH > Auth:
├── Private key file: .ppk (converti avec PuTTYgen)
└── Allow agent forwarding: ✓

Connection > SSH > Tunnels:
├── Local ports: L8080 target:80
└── Remote ports: R9000 localhost:8000
```

## 🔐 Sécurité Avancée

### SSH Agent & Forwarding
```bash
# Démarrer agent SSH
ssh-agent bash                              # Nouveau shell avec agent
eval $(ssh-agent)                           # Dans shell courant

# Ajouter clés à l'agent
ssh-add ~/.ssh/id_rsa
ssh-add -l                                  # Lister clés chargées
ssh-add -D                                  # Supprimer toutes clés

# Agent forwarding (utiliser clés locales sur serveur distant)
ssh -A user@hostname
# Permet d'utiliser clés locales pour connexions depuis le serveur
```

### Certificats SSH
```bash
# Générer CA (Certificate Authority)
ssh-keygen -t ed25519 -f ca_key -C "SSH CA"

# Signer clé utilisateur
ssh-keygen -s ca_key -I "john.doe" -n john -V +52w id_rsa.pub
# -I: identifiant, -n: principals, -V: validité

# Configuration serveur pour certificats
TrustedUserCAKeys /etc/ssh/ca_key.pub
AuthorizedPrincipalsFile /etc/ssh/principals/%u
```

### Multi-Factor Authentication
```bash
# Configuration 2FA avec Google Authenticator
# /etc/ssh/sshd_config
AuthenticationMethods publickey,keyboard-interactive
ChallengeResponseAuthentication yes

# Installation libpam-google-authenticator
apt install libpam-google-authenticator
google-authenticator                         # Configuration utilisateur

# /etc/pam.d/sshd
auth required pam_google_authenticator.so
```

## 📊 Monitoring & Logs

### Journaux SSH
```bash
# Logs système
journalctl -u sshd                          # Systemd logs
tail -f /var/log/auth.log                   # Debian/Ubuntu
tail -f /var/log/secure                     # RedHat/CentOS

# Filtrage logs
grep "Failed password" /var/log/auth.log
grep "Accepted publickey" /var/log/auth.log
journalctl -u sshd --since "1 hour ago"
```

### Fail2Ban Protection
```bash
# Installation et configuration
apt install fail2ban

# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 600
```

## 🚨 Troubleshooting

### Problèmes Courants
| Problème | Symptôme | Solution |
|----------|----------|----------|
| **Connection refused** | Port fermé | Vérifier service: `systemctl status sshd` |
| **Permission denied** | Auth échoue | Vérifier clés et permissions |
| **Host key verification** | Clé changée | Mettre à jour `known_hosts` |
| **Too many authentication failures** | Limite atteinte | Réduire clés ou augmenter `MaxAuthTries` |

### Debug Connexion
```bash
# Client verbose
ssh -v user@hostname                        # Niveau 1 (basique)
ssh -vv user@hostname                       # Niveau 2 (détaillé)
ssh -vvv user@hostname                      # Niveau 3 (très détaillé)

# Test serveur
sshd -d -p 2223                            # Mode debug sur port alternatif
sshd -T                                     # Test configuration

# Vérification réseau
telnet hostname 22                          # Test connectivité port
nmap -p 22 hostname                         # Scan port
```

### Diagnostic Permissions
```bash
# Vérifier permissions critiques
ls -la ~/.ssh/
ls -la ~/.ssh/authorized_keys
stat ~/.ssh/id_rsa

# Test clés
ssh-keygen -l -f ~/.ssh/id_rsa.pub          # Fingerprint clé publique
ssh-add -l                                  # Clés dans agent
```

## 💡 Bonnes Pratiques

### Sécurisation
- ✅ **Désactiver root login** et mots de passe
- ✅ **Changer port par défaut** (security by obscurity)
- ✅ **Utiliser clés Ed25519** pour nouvelles installations
- ✅ **Configurer fail2ban** contre brute force
- ✅ **Limiter utilisateurs autorisés** (AllowUsers/AllowGroups)
- ✅ **Monitorer logs** régulièrement

### Performance
- ✅ **Utiliser ControlMaster** pour connexions multiples
- ✅ **Activer compression** (-C) sur liens lents
- ✅ **Configurer KeepAlive** pour connexions stables
- ✅ **Utiliser algorithmes modernes** (ChaCha20-Poly1305)

### Gestion
- ✅ **Documenter configuration** SSH complexe
- ✅ **Sauvegarder clés privées** de manière sécurisée
- ✅ **Implémenter rotation clés** régulière
- ✅ **Tester configuration** avant déploiement production

---
**💡 Memo** : Ed25519 > RSA, clés > mots de passe, `ssh-copy-id` pour déployer, `-vvv` pour debug !