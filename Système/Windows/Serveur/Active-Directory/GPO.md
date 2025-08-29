# GPO — résumé rapide & pratiques ✅

## Cible & application
- Liée à Site / Domaine / OU (objets user & computer uniquement).  
- Ordre d’application : Local ← Site ← Domaine ← OU (la dernière appliquée a priorité si conflit).

## Concepts importants
- Bloquer l’héritage = empêche héritage depuis supérieurs (utiliser avec prudence).  
- Enforced (Appliqué) = force la GPO même si héritage bloqué.  
- Filtrage de sécurité = appliquer selon permissions sur la GPO.  
- Filtre WMI = appliquer selon état machine (OS, RAM, etc.).

## Actualisation
- Clients : cycle ~90 ±30 min ; DC : toutes les 5 min.  
- Forcer : `gpupdate /force`.

## Default Policies
- Default Domain Policy (DDP) : s’applique à tous les utilisateurs du domaine, dès la racine
- Default Domain Controller Policy (DDCP) : s’applique aux DC uniquement, pour la sécurité AD

## ADMX & magasin central (Modèles d'administrations)
- ADMX/ADML → templates de policies (C:\Windows\PolicyDefinitions).  
- Magasin central (SYSVOL) : `\\DOMAIN\SYSVOL\DOMAIN\Policies\PolicyDefinitions` → partager ADMX/ADML pour cohérence (Répertoire à créer)

## Bonnes pratiques
- Limiter profondeur des OU/GPO ; documenter.  
- Tester sur OU de Test avant déploiement global.  
- Utiliser commentaires & versioning pour GPO critiques.
- Dépendance OS : les stratégies doivent être compatibles avec les OS cibles
- GPO ≠ stratégie locale : les GPO l’emportent toujours si conflit