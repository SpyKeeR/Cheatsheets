# Subnetting & CIDR (essentiel) 🧭

## Rappels
- Nombre de sous-réseaux = `2^n` (n = bits empruntés).
- Hôtes par sous-réseau = `2^h - 2` (h = bits hôtes).

## CIDR ↔ Masque
- `/24` → `255.255.255.0`
- `/26` → `255.255.255.192`
- `/28` → `255.255.255.240`

## Nombre magique (quick calc)
1. Trouver octet non-255.  
2. Nombre magique = `256 - octet`.  
3. Plages = multiples de ce nombre.
- Ex : `192.168.1.10/26` → octet clé 192 → magic = 64 → plages 0,64,128,192 → réseau = `192.168.1.0`, broadcast `192.168.1.63`.

## Exemple — plages libres (199.1.1.0/24)
- Occupés : `/26 0-63`, `/27 128-159`, `/27 160-191`, `/28 192-207`, `/28 224-239`, `/28 240-255`.
- Plages libres : `199.1.1.64-127` → → `199.1.1.64/26` ; `199.1.1.208-223` → `199.1.1.208/28`.

## Exemple découpage /24 en segments
- Besoins : 62, 3×30, 4×14 → attribution optimisée :
  - `192.168.1.0/26` (62)
  - `192.168.1.64/27`, `192.168.1.96/27`, `192.168.1.128/27` (30)
  - `192.168.1.160/28`, `...176/28`, `...192/28`, `...208/28` (14)
- Reste inutile : `192.168.1.224/27`.

## RFC1918 réseaux privés
- A : `10.0.0.0/8`
- B : `172.16.0.0/12`
- C : `192.168.0.0/16`

## Supernetting (exemples)
- Couvrir `212.1.96.0` → `212.1.159.0` : 64 réseaux → tenter `/18` (64 réseaux) → 96 n’est pas multiple de 64 ⇒ `/16` garde la plage (212.1.0.0/16) — solution valide mais large.
- Regrouper 4 x `/24` (212.56.146.0 - 149.0) : `/22` (4 réseaux) *idée* mais 146 n’est pas multiple de 4:
  - Option 1: deux `/23` (146/23 & 148/23) — précis.
  - Option 2: un `/21` (212.56.144.0/21) — plus large, adresses inutilisées.