# Linux — Shell & commandes essentielles (condensé) 🐧

## Sessions & sortie
- `exit` → quitter le shell.
- `logout` → logout (bash compatible).
- `Ctrl+D` → EOF, quitte le shell.

## Streams
- stdin (0) → entrée standard (clavier).  
- stdout (1) → sortie normale (écran).  
- stderr (2) → sortie d'erreur.

## Qui / identité
- `logname` → nom utilisateur courant.  
- `id` → uid/gid/groupes.  
- `id -u user` → uid d’un autre user.  
- `who` / `who -H` / `who -q` → qui est connecté.  
- `last` → historique connexions (/var/log/wtmp).

## Mot de passe
- `passwd` → changer mot de passe.  
- `passwd -S` → état du mot de passe.  
- `passwd -e` → forcer expiration.

## Date / heure / calendrier
- `date` → date système.  
- `date +"%A, %d %B %Y%nIl est %H:%M"` → format perso.  
- `cal` / `cal 2025` / `cal 04 2025`.

## Historique
- `history` → historique.  
- `!!` → dernière commande.  
- `!apt` → dernier débutant par apt.  
- `!?passwd` → dernier contenant passwd.  
- `!42` → commande n°42.  
- `Ctrl+R` → recherche rétroactive.  
- `~/.bash_history`, `$HISTFILE`, `$HISTSIZE`.

## Aide & manuel
- `commande --help` → aide rapide.  
- `man commande` → manuel (sections 1..8).  
- `man -s 5 passwd` → Spécifier une section.
- `whatis` (`man -f`) → Affiche toutes les sections de man d'une cmd.
- `whereis` → Trouver le binaire, les sources et le pages de manuel
- `apropos` (`man -k`) → Recherche un mot-clé pour trouver une cmd.
- `info` : Permet d'obtenir un guide un peu plus narratif.

## Utilitaires shell
- `which cmd` → Affiche le chemin complet de la commande
- `alias NAME='cmd'` ; `source ~/.bashrc` ; `unalias NAME`.
- `file` → Déterminer le type de fichier
- `type` → Déterminer le type de commande (built-in/hashed/binaire)

## Navigation et Structure du manuel
- /mot → recherche dans la page
- n → résultat suivant
- q → quitter

### Sections
- Section 1 → Commandes utilisateurs
- Section 2 → Appels systèmes
- Section 3 → Fonctions des libraries
- Section 4 → Fichiers spéciaux
- Section 5 → Fichiers de configs
- Section 6 → Jeux
- Section 7 → Conventions, bonnes pratiques, RFC, autres...
- Section 8 → Commandes d'administration

## Variables & environnement
- `$HOME`, `$USER`, `$PATH`, `$SHELL`, `$HOSTNAME`...  
- `printenv` / `env` / `set` (Verbeux)  → lister variables.  
- `export VAR="val"` → variable exportée (session).  
- `unset VAR` → supprimer.  
- Persistance : `~/.bashrc`, `~/.profile`, `/etc/environment`.  
- Substitution commande : `heure=$(date +"%H:%M")`.