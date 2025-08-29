# Active Directory — condensé (essentiel) 🧭

## Concepts clés
- Forêt = socle (root) ; contient domaines.  
- Domaine = unité logique (utilisateurs, politiques).  
- OU = organisation pour appliquer GPO/gestion déléguée.  
- Schéma = définition des objets/attributs (unique par forêt).

## Protocoles
- DNS → localise services & DCs.  
- LDAP → interrogation de l'annuaire.  
- Kerberos → auth sécurisée par tickets.

## Objets par défaut
- 🧱 Built-in ➜ groupes locaux issus de la base SAM avant promotion DC
- 💻 Computers ➜ lieu par défaut des objets ordinateurs (postes/serveurs rejoints au domaine)
- ⚙️ System ➜ contient les données critiques du fonctionnement AD (peu manipulé, très sensible)
- 👥 Users ➜ stockage par défaut des utilisateurs/groupes (à ne pas utiliser tel quel en production)

## Groupes
- Type : Sécurité (SID utilisable en ACL) vs Distribution (Messagerie : listes de diffusion).  
- Portée :
	- Globale (GG) ➜ contient : utilisateurs / ordis / groupes globaux du même domaine
	- Domaine local (DL) ➜ utilisé pour accorder des droits sur une ressource du domaine
	- Universelle (GU) ➜ traverse plusieurs domaines, plus flexible mais plus gourmande
	- Locale (LG) ➜ uniquement valable sur la machine locale (hors domaine AD)

## AGDLP (pattern recommandé)
- **A**ccounts → **G**lobal Groups → **D**omain **L**ocal Groups → **P**ermissions (ACL).
- Ex : créer DL_nomressource_CT / _Mod / _Lec / _Refus et affecter ACL sur DL puis inscrire les GG dans les DL.

## Profils & redirection
- Profil stockés par défaut dans `%SystemDrive%\Utilisateurs\UserName%`
- Profils itinérants (roaming) = tout le profil → long/fragile.  
	- Téléchargé à la connexion, pour charger l’environnement habituel
	- Synchronisé à la déconnexion, pour enregistrer les modifs
- Redirection dossiers ciblés = Documents, Desktop, etc. → plus léger et performant.
	- 📁 Vers un dossier partagé commun (ex : \\\srv\dossiers)
	- 🏠 Vers le dossier personnel (répertoire d'accueil) avec sous-dossier %USERNAME%

## FSMO (5 rôles)
- 🏰 Niveau forêt (1 seul DC par rôle, dans toute la forêt)
	- Maître de schéma 🧬 : modifie la structure des objets AD (ajout attributs, nouveaux types…)
	- Maître de noms de domaine 🌐 : gère les ajouts/suppressions de domaines dans la forêt
- 🏠 Niveau domaine (1 seul DC par rôle, par domaine)
	- Maître RID 🆔 : distribue les identifiants uniques (SID) pour chaque objet
	- Maître d’infrastructure 🔁 : assure la correspondance entre objets inter-domaines (ex. : groupes avec membres externes)
	- Émulateur PDC 🕘 : compatibilité avec anciens serveurs NT, priorité pour l’authentification, synchronisation de temps, changements de mot de passe


## Sites & réplication
- Site = regroupement géographique (optimise réplication).  
- Réplication AD = configurable selon liens site/site.

## Import / bulk
- PowerShell `Get-ADUser` / `New-ADUser` + CSV import (le plus flexible).  
- Outils legacy : `csvde` (export/import limité), `ldifde` (plus complet).
