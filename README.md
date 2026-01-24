# Server Bootstrap Script

This is an opinionated, idempotent Bash script designed to bootstrap a fresh Debian/Ubuntu server into a secure, production-ready LAMP environment. It handles everything from user creation and SSH key management to deep security hardening of system services.

## Features

### 1. System Basics & Resource Management
- **OS Updates**: Runs `apt-get update` and `upgrade`.
- **Swap Management**: Checks for and creates a **1GB swap file** (`/swapfile`) if no active swap is found. Persists it in `/etc/fstab`.
- **Journald Hygiene**: Caps systemd journal logs at **200MB** and enables vacuuming to prevent disk exhaustion.
- **Packages**: Installs essential tools (`curl`, `git`, `fish`, `ufw`, `fail2ban`, `certbot`, `composer`) and the LAMP stack (`apache2`, `mariadb-server`, `php`).

### 2. User & Access Management
- **Deploy User**: Creates a dedicated user (default: `lewis`) with `sudo` privileges.
- **SSH Keys**: Automatically fetches and installs public SSH keys from a GitHub keys URL for both `root` and the deploy user.

### 3. Security Hardening

#### SSH (Secure Shell)
Configures `sshd` using a "first-match-wins" strategy via a dedicated drop-in file (`/etc/ssh/sshd_config.d/00-bootstrap.conf`) to ensure settings are enforced:
- **Authentication**: Disables password authentication; enforces public key authentication.
- **Hygiene**: Sets `MaxAuthTries 10`, `LoginGraceTime 20s`, `ClientAliveInterval 300s`.
- **Feature Restrictions**: Disables `X11Forwarding`, `AllowAgentForwarding`, and `PermitTunnel`.

#### Firewall (UFW)
- **Default Policy**: Deny incoming, allow outgoing.
- **SSH**: Rate-limited (`ufw limit OpenSSH`) to mitigate brute-force attacks.
- **Web**: Allows ports 80 (HTTP) and 443 (HTTPS).
- **Logging**: Enabled at 'medium' level.

#### Fail2ban
- **Backend**: Configured to use `systemd`.
- **Banaction**: Uses `ufw` to block IPs.
- **Jails**: Enables `sshd` jail in `aggressive` mode.

#### Unattended Upgrades
- Installs and enables `unattended-upgrades` for automatic security patching.

### 4. Web Stack (LAMP) Configuration

#### Apache
- **Modules**: Enables `rewrite` and `headers`.
- **Security Headers**: Enforces `X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN`, and `Referrer-Policy: strict-origin-when-cross-origin`.
- **Information Hiding**: Sets `ServerTokens Prod`, `ServerSignature Off`, `TraceEnable Off`.
- **Defaults**: Sets `DirectoryIndex` to prefer `index.php`.

#### PHP
Hardens `php.ini` for all installed PHP versions:
- **Exposure**: `expose_php = Off`.
- **Errors**: `display_errors = Off`, `log_errors = On`.
- **Session Security**: Enforces `cookie_secure`, `cookie_httponly`, and `cookie_samesite = Lax`.

#### MariaDB
- Installs the server and reminds the user to run `mysql_secure_installation` for interactive hardening.

## Usage

1. **Download and Run**:
   ```bash
   curl -O https://raw.githubusercontent.com/Terrydaktal/server_bootstrap/main/server_bootstrap
   chmod +x server_bootstrap
   sudo ./server_bootstrap
   ```

2. **Verify**:
   - Check firewall status: `ufw status verbose`
   - Check SSH config: `sshd -T`
   - Check Swap: `free -h`

## Configuration
Variables at the top of the script can be customized:
- `DEPLOY_USER`: The username for the deploy account.
- `GH_KEYS_URL`: URL to fetch SSH public keys (e.g., `https://github.com/username.keys`).
- `SWAP_SIZE`: Size of the swap file (default `1G`).
- `JOURNAL_CAP`: Max size for systemd journals (default `200M`).
