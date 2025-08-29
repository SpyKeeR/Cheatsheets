# Filesystems

## Filesystems & outils
- Créer FS : `mkfs.ext4 /dev/sdc1`
- tune2fs : `tune2fs -L label /dev/sdc1` (labeling), `tune2fs -l /dev/sdc1` (Listing), `tune2fs -i 1m /dev/sdc1` (fsck tous les mois).  
- resize2fs → redimensionner FS après modification LV/partition.  
- vérifier FS : `fsck /dev/sdX1`.

## EXT2/3/4 (rapide)
- EXT2 : pas de journalisation.  
- EXT3 : ajoute journalisation.  
- EXT4 : modernisé, préallocation, moins de fragmentation.  
- EXT4 limites usuelles : fichier max ≈16 TiB (avec 4 KiB blocks), noms ≤255 octets, ~4G fichiers.


## Blocs & inodes (résumé)
- Superbloc → infos vitales FS.
	- Taille des blocs
	- Taille totale du FS
	- Nombre de montages effectués
	- Nombre max de montages avant forçage de vérification
	- Date du dernier montage
	- Intervalle max entre deux vérifications
	- Pointeur vers l'inode racine
- Bloc Inode
	- 📄 Type de fichier (fichier classique, répertoire, lien symbolique...)
	- 🔒 Mode et droits d'accès (lecture, écriture, exécution)
	- 🔗 Nombre de liens physiques (si = 0 ➔ fichier supprimé)
	- 👤 UID (propriétaire) et GID (groupe)
	- 📏 Taille du fichier
	- 📆 Dates importantes :
		- atime : dernière lecture
		- mtime : dernière modification de contenu
		- ctime : dernière modification de métadonnées
- Bloc d'indirection → Permet de rediriger vers d'autres blocs (gros fichiers).
- Blocs de données → Stockent les vraies données du fichier (ex: le contenu de ton .txt, ton image, ta vidéo...).
- Table des inodes → Liste de tous les blocs d'inodes existants dans le FS.
- Table des inodes libres → Liste de tous les inodes libres.
- Table des blocs libres → Liste de tous les blocs de données disponibles.

## UUID / blkid / lsblk / df / du
- `blkid` → UUID, LABEL, TYPE.  
- `lsblk` → arbre disques ; `lsblk -f` → labels & UUID.  
- `df -h` → usage disque (human). `df -i` → inodes.  
- `du -h` / `du -hs` → taille dossiers.

## mount / umount / fstab
- Monter : `mount -t ext4 /dev/sdc1 /mnt` > -t ext4 ➔ Précise que le système de fichiers est ext4.
	- options `-o`
		- sync	→ Force l’écriture immédiate sur le disque (plus sûr, mais plus lent)
		- suid	→ Autorise les permissions SUID pour certains programmes spéciaux
		- exec	→ Autorise l’exécution de programmes depuis ce système de fichiers
		- remount	→ Re-monte un volume déjà monté (modifie ses options sans démonter)
		- ro	→ Monte le système de fichiers en lecture seule
		- rw	→ Monte le système de fichiers en lecture/écriture
		- defaults	→ Options standard (rw, suid, dev, exec, auto, nouser, async)
		- nouser	→ Seul root peut monter/démonter
		- dev	→ Interprète les fichiers spéciaux (périphériques)
		- auto	→ Monte automatiquement au boot
		- netdev	→ Attend que le réseau soit disponible (utile pour NFS/CIFS)
- `findmnt` → affichage propre.  
- Démonter : `umount /mnt` ou `umount /dev/sdc1`.  
- `/etc/fstab` colonnes : source (UUID=.../LABEL=/dev/...), mount point, type, options, dump, pass.  
- Utiliser `UUID=` pour stabilité.
