# Centreon — super-condensé

### Modes de supervision
- Actif local : superviseur lance check local (ex : check_tcp) — pas d’agent.  
- Actif distant : superviseur déclenche commande distante via NRPE/SNMP/WMI/SSH.  
- Passif : hôte pousse les événements (SNMP Trap, NSCA, etc.) → utile pour alertes temps réel.

### Déploiement & installation (notes courtes)
- Options : ISO Centreon (rapide), paquets RPM (RHEL-based), VM préconfigurée, sources (avancé).  
- OS recommandé : RHEL-family (Alma/Rocky) ou images officielles Centreon.

### Cycle config Centreon
- Édition via Web → sauvegarde en BDD.  
- Génération fichiers `.cfg` → stockage temporaire `/usr/share/centreon/filesGeneration/`.  
- Move → `/etc/centreon/*.cfg` puis reload moteur.  
- Génération via Configuration → Pollers → Export Configuration 
	- Test(debug UI) → pour simuler la génération (mode debug)
	- Génération → pour vérifier les fichiers
	- Déplace+Reload → pour appliquer en production

### Fichiers & composants clés
- Centreon Web (PHP) — UI  
- Centreon Engine — moteur (fork Nagios)  
- Centreon Broker — collecte/export  
- Base → MySQL/MariaDB (historique, configs)  
- Agent NRPE conf : /etc/nagios/nrpe.cfg  
- NSClient++ Windows : C:\Program Files\NSClient++\nsclient.ini 
- SNMPd Linux : /etc/snmp/snmpd.conf  
- SNMP queries → UDP 161 | SNMP traps → UDP 162 | NRPE → TCP 5666 | NSClient++ web → TCP 8443

## SNMP (v1/v2c/v3)
- Démon : snmpd
- Paquets : snmp, snmpd ; conf → /etc/snmp/snmpd.conf  
- Principales directives :
	- Listening : `agentAddress udp:161` (ou restreint `udp:127.0.0.1:161`)
	- Communauté RO/RW : `r{o,w}community commu default|IP -V VueSysteme`
	- Vue : `view SystemeOnly included .1.3.6.1.2.1.1`
	- Old config :
		- `com2sec` → `com2sec notSecure 127.0.0.1 public` → notSecure : Nom d'entité, 127.0.0.1 : IP/Réseau, public : Nom de communauté v2c
		- `group` → `group readonlyGroup v2c notSecure` → readonlyGroup : Nom de groupe, v2c : Version SNMP, notSecure : Nom d'entité com2sec
		- `access` → `access readonlyGroup "" any noauth exact complete none none`
			- "readonlyGroup" ➤ groupe défini
			- "" ➤ nom d’utilisateur (vide ici car v1/v2c)
			- any ➤ n’importe quel contexte SNMP
			- noauth ➤ pas d’authentification (cas de v1/v2c)
			- exact ➤ correspondance exacte entre utilisateur/contexte
			- SystemeOnly ➤ view autorisée en lecture
			- none ➤ pas de view en écriture
			- none ➤ pas de view pour les traps

### Installation SNMP sous Windows
- Installation  ➤ `Install-WindowsFeature -Name SNMP -IncludeManagementTools` (Avant Win 10 20H2/11)
- Autre méthode ➤ `Add-WindowsCapability -Online -Name "SNMP.Client~~~~0.0.1.0"`
- Accès à la config ➤ services.msc, clic droit sur "Service SNMP" → onglet Sécurité

### Installation SNMP sous Debian
- Installation ➤ `apt install snmpd`
- Fichier de configuration ➤ /etc/snmp/snmpd.conf
- Redémarrage du service requis ➤ `systemctl restart snmpd`

### SNMP Trap & passive
- Activer traps sur agent/équipement ; superviseur doit écouter/recevoir.  
- Format : trap envoyé → mapping vers service/alerte Centreon via broker/handler.  
- Outils passifs : SNMP Trap Daemon, NSCA pour push d'events.

### SNMP — vues/MIBs rapides
- View = filtrer OIDs exposées (ex: .1.3.6.1.2.1.1 pour info système)  
- MIB/OID → utiliser `snmpwalk`/`snmpget` pour debug  
- Exemples utiles : sysDescr `.1.3.6.1.2.1.1.1.0`, uptime `.1.3.6.1.2.1.1.3.0`

## NRPE
- Archi : Engine → check_nrpe → NRPE demon → plugin local → retour.  
- Installer : `nrpe + nagios-plugins`; conf `allowed_hosts` → IP superviseur.  
- Test : `check_nrpe -H 192.168.1.50 -c check_load`  
- Sécurité : restreindre allowed_hosts, utiliser TLS si dispo, limiter commandes.

### NSClient++ (Windows)
- Installer / activer modules (NRPE, Checkers).  
- UI locale : https://localhost:8443 (auth). 
- Modules : activer les plugins nécessaires (ex. CPU, disque, RAM)
- Queries : tester les plugins avec des requêtes manuelles
- Paramètre important : Allow arguments = true ; Allow nasty characters = true (si paramètres).  
- Tester via Centreon plugin ou interface.

### CMA / OTLP
- CMA : Centreon Monitoring Agent (NEW Agent)
- OTLP : Open Telemetry L Protocol (NEW NRPE)
- **A compléter après un lab** ⚠

## Sondes

### Workflow sonde 

- Lecture config et récupère la commande associée au service/hôte.
- Execution commande (local/distance).
- Interrogation du host, récupère une valeur (charge, disponibilité, etc.).
- Retour code de statut : 0 = OK, 1 = WARNING, 2 = CRITICAL, 3 = UNKNOWN, 4 = PENDING
- Centreon-engine lit le code et met à jour l’état du service/hôte.

### Sondes & chemins
- Nagios plugins : `/usr/lib/nagios/plugins`, `/usr/lib64/nagios/plugins`
- Centreon plugins : `/usr/lib/centreon/plugins`  
- Exemples : `check_ping`, `check_http`, `check_disk`, `check_load`

### Centreon plugins — usage rapide
- Lister modes/help : `--help`, `--list-plugin`, `--list-mode`  
- Exemple SNMP CPU :  
```bash
centreon_linux_snmp.pl --plugin=os::linux::snmp::plugin --mode=cpu --hostname=172.16.1.3 --snmp-community=public --snmp-version=2 --warning-average=80 --critical-average=90
```

### Plugin execution & sortie (format attendu)
- Sortie = `code retour | texte lisible | perfdata`  
  - Ex : `PING OK - 0% loss, rta=0.344ms | rta=0.344ms;1;2;0 pl=0%;50;80;0`    
- Perfdata → utilisé par graphing (RRD/metrics)

### Macros & variables
- Standards (Nagios) : `$HOSTADDRESS$`, `$SERVICESTATE$`, etc. [Issues de Nagios](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/macrolist.html), intégrées nativement (non modifiables).
- Ressources (globales) : `$USER1$`, `$CENTREONPLUGINS$` (définies via Configuration → Pollers → Resources).  
- Arguments : `$ARG1$`…`$ARG32$` (remplacé souvent par macros personnalisées).  
- Macros personnalisées : `$_HOSTnom$`, `$_SERVICEnom$` → lisibles, réutilisables (`$_HOSTmacro$` → assignée à un hôte | `$_SERVICEmacro$` → assignée à un service)
- Macros à la demande : `$HOSTSTATE:nomhôte$` (Calcul valeur temps réel, à éviter).

### Etats & notifications
- SOFT = essais de confirmation (pas de notif). HARD = état confirmé → notif possible.  
- Workflow notifications : 
	- Statut HARD
	- Notifications activées hôte service ET templates
	- Pas de Downtime actif
	- Période de notification OK
	- Type d'état a notif (critical/warning)
	- Ciblage et config contact : 
		- Activation Notification
		- Période de notification OK
		- Notif autorisée pour le type de statut
	- Execution de la commande de notif

### Actions immédiates
- Immediate Check → replanifie un test au prochain cycle de supervision
- Forced reschedule → exécute un test immédiatement
- Mass actions (UI)  

### ACL & catégories (accès)
- 3 blocs : 
	- ACL Resources (objets) → Définissent ce qu’on protège
	- ACL Menus (UI) → Définissent ce qu’on affiche dans l’UI
	- ACL Actions (opérations) → Définissent ce que l’utilisateur peut faire
- Regroupés en ACL Group → associé à users/groups.  
- Categories ≠ groupes : Catégories servent de filtre visibilité + héritage modèles.

### Pollers (collecteurs)
- Rôle : Engine + Broker Module (exécutent checks localement, remontent résultats).  
- Communication Central ↔ Poller via SSH (ou ZMQ) → échange de clés RSA/config YAML obligatoire.  
- Ajout Poller : Configuration → Pollers → Add > copier YAML Gorgone (Central) → coller sur Poller → crée `/etc/centreon-gorgone/config.d/40-gorgoned.yaml`.  
- Sur Poller : `systemctl restart gorgoned` ; `systemctl enable gorgoned`.  
- En cas d’échec : restart `centengine`, `centreon`, `gorgoned` ; réexporter config ; dernier recours reboot.

### Raccourcis utiles
- `snmpwalk -v2c -c <comm> <host> <OID>` → parcourir OID  
- `snmpget -v2c -c <comm> <host> <OID>` → valeur précise  
- `check_nrpe -H <host> -c <cmd>` → tester NRPE  
- `su - centreon-engine` → tester comme moteur

### Logs & debug
- Centreon Engine logs → `journalctl -u centreon-engine|centreon-broker` OU /var/log/centreon-engine
- Logs moteur/poller → `journalctl -u centreon-engine|gorgoned`
- NRPE debug via `nrpe -c ...` OU `systemctl status`
- bind/trap/snmp → `journalctl` OU /var/log/syslog

### Bonnes pratiques rapides 
- Séparer roles (DB, Engine, Broker) pour scalabilité.  
- Tests : simuler comme engine `su - centreon-engine` puis exécuter commandes/plugins.  
- Monitoring redondant : config High-Availability / sondes multiples si critique.
- Tester NRPE/SNMP/Plugins localement depuis poller/central pour isoler. 
- Utiliser macros personnalisées pour lisibilité, généralisation de sondes. 
- Valider en Test avant Déplacer+Reload.  
- Limiter macros en temps réel (ressources). 
- Template attention : une option désactivée dans un template se propage.
- Sauvegarder configs / versionner exports.  

### Notes rapides sécurité
- Restreindre SNMP (v2c communities modifiées ou v3).
- NSClient++ : activer TLS/HTTPS si possible, limiter IP, utiliser auth locale.  
- NRPE : allowed_hosts strict. 
- Chiffrement/ZMQ pour communications Central↔Poller.
- Restreindre accès pollers/agents (IP, clés RSA).  
- Restreindre commandes autorisées sur agents.  
- Sécuriser accès UI (roles/ACL).  