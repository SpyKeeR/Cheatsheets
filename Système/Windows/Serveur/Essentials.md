## Éditions Windows Server
- Standard → PME, rôles usuels.  
- Datacenter → virtualisation intensive, features avancées.  
- CALs requises pour accès clients (licences).

## Mode Core
- Server Core = sans GUI → administration via PowerShell / remote tools.
- Installer rôles : `Install-WindowsFeature -Name <NomDuRole> -IncludeManagementTools`.

## Rôles & services courants
- AD (Active Directory) → annuaire central.  
- DNS → résolution noms ↔ IP.  
- DHCP → distribution IP (DORA).  
- WSUS → gestion centralisée des updates.  
- Hyper-V → Hyperviseur Type 1 (Host OS devient Guest OS) 

## Outils MMC utiles
- services.msc, compmgmt.msc, gpmc.msc, dhcpmgmt.msc, dns.msc.

## PowerShell & Core
- PowerShell indispensable en Core.  
- Installer/retirer rôles via `Install-WindowsFeature` / `Remove-WindowsFeature`.
	- Option `-IncludeManagementTools` : Pour les outils de gestion associés.