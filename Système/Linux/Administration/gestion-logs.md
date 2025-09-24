# 📋 Gestion Logs Linux — Aide-mémoire

## 🗂️ Architecture Logging Linux

### Composants Système
```
Applications → journald → Journaux binaires (/run/log/journal)
            ↓
         rsyslog → Fichiers texte (/var/log/*)
            ↓
        logrotate → Archivage/compression
```

### Emplacements Standards
| Fichier | Contenu | Usage |
|---------|---------|-------|
| `/var/log/messages` | Messages système généraux | Diagnostic global |
| `/var/log/syslog` | Tout syslog (Debian/Ubuntu) | Vue d'ensemble |
| `/var/log/auth.log` | Authentifications | Sécurité, connexions |
| `/var/log/kern.log` | Messages kernel | Hardware, drivers |
| `/var/log/cron.log` | Tâches planifiées | Jobs cron |
| `/var/log/boot.log` | Messages boot | Démarrage système |
| `/var/log/dmesg` | Messages kernel boot | Hardware détection |

## 🔍 journalctl (Systemd)

### Commandes de Base
```bash
# Consultation générale
journalctl                      # Tous les logs depuis début
journalctl -f                   # Suivi temps réel (follow)
journalctl -r                   # Ordre antichronologique (reverse)
journalctl -n 50               # 50 dernières entrées
journalctl --no-pager          # Sans pagination

# Par période
journalctl --since "2023-12-01"     # Depuis date
journalctl --since "1 hour ago"     # Dernière heure
journalctl --since "yesterday"      # Depuis hier
journalctl --until "2023-12-31"     # Jusqu'à date
journalctl --since "09:00" --until "17:00"  # Plage horaire
```

### Filtrage Avancé
```bash
# Par service/unit
journalctl -u sshd              # Service SSH
journalctl -u nginx.service     # Service nginx
journalctl -u "systemd-*"       # Pattern matching
journalctl --user              # Services utilisateur

# Par processus
journalctl _PID=1234           # PID spécifique
journalctl _COMM=sshd          # Nom commande
journalctl _EXE=/usr/sbin/sshd # Exécutable

# Par priorité
journalctl -p err              # Erreurs et plus graves
journalctl -p warning          # Warnings et plus graves
journalctl -p 0..3             # Emergencies à erreurs
```

### Niveaux de Priorité Syslog
| Niveau | Nom | Description | Usage |
|--------|-----|-------------|-------|
| **0** | emerg | Urgence système | Système inutilisable |
| **1** | alert | Alerte immédiate | Action immédiate requise |
| **2** | crit | Critique | Erreurs critiques |
| **3** | err | Erreur | Messages d'erreur |
| **4** | warning | Avertissement | Conditions d'avertissement |
| **5** | notice | Notice | Normal mais significatif |
| **6** | info | Information | Messages informatifs |
| **7** | debug | Debug | Messages de débogage |

### Filtrage par Boot
```bash
journalctl -b                  # Boot actuel
journalctl -b -1               # Boot précédent
journalctl -b 0                # Boot actuel (explicite)
journalctl --list-boots        # Lister boots disponibles
```

### Options de Format
```bash
journalctl -o short            # Format par défaut
journalctl -o verbose          # Tous champs
journalctl -o json             # Format JSON
journalctl -o json-pretty      # JSON formaté
journalctl -o cat              # Messages seulement
journalctl -o export           # Format binaire export
```

## ⚙️ Configuration journald

### Fichier Principal
```bash
# /etc/systemd/journald.conf
[Journal]
Storage=persistent             # auto, volatile, persistent, none
Compress=yes                   # Compression logs
SplitMode=uid                  # Séparer par utilisateur

# Limites espace disque
SystemMaxUse=4G               # Max espace total
SystemKeepFree=1G             # Espace libre minimum
SystemMaxFileSize=128M        # Taille max par fichier
SystemMaxFiles=100            # Nombre max fichiers

# Rétention temporelle
MaxRetentionSec=1month        # Conservation maximum
MaxFileSec=1week              # Rotation fichiers
```

### Persistance Logs
```bash
# Activer persistance (survit aux redémarrages)
sudo mkdir -p /var/log/journal
sudo systemctl restart systemd-journald

# Vérifier stockage
journalctl --disk-usage       # Espace utilisé
journalctl --vacuum-size=1G   # Nettoyer jusqu'à 1G
journalctl --vacuum-time=1month  # Supprimer > 1 mois
```

## 📝 rsyslog (Logging Traditionnel)

### Configuration Principale
```bash
# /etc/rsyslog.conf structure
$ModLoad imuxsock              # Module socket Unix
$ModLoad imklog                # Module kernel logging

# Règles: facility.priority    destination
*.info;mail.none;authpriv.none;cron.none    /var/log/messages
authpriv.*                                  /var/log/auth.log
mail.*                                      -/var/log/maillog
cron.*                                      /var/log/cron.log
*.emerg                                     :omusrmsg:*
```

### Facilities Courantes
| Facility | Description | Usage |
|----------|-------------|--------|
| **auth** | Authentification | Login, su, sudo |
| **authpriv** | Auth privée | SSH, login privé |
| **cron** | Tâches planifiées | Jobs cron/at |
| **daemon** | Services système | Services background |
| **kern** | Kernel | Messages noyau |
| **mail** | Système mail | Postfix, sendmail |
| **user** | Processus utilisateur | Applications user |
| **local0-7** | Usage personnalisé | Applications custom |

### Syntaxe Règles rsyslog
```bash
# Format: facility.priority    destination

# Exemples pratiques
*.info                 /var/log/messages      # Toutes facilities >= info
mail.*                 /var/log/maillog       # Tout de mail
*.=warn               /var/log/warnings       # Exactement warn
*.err;kern.none       /var/log/errors         # Erreurs sauf kernel
auth,authpriv.*       /var/log/auth.log      # Auth combinées

# Destinations
/var/log/file         # Fichier local
-/var/log/file        # Fichier async (performance)
@server:514           # UDP distant
@@server:514          # TCP distant
:omusrmsg:*           # Tous utilisateurs connectés
|/path/to/script      # Pipe vers script
```

### Logging Distant
```bash
# Serveur central (réception)
$ModLoad imudp
$UDPServerRun 514
$UDPServerAddress 0.0.0.0

# Client (envoi)
*.* @logserver:514            # UDP
*.* @@logserver:514           # TCP (fiable)

# Avec templates
$template RemoteHost,"/var/log/remote/%HOSTNAME%/%programname%"
*.* ?RemoteHost
```

## 🔄 logrotate (Rotation Logs)

### Configuration Globale
```bash
# /etc/logrotate.conf
weekly                        # Fréquence rotation
rotate 4                      # Conserver 4 rotations
create                        # Créer nouveau fichier
dateext                       # Extension date
compress                      # Compresser anciens logs
delaycompress                 # Compresser au cycle suivant
```

### Configuration Spécifique
```bash
# /etc/logrotate.d/monapp
/var/log/monapp/*.log {
    daily                     # Rotation quotidienne
    rotate 30                 # Garder 30 jours
    compress                  # Compresser
    delaycompress            # Pas immédiatement
    missingok                # OK si fichier absent
    notifempty               # Pas de rotation si vide
    create 644 root root     # Permissions nouveau fichier
    postrotate               # Script après rotation
        systemctl reload monapp > /dev/null 2>&1 || true
    endscript
}
```

### Commandes logrotate
```bash
# Test configuration
logrotate -d /etc/logrotate.d/nginx    # Debug/dry-run
logrotate -f /etc/logrotate.d/nginx    # Forcer rotation
logrotate -v /etc/logrotate.conf       # Mode verbose

# Status
cat /var/lib/logrotate/status          # Dernières rotations
```

## 🧪 Test & Génération Logs

### logger (Injection Messages)
```bash
# Usage basique
logger "Message de test"               # Syslog standard
logger -p mail.info "Test mail"       # Facility + priority
logger -t monapp "Message application" # Tag personnalisé

# Options avancées
logger -i "Message avec PID"          # Inclure PID
logger -s "Message sur stderr aussi"  # Aussi sur stderr
logger -f /path/file                   # Depuis fichier
echo "test" | logger                   # Depuis pipe

# Exemples pratiques
logger -p auth.warning "Tentative accès suspect"
logger -p cron.info "Job backup terminé"
logger -p local0.err "Erreur application custom"
```

### Tests de Fonctionnement
```bash
# Tester chaîne complète
logger -p auth.info "Test auth" && journalctl -u rsyslog -f

# Vérifier réception logs
tail -f /var/log/auth.log
journalctl -f -p info

# Test logging distant
logger -p local0.info "Test remote" && ssh logserver "tail /var/log/messages"
```

## 📊 Monitoring & Analyse

### Surveillance Espace Disque
```bash
# Taille logs
du -sh /var/log/*              # Taille par fichier
du -sh /var/log/journal/       # Taille journald
df -h /var/log                 # Espace partition

# Nettoyage journald
journalctl --vacuum-size=500M  # Limiter à 500MB
journalctl --vacuum-time=2weeks # Supprimer > 2 semaines
journalctl --vacuum-files=10   # Garder 10 fichiers max
```

### Analyse Statistique
```bash
# Fréquence par service
journalctl --since "today" | awk '{print $5}' | sort | uniq -c | sort -nr

# Erreurs les plus fréquentes
journalctl -p err --since "today" --no-pager | grep -o 'systemd\[[0-9]*\].*' | sort | uniq -c

# Pics d'activité
journalctl --since "today" -o short-iso | awk '{print $1}' | cut -d'T' -f2 | cut -d':' -f1 | sort | uniq -c
```

### Recherche & Filtrage
```bash
# Recherche pattern
journalctl | grep -i "failed"
journalctl -u sshd | grep "Failed password"
journalctl --grep="error|fail" -i     # Regex insensible casse

# Extraction données
journalctl -o json | jq '.MESSAGE'    # Messages JSON
journalctl --output-fields=MESSAGE,_PID,_COMM --no-pager
```

## 🔧 Centralisation Logs

### Architecture Centralisée
```
Serveurs Application → rsyslog → Serveur Log Central
                    ╲             ╱
                     → ELK Stack ←
                    ╱             ╲
                Monitoring    Long-term Storage
```

### ELK Stack Basique
```bash
# Filebeat (collecteur)
/etc/filebeat/filebeat.yml:
output.elasticsearch:
  hosts: ["elasticsearch:9200"]
filebeat.inputs:
- type: log
  paths: ["/var/log/*.log"]

# Logstash (traitement)
input { beats { port => 5044 } }
filter {
  if [fields][logtype] == "syslog" {
    grok { match => { "message" => "%{SYSLOGTIMESTAMP:timestamp} %{IPORHOST:host} %{DATA:program}: %{GREEDYDATA:message}" } }
  }
}
output { elasticsearch { hosts => ["elasticsearch:9200"] } }
```

## 💡 Bonnes Pratiques

### Configuration Optimale
- ✅ **Persistance journald** : `/var/log/journal` sur partition dédiée
- ✅ **Rotation appropriée** : Balance rétention/espace disque
- ✅ **Monitoring espace** : Alertes à 80% usage partition
- ✅ **Centralisation** : Serveur logs dédié pour infrastructure
- ✅ **Séparation** : Logs applicatifs vs système

### Sécurité & Conformité
- ✅ **Intégrité** : Protection modification logs (immutable, WORM)
- ✅ **Chiffrement** : Transport logs distant (TLS)
- ✅ **Rétention** : Conformité réglementaire (RGPD, SOX)
- ✅ **Accès** : Droits lecture logs restreints
- ✅ **Sauvegarde** : Archivage logs critiques

### Troubleshooting Courant
```bash
# Journald ne démarre pas
systemctl status systemd-journald
journalctl -u systemd-journald

# Logs non persistants
ls -la /var/log/journal/
systemctl restart systemd-journald

# rsyslog issues
rsyslogd -N1                   # Test config
systemctl status rsyslog
journalctl -u rsyslog

# Espace disque saturé
journalctl --vacuum-size=1G
find /var/log -name "*.log" -size +100M
```

---
**💡 Memo** : `journalctl -f` pour suivi temps réel, `logger` pour tests, logrotate pour archivage !