# 🔐 PKI, AD CS & OpenSSL — Cheatsheet

## 🏗️ Concepts PKI Fondamentaux

### Hiérarchie Certificats
```
Root CA (Offline)
    ├── Subordinate CA (Online)
    │   ├── Server Certificates
    │   ├── Client Certificates
    │   └── Code Signing
    └── Issuing CA
        ├── User Certificates
        └── Device Certificates
```

### Types de Certificats par Usage
| Type | Usage | Validation | Durée |
|------|-------|------------|-------|
| **DV** | Domain Validated | DNS/HTTP | 1-3 ans |
| **OV** | Organization Validated | Entreprise vérifiée | 1-2 ans |
| **EV** | Extended Validation | Validation poussée | 1 an |
| **Wildcard** | `*.domain.com` | Sous-domaines | 1 an |
| **SAN** | Multiple domains | Multi-domaines | 1-2 ans |

### Algorithmes Recommandés (2024)
- **Asymétrique** : ECDSA P-256/P-384, Ed25519, RSA 2048+ (legacy)
- **Symétrique** : AES-256-GCM, ChaCha20-Poly1305
- **Hash** : SHA-256, SHA-384, SHA-3 (éviter SHA-1, MD5)
- **TLS** : v1.3 > v1.2 (désactiver ≤v1.1)

## 🛠️ OpenSSL — Commandes Essentielles

### Génération Clés & CSR
```bash
# Clé privée ECDSA (recommandé)
openssl ecparam -genkey -name prime256v1 -out private.key

# Clé privée RSA
openssl genrsa -aes256 -out private.key 2048

# CSR avec config
openssl req -new -key private.key -out request.csr -config ssl.conf

# CSR one-liner
openssl req -new -key private.key -out request.csr \
  -subj "/C=FR/ST=IDF/L=Paris/O=Company/CN=domain.com"
```

### Configuration CSR (ssl.conf)
```ini
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = FR
ST = Ile-de-France
L = Paris
O = Mon Entreprise
CN = www.example.com

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = api.example.com
```

### Certificats Auto-signés
```bash
# Auto-signé simple
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes

# Auto-signé avec SAN
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem \
  -days 365 -nodes -config ssl.conf -extensions v3_req
```

### Validation & Tests
```bash
# Afficher certificat
openssl x509 -in cert.pem -text -noout

# Vérifier CSR
openssl req -in request.csr -text -noout

# Vérifier correspondance clé/cert
openssl rsa -in private.key -pubout | openssl md5
openssl x509 -in cert.pem -pubkey -noout | openssl md5

# Test TLS serveur
openssl s_client -connect domain.com:443 -servername domain.com

# Vérifier chaîne complète
openssl s_client -connect domain.com:443 -showcerts
```

### Formats & Conversions
```bash
# PEM → DER
openssl x509 -in cert.pem -outform DER -out cert.der

# PEM → PKCS#12 (.pfx/.p12)
openssl pkcs12 -export -out cert.pfx -inkey private.key -in cert.pem -certfile ca.pem

# PKCS#12 → PEM
openssl pkcs12 -in cert.pfx -out cert.pem -nodes

# Extraire clé publique
openssl x509 -in cert.pem -pubkey -noout > public.key
```

### Chiffrement Fichiers
```bash
# AES-256-CBC
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc
openssl enc -aes-256-cbc -d -in file.enc -out file.txt

# Avec clé dérivée
openssl enc -aes-256-cbc -pbkdf2 -iter 100000 -salt -in file.txt -out file.enc
```

## 🏢 Active Directory Certificate Services (AD CS)

### Installation & Configuration
```powershell
# Installation rôle
Install-WindowsFeature ADCS-Cert-Authority -IncludeManagementTools

# Configuration CA Enterprise
Install-AdcsCertificationAuthority -CAType EnterpriseRootCA -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" -KeyLength 2048 -HashAlgorithmName SHA256
```

### Gestion Templates
1. **Dupliquer template** : `certtmpl.msc` → Duplicate template
2. **Personnaliser** : Extensions, Key Usage, Subject Name
3. **Publier** : `certsrv.msc` → Certificate Templates → New → Template to Issue

### Templates Utiles
| Template | Usage | Auto-enroll |
|----------|-------|-------------|
| **Web Server** | IIS, Apache | Non |
| **Computer** | Auth machine | Oui (GPO) |
| **User** | Auth utilisateur | Oui (GPO) |
| **Code Signing** | Signature code | Non |
| **IPSEC** | VPN/IPSEC | Oui |

### Demande Certificats
```bash
# Via interface web
https://ca-server/certsrv

# Via certreq (Windows)
certreq -submit -attrib "CertificateTemplate:WebServer" request.csr

# Via MMC
certlm.msc → Personal → Certificates → Request New Certificate
```

### Export/Import
```powershell
# Export PFX avec clé privée
$pwd = ConvertTo-SecureString -String "password" -Force -AsPlainText
Export-PfxCertificate -Cert "Cert:\LocalMachine\My\thumbprint" -FilePath cert.pfx -Password $pwd

# Import sur serveur distant
$cred = Get-Credential
Invoke-Command -ComputerName server01 -Credential $cred -ScriptBlock {
    Import-PfxCertificate -FilePath "\\share\cert.pfx" -CertStoreLocation "Cert:\LocalMachine\My" -Password $using:pwd
}
```

### Révocation & CRL
```bash
# Révoquer certificat
certutil -revoke serial_number reason_code

# Publier CRL
certutil -crl

# Vérifier statut révocation
certutil -verify -urlfetch cert.pem
```

## 🤖 ACME & Let's Encrypt

### Certbot (Debian/Ubuntu)
```bash
# Installation
apt install certbot python3-certbot-apache

# Certificat standalone
certbot certonly --standalone -d domain.com -d www.domain.com

# Avec Apache
certbot --apache -d domain.com

# Renouvellement auto
echo "0 2 * * * root certbot renew --quiet" >> /etc/crontab
```

### acme.sh (Alternative)
```bash
# Installation
curl https://get.acme.sh | sh

# Certificat DNS challenge
acme.sh --issue --dns dns_cloudflare -d domain.com -d *.domain.com

# Installation auto
acme.sh --install-cert -d domain.com \
  --key-file /etc/ssl/private/domain.key \
  --fullchain-file /etc/ssl/certs/domain.pem \
  --reloadcmd "systemctl reload nginx"
```

### Validation Challenges
- **HTTP-01** : `http://domain.com/.well-known/acme-challenge/`
- **DNS-01** : TXT record `_acme-challenge.domain.com`
- **TLS-ALPN-01** : TLS avec extension ALPN

## 🔧 Déploiement & Configuration

### Apache/Nginx
```apache
# Apache VirtualHost
<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/domain.pem
    SSLCertificateKeyFile /etc/ssl/private/domain.key
    SSLCertificateChainFile /etc/ssl/certs/chain.pem
    
    # Sécurité TLS
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!MD5:!DSS
    SSLHonorCipherOrder on
</VirtualHost>
```

```nginx
# Nginx Server Block
server {
    listen 443 ssl http2;
    ssl_certificate /etc/ssl/certs/domain.pem;
    ssl_certificate_key /etc/ssl/private/domain.key;
    
    # Sécurité TLS
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:!aNULL:!MD5:!DSS;
    ssl_prefer_server_ciphers on;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

### IIS (Windows)
```powershell
# Importer certificat
Import-PfxCertificate -FilePath cert.pfx -CertStoreLocation Cert:\LocalMachine\My

# Binding HTTPS
New-WebBinding -Name "Default Web Site" -IPAddress "*" -Port 443 -Protocol https -SslFlags 0
```

## 📋 Organisation Fichiers

### Structure Recommandée
```
/etc/ssl/
├── certs/          # Certificats publics
├── private/        # Clés privées (600)
├── csr/           # Certificate Signing Requests
├── ca-certificates/ # CA racines système
└── backup/        # Sauvegardes chiffrées
```

### Permissions & Sécurité
```bash
# Permissions correctes
chmod 644 /etc/ssl/certs/*.pem
chmod 600 /etc/ssl/private/*.key
chown root:ssl-cert /etc/ssl/private/

# Sauvegarde chiffrée
tar czf - /etc/ssl/ | openssl enc -aes-256-cbc -out ssl-backup.tar.gz.enc
```

## 🛡️ Sécurité & Bonnes Pratiques

### Durcissement TLS
- **Protocoles** : TLS 1.3 uniquement si possible, sinon TLS 1.2+
- **Cipher Suites** : Forward Secrecy (ECDHE, DHE)
- **HSTS** : `max-age=31536000; includeSubDomains; preload`
- **OCSP Stapling** : Réduire latence validation

### Monitoring & Alertes
```bash
# Script surveillance expiration
#!/bin/bash
DAYS=30
for cert in /etc/ssl/certs/*.pem; do
    if openssl x509 -checkend $((DAYS*86400)) -noout -in "$cert"; then
        echo "OK: $cert expires in >$DAYS days"
    else
        echo "WARNING: $cert expires in <$DAYS days" | mail -s "Cert Alert" admin@domain.com
    fi
done
```

### Tests Sécurité
```bash
# Test SSL Labs
curl -s "https://api.ssllabs.com/api/v3/analyze?host=domain.com" | jq .

# Scan interne
testssl.sh --fast domain.com:443

# Vérification OCSP
openssl s_client -connect domain.com:443 -status 2>/dev/null | grep -A 17 "OCSP response"
```

## 🚀 Automatisation & Scripts

### Script Renouvellement
```bash
#!/bin/bash
# renew-certs.sh
DOMAIN="domain.com"
CERT_PATH="/etc/ssl/certs"
KEY_PATH="/etc/ssl/private"

# Vérifier expiration
if openssl x509 -checkend 2592000 -noout -in "$CERT_PATH/$DOMAIN.pem"; then
    echo "Certificate still valid for 30+ days"
    exit 0
fi

# Renouveler avec ACME
acme.sh --renew -d "$DOMAIN" --force

# Redémarrer services
systemctl reload nginx
systemctl reload apache2

echo "Certificate renewed for $DOMAIN"
```

### PowerShell AD CS
```powershell
# Auto-enrollment via GPO
$Template = "WebServer"
$Subject = "CN=server.domain.com"

$Request = New-Object -ComObject CertificateAuthority.Request
$Request.Submit(1, $Subject, $Template, "")
```

