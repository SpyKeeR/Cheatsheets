## DHCP — basique
- Options courantes : Option 3 = gateway, 6 = DNS, 15 = domain name, BOOTP, etc.
	- Option 119 → pour search domain-names (Multiple Suffixe DNS) (Créer règle personnalisée) 
		- Doit être [encodé en hex](https://jjjordan.github.io/dhcp119/) 
		- Puis [en décimal](https://gchq.github.io/CyberChef/), 
		- Renseigner les valeurs une à une dans le sens inverse dans le tableau de données de la règle.
- Processus DORA : Discover → Offer → Request → Acknowledge.  
- Durée bail : ex. 6 jours ; renouvellement à 50% et 87.5% du bail.
	- Libération manuelle de bail : Windows : `ipconfig /release` - Linux : `dhclient -r`
- Ports UDP : 67 (Serveur) - 68 (Client)
		
## DHCP avancé (Windows)
- Failover : modes load-balancing (50/50) ou hot standby.  
- Split-scope : répartir plage (ex. 20% / 80%) pour tolérance/charge.
- Serveur autorisé dans AD pour fonctionner.
- DHCP-Relay : Permet de transmettre les DORA hors du domaine de diffusion. 

## Sauvegarde & logs
- Sauvegardes synchrones automatiques (toutes les heures) + backups asynchrones manuels avant maj.  
- Logs : `%windir%\System32\dhcp\` et Event Viewer.