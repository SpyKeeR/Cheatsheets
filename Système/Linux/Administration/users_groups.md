# Utilisateurs & groupes — résumé ultra-condensé 🧑‍💻

## Type d'utilisateurs

- 👑 root (UID 0/GID 0)
- 🛠️ Utilisateurs systèmes (UID 1 à 999/GID 1 a 999)
	- System Account : comptes système internes sans shell (UID < 100 ou 500 ou 1000 selon distro)
	- Service Account : créés pour des services (ex : www-data, mysql) → souvent sans login ni mot de passe
- 👨 Utilisateurs classiques (UID ≥ 1000 / GID ≥ 1000)

## /etc/group (format)
- `group_name:password:GID:user_list` → liste des groupes, GID, membres secondaires.
	- `password` souvent `x` → réel stocké dans `/etc/gshadow`.

## /etc/gshadow (secure)
- `group_name:password:admins:members`  
	- `password` chiffré ; `!` = verrouillé ; `*` = pas utilisable.  
	- `admins` = ceux qui peuvent administrer le groupe (gpasswd).

## Gestion des groupes
- `groupadd nom` → créer groupe (`-g GID` pour fixer le GID, `-r` pour système).  
- `groupmod -n nouveau_nom nom` → renommer ; `-g nouveau_GID` → changer GID.  
- `groupdel nom` → supprimer.  
- `gpasswd -a user group` → ajouter membre secondaire.  
- `gpasswd -d user group` → retirer membre.

## /etc/passwd (format)
- `username:passwd:UID:GID:GECOS:homedir:shell` (`passwd` = `x` si shadow utilisé).
	- `username` : Le nom de l’utilisateur (login)
	- `passwd` : Contient généralement un x, signifiant que le vrai mot de passe est dans /etc/shadow
	- `UID` : Identifiant unique de l’utilisateur (ex : 0 pour root, >=1000 pour les comptes normaux)
	- `GID : Groupe principal de l’utilisateur (correspond à un groupe défini dans /etc/group)
	- `GECOS` : Informations libres sur l’utilisateur (nom complet, téléphone, etc.)
	- `homedir` : Répertoire personnel (ex : /home/maxime)
	- `shell` : Shell par défaut (ex : /bin/bash)

## /etc/shadow (sécurité)
- `username:password:lastchg:min:max:warn:inactive:expire:reserved`  
	- `password` chiffré ; `!` / `*` = verrouillé / pas de mot de passe.  
	- `lastchg` en jours depuis 1970.  
	- `min` : Délai minimum (en jours) avant de pouvoir changer le mot de passe
	- `max` : Durée maximale de validité du mot de passe (au-delà : changement obligatoire)
	- `warn` : Jours avant expiration où un avertissement est affiché
	- `inactive` : Délai (en jours) après l’expiration du mot de passe avant désactivation du compte
	- `expire` : Date d’expiration du compte (toujours en jours depuis 1970)

## Création & gestion utilisateurs
- `useradd -m -s /bin/bash -u UID -g GID -G grp1,grp2 nom` → créer (avec `-m` créer home, `-r` UID < 1000, -d /chemin/home ).  
- `userdel -r nom` → supprimer (+ home avec `-r`).  
- `passwd [nom]` → changer mot de passe ; `passwd -e nom` → forcer expiration ; `passwd -l/-u` → lock/unlock.  
- `usermod user` → Modifier un user
	- `-c "Commentaire"` : Modifie le champ GECOS (ex : nom complet).
	- `-d /nouveau/home` : Change le répertoire personnel.
	- `-m` : Déplace les fichiers vers le nouveau home (à combiner avec -d).
	- `-e YYYY-MM-DD` : Date d’expiration du compte.
	- `-f N` : Nombre de jours d’afk après expiration du PW avant disable.
	- `-g groupe` : Change le groupe principal.
	- `-G groupe1,groupe2` : Remplace tous les groupes secondaires.
	- `-aG groupe1,groupe2` : Ajoute à la liste existante des groupes.
	- `-l nouveau_login` : Change le nom de l’utilisateur.
	- `-s /chemin/shell` : Change le shell.
	- `-u UID` : Change l’UID.
	- `-L` : Verrouille le compte (empêche la connexion).
	- `-U` : Déverrouille le compte.