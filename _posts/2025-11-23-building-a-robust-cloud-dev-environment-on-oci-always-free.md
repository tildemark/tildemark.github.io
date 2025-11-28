---
title: Building a Robust Cloud Dev Environment on OCI Always Free (ARM64)
date: 2025-11-23 17:00:00 +/-TTTT
categories: [Tutorials, DevOps]
tags: [oracle cloud, docker, nginx proxy manager, portainer, code-server, postgres, security]
description: "Unlocking the full potential of Oracle Cloud's Always Free tier: A comprehensive guide to building a secure, self-hosted DevOps stack. Learn how to deploy Nginx Proxy Manager, Portainer, VS Code (Code-Server), and Postgres on ARM64 architecture with full SSL encryption and automated backups."
image:
  path: https://cdn.sanchez.ph/blog/cloud-dev-environment.webp
  alt: Cloud Architecture
---

In this deep-dive tutorial, we will walk through the process of transforming an Oracle Cloud Infrastructure (OCI) Always Free ARM instance into a powerful, secure, browser-based Cloud Development Environment (CDE).

**Why this guide?**
Most tutorials skip the hard stuff: storage limits, SSH lockouts, and ARM64 compatibility quirks. This guide covers the "Happy Path" *and* the "Disaster Prevention" strategies we learned the hard way.

> **Note on Images:** The images in this post are served via my own private CDN running on OCI Object Storage and Cloudflare Workers. You can read how I built that here: [Building a CDN with OCI + Cloudflare Workers](/posts/building-a-cdn-with-oci-plus-cloudflare-workers/).
{: .prompt-info }

## The Tech Stack
* **Infrastructure:** OCI Ampere (ARM64) 4 OCPU, 24GB RAM, **200GB Storage**.
* **OS:** Ubuntu 22.04 LTS (Jammy).
* **Gateway:** Nginx Proxy Manager (SSL/HTTPS).
* **Management:** Portainer.
* **IDE:** Code-Server (VS Code in browser).
* **Database:** PostgreSQL 15 + pgAdmin 4.
* **Cache:** Redis.
* **Gaming:** Minecraft Paper Server.
* **Security:** Dual-User Access, Fail2Ban, Internal Networking.
* **Backups:** Automated Daily Cron.

> **Omit Minecraft** if you are not confident of your IP being exposed. 
{: .prompt-tip }

---

## Prerequisites
1. An Oracle Cloud account with access to the Always Free tier.
2. A registered domain name (e.g., `yourdomain.com`). In this tutorial, we use `domain.ph` as the example.

## Phase 1: OCI Infrastructure Setup
Before launching a server, we need to lay the networking groundwork.

### 1. Networking (The Foundation)
To ensure our domain records never break, we need a static IP.
1.  In the OCI Console, go to **Networking** > **IP Management** > **Reserved Public IPs**.
2.  Reserve a new IP. Note it down (e.g., `141.148.153.48`).

### 2. Configure the Virtual Cloud Network (VCN) Firewall
OCI uses "Security Lists" as a firewall outside your VM. We must open HTTP/HTTPS ports.

1.  Go to **Networking** -> **Virtual Cloud Networks**.
2.  Click your VCN, then your **PUBLIC Subnet**, then the **Default Security List**.
3.  Add the following **Ingress Rules** (Stateful):

| Source CIDR | IP Protocol | Destination Port | Description |
| :--- | :--- | :--- | :--- |
| 0.0.0.0/0 | TCP | 22 | SSH Access |
| 0.0.0.0/0 | TCP | 80 | HTTP & Let's Encrypt |
| 0.0.0.0/0 | TCP | 443 | HTTPS |
| 0.0.0.0/0 | TCP | 81 | NPM (Temporary) |
| 0.0.0.0/0 | TCP | 25565 | Minecraft |
| 0.0.0.0/0 | TCP | 9090 | Cockpit |

> **Security Note:** Do not open ports 5432 (Postgres), 6379 (Redis), or 9000 (Portainer) here. We will keep those internal for security. Remove NPM after you set it up from the NPM interface.
{: .prompt-warning }

### 3. Launching the Instance (The "Safe" Way)
1.  **Image:** Select **Canonical Ubuntu 22.04**. (Avoid 20.04 as it requires manual Docker fixes).
2.  **Shape:** VM.Standard.A1.Flex (4 OCPU, 24GB RAM).
3.  **Storage:**
    * Click "Specify a custom boot volume size".
    * Change `50` to **200 GB**. (Maximize your free tier limit immediately).
4.  **SSH Keys (CRITICAL):**
    * We will use a **Dual-Key Strategy** to prevent lockouts.
    * Save the default key for user `ubuntu`.
    * *Advanced Tip:* Paste a **second** public key in the "Cloud-Init" script for a backup admin user (see Security section).

### 4. Attach the Reserved IP
Once the instance is running:
1.  Click the instance name to view details.
2.  Scroll down to **Attached VNICs** on the left menu.
3.  Click the Primary VNIC name.
4.  Scroll to **IPv4 Addresses**.
5.  Click the actions menu (`...`) next to the private IP and select **Edit**.
6.  Choose **Reserved public IP** and select the IP you reserved earlier.

### 5. Fixing the 47GB Storage Limit (Linux Native Method)
Even if you selected 200GB in the Oracle Console, Ubuntu initially only sees the default 47GB partition. We can expand this live using standard Linux tools, no extra Oracle utilities required.

```bash
# 1. Expand the partition to fill the disk
sudo growpart /dev/sda 1

# 2. Resize the filesystem to use the new space
sudo resize2fs /dev/sda1

# 3. Verify (Should show ~196G)
df -h /
```

### 6. Configuring the OS Firewall (Crucial)
Oracle Ubuntu images come with strict iptables rules. Even if you open ports in the OCI Console, the server itself will block connections unless you run these commands:

```bash
# Open Web Ports
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT

# Open Admin Ports (Temporary Setup + Cockpit)
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 81 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 9090 -j ACCEPT

# Open Game Ports (Minecraft)
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 25565 -j ACCEPT

# Save rules so they persist after reboot
sudo netfilter-persistent save
```

-----

## Phase 3: Security & "Anti-Lockout" Strategy

We learned this the hard way: **Docker volume mappings can break SSH permissions.** If you map `/home/ubuntu` directly, you *will* get locked out.

### 1\. The Backup Admin User

Create a second user with `sudo` access. If `ubuntu` permissions break, login as `sans`.

```bash
# Create user
sudo adduser sans
sudo usermod -aG sudo sans

# Copy SSH keys from ubuntu (or add a new one)
sudo mkdir -p /home/sans/.ssh
sudo cp /home/ubuntu/.ssh/authorized_keys /home/sans/.ssh/
sudo chown -R sans:sans /home/sans
sudo chmod 700 /home/sans/.ssh
sudo chmod 600 /home/sans/.ssh/authorized_keys
```

### 2\. The Golden Rule of Docker Volumes

> **NEVER** map `/home/ubuntu` to a container.  
**ALWAYS** map a sub-folder (e.g., `/home/ubuntu/workspace`).
{: .prompt-danger }

Create your safe directories now:

```bash
# Main stack folder
mkdir -p ~/my-stack

# The Safe Workspace for VS Code
mkdir -p ~/workspace

# Data folders for services
mkdir -p ~/my-stack/npm/data
mkdir -p ~/my-stack/npm/letsencrypt
mkdir -p ~/my-stack/portainer_data
mkdir -p ~/my-stack/code-config
mkdir -p ~/my-stack/postgres_data
mkdir -p ~/my-stack/pgadmin_data
mkdir -p ~/my-stack/minecraft_data
```

### 3\. Installing Cockpit
Having been locked out several times, installing **Cockpit** is a brilliant idea for a "fail-safe" backup.

**Why it works:**
Cockpit runs as a **System Service** (via `systemd`), not as a Docker container.

  * **If Docker crashes?** Cockpit stays alive.
  * **If you mess up Docker networking?** Cockpit stays alive.
  * **If you lock yourself out of SSH keys?** Cockpit provides a **Web Terminal** that accepts your username/password directly.

> It effectively acts as an "Emergency Escape Pod" that bypasses your entire Docker stack.
{: .prompt-tip }

**Step 1: Install Cockpit**

Run these commands on your server (SSH in as `ubuntu`):

```bash
# 1. Update and Install
sudo apt update
sudo apt install -y cockpit

# 2. Enable and Start the service
sudo systemctl enable --now cockpit.socket
```

**Step 2: Open the Firewalls (Critical)**

Cockpit listens on **Port 9090**. You need to open this port in **two** places.

**1. The OS Firewall (iptables):**

```bash
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 9090 -j ACCEPT
sudo netfilter-persistent save
```

**2. The Oracle Cloud Firewall (VCN):**

1.  Go to **Networking** \> **Virtual Cloud Networks**.
2.  Click your VCN \> **Security Lists** \> **Default Security List**.
3.  Add **Ingress Rule**:
      * **Source:** `0.0.0.0/0`
      * **Protocol:** TCP
      * **Destination Port:** `9090`

**Step 3: Access it**

Open your browser and go to:
**`https://<YOUR.PUBLIC.IP>:9090`**

  * **Security Warning:** Your browser will yell at you ("Your connection is not private"). This is normal because Cockpit uses a self-signed certificate.
      * Click **Advanced** \> **Proceed to... (unsafe)**.

**Step 4: Login**

You can log in with **any** user that has a password.

  * **User:** `backup_user` (or `ubuntu` if you set a password).
  * **Password:** The password you created.

Once logged in, click **"Terminal"** on the left menu. You now have full root access via the browser, completely independent of SSH keys\!

**Important: Do NOT put Cockpit behind Nginx**
{: .prompt-warning }
 
You might be tempted to set up `cockpit.sanchez.ph` in Nginx Proxy Manager. **Do not do this.**

**Why?**
If your goal is to have a "Backup Login" in case Docker breaks:

1.  Nginx is a Docker container.
2.  If Docker breaks, Nginx dies.
3.  If Nginx dies, `cockpit.domain.ph` stops working.
4.  You are locked out.

> **Keep Cockpit on the raw IP (Port 9090).** It must remain exposed directly to the internet to serve as a true backup. Since it requires a username/password, it is reasonably secure (and Fail2Ban protects it automatically on Ubuntu).
{: .prompt-tip }

-----

## Phase 3: The Docker Stack

We use **Docker Compose** to manage everything.

### 1\. Install Docker (Ubuntu 22.04)

```bash
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
sudo usermod -aG docker backup_user
newgrp docker
```
### 2\. Install Docker (The ARM64 Way)

{: .prompt-info }
The standard automated install script (`get.docker.com`) often fails on OCI ARM64 instances due to missing specific architecture packages. We must install it manually.

Run these commands to install Docker correctly on ARM64:

```bash
# Remove potentially conflicting packages
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Set up Docker's official Apt repository
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker packages (excluding problematic ones)
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-buildx-plugin docker-ce-rootless-extras

# Add current user to docker group so you don't need sudo
sudo usermod -aG docker $USER
newgrp docker

# Test
docker run hello-world
```

### 2\. The Master Compose File

Create `~/my-stack/docker-compose.yml`.

> **Note:** Replace `net` with whatever network name you prefer, but ensure it is consistent.
{: .prompt-info }

```yaml
services:
  # 1. Nginx Proxy Manager (Gateway)
  npm:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: npm
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
      - '81:81' # Admin port
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
    volumes:
      - ./npm/data:/data
      - ./npm/letsencrypt:/etc/letsencrypt
    networks:
      - net

  # 2. Portainer (Management)
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./portainer_data:/data
    networks:
      - net

  # 3. Code Server (VS Code)
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Manila
      - PASSWORD=securepassword123 # CHANGE THIS
    volumes:
      - ./code-config:/config
      # !!! SAFE MAPPING: Sub-folder only! !!!
      - /home/ubuntu/workspace:/home/coder/workspace
    networks:
      - net

  # 4. Redis (Cache)
  redis:
    image: redis:alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass "redispassword123" # CHANGE THIS
    networks:
      - net

  # 5. Postgres (Database)
  postgres:
    image: postgres:15-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: pguser
      POSTGRES_PASSWORD: pgxcvsdfwer234 # CHANGE THIS
      POSTGRES_DB: pgdb
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
    ports:
      # Expose to Localhost only (for secure SSH Tunneling)
      - "127.0.0.1:5432:5432"
    networks:
      - net

  # 6. pgAdmin (Web DB Admin)
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin
    restart: unless-stopped
    environment:
      PGADMIN_DEFAULT_EMAIL: "admin@sanchez.ph"
      PGADMIN_DEFAULT_PASSWORD: "secureadminpassword123" # CHANGE THIS
    volumes:
      - ./pgadmin_data:/var/lib/pgadmin
    networks:
      - net

  # 7. Watchtower (Auto-Updater)
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --interval 86400 --cleanup
    networks:
      - net

  # 8. Minecraft Server
  minecraft:
    image: itzg/minecraft-server
    container_name: minecraft
    restart: unless-stopped
    ports:
      - "25565:25565" # TCP/UDP Game Port
    environment:
      EULA: "TRUE"
      TYPE: "PAPER"
      MEMORY: "6G"
      MOTD: "Welcome to Sanchez Cloud"
    volumes:
      - ./minecraft_data:/data
    networks:
      - net

networks:
  net:
    name: net
```

### 3\. Start the Stack

Before starting, set permissions for pgAdmin's data folder:

```bash
sudo chown -R 5050:5050 ~/my-stack/pgadmin_data
```

Start the containers:

```bash
docker compose up -d
```

-----

## Phase 4: Networking & Nginx Proxy Manager

This setup uses a mix of **Proxied** and **DNS Only** records in Cloudflare, and specific Nginx configurations.

### 1\. Cloudflare Records

| Type | Name | Content | Proxy Status | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| A | npm | `YOUR_RESERVED_IP` | **Proxied (Orange)** | NPM Dashboard |
| A | code | `YOUR_RESERVED_IP` | **Proxied (Orange)** | VS Code |
| A | portainer | `YOUR_RESERVED_IP` | **Proxied (Orange)** | Portainer |
| A | mc | `YOUR_RESERVED_IP` | **DNS Only (Grey)** | Minecraft (TCP) |


> **Minecraft requires "DNS Only":** Cloudflare's free proxy only handles HTTP/HTTPS. It will block Minecraft traffic. You must turn off the Orange Cloud for the `mc` subdomain.
{: .prompt-warning }

### 2\. OCI Firewall Rules

Go to your VCN Security List and open these ports:

  * **80/443 (TCP):** Web Traffic
  * **25565 (TCP):** Minecraft
  * **22 (TCP):** SSH
  * **81 (TCP):** NPM Setup (Close this after setup is complete).
  * **9090 (TCP):** Cockpit

Access NPM at `http://YOUR_IP:81` initially. Create the following Proxy Hosts:

| Domain | Forward Hostname | Port | Websockets Support | Force SSL |
| :--- | :--- | :--- | :--- | :--- |
| `npm.domain.ph` | `npm` | 81 | Optional | **Yes** |
| `portainer.domain.ph` | `portainer` | 9000 | **Required** | **Yes** |
| `code.domain.ph` | `code-server` | 8443 | **Required** | **Yes** |
| `db.domain.ph` | `pgadmin` | 80 | Optional | **Yes** |

**Critical Configurations:**

  * **Websockets Support:** Must be enabled for **Portainer** (for the console to work) and **Code-Server** (for the terminal to work).
  * **Force SSL:** Always enable this in the SSL tab to ensure all traffic is encrypted.
  * **Block Common Exploits:** Recommended for all hosts.

### 3\. Configuring the OS Firewall (Crucial)

Oracle Ubuntu images come with strict `iptables` rules. Even if you open ports in the OCI Console, the server itself will block connections unless you run these commands:

```bash
# Open Web Ports
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT

# Open Admin Ports (Temporary Setup + Cockpit) 
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 81 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 9090 -j ACCEPT
# ignore cockpit if you already did this at the top

# Open Game Ports (Minecraft)
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 25565 -j ACCEPT

# Save rules so they persist after reboot
sudo netfilter-persistent save
```

-----

## Phase 5: Automated Backups

Don't rely on luck. We set up a script to dump the database and compress the files.

### 1\. The Backup Script

Create `~/auto_backup.sh`:

```bash
#!/bin/bash
BACKUP_DIR="/home/ubuntu/backups"
SOURCE_DIR="/home/ubuntu/my-stack"
WORKSPACE_DIR="/home/ubuntu/workspace"
DATE=$(date +%Y-%m-%d_%H-%M)
FILENAME="backup-$DATE.tar.gz"

mkdir -p $BACKUP_DIR

echo "Starting backup..."
# 1. Hot SQL Dump
docker exec -t postgres pg_dump -U pguser pgdb > $SOURCE_DIR/postgres_backup.sql

# 2. Compress Stack + Workspace
tar -czf $BACKUP_DIR/$FILENAME $SOURCE_DIR $WORKSPACE_DIR

# 3. Cleanup
rm $SOURCE_DIR/postgres_backup.sql
find $BACKUP_DIR -type f -name "*.tar.gz" -mtime +7 -delete
echo "Done."
```

### 2\. Cron Job

Run `crontab -e` and add:

```bash
0 3 * * * /home/ubuntu/auto_backup.sh >> /home/ubuntu/backup.log 2>&1
```

-----

## Phase 6: Security Hardening

1.  **Install Fail2Ban:** Protect SSH from brute-force attacks.
    ```bash
    sudo apt install -y fail2ban
    ```
2.  **Verify Exposed Ports:** Only ports 22, 80, and 443 should be open on your OCI VCN Security list.

### Alternative Secure Database Access: SSH Tunneling

Instead of exposing pgAdmin to the web, you can use a desktop client (like pgAdmin desktop) and tunnel securely through SSH.

  * **DB Host:** `127.0.0.1` (Localhost relative to the tunnel end)
  * **DB Port:** `5432`
  * **SSH Tunnel Host:** `YOUR.PUBLIC.IP`
  * **SSH Tunnel User:** `ubuntu`
  * **SSH Key:** Your private key file.

-----

## Troubleshooting Guide

### Issue 1: Portainer Timeout

**Problem:** Accessing Portainer resulted in a timeout or error page.
**Cause:** Portainer locks its login page for security if an admin isn't created within 5 minutes of startup.
**Fix:** Restart the container and immediately create the user.

```bash
docker restart portainer
```

### Issue 2: Postgres Database Name Not Changing

**Problem:** Changing `POSTGRES_DB` in docker-compose didn't create the new database name.
**Cause:** Postgres only initializes the database on the *first* run. Subsequent runs use existing data in the volume.
**Fix:** Stop containers, wipe the volume data, and restart.

```bash
docker compose down
sudo rm -rf ~/my-stack/postgres_data
docker compose up -d
```

### Issue 3: Code-Server missing Node/NPM

**Problem:** The `code-server` image is minimal. Running `npm init` in the terminal failed.
**Fix:** We must install dependencies as root from outside the container.

```bash
# From the host SSH terminal:
docker exec -it -u 0 code-server bash
# Now inside the container as root:
apt-get update && apt-get install -y nodejs npm
exit
```
### Issue 4: Terminal "sudo" Password Errors
**Problem:** When trying to install packages (like `npm` or `imagemagick`) inside the VS Code terminal using `sudo`, it asks for a password for user `abc`. Entering the web login password fails.
**Cause:** The `linuxserver/code-server` image runs as a limited user named `abc`. The `PASSWORD` variable in Docker Compose only secures the web interface; it does not set the system user's password.
**Fix:** You cannot perform system-level installations (`apt install`) from within the VS Code browser terminal easily. You must access the container as **root** from your server's SSH session.

1.  Open your main SSH terminal (not VS Code).
2.  Log in to the container as root:
    ```bash
    docker exec -it -u 0 code-server bash
    ```
3.  Run your install commands:
    ```bash
    apt-get update
    apt-get install -y imagemagick nodejs npm
    ```
4.  Type `exit` to return to the host.

### Issue 5: "Permission Denied (publickey)" on SSH

  * **Cause:** You mapped `/home/ubuntu` in Docker, and it changed the folder permissions. SSH rejects "insecure" home folders.
  * **Fix:**
    1.  Login as your backup user (`sans`).
    2.  Run `sudo chmod 755 /home/ubuntu`.
    3.  Run `sudo chmod 700 /home/ubuntu/.ssh`.
    4.  Run `sudo chmod 600 /home/ubuntu/.ssh/authorized_keys`.
  * **Prevention:** Use the "Safe Workspace" mapping strategy above.

### Issue 6: Minecraft "Connection Refused"

  * **Cause:** You likely have the Cloudflare Proxy (Orange Cloud) turned on.
  * **Fix:** Switch the `mc` A record to **DNS Only** (Grey Cloud) in the Cloudflare dashboard.

### Issue 7: Code-Server "npm command not found"

  * **Cause:** The Code-Server image is minimal. You cannot use `sudo` inside the browser terminal easily.
  * **Fix:**
    1.  Go to your main SSH terminal.
    2.  Run `docker exec -it -u 0 code-server bash`.
    3.  Run `apt-get update && apt-get install -y nodejs npm`.

### Issue 8: Docker Compose Network Error

  * **Cause:** Renaming networks (e.g., from `sanchez_net` to `net`) without downing the stack first.
  * **Fix:** Run `docker compose down` then `docker compose up -d`.

-----

## Conclusion

You now have a developer environment that rivals paid VPS options costing $50/month. It's backed up, redundant, and runs on high-performance ARM architecture.

Happy Coding\!
