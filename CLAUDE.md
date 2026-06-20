# CLAUDE.md — SG Retirement Planner

## Project overview

A standalone single-page retirement planning tool for Singapore. Helps users answer the question: **"If I want to retire at age X, what annual investment return do I need?"**

Completely separate from the AI × Humanity project.

**GitHub:** https://github.com/kacwq/sg-investment-planner  
**Live:** https://sg-investment-planner.pages.dev  
**Branch:** master  
**Stack:** Vanilla HTML + CSS + JS. Single file (`index.html`). No frameworks, no build step.

---

## File structure

```
index.html     — The entire app (HTML + embedded CSS + embedded JS)
.gitignore     — Ignores .wrangler/, .claude/, OS files
CLAUDE.md      — This file
```

---

## Deployment workflow

**After every meaningful change:**
1. `git add index.html`
2. `git commit -m "Imperative message describing the why"`
3. `git push origin master`
4. `npx wrangler pages deploy . --project-name sg-investment-planner --commit-dirty=true`

All four steps every time. Cloudflare does not auto-deploy from GitHub (not connected) — wrangler deploy is required.

---

## What the tool does

### Input (left panel — 3-step flow)
1. **Goal:** retirement age, monthly spending target in retirement
2. **Where you are:** current age, investment savings (liquid), CPF balance (OA+SA)
3. **Monthly actions:** monthly investment savings, monthly CPF contributions
4. **Advanced mode (collapsed):** per-bucket breakdown — CPF OA, SA, SRS, Stocks/ETFs, SSB/T-bills, return rates

### Output (right panel)
- **Hero:** Required annual investment return % (primary answer)
- **Context label:** rates like "Very conservative" / "Moderate — global ETF range" / "Very aggressive"
- **Target wealth:** monthly spend × 12 × 25 (4% safe withdrawal rule)
- **Gap card:** "At 7% (ETF average), you'd retire at age ___"
- **Chart:** projected wealth vs target line, green/red shading
- **Milestone cards:** retirement age, age 55 (CPF RA), age 65 (CPF LIFE)
- **"How is this calculated?"** expandable section with formula, year-by-year table, CPF timeline

### CPF explainers
Every CPF-related field has an expandable `<details>` tooltip explaining in plain English what the account is, the interest rate, and how it behaves.

---

## Calculation engine

### Key functions (in `<script>` at bottom of `index.html`)

| Function | Purpose |
|---|---|
| `project(investRate)` | Year-by-year simulation from currentAge to 90. Returns array of `{age, cpfOA, cpfSA, cpfRA, stocks, srs, ssb, liquid, total, cpfLifePayout}` |
| `solveForReturnRate(target, retireAge)` | Binary search on invest rate (0–30%) until projected total at retireAge ≥ target. Returns `null` if >30% needed. |
| `solveForRetireAge(rate, target)` | Given a fixed rate (e.g. 7%), finds earliest age where projected total ≥ target |
| `targetWealth()` | `monthlySpend × 12 × 25` |
| `rateContext(rate)` | Returns label + colour for the required rate |
| `getInputs()` | Resolves simple-mode inputs into per-bucket `{bal, con}` objects, with advanced-mode overrides |
| `recalc()` | Main entry point — calls solvers, updates all DOM |
| `renderChart(years, target, retireAge)` | Chart.js line chart (projected vs target) |
| `renderMilestones(years, target, ...)` | Generates milestone snapshot cards |
| `renderYearTable(years, retireAge)` | Year-by-year table in the "How is it calculated?" section |

### CPF rules modelled
- OA earns 2.5% p.a., SA earns 4% p.a. (fixed, government-set)
- At age 55: OA + SA merge into Retirement Account (RA) up to FRS (~$213K in 2025, grows 3.5%/yr)
- Excess above FRS stays in accessible OA
- At age 65: CPF LIFE payout estimated at `RA × (1500 / 213000)` per month (proportional to RA balance)
- If retiring before 55: CPF balance is locked — a warning banner appears

### Simple mode input mapping
| Simple field | Maps to |
|---|---|
| `currentInvest` | `bal.stocks` |
| `currentCPF` | `bal.cpfOA` (79.3%) + `bal.cpfSA` (20.7%) |
| `monthlyInvest` | `con.stocks` |
| `monthlyCPF` | `con.cpfOA` (79.3%) + `con.cpfSA` (20.7%) |

Split ratio: 23/29 OA, 6/29 SA (based on standard employee+employer CPF allocation for under-55).

---

## Design system

Shared CSS variables (same palette as the AI × Humanity site for consistency):

```css
--bg: #F4F5F9        /* page background */
--surface: #FFFFFF   /* cards */
--border: #E2E5EF
--text: #1A1A2E
--muted: #6B7280
--accent: #7C3AED    /* purple */
--green: #10B981
--red: #EF4444
--amber: #F59E0B
```

Light mode (financial data reads better on white). Inter font, weights 300–800.

---

## What's built vs. placeholder

| Feature | Status |
|---|---|
| Goal-first input flow | ✅ Done |
| Required return rate solver | ✅ Done |
| 7% benchmark gap card | ✅ Done |
| Chart (projected vs target) | ✅ Done |
| CPF plain-English explainers | ✅ Done |
| Advanced per-bucket mode | ✅ Done |
| Year-by-year calculation table | ✅ Done |
| CPF events timeline | ✅ Done |
| Milestone cards (retire age, 55, 65) | ✅ Done |
| Early-retirement CPF lock warning | ✅ Done |
| Custom milestone support | Not built |
| SRS-specific tax relief modelling | Not built |
| Inflation adjustment | Not built |
| Mobile layout | ✅ Responsive (stacks on <900px) |

---

## Git conventions

- Imperative commit subject line (e.g. "Add inflation toggle to projection engine")
- Body explains *why*, not *what*
- All commits include `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`
- Push + wrangler deploy after every meaningful change
