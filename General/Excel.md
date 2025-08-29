# Excel — Cheatsheet (très condensé) 📊

## Fonctions de base
- `=SOMME(A1:A10)` → additionne une plage.
- `=MOYENNE(A1:A10)` → moyenne.
- `=MIN(A1:A10)` / `=MAX(A1:A10)` → min / max.

## Fonctions logiques
- `=SI(cond, val_vrai, val_faux)` → condition.
- `=ET(...)` → vrai si toutes vraies.
- `=OU(...)` → vrai si au moins une vraie.

## Texte
- `=GAUCHE(A1, n)` → n premiers caractères.
- `=DROITE(A1, n)` → n derniers caractères.
- `=NBCAR(A1)` → nombre de caractères.

## Recherche
- `=RECHERCHEV(val, tableau, col, FAUX)` → recherche verticale exacte.
- `=RECHERCHEH(val, ligne, col, FAUX)` → recherche horizontale.

## Date & heure
- `=AUJOURDHUI()` → date du jour.
- `=MAINTENANT()` → date + heure.
- `=DATEDIF(début, fin, "d")` → différence en jours.

## Imbrications courantes
- `=SI(A1>10, SOMME(A1:A10), "Trop bas")`
- `=SI(RECHERCHEV(10, A1:B10, 2, FAUX)>100, "Grand", "Petit")`