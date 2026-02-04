# Server Bootstrap Log

This document lists every action performed by the `server_bootstrap` script, categorized by the stack configuration selected during execution.

### Execution Log

**(base)** Validates that only one stack (`--LAMP` or `--GAP`) is selected to prevent conflicts  
**(base)** Sets password for the `root` user to ensure manual login/recovery capability  
**(base)** Updates system package lists and upgrades all installed packages to their latest versions  
**(base)** Installs baseline toolset: `fish`, `curl`, `ca-certificates`, `gnupg`, `lsb-release`, `git`, `fail2ban`, `ufw`, `certbot`  
**(base)** Ensures a 1GB swap file exists at `/swapfile` (using `fallocate` or `dd`) and persists it in `/etc/fstab`  
**(base)** Caps systemd journal logs at 200MB and vacuums existing logs to prevent disk exhaustion  
**(base)** Applies Kernel/Network security hardening via `sysctl`: Disables IP forwarding, ignores ICMP redirects, enables SYN cookie protection, and enables RP filtering (spoofing protection)  
**(base)** Configures Fail2ban with 1h bantime, 10m findtime, and sets SSH jail to `normal` mode  
**(base)** Enables `unattended-upgrades` for automatic background security patching  
**(base)** Creates the deploy user `lewis` (if missing) and adds them to the `sudo` group  
**(base)** Prompts to set a password for the `lewis` user via an interactive TTY session  
**(base)** Imports public SSH keys from GitHub (`Terrydaktal.keys`) for both `root` and the deploy user **(Safety: Done before lockout hardening)**  
**(base)** Performs a timestamped backup of all SSH configuration files to `/root/ssh-backup-...`  
**(base)** Hardens SSH: Enforces settings via `/etc/ssh/sshd_config.d/00-bootstrap.conf` and disables password auth  
**(base)** Restricts SSH features: Disables `X11Forwarding`, `AllowAgentForwarding`, and `PermitTunnel`  
**(base)** Sets SSH connection hygiene: `MaxAuthTries 10`, `LoginGraceTime 20`, and 10-hour idle timeouts  
**(base)** Validates SSH configuration syntax (`sshd -t`) before restarting the daemon  
**(base)** Configures UFW firewall: Sets default deny incoming / allow outgoing policies  
**(base)** Applies UFW rate-limiting on OpenSSH to mitigate connection-based attacks  
**(base)** Enables UFW firewall with 'medium' logging level  

**(lamp)** Installs full stack: `apache2`, `mariadb-server`, `php`, `libapache2-mod-php`, and common PHP extensions  
**(lamp)** Installs stack-specific extras: `composer`, `python3-certbot-apache`  
**(lamp)** Enables and starts the `apache2` and `mariadb` system services  
**(lamp)** Automates MariaDB security: Removes anonymous users, drops test database, and disables remote root login  
**(lamp)** Enables Apache `rewrite` and `headers` modules  
**(lamp)** Sets Apache `DirectoryIndex` to prioritize `index.php` over `index.html`  
**(lamp)** Hardens Apache globally: Hides version info, disables signature, and disables `TraceEnable`  
**(lamp)** Injects global security headers via Apache: `nosniff`, `SAMEORIGIN`, `strict-origin-when-cross-origin`  
**(lamp)** Hardens all installed PHP versions: Disables `expose_php`, `display_errors`, and enables `log_errors`  
**(lamp)** Deep-hardens PHP: Disables dangerous functions (`exec`, `system`, etc.) and restricts file access via `open_basedir`  
**(lamp)** Secures PHP sessions: Enforces `cookie_secure`, `cookie_httponly`, and `cookie_samesite = Lax`  
**(lamp)** Validates Apache configuration syntax (`apache2ctl configtest`) before reloads  
**(lamp)** Updates UFW to allow incoming traffic on ports 80 (HTTP) and 443 (HTTPS)  

**(gap)** Installs full stack: Go (`golang-go`), Nginx, PostgreSQL, and `postgresql-contrib`  
**(gap)** Installs stack-specific extras: `python3-certbot-nginx`  
**(gap)** Enables and starts the `nginx` and `postgresql` system services  
**(gap)** Hardens Nginx: Hides version info, adds security headers (`X-Frame-Options`, `X-Content-Type-Options`), and sets buffer limits  
**(gap)** Hardens PostgreSQL: Tunes `max_connections` and enforces local peer authentication in `pg_hba.conf`  
**(gap)** Updates UFW to allow incoming traffic on ports 80 (HTTP) and 443 (HTTPS)  

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
