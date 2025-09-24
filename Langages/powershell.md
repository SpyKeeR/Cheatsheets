# ⚡ PowerShell — Aide-mémoire

## 🏗️ Concepts Fondamentaux

### Philosophy & Architecture
- **Objets** : PowerShell manipule des objets .NET (pas du texte brut)
- **Pipeline** : Transmission d'objets entre cmdlets (préservation type/propriétés)
- **Nommage** : Convention `Verbe-Nom` (Get-Process, Set-Location)
- **Versions** : Windows PowerShell 5.1 vs PowerShell Core 7+ (cross-platform)

### Structure Objet
```powershell
# Propriétés = caractéristiques (taille, nom, statut)
$process = Get-Process notepad
$process.Name                    # Accès propriété
$process.ProcessName            # Autre propriété

# Méthodes = actions (Stop(), Kill(), Refresh())
$process.Kill()                 # Appel méthode
(Get-Date).AddDays(2)          # Méthode avec paramètre

# Introspection complète
Get-Process | Get-Member        # Liste propriétés/méthodes
$var.GetType()                  # Type de variable
```

## 🔍 Discovery & Aide

### Exploration Commandes
```powershell
# Recherche cmdlets
Get-Command                     # Toutes commandes
Get-Command -Verb Get          # Par verbe
Get-Command -Noun Service      # Par nom
Get-Command *service*          # Pattern matching
Get-Command -CommandType Cmdlet # Types spécifiques

# Verbes disponibles
Get-Verb                       # Verbes approuvés
Get-Alias                      # Alias système
```

### Système d'Aide
```powershell
# Aide contextuelle
Get-Help Get-Process           # Aide basique
Get-Help Get-Process -Full     # Aide complète
Get-Help Get-Process -Examples # Exemples seulement
Get-Help Get-Process -Online   # Documentation web
Get-Help Get-Process -ShowWindow # Fenêtre GUI

# Aide conceptuelle
Get-Help about_Variables       # Concepts PowerShell
Get-Help about_*              # Tous sujets about

# Gestion aide hors-ligne
Update-Help                    # Télécharger aide
Update-Help -UICulture fr-FR   # Langue française
Save-Help -DestinationPath C:\Help  # Aide portable
```

## 📦 Modules & Profils

### Gestion Modules
```powershell
# Exploration modules
Get-Module                     # Modules chargés
Get-Module -ListAvailable      # Modules disponibles
$env:PSModulePath             # Chemins recherche

# Import/Export
Import-Module ActiveDirectory  # Charger module
Get-Command -Module AD*       # Commandes du module
Remove-Module ActiveDirectory  # Décharger

# Installation (PowerShell Gallery)
Install-Module -Name Az        # Installer module
Find-Module *Exchange*        # Rechercher modules
```

### Profils Utilisateur
```powershell
# Emplacements profils
$PROFILE                      # Profil utilisateur courant
$PROFILE.AllUsersCurrentHost  # Tous utilisateurs
Test-Path $PROFILE           # Vérifier existence

# Création/édition profil
New-Item -Path $PROFILE -Type File -Force
notepad $PROFILE             # Éditer profil
```

## 🔧 Variables & Types

### Types de Données
| Type | Déclaration | Exemple | Méthodes Utiles |
|------|-------------|---------|-----------------|
| **String** | `[string]$s` | `"texte"` | `.Length`, `.ToUpper()`, `.Replace()` |
| **Integer** | `[int]$n` | `42` | `.ToString()`, math ops |
| **Boolean** | `[bool]$b` | `$true/$false` | Logical ops |
| **Array** | `[array]$a` | `@(1,2,3)` | `.Count`, `+=` (recréé) |
| **ArrayList** | Collections | `New-Object System.Collections.ArrayList` | `.Add()`, `.Remove()` |
| **Hashtable** | `[hashtable]$h` | `@{key="value"}` | `.Keys`, `.Values` |
| **DateTime** | `[datetime]$d` | `Get-Date` | `.AddDays()`, `.ToString()` |

### Gestion Variables
```powershell
# Déclaration & typage
$var = "valeur"               # Auto-détection type
[int]$number = "42"           # Conversion forcée
[string]$text = 123           # Cast en string

# Portées (scopes)
$global:var = "globale"       # Visible partout
$script:var = "script"        # Script entier
$local:var = "locale"         # Fonction courante
$private:var = "privée"       # Scope actuel seulement

# Utilitaires variables
Get-Variable                  # Lister variables
Get-Variable var*            # Pattern matching
Remove-Variable -Name var    # Supprimer variable
```

### Tableaux & Collections
```powershell
# Array standard (immutable)
$array = @("a", "b", "c")
$array += "d"                # Recrée tableau entier
$array[0]                    # Premier élément
$array[-1]                   # Dernier élément
$array.Count                 # Nombre éléments

# ArrayList (mutable, performance)
$list = New-Object System.Collections.ArrayList
$list.Add("item")           # Ajouter
$list.Remove("item")        # Supprimer
$list.Insert(0, "first")    # Insérer position

# Hashtable (clé-valeur)
$hash = @{
    Name = "John"
    Age = 30
}
$hash.Name                  # Accès valeur
$hash.Keys                  # Toutes clés
$hash["NewKey"] = "value"   # Ajouter élément
```

## 🔄 Pipeline & Traitement

### Opérateurs Pipeline
```powershell
# Pipeline objet
Get-Process | Where-Object Name -eq "notepad"
Get-Service | Sort-Object Status | Format-Table

# Variables contextuelles
$_          # Objet courant (legacy)
$PSItem     # Objet courant (moderne, recommandé)

# Enchaînement commandes
cmd1; cmd2; cmd3           # Séquentiel
cmd1 && cmd2               # Si succès (PS 7+)
cmd1 || cmd2               # Si échec (PS 7+)
```

### Filtrage & Sélection
```powershell
# Where-Object (filtrage)
Get-Process | Where-Object {$_.CPU -gt 100}
Get-Service | Where-Object Status -eq "Running"
Get-ChildItem | Where-Object {$_.Name -like "*.txt" -and $_.Length -gt 1KB}

# Opérateurs de comparaison
-eq, -ne                   # Égal, différent
-gt, -ge, -lt, -le        # Comparaisons numériques
-like, -notlike           # Wildcards (* ?)
-match, -notmatch         # Regex
-contains, -notcontains   # Contient élément
-in, -notin              # Élément dans collection

# Select-Object (projection)
Get-Process | Select-Object Name, CPU, Id
Get-Service | Select-Object -First 5
Get-ChildItem | Select-Object -ExpandProperty Name  # Valeur brute
```

### Tri & Mesures
```powershell
# Sort-Object
Get-Process | Sort-Object CPU -Descending
Get-ChildItem | Sort-Object Length, Name

# Measure-Object (agrégation)
Get-Process | Measure-Object CPU -Sum -Average -Maximum
Get-ChildItem *.txt | Measure-Object Length -Sum
```

## 💾 Import/Export & Formats

### Formats de Données
```powershell
# CSV (Excel compatible)
Get-Process | Export-Csv "processes.csv" -NoTypeInformation -Delimiter ";"
Import-Csv "data.csv" -Delimiter ";" | Where-Object Status -eq "Active"

# JSON (APIs/configs)
Get-Service | ConvertTo-Json | Out-File "services.json"
Get-Content "config.json" | ConvertFrom-Json

# XML PowerShell (préserve types)
Get-Process | Export-Clixml "processes.xml"
$data = Import-Clixml "processes.xml"

# HTML (rapports)
Get-EventLog -LogName System -Newest 10 | ConvertTo-Html | Out-File "report.html"
```

### Lecture/Écriture Fichiers
```powershell
# Lecture
Get-Content "file.txt"        # Tout le fichier
Get-Content "file.log" -Tail 20 -Wait  # Suivi temps réel
Get-Content "big.log" | Select-Object -First 100

# Écriture
"Contenu" | Out-File "output.txt"
"Ajout" | Add-Content "output.txt"
Get-Process | Out-File "processes.txt" -Width 200
```

## 🎯 Propriétés Calculées

### Syntaxe & Exemples
```powershell
# Structure de base
@{Name="NomColonne"; Expression={$_.Propriété}}
@{N="Alias"; E={$_.Propriété}}  # Version courte

# Exemples pratiques
Get-ChildItem | Select-Object Name, @{N="SizeMB"; E={[math]::Round($_.Length/1MB, 2)}}

Get-Process | Select-Object Name, @{
    Name="MemoryMB"
    Expression={[math]::Round($_.WorkingSet/1MB, 2)}
}

# Formatage avancé
Get-Volume | Select-Object @{N="Drive"; E={$_.DriveLetter + ":"}}, 
                          @{N="Size"; E={($_.Size/1GB).ToString("N2") + " GB"}}
```

## 🔀 Structures de Contrôle

### Conditions
```powershell
# If/ElseIf/Else
if ($condition) {
    # Actions si vrai
} elseif ($autre_condition) {
    # Actions alternatives
} else {
    # Actions par défaut
}

# Switch (multi-conditions)
switch ($variable) {
    "value1" { "Action 1" }
    "value2" { "Action 2" }
    {$_ -gt 10} { "Supérieur à 10" }  # Script block
    default { "Autre cas" }
}

# Switch avec regex
switch -Regex ($text) {
    '^\d+$' { "Nombre" }
    '^[a-zA-Z]+$' { "Lettres seulement" }
}
```

### Boucles
```powershell
# ForEach (collection)
$services = Get-Service
foreach ($service in $services) {
    Write-Host $service.Name
}

# For (compteur)
for ($i = 0; $i -lt 10; $i++) {
    Write-Host "Iteration $i"
}

# While (condition avant)
$counter = 0
while ($counter -lt 5) {
    $counter++
}

# Do-While (condition après, min 1 exécution)
do {
    $input = Read-Host "Entrez 'quit' pour sortir"
} while ($input -ne "quit")

# Do-Until (inverse de While)
do {
    $response = Invoke-WebRequest $url -TimeoutSec 5 -ErrorAction SilentlyContinue
} until ($response.StatusCode -eq 200)
```

## 🎨 Formatage Affichage

### Format-* Cmdlets
```powershell
# Format-Table (défaut pour la plupart)
Get-Process | Format-Table Name, CPU, Id -AutoSize
Get-Service | Format-Table -GroupBy Status
Get-ChildItem | Format-Table -Property Name, Length -Wrap

# Format-List (vertical, détaillé)
Get-Process notepad | Format-List *  # Toutes propriétés
Get-EventLog System -Newest 1 | Format-List

# Format-Wide (colonnes, une propriété)
Get-ChildItem | Format-Wide Name -Column 4
Get-Process | Format-Wide -Property ProcessName -AutoSize
```

### Out-* Cmdlets
```powershell
# Destinations sortie
Out-File "result.txt"         # Fichier texte
Out-GridView                  # Fenêtre GUI interactive
Out-Printer                   # Imprimante
Out-Host                      # Console (défaut)
Out-Null                      # Suppression sortie

# Out-GridView avancé
Get-Process | Out-GridView -Title "Processus" -PassThru | Stop-Process
```

## 🔐 Sécurité & Gestion Erreurs

### Execution Policy
```powershell
# Vérifier politique actuelle
Get-ExecutionPolicy -List

# Niveaux de sécurité
Set-ExecutionPolicy Restricted      # Aucun script (défaut)
Set-ExecutionPolicy RemoteSigned    # Scripts locaux OK, distants signés
Set-ExecutionPolicy Unrestricted    # Tout autorisé ⚠️

# Scopes
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser  # Utilisateur courant
Set-ExecutionPolicy RemoteSigned -Scope Process      # Session courante
```

### Gestion Erreurs
```powershell
# Variables d'erreur globales
$Error                        # Tableau toutes erreurs
$Error[0]                    # Dernière erreur
$Error.Clear()               # Vider historique

# ErrorAction par cmdlet
Get-Service NonExistant -ErrorAction SilentlyContinue
Get-Process BadName -ErrorAction Stop
Get-Item MissingFile -ErrorAction Ignore

# ErrorVariable personnalisée
Get-Service -ErrorVariable myErrors
$myErrors                    # Erreurs stockées séparément

# Try/Catch/Finally
try {
    Get-Item "C:\NonExistant.txt" -ErrorAction Stop
} catch [System.IO.FileNotFoundException] {
    Write-Warning "Fichier non trouvé"
} catch {
    Write-Error "Erreur générique: $($_.Exception.Message)"
} finally {
    Write-Host "Nettoyage exécuté"
}
```

## 🌐 PowerShell Remoting

### Configuration & Sessions
```powershell
# Activation côté cible
Enable-PSRemoting -Force      # Active WinRM (ports 5985 HTTP, 5986 HTTPS)

# Sessions interactives
Enter-PSSession -ComputerName SERVER01
Enter-PSSession -ComputerName SERVER01 -Credential $cred
Exit-PSSession               # Quitter session

# Sessions persistantes
$session = New-PSSession -ComputerName SERVER01
Invoke-Command -Session $session -ScriptBlock { Get-Service }
Get-PSSession               # Lister sessions actives
Remove-PSSession $session   # Fermer session
```

### Exécution Distante
```powershell
# Commande ponctuelle
Invoke-Command -ComputerName SERVER01 -ScriptBlock { Get-EventLog System -Newest 10 }

# Multi-serveurs simultané
Invoke-Command -ComputerName SRV01,SRV02,SRV03 -ScriptBlock { Get-Service Spooler }

# Avec fichier script
Invoke-Command -ComputerName SERVER01 -FilePath "C:\Scripts\Audit.ps1"

# Import module distant
$s = New-PSSession -ComputerName DC01
Import-PSSession -Session $s -Module ActiveDirectory -Prefix Remote
Get-RemoteADUser -Identity "jdoe"  # Utilise module distant avec préfixe
```

### Credentials Sécurisés
```powershell
# Demande interactive
$cred = Get-Credential -UserName "DOMAIN\admin" -Message "Authentification requise"

# Depuis variables sécurisées
$password = ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("DOMAIN\admin", $password)

# Usage
New-PSSession -ComputerName SERVER01 -Credential $cred
```

## ⚙️ Fonctions & Paramètres

### Définition Fonctions
```powershell
# Fonction simple
function Get-SystemInfo {
    param(
        [Parameter(Mandatory=$true)]
        [string]$ComputerName,
        
        [Parameter()]
        [switch]$IncludeServices
    )
    
    $info = Get-ComputerInfo -ComputerName $ComputerName
    
    if ($IncludeServices) {
        $info | Add-Member -NotePropertyName Services -NotePropertyValue (Get-Service)
    }
    
    return $info
}

# Fonction avancée (cmdlet-like)
function Get-ProcessInfo {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline=$true)]
        [string[]]$ProcessName = "*"
    )
    
    begin { Write-Verbose "Démarrage traitement" }
    
    process {
        foreach ($name in $ProcessName) {
            Get-Process -Name $name | Select-Object Name, CPU, WorkingSet
        }
    }
    
    end { Write-Verbose "Traitement terminé" }
}
```

### Types de Paramètres
```powershell
# Paramètres typés
[Parameter(Mandatory=$true)]
[ValidateSet("Dev","Test","Prod")]
[string]$Environment

[Parameter()]
[ValidateRange(1,100)]
[int]$Percentage

[Parameter()]
[ValidateScript({Test-Path $_ -PathType Container})]
[string]$FolderPath

# Switch parameters
[Parameter()]
[switch]$Force
```

## 📊 Flux & Redirection

### Streams PowerShell
| Stream | Numéro | Description | Cmdlet |
|--------|--------|-------------|--------|
| **Success** | 1 | Sortie normale | `Write-Output` |
| **Error** | 2 | Erreurs | `Write-Error` |
| **Warning** | 3 | Avertissements | `Write-Warning` |
| **Verbose** | 4 | Infos détaillées | `Write-Verbose` |
| **Debug** | 5 | Debug | `Write-Debug` |
| **Information** | 6 | Informations | `Write-Information` |

### Redirections
```powershell
# Redirections basiques
Command > output.txt          # Sortie vers fichier (écrase)
Command >> output.txt         # Ajout au fichier
Command 2> errors.txt         # Erreurs vers fichier
Command 2>&1                  # Erreurs vers sortie standard

# Redirections PowerShell
Command *> all.txt            # Tous streams vers fichier
Command 3> warnings.txt       # Warnings vers fichier
Command 2> $null             # Ignorer erreurs
```

## 🛠️ Outils Système & WMI

### Informations Système
```powershell
# Informations générales
Get-ComputerInfo             # Infos système complètes
Get-Date                     # Date/heure système
$PSVersionTable              # Version PowerShell
$env:USERNAME                # Variables environnement
$env:COMPUTERNAME
[Environment]::OSVersion     # Version OS

# Processus & Services
Get-Process                  # Processus actifs
Get-Service                  # Services Windows
Get-EventLog System -Newest 50  # Journaux événements
```

### WMI/CIM (Windows Management)
```powershell
# CIM (moderne, cross-platform)
Get-CimInstance -ClassName Win32_ComputerSystem
Get-CimInstance -ClassName Win32_LogicalDisk
Get-CimInstance -ClassName Win32_NetworkAdapter

# Filtrage CIM
Get-CimInstance -ClassName Win32_LogicalDisk -Filter "DriveType = 3"  # Disques fixes
Get-CimInstance -ClassName Win32_Service -Filter "State = 'Running'"

# WMI (legacy Windows)
Get-WmiObject -Class Win32_ComputerSystem  # Legacy, éviter si possible
```

## 💡 Bonnes Pratiques

### Performance & Lisibilité
- ✅ **Filtrer tôt** : `Get-Process -Name "notepad"` vs `Get-Process | Where-Object Name -eq "notepad"`
- ✅ **Utiliser paramètres natifs** quand disponibles plutôt que pipeline
- ✅ **Variables explicites** : `$PSItem` au lieu de `$_` dans scripts
- ✅ **Types appropriés** : ArrayList pour collections modifiables fréquemment

### Sécurité & Robustesse
- ✅ **Validation entrées** : `[ValidateScript()]`, `[ValidateSet()]`
- ✅ **Gestion erreurs** : Try/Catch pour opérations critiques
- ✅ **Credentials sécurisés** : `Get-Credential`, jamais en dur
- ✅ **Execution Policy** appropriée selon environnement

### Code Quality
- ✅ **Nommage cohérent** : Convention Verbe-Nom
- ✅ **Commentaires** : `# Description` et help blocks
- ✅ **Indentation** : 4 espaces recommandés
- ✅ **Tests** : Pester pour tests unitaires

### Debugging
```powershell
# Techniques debug courantes
Write-Host "Debug: variable = $variable" -ForegroundColor Yellow
$variable | Out-Host          # Forcer affichage dans pipeline
$variable | Get-Member        # Vérifier type/propriétés
Set-PSBreakpoint -Script .\script.ps1 -Line 25  # Points arrêt
```