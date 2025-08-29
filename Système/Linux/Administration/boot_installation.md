# Boot, installation & démarrage — condensé ⚙️

## Installation USB & clonage
- Outils USB recommandés : Ventoy, Rufus, Balena Etcher, Unetbootin.  
- FAI (Fully Automatic Install) = script + fichier réponses (preseed) → zéro intervention.  
- Clonage : CloneZilla, Fog, Symantec Ghost, Acronis TrueImage.

## Accès root & modèles d'administration
- Deux écoles : mot de passe `root` (su -) ou sudoers sans root (1er utilisateur admin via `sudo`).  
- Pour devenir root : `su -` (tiret important) ou `sudo -i` / `sudo -s`.

## POST → firmware → bootloader → kernel (pipeline)
1. POST (Power-On Self-Test) → vérifie hardware de base.  
2. BIOS vs UEFI → gère BootOrder.  
   - BIOS → cherche MBR (512 octets : bootloader + table partitions + signature).  
   - UEFI → lit partition EFI (FAT32) → exécutables `.efi`.  
3. GRUB2 prend le relais (Boot Manager + Loader).  
   - GRUB2 = stage1 (MBR/EFI) + stage2 (/boot/grub/).  
4. GRUB charge `vmlinuz` (noyau) et `initramfs`/`initrd` en RAM. 
   - `initramfs` = tmpfs + cpio (méthode actuelle).  
5. Noyau décompresse automatique, exécute initramfs → charge drivers → monte le vrai root et pivot_root.
	- Dès que les bons drivers sont chargés :
	- Le vrai disque contenant le système Debian est monté 🔗.
	- Un pivot_root a lieu :
		- Le noyau transfère la racine du système vers le vrai disque dur.
		- Le FS temporaire initramfs est effacé de la RAM 🧹
6. vmlinuz lance `/sbin/init` → généralement `systemd` (PID 1).  
	- systemd vas lire ses fichiers de config.
	- Charger les services selon les targets (ex : graphical.target, multi-user.target) requis.
7. Authentification via PAM → shell utilisateur.
	- Si console : `login` se lance et le shell user est appelé.

## GRUB — config & modification (pratique)
- Ne jamais éditer directement `/boot/grub/grub.cfg` (regénéré).  
- Modifier `/etc/default/grub` (+ scripts `/etc/grub.d/`) puis regénérer : `sudo grub-mkconfig -o /boot/grub/grub.cfg` ou `sudo update-grub` (Debian).

## Rescue / single-user
- Éditer ligne de boot (touche `e`) → ajouter `init=/bin/bash` pour shell root sans mot de passe.  
- Remonter en écriture : `mount -o remount,rw /` ; avant shutdown : `sync` puis `umount /`.  
- Single-user avec mot de passe : modifier ligne qui finit par `quiet` pour démarrer en mode maintenance.
- Si travail sur disque principal, charger kernel+userland via Boot CD Debian.

## Systemd — commandes clés
- `systemctl get-default` → cible par défaut.  
- `systemctl set-default multi-user.target` → changer.  
- `systemctl isolate rescue.target` → maintenance.  
- `systemctl list-units[ --all]` → unités actives / toutes.  
- `systemctl status/start/stop/restart/enable/disable [service]`.  
- Unit types : 
- `.service` → Un service (Apache, SSH, etc.)
- `.target` → Une cible (multi-user, graphical, rescue...)
- `.socket` → Communication réseau/IPC
- `.device` → Périphérique matériel (clé USB, disque...)

## Arrêt / reboot
- Arrêt immédiat : `shutdown -h now`  
- Arrêt programmé : `shutdown -h +10 "msg"`  
- Redémarrage : `reboot` ou `shutdown -r 16:30`  
- Annuler : `shutdown -c`