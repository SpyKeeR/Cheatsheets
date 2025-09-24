# 🌐 Cisco IOS — Aide-mémoire

## 🏗️ Architecture & Mémoire

### Composants Système
| Mémoire | Contenu | Persistance | Accès |
|---------|---------|-------------|-------|
| **ROM** | Bootstrap, POST, mini-IOS (RxBoot) | Permanent | Lecture seule |
| **Flash** | Image IOS (.bin) | Non-volatile | Lecture/Écriture |
| **NVRAM** | startup-config | Non-volatile | Configuration sauvée |
| **RAM** | running-config, tables, processus | Volatile | Données actives |

### Processus de Démarrage
```
1. POST (Power-On Self Test) depuis ROM
2. Bootstrap charge IOS depuis Flash
3. IOS charge startup-config depuis NVRAM → running-config
4. Si pas de startup-config → Setup mode
```

### Registre de Configuration
```
0x2102 : Configuration normale (défaut)
0x2142 : Ignorer startup-config (recovery password)
0x2101 : Boot depuis ROM (mini-IOS)

# Modifier registre
(config)# config-register 0x2142
(config)# exit
# copy running-config startup-config
# reload
```

## 🎯 Modes & Navigation CLI

### Hiérarchie des Modes
```
User EXEC                    Switch>
├── Privileged EXEC          Switch#
    ├── Global Config        Switch(config)#
        ├── Interface        Switch(config-if)#
        ├── Line            Switch(config-line)#
        ├── Router          Switch(config-router)#
        └── VLAN            Switch(config-vlan)#
```

### Navigation Rapide
```cisco
enable                       # User → Privileged
configure terminal           # Privileged → Global config
interface fastethernet0/1    # → Interface mode
line console 0              # → Line mode
exit                        # Remonter d'un niveau
end                         # → Privileged mode
Ctrl+Z                      # → Privileged mode (raccourci)
```

### Raccourcis CLI
| Raccourci | Action |
|-----------|--------|
| `?` | Aide contextuelle |
| `Tab` | Auto-complétion |
| `↑` `↓` | Historique commandes |
| `Ctrl+A` | Début de ligne |
| `Ctrl+E` | Fin de ligne |
| `Ctrl+U` | Effacer ligne |
| `Ctrl+C` | Interrompre commande |
| `Ctrl+Z` | Retour mode privilégié |

## 💾 Gestion Configuration

### Commandes Configuration Essentielles
```cisco
# Visualisation
show running-config          # Configuration active
show startup-config          # Configuration sauvée
show version                # Infos système, version IOS

# Sauvegarde
copy running-config startup-config    # Sauver config active
write memory                          # Alias de la commande précédente
copy running-config tftp:             # Backup vers TFTP

# Restauration
copy startup-config running-config    # Charger config sauvée
copy tftp: running-config             # Restaurer depuis TFTP
```

### Reset & Effacement
```cisco
# Effacer configuration
erase startup-config         # Supprimer config sauvée
delete flash:vlan.dat       # Supprimer base VLAN (switch)
reload                      # Redémarrer équipement

# Factory reset complet
erase startup-config
delete flash:vlan.dat
reload
```

## 🔧 Configuration de Base

### Paramètres Système
```cisco
# Identité équipement
(config)# hostname SWITCH-01
(config)# banner motd "Accès autorisé uniquement"

# Domaine & DNS
(config)# ip domain-name entreprise.local
(config)# ip name-server 8.8.8.8
(config)# ip name-server 8.8.4.4

# Horloge & NTP
(config)# clock timezone CET 1
(config)# ntp server 0.fr.pool.ntp.org
(config)# ntp server 1.fr.pool.ntp.org
```

### Sécurité Mots de Passe
```cisco
# Mot de passe enable
(config)# enable secret password123    # Chiffré (recommandé)
(config)# enable password password123  # Clair (éviter)

# Chiffrement mots de passe
(config)# service password-encryption  # Chiffrer mdp en clair

# Console
(config)# line console 0
(config-line)# password console123
(config-line)# login
(config-line)# exec-timeout 5 0        # Timeout 5 min

# VTY (Telnet/SSH)
(config)# line vty 0 15
(config-line)# password vty123
(config-line)# login
(config-line)# transport input ssh     # SSH seulement
```

### Configuration SSH
```cisco
# Prérequis SSH
(config)# hostname SWITCH-01
(config)# ip domain-name entreprise.local
(config)# crypto key generate rsa modulus 2048

# Utilisateur local
(config)# username admin privilege 15 secret admin123

# VTY pour SSH
(config)# line vty 0 15
(config-line)# login local
(config-line)# transport input ssh
(config-line)# exec-timeout 10 0

# Version SSH
(config)# ip ssh version 2
(config)# ip ssh time-out 60
(config-line)# ip ssh authentication-retries 3
```

## 🌐 Configuration Réseau

### Interfaces & VLAN
```cisco
# Interface physique
(config)# interface fastethernet0/1
(config-if)# description "Serveur Web"
(config-if)# ip address 192.168.1.10 255.255.255.0
(config-if)# no shutdown
(config-if)# duplex full
(config-if)# speed 100

# VLAN (switch)
(config)# vlan 10
(config-vlan)# name ADMIN
(config-vlan)# exit
(config)# vlan 20
(config-vlan)# name USERS

# Port access
(config)# interface fastethernet0/2
(config-if)# switchport mode access
(config-if)# switchport access vlan 10

# Port trunk
(config)# interface fastethernet0/24
(config-if)# switchport mode trunk
(config-if)# switchport trunk allowed vlan 10,20,30
```

### Interface Virtuelle (SVI)
```cisco
# VLAN interface pour management
(config)# interface vlan 1
(config-if)# ip address 192.168.1.100 255.255.255.0
(config-if)# no shutdown
(config-if)# description "Management VLAN"

# Passerelle par défaut (switch L2)
(config)# ip default-gateway 192.168.1.1
```

## 📡 Routage Basique

### Routes Statiques
```cisco
# Route statique
(config)# ip route 192.168.2.0 255.255.255.0 192.168.1.1
(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.1    # Route par défaut

# Route avec interface de sortie
(config)# ip route 10.0.0.0 255.0.0.0 serial0/0/0

# Route flottante (backup)
(config)# ip route 192.168.2.0 255.255.255.0 192.168.1.2 20  # AD=20
```

### Routage Inter-VLAN (Router-on-a-Stick)
```cisco
# Interface principale
(config)# interface gigabitethernet0/0
(config-if)# no shutdown

# Sous-interfaces
(config)# interface gigabitethernet0/0.10
(config-subif)# encapsulation dot1q 10
(config-subif)# ip address 192.168.10.1 255.255.255.0

(config)# interface gigabitethernet0/0.20
(config-subif)# encapsulation dot1q 20
(config-subif)# ip address 192.168.20.1 255.255.255.0
```

## 🛠️ Recovery & Maintenance

### Récupération Mot de Passe
```cisco
# Méthode 1 : Registre config
1. Redémarrer et Ctrl+Break → ROMmon
2. confreg 0x2142
3. reset
4. copy startup-config running-config
5. (config)# enable secret nouveau_mdp
6. (config)# config-register 0x2102
7. copy running-config startup-config
8. reload
```

### Mode ROMmon
```
Accès : Ctrl+Break durant boot
Commandes principales :
├── boot                     # Démarrer image IOS
├── dir flash:              # Lister fichiers flash
├── confreg 0x2142          # Modifier registre
├── set                     # Variables environnement
├── reset                   # Redémarrer
└── tftpdnld               # Télécharger IOS (⚠️ efface flash)
```

### Restauration IOS via TFTP
```cisco
# Depuis mode privilégié
copy tftp: flash:
Source filename []? c2960-lanbasek9-mz.150-2.SE11.bin
Address or name of remote host []? 192.168.1.50
Destination filename [c2960-lanbasek9-mz.150-2.SE11.bin]? 

# Depuis ROMmon (urgence)
rommon 1> IP_ADDRESS=192.168.1.100
rommon 2> IP_SUBNET_MASK=255.255.255.0
rommon 3> DEFAULT_GATEWAY=192.168.1.1
rommon 4> TFTP_SERVER=192.168.1.50
rommon 5> TFTP_FILE=c2960-lanbasek9-mz.150-2.SE11.bin
rommon 6> tftpdnld
```

## 📊 Monitoring & Diagnostic

### Commandes Show Essentielles
```cisco
# Système
show version                # Version IOS, uptime, matériel
show running-config         # Configuration active
show startup-config         # Configuration sauvée
show flash:                 # Contenu mémoire flash
show memory                 # Utilisation mémoire
show processes             # Processus actifs

# Interfaces
show interfaces            # Toutes interfaces
show interface fa0/1       # Interface spécifique
show interfaces status     # Statut compact (switch)
show interfaces trunk     # Ports trunk (switch)

# VLAN (switch)
show vlan brief           # VLANs configurés
show vlan id 10           # VLAN spécifique
show interfaces switchport # Config switchport

# Routage
show ip route             # Table routage
show ip route summary     # Résumé routes
show arp                 # Table ARP

# Logs
show logging             # Messages système
show logging | include ERROR
```

### Diagnostic Connectivité
```cisco
# Tests réseau
ping 8.8.8.8             # Test connectivité
traceroute 8.8.8.8       # Trace route
telnet 192.168.1.1 23    # Test port TCP

# Debug (attention production!)
debug ip icmp            # Debug ICMP
debug ip routing         # Debug routage
undebug all             # Arrêter tous debug
```

## 🔒 Sécurité & Bonnes Pratiques

### Hardening Basique
```cisco
# Sécuriser console
(config)# line console 0
(config-line)# exec-timeout 5 0
(config-line)# logging synchronous

# Sécuriser VTY
(config)# line vty 0 15
(config-line)# exec-timeout 10 0
(config-line)# transport input ssh
(config-line)# access-class 10 in    # ACL pour limiter accès

# Désactiver services non utilisés
(config)# no ip http-server
(config)# no cdp run
(config)# no service finger
```

### Sauvegarde Configuration
```cisco
# Backup automatique
(config)# archive
(config-archive)# path tftp://192.168.1.50/backups/$h-$t
(config-archive)# write-memory
(config-archive)# time-period 1440    # Sauvegarde quotidienne
```

## 🔧 Console & Accès Physique

### Paramètres Connexion Console
```
Port : RJ-45 ou USB mini-B (selon modèle)
Paramètres série :
├── Vitesse : 9600 bauds
├── Bits données : 8
├── Parité : Aucune
├── Bits d'arrêt : 1
└── Contrôle flux : Aucun
```

### Outils Connexion
- **Windows** : PuTTY, Tera Term, HyperTerminal
- **Linux** : minicom, screen, picocom
- **macOS** : screen, Terminal

## 💡 Astuces & Raccourcis

### Configuration Rapide
- **Tab** : Auto-complétion (gain de temps énorme)
- **?** : Aide contextuelle à n'importe quel moment
- **show run | begin interface** : Afficher config depuis "interface"
- **show run | include password** : Filtrer lignes contenant "password"
- **terminal length 0** : Désactiver pagination temporairement

### Troubleshooting Rapide
1. **Physique** : `show interfaces` → état up/down
2. **L2** : `show mac address-table` → apprentissage MAC
3. **L3** : `show ip route` → table routage
4. **Connectivité** : `ping`, `traceroute`
5. **Configuration** : `show running-config`

### Commandes One-liner Utiles
```cisco
# Voir ports utilisés
show interfaces status | exclude notconnect

# Sauver et redémarrer
copy run start && reload

# Effacer et redémarrer (factory reset)
erase startup-config && delete flash:vlan.dat && reload
```

---
**💡 Memo** : Mode strict = `enable secret` + `service password-encryption` + SSH only + timeouts configurés !