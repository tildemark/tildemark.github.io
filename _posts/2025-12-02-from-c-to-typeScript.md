---
title: "From C++ to TypeScript: A Dinosaur's Journey Back to the Future"
date: 2025-12-02 08:00:00 +0800
categories: [DevLog, Personal]
tags: [history, javascript, typescript, ci/cd, wordpress, movable-type]
toc: true
comments: true
math: false
mermaid: false
pin: true
image:
  path: https://cdn.sanchez.ph/blog/blog-tech-cycle.webp
  alt: A retro computer terminal next to a modern laptop.
---

## The Old Guard Joins the New School

Let’s get one thing straight: I come from the era where "memory management" meant you actually had to manage the memory. I cut my teeth on **C/C++**, scripted the backend with **Perl**, built enterprise giants with **Java**, and hacked together the early web with **PHP**.

![from c to js](https://cdn.sanchez.ph/blog/cpp-to-js.webp)
But today? Today, I am officially jumping ship to the land of **JavaScript and TypeScript**.

Yes, I know. I can hear my old C compiler weeping in the distance. Despite the massive learning curve, I’ve decided to embrace this modern approach that the Gen Z devs seem to love so much. But let me tell you, it hasn’t been a walk in the park.

## The Trauma of "Modern" WordPress

To understand why I’m here, you have to understand where I came from.

I started my blogging journey with **Movable Type**. And honestly? I loved it. It was static, it was robust, and it felt like *engineering*. But then, as all good things do, it went behind a paywall. I dabbled with Open Melody, but eventually, like the rest of the internet, I migrated to **WordPress**.

I stayed in the WordPress ecosystem for a long time. I built themes, I developed plugins, and I thought I was happy. But then came the swarm.
* **Comment Spam:** An endless tide of bots selling things that shouldn't be sold.
* **Hacking Attempts:** waking up to find scripts injected into my headers.
* **DDoS Attacks:** My bandwidth drained by zombie computers, forcing me to take the site down.

It got to the point where I lost content. I was rewriting articles multiple times because of database corruptions or malicious attacks. Blogging became a chore. It was hassle over joy. So, I quit.

## The Irony of the Tech Circle

Here is the funny part about my return to blogging using **Jekyll (Chirpy Theme)** and GitHub Pages.

Back in the Movable Type days, people criticized it heavily. They said, *"It's too troublesome to compile the whole site just to publish a post!"* They laughed at static generation. They said **WordPress** was the future because it was dynamic—instant gratification.

Fast forward to 2025. What is everyone doing?
They are moving away from dynamic CMS platforms in favor of TypeScript, Next.js, and Static Site Generators.

> **The Hypocrisy:**
>
> **2005:** "Movable Type is dumb. Why compile static files? It's overkill to deploy an instance just to add a comma."
>
> **2025:** "WordPress is bloated. We need to compile static files for speed! Let's deploy a containerized build pipeline just to add a comma."

Technology really does revolve in circles. We just gave the "compile" step a fancy new name: **The Build Pipeline**.

## The Hurdle: `vi` vs. CI/CD

Speaking of build pipelines, this is my current nemesis.

I come from the school of **Cowboy Coding**.
1.  Open FileZilla.
2.  Drag file to server.
3.  Refresh browser.
4.  *Error?* SSH into the server.
5.  `vi index.php`
6.  Fix bug live in production.
7.  Go to lunch.

![modern ci/cd](https://cdn.sanchez.ph/blog/modern-cicd.webp)

Now, I have to deal with **CI/CD** (Continuous Integration/Continuous Deployment). I can't just edit a file. I have to:
1.  Write code in VS Code.
2.  `git add .`
3.  `git commit -m "fixed typo"`
4.  `git push`
5.  Wait for GitHub Actions to spin up a virtual machine, download dependencies, build the site, check for errors, and deploy.

It feels like using a bazooka to kill a mosquito. But, I have to admit... when it works, it’s clean. And nobody can hack my static HTML files with a SQL injection.

## A New Chapter with Chirpy

![mt to worpress to ts](https://cdn.sanchez.ph/blog/mt-wp-ts.webp)
So here I am, writing in Markdown, pushing to GitHub, and letting the **Chirpy** theme handle the aesthetics. It feels weirdly similar to my Movable Type days—compiling static content for a safer, faster web—but with the modern power of TypeScript under the hood.

I hope this new chapter brings back the fun of blogging. No spam, no hacks, just code and words.

To the Gen Z devs: You were right about TypeScript, but don't think you invented static sites. We were doing this while you were still watching *Barney*.

Here is to new beginnings and old habits! 🥂
