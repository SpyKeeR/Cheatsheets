# Linux — Opérations sur les fichiers

## Inode (quick)
- inode = métadonnées (type, UID/GID, perms, mtime/ctime/atime, taille, pointeurs).  
- Dernier lien supprimé → inode libéré → données supprimées.
- Types de fichiers : 
	- `-` → Fichier standard
	- `d` → Dossier
	- `l` → Lien symbolique
	- `b` → Bloc (
	- `c` → Character
	- `p` → Pipe
	- `s` → Socket

## Navigation & listing (ls)
- `pwd` → répertoire courant.  
- `ls` → Lister fichiers
	- -l	Format long (permissions, proprio, taille, date…)
	- -a	Affiche les fichiers cachés (commencent par un .)
	- -h	Affiche les tailles lisibles (K, M…)
	- -d	Affiche les infos d’un dossier sans lister son contenu (Impératif pour le globbing)
	- -t	Trie par date de modif
	- -r	Inverse l’ordre de tri
	- -R	Récursivité au sein des sous-dossiers.

## Fichiers : mv / cp / rm / mkdir / rmdir
- `mv src dst` → déplacer/renommer 
	- -v : mode verbeux (affiche les actions)
	- -f : force le remplacement sans confirmation
	- -i : demande confirmation avant d’écraser
	- -n : No Clobber (Pas d'écrasement si déjà présent) 
- `cp src dst` → copier 
	- -r : copie récursive (pour les dossiers)
	- -p : Préserver permissions/details
	- -n : No clobber (Pas de dossier, pas d'écrasement)
- `rm fichier` → supprimer (`-r -f`).  
- `rmdir` → supprimer dossier vide.  
- `mkdir -p` → créer arborescence.

## Liens
- Lien physique : `ln source lien` → même inode, pas pour répertoires/trans-partitions.  
- Lien symbolique : `ln -s source lien` → raccourci, casse si source bougée.

## Recherche & fichiers
- `find . -name "T*EL*"` → recherche, options courantes `-user -group -type -size -mtime`. (`-printf` → permet de formater la sortie)  
- `find . -type f -name "edition" -exec cp {} /tmp/edition.txt \;`.  → Execution de tâches pour chaque résultat
- `find . -name "*.bak" -ok rm {} \;`. → Execution avec confirmation

## locate
- `locate fichier.txt` → base indexée. 
	- -i → Ignore la casse (maj/min)
	- -r → Active les expressions régulières POSIX (ex : `'.*\conf$'`)
- `sudo updatedb` → mise à jour DB.  

## Lecture de fichiers

- `more` → Lecture de fichiers (-s → supprime doublons)
	- /mot → pour chercher un texte, → n et N pour naviguer entre les occurrences.
	- G → pour aller à la fin, → gg pour le début, → j et k pour défiler ligne par ligne.
	- h → pour afficher l’aide intégrée.

- `head` – Affiche le début d'un fichier
	- -n 5 ou -5 (Linux uniquement) pour personnaliser.

- `tail` – Affiche la fin
	- -n pour choisir combien de ligne.
	- -f pour suivre un fichier en temps réel

## Autres opérations
- `wc` – Compter le contenu
	- -l lignes, -w mots, -m caractères, -c octets.