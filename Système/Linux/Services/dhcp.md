## DHCP — isc-dhcp-server / relais / failover (très condensé)

### Fichiers clés
- /etc/dhcp/dhcpd.conf → logique des baux, options globales, subnets, hosts
- /var/lib/dhcp/dhcpd.leases → baux dynamiques
- /etc/default/isc-dhcp-server → interfaces écoutées (INTERFACESv4="ens33")
- `journalctl -u isc-dhcp-server` ou → /var/log/syslog 

### Global (entête)
- `option domain-name` → nom de domaine transmis (ex: "lab.local")
- `option domain-name-servers` → DNS à fournir aux clients (ex: 8.8.8.8)
- `default-lease-time` / `max-lease-time` → durées des baux en secondes (21666 et 42000)
- `ddns-update-style none` → désactive les mises à jour DNS dynamiques
- `authoritative` → déclare que ce serveur est l'autorité DHCP sur ce réseau
- `log-facility local7` → niveau de journalisation, rarement modifié

### Subnet minimal
```bash
subnet 172.19.0.0 netmask 255.255.255.0 {
  range 172.19.0.10 172.19.0.250;
  option routers 172.19.0.1;
  option domain-name "lab.local";
  option domain-name-servers 8.8.8.8;
  host imprimante_hp { hardware ethernet 08:00:27:aa:bb:cc; fixed-address 172.19.0.5; }
  }
```
Toujours créer le subnet du réseau IP de l'interface en listen (en + du l'étendue traitée si besoin)


### Tests & contrôle
- Syntaxe : `dhcpd -t`
- Mode debug (live) : `dhcpd -d`
- Redémarrer : `systemctl restart isc-dhcp-server`
- Vérifier : `systemctl status isc-dhcp-server`

---

### DHCP Relay (isc-dhcp-relay) — bref
- /etc/default/isc-dhcp-relay : 
	- SERVERS="IP1 IP2 ..." → Définit à quel serveur DHCP le relais doit retransmettre.
	- INTERFACES="eth0 eth1 ..." → Liste les interfaces à utiliser (Ecoute/Retransmission Multi-VLAN)
	- OPTIONS="" 
		- -d : mode debug (log dans la console)
		- -q : mode silencieux
		- -a : active giaddr (indique au serveur d’où vient la requête)
- Lancer debug : `isc-dhcp-relay -d -i eth0 -i eth1 192.168.42.2`
- Usage : relais entre VLANs → conserve giaddr pour attribution correcte

---

## DHCP Secours 

- `default-lease-time 1800`,
- `max-lease-time 3600` ➜ baux courts (30min à 1h)
- `not authoritative` ➜ ce serveur ne s’impose pas comme autorité (évite conflits)
- `min-secs 5` ➜ temporisation pour laisser le principal agir


### DHCP Failover (ISC) — points essentiels
- Pair config : 
	- `address`, `peer address`, `port`, `peer port` ➜ adresses et ports des deux serveurs
	- `max-response-delay` : après combien de sec. sans nouvelle du partenaire, on le considère down (ex : 60)
	- `max-unacked-updates` : nombre d’updates envoyés sans ACK avant pause (ex : 10)
	- `load balance max seconds` : délai max avant réponse hors failover (ex : 3 sec)
	- `mclt` (Max Client Lead Time) : durée autorisée pour renouveler un bail de l'autre serveur (ex : 3600 sec)
	- `split` : % de charge du serveur principal (0-255), 128 = équilibré, + élevé = + de clients pour le primaire
- Dans chaque subnet : `failover peer "nom-pair"` > identifie le bloc de synchronisation à utiliser + range commun entre serveurs
- Synchro : même plage IP sur les deux, coordination interne

---

### Sécurisation (OMAPI)
- `omapi-port 7911`
- `key omapi_cle { algorithm hmac-md5; secret "BASE64..."; }`
- `omapi-key omapi_cle` → sécurise échanges/updates
- Génération clé suggérée par outils (ex: `dnssec-keygen` pour keys TSIG-like)

---

### Bonnes pratiques / architecture
- Baux courts en environnements dynamiques (30–60 min) si mobilité élevée
- Split IPs 80% principal / 20% secours (optionnel)
- Sauvegardes régulières des configs / exports avant modifs
- Considérer KEA pour infrastructures cloud/scale (remplace isc pour besoins modernes)