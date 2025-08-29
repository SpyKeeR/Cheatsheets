# IPv6 — essentiels & NDP (condensé) 🌐

## Types d'adresses
- Loopback : `::1/128`.
- Lien-local : `fe80::/10` (non routable, souvent /64).
- Unique locale (ULA) : `fd00::/8` (privé).
- Global : `2000::/3` (routable Internet).
- Anycast : identique à unicast mais routé vers plus proche.
- Multicast : `ff00::/8` (groupes).

## Adresses multicast spécifiques
- `ff02::1` → tous nœuds lien-local.
- Sollicité NDP : `FF02::1:FFXX:XXXX` (24 derniers bits de l'IPv6 cible).
- MAC multicast dérivée : `33:33:FF:XX:XX:XX`.

## NDP — Neighbor Discovery (résumé)
1. Emplacement lien-local créé (`fe80::...`).
2. PC A génère multicast sollicité `FF02::1:FFxx:xxxx` à partir des 24 derniers bits de cible.
3. Envoi Neighbor Solicitation (NS) → MAC multicast.
4. Destinataire répond Neighbor Advertisement (NA) → contient MAC.
5. Table NDP mise à jour, communication possible.

## SLAAC & attribution
- SLAAC : découverte routeur → préfixe global → auto-config adresse.
- Options : SLAAC, DHCPv6, statique.
- L'address fe80 permet échanges initiaux avec routeur.

## Compression & notation
- Supprimer zéros initiaux dans chaque quartet.
- Remplacer séquence continue de zéros par `::` (une seule fois).

## Transition IPv4/IPv6
- Double stack, tunneling, NAT64.