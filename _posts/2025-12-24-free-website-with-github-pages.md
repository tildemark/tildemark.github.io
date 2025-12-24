---
title: "How to Build Your Free Website or Blog: From Simple HTML to Professional Chirpy Theme"
date: 2025-12-24 8:00:00 +0800
categories: [Tutorial, Web Dev]
tags: [github pages, chirpy, html, beginners, markdown]
description: This is the complete, definitive guide for your blog post. It is structured to be "student-friendly," moving from absolute basics to professional-grade configurations.
image:
  path: https://cdn.sanchez.ph/blog/blog-tutorial-preview.webp
  alt: A split screen showing HTML code on one side and a professional Chirpy blog on the other.
---

# 🚀 Launch Your Professional Blog for Free with GitHub Pages

Whether you are a student building a portfolio or a hobbyist sharing your passion, **GitHub Pages** is the gold standard. It is 100% free, highly reliable, and looks great on a resume.

In this guide, we will cover two paths: the **Simple HTML** method for beginners and the **Chirpy Theme** for those who want a professional "developer" look.

---

## Phase 1: Create Your GitHub Account

GitHub is the "Cloud" where your website files live.

1. Go to [GitHub.com](https://github.com).
2. Sign up with a professional username.
> **⚠️ Critical:** Your username determines your URL. If your username is `tildemark`, your blog will be `tildemark.github.io`. Pick something you like!
3. Verify your email address to unlock all features.

---

## Phase 2: Create Your Repository

A "Repository" (or repo) is basically a project folder.

1. Click the **+** icon (top-right) and select **New repository**.
2. **Repository Name:** This **must** be exactly: `yourusername.github.io`
3. **Public/Private:** Select **Public**.
4. Check **Add a README file**.
5. Click **Create repository**.

---

## Phase 3: Choose Your Path

### Option A: The "Code Learner" (Simple HTML)

*Best for: Learning how the web works from scratch.*

1. In your repo, click **Add file** > **Create new file**.
2. Name it `index.html`.
3. Paste this "Starter Code":
```html
<!DOCTYPE html>
<html>
  <head><title>My Student Blog</title></head>
  <body>
    <h1>Hello World!</h1>
    <p>This is my first website hosted on GitHub Pages.</p>
  </body>
</html>
```

4. Click **Commit changes**.

### Option B: The "Professional" (Chirpy Theme)

*Best for: A sleek, dark-mode blog with search, tags, and categories.*

1. Go to the [Chirpy Starter Template](https://github.com/cotes2020/chirpy-starter).
2. Click **Use this template** > **Create a new repository**.
3. Name this repo `yourusername.github.io` (just like in Phase 2).
4. This copies a professional design into your account.

---

## Phase 4: Activating the Website (Settings)

You must tell GitHub to "Turn on" the website.

1. In your repository, click the **Settings** tab.
2. On the left sidebar, click **Pages**.
3. **For Option A (HTML):** Ensure Source is **"Deploy from a branch"**. Select `main` and `/ (root)`.
4. **For Option B (Chirpy):** Change Source to **"GitHub Actions"**. This is required for Chirpy to build itself.

---

## Phase 5: Custom Domains & CNAME

If you buy a domain (like `sanchez.ph`), follow these steps:

1. **On GitHub:** In **Settings > Pages**, type your domain (e.g., `www.yourname.com`) and click **Save**.
2. **The CNAME File:** GitHub will automatically create a file named `CNAME` in your repo.
> **⚠️ Warning:** Do not delete this file! If you delete it, your custom domain will stop working. If you do accidentally delete it, just re-type your domain in the Settings and Save again.

3. **At your Domain Provider:** Add an **A Record** pointing to these IPs:
* `185.199.108.153` 
* `185.199.109.153` 
* `185.199.110.153`
* `185.199.111.153`

---

## Phase 6: Enforce HTTPS (Security)

After your domain is linked, wait for GitHub to issue your SSL certificate (this can take up to 24 hours).

1. Go back to **Settings > Pages**.
2. Look for **Enforce HTTPS**.
3. **Check this box.** This ensures visitors see the "Padlock" icon, indicating your site is secure. If it’s grayed out, wait a few more hours and try again.

---

## 📝 Writing Your First Post (Markdown Guide)

If you chose Chirpy, you write in **Markdown** (`.md`). It’s simpler than coding!

| Feature | How to Type It | Result |
| --- | --- | --- |
| **Headers** | `# Header 1` or `## Header 2` | Large bold titles |
| **Emphasis** | `**Bold Text**` or `*Italic*` | **Bold Text** or *Italic* |
| **Lists** | `* Item 1` or `1. Item 1` | Bulleted or Numbered lists |
| **Links** | `[Click Me](https://google.com)` | [Click Me](https://google.com) |
| **Images** | `![Alt Text](url-to-image.jpg)` | Displays the image |
| **Quotes** | `> This is a quote` | Blockquotes |
| **Code** | ``inline code`` | `inline code` |

---

## 🛠️ Troubleshooting

| Problem | Cause & Fix |
| --- | --- |
| **404 Not Found** | **Fix:** Wait 2 minutes. GitHub needs time to build the site. Also, check that your repo name is exactly `username.github.io`. |
| **Chirpy is all white/broken** | **Fix:** Open `_config.yml` and check the `url` line. It must be `https://username.github.io` with **no** slash at the end. |
| **Post won't show up** | **Fix:** Check the file name in the `_posts` folder. It MUST be `YYYY-MM-DD-title.md`. If the date is in the future, it won't show up yet! |
| **CNAME issues** | Ensure you didn't accidentally delete the `CNAME` file in your main folder. |
| **Red "X" in Actions** | Click the **Actions** tab. It will tell you the error. Usually, it's a typo in the `_config.yml` file. |

---

## Summary for Newbies

* **HTML Path:** Fast, simple, you control every pixel.
* **Chirpy Path:** Professional, uses Markdown, handles the design for you.
* **Settings:** Remember—HTML uses "Deploy from Branch," while Chirpy uses "GitHub Actions."

---
