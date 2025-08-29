## Partage réseau
- Authentification double : partage + NTFS.
- Règle : l’autorisation la plus restrictive s’applique.
- Bonnes pratiques : donner contrôle aux "Utilisateurs authentifiés" sur partage et affiner via NTFS.

## Commandes partages
- `net use X: \\Serveur\Partage` → mapper.
- `net view \\Serveur` → lister partages.
- PowerShell mapping : `New-SmbMapping -LocalPath X: -RemotePath \\Serveur\Partage`
- Créer partage (CMD) : `net share MonPartage=C:\Dossier /GRANT:"Utilisateurs authentifiés",FULL`
- Créer partage (PS) : `New-SmbShare -Name "MonPartage" -Path "C:\Dossier" -FullAccess "Utilisateurs authentifiés"`