# 🩺 Active Directory — Diagnostics & Commandes — Aide-mémoire

## 🏗️ Architecture & Concepts AD

### Composants Principaux
```
Forêt (Forest)
├── Domaine racine (Root Domain)
│   ├── Unités d'organisation (OU)
│   ├── Utilisateurs & Groupes
│   └── GPO (Group Policy Objects)
├── Domaines enfants (Child Domains)
└── Domaines externes (External Trusts)

Services AD:
├── AD DS (Domain Services) - Annuaire principal
├── AD LDS (Lightweight DS) - Instance légère
├── AD CS (Certificate Services) - PKI
├── AD FS (Federation Services) - SSO
└── AD RMS (Rights Management) - Protection documents
```

### Rôles FSMO (Flexible Single Master Operations)
| Rôle | Portée | Fonction | Commande Vérification |
|------|--------|----------|----------------------|
| **Schema Master** | Forêt | Modifications schéma | `netdom query fsmo` |
| **Domain Naming Master** | Forêt | Ajout/suppression domaines | `netdom query fsmo` |
| **PDC Emulator** | Domaine | Synchronisation temps, mot de passe | `netdom query pdc` |
| **RID Master** | Domaine | Attribution pools RID | `netdom query fsmo` |
| **Infrastructure Master** | Domaine | Références inter-domaines | `netdom query fsmo` |

## 🔍 Outils Diagnostics Essentiels

### DCDIAG - Diagnostic Contrôleurs
```cmd
# Tests complets santé DC
dcdiag /v                           # Verbose sur DC local
dcdiag /s:DC01 /v                   # DC spécifique
dcdiag /e /v                        # Tous DCs domaine (enterprise)
dcdiag /a /v                        # Tous DCs forêt (all)

# Tests spécifiques critiques
dcdiag /test:dns                    # DNS et réplication
dcdiag /test:replications           # Réplication uniquement
dcdiag /test:fsmocheck             # Rôles FSMO
dcdiag /test:ridmanager            # Gestion RID
dcdiag /test:machineaccount        # Compte machine DC

# Options utiles
dcdiag /fix                        # Tentative réparation auto
dcdiag /c /v                       # Tests compréhensifs
dcdiag /skip:systemlog             # Ignorer journaux système
```

### REPADMIN - Réplication AD
```cmd
# État réplication
repadmin /showrepl                  # Partenaires réplication DC local
repadmin /showrepl *                # Tous DCs du site
repadmin /showrepl /all /verbose    # Détails complets tous DCs

# Forcer réplication
repadmin /syncall                   # Synchroniser toutes partitions
repadmin /syncall /AdeP             # Sync + ignorer erreurs + cross-site
repadmin /replicate DC02 DC01 "DC=domain,DC=com"  # Réplication ciblée

# Diagnostics réplication
repadmin /showutdvec DC01 "DC=domain,DC=com"       # Vecteur mise à jour
repadmin /showobjmeta DC01 "CN=user,CN=Users,DC=domain,DC=com"  # Métadonnées objet
repadmin /showattr DC01 "DC=domain,DC=com" /atts:msDS-ReplAuthenticationMode

# Export/Monitoring
repadmin /showrepl * /csv > replication_status.csv  # Export CSV
repadmin /kcc                       # Forcer KCC (Knowledge Consistency Checker)
```

### NETDOM - Gestion Domaines/Trusts
```cmd
# Informations domaine
netdom query pdc                    # PDC Emulator du domaine
netdom query fsmo                   # Tous rôles FSMO
netdom query dc                     # Contrôleurs domaine
netdom query trust                  # Relations d'approbation
netdom query workstation            # Stations de travail

# Tests connectivité
netdom query /domain:child.domain.com dc    # DCs domaine enfant
netdom verify computername          # Vérifier enregistrement DNS
```

## 🌐 DNS & Résolution Noms

### Diagnostics DNS AD
```cmd
# Tests DNS spécifiques AD
nslookup                           # Mode interactif
> set type=SRV
> _ldap._tcp.domain.com            # Enregistrements LDAP
> _kerberos._tcp.domain.com        # Enregistrements Kerberos
> _gc._tcp.domain.com              # Catalogues globaux

# PowerShell DNS
Resolve-DnsName _ldap._tcp.domain.com -Type SRV
Test-DnsServer -IPAddress 192.168.1.10 -ZoneName domain.com

# Validation enregistrements critiques
dcdiag /test:dns /DnsBasic         # Tests DNS de base
dcdiag /test:dns /DnsForwarders    # Redirecteurs DNS
dcdiag /test:dns /DnsDelegation    # Délégations DNS
```

### Enregistrements DNS Critiques AD
| Enregistrement | Type | Usage | Vérification |
|----------------|------|-------|--------------|
| `_ldap._tcp.domain.com` | SRV | Services LDAP | Port 389 |
| `_kerberos._tcp.domain.com` | SRV | Authentification | Port 88 |
| `_gc._tcp.domain.com` | SRV | Catalogue global | Port 3268 |
| `domain.com` | A | Résolution domaine | IP du DC |
| `_msdcs.domain.com` | NS | Services AD | Sous-domaine |

### Réparation DNS AD
```cmd
# Re-enregistrement DNS
ipconfig /registerdns              # Client
dnscmd /zonerefresh domain.com     # Serveur DNS

# Nettoyage/reconstruction
dnscmd DC01 /zonerefresh domain.com
net stop netlogon && net start netlogon
```

## 👥 Utilisateurs & Groupes

### Informations Identité
```cmd
# Identification complète
whoami /all                        # Utilisateur + groupes + privilèges
whoami /groups                     # Groupes seulement
whoami /priv                       # Privilèges seulement
whoami /user                       # SID utilisateur

# Informations compte
net user username                  # Détails compte local
net user username /domain          # Détails compte domaine
net user /domain                   # Tous comptes domaine
```

### Gestion Comptes PowerShell
```powershell
# Module Active Directory
Import-Module ActiveDirectory

# Informations utilisateurs
Get-ADUser -Identity "jdoe" -Properties *
Get-ADUser -Filter {Enabled -eq $false} | Select Name, LastLogonDate
Get-ADUser -Filter {PasswordNeverExpires -eq $true}

# Informations groupes
Get-ADGroup -Identity "Domain Admins" -Properties Members
Get-ADGroupMember -Identity "Domain Admins"
Get-ADPrincipalGroupMembership -Identity "jdoe"

# Tests connexion
Test-ADServiceAccount -Identity "serviceaccount"
Get-ADDomain | Select PDCEmulator, RIDMaster, InfrastructureMaster
```

## 🔐 Authentification & Kerberos

### Diagnostics Kerberos
```cmd
# Cache tickets Kerberos
klist                              # Tickets utilisateur courant
klist tickets                      # Détails complets tickets
klist tgt                          # Ticket Granting Ticket
klist purge                        # Purger cache tickets

# Tests authentification
runas /user:domain\admin cmd       # Test avec autres credentials
kinit user@DOMAIN.COM              # Obtenir ticket (si MIT Kerberos)
```

### Résolution Problèmes Auth
```cmd
# Synchronisation temps (critique Kerberos)
w32tm /query /status               # État service temps
w32tm /resync                      # Forcer synchronisation
w32tm /query /peers                # Sources temps configurées

# Tests comptes services
setspn -L serviceaccount           # Lister SPNs compte
setspn -Q HTTP/server.domain.com   # Rechercher SPN spécifique
```

## 📊 Performance & Monitoring

### Compteurs Performance AD
```cmd
# PerfMon compteurs critiques AD
# NTDS\DRA Inbound Values (Total)/sec - Réplication entrante
# NTDS\DRA Outbound Values (Total)/sec - Réplication sortante  
# NTDS\LDAP Searches/sec - Requêtes LDAP
# NTDS\LDAP Binds/sec - Connexions LDAP

# Via TypePerf (ligne commande)
typeperf "\NTDS\LDAP Searches/sec" -sc 10
typeperf "\NTDS\DRA Inbound Values (Total)/sec" -sc 20
```

### Journaux Événements AD
| Log | Event IDs Critiques | Description |
|-----|-------------------|-------------|
| **Directory Service** | 1844, 2042 | Réplication, problèmes FSMO |
| **Security** | 4624, 4625, 4768 | Authentifications, Kerberos |
| **System** | 5805, 5807 | DNS, services AD |
| **DNS Server** | 4013, 4015 | Problèmes DNS/AD |

```cmd
# Consultation événements
wevtutil qe "Directory Service" /c:50 /rd:true /f:text
Get-WinEvent -LogName "Directory Service" -MaxEvents 20 | Format-Table TimeCreated, Id, LevelDisplayName, Message
```

## 🔧 Maintenance & Réparation

### Nettoyage Base AD
```cmd
# Nettoyage métadonnées (objets orphelins)
ntdsutil                           # Outil interactif
> metadata cleanup
> connections
> bind to server DC01
> quit
> select operation target
> list domains
> select domain 0
> list sites
> select site 0
> list servers in site
> select server <number>
> quit
> remove selected server

# Alternative PowerShell
Remove-ADObject -Identity "CN=OldDC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=domain,DC=com" -Confirm:$false
```

### Vérification Intégrité
```cmd
# Vérification base NTDS
ntdsutil "activate instance ntds" "files" "integrity" quit
ntdsutil "activate instance ntds" "semantic database analysis" "go" quit

# Réparation database (mode sans échec)
ntdsutil "activate instance ntds" "files" "recover" quit
```

### Sauvegarde État Système
```cmd
# Windows Server Backup
wbadmin start systemstatebackup -backuptarget:E:
wbadmin get versions -backuptarget:E:
wbadmin start systemstaterecovery -version:01/15/2024-10:30 -backuptarget:E:

# Via PowerShell
Checkpoint-Computer -Description "Before AD Changes"
```

## 🚨 Résolution Problèmes Courants

### Problèmes Réplication
| Symptôme | Cause Probable | Diagnostic | Solution |
|----------|---------------|------------|----------|
| **Réplication lente** | Bande passante/latence | `repadmin /showrepl` | Optimiser liens sites |
| **Erreur 8524** | Connexions réseau | `dcdiag /test:connectivity` | Vérifier firewall/DNS |
| **Erreur 1722** | RPC endpoint mapper | `netstat -an \| find :135` | Services RPC |
| **Objets orphelins** | DC supprimé incorrectement | `repadmin /showutdvec` | Nettoyage métadonnées |

### Problèmes DNS/Authentification
| Symptôme | Diagnostic | Solution |
|----------|------------|----------|
| **Échec authentification** | `klist`, `dcdiag /test:kerberos` | Sync temps, DNS |
| **SPN manquants** | `setspn -L account` | `setspn -A HTTP/server account` |
| **Résolution DNS lente** | `nslookup`, `dcdiag /test:dns` | Redirecteurs, cache DNS |

### Commandes Urgence
```cmd
# Redémarrage services AD (précaution)
net stop "Active Directory Domain Services" /y
net start "Active Directory Domain Services"

# Forcer réplication immédiate
repadmin /syncall /force

# Reset trust domaine (en dernier recours)
netdom resetpwd /server:DC01 /userd:domain\admin /passwordd:*
```

## 🛠️ PowerShell AD Avancé

### Module ActiveDirectory
```powershell
# Installation module (Windows 10/Server)
Install-WindowsFeature -Name "RSAT-AD-PowerShell"
Import-Module ActiveDirectory

# Requêtes LDAP avancées
Get-ADUser -LDAPFilter "(department=IT)" -Properties Department, Title
Get-ADComputer -Filter {OperatingSystem -like "*Server*"} -Properties OperatingSystem

# Recherche dans toute la forêt  
Get-ADUser -Identity "jdoe" -Server "GC.domain.com:3268"
Get-ADForest | Select SchemaMaster, DomainNamingMaster

# Gestion en lot
Get-ADUser -Filter {City -eq "Paris"} | Set-ADUser -Department "IT-Paris"
```

## 📋 Checklist Santé AD

### Vérifications Quotidiennes
- [ ] `dcdiag /e /v` sans erreurs critiques
- [ ] `repadmin /showrepl` - réplication < 15min
- [ ] Journaux Directory Service sans erreurs 1844/2042
- [ ] Services AD démarrés sur tous DCs
- [ ] Synchronisation temps < 5min écart

### Vérifications Hebdomadaires  
- [ ] `dcdiag /test:dns` complet
- [ ] Espace disque database NTDS (C:\Windows\NTDS)
- [ ] Sauvegarde état système réussie
- [ ] Analyse performance (LDAP queries/sec)
- [ ] Vérification trusts externes

### Vérifications Mensuelles
- [ ] Nettoyage objets orphelins (`ntdsutil metadata cleanup`)
- [ ] Vérification intégrité database (`ntdsutil integrity`)
- [ ] Audit comptes privilégiés inactifs
- [ ] Test restauration d'état système
- [ ] Documentation changements infrastructure

---
**💡 Memo** : `dcdiag /e /v` pour santé globale, `repadmin /showrepl` pour réplication, toujours sauvegarder avant modifications !
