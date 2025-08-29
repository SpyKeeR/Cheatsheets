## Scope

- 👤 User (u) : l’utilisateur propriétaire
- 👥 Group (g) : le groupe propriétaire
- 🌍 Others (o) : tous les autres utilisateurs


## Permissions & octal (r/w/x)

| Droit                     | Octal | Impact sur un fichier            | Impact sur un répertoire       |
|---------------------------|-------|----------------------------------|--------------------------------|
| r (read)                  | 4     | Lire le contenu (ex : cat, less) | Lister les fichiers (ex : ls)  |
| w (write) (dépend de r)   | 2     | Modifier / supprimer le fichier  | Créer / supprimer fichiers     |
| x (execute) (dépend de r) | 1     | Exécuter un script ou un binaire | Traverser le dossier (ex : cd) |

## Droits spéciaux
| Droit spécial | Valeur octale | Effet sur fichiers                                         | Effet sur dossiers                                                      |
|---------------|---------------|------------------------------------------------------------|-------------------------------------------------------------------------|
| Setuid        | 4             | Le fichier s’exécute avec les droits du propriétaire       | Aucun effet                                                             |
| Setgid        | 2             | Le fichier s’exécute avec les droits du groupe             | Fichiers et sous-dossiers(setGID aussi) créés héritent du groupe parent |
| Sticky Bit    | 1             | Maintenait un binaire en swap pour accès rapide (obsolète) | Seul le propriétaire du fichier ou root peut supprimer ses fichiers     |

## Modifier les modes d'accès et propriétaires

- Special bits : `setuid (4xxx)`, `setgid (2xxx)`, `sticky (1xxx)` → visible dans `ls -l` (s/S, t/T).  
- `chmod` symbolique : `u+rw`, `g-s`, `o=rx`.  
- `chown user:group fichier` → changer proprio ; `chown -R` pour récursif.

## umask
- Définit permissions par défaut (ex : `umask 022` → fichiers 644, dossiers 755).  

### Où définir le umask selon le besoin ?
- Dans un terminal umask 022	Session utilisateur en cours (temporaire)
- \~/.bashrc, \~/.profile, etc.	Toutes les sessions futures de l’utilisateur
- /etc/profile	Tous les utilisateurs (session shell)
- /etc/login.defs	Valeur par défaut pour les nouveaux utilisateurs
- /etc/default/useradd	Affecte les nouveaux comptes créés
Fichiers ou scripts de services	Umask spécifique à un service ou une application