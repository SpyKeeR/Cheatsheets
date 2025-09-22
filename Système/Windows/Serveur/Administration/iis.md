# 🌐 Internet Information Services (IIS) — Cheatsheet

## 📦 Installation & Configuration Initiale

### Installation via PowerShell
```powershell
# Installation IIS avec fonctionnalités essentielles
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerRole
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServer
Enable-WindowsOptionalFeature -Online -FeatureName IIS-CommonHttpFeatures
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpRedirect
Enable-WindowsOptionalFeature -Online -FeatureName IIS-NetFxExtensibility45

# Installation IIS Manager
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ManagementConsole

# Via Server Manager (GUI alternative)
Add-WindowsFeature Web-Server, Web-Common-Http, Web-Mgmt-Tools
```

### Structure IIS
```
C:\inetpub\
├── wwwroot\              # Site par défaut
├── logs\LogFiles\        # Logs IIS
└── temp\                 # Fichiers temporaires

C:\Windows\System32\inetsrv\
├── config\               # Configuration IIS
│   ├── applicationHost.config
│   └── administration.config
└── appcmd.exe            # Outil ligne de commande
```

## 🏗️ Gestion Sites Web

### Création Site via PowerShell
```powershell
# Nouveau site web
New-Website -Name "MonSite" -Port 80 -PhysicalPath "C:\inetpub\monsite" -ApplicationPool "DefaultAppPool"

# Avec binding spécifique
New-Website -Name "MonSite" -BindingInformation "*:80:monsite.local" -PhysicalPath "C:\inetpub\monsite"

# Site HTTPS
New-Website -Name "MonSiteSSL" -Port 443 -PhysicalPath "C:\inetpub\monsite" -Ssl
```

### Gestion Sites
```powershell
# Lister sites
Get-Website

# Démarrer/Arrêter site
Start-Website -Name "MonSite"
Stop-Website -Name "MonSite"

# Supprimer site
Remove-Website -Name "MonSite"

# Modifier binding
New-WebBinding -Name "MonSite" -IPAddress "*" -Port 8080 -Protocol http
Remove-WebBinding -Name "MonSite" -Port 80 -Protocol http
```

### Configuration via AppCmd
```cmd
# Lister sites
%windir%\system32\inetsrv\appcmd list sites

# Créer site
%windir%\system32\inetsrv\appcmd add site /name:"MonSite" /bindings:http/*:80:monsite.local /physicalPath:"C:\inetpub\monsite"

# Configurer site
%windir%\system32\inetsrv\appcmd set site "MonSite" /bindings.[protocol='http',bindingInformation='*:80:'].bindingInformation:*:8080:
```

## 🔧 Application Pools

### Gestion via PowerShell
```powershell
# Créer Application Pool
New-WebAppPool -Name "MonAppPool"

# Configuration avancée
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name processModel.identityType -Value ApplicationPoolIdentity
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name recycling.periodicRestart.time -Value "01:00:00"
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name processModel.maxProcesses -Value 1

# Redémarrer pool
Restart-WebAppPool -Name "MonAppPool"

# Paramètres recommandés production
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name processModel.idleTimeout -Value "00:00:00"
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name recycling.logEventOnRecycle -Value "Time,Requests,Schedule,Memory,IsapiUnhealthy,OnDemand,ConfigChange,PrivateMemory"
```

### Configuration .NET
```powershell
# Version .NET Framework
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name managedRuntimeVersion -Value "v4.0"

# Mode pipeline
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name managedPipelineMode -Value "Integrated"

# Pour .NET Core (No Managed Code)
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name managedRuntimeVersion -Value ""
```

## 🔒 Sécurité & SSL/TLS

### Certificats SSL
```powershell
# Certificat auto-signé
New-SelfSignedCertificate -DnsName "monsite.local" -CertStoreLocation "cert:\LocalMachine\My" -FriendlyName "MonSite SSL"

# Importer certificat PFX
$pwd = ConvertTo-SecureString -String "motdepasse" -Force -AsPlainText
Import-PfxCertificate -FilePath "C:\cert\monsite.pfx" -CertStoreLocation "Cert:\LocalMachine\My" -Password $pwd

# Binding HTTPS avec certificat
$cert = Get-ChildItem -Path "Cert:\LocalMachine\My" | Where-Object {$_.Subject -like "*monsite.local*"}
New-WebBinding -Name "MonSite" -Protocol https -Port 443 -SslFlags 1
$binding = Get-WebBinding -Name "MonSite" -Protocol https
$binding.AddSslCertificate($cert.Thumbprint, "My")
```

### Configuration SSL/TLS Sécurisée
```powershell
# Désactiver SSLv2, SSLv3, TLS 1.0, TLS 1.1
$protocols = @('SSL 2.0', 'SSL 3.0', 'TLS 1.0', 'TLS 1.1')
foreach ($protocol in $protocols) {
    New-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$protocol\Server" -Force
    Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$protocol\Server" -Name "Enabled" -Value 0
    Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$protocol\Server" -Name "DisabledByDefault" -Value 1
}

# Activer TLS 1.2, TLS 1.3
$secureTLS = @('TLS 1.2', 'TLS 1.3')
foreach ($tls in $secureTLS) {
    New-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$tls\Server" -Force
    Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$tls\Server" -Name "Enabled" -Value 1
    Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\$tls\Server" -Name "DisabledByDefault" -Value 0
}
```

### Authentification
```powershell
# Désactiver authentification anonyme
Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/anonymousAuthentication" -Name enabled -Value False -PSPath "IIS:" -Location "MonSite"

# Activer authentification Windows
Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/windowsAuthentication" -Name enabled -Value True -PSPath "IIS:" -Location "MonSite"

# Authentification basique
Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/basicAuthentication" -Name enabled -Value True -PSPath "IIS:" -Location "MonSite"
```

## 🛡️ Durcissement Sécurité

### Headers Sécurité
```xml
<!-- web.config -->
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <add name="X-Frame-Options" value="SAMEORIGIN" />
      <add name="X-Content-Type-Options" value="nosniff" />
      <add name="X-XSS-Protection" value="1; mode=block" />
      <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains" />
      <add name="Referrer-Policy" value="strict-origin-when-cross-origin" />
    </customHeaders>
    <redirectHeaders>
      <clear />
    </redirectHeaders>
  </httpProtocol>
  
  <!-- Supprimer headers révélateurs -->
  <httpProtocol>
    <customHeaders>
      <remove name="X-Powered-By" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

### Redirection HTTP → HTTPS
```xml
<!-- URL Rewrite Module requis -->
<system.webServer>
  <rewrite>
    <rules>
      <rule name="Redirect to HTTPS" stopProcessing="true">
        <match url="(.*)" />
        <conditions>
          <add input="{HTTPS}" pattern="off" ignoreCase="true" />
        </conditions>
        <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

### Filtrage Requêtes
```xml
<system.webServer>
  <security>
    <requestFiltering>
      <!-- Limiter taille upload -->
      <requestLimits maxAllowedContentLength="10485760" maxQueryString="2048" />
      
      <!-- Bloquer extensions dangereuses -->
      <fileExtensions>
        <add fileExtension=".exe" allowed="false" />
        <add fileExtension=".bat" allowed="false" />
        <add fileExtension=".cmd" allowed="false" />
      </fileExtensions>
      
      <!-- Bloquer séquences dangereuses -->
      <denyUrlSequences>
        <add sequence=".." />
        <add sequence=":" />
      </denyUrlSequences>
    </requestFiltering>
  </security>
</system.webServer>
```

## 📊 Monitoring & Logs

### Configuration Logs
```powershell
# Activer logs détaillés
Set-WebConfigurationProperty -Filter "/system.webServer/httpLogging" -Name dontLog -Value False -PSPath "IIS:" -Location "MonSite"

# Format logs W3C étendu
Set-WebConfigurationProperty -Filter "/system.webServer/httpLogging" -Name logFormat -Value W3C -PSPath "IIS:" -Location "MonSite"

# Champs personnalisés
$fields = "date time s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) sc-status sc-substatus sc-win32-status time-taken"
Set-WebConfigurationProperty -Filter "/system.webServer/httpLogging" -Name logExtFileFlags -Value $fields -PSPath "IIS:" -Location "MonSite"
```

### Localisation Logs
- **Access Logs** : `%SystemDrive%\inetpub\logs\LogFiles\W3SVC[SiteID]\`
- **Error Logs** : Event Viewer → Windows Logs → System
- **Application Logs** : Event Viewer → Applications and Services Logs → Microsoft → Windows → IIS-*

### Analyse Logs PowerShell
```powershell
# Parser logs IIS
Get-Content "C:\inetpub\logs\LogFiles\W3SVC1\*.log" | Where-Object {$_ -notlike "#*"} | 
    ForEach-Object {
        $fields = $_ -split ' '
        [PSCustomObject]@{
            Date = $fields[0]
            Time = $fields[1]
            ClientIP = $fields[2]
            Method = $fields[3]
            URI = $fields[4]
            Status = $fields[7]
            UserAgent = $fields[9]
        }
    } | Where-Object {$_.Status -eq "404"} | Group-Object URI | Sort-Object Count -Descending
```

## 🚀 Performance & Optimisation

### Compression
```powershell
# Activer compression statique
Set-WebConfigurationProperty -Filter "/system.webServer/httpCompression" -Name staticCompressionLevel -Value 9
Set-WebConfigurationProperty -Filter "/system.webServer/httpCompression" -Name dynamicCompressionLevel -Value 4

# Types MIME pour compression
Add-WebConfigurationProperty -Filter "/system.webServer/httpCompression/staticTypes" -Name "." -Value @{mimeType="text/css"; enabled="true"}
Add-WebConfigurationProperty -Filter "/system.webServer/httpCompression/staticTypes" -Name "." -Value @{mimeType="application/javascript"; enabled="true"}
```

### Cache Statique
```xml
<system.webServer>
  <staticContent>
    <!-- Cache CSS/JS 30 jours -->
    <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="30.00:00:00" />
  </staticContent>
  
  <!-- Cache par type -->
  <location path="*.css">
    <system.webServer>
      <staticContent>
        <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="365.00:00:00" />
      </staticContent>
    </system.webServer>
  </location>
</system.webServer>
```

### Limites & Timeouts
```powershell
# Timeout connexion
Set-WebConfigurationProperty -Filter "/system.webServer/connectionManagement" -Name connectionTimeout -Value "00:02:00"

# Limites concurrent connections
Set-WebConfigurationProperty -Filter "/system.webServer/connectionManagement" -Name maxConcurrentRequestsPerCpu -Value 1000

# Application Pool limits
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name processModel.idleTimeout -Value "00:20:00"
Set-ItemProperty -Path "IIS:\AppPools\MonAppPool" -Name recycling.periodicRestart.privateMemory -Value 524288  # 512MB
```

## 🔧 Modules Utiles

### Installation Modules
```powershell
# URL Rewrite Module
# Télécharger depuis Microsoft.com

# Application Request Routing (ARR)
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationRequestRouting

# WebDAV
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebDAV

# FTP Server
Enable-WindowsOptionalFeature -Online -FeatureName IIS-FTPServer
```

### Configuration Load Balancing (ARR)
```xml
<system.webServer>
  <rewrite>
    <rules>
      <rule name="ARR_farm_loadbalance" patternSyntax="Wildcard" stopProcessing="true">
        <match url="*" />
        <action type="Rewrite" url="http://farm/{R:0}" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

## 🛠️ Troubleshooting

### Commandes Diagnostics
```powershell
# État services IIS
Get-Service W3SVC, WAS
Get-Website | Select Name, State, PhysicalPath
Get-WebAppPool | Select Name, State

# Vérifier bindings
Get-WebBinding

# Tester configuration
Test-WebConfigFile -Path "C:\inetpub\wwwroot\web.config"

# Processus IIS
Get-Process w3wp | Select Id, WorkingSet, ProcessName
```

### Événements Courants
| Event ID | Description | Action |
|----------|-------------|---------|
| 1309 | Application pool recycle | Vérifier logs application |
| 5057 | SSL certificate error | Vérifier certificat |
| 5059 | Crypto operation failed | Vérifier permissions certificat |
| 5152 | Health monitoring | Vérifier santé application |

### Failed Request Tracing
```powershell
# Activer FRT
Set-WebConfigurationProperty -Filter "/system.webServer/tracing/traceFailedRequests" -Name enabled -Value True

# Règle FRT pour erreurs 500
Add-WebConfigurationProperty -Filter "/system.webServer/tracing/traceFailedRequests" -Name "." -Value @{
    path="*";
    statusCodes="500-999";
    timeTaken="00:00:30"
}
```

### Reset IIS Complet
```powershell
# Reset configuration IIS
iisreset /stop
net stop was /y
del %windir%\system32\inetsrv\config\*.* /q
%windir%\system32\inetsrv\appcmd restore backup "Nom_Backup"
iisreset /start
```
