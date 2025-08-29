# Commandes réseau & états — Cheat rapide 🛠️

## ARP
- `arp -a` → afficher table ARP (Win/Linux).
- `arp -s IP MAC` → ajouter entrée statique (Windows).
- `arp -d *` → vider table ARP (Windows).
- `sudo ip neigh flush all` → vider ARP (Linux).

## Windows IP (ipconfig)
- `ipconfig` → résumé IP.
- `ipconfig /all` → détails (MAC, DHCP, DNS...).
- `ipconfig /release` → libérer DHCP.
- `ipconfig /renew` → renouveler DHCP.
- `ipconfig /flushdns` → vider cache DNS.
- `ipconfig /displaydns` → afficher cache DNS.
- `ipconfig /registerdns` → renouveler enregistrements DNS dynamiques.

## Linux IP (ip)
- `ip link show` → interfaces.
- `ip addr show` → adresses.
- `ip addr add 192.168.1.200/24 dev eth0` → ajouter IP.
- `ip addr del 192.168.1.200/24 dev eth0` → supprimer IP.
- `ip route show` → table routage.
- `ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0` → ajouter route.
- `ip route del 192.168.2.0/24` → supprimer route.
- `ip neigh show` → table voisins (ARP).

## Connexions & sockets
- `netstat -a` → toutes connexions + ports écoute.
- `netstat -n` → sans résolution DNS.
- `netstat -t` → TCP ; `-u` → UDP ; `-l` → écoute ; `-p` → processus.
- Exemple root → `sudo netstat -tulpn`.
- Linux moderne : `ss` (remplace netstat).
- Windows PowerShell : `Get-NetTCPConnection`.

## Ping
- `ping -s 1000 host` → taille paquet ICMP 1000.
- `ping -i 0.5 host` → intervalle 0.5s.
- `ping -q host` → q/ stat seulement.
- `ping -t host` → continu (Windows).
- Erreurs classiques : Request timeout, Network is unreachable, Destination Host Unreachable, Unknown host, Packet loss.

## Traceroute
- Windows : `tracert [options]` ; Linux : `traceroute [options]`.
- `-n` / `/d` → désactiver résolution DNS.
- `-m` / `/h` → TTL max.
- `-w` / `/w` → timeout.
- `-4` / `-6` → forcer IPv4/IPv6.
- `-I` (Linux) → ICMP mode.

## États TCP (rapide)
- `LISTEN` → attente connexions.
- `SYN_SENT` → demande envoyée.
- `ESTABLISHED` → connexion active.
- `TIME_WAIT` → fermé en attente.
- `CLOSE_WAIT` → attente fermeture locale.