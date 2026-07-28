# Toy scanner — user manual

A quick guide to setting up and running the toy scanner app. For technical/deployment
details, see `README.md`.

## Initial setup

Do this once when you first start using the app (or any time you move the camera, change
the lighting, or want to refresh your size calibration).

The app has three tabs: **Setup**, **Inventory**, and **Checkout**. The camera preview is
always visible on the left regardless of which tab you're on, but the setup controls
(checklist, config file, calibration) only appear on the **Setup** tab — switch to
Inventory or Checkout once setup is complete to start scanning for real.

The **Setup checklist** card tracks four steps and shows a "done" / "pending" badge for
each. **Scanning can't start until all four are done** — this is intentional, so you can't
accidentally scan before the app is actually ready. Work through the steps below in
order; the checklist updates itself as you go.

### 1. Start the camera
1. Click **Start camera** (it becomes **Stop camera** once the feed is live).
2. Allow camera access when your browser asks — this only appears the first time.
3. On phones/tablets, the app requests the **rear camera** by default. If it opens the
   wrong one, click **Switch camera** to flip between rear and front — this restarts the
   feed, so recapture the background afterwards.
4. Click **Stop camera** any time to release the webcam (e.g. when you're done for the
   day). Starting it again will ask you to recapture the background, since the camera may
   have moved.

### 2. Capture the empty background
1. Clear the tray or table in front of the camera completely — no toys, no hands.
2. Click **Capture empty background**.

> Redo this step whenever you move or bump the camera, or change the lighting — and
> whenever you've stopped and restarted the camera.

### 3. Load the config file
1. Under **Config file**, click the file picker and select your `config.csv`.
2. Check the message underneath confirms how many rows were loaded.

If you're hosting the app with `config.csv` sitting next to `index.html`, it tries to load
automatically when the page opens.

### 4. Calibrate sizes
The app doesn't use fixed measurements for "small", "medium", "large" (or whatever size
names are in your config) — instead, you show it one example of each:

1. Once your config is loaded, the **Calibrate sizes** panel lists every size name found
   in it.
2. Hold up one toy that represents "small" and click **Capture example** next to it.
3. Repeat for "medium", "large", and any other sizes in your list.
4. Once every size shows a green "done" badge (both here and in the checklist above),
   setup is complete.

Every toy scanned afterwards is matched to whichever calibrated size it's closest to —
there's no in-between gap where an object doesn't fit anywhere.

**To avoid recalibrating every time you open the app:** click **Save calibration** to
download a small `calibration.json` file. Next session, use **Load calibration** to load
it back in instead of holding up examples again. Only recalibrate if you've noticeably
changed the camera's distance or angle from the scanning area.

### Starting, pausing, and resuming scanning
Once all four checklist steps show "done", the **Start scanning** button becomes
available at the top of the checklist card (on the **Setup** tab). Click it to begin —
switch to the **Inventory** or **Checkout** tab and the camera will now count or price
objects as you hold them up; the live camera preview stays visible on every tab, so you
can always see what's being detected.

You can go back to the **Setup** tab and click **Pause scanning** at any time (e.g. to
answer a question, deal with a customer query, or step away) without losing any setup —
background, config, and calibration all stay in memory, so clicking **Start scanning**
again picks straight back up. You only need to redo a step if you actually change
something (moved the camera, loaded a different config, etc.) — the checklist will tell
you if a step needs redoing by switching back to "pending".

### Updating config.csv

The config file controls what colour/size combinations exist and what they cost:

```
color,size,price
blue,small,1.50
blue,medium,3.00
green,large,5.00
```

- **color** — must be one of: `red, blue, green, yellow, orange, purple`.
- **size** — any name you choose (e.g. `small`, `medium`, `large`, `xlarge`). A colour
  doesn't need every size — only include the combinations you actually price.
- **price** — in AUD, e.g. `3.00`.

**To add a new size (e.g. `xlarge`):** add rows for it in `config.csv`, save the file,
then reload it in the app — the new size will automatically appear in the Calibrate sizes
panel, ready to capture an example for.

**To change a price:** just edit the number in the relevant row and reload the file. No
recalibration needed — prices and sizing are independent.

Keep a backup of your working `config.csv` before making changes, in case you want to
revert.

## Handling scanning mistakes

However confident the automatic detection is, you always have two ways to correct it:

### Option 1 — the dropdown (works every time, no prep needed)
Every time an object is counted, the **Last scanned** panel shows what was detected
(colour + size) with a dropdown next to it. If it's wrong, just pick the right size from
the dropdown — the tally or cart updates immediately, no rescanning needed.

### Option 2 — a dot sticker (for toys you scan often, or persistent edge cases)
If a particular toy keeps getting misread (e.g. it sits right on the boundary between two
sizes), you can mark it once with a single small dot sticker so it's recognised correctly
every time from then on, with no dropdown correction needed at checkout.

**How it works:** each size has its own reserved dot colour, so the dot's *colour* tells
the app the size directly — you only ever need one dot, never more, regardless of how
many size tiers you use:

| Size | Reserved dot colour |
|---|---|
| small | neon pink |
| medium | neon green |
| large | neon orange |
| xlarge | neon yellow |

**What you need:** small, plain dot stickers in these (or similar bright/fluorescent)
tones. A cheap multi-colour pack works well as a starting point, for example:
[Avery Multi-Coloured Dot Stickers, 8mm, 416 pack (Officeworks)](https://www.officeworks.com.au/shop/officeworks/p/avery-multi-coloured-dot-stickers-8mm-416-pack-av932291?msockid=2f5d8fcbbe0b6e4230669842bf056f2b) —
note that pack's shades are fairly standard (red/yellow/green/blue) rather than true
neon/fluorescent, so check them against the app before relying on them; a craft or
stationery fluorescent dot pack will match the defaults more closely.

**If a reserved colour is too close to a toy's own colour** (e.g. the "neon orange" dot
on an orange toy, or "neon pink" on a red toy): don't use a dot for that item — just use
the dropdown correction (Option 1) every time instead. This is an expected, handled case,
not a fault in the app.

**How to apply a dot:**
1. Scan the toy as normal.
2. If it's misread, correct it using the dropdown (Option 1 above).
3. The panel will show a tip naming the reserved colour for the corrected size, e.g.
   *"stick a neon green dot on this item so it scans as 'medium' automatically next
   time"* — unless that colour would clash with the toy's own colour, in which case just
   keep using the dropdown for that item.
4. Stick one dot of that colour somewhere visible on the toy.
5. From then on, the camera reads the dot directly and always classifies that toy
   correctly — no dropdown needed.

This is worth doing for toys you'll scan repeatedly (e.g. at both intake and checkout);
for a one-off correction, the dropdown alone is enough.

## Understanding the status pill while scanning

The small pill on top of the camera view tells you exactly what the app is doing at each
moment — it's worth knowing what each message means so scanning feels predictable rather
than mysterious:

| Status | What it means |
|---|---|
| *place an item* | Nothing is in frame yet — go ahead and place/hold up a toy. |
| *movement detected — hold still* | Something is currently moving (a hand placing or removing an item). The app deliberately does **not** try to read anything while this is showing. |
| *settling…* | Movement has just stopped; the app is confirming the scene is genuinely still before trusting a reading. |
| *reading…* | The scene is still and the app is actively classifying — this only lasts a moment. |
| *counted: [colour] · [size] — remove item to continue* | The item has been counted/priced. Remove it from frame before the next item will be counted. |
| *scanning — no toy colour detected* | Something is present and still, but its colour couldn't be confidently identified — try adjusting lighting or the item's position. |

This sequence exists specifically so an item is never measured *while* a hand is placing
or picking it up — only once everything has settled and stayed still for a moment.

## Usage A — Inventory

Use this mode to count second-hand toys into stock by colour and size.

1. Make sure setup above is complete (camera started, background captured, config loaded,
   sizes calibrated).
2. Click the **Inventory** tab.
3. Hold up the first toy and keep it steady for under a second, until it's counted — the
   status pill will briefly show "counted: [colour] · [size]".
4. Check the **Last scanned** panel — correct it via the dropdown if it's wrong (see
   above).
5. Remove the toy from frame before scanning the next one (the app needs the frame to go
   back to empty before it will count a new object).
6. Repeat for each toy. The **Counted this session** panel keeps a running tally by
   colour and size.
7. If you make a mistake or want to start over, click **Reset session** — this clears the
   tally on screen (it doesn't undo anything already exported).
8. When you're done, click **Export CSV** to download `inventory.csv` with each
   colour/size combination and its count.

## Usage B — Checkout

Use this mode to ring up a customer's toys and total the sale.

1. Make sure setup above is complete.
2. Click the **Checkout** tab.
3. Scan each toy the customer is buying the same way as Inventory — hold steady, then
   remove before scanning the next one.
4. Each scanned toy is added to the **Cart**, showing colour, size, quantity, and
   subtotal.
5. Check the **Last scanned** panel after each scan — correct via the dropdown if needed,
   right there, no need to make the customer wait.
6. The **Total (AUD)** updates as you go.
7. If you need to start over for this customer, click **Clear cart**.
8. Once everything is scanned and the total looks right, click **Complete sale** — this:
   - Logs the sale into **Transaction history** below (with time, items, and total).
   - Clears the cart, ready for the next customer.
9. At any point (e.g. end of day), click **Download history CSV** to save all
   transactions from this session to `transactions.csv`.

> Transaction history only lasts for the current browser session — refreshing or closing
> the page clears it. Download the CSV before you close the app if you want to keep a
> record.
