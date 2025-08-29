## Connexion & opérations basiques
- Lancer client: `mysql -u root -p`.

## Sauvegardes / restauration (mysqldump)
- Sauvegarde DB: `mysqldump -u root -p ma_base > ma_base.sql`.  
- Plusieurs DB: `mysqldump -u root -p --databases base1 base2 > bases.sql` (inclut CREATE DATABASE).  
- Toutes DB: `mysqldump -u root -p --all-databases > full_backup.sql`.  
- Ajouter `--add-drop-database` pour DROP+CREATE au restore.  
- Options importantes: 
	- `--opt` (par défaut), inclu :
		- `--add-drop-table` : Avant chaque CREATE TABLE, il ajoute un DROP TABLE IF EXISTS.
		- `--add-locks` : Entoure les inserts de LOCK TABLES et UNLOCK TABLES.
		- `--create-options` :  Ajoute toutes les options spécifiques d’une table (comme ENGINE=InnoDB, CHARSET=utf8mb4, etc.) dans le CREATE TABLE.
		- `--disable-keys` : Avant les INSERT, il met ALTER TABLE ... DISABLE KEYS, puis à la fin ENABLE KEYS.
		- `--extended-insert` : Au lieu de faire un INSERT par ligne, il met plusieurs lignes dans une seule commande INSERT.
		- `--lock-tables` : Pendant le dump, il verrouille les tables (READ LOCK).
		- `--quick` : Lit les lignes directement du serveur sans tout charger en mémoire.
		- `--set-charset` : Ajoute SET NAMES ... dans le dump.
	- `-c`/`--complete-insert` : Inclut les noms de colonnes dans chaque INSERT.
	- `-e`/`--extended-insert` : Regroupe plusieurs lignes dans un seul INSERT. 
- Exclure table: `--ignore-table=ma_base.nom_table` (répéter pour plusieurs).

- Restauration: `mysql -u root -p nom_base < sauvegarde.sql` (créer DB si nécessaire).

## Astuces
- Pour gros dumps, compresser à la volée: `mysqldump ... | gzip > dump.sql.gz`.  
- Tester restauration sur instance de dev avant prod.  
- Gérer utilisateurs DB avec droits restreints pour les bases (principe du moindre privilège).