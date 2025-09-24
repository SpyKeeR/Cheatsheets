# 🌐 Apache HTTP Server — Aide-mémoire

## 🏗️ Architecture & Concepts

### Modules de Traitement (MPM)
| MPM | Usage | Avantages | Inconvénients |
|-----|-------|-----------|---------------|
| **Event** | Moderne (défaut) | Haute performance, async | Complexité debug |
| **Worker** | Threaded legacy | Bon compromis | Stabilité threads |
| **Prefork** | Processus séparés | Compatible mod_php | Consommation mémoire |

### Structure Configuration
```
/etc/apache2/
├── apache2.conf          # Configuration principale
├── ports.conf            # Ports d'écoute
├── envvars              # Variables environnement
├── sites-available/      # VHosts disponibles
├── sites-enabled/        # VHosts actifs (symlinks)
├── mods-available/       # Modules disponibles
├── mods-enabled/         # Modules actifs (symlinks)
├── conf-available/       # Configurations additionnelles
└── conf-enabled/         # Configurations actives

/var/www/html/           # DocumentRoot par défaut
/var/log/apache2/        # Logs (access.log, error.log)
```

## 📦 Installation & Gestion Service

### Installation & Démarrage
```bash
# Debian/Ubuntu
apt update && apt install apache2 apache2-utils

# Red Hat/CentOS
yum install httpd httpd-tools
# ou: dnf install httpd httpd-tools

# Gestion service
systemctl enable apache2     # Auto-start
systemctl start apache2      # Démarrer
systemctl status apache2     # État
systemctl reload apache2     # Recharger config
systemctl restart apache2    # Redémarrer complet

# Commandes Apache spécifiques
apache2ctl configtest       # Test configuration
apache2ctl graceful         # Rechargement doux
apache2ctl -S               # Liste VirtualHosts
apache2ctl -M               # Modules chargés
apache2ctl -V               # Version & options compilation
```

### Informations Système
```bash
# Version et modules
apache2 -v                  # Version
apache2 -l                  # Modules compilés
apache2ctl -M               # Modules chargés
a2enmod -l                  # Modules disponibles

# Processus et performance  
apache2ctl status           # Statut (si mod_status actif)
ps aux | grep apache2       # Processus actifs
ss -tlnp | grep :80         # Vérifier écoute ports
```

## ⚙️ Gestion Modules

### Modules Essentiels
```bash
# Activation modules courants
a2enmod rewrite             # Réécriture URL (.htaccess)
a2enmod ssl                 # Support HTTPS/TLS
a2enmod headers             # Manipulation headers HTTP  
a2enmod expires             # Cache navigateur
a2enmod deflate             # Compression gzip
a2enmod security2           # ModSecurity WAF (si installé)
a2enmod status              # Page statut serveur
a2enmod info                # Informations serveur

# Désactivation
a2dismod autoindex          # Listing répertoires
a2dismod server_status      # Si non utilisé en prod

# Application changements
systemctl reload apache2
```

### Modules par Catégorie
| Catégorie | Modules | Usage |
|-----------|---------|-------|
| **Core** | `mod_rewrite`, `mod_alias` | URL handling |
| **SSL/TLS** | `mod_ssl` | Chiffrement HTTPS |
| **Performance** | `mod_deflate`, `mod_expires`, `mod_cache` | Optimisation |
| **Sécurité** | `mod_security2`, `mod_evasive` | Protection |
| **Auth** | `mod_auth_basic`, `mod_auth_digest` | Authentification |
| **Développement** | `mod_php`, `mod_wsgi`, `mod_fcgid` | Langages |

## 🌍 VirtualHosts Configuration

### VirtualHost HTTP Basique
```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example
    
    # Logs spécifiques
    CustomLog ${APACHE_LOG_DIR}/example_access.log combined
    ErrorLog ${APACHE_LOG_DIR}/example_error.log
    
    # Options répertoire
    <Directory /var/www/example>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### VirtualHost HTTPS Sécurisé
```apache
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/example
    
    # SSL Configuration moderne
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/example.crt
    SSLCertificateKeyFile /etc/ssl/private/example.key
    SSLCertificateChainFile /etc/ssl/certs/chain.pem
    
    # Protocoles et ciphers sécurisés (2024)
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:!aNULL:!MD5:!DSS
    SSLHonorCipherOrder off   # TLS 1.3 compatible
    
    # Security headers
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    
    <Directory /var/www/example>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Redirections HTTP → HTTPS
```apache
# Méthode 1: Redirect simple
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>

# Méthode 2: RewriteRule (plus flexible)
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/example
    
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```

### Gestion Sites
```bash
# Activer/Désactiver sites
a2ensite example.conf        # Activer site
a2dissite 000-default.conf   # Désactiver site par défaut
a2ensite default-ssl.conf    # Activer SSL par défaut

# Vérifier configuration avant activation
apache2ctl configtest

# Appliquer changements
systemctl reload apache2

# Lister sites actifs
apache2ctl -S
```

## 🔒 Sécurité & Durcissement

### Configuration Sécurisée Globale
```apache
# /etc/apache2/conf-available/security.conf

# Masquer informations serveur
ServerTokens Prod
ServerSignature Off

# Désactiver méthodes dangereuses
TraceEnable Off

# Limites requêtes
LimitRequestBody 10485760      # 10MB max upload
Timeout 60
KeepAliveTimeout 5

# Headers de sécurité globaux
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-Content-Type-Options "nosniff"
Header always set X-XSS-Protection "1; mode=block"
Header always unset X-Powered-By
Header always unset Server

# Protection fichiers sensibles
<FilesMatch "^\.ht">
    Require all denied
</FilesMatch>

<FilesMatch "\.(conf|log|bak|backup|old)$">
    Require all denied
</FilesMatch>
```

### SSL/TLS Optimal
```apache
# Configuration SSL moderne (/etc/apache2/mods-available/ssl.conf)

# Protocoles sécurisés uniquement
SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1

# Suites de chiffrement modernes
SSLCipherSuite ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!MD5:!DSS:!RC4
SSLHonorCipherOrder off       # Laisser client choisir (TLS 1.3)

# Session et cache
SSLSessionCache shmcb:${APACHE_RUN_DIR}/ssl_scache(512000)
SSLSessionCacheTimeout 300

# OCSP Stapling
SSLUseStapling on
SSLStaplingCache shmcb:${APACHE_RUN_DIR}/ocsp_cache(128000)

# Compression SSL désactivée (attaque CRIME)
SSLCompression off

# Perfect Forward Secrecy
SSLOpenSSLConfCmd DHParameters /etc/ssl/dhparam.pem
```

### Contrôle d'Accès
```apache
# Restriction par IP
<Directory /var/www/admin>
    Require ip 192.168.1.0/24
    Require ip 10.0.0.100
</Directory>

# Authentification HTTP Basic
<Directory /var/www/private>
    AuthType Basic
    AuthName "Zone Privée"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>

# Authentification + IP (ET logique)
<Directory /var/www/secure>
    <RequireAll>
        Require ip 192.168.1.0/24
        Require valid-user
    </RequireAll>
</Directory>

# Alternative: IP OU authentification (OU logique)
<Directory /var/www/flexible>
    <RequireAny>
        Require ip 192.168.1.0/24
        Require valid-user
    </RequireAny>
</Directory>
```

### Protection Applicative
```apache
# Bloquer exécution PHP dans uploads
<Directory /var/www/uploads>
    php_admin_flag engine off
    AddType text/plain .php .php3 .phtml .pht
    
    # Headers pour forcer téléchargement
    <FilesMatch "\.(php|phtml|php3)$">
        Header set Content-Disposition "attachment"
    </FilesMatch>
</Directory>

# Rate limiting (mod_evasive si installé)
<IfModule mod_evasive24.c>
    DOSHashTableSize    2048
    DOSPageCount        10
    DOSPageInterval     1
    DOSSiteCount        50
    DOSSiteInterval     1
    DOSBlockingPeriod   600
</IfModule>
```

## 🚀 Performance & Optimisation

### Tuning MPM
```apache
# MPM Event (production moderne)
<IfModule mpm_event_module>
    StartServers             3
    MinSpareThreads          75
    MaxSpareThreads          250  
    ThreadsPerChild          25
    MaxRequestWorkers        400
    MaxConnectionsPerChild   10000
    ThreadLimit              64
</IfModule>

# MPM Prefork (avec mod_php)
<IfModule mpm_prefork_module>
    StartServers             5
    MinSpareServers          5
    MaxSpareServers          10
    MaxRequestWorkers        150
    MaxConnectionsPerChild   10000
</IfModule>
```

### Compression & Cache
```apache
# Compression (mod_deflate)
<IfModule mod_deflate.c>
    # Types MIME à compresser
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/xml
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE text/javascript
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE application/xhtml+xml
    AddOutputFilterByType DEFLATE application/rss+xml
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/x-javascript
    AddOutputFilterByType DEFLATE application/json
    
    # Exclure images (déjà compressées)
    SetEnvIfNoCase Request_URI \
        \.(?:gif|jpe?g|png|ico|woff|woff2)$ no-gzip dont-vary
        
    # Niveau compression
    DeflateCompressionLevel 6
</IfModule>

# Cache navigateur (mod_expires)
<IfModule mod_expires.c>
    ExpiresActive On
    
    # Ressources statiques
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/jpg "access plus 1 year"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/gif "access plus 1 year"
    ExpiresByType image/ico "access plus 1 year"
    ExpiresByType font/woff "access plus 1 year"
    ExpiresByType font/woff2 "access plus 1 year"
    
    # Documents
    ExpiresByType text/html "access plus 1 hour"
    ExpiresByType application/pdf "access plus 1 month"
</IfModule>
```

### Optimisations Système
```bash
# Limites processus Apache
echo "www-data soft nofile 65536" >> /etc/security/limits.conf
echo "www-data hard nofile 65536" >> /etc/security/limits.conf

# Paramètres réseau
echo 'net.core.somaxconn = 65536' >> /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65536' >> /etc/sysctl.conf
echo 'net.ipv4.tcp_tw_reuse = 1' >> /etc/sysctl.conf
sysctl -p

# Variables Apache (dans /etc/apache2/envvars si nécessaire)
export APACHE_ULIMIT_MAX_FILES="ulimit -n 65536"
```

## 📄 .htaccess Essentiels

### Réécritures URL Communes
```apache
# .htaccess dans DocumentRoot
RewriteEngine On

# Forcer HTTPS
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# Supprimer www
RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
RewriteRule ^(.*)$ https://%1/$1 [R=301,L]

# Clean URLs (WordPress style)
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]

# API routing
RewriteRule ^api/(.*)$ /api/index.php?request=$1 [QSA,L]

# Redirections spécifiques
Redirect 301 /old-page /new-page
Redirect 301 /old-section/ /new-section/
```

### Sécurité .htaccess
```apache
# Bloquer bots malicieux
RewriteCond %{HTTP_USER_AGENT} (badbot|scrapy|wget) [NC]
RewriteRule .* - [F,L]

# Bloquer IPs
<RequireAll>
    Require all granted
    Require not ip 192.168.1.100
    Require not ip 10.0.0.0/8
</RequireAll>

# Protection hotlinking images
RewriteCond %{HTTP_REFERER} !^https?://(.+\.)?example\.com [NC]
RewriteCond %{HTTP_REFERER} !^$
RewriteRule \.(jpe?g|png|gif)$ - [F]

# Limiter méthodes HTTP
<LimitExcept GET POST HEAD>
    Require all denied
</LimitExcept>
```

## 📊 Logging & Monitoring

### Configuration Logs
```apache
# Formats de logs personnalisés
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\" %D" combined_with_time
LogFormat "%h %l %u %t \"%r\" %>s %O %{ms}T" performance
LogFormat "%{%Y-%m-%d %H:%M:%S}t %h \"%r\" %>s %b" simple

# Application dans VirtualHost
CustomLog ${APACHE_LOG_DIR}/example_access.log combined_with_time
ErrorLog ${APACHE_LOG_DIR}/example_error.log

# Niveaux de log pour debug
LogLevel warn
LogLevel rewrite:trace3        # Debug réécriture URL temporaire
```

### Module mod_status
```apache
# Activation monitoring
<Location "/server-status">
    SetHandler server-status
    Require ip 127.0.0.1
    Require ip 192.168.1.0/24
</Location>

<Location "/server-info">
    SetHandler server-info
    Require ip 127.0.0.1
</Location>

ExtendedStatus On
```

### Analyse Logs Pratique
```bash
# Top 10 IPs visiteurs
tail -n 10000 /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10

# Pages les plus demandées
awk '{print $7}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head -10

# Erreurs 404
grep " 404 " /var/log/apache2/access.log | awk '{print $7}' | sort | uniq -c | sort -nr

# Codes de réponse
awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -nr

# Bande passante par heure
awk '{print $4, $10}' /var/log/apache2/access.log | awk -F'[:/]' '{print $1":"$2":"$3":"$4, $NF}' | awk '{sum[$1] += $2} END {for (i in sum) print i, sum[i]}' | sort

# User-Agents suspects
grep -i "bot\|crawl\|spider\|scan" /var/log/apache2/access.log | awk '{print $12}' | sort | uniq -c | sort -nr
```

## 🔧 Troubleshooting

### Diagnostics Configuration
```bash
# Test configuration complète
apache2ctl configtest
apache2ctl -t

# Afficher VirtualHosts et leur config
apache2ctl -S
apache2ctl -t -D DUMP_VHOSTS

# Vérifier modules chargés
apache2ctl -M | grep -i ssl
apache2ctl -M | grep -i rewrite

# Test connectivité
curl -I http://localhost
curl -k -I https://localhost

# Vérifier processus
ps aux | grep apache2
pstree -p | grep apache
```

### Problèmes Courants
| Erreur | Symptôme | Solution |
|--------|----------|----------|
| **AH00558** | ServerName warning | Ajouter `ServerName localhost` dans apache2.conf |
| **AH01630** | SSL handshake failed | Vérifier certificats SSL avec `openssl` |
| **AH00526** | Invalid command | Module non activé (`a2enmod`) |
| **Permission denied** | 403 Forbidden | Vérifier droits `chown -R www-data:www-data` |
| **Port already in use** | Bind échoue | Autre service sur port (`ss -tlnp \| grep :80`) |

### Debug SSL/TLS
```bash
# Test certificat serveur
openssl s_client -connect example.com:443 -servername example.com

# Vérifier certificats locaux
openssl x509 -in /etc/ssl/certs/example.crt -text -noout
openssl x509 -in /etc/ssl/certs/example.crt -noout -dates

# Test cipher suites
nmap --script ssl-enum-ciphers -p 443 example.com
curl -I --tlsv1.2 https://example.com
curl -I --tlsv1.3 https://example.com

# Validité chaîne certificats
openssl verify -CAfile /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/example.crt
```

### Performance Debugging
```bash
# Processus et mémoire
ps aux --sort=-%mem | grep apache2
top -p $(pgrep apache2 | tr '\n' ',' | sed 's/,$//')

# Connexions actives
ss -tn state established '( sport = :80 or sport = :443 )'

# Load average et I/O
uptime
iostat -x 1 5

# Monitoring temps réel avec mod_status
watch -n 2 'curl -s http://localhost/server-status?auto'
```

## 💡 Bonnes Pratiques

### Sécurité Production
- ✅ **Masquer version** : `ServerTokens Prod`
- ✅ **HTTPS uniquement** : Rediriger HTTP → HTTPS
- ✅ **Headers sécurité** : HSTS, CSP, X-Frame-Options
- ✅ **Limitation uploads** : `LimitRequestBody`
- ✅ **Protection fichiers** : `.htaccess`, `.conf`, etc.
- ✅ **Authentification forte** : Éviter Basic sur HTTP

### Performance Production  
- ✅ **MPM approprié** : Event pour sites modernes
- ✅ **Compression** : mod_deflate pour texte/CSS/JS
- ✅ **Cache navigateur** : mod_expires pour statiques
- ✅ **Keep-Alive** : Optimiser `KeepAliveTimeout`
- ✅ **Monitoring** : mod_status + logs analysis

### Maintenance
- ✅ **Logs rotation** : logrotate configuré
- ✅ **Sauvegarde config** : `/etc/apache2/` versionné
- ✅ **Tests réguliers** : `apache2ctl configtest`
- ✅ **Mises à jour** : Patchs sécurité Apache
- ✅ **Monitoring** : Alertes sur erreurs 5xx

---
**💡 Memo** : `apache2ctl configtest` avant reload, compression + cache = performance, HTTPS + headers = sécurité !