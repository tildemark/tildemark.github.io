---
title: "LAN-Based Exit Pass Form with Auto Tracking Numbers in Word"
date: 2025-10-29 14:00:00 +0800
categories: [Office Automation, Programming]
tags: [Word, Template, Macro, TrackingNumber, InternalTools, Tutorial, VBA]
description: "How to set up a LAN-based Exit Pass form with automatic tracking numbers using Microsoft Word templates (.dotm) — no API required."
---

## 🧩 Overview

When managing employees who need to leave the office temporarily, we wanted a way to record and track every **Exit Pass** request — with a **unique tracking number** — even before our ERP system is ready.

This post documents how we created a **LAN-based Exit Pass form** using a **Word macro-enabled template (.dotm)**.  
No internet or API required.  
All numbering is generated locally, synchronized across departments using a shared folder on the local network.

---

## 🧩 Objective

Create a **Word form template** that:
- Auto-generates a **unique tracking number** each time it’s opened  
- Fills in the **current date automatically**  
- Lets users select their **department** and **auto-fills the approver**  
- Works even without an internet connection  
- Uses only a shared network folder (e.g. `\\10.10.0.3\Shared Files\ExitPass`)

---

## 🧱 Shared Folder Setup

We placed the master template and counter file in a shared folder accessible to all employees:

Shared Folder Path:
`\\10.10.0.3\Shared Files\ExitPass`

Contents of the folder:

* ExitPass.dotm – the master Word template with macros
* counter.txt – a simple text file that stores the current tracking number
* README.txt – optional file for notes or change logs

Each employee will copy the template to their own Templates folder on their PC before using it.

---

## 🧱 Local Template Folder (per user)
Each user should paste the ExitPass.dotm file into:

`C:\Users\<username>\AppData\Roaming\Microsoft\Templates`

or simply open the location using the Windows Run command:

`%appdata%\Microsoft\Templates`

This is the default folder where Microsoft Word looks for personal templates.

## 🧱 Tracking Number Logic
When a user opens the template, a small macro runs automatically. It:

1. Opens the `counter.txt` file from the shared folder
1. Reads the last used number
1. Increments it by one
1. Inserts the new tracking number into the document
1. Saves the updated number back into `counter.txt`

The result is a unique tracking number for each new Exit Pass, even if multiple employees are using it at the same time.

Example tracking number:  
**EP-MNL-2025-0045**

---
