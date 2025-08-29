# CLI & PowerShell — condensé et utile ⚙️

## CMD (rapide)
- `cls` → effacer écran.
- `notepad` → ouvrir Bloc-notes.
- `timedate.cpl` → panneau date/heure.
- `help` ou `commande /?` → aide rapide.

## Syntaxe commune (doc)
- Obligatoire = texte sans `[]`/`{}` ; Facultatif = entre `[]` ; `|` = OU ; `...` = répétition.

## PowerShell — commandes d’aide
- `Update-Help` (admin) → met à jour l’aide.
- `Get-Help` → aide générale.
- `Help` → page par page.
- `man` → alias UNIX pour aide.
- `Get-Help <Cmdlet> -Examples` → exemples.
- `Get-Help <Cmdlet> -ShowWindow` → fenêtre avec recherche.
- `Get-Help <Cmdlet> -Online` → doc en ligne.

## Concepts PowerShell
- Propriétés = caractéristiques d’un objet (ex : taille fichier).
- Méthodes = actions (ex : Rename(), Stop()).
- `Get-Member` → lister méthodes/propriétés : `Get-Content | Get-Member`
- `$PSVersionTable` → info version PowerShell.
- Structure nommage : `Verbe-Nom` (Get, Set, New, Remove, Add).
- `Get-Command` → lister commandes ; `Get-Command -CommandType Cmdlet` → uniquement cmdlets.
- `Get-Alias`, `Get-Verb` → utilitaires pour discovery.