# Cron

## Cron (crontab)
- `crontab -e` → éditer crontab user ; `crontab -l` → lister.  
- Format : 
```bash
* * * * * commande

- - - - -

| | | | |

| | | | +---- Jour de la semaine (0 - 7) (Dimanche=0 ou 7)

| | | +------ Mois (1 - 12)

| | +-------- Jour du mois (1 - 31)

| +---------- Heure (0 - 23)

+------------ Minute (0 - 59)
```
- Spéciaux : `*` (joker), `,` (liste), `-` (intervalle), `x/y` (step partant de x tous les y).  
- Exemple : `0 3 * * * /usr/bin/backup` → quotidien 03:00.

## Anacron

Dans /etc/crontab, vérification de la présence d'anacron dans le système, pour savoir qui gère les tâches (cron/anacron) 
```bash
`25 6 * * * root test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
```

- Il garde en mémoire la date de la dernière exécution dans des fichiers comme /var/spool/anacron/cron.daily
- Lancement via un timer systemd : `systemctl list-timers | grep anacron`
	- Lance anacron 5 minutes après le boot (OnBootSec=5min)
	- Puis relance-le toutes les 24h (OnUnitActiveSec=1d)
