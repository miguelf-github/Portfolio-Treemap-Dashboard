# Portfolio Treemap Dashboard

A self-contained, live-updating portfolio treemap dashboard. Pulls data from
Google Sheets in real time and renders holdings as proportionally-sized,
color-coded pills grouped by sleeve — with hover tooltips, dark mode, and
scorecards for value, gain, dividends, and fees.

**No build tools, no dependencies — just one HTML file.**

## 🔴 Live demo

Once GitHub Pages is enabled for this repo (Settings → Pages → Deploy from
branch `main`), your dashboard will be live at:

```
https://miguelf-github.github.io/portfolio-treemap/
```

## Features

- Row-based group layout — each sleeve gets a dark header row with return %,
  weight %, and dollar value
- Holdings shown as pills sized proportionally by weight
- Hover tooltips: weight, value, cost basis, return, dividends, fees
- Scorecards: Total Value, Total Invested, Total Gain, Est. Annual Dividends,
  Annual Fees, Positions
- Multi-account tabs
- Dark mode via CSS variables
- Live data via Google Sheets `gviz/tq` endpoint (JSONP, no proxy/server needed)

## Setting up your own data

Open `index.html` and edit the `ACCOUNTS` array near the top of the `<script>`:

```js
const ACCOUNTS = [
  { name: "Traditional IRA", sheetId: "YOUR_SHEET_ID", gid: "YOUR_TAB_GID" },
];
```

- **sheetId**: the string between `/d/` and `/edit` in your Google Sheets URL
- **gid**: the number after `gid=` in the URL when you're on the correct tab

Your sheet must be shared as **"Anyone with the link can view"** (Share button —
not "Publish to web") for the live connection to work.

### Expected columns (A–O)

`Description | Ticker | Weight | Shares | Worth | Avg Cost | Div Yield | Div Growth | Type | Dividend | Link | Return | Account | Expense Ratio | Order`

> ⚠️ **Note on privacy:** this repo is public, so your sheet ID/GID will be
> visible to anyone who views the source. The actual data is only protected
> by your Google Sheet's own sharing settings — keep those set to "Anyone
> with link can view" (read-only) rather than edit access, and don't put
> anything in the sheet you wouldn't want a determined viewer to see.

## License

Personal project — use and adapt freely.
