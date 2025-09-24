# 🔐 PKI, AD CS & OpenSSL — Aide-mémoire

## 🏗️ Architecture PKI

### Hiérarchie Certificats
```
Root CA (Offline/HSM)
    ├── Intermediate CA (Online)
    │   ├── Issuing CA (Server Certs)
    │   ├── User CA (Client Certs)
    │   └── Code Signing CA
    └── Cross-Certification
        └── External Trust Anchors
```

### Types de Certificats
| Type | Validation | Usage | Durée Typique |
|------|------------|-------|---------------|
| **DV** | Domain seul | Sites basiques | 1-3 ans |
| **OV** | Organisation | Sites commerciaux | 1-2 ans |
| **EV** | Validation étendue | Banques, e-commerce | 1 an |
| **Wildcard** | `*.domain.com` | Sous-domaines illimités | 1 an |
| **SAN/UCC** | Multi-domaines | Consolidation | 1-2 ans |
| **Client** | Utilisateur/Device | Authentification | 1-3 ans |
| **Code Signing** | Développeur | Signature logiciel | 1-3 ans |

### Standards Cryptographiques (2024)
| Usage | Recommandé | Legacy Acceptable | À Éviter |
|-------|------------|-------------------|----------|
| **Asymétrique** | ECDSA P-256/384, Ed25519 | RSA 2048+ | RSA <2048, DSA |
| **Symétrique** | AES-256-GCM, ChaCha20 | AES-128-GCM | DES, 3DES, RC4 |
| **Hash** | SHA-256, SHA-384, SHA-3 | SHA-256 | SHA-1, MD5 |
| **TLS** | v1.3 | v1.2 | ≤v1.1 |
| **Durée Cert** | 1 an | 2 ans | >2 ans |

## 🛠️ OpenSSL Essentiels

### Génération Clés & CSR
```bash
# Clé ECDSA (recommandée pour nouveaux déploiements)
openssl ecparam -genkey -name prime256v1 -out private.key
openssl ecparam -genkey -name secp384r1 -out private.key  # Plus sécurisé

# Clé RSA (legacy/compatibilité)
openssl genrsa -aes256 -out private.key 2048    # Chiffrée
openssl genrsa -out private.key 2048            # Non chiffrée

# CSR interactif
openssl req -new -key private.key -out request.csr

# CSR avec configuration
openssl req -new -key private.key -out request.csr -config csr.conf
```

### Template Configuration CSR
```ini
# csr.conf
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = FR
ST = Ile-de-France  
L = Paris
O = Mon Entreprise
OU = IT Department
CN = www.example.com

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = api.example.com
IP.1 = 192.168.1.100
```

### Certificats Auto-signés
```bash
# Auto-signé simple (développement)
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem \
    -days 365 -nodes -subj "/CN=localhost"

# Auto-signé avec SAN (test/lab)
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem \
    -days 365 -nodes -config csr.conf -extensions v3_req

# CA racine maison
openssl req -x509 -newkey rsa:4096 -keyout ca-key.pem -out ca-cert.pem \
    -days 3650 -nodes -subj "/CN=My Root CA"
```

### Validation & Diagnostic
```bash
# Afficher certificat détaillé
openssl x509 -in cert.pem -text -noout
openssl x509 -in cert.pem -noout -dates -subject -issuer

# Vérifier CSR
openssl req -in request.csr -text -noout -verify

# Correspondance clé/certificat
openssl pkey -in private.key -pubout | openssl md5
openssl x509 -in cert.pem -pubkey -noout | openssl md5

# Test connexion TLS
openssl s_client -connect domain.com:443 -servername domain.com
openssl s_client -connect domain.com:443 -showcerts | openssl x509 -text

# Vérifier chaîne complète
openssl verify -CAfile ca-bundle.pem cert.pem
openssl s_client -connect site.com:443 -verify_return_error
```

### Conversions Formats
```bash
# PEM ↔ DER
openssl x509 -in cert.pem -outform DER -out cert.der
openssl x509 -in cert.der -inform DER -outform PEM -out cert.pem

# Créer PKCS#12 (.pfx/.p12)
openssl pkcs12 -export -out cert.pfx \
    -inkey private.key -in cert.pem -certfile ca-chain.pem \
    -name "My Certificate"

# Extraire de PKCS#12
openssl pkcs12 -in cert.pfx -out cert-and-key.pem -nodes
openssl pkcs12 -in cert.pfx -nokeys -out cert.pem        # Cert seulement
openssl pkcs12 -in cert.pfx -nocerts -out key.pem -nodes # Clé seulement

# Formats Windows/Java
openssl crl2pkcs7 -nocrl -certfile cert.pem -out cert.p7b  # PKCS#7
keytool -import -alias mycert -file cert.pem -keystore keystore.jks
```

### Utilitaires Avancés
```bash
# Infos SSL/TLS serveur distant
echo | openssl s_client -connect site.com:443 2>/dev/null | \
    openssl x509 -noout -dates -subject

# Test cipher suites
openssl s_client -connect site.com:443 -cipher 'ECDHE+AESGCM'

# Générer DH params
openssl dhparam -out dhparam.pem 2048

# Vérifier révocation OCSP
openssl ocsp -issuer ca.pem -cert cert.pem -url http://ocsp.example.com \
    -CAfile ca-bundle.pem
```

## 🏢 Active Directory Certificate Services

### Installation & Configuration
```powershell
# Installation rôle AD CS
Install-WindowsFeature ADCS-Cert-Authority -IncludeManagementTools
Install-WindowsFeature ADCS-Web-Enrollment    # Interface web optionnelle

# Configuration CA Enterprise Root
Install-AdcsCertificationAuthority `
    -CAType EnterpriseRootCA `
    -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
    -KeyLength 4096 `
    -HashAlgorithmName SHA256 `
    -ValidityPeriod Years `
    -ValidityPeriodUnits 20

# Configuration CA Subordinate
Install-AdcsCertificationAuthority `
    -CAType EnterpriseSubordinateCA `
    -ParentCA "RootCA-Server\Root CA Name" `
    -KeyLength 2048 `
    -ValidityPeriod Years `
    -ValidityPeriodUnits 10
```

### Templates Certificats
```powershell
# Commandes PowerShell utiles
Get-CATemplate                          # Lister templates disponibles
Get-CertificateTemplate                 # Templates AD

# Via interface graphique
certtmpl.msc                           # Gestion templates
certsrv.msc                            # Console CA
```

### Templates Standards
| Template | Usage | Auto-Enroll | Key Usage |
|----------|-------|-------------|-----------|
| **Computer** | Auth machine | ✓ (GPO) | Client/Server Auth |
| **User** | Auth utilisateur | ✓ (GPO) | Client Auth, Email |
| **Web Server** | IIS/Apache | ✗ | Server Auth |
| **Code Signing** | Signature code | ✗ | Code Signing |
| **IPSec** | VPN/IPSec | ✓ | Client/Server Auth |
| **Domain Controller** | DC Auth | ✓ | Client/Server Auth |
| **Workstation** | Poste de travail | ✓ | Client Auth |

### Demande & Gestion Certificats
```powershell
# Demande via certreq
certreq -submit -attrib "CertificateTemplate:WebServer" request.csr

# Demande via PowerShell
$Template = Get-CertificateTemplate -Name "WebServer"
Get-Certificate -Template $Template -Url "ldap:" -CertStoreLocation Cert:\LocalMachine\My

# Export certificat avec clé privée
$cert = Get-ChildItem Cert:\LocalMachine\My | Where {$_.Subject -like "*domain.com*"}
$pwd = ConvertTo-SecureString -String "P@ssw0rd" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath cert.pfx -Password $pwd

# Import sur serveur distant
$session = New-PSSession -ComputerName "server01"
Copy-Item cert.pfx -Destination C:\temp\ -ToSession $session
Invoke-Command -Session $session -ScriptBlock {
    Import-PfxCertificate -FilePath C:\temp\cert.pfx -CertStoreLocation Cert:\LocalMachine\My -Password $using:pwd
}
```

### Révocation & CRL
```bash
# Administration CA
certutil -revoke SerialNumber ReasonCode  # Révoquer certificat
certutil -crl                             # Publier CRL
certutil -getreg CA\CRLPublicationURLs    # URLs publication CRL

# Codes raison révocation
# 0=Unspecified, 1=KeyCompromise, 2=CACompromise, 3=AffiliationChanged
# 4=Superseded, 5=CessationOfOperation, 6=CertificateHold

# Vérification statut
certutil -verify -urlfetch cert.cer
certutil -url cert.cer                    # Test URLs dans certificat
```

## 🤖 ACME & Let's Encrypt

### Certbot (Client Officiel)
```bash
# Installation Debian/Ubuntu
apt install certbot python3-certbot-apache python3-certbot-nginx

# Certificat standalone (port 80 libre)
certbot certonly --standalone -d domain.com -d www.domain.com

# Avec webroot (serveur web actif)
certbot certonly --webroot -w /var/www/html -d domain.com

# Intégration Apache/Nginx
certbot --apache -d domain.com    # Configure automatiquement Apache
certbot --nginx -d domain.com     # Configure automatiquement Nginx

# Certificat wildcard (DNS challenge requis)
certbot certonly --manual --preferred-challenges dns -d "*.domain.com"

# Renouvellement automatique
echo "0 2 * * * root certbot renew --quiet" > /etc/cron.d/certbot
systemctl enable certbot.timer    # Sur systemd
```

### acme.sh (Alternative Populaire)
```bash
# Installation
curl https://get.acme.sh | sh -s email=admin@domain.com

# Certificat avec DNS API (ex: Cloudflare)
export CF_Token="your-cloudflare-token"
acme.sh --issue --dns dns_cf -d domain.com -d "*.domain.com"

# Installation automatique
acme.sh --install-cert -d domain.com \
    --key-file /etc/ssl/private/domain.key \
    --fullchain-file /etc/ssl/certs/domain.pem \
    --reloadcmd "systemctl reload nginx"

# Renouvellement (automatique via cron)
acme.sh --cron
```

### Types de Validation ACME
| Challenge | Port | Usage | Avantages | Limitations |
|-----------|------|--------|-----------|-------------|
| **HTTP-01** | 80 | Domain validation | Simple | Pas de wildcard |
| **DNS-01** | 53 | DNS TXT record | Wildcard possible | API DNS requise |
| **TLS-ALPN-01** | 443 | TLS extension | Pas de port 80 | Support limité |

## 🌐 Configuration Serveurs Web

### Apache HTTPD
```apache
# /etc/apache2/sites-available/domain-ssl.conf
<VirtualHost *:443>
    ServerName domain.com
    DocumentRoot /var/www/html
    
    # SSL Configuration
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/domain.pem
    SSLCertificateKeyFile /etc/ssl/private/domain.key
    SSLCertificateChainFile /etc/ssl/certs/chain.pem
    
    # Modern SSL configuration
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:!aNULL:!MD5:!DSS
    SSLHonorCipherOrder on
    
    # Security headers
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header always set X-Content-Type-Options nosniff
    Header always set X-Frame-Options DENY
    
    # OCSP Stapling
    SSLUseStapling on
    SSLStaplingCache shmcb:/var/run/apache2/stapling_cache(128000)
</VirtualHost>

# Force HTTPS redirect
<VirtualHost *:80>
    ServerName domain.com
    Redirect permanent / https://domain.com/
</VirtualHost>
```

### Nginx
```nginx
# /etc/nginx/sites-available/domain
server {
    listen 80;
    server_name domain.com www.domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name domain.com www.domain.com;
    
    # SSL Configuration
    ssl_certificate /etc/ssl/certs/domain.pem;
    ssl_certificate_key /etc/ssl/private/domain.key;
    ssl_trusted_certificate /etc/ssl/certs/chain.pem;
    
    # Modern SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:!aNULL:!MD5:!DSS;
    ssl_prefer_server_ciphers off;  # TLS 1.3 compatibility
    
    # SSL session cache
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-Frame-Options DENY always;
}
```

### IIS (Windows)
```powershell
# Import certificat
Import-PfxCertificate -FilePath cert.pfx -CertStoreLocation Cert:\LocalMachine\My -Password $pwd

# Configuration site
New-WebSite -Name "SecureSite" -Port 443 -Protocol https -PhysicalPath C:\inetpub\wwwroot
$cert = Get-ChildItem Cert:\LocalMachine\My | Where {$_.Subject -like "*domain.com*"}
New-WebBinding -Name "SecureSite" -Protocol https -Port 443 -SslFlags 1 -CertificateThumbprint $cert.Thumbprint

# IIS SSL Settings (web.config)
<system.webServer>
    <rewrite>
        <rules>
            <rule name="Redirect to HTTPS" stopProcessing="true">
                <match url=".*" />
                <conditions>
                    <add input="{HTTPS}" pattern="off" ignoreCase="true" />
                </conditions>
                <action type="Redirect" url="https://{HTTP_HOST}/{R:0}" redirectType="Permanent" />
            </rule>
        </rules>
    </rewrite>
</system.webServer>
```

## 🛡️ Sécurité & Hardening

### Configuration TLS Sécurisée
```bash
# Test cipher suites
nmap --script ssl-enum-ciphers -p 443 domain.com
testssl.sh --fast domain.com

# Configuration dhparam (si DHE utilisé)
openssl dhparam -out /etc/ssl/dhparam.pem 2048

# Vérification HSTS
curl -I https://domain.com | grep -i strict-transport-security
```

### Monitoring Expiration
```bash
# Script surveillance certificats
#!/bin/bash
check_cert_expiry() {
    local domain=$1
    local days_warn=${2:-30}
    
    expiry_date=$(echo | openssl s_client -servername $domain -connect $domain:443 2>/dev/null | \
                  openssl x509 -noout -enddate | cut -d= -f2)
    expiry_epoch=$(date -d "$expiry_date" +%s)
    current_epoch=$(date +%s)
    days_until_expiry=$(( (expiry_epoch - current_epoch) / 86400 ))
    
    if [ $days_until_expiry -lt $days_warn ]; then
        echo "WARNING: $domain expires in $days_until_expiry days"
        # Envoyer alerte mail/Slack/Teams
    fi
}

check_cert_expiry "domain.com" 30
```

### Certificate Transparency
```bash
# Vérifier CT logs
curl -s "https://crt.sh/?q=domain.com&output=json" | jq '.[].common_name' | sort -u

# Monitoring nouveaux certificats
curl -s "https://certspotter.com/api/v0/certs?domain=domain.com" | jq '.[]'
```

## 📋 Gestion & Organisation

### Structure Fichiers
```
/etc/ssl/
├── certs/                 # Certificats publics (644)
│   ├── domain.pem
│   └── ca-bundle.pem
├── private/               # Clés privées (600, root:ssl-cert)
│   └── domain.key
├── csr/                   # Certificate requests
│   └── domain.csr
└── archive/               # Certificats expirés/révoqués
    └── old-certs/
```

### Permissions & Sécurité
```bash
# Permissions correctes
chmod 644 /etc/ssl/certs/*
chmod 600 /etc/ssl/private/*
chown root:ssl-cert /etc/ssl/private/
usermod -a -G ssl-cert nginx    # Ajouter nginx au groupe ssl-cert

# Audit sécurité
find /etc/ssl -type f -perm /o+r -path "*/private/*" -exec ls -la {} \;
```

### Sauvegarde & Restauration
```bash
# Sauvegarde chiffrée
tar czf - /etc/ssl/ | gpg --cipher-algo AES256 --compress-algo 1 \
    --symmetric --output ssl-backup-$(date +%Y%m%d).tar.gz.gpg

# Inventaire certificats
find /etc/ssl/certs -name "*.pem" -exec openssl x509 -in {} -noout -subject -enddate \; > cert-inventory.txt
```

## 🔍 Tests & Validation

### Outils d'Audit SSL/TLS
```bash
# SSL Labs API (automatisé)
curl -s "https://api.ssllabs.com/api/v3/analyze?host=domain.com&publish=off&all=done" | \
    jq '.endpoints[0].grade'

# testssl.sh (audit complet)
testssl.sh --jsonfile results.json domain.com

# nmap SSL scripts
nmap --script ssl-cert,ssl-enum-ciphers,ssl-heartbleed -p 443 domain.com

# Vérification interne
openssl verify -CAfile /etc/ssl/certs/ca-certificates.crt cert.pem
```

### Checklist Déploiement
- [ ] **Certificat** : SAN inclut tous domaines/IPs nécessaires
- [ ] **Chaîne** : Certificats intermédiaires inclus
- [ ] **Clé** : Générée de manière sécurisée, permissions correctes
- [ ] **TLS** : Version 1.2+ uniquement, cipher suites sécurisées
- [ ] **HSTS** : Header configuré avec preload si possible
- [ ] **OCSP** : Stapling activé pour performance
- [ ] **Monitoring** : Alerte expiration configurée
- [ ] **Sauvegarde** : Certificats et clés sauvegardés de manière chiffrée
- [ ] **Tests** : Validation SSL/TLS via outils externes

---
**💡 Memo** : ECDSA > RSA pour nouveaux déploiements • TLS 1.3 uniquement si possible • HSTS + preload recommandé !

