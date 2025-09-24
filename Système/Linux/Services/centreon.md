# 📊 Centreon — Aide-mémoire

## 🏗️ Architecture & Concepts

### Composants Centreon
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Central Web   │    │    Poller       │    │    Agents       │
│  (Interface)    │    │   (Engine)      │    │  (NRPE/SNMP)    │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ • PHP Web UI    │◄──►│ • Engine        │◄──►│ • NRPE Daemon   │
│ • Configuration │    │ • Broker        │    │ • SNMP Agent    │
│ • Base de donnée│    │ • Gorgone       │    │ • NSClient++    │
│ • Gorgone       │    │ • Plugins       │    │ • CMA (nouveau) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Flux de Données
1. **Configuration** : Web UI → Base MySQL → Génération fichiers .cfg
2. **Supervision** : Engine → Plugins → Agents → Résultats
3. **Collecte** : Broker → Métriques → RRD/InfluxDB → Graphiques

## ⚙️ Cycle Configuration

### Workflow Configuration
```bash
# 1. Configuration via Web UI
Configuration → Hosts/Services → Modification

# 2. Génération & Test
Configuration → Pollers → Export Configuration
├── Test       # Simulation (debug)
├── Génération # Vérification fichiers .cfg
└── Déplace+Reload # Application production

# 3. Fichiers générés
/usr/share/centreon/filesGeneration/  # Temporaire
/etc/centreon-engine/               # Production
```

### États Services
| Code | État | Description |
|------|------|-------------|
| **0** | OK | Service fonctionnel |
| **1** | WARNING | Avertissement |
| **2** | CRITICAL | Critique |
| **3** | UNKNOWN | État indéterminé |
| **4** | PENDING | En attente de vérification |

### États SOFT vs HARD
- **SOFT** : État temporaire, en cours de confirmation (pas de notification)
- **HARD** : État confirmé après X tentatives (notifications possibles)

## 🔌 Protocoles de Supervision

### SNMP (Simple Network Management Protocol)
```bash
# Ports & Versions
UDP 161   # Requêtes SNMP
UDP 162   # Traps SNMP
v1/v2c    # Communautés (public/private)
v3        # Authentification + chiffrement

# Configuration Linux (/etc/snmp/snmpd.conf)
agentAddress udp:161                    # Port d'écoute
rocommunity public 192.168.1.0/24      # Communauté lecture
rwcommunity private 192.168.1.10       # Communauté écriture

# Vue SNMP (restriction OID)
view SystemOnly included .1.3.6.1.2.1.1
access readonlyGroup "" any noauth exact SystemOnly none none

# Tests SNMP
snmpwalk -v2c -c public 192.168.1.10 .1.3.6.1.2.1.1.1.0  # sysDescr
snmpget -v2c -c public 192.168.1.10 .1.3.6.1.2.1.1.3.0   # uptime
```

### NRPE (Nagios Remote Plugin Executor)
```bash
# Port & Configuration
TCP 5666  # Port par défaut

# Configuration NRPE (/etc/nagios/nrpe.cfg)
allowed_hosts=127.0.0.1,192.168.1.5     # IPs autorisées
server_address=0.0.0.0                  # Écoute toutes interfaces

# Commandes définies
command[check_load]=/usr/lib/nagios/plugins/check_load -w 15,10,5 -c 30,25,20
command[check_disk]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /

# Test depuis poller
check_nrpe -H 192.168.1.10 -c check_load
/usr/lib/nagios/plugins/check_nrpe -H 192.168.1.10 -c check_disk
```

### NSClient++ (Windows)
```powershell
# Installation & Configuration
Install-WindowsFeature -Name SNMP -IncludeManagementTools
# ou télécharger NSClient++ depuis nscp.org

# Configuration (C:\Program Files\NSClient++\nsclient.ini)
[/settings/default]
allowed hosts = 127.0.0.1,192.168.1.5
password = MonMotDePasse

[/modules]
CheckSystem = enabled        # CPU, RAM, Disque
CheckDisk = enabled         # Vérifications disques
NRPE = enabled              # Support NRPE
NSCAClient = enabled        # Mode passif

# Interface Web
https://localhost:8443      # Interface web locale
```

## 🔍 Plugins & Sondes

### Emplacements Plugins
```bash
# Plugins Nagios standard
/usr/lib/nagios/plugins/
/usr/lib64/nagios/plugins/

# Plugins Centreon
/usr/lib/centreon/plugins/

# Plugins personnalisés
/usr/local/nagios/plugins/
```

### Plugins Centreon (Nouvelle Génération)
```bash
# Structure commande
centreon_linux_snmp.pl --plugin=os::linux::snmp::plugin \
    --mode=cpu \
    --hostname=192.168.1.10 \
    --snmp-community=public \
    --snmp-version=2c \
    --warning-average=80 \
    --critical-average=90

# Options communes
--help                      # Aide
--list-plugin              # Lister plugins disponibles
--list-mode               # Lister modes pour plugin
--verbose                 # Mode verbeux
```

### Format Sortie Plugin
```
ÉTAT MESSAGE | perfdata
├── ÉTAT: OK, WARNING, CRITICAL, UNKNOWN
├── MESSAGE: Description lisible
└── perfdata: métriques pour graphiques

Exemple:
CPU OK - 15% used | cpu=15%;80;90;0;100
```

## 🎯 Macros & Variables

### Types de Macros
| Type | Format | Exemple | Usage |
|------|--------|---------|-------|
| **Standard** | `$VARIABLE$` | `$HOSTADDRESS$` | Variables Nagios intégrées |
| **Ressources** | `$USERx$` | `$USER1$` | Chemins globaux |
| **Arguments** | `$ARGx$` | `$ARG1$` | Paramètres commandes |
| **Personnalisées** | `$_HOSTnom$` | `$_HOSTSNMP$` | Macros utilisateur |

### Macros Essentielles
```bash
# Macros hôte
$HOSTADDRESS$               # Adresse IP hôte
$HOSTNAME$                  # Nom hôte
$HOSTSTATE$                 # État hôte (UP, DOWN, etc.)

# Macros service
$SERVICEDESC$               # Description service
$SERVICESTATE$              # État service
$SERVICEOUTPUT$             # Sortie plugin

# Macros ressources (/etc/centreon/resource.cfg)
$USER1$=/usr/lib/nagios/plugins        # Chemin plugins
$CENTREONPLUGINS$=/usr/lib/centreon/plugins
```

### Définition Macros Personnalisées
```bash
# Dans configuration hôte
_SNMPCOMM    public         # $_HOSTSNMPCOMM$
_HTTPPORT    8080          # $_HOSTHTTPPORT$

# Dans configuration service  
_THRESHOLD   80,90         # $_SERVICETHRESHOLD$
_DISK        /var          # $_SERVICEDISK$
```

## 🌐 Pollers & Architecture Distribuée

### Types d'Architecture
- **Central** : Tout sur un serveur (Web + Engine + DB)
- **Distributed** : Central + Pollers distants
- **Redundant** : Haute disponibilité avec failover

### Configuration Poller
```bash
# 1. Installation poller
yum install centreon-poller-centreon-engine centreon-poller-gorgone

# 2. Configuration Gorgone (/etc/centreon-gorgone/config.d/40-gorgoned.yaml)
name: poller-paris
description: Poller site Paris
gorgone:
  gorgonecore:
    id: 2
    external_com_type: tcp
    external_com_path: "*:5556"
    authorized_clients:
      - key: "clé-publique-central"
    privkey: "/var/lib/centreon-gorgone/.keys/rsa_priv.key"
    pubkey: "/var/lib/centreon-gorgone/.keys/rsa_pub.key"

# 3. Échange clés SSH Central ↔ Poller
ssh-copy-id centreon@poller-ip

# 4. Redémarrage services
systemctl restart gorgoned centengine
systemctl enable gorgoned centengine
```

### Communication Central-Poller
```bash
# Protocols
SSH         # Configuration & fichiers
ZMQ         # Communication temps réel (port 5556)
BBDO        # Broker Binary Data Objects

# Test connectivité
su - centreon
ssh poller-hostname
```

## 📊 Supervision Passive

### SNMP Traps
```bash
# Configuration récepteur traps
/etc/snmp/snmptrapd.conf:
authCommunity log,execute,net public
traphandle default /usr/bin/centreon_trap_send

# Traitement traps
centreon_trap_recv → centreon_trap_send → Centreon DB
```

### NSCA (Nagios Service Check Acceptor)
```bash
# Configuration NSCA
/etc/nsca.cfg:
server_port=5667
decryption_method=1
password=MonMotDePasse

# Envoi résultat passif
echo "hostname;service;status;output" | send_nsca -H central-ip -c nsca.cfg
```

## 🔒 ACL & Sécurité

### Structure ACL Centreon
```
ACL Group (Groupe d'accès)
├── ACL Resources (Objets accessibles)
│   ├── Hosts
│   ├── Services  
│   └── Host Categories
├── ACL Menus (Menus UI accessibles)
│   ├── Configuration
│   ├── Monitoring
│   └── Administration
└── ACL Actions (Actions autorisées)
    ├── Service Check
    ├── Host Check
    └── Generate Configuration
```

### Bonnes Pratiques Sécurité
```bash
# SNMP v3 avec authentification
snmpget -v3 -l authPriv -u user -a SHA -A authpass -x AES -X privpass \
    192.168.1.10 .1.3.6.1.2.1.1.1.0

# NRPE avec SSL
nrpe_args="-n"              # Désactiver SSL (pas recommandé)
# Préférer: Certificats SSL + allowed_hosts restrictifs

# NSClient++ sécurisé
password = ComplexPassword123!
allowed hosts = 192.168.1.5/32
use ssl = true
```

## 🔧 Maintenance & Dépannage

### Downtime & Acquittements
```bash
# Planifier maintenance (Downtime)
Monitoring → Downtimes → Add
├── Type: Host/Service
├── Durée: Fixe/Flexible  
├── Notification: Oui/Non
└── Commentaire: Raison maintenance

# Acquitter alerte
Monitoring → Event Logs → Acknowledge
├── Sticky: Maintenir acquittement
├── Notify: Notifier contacts
└── Persistent: Survivre redémarrage
```

### Commandes de Debug
```bash
# Logs Centreon
journalctl -u centreon-engine -f
journalctl -u centreon-broker -f
journalctl -u gorgoned -f
tail -f /var/log/centreon-engine/centengine.log

# Test plugins manuellement
su - centreon-engine
/usr/lib/nagios/plugins/check_ping -H 192.168.1.10 -w 100,10% -c 500,50%

# Vérification configuration
centreon-engine -v /etc/centreon-engine/centengine.cfg
```

### Résolution Problèmes Courants
| Problème | Cause Probable | Solution |
|----------|----------------|----------|
| **Poller non connecté** | Clés SSH/ZMQ | Régénérer clés, restart gorgoned |
| **Plugin UNKNOWN** | Chemin/permissions | Vérifier $USER1$, chmod +x |
| **Pas de notifications** | Configuration contacts | Vérifier periods, enable notifications |
| **Graphiques vides** | Broker/RRD | Restart broker, check perfdata |
| **NRPE Connection refused** | Firewall/config | Check port 5666, allowed_hosts |

## 📈 Métriques & Performance

### Optimisation Base de Données
```sql
-- Purge données anciennes
DELETE FROM data_bin WHERE ctime < UNIX_TIMESTAMP(DATE_SUB(NOW(), INTERVAL 30 DAY));

-- Index performance
SHOW INDEX FROM data_bin;
ANALYZE TABLE data_bin;
```

### Tuning Engine
```bash
# /etc/centreon-engine/centengine.cfg
check_result_reaper_frequency=5     # Fréquence collecte résultats
max_concurrent_checks=400           # Checks simultanés max
cached_host_check_horizon=900       # Cache résultats hôte
cached_service_check_horizon=900    # Cache résultats service
```

### Surveillance Centreon
```bash
# Métriques importantes à surveiller
- CPU/RAM poller
- Latence checks
- Queue Broker
- Taille base données
- Espace disque RRD

# Commandes utiles
centreon -u admin -p passwd -a POLLERLIST
centreon -u admin -p passwd -a POLLERGENERATE -v 1
```

## 📋 Check-list Déploiement

### Installation Nouvelle Instance
- [ ] **OS** : RHEL/CentOS/Alma Linux à jour
- [ ] **Dépôts** : Centreon, EPEL, Remi configurés
- [ ] **MariaDB** : Optimisée (my.cnf)
- [ ] **Apache/PHP** : Configuration selon prérequis
- [ ] **Firewall** : Ports ouverts (80, 443, 5666, 161/162)
- [ ] **SELinux** : Configuré ou désactivé selon politique

### Post-Installation
- [ ] **Wizard** : Configuration initiale Web
- [ ] **Pollers** : Ajout et configuration 
- [ ] **Plugins** : Installation packs nécessaires
- [ ] **Modèles** : Import templates métier
- [ ] **ACL** : Configuration accès utilisateurs
- [ ] **Monitoring** : Supervision Centreon lui-même

---
**💡 Memo** : Générer config après modifications, tester plugins en CLI, vérifier ACL pour accès, surveiller logs pour debug !