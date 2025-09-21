# Virtualisation — Résumé ultra-condensé

## Formats & fichiers VM
- `.vmx` → config VM (texte).  
- `.vmdk` → disque virtuel (dynamic).  
- OVF = ensemble, OVA = archive unique.  

## Fichiers Hyper-V
- Stockage Hyper-V : `C:\ProgramData\Microsoft\Windows\Hyper-V\`.  
- Disques = `.vhdx`; configs & états = `.vmcx`, `.vmrs`, etc.

## Hyperviseurs
- Type 1 (bare-metal) → ESXi, Hyper-V Server.  
- Type 2 (hébergé) → Workstation, VirtualBox.  
- Choix selon besoins (scale, intégration, coûts).

## Migration & stockage
- vMotion → migration live (CPU+RAM).  
- Storage vMotion → migration stockage sans downtime.  
- Nécessite stockage partagé et config réseau vmkernel dédiée.

## Fonctions d’orchestration (VMware)
- DRS = équilibre automatique ressources.  
- Storage DRS = équilibre datastores.  
- HA = restart automatique VM lors panne hôte.  
- FT = Fault Tolerance (zero downtime pour VM supportées).  
- DPM = power management des hôtes.

## Para-virtualisation vs native
- Avant VT-x/SVM → binary translation / para-virtualisation (OS modifié).  
- Avec VT-x/SVM → virtualisation matérielle (meilleure perf).  
- Tools/Additions/Integration Services = drivers invités pour IO optimisé.

## Mémoire & SLAT
- SLAT évite double réservation mémoire (Intel EPT / AMD NPT/RVI).  
- Important pour densité mémoire et performances.

## Import / Export VM
- Import sans duplication = déplacement (un seul import possible).  
- Import avec duplication = copie (nouvelles MAC possibles).  
- Attention aux conflits MAC et licences.

## vCenter / compatibilité
- VCSA = vCenter Appliance (deployment OVA).  
- Maintenir environnement homogène(versionning) pour fonctions avancées.

## Licences & coûts (notes)
- Licences modernes = par cœur + abonnements (bundles, vSphere+).  
- ESXi free = limitations (pas de vCenter, API limitées, vCPU max selon version).

## Pratiques rapides
- Désactiver DHCP hyperviseur si DHCP infra existant.  
- Tester trunking uplink avant déploiement VLAN.  
- Monitorer compatibilité vMotion/FT avant upgrades.  
- Raccourcis VM Workstation/ESXi : libération souris (Ctrl+Alt), Ctrl+Alt+Insert (VMware)/Ctrl+Alt+End(HyperV) pour Ctrl+Alt+Del.
