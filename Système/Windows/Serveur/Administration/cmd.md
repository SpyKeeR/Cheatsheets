# CLI cmd.exe (DOS) ⚙️

## Principes rapides
- `help` ou `commande /?` → aide intégrée pour chaque commande.  
- Syntaxe : Obligatoire = texte sans `[]`/`{}` ; Facultatif = entre `[]` ; `|` = OU ; `...` = répétition.  
- Exécuter en administrateur : 
	- ouvrir CMD via "Exécuter en tant qu’administrateur" (UAC). 
	- `runas /user:DOMAIN\Administrateur cmd` **ne contourne pas** toujours UAC (utile pour changer d'utilisateur, pas pour élever).

## Navigation & fichiers
- `cd [chemin]` → changer dossier (`cd ..`, `cd \`, `cd /d D:\chemin` pour changer de lecteur).  
- `dir [options]` → lister (`dir /a`, `/s`, `/b` pour sortie basique).  
- `tree [chemin] /f` → arborescence (+ fichiers).  
- `mkdir nom_dossier` ou `md nom_dossier` → créer dossier(s).  
- `rmdir /s /q nom_dossier` → supprimer dossier et contenu (attention).  
- `copy source dest` → copier fichier(s) simple.  
- `move source dest` → déplacer/renommer.  
- `del /f /q *.log` → supprimer fichiers (`/f` force, `/q` silencieux).  
- `rename ancien.ext nouveau.ext` → renommer.


## Copie robuste / synchronisation
- `xcopy src dst /E /I /H /K /Y` → copie arborescente, conserve attributs ; `xcopy` ancien, attention aux limites.
	- `/E` → Copie tous les sous-dossiers, y compris vides. ; `/S` copie sous-dossiers **sauf** vides.)
	- `/I` → Indique au système que la destination est un dossier. No more “file or directory?” interrogations.
	- `/H` → Copie fichiers cachés et systèmes.
	- `/K` → Conserve les attributs en lecture/écriture.
	- `/Y` → Supprime la confirmation avant d'écraser un fichier existant.
	- `/C` → Continuer copie malgré erreurs.
- `robocopy src dst [filtres] /MIR /Z /R:3 /W:5 /MT:8 /LOG+:C:\log.txt`  
	- `/MIR` → mirror (attention, supprime les fichiers supprimés), `/Z` → restartable, `/MT` → multithread, `/LOG` → journalisation(`+`=Append). 
	- `/R:n` → Nombre de tentatives en cas d’erreur, `/W:n` → Temps d’attente (secondes) entre chaque tentative.
	- `/XA:attr` → Exclut fichiers par attributs, par ex `/XA:H` ignore fichiers cachés.
	- `/XO` (exclude older), `/XN` (exclude newer), `/FFT` (pour différences de timestamp sur NAS), `/SEC` (copie les permissions NTFS).


## Diagnostics système & réparation
- `chkdsk C: /f /r` → vérifie et répare disque (peut demander redémarrage).  
- `sfc /scannow` → vérifie intégrité des fichiers système.  
- `DISM /Online /Cleanup-Image /RestoreHealth` → corrige l'image Windows (utiliser avant/avec SFC si nécessaire).  
- `systeminfo` → résumé machine (OS, BIOS, etc.).  
- `wmic logicaldisk get caption,size,freespace,filesystem` → infos disques (WMIC encore utile parfois).


## Services & processus
- `tasklist` → liste processus (`/v` verbose).  
- `taskkill /PID 1234 /F` ou `taskkill /IM nom.exe /F` → tuer processus.  
- `sc query <ServiceName>` → état d’un service.  
- `sc start|stop|restart <ServiceName>` → gérer service.  
- `net start` / `net stop <ServiceName>` → alternative simple.  
- `schtasks /Create /SC DAILY /TN "Sauve" /TR "C:\scripts\backup.bat" /ST 02:00` → planifier tâche.
	- `/Create` → Crée une nouvelle tâche, `/SC` → schedule (Fréquence : MINUTE, HOURLY, DAILY, WEEKLY, MONTHLY, ONCE, ONSTART, ONLOGON, etc...)
	- `/TN` → Nom de la tâche, `/TR` → Chemin de l'exécutable/script, `/ST` →  Heure de début (format 24h), `/RU` username  → Compte sous lequel la tâche s’exécute, `/RP` → mot de passe du compte.
	- `/RL` → Niveau de privilèges : LIMITED ou HIGHEST. `/F` → Force la création en écrasant, si tâche existante.

## Réseau (cmd classique)
- `ipconfig /all` → config IP complète.  
- `ipconfig /release` & `ipconfig /renew` → DHCP refresh.  
- `ipconfig /flushdns` → vider cache DNS.  
- `ping host -n 4` → test latence.  
- `tracert host` → traceroute.  
- `nslookup host` → requête DNS interactive.  
- `netstat -ano` → connexions et PID (utile pour corréler avec tasklist).  
- `route print` / `route add` / `route delete` → manipuler table routage.  
- `arp -a` → table ARP.


## Permissions & sécurité fichiers
- `takeown /f C:\folder /r /d Y` → prendre possession (admin).  
- `icacls "C:\folder" /grant "UTILISATEUR:(F)" /T` → accorder droits (F = Full).  
- `icacls path /reset /T` → réinitialiser ACLs hérités.  
- `cipher /w:C:\` → écraser l’espace libre (secure wipe).  
- `net user` → gérer comptes locaux (`net user username /add`, `/delete`, `/active:no`).


## Redirections, pipes & filtres
- `>` écrase, `>>` ajoute (`commande > fichier.txt`).  
- `2>` redirige erreur standard (`2> errors.txt`).  
- `2>&1` combine STDERR et STDOUT (`commande > out.txt 2>&1`).  
- `|` pipe vers autre commande (`dir /s | findstr /i ".log"`).  
- `more`, `find`, `findstr` (regex simple), `sort`, `type` pour filtrage/lire gros fichiers.


## Batch & automatisation (basique)
- Variables : `set VAR=val` puis `%VAR%` pour l'utiliser.  
- Persistantes : `setx VAR "value"` (nécessite nouvelle session pour être visible).  
- Conditions / boucle :  
  - `if exist "fichier" echo ok`  
  - `for /r %i in (*.log) do echo %i` (dans script `.bat` utiliser `%%i`), (`/r` = Current Folder)  
  - `forfiles /P C:\Logs /S /M *.log /D -30 /C "cmd /c del @path"` → supprimer fichiers plus vieux que 30 jours. 
	- `/P` → chemin de départ. `/S` → inclure sous-dossiers. `/M` → masque/filtres (ex *.log).
	- `/D -30` → fichiers date de modif. est antérieure à 30 jours. `+30` = plus récents que 30 jours.
	- `/C "cmd /c ..."` → commande à exécuter ; variables disponibles : `@path`, `@file`, `@fname`, `@ext`, `@isdir`, `@fsize`, `@fdate`, `@ftime`.


## Commandes utiles supplémentaires
- `echo` → afficher texte; `echo off` pour silence dans les batchs.  
- `more` → page par page.  
- `assoc` / `ftype` → types de fichiers et associations.  
- `powercfg` → diagnostic énergie (e.g. `powercfg /batteryreport`).  
- `certutil -hashfile fichier SHA256` → checksums rapides.  
- `net use \\serveur\share /user:DOM\user` → monter partage réseau.  
- `net share` → gérer partages locaux.  
- `gpupdate /force` → appliquer GPO immédiatement.  
- `gpresult /R` → vérifier résultats GPO pour l'utilisateur/machine.


## Exemples pratiques — "one-liners" utiles
- Trouver process qui écoute sur port 1433 :  
  `netstat -ano | findstr :1433` → note PID, puis `tasklist /FI "PID eq 1234"`  
- Prendre possession + donner droits complets :  
  `takeown /f "C:\Protected" /r /d y && icacls "C:\Protected" /grant Administrators:F /t`


## Raccourcis pratiques (CMD)
- Flèche Haut/Bas → parcourir historique.  
- `Tab` → autocomplétion chemins.  
- `Ctrl + C` → interrompre commande (ou copie).  
- `F7` → historique en popup (puis touche pour choisir).  
- `F8` → cherche dans l’historique caractère par caractère.  
- `Alt + Enter` → basculer en plein écran (si supporté).


## Notes & conseils pratiques
- Privilège : teste d’abord en compte standard, puis élève si nécessaire. Toujours documenter opérations destructrices (`/MIR`, `rmdir /s`).  
- Logs : ajoute toujours ` /LOG:` ou redirection `> log 2>&1` pour les tâches planifiées.  
- Robocopy > xcopy pour fiabilité ; `robocopy` est ton ami pour sauvegardes.  

