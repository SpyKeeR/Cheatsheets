## ACL & DACL (essentiel)
- DACL = liste ACE (Access Control Entries) → qui peut quoi.
- Refus explicite > Autorisation explicite > Hérité.
- Héritage : utile pour simplifier ; attention aux copies entre volumes (perms changent).
- Éviter `Deny` inutilement.
- Autorisations de base : Lecture, Liste de dossier, Lecture/Exécution, Écriture, Modification, Contrôle total
- Autorisations avancées : Création de fichiers/dossiers, Suppression, Lecture des autorisations, Appropriation, Écriture d’attributs étendus…
- Déplacement sur même volume conserve permissions ; copie entre volumes hérite du dossier cible.
