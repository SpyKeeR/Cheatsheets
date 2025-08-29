# Réseau — configuration & NMCLI (condensé) 🌐

## Noms interfaces
- Noms d'interfaces prédictifs : ensX / enpXsY / lo
- Déterminer noms d'interfaces : `dmesg | grep -i eth`

## ifupdown (exemple `/etc/network/interfaces`)
- `auto ens33` → activer au boot.  
- `allow-hotplug ens33` → activer à branchement.  
- DHCP : `iface ens33 inet dhcp`  
- Statique :  
  - `iface ens33 inet static`  
  - `address 10.1.1.10`  
  - `netmask 255.255.255.0` (ou `address 10.1.1.10/24`)  
  - `gateway 10.1.1.1`
 - Créer une interface virtuelle
	- `iface ens33:1 inet static`

## Redémarrer le réseau ou NetworkManager
- `systemctl stop networking.service`  
- `systemctl start networking.service`
- `systemctl restart NetworkManager`

## NetworkManager & nmcli (exemples)
- Ajouter connexion (statique) :  
  `nmcli con add con-name my-con-em1 ifname em1 type ethernet ip4 192.168.100.100/24 gw4 192.168.100.1 ip4 1.2.3.4 ip6 abbe::cafe` 
	- `con-name my-con-em1` ➝ Nom de la connexion (arbitraire, utilisé pour la gestion avec nmcli)
	- `ifname em1` ➝ Interface physique concernée (em1)
	- `type ethernet` ➝ Type de connexion (ici Ethernet filaire)
	- `ip4 192.168.100.100/24` ➝ Adresse IPv4 statique et son masque
	- `gw4 192.168.100.1` ➝ Passerelle IPv4 par défaut
	- `ip4 1.2.3.4` ➝ Ajout d’un DNS IPv4
	- `ip6 abbe::cafe` ➝ Adresse IPv6 statique
- Modifier connexion (DNS par exemple) : `nmcli con mod my-con-em1 ipv4.dns "8.8.8.8 8.8.4.4"` ; ajouter : `+ipv4.dns 1.2.3.4`  
	- Modifier IP : `nmcli connection modify "Wired connection 1" ipv4.addresses 192.168.1.100/24 ipv4.method manual`
	- Modifier GW : `nmcli connection modify <nom_connection> ipv4.gateway <gateway>`
- Afficher détails : `nmcli -p con show my-con-em1`  
- Files gérés : `/etc/NetworkManager/system-connections/*.nmconnection` 
	- Ubuntu : /etc/netplan.*
	- RHEL : /etc/sysconfig/network-scripts/ifcfg-*

## Commandes pratiques
- Voir interfaces/IP : `ip a` / `ip address`  
- Start / Stop interface : `ip link set dev eth0 {up|down}`
- Ajouter/supprimer une IP à une interface : `ip addr {add|del} 192.168.1.100/32 dev eth0`
- Supprimer toutes les IP's d'une interface : `ip address flush eth0`
- Changer de mac address : `ip link set eth0 address aa:bb:cc:dd:ee:ff`
- ARP / voisins : `ip neigh show`  

## Routage & NAT (condensé)
- Voir routes : `ip r` / `ip route` (-6 IPv6)
- Ajouter une route par défaut : `ip route add default via 192.168.1.1`
- Ajouter une route à une interface : `ip route add 192.168.0.0/24 dev eth0`
- Ajouter une route par un next hop : `ip route add 10.56.0.0/16 via 172.16.6.253`
- Obtenir une route pour une IP : `ip route get to 8.8.8.8`
- Supprimer les routes : `ip route flush`
- Persistance : scripts boot ou prefix pre-/post-/up dans /etc/network/interfaces
- Activer forwarding : `sysctl net.ipv4.ip_forward=1` (ou dans /etc/sysctl.conf `net.ipv4.ip_forward = 1` + `sysctl -p`)
- NAT : `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE` (legacy, utiliser `nftables`). 
	- Alternatives : Shorewall, Firewalld (frameworks simplifiant règles) - pfSense (appliance).
- Test avant reboot (interfaces, routes, firewall rules persistantes)

## Hostname & résolution locale
- Fichier : /etc/hostname (hostnamectl set-hostname srv-lan.demo.eni)
- /etc/hosts : 127.0.1.1 srv-lan.demo.eni srv-lan
- Ordre de résolution : /etc/nsswitch.conf 
	- Basic install → `hosts: files dns` → on lit d’abord /etc/hosts, puis on interroge les DNS
	- Si GUI(ou avahi-daemon) → `files mdns4_minimal [NOTFOUND=return] dns myhostname` → /etc/hosts → Interrogation mDNS (\*.local) → DNS → nom local du poste

## Configuration DNS
- `/etc/resolv.conf` 
	- `nameserver` → designer des Serveurs résolveurs DNS
	- `search` → pour domaines à utiliser pour compléter les hostname seuls (`domain` : déprécié)
	- `options` → pour configurer des options supplémentaires (`debug`, `ndots:n`, `timeout:n`, `attempts:n`, `rotate`, `no-aaaa`, `no-check-names`, `inet6`, `edns0`, `use-vc`, `no-reload`)
- `systemd-resolved` → Resolver DNS systemd
	- `resolvectl status` → Affiche la configuration de resolved et des DNS
	- `resolvectl query <hostname>` → Résoudre un nom
	- `resolvectl dns <interface> <IP>` → Déclarer des serveurs DNS pour une interface
	- `resolvectl domain <interface> <name>` → Configurer les suffixes DNS pour une interface
	- `resolvectl flush-caches` → Nettoyer le cache DNS de resolved
- `host`, `nslookup`, `dig` → Commandes (`dns-utils` paquet) pour la résolution DNS
- `avahi-daemon` → mDNS paquet à installer (Resoudre les noms NetBIOS .local)

## Diagnostic
- `lsof -i :22` → processus sur port.  
- `ss -tuln` / `ss -tnp` / `netstat` → sockets connexions.  
- `ipcalc` → aide subnetting.
- `iftop` → trafic par IP.  
- `nethogs` → trafic par process. 
- `ethtool` → pour lien
- `dig @server name A/AAAA`, `host`, `nslookup` → DNS lookup debug
- `traceroute`, `mtr` → Analyse route + connectivité hops.
- `conntrack -L` (state)
- `iptables -vnL`
- `tcpdump -i eth0 -nn -s0 -c 100`
- `nft list ruleset`