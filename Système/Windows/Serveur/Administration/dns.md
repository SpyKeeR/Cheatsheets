# DNS— cheat rapide 🌐

## DNS — basique
- Zones : directe (A / AAAA / CNAME / MX) et inverse (PTR).
	- Champ SOA → Identifie le serveur maître de la zone et les paramètres de la zone comme son numéro de série (incrémenter à chaque modif).
	- Champ NS : Liste des serveurs faisant autorité
- Types de zone : primaire (maître, editable) / secondaire (esclave, lecture seule).  
- Transferts : AXFR = full, IXFR = incrémental.  

## Types requêtes
- Requête récursive : demande une réponse finale, complète à un résolveur 🧾.
- Requête itérative : Demande un indice pour aboutir à une réponse du serveur faisant autorité sur la zone demandée 📬.


## Types Enregistrements
- Statiques : saisis à la main (ex : MX, A, CNAME pour mail, web...) → fiables, figés
- Dynamiques : créés automatiquement (ex : un poste rejoint un AD et s’enregistre dans DNS, avec A et PTR) → souples, adaptés aux IPs volatiles > TCP 53

## Synchronisation des serveurs
Si Primaire et Secondaire, il faut synchroniser les zones.➡️ assurer la redondance et la cohérence.
- Process manuel :
	- 🧭 Le secondaire interroge le maître : numéro de série changé ?
	- ✅ Si oui, il lance un transfert de zone AXFS ou IXFR
- Process auto : 🔔 Le maître peut notifier les secondaires après modif → plus réactif

## Résolution locale
- Ordre : cache local → hosts → mDNS (Résolution NetBIOS) → serveur DNS configuré.  
- Vider cache Windows : `ipconfig /flushdns`.
- Fichier LMHosts 
	- Windows : `C:\Windows\System32\drivers\etc\hosts`
	- Linux : `/etc/hosts`

## EDNS RFC 6891
- 📦 Support de paquets DNS > 512 octets (TCP 53) (pour IPv6, DNSSEC…)
- ⚙️ Permet d’ajouter des options supplémentaires sans modifier le protocole de base
- 🔄 Compatible avec les serveurs DNS récents
