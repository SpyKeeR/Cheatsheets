## Version & identification
- `cat /etc/os-release` (ou `/etc/debian_version`) → version distro.  
- `uname -a` → noyau & archi.  
- `lscpu` → CPU. `lsmem` → mémoire. `lspci` → PCI. `lsusb` → USB.

## Monitors & perf
- `top` → (raccourcis: `M` tri mémoire, `P` tri CPU).
- `htop`, `glances` → monitoring interactif. 
- `ps -ef` → liste process statique. `procs` → friendly ps.  
- `free -h` → mémoire / swap.
- `iotop` → IO par process.
- `iostat` (sysstat) → disques/CPU stats.
- `dstat`, `dstat -...` → overview.  
- `systemd-analyze blame` → services qui ralentissent boot.  
- `systemd-analyze critical-chain` → chaîne critique systemd.  
- `procs` → `ps` friendly.  
- `watch cmd` → exécuter en boucle (live).  
- `progress` → voir progression copies.  
- `shred` → suppression multiple passée.  
- `ts` → timestamp lines (piping).  
- `errno` → doc code erreur.

# Diagnostic 
- `lsof` → (Fichiers ouverts)
- `strace` → (debug process), 
- `perf` → (profiling) — si besoin.
- `systemctl` → gestion services (start/stop/status/enable/disable).