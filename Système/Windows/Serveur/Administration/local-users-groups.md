# Utilisateurs, groupes, profils 🔐

## SIDs & jeton d’accès
- Tout utilisateur = SID ; groupes ajoutent SID au jeton.
- Jeton = identité + droits, utilisé à chaque check (`whoami /user`).

## Profils utilisateurs
- Local = stocké dans SAM.
- Domaine = AD (DC on-prem / Entra ID cloud). Auth via Kerberos.
- `C:\Users\Public` → contenu partagé à tous.
- `C:\Users\Default` → modèle pour nouveaux comptes.
- Gérer profils : Panneau → Système → Paramètres système avancés → Profils utilisateurs.
- Supprimer proprement : `Remove-WmiObject Win32_UserProfile -Filter "LocalPath='C:\Users\NomUtilisateur'"` (PowerShell).

## Groupes Windows (catégorie)
- Groupes locaux (gestion ressources locales).
- Groupes prédéfinis (Backup Operators, Event Log Readers...).
- Groupes intégrés (Authenticated Users, Everyone, Administrators).

## Commandes local accounts (CMD)
- `net user` → lister utilisateurs.
- `net user NomUtilisateur MotDePasse /add` → créer.
- `net user NomUtilisateur /delete` → supprimer.
- `net localgroup` → lister groupes.
- `net localgroup NomGroupe NomUtilisateur /add` → ajouter à groupe.
- `net localgroup NomGroupe NomUtilisateur /delete` → retirer.

## PowerShell users/groups
- `Get-LocalUser` → lister utilisateurs.
- `New-LocalUser -Name "Nom" -Password (ConvertTo-SecureString "Pwd" -AsPlainText -Force)` → créer.
- `Remove-LocalUser -Name "Nom"` → supprimer.
- `Get-LocalGroup` → lister groupes.
- `Add-LocalGroupMember -Group "Administrators" -Member "Nom"` → ajouter.
- `Remove-LocalGroupMember -Group "Administrators" -Member "Nom"` → retirer.