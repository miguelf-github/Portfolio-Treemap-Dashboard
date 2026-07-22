---
name: portfolio-treemap
description: >
  Generates a self-contained HTML portfolio treemap dashboard from sleeve/position data.
  Use this skill whenever the user wants to visualize a portfolio, investment account, or
  set of holdings as a treemap — including Roth IRA, Traditional IRA, taxable brokerage,
  mom's portfolio, or any other account. Triggers on phrases like "create a treemap",
  "make a portfolio dashboard", "visualize my portfolio", "turn my holdings into a treemap",
  or "add live Google Sheets connection". Also use when the user pastes tabular position
  data with columns like Exposure, Description, Breakdown, Worth, Cost, Dividends, Returns,
  Fees — even if they don't explicitly say "treemap". Always produces a downloadable HTML
  file as the output.
---

# Portfolio Treemap Skill

Generates a fully self-contained, legible HTML treemap dashboard for any investment portfolio.
Matches an established design system across all of the user's portfolios.

## Design System (non-negotiable — do not deviate)

- **Layout**: Row-based CSS flex layout. Each sleeve/group gets a dark-background row.
  Holdings inside each row are colored pills sized with `flex` proportional to weight.
  Legibility over perfect proportionality — every pill must show ticker + weight %.
- **Scorecards**: Total Value, Total Invested, Total Gain (color-coded), Est. Annual
  Dividends (with yield %), Annual Fees, Positions count.
- **Group header**: group name · return badge (green/red) · weight % · dollar value
- **Holding pills**: ticker (13px bold) + weight % (11px muted) stacked. Min-width 52px.
  Opacity scaled by relative weight within the group (0.55–1.0).
- **Tooltips**: Fixed-position on hover — weight, value, cost basis, return, dividends, fees.
- **Dark mode**: Full CSS variable support via `prefers-color-scheme: dark`.
- **Colors**: Positive returns → `#7ce3b0` (green). Negative → `#f7a5a5` (red).
  Scorecard gain → `.pos` (`#3B6D11`) or `.neg` (`#A32D2D`).
- **Output**: Single self-contained `.html` file. No external dependencies.

## Color Palette (use consistently across portfolios)

```
S&P 500 / broad market:    color #185FA5  bg #0C447C
Tech ETFs (VGT, QQQ):      color #534AB7  bg #3C3489
MAG 7 / growth stocks:     color #7F77DD  bg #534AB7
Speculative / ARKQ / QTUM: color #993C1D  bg #712B13
Dividend / SCHD:           color #3B6D11  bg #27500A
International:             color #0F6E56  bg #085041
REIT & BDC:                color #1D9E75  bg #0F6E56
Energy / infrastructure:   color #854F0B  bg #633806
Bonds / cash:              color #5F5E5A  bg #444441
Precious metals:           color #888780  bg #5F5E5A
```

For any ticker not in the palette, cycle through:
`["#185FA5","#3B6D11","#993C1D","#534AB7","#854F0B","#0F6E56","#5F5E5A"]`

## Step-by-step workflow

### Step 1 — Collect inputs

You need three things. Ask for any that are missing:

1. **Portfolio data** — the sleeve table (see Input Format below)
2. **Google Sheet ID** — the string between `/d/` and `/edit` in the sheet URL
   e.g. `1QXxJ-rH0fX6qhJXTXo-Xh5h5gmDvZXfgZqal2hzd-m0`
3. **Sheet GID** — the number after `gid=` in the URL when on the correct tab
   e.g. `374061404`

If the user wants a **static file only** (no live data), skip Steps 2–3 and hardcode the data.

### Step 2 — Parse the input data

Input format (tab or pipe separated):
```
Exposure | Description | Breakdown | Worth | Cost | Dividends | Returns | Fees
```

- **Exposure**: portfolio weight % (e.g. `29.46%`)
- **Description**: sleeve/group name (e.g. `S&P 500`)
- **Breakdown**: holdings with weights (e.g. `VOO (12.06%), QQQM (3.73%)`)
- **Worth**: current market value
- **Cost**: total cost basis
- **Dividends**: estimated annual dividends
- **Returns**: total return % (e.g. `3.45%`)
- **Fees**: annual fees in dollars (already pre-calculated — do NOT multiply)

Parse the Breakdown field to extract individual holdings:
```js
// "VOO (12.06%), QQQM (3.73%)" → [{t:"VOO", p:12.06, w: worth * 12.06/totalPct}, ...]
```

### Step 3 — Build the HTML

Use this exact structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- CSS variables for light/dark mode (see CSS section below) -->
</head>
<body>
  <h1>[Portfolio Name]</h1>
  <p class="subtitle">All groups · hover any position for details</p>

  <div class="cards" id="scorecards"></div>
  <div class="divider"></div>
  <div class="tm-grid" id="grid"></div>
  <div class="legend-row" id="legend"></div>
  <div class="tip" id="tip"></div>

  <script>
    const groups = [ /* parsed data */ ];
    /* scorecard render, group render, tooltip logic */
  </script>
</body>
</html>
```

**For live Google Sheets version**, add above the body:
```html
<div class="header">
  <div><h1>[Name]</h1><p class="subtitle" id="subtitle">Loading…</p></div>
  <button class="refresh-btn" id="refreshBtn" onclick="loadData()">
    <span class="spin">↻</span> Refresh
  </button>
</div>
```

### Step 4 — Wire up Google Sheets (live version)

Use the `gviz/tq` JSONP endpoint — **no proxy needed**, works in any browser:

```js
const SHEET_ID  = "[USER'S SHEET ID]";
const SHEET_GID = "[USER'S GID]";

async function loadData() {
  const callbackName = 'gvizCallback_' + Date.now();
  const url = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?gid=${SHEET_GID}&tqx=out:json,responseHandler:${callbackName}`;

  const data = await new Promise((resolve, reject) => {
    const script = document.createElement('script');
    const timer = setTimeout(() => { reject(new Error("Timed out")); cleanup(); }, 10000);
    window[callbackName] = (d) => { resolve(d); cleanup(); };
    function cleanup() { clearTimeout(timer); delete window[callbackName]; document.head.removeChild(script); }
    script.onerror = () => { reject(new Error("Cannot reach sheet — set sharing to 'Anyone with link can view'")); cleanup(); };
    script.src = url;
    document.head.appendChild(script);
  });

  const rows = gvizToRows(data); // see below
  render(rows);
}

function gvizToRows(data) {
  const cols = data.table.cols.map(c => c.label);
  return data.table.rows.map(r => {
    const obj = {};
    r.c.forEach((cell, i) => { obj[cols[i]] = cell ? (cell.v ?? cell.f ?? "") : ""; });
    return obj;
  }).filter(r => r["Ticker"] && r["Worth"]);
}
```

**Critical fee rule**: The `Expense Ratio` column in the sheet is already a pre-calculated
dollar amount (not a ratio). Read it directly:
```js
const fees = parseFloat(r["Expense Ratio"]) || 0;  // ✅
// NOT: worth * parseFloat(r["Expense Ratio"])      // ❌ gives millions
```

**Div yield rule**: `Div Yield` comes as a decimal (e.g. `0.0102` = 1.02%). Multiply by worth:
```js
const divs = worth * (parseFloat(r["Div Yield"]) || 0);
```

**Return rule**: `Return` comes as a decimal (e.g. `0.0345` = 3.45%). Multiply by 100 for display:
```js
const ret = parseFloat(r["Return"]) || 0;
// Display: retStr(ret) → `${(ret*100).toFixed(2)}%`
```

**Weight rule**: `Weight` comes as a decimal (e.g. `0.2946` = 29.46%):
```js
const pct = parseFloat(r["Weight"]) || 0;
// Display: `${(pct*100).toFixed(1)}%`
// Holding pill: h.p = pct * 100
```

### Step 5 — Key CSS to include verbatim

```css
:root {
  --color-background-primary: #ffffff;
  --color-background-secondary: #f4f3ef;
  --color-text-primary: #1a1a18;
  --color-text-secondary: #6b6b66;
  --color-border-tertiary: rgba(0,0,0,0.12);
  --color-border-secondary: rgba(0,0,0,0.22);
  --border-radius-md: 8px;
  --font-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}
@media (prefers-color-scheme: dark) {
  :root {
    --color-background-primary: #1c1c1a;
    --color-background-secondary: #252523;
    --color-text-primary: #f0efe9;
    --color-text-secondary: #9a9990;
    --color-border-tertiary: rgba(255,255,255,0.1);
    --color-border-secondary: rgba(255,255,255,0.2);
  }
}
.pos { color: #3B6D11; }
.neg { color: #A32D2D; }
.ret-pos { color: #7ce3b0; }
.ret-neg { color: #f7a5a5; }
```

### Step 6 — Output the file

Save to `/mnt/user-data/outputs/[portfolio_name]_treemap.html` and call `present_files`.

Naming convention:
- `roth_ira_treemap.html`
- `traditional_ira_treemap_live.html` (live = has Google Sheets connection)
- `moms_portfolio_treemap.html`
- `taxable_treemap_live.html`

## Google Sheets sharing requirements

The sheet must be shared as **"Anyone with the link can view"** (Share button → Change access).
This is different from "Publish to web". Remind the user of this if they hit auth errors.

## Reference file — gold standard

`assets/reference_treemap.html` is the **gold standard / ground truth** for this skill. It is
Miguel's live multi-account (Roth IRA + Traditional IRA) treemap dashboard. When any instruction
above is ambiguous, or a new dashboard needs to match pixel-for-pixel, **defer to this file over
the prose in this SKILL.md** — copy its CSS, JS, and markup patterns directly rather than
re-deriving them. Notable patterns in this reference that supersede earlier/simpler examples
in this doc:

- **Multi-account tabs**: `ACCOUNTS` config array (name, sheetId, gid) at the top of the script,
  rendered as pill tabs via `renderTabs()`/`switchAccount()`. Only shown when there's more than
  one account. Use this pattern whenever the user has multiple portfolios in one file.
- **Fixed-position column reads (not header-based)**: `gvizToRows()` reads columns by index
  (A–O) via `c[i].v ?? c[i].f ?? ""`, not by matching header label text. This avoids gviz
  header-detection quirks. Column order: Description, Ticker, Weight, Shares, Worth, Avg Cost,
  Div Yield, Div Growth, Type, Dividend, Link, Return, Account (skip), Expense Ratio, Order.
- **Order column for color + sort stability**: an `Order` column (1–12+) in the sheet maps to
  `ORDER_COLORS`, so a given slot number always renders the same color and groups always sort
  in the same order — independent of sleeve name or row position. Prefer this over name-based
  color matching when the sheet has an Order column available.
- **Forward-fill blank Description cells**: sleeve names that are visually merged across rows
  in the sheet leave underlying cells blank — forward-fill with `lastSleeve` and normalize
  whitespace before grouping, so rows group correctly.
- **Weighted-average sleeve return**: sleeve-level return badge is a dollar-weighted average of
  its holdings' returns, not a simple average.
- **Scrollable holdings row**: when a group has more than 5 holdings, the row switches to
  horizontal scroll (`.holdings-row.scrollable`) with fixed-width pills, instead of squeezing
  every pill into view.

Use it as ground truth for layout, JS logic, and CSS when in doubt.
