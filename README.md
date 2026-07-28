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

## Setup flow

The app has three tabs: **Setup**, **Inventory**, **Checkout**. The camera preview is
always visible regardless of tab, but the setup controls (checklist, config loader,
calibration) live only on the **Setup** tab, keeping the Inventory/Checkout views
uncluttered once you're up and running.

The Setup tab walks the operator through four gated steps — start camera, capture
background, load config, calibrate sizes — via a checklist card, and **won't allow
scanning to start until all four show "done"**. This avoids the failure mode of scanning
starting before calibration exists. Scanning itself can be paused and resumed freely
afterwards without losing any setup state; only actually changing something (camera
restarted, new config loaded) resets the relevant step back to "pending".

## How classification works

- **Motion-gated detection**: every frame is checked against both the empty background
  (is anything here?) and the *previous* frame (is anything currently moving?). Colour
  and size are only ever measured once the scene has been still for a short run of
  frames — this is what stops a hand mid-placement or mid-removal from being read as the
  item itself. The status pill reflects this directly: "place an item" → "movement
  detected — hold still" → "settling…" → "counted: … — remove item to continue".
- **Colour and size are measured together**: the app finds foreground pixels (ones that
  differ from the captured background) whose **hue** closely matches one of the six toy
  colours (hue, not raw RGB, so lighting/brightness changes don't throw it off — a red
  toy in shadow is still "red" in hue even though its RGB values got darker). Very
  washed-out or very dark pixels are excluded from matching entirely, since they're not
  reliably any particular colour. Whichever colour has the most matching pixels is the
  detected colour.
- The **size measurement** is the bounding box drawn around just those colour-matched
  pixels — not the raw foreground/diff region. This is what keeps a hand/arm holding the
  toy from inflating the measurement (skin tone doesn't match any toy colour, so it's
  excluded entirely), while still capturing the toy's true physical extent even if part
  of it is a different colour (e.g. wheels, trim).
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

- Detection is background-diff + colour-matched pixel counting — works best with a
  plain, static background and one object in frame at a time.
- Colour classification picks whichever of the six toy colours has the most matching
  foreground pixels; multi-toned toys are classified by their dominant colour patch.
  If a toy's actual colour is very close to skin tone (e.g. a pale orange/tan toy), the
  hand-exclusion logic may undercount it — hold more of the toy's coloured surface toward
  the camera in that case.
- A count only commits once a colour/size reading wins a majority within a short rolling
  window of recent frames (not every single frame needs to agree) — this is more
  tolerant of the odd noisy frame than requiring an unbroken streak, but detection is
  still a lightweight heuristic (not a trained ML model), so lighting and background
  still meaningfully affect accuracy. For best results: even, consistent lighting, and a
  background/mat that contrasts clearly with all six toy colours and with skin tone (a
  plain dark mat works well).
- A genuinely more accurate approach would use a trained ML model (e.g. TensorFlow.js)
  to detect the object directly rather than inferring it from colour/background
  differences — a larger rebuild than this version, and out of scope for now, but worth
  considering if heuristic accuracy proves insufficient in practice.
- Dot detection assumes each reserved size colour never otherwise appears on your toys.
  The defaults are fluorescent/neon tones chosen to be unusual for moulded toy plastic,
  but if a specific toy clashes anyway, skip the dot for that item and use the dropdown
  correction instead — see the user manual.
- Only sizes with a reserved colour defined in `SIZE_DOT_COLORS` (in `index.html`) support
  dot overrides; the dropdown correction always works regardless.
- Transaction history is kept in memory for the browser session only — download the CSV
  before closing the page if you want to keep a record.
