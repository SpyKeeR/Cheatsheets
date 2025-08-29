# Imprimerie & déploiement — résumé ⚙️🖨️

## Concepts
- Imprimante (logique) vs Périphérique d'impression (matériel).  
- Files d’attente = jobs en attente 
- Ports = communication (TCP/IP, USB, LPT).  
- Pilotes serveur → clients récupèrent automatiquement si configuré.

## Permissions principales
- Imprimer (envoyer job).  
- Gérer documents (annuler/ordonner).  
- Gérer imprimante (config, pause, purge).
	- Lire/modifier les autorisations 
	- prendre possession de l’imprimante (cas avancés)

## Déploiement
- Manuel : `\\serveur-impression` → double-clic installe.  
- Script / PowerShell : automatiser ajout imprimantes à logon.  
- GPO : déployer imprimantes par OU (préférable pour scope large).

## Bonnes pratiques
- Héberger pilotes x86/x64 séparés ; tester compatibilité.  
- Documenter drivers & versions ; prévoir rollback.
- Pour autorisations, utiliser groupes (DL) et appliquer NTFS/partages correctement.