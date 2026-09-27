# My.Crypto v2

A private-by-design crypto portfolio tracker: effective cost, current value and P/L, running entirely on your device. A minimal, mobile-first PWA with live prices, EUR/PLN FX, a trend-colored value chart with range filters, per-asset sparklines and price alerts — backed by a strict-FIFO cost engine in JavaScript.

**Live site:** https://54y7jys5zr-coder.github.io/My.Crypto-v2/

## Features

- **Overview** — live portfolio value (All Holdings / Invested Only), Net invested + Realized stats, and an inline 365-day value chart with **1Y / 1M / 1W range filter**. The line and gradient are colored by trend (green/red) and tapping the chart shows the date and that day's change in value. Stats include current value, range high/low and best/worst day.
- **Portfolio** — a searchable, sortable flat list (by value, profit/loss, return % or name) with the current market price beside each quantity (e.g. `11 | €63.22`). Tapping an asset opens a detail sheet with a **30-day sparkline**, full cost breakdown and **price alerts** (bell badge on the row when a target is set, "hit" state when crossed).
- **Tables** — full cost-basis table with sortable columns (All Holdings / Invested Only), EUR-flows summary, **CSV export** and share-snapshot.
- **Settings** — CSV import (Balances / Ledger / Trades) feeding a **strict-FIFO engine** that recomputes everything locally; text-size and auto-refresh controls.
- **Auto-refresh** every 15s + manual refresh; live CoinGecko prices, EUR/PLN FX from ECB data.
- **Offline-capable PWA** — installable to the home screen, works without a connection, keeps your imported data.

## Run locally

```bash
python3 -m http.server 8000
```

Open http://localhost:8000 — the service worker needs localhost (or HTTPS).

## Deploy on GitHub Pages

1. Push this folder to the root of a GitHub repo's `main` branch.
2. Repo **Settings → Pages** → Source: `main` branch, folder `/ (root)`.
3. Open https://\<user\>.github.io/\<repo\>/ — HTTPS is required for PWAs.

### Install on iPhone (Safari)

- Open the URL in **Safari** → **Share** → **Add to Home Screen**.
- Launch from the home-screen icon: runs full-screen, works offline, keeps your imported data.
- Note (iOS): installed PWAs get limited storage and unused data can be evicted after ~7 days. Keep your CSV exports as the source of truth.

## Files

- `index.html`, `styles.css`, `app.js` — UI (bottom nav, overview, portfolio, tables, settings)
- `engine.js` — strict-FIFO cost engine (JS)
- `chart.umd.min.js` — self-hosted charting library (no CDN)
- `data.json` — starter dataset (empty in the repo; import your own CSVs in Settings)
- `manifest.webmanifest`, `sw.js`, `icons/` — PWA install + offline shell

## Data / method notes

- Balances are ground truth; strict-FIFO builds the EUR cost basis; staking rewards, gifts and airdrops are zero-cost and dilute the average.
- Prices: CoinGecko (EUR). FX: frankfurter.dev (ECB rates) with a CoinGecko fallback.
- The value chart is an approximation — current holdings applied across historical daily prices.

**Analysis/accounting tool — not tax or investment advice.**

## Security posture

Static, client-side app with **no backend, no accounts, no secrets/API keys, and no PII**. All computation runs on your device; the only outbound calls are read-only price/FX lookups.

Hardening applied before deployment:

- **Content-Security-Policy** (meta tag): `default-src 'self'`; scripts only from self (no inline scripts, no eval); network limited to `api.coingecko.com` and `api.frankfurter.dev`.
- **Output escaping**: all untrusted strings (CSV asset symbols, API fields) are HTML-escaped before rendering, preventing DOM-XSS from a crafted CSV.
- **No CDN**: the chart library is self-hosted (no third-party script trust; fully offline).
- **URL encoding**: asset IDs are `encodeURIComponent`-ed into API URLs.
- **Service worker** caches only the app shell, never API responses.

What leaves your device: the CoinGecko coin IDs you hold (in the price query string) and a currency-pair request for EUR/PLN. Quantities, cost basis and P/L never leave the browser.

Residual considerations:

- Holdings and last prices are stored in the browser's `localStorage`/cache — anyone with access to the unlocked device could open the app. Don't keep secrets in it.
- GitHub Pages can't set HTTP security headers; the CSP ships via the meta tag. On a host that supports headers, also send CSP, `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer` and HSTS.
- Third-party APIs (CoinGecko, frankfurter.dev) are trusted for price data only; their responses are treated as untrusted input (escaped, numeric-parsed).

## License

MIT — see [LICENSE](LICENSE).