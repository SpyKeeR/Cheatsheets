# Stockage — formats & commandes rapides 💾

## Filesystems — résumé
- **NTFS** → chiffrement, compression, déduplication, permissions avancées (win10: volumes jusqu’à ~256 To).
- **FAT32** → limité fichier ≤ 4 Go (inadapté pour gros fichiers).
- **EXT4** → journalisation (Linux).
- **ReFS** → résilience & récupération (Windows Server).
- **UDF** → CD/DVD/BluRay / ISO.

## Disk Management (GUI)
- `diskmgmt.msc` → gestion des disques.

## Diskpart (CLI)
- `diskpart` → lancer.
- `list disk` → lister disques.
- `select disk X` → sélectionner.
- `convert dynamic` → passe un disque de base à dynamique
- `create partition primary` → créer partition.
- `format fs=ntfs quick` → formater rapide.
- `assign letter=E` → attribuer lettre.
- `active` → rendre partition active (boot).
- `delete partition` → supprimer.

## PowerShell stockage
- `Get-Disk` → lister disques.
- `Initialize-Disk -Number <n> -PartitionStyle GPT` → initialiser.
- `New-Partition -DiskNumber <n> -UseMaximumSize -AssignDriveLetter` → créer partition.
- `Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel "DATA"` → formater.

## Partition tables
- **MBR** → ancien ; max 4 partitions principales ; ≤ 2,2 To ; stocke table au secteur 0.
- **GPT** → moderne ; jusqu'à 128 partitions (Windows) ; supporte >2,2 To ; nécessite UEFI.
- **Disques dynamiques** → volumes extensibles, spanning, RAID logiciel.

## RAID & volumes
- RAID = regrouper disques pour perf / redondance.
- RAID0 : striping → +perf, **pas** de redondance.
- RAID1 : miroir → sécurité (copie exacte).
- RAID5 : striping + parité → bon compromis (≥3 disques).
- RAID10 : mirror+stripe → perf + redondance (≥4 disques).
- Matériel vs logiciel : 
	- Privilégier matériel pour perf & fiabilité (BIOS RAID / carte RAID)
	- Windows propose RAID logiciel via **Gestion des disques** (limité).

## Volumes dynamiques (Windows)
- Simple = partition classique.
- Spanned = agrège plusieurs disques (capacité, **pas** de redondance).
- Striped = RAID0 (performance).
- Mirrored = RAID1 (redondance).
- RAID-5 = nécessite Windows Server (tolérance + perf).

## Bonnes pratiques & notes rapides
- Sauvegarder avant conversion de disque.  
- RAID matériel + backup = meilleure sécurité.  
- Vérifier compatibilité firmware/UEFI pour démarrer sur volumes >2,2 To.