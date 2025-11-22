---
title: "From Zero to Secured Cloud: Building a Robust Docker Web Stack on Oracle Cloud (OCI)"
date: 2025-11-21 14:00:00 +0800
categories: [DevOps, Cloud]
tags: [Oracle Cloud, Docker, Nginx, Troubleshooting]
description: "Detailed guide to setting up a fully secured Docker web stack on Oracle Cloud (OCI). Covers VM provisioning, dual-layer firewall configuration, NPM deployment, and the definitive fix for 502 Bad Gateway errors caused by complex Docker network routing."
toc: true
---

## Introduction

This guide documents the full setup process for establishing a secure, self-hosted web development environment using the free-tier Oracle Cloud Infrastructure (OCI). We deployed key containerized services (Portainer, Nginx Proxy Manager, and Code-Server) on an Ubuntu VM. The core challenge involved stabilizing inter-container networking and resolving OCI's critical dual-firewall configuration.

---

## Phase 1: Oracle Cloud Infrastructure (OCI) and Network Setup

This phase is the backbone of the entire project, ensuring our virtual machine (VM) is properly created and accessible from the internet.

### 1. Provisioning the Ubuntu VM

The core infrastructure was provisioned via the OCI web console:

* **Instance Shape:** **Ampere A1 Compute Instance** (utilizing the Always Free tier).
* **Operating System:** **Ubuntu 22.04 LTS** (Arm-based image).
* **Network:** A new **VCN (Virtual Cloud Network)** and **Subnet** were created automatically during the instance setup.
* **Result:** A fully provisioned VM accessible via SSH using the public IP address: **`xxx.xxx.xxx.xxx`**.

### 2. Dual-Layer Firewall Configuration (The Critical Step)

OCI implements a dual-firewall system. Traffic must be explicitly allowed at the **Cloud Network Level (NSG)** and then again at the **Host OS Level (UFW)**. If traffic is blocked at either layer, the service is unreachable.

#### A. Layer 1: Cloud Network Security Group (NSG) Configuration

In the OCI Console, the **Ingress Rules** for the NSG attached to the instance were modified to allow traffic from the entire internet (`0.0.0.0/0`) to the following destination ports:

| Port | Protocol | Purpose |
| :--- | :--- | :--- |
| **22** | TCP | SSH Access (Default) |
| **80, 443** | TCP | Public Web Traffic (NPM) |
| **81** | TCP | NPM Admin Panel |
| **9443** | TCP | Portainer UI |
| **8443** | TCP | Code-Server Access |

#### B. Layer 2: Host OS Firewall (`ufw`) Configuration

Once logged into the server via SSH, the host-level firewall (`ufw`) was configured to mirror the same ingress rules. This ensures the operating system accepts connections on the designated ports.

```bash
# SSH into the Ubuntu instance first.

# Allow required ports
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 81/tcp
sudo ufw allow 9443/tcp
sudo ufw allow 8443/tcp

# Reload/Enable the UFW firewall rules
sudo ufw enable
````

### 3\. DNS and External Access

The final step for external access was pointing the domain and subdomains to the OCI Public IP at the registrar.

  * **A Records:** `@`, `*`, `portainer`, `npm`, `code`, and `blog`
  * **Value:** **`xxx.xxx.xxx.xxx`**

-----

## Phase 2: Base Infrastructure (Docker, NPM, & Code-Server)

The entire environment was deployed using a single, consolidated `docker-compose.yml` file, leveraging a key networking strategy.

### 1\. The Unified Docker Networking Strategy

To prevent future 502 Bad Gateway errors, a user-defined network, `web-network`, was created. This allows services to communicate using their **service names** instead of unreliable host IPs.

**Configuration in `docker-compose.yml`:**

```yaml
networks:
  default:
    external:
      name: web-network
```

### 2\. NPM and Service Renaming

For clarity and maintenance, services were renamed in the `docker-compose.yml`:

  * Nginx Proxy Manager: Renamed from `app` to **`npm`**.
  * Static Server: Renamed from `static-server` to **`blog`**.

### 3\. The 502 Bad Gateway Crisis (The Definitive Fix)

The most persistent issue was the **"502 Bad Gateway"** error for proxied services. This was caused by **Docker routing failure** when attempting to connect using the host's Private IP (e.g., `xx.xx.xx.xx`).

The definitive fix involved switching the **NPM Proxy Host configuration** to use the internal **Service Name**.

| Problematic Routing | Correct Routing |
| :--- | :--- |
| **Forward IP:** `xx.xx.xx.xx` (Private IP) | **Forward IP:** `code-server` (Service Name) |
| **Failure:** Request leaves the Docker network and is blocked trying to re-enter. | **Success:** Request stays within the `web-network` and routes instantly via Docker's internal DNS. |

This routing fix, combined with essential **Websocket Headers** (for Code-Server), stabilized all external access.

## Phase 3: Final Environment Configuration

The environment is now fully secured, managed via the unified stack.

### Final NPM Host Routing

| Service | Domain | Forward Hostname / IP | Forward Port |
| :--- | :--- | :--- | :--- |
| **NPM Admin** | `npm.domain.com` | **`npm`** | `81` |
| **Portainer** | `portainer.domain.com` | **`portainer`** | `9443` |
| **Code-Server** | `code.domain.com` | **`code-server`** | `8443` |

### File Permission Fix (The `chown` Command)

To resolve "Permission denied" errors during file operations, the ownership of the host directories mapped to the containers was corrected using the container's User ID (UID 1000):

```bash
# This grants full ownership of the home directory to the container user (UID 1000)
sudo chown -R 1000:1000 /home/ubuntu 
```

## Conclusion

The project successfully established a highly reliable, self-hosted platform. The primary success was in stabilizing the OCI networking environment by implementing the **service name routing pattern** across all proxies. The server is now ready for deployment and continuous integration workflows.

**The final secure development links are:**

  * **NPM Admin:** `https://npm.domain.com`
  * **Code-Server:** `https://code.domain.com`

