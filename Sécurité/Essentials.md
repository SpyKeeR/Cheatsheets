# 🛡️ Cybersécurité — Cheatsheet Essentials

## 🏗️ Architecture & Sécurisation Réseau

### Segmentation & Cloisonnement
- **VLAN/Sous-réseaux** : Isoler par fonction (prod/dev/admin)
- **DMZ** : Zone tampon pour services exposés (web, mail, DNS)
- **Microsegmentation** : Firewall interne entre segments critiques
- **ZTNA** : *Never trust, always verify* → accès minimal + vérification contexte

### Authentification & Accès
- **802.1X + RADIUS** : Contrôle d'accès réseau (BYOD)
- **VPN + MFA** : Accès distants sécurisés
- **Bastion/Jump Server** : Point d'accès admin contrôlé + journalisé
- **RBAC** : Principe du moindre privilège

### Wi-Fi Sécurisé
- **Protocoles** : WPA3 > WPA2-Enterprise > WPA2-PSK
- **Chiffrement** : CCMP (AES), éviter TKIP
- **Clés** : >15 caractères, rotation périodique
- **Isolation client** : Empêcher communication inter-clients

## 🔐 Gestion Identités & Accès

### Politique Mots de Passe
| Critère | Standard | Admin |
|---------|----------|-------|
| Longueur min | 8 chars | 12 chars |
| Complexité | 3/4 types | 4/4 types |
| Rotation | 6 mois | 3 mois |
| Historique | 12 derniers | 24 derniers |

### Lifecycle IAM
```
Provision → Modifier → Désactiver → Supprimer
    ↓         ↓          ↓           ↓
  Onboard   Changement  Départ    Purge logs
```

### Multi-Factor Authentication
- **Facteurs** : Connaissance + Possession + Inhérence
- **Solutions** : TOTP, SMS, Push, FIDO2, certificats
- **Obligatoire** : Comptes admin, VPN, services critiques

## 🚨 Détection & Réponse aux Incidents

### Solutions de Détection
| Type | Fonction | Déploiement |
|------|----------|-------------|
| **IDS** | Détection passive → Alerte | Passif (TAP/SPAN) |
| **IPS** | Détection + Blocage inline | Inline (bridge/route) |
| **EDR** | Endpoints + réponse | Agents sur postes |
| **XDR** | Corrélation multi-sources | Central + agents |
| **MDR** | Service managé 24/7 | SOC externe |

### Réponse Incident (NIST)
1. **Préparation** : Plan IR, équipe, outils
2. **Détection** : Identification + classification
3. **Confinement** : Isolation + préservation preuves
4. **Éradication** : Suppression cause racine
5. **Récupération** : Restauration + monitoring renforcé
6. **Leçons** : Post-mortem + amélioration

## 🔒 Cryptographie & PKI

### Algorithmes Recommandés
- **Symétrique** : AES-256, ChaCha20
- **Asymétrique** : RSA-2048+, ECDSA P-256+, Ed25519
- **Hash** : SHA-256+, éviter MD5/SHA-1
- **TLS** : v1.3 > v1.2, cipher suites AEAD

### PKI & Certificats
```
AC Racine → AC Intermédiaire → Certificats finaux
     ↓              ↓               ↓
   Offline      Semi-online     Online/Auto
```
- **Interne** : AD CS + GPO distribution
- **Public** : Let's Encrypt (ACME) + renouvellement auto
- **Révocation** : CRL + OCSP

## 📊 Supervision & Logging

### Journalisation Critique
- ✅ **Authentifications** (succès/échec)
- ✅ **Modifications privilégiées** (comptes, configs)
- ✅ **Accès ressources sensibles**
- ✅ **Trafic réseau** (connexions, dénis)
- ✅ **Modifications système** (fichiers, registre)

### SIEM & Corrélation
```bash
# Formats standards
Syslog (RFC 3164/5424)
CEF (Common Event Format)
JSON structuré
```

### Métriques & Alertes
- **Seuils** : Failed logins, disk usage, bandwidth
- **Anomalies** : Trafic inhabituel, connexions hors heures
- **Rétention** : 1 an minimum (compliance)

## 🛠️ Hardening & Maintenance

### Système (Linux/Windows)
- **Services** : Désactiver non-essentiels
- **Ports** : Fermer non-utilisés (nmap scan)
- **Updates** : Patches sécurité automatiques
- **Comptes** : Supprimer défaut, désactiver guest

### Réseau & Firewall
```bash
# Règles par défaut
Default DENY → Allow spécifique
Logging des refus
Rate limiting (DDoS)
```

### Commandes Utiles
```bash
# Audit sécurité Linux
ss -tuln              # Ports ouverts
ps aux                # Processus actifs
systemctl list-units  # Services
last                  # Dernières connexions

# Réseau
tcpdump -i eth0       # Capture trafic
netstat -an           # Connexions
iptables -L           # Règles firewall
```

## 🎯 Menaces & Contre-mesures

### Top Menaces OWASP/MITRE
1. **Injection** (SQL, XSS, LDAP) → Validation input
2. **Auth brisée** → MFA + session management
3. **Exposition données** → Chiffrement + access control
4. **Phishing** → Formation + filtrage mail
5. **Ransomware** → Sauvegardes + segmentation
6. **Supply chain** → Validation composants

### Kill Chain & Défense
```
Reconnaissance → Weaponization → Delivery → Exploitation
       ↓               ↓            ↓           ↓
   Threat Intel    Sandboxing   Email Sec   Patching
       ↓               ↓            ↓           ↓
Installation → C&C → Actions on Objectives
       ↓         ↓           ↓
    EDR/AV    Network Mon   DLP/SIEM
```

## 🔄 Continuité & Récupération

### Plan de Continuité (PCA/PRA)
- **RTO** : Recovery Time Objective (<4h critique)
- **RPO** : Recovery Point Objective (<1h données)
- **Tests** : Trimestriels minimum
- **Redondance** : Sites, liens, alimentations

### Stratégie Sauvegarde (3-2-1)
- **3** copies des données
- **2** supports différents
- **1** copie hors site/cloud

### Tests de Restauration
```bash
# Validation régulière
Intégrité données
Temps restauration
Procédures d'urgence
Formation équipes
```

## 📋 Compliance & Frameworks

### Standards Majeurs
- **ISO 27001** : SMSI + contrôles sécurité
- **NIST CSF** : Identify → Protect → Detect → Respond → Recover
- **CIS Controls** : 20 contrôles prioritaires
- **RGPD** : Protection données personnelles

### Audit & Évaluation
- **Tests** : Pentest externe annuel
- **Vulnérabilités** : Scan automatisé mensuel
- **Maturity** : Évaluation contrôles (1-5)
- **KPI** : MTTR, MTTD, taux incidents

## 🚀 Checklist Rapide

### Déploiement Nouveau Service
- [ ] Hardening système/applicatif
- [ ] Segmentation réseau appropriée
- [ ] Authentification + autorisation
- [ ] Chiffrement données (transit/repos)
- [ ] Logging + monitoring
- [ ] Sauvegarde + restauration
- [ ] Documentation sécurité
- [ ] Formation utilisateurs

### Réponse Incident
- [ ] Isoler système compromis
- [ ] Préserver preuves (images, logs)
- [ ] Documenter actions
- [ ] Communiquer (interne/externe)
- [ ] Analyser cause racine
- [ ] Implémenter corrections
- [ ] Tester efficacité
- [ ] Mettre à jour procédures

### Audit Sécurité Annuel
- [ ] Scanner vulnérabilités
- [ ] Revue configurations (firewall, IAM)
- [ ] Vérifier journaux critiques
- [ ] Audit accès privilégiés
- [ ] Tester plans PCA/PRA
- [ ] Évaluer sensibilisation utilisateurs
- [ ] Rapport + plan d'action
