# Brief: Port Order → Files → Elements Architecture to Corpus Installation Calculator

> Self-contained brief for a new chat session. Contains everything needed to implement
> the architecture changes — read both reference files before writing any code.

---

## What you're building

The corpus installation calculator at `/Users/ruben/Documents/Projects/corpus-currency/index.html`
needs to be restructured from a single-file-at-a-time calculator to a multi-file order-based
calculator. The architecture mirrors what was recently implemented in the design-hours calculator.

## Reference implementation

The design-hours calculator at `/Users/ruben/Documents/Projects/design-hours/index.html` (shipped
2026-06-09) implements this architecture. **Read it fully before writing any code** — it's your
primary reference for patterns, CSS, and JS helpers.

Also read:
- `/Users/ruben/Documents/Projects/corpus-currency/V2-ROADMAP.md` — V2 decisions and scope
- `/Users/ruben/Documents/Projects/corpus-currency/index.html` — current calculator (your starting point)

---

## Architecture: Order → Files → Elements

### Three levels

**Order** (top level)
- One text input: order name (e.g. "Brasserie Nord")
- Stored as `JOB.orderName`

**Files** (second level)
- Each file = one file from the Cutr platform
- Has: `id`, `name`, `isDuplicateOf`, and file-level settings (installation type, floor)
- Multiple files per order

**Elements** (third level)
- Each element = one product type card within a file (Cabinets, Benches, etc.)
- The existing V2 "cards" — keep all calculation logic intact

### State shape

```javascript
const JOB = {
  orderName: '',
  files: [
    {
      id: 'f-abc123',       // nanoid or Date.now()
      name: 'Bestand 01',
      installType: 'built-in',  // file-level (was global in V2)
      floor: 0,                  // file-level (was global in V2)
      isDuplicateOf: null,       // null OR id of another file — see below
      elements: [ /* V2 cards */ ]
    }
  ]
}
```

Storage keys:
- Admin config: keep `STORAGE_KEY = 'cutr-corpus-v2-config'` — **do not change**
- Job state: new `JOB_KEY = 'cutr-corpus-v2-job'`

Migration: if old flat state is found on load, wrap it:
```javascript
// Old shape had: { installType, floor, cards }
// Wrap as:
JOB = { orderName: '', files: [{ id: 'f1', name: 'File 1', ...oldFileSettings, isDuplicateOf: null, elements: oldCards }] }
```

---

## The duplicate file feature — READ CAREFULLY

> **This is NOT the same as the design-hours duplicate logic.**

In design-hours, `isDuplicateOf` means **billing exclusion** (the file contributes 0h to the
order total). **That logic does not apply here.**

In corpus installation, `isDuplicateOf` means **copy configuration**:

When a user selects "Duplicate of: File X" from the dropdown on a file:
1. **Copy all element cards from File X into the current file**, replacing whatever was there.
2. The user can then edit those elements independently — the link is severed after the copy.
3. The file's corpus numbers and install hours are **fully counted** in the order total. No exclusion.
4. This is a pure UX time-saver — not a billing mechanism.

**Why this exists:** A project has 5 floors. Each floor is a separate file in the Cutr platform
(different file numbers), but every floor has the same corpus installation configuration —
same cabinets, same panels, same sizes. The user configures File 1 fully, then for Files 2–5
selects "Duplicate of: File 1" to instantly copy that configuration. Each floor is still
separately counted in the installation quote.

### UI for the duplicate feature

- Per-file dropdown: `Duplicate of: — / File 1 / File 2 / ...` (excludes self; only lists other files)
- On change: immediately copy all elements from the referenced file into this file
- Show a subtle dismissable note in the file header: `"Configuration copied from [File X] — edit freely"`
- No `overrideHours` concept. Just copy and let the user edit.

---

## Features to add (in priority order)

### 1. Order level
- Text input at the top: order name (pre-fill from localStorage if returning user)
- Replace the current single global calculation header

### 2. File cards
- Each file gets a collapsible card with:
  - **Header**: file name input, installation type selector, floor input, "Duplicate of" dropdown,
    add/remove file buttons
  - **Body**: the existing V2 product type card stack (elements)
  - **Footer/result**: per-file corpus total, install time, per-card breakdown

### 3. Bulk file import
User can add multiple files at once by pasting:

```
[paste area in the add-files section]
Bestand 01 - Klant naam
Bestand 02 - Klant naam
Bestand 03 - Klant naam
```

- One file name per line, or Cutr partlist-style `Bestand XX` prefixes
- Clicking "Add files" appends one new empty file per line (existing files stay)
- Parsing: strip leading/trailing whitespace; skip blank lines; take the whole line as the file name

### 4. Multi-file state management
- `saveJob()` / `loadJob()` — full tree persisted to `JOB_KEY`
- `computeJob()` — returns `{ files: [{ fileId, corpus, installMinutes, perCardBreakdown }], orderTotals }`
- Order total = sum of all files, no exclusions

### 5. Quote line items (copyable, platform format)

Same CopyField pattern as design-hours (amber flash, ⎘/✓ icons, no border). Platform format:

- **Parent header** (bold, non-copyable): `DSP - Installatie`
- **Child rows** (one per file, copyable): `Installatie - Bestand: [filename]`
- **Columns**: Line item name | Qty (corpus) | Rate (€/corpus) | File reference
- Use CSS grid: `grid-template-columns: minmax(0,2fr) 88px 80px minmax(0,1.5fr)`

> Ask the user to confirm the exact DSP line item names from the Cutr platform before hardcoding.

### 6. Backtrace
Copy-pastable text block (single button). Extend the existing V2 backtrace to include:
- Order name
- Per-file section with: file name, install type, floor, per-card raw contributions, corpus, install time
- Order totals
- Instruction: "Paste as a comment on the order in the Cutr platform"

### 7. Onboarding overlay (first load)
```javascript
const ONBOARDING_KEY = 'cutr_corpus_onboarding_seen';
// Show on first load if key not set
// Dismiss sets the key
// Header button re-opens it
```
Content: explain the three levels, what a corpus is, how the duplicate-copy feature works.

### 8. Changelog overlay
Same pattern as onboarding. Accessible via a "What's new" button in the header.
Content: list the V2 → V3 changes (Order level, multi-file, bulk import, duplicate copy).

---

## CSS & JS patterns to copy from design-hours

Read `/Users/ruben/Documents/Projects/design-hours/index.html` for the full implementation.
Key classes to replicate:

```css
/* File blocks */
.file-block { border: 1px solid var(--cutr-n-400); margin-bottom: 32px; background: white;
  box-shadow: 0 2px 10px rgba(0,0,0,0.07); border-radius: 3px; overflow: hidden; }
.file-header { display: flex; align-items: center; gap: 10px; padding: 12px 14px;
  background: var(--cutr-n-200); border-bottom: 2px solid var(--cutr-n-400); }

/* CopyField */
.copy-field { display: inline-flex; flex-direction: column; padding: 5px 8px; border: none;
  background: none; border-radius: 3px; cursor: pointer; text-align: left; transition: background 0.1s; }
.copy-field:hover { background: var(--cutr-amber-100); }
.copy-field.flash { background: var(--cutr-amber-100); }
.cf-label { font-size: 9px; color: var(--cutr-n-400); line-height: 1; margin-bottom: 3px; white-space: nowrap; }
.cf-value { display: flex; align-items: center; gap: 3px; font-size: 12.5px; line-height: 1.3; color: var(--cutr-text); }
.copy-field.flash .cf-value { color: var(--cutr-brand); }
.cf-text { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.cf-icon-hover { font-size: 9px; color: var(--cutr-n-300); opacity: 0; transition: opacity 0.1s; flex-shrink: 0; }
.copy-field:hover .cf-icon-hover { opacity: 1; }
.cf-icon-flash { font-size: 9px; color: var(--cutr-brand); display: none; flex-shrink: 0; }
.copy-field.flash .cf-icon-flash { display: inline; }
.copy-field.flash .cf-icon-hover { display: none; }

/* Quote line items */
.ql-parent { display: flex; align-items: center; gap: 8px; padding: 8px 4px 10px;
  font-size: 13px; font-weight: 700; color: var(--cutr-n-900); border-bottom: 2px solid var(--cutr-n-900); }
.ql-thead-row, .ql-row { display: grid;
  grid-template-columns: minmax(0,2fr) 88px 80px minmax(0,1.5fr); align-items: start; }
.ql-thead-row { border-bottom: 1px solid var(--cutr-n-300); }
.ql-th { font-size: 10px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.08em;
  color: var(--cutr-text-secondary); padding: 8px 8px 6px; white-space: nowrap; }
.ql-row { border-bottom: 1px solid var(--cutr-n-100); }
.ql-row:last-child { border-bottom: none; }
```

JS helpers to copy verbatim:
```javascript
function copyField(el, value) {
  const text = String(value);
  const doFlash = () => { el.classList.add('flash'); setTimeout(() => el.classList.remove('flash'), 1000); };
  if (navigator.clipboard && navigator.clipboard.writeText) {
    navigator.clipboard.writeText(text).then(doFlash).catch(() => { _copyFallback(text); doFlash(); });
  } else { _copyFallback(text); doFlash(); }
}
function _copyFallback(text) {
  const ta = document.createElement('textarea'); ta.value = text;
  ta.style.position = 'fixed'; ta.style.opacity = '0';
  document.body.appendChild(ta); ta.select();
  try { document.execCommand('copy'); } catch(e) {} document.body.removeChild(ta);
}
```

---

## What NOT to change

- Corpus calculation logic (module buckets, panel ratios, raw → ceil math)
- Admin panel (product types, parser, install constants, import/export)
- V2 product type catalogue and all their settings
- Partlist paste import feature
- Admin storage key `cutr-corpus-v2-config`
- Platform install-time formula and constants
- Existing CSS design tokens and color variables

---

## Key difference summary: design-hours vs corpus installation duplicate

| | Design Hours | Corpus Installation |
|---|---|---|
| **Trigger** | User marks file as duplicate of another | User marks file as duplicate of another |
| **What happens** | File contributes 0h to order total | File's elements are replaced with a copy of the referenced file's elements |
| **Hours/corpus** | Excluded from total (billing logic) | Fully counted in total (no exclusion) |
| **Purpose** | "Don't charge design time again" | "Save me re-entering the same cards for a repeat floor" |
| **Link persists?** | Yes — `isDuplicateOf` stays set | No — copy is made, user edits independently |

---

## Working principles

- Read both reference files fully before writing anything.
- Don't touch calculation logic — only structure, UI, and state management.
- Single-file vanilla JS/HTML. No build tools, no npm, no frameworks.
- Ask before deciding on anything related to the quote line item names (DSP category) — confirm from the platform.
- Surface assumptions rather than guessing.
