# 🔢 Conversions Bases & Unités — Aide-mémoire

## 📊 Bases Numériques

### Correspondances Fondamentales
| Décimal | Binaire | Octal | Hexadécimal |
|---------|---------|--------|-------------|
| 0 | 0000 | 0 | 0 |
| 1 | 0001 | 1 | 1 |
| 8 | 1000 | 10 | 8 |
| 10 | 1010 | 12 | A |
| 15 | 1111 | 17 | F |
| 16 | 10000 | 20 | 10 |
| 255 | 11111111 | 377 | FF |

### Puissances de 2 (Mémorisation)
```
2⁰ = 1       2⁸ = 256
2¹ = 2       2⁹ = 512  
2² = 4       2¹⁰ = 1024 (1K)
2³ = 8       2¹⁶ = 65536 (64K)
2⁴ = 16      2²⁰ = 1048576 (1M)
2⁵ = 32      2³⁰ = 1073741824 (1G)
2⁶ = 64      2⁴⁰ = 1099511627776 (1T)
2⁷ = 128
```

## 🔄 Méthodes de Conversion

### Décimal → Autres Bases
```
Division successive par la base :
Décimal 687 → Hex :
687 ÷ 16 = 42 reste 15 (F)
42 ÷ 16 = 2 reste 10 (A)  
2 ÷ 16 = 0 reste 2
Résultat : 2AF (lire de bas en haut)

Décimal 687 → Binaire par soustraction :
687 - 512 (2⁹) = 175 → bit 9 = 1
175 - 128 (2⁷) = 47 → bit 7 = 1  
47 - 32 (2⁵) = 15 → bit 5 = 1
15 - 8 (2³) = 7 → bit 3 = 1
7 - 4 (2²) = 3 → bit 2 = 1
3 - 2 (2¹) = 1 → bit 1 = 1  
1 - 1 (2⁰) = 0 → bit 0 = 1
Résultat : 1010101111
```

### Autres Bases → Décimal
```
Multiplication positionnelle :
Hex 2AF → Décimal :
2×16² + A(10)×16¹ + F(15)×16⁰
= 2×256 + 10×16 + 15×1
= 512 + 160 + 15 = 687

Octal 237 → Décimal :
2×8² + 3×8¹ + 7×8⁰
= 2×64 + 3×8 + 7×1
= 128 + 24 + 7 = 159
```

### Conversions Directes (Sans Décimal)

#### Binaire ↔ Hexadécimal
```
Grouper par 4 bits (padding à gauche si nécessaire) :

Binaire 10101111 → Hex :
1010 1111 → A F → AF

Hex 2AF → Binaire :
2 → 0010
A → 1010  
F → 1111
Résultat : 001010101111 (ou 10101111)
```

#### Binaire ↔ Octal
```
Grouper par 3 bits :

Binaire 110110 → Octal :
110 110 → 6 6 → 66

Octal 237 → Binaire :
2 → 010
3 → 011
7 → 111  
Résultat : 010011111 (ou 10011111)
```

## 💾 Unités Informatiques

### Système Binaire vs Décimal
| Unité | Binaire (IEC) | Décimal (SI) | Facteur |
|-------|---------------|--------------|---------|
| **Kilo** | 1 KiB = 1024 B | 1 KB = 1000 B | 2¹⁰ vs 10³ |
| **Mega** | 1 MiB = 1024² B | 1 MB = 1000² B | 2²⁰ vs 10⁶ |  
| **Giga** | 1 GiB = 1024³ B | 1 GB = 1000³ B | 2³⁰ vs 10⁹ |
| **Tera** | 1 TiB = 1024⁴ B | 1 TB = 1000⁴ B | 2⁴⁰ vs 10¹² |

### Conversions Pratiques
```
Exemple : 524288 KB → MiB
524288 KB = 524 288 000 octets (décimal)

Méthode 1 - Via KiB :
524 288 000 ÷ 1024 = 512 000 KiB
512 000 ÷ 1024 = 500 MiB

Méthode 2 - Directe :
524 288 000 ÷ (1024²) = 524 288 000 ÷ 1 048 576 = 500 MiB

Différence décimal/binaire :
524288 KB = 524.288 MB (décimal)
524288 KB = 500 MiB (binaire)
```

## 🧮 Applications Pratiques

### Adressage IPv4
```
Adresse IP : 192.168.1.100
En binaire : 11000000.10101000.00000001.01100100

Masque /24 : 255.255.255.0
En binaire : 11111111.11111111.11111111.00000000
            ← 24 bits réseau →|← 8 bits hôte →
```

### Masques de Sous-Réseau
| CIDR | Décimal | Binaire (dernier octet) | Hôtes |
|------|---------|------------------------|-------|
| /24 | 255.255.255.0 | 00000000 | 254 |
| /25 | 255.255.255.128 | 10000000 | 126 |
| /26 | 255.255.255.192 | 11000000 | 62 |
| /27 | 255.255.255.224 | 11100000 | 30 |
| /28 | 255.255.255.240 | 11110000 | 14 |

### Permissions Unix (Octal)
```
rwxrwxrwx = 111 111 111 (binaire) = 777 (octal)
rwxr-xr-x = 111 101 101 (binaire) = 755 (octal)  
rw-r--r-- = 110 100 100 (binaire) = 644 (octal)
r-------- = 100 000 000 (binaire) = 400 (octal)

Calcul rapide :
r(4) + w(2) + x(1) par groupe (user, group, other)
755 = rwx(4+2+1) + r-x(4+0+1) + r-x(4+0+1)
```

## 🎯 Astuces & Raccourcis

### Reconnaissance de Base
- **0x** ou **0X** préfixe → Hexadécimal (0xFF = 255)
- **0b** ou **0B** préfixe → Binaire (0b1111 = 15)  
- **0** préfixe → Octal en C/Unix (077 = 63)
- **%** préfixe → Binaire en assembleur (%1010 = 10)

### Vérifications Rapides
```
Puissance de 2 ? → n & (n-1) == 0
Exemple : 16 & 15 = 10000 & 01111 = 0 ✓

Nombre impair ? → n & 1 == 1
Exemple : 13 & 1 = 1101 & 0001 = 1 ✓

Division par 2ⁿ → Décalage droite (>> n)
Exemple : 20 >> 2 = 20 ÷ 4 = 5

Multiplication par 2ⁿ → Décalage gauche (<< n)  
Exemple : 5 << 3 = 5 × 8 = 40
```

### Complément à 2 (Nombres Signés)
```
Représentation nombres négatifs sur 8 bits :
+5 = 00000101
-5 = 11111011 (inversion bits + 1)

Calcul -5 :
1. Inverser bits de +5 : 11111010
2. Ajouter 1 : 11111011

Vérification : -5 + 5 = 11111011 + 00000101 = 00000000 (overflow ignoré)
```

## 🔧 Outils de Conversion

### Calculatrices Système
- **Windows** : calc.exe mode Programmeur
- **Linux** : `bc`, `printf`, `echo $((...))`  
- **macOS** : Calculator mode Programmeur
- **En ligne** : RapidTables, Calculator.net

### Commandes Shell Utiles
```bash
# Linux/macOS conversions
echo "ibase=10; obase=2; 255" | bc     # Décimal → Binaire
echo "ibase=16; obase=10; FF" | bc     # Hex → Décimal  
printf "%x\n" 255                       # Décimal → Hex
printf "%d\n" 0xFF                      # Hex → Décimal

# Calculs avec bases
echo $((2#1010))                        # Binaire → Décimal (10)
echo $((16#FF))                         # Hex → Décimal (255)
echo $((8#77))                          # Octal → Décimal (63)
```

---
**💡 Memo** : Puissances de 2, groupements par 3/4 bits, et ne pas confondre KB (1000) vs KiB (1024) !