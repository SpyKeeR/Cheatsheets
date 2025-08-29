# GLPI — Cheatsheet condensé 🎯

## Concepts & acronyms
- ITSM = IT Service Management
- ITIL = Information Technology Infrastructure Library
- GLPI = Gestion Libre de Parc Informatique.
- ITAM = IT Assets Management
- CMDB = Configuration Management DataBase
- KEDB = Known Error Database 
- SKMS = Service Knowledge Management System
- KB = Knowledge Base
- KPI = Key Performance Indicator
- SLA = Services level Agreement - Accords de niveaux de service
- GTI = Garantie de Temps d’Intervention
- GTR = Garantie de Temps de Rétablissement
- OLA = Operational Level Agreements - accords internes
- TTO = Time To Own : Délai pour la prise en charge
- TTR = Time To Resolve : Délai pour résoudre le ticket.
- PGI/ERP = Progiciel de Gestion Intégré / ERP Enterprise Ressource Planing 
- CI = Configuration Item - PC, imprimante, service...

## Stack requise (LAMP)
- Apache + MariaDB/MySQL + PHP (+ modules usuels : php-mysql, php-ldap, php-imap, php-xmlrpc, php-intl, php-apcu, php-bz2, php-cas.)

## Installation web (rapide)
- Installer MariaDB: `apt install mariadb-server mariadb-client`.  
- Sécuriser DB: `mysql_secure_installation`.  
- Créer DB GLPI: dans mysql → `CREATE DATABASE glpidata;` et `GRANT ALL PRIVILEGES ON glpidata.* TO 'glpiuser'@'localhost' IDENTIFIED BY 'MotDePasseFort';`.  
- Télécharger GLPI: `wget https://github.com/glpi-project/glpi/releases/download/X.Y.Z/glpi-X.Y.Z.tgz`.  
- Déployer: `tar -xvzf ... -C /var/www/html` ; `chown -R www-data:www-data /var/www/html/glpi` ; `chmod -R 755 /var/www/html/glpi`.  
- Accéder via browser: `http://<IP>/glpi` → suivre install web.  
- Supprimer installer: `rm /var/www/html/glpi/install/install.php`.

## Comptes par défaut
- Admin: `glpi/glpi` ; Tech: `tech/tech` ; User: `normal/normal` ; Post-only: `post-only/post-only`.

## Authentification
- Sources possibles : LDAP/AD, IMAP, X.509(Certificats clients), base interne.  
- LDAP/AD : port 389 (LDAP) / 636 (LDAPS); 
	- Bind DN = compte dédié (lecture seule).
	- Filtre recommandation pour exclure comptes désactivés > `(&(objectClass=user)(objectCategory=person)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))`
	- Toujours `Tester la connexion`.

## Profils & entités
- Profils (Template de permissions) :
| Profil       | Droits Parc       | Droits Tickets         | Interface  | Usage typique                          |
|--------------|-------------------|------------------------|------------|----------------------------------------|
| Super-Admin  | ✅ Tout            | ✅ Tout                 | Standard   | Dieu dans GLPI 🤣                       |
| Admin        | ✅ Large           | ✅ Large                | Standard   | Gestion avancée (sauf config critique) |
| Supervisor   | ✅ Comme Tech      | ✅ + gestion équipe     | Standard   | Chef d’équipe IT                       |
| Technician   | ✅ Créa/Modif/Supp | ✅ Cycle de vie complet | Standard   | Technicien support                     |
| Hotliner     | ❌                 | ✅ Cycle de vie complet | Standard   | Hotline / Helpdesk                     |
| Observer     | ✅ Lecture seule   | 🔶 Participe (limité)   | Standard   | Suivi / reporting                      |
| Self-Service | ❌                 | ✅ Ses propres tickets  | Simplifiée | Utilisateurs finaux                    |
| Read-only    | ✅ Lecture seule   | ❌                      | Standard   | Consultation pure                      | 
- Habilitation = (profil fonctionnel + entité + récursivité).  

## Gabarits & automatisation
- Créer catégories (max 2 niveaux recommandés).  
- Gabarits nommés clairement (ex: Incident_Logiciel).  
- Balises automatiques : `#` incrément, `\Y` année, `\y` année(2), `\m` mois, `\d` jour. Exemple inventaire : `LAPTOP-###`.

## Cron GLPI (essentiel)
- Éditer crontab du www-data: `crontab -u www-data -e`.  
- Ligne importante (toutes les minutes) : `* * * * * /usr/bin/php8.8 /var/www/glpi/front/cron.php >/dev/null 2>&1` (ajuster chemin PHP).
- Autocron par défaut trop aléatoire pour la production

## Plugins & Marketplace
- Marketplace requiert clé API (Configuration > Général > Clé d’authentification Marketplace).  
- Installation manuelle si absent : décompresser → `/var/www/glpi/plugins` → droits www-data → activer via UI.  
- Si désinstallation : supprimer dossier `/var/www/glpi/plugins` résiduel.

## Data Injection (plugin)
- Menu : Accueil > Outils > Data Injection → importer CSV pour gros imports (matériel, utilisateurs, logiciels).

## Sécurité & bonnes pratiques
- Utiliser HTTPS, restreindre accès Marketplace si nécessaire, nettoyer `/var/www/glpi/marketplace` si besoin.  
- Faire sauvegarde DB + dossier `/var/www/glpi` avant mises à jour.


# GLPI Agent — install & config rapide 🤖

## Agent overview
- Agent pour inventaire centralisé + exécution tâches (port par défaut agent HTTP mini-server 62354) - [Documentation officielle](https://glpi-agent.readthedocs.io/en/latest/)

## Windows (installer)
- Télécharger installateur → lancer wizard → Typical.  
- Target URL = `http(s)://<serveur>/front/inventory.php`.  
- Options utiles : SSL, proxy, TAG (classer entité), Debug (logs), Start inventory after install, install as service.  
- Vérifier mini-server web client : `http://localhost:62354`.

## Linux (installer)
- Récupérer le script perl d'installation sur le [GitHub](https://github.com/glpi-project/glpi-agent/releases)
- Installer paquet, éditer `/etc/glpi-agent/agent.cfg` (ou chemin projet): `server=`, `tasks=INVENTORY`, `tag=`.  
- Fichiers volatile: `00-install.cfg` (modifié au update).  
- Permissions: droits exécution, dépendances (Perl) selon modules.

## Tips
- Agent Windows → firewall rules à prévoir.  
- Tags → trient automatiquement les machines dans entités.  
- Logs agent pour debug; vérifier connectivity et certificats SSL si LDAPS.
