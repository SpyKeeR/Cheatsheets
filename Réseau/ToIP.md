# 📞 ToIP/VoIP — Aide-mémoire

## 🏗️ Architecture & Concepts

### Terminologie Essentielle
| Terme | Définition | Usage |
|-------|------------|-------|
| **VoIP** | Voice over IP - Transport voix sur réseau IP | Technologie de base |
| **ToIP** | Telephony over IP - Infrastructure téléphonique complète | Architecture globale |
| **PABX** | Private Automatic Branch eXchange (analogique/numérique) | Centrale traditionnelle |
| **IPBX** | IP Private Branch eXchange | Centrale VoIP |
| **SBC** | Session Border Controller | Sécurité/NAT/Interco |
| **SDA/DDI** | Sélection Directe à l'Arrivée / Direct Dial In | Numérotation directe |

### Types d'Architecture
```
On-Premise IPBX:
├── Serveur IPBX local (Asterisk, FreePBX, 3CX)
├── Postes IP internes
├── Trunk SIP opérateur
└── Passerelle RTC (optionnel)

Cloud/Centrex:
├── IPBX hébergé chez opérateur
├── Postes IP locaux ou softphones
├── Connexion Internet + QoS
└── Services managés (TaaS)

Hybride:
├── IPBX local + services cloud
├── Trunk SIP + ligne RTC backup
└── Postes mixtes (IP + analogiques)
```

## 📡 Protocoles & Signalisation

### SIP (Session Initiation Protocol)
| Méthode | Port | Description | Usage |
|---------|------|-------------|-------|
| **INVITE** | 5060 UDP/TCP | Initier appel | Établissement session |
| **ACK** | 5060 | Confirmer réception | Finaliser handshake |
| **BYE** | 5060 | Terminer appel | Fermeture session |
| **CANCEL** | 5060 | Annuler INVITE | Abandon avant ACK |
| **REGISTER** | 5060 | Enregistrement | Authentification poste |
| **OPTIONS** | 5060 | Capacités | Keep-alive/diagnostic |

### Flux d'Appel Typique
```
Appelant          Proxy SIP          Appelé
   |                  |                |
   |---INVITE-------->|                |
   |                  |---INVITE------>|
   |                  |<--100 Trying---|
   |<--100 Trying-----|                |
   |                  |<--180 Ringing--|
   |<--180 Ringing----|                |
   |                  |<--200 OK-------|
   |<--200 OK---------|                |
   |---ACK----------->|---ACK--------->|
   |<======= RTP Media Stream ========>|
   |---BYE----------->|---BYE--------->|
   |<--200 OK---------|<--200 OK-------|
```

### SDP (Session Description Protocol)
```
Négociation dans INVITE/200 OK:
v=0                          # Version
o=user 123456 654321 IN IP4 192.168.1.100  # Origine
s=Session                    # Session name
c=IN IP4 192.168.1.100      # Connection info
m=audio 8000 RTP/AVP 0 8    # Media (port, protocole, codecs)
a=rtpmap:0 PCMU/8000        # Codec G.711 µ-law
a=rtpmap:8 PCMA/8000        # Codec G.711 A-law
```

## 🎧 Codecs & Qualité

### Codecs Audio Courants
| Codec | Bitrate | Qualité | Bande Passante* | Usage |
|-------|---------|---------|-----------------|-------|
| **G.711 (PCMU/PCMA)** | 64 kbps | Excellente | ~87 kbps | Standard, pas de compression |
| **G.729** | 8 kbps | Bonne | ~31 kbps | Économie bande passante |
| **G.722** | 64 kbps | HD (7kHz) | ~87 kbps | Haute qualité |
| **iLBC** | 13.3/15.2 kbps | Bonne | ~35 kbps | Résistant aux pertes |
| **Opus** | 6-510 kbps | Variable | Selon config | Moderne, adaptatif |

*Avec overhead RTP/UDP/IP (Ethernet)

### Calculs Bande Passante
```
G.711 Exemple (paquet 20ms):
├── Payload : 160 octets (20ms × 8000Hz × 1 octet)
├── RTP : 12 octets
├── UDP : 8 octets  
├── IP : 20 octets
├── Ethernet : 18 octets (14 + 4 FCS)
└── Total : 218 octets × 50 pps = 87.2 kbps

Formule rapide:
Simultanés × 100 kbps = Bande passante totale
Exemple: 20 appels = 2 Mbps (avec marge)
```

### Qualité Voix (Métriques)
| Métrique | Cible | Impact si dépassé |
|----------|-------|-------------------|
| **Latence** | <150ms | Écho, conversations difficiles |
| **Gigue** | <30ms | Voix saccadée |
| **Perte paquets** | <1% | Coupures, qualité dégradée |
| **MOS** | >4.0 | Score qualité subjective (1-5) |

## 🌐 QoS & Réseau

### Marquage DSCP/CoS
```
Classification Trafic VoIP:
┌─────────────────┬─────────┬─────────┐
│ Type Trafic     │ DSCP    │ 802.1p  │
├─────────────────┼─────────┼─────────┤
│ Voix (RTP)      │ EF (46) │ 5       │
│ Signalisation   │ CS3(24) │ 3       │
│ Vidéo           │ AF41(34)│ 4       │
│Données critiques│ AF31(26)│ 3       │
│ Best effort     │ 0       │ 0       │
└─────────────────┴─────────┴─────────┘
```

### Configuration QoS Type
```cisco
# Classification (access switch)
class-map match-all VOICE
 match ip dscp ef
class-map match-all VOICE-SIGNALING  
 match ip dscp cs3

# Policy (bandwidth allocation)
policy-map QOS_POLICY
 class VOICE
  priority percent 30
 class VOICE-SIGNALING
  bandwidth percent 10
 class class-default
  fair-queue
```

### VLAN Voice Configuration
```cisco
# Port accès avec VLAN voice
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10        # VLAN données
 switchport voice vlan 110        # VLAN voix
 mls qos trust cos               # Trust CoS du téléphone
 spanning-tree portfast          # Convergence rapide
```

## 🔌 Infrastructure & PoE

### Standards PoE
| Standard | Puissance Switch | Puissance Device | Usage VoIP |
|----------|------------------|------------------|------------|
| **802.3af (PoE)** | 15.4W | 12.95W | Téléphones basiques |
| **802.3at (PoE+)** | 30W | 25.5W | Téléphones avancés + écran |
| **802.3bt (PoE++)** | 60W/100W | 51W/71W | Visioconférence |

### Calcul Budget PoE
```
Exemple: Switch 24 ports, budget 370W
├── 20 téléphones × 7W = 140W
├── 4 points d'accès × 15W = 60W  
├── Marge sécurité 20% = 40W
└── Total utilisé = 240W < 370W ✓
```

## 📞 Solutions & Plateformes

### IPBX Open Source
| Solution | Description | Points Forts |
|----------|-------------|--------------|
| **Asterisk** | Engine téléphonie | Très flexible, large communauté |
| **FreePBX** | Interface web Asterisk | Facilité configuration |
| **FusionPBX** | IPBX basé FreeSWITCH | Multi-tenant, moderne |
| **Kamailio/OpenSIPS** | Proxy SIP | High performance, carrier-grade |

### Solutions Commerciales
- **Cisco** : CUCM, Unity, Contact Center
- **3CX** : IPBX Windows/Linux, mobile apps
- **Avaya** : IP Office, Aura
- **Microsoft** : Teams Phone System

### Services Cloud (Centrex)
- **Avantages** : Pas d'infrastructure, mises à jour auto, évolutivité
- **Inconvénients** : Dépendance Internet, coût récurrent, moins de contrôle

## 🔒 Sécurité & NAT

### Problématiques NAT
```
Problèmes courants:
├── Signalisation SIP (5060) OK mais RTP bloqué
├── SDP contient IP privée → RTP impossible  
├── Ports RTP dynamiques non ouverts
└── Timeouts connexion NAT

Solutions:
├── STUN/TURN servers (découverte IP publique)
├── SBC (Session Border Controller)  
├── UPnP/NAT-PMP (ouverture ports auto)
└── SIP ALG désactivé (souvent problématique)
```

### Configuration Firewall Type
```
Ports à ouvrir (exemple Asterisk):
├── SIP : 5060-5061 UDP/TCP
├── RTP : 10000-20000 UDP (configurable)  
├── Provisioning : 80/443 TCP (HTTP/HTTPS)
└── Management : 22 TCP (SSH), 8088 TCP (AMI)
```

### Sécurisation SIP
```
Bonnes pratiques:
├── Authentification digest (pas plain text)
├── TLS pour signalisation (SIPS:5061)
├── SRTP pour média (chiffrement RTP)
├── Fail2ban contre brute force
├── Whitelist IP si possible
└── Comptes forts (pas extension = password)
```

## 🛠️ Provisioning & Configuration

### Auto-Provisioning
```
Méthodes déploiement:
├── DHCP Option 66 : URL serveur TFTP/HTTP
├── PnP (Plug and Play) : découverte automatique
├── Redirection DNS : provision.domain.com
└── Configuration manuelle : URL dans téléphone

Fichiers configuration:
├── Template global (modèle par série)
├── Config spécifique (par MAC address)  
├── Firmware updates
└── Phonebook centralisé
```

### DHCP Options VoIP
```cisco
# Option 66 - TFTP Server
ip dhcp pool VOICE_VLAN
 network 192.168.110.0 255.255.255.0
 default-router 192.168.110.1
 dns-server 192.168.110.1
 option 66 ascii "tftp://192.168.110.10"
 option 150 ip 192.168.110.10        # Cisco specific
```

## 📊 Monitoring & Diagnostic

### Métriques Clés
```
Surveillance temps réel:
├── Appels simultanés / capacité max
├── Qualité MOS moyenne par codec
├── Taux échec d'appel (ASR - Answer Seizure Ratio)
├── Latence/gigue moyenne RTP
└── Utilisation CPU/mémoire IPBX

Historique CDR (Call Detail Record):
├── Qui a appelé qui, quand, durée
├── Coût par appel (taxation)
├── Statistiques par utilisateur/département  
└── Rapports conformité/légaux
```

### Outils Diagnostic
```bash
# Test connectivité SIP
sip-tester -sf scenario.xml target_ip:5060

# Analyse trafic
tcpdump -i eth0 -w voip.pcap host sip_server and port 5060
wireshark voip.pcap          # Filtres: sip, rtp

# Test qualité RTP
rtpbreak -d capture.pcap     # Analyse flux RTP
```

### Codes Réponse SIP Courants
| Code | Signification | Action |
|------|---------------|--------|
| **100** | Trying | En cours traitement |
| **180** | Ringing | Sonnerie côté appelé |
| **200** | OK | Succès |
| **401** | Unauthorized | Authentification requise |
| **404** | Not Found | Utilisateur inexistant |
| **486** | Busy Here | Ligne occupée |
| **503** | Service Unavailable | Serveur indisponible |

## 📋 Checklist Déploiement

### Prérequis Infrastructure
- [ ] **VLAN voice** configuré et isolé
- [ ] **QoS** end-to-end (DSCP/CoS)
- [ ] **PoE** budget suffisant + UPS
- [ ] **Bande passante** dimensionnée (100kbps/canal)
- [ ] **Firewall** ports ouverts selon matrice
- [ ] **DNS** résolution noms FQDN

### Configuration IPBX
- [ ] **Trunk SIP** opérateur testé
- [ ] **Extensions** créées + authentification
- [ ] **Plan numérotation** configuré
- [ ] **Services** (répondeur, renvoi, conférence)
- [ ] **Musique attente** formats compatibles
- [ ] **CDR** logging activé

### Tests & Validation
- [ ] **Appels internes** entre extensions
- [ ] **Appels sortants** vers RTC/mobile
- [ ] **Appels entrants** depuis extérieur  
- [ ] **Qualité audio** (MOS >4.0)
- [ ] **Services** (transfert, mise en attente)
- [ ] **Failover** coupure Internet/panne

## 💡 Bonnes Pratiques

### Dimensionnement
- ✅ **Règle 80/20** : 80% utilisateurs, 20% en simultané
- ✅ **Codecs multiples** : G.711 local, G.729 WAN
- ✅ **Redondance** : Trunk backup, serveur HA
- ✅ **Monitoring** : Alertes proactives qualité

### Sécurité
- ✅ **Segmentation** : VLAN voice isolé du data
- ✅ **Authentification forte** : pas de comptes par défaut
- ✅ **Chiffrement** : TLS/SRTP si sensible
- ✅ **Surveillance** : logs tentatives intrusion

### Performance
- ✅ **QoS prioritaire** : Voice > Video > Data
- ✅ **Jitter buffer** : adaptatif selon réseau
- ✅ **Echo cancellation** : sur passerelles/postes
- ✅ **Codec négociation** : préférer HD si BP disponible

---
**💡 Memo** : G.711 = qualité max • QoS mandatory • NAT = complexité • Monitoring = proactif !