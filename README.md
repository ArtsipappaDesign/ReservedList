# Reserve — Reserved List Tracker

A Magic: The Gathering Reserved List price tracker. Built as a single-file PWA.

## What it does

- Fetches all ~571 Reserved List cards from the **Scryfall API** on first open
- Shows current **Cardmarket EUR** and **TCGplayer USD** prices side-by-side
- Computes **EU/US spread** using a live EUR/USD rate (from frankfurter.dev)
- Builds **historical deltas (7d / 30d / 365d) locally** — every refresh saves a snapshot to your browser, and deltas appear once enough snapshots exist
- Full filter and sort: color, format legality (Premodern / Legacy / Vintage / Commander), price range, all the gain/loss columns
- Click any row for card detail with image, full historical comparison, and link to Scryfall

## How the history works (read this — it's the key idea)

The very first time you open the app, the price columns are populated but the "7d / 30d / 365d" columns will all show `—`. That's expected. There's no historical data yet because you just started.

Every time you tap "Refresh prices" (or open the app after 6+ hours away), the app:
1. Fetches today's prices from Scryfall
2. Saves a timestamped snapshot to your browser's localStorage
3. Looks for older snapshots roughly 7, 30, and 365 days ago and computes percent change

So:
- After **1 day** of use → no deltas yet
- After **1 week** of use → 7d column comes alive
- After **1 month** of use → 30d column populated
- After **1 year** → full 365d annual deltas

The longer you use it, the more useful it gets. It's an investment in your own private price history.

## Test it locally

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## What's here

- `index.html` — full app
- `manifest.json` — PWA manifest
- `sw.js` — service worker (caches shell, never caches Scryfall data so prices are always fresh)
- `icon-192.png`, `icon-512.png` — icons

## Architecture notes

- **No backend, no server.** Everything runs in the browser.
- **No accounts.** All data is in your browser's localStorage.
- **Scryfall rate limits**: the app respects Scryfall's published 100ms request spacing. Initial fetch hits 4 pages with delays = roughly 1-2 seconds total.
- **Storage**: ~571 cards × current prices ≈ 100KB. Each daily snapshot ≈ 25KB. 400 days of history ≈ 10MB, well within localStorage limits.
- **FX rate**: EUR/USD pulled from frankfurter.dev (free, no key). If that fails, falls back to a hardcoded 1.08.

## Path to Google Play Store

Same as Two Minutes and Lumo: push to a GitHub repo, enable Pages, optionally wrap with Bubblewrap.

If you do publish: **the Play Store has policies about apps that reference third-party brands (Magic: The Gathering, Wizards of the Coast).** A price-tracking app that's clearly a tool, doesn't reproduce card art (we only display via Scryfall API which is licensed) and doesn't pretend to be official, has generally been fine on the store. But worth checking the latest fan content policy before publishing publicly.

## Caveats built into the UI

- The About modal explains that prices are averaged feeds, not real-time market depth
- "EU/US spread" is flagged as a directional signal, with shipping/VAT mentioned
- The card detail modal honestly shows "No history yet" when there's nothing to compare against
- The footer states clearly that history is local and not shared

## Privacy

Three outbound requests:
1. **Scryfall API** — to fetch card data and prices (their published service, intended for this use)
2. **frankfurter.dev** — for the EUR/USD exchange rate (free, no auth, anonymous)
3. **Scryfall image CDN** — to display card images when you open the detail view

No analytics, no accounts, no telemetry. Everything else stays on your device.
