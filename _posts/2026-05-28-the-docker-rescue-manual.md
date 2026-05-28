---
layout: post
title: "The Docker Rescue Manual: Troubleshooting Containers and Deployments"
date: 2026-05-28 19:41:00 +0800
categories: [DevOps, Docker]
tags: [docker, docker-compose, troubleshooting, terminal, cheat-sheet, cicd]
excerpt: "When your stack refuses to deploy or containers crash in an endless loop, you need a reliable set of commands to diagnose the issue. Here is your Docker troubleshooting Swiss Army knife."
image:
  path: https://cdn.sanchez.ph/blog/docker-rescue-manual.webp
  alt: The image features a stylized Docker whale in a lifebuoy rescuing a container from waves on a dark, technical background with terminal text and circuit patterns.
---

If you’ve worked with Docker long enough, you know the feeling: an image builds perfectly on your local machine, but the moment it hits the CI/CD pipeline or the staging server, it crashes and burns. Or worse, the container says it's "running," but your app is throwing 404s because a crucial configuration file is missing. 

Following up on my previous posts like *The Git Rescue Manual* and *The Windows SSH Swiss Army Knife*, I realized that Docker demands its own survival guide. When your stack refuses to deploy or containers crash in an endless loop, you need a reliable set of commands to diagnose the issue. 

Here is your Docker troubleshooting Swiss Army knife.

---

## 1. Inspecting the Crime Scene (Basic Diagnostics)

When a container fails, your first step is to figure out *why*. These commands help you gather the initial clues.

* **See the living and the dead:**
    ```bash
    docker ps -a
    ```
    *Why you need it:* `docker ps` only shows running containers. Adding `-a` shows you containers that have exited or crashed, along with their exit codes (e.g., `Exited (137)` usually means an Out-Of-Memory kill, while `Exited (1)` means an application error).

* **Follow the logs:**
    ```bash
    docker logs --tail 100 -f <container_name_or_id>
    ```
    *Why you need it:* This tails the last 100 lines of the container's standard output/error and follows along in real-time. This is where you’ll usually spot your Python tracebacks, Node.js crashes, or Nginx syntax errors.

* **The Deep Dive (Metadata & Environment):**
    ```bash
    docker inspect <container_name_or_id>
    ```
    *Why you need it:* This dumps a massive JSON object with everything about the container. Use it to verify that your Environment Variables (`Env`), volume mounts (`Mounts`), and networking setups (`NetworkSettings`) were actually passed into the container correctly.

---

## 2. "Are My Files Actually There?" (Investigating the Filesystem)

One of the most common CI/CD issues is a `COPY` command in your `Dockerfile` failing silently, or a volume mount overshadowing your deployed files. 

* **Shelling into a running container:**
    ```bash
    docker exec -it <container_name> /bin/sh
    # or /bin/bash if available
    ```
    *Why you need it:* This gets you inside the running environment. From here, you can run `ls -la`, `cat config.yml`, or `curl localhost:8080` to see exactly what the container sees.

* **Exploring a container that crashes immediately:**
    If a container dies before you can `docker exec` into it, you need to intercept it. You can override the entrypoint to launch a shell instead of the failing app:
    ```bash
    docker run --rm -it --entrypoint /bin/sh <image_name>
    ```
    *Why you need it:* This prevents the app from crashing the container, giving you an interactive shell to explore the filesystem, check permissions, and manually run your start script to see exactly where it fails.

* **Extracting a file to inspect locally:**
    ```bash
    docker cp <container_name>:/usr/src/app/config.json ./local-config.json
    ```
    *Why you need it:* Sometimes you don't have the right tools inside the container (like `vim` or `jq`) to read a file. This copies the file out to your host machine so you can inspect it comfortably.

---

## 3. When the Stack Won't Deploy (Docker Compose Issues)

Deploying complex stacks introduces networking and dependency headaches. When `docker compose up -d` doesn't work as expected:

* **Validate the Compose file:**
    ```bash
    docker compose config
    ```
    *Why you need it:* If you are using multiple `.env` files or compose overrides (`docker-compose.override.yml`), this command parses them all and spits out the final, merged configuration. It’s perfect for checking if your variables interpolated correctly.

* **Check the aggregate logs:**
    ```bash
    docker compose logs -f
    ```
    *Why you need it:* Watch the logs for the entire stack at once. Often, the web container is crashing because the database container failed to initialize. This helps you see the chronological relationship between different services.

* **Investigate network isolation:**
    ```bash
    docker network ls
    docker network inspect <network_name>
    ```
    *Why you need it:* If Container A can't talk to Container B, use `network inspect` to ensure they are actually attached to the same network and check what IP addresses they were assigned.

---

## 4. CI/CD Pipeline & Build Struggles

When the build fails in Jenkins, GitHub Actions, or GitLab CI due to image composition:

* **Audit the image layers:**
    ```bash
    docker history <image_name>
    ```
    *Why you need it:* If your image size suddenly balloons by 2GB, `docker history` shows you exactly which command (or layer) introduced the bloat.

* **Bust the cache:**
    ```bash
    docker build --no-cache -t <image_name> .
    ```
    *Why you need it:* Sometimes Docker uses a cached layer (like an old `npm install` or `apt-get update`) that contains outdated dependencies, causing the build to fail further down the line. Forcing a build with no cache proves whether your Dockerfile actually works from scratch.

---

## 5. The Silent Killers (Resource Constraints)

Sometimes the code is fine, but the container's environment constraints are choking it out.

* **Monitor live resource usage:**
    ```bash
    docker stats
    ```
    *Why you need it:* It provides a live, `top`-like view of all running containers. If you see a container hitting 100% of its memory limit, you’ve found the reason it's intermittently crashing (OOMKill).

* **Reclaim stolen disk space:**
    ```bash
    docker system df
    docker system prune -a --volumes
    ```
    *Why you need it:* CI/CD runners often fail simply because they run out of disk space from keeping hundreds of dangling images and orphaned volumes. `system df` tells you where the space went, and `prune` acts as the nuclear option to clean up unused data (Use with caution!).

---

## 6. Networking Nightmares (Advanced Routing & Connectivity)

Basic networking checks are great, but what happens when your container refuses to connect to an external API, or your database container rejects the connection?

* **Verify Host Port Bindings:**
    ```bash
    docker port <container_name>
    ```
    *Why you need it:* Sometimes you map port `8080:80` in Compose, but another service silently hijacked it, or the binding failed. This command instantly tells you exactly which host port is mapping to which container port, cutting through the noise of `docker ps`.

* **The "Netshoot" Sidecar Hack:**
    ```bash
    docker run -it --rm --net container:<target_container_name> nicolaka/netshoot
    ```
    *Why you need it:* This is a god-tier troubleshooting trick. When your failing container is running a minimal image (like `scratch` or `alpine`) and lacks diagnostic tools, this command attaches a fully loaded network troubleshooting container (containing `tcpdump`, `nmap`, `curl`, `nslookup`) directly to the failing container's network namespace. You can debug the network exactly as if you were inside the broken container.

---

## 7. The "Permission Denied" Purgatory

Volume mounts are notorious for causing UID/GID (User ID / Group ID) conflicts. A file is created by root inside the container, and suddenly your host user can't edit it. 

* **Check the Container's Identity:**
    ```bash
    docker exec -it <container_name> id
    ```
    *Why you need it:* This tells you exactly which user context the container is currently running under. If it returns `uid=1000(node)` but your host mounted files are owned by `root`, you have found your problem.

* **Force a Root Shell (The Override):**
    ```bash
    docker exec -u 0 -it <container_name> /bin/sh
    ```
    *Why you need it:* If your container runs as an unprivileged user but you need to read restricted logs, install a quick debugging tool via `apt`/`apk`, or change permissions on the fly to test a fix, passing `-u 0` forces the execution as the root user.

---

## 8. CI/CD & BuildKit Hacks (Seeing Through the Matrix)

CI/CD runners handle Docker differently than your local terminal. They don't handle interactive outputs well, and sometimes they swallow the exact error message you need.

* **Force Plain Text Build Logs:**
    ```bash
    DOCKER_BUILDKIT=1 docker build --progress=plain -t <image_name> .
    ```
    *Why you need it:* Modern Docker uses BuildKit, which displays a fancy, interactive, collapsing progress spinner. In a CI pipeline, this can obscure the actual error message or truncate `stdout` from your package manager. Setting `--progress=plain` forces Docker to print every single line of output chronologically.

* **The "Hot-Patch" (Skipping the Build Wait):**
    ```bash
    docker cp ./fixed-script.js <container_name>:/app/src/fixed-script.js
    docker restart <container_name>
    ```
    *Why you need it:* When you are iterating on a bug in a staging environment, waiting 10 minutes for the entire CI/CD pipeline to rebuild and push the image is agonizing. You can use `docker cp` to copy your local fix *into* the running container, restart it, and test your theory instantly before committing the code.

---

## 9. When Docker Itself is Choking (Host-Level Issues)

Sometimes it's not your code. Sometimes it's not the container. Sometimes the Docker daemon itself is failing.

* **Check for Inode Exhaustion:**
    ```bash
    df -i
    ```
    *Why you need it:* You run `docker system df` and see you have 50GB of free space, yet Docker says `No space left on device`. You haven't run out of gigabytes; you've run out of *inodes* (the metadata tracking files). This happens frequently on CI runners that build Docker images with millions of tiny files (like `node_modules`).

* **Interrogate the Docker Daemon Logs:**
    ```bash
    journalctl -u docker.service --no-pager --tail 100
    ```
    *Why you need it:* If `docker run` just hangs indefinitely, or the Docker socket refuses to connect, the problem is at the system level. This command dumps the system logs for the Docker service itself (on Linux hosts), revealing core issues like networking daemon crashes or storage driver failures.
