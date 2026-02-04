# Server Bootstrap Script

This is an opinionated, idempotent Bash script designed to bootstrap a fresh Debian/Ubuntu server into a secure, production-ready environment. It handles everything from user creation and SSH key management to deep security hardening of system services.

## Features

### 1. System Basics & Resource Management
- **OS Updates**: Runs `apt-get update` and `upgrade`.
- **Swap Management**: Checks for and creates a **1GB swap file** (`/swapfile`) if no active swap is found. Persists it in `/etc/fstab`.
- **Journald Hygiene**: Caps systemd journal logs at **200MB** and enables vacuuming to prevent disk exhaustion.
- **Base Packages**: Installs essential tools (`curl`, `git`, `fish`, `ufw`, `fail2ban`, `certbot`).
- **Enhanced Logging**: Every step is logged, and existing packages/configurations are skipped automatically (idempotency).

### 2. Stack Options

The script supports two mutually exclusive application stacks. You can only install one at a time.

#### LAMP Stack (`--LAMP`)
- **Web Server**: Apache2 with hardening (`ServerTokens Prod`, `ServerSignature Off`).
- **PHP**: Hardened `php.ini` (`expose_php = Off`, secure session cookies).
- **Database**: MariaDB.
- **Extras**: Composer, `python3-certbot-apache`.

#### GAP Stack (`--GAP`)
- **Language**: Go (`golang-go`).
- **Web Server/Proxy**: Nginx.
- **Database**: PostgreSQL.
- **Extras**: `python3-certbot-nginx`.

### 3. User & Access Management
- **Password Prompts**: The script will prompt you to set passwords for both the `root` user and the deploy user during execution.
- **Deploy User**: Creates a dedicated user (default: `lewis`) with `sudo` privileges.
- **SSH Keys**: Automatically fetches and installs public SSH keys from a GitHub keys URL for both `root` and the deploy user.

### 4. Security Hardening

#### SSH (Secure Shell)
Configures `sshd` using a "first-match-wins" strategy via a dedicated drop-in file (`/etc/ssh/sshd_config.d/00-bootstrap.conf`) to ensure settings are enforced:
- **Authentication**: Disables password authentication; enforces public key authentication.
- **Hygiene**: Sets `MaxAuthTries 10`, `LoginGraceTime 20s`, `ClientAliveInterval 300s`.
- **Feature Restrictions**: Disables `X11Forwarding`, `AllowAgentForwarding`, and `PermitTunnel`.

#### Firewall (UFW)
- **Default Policy**: Deny incoming, allow outgoing.
- **SSH**: Rate-limited (`ufw limit OpenSSH`) to mitigate brute-force attacks.
- **Web**: Allows ports 80 (HTTP) and 443 (HTTPS) only if `--LAMP` or `--GAP` is selected.
- **Logging**: Enabled at 'medium' level.

#### Fail2ban
- **Backend**: Configured to use `systemd`.
- **Banaction**: Uses `ufw` to block IPs.
- **Jails**: Enables `sshd` jail in `normal` mode with a **1 hour** ban time.

#### Unattended Upgrades
- Installs and enables `unattended-upgrades` for automatic security patching.

## Usage

1. **Download and Run**:
   ```bash
   curl -O https://raw.githubusercontent.com/Terrydaktal/server_bootstrap/main/server_bootstrap
   chmod +x server_bootstrap
   ```

2. **Select your stack**:
   - For **LAMP**: `sudo ./server_bootstrap --LAMP`
   - For **GAP**: `sudo ./server_bootstrap --GAP`
   - For **Baseline security only**: `sudo ./server_bootstrap`

3. **Verify**:
   - Check firewall status: `ufw status verbose`
   - Check SSH config: `sshd -T`
   - Check Swap: `free -h`

## Configuration
Variables at the top of the script can be customized:
- `DEPLOY_USER`: The username for the deploy account.
- `GH_KEYS_URL`: URL to fetch SSH public keys (e.g., `https://github.com/username.keys`).
- `SWAP_SIZE`: Size of the swap file (default `1G`).
- `JOURNAL_CAP`: Max size for systemd journals (default `200M`).

## Directory Structure
- `server_bootstrap`: The main installation script.
- `README.md`: Documentation.
