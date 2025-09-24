# 🎯 GLPI & GLPI Agent — Aide-mémoire

## 🏗️ Architecture & Concepts GLPI

### Concepts ITSM/ITIL Essentiels
| Acronyme | Signification | Usage GLPI |
|----------|---------------|------------|
| **ITSM** | IT Service Management | Cadre général de gestion IT |
| **ITIL** | IT Infrastructure Library | Méthodologie bonnes pratiques |
| **CMDB** | Configuration Management DB | Base des éléments configurés |
| **KEDB** | Known Error Database | Base erreurs connues |
| **SLA** | Service Level Agreement | Accords niveaux service |
| **OLA** | Operational Level Agreement | Accords internes équipes |
| **CI** | Configuration Item | Élément géré (PC, service, etc.) |

### Métriques Temps (KPI)
```
TTO (Time To Own) → Délai prise en charge ticket
TTR (Time To Resolve) → Délai résolution complète
GTI (Garantie Temps d'Intervention) → Engagement intervention
GTR (Garantie Temps de Rétablissement) → Engagement résolution
```

### Stack Technique
```
GLPI = LAMP Stack:
├── Apache/Nginx (serveur web)
├── MySQL/MariaDB (base données)
├── PHP 8.0+ (application)
└── Extensions PHP requises:
    ├── php-mysql, php-ldap, php-imap
    ├── php-xmlrpc, php-intl, php-apcu
    └── php-bz2, php-cas, php-gd
```

## 📦 Installation & Configuration

### Installation Base de Données
```bash
# Installation MariaDB
apt update && apt install mariadb-server mariadb-client
systemctl enable mariadb

# Sécurisation
mysql_secure_installation

# Création base GLPI
mysql -u root -p
CREATE DATABASE glpidb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON glpidb.* TO 'glpiuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Déploiement GLPI Web
```bash
# Téléchargement dernière version
cd /tmp
wget https://github.com/glpi-project/glpi/releases/download/10.0.10/glpi-10.0.10.tgz
tar -xzf glpi-10.0.10.tgz

# Installation
sudo mv glpi /var/www/html/
sudo chown -R www-data:www-data /var/www/html/glpi
sudo chmod -R 755 /var/www/html/glpi

# Permissions sécurisées
sudo chmod 750 /var/www/html/glpi/config
sudo chmod 750 /var/www/html/glpi/files
```

### Configuration Apache
```apache
# /etc/apache2/sites-available/glpi.conf
<VirtualHost *:80>
    ServerName glpi.domain.local
    DocumentRoot /var/www/html/glpi
    
    <Directory /var/www/html/glpi>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    # Sécurité
    <Directory /var/www/html/glpi/config>
        Require all denied
    </Directory>
    
    <Directory /var/www/html/glpi/files>
        Require all denied
    </Directory>
    
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
    ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
</VirtualHost>
```

### Installation Wizard Web
```
1. Accéder : http://server/glpi
2. Sélectionner langue : Français
3. Licence : Accepter GPL v3+
4. Installation : Nouvelle installation
5. Base données :
   ├── Serveur : localhost
   ├── Utilisateur : glpiuser
   ├── Mot de passe : [password]
   └── Base : glpidb
6. Initialisation base données
7. Suppression sécurisée : rm /var/www/html/glpi/install/install.php
```

## 👤 Authentification & Comptes

### Comptes par Défaut
| Compte | Mot de passe | Profil | Usage |
|--------|--------------|--------|-------|
| `glpi` | `glpi` | Super-Admin | Administration complète |
| `tech` | `tech` | Technician | Support technique |
| `normal` | `normal` | Self-Service | Utilisateur standard |
| `post-only` | `post-only` | Post-only | Création tickets uniquement |

⚠️ **Changer immédiatement** les mots de passe par défaut !

### Configuration LDAP/Active Directory
```
Administration > Authentification > Annuaires LDAP:

Serveur principal : dc.domain.local
Port : 389 (LDAP) / 636 (LDAPS)
Racine DN : DC=domain,DC=local
Compte connexion : CN=glpi-service,CN=Users,DC=domain,DC=local
Mot de passe : [service account password]

Filtres utilisateurs :
Base DN : CN=Users,DC=domain,DC=local
Filtre connexion : (&(objectClass=user)(objectCategory=person))
Filtre recherche : (&(objectClass=user)(sAMAccountName=*))

# Exclusion comptes désactivés
(&(objectClass=user)(objectCategory=person)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
```

## 🏢 Entités & Profils

### Structure Profils
| Profil | Parc | Tickets | Interface | Usage Typique |
|--------|------|---------|-----------|---------------|
| **Super-Admin** | ✅ Complet | ✅ Complet | Standard | Administration GLPI |
| **Admin** | ✅ Étendu | ✅ Étendu | Standard | Gestion avancée |
| **Supervisor** | 🔶 Tech + | ✅ + Équipe | Standard | Responsable équipe |
| **Technician** | ✅ CRUD | ✅ Complet | Standard | Technicien support |
| **Hotliner** | ❌ | ✅ Complet | Standard | Premier niveau |
| **Observer** | 👁️ Lecture | 🔶 Limité | Standard | Consultation/reporting |
| **Self-Service** | ❌ | 🔶 Propres tickets | Simplifiée | Utilisateur final |

### Gestion Entités
```
Entités = Structure organisationnelle hiérarchique

Exemple:
├── Racine
    ├── Siège Social
    │   ├── Direction
    │   ├── Comptabilité
    │   └── RH
    ├── Agence Paris
    └── Agence Lyon

Habilitation = Profil + Entité + Récursivité
```

## 🎫 Gestion Tickets & SLA

### Cycle de Vie Ticket
```
Nouveau → Assigné → Planifié → En cours → Résolu → Clos
    ↓         ↓         ↓         ↓         ↓      ↓
  Création  Prise en  Planning  Travail   Test   Archive
            charge
```

### Configuration SLA
```
Administration > SLA:

Types de SLA:
├── SLA Résolution (TTR)
├── SLA Temps de prise en compte (TTO)
└── OLA (niveaux internes)

Niveaux d'escalade:
├── Niveau 1: 4h → Notification superviseur
├── Niveau 2: 8h → Escalade manager
└── Niveau 3: 24h → Direction
```

### Catégories & Gabarits
```bash
# Structure catégories (max 2 niveaux)
Matériel
├── Poste de travail
├── Imprimante
└── Réseau

Logiciel
├── Bureautique
├── Métier
└── Système

# Gabarits avec balises automatiques
Modèle ticket:
- Titre: [#categorie#] - #title#
- Description: Template pré-rempli
- Assignation: Auto selon catégorie
- Priorité: Selon règles métier
```

## 🔧 Automatisation & Règles

### Cron GLPI (Critique)
```bash
# Configuration crontab pour www-data
sudo crontab -u www-data -e

# Exécution toutes les minutes (recommandé production)
* * * * * /usr/bin/php /var/www/html/glpi/front/cron.php >/dev/null 2>&1

# Alternative: toutes les 5 minutes (charge réduite)
*/5 * * * * /usr/bin/php /var/www/html/glpi/front/cron.php >/dev/null 2>&1

# Vérifier état cron dans GLPI
Administration > Actions automatiques
```

### Règles Métier
```
Administration > Règles:

Types de règles:
├── Affectation entité (import)
├── Règles tickets (automatisation)
├── Règles matériel (classification)
└── Règles utilisateurs (provisioning)

Exemple règle ticket:
Critère: Catégorie = "Imprimante"
Action: 
├── Assigner à: Groupe "Support Niveau 1"
├── Priorité: 3 (Moyenne)
└── SLA: "Standard - 4h"
```

## 📊 Inventaire & Parc

### GLPI Agent Architecture
```
GLPI Agent → Serveur GLPI
    ↓              ↓
Inventaire     Base CMDB
local          centralisée

Agent HTTP Server: Port 62354 (par défaut)
Communication: HTTP/HTTPS vers /front/inventory.php
```

### Installation Agent Windows
```powershell
# Téléchargement depuis GitHub releases
# https://github.com/glpi-project/glpi-agent/releases

# Configuration wizard:
Server URL: https://glpi.domain.local/front/inventory.php
Options recommandées:
├── SSL certificate check: Activé (prod)
├── Proxy settings: Si nécessaire
├── TAG: Entité automatique (ex: "PARIS", "SIEGE")
├── Debug: Activé (troubleshooting)
├── Install as service: Activé
└── Start inventory: Activé

# Vérification post-install
http://localhost:62354
```

### Installation Agent Linux
```bash
# Installation via package
wget https://github.com/glpi-project/glpi-agent/releases/download/1.7/glpi-agent_1.7-1_all.deb
sudo dpkg -i glpi-agent_1.7-1_all.deb
sudo apt-get install -f  # Résoudre dépendances

# Configuration
sudo nano /etc/glpi-agent/agent.cfg

# Configuration essentielle
server = https://glpi.domain.local/front/inventory.php
tag = DATACENTER-LINUX
debug = 1

# Modules actifs
no-task = 
include-file = 
exclude-file = 

# Démarrage service
sudo systemctl enable glpi-agent
sudo systemctl start glpi-agent
sudo systemctl status glpi-agent

# Test manuel
sudo glpi-agent --server https://glpi.domain.local/front/inventory.php --debug
```

## 🔌 Extensions & Plugins

### Plugins Essentiels
| Plugin | Usage | Installation |
|--------|-------|-------------|
| **Formcreator** | Formulaires personnalisés | Marketplace |
| **Dashboard** | Tableaux de bord avancés | Marketplace |
| **Reports** | Rapports personnalisés | Marketplace |
| **Behaviors** | Automatisations avancées | Marketplace |
| **Data Injection** | Import CSV massif | Marketplace |
| **Fields** | Champs personnalisés | Marketplace |

### Installation Plugin
```bash
# Via Marketplace (recommandé)
Configuration > Plugins > Marketplace
├── Rechercher plugin
├── Installer
└── Activer

# Installation manuelle
cd /tmp
wget plugin-archive.tar.gz
tar -xzf plugin-archive.tar.gz
sudo mv plugin-directory /var/www/html/glpi/plugins/
sudo chown -R www-data:www-data /var/www/html/glpi/plugins/plugin-directory
# Activer via Interface Web
```

### Data Injection (Import Massif)
```
Outils > Data Injection:

Formats supportés:
├── CSV (délimiteur configurable)
├── Mapping colonnes automatique
├── Validation avant import
└── Log des erreurs

Cas d'usage:
├── Import parc existant
├── Migration depuis autre outil
├── Mise à jour massive
└── Provisioning utilisateurs
```

## 🔒 Sécurité & Maintenance

### Durcissement Sécurisé
```bash
# Permissions système
sudo chmod 750 /var/www/html/glpi/config
sudo chmod 750 /var/www/html/glpi/files
sudo chmod 750 /var/www/html/glpi/install  # Si présent

# Configuration Apache/Nginx
<Directory /var/www/html/glpi/config>
    Require all denied
</Directory>

<Directory /var/www/html/glpi/files>
    Require all denied
</Directory>

# Suppression post-install
rm -f /var/www/html/glpi/install/install.php
```

### Configuration SSL/TLS
```apache
<VirtualHost *:443>
    ServerName glpi.domain.local
    DocumentRoot /var/www/html/glpi
    
    # SSL Configuration
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/glpi.crt
    SSLCertificateKeyFile /etc/ssl/private/glpi.key
    
    # Security headers
    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
</VirtualHost>
```

### Sauvegarde & Restauration
```bash
# Sauvegarde complète
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/glpi"

# Base de données
mysqldump -u glpiuser -p glpidb > $BACKUP_DIR/glpi_db_$DATE.sql

# Fichiers GLPI
tar -czf $BACKUP_DIR/glpi_files_$DATE.tar.gz -C /var/www/html glpi

# Configuration système
cp -r /etc/apache2/sites-available/glpi* $BACKUP_DIR/apache_config_$DATE/

# Nettoyage anciens backups (30 jours)
find $BACKUP_DIR -name "*.sql" -mtime +30 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete
```

## 📈 Monitoring & Performance

### Métriques Importantes
```bash
# Base de données
SHOW TABLE STATUS FROM glpidb;
SELECT COUNT(*) FROM glpi_tickets WHERE is_deleted = 0;
SELECT COUNT(*) FROM glpi_computers WHERE is_deleted = 0;

# Logs Apache
tail -f /var/log/apache2/glpi_access.log
grep "POST /glpi/front/inventory.php" /var/log/apache2/glpi_access.log

# Performance système
htop  # Utilisation CPU/RAM
df -h  # Espace disque
```

### Optimisation Base de Données
```sql
-- Nettoyage périodique
DELETE FROM glpi_logs WHERE date_creation < DATE_SUB(NOW(), INTERVAL 6 MONTH);
DELETE FROM glpi_events WHERE date < DATE_SUB(NOW(), INTERVAL 3 MONTH);

-- Maintenance
OPTIMIZE TABLE glpi_tickets;
OPTIMIZE TABLE glpi_computers;
ANALYZE TABLE glpi_tickets;
```

## 🎯 Workflow Types

### Workflow Incident
```
1. Détection → Alerte/Ticket
2. Classification → Catégorie/Priorité
3. Investigation → Diagnostic
4. Résolution → Solution appliquée
5. Validation → Test utilisateur
6. Clôture → Documentation KEDB
```

### Workflow Demande de Service
```
1. Demande → Formulaire standardisé
2. Approbation → Workflow validation
3. Planification → Assignation ressources
4. Réalisation → Exécution prestation
5. Livraison → Mise en service
6. Clôture → Satisfaction utilisateur
```

## 💡 Bonnes Pratiques

### Administration
- ✅ **Sauvegardes** quotidiennes automatisées
- ✅ **Mise à jour** régulière (security patches)
- ✅ **Monitoring** performances et disponibilité
- ✅ **Documentation** procédures et configurations
- ✅ **Formation** utilisateurs et administrateurs

### Utilisation
- ✅ **Catégorisation** précise des tickets
- ✅ **Templates** standardisés par type
- ✅ **SLA** adaptés aux niveaux de service
- ✅ **Automatisation** maximum des tâches répétitives
- ✅ **Reporting** régulier pour pilotage

### Sécurité
- ✅ **HTTPS** obligatoire en production
- ✅ **Comptes nominatifs** (pas de comptes partagés)
- ✅ **Droits minimals** selon principe du moindre privilège
- ✅ **Authentification** centralisée (LDAP/SSO)
- ✅ **Audit** des actions administratives

---
**💡 Memo** : Cron GLPI obligatoire, TAG pour entités auto, HTTPS en prod, sauvegardes avant MAJ !
