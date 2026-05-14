# corpus-currency

Internal tool for Cutr's quoting platform. Helps users translate what they see in a design (preassembled boxes, loose panels, etc.) into the **corpus number** and **installation type** to enter on the platform per quoted element.

## How it works

The Cutr platform's installation time formula is:

```
install_min = corpus × (unloading + moving + install_by_type + floor × floor_per_corpus)
```

The user can only influence install time by adjusting **corpus number**, **installation type** (freestanding / built-in), and **floor**. This tool standardises how those inputs are set across scenarios like cabinets, benches, counters, wall paneling, planters, dividers, shelves, hanging racks, and nested worktops.

## Two modes

**User mode** — Pick the dominant scenario, fill in what you see in the design (e.g. _3 cabinet corpuses + 2 loose panels + 1 door_), optionally add layered sub-elements (_+ nested worktop, 1 piece, 1 cutout_), get the corpus number and type to enter on the platform.

**Admin mode** — Edit scenarios, inputs, per-corpus divisors, and the underlying calculation constants. Changes save to browser `localStorage`. Use **Export JSON** to share a config with the team, or **Import JSON** to load one. Settings persist per browser; for shared config across the team, distribute the exported JSON.

## Config schema

```json
{
  "constants": {
    "unloadingMin": 7.5,
    "movingMin": 15,
    "installFreestandingMin": 30,
    "installBuiltInMin": 60,
    "floorMinPerCorpus": 15
  },
  "scenarios": [
    {
      "id": "cabinets",
      "name": "Standard cabinets",
      "description": "...",
      "type": "built-in",
      "inputs": [
        { "id": "corpuses", "label": "Cabinet corpuses", "unit": "corpuses", "perCorpus": 1 },
        { "id": "panels", "label": "Loose panels", "unit": "panels", "perCorpus": 3 }
      ]
    }
  ]
}
```

- `perCorpus` is the divisor. `1` = 1:1. `3` = 1 corpus per 3 units (cheap to install). `0.5` = 1 unit contributes 2 corpus (premium, e.g. ceiling-hung racks).
- Corpus contributions from main + add-on layers are summed as raw decimals, then ceiled once at the end.

## Deploy to Vercel

```bash
npm i -g vercel
vercel
```

Or push to GitHub and connect via the Vercel dashboard. The `vercel.json` here sets `cleanUrls` and short cache on `index.html` so config tweaks ship fast.

For internal-only access:
- Vercel Pro has built-in password protection on deployments
- Or sit it behind Cloudflare Access
- Or just live with a hard-to-guess `*.vercel.app` URL

## Local dev

It's a single static HTML file. Open `index.html` in a browser. Done.
