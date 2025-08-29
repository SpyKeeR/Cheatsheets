# Infrastructures réseaux — condensé ⚙️

## PDU / SDU / PCI
- PDU : unité encapsulée à chaque couche. (Protocol Data Unit)
- SDU : donnée utile reçue de la couche supérieure.  (Service Data Unit)
- PCI : info de contrôle (en-têtes, métadonnées) ajoutée à la SDU. (Protocol Control Information)

## Modèle OSI (ultra-court)
- ⚙️ Physique (Layer 1) : transmission bit par bit sur le média (signal, connectique, voltage…) → bit
- 🔗 Liaison de données (Layer 2) : communication directe entre machines (adressage MAC, détection/correction d'erreurs) → Trame
- 🌐 Réseau (Layer 3) : acheminement entre réseaux (routage, adresses IP) → Paquet
- 📦 Transport (Layer 4) : fiabilité de bout en bout, découpage, contrôle de flux (TCP/UDP) → Segment
- 🔐 Session (Layer 5) : gestion de session (authentification, points de reprise)
- 🎨 Présentation (Layer 6) : traduction, chiffrement, compression des données
- 🖥️ Application (Layer 7) : interface avec l’utilisateur, accès réseau pour les applis

## Trame Ethernet (IEEE 802.3) — champs clés
- Préambule : 7 x 10101010.  
- SFD : 10101011.  
- Dest MAC (6 octets).  
- Src MAC (6 octets).  
- Ethertype (ex. 0x0800 IPv4, 0x86DD IPv6, 0x8100 Dot1Q).  
- Payload (données). 
- FCS (CRC, 4 octets).

## VLAN tag (802.1Q)
- TPID = 0x8100 (indique tag).  
- TCI = Priority(3)-CoS | DEI(Drop Eligible Indicator)(ex-CFI)(1) > | VLAN ID(12).  
- VLAN ID : 1–4094.  
- Si Ethertype 0x8100 → Lecture TCI

## IPv4 — en-tête (champs essentiels)
- 📌 Version : identifie IPv4 (valeur 4)
- 📏 Longueur d’en-tête (IHL) : taille de l’en-tête en mots de 32 bits
- 🎯 Type de service (ToS) : QoS/DSCP – priorité du paquet
- 🧱 Longueur totale : taille complète du paquet (en-tête + données)
- 🧩 Identification : identifie les fragments d’un même paquet
- 🚩 Indicateur (Flags) : contrôle la fragmentation (DF, MF)
- 📐 Fragment Offset : position du fragment dans le paquet original
- ⏳ Time to Live (TTL) : nombre de sauts max avant suppression
- 📡 Protocole : indique le protocole encapsulé (ex : 6 = TCP, 17 = UDP)
- 🔍 Checksum de l’en-tête : vérifie l’intégrité du header
- 📬 Adresse source : IP de l’expéditeur
- 📥 Adresse de destination : IP du destinataire
- ⚙️ Options (facultatif) : ajout de fonctionnalités spécifiques
- 📶 Remplissage (padding) : alignement sur 32 bits

## Architecture 3-tiers (entreprise)
- Accès (Access) → ports utilisateurs, VLANs.  
- Distribution → ACLs, agrégation, QoS.  
- Cœur (Core) → routage haute vitesse, backbone.

## Routage — types & protocoles (liste courte)
- Statique → config manuelle (stable).  
- Dynamique → échange automatique 
	- IGP (Internal Gateway Protocol) (Inside AS)
		- Vecteur de distance : 
			- RIP (Routing Information Protocol)
			- IGRP (Interior Gateway Routing Protocol)
		- Etats de lien : 
			- IS-IS (IntermediateSystem 2 IntermediateSystem)
			- OSPF (Open Shortest Path First)
		- Mixte : 
			- EIGRP (Cisco) (Extended-IGRP)
	- EGP  (External Gateway Protocol) (Outside AS)
		- Vecteur de chemin :
			BGP (Border Gateway Protocol)

## QoS 
- Best effort = "au mieux", sans garantie → suffisant pour les e-mails ou fichiers, mais insuffisant pour voix/vidéo
- QoS = nécessaire pour assurer qualité, priorité, et fluidité des flux sensibles
- Sans QoS → congestion, latence, coupures = expérience utilisateur dégradée

- Bande passante : débit disponible (en Mbps/Gbps)
- Délai (latence) : temps d’acheminement d’un paquet
- Gigue : variation des délais entre les paquets
- Perte de paquets : paquets envoyés ≠ paquets reçus

# Wi-Fi & Concepts réseau — condensé 📶

## Wi-Fi — modes & composants
- Composants : client (STA), AP, WLC (contrôleur), router/gateway.  
- Modes : Ad-hoc (IBSS) = peer-to-peer ; Infrastructure = AP-client (accès Internet).  
- Duplex : half-duplex (une seule station transmit à la fois).

## Contrôle d’accès média
- CSMA/CA (éviter collisions) → RTS/CTS handshake si nécessaire.  
- BSS : un AP + ses clients. ESS : plusieurs APs, même SSID, mobilité.

## Identifiants
- SSID : nom réseau.  
- BSSID : MAC de l’AP.  
- ESSID : SSID partagé sur APs multiples.

## Roaming & coverage
- Plusieurs APs → même SSID + overlap contrôlé → handoff client géré par client/roaming algos.  
- Channel planning (2.4GHz: 1/6/11 non-overlap), éviter co-channel interference.

## QoS VoIP over Wi-Fi
- DSCP/802.11e (WMM) → prioriser voice traffic (AC_VO).  
- Trust COS on switch access port for voice VLAN.

## Déploiement & gestion
- Centraliser via controller pour politiques, RF planning, firmware.  
- Monitor RSSI, SNR, channel utilization, client count.