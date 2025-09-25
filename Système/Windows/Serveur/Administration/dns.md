# 🌐 DNS Server — Aide-mémoire

## 🏗️ Architecture DNS

### Hiérarchie & Délégation
```
Root (.) 
├── .com (TLD)
│   └── google.com (2nd level)
│       └── www.google.com (3rd level)
├── .org
└── .fr
    └── gouv.fr
        └── service-public.fr
```

### Résolution Récursive vs Itérative
| Type | Description | Usage | Exemple |
|------|-------------|-------|---------|
| **Récursive** | Résolveur fait le travail complet | Client → DNS local | "Donne-moi l'IP de google.com" |
| **Itérative** | Chaque serveur renvoie vers suivant | DNS → DNS autoritaires | "Je ne connais pas, demande à..." |

### Processus Résolution Complète
```
1. Client → Cache local
2. Si absent → Fichier hosts  
3. Si absent → DNS configuré (récursif)
4. DNS local → Root servers (itératif)
5. Root → TLD servers (.com)
6. TLD → Authoritative servers (google.com)
7. Retour avec réponse + mise en cache
```

## 🗂️ Types d'Enregistrements

### Enregistrements Principaux
| Type | Description | Exemple | Usage |
|------|-------------|---------|--------|
| **A** | IPv4 | `www IN A 192.168.1.10` | Site web, service |
| **AAAA** | IPv6 | `www IN AAAA 2001:db8::1` | IPv6 |
| **CNAME** | Alias canonique | `ftp IN CNAME www` | Redirection |
| **MX** | Mail exchange | `@ IN MX 10 mail.domain.com` | Email |
| **NS** | Name server | `@ IN NS ns1.domain.com` | Délégation |
| **PTR** | Reverse lookup | `10 IN PTR www.domain.com` | IP → nom |
| **TXT** | Texte libre | `@ IN TXT "v=spf1 include:..."` | SPF, DKIM |
| **SRV** | Service | `_http._tcp IN SRV 0 5 80 www` | Autodiscovery |

### Enregistrements Spéciaux
```
SOA (Start of Authority) :
├── Primary NS : ns1.domain.com
├── Admin email : admin.domain.com
├── Serial : 2024011501 (YYYYMMDDNN)
├── Refresh : 3600s (1h)
├── Retry : 600s (10min)  
├── Expire : 86400s (24h)
└── Minimum TTL : 300s (5min)

CNAME Restrictions :
- Pas d'autres enregistrements sur même nom
- Pas de CNAME sur apex domain (@)
- CNAME ne peut pointer vers CNAME (chaînage)
```

## 🏢 Types de Zones

### Zones Principales
| Type Zone | Description | Modifications | Usage |
|-----------|-------------|---------------|--------|
| **Primaire** | Zone maître | ✅ Lecture/écriture | Serveur principal |
| **Secondaire** | Zone esclave | ❌ Lecture seule | Redondance, performance |
| **Stub** | NSS seulement | ❌ Transferts NS | Délégation |
| **Conditionnelle** | Forward spécifique | ❌ Configuration | Split DNS |

### Zones Intégrées AD (Windows)
```
Avantages Zones AD :
├── Réplication via AD (pas AXFR/IXFR)
├── Multi-maître (tous DC R/W)  
├── Sécurité intégrée Kerberos
├── Updates dynamiques sécurisées
└── Pas de fichiers .dns locaux
```

## 🔄 Transferts de Zone

### Types de Transferts
| Type | Description | Bande Passante | Usage |
|------|-------------|----------------|--------|
| **AXFR** | Transfert complet | Élevée | Initial, corruption |
| **IXFR** | Transfert incrémental | Faible | Mises à jour |
| **NOTIFY** | Notification push | Minimale | Réactivité |

### Configuration Transferts
```cmd
# Autoriser transferts (DNS Manager)
Zone Properties > Zone Transfers:
├── To any server (⚠️ risque sécurité)
├── Only to servers listed on Name Servers tab
├── Only to following servers: [IP list]
└── Notify: Automatic/Manual list

# Commandes PowerShell
Add-DnsServerZoneTransferPolicy -ZoneName "domain.com" -ClientSubnet "192.168.1.0/24"
Set-DnsServerZoneTransferPolicy -ZoneName "domain.com" -Action Allow
```

## 🔍 Requêtes & Résolution

### Types de Requêtes DNS
```
Standard Query (A, AAAA, MX...) :
├── Header : ID, Flags, Counts
├── Question : QNAME, QTYPE, QCLASS  
├── Answer : RR matching query
├── Authority : NS records
└── Additional : Glue records

Inverse Query (PTR) :
├── IP → Nom : 10.1.168.192.in-addr.arpa
├── IPv6 → Nom : 1.0.0.0...ip6.arpa
└── Usage : logs, security, email
```

### Codes de Réponse
| Code | Nom | Signification |
|------|-----|---------------|
| **0** | NOERROR | Succès |
| **1** | FORMERR | Format error |
| **2** | SERVFAIL | Server failure |
| **3** | NXDOMAIN | Domain not exist |
| **4** | NOTIMP | Not implemented |
| **5** | REFUSED | Query refused |

## ⚙️ Configuration Windows DNS

### Serveur DNS (Windows Server)
```cmd
# Installation rôle
Install-WindowsFeature DNS -IncludeManagementTools

# Configuration via PowerShell
Add-DnsServerPrimaryZone -Name "contoso.local" -ZoneFile "contoso.local.dns"
Add-DnsServerResourceRecordA -ZoneName "contoso.local" -Name "www" -IPv4Address "192.168.1.10"
Add-DnsServerResourceRecordCName -ZoneName "contoso.local" -Name "ftp" -HostNameAlias "www.contoso.local"

# Zone reverse
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" -ZoneFile "1.168.192.in-addr.arpa.dns"
Add-DnsServerResourceRecordPtr -ZoneName "1.168.192.in-addr.arpa" -Name "10" -PtrDomainName "www.contoso.local"
```

### Redirecteurs (Forwarders)
```cmd
# Configuration redirecteurs
Set-DnsServerForwarder -IPAddress 8.8.8.8,1.1.1.1 -UseRootHint $false

# Redirecteurs conditionnels
Add-DnsServerConditionalForwarderZone -Name "external.com" -MasterServers 10.0.0.1,10.0.0.2
```

### Mises à Jour Dynamiques
```
Zone Properties > General:
├── None : Pas de MAJ dynamiques
├── Nonsecure and secure : Tous clients
└── Secure only : Authentification requise (AD)

Sécurisé (recommandé) :
- Authentification Kerberos
- Comptes ordinateurs AD
- Service NetLogon
```

## 🖥️ Outils de Diagnostic

### Commandes Windows
```cmd
# Tests résolution
nslookup google.com              # Basique
nslookup google.com 8.8.8.8      # Serveur spécifique
nslookup -type=MX google.com     # Type d'enregistrement

# Mode interactif nslookup
> set type=A                     # Type A records
> set type=ANY                   # Tous types
> ls -d domain.com               # Lister zone (si autorisé)
> server 8.8.8.8                 # Changer serveur

# PowerShell moderne
Resolve-DnsName google.com -Type A
Resolve-DnsName google.com -Server 1.1.1.1
Test-DnsServer -IPAddress 192.168.1.1 -ZoneName contoso.local
```

### Outils Linux (dig)
```bash
# dig - Outil avancé
dig google.com                   # A record
dig @8.8.8.8 google.com         # Serveur spécifique  
dig google.com MX               # Mail exchangers
dig +trace google.com           # Trace complète
dig +short google.com           # Réponse courte
dig -x 8.8.8.8                 # Reverse lookup

# Tests de zone
dig @ns1.domain.com domain.com AXFR  # Transfert zone (si autorisé)
```

### Analyse Fichiers Logs
```
Windows DNS Logs :
├── %SystemRoot%\System32\dns\dns.log
├── Event Viewer > DNS Server
└── Debug logging (DNS Manager)

Types d'événements :
├── Queries (requêtes reçues)
├── Responses (réponses envoyées)  
├── Updates (mises à jour dynamiques)
└── Zone transfers (transferts)
```

## 🔐 Sécurité DNS

### DNSSEC (DNS Security Extensions)
```
Concept DNSSEC :
├── Signature cryptographique des enregistrements
├── Chaîne de confiance depuis root
├── Détection falsification/empoisonnement  
└── Nécessite support client/serveur

Types d'enregistrements DNSSEC :
├── RRSIG : Signature des RR
├── DNSKEY : Clé publique zone
├── DS : Delegation Signer (parent)  
└── NSEC/NSEC3 : Proof of non-existence
```

### Sécurisation Serveur
```cmd
# Restrictions accès (DNS Manager)
Server Properties > Security:
├── Restrict zone transfers
├── Secure cache against pollution
├── Disable recursion for external queries
└── Enable BIND secondaries compatibility

# ACL par zone
Zone Properties > Zone Transfers:
├── Deny transfers (par défaut)
├── Allow to specific servers only
└── Notify configuration
```

## 📊 Monitoring & Performance

### Métriques Importantes
| Métrique | Description | Seuil Normal |
|----------|-------------|--------------|
| **Query Rate** | Requêtes/seconde | < 1000/s (standard) |
| **Cache Hit Ratio** | % réponses cache | > 80% |
| **Recursive Queries** | Requêtes récursives | Variable selon rôle |
| **Zone Transfer** | Transferts réussis | 100% |

### Optimisation Cache
```cmd
# Configuration cache DNS
Set-DnsServerCache -MaxKBSize 10240 -MaxNegativeTtl 00:15:00 -MaxTtl 1:00:00

# Vider cache serveur  
Clear-DnsServerCache -Force

# Statistiques cache
Get-DnsServerStatistics | Select-Object CacheStatistics
```

### Surveillance Zone
```powershell
# Vérifier santé zones
Get-DnsServerZone | Where-Object {$_.IsAutoCreated -eq $false}
Get-DnsServerZoneTransferPolicy
Test-DnsServer -IPAddress localhost -ZoneName "contoso.local"

# Audit enregistrements
Get-DnsServerResourceRecord -ZoneName "contoso.local" | 
Where-Object {$_.TimeStamp -lt (Get-Date).AddDays(-30)}
```

## 🚨 Troubleshooting Courant

### Problèmes Fréquents
| Symptôme | Cause Probable | Diagnostic | Solution |
|----------|---------------|------------|----------|
| **Résolution lente** | DNS distant, timeout | `nslookup`, temps réponse | Forwarders locaux |
| **NXDOMAIN** | Enregistrement manquant | Vérifier zone | Ajouter RR |
| **Zone transfer failed** | Permissions, firewall | Event logs, ACL | Configurer transferts |
| **Updates refused** | Sécurité AD | Secure updates | Permissions ordinateur |

### Commandes Debug
```cmd
# Debug logging (attention performance)
Set-DnsServerDiagnostics -All $true
Set-DnsServerDiagnostics -LogFilePath "C:\dns-debug.log"

# Tests connectivité  
Test-NetConnection dc01 -Port 53
telnet dc01 53

# Vérifier configuration
Get-DnsClientServerAddress
Get-DnsClient | Select-Object InterfaceAlias, ConnectionSpecificSuffix
```

## 🔧 Configurations Spécialisées

### Split DNS (Split-Brain)
```
Concept :
├── Zone interne : contoso.local (192.168.x.x)
├── Zone externe : contoso.com (IP publiques)
├── Même domaine, enregistrements différents
└── Selon source requête (interne/externe)

Implémentation :
├── DNS interne : Vue réseau privé
├── DNS externe : Vue Internet
└── Conditional forwarders si mixte
```

### DNS Round Robin
```cmd
# Plusieurs A records même nom
Add-DnsServerResourceRecordA -ZoneName "contoso.local" -Name "www" -IPv4Address "192.168.1.10"
Add-DnsServerResourceRecordA -ZoneName "contoso.local" -Name "www" -IPv4Address "192.168.1.11"

# Équilibrage automatique (rotation réponses)
Set-DnsServerGlobalNameZone -Enable $false -PassThru
```

### Zones Stub
```cmd
# Zone stub (délégation)
Add-DnsServerStubZone -Name "branch.contoso.local" -MasterServers 192.168.2.10
# Contient seulement NS records de la zone déléguée
```

## 💡 Bonnes Pratiques

### Architecture
- ✅ **Redondance** : Minimum 2 serveurs DNS
- ✅ **Séparation rôles** : DNS autoritaire ≠ récursif
- ✅ **Zones intégrées AD** pour environnements Windows
- ✅ **Forwarders externes** (1.1.1.1, 8.8.8.8)
- ✅ **Zones reverse** pour tous réseaux

### Sécurité  
- ✅ **Restriction transferts** zones sensibles
- ✅ **Updates sécurisées** uniquement (AD)
- ✅ **Monitoring** requêtes inhabituelles
- ✅ **Firewall** : Port 53 TCP/UDP contrôlé
- ✅ **Cache poisoning** protection activée

### Maintenance
- ✅ **TTL adaptés** : Courts pour changes, longs pour stabilité
- ✅ **Serial numbers** incrémentés à chaque modification
- ✅ **Sauvegarde zones** avant changements
- ✅ **Tests résolution** après modifications
- ✅ **Documentation** architecture DNS

---
**💡 Memo** : Port 53 UDP/TCP, hiérarchie Root→TLD→Domain, AXFR complet vs IXFR incrémental, cache local d'abord !
