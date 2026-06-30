---
title: Introducing Vault Drop Explorer — A Lightweight S3 and OCI Desktop File Manager
date: 2026-07-01 12:00:00 +0000
categories: [projects, tools]
tags: [s3, oci, aws, tauri, rust, react]
image:
  path: https://cdn.sanchez.ph/blog/vault-drop-explorer.webp
  alt: Vault Drop Explorer Application Layout
---

Managing files in the cloud shouldn't require logging into bloated web consoles or typing commands in a terminal. 

Today, I am officially releasing **Vault Drop Explorer (v1.0.0)**, a lightweight, secure, and open-source desktop client built for AWS S3 and Oracle Cloud Infrastructure (OCI) Object Storage. 

It is designed to do one thing exceptionally well: let you drop files from your desktop straight into your cloud buckets—no configuration hassle, and no telemetry.

## The Problem: Bloated Web Consoles & Complex CLI Tools

AWS S3 and OCI are industry standards for object storage, but their official web interfaces are built for enterprise system administrators, not fast-paced developers. Simply uploading a file or downloading assets requires navigating multiple nested menus, handling session timeouts, and waiting for heavy single-page applications to render.

On the other hand, CLI tools like `aws-cli` or `rclone` are powerful but lack visual feedback. When you just want to verify that an asset uploaded, drag-and-drop a folder structure, or double-check a layout, a desktop client is far more efficient.

## The Solution: Vault Drop Explorer

Vault Drop Explorer is built on **Tauri (v2)**, **Rust**, and **React** with a dark, premium glassmorphism interface. It acts as a lightweight companion that sits on your desktop, ready to handle file drops and downloads instantly.

Here is what makes it unique:

### 1. Native Drag & Drop
By hooking into Tauri's native window events, the application bypasses Chromium's webview sandbox. You can drag single files or folder groups directly from Windows Explorer into the app to trigger immediate S3/OCI uploads.

### 2. Collapsing Directory Browsing
Unlike basic S3 clients that list objects in a flat, confusing list, Vault Drop Explorer implements directory parsing. It uses delimiters (`/`) and S3's `common_prefixes` to auto-collapse folders, displaying files in a clean, expandable tree structure similar to Windows Explorer.

### 3. Deep OCI S3-Compatibility Support
One of the biggest hurdles when building custom clients for Oracle Cloud (OCI) Object Storage is their S3-compatible API compliance. OCI does not support standard AWS chunked transfer encoding or body payload checksum signing (`x-amz-content-sha256`), which modern S3 SDK clients enforce by default. 

Vault Drop Explorer solves this internally by configuring client requests to use unsigned payloads and disabling request/response checksum calculations programmatically, enabling flawless OCI uploads.

### 4. Credentials Persistence
If you manage multiple S3-compatible accounts, typing key IDs and secret keys repeatedly is annoying. Vault Drop Explorer automatically checks and loads credential profiles from your local system (`~/.aws/credentials`) and `.env` files. If you type them inline, checking **"Remember connection"** saves them locally in your browser's secure `localStorage`.

### 5. Local-First & Private
As indicated by the header lock badge, all keys are saved and processed **100% locally**. The app communicates directly with AWS/OCI endpoints using HTTPS. No remote servers or cloud database services are used, ensuring your access keys stay strictly private.

---

## Technical Stack Under the Hood

*   **Frontend**: React (TypeScript) + Tailwind CSS (using standard shadcn/ui components).
*   **Backend**: Rust (Tauri commands) utilizing the official `aws-sdk-s3` Cargo crates for fast, multi-threaded S3 operations.
*   **Packaging**: Optimized MSIs and NSIS setup installers (`.exe`) compiled directly for Windows platforms.

## How to Get Started

You can download the stable release (`v1.0.0`) directly from GitHub:

*   **Setup Installer**: [Download Vault Drop Explorer v1.0.0](https://github.com/tildemark/vault-drop-explorer/releases/latest/download/Vault.Drop.Explorer_1.0.0_x64-setup.exe)
*   **Web Version**: Try it online at [vaultdrop.sanchez.ph](https://vaultdrop.sanchez.ph)
*   **Source Code**: Check out the repository on [GitHub](https://github.com/tildemark/vault-drop-explorer)

*Dev: Alfredo Sanchez, Jr (sanchez.ph)*
