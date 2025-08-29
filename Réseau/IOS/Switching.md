## Duplex / speed
- half-duplex = → CSMA/CD actif
- full-duplex = → CSMA/CD désactivé
- `duplex {auto | full | half}`
- `speed {10 | 100 | 1000 | auto}`



## SVI (Switch Virtual Interface) — remote management
- `configure terminal`  
- `interface vlan 1`  
- `ip address 192.168.1.20 255.255.255.0`  
- `no shutdown`

## MAC learning & forwarding
- Apprentissage MAC : table MAC associe MAC⇄port.  
- Destination connue → envoi unicast sur port correspondant.  
- Inconnue → flood sur tous ports du VLAN.  
- Broadcast/multicast → diffusé à tous les ports (sauf origine).

## Forwarding modes
- Store-and-Forward → stocke complet, vérifie FCS, latence élevée, intégrité OK.  
- Cut-Through → forward dès lecture dst MAC, faible latence, pas de FCS vérif.

## VLAN & trunking (essentiel)
- Créer VLAN : `vlan <id>` → `name <nom>`.  
- Port access : `interface <id>` → `switchport mode access` → `switchport access vlan <id>`.  
- Voice VLAN : `switchport voice vlan <id>` + `mls qos trust cos`.  
- Trunk : `interface <id>` → `switchport mode trunk` → `switchport trunk native vlan <id>` → `switchport trunk allowed vlan <liste>`.  
- Désactiver DTP : `switchport nonegotiate` 

## QoS basique pour voix
- Trust CoS on access port : `mls qos trust cos`.  
- Voice VLAN priorisé via `switchport voice vlan <id>`.

## Bonnes pratiques rapides
- Utiliser `switchport nonegotiate` pour ports fixes.  
- Marquer uplinks trust pour DHCP Snooping & DAI.  
- Activer BPDU Guard sur portfast.  
- Sauvegarder configs souvent : `copy run start` + off-site backup TFTP/SCP.