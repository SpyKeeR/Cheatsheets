# Outlook — Raccourci dépannage (très condensé) ✉️

## Réparer .pst
- `ScanPST.exe` → diagnostic & réparation des fichiers `.pst`.
- Microsoft EasyFix (ex : 20101) → réparation rapide.

## Recréer profil Outlook
- Panneau de configuration > Courrier > Gérer les profils.
- Utiliser Autodiscover pour recréer proprement.

## Tests & connectivité
- `Test-OutlookConnectivity` (PowerShell) → test connexion Exchange.
- Test Exchange / M365 → utiliser TestConnectivity (web tool).

## Journalisation & logs
- Logs Outlook : `%AppData%/Local/Temp/Journal Outlook` (activer si debug).
- Ouvrir Outlook en mode sans échec : `Outlook.exe /safe`.

## Modules / compléments
- Désactiver compléments gourmand : Fichier > Options > Compléments.