# Ports well-known & ranges (résumé) ⚓

- Well-known: `0-1023` (services système ; droits admin requis côté serveur).
- Registered: `1024-49151`.
- Dynamic/Private: `49152-65535` (ephemeral).

| Port | Proto | Service |
|------|-------|---------|
| 20   | TCP   | FTP données |
| 21   | TCP   | FTP contrôle |
| 22   | TCP   | SSH |
| 23   | TCP   | Telnet (obsolète) |
| 25   | TCP   | SMTP |
| 53   | UDP/TCP | DNS |
| 67/68| UDP   | DHCP (Srv/Client) |
| 69   | UDP   | TFTP |
| 80   | TCP   | HTTP |
| 110  | TCP   | POP3 |
| 123  | UDP   | NTP |
| 137-139 | UDP/TCP | NetBIOS |
| 143  | TCP   | IMAP |
| 161/162 | UDP | SNMP |
| 389  | TCP/UDP | LDAP |
| 443  | TCP   | HTTPS |
| 445  | TCP   | SMB/CIFS |
| 514  | UDP   | Syslog |
| 587  | TCP   | SMTP (STARTTLS) |
| 636  | TCP   | LDAP SSL |
| 993  | TCP   | IMAP SSL |
| 995  | TCP   | POP3 SSL |
| 1433 | TCP   | MSSQL |
| 3306 | TCP   | MySQL |
| 3389 | TCP/UDP | RDP |