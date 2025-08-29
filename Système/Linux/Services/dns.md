# BIND (named) — résolveur / serveur DNS (condensé)

## Fichiers principaux
- /etc/bind/named.conf (includes)
	- `acl rsxclts { 127.0.0.0/8; 192.168.53.0/24; 192.168.1.0/24; };` → ACL autorisée à interroger
- /etc/bind/named.conf.options (forwarders, ACLs, recursion, dnssec-validation, version)
	- `forwarders { 9.9.9.9; };` → définit les DNS publics vers lesquels rediriger (ex : Quad9, Google, DNS FAI, etc.)
	- `directory "/var/cache/bind";` → Répertoire de travail de BIND (où il stocke les fichiers temporaires)  
	- `forward only;` → Ne demande JAMAIS les serveurs racines, utilise SEULEMENT les forwarders  
	- `allow-query { rsxclts; };` → Autorise les requêtes uniquement depuis les IPs définies dans 'rsxclts'  
	- `allow-recursion { rsxclts; };` → Autorise la récursivité (résolution complète) uniquement pour ces IPs  
	- `dnssec-validation no;` → Désactive la vérification DNSSEC  
	- `version "none";` → Masque la version de BIND dans les réponses DNS  
- /etc/bind/named.conf.local (zones locales)
	- Zone directe maitre
	```bash
	zone "demo.eni" {  
	type master; # Serveur maître de la zone  
	file "/etc/bind/db.demo.eni"; # Fichier de zone défini manuellement (à créer)  
	allow-transfer { 10.11.0.53; };  # Contrôle les IP autorisées à copier la zone
	};
	```
	- Redirection conditionelle 
	```bash
	zone "macharmantevoisine.eni" {  
	type forward; # Déclare une zone de redirection conditionnelle  
	forward only; # Utilise uniquement le redirecteur défini  
	forwarders { 10.0.0.53; }; # Serveur DNS vers lequel rediriger les requêtes pour ce domaine  
	};
	```
	- Zone directe inverse 
	```bash
	zone "1.168.192.in-addr.arpa" {
	type master;
	file "/etc/bind/db.192.168.1";
	};
	```
- /etc/bind/named.conf.default-zones (zones par défaut)

- Zone files → /var/cache/bind ou /etc/bind/db.*

## Fichier de zone — structure minimale
- `$ORIGIN demo.eni.` → définit le nom du domaine principal de la zone (@ alias de $ORIGIN)
- `$TTL 86400` → valeur par défaut du temps de cache (en secondes)
- `@ IN SOA ns1.demo.eni. admin.demo.eni.` → nom FQDN du serveur autoritaire primaire (terminé par un point) / Adresse mail admin
	- `(2025010101` → Serial : Numéro de version de la zone
	- `86400` → Refresh → Fréquence de MAJ de la zone par les serveurs secondaires
	- `7200` → Retry → Si échec de MAJ, intervalle de nouvelle tentative
	- `3600000` → Expire → Durée max avant qu'un secondaire conserve zone sans contact avec primaire
	- `86400` → Neg. TTL → Durée de cache pour les réponses négatives
	- `)` → Fin de bloc de propriétés de zone.
- `@ IN NS ns1.demo.eni.` → Serveur(s) DNS faisant autorité sur la zone
- `ns1 IN A 10.5.3.10` → Enregistrement exemple
- PTRs dans fichier inverse

## Types d'enregistrements

- 🧾 SOA → Caractéristiques de la zone (serveur maître, TTL, serial...)
- 🧭 NS	→ Serveur(s) DNS faisant autorité sur la zone
- 🏠 A → Nom → adresse IPv4
- 🌐 AAAA → Nom → adresse IPv6
- 📦 SRV → Service (ex : SIP, LDAP...) → FQDN du serveur
- 📬 MX → Serveur(s) de messagerie du domaine
- 🔁 CNAME → Alias : un nom → un autre nom
- 🔙 PTR → Inverse : IP → nom (présent uniquement en zone inverse)

### DynDNS / mise à jour dynamique
- Options Bind : `allow-update { ip_serveur_DHCP; key "TSIG"; };`
- côté DHCP : 
	- `ddns-updates on;` → Active les updates
	- `ddns-update-style standard;` → Mode classique
	- `ddns-domainname "demo.eni";` → Nom du domaine DNS
	- `ignore client-updates;` → Le serveur DHCP prend le relais, pas les clients
	- `update-static-leases on;` → Pour que les réservations soient aussi mises à jour
- Outils clients : `nsupdate` pour updates manuelles
- Modifications manuelles : 
	- Geler la zone avec `rndc freeze nom.zone`
	- Modifier le fichier de zone
	- Dégeler `rndc unfreeze nom.zone`

### Transferts & délégation
- `allow-transfer { ip_secondaire; }` → autorise AXFR/IXFR
- Délégation : ajouter NS pour sous-domaine dans zone parente + glue A si NS dans domaine délégué
- Si résolveur inconditionnel, ajouter `forwarders {};` pour forcer la requête itérative du sous-domaine délégué.
- Pour notifier les slaves, afin de déclencher un transfert de zone de leur part :
	- `notify yes;` Notifier tous les NS de zone.
	- `notify yes; also-notify { x.x.x.x; y.y.y.y; };` Notifier tous les NS de zone + les IP's/ACL listées.
	- `notify explicit; also-notify { x.x.x.x; y.y.y.y; };` Notifier uniquement les IP's/ACL listées.

## Administration - Diagnostic
- Vérif conf : `named-checkconf`
- Vérif zone : `named-checkzone <zone> <zonefile>`
- Reload : `rndc reload` (ou `systemctl restart bind9`)
- Logs : `journalctl -u bind9` ou /var/log/syslog

### Sécurité & robustesse
- Restreindre recursion aux réseaux de confiance (`allow-recursion`)
- Masquer version (`version "none"`)
- DNSSEC : activer si possible — sinon `dnssec-validation no`

### Conseils pratiques
- Incrementer serial (format AAAAMMJJnn) à chaque modif manuelle
- Utiliser TSIG pour sécuriser transferts et mises à jour via DHCP
