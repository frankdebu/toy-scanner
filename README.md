# Toy scanner

A browser-only app that uses your webcam to detect a toy held in front of the camera,
classify it by colour and size, and either:

- **Inventory mode** — count objects per colour/size and export a CSV.
- **Checkout mode** — look up the price per colour/size (in AUD), tally a cart, show a
  total, and log completed sales to a session transaction history.

Everything runs client-side in the browser. No server, no backend, no build step.

## Files

- `index.html` — the whole app (camera, detection logic, calibration, both modes).
- `config.csv` — editable list of `color,size,price` rows in AUD.

## How classification works

- **Colour** is matched against a fixed palette (`red, blue, green, yellow, orange,
  purple`) — nearest match by average colour of the detected object.
- **Size** is matched by nearest calibrated example, not fixed pixel ranges — see
  "Calibrating sizes" below. There's no gap or undefined zone: every object always maps
  to whichever calibrated size it's closest to.
- **Manual override** — a "Last scanned" panel always shows what was just detected, with
  a dropdown to correct it on the spot (no waiting, no re-scanning).
- **Dot-sticker override** — for recurring edge cases, sticking a single small dot on a
  toy tells the app definitively which size it is, skipping the automatic guess entirely.
  Each size has its own reserved dot colour (small/medium/large/xlarge each map to a
  distinct neon tone), so one dot is enough regardless of how many size tiers you use —
  no counting needed. See the user manual for the full workflow.

## Calibrating sizes

Sizes aren't defined by pixel ranges in the config file — they're calibrated live:

1. Start the camera and capture the empty background.
2. In the **Calibrate sizes** panel, hold up one example toy per size (e.g. one small,
   one medium, one large) and click **Capture example** for each.
3. From then on, every scanned object is matched to whichever calibrated size its
   measured size is closest to.
4. Click **Save calibration** to download `calibration.json` so you don't have to redo
   this next session — load it back in with **Load calibration**.

Recalibrate if you change camera distance/angle significantly, since size measurement is
relative to how large the object appears in frame.

## config.csv format

```
color,size,price
blue,small,1.50
blue,medium,3.00
green,large,5.00
```

- `color` must be one of the six supported colours.
- `size` is any label you choose (e.g. `small`, `medium`, `large`, `xlarge`) — the app
  picks up whichever size names appear anywhere in the file and calibrates for exactly
  those.
- A colour doesn't need every size — only add the rows that actually apply.
- `price` is in AUD.

## Deploying (GitHub Pages)

1. Push `index.html` + `config.csv` to a GitHub repo.
2. Repo settings → **Pages** → source: deploy from branch `main`, folder `/root`.
3. Live at `https://<username>.github.io/<repo>/` within a minute or two, served over
   HTTPS automatically — required for camera access to work.

## Known limitations (v1)

- Detection is background-diff + single-largest-blob — works best with a plain, static
  background and one object in frame at a time.
- Colour classification is nearest-match against a small fixed palette; multi-toned toys
  are classified by their average colour.
- Dot detection assumes each reserved size colour never otherwise appears on your toys.
  The defaults are fluorescent/neon tones chosen to be unusual for moulded toy plastic,
  but if a specific toy clashes anyway, skip the dot for that item and use the dropdown
  correction instead — see the user manual.
- Only sizes with a reserved colour defined in `SIZE_DOT_COLORS` (in `index.html`) support
  dot overrides; the dropdown correction always works regardless.
- Transaction history is kept in memory for the browser session only — download the CSV
  before closing the page if you want to keep a record.
