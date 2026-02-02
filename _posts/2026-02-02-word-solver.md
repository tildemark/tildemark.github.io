---
title: "I Built a Wordle Solver Because I Hate Losing"
date: 2026-02-03 12:00:00 +0800
categories: [Projects, Web Development]
tags: [nextjs, tailwind, typescript, tools, humor]
image:
  path: https://cdn.sanchez.ph/blog/word-solver-preview.webp 
  alt: A screenshot of the Word Solver interface saving a Wordle streak
---

Critics often tell me, *"Mark, why do you only build useless apps? Why don't you cure cancer or fix the economy?"*

To those critics, I say: **You have clearly never lost a Wordle streak on the 6th guess because you couldn't decide between 'SHAVE' and 'SHARE'.**

That kind of pain changes a man.

So, in my quest to continue shipping "useless" software that actually solves my immediate, petty problems, I present to you: **The Word Solver**.

## The Problem

We have all been there. You have `S _ A R _`. The letters `E` and `T` are dead to you (gray). You are sweating. Is it *STARK*? *SHARD*? *SNARL*?

You could use your brain, which is noble but exhausting. Or, you could use a machine that filters 10,000 words in 4 milliseconds.

## The Solution

I built a lightweight web app that acts as a "Dictionary Sieve." It’s not AI. It’s not ChatGPT hallucinating a word that doesn't exist. It is pure, unadulterated logic powered by **Next.js**, **Tailwind CSS**, and the dark arts of **Regular Expressions**.

![The Word Solver Interface](https://cdn.sanchez.ph/blog/word-solver-demo.webp){: .shadow .rounded-10 .w-75 .normal }
_The interface: Simple, dark mode, and saves you from embarrassment._

👉 **Live Demo:** [solver.sanchez.ph](https://solver.sanchez.ph)  
👉 **Source Code:** [github.com/tildemark/word-solver](https://github.com/tildemark/word-solver)

## How It Works (The "Science")

The app essentially takes the dictionary and puts it through a three-stage TSA security checkpoint:

1.  **The Pattern (Green):** You tell it what you know. e.g., `S..RE`.
2.  **The Must-Haves (Yellow):** You tell it what letters are floating around somewhere.
3.  **The Ban List (Gray):** The letters that have betrayed you.

Under the hood, it’s running a client-side filter against the Google 10k English dictionary (the "No Swears" version, because my mom reads this blog).

### The "Tech Stack"
*(I use the term loosely)*

* **Next.js (App Router):** Because using React for a simple string filter is overkill, and I love overkill.
* **Tailwind CSS:** Because writing actual CSS in 2026 is illegal.
* **Regex:** The engine that powers the search. If you have two problems and you use Regex, you now have 300 problems. But in this case, it works.

## Is This Cheating?

I prefer the term *"Augmented Intelligence."*

If you use a calculator to do your taxes, are you cheating at math? No. You are being efficient. If you use my app to solve Wordle in 2 guesses, you are simply... optimizing your linguistic throughput.

(Okay, yeah, it's definitely cheating. But I won't tell if you don't.)

## Future Roadmap

I plan to add absolutely nothing else to this app because it does exactly one thing and does it well. However, if you want to fork it and add a "Scrabble Mode" or an "I'm feeling lucky" button that just prints "HELLO," feel free.

Check out the repo, give it a star, and stop losing your streaks.

[View on GitHub](https://github.com/tildemark/word-solver){: .btn .btn--primary }