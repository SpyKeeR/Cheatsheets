# ToIP / VoIP — condensé 🎧🔌

## Solutions & outils (exemples)
- Open source : Asterisk, FreePBX, XiVO, Elastix, Kamailio/OpenSIPS (SIP Proxy).  
- Commerciaux : Cisco, 3CX, Nokia, services opérateurs.

## Terminals & clients
- Softphones : Zoiper, Linphone, Jitsi, etc.  
- IP Phones : SIP hardware (PoE, LTE fallback possible).  
- Mobile apps / CTI integration (Teams, WebRTC gateways).

## Glossaire rapide
- VoIP : voix transportée sur IP (paquets).  
- ToIP : VoIP + toute l’architecture téléphonique (PABX/IPBX, trunks, terminaux).  
- PABX : Private Automatic Branch Exchange (RTC Central téléphonique)
- IPBX : Centrale téléphonique en IP (on-prem ou VM).  
- Trunk SIP : liaison VoIP opérateur ↔ IPBX.  
- Centrex / TaaS : solution hébergée (IPBX chez opérateur).  
- SDA / DDI : numéros directs publics.  
- SVI : menu vocal interactif (DTMF / ASR).
- RTC(P)/PSTN : Réseau Téléphonique Commuté (Public Switch Telephony Network)
- RNIS/ISDN : Réseau numérique à intégration de services (Integrated Services Digital Network) 


## Signalisation & médias
- SIP : protocole de signalisation (ASCII, proche HTTP).  
  - Méthodes : REGISTER, INVITE, ACK, BYE, CANCEL, OPTIONS, INFO, REINVITE.  
  - Ports : 5060 (UDP/TCP non sécurisé), 5061 (TLS/SIPS).  
  - SDP : négociation des flux (codecs, ports RTP).  
- RTP : transport des médias (voix/vidéo).  
- RTCP : contrôle/synchronisation, statistiques.  
- SRTP : RTP chiffré (sécuriser média).  
- MOS : score qualité (1–5).  


## Codecs & bande passante
- G.711 (µ-law / A-law) → ~64 kbps (codec) → ~80–90 kbps avec overhead (RTP/UDP/IP).  
- G.729 → ~8 kbps → ~24–30 kbps avec overhead.  
- Règle rapide : estimer 100 kbps par canal pour calcul simple (marge).  
  - Exemple : 20 canaux → 100 × 20 = 2000 kbps (2 Mbps).  
- Choix codec = compromis qualité ↔ consommation.


## Calcul plus réaliste (overhead RTP)
- G.711 ≈ 64 kbps payload + ~40–55 kbps overhead → ~105–120 kbps/canal (utile si précision requise).  
- Toujours ajouter marge (20–30%) pour pics et signalisation.


## QoS & SLA (targets VoIP)
- Objectifs : latence <100–150 ms, jitter <20–30 ms, perte <1%.  
- Marquage : DSCP EF (46) pour voix ; AF/CS pour vidéo/data.  
- 802.1p (L2) : map EF → priority (souvent 5).  
- Mise en œuvre : trust DSCP/CoS sur access uplink, queueing et policing sur uplinks, classification au bord.  
- Prioriser : signalling (SIP/TS) puis RTP (voice), laisser best-effort aux bulk transfers.


## Réseau & design (prérequis)
- VLAN voix séparé (voice VLAN).  
- DHCP scope + options (option de provisioning TFTP/HTTP si utilisé).  
- PoE pour téléphones IP (voir PoE).  
- MTU / fragmentation : éviter fragmentation UDP (RTP).  
- NAT traversal : prévoir SBC/STUN/TURN/ALG selon architecture (ALG souvent problématique).


## PoE (alimentation)
- 802.3af (PoE) : up to 15.4 W (≈12.95 W dispo device).  
- 802.3at (PoE+) : up to 30 W (≈25.5 W dispo device).  
- Vérifier budget PoE du switch (W total) + réserve UPS.


## Trunks / offres opérateurs
- Trunk SIP (SIP trunk) : IP trunk vers opérateur.  
- Centrex / Hosted PBX : opérateur héberge l’IPBX.  
- TaaS : service complet téléphonie cloud.  
- Choisir SLA / QoS / codec support / fax/T.38 support.


## Interconnexion & gateways
- Media Gateway : conversion entre ToIP ↔ RTC (TDM), codecs, echo cancel.  
- PSTN gateway / FXO / E1/T1 modules selon besoin.  
- SBC (Session Border Controller) : sécurité, NAT, interco opérateur, transcoding.


## NAT & traversée
- SIP signalling (5060/5061) ≠ RTP media (ports dynamiques).  
- NAT issues : RTP bloqué, wrong SDP IP (private IP).  
- Solutions : STUN/TURN, SBC, symmetric RTP, ALG (souvent déconseillé).  
- Asterisk default RTP range ≈ 10000–20000 (varie selon stack) — ouvrir/forwarder si NAT.


## Provisioning & authentification
- Provision automatique via HTTP/HTTPS/TFTP/FTP (Option 66 pour TFTP/DHCP).  
- Compte SIP : ID, username, secret (dans profil extension).  
- Exemple champ user : extension ID SIP + mot de passe (utilisé par softphone).


## Feature snippets (pabx/flow)
- Music on Hold (MoH) : format recommandé WAV 16-bit PCM (ou mp3 si support).  
- Activation filtre d’appels : code d’activation (exemple donné : *372 pour poste 37).  
- Call logging / taxation : serveur CDR (qui, quand, durée, coût).  
- SVI : DTMF menu / ASR / intégration base métier.  
- SDA/DDI : mapping numéro public → extension interne.


## Qualité & troubleshooting rapide
- Mesures : MOS, packet loss, jitter, one-way delay, RTT.  
- Outils : sipp (test), rtpproxy/rtpdump, Wireshark (SIP/RTP), MOS calculators.  
- Vérifier : VLAN voice, DSCP passthrough, PoE, CPU load IPBX, codec mismatch, NAT.


## Echo & qualité acoustique
- Types : électronique (delays in chain) et acoustique (micro ↔ speaker).  
- Mitigation : echo cancellation (gateway/phone), proper handset design, echo tail tuning.


## Quick checklist déploiement
- VLAN voice + DHCP + Option provisioning.  
- PoE budget vérifié.  
- QoS (DSCP/CoS) end-to-end.  
- SIP trunk with operator + SBC if public.  
- RTP ports opened/translated correctly.  
- MoH files loaded (wav, right bitrate).  
- Monitoring CDR + quality metrics active.