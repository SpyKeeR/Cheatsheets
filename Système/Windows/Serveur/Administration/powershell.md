# PowerShell — Cheatsheet

## Concepts PowerShell
- Propriétés = caractéristiques d’un objet (ex : taille fichier).
- Méthodes = actions (ex : Rename(), Stop()).
- `Get-Member` → lister méthodes/propriétés : `Get-Content | Get-Member`
- `$PSVersionTable` → info version PowerShell.
- Structure nommage : `Verbe-Nom` (Get, Set, New, Remove, Add).
- `Get-Command` → lister commandes
	- `Get-Command -CommandType Cmdlet` → uniquement cmdlets.
	- `-Name`, `-Verb`, `-Noun` → Options de filtrage
- `Get-Alias`, `Get-Verb` → utilitaires pour discovery.
- PowerShell Core ≠ PowerShell Windows (No WMI/CIM, No Remoting, No scheduling mgmt et moins d'alias)


## Recherche & aide
- `Get-Command` → trouver cmdlet/fonction/alias/script. Ex : `Get-Command -Name *service` ou `Get-Command -Verb Get`
- `Get-Help <Cmdlet>` → aide (description + paramètres)
  - `-Full` → aide complète  
  - `-Examples` → exemples seulement  
  - `-Online` → ouvre la doc Microsoft  
  - `-ShowWindow` → fenêtre GUI d'aide
- `Get-Help about_Variables` ⇒ Fiches informatives concepts PSH commençant par about_
- Mettre à jour l’aide : `Update-Help` ; `-UICulture fr-FR` ➜ obtenir version française
- Offline help : `Save-Help -DestinationPath D:\HelpOffline` puis `Update-Help -SourcePath D:\HelpOffline`


## Modules
- Lister modules chargés : `Get-Module`
- Lister modules disponibles : `Get-Module -ListAvailable`
- Chemins modules : `$env:PSModulePath`
- Installer/poser un module : déposer .PSM1 dans un des chemins et `Import-Module NomDuModule`
- Vérifier import : `Get-Command -Module NomDuModule`


## Profil utilisateur
- Créer profil si absent : `New-Item -Path $Profile -Type File -Force`
- Éditer : `notepad $Profile`
- Exemples utiles dans $Profile : 
	```powershell
	Import-Module ActiveDirectory
	Set-Location C:\Scripts
	Write-Host "Bienvenue Maxime 👋"
	Set-Alias ll Get-ChildItem
	```


## Objets & introspection
- PowerShell manipule des objets (propriétés + méthodes) — pas du texte brut.
- Voir propriétés/méthodes : `Get-Process | Get-Member` (🚫 éviter sur les cmdlets Set-)
- Appel méthode : `(Get-Date).AddDays(2)` → ajoute 2 jours
	- Les méthodes attendent des types de variables précis.
	- Les méthodes ToType() (ex: .ToInt16()) servent à convertir un type en un autre.
- Accès propriété : `(Get-Date).Year` → propriété Year


## Variables & portée
- Déclaration : `$Nom = Valeur` (typage automatique).  
- Forcer type : `[int]$n = 1`, `[string]$s = "txt"`.  
- Portées : `$global:var`, `$script:var`.  
- Lister : `Get-Variable`.  
- Type : `$var.GetType()` ; inspecter objets : `| Get-Member`.


## Types de variables
- `String` → chaîne (`"texte"`) ; méthodes `.Length`, `.Replace()`, `.ToUpper()`.  
- `Integer` → nombre (`1`) ; opérations + - \* / ; `.ToString()` ; `.Round(n)`.
- `Bool` → `$true` / `$false`.  
- `Object` → résultat d'une cmdlet (propriétés + méthodes).  
- `Array` → `@("A","B")` (taille fixe, simple, performant. ; `$array += "C"` recrée le tableau).  
- `ArrayList` → dynamique (ajouts/suppressions fréquents) : `$list = New-Object System.Collections.ArrayList` ; `.Add()`, `.Remove()`, `.Insert()`, `.RemoveAt()`.


## Pipes & enchaînements
- `|` → transmet objet au suivant (preserve object).
	- Pipeline : commande1 | commande2  
	- Objet courant dans pipeline : `$_` ou `$PSItem`
- `;` → enchaîne commandes indépendantes.  
- Continuer une ligne en dessous : backtick en fin de ligne du début
- Commentaire : `#` (ligne) ; bloc `#< ... >#`.


## Sélection / tri / agrégation
- `Select-Object` (alias `Select`) → choisir propriétés, `-First n`, `-Last n`, `-Unique`, `-ExpandProperty`
  - Ex : `Get-Service | Select Name,Status`
  - Ex : `Get-Process | Select -First 5`
- `Sort-Object` (alias `Sort`) → tri ; `-Descending` pour inverser
  - Ex : `Get-ChildItem | Sort-Object Length -Descending`
- `Measure-Object` - Calculs sur la sortie
  - Ex : `Get-Process | Measure-Object -Property <NomProp> -Sum -Average -Minimum -Maximum`


## Filtrage
- Filtrer : `| Where-Object { $_.Propriete -eq 'Valeur' }`  
	- Opérateurs simples : `-eq, -ne, -gt, -lt, -ge, -le, -like, -notlike` (versions sensibles casse : `-ceq`, `-cne` ...)
	- Opérateurs complexes : Regex : `-match`, `-notmatch` | Valeur dans tableau : `-contains`, `-notcontains` | Valeur dans string : `-in`, `-notin`
	- Combiner plusieurs conditions : `-and`, `-or` et blocs de conditions avec `()`
- Exemples :
  - `Get-Service | Where-Object { $_.Status -eq 'Running' }`
  - `Get-ADUser -Filter * -Properties * | Where-Object { ($_.Name -like '*R*' -and $_.Enabled -eq $false) -or ($_.GivenName -like '*A*') }`


## Mise en forme (affichage)
- `Format-Table` (`ft`) → tableau 
	- `-Property` ➜ Définir propriété(s)
	- `-GroupBy` ➜ Regrouper par propriété
	- `-Wrap` ➜ Force le retour à la ligne dans une celle pour afficher tout le contenu
	- `-HideTableHeaders` ➜ Pas d'entête colomnes.
- `Format-Wide` → une propriété, largeur de colomne configurable (`-Column n`, `-AutoSize`)
- `Format-List` → liste verticale ; `Format-List *` pour toutes les propriétés
- `Select -ExpandProperty <Prop>` → extraire valeur brute (utile pour scripts)


## Flux & redirections
- STDIN = 0 (clavier), STDOUT = 1 (sortie normale), STDERR = 2 (erreurs)  
- Redirections courantes : `>` écrase, `>>` ajoute, `2>` erreurs, `2>>` ajoute erreurs, `2>&1` fusionne erreurs+sortie, `2> $null` supprime erreurs.


## Export / Import & formats
- CSV : `Get-ADUser -Filter * | Export-Csv -Path "\\serveur\share\users.csv" -Delimiter ";" -NoTypeInformation`
  - `-Path` → chemin (UNC recommandé)
  - `-Delimiter` → séparateur (`;` ou `,`)
  - `-NoTypeInformation` → supprime l'entête Microsoft
  - `-Append` → ajouter au fichier existant
- Lire CSV : `Import-Csv "chemin\fichier.csv" -Delimiter ";" | <pipeline>` (Entête deviennent des propriétés)
- CliXML : `Export-CliXml` / `Import-CliXml` → XML préservant types PowerShell
- JSON : `ConvertTo-Json | Out-File chemin.json` ; lire : `Get-Content chemin.json | ConvertFrom-Json`
- HTML : `ConvertTo-Html | Out-File page.html` → rapport Web
- XLSX : module `ImportExcel` (installation requise) → export direct en .xlsx


## Lecture/écriture brute
- Lire fichier ligne par ligne : `Get-Content C:\fichier.txt`
- `-Tail 10` → Lire dernier N lignes
- `-wait` → Lire en temps réel 
- `Out-File "C:\out.txt"` → écrit la sortie dans un fichier
  - `-Append` → ajoute au fichier
  - `-Width <n>` → limite largeur ligne (par défaut 80)
- Exemple rapide : `Get-Service | Out-File "C:\ENI\Services.txt" -Width 120 -Append`


## Propriété calculée (Select / Select-Object)
- Syntaxe : `@{n='Nom';e={ <expression> }}` (ou `@{Name='Nom';Expression={...}}`)
- Exemples :
  - `Get-ChildItem -File | Select Name, @{n='Taille';e={[math]::Round($_.Length/1MB,2)}}`
  - `Get-Volume | Select @{n='Lecteur';e={$_.DriveLetter}}, @{n='Taille';e={($_.Size/1GB).ToString("N2") + " GB"}}`
- Utiliser pour formater/arrondir/concaténer avant export


## Contrôle : IF / SWITCH (cond.)
- IF :  
  - `if (condition) { ... } elseif (autre) { ... } else { ... }`
  - Exemple : `if ($user.Enabled) { Write-Host "Actif" } else { Write-Host "Inactif" }`
- SWITCH : valeur → plusieurs cas  
  - `switch ($val) { "a" {..} "b" {..} default {..} }`
  - Regex : `switch -regex ($var) { '\d' {..} }`


## Boucles
- WHILE (test avant) :
  - `$x=1 ; while ($x -lt 10) { $x++; }`
  - Nécessite initialisation `$x` avant
- DO { } WHILE (test après, exécute au moins 1 fois) :
  - `do { $x = Read-Host "Choix (Q pour quitter)"} while ($x -ne "q")`
- DO { } UNTIL (inverse) :
  - `do { ... } until ($x -eq "q")`
- FOREACH (itération collection) :
  - `foreach ($item in $collection) { ... }`
  - Ex : `$users = Get-ADUser -Filter * ; foreach ($u in $users) { Write-Host $u.Name }`
- Break / Continue :
  - `break` → quitte la boucle immédiatement
  - `continue` → saute l'itération courante


## Gestion des erreurs
- `$Error` → tableau d'erreurs (dernier = `$Error[0]`).  
- Cmdlet-level/Violation : `-ErrorAction Continue|SilentlyContinue|Ignore|Stop|Inquire|Suspend`
	- Continue : affiche l’erreur et continue (valeur par défaut)
	- SilentlyContinue : ignore l’affichage, mais stocke l’erreur dans $Error
	- Ignore : ne fait rien, n’enregistre pas l’erreur
	- Stop : stoppe immédiatement l’exécution
	- Inquire : demande à l’utilisateur quoi faire
	- Suspend : suspend un workflow en cours;
- `-ErrorVariable nomVar` → Stockage erreur dans nomVar
- Définir violation au Global : `$ErrorActionPreference = "Stop"`  
- Structure :  
  - `try { ... } catch { ... } finally { ... }`  
  - `finally` s'exécute toujours (cleanup).


## Fonctions (mini)
- Syntaxe : 
	```powershell
	function Nom {
		param($p1, [Parameter(Mandatory=$true)][int]$p2)
		$p3 = $p1 + $p2
		return $p3
	}
	```	
- Retour : `return` ou sortie d'objet.  
- Exemple : `function Add($a,$b){ return $a+$b }` → `$r = Add 1 2`.


## Remoting (WinRM / PSSession)
- Activer cible : `Enable-PSRemoting -Force` (ouvre ports 5985 HTTP / 5986 HTTPS).  
- Session interactive : `Enter-PSSession -ComputerName HOST` → `Exit-PSSession`.
- Lister les sessions : `Get-PSSession` ; Quitter Session : `Exit-PSSession`
- Reconnecter une session déconnectée : `Connect-PSSession -Session $session`
- Commande distante ponctuelle : `Invoke-Command -ComputerName HOST -ScriptBlock { Get-Service }`  
- Session persistante : 
	```powershell
	$session = New-PSSession -ComputerName DC01  
	Invoke-Command -Session $session -ScriptBlock { Get-Service }  
	Remove-PSSession -Session $session
	```
- Importer modules depuis session distante : `Import-PSSession -Session $s -Module ActiveDirectory -Prefix D`  
- Multi-hôtes : `Invoke-Command -ComputerName A,B -ScriptBlock { ... }` 
- Auth & délégation : CredSSP / Kerberos nécessaire pour délégation multi-sauts (A -> B -> C).


## Get-Credential (identifiants sécurisés)
- ` $cred = Get-Credential -Message "..." -UserName "DOM\user"` → objet `System.Management.Automation.PSCredential` (username + SecureString password).  
- Usage : `Invoke-Command -ComputerName HOST -Credential $cred -ScriptBlock { ... }` ou `New-PSSession -Credential $cred`.

## Exécution de scripts
- `Set-ExecutionPolicy -ExecutionPolicy <Level> -Scope <Scope>` → définit politique d’exécution
	- Levels : 
		- Restricted → Par défaut. Aucun script n’est autorisé
		- AllSigned	→ Seulement les scripts signés sont exécutables
		- RemoteSigned → Scripts locaux : OK / Scripts distants : signés
		- Unrestricted → Tout peut s'exécuter (⚠️ Risqué)
		- Bypass → Aucune restriction, pas même d'avertissement
		- Default → Réinitialise selon le contexte (souvent Restricted)
	- Scopes : MachinePolicy, UserPolicy, Process, LocalMachine (Default), CurrentUser


## Bonnes pratiques rapides
- Préférer objets lors du piping de commande 
- Ne pas mélanger `Format-*` avant export/traitement.  
- Tester `$PSVersionTable.PSVersion` pour compatibilité (WinPS 5.1 vs PS7+).  
- Utiliser VSCode + extension PowerShell pour édition / debug.  
- Préférer `Select-Object` (propriétés) avant `Export-Csv` pour définir colonnes et ordre.
- Pour gros fichiers logs, `Get-Content -Tail` évite de charger tout le fichier.
- Tester les expressions calculées sur un petit jeu (`Select -First 5`) avant export massif.
- Utiliser UNC pour chemins partagés et vérifier droits écriture.
- Utilise `Get-Command` → trouve cmdlets avant d’essayer ; `Get-Help -Examples` pour usage rapide.
- `Get-Member` → indispensable pour savoir quoi tester dans `Where-Object`.
- Toujours filtrer tôt (`Where-Object`) pour réduire charge ; préfèrer les paramètres natifs des cmdlets (ex : `Get-Process -Name foo`) quand possible.
- `$_` est contextuel : lisibilité → préférer `$PSItem` dans scripts complexes.
- Attention aux types : `Measure-Object` nécessite propriété numérique.