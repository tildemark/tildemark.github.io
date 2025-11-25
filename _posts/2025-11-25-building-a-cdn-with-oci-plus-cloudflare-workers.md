---
title: How I Built a Free, Enterprise-Grade CDN with OCI and Cloudflare Workers
date: 2025-11-25 12:00:00 +0000
categories: [Tutorial, Devops]
tags: [cloudflare, oci, cdn, serverless, self-hosting]
image:
  path: https://cdn.sanchez.ph/blog/free-cdn-with-oci-plus-cloudflare-workers.webp
  alt: Architecture of OCI Object Storage and Cloudflare Workers
---

I recently set out to build a custom CDN (like `cdn.sanchez.ph`) to host assets for my blog and projects. My requirements were simple: **high performance, custom domain, and $0 cost.**

I combined **Oracle Cloud Infrastructure (OCI)** Always Free Object Storage (20GB) with **Cloudflare's** global edge network.

However, I hit a snag: OCI requires specific Host Headers to access files, and Cloudflare locks the "Host Header Rewrite" rule behind their $200/month Enterprise plan. Additionally, simply pointing the DNS exposed a directory listing of all my files to the public.

**The Solution:** I used a **Cloudflare Worker** to bypass the header limitation, secure the root directory, and build a powerful, free CDN. Here is exactly how I built it.

## The Architecture

1.  **Origin:** OCI Object Storage bucket (hosting the files).
2.  **Edge:** Cloudflare (caching and SSL).
3.  **The Bridge:** A Cloudflare Worker script that rewrites URLs, Headers, and enforces security on the fly.

## Step 1: Setting up Oracle Cloud (OCI)

I created a public bucket in OCI Object Storage.
* **Region:** US Phoenix
* **Bucket Name:** `cdn-bucket`
* **Visibility:** Public (Objects are publicly readable)

*Note:* OCI recently updated their URL structure to use namespace-specific endpoints. My direct file link looked like this:
`https://axjnmgygqr0w.objectstorage.us-phoenix-1.oci.customer-oci.com/n/axjnmgygqr0w/b/cdn-bucket/o/welcome.webp`

![File at OCI](https://axjnmgygqr0w.objectstorage.us-phoenix-1.oci.customer-oci.com/n/axjnmgygqr0w/b/cdn-bucket/o/welcome.webp){: .normal }

My goal was to turn that long URL into `https://cdn.sanchez.ph/welcome.jpg`.

## Step 2: Onboarding to Cloudflare (Zero Downtime)

Before changing my nameservers, I added `sanchez.ph` to Cloudflare. This is where a huge time-saver comes in: **Automatic DNS Scanning**.

1.  I clicked **Add Site** and selected the **Free Plan**.
2.  Cloudflare automatically scanned my domain and imported all my existing DNS records (like my VPS IP, mail records, etc.).
    * *Pro Tip:* This saved me from typing them manually!
3.  With my existing records safe, I added my **new** CDN record:
    * **Type:** `CNAME`
    * **Name:** `cdn`
    * **Target:** `axjnmgygqr0w.objectstorage.us-phoenix-1.oci.customer-oci.com`
    * **Proxy Status:** Proxied (Orange Cloud)

Only *after* verifying everything looked correct did I log into my registrar (`dot.ph`) and update the Nameservers to point to Cloudflare.

## Step 3: The "Worker" Workaround

If you try to use standard Cloudflare Page Rules to rewrite the Host Header, you will find the option greyed out on the free plan. Instead, I used **Cloudflare Workers** (which offers 100,000 requests/day for free).

1.  I navigated to **Compute (Workers)** in the Cloudflare dashboard.
2.  I created a "Hello World" worker and replaced the code with the script below.
    * *Update:* I added a check to block the root path (`/`). Without this, anyone visiting `cdn.sanchez.ph` would see a raw XML list of every file in my bucket!

```javascript
export default {
  async fetch(request) {
    const url = new URL(request.url);
    
    // My OCI Configuration
    const ociHost = "axjnmgygqr0w.objectstorage.us-phoenix-1.oci.customer-oci.com";
    const bucketPath = "/n/axjnmgygqr0w/b/cdn-bucket/o";

    // Only intercept my CDN subdomain
    if (url.hostname === "cdn.sanchez.ph") {
      
      // SECURITY: Block users from listing all files at the root URL
      if (url.pathname === "/" || url.pathname === "") {
        return new Response("Access Denied", { status: 403 });
      }

      // 1. Point to Oracle's Server
      url.hostname = ociHost;

      // 2. Rewrite the path (add the long namespace/bucket prefix)
      url.pathname = bucketPath + url.pathname;

      // 3. Create a new request with the correct Host Header
      // This is the key step that the standard Rules couldn't do!
      const newRequest = new Request(url.toString(), {
        method: request.method,
        headers: request.headers,
        body: request.body,
        redirect: 'follow'
      });
      newRequest.headers.set("Host", ociHost);

      return fetch(newRequest);
    }
    return fetch(request);
  }
};
````

3.  Finally, I went to **Workers Routes** in my domain settings and assigned the route `cdn.sanchez.ph/*` to this worker.

## Troubleshooting & Common Pitfalls

Everything rarely goes perfectly on the first try. Here are the real-world issues I encountered and how I fixed them.

### 1\. The "Negative Cache" Trap

After setting up a new subdomain like `npm.sanchez.ph`, I kept getting `ERR_NAME_NOT_RESOLVED` on my computer, even though I had added the DNS record.

  * **The Cause:** My computer (and ISP) had "remembered" that the site didn't exist 10 minutes ago.
  * **The Fix:**
      * Use a tool like **DNSChecker.org** to verify the record is actually live globally.
      * Flush local DNS (`ipconfig /flushdns`).
      * Toggle Airplane mode on mobile devices to force a fresh lookup.

### 2\. Nginx Proxy Manager (NPM) Configuration

I set up the DNS correctly in Cloudflare, but the browser gave me "502 Bad Gateway" errors when trying to access my `npm` dashboard.

  * **The Cause:** Cloudflare knew where to send the traffic, but my Nginx Proxy Manager didn't know it was supposed to receive it.
  * **The Fix:** I had to log into NPM and create a **Proxy Host** specifically for `npm.sanchez.ph` that pointed to the container itself (`host.docker.internal` on port 81).

### 3\. Email Stopped Working

Cloudflare showed a warning on my `mail` record: *"This record exposes the IP address..."*

  * **The Cause:** I initially had the `mail` record set to **Proxied (Orange Cloud)**. Cloudflare only proxies web traffic (HTTP), so it was inadvertently blocking my email traffic (SMTP).
  * **The Fix:** I changed the `mail` record to **DNS Only (Grey Cloud)**.
      * *Note:* The warning about exposing the IP is normal. You cannot host email without exposing the mail server's IP address. Since my mail server is on a different IP than my CDN setup, there was no security risk.

## The Result

 ![Final output](https://cdn.sanchez.ph/welcome.webp){: .normal }


Now, when I request `https://cdn.sanchez.ph/welcome.webp`:

1.  Cloudflare intercepts the request.
2.  The Worker rewrites the destination to the deep OCI path.
3.  The Worker fetches the file securely from Oracle.
4.  Cloudflare caches the result globally.

I now have a professional, SSL-secured CDN with a custom domain—running entirely on free-tier infrastructure.
