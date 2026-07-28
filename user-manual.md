# Toy scanner — user manual

A quick guide to setting up and running the toy scanner app. For technical/deployment
details, see `README.md`.

## How the app works, in a nutshell

- **Size** is measured automatically from the item's shape/silhouette — no colour
  involved at all. This makes it reliable even for grey, glossy, or reflective toys.
- **Colour** is read only from a small dot sticker on the item. A red dot means "red
  category", directly. If there's no dot (yet), the item is counted as **uncategorized**
  and you pick the right colour from a dropdown — this is a completely normal, everyday
  case, not an error.
- Both the camera preview and the tab bar stay pinned to the top of the screen as you
  scroll, so you can always see the camera feed even while working through setup on a
  phone.

## Initial setup

Do this once when you first start using the app (or any time you move the camera, change
the lighting, or want to refresh your size calibration).

The app has three tabs: **Setup**, **Inventory**, and **Checkout**. The camera preview
stays visible regardless of which tab you're on, but the setup controls (checklist,
config file, calibration) only appear on the **Setup** tab — switch to Inventory or
Checkout once setup is complete to start scanning for real.

The **Setup checklist** card tracks four steps and shows a "done" / "pending" badge for
each. **Scanning can't start until all four are done.**

### 1. Start the camera
1. Click **Start camera** (it becomes **Stop camera** once the feed is live).
2. Allow camera access when your browser asks — this only appears the first time.
3. On phones/tablets, the app requests the **rear camera** by default. If it opens the
   wrong one, click **Switch camera** to flip between rear and front — this restarts the
   feed, so recapture the background afterwards.
4. Click **Stop camera** any time to release the webcam. Starting it again will ask you
   to recapture the background, since the camera may have moved.

### 2. Capture the empty background
1. Clear the scanning area completely — no toys, no hands.
2. Click **Capture empty background**.

> Redo this step whenever you move or bump the camera, change the lighting, or restart
> the camera. For best results, use a plain, **matte** (non-glossy) surface or mat under
> the scanning area — a reflective/glossy surface can create glare that shows up as gaps
> in the detected shape.

### 3. Load the config file
1. Under **Config file**, click the file picker and select your `config.csv`.
2. Check the message underneath confirms how many rows were loaded.

If you're hosting the app with `config.csv` sitting next to `index.html`, it tries to load
automatically when the page opens.

### 4. Calibrate sizes
The app doesn't use fixed measurements for "small", "medium", "large" (or whatever size
names are in your config) — instead, you show it one example of each, and this step is
now entirely colour-independent (no "could not identify colour" errors possible here,
since colour is never part of calibration):

1. Once your config is loaded, the **Calibrate sizes** panel lists every size name found
   in it.
2. Place one item that represents "small" in the scanning area and click **Capture
   example** next to it.
3. Repeat for "medium", "large", and any other sizes in your list.
4. Once every size shows a green "done" badge (both here and in the checklist above),
   setup is complete.

Every item scanned afterwards is matched to whichever calibrated size its measured
silhouette is closest to — there's no in-between gap where an object doesn't fit
anywhere.

**To avoid recalibrating every time you open the app:** click **Save calibration** to
download a small `calibration.json` file. Next session, use **Load calibration** to load
it back in instead of re-measuring examples. Only recalibrate if you've noticeably
changed the camera's distance or angle from the scanning area.

### Starting, pausing, and resuming scanning
Once all four checklist steps show "done", the **Start scanning** button becomes
available at the top of the checklist card (on the **Setup** tab). Click it to begin —
switch to the **Inventory** or **Checkout** tab and the camera will now measure and
categorise items as you place them; the live camera preview stays visible on every tab.

You can go back to the **Setup** tab and click **Pause scanning** at any time without
losing any setup — background, config, and calibration all stay in memory, so clicking
**Start scanning** again picks straight back up.

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

**To add a new size:** add rows for it in `config.csv`, save the file, then reload it in
the app — the new size will automatically appear in the Calibrate sizes panel.

**To change a price:** just edit the number in the relevant row and reload the file. No
recalibration needed — prices and sizing are independent.

## Handling colour and sizing on the spot

### The "Last scanned" panel
Every time an item is counted, the **Last scanned** panel shows exactly what was detected
— a colour swatch and name (or "uncategorized"), plus the size — with **two dropdowns**
next to it: one for colour, one for size. Pick the correct value from either and the
tally or cart updates immediately, no rescanning needed.

**Colour correction will be the one you use most often**, since many items won't have a
dot sticker yet. Seeing "uncategorized" is completely normal — just pick the right
colour from the dropdown.

### Dot stickers — for items you scan repeatedly
If you'll scan the same item more than once (e.g. at both intake and checkout), it's
worth marking it with a dot sticker so colour is read automatically from then on, with
no dropdown needed.

**How it works:** the dot is simply the same colour as the category — a red dot means
red, a blue dot means blue, and so on. No separate colour system to remember.

**What you need:** small, plain, matte dot stickers in the same six colours as your
config. A cheap multi-colour pack works well, for example:
[Avery Multi-Coloured Dot Stickers, 8mm, 416 pack (Officeworks)](https://www.officeworks.com.au/shop/officeworks/p/avery-multi-coloured-dot-stickers-8mm-416-pack-av932291?msockid=2f5d8fcbbe0b6e4230669842bf056f2b).
Avoid glossy/laminated dots if possible — a shiny sticker can pick up glare and read
unreliably, the same way a shiny toy body would.

**How to apply a dot:**
1. Scan the item as normal.
2. If it shows "uncategorized" (or the wrong colour), correct it using the colour
   dropdown in the Last Scanned panel.
3. The panel will show a tip: *"stick a [colour] dot on this item so it scans as
   '[colour]' automatically next time."*
4. Stick one dot of that colour somewhere visible on the item.
5. From then on, the camera reads the dot directly — no dropdown needed for that item.

## Understanding the status pill while scanning

The small pill on top of the camera view tells you exactly what the app is doing at each
moment:

| Status | What it means |
|---|---|
| *place an item* | Nothing is in the scanning area yet. |
| *movement detected — hold still* | Something is currently moving (a hand placing or removing an item). The app deliberately does **not** try to read anything while this shows. |
| *settling…* | Movement has just stopped; the app is confirming the scene is genuinely still before trusting a reading. |
| *reading…* | The scene is still and the app is actively classifying — this only lasts a moment. |
| *counted: [colour] · [size] — remove item to continue* | The item has been counted/priced. Remove it before the next item will be counted. |

This sequence exists specifically so an item is never measured *while* a hand is placing
or picking it up — only once everything has settled and stayed still for a moment.

## Usage A — Inventory

Use this mode to count second-hand toys into stock by colour and size.

1. Make sure setup above is complete (camera started, background captured, config loaded,
   sizes calibrated).
2. Click the **Inventory** tab.
3. Place the first item in the scanning area and wait for it to be counted — the status
   pill will guide you through "settling…" → "counted: …".
4. Check the **Last scanned** panel — if it shows "uncategorized" or the wrong size, pick
   the correct value from the dropdowns.
5. Remove the item before placing the next one (the app needs the area to go back to
   empty before it will count a new object).
6. Repeat for each item. The **Counted this session** panel keeps a running tally by
   colour and size (including an "uncategorized" bucket if any items haven't been
   assigned a colour yet).
7. If you make a mistake or want to start over, click **Reset session** — this clears the
   tally on screen (it doesn't undo anything already exported).
8. When you're done, click **Export CSV** to download `inventory.csv` with each
   colour/size combination and its count.

## Usage B — Checkout

Use this mode to ring up a customer's toys and total the sale.

1. Make sure setup above is complete.
2. Click the **Checkout** tab.
3. Scan each toy the customer is buying the same way as Inventory — place, wait for the
   count, then remove before scanning the next one.
4. Each scanned item is added to the **Cart**, showing colour, size, quantity, and
   subtotal. An "uncategorized" item shows $0 until you assign its colour via the
   dropdown — check the Last Scanned panel after each scan to catch this quickly.
5. The **Total (AUD)** updates as you go.
6. If you need to start over for this customer, click **Clear cart**.
7. Once everything is scanned and the total looks right, click **Complete sale** — this
   logs the sale into **Transaction history** and clears the cart for the next customer.
8. At any point (e.g. end of day), click **Download history CSV** to save all
   transactions from this session to `transactions.csv`.

> Transaction history only lasts for the current browser session — refreshing or closing
> the page clears it. Download the CSV before you close the app if you want to keep a
> record.
