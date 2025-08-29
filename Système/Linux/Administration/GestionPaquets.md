# Gestion paquets — APT & dpkg (condensé) 📦

## APT (actions courantes)
- `apt update` → mettre à jour index (préférer `apt` moderne).  
- `apt upgrade` / `apt full-upgrade` → mettre à jour paquets.  
- `apt install pkg` → installer.  
- `apt remove pkg` → supprimer (garde configs).  
- `apt purge pkg` → supprimer + configs.  
- `apt clean` → nettoyer cache.  
- `apt search mot` → chercher paquets.  
- `apt show pkg` → détails.

## Obtenir les dépots 
```bash
cat /usr/share/doc/apt/examples/sources.list > /etc/apt/sources.list
```

### Debian 13 (Trixie) — deb822
- Sources format deb822 : `/etc/apt/sources.list.d/debian.sources` (remplace `sources.list`).  
	- Directives principales (clés)
	- Types : deb, deb-src
	- URIs : une ou plusieurs URL du dépôt
	- Suites : distribution(s) ciblée(s) (ex. trixie, trixie-updates, trixie-security)
	- Components : main, contrib, non-free, non-free-firmware
	- Signed-By : chemin du trousseau autorisé pour ce dépôt
- Directives utiles (optionnelles) :
	- Architectures : ex. amd64 i386
	- Enabled : yes/no pour activer/désactiver un bloc
	- By-Hash : yes/no (intégrité et cache)
	- Check-Valid-Until : yes/no (validation des métadonnées)
- Convertir fichiers sources.list : `sudo apt modernize-sources` 
	- Sauvegarde automatiques : `/etc/apt/sources.list.bak`, `.save`.
	

## dpkg (manuel)
- `dpkg -l` → paquets installés.  
- `dpkg -L pkg` → fichiers installés par paquet.  
- `dpkg -S /chemin/fichier` → quel paquet possède ce fichier.  
- `dpkg -i fichier.deb` → installer .deb local.
	
	
## Compilation (make)
- `./configure` → check dépendances & préparer build.  
- `make` → compiler.  
- `sudo make install` → installer (nécessite root).  
- Erreur sur `./configure` = dépendance manquante → installer et relancer.