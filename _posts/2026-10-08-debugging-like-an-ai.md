---
title: "Debugging Like an AI: Advanced Techniques for PHP and TypeScript"
date: 2026-10-08 08:00:00 +0800
categories: [Programming, Debugging]
tags: [git, php, typescript, debugging, docker, codeigniter]
description: "Learn how AI agents systematically track down hard-to-find bugs, and how you can adopt their techniques to speed up your own development workflow."
pin: false
math: true
mermaid: true
toc: true
comments: true
image:
  path: https://cdn.sanchez.ph/blog/debug-like-an-ai.webp
  alt: "AI Debugging Techniques for PHP and TypeScript"
---

If you’ve ever watched an AI coding assistant (like GitHub Copilot Workspace, Cursor, or Gemini) debug an application, you might have noticed something interesting. While experienced human developers often rely on intuition—guessing where a bug is based on past experience—AI agents don't have a "gut feeling." 

Instead, an AI relies on a highly systematic approach: **deterministic search, historical analysis, and isolated reproduction.** 

When your intuition fails and a bug is hiding deep within your CodeIgniter or TypeScript app, adopting this "AI mindset" will save you hours of frustration. Let's break down the exact commands and workflows you can use to debug like a machine.

---

## 1. Advanced Code Searching (The AI's Eyes)

When we look for a bug, we usually open a few files we *think* are responsible. An AI searches the entire codebase instantly to find exact references. 

To do this, leave your IDE's standard search behind and embrace the command line.

### Using `git grep`
`git grep` searches for a specific string across all tracked files in your Git repository. It is blazing fast because it ignores everything in your `.gitignore` (like your `vendor` or `node_modules` folders).

```bash
# Search for a function name and display the line numbers (-n)
git grep -n "update_user_status"

# Search for a specific string in TypeScript files only
git grep -n "throw new Error" -- "*.ts"
```

### Using `ripgrep` (`rg`)
If you want something even faster and more powerful, install [ripgrep](https://github.com/BurntSushi/ripgrep). It respects your `.gitignore` by default and is the gold standard for codebase archaeology.

```bash
# Case-insensitive (-i) search for a database error in your app directory
rg -i "database_error" app/

# Search for an interface in a TypeScript project, showing 2 lines of context (-C 2)
rg "interface User" src/ -C 2
```

---

## 2. Git Archaeology: Time-Travel Debugging

If you know a feature worked last week but is broken today, an AI doesn't guess what changed. It asks Git.

### The "Pickaxe" Search (`git log -S`)
This is arguably the most underused Git command. The "Pickaxe" searches through the *actual changes* in your commit history, not just the commit messages. 

If you want to know when a specific line of code was added or removed, use `-S`:

```bash
# Find exactly when this query was introduced or deleted
git log -S "db->where('status', 'active')" -p
```
*(The `-p` flag tells Git to show the actual code diff for those commits).*

### Line Tracking (`git log -L`)
If a specific function is acting up, you can track the complete history of just those lines in a file.

```bash
# Show the history of lines 45 through 70 in a PHP file
git log -L 45,70:app/Models/UserModel.php

# Show the history of a specific function in TypeScript
git log -L :calculateTotal:src/utils/math.ts
```

### The Ultimate Bug Hunter: `git bisect`
When you have a completely mysterious bug and no idea when it was introduced, `git bisect` uses a binary search to find the exact commit that broke your app.

1. Start the process: `git bisect start`
2. Tell Git the current state is broken: `git bisect bad`
3. Tell Git a past version that worked: `git bisect good v1.2.0`

Git will now check out a commit halfway between the good and bad states. You test your app (reload the page or run a test), and type either `git bisect good` or `git bisect bad`. Git repeats this, cutting the commits in half every time, until it hands you the exact commit that caused the bug!

![Git Bisect Binary Search Workflow](https://cdn.sanchez.ph/blog/git-bisect-flow.webp){: .shadow .rounded-10 .w-100 .normal }
_Binary search debugging with `git bisect`: halving commit ranges to pinpoint the exact breaking commit._

---

## 3. Runtime Execution (Getting Out of the Browser)

When testing a web app, refreshing the browser introduces a lot of noise: HTTP overhead, middleware, session cookies, and rendering. AI agents prefer to execute code directly via the Command Line Interface (CLI) or inside isolated Docker containers.

### Container Execution with Docker
If you use Docker, you can run commands directly inside your isolated environment without setting up SSH. The `-it` flags stand for "interactive terminal."

```bash
# Run a CodeIgniter migration status check inside the container
docker exec -it my_ci_app php spark migrate:status

# Run a TypeScript script inside a Node container
docker exec -it my_node_app npx ts-node scripts/test.ts
```

### Forcing Errors to Show
Sometimes frameworks suppress errors and show a generic "Whoops, something went wrong" page. You can force PHP to tell you the truth via the CLI:

```bash
php -d display_errors=1 -d error_reporting=E_ALL public/index.php
```

---

## 4. The "Isolated Reproduction" Method

This is the biggest secret of AI debugging. When an AI encounters a bug, it often creates a temporary file to test *just the broken function*. This is called a **Minimal Working Example (MWE)**.

Why do this? Because a bug in a Controller might actually be a bug in a Model, a Service, or a Database connection. By creating a temporary script, you eliminate the browser and the framework's routing entirely.

![Isolated Reproduction vs Full Web Stack](https://cdn.sanchez.ph/blog/isolated-reproduction-mwe.webp){: .shadow .rounded-10 .w-100 .normal }
_Bypassing the entire web stack: Running a lightweight CLI script isolates core domain logic and database calls directly in milliseconds._

### How to do this in PHP (CodeIgniter 4)
Create a file named `test_bug.php` in your project root:

```php
<?php
// test_bug.php
// 1. Load Composer dependencies
require 'vendor/autoload.php';

// 2. Bootstrap CodeIgniter manually without booting the HTTP routing
$app = require_once SYSTEMPATH . 'bootstrap.php';
$app->initialize();

// 3. Directly instantiate the problematic class and test it
$userModel = new \App\Models\UserModel();
$result = $userModel->find(123);

// 4. Dump the result to the terminal
var_dump($result);
```
Run it instantly from your terminal: `php test_bug.php` (or via `docker exec`).

### How to do this in TypeScript
Testing deeply nested TypeScript functions can be annoying if you have to spin up an entire Express or Next.js server. Instead, use a tool like `tsx` or `ts-node` to run a temporary script.

Create `test_bug.ts`:

```typescript
// test_bug.ts
import { calculateDiscount } from './src/services/billingService';
import { db } from './src/config/database';

async function runTest() {
  console.log("Starting test...");
  
  try {
    // Isolate the exact logic
    const user = await db.user.findUnique({ where: { id: 123 } });
    const finalPrice = calculateDiscount(100, user.tier);
    
    console.log("Result:", finalPrice);
  } catch (error) {
    console.error("Test failed:", error);
  } finally {
    // Clean up DB connection if necessary
    await db.$disconnect();
  }
}

runTest();
```
Run it instantly: `npx tsx test_bug.ts` (or `npx ts-node test_bug.ts`).

**The Benefits:**
1. **Speed:** It runs in milliseconds.
2. **Clarity:** If this script works, you *know* the bug is in your HTTP/Controller layer. If it fails, the bug is in your logic/database layer.
3. **Safety:** You can test edge cases safely without breaking your UI.

---

## Summary: The AI Debugging Workflow

Next time you are stuck on a difficult bug, stop randomly opening files and refreshing your browser. Follow this AI-inspired workflow:

![The 5-Step AI Debugging Loop](https://cdn.sanchez.ph/blog/ai-debugging-loop.webp){: .shadow .rounded-10 .w-100 .normal }
_The 5-step systematic debugging loop: Inspect logs, search deterministically, isolate reproduction, time-travel, and verify._

1. **Check the raw logs:** Use `tail -f writable/logs/log-*.log` (PHP) or check your Node console output.
2. **Search deterministically:** Use `git grep` or `rg` to find exactly where the error string originates.
3. **Isolate the code:** Write a 10-line `test_bug.php` or `test_bug.ts` script to run the function completely outside of the web browser.
4. **Time travel if necessary:** If the isolated script fails but you know it used to work, use `git log -S` or `git bisect` to find the exact commit that broke it.
5. **Fix, verify, and clean up:** Fix the bug, run your test script one last time, delete the temporary file, and commit your code!

By adopting these techniques, you'll stop relying on "guessing" and start debugging with the precision of a machine.