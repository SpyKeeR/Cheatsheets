# ⏰ Cron & Planification — Aide-mémoire

## 🏗️ Architecture Planification Linux

### Services de Planification
| Service | Usage | Avantages | Limitations |
|---------|-------|-----------|-------------|
| **cron** | Tâches récurrentes | Simple, universel | Pas si machine éteinte |
| **anacron** | Tâches manquées | Rattrapage après arrêt | Précision jour seulement |
| **systemd timer** | Alternative moderne | Intégration systemd, logs | Syntaxe plus complexe |
| **at** | Tâches ponctuelles | Planification unique | Pas de récurrence |

### Hiérarchie Fichiers
```
/etc/crontab              # Crontab système (multi-utilisateurs)
/etc/cron.d/             # Crontabs système additionnels
/var/spool/cron/crontabs/ # Crontabs utilisateurs (crontab -e)
/etc/cron.{hourly,daily,weekly,monthly}/ # Scripts run-parts
/etc/anacrontab          # Configuration anacron
```

## ⚙️ Crontab Utilisateur

### Gestion Crontab
```bash
# Édition crontab personnelle
crontab -e                    # Éditer (utilise $EDITOR)
crontab -l                    # Lister tâches
crontab -r                    # Supprimer toutes tâches
crontab -u user -e            # Éditer pour autre utilisateur (root)

# Import/Export
crontab -l > backup.cron      # Sauvegarder
crontab backup.cron           # Restaurer
```

### Format Crontab
```bash
# Format: MIN HOUR DAY MONTH WEEKDAY COMMAND
* * * * * /path/to/command

# Champs détaillés:
┌───────────── minute (0-59)
│ ┌─────────── heure (0-23)
│ │ ┌───────── jour du mois (1-31)
│ │ │ ┌─────── mois (1-12)
│ │ │ │ ┌───── jour de la semaine (0-7, 0 et 7 = dimanche)
│ │ │ │ │
* * * * * commande_à_exécuter
```

### Syntaxe Spéciale Crontab
| Symbole | Signification | Exemple | Description |
|---------|---------------|---------|-------------|
| `*` | Toutes valeurs | `* * * * *` | Chaque minute |
| `,` | Liste valeurs | `1,15,30 * * * *` | Minutes 1, 15 et 30 |
| `-` | Plage valeurs | `9-17 * * * *` | De 9h à 17h |
| `/` | Intervalle | `*/5 * * * *` | Toutes les 5 minutes |
| `@` | Raccourcis | `@daily` | Une fois par jour |

### Raccourcis Temporels
```bash
@reboot        # Au démarrage système
@yearly        # 0 0 1 1 * (1er janvier minuit)
@annually      # Identique à @yearly
@monthly       # 0 0 1 * * (1er de chaque mois)
@weekly        # 0 0 * * 0 (dimanche minuit)
@daily         # 0 0 * * * (chaque jour minuit)
@midnight      # Identique à @daily
@hourly        # 0 * * * * (chaque heure)
```

## 📋 Exemples Pratiques Crontab

### Tâches Courantes
```bash
# Sauvegarde quotidienne à 2h30
30 2 * * * /scripts/backup.sh

# Nettoyage logs chaque dimanche à 1h
0 1 * * 0 /scripts/cleanup-logs.sh

# Rapport hebdomadaire vendredi 17h
0 17 * * 5 /scripts/weekly-report.sh

# Redémarrage serveur premier lundi du mois à 3h
0 3 1-7 * 1 /sbin/reboot

# Monitoring toutes les 5 minutes en journée
*/5 9-17 * * 1-5 /scripts/check-services.sh

# Synchronisation horaire quotidienne
0 4 * * * /usr/sbin/ntpdate pool.ntp.org

# Rotation logs personnalisés
0 0 * * * /usr/sbin/logrotate /etc/logrotate.conf
```

### Gestion Environnement
```bash
# Variables d'environnement dans crontab
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=admin@domain.com
HOME=/home/user

# Exécution avec environnement spécifique
0 2 * * * cd /app && /usr/bin/python3 script.py

# Source profile utilisateur
0 3 * * * /bin/bash -l -c 'cd /app && ./script.sh'

# Redirection sorties
0 1 * * * /scripts/backup.sh > /var/log/backup.log 2>&1
0 2 * * * /scripts/task.sh >/dev/null 2>&1  # Silence complet
```

## 🗓️ Crontab Système

### /etc/crontab (Multi-utilisateurs)
```bash
# Format avec champ utilisateur supplémentaire
# MIN HOUR DAY MONTH WEEKDAY USER COMMAND

# Exemples /etc/crontab
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Tâches système quotidiennes
17 * * * * root cd / && run-parts --report /etc/cron.hourly
25 6 * * * root test -x /usr/sbin/anacron || run-parts /etc/cron.daily
47 6 * * 7 root test -x /usr/sbin/anacron || run-parts /etc/cron.weekly
52 6 1 * * root test -x /usr/sbin/anacron || run-parts /etc/cron.monthly
```

### /etc/cron.d/ (Packages)
```bash
# Fichiers individuels pour applications
/etc/cron.d/logrotate        # Rotation logs
/etc/cron.d/sysstat          # Statistiques système
/etc/cron.d/popularity-contest # Ubuntu feedback

# Format identique à /etc/crontab (avec utilisateur)
# Exemple /etc/cron.d/backup
0 2 * * * backup /usr/local/bin/backup-script.sh
```

## ⚡ Anacron (Rattrapage Tâches)

### Configuration /etc/anacrontab
```bash
# Format: PERIOD DELAY JOB-IDENTIFIER COMMAND
# PERIOD: jours entre exécutions
# DELAY: minutes d'attente après démarrage
# JOB-IDENTIFIER: nom unique pour suivi

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# Tâches anacron standard
1  5    cron.daily       nice run-parts /etc/cron.daily
7  25   cron.weekly      nice run-parts /etc/cron.weekly
@monthly 45 cron.monthly nice run-parts /etc/cron.monthly
```

### Commandes Anacron
```bash
# Gestion anacron
anacron -T                  # Tester configuration
anacron -f                  # Forcer exécution toutes tâches
anacron -n                  # Exécuter maintenant (pas de délai)
anacron -d                  # Mode debug

# Vérifier état
cat /var/spool/anacron/cron.daily    # Date dernière exécution
ls -la /var/spool/anacron/           # Tous timestamps
```

### Intégration Cron-Anacron
```bash
# Test présence anacron dans /etc/crontab
25 6 * * * root test -x /usr/sbin/anacron || run-parts /etc/cron.daily

# Si anacron présent: anacron gère les tâches quotidiennes
# Si anacron absent: cron exécute directement run-parts
```

## 🔧 Systemd Timers (Alternative Moderne)

### Concepts Timers
```bash
# Lister timers actifs
systemctl list-timers
systemctl list-timers --all        # Inclure inactifs

# Timer + Service associé
backup.timer  → déclenche → backup.service
```

### Exemple Timer Systemd
```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily backup
Requires=backup.service

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=30min

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Backup script
Wants=backup.timer

[Service]
Type=oneshot
ExecStart=/scripts/backup.sh
User=backup
```

### Gestion Timers
```bash
# Activation
systemctl enable backup.timer
systemctl start backup.timer

# Monitoring
systemctl status backup.timer
journalctl -u backup.service       # Logs exécution
```

## 📧 At (Tâches Ponctuelles)

### Utilisation At
```bash
# Planifier tâche unique
at 14:30                          # Aujourd'hui 14h30
at 2pm tomorrow                   # Demain 14h
at now + 2 hours                  # Dans 2 heures
at 09:00 2024-12-25              # Date spécifique

# Exemples interactifs
echo "/scripts/maintenance.sh" | at 02:00
at now + 10 minutes <<EOF
/usr/bin/wall "Maintenance terminée"
EOF

# Gestion queue
atq                              # Lister tâches en attente
atrm 2                           # Supprimer tâche numéro 2
```

## 📊 Monitoring & Logs

### Journaux Cron
```bash
# Logs système
journalctl -u cron               # Systemd (Debian 10+)
tail -f /var/log/cron           # Fichier direct (Red Hat)
tail -f /var/log/syslog | grep CRON  # Debian/Ubuntu

# Filtrage logs
journalctl -u cron --since "1 hour ago"
grep "CRON" /var/log/syslog | tail -20
```

### Variables Environnement Utiles
```bash
# Dans crontab pour debugging
MAILTO=admin@domain.com          # Envoi mails erreurs
SHELL=/bin/bash                  # Shell explicite
PATH=/usr/local/bin:/usr/bin:/bin  # PATH complet

# Debug exécution
0 * * * * /bin/date >> /tmp/cron-debug.log 2>&1
```

## 🛡️ Sécurité & Bonnes Pratiques

### Contrôle Accès
```bash
# Fichiers de contrôle
/etc/cron.allow                 # Utilisateurs autorisés (prioritaire)
/etc/cron.deny                  # Utilisateurs interdits

# Si cron.allow existe: seuls ces utilisateurs
# Si seul cron.deny existe: tous sauf ces utilisateurs
# Si aucun: seul root autorisé
```

### Sécurisation Tâches
```bash
# Chemins absolus obligatoires
0 2 * * * /usr/bin/rsync /data/ /backup/

# Vérification prérequis
0 3 * * * [ -f /data/ready ] && /scripts/process.sh

# Limitation ressources
0 1 * * * nice -n 19 ionice -c 3 /scripts/heavy-task.sh

# Gestion erreurs
0 2 * * * /scripts/backup.sh || echo "Backup failed" | mail -s "Alert" admin@domain.com
```

### Variables Environnement Sécurisées
```bash
# Éviter mots de passe en dur
0 2 * * * /scripts/backup.sh 2>&1 | logger -t backup

# Utiliser fichiers configuration
0 3 * * * cd /app && /usr/bin/python3 -c "import config; run_task()"
```

## 🔍 Diagnostic & Troubleshooting

### Vérifications Courantes
```bash
# Service cron actif ?
systemctl status cron           # Systemd
service cron status             # SysV

# Syntaxe crontab
crontab -l | head -5            # Vérifier format

# Test manuel commande
sudo -u user /bin/bash -c "cd /home/user && /scripts/task.sh"

# Permissions fichiers
ls -la /scripts/task.sh         # Exécutable ?
ls -la /var/spool/cron/crontabs/user  # Crontab lisible ?
```

### Problèmes Fréquents
| Problème | Cause Probable | Solution |
|----------|----------------|----------|
| **Tâche ne s'exécute pas** | PATH incomplet | Chemins absolus dans crontab |
| **"Command not found"** | Environnement différent | Source profile ou export PATH |
| **Script fonctionne manuellement** | Variables manquantes | Définir variables dans crontab |
| **Pas de mail d'erreur** | MAILTO non défini | Ajouter MAILTO dans crontab |
| **Exécution partielle** | Erreur silencieuse | Rediriger 2>&1 vers fichier log |

### Debug Avancé
```bash
# Trace complète
0 * * * * /bin/bash -x /scripts/task.sh > /tmp/cron-trace.log 2>&1

# Environnement cron vs shell
0 * * * * env > /tmp/cron-env.log
# Comparer avec: env > /tmp/shell-env.log

# Test timing
0 * * * * echo "$(date): Cron executed" >> /tmp/cron-timing.log
```

## ⚙️ Migration & Alternatives

### Cron vers Systemd Timer
```bash
# Cron: 0 2 * * * /scripts/backup.sh
# Équivalent systemd:

# backup.timer
[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

# backup.service  
[Service]
Type=oneshot
ExecStart=/scripts/backup.sh
```

### Outils Modernes
- **fcron** : Cron avancé avec rattrapage
- **cronie** : Cron moderne (Red Hat default)
- **systemd-cron** : Émulation cron via systemd
- **jobber** : Alternative moderne multi-plateforme

---
**💡 Memo** : Chemins absolus obligatoires, `2>&1` pour debug, anacron pour rattrapage, systemd timers pour le moderne !
