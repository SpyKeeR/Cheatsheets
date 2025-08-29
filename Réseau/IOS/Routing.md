# RIP & Routage — condensé ⚡

## Administrative Distance

- Interface connectée : 0,
- Route statique : 1,
- EIGRP (Interne) : 90,
- OSPF : 110,
- RIP : 120


## Route type Codes

- C : Connectée directement,
- L : Interface locale dans le réseau,
- S : Statique,
- O : OSPF,
- R : RIP,
- D : EIGRP,
- * : Candidat pour route par défaut

## RIP (rapide)
- Variantes : RIPv1 (classful), RIPv2 (classless), RIPng (IPv6).  
- Type : IGP. AD = 120. Métrique = x hops. Max = 15 (≥16 = unreachable).  
- Transport : UDP (17). Ports : 520 (IPv4), 521 (IPv6).  
- Updates : toutes les 30s. RIPv1 = broadcast 255.255.255.255 ; RIPv2 = multicast 224.0.0.9.  
- Config basique :
  ```console
  router rip
  version 2
  network 10.0.0.0
  network 192.168.1.0
  ```

## Routage Inter-VLAN (3 méthodes)
- Router-on-a-Stick (ROAS) : 1 interface physique → sous-interfaces dot1Q.
	```console
	interface Gig0/0.10
    encapsulation dot1Q 10
    ip address 10.10.10.1 255.255.255.0
	```
- Switch L3 : créer SVIs :
	```console
	interface Vlan10
	ip address 10.10.10.1 255.255.255.0
	ip routing
	```
- Routeur 3 interfaces : 1 interface physique par VLAN (physique dédiée).

## Vérifs & diag rapides
- `show ip route` → table (sources: C, L, S, R, O, D, * = candidate default).  
- `show ip interface brief` → état IP.  
- `show interfaces` → détails phys/logic.  
- `show interfaces trunk` → trunks & VLANs.  
- Check-list inter-VLAN : VLANs créés, trunk actif, SVI/sub-interfaces up, gateway par VLAN, pings inter-VLAN.

## Route statique
- Simple : `ip route <réseau> <mask> <next-hop|interface>`.  
- Default : `ip route 0.0.0.0 0.0.0.0 <next-hop|if>`.  
- Sauvegarder : `copy running-config startup-config`.

## Comportement d’un routeur (packet flow)
- Désencapsulation L2 → accéder au paquet IP.  
- TTL-- (évite boucle).  
- Lookup table → choisir next-hop/interface.  
- Réencapsulation L2 → forward.

## Lecture table route (champs essentiels)
- Source : comment la route a été apprise (C, S, R, L...)
- Destination : réseau visé + préfixe (ex : 192.168.10.0/24)
- Distance administrative : fiabilité de la source
- Métrique : coût du chemin
- Next-hop : IP du routeur suivant
- Horodatage : temps écoulé depuis découverte (utilisé en dynamique)
- Interface de sortie : port utilisé pour envoyer le paquet