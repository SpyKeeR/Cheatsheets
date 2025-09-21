# Virtualisation — Réseau & Stockage (ultra-condensé)

## Modes réseau VM
- Bridge (VMnet0 / External) → VM sur LAN physique.  
- NAT (VMnet8) → VM via IP de l’hôte.  
- Host-Only (VMnet1) → VM ↔ hôte (pas d’Internet).  
- LAN Segment → réseau isolé VM↔VM (pas d’hôte).  
- Hyper-V : Private = VM↔VM, Internal = VM↔VM↔Hôte, External = LAN physique.

## VLAN & trunking
- Port group VLAN ID → segmentation logique.  
- Uplink (pNIC/vmnic) doit être trunk sur switch physique si multi-VLANs.  
- Port physical : `access` (1 VLAN) / `trunk` (802.1Q).  
- Tagging possible côté guest (vNIC) ou hyperviseur.

## Terminologie réseau
- pNIC / vmnic = interface physique.  
- vNIC = interface virtuelle d’une VM.  
- vmkNIC = VMkernel (vMotion, NFS, iSCSI, management).  
- Port group (VM Network) = port virtuel pour VM.  
- VMkernel port = IP propre pour services hyperviseur.

## Teaming / load balancing
- Modes : débit cumulé (active-active), tolérance (active-passive).  
- Algorithmes : par VM / par MAC / par paquet / ordre.  
- Impact : redondance vs agrégation de bande passante.

## VLAN / Port groups
- Port group + VLAN tag → isolation : VM ne voit que son VLAN.  
- VMkernel port group = réservé aux services ESXi (vCenter mgmt, vMotion, NFS/iSCSI, FT). Chaque VMkernel = IP dédiée + service associé.

## vSwitch / VSS vs VDS
- VSS (Standard) : config locale par hôte, simple.  
- VDS (Distribué) : géré via vCenter, cohérence infra, mobile-friendly (vMotion).  
- À la création VDS → choisir hôtes & vmnics associés (existe via vCenter, pas physiquement local).

## iSCSI / SAN / NAS (résumé)
- iSCSI = bloc sur TCP/IP : Initiator (hôte) ↔ Target (stockage).  
- SAN (FC / FCoE / iSCSI) = mode bloc, réseau dédié, hautes perf.  
- NAS = mode fichier (NFS/SMB) pour partages, moins perf pour IO intensif.  
- Reco iSCSI : réseau dédié, redondance, Jumbo Frames MTU=9000 si infra supporte (set MTU sur switch/hôte).

## Adaptateurs stockage
- Physique (HBA FC, HBA iSCSI, CNA) → meilleures perfs, faible CPU.  
- Virtuel (software iSCSI) → simple mais plus CPU.  
- Choisir selon perf & budget.

## Datastore types
- VMFS (VMware FS) → accès bloc partagé, performant (VMFS5/6).  
- RDM (Raw Device Mapping) → accès direct LUN (cas spéciaux).  
- NFS → datastore en mode fichier (ISO, templates, faible perf IO).  
- VMDK = fichier disque VM (jusqu’à limites VMFS); Thin vs Thick (provisionnement thin = économie, risque overcommit; thick = perf et réservation).

## Provisioning disque
- Thick = réserve tout l’espace (performances constantes).  
- Thin = allocation dynamique (rapide, économie, risque de surallocation).

## Templates & formats
- OVF/OVA = standard inter-plates-formes (figé, portable).  
- VMTX = template vSphere (modifiable, clonable, idéal prod).

## vCSA & ports essentiels
- Port 5480 → VAMI (appliance mgmt: IP/DNS/NTP, updates, sauvegarde).  
- Port 443 → vSphere Client (vCenter GUI / gestion infra).

## Auth & AD/SSO
- vCenter → intégration AD + SSO pour gestion centralisée des droits (reco pour infra pro).

## vMotion / Storage vMotion (prérequis & risques)
- vMotion → réseau vMotion dédié (VMkernel), IP dédiée, low-latency, accès stockage partagé simultané sur hôtes.  
- Storage vMotion → déplacement live du disque entre datastores.  
- À vérifier avant migration : aucun ISO monté, port group existant sur cible, compat CPU (Caveat), ressources dispo.

## Bonnes pratiques rapides
- Dédier réseaux iSCSI / vMotion / mgmt via VMkernel séparés.  
- Activer Jumbo Frames uniquement si bout en bout (hosts + switches).  
- Supprimer périphériques virtuels non essentiels avant migration (ISO, disquette).  
- Valider existence du même port group/VLAN sur hôte de destination.  
- Documenter MTU, teaming & algorithmes pour cohérence infra.

## Points d’attention
- Perf = choix adaptateur (physique > virtuel) + SAN vs NAS.  
- VDS requis pour mobilité avancée (vMotion/DRS optimaux).  
- Thin provisioning = économie mais surveiller datastore usage.
- Toujours tester résilience réseau (failover) et perf (throughput) selon besoin.