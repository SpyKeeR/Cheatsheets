# 🧭 Subnetting & CIDR — Aide-mémoire

## 📐 Concepts Fondamentaux

### Structure Adresse IPv4
```
192.168.1.100/24
├── Partie Réseau : 192.168.1 (24 premiers bits)
├── Partie Hôte : 100 (8 derniers bits)
└── Masque : 255.255.255.0 (/24)
```

### Formules Essentielles
| Calcul | Formule | Exemple /26 |
|--------|---------|-------------|
| **Sous-réseaux** | `2^n` (n = bits empruntés) | `2^2 = 4` sous-réseaux |
| **Hôtes/sous-réseau** | `2^h - 2` (h = bits hôtes) | `2^6 - 2 = 62` hôtes |
| **Adresses totales** | `2^h` | `2^6 = 64` adresses |

## 🔢 Correspondances CIDR

### Table de Conversion Rapide
| CIDR | Masque décimal | Masque binaire (dernier octet) | Hôtes | Sous-réseaux (/24) |
|------|----------------|--------------------------------|-------|-------------------|
| `/24` | `255.255.255.0` | `00000000` | 254 | 1 |
| `/25` | `255.255.255.128` | `10000000` | 126 | 2 |
| `/26` | `255.255.255.192` | `11000000` | 62 | 4 |
| `/27` | `255.255.255.224` | `11100000` | 30 | 8 |
| `/28` | `255.255.255.240` | `11110000` | 14 | 16 |
| `/29` | `255.255.255.248` | `11111000` | 6 | 32 |
| `/30` | `255.255.255.252` | `11111100` | 2 | 64 |

### Classes & CIDR par Défaut
| Classe | Plage | CIDR défaut | Masque | Hôtes max |
|--------|-------|-------------|--------|-----------|
| **A** | 1.0.0.0 - 126.255.255.255 | `/8` | 255.0.0.0 | 16 777 214 |
| **B** | 128.0.0.0 - 191.255.255.255 | `/16` | 255.255.0.0 | 65 534 |
| **C** | 192.0.0.0 - 223.255.255.255 | `/24` | 255.255.255.0 | 254 |

## ⚡ Méthode du Nombre Magique

### Principe
1. **Identifier** l'octet non-255 dans le masque
2. **Calculer** : Nombre magique = `256 - valeur_octet`
3. **Déterminer** les plages par multiples du nombre magique

### Exemples Pratiques
```
Exemple 1 : 192.168.1.50/26
├── Masque : 255.255.255.192
├── Octet clé : 192 → Nombre magique : 256 - 192 = 64
├── Plages : 0, 64, 128, 192
├── 50 est dans 0-63 → Réseau : 192.168.1.0/26
└── Broadcast : 192.168.1.63

Exemple 2 : 172.16.45.100/20
├── Masque : 255.255.240.0
├── Octet clé : 240 → Nombre magique : 256 - 240 = 16
├── Plages (3e octet) : 0, 16, 32, 48, 64...
├── 45 est dans 32-47 → Réseau : 172.16.32.0/20
└── Broadcast : 172.16.47.255
```

## 🏗️ Découpage de Réseaux (VLSM)

### Approche Optimisée
1. **Lister** besoins par ordre décroissant
2. **Calculer** CIDR nécessaire : `log₂(hôtes + 2)`
3. **Attribuer** séquentiellement sans chevauchement

### Exemple Complet : 192.168.1.0/24
```
Besoins :
├── Service A : 62 hôtes → /26 (64 adresses)
├── Service B : 30 hôtes → /27 (32 adresses) × 3
└── Service C : 14 hôtes → /28 (16 adresses) × 4

Attribution :
├── 192.168.1.0/26    (0-63)     → Service A
├── 192.168.1.64/27   (64-95)    → Service B1  
├── 192.168.1.96/27   (96-127)   → Service B2
├── 192.168.1.128/27  (128-159)  → Service B3
├── 192.168.1.160/28  (160-175)  → Service C1
├── 192.168.1.176/28  (176-191)  → Service C2
├── 192.168.1.192/28  (192-207)  → Service C3
├── 192.168.1.208/28  (208-223)  → Service C4
└── 192.168.1.224/27  (224-255)  → Disponible
```

## 🌐 Réseaux Privés (RFC 1918)

### Plages Autorisées
| Classe | Plage complète | CIDR | Utilisation |
|--------|----------------|------|-------------|
| **A** | `10.0.0.0 - 10.255.255.255` | `10.0.0.0/8` | Grandes entreprises |
| **B** | `172.16.0.0 - 172.31.255.255` | `172.16.0.0/12` | Moyennes entreprises |
| **C** | `192.168.0.0 - 192.168.255.255` | `192.168.0.0/16` | Petites entreprises/domicile |

### Autres Plages Spéciales
- **Loopback** : `127.0.0.0/8` (127.0.0.1)
- **Link-Local** : `169.254.0.0/16` (APIPA/DHCP échec)
- **Multicast** : `224.0.0.0/4` (224.0.0.0 - 239.255.255.255)
- **Test/Documentation** : `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`

## 🔗 Supernetting (Agrégation)

### Principe & Conditions
- **Objectif** : Regrouper plusieurs réseaux contigus
- **Condition** : Premier réseau doit être multiple de la taille agrégée
- **Calcul** : Masque réduit pour englober tous les réseaux

### Exemples d'Agrégation
```
Cas 1 : Regrouper 4 réseaux /24 consécutifs
├── 192.168.20.0/24, 192.168.21.0/24, 192.168.22.0/24, 192.168.23.0/24
├── Vérification : 20 ÷ 4 = 5 (multiple) ✓
└── Agrégation → 192.168.20.0/22

Cas 2 : Problème de non-alignement
├── 192.168.21.0/24, 192.168.22.0/24, 192.168.23.0/24, 192.168.24.0/24
├── Vérification : 21 ÷ 4 = 5.25 (non-multiple) ✗
└── Solutions : 
    ├── Deux /23 : 192.168.21.0/23 + 192.168.23.0/23 (non-contigu)
    └── Un /21 : 192.168.16.0/21 (inclut adresses inutilisées)
```

## 🔍 Identification Réseau/Broadcast

### Méthode Binaire (Précise)
```
Exemple : 172.16.85.200/19
1. Conversion binaire 3e octet : 85 = 01010101
2. Masque /19 : 11111111.11111111.11100000.00000000
3. AND logique : 01010101 & 11100000 = 01000000 = 64
4. Réseau : 172.16.64.0/19
5. Broadcast : 172.16.95.255 (bits hôte à 1)
```

### Méthode Rapide (Nombre Magique)
```
1. Masque /19 → 255.255.224.0
2. Nombre magique : 256 - 224 = 32
3. Plages 3e octet : 0, 32, 64, 96, 128...
4. 85 ∈ [64,96[ → Réseau 172.16.64.0/19
```

## 📊 Outils de Calcul

### Commandes Utiles
```bash
# Linux/macOS - Calcul binaire
echo "obase=2; 192" | bc                    # Décimal → Binaire
echo $((2#11000000))                        # Binaire → Décimal

# ipcalc (si installé)
ipcalc 192.168.1.100/26                    # Informations complètes

# Windows PowerShell
[convert]::ToString(192, 2)                 # Décimal → Binaire
[convert]::ToInt32("11000000", 2)           # Binaire → Décimal
```

### Vérifications Manuelles
```
Vérifier appartenance réseau :
1. IP & Masque = Adresse réseau ?
2. Exemple : 192.168.1.100 & 255.255.255.192
   ├── 100 (01100100) & 192 (11000000) = 64 (01000000)
   └── Réseau : 192.168.1.64 ✓

Calculer nombre hôtes disponibles :
1. Compter bits à 0 dans le masque
2. Formule : 2^n - 2 (réseau + broadcast exclus)
3. /26 → 6 bits hôte → 2^6 - 2 = 62 hôtes
```

## 🎯 Cas Pratiques Courants

### Liaison Point-à-Point (/30)
```
Connexion routeur-à-routeur :
├── Réseau : 192.168.100.0/30
├── Passerelle 1 : 192.168.100.1
├── Passerelle 2 : 192.168.100.2  
└── Broadcast : 192.168.100.3
```

### DMZ (/28)
```
Zone démilitarisée (14 hôtes) :
├── Réseau : 192.168.10.0/28
├── Passerelle : 192.168.10.1
├── Serveur Web : 192.168.10.2
├── Serveur Mail : 192.168.10.3
├── ... (10 adresses disponibles)
└── Broadcast : 192.168.10.15
```

### VLAN Utilisateurs (/24)
```
Réseau utilisateurs standard :
├── Réseau : 192.168.20.0/24
├── Passerelle : 192.168.20.1
├── DHCP Pool : 192.168.20.10-200
├── Serveurs : 192.168.20.240-250
└── Broadcast : 192.168.20.255
```

## 💡 Astuces & Bonnes Pratiques

### Mémorisation Rapide
- **Puissances de 2** : 1, 2, 4, 8, 16, 32, 64, 128, 256
- **Masques courants** : /24=254, /25=126, /26=62, /27=30, /28=14, /30=2
- **Classes par premier octet** : A(1-126), B(128-191), C(192-223)

### Planification Réseau
- ✅ **Prévoir croissance** : Dimensionner avec marge
- ✅ **Documenter** : Plan d'adressage centralisé  
- ✅ **Standardiser** : Conventions nommage cohérentes
- ✅ **Sécuriser** : Séparation par fonction (DMZ, LAN, MGMT)

### Erreurs à Éviter
- ❌ **Chevauchement** de plages IP
- ❌ **Gaspillage** d'adresses (surdimensionnement)
- ❌ **Oubli** des adresses réseau/broadcast
- ❌ **Négligence** de la croissance future
- ❌ **Mauvaise documentation** des changements