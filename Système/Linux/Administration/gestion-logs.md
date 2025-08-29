# Logs, journalctl & rsyslog — condensé 🗃️

## journald / journalctl
- `journalctl` → lire logs systemd (volatile si pas persistant).  
- En direct : `journalctl -f` (follow).  
- Par service : `journalctl -u sshd`.  
- Par PID : `journalctl _PID=1234`.  
- Par priorité : `journalctl -p err` (levels: emerg 0, alert 1, crit 2, err 3, warning 4, notice 5, info 6, debug 7).  
- Rendre journald persistant : `mkdir -p /var/log/journal` → journald écrit durablement.  
- Limites & config : `/etc/systemd/journald.conf` → `SystemMaxUse=`, `SystemMaxFileSize=`.

## Logs classiques & rotation
- Fichiers logs : `/var/log/*.log` ; rotation gérée par `logrotate` (`/etc/logrotate.conf`, `/etc/logrotate.d/`).  
- Exemple `logrotate` snippet : `daily`, `rotate 366`, `create 640`, `compress`.

## rsyslog basics
- Facilities (source)
	- `auth` → Authentification
	- `authpriv` → Infos de connexion privées
	- `cron` → Tâches planifiées
	- `daemon` → Services de fond
	- `kern` → Kernel
	- `mail` → Service mail
	- `user` → Processus utilisateurs
	- `local0..local7` → Utilisation perso.
- Syntaxe config : `facility.priority /var/log/target.log` (ex: `auth,authpriv.* /var/log/auth.log`).  
	- `*` = toutes les priorités
	- `;` = exclusion de la cible avant le ;, suivi des facilities et priorities exclues
	- `*.=warn` : spécifie une seule priority, si pas de = cible toutes les priorities supérieures
	- `none` = exclusion totale de cette facility
- Exemple pratique : `auth,authpriv.* /var/log/auth.log` ; `*.=warn;auth,authpriv.none /var/log/messages` (exclure auth).

## Logger (tester)
- `logger -p cron.info "message de test"` → injecte message syslog.

## Bonnes pratiques
- Mettre `/var/log/journal` sur partition dédiée si possible (éviter saturation).  
- Surveiller taille journald (`SystemMaxUse`) et logrotate retention.  
- Centraliser logs (rsyslog remote, syslog-ng, ELK/EFK) pour infra multi-serveurs.