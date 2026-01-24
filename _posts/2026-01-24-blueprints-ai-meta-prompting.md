---
title: "I Built a Robot to Talk to the Robot (So I Don't Have To)"
date: 2026-01-25 14:00:00 +0800
categories: [Coding, AI Experiments]
tags: [artificial intelligence, gemini, nextjs, oci, automation]
image:
  path: https://cdn.sanchez.ph/blog/blueprints-ai-preview.webp
  alt: "The Blueprints.ai interface running on OCI, showing a recursive prompt generation"
---

Let’s be honest: "Prompt Engineering" is just a fancy term for begging a computer to be nice to you. 

I was promised a cyberpunk future with flying cars. Instead, I spend my days typing, *"Act as a senior software engineer..."* into a text box. I got bored. And when a programmer gets bored, they don't just complain—they spend 40 hours automating a 5-minute task.

Introducing **Blueprints.ai**, a tool I built to nag the AI for me.

## The Problem: I Am Lazy

I have a vague idea: *"I need a Python script to scrape cat photos."*
The AI needs a novel: *"Write a Python script using Beautiful Soup, handle errors, respect robots.txt, and comment the code."*

I refuse to type that.

## The Solution: Recursive Laziness

I built an engine that takes my lazy, half-baked thoughts and expands them into the robust, context-heavy prompts that LLMs crave. 

But I didn't stop there. I realized that writing hard-coded templates was *also* too much work. So, I hooked the app up to **Gemini 2.5 Fast**.

Now, when I type "make a react app," my app sends that fragment to Gemini, asking it to write a better prompt *for* Gemini.

Yes. **I am using Gemini to explain to Gemini what I want from Gemini.** It is an Ouroboros of artificial intelligence. It is the Spider-Man pointing meme, but with neural networks. And it works famously.

## The Tech Stack: Next.js All The Things

I built the whole thing using **Next.js**, because obviously, if you aren't using Server Components to render a simple text box in 2026, are you even coding?

## Choose Your Weapon: CLI or UI?

I couldn't decide if I wanted to look like a 90s hacker or a modern web user, so I built two interfaces:
* **The CLI:** For when you want to look busy in a coffee shop.
* **The Web UI:** For when you want to drag-and-drop your vague ideas into reality.

## Try It Live

I deployed this monument to over-engineering on my Oracle Cloud (OCI) instance. You can try the Web UI right now:

🚀 **[https://blueprint.sanchez.ph](https://blueprint.sanchez.ph)**

(Please be gentle. It’s running on a cloud instance that is trying its best.)

## Get the Code

If you want to see the source code (or judge my `useEffect` dependencies), check out the repository:

🔗 **[https://github.com/tildemark/blueprints.ai](https://github.com/tildemark/blueprints.ai)**

Now, if you'll excuse me, I need to go ask the AI to generate a commit message for the code that asks the AI to generate code.