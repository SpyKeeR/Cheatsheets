# 🔒 Permissions Fichiers Linux — Aide-mémoire

## 🏗️ Architecture Permissions

### Modèle de Sécurité UNIX
```
Chaque fichier/répertoire a :
├── Propriétaire (User) - u
├── Groupe (Group) - g  
└── Autres (Others) - o

Chaque entité a 3 permissions :
├── r (Read) - 4
├── w (Write) - 2
└── x (eXecute) - 1
```

### Affichage Permissions
```bash
ls -l fichier.txt
# -rw-r--r-- 1 user group 1234 Dec 15 10:30 fichier.txt
#  ↑↑↑ ↑↑↑ ↑↑↑
#   u   g   o

# Décodage : - rw- r-- r--
# Type: - (fichier), d (dossier), l (lien), etc.
# User: rw- (lecture + écriture)
# Group: r-- (lecture seule)
# Others: r-- (lecture seule)
```

## 🎯 Permissions Standard

### Impact par Type d'Objet
| Permission | Fichier | Répertoire |
|------------|---------|------------|
| **r (4)** | Lire contenu (`cat`, `less`) | Lister fichiers (`ls`) |
| **w (2)** | Modifier/supprimer fichier | Créer/supprimer fichiers |
| **x (1)** | Exécuter script/binaire | Traverser (`cd`) dans dossier |

### Combinaisons Courantes
| Octal | Binaire | Symbolique | Usage |
|-------|---------|------------|-------|
| **7** | 111 | `rwx` | Propriétaire complet |
| **6** | 110 | `rw-` | Lecture/écriture |
| **5** | 101 | `r-x` | Lecture/exécution |
| **4** | 100 | `r--` | Lecture seule |
| **0** | 000 | `---` | Aucun droit |

### Permissions Type Standard
```bash
# Fichiers
644 (rw-r--r--) # Fichier standard utilisateur
600 (rw-------) # Fichier privé (mots de passe)
755 (rwxr-xr-x) # Script exécutable
444 (r--r--r--) # Fichier lecture seule

# Répertoires  
755 (rwxr-xr-x) # Dossier standard
750 (rwxr-x---) # Dossier groupe seulement
700 (rwx------) # Dossier privé
```

## 🔧 Modification Permissions

### chmod (Change Mode)
```bash
# Mode octal
chmod 755 fichier               # Permissions explicites
chmod 644 *.txt                # Wildcard sur plusieurs fichiers
chmod -R 755 dossier/          # Récursif (-R)

# Mode symbolique
chmod u+x script.sh            # Ajouter exécution propriétaire
chmod g-w fichier              # Retirer écriture groupe
chmod o=r fichier              # Définir lecture seule autres
chmod a+r fichier              # Ajouter lecture tous (a = all)

# Combinaisons
chmod u+rw,g+r,o-rwx fichier   # Multiple changements
chmod +x script                # Ajouter exécution à tous
chmod -w fichier               # Retirer écriture à tous
```

### Propriétaires (chown/chgrp)
```bash
# Changer propriétaire
chown user fichier             # Nouveau propriétaire
chown user:group fichier       # Propriétaire + groupe
chown :group fichier           # Groupe seulement
chown -R user:group dossier/   # Récursif

# Changer groupe seulement
chgrp group fichier            # Nouveau groupe
chgrp -R group dossier/        # Récursif

# Exemples pratiques
chown www-data:www-data /var/www/html/  # Web server
chown root:wheel /usr/local/bin/tool    # Admin tool
```

## ⚡ Permissions Spéciales

### Bits Spéciaux (4ème chiffre octal)
| Bit | Valeur | Nom | Sur Fichier | Sur Répertoire |
|-----|--------|-----|-------------|----------------|
| **4xxx** | setuid | SUID | Exécute avec droits propriétaire | Aucun effet |
| **2xxx** | setgid | SGID | Exécute avec droits groupe | Héritage groupe parent |
| **1xxx** | sticky | Sticky | Obsolète (était: garde en mémoire) | Seul propriétaire peut supprimer |

### Affichage Bits Spéciaux
```bash
# Dans ls -l, remplace x par :
s/S # setuid/setgid (s=avec x, S=sans x)
t/T # sticky bit (t=avec x, T=sans x)

# Exemples système
-rwsr-xr-x  # SUID activé (/usr/bin/passwd)
-rwxr-sr-x  # SGID activé  
drwxrwxrwt  # Sticky bit (/tmp)
```

### Configuration Bits Spéciaux
```bash
# setuid (4000)
chmod 4755 programme           # SUID + 755
chmod u+s programme            # Ajouter SUID

# setgid (2000)  
chmod 2755 dossier             # SGID + 755
chmod g+s dossier              # Ajouter SGID

# sticky bit (1000)
chmod 1777 /tmp                # Sticky + 777
chmod +t dossier               # Ajouter sticky
```

## 🛡️ umask (Masque Permissions)

### Principe umask
```
umask = permissions par défaut à SOUSTRAIRE
Fichiers max: 666 (rw-rw-rw-)
Dossiers max: 777 (rwxrwxrwx)

Calcul:
umask 022:
- Fichiers: 666 - 022 = 644 (rw-r--r--)
- Dossiers: 777 - 022 = 755 (rwxr-xr-x)

umask 077:
- Fichiers: 666 - 077 = 600 (rw-------)
- Dossiers: 777 - 077 = 700 (rwx------)
```

### Gestion umask
```bash
# Consulter umask actuel
umask                          # Format octal (ex: 0022)
umask -S                       # Format symbolique (ex: u=rwx,g=rx,o=rx)

# Modifier umask
umask 027                      # Temporaire (session)
umask u=rwx,g=rx,o=           # Format symbolique

# Test création
touch test_file; ls -l test_file    # Vérifier permissions appliquées
mkdir test_dir; ls -ld test_dir     # Vérifier sur répertoire
```

### Configuration Persistante umask
| Fichier | Portée | Usage |
|---------|--------|-------|
| `~/.bashrc` | Utilisateur | Sessions shell futures |
| `~/.profile` | Utilisateur | Sessions login |
| `/etc/profile` | Global | Tous utilisateurs |
| `/etc/login.defs` | Global | Défaut nouveaux comptes |
| `/etc/pam.d/common-session` | PAM | Via `pam_umask.so` |

## 🔍 Consultation & Diagnostic

### Commandes d'Analyse
```bash
# Informations détaillées
ls -la                         # Tout afficher (inclut cachés)
ls -ld dossier/                # Permissions du dossier lui-même
stat fichier                   # Informations complètes (times, inode, etc.)

# Recherche par permissions
find /path -perm 755           # Permissions exactes
find /path -perm -755          # Au moins ces permissions
find /path -perm /755          # Au moins une de ces permissions
find /home -perm 4000          # Fichiers SUID
find /tmp -perm 1000           # Fichiers sticky

# Recherche par propriétaire
find /path -user john          # Propriétaire spécifique
find /path -group admin        # Groupe spécifique
find /path -nouser             # Propriétaire inexistant
find /path -nogroup            # Groupe inexistant
```

### Tests Permissions
```bash
# Tester accès
test -r fichier && echo "Lisible"
test -w fichier && echo "Écrivable"  
test -x fichier && echo "Exécutable"

# Dans scripts
if [[ -r "$file" ]]; then
    echo "Peut lire $file"
fi

# Accès effectif utilisateur
access fichier                 # Commande si disponible
sudo -u user test -w fichier   # Test en tant qu'autre user
```

## 🎯 Cas d'Usage Pratiques

### Web Server (Apache/Nginx)
```bash
# Structure recommandée
chown -R www-data:www-data /var/www/html/
chmod -R 755 /var/www/html/           # Dossiers
find /var/www/html -type f -exec chmod 644 {} \;  # Fichiers
chmod 600 /var/www/html/config.php   # Fichiers sensibles
```

### Partage Collaboratif
```bash
# Groupe de travail
groupadd developers
usermod -aG developers alice,bob,charlie

# Dossier partagé avec SGID
mkdir /shared/project
chgrp developers /shared/project
chmod 2775 /shared/project            # SGID + rwxrwxr-x
# Nouveaux fichiers hériteront du groupe "developers"
```

### Scripts & Binaires
```bash
# Script utilisateur
chmod 755 ~/.local/bin/monscript     # Exécutable par tous
chmod 700 ~/bin/script-prive         # Exécutable par propriétaire seul

# Installation système
sudo cp tool /usr/local/bin/
sudo chmod 755 /usr/local/bin/tool
sudo chown root:root /usr/local/bin/tool
```

### Répertoire Temporaire Sécurisé
```bash
# Style /tmp avec sticky bit
mkdir /shared/temp
chmod 1777 /shared/temp              # Tous peuvent créer, seul propriétaire supprime
# Alternative plus restrictive
chmod 1770 /shared/temp              # Seul groupe peut créer
```

## 🚨 Sécurité & Bonnes Pratiques

### Principes de Sécurité
- ✅ **Moindre privilège** : Permissions minimales nécessaires
- ✅ **Séparation duties** : Comptes dédiés par service
- ✅ **Audit régulier** : Vérifier permissions sensibles
- ✅ **Nettoyage** : Supprimer comptes/fichiers obsolètes

### Permissions Dangereuses à Surveiller
```bash
# Fichiers SUID root (exécutent avec privilèges root)
find / -perm -4000 -user root 2>/dev/null

# Fichiers monde-écrivable
find / -perm -002 -type f 2>/dev/null

# Répertoires monde-écrivable sans sticky
find / -perm -002 -type d ! -perm -1000 2>/dev/null

# Fichiers sans propriétaire
find / -nouser -o -nogroup 2>/dev/null
```

### Exemples Sécurisés
```bash
# SSH keys
chmod 700 ~/.ssh/                    # Répertoire privé
chmod 600 ~/.ssh/id_rsa             # Clé privée
chmod 644 ~/.ssh/id_rsa.pub         # Clé publique
chmod 600 ~/.ssh/authorized_keys    # Clés autorisées

# Fichiers système critiques  
chmod 644 /etc/passwd               # Lisible par tous
chmod 640 /etc/shadow               # Groupe shadow seulement
chmod 600 /etc/sudoers              # Root seul
chmod 755 /etc/init.d/*             # Scripts init
```

### Dépannage Courant
```bash
# Permission denied sur script
chmod +x script.sh                  # Ajouter exécution

# Impossible cd dans répertoire  
chmod +x dossier/                   # Ajouter traverse (x)

# Impossible lister contenu répertoire
chmod +r dossier/                   # Ajouter lecture (r)

# Web server 403 Forbidden
chmod 755 /var/www/html/            # Répertoire traversable
chmod 644 /var/www/html/index.html  # Fichier lisible
```

---
**💡 Memo** : `chmod 755` pour exécutables, `644` pour fichiers, `700`/`600` pour privé, `find` avec `-perm` pour audit !