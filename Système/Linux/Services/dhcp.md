# 🌐 DHCP — Aide-mémoire

## 🏗️ Architecture & Concepts

### Protocole DHCP
- **Port** : UDP 67 (serveur), UDP 68 (client)
- **Processus** : DORA (Discover, Offer, Request, Acknowledge)
- **Broadcast** : Client diffuse sur réseau local
- **Relay** : Routeur/Switch forward vers serveur DHCP distant

### Types de Baux
| Type | Durée | Usage |
|------|-------|-------|
| **Dynamique** | Limitée (configurable) | Postes standards |
| **Statique** | Illimitée | Réservations MAC |
| **Automatique** | Permanente | Première attribution = définitive |

### États du Bail
```
DISCOVER → OFFER → REQUEST → ACKNOWLEDGE → BOUND
    ↓        ↓        ↓          ↓          ↓
  Recherche Proposition Demande  Validation Attribution
                                              ↓
                              RENEWING (T1) → 50% durée bail
                                     ↓
                              REBINDING (T2) → 87.5% durée bail
```

## ⚙️ Configuration ISC DHCP Server

### Fichiers Principaux
```
/etc/dhcp/dhcpd.conf              # Configuration principale
/var/lib/dhcp/dhcpd.leases        # Baux actifs (état runtime)
/etc/default/isc-dhcp-server      # Interfaces d'écoute
/var/log/syslog                   # Logs système (ou journalctl)
```

### Structure Configuration
```bash
# Configuration globale
option domain-name "lab.local";
option domain-name-servers 8.8.8.8, 1.1.1.1;
default-lease-time 21600;           # 6 heures
max-lease-time 43200;               # 12 heures
ddns-update-style none;
authoritative;                      # Ce serveur est l'autorité

# Subnet déclaration (obligatoire pour interface d'écoute)
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;        # Plage dynamique
    option routers 192.168.1.1;               # Passerelle par défaut
    option broadcast-address 192.168.1.255;   # Adresse broadcast
    option subnet-mask 255.255.255.0;         # Masque réseau
    
    # Réservation statique
    host printer-hp {
        hardware ethernet 08:00:27:aa:bb:cc;
        fixed-address 192.168.1.10;
        option host-name "printer-hp";
    }
}
```

### Options DHCP Courantes
| Option | Code | Usage | Exemple |
|--------|------|-------|---------|
| `routers` | 3 | Passerelle par défaut | `192.168.1.1` |
| `domain-name-servers` | 6 | Serveurs DNS | `8.8.8.8, 1.1.1.1` |
| `domain-name` | 15 | Domaine DNS | `"lab.local"` |
| `broadcast-address` | 28 | Adresse broadcast | `192.168.1.255` |
| `ntp-servers` | 42 | Serveurs NTP | `pool.ntp.org` |
| `netbios-name-servers` | 44 | Serveurs WINS | `192.168.1.5` |

### Interface d'Écoute
```bash
# /etc/default/isc-dhcp-server
INTERFACESv4="eth0 eth1"            # Interfaces IPv4
INTERFACESv6=""                     # Interfaces IPv6 (si utilisé)
```

## 🔄 DHCP Relay

### Principe & Usage
- **But** : Faire passer requêtes DHCP entre VLANs/réseaux
- **Champ GIADDR** : Adresse IP du relay (identifie le réseau source)
- **Configuration** : Simple forwarding vers serveur(s) DHCP

### Configuration Relay
```bash
# /etc/default/isc-dhcp-relay
SERVERS="192.168.10.5 192.168.10.6"     # Serveurs DHCP cibles
INTERFACES="eth0 eth1 eth2"             # Interfaces à relayer
OPTIONS="-a"                            # -a: ajout GIADDR automatique

# Options utiles
# -d : mode debug (foreground)
# -q : mode silencieux
# -a : enable giaddr checking
# -A : abandon si pas de DHCP servers joignables
```

### Subnet pour Relay
```bash
# Sur le serveur DHCP : déclarer subnet pour chaque réseau relayé
subnet 172.16.10.0 netmask 255.255.255.0 {
    range 172.16.10.50 172.16.10.100;
    option routers 172.16.10.1;           # Gateway du VLAN distant
    option domain-name-servers 192.168.1.5;
}
```

## 🏋️ Haute Disponibilité (Failover)

### Architecture Failover ISC
- **Primary/Secondary** : Un serveur principal, un backup
- **Load Balancing** : Répartition de charge configurable
- **Synchronisation** : État des baux partagé entre serveurs
- **Split** : Pourcentage de répartition (0-255, 128=50/50)

### Configuration Failover
```bash
# Configuration peer (identique sur les deux serveurs)
failover peer "dhcp-failover" {
    # Serveur primaire
    primary;                              # ou 'secondary;' sur le backup
    address 192.168.1.10;               # IP de ce serveur
    port 647;                            # Port failover
    peer address 192.168.1.11;          # IP serveur partenaire
    peer port 647;                      # Port partenaire
    
    # Paramètres temporels
    max-response-delay 60;              # Délai max réponse partenaire
    max-unacked-updates 10;             # Updates non-ACK max
    mclt 3600;                          # Max Client Lead Time (1h)
    split 128;                          # Répartition 50/50
    
    # Load balancing
    load balance max seconds 3;         # Délai max équilibrage
}

# Application dans subnet
subnet 192.168.1.0 netmask 255.255.255.0 {
    pool {
        failover peer "dhcp-failover";
        range 192.168.1.100 192.168.1.200;
    }
    option routers 192.168.1.1;
}
```

### États Failover
| État | Description | Comportement |
|------|-------------|--------------|
| **NORMAL** | Les deux serveurs actifs | Répartition selon split |
| **COMMUNICATIONS-INTERRUPTED** | Perte contact partenaire | Continue avec baux existants |
| **PARTNER-DOWN** | Partenaire confirmé HS | Prend tous nouveaux baux |
| **POTENTIAL-CONFLICT** | Désynchronisation détectée | Mode sécurisé |
| **RECOVER** | Redémarrage après panne | Resynchronisation |

## 🛡️ Sécurité & Performance

### Configuration Sécurisée
```bash
# Limitation taux de requêtes (anti-DoS)
class "throttle" {
    match if binary-to-ascii(10, 8, ".", leased-address) 
          = binary-to-ascii(10, 8, ".", option dhcp-client-identifier);
    lease limit 2;                   # Max 2 baux par client
}

# Filtrage par classes
class "known-clients" {
    match hardware;                  # Filtrer par MAC
}
class "unknown-clients" {
    match not hardware;
}

subnet 192.168.1.0 netmask 255.255.255.0 {
    pool {
        allow members of "known-clients";
        range 192.168.1.100 192.168.1.150;
    }
    pool {
        deny members of "unknown-clients";
        range 192.168.1.151 192.168.1.200;
    }
}
```

### OMAPI (Object Management API)
```bash
# Configuration OMAPI pour gestion distante
omapi-port 7911;
key omapi-key {
    algorithm hmac-md5;
    secret "base64-encoded-secret-key==";
}
omapi-key omapi-key;

# Génération clé
dnssec-keygen -a HMAC-MD5 -b 128 -n HOST omapi-key
# Utiliser la clé générée dans le fichier .key
```

## 🔧 Gestion & Maintenance

### Commandes Administration
```bash
# Test configuration
dhcpd -t -cf /etc/dhcp/dhcpd.conf

# Mode debug (foreground)
dhcpd -d -f

# Gestion service
systemctl status isc-dhcp-server
systemctl restart isc-dhcp-server
systemctl reload isc-dhcp-server    # Recharge config sans couper services

# Monitoring logs
journalctl -u isc-dhcp-server -f
tail -f /var/log/syslog | grep dhcpd
```

### Analyse Baux
```bash
# Consultation fichier leases
cat /var/lib/dhcp/dhcpd.leases | grep -E "(lease|binding state|hardware ethernet)"

# Statistiques baux actifs
grep "binding state active" /var/lib/dhcp/dhcpd.leases | wc -l

# Dernières attributions
tail -n 100 /var/lib/dhcp/dhcpd.leases
```

### Clients DHCP (Test)
```bash
# Test client Linux
dhclient -v eth0                    # Demander bail DHCP
dhclient -r eth0                    # Libérer bail
dhclient -x                         # Libérer tous baux

# Informations bail actuel
cat /var/lib/dhcp/dhclient.leases

# Simulation découverte
nmap --script broadcast-dhcp-discover
```

## 🎯 Configurations Spécialisées

### DHCP pour PXE Boot
```bash
host pxe-client {
    hardware ethernet 00:11:22:33:44:55;
    fixed-address 192.168.1.50;
    next-server 192.168.1.100;        # Serveur TFTP
    filename "pxelinux.0";            # Fichier boot
}

# Options PXE globales
option space PXE;
option PXE.mtftp-ip code 1 = ip-address;
option PXE.mtftp-cport code 2 = unsigned integer 16;
```

### DHCP Snooping (Concepts)
- **But** : Sécuriser DHCP en environnement switchés
- **Trusted Ports** : Ports vers serveurs DHCP légitimes
- **Untrusted Ports** : Ports clients (bloque offres DHCP)
- **Binding Table** : Association IP-MAC-Port validée

### Option 82 (Circuit ID)
```bash
# Configuration pour support Option 82
option agent.circuit-id code 1 = string;
option agent.remote-id code 2 = string;

# Classe par circuit ID
class "vlan-100" {
    match if binary-to-ascii(10, 32, "", 
          substring(option agent.circuit-id, 2, 4)) = "100";
}
```

## 🏢 Environnements Multiples

### Configuration Multi-Sites
```bash
# Serveur central avec multiples subnets
shared-network "site-principal" {
    subnet 192.168.1.0 netmask 255.255.255.0 {
        range 192.168.1.100 192.168.1.200;
        option routers 192.168.1.1;
    }
    subnet 192.168.2.0 netmask 255.255.255.0 {
        range 192.168.2.100 192.168.2.200;
        option routers 192.168.2.1;
    }
}

# Groupes d'options par site
group {
    option domain-name "paris.company.com";
    option domain-name-servers 192.168.1.5;
    
    subnet 192.168.10.0 netmask 255.255.255.0 { ... }
    subnet 192.168.11.0 netmask 255.255.255.0 { ... }
}
```

### Alternative Moderne : Kea DHCP
- **ISC Kea** : Remplaçant moderne d'ISC DHCP
- **JSON Config** : Configuration en JSON vs texte
- **API REST** : Gestion via API
- **Hooks** : Extensibilité par plugins
- **Performance** : Optimisé pour cloud/containers

## 🔍 Troubleshooting

### Problèmes Courants
| Symptôme | Cause Probable | Solution |
|----------|----------------|----------|
| **Pas d'IP attribuée** | Pas de subnet déclaré | Vérifier subnet pour interface |
| **Bail pas renouvelé** | Serveur non joignable | Vérifier connectivité/firewall |
| **Conflit IP** | Attribution statique dupliquée | Vérifier réservations |
| **Lease expired** | Durée trop courte | Augmenter lease-time |

### Debug DHCP
```bash
# Capture trafic DHCP
tcpdump -i eth0 port 67 or port 68 -v

# Test depuis client
dhcping -s 192.168.1.10 -c 192.168.1.100

# Vérification pools
grep -A 10 "subnet" /etc/dhcp/dhcpd.conf | grep range

# Validation syntaxe avancée
dhcpd -T                            # Test + validation complète
```

### Métriques Surveillance
- **Pool Usage** : % baux utilisés vs disponibles
- **Lease Duration** : Durée moyenne des baux
- **Request Rate** : Requêtes DHCP par seconde
- **Failover State** : État synchronisation si HA

---
**💡 Memo** : Déclarer subnet pour interface d'écoute, GIADDR pour relay, split 128 pour HA équilibrée !