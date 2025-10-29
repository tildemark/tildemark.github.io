---
title: "Creating a Shared Exit Pass Template with Auto-Generated Tracking Numbers in Microsoft Word"
date: 2025-10-29 14:00:00 +0800
categories: [VBA, Microsoft Word]
tags: [Word, Template, Macro, TrackingNumber, InternalTools, Tutorial, VBA, Automation]
description: "How to set up a LAN-based Exit Pass form with automatic tracking numbers using Microsoft Word templates (.dotm) — no API required."
status: "Phase 1 – LAN-based. API integration planned."
toc: true
---

## Introduction
This guide explains how to create a shared, automated Exit Pass form using Microsoft Word’s template system (`.dotm`). The form automatically generates unique tracking numbers for each request.

The system works over a shared folder accessible to all employees in a local network (LAN). Each employee uses their own copy of the template file, but all share a single counter file that keeps the tracking number sequence in sync.

This setup is ideal for small to medium-sized organizations without a centralized database or ERP system.

---

## How the System Works
1. A shared folder on the network stores:
   * The master Word template (`ExitPass.dotm`)
   * A shared counter file (`counter.txt`)
2. Each user copies the template into their own Microsoft Word Templates folder.
3. When a user creates a new Exit Pass, the template automatically:
   * Reads the current number from the shared counter file.
   * Increments it by one.
   * Displays it in the form.
   * Saves the new number back to the shared file.
4. The employee fills in their name, department, and purpose.
5. The form prints on half-letter-sized paper to save resources.

---

## Folder Setup

### Shared Network Folder
Ask your system administrator to create a shared folder accessible to everyone:

```
\\10.10.0.3\Shared Files\ExitPass
```

Inside that folder:

```
ExitPass.dotm    ← Master template file
counter.txt      ← Shared tracking number file
README.txt       ← Notes or change log
```

Make sure all users have read and write access to this folder.

---

## Copying the Template to User’s Local Template Folder

Each employee must copy the `ExitPass.dotm` file to their local Microsoft Templates folder.

1. Press `Win` + `R`
1. Type the following and press Enter:

```
%appdata%\Microsoft\Templates
```

3. Copy the file `ExitPass.dotm` into this folder.
1. You may also copy `counter.txt` here if you are testing individually, but normally it stays in the shared folder.

---

## Setting Up the Personal Templates Location in Word
If the Personal tab does not appear when creating a new document:

1. Open Microsoft Word.
1. Go to `File` → `Options` → `Save`.
1. Under `Save` documents, find the setting `Default personal templates location`.
1. Enter this path:
```
%appdata%\Microsoft\Templates
```
5. Click `OK` and restart Word.

You should now see a Personal tab in the New Document screen showing ExitPass.

---

## Enabling Macros

Since the file contains VBA (macros) to generate tracking numbers, users must enable macros.

1. Open `ExitPass.dotm`.
1. If you see a Security Warning: Macros have been disabled, click `Enable Content`.
1. If macros remain blocked:
   * Go to `File` → `Options` → `Trust Center` → `Trust Center Settings` → `Macro Settings`.
   * Select `Disable all macros with notification`.
   * Avoid `“Enable all macros”` for security reasons.

---

## Adding a Trusted Location

Some users may get errors like:

```quote
“Remote location is not allowed by your current security settings.”
```

To fix this safely:
1. Map the shared folder to a network drive (e.g. **Z:**).
```
Z: → \\10.10.0.3\Shared Files\ExitPass
```
2. In Word, open `File` → `Options` → `Trust Center` → `Trust Center Settings` → `Trusted Locations`.
1. Click `Add new location`.
1. Browse to the mapped drive `Z:\ExitPass`.
1. Check `“Subfolders of this location are also trusted.”`
1. Click `OK`.

If Word still blocks network paths, Microsoft 365 users must copy the `.dotm` file locally as described earlier, since cloud security often restricts direct execution from network drives.

---

## Understanding the `.dotm` File
### DOTM vs DOCX
| File Type | Description                                |
| --------- | ------------------------------------------ |
| `.docx`   | Standard Word document, no macros allowed. |
| `.dotx`   | Word template without macros.              |
| `.dotm`   | Word template that supports macros (VBA).  |

---

## Opening and Using the Template
### Opening Correctly

Do **not** double-click `ExitPass.dotm` to edit it.
Instead, open Word and choose:

1. `File` → `New` → `Personal`
1. Select `ExitPass`
1. A new document opens, usually named `Document1.docx`
1. This document has a unique tracking number and can be printed or saved.

---

## VBA Macro Logic (Simplified)
The macro inside the template does the following:
1. When a new document is created:
   * Reads the last number from `counter.txt` in the shared folder.
   * Increments it by 1.
   * Displays the number in the form (e.g., EP-0001).
   * Writes the new number back to `counter.txt`.
2. Auto-fills:
   * Current date
   * Optional department drop-down
   * Approver based on department
3. Keeps some fields editable:
   * Employee name
   * Purpose
   * Remarks

---

## Pre-Filled and Editable Fields
* Date → Automatically fills with today’s date.
* Department → Drop-down list to choose from (e.g., Accounting, Operations, HR).
* Approver → Automatically updates based on department.
* Name, Purpose, and Signature → Editable text fields.

---

## Paper Size Setup (Half-Letter)

To save paper:
1. Go to `Layout` → `Size` → `More Paper Sizes`.
1. Set the dimensions:
   * Width: `8.5 inches`
   * Height: `5.5 inches`
1. Save this layout inside the template.

Word’s metric equivalent (for users in cm):
* Width: `21.59 cm`
* Height: `13.97 cm`

---

## Programming the Tracking Number Counter (VBA Steps)
1. Press `Alt` + `F11` to open the VBA editor.
1. Under `“ThisDocument”`, paste your VBA code.
1. Use variables for:
   * SharedPath = `"Z:\ExitPass\counter.txt"`
   * TemplatePath = `Environ("AppData") & "\Microsoft\Templates\"`
1. Add error handling if the counter file is not found.
1. Test with multiple users to ensure it increments sequentially.

---

## Full VBA Source Code for the Exit Pass Template
This VBA script reads and updates the shared `counter.txt` file to generate a unique tracking number every time someone creates a new Exit Pass.
It also automatically inserts the current date, and optionally fills department/approver fields.

### How to Add the Code
1. Open your `ExitPass.dotm` file.
2. Press `Alt` + `F11` to open the VBA Editor.
3. In the Project Explorer, find your template:
  
```
Project (ExitPass.dotm)
└─ Microsoft Word Objects
     └─ ThisDocument
```
4. Double-click `ThisDocument`.
5. Paste the following VBA code into the editor.
6. Save and close the editor.

### VBA Source Code
```vb
' ===============================================================
' ExitPass.dotm - Auto Tracking Number Macro
' Version 1.0 - October 2025
' ===============================================================
' This macro automatically generates a unique tracking number for
' each new Exit Pass document using a shared counter file.
' It also fills in the current date and handles department logic.
' ===============================================================

Private Sub Document_New()
    ' Declare variables
    Dim counterFile As String
    Dim counterValue As Long
    Dim fileNum As Integer
    Dim trackingNo As String
    Dim sharedFolder As String
    Dim currentDate As String
    Dim dept As String
    Dim approver As String

    ' Path to shared folder (adjust if mapped differently)
    sharedFolder = "Z:\ExitPass\"  ' Example: mapped from \\10.10.0.3\Shared Files\ExitPass
    counterFile = sharedFolder & "counter.txt"

    ' Error handling
    On Error GoTo ErrorHandler

    ' Read counter value
    fileNum = FreeFile
    Open counterFile For Input As #fileNum
    Input #fileNum, counterValue
    Close #fileNum

    ' Increment the counter
    counterValue = counterValue + 1

    ' Save new counter value
    fileNum = FreeFile
    Open counterFile For Output As #fileNum
    Print #fileNum, counterValue
    Close #fileNum

    ' Format the tracking number (e.g., EP-0001)
    trackingNo = "EP-" & Format(counterValue, "0000")

    ' Insert values into bookmarks or form fields
    currentDate = Format(Date, "mmmm dd, yyyy")

    ' Example: assumes bookmarks named TrackingNo and DateFiled
    On Error Resume Next
    ActiveDocument.Bookmarks("TrackingNo").Range.Text = trackingNo
    ActiveDocument.Bookmarks("DateFiled").Range.Text = currentDate

    ' Optional: department drop-down logic
    ' Requires a content control named Department
    ' and a bookmark named Approver
    dept = ""
    approver = ""

    If ActiveDocument.ContentControls.Count > 0 Then
        dept = ActiveDocument.ContentControls(1).Range.Text
        Select Case LCase(dept)
            Case "accounting": approver = "Maria Santos"
            Case "operations": approver = "Juan Dela Cruz"
            Case "hr": approver = "Liza Reyes"
            Case "it": approver = "Mark Villanueva"
            Case Else: approver = "Department Head"
        End Select
        ActiveDocument.Bookmarks("Approver").Range.Text = approver
    End If

    ' Confirmation (optional, for testing)
    MsgBox "Tracking Number Generated: " & trackingNo, vbInformation, "Exit Pass"

    Exit Sub

ErrorHandler:
    MsgBox "Error generating tracking number. Please check counter.txt or permissions.", vbCritical, "Exit Pass Error"
End Sub
```
### File and Bookmark Setup
Before this macro works properly, make sure your Word form includes:
| Element         | Description                   | Example Name |
| --------------- | ----------------------------- | ------------ |
| Bookmark        | Where tracking number appears | `TrackingNo` |
| Bookmark        | Where current date appears    | `DateFiled`  |
| Bookmark        | Department approver           | `Approver`   |
| Content Control | Drop-down for department      | `Department` |

You can add bookmarks by:

1. Highlighting the desired text location.
1. Go to `Insert` → `Bookmark` → Name it (e.g., “TrackingNo”).
1. Click `Add`.

### How the Counter Works
1. The macro opens `counter.txt` in the shared folder.
1. Reads the last number inside (e.g., 25).
1. Adds 1 → new number = 26.
1. Writes it back to `counter.txt`.
1. Displays “EP-2025-0026” on the form.
1. The `counter.txt` file should initially contain:

```
0
```

### Testing the Template
To test:
1. Ensure counter.txt exists in the shared folder and is writable.
2. Open Word → `File` → `New` → `Personal` → `ExitPass`
3. Confirm a new tracking number appears each time.
4. Print or save the form (optional as `.docx`).

### Optional Improvements
* Add a prefix per department (e.g., HR-0001, IT-0001).
* Add time-based resets (e.g., restart numbering monthly).
* Add a log file to record who generated each number.
* Add error-checking for concurrent file access.

---

## Common Issues and Fixes

| Issue                         | Cause                                    | Solution                                               |
| ----------------------------- | ---------------------------------------- | ------------------------------------------------------ |
| Word shows “Security Risk”    | File opened directly from a network      | Copy to local Templates folder or add trusted location |
| Counter file not updating     | No write permission to shared folder     | Grant modify rights to all employees                   |
| “Remote location not allowed” | Network path blocked by Trust Center     | Use mapped drive (Z:) or local copy                    |
| New document does not appear  | User opened the `.dotm` directly         | Must create via File → New → Personal                  |
| Personal tab missing          | No default templates location set        | Add `%appdata%\Microsoft\Templates` in settings        |
| Tracking number resets        | Using a different `counter.txt` per user | Ensure everyone uses the same shared counter file      |


---

## Version History
### Version 1.0 – October 2025
  * Initial release with auto tracking number.
  * Shared counter file support.
  * Half-letter page setup.
  * Department and approver auto selection.
  * Added trusted locations and macro security setup instructions.
### Planned for Version 1.1
  * Integration with ERP (CodeIgniter 4 or ExpressJR API).
  * Online number generator for remote sites.
  * Automatic saving to central database.
  * Digital signature support.

---

## Future Enhancements
* Web API Tracking Counter: To synchronize tracking numbers across sites.
* Online Submission Form: For remote employees outside the LAN.
* Version Control Log: Save generated passes in a database for auditing.

---

## Summary

This setup provides an efficient, low-cost, and fully offline solution for managing Exit Pass requests.
By combining Word’s template features with a simple VBA counter and shared network folder, organizations can maintain consistent tracking numbers and uniform document layout even without a database or ERP system.

---
