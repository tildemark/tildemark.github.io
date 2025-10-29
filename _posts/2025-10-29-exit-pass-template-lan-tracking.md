---
title: "LAN-Based Exit Pass Form with Auto Tracking Numbers in Word"
date: 2025-10-29 14:00:00 +0800
categories: [Office Automation, Programming]
tags: [Word, Template, Macro, TrackingNumber, InternalTools, Tutorial, VBA]
description: "How to set up a LAN-based Exit Pass form with automatic tracking numbers using Microsoft Word templates (.dotm) — no API required."
status: "Phase 1 – LAN-based. API integration planned."
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
**EP-NRA-2025-0045**

---

## File Naming Convention
Generated documents use this pattern:

`EP-<BRANCH>-<YEAR>-<NUMBER>.docx`

Example: `EP-MNL-2025-0045.docx`

This helps HR or Admin staff file forms in sequence.

---

## Template Features

* Auto tracking number from shared counter file
* Auto-filled current date
* Dropdown menu for selecting department
* Auto-filled approver name based on department
* Editable fields for employee name, purpose, and time in/out
* Optimized for half-letter printing (5.5 x 8.5 inches)

## VBA Macro Summary
Below is the core VBA logic (simplified):

```vbnet
Sub AutoNew()
    Dim fso As Object, txtFile As Object
    Dim lastNo As Long, newNo As Long
    Dim path As String, prefix As String

    prefix = "EP-MNL-" & Year(Date) & "-"
    path = "\\10.10.0.3\Shared Files\ExitPass\counter.txt"

    Set fso = CreateObject("Scripting.FileSystemObject")

    ' Read current counter
    If fso.FileExists(path) Then
        Set txtFile = fso.OpenTextFile(path, 1)
        lastNo = CLng(txtFile.ReadAll)
        txtFile.Close
    Else
        lastNo = 0
    End If

    ' Increment and save
    newNo = lastNo + 1
    Set txtFile = fso.CreateTextFile(path, True)
    txtFile.Write newNo
    txtFile.Close

    ' Apply to document
    ActiveDocument.Bookmarks("TrackingNo").Range.Text = prefix & Format(newNo, "0000")
    ActiveDocument.Bookmarks("DateField").Range.Text = Format(Date, "MMMM DD, YYYY")
End Sub
```

## Employee Setup Guide

1. Open the shared folder:
`\\10.10.0.3\Shared Files\ExitPass`
1. Copy ExitPass.dotm
1. Paste it into your Templates folder:
`%appdata%\Microsoft\Templates`
1. In Word, open File → Options → Save
   * Set the Default personal templates location to that same folder
1. Go to File → Options → Trust Center → Trusted Locations
   * Add the same Templates folder as a trusted location
1. Under Macro Settings, select “Disable all macros with notification.”
1. To use the form:
   * Go to File → New → Personal → ExitPass
   * Click Enable Content when prompted
   * The form will open with today’s date and a new tracking number
   * Fill in your details and save as .docx

## Admin Setup Notes
* Ensure `counter.txt` is writable for all users on the LAN
* Map the shared folder as a network drive (e.g. Z:\ExitPass)
* Only HR/Admin should modify the template or counter manually
* Template is formatted for half-letter printing

## Common Issues
| **Problem**                                      | **Possible Cause**                                       | **Solution**                                                                                                                          |
| ------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Word shows “Security Risk” warning**           | The template folder is not yet marked as trusted.        | Go to **File → Options → Trust Center → Trusted Locations** and add your personal Templates folder (`%appdata%\Microsoft\Templates`). |
| **Tracking number doesn’t change**               | The `counter.txt` file may be read-only or inaccessible. | Check that the file in `\\10.10.0.3\Shared Files\ExitPass` has read/write permissions for all users.                                  |
| **Word opens a blank document**                  | The user double-clicked the `.dotm` file directly.       | Use **File → New → Personal → ExitPass** instead of opening the template itself.                                                      |
| **Macro disabled**                               | Word’s macro settings prevent it from running.           | When opening the form, click **Enable Content** or adjust your macro settings in **Trust Center → Macro Settings**.                   |
| **Date not updating**                            | The auto date macro didn’t run.                          | Make sure the template file has macros enabled and was opened as a new document.                                                      |
| **Department drop-down doesn’t change approver** | Macro may not include department logic yet.              | Wait for the updated version (planned in v0.2 API update).                                                                            |

## Future Update: Centralized Tracking API (Planned)

We plan to replace the local text file with a small API that will generate and manage tracking numbers. This will allow all branches (even those outside the LAN) to share one numbering system.

Planned improvements:

* Number generation via a secure internal API
* Centralized database for tracking counters
* Prevention of duplicate numbers across branches
* Integration with user authentication (JWT) in ABCDE ERP

Possible Frameworks:
* CodeIgniter 4 (PHP-based, fits current ABCDE API)
* ExpressJR (Node.js, for microservice deployment)

**TODO:** Finalize which framework to use and publish API documentation later.

## Version History
| Version       | Date       | Notes                                          |
| ------------- | ---------- | ---------------------------------------------- |
| 0.1           | 2025-10-29 | Initial LAN-based Word Template implementation |
| 0.2 (Planned) | TBD        | Migrate tracking to centralized API            |

