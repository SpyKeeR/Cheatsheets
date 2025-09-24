# 👥 Utilisateurs & Groupes Linux — Aide-mémoire

## 🏗️ Architecture & Concepts

### Types d'Utilisateurs
| Type | UID | Description | Exemples |
|------|-----|-------------|----------|
| **root** | 0 | Super-utilisateur | Administrateur système |
| **Système** | 1-999 | Services et démons | www-data, mysql, postfix |
| **Utilisateurs** | ≥1000 | Comptes humains | users, développeurs |
| **Nobody** | 65534 | Utilisateur sans privilège | Processus isolés |

### Groupes & Appartenance
- **Groupe primaire** : Groupe principal de l'utilisateur (défini dans `/etc/passwd`)
- **Groupes secondaires** : Groupes additionnels (définis dans `/etc/group`)
- **Héritage** : Nouveaux fichiers créés avec GID du groupe primaire
- **Limite** : Maximum 16 groupes secondaires simultanés (historique), maintenant plus élevé

## 📁 Fichiers de Configuration

### /etc/passwd (Informations Utilisateurs)
```bash
# Format: username:password:UID:GID:GECOS:homedir:shell
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
maxime:x:1001:1001:Maxime Dupont,,,:/home/maxime:/bin/bash
```

| Champ | Description | Exemple |
|-------|-------------|---------|
| **username** | Nom de connexion | maxime |
| **password** | x = mot de passe dans /etc/shadow | x |
| **UID** | Identifiant utilisateur unique | 1001 |
| **GID** | Groupe primaire | 1001 |
| **GECOS** | Infos utilisateur (nom, téléphone) | Maxime Dupont |
| **homedir** | Répertoire personnel | /home/maxime |
| **shell** | Shell par défaut | /bin/bash |

### /etc/shadow (Mots de Passe Sécurisés)
```bash
# Format: username:password:lastchg:min:max:warn:inactive:expire:reserved
maxime:$6$salt$hash:19000:0:99999:7:30:19365:
```

| Champ | Description | Valeurs Courantes |
|-------|-------------|-------------------|
| **password** | Hash chiffré ou état | $6$... (SHA-512), ! (verrouillé) |
| **lastchg** | Jours depuis 1970 du dernier changement | 19000 |
| **min** | Délai minimum entre changements (jours) | 0 (aucun délai) |
| **max** | Validité maximum mot de passe (jours) | 99999 (pas d'expiration) |
| **warn** | Avertissement avant expiration (jours) | 7 |
| **inactive** | Délai après expiration avant verrouillage | 30 |
| **expire** | Date expiration compte (jours depuis 1970) | 19365 |

### /etc/group (Informations Groupes)
```bash
# Format: groupname:password:GID:members
root:x:0:
sudo:x:27:maxime,alice
developers:x:1100:maxime,bob,charlie
```

### /etc/gshadow (Sécurité Groupes)
```bash
# Format: groupname:password:admins:members
sudo:!::maxime,alice
developers:!:maxime:maxime,bob,charlie
```

## 👤 Gestion Utilisateurs

### Création Utilisateurs
```bash
# Création basique
useradd username                    # Utilisateur minimal
useradd -m username                 # Avec répertoire home
useradd -m -s /bin/bash username    # Avec home + shell

# Création complète
useradd -m -u 1500 -g users -G sudo,developers \
        -s /bin/bash -c "John Doe" \
        -e 2025-12-31 john

# Options principales
useradd [options] username
  -m                # Créer répertoire home
  -u UID           # UID spécifique
  -g groupe        # Groupe primaire
  -G grp1,grp2     # Groupes secondaires
  -s /bin/bash     # Shell par défaut
  -c "commentaire" # GECOS (nom complet)
  -d /path/home    # Répertoire home personnalisé
  -e YYYY-MM-DD    # Date expiration
  -r               # Compte système (UID < 1000)
```

### Modification Utilisateurs
```bash
# Changements courants
usermod -aG sudo username           # Ajouter au groupe sudo
usermod -G groupe1,groupe2 user     # Remplacer tous groupes secondaires
usermod -g newgroup user            # Changer groupe primaire
usermod -s /bin/zsh user            # Changer shell
usermod -c "Nouveau nom" user       # Modifier GECOS
usermod -d /new/home -m user        # Déplacer home (-m = move files)
usermod -e 2025-06-30 user          # Définir expiration compte
usermod -L user                     # Verrouiller compte
usermod -U user                     # Déverrouiller compte

# Changement nom utilisateur (délicat)
usermod -l newname oldname          # Changer nom de login
groupmod -n newname oldname         # Renommer groupe primaire si nécessaire
```

### Suppression Utilisateurs
```bash
userdel username                    # Supprimer utilisateur (garder home)
userdel -r username                 # Supprimer + répertoire home
userdel -f username                 # Forcer suppression (même si connecté)

# Nettoyage manuel après suppression
find / -nouser -exec ls -l {} \;    # Trouver fichiers orphelins
find / -uid 1500 -exec chown newuser {} \;  # Réaffecter fichiers
```

## 🔒 Gestion Mots de Passe

### Commandes passwd
```bash
# Gestion mot de passe
passwd                              # Changer son propre mot de passe
passwd username                     # Changer mot de passe utilisateur (root)
passwd -l username                  # Verrouiller compte
passwd -u username                  # Déverrouiller compte
passwd -d username                  # Supprimer mot de passe (dangereux)
passwd -e username                  # Forcer changement au prochain login
passwd -S username                  # Statut mot de passe

# Génération mots de passe sécurisés
openssl rand -base64 12             # Mot de passe aléatoire 12 caractères
pwgen -s 16 1                       # Avec pwgen (si installé)
```

### Politique Mots de Passe
```bash
# Configurer expiration avec chage
chage username                      # Interface interactive
chage -l username                   # Afficher informations expiration
chage -d 0 username                 # Forcer changement immédiat
chage -M 90 username                # Maximum 90 jours
chage -m 7 username                 # Minimum 7 jours entre changements
chage -W 14 username                # Avertir 14 jours avant expiration
chage -E 2025-12-31 username        # Expiration compte

# Exemple politique stricte
chage -M 90 -m 7 -W 14 -I 30 username
```

## 👥 Gestion Groupes

### Opérations sur Groupes
```bash
# Création/suppression
groupadd groupname                  # Créer groupe
groupadd -g 1500 groupname         # Avec GID spécifique
groupadd -r systemgroup            # Groupe système
groupmod -n newname oldname        # Renommer groupe
groupmod -g 1600 groupname         # Changer GID
groupdel groupname                  # Supprimer groupe (si pas utilisé)

# Gestion membres avec gpasswd
gpasswd groupname                   # Définir mot de passe groupe
gpasswd -a username group           # Ajouter membre
gpasswd -d username group           # Retirer membre
gpasswd -A admin1,admin2 group      # Définir administrateurs groupe
gpasswd -r group                    # Supprimer mot de passe groupe
```

### Appartenance Groupes
```bash
# Consultation
id username                         # Informations complètes utilisateur
groups username                     # Groupes de l'utilisateur
getent group                        # Tous les groupes
getent group groupname              # Informations groupe spécifique
members groupname                   # Membres du groupe (si installé)

# Groupes de l'utilisateur actuel
id                                  # UID, GID, groupes
whoami                             # Nom utilisateur
```

## 🔐 Sudo & Privilèges

### Configuration sudo
```bash
# Édition fichier sudoers (TOUJOURS avec visudo)
visudo                              # Éditer /etc/sudoers avec vérification
visudo -f /etc/sudoers.d/custom    # Éditer fichier dans sudoers.d

# Syntaxe sudoers
# user  hosts=(runas) commands
maxime  ALL=(ALL:ALL) ALL           # Tous droits
%admin  ALL=(ALL) ALL               # Groupe admin
john    ALL=(root) /bin/systemctl   # Commande spécifique
alice   ALL=(ALL) NOPASSWD: ALL     # Sans mot de passe (dangereux)

# Exemples pratiques
%developers ALL=(www-data) /usr/bin/deploy
%backup ALL=(root) /bin/systemctl restart backup
```

### Utilisation sudo
```bash
sudo command                        # Exécuter avec privilèges root
sudo -u username command            # Exécuter en tant qu'autre utilisateur
sudo -g group command               # Exécuter avec autre groupe
sudo -s                             # Shell root
sudo -i                             # Login shell root
sudo -k                             # Oublier mot de passe sudo
sudo -l                             # Lister permissions sudo
sudo -v                             # Renouveler ticket sudo
```

## 🔍 Consultation & Diagnostic

### Informations Utilisateurs
```bash
# Qui est connecté
who                                 # Utilisateurs connectés
w                                   # Utilisateurs + activité
last                                # Historique connexions
lastlog                             # Dernière connexion par utilisateur
finger username                     # Infos détaillées (si installé)

# Processus utilisateur
ps -u username                      # Processus d'un utilisateur
pgrep -u username                   # PID processus utilisateur
pkill -u username                   # Tuer processus utilisateur
```

### Base de Données Système
```bash
# Consultation avec getent (NSS aware)
getent passwd                       # Tous utilisateurs (local + LDAP/NIS)
getent passwd username              # Utilisateur spécifique
getent group                        # Tous groupes
getent shadow                       # Mots de passe (root seulement)

# Validation cohérence
pwck                                # Vérifier /etc/passwd et /etc/shadow
grpck                               # Vérifier /etc/group et /etc/gshadow
```

## 🛡️ Sécurité & Bonnes Pratiques

### Comptes de Service
```bash
# Créer compte service sécurisé
useradd -r -d /var/lib/myservice \
        -s /usr/sbin/nologin \
        -c "MyService daemon" myservice

# Shell nologin pour interdire connexion interactive
/usr/sbin/nologin                   # Shell standard "pas de login"
/bin/false                          # Alternative plus restrictive
```

### Surveillance & Audit
```bash
# Comptes sans mot de passe
awk -F: '$2 == "" {print $1}' /etc/shadow

# Comptes avec UID 0 (danger si pas root)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Comptes expirés
awk -F: '$8 < '$(date +%s)'/86400 && $8 != "" {print $1}' /etc/shadow

# Fichiers appartenant à utilisateur supprimé
find / -nouser -ls 2>/dev/null

# Sessions actives
loginctl list-sessions              # Systemd
```

### Configuration Système
```bash
# Paramètres dans /etc/login.defs
PASS_MAX_DAYS   90                  # Expiration mot de passe
PASS_MIN_DAYS   7                   # Délai minimum changement
PASS_WARN_AGE   14                  # Avertissement expiration
UID_MIN         1000                # UID minimum utilisateurs
UID_MAX         60000               # UID maximum
UMASK           022                 # Permissions par défaut

# Squelette nouveaux utilisateurs
/etc/skel/                          # Template répertoire home
ls -la /etc/skel                    # Fichiers copiés pour nouveaux users
```

## 💡 Cas d'Usage Pratiques

### Migration Utilisateur
```bash
# Sauvegarder informations utilisateur
getent passwd olduser > user_backup.txt
getent group | grep olduser >> user_backup.txt
tar -czf olduser_home.tar.gz /home/olduser

# Recréer sur nouveau système
# (adapter UID/GID selon backup)
useradd -u 1001 -g 1001 -m -s /bin/bash newuser
```

### Nettoyage Système
```bash
# Trouver comptes inutilisés (pas connectés depuis 90 jours)
lastlog -b 90 | grep -v "Never"

# Lister utilisateurs système vs humains
awk -F: '$3 < 1000 {print "Système:", $1}' /etc/passwd
awk -F: '$3 >= 1000 && $3 != 65534 {print "Humain:", $1}' /etc/passwd

# Vérifier cohérence groupes primaires
awk -F: '{print $4}' /etc/passwd | sort -u | while read gid; do
    getent group | awk -F: -v gid="$gid" '$3 == gid {print gid, $1}'
done
```

### Scripts d'Administration
```bash
# Créer utilisateur développeur type
create_dev_user() {
    local username=$1
    local fullname=$2
    
    useradd -m -s /bin/bash -G developers,docker \
            -c "$fullname" "$username"
    
    passwd "$username"
    echo "Utilisateur $username créé et ajouté aux groupes developers, docker"
}

# Usage: create_dev_user "jdoe" "John Doe"
```

## 🔧 Dépannage Courant

### Problèmes Fréquents
| Problème | Symptôme | Solution |
|----------|----------|----------|
| **Compte verrouillé** | Login refusé | `passwd -u user` ou `usermod -U user` |
| **Mot de passe expiré** | Demande changement | `passwd user` ou `chage -d 0 user` |
| **Home inexistant** | Erreur à la connexion | `mkhomedir_helper user` ou copier /etc/skel |
| **Pas dans sudoers** | Permission denied | `usermod -aG sudo user` |
| **Groupe primaire supprimé** | Erreurs permissions | Recréer groupe ou `usermod -g newgroup user` |

### Commandes de Diagnostic
```bash
# Tester authentification
su - username                       # Test changement utilisateur
sudo -u username whoami            # Test sudo vers utilisateur

# Déboguer permissions
ls -la /home/username              # Vérifier permissions home
id username                        # Vérifier groupes
getent passwd username             # Vérifier existence dans NSS
```

---
**💡 Memo** : `visudo` pour éditer sudoers, `getent` pour consultation NSS-aware, `usermod -aG` pour ajouter aux groupes !