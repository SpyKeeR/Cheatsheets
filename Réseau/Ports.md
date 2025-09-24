# ⚓ Ports & Services Réseau — Aide-mémoire

## 📋 Catégories de Ports

### Classification IANA
| Plage | Nom | Description | Usage |
|-------|-----|-------------|-------|
| **0-1023** | Well-Known | Services système standards | Privilèges admin requis |
| **1024-49151** | Registered | Applications enregistrées | Usage commercial/public |
| **49152-65535** | Dynamic/Private | Éphémères | Connexions sortantes temporaires |

### Concepts Clés
- **Port Source** : Port côté client (généralement éphémère)
- **Port Destination** : Port côté serveur (service écouté)
- **Socket** : Combinaison IP:Port unique
- **Binding** : Processus d'écoute sur un port spécifique

## 🌐 Services Web & Transfert

### HTTP/HTTPS & Web
| Port | Protocol | Service | Sécurité | Usage |
|------|----------|---------|----------|-------|
| **80** | TCP | HTTP | ❌ Clair | Web standard |
| **443** | TCP | HTTPS | ✅ TLS/SSL | Web sécurisé |
| **8080** | TCP | HTTP Alt | ❌ Clair | Proxy, dev |
| **8443** | TCP | HTTPS Alt | ✅ TLS/SSL | Admin web |

### Transfert de Fichiers
| Port | Protocol | Service | Chiffré | Notes |
|------|----------|---------|---------|-------|
| **20** | TCP | FTP-Data | ❌ | Canal données FTP |
| **21** | TCP | FTP-Control | ❌ | Canal contrôle FTP |
| **22** | TCP | SSH/SFTP | ✅ | Shell + transfert sécurisé |
| **69** | UDP | TFTP | ❌ | Transfert simple (PXE boot) |
| **989** | TCP | FTPS-Data | ✅ | FTP sur TLS |
| **990** | TCP | FTPS-Control | ✅ | FTP sécurisé |

## 📧 Services Messagerie

### Envoi (SMTP)
| Port | Protocol | Service | Chiffrement | Usage |
|------|----------|---------|-------------|-------|
| **25** | TCP | SMTP | ❌ Clair | Serveur à serveur |
| **465** | TCP | SMTPS | ✅ SSL implicite | Client → serveur (legacy) |
| **587** | TCP | SMTP | ✅ STARTTLS | Client → serveur (moderne) |

### Réception
| Port | Protocol | Service | Chiffrement | Type |
|------|----------|---------|-------------|------|
| **110** | TCP | POP3 | ❌ Clair | Téléchargement |
| **995** | TCP | POP3S | ✅ SSL/TLS | POP3 sécurisé |
| **143** | TCP | IMAP | ❌ Clair | Synchronisation |
| **993** | TCP | IMAPS | ✅ SSL/TLS | IMAP sécurisé |

## 🔍 Services Réseau & Infrastructure

### Résolution & Configuration
| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| **53** | UDP/TCP | DNS | Résolution noms (UDP normal, TCP si >512 octets) |
| **67** | UDP | DHCP Server | Attribution IP automatique |
| **68** | UDP | DHCP Client | Réception config DHCP |
| **123** | UDP | NTP | Synchronisation temps |
| **514** | UDP | Syslog | Journalisation centralisée |

### Surveillance & Gestion
| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| **161** | UDP | SNMP | Monitoring équipements réseau |
| **162** | UDP | SNMP Trap | Alertes SNMP |
| **514** | TCP | RSH | Remote Shell (obsolète) |
| **623** | UDP | IPMI | Gestion serveurs hors bande |

## 🏢 Services Entreprise & Annuaires

### Active Directory & Authentification
| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| **389** | TCP/UDP | LDAP | Annuaire non sécurisé |
| **636** | TCP | LDAPS | LDAP over SSL |
| **88** | TCP/UDP | Kerberos | Authentification AD |
| **464** | TCP/UDP | Kerberos Change | Changement mots de passe |
| **3268** | TCP | Global Catalog | LDAP global AD |
| **3269** | TCP | Global Catalog SSL | GC sécurisé |

### Services Windows
| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| **135** | TCP | RPC Endpoint | Mapper services RPC |
| **137** | UDP | NetBIOS Name | Résolution noms NetBIOS |
| **138** | UDP | NetBIOS Datagram | Services datagramme |
| **139** | TCP | NetBIOS Session | Sessions NetBIOS |
| **445** | TCP | SMB/CIFS | Partages fichiers Windows |

## 🖥️ Accès Distant & Terminaux

### Protocoles Courants
| Port | Protocol | Service | Sécurité | Plateforme |
|------|----------|---------|----------|------------|
| **22** | TCP | SSH | ✅ Chiffré | Unix/Linux |
| **23** | TCP | Telnet | ❌ Clair | Multiplateforme (obsolète) |
| **3389** | TCP/UDP | RDP | ✅ Chiffré | Windows |
| **5900** | TCP | VNC | ⚠️ Faible | Multiplateforme |
| **5901+** | TCP | VNC Displays | ⚠️ Faible | Display :1, :2... |

### Tunneling & VPN
| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| **1194** | UDP | OpenVPN | VPN SSL standard |
| **1723** | TCP | PPTP | VPN Microsoft (obsolète) |
| **4500** | UDP | IPSec NAT-T | IPSec avec NAT traversal |
| **500** | UDP | IKE/IPSec | Négociation IPSec |

## 🗄️ Bases de Données

### SGBD Populaires
| Port | Protocol | Service | Fournisseur |
|------|----------|---------|-------------|
| **1433** | TCP | MSSQL | Microsoft SQL Server |
| **1521** | TCP | Oracle TNS | Oracle Database |
| **3306** | TCP | MySQL | MySQL/MariaDB |
| **5432** | TCP | PostgreSQL | PostgreSQL |
| **27017** | TCP | MongoDB | MongoDB |
| **6379** | TCP | Redis | Redis Cache |

## 🎮 Applications Spécialisées

### Multimédia & Streaming
| Port | Protocol | Service | Usage |
|------|----------|---------|-------|
| **554** | TCP/UDP | RTSP | Streaming temps réel |
| **1935** | TCP | RTMP | Flash streaming |
| **5060** | UDP | SIP | VoIP signalisation |
| **5004** | UDP | RTP | VoIP données |

### Gaming & P2P
| Port | Protocol | Service | Usage |
|------|----------|---------|-------|
| **6881-6999** | TCP | BitTorrent | P2P file sharing |
| **25565** | TCP | Minecraft | Serveur jeu par défaut |
| **27015** | UDP | Steam/Source | Gaming Valve |

## 🔒 Sécurité & Analyse

### Ports Sensibles à Surveiller
```
Accès distant non sécurisé :
├── 23 (Telnet) - Remplacer par SSH
├── 513-514 (rlogin/rsh) - Obsolètes  
├── 5900+ (VNC) - Chiffrer ou tunneler
└── 5432 (PostgreSQL) - Ne pas exposer

Ports couramment scannés :
├── 21, 22, 23 (FTP, SSH, Telnet)
├── 25, 53, 80, 443 (Mail, DNS, Web)  
├── 135, 139, 445 (Windows SMB)
├── 1433, 3306, 5432 (Databases)
└── 3389 (RDP)
```

### Bonnes Pratiques
- ✅ **Changer ports par défaut** pour services exposés
- ✅ **Firewall** : Deny all + Allow spécifique
- ✅ **Fail2Ban** : Protection brute force
- ❌ **Éviter** Telnet, FTP, HTTP pour données sensibles
- ✅ **VPN/Bastion** : Accès indirect aux services internes

## 🛠️ Outils de Diagnostic

### Scan & Test Ports
```bash
# Nmap (scan réseau)
nmap -sT target                 # TCP connect scan
nmap -sU target                 # UDP scan  
nmap -sS target                 # SYN stealth scan
nmap -A target                  # Détection OS + services

# Netcat (test manuel)
nc -zv hostname 80              # Test port TCP
nc -u -zv hostname 123          # Test port UDP

# Telnet (test basique)
telnet hostname 80              # Test connectivité TCP

# PowerShell Windows
Test-NetConnection -Port 443 hostname
```

### Écoute Locale
```bash
# Linux
ss -tuln                        # Ports écoute (moderne)
netstat -tuln                   # Ports écoute (legacy)
lsof -i :80                     # Processus sur port 80

# Windows  
netstat -an                     # Tous ports
netstat -ano | findstr :80      # Port spécifique avec PID
Get-NetTCPConnection -LocalPort 80  # PowerShell
```

## 📊 Ports par Catégorie d'Usage

### Administration Système
```
SSH/Telnet : 22, 23
SNMP       : 161, 162  
Syslog     : 514
NTP        : 123
IPMI       : 623
```

### Services Utilisateurs
```
Web        : 80, 443, 8080, 8443
Mail       : 25, 110, 143, 465, 587, 993, 995  
DNS        : 53
DHCP       : 67, 68
```

### Bases de Données
```
MySQL      : 3306
PostgreSQL : 5432
MSSQL      : 1433
Oracle     : 1521
MongoDB    : 27017
Redis      : 6379
```

### Accès Distant
```
SSH        : 22
RDP        : 3389  
VNC        : 5900+
OpenVPN    : 1194
```

---
**💡 Memo** : Port = adresse service • Well-known 0-1023 = admin requis • Toujours sécuriser/chiffrer si exposition Internet !