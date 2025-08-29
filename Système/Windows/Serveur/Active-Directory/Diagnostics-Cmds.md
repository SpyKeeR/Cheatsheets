# 🧾 Active Directory – Commandes utiles Diagnostics

## 🔄 Replication (Repadmin)

- `repadmin /showutdvec . dc=domain,dc=com` ➝ Affiche l’état de mise à jour de la réplication (up-to-dateness-vector).
- `repadmin /showobjmeta <NomServeur> "<DN de l’objet>"` ➝ Montre les métadonnées : version des attributs, USN, etc.
- `repadmin /showrepl * file.csv` ➝ Dump l’état de réplication de la majorité du réseau AD.
- `repadmin /showreps` ➝ Liste les partenaires de réplication d’un DC.
- `repadmin /replicate <DC_Destination> <DC_Source> <DN_Domaine_NC>` ➝ Déclenche manuellement une réplication ciblée.

## 👤 Utilisateur & groupes

- `whoami /all` ➝ Donne tous les groupes, privilèges et droits liés au compte courant.

## 🏥 Diagnostic contrôleurs de domaine (Dcdiag)

- `dcdiag /v /e` ➝ Fait un check détaillé de la santé de tous les DCs du domaine.
- `dcdiag /test:dns` ➝ Vérifie les problèmes DNS impactant la réplication.
