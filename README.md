# My.Crypto

A private-by-design portfolio tracker for crypto: effective cost, current value and P/L, running entirely on your device. Installable PWA with live prices, an EUR/PLN toggle, cards + tables, a 365-day value chart, and a full strict-FIFO cost engine in JavaScript.

**Live site:** https://54y7jys5zr-coder.github.io/My.Crypto/

## Features
- **Summary** — portfolio value, unrealised and lifetime realised P/L; All Holdings vs Invested Only views.
- **Cards** — one card per asset, colour-coded by value ÷ cost (heatmapped like tickers).
- **Tables** — cost basis, value, unrealised P/L and return %, plus an EUR-flows summary.
- **Chart** — 365-day portfolio value vs cost basis (approx: current holdings × historical daily prices), with 365-day change / high / low.
- **Auto-refresh** every 15s + manual refresh; live CoinGecko prices, EUR/PLN FX from ECB data.
- **CSV import** in Settings — Balances / Ledger / Trades exports; a strict-FIFO engine recomputes everything locally.
- **Offline-capable PWA** — installable to the home screen, works without a connection.

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
- `index.html`, `styles.css`, `app.js` — UI (bottom nav, summary, cards, tables, chart, settings)
- `engine.js` — strict-FIFO cost engine (JS)
- `chart.umd.min.js` — self-hosted charting library (no CDN)
- `data.json` — starter dataset (empty in the repo; import your own CSVs in Settings)
- `manifest.webmanifest`, `sw.js`, `icons/` — PWA install + offline shell

## Data / method notes
- Balances are ground truth; strict-FIFO builds the EUR cost basis; staking rewards, gifts and airdrops are zero-cost and dilute the average.
- Prices: CoinGecko (EUR). FX: frankfurter.dev (ECB rates) with a CoinGecko fallback.
- The 365-day chart is an approximation — current holdings applied across historical daily prices.

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
