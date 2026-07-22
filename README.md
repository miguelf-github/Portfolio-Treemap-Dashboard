# Google Sheets Stock Portfolio Treemap Dashboard

A self-contained, live-updating portfolio treemap dashboard. Pulls data from
Google Sheets in real time and renders holdings as proportionally-sized,
color-coded pills grouped by sleeve with hover tooltips, dark mode, and
scorecards for value, gain, dividends, and fees.

**No build tools, no dependencies — just one HTML file.**

## 🔴 Live demo

Once GitHub Pages is enabled for this repo (Settings → Pages → Deploy from
branch `main`), your dashboard will be live at:

```
https://miguelf-github.github.io/Portfolio-Treemap-Dashboard/
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

`Description of Sleeve | Ticker | Weight % | Shares | Position Worth | Position Cost | Div Yield % | Div Growth % | Type (etf/stock/mutf) | Projected Annual Dividend | Stock Analysis Link | Return % | Account name | Expense Ratio | Order by number`


### Sheet columns & formulas (row 2 example)
 
Set up a header row (row 1), then row 2 onward is one holding per row. `Q16`
is a running total cell (see note below) and `R14` is a manual cash-balance
cell — adjust those references to wherever you keep totals in your own sheet.
 
| Cell | Column | What it is | Formula / value |
|---|---|---|---|
| A2 | Description | Sleeve/group name | typed manually |
| B2 | Ticker | Stock/ETF ticker | typed manually |
| C2 | Weight | % of total account | `=E2/$Q$16` |
| D2 | Shares | # of shares held | typed manually |
| E2 | Worth | Current market value | `=(GOOGLEFINANCE(B2)*D2)` |
| F2 | Avg Cost | Cost basis (avg share price × shares) | typed manually, e.g. `=(236.72*D2)` |
| G2 | Div Yield | Dividend yield | typed manually, or `=IFERROR("$4.88"/GOOGLEFINANCE(B2,"price"),"0.00%")` (swap in the annual $ dividend per share) |
| H2 | Div Growth | Dividend growth rate | typed manually |
| I2 | Type | Security type — used to build the research link | one of: `mutf`, `etf`, `stocks` |
| J2 | Dividend | Projected annual dividend $ for the position | `=(E2*G2)` |
| K2 | Link | Research link | `=IF(I2="mutf","https://stockanalysis.com/quote/"&I2&"/"&B2&"/dividend/","https://stockanalysis.com/"&I2&"/"&B2&"/dividend/")` |
| L2 | Return | % gain/loss | `=(E2-F2)/F2` |
| M2 | Account | Where it's held (optional, e.g. Robinhood, Fidelity) | typed manually |
| N2 | Expense Ratio | Annual fee $ (pre-calculated, not a %) | `=IFERROR(E2*0.02%,"")` |
| O2 | Order | Sort order for sleeve grouping in the dashboard | typed manually, a number |
 
**Supporting total cells** (adjust locations to your sheet):
- `Q16` — total account value used for weighting: `=SUM($E$2:$E)+R14`
- `R14` — manual entry for any un-invested cash balance in the account
> The dashboard reads `Expense Ratio` (N) as an already-calculated dollar
> amount, not a percentage — so don't multiply it again against worth in
> any downstream formulas.


> ⚠️ **Note on privacy:** this repo is public, so your sheet ID/GID will be
> visible to anyone who views the source. The actual data is only protected
> by your Google Sheet's own sharing settings — keep those set to "Anyone
> with link can view" (read-only) rather than edit access, and don't put
> anything in the sheet you wouldn't want a determined viewer to see.

## License

Personal project — use and adapt freely.
