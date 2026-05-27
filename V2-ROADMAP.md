# Corpus Currency Calculator — V2 Roadmap & Spec

> Working doc. Source of truth for what V2 is and where each task stands. Sync to Notion once the team has signed off on the input model.
>
> **Notion initiative:** https://www.notion.so/cutr/Corpus-Currency-Calculator-36a2552c8ff180f1ab9fce45c3d4fcf4
> **V1 (don't break):** `/index.html` deployed on Vercel
> **V2 prototype:** `/v2/index.html` (this work)

---

## 1. North star

Any two users score the same job within **±5%**. Every quote-linked corpus calculation has a **backtraceable input→output trail**. Adding a new product type to documentation takes **under 2 hours** thanks to a standardized template.

---

## 2. V1 → V2 delta (what changes)

**Removed from V1:**

- Doors / drawer fronts as scored inputs
- Cutouts (sink / hob / fitting) as 1 corpus per cutout
- Upholstery as a corpus contribution (covered by upholstery calculator)
- Countertop install time as a corpus contribution (covered by countertop calculator)
- Per-order save/load of full configurations
- Global preconditions (countertop/upholstery were one-screen-up — now per-card when applicable to that product type)
- Preassembled panel scoring (preassembled panels are folded into module install — no separate input)

**Added in V2:**

- One calculation = one file. **A file can contain multiple product types** — each added as a card. Total corpus = `ceil(sum of all card raw contributions)` (matches V1's add-on layer semantics).
- Top-level **installation type** selector (Freestanding / Built-in / Water-Electric outlets) and **floor number** input — file-level (one per file), feeding the platform install-time formula. Auto-prefills from the first card's product type default; user can override.
- **Platform interpretation** panel — shows per-corpus minutes + total install time (min + hours) for the picked setting and floor, matching the platform's own formula
- Standardized S/M/L/Oversized size buckets per product type with explicit dimension ranges, inline example strings, and image gallery per bucket (admin-editable)
- **Per-input corpus contribution** displayed inline next to every module bucket and every panel input (e.g. "+0.60 corpus each")
- Panels measured by m² **or** count-by-bucket — **user-toggleable** per file. Equivalence enforced: per-panel ratio = m² ratio × avg m² of bucket
- Panels split into **on-site / unknown (provisional)** rows. Preassembled panels are NOT entered (they don't drive extra time). Inline note explains.
- Headboards as a new product type
- Per-product-type **intro text** (admin-editable) — explanation + instructions shown above the inputs
- **hasCountertops / hasUpholstery** flags per product type — the exclusion banner only appears on product types where it applies (e.g., Cabinets/Counters for countertops; Benches/Headboards for upholstery)
- **Per-product-type default installation type** (admin-editable), overridable per file by the user
- Backtrace block: copy-pastable text with timestamp, all settings, every input, math, ceil, install time, provisional warnings, paste-into-quote instruction
- Partlist paste import (experimental): scoped to the file's product type; uses the Cutr default partlist template; results land in the same main UI fields (no separate confirmation screen)
- Admin panel rebuilt: product types (incl. intro, has-flags, default install, images), parser synonyms, install-time constants, JSON import/export

---

## 3. Input model

### Activities

| Activity | What it is | Unit | Notes |
|---|---|---|---|
| Module install | Placing/fixing a preassembled box | Count by S/M/L/Oversized bucket | Each bucket has its own per-corpus ratio |
| Panel install | External panels installed on site | m² OR count-by-bucket (user-toggle) | Equivalence enforced. Preassembled panels are NOT scored — folded into module install. |

### Panel input modes — user picks per file

- **By m²** (default): two rows — On-site / Unknown-provisional
- **By count (per bucket)**: per bucket — two number inputs side-by-side (On-site / Unknown-provisional)

Per-panel ratio in count mode is **derived**: `ratio_per_panel = ratio_per_m² × avg_m²_of_that_bucket`. Same physical coverage → same corpus number regardless of input mode.

### Provisional handling

When the user doesn't yet know if a panel will be preassembled or installed on site, they enter that quantity in the **Unknown** row. It's scored at the on-site ratio (conservative). The card, the result panel and the backtrace are all flagged provisional, with the instruction to re-run once the assembly plan is confirmed.

### Per-product-type exclusion notes

- `hasCountertops` (e.g., Cabinets, Counters, Reception desks) → show the "Countertop scored via countertop calculator?" toggle. Default Yes/excluded.
- `hasUpholstery` (e.g., Benches, Headboards) → show the "Upholstery scored via upholstery calculator?" toggle. Default Yes/excluded.

Other product types never see these toggles.

### Installation type + floor (top-level)

- 3 installation types: Freestanding (30 min install), Built-in (60), Water/Electric outlets (75)
- Each product type has a `defaultInstallType` (admin-editable) — pre-selected when product type is picked. User can override.
- Floor: integer ≥ 0. Adds `floor × 15` min/corpus to install time.
- The calculator's output is just the **corpus number**. The "Platform interpretation" panel shows what install time the platform will compute given (corpus, installation type, floor).

---

## 4. Platform install-time formula (matches platform `installation.js`)

```
time = qty × corpusQty × (UNLOADING + MOVING + install_for_type + floor × FLOOR_MIN)
```

Constants (admin-editable):

| Constant | Default | Notes |
|---|---|---|
| `UNLOADING_MIN` | 7.5 | per corpus |
| `MOVING_MIN` | 15 | per corpus |
| `FLOOR_MIN` | 15 | per corpus per floor |
| `freestanding` install | 30 | |
| `built-in` install | 60 | |
| `water-electric` install | 75 | |

The Admin → Install constants tab includes a 1–20 corpus sanity matrix that should match the values from the platform team.

---

## 5. Product type catalogue — V2 scope

| Product type | Modules | Panels | Countertops flag | Upholstery flag | Default install |
|---|---|---|---|---|---|
| Cabinets | ✓ S/M/L/Os | ✓ | ✓ | — | built-in |
| Benches | ✓ S/M/L/Os | ✓ | — | ✓ | freestanding |
| Counters / reception desks | ✓ S/M/L/Os | ✓ | ✓ | — | built-in |
| Wall paneling | — | ✓ | — | — | built-in |
| Planters | ✓ S/M/L/Os | — | — | — | freestanding |
| Wall dividers | ✓ S/M/L/Os | ✓ | — | — | freestanding |
| Floating shelves | ✓ S/M/L/Os | — | — | — | built-in |
| Hanging racks | ✓ S/M/L/Os | — | — | — | built-in |
| **Headboards (new)** | ✓ S/M/L/Os | ✓ | — | ✓ | freestanding |

Worktop is **removed as a product type in V2** — its install time is owned entirely by the countertop calculator. The exclusion note appears inline on Cabinets / Counters where countertops typically attach.

All bucket dimensions, ratios, examples, intro text and images in the prototype are **placeholders** — needs contractor calibration.

---

## 6. User flow

1. **File settings** — file name, installation type (one for the whole file; auto-prefills from the first card's product type default), floor (one for the whole file).
2. **What's in this file?** — chip picker. Clicking a chip **adds a card** for that product type. Same product type can be added multiple times (auto-numbered: "Wall paneling", "Wall paneling #2", …).
3. **Cards stack** — one card per product type instance. Each card has:
   - Editable name input (defaults to product type name + counter)
   - Intro text from admin (per product type)
   - Exclusion banners if applicable to that product type (countertop / upholstery — per-card toggle)
   - `📋 Paste partlist (experimental)` button — scoped to this card
   - **Modules** rows — thumbnail row, name + range + example, contribution chip, number input
   - **Panels** section — m²/count toggle (per card), on-site + unknown rows
   - Live raw contribution chip in the header
   - Remove button
4. **Result** — file's ceil corpus, raw → ceil, per-card raw breakdown, platform interpretation, backtrace block with copy button, provisional flag if any unknown panel is non-zero on any card.

Sticky total bar at the bottom: corpus number + setting + floor + total install minutes/hours.

Math: total `raw = Σ card.raw`; `corpus = ceil(raw)` once. Per-card raws stay as decimals — only the file total is ceiled.

---

## 7. Backtrace format

Copy-pastable text block, single button. Includes:

- Timestamp (local time + timezone)
- File name, product type, installation type, floor
- Countertop / upholstery exclusion state when applicable
- Source line if values came from a partlist paste
- Every input with contribution and calc string
- Corpus number (raw → ceil)
- Platform interpretation (per-corpus breakdown + total minutes/hours)
- Provisional warning block (if any unknown panel rows are non-zero) listing each provisional line and the re-run instruction
- Instruction: "Paste this block into the comment section of the quote."

---

## 8. Partlist paste import (experimental)

### Flow

1. User picks the product type for the file (calculator's already in that product type).
2. User clicks **Paste partlist** in the inputs section.
3. Modal explains: enable the platform's **Qty column** in partlist view settings, filter the partlist to only the parts belonging to this product type, then paste.
4. Parser:
   - Groups rows by `M#` prefix (configurable regex).
   - Classifies each group as **Module** (has Top + Bottom + Side + Backwall) or **Panel** (all parts are panel-like) or **Unknown**.
   - For modules: infers W (max Top L), H (max Side L), D (max Top W), maps to a bucket per the product type's thresholds.
   - For panels: each row expands by Qty; each panel's m² = W×L/1e6; if the user's current panel mode is `count`, each panel is bucketed by closest avg m².
   - Doors and drawer fronts are ignored.
5. Modal shows a parse-summary line ("X groups parsed, Y doors ignored, Z modules + N m² panels"). User clicks **Apply to form**.
6. The main form fields fill in — modules in their bucket counts, panels into the **on-site** row (m² or count, matching mode). User can then move some panels to **Unknown** manually if the assembly plan isn't confirmed.

### Default parser template

```
P-#  ·  M# - Part type  ·  T (mm)  ·  Part type  ·  T label  ·  Material  ·  Req. material  ·  W  ·  L  ·  Qty
```

Cutr default. **User must enable the Qty column** in the platform's partlist view settings before pasting. Admin can edit the column-map, module-ID regex, excluded part types, module-part-type synonyms, and panel-part-type synonyms in Admin → Parser.

### Known failure modes (set expectations)

- Module ID isn't `M#` (e.g., `Module 01`) → parser falls back to `__ungrouped__`; user adjusts manually after import or fixes regex.
- Localised part type names (`Achterwand` for `Backwall`) → not matched; user adds synonyms in admin.
- Mixed-content modules (a group with both module parts and extra side panels) → classified as module; the side-panel parts are absorbed; user adjusts the panel total manually.
- Doors hidden under different part type names (e.g., `Front`) → not ignored; user adds to excluded list.

---

## 9. Admin panel

Tabs:

- **Product types** — has-modules, has-panels, has-countertops, has-upholstery flags; default install type; panel default mode (m²/count); intro text; module bucket CRUD (name, W/H/D thresholds, ratio, example, image URLs); panel ratio per m²; panel bucket CRUD (name, avg m², example, derived per-panel ratio shown). All edits persist to `localStorage` key `cutr-corpus-v2-config`.
- **Parser** — default template id + column note + column map (read-only), module-ID regex, excluded/module/panel part-type synonym lists.
- **Install constants** — `UNLOADING_MIN`, `MOVING_MIN`, `FLOOR_MIN`, per-type install minutes, plus a 1–20 corpus sanity matrix for cross-checking with the platform team.
- **Import / export** — raw JSON edit, apply, reset-to-defaults.

---

## 10. Task status (mirrors Notion task DB)

| # | Task | Status | Notes |
|---|---|---|---|
| 1 | Define standardized size buckets per product type | 🟡 Placeholders | All product types have S/M/L/Os with W×H×D thresholds. Needs contractor validation. |
| 2 | Remove doors/drawer fronts as scored input | ✅ Done (V2) | Excluded in form + ignored by parser |
| 3 | Remove cutouts from Worktop scenario | ✅ Done (V2) | Worktop removed as a product type — owned by countertop calc |
| 4 | Convert Wall paneling input from count → m² | ✅ Done (V2) | m² is default; user can toggle to count |
| 5 | Audit each scenario for correct measurement unit | 🟡 First pass done | Product type catalogue documents activity per type. Needs contractor pass. |
| 6 | Define standardized manual template per product type | 🟡 Started | Intro text field exists per product type. Visual template TBD. |
| 7 | Write user manual per product type | 🟡 Placeholder intros | Each product type has draft intro text. Needs contractor to rewrite. |
| 8 | Add visual examples per product type | 🟡 Mechanism in place | Per-bucket image gallery wired; placeholder SVGs in default config. Admin can paste real URLs. |
| 9 | Write exclusions list with rationale | 🟡 Inline in intro + per-type | Exclusion banners surface countertop/upholstery rationale where applicable. Doors/cutouts excluded silently. |
| 10 | Add in-app double-count explainer | ✅ Done (V2) | Exclusion banners per applicable product type. |
| 11 | Build backtracing output | ✅ Done (V2) | Copy-pastable block with timestamp + paste-into-quote instruction. |
| 12 | Document handling for oversized / non-fitting parts | 🟡 Mechanism in place | Oversized bucket exists per type. Written guidance TBD. |
| 13 | Run V2 validation session with contractor on 2–5 real jobs | ⬜ Not started | Needs calibrated ratios first. |
| 14 | Stretch: partlist auto-prefill | 🟡 Experimental in V2 | Single paste button, results fill the main form. Tolerance / edge cases TBD. |
| **NEW** | Surface platform install-time formula in result | ✅ Done (V2) | Platform interpretation panel shows per-corpus + total minutes/hours per setting + floor. |
| **NEW** | Per-product-type intro text | ✅ Done (V2) | Admin-editable. |
| **NEW** | Per-product-type images per bucket | ✅ Done (V2) | Admin-editable URL list per bucket. |
| **NEW** | Show per-input corpus contribution inline | ✅ Done (V2) | Chip next to each module / panel input. |
| **NEW** | Multi-product-type per file | ✅ Done (V2) | Cards stack under file-level settings (install type, floor). Sum-then-ceil for the file's corpus number. |

Legend: ✅ Done · 🟡 In progress / partial · ⬜ Not started

---

## 11. Open decisions for the team / contractor

Listed in priority order — the prototype works with placeholder values, but these are the gates before contractor validation can be meaningful.

1. **Bucket dimension thresholds (W × H × D in cm) for each module-using product type.** Including Headboards. Owner: contractor + Ruben.
2. **Per-corpus ratios** for each module bucket and for on-site panels per product type. Owner: contractor.
3. **Oversized policy** — own ratio per type (current implementation) vs flat multiplier (e.g., 2× L). Confirm approach.
4. **Wall paneling default unit** — m² (current default) or count-by-bucket?
5. **Default install type per product type** — first-pass guesses are seeded. Contractor to confirm.
6. **Bucket examples** — rewrite the inline example strings per bucket per type with the contractor.
7. **Bucket images** — collect real photos per bucket per product type for the gallery.
8. **Intro text** — rewrite per product type with the contractor.

---

## 12. Working principles

- **Don't break V1 in production until V2 is validated.** V1 stays at `/`, V2 lives at `/v2/`.
- **Read before you write.** Re-read the relevant Notion task and any linked manual sections before touching code.
- **Ask before deciding.** Don't make decisions on input model, size buckets, product-type definitions, or tech stack without checking with Ruben. These belong with the contractor partner where applicable.
- **Surface assumptions.** State them, ask if they're right.
- **Don't re-add killed inputs.** The out-of-scope list is non-negotiable for V2.

---

## 13. Next steps

1. Open `/v2/index.html` in a browser, walk through the flow with a real job in mind.
2. Walk through a partlist paste to feel the experimental import.
3. Bring the prototype + this roadmap to the contractor for a working session on bucket thresholds and ratios.
4. Calibrate ratios on 2–5 real jobs; check the ±5% inter-rater variance.
5. Sync this doc back into the Notion initiative once the input model has team sign-off.
