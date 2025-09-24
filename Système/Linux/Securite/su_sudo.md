# 🔐 su & sudo — Aide-mémoire

## 🏗️ Concepts Fondamentaux

### Différences su vs sudo
| Aspect | su | sudo |
|--------|----|----- |
| **Authentification** | Mot de passe utilisateur cible | Mot de passe utilisateur courant |
| **Session** | Nouvelle session utilisateur | Exécution ponctuelle |
| **Configuration** | Accès via mot de passe | Règles dans /etc/sudoers |
| **Audit** | Limité | Journalisation détaillée |
| **Granularité** | Tout ou rien | Contrôle fin par commande |

### Modèles de Sécurité
- **su traditionnel** : Connaissance mot de passe root = accès total
- **sudo moderne** : Délégation contrôlée + traçabilité
- **Principe moindre privilège** : Accès minimal nécessaire

## 🔄 Commande su (Switch User)

### Syntaxe & Options
```bash
# Basique
su                              # Devenir root (garde environnement)
su -                            # Shell de login root complet
su - username                   # Devenir autre utilisateur

# Options principales
su -l username                  # Login shell (identique à -)
su -s /bin/bash username        # Shell spécifique
su -c "commande" username       # Exécuter commande puis revenir
su -m username                  # Préserver environnement courant
su -p username                  # Identique à -m (preserve)

# Exemples pratiques
su - postgres -c "psql -l"      # Commande en tant que postgres
su -s /bin/sh -c "ls /root"     # Shell sh pour commande
```

### Variables Environnement
```bash
# Différences environnement
su root                         # Garde $HOME, $USER courant
su - root                       # Charge environnement root complet

# Vérification
su -c 'echo $HOME $USER $PATH'  # Environnement mixte
su - -c 'echo $HOME $USER $PATH' # Environnement root pur
```

## ⚙️ Configuration sudo

### Fichier /etc/sudoers
```bash
# Édition sécurisée (OBLIGATOIRE)
visudo                          # Éditeur avec vérification syntaxe
visudo -f /etc/sudoers.d/custom # Fichier supplémentaire

# Structure ligne sudoers
# USER HOST=(RUNAS) COMMANDS
# ou
# %GROUP HOST=(RUNAS) COMMANDS
```

### Syntaxe Règles Basiques
```bash
# Accès complet
root ALL=(ALL:ALL) ALL
%sudo ALL=(ALL:ALL) ALL         # Groupe sudo

# Utilisateur spécifique
alice ALL=(ALL) ALL             # Alice peut tout faire
bob localhost=(root) /bin/ls    # Bob peut ls en root sur localhost

# Sans mot de passe
alice ALL=(ALL) NOPASSWD: ALL   # Dangereux, éviter
bob ALL=(root) NOPASSWD: /usr/bin/systemctl restart apache2
```

### Règles Avancées
```bash
# Groupes de commandes (Alias)
Cmnd_Alias NETWORKING = /sbin/route, /sbin/ifconfig, /bin/ping
Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
Cmnd_Alias SERVICES = /sbin/service, /usr/bin/systemctl

# Utilisation alias
%netadmin ALL = NETWORKING
%developers ALL = SOFTWARE, SERVICES

# Restrictions par hôte
alice server1=(root) /usr/bin/systemctl
bob ALL=(www-data) /var/www/scripts/*

# Exclusions (interdire commandes)
alice ALL=(ALL) ALL, !/bin/su, !/usr/bin/passwd root
```

### Gestion Groupes & Utilisateurs
```bash
# Ajouter utilisateur au groupe sudo
usermod -aG sudo username       # Debian/Ubuntu
usermod -aG wheel username      # RHEL/CentOS

# Vérifier appartenance groupe
groups username                 # Groupes utilisateur
id username                     # Info complète (uid, gid, groupes)

# Groupe sudo vs wheel
# sudo: Debian/Ubuntu (groupe par défaut)  
# wheel: RHEL/CentOS/traditionnel Unix
```

## 🔧 Utilisation sudo

### Commandes de Base
```bash
# Exécution simple
sudo command                    # Avec privilèges root
sudo -u username command        # En tant qu'autre utilisateur  
sudo -g group command           # Avec autre groupe principal

# Sessions interactives
sudo -i                         # Shell root interactif (login)
sudo -s                         # Shell root (environnement courant)
sudo -u postgres -i             # Shell postgres interactif

# Validation & timing
sudo -v                         # Renouveler ticket (pas de commande)
sudo -k                         # Invalider ticket immédiatement
sudo -K                         # Invalider tous tickets utilisateur
```

### Options Utiles
```bash
# Environnement
sudo -E command                 # Préserver variables environnement
sudo VAR=value command          # Passer variable spécifique
sudo -H command                 # Définir $HOME utilisateur cible

# Debug & information
sudo -l                         # Lister permissions utilisateur
sudo -l -U username             # Permissions utilisateur spécifique (root only)
sudo -ll                        # Format long (plus détails)

# Exemples pratiques
sudo -u www-data -g www-data ls /var/www/  # User + group spécifiques
sudo -E -u deploy env           # Voir variables héritées
```

## 📊 Journalisation & Audit

### Logs sudo
```bash
# Emplacements logs
/var/log/auth.log               # Debian/Ubuntu
/var/log/secure                 # RHEL/CentOS  
journalctl -f SYSLOG_IDENTIFIER=sudo  # systemd

# Recherche dans logs
grep sudo /var/log/auth.log
tail -f /var/log/auth.log | grep sudo

# Informations enregistrées
# - Utilisateur ayant utilisé sudo
# - Commande exécutée
# - Répertoire de travail  
# - Date/heure
# - Succès/échec
```

### Configuration Logging Avancée
```bash
# Dans /etc/sudoers (via visudo)
Defaults log_host, log_year, logfile="/var/log/sudo.log"
Defaults loglinelen=0           # Pas de troncature lignes
Defaults timestamp_timeout=0    # Demander mot de passe à chaque fois
```

## 🛡️ Sécurité & Bonnes Pratiques

### Règles Sécurisées
```bash
# Éviter les dangers
# DANGEREUX:
alice ALL=(ALL) NOPASSWD: ALL   # Accès root sans mot de passe
bob ALL=(root) /bin/bash        # Shell root = accès total
charlie ALL=(ALL) /bin/su       # Contournement sudo

# SÉCURISÉ:
alice ALL=(root) /usr/bin/systemctl restart nginx
bob ALL=(www-data) /var/www/deployment/deploy.sh
charlie ALL=(backup) /usr/local/bin/backup-script
```

### Variables Environnement Sécurisées
```bash
# Dans /etc/sudoers
Defaults env_reset              # Reset env par défaut (sécurisé)
Defaults env_keep="LANG LC_* HOME"  # Garder variables spécifiques
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Interdire variables dangereuses
Defaults env_delete="LD_LIBRARY_PATH LD_PRELOAD"
```

### Timeout & Session
```bash
# Configuration temporisation
Defaults timestamp_timeout=5    # 5 minutes (défaut: 15)
Defaults timestamp_timeout=0    # Toujours demander mot de passe
Defaults passwd_timeout=1       # 1 minute pour saisir mot de passe

# Tentatives
Defaults passwd_tries=3         # 3 essais mot de passe max
```

## 🔍 Diagnostic & Troubleshooting

### Tests & Validation
```bash
# Tester configuration sudoers
sudo -l                         # Voir ses propres permissions
sudo -l -U username             # Permissions autre utilisateur (root)
visudo -c                       # Vérifier syntaxe sudoers

# Debug règles
sudo -v                         # Tester authentification
sudo whoami                     # Vérifier identité effective
sudo -u nobody whoami           # Test utilisateur spécifique
```

### Problèmes Courants
| Problème | Cause | Solution |
|----------|-------|----------|
| `user is not in sudoers` | Pas dans groupe/fichier | `usermod -aG sudo user` |
| `sudo: command not found` | PATH incorrect | Vérifier secure_path |
| `Authentication failure` | Mot de passe incorrect | Vérifier compte actif |
| `Sorry, try again` | Trop de tentatives | Attendre ou `sudo -k` |

### Récupération Accès Root
```bash
# Si lockout complet
# Boot en single-user mode ou rescue
# Monter / en écriture: mount -o remount,rw /
# Modifier /etc/sudoers ou réinitialiser mot de passe
```

## 📋 Cas d'Usage Courants

### Administrateur Système
```bash
# /etc/sudoers.d/sysadmin
%sysadmin ALL=(ALL) NOPASSWD: /usr/bin/systemctl, /usr/sbin/service
%sysadmin ALL=(ALL) /usr/bin/apt, /usr/bin/yum
%sysadmin ALL=(root) /bin/mount, /bin/umount
```

### Développeur Web  
```bash
# /etc/sudoers.d/webdev
%webdev ALL=(www-data) NOPASSWD: /var/www/scripts/deploy.sh
%webdev ALL=(www-data) /usr/bin/composer
%webdev ALL=(root) /usr/bin/systemctl restart nginx, /usr/bin/systemctl reload nginx
```

### Utilisateur Service
```bash
# /etc/sudoers.d/monitoring
nagios ALL=(root) NOPASSWD: /usr/lib/nagios/plugins/*
zabbix ALL=(root) NOPASSWD: /bin/netstat, /bin/ss, /usr/bin/lsof
```

## 💡 Conseils Pratiques

### Gestion Quotidienne
- ✅ **Utiliser sudo** plutôt que su pour traçabilité
- ✅ **Groupes dédiés** par fonction (webdev, sysadmin, backup)
- ✅ **Règles granulaires** : éviter ALL=(ALL) ALL
- ✅ **NOPASSWD avec parcimonie** : seulement scripts automatisés
- ✅ **Fichiers séparés** dans /etc/sudoers.d/ par service

### Audit & Monitoring
- ✅ **Surveiller logs** sudo régulièrement  
- ✅ **Alertes** sur commandes sensibles (/bin/su, passwd root)
- ✅ **Révision périodique** des règles sudoers
- ✅ **Formation utilisateurs** sur bonnes pratiques

### Migration su → sudo
1. **Audit** : Identifier qui utilise su
2. **Règles** : Créer sudoers équivalents
3. **Formation** : Expliquer transition aux utilisateurs
4. **Désactivation** : passwd -l root (optionnel)
5. **Monitoring** : Vérifier adoption sudo

---
**💡 Memo** : `visudo` pour éditer, `sudo -l` pour voir permissions, logs dans auth.log, éviter NOPASSWD sauf nécessité !