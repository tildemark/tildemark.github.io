---
title: Introducing CCard Studio v1.2.0 — Open-Source Calling Card Designer and A4 Print Suite
date: 2026-08-20 12:00:00 +0000
categories: [projects, tools]
tags: [nextjs, fabricjs, tauri, rust, react, typescript, pdf, printing]
image:
  path: https://cdn.sanchez.ph/blog/ccard-studio.webp
  alt: CCard Studio Application Layout
---

I used to design calling cards in Photoshop, export the assets, and drop them into InDesign just to layout 10 cards per page for printing. 

At first, doing this for one or two people was fine. But as more team members requested their own calling cards, updating text frames, re-exporting layouts, and fixing back-to-back print alignments for dozens of employees quickly turned into a maintenance nightmare.

That pain point led me to build **CCard Studio (v1.2.0)** — a dedicated web and desktop app designed specifically for designing calling cards and outputting 10-up A4 print sheets automatically.

Here is how CCard Studio works and how the technical pipeline handles design, data mapping, and printing.

## What CCard Studio Does

CCard Studio simplifies both the visual design phase and the batch print phase:

- **Design Once, Print 10-Up:** Layout a single 90mm × 54mm card. The print engine automatically duplicates the design into a 2×5 grid across A4 pages (10 cards per sheet).
- **Back-to-Back Layout Support:** Design Front and Back sides independently on dual canvases, with an interactive preview to check visual alignment.
- **Built-In Crop & Cutting Markers:** Printable PDFs automatically generate corner crop marks and center trim guides so you know exactly where to cut with a paper cutter.
- **Built-In Vector Library:** Includes categorized vector icons (contact info, business details, action badges), line nodes, decorative background waves, dot grids, stripes, curves, and 10 geometric shapes (Circle, Oval, Square, Rectangle, Rounded Rect, Triangle, Hexagon, Octagon, Rhombus, Star).
- **HRIS Data Auto-Mapping:** Import employee lists via JSON or API bearer tokens. Map text layers to fields like `fullName`, `position`, `email`, and `mobile` to batch-generate cards without touching layout files.
- **Scannable QR Generator:** Real-time generation of vCard 3.0, website URLs, phone call (`tel:`), or email (`mailto:`) QR codes directly on the canvas.
- **Reliable Native System Printing:** Uses the OS print dialog for reliable driver handling, duplex orientation control, and hardware compatibility.

---

## Technical Architecture

CCard Studio is built with **Next.js 16 (App Router)**, **Fabric.js v6**, **Tauri v2 (Rust)**, and **`@react-pdf/renderer`**.

```
+-----------------------------------------------------------------------+
|                           CCard Studio v1.2.0                         |
+-----------------------------------+-----------------------------------+
|            Canvas UI              |          Print & Export           |
|  - Fabric.js v6 Canvas            |  - @react-pdf/renderer Engine     |
|  - Dual Front/Back Workspaces     |  - 10-Up A4 Grid Layout           |
|  - Vector Library & Tagged Roles  |  - Duplex Column Mirroring        |
|  - Real-Time QR Code Generator    |  - Corner & Center Cut Markers    |
+-----------------------------------+-----------------------------------+
|                           Tauri v2 Wrapper                            |
|                 Native File I/O & System Print Dialog                 |
+-----------------------------------------------------------------------+
```

### 1. Dual Canvas & Role-Based Variable Tagging

The editor runs two separate Fabric.js v6 canvas instances for the front and back layouts. Instead of duplicating elements manually for each staff member, plain text objects are tagged with a target field role:

```tsx
export interface FieldRoleConfig {
  role: 'fullName' | 'position' | 'department' | 'mobile' | 'email' | 'company' | 'custom';
  fallbackText: string;
}

export function applyFieldRoleToCanvasText(textObject: fabric.IText, roleConfig: FieldRoleConfig) {
  textObject.set({
    fieldRole: roleConfig.role,
    dataVariable: `{{${roleConfig.role}}}`,
  });
  textObject.canvas?.requestRenderAll();
}
```

During export, the rendering engine substitutes `{{fullName}}` with real employee records while maintaining the exact font family, size, line-height, text alignment, and color defined in the visual editor.

### 2. A4 Grid Population & Duplex Mirroring Math

A standard A4 page (210mm × 297mm) fits 10 standard calling cards (90mm × 54mm) arranged in a 2-column by 5-row layout with margin spacing.

The main challenge with double-sided printing on standard desktop or office printers is page alignment. When you flip an A4 sheet over along the long edge, Column 1 on the front becomes Column 2 on the back. If you don't mirror the columns, the back design of Card 1 prints behind Card 2.

The `@react-pdf/renderer` pipeline handles this by transposing column coordinates on every reverse page:

```tsx
export function calculateDuplexSlot(
  cardIndex: number, 
  side: 'front' | 'back',
  cols = 2, 
  rows = 5
) {
  const pageIndex = Math.floor(cardIndex / (cols * rows));
  const slotOnPage = cardIndex % (cols * rows);
  const row = Math.floor(slotOnPage / cols);
  const col = slotOnPage % cols;

  // Front side uses standard left-to-right grid: col 0, col 1
  // Back side mirrors horizontally: col 1, col 0
  const effectiveCol = side === 'front' ? col : (cols - 1 - col);

  return {
    pageIndex,
    x: MARGIN_LEFT + effectiveCol * (CARD_WIDTH + GAP_X),
    y: MARGIN_TOP + row * (CARD_HEIGHT + GAP_Y),
  };
}
```

### 3. Cutting Guides & Trim Markers

To make cutting 10 cards out of an A4 sheet straightforward, the PDF engine draws hairline crop markers outside each card's border. 

Center tick marks line up between rows and columns, allowing you to align a ruler or paper cutter across the full width of the sheet instead of measuring individual cards.

### 4. Dynamic vCard 3.0 QR Generator

QR codes placed on the canvas render dynamically using `qrcode`. For contact cards, the app formats the current record into vCard 3.0 specification:

```typescript
export function generateVCardString(employee: EmployeeRecord): string {
  return [
    'BEGIN:VCARD',
    'VERSION:3.0',
    `N:${employee.lastName};${employee.firstName};;;`,
    `FN:${employee.firstName} ${employee.lastName}`,
    `ORG:${employee.company || ''};${employee.department || ''}`,
    `TITLE:${employee.position}`,
    `TEL;TYPE=CELL:${employee.mobile}`,
    `EMAIL:${employee.email}`,
    'END:VCARD'
  ].join('\n');
}
```

### 5. Transitioning from Silent CLI Printing to Native System Printing

In v1.1.0, I experimented with background silent printing by executing SumatraPDF in CLI mode. While it sounded great in theory, it introduced annoying edge cases in practice: missing font substitutions, printer driver mismatches, and occasional silent execution hangs across different hardware setups.

In **v1.2.0**, I replaced CLI silent printing with the native OS print workflow. Showing the standard Windows print dialog adds one or two extra clicks, but it completely eliminates driver issues and gives users full control over paper trays, color profiles, and duplex settings.

---

## Technical Summary

| Component | Tech Used | Function |
| :--- | :--- | :--- |
| **UI & Shell** | Next.js 16 (App Router), Tailwind CSS v4 | Application layout, panels, modal routes |
| **Canvas Engine** | Fabric.js v6 | Visual object editor, vector shapes, layers |
| **Desktop Wrapper** | Tauri v2 (Rust) | Native file picker, local template storage, print IPC |
| **PDF Renderer** | `@react-pdf/renderer` | Client-side 10-up A4 layout, crop marks, duplex math |
| **State Management** | Zustand | Canvas undo/redo state, HRIS dataset binding |

---

## Source Code & Installation

CCard Studio is open source under the MIT License.

- **GitHub Repository:** [tildemark/ccard-studio](https://github.com/tildemark/ccard-studio)
- **Latest Release:** [CCard Studio v1.2.0 Release](https://github.com/tildemark/ccard-studio/releases/tag/v1.2.0)