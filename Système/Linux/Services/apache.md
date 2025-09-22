# 🌐 Apache HTTP Server — Cheatsheet

## 📦 Installation & Démarrage

### Installation Debian/Ubuntu
```bash
# Installation
apt update && apt install apache2

# Démarrage & activation
systemctl enable apache2
systemctl start apache2
systemctl status apache2

# Version & modules
apache2 -v
apache2ctl -M
```

### Structure Fichiers
```
/etc/apache2/
├── apache2.conf          # Configuration principale
├── ports.conf            # Ports d'écoute
├── sites-available/      # VHosts disponibles
├── sites-enabled/        # VHosts actifs (symlinks)
├── mods-available/       # Modules disponibles
├── mods-enabled/         # Modules actifs (symlinks)
├── conf-available/       # Configs additionnelles
└── conf-enabled/         # Configs actives (symlinks)

/var/www/                 # Répertoire web par défaut
/var/log/apache2/         # Logs (access.log, error.log)
```

## ⚙️ Gestion Services & Modules

### Commandes Essentielles
```bash
# Test configuration
apache2ctl configtest

# Reload sans interruption
apache2ctl graceful
systemctl reload apache2

# Informations
apache2ctl -S              # Liste VirtualHosts
apache2ctl -M              # Modules chargés
apache2ctl -V              # Version & options compilation
```

### Gestion Modules
```bash
# Activer/Désactiver modules
a2enmod rewrite ssl headers expires
a2dismod autoindex status

# Modules essentiels
a2enmod rewrite           # URL rewriting
a2enmod ssl               # HTTPS/TLS
a2enmod headers           # Manipulation headers HTTP
a2enmod expires           # Cache expires
a2enmod deflate           # Compression gzip
a2enmod security2         # ModSecurity WAF
```

### Gestion Sites (VirtualHosts)
```bash
# Activer/Désactiver sites
a2ensite monsite.conf
a2dissite 000-default.conf

# Après changements
systemctl reload apache2
```

## 🏗️ Configuration VirtualHosts

### VirtualHost HTTP Simple
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

### VirtualHost HTTPS Complet
```apache
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example
    
    # SSL Configuration
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/example.pem
    SSLCertificateKeyFile /etc/ssl/private/example.key
    SSLCertificateChainFile /etc/ssl/certs/chain.pem
    
    # Sécurité SSL
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:!aNULL:!MD5:!DSS
    SSLHonorCipherOrder on
    
    # Headers sécurité
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    
    # Compression
    <Location />
        SetOutputFilter DEFLATE
        SetEnvIfNoCase Request_URI \
            \.(?:gif|jpe?g|png)$ no-gzip dont-vary
        SetEnvIfNoCase Request_URI \
            \.(?:exe|t?gz|zip|bz2|sit|rar)$ no-gzip dont-vary
    </Location>
    
    <Directory /var/www/example>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Redirection HTTP → HTTPS
```apache
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    
    # Redirection permanente
    Redirect permanent / https://example.com/
    
    # Alternative avec RewriteEngine
    # RewriteEngine On
    # RewriteCond %{HTTPS} off
    # RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```

## 🔧 Directives Importantes

### Contrôle d'Accès
```apache
# Accès global
<Directory /var/www>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

# Restriction par IP
<Directory /var/www/admin>
    Require ip 192.168.1.0/24
    Require ip 10.0.0.100
</Directory>

# Authentification basique
<Directory /var/www/private>
    AuthType Basic
    AuthName "Zone Privée"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>

# Combinaisons complexes
<Directory /var/www/complex>
    <RequireAll>
        Require ip 192.168.1.0/24
        Require valid-user
    </RequireAll>
</Directory>
```

### Configuration Performance
```apache
# Compression
LoadModule deflate_module modules/mod_deflate.so
<Location />
    SetOutputFilter DEFLATE
    DeflateCompressionLevel 6
    
    # Exclure certains types
    SetEnvIfNoCase Request_URI \
        \.(?:gif|jpe?g|png|ico)$ no-gzip dont-vary
</Location>

# Cache statique
LoadModule expires_module modules/mod_expires.so
ExpiresActive On
ExpiresByType text/css "access plus 1 month"
ExpiresByType application/javascript "access plus 1 month"
ExpiresByType image/png "access plus 1 year"
ExpiresByType image/jpg "access plus 1 year"
ExpiresByType image/jpeg "access plus 1 year"
ExpiresByType image/gif "access plus 1 year"
ExpiresByType image/ico "access plus 1 year"

# Keep-Alive
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
```

## 🔒 Sécurité & Durcissement

### Configuration Sécurisée
```apache
# Masquer version Apache
ServerTokens Prod
ServerSignature Off

# Sécurité headers
LoadModule headers_module modules/mod_headers.so
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-Content-Type-Options "nosniff"
Header always set X-XSS-Protection "1; mode=block"
Header always set Referrer-Policy "strict-origin-when-cross-origin"

# Désactiver TRACE
TraceEnable Off

# Limiter taille uploads
LimitRequestBody 10485760  # 10MB
```

### SSL/TLS Optimal
```apache
# Protocoles modernes uniquement
SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1

# Cipher suites sécurisées
SSLCipherSuite ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!MD5:!DSS
SSLHonorCipherOrder on

# Compression SSL désactivée (CRIME attack)
SSLCompression off

# OCSP Stapling
SSLUseStapling on
SSLStaplingCache shmcb:/var/run/ocsp(128000)

# Session cache
SSLSessionCache shmcb:/var/run/ssl_scache(512000)
SSLSessionCacheTimeout 300
```

### Protection Fichiers Sensibles
```apache
# Protéger .htaccess, .htpasswd
<FilesMatch "^\.ht">
    Require all denied
</FilesMatch>

# Protéger fichiers config
<FilesMatch "\.(inc|conf|config)$">
    Require all denied
</FilesMatch>

# Bloquer exécution PHP dans uploads
<Directory /var/www/uploads>
    php_admin_flag engine off
    AddType text/plain .php .php3 .phtml .pht
</Directory>
```

## 📄 .htaccess Utiles

### Réécritures URL Courantes
```apache
# .htaccess dans /var/www/site/
RewriteEngine On

# WWW vers non-WWW
RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
RewriteRule ^(.*)$ https://%1/$1 [R=301,L]

# Clean URLs (WordPress style)
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]

# Redirection 301
Redirect 301 /old-page /new-page

# Blocage bots
RewriteCond %{HTTP_USER_AGENT} (badbot|scrapy) [NC]
RewriteRule .* - [F,L]
```

### Cache & Performance
```apache
# Cache navigateur
<IfModule mod_expires.c>
ExpiresActive On
ExpiresByType text/css A2592000
ExpiresByType text/javascript A2592000
ExpiresByType image/png A31536000
</IfModule>

# Compression
<IfModule mod_deflate.c>
AddOutputFilterByType DEFLATE text/plain text/html text/xml text/css text/javascript application/javascript
</IfModule>
```

## 📊 Monitoring & Logs

### Configuration Logs
```apache
# Format personnalisé
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\" %D" combined_with_time

# Logs par VirtualHost
CustomLog ${APACHE_LOG_DIR}/site_access.log combined_with_time
ErrorLog ${APACHE_LOG_DIR}/site_error.log

# Log niveau debug temporaire
LogLevel info rewrite:trace3
```

### Analyse Logs
```bash
# Top IPs
tail -10000 /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10

# Erreurs 404
grep " 404 " /var/log/apache2/access.log | tail -10

# User agents suspects
grep -i "bot\|crawl\|spider" /var/log/apache2/access.log | tail -10

# Bande passante par IP
awk '{print $1, $10}' /var/log/apache2/access.log | awk '{sum[$1] += $2} END {for (i in sum) print i, sum[i]}' | sort -k2 -nr | head -10
```

### Module mod_status
```apache
# Activation
a2enmod status

# Configuration
<Location "/server-status">
    SetHandler server-status
    Require ip 127.0.0.1
    Require ip 192.168.1.0/24
</Location>

ExtendedStatus On
```

## 🚀 Optimisation Performance

### Tuning MPM
```apache
# MPM Event (recommandé)
<IfModule mpm_event_module>
    StartServers             3
    MinSpareThreads          75
    MaxSpareThreads          250
    ThreadsPerChild          25
    MaxRequestWorkers        400
    MaxConnectionsPerChild   10000
</IfModule>

# MPM Prefork (si mod_php)
<IfModule mpm_prefork_module>
    StartServers             5
    MinSpareServers          5
    MaxSpareServers          10
    MaxRequestWorkers        150
    MaxConnectionsPerChild   10000
</IfModule>
```

### Optimisations Système
```bash
# Limites système
echo "apache soft nofile 65536" >> /etc/security/limits.conf
echo "apache hard nofile 65536" >> /etc/security/limits.conf

# Kernel tuning
echo 'net.core.somaxconn = 65536' >> /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65536' >> /etc/sysctl.conf
sysctl -p
```

## 🛠️ Troubleshooting

### Diagnostics Courants
```bash
# Vérification config
apache2ctl configtest

# Test syntaxe VirtualHost
apache2ctl -t -D DUMP_VHOSTS

# Modules chargés
apache2ctl -M | grep -i ssl

# Ports en écoute
ss -tlnp | grep apache

# Processus Apache
ps aux | grep apache2

# Utilisation mémoire
apache2ctl status
```

### Erreurs Fréquentes
| Erreur | Cause | Solution |
|--------|-------|----------|
| `AH00558` | ServerName non défini | Ajouter `ServerName localhost` |
| `Permission denied` | Droits fichiers | `chown -R www-data:www-data /var/www` |
| `Port 80 already in use` | Conflit service | `systemctl stop nginx` |
| `SSL routine:ssl3_read_bytes` | Config SSL | Vérifier certificats |

### Debug SSL
```bash
# Test certificat
openssl s_client -connect domain.com:443 -servername domain.com

# Vérifier config SSL
apache2ctl -M | grep ssl
apache2ctl -S | grep 443

# Test cipher suites
nmap --script ssl-enum-ciphers -p 443 domain.com
```