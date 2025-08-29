## Privilege EXEC Security
- `enable password xxxx` ➜ protège l’accès au mode privilégié (en clair)
- `enable secret xxxx` ➜ protège l’accès au mode privilégié (chiffré[5])
- `service password-encryption` ➜ chiffre(7) tous les mots de passe en clair dans le running-config

### Banner MOTD
- `banner motd #Votre message ici#`

## Securiser Console
- `line console 0` ➜ on entre dans la ligne
- `password xxxx` ➜ mot de passe requis à l’accès
- `login` ➜ rend le mot de passe obligatoire à la connexion

## SSH — setup minimal
1. `ip domain-name example.tld`.  
2. `crypto key generate rsa` (>=2048).  
3. `ip ssh version 2`.  
4. `username user secret pass`.  
5. `line vty 0 4` → `login local` `transport input ssh`.  
- Vérif : `show ip ssh`, `show ssh`, `show user`,`show run | section vty`.
- Activer un timeout d'inactivité : `exec-timeout 5 0` (5 min)
- K9 dans image = chiffrement disponible (ex: `k9` dans nom de l'image).

## Port Security (switchport port-security)
- Activer : interface → `switchport port-security`.  
- Modes : `protect` (drop silencieux), `restrict` (log & count), `shutdown` (err-disable).  
- Sticky/Static : `switchport port-security mac-address sticky|static`.  
- Max MAC adress allowed: `switchport port-security maximum <nb>`.  
- Voir : `show port-security interface <id>`, `show port-security address`.  
- Réactiver err-disabled : interface `shutdown` puis `no shutdown`.

## Spanning Tree (STP) — options utiles
- PortFast (edge ports) : `spanning-tree portfast` (interface).  
- BPDU Guard : `spanning-tree bpduguard enable` → shutdown on BPDU.  
- BPDU Filter : `spanning-tree bpdufilter enable` (risqué).  

## DHCP Snooping & DAI
- DHCP Snooping : `ip dhcp snooping` + `ip dhcp snooping vlan <id>`.  
- Marquer uplink trust : `interface <id>` → `ip dhcp snooping trust`.  
- Rate limit DHCP : `ip dhcp snooping limit rate <val>`.  
- DAI (Dynamic ARP Inspection) : `ip arp inspection vlan <id>` (nécessite DHCP snooping).  
- Trust ports for DAI : `interface <id>` → `ip arp inspection trust`.

## WiFI — best practices
- WPA2/WPA3 + AES (WPA3 preferred).  
- Désactiver WPS.  
- SSID non-informatif ; changer par défaut.  
- Isoler SSID invité (VLAN séparé, firewall rules).  
- Désactiver association automatique côté client.  
- Activer isolation AP/client pour hotspots si besoin.

# ACL & NAT 

## ACL — types & usages
- Standard : filtre par IP source (numérotée 1-99,1300-1999). Placer proche de destination.  
- Étendue : filtre src+dst+proto+ports (100-199,2000-2699). Placer proche source du trafic à bloquer.  
- Syntaxe numérotée : `access-list 10 permit 192.168.1.0 0.0.0.255`.  
- Syntaxe nommée : `ip access-list extended NAME` … (plus lisible).
- Oneline ACE : `{ip} access-list <num>{nom} deny|permit <proto> <IP src> <wildcard> <IP dst> <wildcard> eq <port>`
- Wildcard = inverse du mask (ex: /26 → 0.0.0.63).  
- Scope Keywords : 
	- HOST = wildcard 0.0.0.0 → cible 1 seule IP 
	- ANY = wildcard 255.255.255.255 → autorise toutes les IP
- Appliquer : `interface G0/1` → `ip access-group NAME in|out`.  
- Commandes utiles : `show access-lists`, `show ip interface <iface>`, `clear access-list counters`.  
- ACE sequencing : numéros (10,20,30) → insertion possible entre lignes.  
- Apply VTY : `access-class ADMIN in` (ou `permit host x.x.x.x` puis `deny any`).

## NAT (résumé)
- Concepts : inside-local (privé), inside-global (publique), outside-global/local.  
- Config SNAT) dynamique (PAT/NAPT) :
	```console
	interface g0/0 - ip nat outside
	interface g0/1 - ip nat inside
	access-list 5 permit 172.25.10.0 0.0.0.255
	ip nat inside source list 5 interface g0/0 overload
	```
- Static DNAT (port forwarding) :
  `ip nat inside source static tcp 10.0.0.5 3389 51.234.1.2 3389`
- Vérif : `show ip nat translations`, `show ip nat statistics`.