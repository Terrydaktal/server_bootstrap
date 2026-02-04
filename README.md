# Server Bootstrap Log

This document lists every action performed by the `server_bootstrap` script, categorized by the stack configuration selected during execution.

### Execution Log

**(base)** Sets password for the `root` user  
**(base)** Updates system package lists and upgrades all installed packages  
**(base)** Installs base tools: `fish`, `curl`, `ca-certificates`, `gnupg`, `lsb-release`, `git`, `fail2ban`, `ufw`, `certbot`  
**(base)** Ensures a 1GB swap file exists at `/swapfile` and is persisted in `/etc/fstab`  
**(base)** Caps systemd journal logs at 200MB and vacuums old entries  
**(base)** Configures Fail2ban with 1h bantime, 10m findtime, and normal SSH mode  
**(base)** Enables `unattended-upgrades` for automatic security patching  
**(base)** Creates the deploy user `lewis` (if missing) and adds them to the `sudo` group  
**(base)** Sets password for the `lewis` user  
**(base)** Imports public SSH keys from GitHub for both `root` and `lewis`  
**(base)** Hardens SSH: Disables password auth, limits retries, and disables unneeded features (X11, tunnels)  
**(base)** Configures UFW firewall: Default deny incoming, allow outgoing, and rate-limit SSH  
**(base)** Enables UFW firewall and sets logging level to medium  

**(lamp)** Installs Apache2, MariaDB, PHP stack, Composer, and `python3-certbot-apache`  
**(lamp)** Enables and starts `apache2` and `mariadb` services  
**(lamp)** Enables Apache `rewrite` and `headers` modules  
**(lamp)** Sets Apache `DirectoryIndex` to prefer `index.php`  
**(lamp)** Configures Apache security hardening: Hides version info, disables Trace, and adds security headers  
**(lamp)** Hardens PHP: Disables `expose_php`, hides errors, and secures session cookies  
**(lamp)** Opens UFW ports 80 (HTTP) and 443 (HTTPS)  
**(lamp)** Provides a reminder to run `mysql_secure_installation` for the database  

**(gap)** Installs Go (`golang-go`), Nginx, PostgreSQL, and `python3-certbot-nginx`  
**(gap)** Enables and starts `nginx` and `postgresql` services  
**(gap)** Opens UFW ports 80 (HTTP) and 443 (HTTPS)  

---

## Usage

1. **Download**:
   ```bash
   curl -O https://raw.githubusercontent.com/Terrydaktal/server_bootstrap/main/server_bootstrap
   chmod +x server_bootstrap
   ```

2. **Run**:
   - **LAMP**: `sudo ./server_bootstrap --LAMP`
   - **GAP**: `sudo ./server_bootstrap --GAP`
   - **Base Only**: `sudo ./server_bootstrap`