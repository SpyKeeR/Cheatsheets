# Migration de données

## Images d'installation
- `install.wim` (≈4,29 Go) → contient tous les fichiers système.
- `boot.wim` (WinPE) → lance l’installation / environnement de pré-installation.

## Migration & sauvegarde (essentiel)
- Élément à migrer : comptes & profils, configs logiciels, outils bureautiques, paramètres système (pilotes, messagerie, polices), fichiers/dossiers.
- Sauvegarde : solution centralisée (NAS / fichier serveur / cloud) ou outil de sauvegarde Windows intégré.
- USMT (User State Migration Tool) → migration profils & paramètres.
  - Exemple d’utilisation rapide : `USMT /capture /i:migration.xml /l:logfile.log`

## Récap rapide flux migration
- Sauvegarde centralisée ← (USMT capture) ← poste source  
- Déploiement image (install.wim/boot.wim) → restauration des états (USMT restore)