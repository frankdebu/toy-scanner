# Toy scanner

A browser-only app that uses your webcam to detect a toy placed in the scanning area,
measure its **size** from its shape (colour-independent), read its **colour** from a dot
sticker if one is present, and either:

- **Inventory mode** — count objects per colour/size and export a CSV.
- **Checkout mode** — look up the price per colour/size (in AUD), tally a cart, show a
  total, and log completed sales to a session transaction history.

Everything runs client-side in the browser. No server, no backend, no build step.

## Files

- `index.html` — the whole app (camera, detection logic, calibration, both modes).
- `config.csv` — editable list of `color,size,price` rows in AUD.

## Setup flow

The app has three tabs: **Setup**, **Inventory**, **Checkout**. The camera preview and
tab bar are both pinned (`position: sticky`) to the top of the screen, so they stay in
view while you scroll through the Setup tab's checklist/config/calibration sections —
important on narrow/mobile screens where those sections don't all fit on one page.

The Setup tab walks the operator through four gated steps — start camera, capture
background, load config, calibrate sizes — via a checklist card, and **won't allow
scanning to start until all four show "done"**. Scanning itself can be paused and resumed
freely afterwards without losing any setup state; only actually changing something
(camera restarted, new config loaded) resets the relevant step back to "pending".

## How detection works

### Size — pure shape/contrast, no colour involved
Every frame is converted to grayscale (brightness only) and diffed against the captured
empty background. Whichever pixels are different enough form the object's silhouette;
the **number of those pixels is the size measurement**, matched against nearest
calibrated example (see below). Because this never looks at colour at all, it's immune
to glare, reflective/glossy toys, and grey/neutral-coloured toys — all things that would
confuse a colour-based measurement.

### Colour — read only from a dot sticker
The toy's own body colour is never used for classification (too unreliable in practice —
lighting, glare, and grey/reflective surfaces all interfere with it). Instead, colour
comes **only** from a small dot sticker: a red dot means "red category", a blue dot means
"blue category", directly — no separate reserved palette to think about. If no dot is
found, the item is registered as **uncategorized** and the operator picks the correct
colour from a dropdown (see "Last scanned" below) — this is a normal, expected path, not
an error state.

### Motion-gated timing
Every frame is also compared to the *previous* frame (not the background) to detect
whether anything is currently moving — e.g. a hand placing or removing an item. A
reading is only trusted once the scene has held still for a short run of frames. This
comparison is deliberately:
- **Restricted to the object's own bounding box**, not the whole frame — background-only
  jitter (camera vibration, sensor noise) never counts as motion.
- **Averaged over coarse blocks**, not raw pixels — small per-pixel noise/vibration
  mostly cancels out within a block, while a real hand moving through the object area
  still changes a large share of blocks at once.

The status pill reflects each stage: *"place an item"* → *"movement detected — hold
still"* → *"settling…"* → *"reading…"* → *"counted: … — remove item to continue"*.

### Manual correction
The **Last scanned** panel always shows the most recent reading — colour swatch, colour
name (or "uncategorized"), and size — with two dropdowns to correct either on the spot,
no waiting or re-scanning required. Colour correction is the more commonly needed one,
since many items won't have a dot; the panel gives a tip on the dot colour to use for
that item next time once you've corrected it manually.

## Calibrating sizes

Sizes aren't defined by pixel ranges in the config file — they're calibrated live, and
this step involves no colour identification at all now:

1. Start the camera and capture the empty background.
2. In the **Calibrate sizes** panel, place one example toy per size (e.g. one small, one
   medium, one large) in the scanning area and click **Capture example** for each.
3. From then on, every scanned object is matched to whichever calibrated size its
   measured silhouette size is closest to.
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

- `color` must be one of the six supported colours (used both for dot matching and price
  lookup).
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

- Detection still relies on a plain, mostly static background/mat — the silhouette is
  only as clean as the contrast between the object and its background. A dark, matte
  (non-reflective) surface works best; a glossy/reflective surface can introduce glare
  that creates gaps in the detected silhouette.
- Dot stickers should be matte, not glossy/laminated — a shiny sticker can suffer the
  same glare issue in miniature and read unreliably.
- A count only commits once a reading wins a majority within a short rolling window of
  recent settled frames — tolerant of the odd noisy frame, but this is still a
  lightweight heuristic, not a trained ML model.
- A genuinely more accurate approach would use a trained ML model (e.g. TensorFlow.js)
  to detect the object directly — a larger rebuild than this version, and out of scope
  for now.
- Transaction history is kept in memory for the browser session only — download the CSV
  before closing the page if you want to keep a record.
