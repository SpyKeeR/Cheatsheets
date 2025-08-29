# Filesystem Hierarchy Standard
```bash
/	# Racine de toute l’arborescence
/home	# Dossiers personnels des utilisateurs (= C:\Users sous Windows)
/root	# Dossier perso de root, le super-utilisateur (séparé de /home pour la sécurité)
/media & /mnt	# Montage des périphériques (USB, disques externes…)
/bin, /usr/bin, /usr/local/bin	# Commandes utilisateurs (ex : ls, cat)
/sbin, /usr/sbin	# Commandes admin/root (ex : reboot, iptables)
/etc	# Fichiers de configuration du système (l’équivalent du "registre" de Windows)
/boot	# Fichiers du bootloader (genre GRUB), indispensables au démarrage
/dev	# Fichiers représentant les périphériques (ex : /dev/sda, /dev/null)
/var	# Données variables : logs (/var/log), mails, files d’attente, BDD locales…
/tmp	# Fichiers temporaires, effacés au reboot
/proc /sys	# Répertoire virtuel pour interagir avec le noyau et obtenir des informations dynamique sur le système Répertoire virtuel permettant l'intéraction et la consultation des informations des périphériques et de leurs pilotes
/opt	# Logiciels installés manuellement, en dehors des dépôts
/run	# Répertoire virtuel avec données temporaires d'exécution des processus et services.
/lib, /usr/lib, /lib64	# Bibliothèques partagées (équivalents des .DLL)
/srv	# Dossiers pour les services réseau (sites web, FTP, etc.)
```