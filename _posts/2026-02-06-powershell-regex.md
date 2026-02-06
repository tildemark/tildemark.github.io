---
title: I Ignored PowerShell for 15 Years. Then an AI Wrote Me a Regex.
date: 2026-02-06 08:00:00 +0800
categories: [Development, DevOps]
tags: [powershell, regex, perl, automation, ai, productivity]
image:
  path: https://cdn.sanchez.ph/blog/powershell-regex-preview.webp
  alt: A split screen showing a retro green monochrome monitor with Perl code on the left and a modern sleek monitor with PowerShell code and an AI robot on the right.
---

I have a confession to make. It’s embarrassing, but here goes: Up until about six months ago, I thought PowerShell was just `cmd.exe` with a blue background and overly verbose commands.

You know how Gen Z looks at a `C:\>` prompt and wonders where the "Like" button is? They were never taught DOS. It just existed before their time. Well, I’m the millennial equivalent with PowerShell. It showed up one day in Windows 7, I ignored it, and I kept right on typing `dir /w` and hacking together batch files like it was 1999.

If I needed real power, I didn't look to Microsoft. I went to the granddaddy of duct-tape programming: **Perl**. I loved Perl. It was messy, powerful, and if you knew regular expressions, you were a wizard.

## Enter the AI

Fast forward to today. I’m building a Next.js application, and I’m using AI copilots to speed things up. I asked the AI to write a quick script to parse some local server logs and extract specific error codes.

I expected Python. Maybe some Node.js `fs` magic.

The AI spat out a file ending in `.ps1`.

My first instinct was, *"Ugh, a PowerShell script? Can you just convert this to bash so I can run it in WSL?"* But then my eyes caught something in the code. It looked familiar. *Suspiciously* familiar.

```powershell
if ($line -match "Error code: (?<digits>\d{3})") { ... }

```

Wait a minute. That’s a capture group. That’s `\d{3}`. Are those... regular expressions? Natively? Without shelling out to `sed` or installing a frantic mess of dependencies?

I suddenly felt like I’d discovered a secret room in a house I’d lived in for a decade.

## The Perl Connection

I dove in. It turns out, PowerShell isn't just "DOS 2.0." It’s what happens when the .NET framework and a Unix shell have a highly capable baby.

And yes, it supports regex. **Beautifully.**

If you are an old-school Perl monk like me, you will feel eerily at home in the blue terminal. PowerShell uses the .NET regex engine, which is largely compatible with Perl 5 syntax.

Check out the similarities.

### The "Does this string contain X?" check

**Perl:**

```perl
if ($string =~ m/server\d+/) { say "Found it"; }

```

**PowerShell:**

```powershell
if ($string -match "server\d+") { Write-Host "Found it" }

```

> **The Key Difference:** PowerShell is **case-insensitive by default** because it’s nice like that. If you want it to be grumpy and specific, you must use `-cmatch` (case-sensitive match).
> {: .prompt-info }

### The "Search and Replace"

**Perl:**

```perl
$string =~ s/foo/bar/g;

```

**PowerShell:**

```powershell
$newString = $string -replace "foo", "bar"

```

### The Magic of Captures

This is where I fell in love. Remember Perl's `$1`, `$2` capture variables? PowerShell does it cleaner. When you run a `-match`, it automatically populates an automatic variable called `$matches`.

```powershell
$logEntry = "Failed at [timestamp: 10:45am] - User: Bob"

# Look at these named capture groups!
if ($logEntry -match "timestamp: (?<time>.*)] - User: (?<who>.*)") {
     Write-Host "The user was: $($matches.who)"
     Write-Host "It happened at: $($matches.time)"
}

```

It just works. No modules to import. It’s baked right into the shell.

## Why This Matters for Modern Dev

*(Yes, even for Next.js)*

Okay, so it has regex. Big deal. Why should you, a modern web developer living in VS Code, care?

Because once you realize PowerShell is actually a powerful scripting language, you stop fighting your OS.

### 1. The Realization: Everything is an Object

In the old days (DOS/Bash), everything is text. If you run `ls -l`, you get a block of text you have to parse with `awk` or `cut` to get the file size.

In PowerShell, everything is an *object*. When you run `Get-ChildItem` (the equivalent of `ls`), it doesn't return text; it returns actual .NET `FileInfo` objects.

You don't grep for file sizes. You just ask for the property.

```powershell
# Find all files larger than 100MB in my dev folder
Get-ChildItem ./my-project -Recurse | Where-Object { $_.Length -gt 100MB }

```

### 2. The "Zombie Node Process" Killer

We’ve all been there. Next.js hangs. You try to restart `npm run dev`, and it screams that port 3000 is in use.

* **The Old Way:** Open Task Manager, hunt for Node. Or try to remember that `taskkill /F /IM node.exe` nonsense.
* **The PowerShell Way:** Treat processes like objects.

```powershell
# "Get every process named node, and stop it."
Get-Process node | Stop-Process

```

It’s so elegant I could cry.

### 3. The "Super Grep" (Select-String)

When debugging local logs, I used to rely on VS Code's search or install win-grep. PowerShell has `Select-String` (aliased to `sls`), and it supports regex naturally.

```powershell
# Scan all log files in the directory for 500 errors, show 2 lines of context
sls -Path logs\*.log -Pattern "Error 500" -Context 2

```

## Embrace the Blue

If you’re like me and you’ve been ignoring PowerShell because you thought it was just Microsoft trying to reinvent the wheel, give it another look. Especially now that AI copilots are happy to generate the syntax for you.

It’s powerful, it’s object-oriented, and it has the regex soul of Perl hidden beneath its corporate blue exterior.

Now, if you’ll excuse me, I have some batch files from 2008 to rewrite.
