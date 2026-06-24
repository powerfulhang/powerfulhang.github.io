---
title: "Fixing Calibre LVS OSSHNL-116: A CDL View List Mismatch Guide"
date: 2026-06-24T10:30:00+08:00
draft: false
author: "SiliconBlog"
tags: ["Calibre", "LVS", "OSSHNL-116", "auCdl", "Virtuoso", "PDK", "EDA"]
categories: ["DRC/LVS"]
summary: "How to fix Calibre LVS OSSHNL-116 source netlist export failures by inspecting PDK cell views and aligning the CDL View List, Stop List, and export template."
ShowToc: true
TocOpen: true
---

> **TL;DR** — Calibre LVS fails with OSSHNL-116 because the CDL netlister cannot find a matching view (typically `cdl` or `auCdl`) on PDK primitive cells. The fix is to align the View List / Stop List in the **Calibre Netlist Export Setup** with the views your PDK actually provides, and access that setup through **Virtuoso → Calibre → Setup → Netlist Export...**, not from within Calibre Interactive.

---

## 1. The Problem

When running Calibre LVS from the Virtuoso layout editor, the source (schematic) netlist export fails immediately, producing a dialog:

```
Exporting source failed
```

The Virtuoso CIW log reveals the root cause:

```
ERROR (OSSHNL-116): Unable to descend into any of the views defined
in the view list, 'cdl schematic', for the instance 'M0' in cell
'test'. Add one of these views to the cell 'dgpfet_33' in the
library '55FSI_3p3', or modify the view list so that it contains
an existing view.

ERROR (OSSHNL-514): Netlist generation failed because of the errors
reported above.
```

## 2. Root Cause Analysis

The Cadence CDL netlister traverses the design hierarchy top-down. For each instance it encounters, it looks for a cell view whose name appears in the **View List** (checked in order). When it finds one, it either descends into it (if the view is not in the **Stop List**) or treats the cell as a leaf and outputs a `.SUBCKT` call.

OSSHNL-116 fires when **none** of the views in the View List exist on a particular cell. In the example above:

- View List = `cdl schematic`
- The PDK cell `dgpfet_33` in library `55FSI_3p3` has neither a `cdl` view nor a `schematic` view that the netlister can use.
- Many foundry PDKs provide an **`auCdl`** view instead of `cdl` for their primitive device cells.

The mismatch → netlister cannot descend → netlist generation aborts → Calibre LVS cannot proceed.

### Why `CDS_Netlisting_Mode=Analog` Does Not Help

A commonly suggested fix online is:

```bash
export CDS_Netlisting_Mode=Analog
```

This environment variable controls how CDF parameters are resolved during netlisting (analog vs. digital mode), but it does **not** change the View List or Stop List. If the fundamental view mismatch exists, setting this variable has no effect.

## 3. Solution: Configure View List and Stop List Correctly

### Step 1 — Identify Available Views on PDK Cells

In the Virtuoso CIW, run:

```scheme
ddGetObj("55FSI_3p3" "dgpfet_33")~>views~>name
```

This returns a list of all views on the cell, e.g., `("auCdl" "symbol" "spectre" "layout")`. Note which CDL-related view is present — most commonly `auCdl`.

### Step 2 — Open the Calibre Netlist Export Setup

This is **not** inside the Calibre Interactive window. Access it from Virtuoso:

**Virtuoso menu bar → Calibre → Setup → Netlist Export...**

This opens the **Calibre Netlist Export Setup** dialog, which controls how Virtuoso exports the schematic netlist for Calibre.

### Step 3 — Update View List and Stop List

In the Calibre Netlist Export Setup dialog:

| Field | Before (broken) | After (fixed) |
|---|---|---|
| **View List** | `cdl schematic` | `auCdl cdl schematic` |
| **Stop List** | `cdl` | `auCdl cdl` |

By adding `auCdl` to both lists (and placing it first), the netlister will now find the `auCdl` view on PDK primitives, treat them as leaf cells, and successfully generate the netlist.

### Step 4 — Handle Greyed-Out Fields (Template Lock)

If the View List and Stop List fields are greyed out and non-editable, they are locked by the **Template File** (e.g., `cdl_subckt`). Two workarounds:

**Option A — Edit the template file directly:**

```bash
# Find the template
find $MGC_HOME -name "cdl_subckt" -type f 2>/dev/null

# Copy and edit
cp /path/to/cdl_subckt ~/my_cdl_subckt
```

Edit `viewList` and `stopList` in the copied file, then point the Template File field to your modified version and click **Load**.

**Option B — Clear the template:**

Clear the Template File field to unlock all settings, make your changes, then click **Save** to create a custom template for future use.

### Step 5 — Re-run LVS

Return to the Calibre Interactive window and click **Run LVS**. The netlist export should now succeed.

## 4. Workaround: Manual CDL Export

If you cannot modify the Netlist Export Setup (e.g., PDK restrictions), you can bypass the automatic export:

1. In Virtuoso: **File → Export → CDL** — manually export the schematic netlist to a file.
2. In Calibre Interactive → Inputs → Source Path:
   - **Uncheck** "Export from source viewer"
   - Set **SPICE File** to the path of your manually exported CDL netlist
3. Run LVS.

This works but requires re-exporting every time the schematic changes.

## 5. Summary

| Symptom | Cause | Fix |
|---|---|---|
| OSSHNL-116 on PDK cells | View List missing `auCdl` | Add `auCdl` to View List and Stop List |
| Fields greyed out | Template file locking | Edit template or clear it |
| Setup dialog not found | Looking inside Calibre Interactive | Access from Virtuoso: Calibre → Setup → Netlist Export... |
| `CDS_Netlisting_Mode` ineffective | Wrong diagnosis | This env var does not affect view list |

---

**Environment:** Cadence Virtuoso ICADVM20.1 / IC23.1, Siemens Calibre nmLVS v2024.x, RHEL 7.9 / Rocky Linux 8.10

**Tags:** `Calibre` `LVS` `OSSHNL-116` `auCdl` `Virtuoso` `PDK` `EDA`
