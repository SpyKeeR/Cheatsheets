## IOS — mémoire & rôle
- ROM → bootstrap, POST, mini-IOS (RxBoot).  
- Flash → image IOS (.bin).  
- NVRAM → startup-config (persistant).  
- RAM → running-config, table de routage (volatile).  
- Sauver config : `copy running-config startup-config`.  
- Voir config active : `show running-config`.

## Reset / mot de passe / ROMmon
- Erase startup : `erase startup-config` → `reload`.  
- Supprimer vlan table : `delete flash:vlan.dat`.  
- Registre config : `show version` (valeur par défaut 0x2102).  
- Ignorer startup-config pour reset PW : `config-register 0x2142` ou ROMmon `confreg 2142`.  (Default : 0x2102)
- Accès au mode ROMmon : Faire un Ctrl + Break au démarrage
- ROMmon cmds : 
	- `boot` → tenter de booter manuellement un fichier
	- `confreg` → changer le registre de config
	- `dir` → lister les fichiers présents
	- `reset` → redémarrer l’équipement
	- `set` / `unset` → afficher/modifier les variables d’environnement
	- `tftpdnld` → récupérer une image IOS depuis un TFTP (⚠️ efface toute la flash !)

## Restauration / transfert IOS & configs
- Restore running-config depuis TFTP : `copy tftp: running-config`.  
- Copier image IOS flash : `copy tftp: flash:` (fichier .bin).  
- Backup config : `copy running-config tftp:` ; restore : `copy tftp: startup-config` ou `running-config`.
- Ecraser un IOS depuis ROMmon : 
	- Définir les variables réseau dans ROMmon :
		- IP_ADDRESS, IP_SUBNET_MASK, DEFAULT_GATEWAY = Node Config IP
		- TFTP_SERVER = IP du serveur TFTP
		- TFTP_FILE = nom exact du fichier .bin
	- Lancer `tftpdnld`, fichier téléchargé → reboot automatique une fois terminé

## Console & paramètres série
- Ports : RJ-45 console, USB mini-B.  
- Settings : 9600 bauds, 8 bits, no parity, 1 stop bit, no flow control

## Mode utilisateur → privilégié → config
- `enable` → privileged (Switch#).  
- `configure terminal` → global config (Switch(config)#).  
- `interface <id>` → mode interface (Switch(config-if)#).  
- `end` ou `Ctrl+Z` → sortir.

## CLI aide & édition
- `?` → complétion / options.  
- `Tab` → autocomplete.  
- ↑ / ↓ → historique.  
- Backspace, ← → mouvements.
- `show version` → Voir les infos ROM, version, licence active