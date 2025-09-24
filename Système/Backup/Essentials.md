# 💾 Sauvegarde & Restauration — Aide-mémoire

## 🎯 Stratégies de Sauvegarde

### Règle 3-2-1-1-0 (Gold Standard)
```
3 copies des données
2 types de supports différents
1 copie hors site (offsite)
1 copie isolée/immutable (air-gap)
0 erreur lors des tests de restauration
```

### Types de Sauvegarde
| Type | Description | Avantage | Inconvénient |
|------|-------------|----------|--------------|
| **Complète** | Tous les fichiers | Restauration simple | Temps/espace important |
| **Incrémentale** | Modifiés depuis dernière sauvegarde | Rapide, économe | Restauration complexe |
| **Différentielle** | Modifiés depuis complète | Compromis équilibré | Croissance dans le temps |
| **Synthétique** | Complète virtuelle | Évite impact production | Traitement serveur backup |

### Méthodes d'Exécution
| Méthode | Description | Usage | Contraintes |
|---------|-------------|-------|-------------|
| **Hot/Online** | En cours d'utilisation | 24/7 disponibilité | Complexité VSS/snapshots |
| **Cold/Offline** | Service arrêté | Cohérence garantie | Interruption service |
| **Snapshot** | Cliché instantané (VSS) | Cohérence applicative | Espace disque temporaire |

## 🏗️ Architecture & Composants

### Chaîne de Sauvegarde
```
Source → Agent → Média Server → Storage
  ↓        ↓         ↓           ↓
Données  Capture  Orchestration Stockage
```

### Types de Stockage
| Type | Protocole | Usage | Avantages | Inconvénients |
|------|-----------|-------|-----------|---------------|
| **DAS** | SCSI/SATA | Local direct | Simple, rapide | Pas de partage |
| **NAS** | SMB/NFS | Partage fichiers | Facile, économique | Limitation réseau |
| **SAN** | iSCSI/FC | Bloc haute perf | Performance | Complexité, coût |
| **Cloud** | HTTPS/S3 | Offsite/archivage | Élasticité | Latence, dépendance |
| **Tape** | LTO/LTFS | Archivage long terme | Durabilité, air-gap | Accès séquentiel |

### Classes de Stockage
- **Online** : Accès immédiat (disque, SSD)
- **Near-line** : Accès rapide (disque SATA 7200rpm)
- **Offline** : Supports déconnectés (bandes, RDX)
- **Archival** : Long terme (cloud glacier, bandes LTO)

## 📅 Planification & Rétention

### Rotation GFS (Grandfather-Father-Son)
```
Quotidien (Son) 👶:
├── Lun, Mar, Mer, Jeu (recyclage 4 jours)
└── Utilisé pour restaurations récentes

Hebdomadaire (Father) 👨:
├── Vendredi de chaque semaine
├── Conservé 4-5 semaines
└── Archive mensuelle

Mensuel (Grandfather) 👴:
├── Dernier vendredi du mois
├── Conservé 12 mois ou plus
└── Archive annuelle/légale
```

### Calendrier Type
```
Daily    : Incrémentale (L-V)
Weekly   : Différentielle (Samedi)
Monthly  : Complète (Dernier dimanche)
Yearly   : Archive (1er janvier)
```

## 🛠️ Solutions Logicielles

### Veritas Backup Exec
```
Architecture:
├── Media Server (serveur principal)
├── Remote Agent (clients)
├── Central Admin Server (multi-serveurs)
└── Storage (disk, tape, cloud)

Workflow:
1. Ajouter Storage (disk pool, tape library)
2. Installer Agent sur clients
3. Créer Job de sauvegarde
4. Planifier exécution
5. Monitorer + alertes
```

### Veeam Backup & Replication
```
Fonctionnalités:
├── Image-level backup (VM entières)
├── Application-aware processing (VSS)
├── Instant VM recovery
├── Granular restore (fichiers, AD, Exchange)
├── Replication (synchrone/asynchrone)
└── Backup Copy (duplication vers sites distants)

Types de Jobs:
├── Backup Job : Sauvegarde primaire
├── Backup Copy : Duplication vers repo secondaire  
├── Replication Job : Réplique VM vers autre site
└── SureBackup : Tests automatiques
```

### Solutions Open Source
| Solution | Type | Forces | Usage |
|----------|------|--------|-------|
| **Bacula** | Client/serveur | Entreprise, modulaire | Infrastructure complexe |
| **Amanda** | Network backup | Unix/Linux focus | Environnements mixtes |
| **rsync** | Synchronisation | Simple, efficace | Scripts personnalisés |
| **Duplicity** | Chiffrement | GPG intégré | Données sensibles |

## 💽 Technologies de Stockage

### RAID (Redondance)
| RAID | Configuration | Tolérance | Performance | Usage |
|------|---------------|-----------|-------------|-------|
| **0** | Striping | Aucune | Lecture/écriture élevée | Performance pure |
| **1** | Mirroring | 1 disque | Lecture élevée | Données critiques |
| **5** | Striping + parité | 1 disque | Lecture élevée | Compromis coût/perf |
| **6** | Double parité | 2 disques | Lecture élevée | Sécurité renforcée |
| **10** | Mirror + Stripe | 50% disques | Très élevée | Haute performance |

### LTO (Linear Tape-Open)
| Génération | Capacité Native | Compressée | Débit | Usage |
|------------|-----------------|------------|-------|-------|
| **LTO-6** | 2.5 TB | 6.25 TB | 160 MB/s | Archive standard |
| **LTO-7** | 6 TB | 15 TB | 300 MB/s | Backup quotidien |
| **LTO-8** | 12 TB | 30 TB | 360 MB/s | Haute capacité |
| **LTO-9** | 18 TB | 45 TB | 400 MB/s | Dernière génération |

## 🔐 Sécurité & Conformité

### Chiffrement
```
Données au repos:
├── AES-256 pour disques/bandes
├── Clés gérées par HSM/KMS
└── Chiffrement hardware (auto-encryption)

Données en transit:
├── TLS 1.3 pour transferts réseau
├── SSL/HTTPS pour interfaces web
└── VPN/IPSEC pour sites distants
```

### Authentification & Autorisation
```
Comptes dédiés:
├── Service Account (sauvegarde automatique)
├── Backup Operator (groupe Windows)
├── Moindre privilège (lecture seule + backup rights)
└── Rotation mots de passe régulière

Permissions requises:
├── SeBackupPrivilege (Windows)
├── Volume Shadow Copy access
├── Database backup rights (SQL, Oracle)
└── Application-specific rights
```

### Conformité Réglementaire
- **RGPD** : Droit à l'oubli, pseudonymisation
- **SOX** : Rétention 7 ans, intégrité données
- **HIPAA** : Chiffrement, traçabilité accès
- **PCI-DSS** : Protection données cartes

## 🔄 Application-Aware Backup

### Windows VSS (Volume Shadow Copy)
```
Composants:
├── VSS Service (coordination)
├── VSS Writers (applications)
├── VSS Providers (storage)
└── VSS Requestor (backup software)

Applications supportées:
├── Active Directory
├── SQL Server
├── Exchange Server
├── SharePoint
└── IIS/Registry
```

### Base de Données
| SGBD | Méthode Native | Cohérence | Point-in-time Recovery |
|------|----------------|-----------|------------------------|
| **SQL Server** | T-SQL backup | Transaction log | ✅ |
| **Oracle** | RMAN | Archive logs | ✅ |
| **MySQL** | mysqldump/xtrabackup | Binary logs | ✅ |
| **PostgreSQL** | pg_dump/pg_basebackup | WAL | ✅ |

### Commandes Utiles
```bash
# SQL Server
BACKUP DATABASE [MyDB] TO DISK = 'C:\Backup\MyDB.bak'
BACKUP LOG [MyDB] TO DISK = 'C:\Backup\MyDB_log.trn'

# MySQL
mysqldump -u root -p --single-transaction --routines MyDB > backup.sql
mysql -u root -p MyDB < backup.sql

# Oracle RMAN
RMAN> BACKUP INCREMENTAL LEVEL 0 DATABASE PLUS ARCHIVELOG;
RMAN> RESTORE DATABASE; RECOVER DATABASE;

# PostgreSQL
pg_dump -U postgres -h localhost MyDB > backup.sql
psql -U postgres -h localhost MyDB < backup.sql
```

## 📈 Monitoring & Reporting

### KPIs Essentiels
| Métrique | Objectif | Seuil Alerte |
|----------|----------|--------------|
| **Taux de succès** | >99% | <95% |
| **RTO** | <4h critique | >RTO défini |
| **RPO** | <1h données | >RPO défini |
| **Déduplication** | >50% | <30% |
| **Utilisation stockage** | <80% | >90% |

### Alertes Critiques
- Échec sauvegarde >24h
- Espace stockage <10%
- Erreur restauration test
- Média défaillant
- Service backup arrêté

### Rapports Réguliers
```
Quotidien:
├── Jobs failed/succeeded
├── Alertes critiques
└── Utilisation stockage

Hebdomadaire:
├── Synthèse succès/échecs
├── Performance trends
└── Tests restauration

Mensuel:
├── SLA compliance
├── Coûts stockage
└── Capacité planning
```

## 🚨 Tests & Restauration

### Types de Tests
| Test | Fréquence | Objectif |
|------|-----------|----------|
| **Restore test** | Mensuel | Vérifier intégrité |
| **Bare metal** | Trimestriel | Test complet |
| **DR drill** | Semestriel | Plan continuité |
| **Application** | Mensuel | Cohérence données |

### Procédure de Restauration
```
1. Évaluation
   ├── Identifier cause (corruption, suppression, attaque)
   ├── Déterminer point de restauration
   └── Évaluer impact business

2. Préparation
   ├── Isoler environnement (si nécessaire)
   ├── Préparer infrastructure cible
   └── Notifier équipes

3. Restauration
   ├── Restaurer données/système
   ├── Vérifier intégrité
   └── Tester fonctionnalités

4. Validation
   ├── Tests applicatifs
   ├── Validation utilisateurs
   └── Documentation incident
```

### Restauration par Type
```
Fichiers:
├── Restauration sélective
├── Emplacement alternatif (recommandé)
└── Vérification permissions

VM complète:
├── Instant recovery (Veeam)
├── Restauration vers nouveau datastore
└── Tests avant mise en production

Bare Metal:
├── Boot depuis média rescue
├── Restauration image système
└── Reconfiguration réseau/drivers

Active Directory:
├── Corbeille AD (objets simples)
├── Restauration authoritative
└── Non-authoritative (DC complet)
```

## 💡 Bonnes Pratiques

### Architecture
- ✅ **Séparation** : Domaine backup distinct
- ✅ **Redondance** : Pas de SPOF (Single Point of Failure)
- ✅ **Réseau dédié** : VLAN backup, MTU 9000
- ✅ **Monitoring** : Supervision continue
- ✅ **Documentation** : Procédures à jour

### Sécurité
- ✅ **Comptes dédiés** : Principe moindre privilège
- ✅ **Chiffrement** : End-to-end encryption
- ✅ **Air-gap** : Copie isolée (ransomware)
- ✅ **Tests** : Validation régulière
- ✅ **Logs** : Traçabilité complète

### Opérationnel
- ✅ **Automatisation** : Jobs planifiés
- ✅ **Alertes** : Notifications ciblées
- ✅ **Tests** : Restauration mensuelle
- ✅ **Documentation** : Runbooks détaillés
- ✅ **Formation** : Équipes à jour

### Économique
- ✅ **Déduplication** : Optimisation stockage
- ✅ **Compression** : Réduction volume
- ✅ **Tiering** : Hot/cold storage
- ✅ **Cloud hybride** : Coûts optimisés
- ✅ **Planification** : Croissance anticipée

## 🎯 Cas d'Usage Spécifiques

### PME (10-50 utilisateurs)
```
Solution recommandée:
├── NAS local (sauvegarde quotidienne)
├── Cloud backup (offsite)
├── USB/RDX (rotation mensuelle)
└── Veeam Essentials ou Backup Exec

Budget type: 5-15K€/an
```

### Entreprise (500-5000 utilisateurs)
```
Solution recommandée:
├── SAN principal + SAN DR
├── Bibliothèque bandes LTO
├── Cloud tier (archivage)
├── Veeam B&R Enterprise ou NetBackup
└── Équipe dédiée

Budget type: 50-200K€/an
```

### Environnement Critique
```
Exigences:
├── RTO < 1h
├── RPO < 15min
├── 99.99% disponibilité
├── Conformité réglementaire
└── Tests automatisés

Solutions:
├── Réplication synchrone
├── Cluster haute disponibilité
├── Sites multiples
└── Monitoring 24/7
```

---
**💡 Memo** : 3-2-1-1-0 + tests réguliers + documentation = sauvegarde réussie !
