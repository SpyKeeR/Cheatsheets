## su

`su -` → session root (tiret charge env) ; `su - username` → switch user
- \- ou -l : Lance un shell de login complet → charge les variables d’environnement de l’utilisateur cible (ex : $PATH du root).
- -s /bin/shell : Spécifie un shell différent (ex : /bin/bash, /bin/sh…).
- -c "commande" : Exécute une commande directement sans lancer une session interactive.

## sudo

- `sudo commande` → exécuter avec droits root (si autorisé).
	- `-i` : ouvre une session shell complète de type root (comme su -).
	- `-u utilisateur commande` : exécute une commande en tant qu’un autre utilisateur (pas forcément root).
  - Ajouter user au groupe `sudo`: `usermod -aG sudo user`.  
  - Modifier `/etc/sudoers` via `visudo`
		- Ajout d’une règle personnalisée, ex : nom_utilisateur ALL=(ALL:ALL) ALL
		- Format d'une ligne : 1 User ou %Group 2 Host (souvent ALL) 3 User ciblé (souvent ALL) 4 Commandes autorisées

### Exemples :
```bash
- root ALL=(ALL:ALL) ALL # root peut tout faire partout
- %admin ALL=(ALL) ALL # membres du groupe admin peuvent tout faire
- sarah localhost=/usr/bin/shutdown -r now # Sarah peut redémarrer la machine
```