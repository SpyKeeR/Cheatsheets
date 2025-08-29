# Bases conversions & unités (condensé) 🔢

## Binaire ↔ Hex
- Exemple hex `2AF` → `2*16² + A(10)*16 + F(15) = 687`.
- Binaire `10101111` → grouper par 4 → `1010 1111` → `A F` → `AF`.

## Binaire ↔ Octal
- Exemple octal `237` → `2*8² + 3*8 + 7 = 159`.
- Binaire `110110` → grouper par 3 → `110 110` → `6 6` → `66` (octal).

## Méthodes rapides
- Tableaux de puissances de 2 pour convertir décimal→binaire par soustraction itérative.
- Décimal→Hex via décimal→binaire puis groupes de 4 bits.
- Décimal→Octal : divisions successives par 8, lire restes bas→haut.

## Unités (exemple pratique)
- 524288 Ko (base10) → 524 288 000 octets.
- KiB = /1024 ; MiB = /1024².
- 524 288 000 octets → /1024 = 512000 KiB → /1024 = 500 MiB.