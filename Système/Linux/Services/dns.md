# 🌐 DNS BIND — Aide-mémoire

## 🏗️ Architecture DNS

### Concepts Fondamentaux
- **Zone** : Portion d'espace de noms DNS sous autorité d'un serveur
- **Autorité** : Serveur possédant les données officielles d'une zone
- **Récursion** : Résolution complète au nom du client
- **Itération** : Renvoi de références vers autres serveurs
- **Cache** : Stockage temporaire des réponses pour optimisation

### Types de Serveurs
| Type | Rôle | Usage |
|------|------|-------|
| **Autoritaire** | Données officielles zone | Serveur maître/esclave |
| **Récursif** | Résolution pour clients | Résolveur entreprise |
| **Forwarder** | Relais vers autres DNS | Proxy DNS |
| **Root/TLD** | Serveurs racines | Infrastructure Internet |

## ⚙️ Configuration BIND

### Structure Fichiers
```
/etc/bind/
├── named.conf                 # Configuration principale (includes)
├── named.conf.options         # Options globales
├── named.conf.local           # Zones locales
├── named.conf.default-zones   # Zones par défaut
├── db.* ou zones/             # Fichiers de zones
└── keys/                      # Clés TSIG/DNSSEC
```

### Configuration Options Globales
```bash
# /etc/bind/named.conf.options
acl "clients-internes" {
    127.0.0.0/8;
    192.168.0.0/16;
    10.0.0.0/8;
};

options {
    directory "/var/cache/bind";
    
    # Récursion et forwarding
    recursion yes;
    allow-recursion { clients-internes; };
    allow-query { clients-internes; };
    
    forwarders {
        9.9.9.9;         # Quad9
        1.1.1.1;         # Cloudflare
    };
    forward only;        # Utilise SEULEMENT les forwarders
    
    # Sécurité
    version "DNS Server";
    dnssec-validation auto;
    auth-nxdomain no;
    
    # Performance
    max-cache-size 256m;
    cleaning-interval 60;
};
```

## 🗂️ Gestion des Zones

### Déclaration Zone Maître
```bash
# /etc/bind/named.conf.local
zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
    allow-transfer { 
        192.168.1.10;    # Serveur secondaire
        key "transfer-key"; 
    };
    allow-update { 
        key "ddns-key";  # Mises à jour dynamiques
    };
    notify yes;
    also-notify { 192.168.1.10; };
};

# Zone inverse
zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.1";
    allow-transfer { 192.168.1.10; };
};
```

### Zone Esclave (Secondaire)
```bash
zone "example.com" {
    type slave;
    masters { 192.168.1.5; };
    file "/var/cache/bind/db.example.com";
    allow-notify { 192.168.1.5; };
};
```

### Redirection Conditionnelle
```bash
zone "partenaire.local" {
    type forward;
    forward only;
    forwarders { 10.20.0.53; };
};
```

## 📄 Fichiers de Zone

### Structure Zone Directe
```bash
# /etc/bind/zones/db.example.com
$ORIGIN example.com.
$TTL 86400

; SOA (Start of Authority)
@   IN  SOA ns1.example.com. admin.example.com. (
        2024011501  ; Serial (YYYYMMDDNN)
        86400       ; Refresh (24h)
        7200        ; Retry (2h)
        3600000     ; Expire (1000h)
        86400       ; Negative Cache TTL (24h)
)

; Serveurs de noms
@           IN  NS      ns1.example.com.
@           IN  NS      ns2.example.com.

; Serveurs mail (priorité)
@           IN  MX  10  mail.example.com.
@           IN  MX  20  mail2.example.com.

; Enregistrements A (IPv4)
ns1         IN  A       192.168.1.10
ns2         IN  A       192.168.1.11
www         IN  A       192.168.1.20
mail        IN  A       192.168.1.30
ftp         IN  A       192.168.1.40

; Enregistrements AAAA (IPv6)
www         IN  AAAA    2001:db8::20

; Alias (CNAME)
webmail     IN  CNAME   mail.example.com.
blog        IN  CNAME   www.example.com.

; Services (SRV) - format: _service._protocol
_sip._tcp   IN  SRV     10 5 5060 sip.example.com.
_ldap._tcp  IN  SRV     0  5 389  ldap.example.com.

; Enregistrements TXT
@           IN  TXT     "v=spf1 mx a:mail.example.com ~all"
_dmark      IN  TXT     "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"
```

### Zone Inverse (PTR)
```bash
# /etc/bind/zones/db.192.168.1
$ORIGIN 1.168.192.in-addr.arpa.
$TTL 86400

@   IN  SOA ns1.example.com. admin.example.com. (
        2024011501  ; Serial
        86400       ; Refresh
        7200        ; Retry
        3600000     ; Expire
        86400       ; Negative Cache TTL
)

@       IN  NS      ns1.example.com.
@       IN  NS      ns2.example.com.

; PTR Records (IP → Nom)
10      IN  PTR     ns1.example.com.
11      IN  PTR     ns2.example.com.
20      IN  PTR     www.example.com.
30      IN  PTR     mail.example.com.
```

## 🔄 DNS Dynamique (DDNS)

### Configuration BIND pour DDNS
```bash
# Génération clé TSIG
tsig-keygen -a HMAC-SHA256 ddns-key

# /etc/bind/named.conf.local
key "ddns-key" {
    algorithm hmac-sha256;
    secret "base64-encoded-secret";
};

zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
    allow-update { key "ddns-key"; };
    update-policy {
        grant ddns-key zonesub any;
    };
};
```

### Integration DHCP-DDNS
```bash
# Configuration DHCP (ISC DHCP)
ddns-updates on;
ddns-update-style standard;
ddns-domainname "example.com";
ddns-rev-domainname "in-addr.arpa";

key "ddns-key" {
    algorithm hmac-sha256;
    secret "base64-encoded-secret";
}

zone example.com. {
    primary 192.168.1.10;
    key ddns-key;
}

zone 1.168.192.in-addr.arpa. {
    primary 192.168.1.10;
    key ddns-key;
}
```

### Mise à Jour Manuelle
```bash
# Geler zone pour modification manuelle
rndc freeze example.com

# Modifier fichier zone
nano /etc/bind/zones/db.example.com
# Incrémenter serial!

# Dégeler zone
rndc unfreeze example.com

# Ou utiliser nsupdate
nsupdate -k /etc/bind/keys/ddns-key.private << EOF
server 192.168.1.10
zone example.com
update add test.example.com. 300 A 192.168.1.100
send
EOF
```

## 📊 Types d'Enregistrements DNS

### Enregistrements Essentiels
| Type | Purpose | Format | Exemple |
|------|---------|--------|---------|
| **A** | IPv4 | `nom IN A adresse` | `www IN A 192.168.1.10` |
| **AAAA** | IPv6 | `nom IN AAAA adresse` | `www IN AAAA 2001:db8::1` |
| **CNAME** | Alias | `alias IN CNAME cible` | `blog IN CNAME www.example.com.` |
| **MX** | Mail | `@ IN MX priorité serveur` | `@ IN MX 10 mail.example.com.` |
| **NS** | Name Server | `@ IN NS serveur` | `@ IN NS ns1.example.com.` |
| **PTR** | Reverse | `ip IN PTR nom` | `10 IN PTR www.example.com.` |
| **TXT** | Texte | `nom IN TXT "texte"` | `@ IN TXT "v=spf1 mx ~all"` |
| **SRV** | Service | `_svc._proto IN SRV pri poids port cible` | `_sip._tcp IN SRV 10 5 5060 sip.example.com.` |

### Enregistrements Sécurité/Email
```bash
# SPF (Sender Policy Framework)
@       IN  TXT     "v=spf1 mx a:mail.example.com ip4:192.168.1.30 ~all"

# DKIM (Domain Keys Identified Mail)
default._domainkey  IN  TXT  "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBA..."

# DMARC (Domain-based Message Authentication)
_dmarc  IN  TXT     "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"

# CAA (Certificate Authority Authorization)
@       IN  CAA     0 issue "letsencrypt.org"
@       IN  CAA     0 iodef "mailto:admin@example.com"
```

## 🔧 Administration & Maintenance

### Commandes rndc
```bash
# Contrôle serveur
rndc status                    # État serveur
rndc reload                    # Recharger configuration
rndc reload example.com        # Recharger zone spécifique
rndc refresh example.com       # Forcer refresh zone esclave

# Gestion cache
rndc flush                     # Vider cache complet
rndc flush example.com         # Vider cache zone
rndc flushname www.example.com # Vider entrée spécifique

# Gestion zones
rndc freeze example.com        # Geler zone (édition manuelle)
rndc unfreeze example.com      # Dégeler zone
rndc sync example.com          # Synchroniser sur disque

# Logs et debug
rndc querylog on               # Activer log requêtes
rndc trace 3                   # Niveau debug (0-10)
```

### Vérification Configuration
```bash
# Test syntaxe
named-checkconf                           # Configuration générale
named-checkconf /etc/bind/named.conf.local # Fichier spécifique

# Test zones
named-checkzone example.com /etc/bind/zones/db.example.com
named-checkzone 1.168.192.in-addr.arpa /etc/bind/zones/db.192.168.1

# Test résolution
dig @localhost example.com               # Test local
dig @localhost -x 192.168.1.10          # Test reverse
nslookup www.example.com localhost       # Alternative nslookup
```

## 🔒 Sécurité DNS

### Authentification TSIG
```bash
# Génération clé
tsig-keygen -a HMAC-SHA256 transfer-key > /etc/bind/keys/transfer-key.conf

# Utilisation
key "transfer-key" {
    algorithm hmac-sha256;
    secret "generated-secret-here";
};

# Application
allow-transfer { key "transfer-key"; };
```

### DNSSEC (Domain Name System Security Extensions)
```bash
# Signature zone automatique
dnssec-policy "default" {
    keys {
        ksk lifetime unlimited algorithm ecdsap256sha256;
        zsk lifetime 30d algorithm ecdsap256sha256;
    };
};

zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
    dnssec-policy "default";
    inline-signing yes;
};
```

### ACL & Restrictions
```bash
# Définition ACL
acl "admin-hosts" {
    192.168.1.5;
    192.168.1.10;
};

acl "internal-nets" {
    192.168.0.0/16;
    10.0.0.0/8;
    127.0.0.0/8;
};

# Application restrictions
options {
    allow-query { internal-nets; };
    allow-recursion { internal-nets; };
    allow-transfer { admin-hosts; };
    blackhole { 0.0.0.0/0; };  # Bloquer sources
};
```

## 📈 Performance & Optimisation

### Tuning Cache
```bash
options {
    max-cache-size 512m;           # Taille cache max
    max-cache-ttl 86400;           # TTL cache max
    max-negative-cache-ttl 3600;   # Cache négatif
    cleaning-interval 60;          # Nettoyage cache (min)
    
    # Limites requêtes
    recursive-clients 1000;
    tcp-clients 100;
    max-clients-per-query 100;
};
```

### Monitoring
```bash
# Statistiques serveur
rndc stats                        # Génère fichier stats
rndc dumpdb -cache               # Dump cache

# Logs spécialisés
logging {
    channel query_log {
        file "/var/log/bind/query.log" versions 3 size 100m;
        severity info;
        print-time yes;
        print-category yes;
    };
    
    category queries { query_log; };
    category security { default_syslog; };
};
```

## 🌐 Délégation & Hiérarchie

### Délégation Sous-Domaine
```bash
# Dans zone parente example.com
subdomain   IN  NS      ns1.subdomain.example.com.
subdomain   IN  NS      ns2.subdomain.example.com.

# Glue records (si NS dans domaine délégué)
ns1.subdomain  IN  A   192.168.2.10
ns2.subdomain  IN  A   192.168.2.11
```

### Configuration Multi-View
```bash
# Vues selon source client
view "internal" {
    match-clients { internal-nets; };
    recursion yes;
    
    zone "example.com" {
        type master;
        file "/etc/bind/zones/internal.db.example.com";
    };
};

view "external" {
    match-clients { any; };
    recursion no;
    
    zone "example.com" {
        type master;
        file "/etc/bind/zones/external.db.example.com";
    };
};
```

## 🔍 Diagnostic & Troubleshooting

### Outils de Test
```bash
# dig (Domain Information Groper)
dig example.com                   # Requête A
dig @8.8.8.8 example.com MX      # Serveur + type spécifique
dig +trace example.com            # Trace résolution complète
dig +short example.com            # Réponse courte
dig -x 192.168.1.10              # Requête inverse

# Tests spécialisés
dig example.com AXFR              # Transfert zone (si autorisé)
dig +dnssec example.com           # Test DNSSEC
dig +tcp example.com              # Forcer TCP
```

### Logs & Debug
```bash
# Journaux système
tail -f /var/log/syslog | grep named
journalctl -u bind9 -f

# Debug niveaux
rndc trace 1                      # Erreurs + importantes
rndc trace 3                      # Requêtes + réponses
rndc trace 10                     # Debug complet

# Fichiers debug
ls -la /var/cache/bind/           # Zones + cache dumps
```

### Problèmes Courants
| Symptôme | Cause Probable | Solution |
|----------|----------------|----------|
| **Zone transfer failed** | ACL restrictives | Vérifier `allow-transfer` |
| **SERVFAIL** | Configuration zone | `named-checkzone` |
| **NXDOMAIN** | Enregistrement manquant | Vérifier fichier zone |
| **Timeout** | Firewall/réseau | Tester ports 53 UDP/TCP |
| **Cache poisoning** | Sécurité faible | Activer DNSSEC |

## 💡 Bonnes Pratiques

### Configuration Production
- ✅ **Serveurs multiples** : Maître + au moins 1 esclave
- ✅ **DNSSEC** : Activer pour zones publiques
- ✅ **Monitoring** : Alertes sur pannes/performances
- ✅ **Sauvegarde** : Zones + clés DNSSEC
- ✅ **ACL restrictives** : Limiter transferts/requêtes
- ✅ **Logs** : Archivage et analyse sécurité

### Maintenance Régulière
- ✅ **Serial** : Incrémenter à chaque modification
- ✅ **TTL** : Réduire avant modifications importantes
- ✅ **Cache** : Purger après changements critiques
- ✅ **Certificats** : Renouvellement clés TSIG/DNSSEC
- ✅ **Tests** : Validation résolution interne/externe

### Checklist Déploiement
- [ ] Configuration syntaxiquement correcte
- [ ] Zones validées avec named-checkzone
- [ ] Tests résolution depuis clients internes
- [ ] Tests résolution depuis Internet (si public)
- [ ] Transferts zones fonctionnels
- [ ] Logs configurés et fonctionnels
- [ ] Monitoring en place
- [ ] Documentation à jour

---
**💡 Memo** : Incrémenter serial à chaque modif, `rndc reload` après changements, `dig +trace` pour debug résolution !
