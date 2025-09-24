# 🗄️ SQL — Aide-mémoire

## 🔌 Connexion & Navigation

### Connexion Basique
```sql
-- MySQL/MariaDB
mysql -u root -p
mysql -u user -p -h hostname -P 3306 database_name

-- PostgreSQL
psql -U username -d database_name -h hostname

-- SQLite
sqlite3 database.db

-- SQL Server
sqlcmd -S server -U username -P password -d database
```

### Navigation Environnement
```sql
-- Lister bases de données
SHOW DATABASES;                    -- MySQL/MariaDB
\l                                 -- PostgreSQL
.databases                         -- SQLite

-- Utiliser une base
USE nom_base;                      -- MySQL/SQL Server
\c nom_base                        -- PostgreSQL

-- Lister tables
SHOW TABLES;                       -- MySQL/MariaDB
\dt                               -- PostgreSQL
.tables                           -- SQLite

-- Structure table
DESCRIBE table_name;               -- MySQL/MariaDB
\d table_name                     -- PostgreSQL
.schema table_name                -- SQLite
```

## 🏗️ Structure Données (DDL)

### Création/Modification Tables
```sql
-- Créer table
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    age INT CHECK (age > 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Modifier structure
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users MODIFY COLUMN username VARCHAR(100);
ALTER TABLE users DROP COLUMN age;
ALTER TABLE users RENAME TO customers;

-- Index
CREATE INDEX idx_email ON users(email);
CREATE UNIQUE INDEX idx_username ON users(username);
DROP INDEX idx_email;

-- Contraintes
ALTER TABLE orders ADD CONSTRAINT fk_user 
    FOREIGN KEY (user_id) REFERENCES users(id);
```

### Types de Données Courants
| Type | MySQL | PostgreSQL | SQL Server | Description |
|------|--------|------------|------------|-------------|
| **Entier** | `INT`, `BIGINT` | `INTEGER`, `BIGINT` | `INT`, `BIGINT` | Nombres entiers |
| **Décimal** | `DECIMAL(p,s)` | `NUMERIC(p,s)` | `DECIMAL(p,s)` | Nombres précis |
| **Texte** | `VARCHAR(n)`, `TEXT` | `VARCHAR(n)`, `TEXT` | `NVARCHAR(n)` | Chaînes caractères |
| **Date/Heure** | `DATETIME`, `DATE` | `TIMESTAMP`, `DATE` | `DATETIME2` | Dates et heures |
| **Booléen** | `BOOLEAN` | `BOOLEAN` | `BIT` | Vrai/Faux |
| **Binaire** | `BLOB` | `BYTEA` | `VARBINARY` | Données binaires |

## 🔍 Requêtes de Base (DQL)

### SELECT Fondamentaux
```sql
-- Sélection basique
SELECT * FROM users;                    -- Toutes colonnes
SELECT username, email FROM users;      -- Colonnes spécifiques
SELECT DISTINCT city FROM users;        -- Valeurs uniques

-- Alias
SELECT username AS nom, email AS mail FROM users;
SELECT u.username, u.email FROM users u;    -- Alias table

-- Calculs
SELECT price, quantity, price * quantity AS total FROM orders;
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
```

### Filtrage (WHERE)
```sql
-- Opérateurs de comparaison
SELECT * FROM users WHERE age = 25;
SELECT * FROM users WHERE age != 30;      -- ou <> 30
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE age BETWEEN 18 AND 65;

-- Opérateurs logiques
SELECT * FROM users WHERE age > 18 AND city = 'Paris';
SELECT * FROM users WHERE age < 18 OR age > 65;
SELECT * FROM users WHERE NOT city = 'Lyon';

-- Pattern matching
SELECT * FROM users WHERE username LIKE 'john%';    -- Commence par john
SELECT * FROM users WHERE email LIKE '%@gmail.com'; -- Finit par @gmail.com
SELECT * FROM users WHERE phone LIKE '06________';  -- 06 + 8 caractères

-- Valeurs nulles
SELECT * FROM users WHERE phone IS NULL;
SELECT * FROM users WHERE phone IS NOT NULL;

-- Listes de valeurs
SELECT * FROM users WHERE city IN ('Paris', 'Lyon', 'Marseille');
SELECT * FROM users WHERE age NOT IN (18, 21, 25);
```

## 🔗 Jointures (JOINs)

### Types de Jointures
```sql
-- INNER JOIN (correspondances exactes)
SELECT u.username, o.amount 
FROM users u 
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN (tous les enregistrements de gauche)
SELECT u.username, o.amount 
FROM users u 
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN (tous les enregistrements de droite)
SELECT u.username, o.amount 
FROM users u 
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN (tous les enregistrements)
SELECT u.username, o.amount 
FROM users u 
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- CROSS JOIN (produit cartésien)
SELECT u.username, p.name 
FROM users u 
CROSS JOIN products p;
```

### Jointures Multiples
```sql
SELECT u.username, o.amount, p.name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;
```

## 📊 Fonctions d'Agrégation

### Fonctions de Base
```sql
-- Fonctions courantes
SELECT COUNT(*) FROM users;                    -- Nombre total
SELECT COUNT(phone) FROM users;               -- Non-null uniquement
SELECT COUNT(DISTINCT city) FROM users;       -- Valeurs uniques

SELECT SUM(amount) FROM orders;               -- Somme
SELECT AVG(amount) FROM orders;               -- Moyenne
SELECT MIN(amount), MAX(amount) FROM orders;  -- Min/Max

-- Avec GROUP BY
SELECT city, COUNT(*) as nb_users 
FROM users 
GROUP BY city;

SELECT city, AVG(age) as age_moyen 
FROM users 
GROUP BY city 
HAVING AVG(age) > 30;                         -- HAVING pour conditions sur agrégats
```

### Fonctions de Chaînes
```sql
-- Manipulation texte
SELECT UPPER(username) FROM users;            -- Majuscules
SELECT LOWER(email) FROM users;               -- Minuscules
SELECT LENGTH(username) FROM users;           -- Longueur
SELECT SUBSTRING(username, 1, 3) FROM users; -- Sous-chaîne

-- Concaténation
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;  -- MySQL
SELECT first_name || ' ' || last_name AS full_name FROM users;      -- PostgreSQL/SQLite
```

### Fonctions de Date
```sql
-- Date courante
SELECT NOW();                                 -- MySQL/PostgreSQL
SELECT GETDATE();                            -- SQL Server
SELECT datetime('now');                      -- SQLite

-- Extraction composants
SELECT YEAR(created_at), MONTH(created_at) FROM orders;        -- MySQL
SELECT EXTRACT(YEAR FROM created_at) FROM orders;              -- PostgreSQL/SQL Server

-- Calculs de dates
SELECT DATEDIFF(NOW(), created_at) AS days_ago FROM orders;    -- MySQL
SELECT AGE(NOW(), created_at) FROM orders;                     -- PostgreSQL
```

## 📝 Manipulation Données (DML)

### Insertion (INSERT)
```sql
-- Insertion simple
INSERT INTO users (username, email, age) 
VALUES ('john_doe', 'john@example.com', 25);

-- Insertions multiples
INSERT INTO users (username, email) VALUES 
('alice', 'alice@example.com'),
('bob', 'bob@example.com'),
('charlie', 'charlie@example.com');

-- Insertion depuis SELECT
INSERT INTO users_archive 
SELECT * FROM users WHERE created_at < '2020-01-01';
```

### Mise à Jour (UPDATE)
```sql
-- Mise à jour simple
UPDATE users SET age = 26 WHERE username = 'john_doe';

-- Mise à jour multiple
UPDATE users SET 
    age = age + 1,
    last_login = NOW() 
WHERE city = 'Paris';

-- Mise à jour conditionnelle
UPDATE products SET price = 
    CASE 
        WHEN category = 'Electronics' THEN price * 1.1
        WHEN category = 'Clothing' THEN price * 1.05
        ELSE price
    END;
```

### Suppression (DELETE)
```sql
-- Suppression conditionnelle
DELETE FROM users WHERE age < 18;
DELETE FROM orders WHERE created_at < '2020-01-01';

-- Suppression avec jointure (MySQL)
DELETE u FROM users u 
INNER JOIN orders o ON u.id = o.user_id 
WHERE o.status = 'cancelled';

-- Vider table (plus rapide que DELETE)
TRUNCATE TABLE logs;                          -- Remet auto-increment à 0
```

## 🔧 Requêtes Avancées

### Sous-Requêtes
```sql
-- Sous-requête dans WHERE
SELECT username FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE amount > 100);

-- Sous-requête corrélée
SELECT username FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- Sous-requête dans FROM
SELECT city, avg_age FROM (
    SELECT city, AVG(age) as avg_age 
    FROM users 
    GROUP BY city
) AS city_stats 
WHERE avg_age > 30;
```

### Expressions Conditionnelles
```sql
-- CASE simple
SELECT username,
    CASE age 
        WHEN 18 THEN 'Majeur'
        WHEN 16 THEN 'Mineur'
        ELSE 'Autre'
    END as status
FROM users;

-- CASE conditionnel
SELECT username,
    CASE 
        WHEN age >= 65 THEN 'Senior'
        WHEN age >= 18 THEN 'Adulte'
        ELSE 'Mineur'
    END as category
FROM users;

-- Fonctions conditionnelles
SELECT username, COALESCE(phone, 'N/A') as contact FROM users;  -- Première valeur non-null
SELECT username, NULLIF(age, 0) as valid_age FROM users;        -- NULL si égal
```

### Fenêtres (Window Functions)
```sql
-- Numérotation
SELECT username, age,
    ROW_NUMBER() OVER (ORDER BY age DESC) as rank,
    RANK() OVER (ORDER BY age DESC) as rank_with_ties
FROM users;

-- Par partition
SELECT username, city, age,
    RANK() OVER (PARTITION BY city ORDER BY age DESC) as city_rank
FROM users;

-- Fonctions agrégées
SELECT username, amount,
    SUM(amount) OVER (ORDER BY created_at) as running_total,
    AVG(amount) OVER (PARTITION BY user_id) as user_avg
FROM orders;
```

## 📋 Tri & Pagination

### ORDER BY
```sql
-- Tri simple
SELECT * FROM users ORDER BY age;                    -- Croissant (défaut)
SELECT * FROM users ORDER BY age DESC;               -- Décroissant

-- Tri multiple
SELECT * FROM users ORDER BY city ASC, age DESC;

-- Tri par expression
SELECT * FROM users ORDER BY LENGTH(username), username;

-- Gestion des NULL
SELECT * FROM users ORDER BY phone NULLS FIRST;      -- PostgreSQL
SELECT * FROM users ORDER BY phone NULLS LAST;       -- PostgreSQL
```

### Pagination
```sql
-- MySQL/PostgreSQL/SQLite
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;  -- Page 3 (20 records skipped)
SELECT * FROM users ORDER BY id LIMIT 10;            -- Premiers 10

-- SQL Server
SELECT * FROM users ORDER BY id OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;

-- Alternative SQL Server (legacy)
SELECT TOP 10 * FROM users WHERE id > 20 ORDER BY id;
```

## 🔒 Transactions & Intégrité

### Gestion Transactions
```sql
-- Transaction basique
BEGIN TRANSACTION;              -- ou START TRANSACTION; (MySQL/PostgreSQL)
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;                         -- Valider changements

-- Rollback en cas d'erreur
BEGIN TRANSACTION;
    UPDATE products SET stock = stock - 5 WHERE id = 1;
    -- Si erreur détectée
ROLLBACK;                       -- Annuler changements
```

### Points de Sauvegarde
```sql
BEGIN TRANSACTION;
    UPDATE table1 SET col = 'value1';
    SAVEPOINT sp1;
    UPDATE table2 SET col = 'value2';
    -- Erreur détectée
    ROLLBACK TO SAVEPOINT sp1;   -- Retour au point de sauvegarde
    UPDATE table2 SET col = 'value3';
COMMIT;
```

## 🎯 Optimisation & Performance

### Index & Analyse
```sql
-- Création index
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_order_date_user ON orders(created_at, user_id);

-- Analyse requête (MySQL)
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';
EXPLAIN FORMAT=JSON SELECT * FROM orders o JOIN users u ON o.user_id = u.id;

-- Analyse requête (PostgreSQL)
EXPLAIN ANALYZE SELECT * FROM users WHERE age > 25;

-- Statistiques tables
ANALYZE TABLE users;            -- MySQL
ANALYZE users;                  -- PostgreSQL
UPDATE STATISTICS users;        -- SQL Server
```

### Bonnes Pratiques Performance
- ✅ **Index sur colonnes WHERE/JOIN** fréquentes
- ✅ **Éviter SELECT *** sur grandes tables
- ✅ **Utiliser LIMIT** pour pagination
- ✅ **Préférer EXISTS** à IN pour sous-requêtes
- ✅ **Filtrer avant jointure** (WHERE avant JOIN si possible)

## 🛠️ Vues & Procédures

### Vues
```sql
-- Créer vue
CREATE VIEW active_users AS
SELECT username, email, created_at 
FROM users 
WHERE status = 'active';

-- Utiliser vue
SELECT * FROM active_users WHERE created_at > '2023-01-01';

-- Vue avec jointure
CREATE VIEW user_orders AS
SELECT u.username, u.email, o.amount, o.created_at
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- Supprimer vue
DROP VIEW active_users;
```

### Procédures Stockées (MySQL exemple)
```sql
-- Créer procédure
DELIMITER //
CREATE PROCEDURE GetUserOrders(IN user_id INT)
BEGIN
    SELECT o.id, o.amount, o.created_at 
    FROM orders o 
    WHERE o.user_id = user_id;
END //
DELIMITER ;

-- Appeler procédure
CALL GetUserOrders(1);

-- Procédure avec paramètres OUT
DELIMITER //
CREATE PROCEDURE GetUserStats(IN user_id INT, OUT total_orders INT, OUT total_amount DECIMAL(10,2))
BEGIN
    SELECT COUNT(*), SUM(amount) 
    INTO total_orders, total_amount
    FROM orders 
    WHERE orders.user_id = user_id;
END //
DELIMITER ;
```

## 🔧 Fonctions Utilitaires

### Conversion Types
```sql
-- Conversion explicite
SELECT CAST(age AS VARCHAR(10)) FROM users;
SELECT CONVERT(VARCHAR(10), age) FROM users;        -- SQL Server

-- Conversion dates
SELECT STR_TO_DATE('2023-12-25', '%Y-%m-%d');       -- MySQL
SELECT TO_DATE('2023-12-25', 'YYYY-MM-DD');         -- PostgreSQL/Oracle
```

### Fonctions Mathématiques
```sql
SELECT ABS(-15);                 -- Valeur absolue
SELECT ROUND(3.14159, 2);        -- Arrondi (3.14)
SELECT CEIL(3.2);                -- Entier supérieur (4)
SELECT FLOOR(3.8);               -- Entier inférieur (3)
SELECT RANDOM();                 -- Nombre aléatoire (PostgreSQL)
SELECT RAND();                   -- Nombre aléatoire (MySQL)
```

## 🔍 Requêtes Pratiques Courantes

### Top N par Groupe
```sql
-- Top 3 commandes par utilisateur
SELECT * FROM (
    SELECT username, amount, created_at,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY amount DESC) as rn
    FROM orders o
    JOIN users u ON o.user_id = u.id
) ranked 
WHERE rn <= 3;
```

### Doublons
```sql
-- Trouver doublons
SELECT email, COUNT(*) 
FROM users 
GROUP BY email 
HAVING COUNT(*) > 1;

-- Supprimer doublons (garder le plus récent)
DELETE u1 FROM users u1
INNER JOIN users u2 
WHERE u1.id < u2.id AND u1.email = u2.email;
```

### Pivot Simple
```sql
-- Transformer lignes en colonnes
SELECT 
    user_id,
    SUM(CASE WHEN category = 'Electronics' THEN amount ELSE 0 END) as electronics,
    SUM(CASE WHEN category = 'Clothing' THEN amount ELSE 0 END) as clothing,
    SUM(CASE WHEN category = 'Books' THEN amount ELSE 0 END) as books
FROM orders o
JOIN products p ON o.product_id = p.id
GROUP BY user_id;
```