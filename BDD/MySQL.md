# 🗄️ MySQL — Aide-mémoire

## 🔌 Connexion & Navigation

### Connexion Client
```bash
# Connexion locale
mysql -u root -p

# Connexion distante
mysql -u user -p -h hostname -P 3306 database_name

# Connexion avec commande directe
mysql -u root -p -e "SHOW DATABASES;"

# Fichier de config (~/.my.cnf)
[client]
user=myuser
password=mypass
host=localhost
```

### Navigation Basique
```sql
-- Lister bases de données
SHOW DATABASES;

-- Utiliser une base
USE ma_base;

-- Lister tables
SHOW TABLES;

-- Structure d'une table
DESCRIBE ma_table;
SHOW CREATE TABLE ma_table;

-- Statut serveur
SHOW STATUS;
SHOW VARIABLES LIKE 'version%';
```

## 🗃️ Gestion Bases & Tables

### Bases de Données
```sql
-- Créer base
CREATE DATABASE ma_base CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Supprimer base
DROP DATABASE ma_base;

-- Modifier charset base
ALTER DATABASE ma_base CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Tables Essentielles
```sql
-- Créer table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB CHARACTER SET utf8mb4;

-- Modifier table
ALTER TABLE users ADD COLUMN last_login DATETIME;
ALTER TABLE users MODIFY COLUMN username VARCHAR(100);
ALTER TABLE users DROP COLUMN old_column;

-- Supprimer table
DROP TABLE ma_table;
```

### Types de Données Courants
| Type | Description | Exemple |
|------|-------------|---------|
| `INT` | Entier 4 bytes | `id INT AUTO_INCREMENT` |
| `BIGINT` | Entier 8 bytes | `user_id BIGINT` |
| `VARCHAR(n)` | Chaîne variable | `username VARCHAR(50)` |
| `TEXT` | Texte long | `description TEXT` |
| `DECIMAL(p,s)` | Décimal précis | `price DECIMAL(10,2)` |
| `DATETIME` | Date et heure | `created_at DATETIME` |
| `TIMESTAMP` | Timestamp Unix | `updated_at TIMESTAMP` |
| `BOOLEAN` | Booléen (TINYINT) | `is_active BOOLEAN` |
| `JSON` | Document JSON | `metadata JSON` |

## 🔍 Requêtes & Manipulation

### CRUD Essentielles
```sql
-- SELECT avec conditions
SELECT username, email FROM users 
WHERE created_at > '2023-01-01' 
ORDER BY username 
LIMIT 10 OFFSET 20;

-- INSERT
INSERT INTO users (username, email) VALUES 
('john', 'john@example.com'),
('jane', 'jane@example.com');

-- UPDATE
UPDATE users SET last_login = NOW() WHERE username = 'john';

-- DELETE
DELETE FROM users WHERE created_at < '2022-01-01';
```

### Jointures Courantes
```sql
-- INNER JOIN
SELECT u.username, p.title 
FROM users u 
INNER JOIN posts p ON u.id = p.user_id;

-- LEFT JOIN (tous les users, même sans posts)
SELECT u.username, COUNT(p.id) as post_count 
FROM users u 
LEFT JOIN posts p ON u.id = p.user_id 
GROUP BY u.id;

-- Sous-requête
SELECT username FROM users 
WHERE id IN (SELECT user_id FROM posts WHERE status = 'published');
```

### Fonctions Utiles
```sql
-- Agrégation
SELECT COUNT(*), AVG(age), MAX(salary), MIN(salary) FROM employees;

-- Chaînes
SELECT CONCAT(first_name, ' ', last_name) as full_name FROM users;
SELECT SUBSTRING(email, 1, LOCATE('@', email)-1) as username FROM users;

-- Dates
SELECT DATE(created_at), YEAR(created_at), DATEDIFF(NOW(), created_at) FROM posts;

-- Conditionnelles
SELECT username, 
       CASE WHEN last_login > DATE_SUB(NOW(), INTERVAL 7 DAY) 
            THEN 'Active' ELSE 'Inactive' END as status 
FROM users;
```

## 💾 Sauvegardes & Restauration

### mysqldump — Export
```bash
# Sauvegarde base simple
mysqldump -u root -p ma_base > ma_base.sql

# Plusieurs bases
mysqldump -u root -p --databases base1 base2 > bases.sql

# Toutes les bases
mysqldump -u root -p --all-databases > full_backup.sql

# Options importantes --opt (par défaut) inclut :
# --add-drop-table : DROP TABLE IF EXISTS avant CREATE
# --add-locks : LOCK/UNLOCK TABLES pour performance
# --create-options : Options complètes (ENGINE, CHARSET)
# --disable-keys : Désactive index pendant INSERT
# --extended-insert : Regroupe plusieurs INSERT
# --lock-tables : Verrouillage lecture pendant dump
# --quick : Lecture directe sans cache mémoire
# --set-charset : Ajoute SET NAMES dans le dump

# Options utiles
mysqldump -u root -p --opt --single-transaction --routines --triggers ma_base > ma_base.sql

# Exclure tables
mysqldump -u root -p --ignore-table=ma_base.logs --ignore-table=ma_base.cache ma_base > ma_base.sql

# Structure seule (sans données)
mysqldump -u root -p --no-data ma_base > structure.sql

# Données seules (sans structure)
mysqldump -u root -p --no-create-info ma_base > data.sql

# Compression à la volée
mysqldump -u root -p ma_base | gzip > ma_base.sql.gz
```

### Restauration
```bash
# Restaurer base
mysql -u root -p nom_base < sauvegarde.sql

# Créer base puis restaurer
mysql -u root -p -e "CREATE DATABASE ma_base;"
mysql -u root -p ma_base < ma_base.sql

# Depuis fichier compressé
gunzip < ma_base.sql.gz | mysql -u root -p ma_base

# Avec monitoring progression
pv ma_base.sql | mysql -u root -p ma_base
```

## 👥 Gestion Utilisateurs & Sécurité

### Comptes Utilisateurs
```sql
-- Créer utilisateur
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'secure_password';
CREATE USER 'readonly'@'%' IDENTIFIED BY 'readonly_pass';

-- Modifier mot de passe
ALTER USER 'app_user'@'localhost' IDENTIFIED BY 'nouveau_password';

-- Supprimer utilisateur
DROP USER 'old_user'@'localhost';

-- Lister utilisateurs
SELECT user, host, authentication_string FROM mysql.user;
```

### Privilèges (Principe du moindre privilège)
```sql
-- Accès complet à une base
GRANT ALL PRIVILEGES ON ma_base.* TO 'app_user'@'localhost';

-- Lecture seule
GRANT SELECT ON ma_base.* TO 'readonly'@'%';

-- Privilèges spécifiques
GRANT SELECT, INSERT, UPDATE ON ma_base.users TO 'limited_user'@'localhost';

-- Appliquer les changements
FLUSH PRIVILEGES;

-- Voir privilèges
SHOW GRANTS FOR 'app_user'@'localhost';

-- Révoquer privilèges
REVOKE ALL PRIVILEGES ON ma_base.* FROM 'app_user'@'localhost';
```

## ⚡ Performance & Index

### Index Essentiels
```sql
-- Créer index
CREATE INDEX idx_username ON users(username);
CREATE INDEX idx_email_status ON users(email, status);

-- Index unique
CREATE UNIQUE INDEX idx_email_unique ON users(email);

-- Index partiel (avec condition)
CREATE INDEX idx_active_users ON users(username) WHERE status = 'active';

-- Supprimer index
DROP INDEX idx_username ON users;

-- Analyser utilisation index
SHOW INDEX FROM users;
EXPLAIN SELECT * FROM users WHERE username = 'john';
```

### Optimisation Requêtes
```sql
-- Analyser plan d'exécution
EXPLAIN FORMAT=JSON SELECT * FROM users u 
JOIN posts p ON u.id = p.user_id 
WHERE u.status = 'active';

-- Statistiques table
ANALYZE TABLE users;

-- Optimiser table
OPTIMIZE TABLE users;

-- Réparer table (en cas de corruption)
REPAIR TABLE users;
```

## 📊 Monitoring & Maintenance

### Informations Système
```sql
-- Processus en cours
SHOW PROCESSLIST;

-- Variables importantes
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'query_cache_size';

-- Statut serveur
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Queries';
SHOW STATUS LIKE 'Uptime';

-- Taille des bases
SELECT 
    table_schema as 'Database',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) as 'Size (MB)'
FROM information_schema.tables 
GROUP BY table_schema;
```

### Logs & Debug
```sql
-- Activer log des requêtes lentes
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;

-- Log général (attention performance)
SET GLOBAL general_log = 'ON';

-- Variables de log
SHOW VARIABLES LIKE 'log_%';
```

## 🔄 Réplication (Concepts)

### Configuration Master-Slave
```bash
# Configuration master (my.cnf)
[mysqld]
server-id=1
log-bin=mysql-bin
binlog-format=ROW

# Configuration slave
[mysqld]
server-id=2
relay-log=relay-bin
read-only=1
```

```sql
-- Sur le master : créer utilisateur réplication
CREATE USER 'repl'@'%' IDENTIFIED BY 'repl_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

-- Obtenir position binlog
SHOW MASTER STATUS;

-- Sur le slave : configurer réplication
CHANGE MASTER TO
    MASTER_HOST='master_ip',
    MASTER_USER='repl',
    MASTER_PASSWORD='repl_password',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=154;

-- Démarrer réplication
START SLAVE;

-- Vérifier statut
SHOW SLAVE STATUS\G
```

## 🛠️ Configuration & Tuning

### Paramètres Critiques (my.cnf)
```ini
[mysqld]
# Mémoire
innodb_buffer_pool_size = 1G          # 70-80% RAM disponible
key_buffer_size = 256M                 # Pour MyISAM (si utilisé)
query_cache_size = 64M                 # Cache requêtes SELECT

# Connexions
max_connections = 200
connect_timeout = 10
wait_timeout = 3600

# InnoDB
innodb_log_file_size = 256M            # Logs transactions
innodb_flush_log_at_trx_commit = 1     # Durabilité ACID
innodb_file_per_table = ON             # Fichier par table

# Charset par défaut
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# Logs
log-error = /var/log/mysql/error.log
slow_query_log = ON
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

## 🚨 Commandes d'Urgence

### Récupération & Debug
```bash
# Démarrage sans tables grant (mode recovery)
mysqld_safe --skip-grant-tables &

# Réinitialiser mot de passe root
mysql -u root
ALTER USER 'root'@'localhost' IDENTIFIED BY 'nouveau_password';
FLUSH PRIVILEGES;

# Vérifier intégrité tables
mysqlcheck --all-databases --check --auto-repair

# Forcer démarrage après crash
mysqld_safe --innodb-force-recovery=1

# Tuer processus bloquant
KILL <process_id>;  # Obtenu via SHOW PROCESSLIST
```

### Maintenance Rapide
```sql
-- Nettoyer logs binaires anciens
PURGE BINARY LOGS BEFORE '2023-01-01 00:00:00';

-- Reconstruire statistiques
ANALYZE TABLE ma_table;

-- Défragmenter table
OPTIMIZE TABLE ma_table;
```

## 💡 Bonnes Pratiques

### Sécurité
- ✅ Utiliser comptes dédiés par application avec privilèges minimaux
- ✅ Désactiver comptes inutiles : `DROP USER ''@'localhost';`
- ✅ Chiffrer connexions : `REQUIRE SSL` dans `GRANT`
- ✅ Surveiller tentatives de connexion suspectes

### Performance
- ✅ Index sur colonnes fréquemment utilisées dans WHERE/JOIN
- ✅ Éviter `SELECT *`, spécifier colonnes nécessaires  
- ✅ Utiliser `LIMIT` pour pagination
- ✅ Monitorer requêtes lentes régulièrement

### Sauvegarde
- ✅ Sauvegardes automatisées quotidiennes avec rotation
- ✅ Tester restauration sur environnement séparé
- ✅ Sauvegardes hors-site pour disaster recovery
- ✅ Documenter procédures de restauration