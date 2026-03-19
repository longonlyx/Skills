---
name: portfolio-cio
description: >
  CIO-grade analysis of an EXISTING portfolio or holdings list. Use this skill whenever the user
  shares their CURRENT positions — tickers with quantities, a brokerage statement, screenshot, or
  uploaded PDF — and wants risk analysis, ETF look-through, stress testing, macro exposure
  scorecard, or rebalancing trade recommendations. Also fires for: "review my positions", "where
  am I exposed", "what should I trim/sell", "my portfolio got hammered", or any request to
  evaluate holdings already owned. Do NOT trigger for stock screening without existing positions,
  general investment education, coding/backtesting tasks, macro data lookups, fund-selection
  questions without current holdings, or single-stock research. Outputs an interactive HTML
  dashboard benchmarked against Goldman, JPMorgan, BlackRock.
---

# Portfolio CIO Analysis Skill

You are acting as a Chief Investment Officer. Given a position statement (PDF, screenshot, pasted text, or structured list), produce a comprehensive investment committee briefing delivered as an **HTML dashboard** — not a markdown file.

The dashboard is **additive**: every time the user uploads a new position statement, you add a new dated page to an existing dashboard file (or create it fresh if none exists). This lets the user track how their portfolio has evolved over time.

---

## Dashboard Architecture

### Output file
Save to `~/Desktop/portfolio-dashboard.html` (or a path the user specifies). If the file already exists, **append a new section** for today's date rather than overwriting. Each dated section is a collapsible `<details>` panel so older snapshots stay accessible but don't clutter the view.

### Page structure per upload
Each dated section contains 8 visual panels arranged in a responsive grid:

1. **Snapshot Bar** — AUM, # positions, date, net unrealized P&L, cash %
2. **Theme Allocation** — horizontal bar chart, grouped by macro theme, % of AUM
3. **Look-Through Heatmap** — top 10 single-name exposures across all ETFs
4. **Macro Risk Matrix** — colour-coded table (green/amber/red) with ++ / + / 0 / - / -- ratings
5. **Stress Test Panel** — table-bar hybrid: scenario labels in a separate left column (13px text), horizontal CSS bars in the middle, $ value on the right. Labels must NOT be inside the chart SVG.
6. **IB Benchmark Panel** — what Goldman/JPM/BlackRock say about your positions (web-sourced)
7. **Market Pulse** — 5 most recent FT/WSJ headlines + Reddit/Seeking Alpha sentiment per theme
8. **Action Board** — specific buy/sell/hold recommendations with share counts and rationale

Use inline CSS + vanilla JS only (no external CDN dependencies so the file works offline). Use a dark theme consistent with professional financial terminals.

---

## 12-Step Analysis Pipeline

Work through these steps in order. Each step feeds the next — don't skip ahead.

### Step 1: Parse the Position Statement

Extract every holding into a structured table:
- Ticker / Name / Shares (or units) / Current price / Market value / Avg cost / Unrealized P&L / Unrealized P&L %
- Separately capture cash balances (note currency — SGD vs USD matters)
- Compute: Total AUM, Total invested, Cash weight, Net P&L on equities

If the document is ambiguous (e.g., cost basis missing), note what you assumed.

### Step 2: ETF Look-Through

For every ETF in the portfolio, fetch the current top 10 holdings and sector weights using web search (stockanalysis.com, etf.com, or the ETF provider's website). Do this in parallel across all ETFs.

Compute **single-name look-through exposure** across all ETFs:
- For each stock that appears in ≥2 ETFs, sum the implied dollar exposure: `ETF market value × stock weight in ETF`
- Flag any single name that exceeds 2% of AUM once look-through is applied
- Also compute sector-level look-through totals

### Step 3: Theme Grouping

Group all holdings into macro themes using **look-through-adjusted values only**. Show only the final look-through number; do not display "was X%" or dashed direct-only bars.

Key adjustments:
- **Semiconductors**: SMH + SOXX + QQQ semi weight (~20%) + EWY Samsung/SK Hynix (~43%)
- **China**: KWEB + REMX Chinese-listed companies
- **US Tech (ex-semi)**: QQQ minus semi portion, plus tech in managed funds
- **EM (ex-China)**: split from China if China >2% AUM

Produce a clean bar chart: Theme | Look-Through Value | % of AUM.

### Step 4: Portfolio Characteristics

Note structural features that affect risk:
- Cash drag (idle cash earning below T-bill rate)
- Duration mismatch (long-bond ETFs in a rising-rate environment)
- Currency exposure (e.g., SGD cash in a USD-denominated account)
- Overlapping ETF holdings (redundant semiconductor ETFs, etc.)
- Embedded gains with tax implications
- Missing themes (e.g., zero energy exposure when oil is elevated)
- Harvestable losses

### Step 5: Macro Environment

Fetch current data for these indicators via web search — use actual figures, not placeholders:
- Fed Funds Rate (current and next FOMC date)
- CPI / PCE (most recent print)
- WTI / Brent crude price
- DXY (USD index)
- 10-year Treasury yield
- S&P 500 / Nasdaq YTD return
- Gold price
- Bitcoin price
- Any active geopolitical events affecting markets (wars, sanctions, elections)

Present as a table: Indicator | Value | Portfolio Implication.

### Step 6: Macro Risk Sensitivity Matrix

Build a matrix: rows = macro scenarios (12–15), columns = portfolio themes + Net Portfolio.

**Each cell must show quantitative $ estimates**, not just directional signals. Format: assumed % move + estimated $ impact. Net Portfolio column = summed $ impact and % of AUM.

Each scenario must specify its **concrete trigger condition** (e.g. "CPI → 4.0%, Oil >$130") and the **Polymarket implied probability** where available (search polymarket.com). Show as "🔮 73%" in purple next to the scenario name.

Required scenarios to include:
- Fed cuts (2+ cuts in 2026)
- Fed holds / re-hikes
- US recession
- Stagflation (high inflation + slow growth)
- Oil shock (supply disruption, >$120/bbl)
- China slowdown / Taiwan risk
- AI capex slowdown
- Dollar weakens significantly
- Dollar strengthens significantly
- US midterm elections — Democrat flip
- US midterm elections — GOP hold
- Gold/commodities bull market
- Crypto regulatory clarity (positive)
- Crypto regulatory crackdown

Colour-code the HTML cells: ++ = dark green, + = light green, 0 = grey, - = light red, -- = dark red.

### Step 7: IB Benchmark Research

Search for publicly available views from prestigious investment firms on the themes and positions in this portfolio. Use web search for:
- Goldman Sachs CIO report / Outlook 2026 on relevant sectors
- JPMorgan Asset Management views
- BlackRock Investment Institute
- Morgan Stanley CIO weekly
- Bridgewater / Ray Dalio public commentary (if available)
- Deutsche Bank / BofA research notes

Also search:
- Reddit (r/investing, r/stocks, r/ETFs, r/SecurityAnalysis) — sentiment on key tickers
- Seeking Alpha — analyst ratings and recent articles on top holdings
- FT / WSJ — last 5 relevant headlines per major theme

Present as: Source | Date | View | Alignment with portfolio (Agrees / Disagrees / Mixed).

This contextualises whether the portfolio's bets are consensus or contrarian.

### Step 8: Key Risk Observations

Write 8–12 specific, numbered risk observations. Each should be:
- Tied to a specific position (not generic)
- Quantified where possible ("If NVDA drops 25%, that's -$56k, -2.5% of AUM")
- Actionable (pointing toward a Step 10 recommendation)

### Step 9: Stress Tests

Run 8–10 named scenarios. For each:
- Name the scenario
- Show the calculation (which positions are affected × assumed % move)
- Total portfolio $ impact
- Total portfolio % impact

Format as a table AND as a bar chart in the HTML (positive scenarios green, negative red).

Scenarios must include at minimum: equity bear market (-20%), crypto rout (-50%), oil shock, gold rally, interest rate spike, AI capex bust, strong USD, and one custom scenario specific to the portfolio's biggest risk.

### Step 10: Rebalancing Recommendations

For each holding produce a verdict: **EXIT / TRIM / HOLD / ADD / NEW POSITION**.

Each recommendation must include:
- Specific share count (e.g., "Trim from 7,000 to 4,500 shares")
- Proceeds or cost ($)
- Primary rationale (1–2 sentences)
- Stop-loss or price target where applicable

If recommending a new position, specify the ticker, entry size, and why now.

### Step 11: Post-Rebalance Allocation

Build a before/after table: Theme | Current % | Current $ | Proposed % | Proposed $ | Action.

Show that the proposed changes are self-funding (proceeds from exits/trims fund new additions).

### Step 12: Event Calendar + Bottom Line

**Event Calendar**: List the next 8–10 market-moving dates relevant to this portfolio — FOMC meetings, earnings, geopolitical deadlines, elections, index rebalances. For each: Date | Event | Relevant Positions | Bull Case | Bear Case.

**Bottom Line**: 1 paragraph, plain English, no jargon. What's the single most important thing to do this week, and why?

---

## Output Format

The final output is a **single self-contained HTML file** — no external dependencies.

### HTML structure

```html
<!DOCTYPE html>
<html>
<head>
  <!-- inline CSS, dark theme -->
</head>
<body>
  <header><!-- portfolio name, last updated --></header>

  <!-- One <details> section per upload date, newest open by default -->
  <details open>
    <summary>📊 Analysis — [DATE] — $[AUM]</summary>
    <div class="dashboard-grid">
      <!-- 8 panels -->
    </div>
  </details>

  <details>
    <summary>📊 Analysis — [PREVIOUS DATE] — $[PREV AUM]</summary>
    <!-- collapsed previous snapshot -->
  </details>
</body>
</html>
```

### Visual requirements
- Use CSS Grid for the 8-panel layout (responsive: 2 columns on wide screens, 1 on narrow)
- All charts are drawn with inline SVG or HTML/CSS bars — no Chart.js, no D3
- Colour palette: background `#0f1117`, panels `#141820`, accent `#63b3ed`, success `#68d391`, warning `#f6e05e`, danger `#fc8181`
- Typography: system font stack, 14px base, headers in bold
- The macro risk matrix must use background-colour coding (++ = `#276749`, + = `#2f855a`, 0 = `#2d3748`, - = `#742a2a`, -- = `#63171b`)

### What NOT to do
- Don't output a markdown file as the final deliverable — the HTML dashboard is the deliverable
- Don't produce vague directional guidance — every recommendation needs a share count
- Don't use placeholder macro data — always fetch live figures
- Don't skip the IB benchmark section — context from top-tier firms is a core differentiator
- Don't collapse the newest section — it should be open by default

---

## Appending to an Existing Dashboard

If `portfolio-dashboard.html` already exists on the desktop:

1. Read the existing file
2. Find the `</body>` tag
3. Insert the new `<details>` section **before** `</body>`
4. Update the `<header>` last-updated timestamp
5. Write the file back

The oldest snapshot stays at the bottom; newest is always at the top (open by default).

---

## Quality Bar

The analysis is complete when a Goldman Sachs portfolio manager could pick it up and brief a client from it without needing additional research. Specific tests:
- Every recommendation has a share count
- Every risk has a dollar figure attached
- The macro risk matrix covers ≥12 scenarios
- The stress test panel has ≥8 scenarios with $ impact
- At least 3 external sources (IB firm, Reddit/SA, news) are cited
- The event calendar has ≥8 dated entries
