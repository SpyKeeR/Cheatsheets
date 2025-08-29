# Méta-caractères & Regex

## Métacaractères
- `*` zéro ou plusieurs caractères, `?` un seul, `[]` plage, `{}` listes.
- 🚫 [!...] ou [^...] — Négation
- extglob (`shopt -s extglob`) :
	- `?(...)` → 0 ou 1 fois (optionnel)
	- `*(...)` → 0, 1 ou plusieurs fois
	- `+(...)` → 1 fois ou plus
	- `@(...)` → exactement 1 fois
	- `!(...)` → l'expression ne doit pas apparaître
```text
ls *+([eE])* # Fichiers contenant au moins un 'e' ou 'E'
ls !(*.txt) # Tous les fichiers SAUF ceux en .txt
```

- Doubles quotes " → interprètent les variables
- Simples quotes ' → affichent tel quel (pas d’interprétation)
- Antislash \\ → Échappement → Utilisé pour désactiver un méta-caractère
- `$()` → Appel du résultat d'une commande
- `{choix1,choix2}` → Les accolades permettent de faire plusieurs recherches ou opérations d’un coup.

## **🧩 REG-EX**

Ces symboles permettent de créer des motifs souples et précis :

- . : N’importe quel caractère (sauf retour à la ligne)
- ^ : Début de ligne
- $ : Fin de ligne
- \* : Zéro ou plusieurs fois le caractère précédent
- \+ : Une ou plusieurs fois le caractère précédent
- \? : Zéro ou une fois le caractère précédent
- \[] : Classe de caractères (ex : [abc] → a, b ou c)
- | : OU logique (ex : a|b → a ou b)

### **🔢 Quantificateurs avancés**

Ils permettent de définir **le nombre de répétitions** exactes ou approximatives :

- {n} : Exactement **n fois**
- {n,} : **Au moins n fois**
- {n,m} : **Entre n et m fois**

#### **✅ Exemples concrets :**

- .* : Zero ou plusieurs caractères quelconques
- \\+ : Un ou plusieurs caractères (comme +, mais échappé en Bash)
- \\? : Aucun ou un seul caractère
- ab\\{3,\\}c : a, suivi de **au moins 3 fois b**, suivi de c
- ab\\{5\\}c : a, **exactement 5 fois b**, puis c
- ^\[15]\\* : Début de ligne avec **0 ou plus de 1 ou 5**
- ^\[a-z]\\+ : Début de ligne avec **une ou plusieurs lettres minuscules**

### **🔁 Groupes et sous-expressions**

- \\( … \\) : Définit un **groupe** capturé
- \\1, \\2, etc. : Réutilise les groupes précédents

**💡 Regex utiles à retenir**

- ^ → Ancre le début de la ligne
- $ → Ancre la fin de la ligne
- ^$ → Ligne vide
- ^.*$ → Ligne avec **du contenu**


### **🏷️ Classes de caractères**

- \[\[:alpha:]] : Lettres (a-z, A-Z)
- \[\[:alnum:]] : Lettres et chiffres
- \[\[:upper:]] : Lettres majuscules
- \\d : Chiffre [0-9]
- \\D : Tout sauf chiffre
- \\s : Séparateur blanc (espace, tab, etc.)
- \\S : Tout sauf séparateur
- \\w : Alphanumérique + underscore [a-zA-Z0-9_]
- \\W : Tout sauf caractère alphanumérique
- \\n : Nouvelle ligne
- \\r : Retour chariot
- \\t : Tabulation



### **🔤 Substitution de casse (utilisable avec sed)**

- \\u : Met en **majuscule** la **lettre suivante**
- \\U : Met en **majuscule** tout ce qui suit jusqu’à \E
- \\l : Met en **minuscule** la **lettre suivante**
- \\L : Met en **minuscule** tout ce qui suit jusqu’à \E
- \\E : Marque la fin d’un effet (\U, \L, etc.)