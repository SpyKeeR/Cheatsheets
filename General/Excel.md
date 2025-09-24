# 📊 Excel — Aide-mémoire

## 🧮 Fonctions Mathématiques & Statistiques

### Calculs de Base
```excel
=SOMME(A1:A10)           # Additionne une plage
=MOYENNE(A1:A10)         # Moyenne arithmétique
=MIN(A1:A10) / =MAX(A1:A10)  # Valeurs minimale/maximale
=MEDIANE(A1:A10)         # Valeur médiane
=MODE(A1:A10)            # Valeur la plus fréquente
=ECARTYPE(A1:A10)        # Écart-type
=NB(A1:A10)              # Nombre de cellules numériques
=NBVAL(A1:A10)           # Nombre de cellules non vides
```

### Calculs Avancés
```excel
=SOMME.SI(A1:A10,">100")        # Somme conditionnelle
=MOYENNE.SI(A1:A10,"<>0")       # Moyenne sans zéros
=NB.SI(A1:A10,">=50")           # Compter si condition
=SOMME.SI.ENS(B1:B10,A1:A10,">100",C1:C10,"Vente")  # Multi-critères
=ARRONDI(A1,2)                  # Arrondir à 2 décimales
=ENT(A1)                        # Partie entière
=ABS(A1)                        # Valeur absolue
```

## 🔍 Fonctions de Recherche & Référence

### RECHERCHEV & RECHERCHEH
```excel
=RECHERCHEV(val, A1:C10, 3, FAUX)    # Recherche verticale exacte
=RECHERCHEV(val, A1:C10, 3, VRAI)    # Recherche approchée (trié)
=RECHERCHEH(val, A1:C3, 2, FAUX)     # Recherche horizontale
=INDEX(A1:A10, EQUIV(val, B1:B10, 0)) # Alternative plus flexible
```

### Nouvelles Fonctions (Office 365)
```excel
=RECHERCHEX(val, A1:A10, B1:B10)     # RECHERCHEV améliorée
=FILTRE(A1:B10, A1:A10>100)          # Filtrer données dynamiquement
=UNIQUE(A1:A10)                      # Valeurs uniques
=TRIER(A1:B10, 1)                    # Trier par colonne 1
```

### Gestion Erreurs de Recherche
```excel
=SIERREUR(RECHERCHEV(...), "Non trouvé")     # Gestion erreur #N/A
=SIVIDE(A1, "Vide", A1)                      # Gestion cellules vides
=ESTNA(RECHERCHEV(...))                      # Tester si #N/A
```

## 🧠 Fonctions Logiques

### Conditions de Base
```excel
=SI(A1>10, "Grand", "Petit")               # Condition simple
=ET(A1>10, B1<20)                          # Toutes conditions vraies
=OU(A1>100, B1="OK")                       # Au moins une vraie
=NON(A1=0)                                 # Négation
```

### Conditions Multiples
```excel
=SI(A1>100, "Élevé", SI(A1>50, "Moyen", "Bas"))     # SI imbriqués
=SI(ET(A1>10, B1="Actif"), "Valide", "Invalide")    # SI + ET
=SI(OU(A1="", A1=0), "Vide", A1)                    # Vérifier vide/zéro
```

### Nouvelles Fonctions Logiques
```excel
=SI.CONDITIONS(A1>100,"Élevé",A1>50,"Moyen",VRAI,"Bas")  # Remplace SI imbriqués
=SI.MULTIPLE(A1,1,"Un",2,"Deux",3,"Trois")               # Switch/case
```

## 📝 Fonctions Texte

### Extraction & Manipulation
```excel
=GAUCHE(A1, 3)              # 3 premiers caractères
=DROITE(A1, 2)              # 2 derniers caractères
=STXT(A1, 2, 5)             # 5 caractères à partir position 2
=NBCAR(A1)                  # Nombre de caractères
=SUPPRESPACE(A1)            # Supprimer espaces superflus
=MAJUSCULE(A1)              # Tout en majuscules
=MINUSCULE(A1)              # Tout en minuscules
=NOMPROPRE(A1)              # Première lettre majuscule
```

### Recherche dans Texte
```excel
=TROUVE("mot", A1)          # Position du mot (sensible casse)
=CHERCHE("mot", A1)         # Position du mot (insensible casse)
=SUBSTITUE(A1, "ancien", "nouveau")  # Remplacer texte
=CONCATENER(A1, " ", B1)    # Joindre textes
=A1&" "&B1                  # Alternative concaténation
```

### Fonctions Avancées
```excel
=JOINDRE.TEXTE(" ", VRAI, A1:A3)      # Joindre avec séparateur
=DIVISER.TEXTE(A1, " ")               # Diviser en colonnes
=EXACT(A1, B1)                        # Comparaison exacte
=CODE(A1)                             # Code ASCII premier caractère
```

## 📅 Fonctions Date & Heure

### Dates Courantes
```excel
=AUJOURDHUI()               # Date du jour (se met à jour)
=MAINTENANT()               # Date + heure actuelles
=DATE(2024, 1, 15)          # Créer date spécifique
=TEMPS(14, 30, 0)           # Créer heure (14:30:00)
=JOUR(A1) / =MOIS(A1) / =ANNEE(A1)  # Extraire composants
```

### Calculs de Dates
```excel
=DATEDIF(A1, B1, "d")       # Différence en jours
=DATEDIF(A1, B1, "m")       # Différence en mois
=DATEDIF(A1, B1, "y")       # Différence en années
=JOURSEM(A1)                # Numéro jour semaine (1=dimanche)
=FIN.MOIS(A1, 0)            # Dernier jour du mois
=JOURS.OUVRES(A1, B1)       # Jours ouvrés entre deux dates
```

### Formatage Dates
```excel
=TEXTE(A1, "jjjj dd/mm/aaaa")     # Format personnalisé
=TEXTE(MAINTENANT(), "hh:mm")      # Heure seule
=NO.SEMAINE(A1)                   # Numéro de semaine
```

## 💰 Fonctions Financières

### Calculs de Base
```excel
=VA(taux, npm, vpm)              # Valeur actuelle
=VC(taux, npm, vpm, va)          # Valeur future
=VPM(taux, npm, va)              # Paiement périodique
=TAUX(npm, vpm, va, vc)          # Taux d'intérêt
=NPM(taux, vpm, va)              # Nombre de périodes
```

### Amortissement
```excel
=AMORLIN(coût, résid, durée)     # Amortissement linéaire
=PRINPER(taux, per, npm, va)     # Capital remboursé période donnée
=INTPER(taux, per, npm, va)      # Intérêts période donnée
```

## 📊 Tableaux Croisés Dynamiques (Concepts)

### Éléments Clés
- **Source** : Données structurées (en-têtes, pas de lignes vides)
- **Lignes** : Dimensions d'analyse (catégories)
- **Colonnes** : Sous-dimensions (dates, régions)
- **Valeurs** : Métriques calculées (sommes, moyennes)
- **Filtres** : Critères de sélection

### Fonctions TCD
```excel
=LIREDONNEESTABCROISDYN("Ventes", A1)     # Lire valeur TCD
=GETPIVOTDATA("Ventes", A1, "Mois", "Jan") # Extraire donnée spécifique
```

## 🔧 Raccourcis Clavier Essentiels

### Navigation
- `Ctrl + Flèches` : Déplacement rapide
- `Ctrl + Shift + Flèches` : Sélection rapide
- `Ctrl + G` : Atteindre cellule
- `F5` : Boîte de dialogue Atteindre

### Édition
- `F2` : Éditer cellule
- `Ctrl + D` : Recopier vers le bas
- `Ctrl + R` : Recopier vers la droite
- `Ctrl + ;` : Insérer date du jour
- `Ctrl + Shift + ;` : Insérer heure

### Format
- `Ctrl + 1` : Format de cellule
- `Ctrl + Shift + $` : Format monétaire
- `Ctrl + Shift + %` : Format pourcentage
- `Alt + Entrée` : Retour ligne dans cellule

## 🎯 Références de Cellules

### Types de Références
```excel
A1          # Relative (change en recopiant)
$A$1        # Absolue (ne change jamais)
$A1         # Mixte (colonne fixe, ligne relative)
A$1         # Mixte (ligne fixe, colonne relative)
```

### Plages Nommées
```excel
=SOMME(MesVentes)           # Utiliser nom de plage
=DECALER(A1, 0, 0, 5, 1)   # Référence dynamique
=INDIRECT("A"&LIGNE())      # Référence construite
```

## 📋 Formats & Validation

### Formats Personnalisés
```
0 000           # Milliers avec espace (1 234)
0,00 €          # Monétaire français
[>1000]0,0"K";0 # Milliers en K si >1000
"Texte: "@      # Préfixe texte
```

### Validation de Données
- **Liste** : Créer menu déroulant
- **Nombre** : Limiter plage numérique
- **Date** : Contrôler période
- **Longueur** : Limiter caractères
- **Personnalisé** : Formule de validation

## ⚠️ Erreurs Courantes & Solutions

### Types d'Erreurs
| Erreur | Cause | Solution |
|--------|-------|----------|
| `#DIV/0!` | Division par zéro | `=SI(B1=0,"",A1/B1)` |
| `#N/A` | Valeur non trouvée | `=SIERREUR(RECHERCHEV(...), "")` |
| `#REF!` | Référence invalide | Vérifier références cellules |
| `#VALEUR!` | Type données incorrect | Vérifier format données |
| `#NOM?` | Fonction/nom inconnu | Vérifier orthographe |

### Bonnes Pratiques
- ✅ **Nommer les plages** importantes
- ✅ **Documenter** formules complexes (commentaires)
- ✅ **Éviter** références circulaires
- ✅ **Utiliser** validation données pour saisie
- ✅ **Séparer** données et calculs
- ✅ **Verrouiller** cellules de formules importantes

## 🔍 Analyse de Données

### Outils Analyse Rapide
- **Ctrl + Q** : Analyse rapide (graphiques, totaux, formats)
- **Alt + D + P** : Assistant tableau croisé dynamique
- **Données > Analyse de simulation** : Tables de données, valeur cible

### Fonctions Avancées
```excel
=FREQUENCE(données, classes)        # Distribution fréquences
=CENTILE(A1:A100, 0.9)             # 90e centile
=RANG(A1, A1:A100, 0)              # Rang décroissant
=CORRELATION(A1:A10, B1:B10)       # Coefficient corrélation
```

---
**💡 Astuce** : `F1` pour aide contextuelle, `Alt + =` pour somme automatique, `Ctrl + Z` pour annuler !