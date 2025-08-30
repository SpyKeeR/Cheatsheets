# SQL

## Connexion & opérations basiques
- Lancer client: `mysql -u root -p`.  
- Lister DB: `SHOW DATABASES;`.  
- Utiliser DB: `USE nom_base;`.  
- Lister tables: `SHOW TABLES;`.  
- Requête simple: `SELECT * FROM client;` ; 
- Filtrer: `WHERE ville='Caen'`.  
	- Opérateurs: `=, <, >, <=, >=, !=` (ou `<>`), `IS NULL`, `IS NOT NULL`, `<=>` (NULL-safe).  
	- Logique: `AND`, `OR`, `NOT`.  
- Trier: `ORDER BY colonne DESC, autre ASC`.  
- LIMIT/OFFSET: `LIMIT N OFFSET M`.  
- Fonctions: `COUNT()`, `MAX()`, `MIN()`, `SUM()`, `AVG()`.

## JOINs
- INNER JOIN: `SELECT * FROM achat INNER JOIN client ON client.id = achat.id_client WHERE client.prenom='Yan';` → joint tables correspondant.
