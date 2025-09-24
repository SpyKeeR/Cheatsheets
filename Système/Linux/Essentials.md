# 🐧 Linux Essentials — Aide-mémoire

## 💻 Concepts Fondamentaux

### Philosophie Unix/Linux
- **Tout est fichier** : périphériques, processus, sockets
- **Un outil = une fonction** : combiner via pipes
- **Texte universel** : configuration et échange de données
- **Hiérarchie unique** : système de fichiers unifié depuis `/`

### Shell & Terminal
```
Terminal ← Shell ← Kernel ← Hardware
    ↓        ↓        ↓         ↓
Interface  Interpréteur  Noyau   Matériel
```

- **Shell** : Interface commande (bash, zsh, fish)
- **Terminal** : Émulateur interface texte
- **Console** : Terminal physique système

## 🔐 Identité & Sessions

### Informations Utilisateur
```bash
# Identification actuelle
whoami                       # Nom utilisateur courant
id                          # UID, GID, groupes
id -u username              # UID utilisateur spécifique
logname                     # Nom utilisateur login original
groups                      # Groupes de l'utilisateur

# Utilisateurs connectés
who                         # Sessions actives
who -H                      # Avec en-têtes
w                          # Sessions + activité
last                       # Historique connexions
last username              # Historique utilisateur spécifique
```

### Gestion Mot de Passe
```bash
# Modification
passwd                      # Changer son mot de passe
sudo passwd username        # Changer mot de passe autre utilisateur

# Informations
passwd -S username          # Statut mot de passe
chage -l username          # Politique expiration
sudo passwd -e username     # Forcer expiration au prochain login
```

## 📅 Date & Temps

### Affichage & Format
```bash
# Date de base
date                        # Date/heure système
date -u                     # UTC
cal                        # Calendrier mois courant
cal 2024                   # Calendrier année
cal 12 2024                # Décembre 2024

# Formats personnalisés
date +"%Y-%m-%d %H:%M:%S"   # 2024-01-15 14:30:00
date +"%A %d %B %Y"         # Lundi 15 janvier 2024
date +"%s"                  # Timestamp Unix
```

### Calculs de Date
```bash
# Date relative
date -d "tomorrow"          # Demain
date -d "next monday"       # Lundi prochain
date -d "2 weeks ago"       # Il y a 2 semaines
date -d "2024-12-25"        # Date spécifique

# Conversion timestamp
date -d @1672531200         # Timestamp → date lisible
```

## 🔧 Variables & Environnement

### Variables Système Importantes
| Variable | Description | Exemple |
|----------|-------------|---------|
| `$HOME` | Répertoire personnel | `/home/user` |
| `$USER` | Nom utilisateur | `john` |
| `$PATH` | Chemins exécutables | `/usr/bin:/bin` |
| `$SHELL` | Shell courant | `/bin/bash` |
| `$PWD` | Répertoire courant | `/home/user/docs` |
| `$HOSTNAME` | Nom machine | `server01` |
| `$LANG` | Langue/locale | `fr_FR.UTF-8` |

### Gestion Variables
```bash
# Consultation
printenv                    # Toutes variables environnement
env                        # Alternative printenv
set                        # Variables shell + environnement (verbeux)
echo $HOME                 # Valeur variable spécifique

# Définition (session courante)
MYVAR="valeur"             # Variable locale shell
export MYVAR="valeur"      # Variable environnement
unset MYVAR               # Supprimer variable

# Persistance
~/.bashrc                 # Configuration bash utilisateur
~/.profile               # Variables environnement utilisateur
/etc/environment         # Variables système globales
```

### Substitution Commandes
```bash
# Anciennes/nouvelles syntaxes
heure=`date +"%H:%M"`      # Backticks (legacy)
heure=$(date +"%H:%M")     # Préféré (moderne)

# Usage pratique
echo "Il est $(date +%H:%M)"
fichier="backup_$(date +%Y%m%d).tar"
for server in $(cat servers.txt); do ping -c1 $server; done
```

## 📚 Système d'Aide

### Commandes d'Aide
```bash
# Aide rapide
commande --help            # Option universelle
commande -h               # Variante courte

# Manuel système
man commande              # Page manuel complète
man -s 5 passwd          # Section spécifique (5=config files)
man -k keyword           # Recherche par mot-clé (apropos)
whatis commande          # Description courte

# Autres sources
info commande            # Documentation GNU (plus détaillée)
/usr/share/doc/          # Documentation packages
```

### Sections Manual (man)
| Section | Contenu | Exemple |
|---------|---------|---------|
| **1** | Commandes utilisateur | `man ls` |
| **2** | Appels système | `man fork` |
| **3** | Fonctions bibliothèques | `man printf` |
| **4** | Fichiers spéciaux | `man null` |
| **5** | Formats fichiers config | `man passwd` |
| **6** | Jeux | `man fortune` |
| **7** | Conventions, protocoles | `man ascii` |
| **8** | Commandes administrateur | `man mount` |

### Navigation Manuel
- `Space` : Page suivante
- `b` : Page précédente
- `/terme` : Rechercher terme
- `n` : Occurrence suivante
- `N` : Occurrence précédente
- `q` : Quitter

## 📜 Historique Commandes

### Utilisation Historique
```bash
# Consultation
history                    # Afficher historique complet
history 10                 # 10 dernières commandes
history | grep ssh         # Rechercher dans historique

# Réexécution
!!                        # Dernière commande
!n                        # Commande numéro n
!ssh                      # Dernière commande commençant par 'ssh'
!?passwd                  # Dernière contenant 'passwd'
^old^new                  # Remplace 'old' par 'new' dans dernière cmd
```

### Recherche Interactive
```bash
# Raccourcis clavier
Ctrl+R                    # Recherche rétroactive (reverse-i-search)
Ctrl+S                    # Recherche avant (si activé)
Ctrl+G                    # Annuler recherche
```

### Configuration Historique
```bash
# Variables importantes
$HISTFILE                 # Fichier historique (~/.bash_history)
$HISTSIZE                 # Taille historique en mémoire (500)
$HISTFILESIZE            # Taille fichier historique (500)

# Options dans ~/.bashrc
export HISTSIZE=2000
export HISTFILESIZE=2000
export HISTCONTROL=ignoredups:erasedups  # Éviter doublons
export HISTTIMEFORMAT="%Y-%m-%d %T "     # Horodatage
```

## 🔍 Localisation Commandes

### Outils de Localisation
```bash
# Localisation binaires
which commande            # Chemin exécutable dans $PATH
whereis commande         # Binaire + sources + man pages
type commande            # Type commande (builtin/alias/fonction/binaire)

# Recherche avancée
find /usr -name "python*" -type f -executable  # Recherche fine
locate python            # Base indexée (updatedb)
```

### Types de Commandes
```bash
# Vérification type
type ls                  # ls is aliased to `ls --color=auto`
type cd                  # cd is a shell builtin
type python              # python is /usr/bin/python

# Informations détaillées
file /usr/bin/ls         # Type fichier détaillé
ldd /usr/bin/ls          # Dépendances bibliothèques
```

## 🎭 Alias & Raccourcis

### Gestion Alias
```bash
# Création alias
alias ll='ls -la'         # Alias simple
alias grep='grep --color=auto'  # Avec options
alias la='ls -A'          # Fichiers cachés sans . et ..

# Consultation
alias                     # Tous les alias
alias ll                  # Définition alias spécifique

# Suppression
unalias ll               # Supprimer alias
unalias -a               # Supprimer tous alias
```

### Alias Courants Utiles
```bash
# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias cd..='cd ..'

# Listing amélioré
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Sécurité
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Réseau
alias ping='ping -c 5'
alias ports='netstat -tulanp'
```

## 💽 Flux & Redirections

### Descripteurs Standards
| Stream | Numéro | Description | Défaut |
|--------|--------|-------------|--------|
| **stdin** | 0 | Entrée standard | Clavier |
| **stdout** | 1 | Sortie standard | Écran |
| **stderr** | 2 | Sortie erreur | Écran |

### Redirections de Base
```bash
# Sortie
commande > fichier        # Redirection stdout (écrase)
commande >> fichier       # Ajout stdout
commande 2> erreurs.log   # Redirection stderr
commande &> tout.log      # stdout + stderr
commande > /dev/null 2>&1 # Silence complet

# Entrée
commande < fichier        # Lire depuis fichier
commande <<< "texte"      # Here string
```

### Pipes & Chaînage
```bash
# Pipes simples
ls | grep ".txt"          # Filtrer sortie
ps aux | grep apache      # Rechercher processus
cat file | sort | uniq    # Trier et dédoublonner

# Tee (dupliquer flux)
commande | tee fichier.log  # Afficher ET sauvegarder
commande | tee -a log.txt   # Ajouter au fichier
```

## 🚪 Sortie & Sessions

### Quitter Sessions
```bash
# Méthodes de sortie
exit                      # Quitter shell (propre)
logout                    # Logout (shells login seulement)
Ctrl+D                    # EOF, quitte shell
Ctrl+C                    # Interrompre commande courante
Ctrl+Z                    # Suspendre processus (bg/fg)

# Codes de sortie
exit 0                    # Succès
exit 1                    # Erreur générale
echo $?                   # Code retour dernière commande
```

### Gestion Processus Basique
```bash
# Contrôle processus
Ctrl+C                    # SIGTERM (arrêt)
Ctrl+Z                    # SIGSTOP (suspension)
bg                        # Processus en arrière-plan
fg                        # Processus au premier plan
jobs                      # Processus suspendus/arrière-plan

# Exécution arrière-plan
commande &                # Lancer en arrière-plan
nohup commande &          # Résistant à déconnexion
```

## 🔧 Configuration Shell

### Fichiers Configuration Bash
| Fichier | Portée | Quand Exécuté |
|---------|--------|---------------|
| `/etc/profile` | Système | Login shells |
| `~/.bash_profile` | Utilisateur | Login bash |
| `~/.bashrc` | Utilisateur | Shells non-login |
| `~/.bash_logout` | Utilisateur | Sortie login shell |

### Personnalisation Prompt
```bash
# Variables PS1 courantes
\u                        # Nom utilisateur
\h                        # Hostname (court)
\w                        # Répertoire courant (chemin complet)
\W                        # Répertoire courant (nom seulement)
\$                        # $ ou # selon privilèges

# Exemples PS1
PS1='\u@\h:\w\$ '         # user@host:/path$
PS1='[\t] \u@\h:\w\$ '    # [14:30:00] user@host:/path$
```

## 💡 Bonnes Pratiques

### Sécurité
- ✅ **Vérifier commandes** avec `which`/`type` avant exécution
- ✅ **Utiliser alias sécurisés** (`rm -i`, `cp -i`)
- ✅ **Éviter** mots de passe en ligne de commande (historique)
- ✅ **Nettoyer historique** sensible avec `history -d`

### Efficacité
- ✅ **Maîtriser** Ctrl+R pour recherche historique
- ✅ **Créer alias** pour commandes fréquentes
- ✅ **Utiliser tab** pour auto-complétion
- ✅ **Apprendre pipes** pour chaîner commandes

### Organisation
- ✅ **Documenter** alias complexes dans ~/.bashrc
- ✅ **Organiser** variables environnement dans ~/.profile
- ✅ **Sauvegarder** configurations importantes
- ✅ **Tester** modifications dans nouvelle session

---
**💡 Memo** : `man` pour aide détaillée, `Ctrl+R` pour historique, `which` pour localiser, alias pour efficacité !