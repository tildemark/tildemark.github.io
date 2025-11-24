---
title: Building a Robust Cloud Dev Environment on OCI Always Free (ARM64)
date: 2025-11-23 17:00:00 +/-TTTT
categories: [Tutorials, DevOps]
tags: [oracle cloud, docker, nginx proxy manager, portainer, code-server, postgres, security]
description: "Unlocking the full potential of Oracle Cloud's Always Free tier: A comprehensive guide to building a secure, self-hosted DevOps stack. Learn how to deploy Nginx Proxy Manager, Portainer, VS Code (Code-Server), and Postgres on ARM64 architecture with full SSL encryption and automated backups."
image:
  path: https://res.cloudinary.com/dxsshynmg/image/upload/cloud-dev-environment_vedujk.webp
  alt: Cloud Architecture
---

In this deep-dive tutorial, we will walk through the process of transforming an Oracle Cloud Infrastructure (OCI) Always Free ARM instance into a powerful, secure, browser-based Cloud Development Environment (CDE).

By the end of this guide, you will have a server running VS Code in the browser, a hidden Postgres/Redis backend, a web-based database management tool, and a beautiful dashboard to manage your containers—all secured behind free SSL certificates.

Here is the stack we are building:
* **Infrastructure:** OCI Ampere (ARM64) 4 OCPU, 24GB RAM.
* **OS:** Ubuntu 20.04 LTS.
* **Reverse Proxy & SSL:** Nginx Proxy Manager (NPM).
* **Container Management:** Portainer.
* **IDE:** Code-Server (VS Code).
* **Database:** PostgreSQL 15 (Hidden internally).
* **DB GUI:** pgAdmin 4 (Web).
* **Cache:** Redis.
* **Security:** Fail2Ban, OS Firewall, Internal Docker Networking.
* **Maintenance:** Automated Daily Backups via Cron.

Let's get building.

## Prerequisites

1.  An Oracle Cloud account with access to the Always Free tier.
2.  A registered domain name (e.g., `yourdomain.com`). In this tutorial, we use `domain.ph` as the example.

---

## Phase 1: OCI Infrastructure Setup

Before launching a server, we need to lay the networking groundwork.

### 1. Reserve a Public IP
To ensure our domain records never break, we need a static IP address.

1.  In the OCI Console, go to **Networking** -> **IP Management** -> **Reserved Public IPs**.
2.  Click **Reserve Public IP Address**. Give it a name (e.g., `cde-static-ip`).
3.  Note down the IP address you are assigned (e.g., `111.111.111.111`).

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

{: .prompt-warning }
**Security Note:** Do not open ports 5432 (Postgres), 6379 (Redis), or 9000 (Portainer) here. We will keep those internal for security.

### 3. Launch the Instance
1.  Go to **Compute** -> **Instances** -> **Create Instance**.
2.  **Image:** Select **Ubuntu 20.04** (or 22.04).
3.  **Shape:** Select the **Ampere** (VM.Standard.A1.Flex) shape. Max out the OCPUs (4) and RAM (24GB) if you want the full power.
4.  **Networking (Advanced Options):**
    * Specify a private IP if you wish (e.g., `10.0.0.196`).
    * **IMPORTANT:** Select "Do not assign a public IPv4 address". We will attach our reserved one later.
5.  **SSH Keys:** Download your private key and keep it safe.
6.  Click **Create**.

### 4. Attach the Reserved IP
Once the instance is running:
1.  Click the instance name to view details.
2.  Scroll down to **Attached VNICs** on the left menu.
3.  Click the Primary VNIC name.
4.  Scroll to **IPv4 Addresses**.
5.  Click the actions menu (`...`) next to the private IP and select **Edit**.
6.  Choose **Reserved public IP** and select the IP you reserved earlier.

---

## Phase 2: Domain DNS Configuration

Go to your domain registrar's DNS settings. We will create `A` records pointing subdomains to your new OCI Public IP.

| Type | Host/Name | Value/Target |
| :--- | :--- | :--- |
| A | npm | `YOUR.OCI.PUBLIC.IP` |
| A | portainer | `YOUR.OCI.PUBLIC.IP` |
| A | code | `YOUR.OCI.PUBLIC.IP` |
| A | db | `YOUR.OCI.PUBLIC.IP` |

---

## Phase 3: Server Preparation

SSH into your new server:
```bash
ssh -i path/to/your.key ubuntu@YOUR.IP
````

### 1\. Update and Secure OS Firewall

Oracle's Ubuntu images come with `iptables` rules pre-configured. We must allow web traffic through the OS firewall.

```bash
sudo apt update && sudo apt upgrade -y
# Allow HTTP and HTTPS
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT
# Save the rules persistently
sudo netfilter-persistent save
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

-----

## Phase 4: The Docker Compose Stack

We will use Docker Compose to define our entire environment in a single file.

### 1\. Directory Structure

Create a home for your stack and its persistent data.

```bash
mkdir -p ~/my-stack/{npm/{data,letsencrypt},portainer_data,code-config,project-data,postgres_data,pgadmin_data}
cd ~/my-stack
```

### 2\. The `docker-compose.yml` File

Create the file: `nano docker-compose.yml`.

Be sure to change the passwords marked with `# CHANGE THIS`.

```yaml
services:
  # 1. Nginx Proxy Manager (The Gateway)
  npm:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: npm
    restart: unless-stopped
    ports:
      - '80:80'   # HTTP
      - '443:443' # HTTPS
      - '81:81'   # NPM Admin Interface (Temporary)
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
    volumes:
      - ./npm/data:/data
      - ./npm/letsencrypt:/etc/letsencrypt
    networks:
      - net

  # 2. Portainer (Docker Management)
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    volumes:
      - /etc/localtime:/etc/localtime:ro
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
      - PASSWORD=ChangeThis123 # CHANGE THIS for web login
    volumes:
      - ./code-config:/config
      - ./project-data:/home/coder/project
    networks:
      - net

  # 4. Redis (Cache)
  redis:
    image: redis:alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass "ChangeThis123" # CHANGE THIS
    networks:
      - net

  # 5. Postgres (Database)
  postgres:
    image: postgres:15-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: ChangeThis123 # CHANGE THIS
      POSTGRES_DB: db
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
    networks:
      - net
    # NOTE: No 'ports' section. Database is hidden from the internet.

  # 6. pgAdmin (Web Database GUI)
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin
    restart: unless-stopped
    environment:
      PGADMIN_DEFAULT_EMAIL: "admin@domain.ph"  # Login Email
      PGADMIN_DEFAULT_PASSWORD: "ChangeThis123" # Login Password
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

## Phase 5: Nginx Proxy Manager Configuration

We need to configure the reverse proxy to route traffic from domains to containers and handle SSL.

### The "Chicken and Egg" Problem (Important\!)

Nginx Proxy Manager's admin interface runs on port 81. To secure it behind a domain (like `npm.yourdomain.com`), we first need to access it via the raw IP to set it up.

### 1\. Access NPM Initially

1.  Ensure Port 81 is allowed in your OCI Security List and OS `iptables` (we did this in the compose file and OS firewall setup, but OCI VCN might need a temporary rule).
2.  Navigate to `http://YOUR.IP:81`.
3.  Login with default credentials (`admin@example.com` / `changeme`) and immediately change them.

### 2\. Configure Proxy Hosts

For each subdomain, create a Proxy Host.

**Example: Portainer (`portainer.domain.ph`)**

  * **Domain Names:** `portainer.domain.ph`
  * **Scheme:** `http`
  * **Forward Hostname:** `portainer` (Matches container name in YAML)
  * **Forward Port:** `9000`
  * **Websockets Support:** **Enable** (Crucial for Portainer/Code-Server).
  * **SSL Tab:** "Request a new SSL Certificate", Force SSL, Agree to TOS.

Repeat for `code.domain.ph` (port 8443) and `db.domain.ph` (port 80, forwarding to `pgadmin` container).

### 3\. Secure NPM Itself and Close Port 81

Now, configure `npm.domain.ph` to point to the NPM container.

{: .prompt-tip }
**Troubleshooting Tip:** When configuring NPM to proxy *itself*, do **not** use the public IP as the Forward Hostname if Port 81 is blocked on the firewall. Use `127.0.0.1` or `npm` as the Forward Hostname and port `81`. This keeps traffic internal.

Once `https://npm.domain.ph` is working:

1.  Edit `docker-compose.yml` and remove `  - '81:81' ` from the `npm` service ports.
2.  Run `docker compose up -d` to apply.
3.  Remove Port 81 from your OCI VCN Security List if you added it.

-----

## Phase 6: Troubleshooting & Customization

During this setup, we encountered and solved several real-world issues.

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

-----

## Phase 7: Security Hardening

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

## Phase 8: Automated Backups

We need a robust backup strategy that captures database data and configurations.

### 1\. The Backup Script

Create `~/auto_backup.sh` and make it executable (`chmod +x`).

```bash
#!/bin/bash
# --- Configuration ---
BACKUP_DIR="/home/ubuntu/backups"
SOURCE_DIR="/home/ubuntu/my-stack"
DATE=$(date +%Y-%m-%d_%H-%M)
FILENAME="backup-$DATE.tar.gz"
DB_CONTAINER="postgres"
DB_USER="pguser"
DB_NAME="pgdb"

mkdir -p $BACKUP_DIR

echo "Starting backup for $DATE..."
# 1. Hot SQL Dump
docker exec -t $DB_CONTAINER pg_dump -U $DB_USER $DB_NAME > $SOURCE_DIR/postgres_backup.sql
# 2. Compress Stack
tar -czf $BACKUP_DIR/$FILENAME $SOURCE_DIR
# 3. Cleanup Temp SQL
rm $SOURCE_DIR/postgres_backup.sql
# 4. Delete backups older than 7 days
find $BACKUP_DIR -type f -name "*.tar.gz" -mtime +7 -delete
echo "Backup Complete: $BACKUP_DIR/$FILENAME"
```

### 2\. The Cron Job

Run `crontab -e` and add this line to run daily at 3 AM:

```bash
0 3 * * * /home/ubuntu/auto_backup.sh >> /home/ubuntu/backup.log 2>&1
```

### How to Restore

To restore from disaster:

1.  `cd ~/my-stack && docker compose down`
2.  `sudo tar -xzvf ~/backups/YOUR_BACKUP_FILE.tar.gz -C /`
3.  `docker compose up -d`

-----

## Conclusion

You have successfully built a professional, secure Cloud Development Environment for free on Oracle Cloud. You have overcome ARM64 specific challenges, navigated complex networking, and established a robust backup routine. Happy coding in the cloud\!
