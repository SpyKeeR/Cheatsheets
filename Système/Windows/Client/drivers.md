# Pilotes & RDP — résumé rapide 🖥️🔧

## RDP
- Lancer client : `mstsc /v:NomDuPC` → connexion.
- Admin session : `mstsc /admin`.

## Structure pilote
- `.inf` → définition texte du pilote.
- `.sys` → binaire du pilote.
- `.cat` → signature numérique.

## Emplacements pilotes
- `C:\Windows\INF` → .inf
- `C:\Windows\System32\drivers` → .sys
- `C:\Windows\System32\DriverStore` → magasin pilotes