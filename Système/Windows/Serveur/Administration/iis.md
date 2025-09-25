# 🌐 Internet Information Services (IIS) — Aide-mémoire

## 🏗️ Architecture & Concepts

### Composants IIS
```
IIS Architecture:
├── World Wide Web Publishing Service (W3SVC)
├── Windows Process Activation Service (WAS)
├── HTTP.sys (Kernel mode driver)
└── Worker Processes (w3wp.exe)
```

| Composant | Rôle | Niveau |
|-----------|------|--------|
| **HTTP.sys** | Réception requêtes HTTP | Kernel |
| **WAS** | Activation processus | Service |
| **W3SVC** | Administration sites web | Service |
| **w3wp.exe** | Processus worker | User mode |

### Structure Fichiers
```
C:\inetpub\
├── wwwroot\                    # Site par défaut
├── logs\LogFiles\W3SVC[ID]\    # Logs par site
├── temp\                       # Fichiers temporaires
└── history\                    # Historique config

C:\Windows\System32\inetsrv\
├── config\
│   ├── applicationHost.config  # Configuration maître
│   ├── administration.config   # Config outils admin
│   └── redirection.config      # Redirection config
├── appcmd.exe                  # Outil CLI
└── iisreset.exe               # Reset services
```

## 📦 Installation & Configuration Initiale

### Installation Rôles Windows
```powershell
# Installation complète IIS
Enable-WindowsOptionalFeature -Online -All -FeatureName @(
    "IIS-WebServerRole",
    "IIS-WebServer", 
    "IIS-CommonHttpFeatures",
    "IIS-HttpRedirect",
    "IIS-HttpCompressionStatic",
    "IIS-HttpCompressionDynamic",
    "IIS-Security",
    "IIS-RequestFiltering",
    "IIS-ManagementConsole",
    "IIS-NetFxExtensibility45",
    "IIS-ASPNET45"
)

# Via DISM (alternative)
dism /online /enable-feature /featurename:IIS-WebServerRole /all

# Vérifier installation
Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -like "IIS-*" -and $_.State -eq "Enabled"}
```

### Fonctionnalités Spécialisées
| Feature | Usage | Installation |
|---------|-------|--------------|
| **URL Rewrite** | Réécriture URLs | Module séparé Microsoft |
| **ARR** | Load balancing | `IIS-ApplicationRequestRouting` |
| **WebDAV** | Partage fichiers web | `IIS-WebDAV` |
| **FTP** | Serveur FTP | `IIS-FTPServer` |

## 🏢 Gestion Sites & Applications

### Sites Web (PowerShell)
```powershell
# Créer site complet
New-Website -Name "MonSite" -Port 80 -HostHeader "monsite.local" -PhysicalPath "C:\inetpub\monsite"

# Site avec multiple bindings
New-Website -Name "MultiSite" -PhysicalPath "C:\inetpub\multisite"
New-WebBinding -Name "MultiSite" -Protocol http -Port 80 -HostHeader "site1.local"
New-WebBinding -Name "MultiSite" -Protocol http -Port 80 -HostHeader "site2.local"
New-WebBinding -Name "MultiSite" -Protocol https -Port 443 -HostHeader "site1.local" -SslFlags 1

# Configuration avancée site
Set-ItemProperty "IIS:\Sites\MonSite" -Name limits.maxBandwidth -Value 1048576  # 1MB/s
Set-ItemProperty "IIS:\Sites\MonSite" -Name limits.maxConnections -Value 100
```

### Applications & Répertoires Virtuels
```powershell
# Créer application dans site
New-WebApplication -Site "MonSite" -Name "app1" -PhysicalPath "C:\inetpub\monsite\app1" -ApplicationPool "MonAppPool"

# Répertoire virtuel
New-WebVirtualDirectory -Site "MonSite" -Name "images" -PhysicalPath "D:\SharedImages"

# Lister structure
Get-Website | Get-WebApplication
Get-Website | Get-WebVirtualDirectory
```

### AppCmd (Alternative CLI)
```cmd
# Gestion sites
%windir%\system32\inetsrv\appcmd list sites
%windir%\system32\inetsrv\appcmd add site /name:"TestSite" /bindings:http/*:8080: /physicalPath:"C:\inetpub\testsite"
%windir%\system32\inetsrv\appcmd set site "Default Web Site" /bindings.[protocol='http',bindingInformation='*:80:'].bindingInformation:*:8080:

# Configuration en lot
%windir%\system32\inetsrv\appcmd list sites /config /xml | %windir%\system32\inetsrv\appcmd add site /in
```

## 🔄 Application Pools

### Configuration Pools
```powershell
# Créer pool avec configuration avancée
New-WebAppPool -Name "MonAppPool"
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name processModel.identityType -Value ApplicationPoolIdentity
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name processModel.maxProcesses -Value 1
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name processModel.idleTimeout -Value "00:00:00"  # Pas de timeout
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name recycling.periodicRestart.time -Value "01:00:00"  # Recyclage quotidien

# Limites mémoire et CPU
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name recycling.periodicRestart.privateMemory -Value 524288  # 512MB
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name cpu.limit -Value 80000  # 80% CPU sur 100000
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name cpu.action -Value Throttle

# Configuration .NET
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name managedRuntimeVersion -Value "v4.0"  # .NET Framework
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name managedRuntimeVersion -Value ""      # .NET Core
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name managedPipelineMode -Value Integrated
```

### Identités & Sécurité
| Identity Type | Description | Usage | Permissions |
|---------------|-------------|-------|-------------|
| **ApplicationPoolIdentity** | Compte virtuel | Recommandé | Minimales automatiques |
| **NetworkService** | Compte système | Legacy | Network access |
| **LocalService** | Compte local | Rare | Local seulement |
| **LocalSystem** | Administrateur | ⚠️ Éviter | Trop élevées |
| **Custom Account** | Compte dédié | Enterprise | Configurables |

```powershell
# Pool avec compte personnalisé
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name processModel.identityType -Value SpecificUser
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name processModel.userName -Value "DOMAIN\svc-webapp"
Set-ItemProperty "IIS:\AppPools\MonAppPool" -Name processModel.password -Value "MotDePasse123!"
```

## 🔒 SSL/TLS & Certificats

### Gestion Certificats
```powershell
# Certificat auto-signé (dev/test)
$cert = New-SelfSignedCertificate -DnsName "monsite.local","*.monsite.local" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(2)

# Importer certificat commercial
$password = ConvertTo-SecureString -String "MotDePasse" -Force -AsPlainText
Import-PfxCertificate -FilePath "C:\Certs\monsite.pfx" -CertStoreLocation "Cert:\LocalMachine\My" -Password $password

# Lister certificats disponibles
Get-ChildItem -Path "Cert:\LocalMachine\My" | Select-Object Subject, FriendlyName, NotAfter, Thumbprint
```

### Configuration HTTPS
```powershell
# Binding HTTPS avec SNI (Server Name Indication)
$cert = Get-ChildItem -Path "Cert:\LocalMachine\My" | Where-Object {$_.Subject -like "*monsite.local*"}
New-WebBinding -Name "MonSite" -Protocol https -Port 443 -HostHeader "monsite.local" -SslFlags 1  # SNI Required
$binding = Get-WebBinding -Name "MonSite" -Protocol https -Port 443 -HostHeader "monsite.local"
$binding.AddSslCertificate($cert.Thumbprint, "My")

# HTTPS uniquement (redirect HTTP)
Remove-WebBinding -Name "MonSite" -Protocol http -Port 80
```

### Durcissement SSL/TLS
```powershell
# Désactiver protocoles faibles (nécessite redémarrage)
$weakProtocols = @('SSL 2.0', 'SSL 3.0', 'TLS 1.0', 'TLS 1.1')
foreach ($protocol in $weakProtocols) {
    $regPath = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$protocol\Server"
    New-Item -Path $regPath -Force | Out-Null
    Set-ItemProperty -Path $regPath -Name "Enabled" -Value 0 -Type DWord
    Set-ItemProperty -Path $regPath -Name "DisabledByDefault" -Value 1 -Type DWord
}

# Cipher suites sécurisées (PowerShell 5.1+)
Enable-TlsCipherSuite -Name "TLS_AES_256_GCM_SHA384"
Disable-TlsCipherSuite -Name "TLS_RSA_WITH_AES_256_CBC_SHA256"
```

## 🛡️ Sécurité & Authentification

### Types d'Authentification
| Type | Utilisation | Configuration | Avantages/Inconvénients |
|------|-------------|---------------|-------------------------|
| **Anonymous** | Public | Par défaut | Simple / Pas sécurisé |
| **Windows** | Intranet | Intégration AD | SSO / Windows seulement |
| **Basic** | Standard | Base64 encoding | Compatible / Mots de passe clairs |
| **Digest** | Amélioré | Hash MD5 | Plus sûr que Basic / Limité |
| **Client Certificate** | Très sécurisé | PKI required | Très sûr / Complexe |

```powershell
# Configuration authentification
Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/anonymousAuthentication" -Name enabled -Value False -Location "MonSite"
Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/windowsAuthentication" -Name enabled -Value True -Location "MonSite"

# Authentification par répertoire
Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/basicAuthentication" -Name enabled -Value True -Location "MonSite/admin"
```

### Autorisations
```xml
<!-- web.config - Autorisations .NET -->
<system.web>
  <authorization>
    <allow users="DOMAIN\AdminGroup" />
    <allow roles="Administrators" />
    <deny users="*" />
  </authorization>
</system.web>

<!-- IIS URL Authorization -->
<system.webServer>
  <security>
    <authorization>
      <add accessType="Allow" users="DOMAIN\user1,DOMAIN\user2" />
      <add accessType="Allow" roles="DOMAIN\WebAdmins" />
      <add accessType="Deny" users="*" />
    </authorization>
  </security>
</system.webServer>
```

### Headers de Sécurité
```xml
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <!-- Sécurité navigateur -->
      <add name="X-Frame-Options" value="SAMEORIGIN" />
      <add name="X-Content-Type-Options" value="nosniff" />
      <add name="X-XSS-Protection" value="1; mode=block" />
      <add name="Referrer-Policy" value="strict-origin-when-cross-origin" />
      
      <!-- HTTPS strict -->
      <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains; preload" />
      
      <!-- CSP (Content Security Policy) -->
      <add name="Content-Security-Policy" value="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" />
      
      <!-- Supprimer headers révélateurs -->
      <remove name="X-Powered-By" />
      <remove name="Server" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

### Filtrage Requêtes
```xml
<system.webServer>
  <security>
    <requestFiltering>
      <!-- Limites globales -->
      <requestLimits maxAllowedContentLength="52428800" maxQueryString="4096" maxUrl="4096" />
      
      <!-- Extensions interdites -->
      <fileExtensions allowUnlisted="true">
        <add fileExtension=".exe" allowed="false" />
        <add fileExtension=".bat" allowed="false" />
        <add fileExtension=".config" allowed="false" />
      </fileExtensions>
      
      <!-- Verbes HTTP autorisés -->
      <verbs allowUnlisted="false">
        <add verb="GET" allowed="true" />
        <add verb="POST" allowed="true" />
        <add verb="PUT" allowed="true" />
        <add verb="DELETE" allowed="true" />
      </verbs>
      
      <!-- Séquences interdites -->
      <denyUrlSequences>
        <add sequence=".." />
        <add sequence=":" />
        <add sequence="\" />
      </denyUrlSequences>
      
      <!-- Headers interdits -->
      <denyQueryStringSequences>
        <add sequence="&lt;script" />
        <add sequence="javascript:" />
      </denyQueryStringSequences>
    </requestFiltering>
  </security>
</system.webServer>
```

## ⚡ Performance & Optimisation

### Compression HTTP
```powershell
# Compression statique et dynamique
Set-WebConfigurationProperty -Filter "/system.webServer/httpCompression" -Name staticCompressionLevel -Value 9
Set-WebConfigurationProperty -Filter "/system.webServer/httpCompression" -Name dynamicCompressionLevel -Value 4
Set-WebConfigurationProperty -Filter "/system.webServer/httpCompression" -Name staticCompressionIgnoreHitFrequency -Value True

# Types MIME pour compression
$staticTypes = @("text/*", "application/javascript", "application/json", "application/xml")
foreach ($type in $staticTypes) {
    Add-WebConfigurationProperty -Filter "/system.webServer/httpCompression/staticTypes" -Name "." -Value @{mimeType=$type; enabled="true"}
}
```

### Cache Statique
```xml
<system.webServer>
  <staticContent>
    <!-- Cache global 7 jours -->
    <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="7.00:00:00" />
  </staticContent>
  
  <!-- Cache par extension -->
  <location path="*.css">
    <system.webServer>
      <staticContent>
        <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="365.00:00:00" />
      </staticContent>
    </system.webServer>
  </location>
  
  <location path="*.js">
    <system.webServer>
      <staticContent>
        <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="365.00:00:00" />
      </staticContent>
    </system.webServer>
  </location>
</system.webServer>
```

### Output Caching
```xml
<system.webServer>
  <caching>
    <profiles>
      <add extension=".php" policy="DontCache" />
      <add extension=".asp" policy="DontCache" />
    </profiles>
  </caching>
  
  <!-- Cache kernel-mode -->
  <caching enabled="true" enableKernelCache="true">
    <profiles>
      <add extension="*" policy="CacheUntilChange" kernelCachePolicy="CacheUntilChange" />
    </profiles>
  </caching>
</system.webServer>
```

### Limites Connexions
```powershell
# Limites par site
Set-ItemProperty "IIS:\Sites\MonSite" -Name limits.maxConnections -Value 200
Set-ItemProperty "IIS:\Sites\MonSite" -Name limits.connectionTimeout -Value "00:02:00"
Set-ItemProperty "IIS:\Sites\MonSite" -Name limits.maxBandwidth -Value 2097152  # 2MB/s

# Limites globales IIS
Set-WebConfigurationProperty -Filter "/system.webServer/serverRuntime" -Name maxRequestEntityAllowed -Value 52428800  # 50MB
Set-WebConfigurationProperty -Filter "/system.webServer/asp" -Name limits.maxRequestEntityAllowed -Value 204800     # 200KB
```

## 📊 Logging & Monitoring

### Configuration Logs W3C
```powershell
# Activer logs étendus
Set-WebConfigurationProperty -Filter "/system.webServer/httpLogging" -Name dontLog -Value False
Set-WebConfigurationProperty -Filter "/system.webServer/httpLogging" -Name logFormat -Value W3C

# Champs personnalisés étendus
$logFields = @(
    "date", "time", "s-ip", "cs-method", "cs-uri-stem", "cs-uri-query", 
    "s-port", "cs-username", "c-ip", "cs(User-Agent)", "cs(Referer)", 
    "sc-status", "sc-substatus", "sc-win32-status", "time-taken", 
    "sc-bytes", "cs-bytes"
)
Set-WebConfigurationProperty -Filter "/system.webServer/httpLogging" -Name logExtFileFlags -Value ($logFields -join ",")

# Rotation logs
Set-WebConfigurationProperty -Filter "/system.webServer/httpLogging" -Name period -Value Daily
Set-WebConfigurationProperty -Filter "/system.webServer/httpLogging" -Name truncateSize -Value 104857600  # 100MB
```

### Failed Request Tracing (FREB)
```powershell
# Activer Failed Request Tracing
Set-WebConfigurationProperty -Filter "/system.webServer/tracing/traceFailedRequests" -Name enabled -Value True -Location "MonSite"

# Règles de trace
Add-WebConfigurationProperty -Filter "/system.webServer/tracing/traceFailedRequests" -Name "." -Value @{
    path="*";
    statusCodes="400-999";
    timeTaken="00:00:10"
} -Location "MonSite"

# Trace détaillée pour ASP.NET
Add-WebConfigurationProperty -Filter "/system.webServer/tracing/traceFailedRequests" -Name "." -Value @{
    path="*.aspx";
    statusCodes="500";
    areas="Authentication,Security,Filter,StaticFiles,CGI,Compression,Cache,RequestNotifications,Module,Rewrite,RequestRouting"
} -Location "MonSite"
```

### Performance Counters
| Compteur | Description | Seuil Normal |
|----------|-------------|--------------|
| **Web Service\Current Connections** | Connexions simultanées | < 1000 |
| **ASP.NET Apps\Requests/Sec** | Throughput | Variable |
| **ASP.NET Apps\Request Execution Time** | Latence | < 1000ms |
| **Process(w3wp)\% Processor Time** | CPU usage | < 80% |
| **Process(w3wp)\Working Set** | Mémoire | < 2GB |

## 🔧 Modules & Extensions

### Modules Natifs Essentiels
| Module | Fonction | Configuration |
|--------|----------|---------------|
| **HttpRedirectionModule** | Redirections HTTP | `<httpRedirect>` |
| **StaticCompressionModule** | Compression statique | `<httpCompression>` |
| **DefaultDocumentModule** | Documents par défaut | `<defaultDocument>` |
| **DirectoryListingModule** | Listing répertoires | `<directoryBrowse>` |
| **StaticFileModule** | Fichiers statiques | `<staticContent>` |

### URL Rewrite (Module externe)
```xml
<system.webServer>
  <rewrite>
    <rules>
      <!-- Redirection HTTP vers HTTPS -->
      <rule name="HTTP to HTTPS" stopProcessing="true">
        <match url="(.*)" />
        <conditions>
          <add input="{HTTPS}" pattern="off" ignoreCase="true" />
          <add input="{REQUEST_METHOD}" pattern="^get$|^head$" />
        </conditions>
        <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
      </rule>
      
      <!-- WWW canonique -->
      <rule name="Add WWW" stopProcessing="true">
        <match url="(.*)" />
        <conditions>
          <add input="{HTTP_HOST}" pattern="^domain\.com$" />
        </conditions>
        <action type="Redirect" url="https://www.domain.com/{R:1}" redirectType="Permanent" />
      </rule>
      
      <!-- Réécriture API -->
      <rule name="API Rewrite">
        <match url="^api/([^/]+)/(.*)$" />
        <action type="Rewrite" url="api.php?controller={R:1}&amp;action={R:2}" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

## 🚨 Troubleshooting & Diagnostics

### Commandes Diagnostics
```powershell
# État services IIS
Get-Service W3SVC, WAS | Select-Object Name, Status, StartType
Get-Website | Select-Object Name, State, PhysicalPath, Bindings
Get-WebAppPool | Select-Object Name, State, ProcessModel, ManagedRuntimeVersion

# Processus worker
Get-Process w3wp -ErrorAction SilentlyContinue | Select-Object Id, ProcessName, WorkingSet64, PagedMemorySize64
Get-WmiObject Win32_Process -Filter "Name='w3wp.exe'" | Select-Object ProcessId, CommandLine

# Test connectivité
Test-NetConnection -ComputerName localhost -Port 80
Test-NetConnection -ComputerName localhost -Port 443
```

### Event IDs Critiques
| Event ID | Source | Description | Action |
|----------|--------|-------------|---------|
| **1309** | WAS | Application pool recycle | Vérifier logs application |
| **5057** | W3SVC | SSL binding error | Vérifier certificat et binding |
| **5059** | W3SVC | Crypto operation failed | Permissions certificat |
| **5152** | W3SVC | Health monitoring heartbeat | État application pool |
| **1000** | Application Error | Application crash | Logs application détaillés |

### Reset & Récupération
```powershell
# Reset IIS services
iisreset /stop
iisreset /start
iisreset /restart

# Reset complet (attention!)
iisreset /stop
net stop was /y
del %windir%\system32\inetsrv\config\*.* /q
copy %windir%\system32\inetsrv\config\schema\*.xml %windir%\system32\inetsrv\config\
%windir%\system32\inetsrv\appcmd restore backup "BACKUP_NAME"
iisreset /start

# Vérification configuration
%windir%\system32\inetsrv\appcmd list config -section:system.webServer/sites
Test-Path "IIS:\Sites\Default Web Site"
```

### Analyse Performance
```powershell
# Métriques temps réel
Get-Counter "\Web Service(Default Web Site)\Current Connections"
Get-Counter "\ASP.NET Applications(__Total__)\Requests/Sec"
Get-Counter "\Process(w3wp)\% Processor Time"

# Dump processus pour analyse
$processId = (Get-Process w3wp).Id
& "C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\adplus.exe" -hang -pn w3wp.exe -o C:\dumps\
```

## 💡 Bonnes Pratiques

### Sécurité
- ✅ **Principe moindre privilège** : Application Pool Identity dédiées
- ✅ **HTTPS partout** : Certificats valides + HSTS
- ✅ **Headers sécurisé** : X-Frame-Options, CSP, etc.
- ✅ **Filtrage requêtes** : Extensions, verbes, tailles
- ✅ **Logs détaillés** : Monitoring et forensics

### Performance
- ✅ **Compression** : Statique + dynamique activées
- ✅ **Cache** : Static content avec TTL appropriés
- ✅ **Connection limits** : Adaptés à la charge
- ✅ **Application pools** : Isolation et recyclage
- ✅ **Monitoring continu** : Counters et alertes

### Maintenance
- ✅ **Sauvegarde config** : `appcmd backup` régulier
- ✅ **Logs rotation** : Espace disque contrôlé
- ✅ **Certificats** : Monitoring expiration
- ✅ **Updates** : Windows Updates et patches IIS
- ✅ **Documentation** : Architecture et procédures

---
**💡 Memo** : `iisreset` pour restart rapide, AppCmd pour CLI avancé, toujours HTTPS avec headers sécurisés !
