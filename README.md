# BRI Sydney DC — Daily Resource Planner

A single-file HTML tool that calculates daily picking/packing/admin headcount requirements from the WMS Outstanding Orders Report, based on agreed customer KPI rates.

## What it does

Upload the daily "Outstanding Orders Report" export (xlsx) from the WMS, and the planner:

- Calculates required headcount (in people) for each KPI-mapped customer, broken down by activity (Pick / Pack / Admin)
- Flags overdue orders vs orders due the next business day
- Tracks variance between KPI-calculated headcount and actual rostered headcount, with a daily/annual cost estimate for over-rostering
- Surfaces any customer codes in the file that have **no** KPI mapping, so their labour demand isn't silently missed
- Lets the Ops Manager enter manual hours/quantities for unmapped customers or activities where the WMS data doesn't match the KPI unit of measure (e.g. Flying Tiger cartons vs lines)
- Shows a 5-day forward pipeline of orders due by customer
- Live countdown to the 4:00 PM agency confirmation deadline

## Mapped customers (as of this build)

| Customer | WMS code | Activities |
|---|---|---|
| BBW EComm | AVRBBW | Fulfilment pick |
| TBS | TBS | Pick, Pack |
| Ten Tops | TENTSYD-WH | High-reach wave pull-down, Carton build (manual entry), Admin |
| BBW B2B | AVBBBW | Pick |
| Makita | MAKAUSSYD | Batch pick, Pack station, Labels, Pallet wrap |
| Flying Tiger | FLYTSYD-WH-F | Pre-Pick (high reach), Pick carton (manual override), Post-Pick (high reach) |

Any other customer code present in the file appears in the **Unmapped customers** table at the bottom of the Roster view, where hours can be entered manually to roll into the grand total.

## Running it

This is a static single HTML file — no build step, no server required.

- **Locally:** double-click `index.html` to open it in a browser
- **Hosted (GitHub Pages, etc.):** just serve the file as-is

The XLSX parsing library ([SheetJS](https://sheetjs.com)) loads from a CDN (`cdn.jsdelivr.net`). This requires the page to be served over `http(s)://` — it will **not** work if opened directly from disk as `file://` due to browser security restrictions on local files loading external scripts. If you need full offline/local-file support, the SheetJS library can be embedded inline instead.

## Updating KPI rates

All customer/task/rate definitions live in the `CUSTOMERS` array near the top of the `<script>` block in `index.html`. Each task has:

```js
{ id, team, label, driver, rate, unit, note }
```

- `driver` determines what WMS field feeds the calculation (`lines`, `orders`, `units`, `waveTasks`, `estCartons`, `estPallets`, or `manualEntry` for tasks requiring an operator-entered quantity)
- `rate` is units-per-hour from the agreed KPI sheet
- Rates can also be overridden live in the UI per session (stored in browser memory only, not saved)

## Known data caveats

- **Flying Tiger** pick activity uses WMS lines as a stand-in for cartons unless the operator manually enters the actual carton count (the WMS report doesn't track cartons directly)
- **Ten Tops carton build** requires a mandatory manual carton count entry — there is no reliable WMS-derived estimate for this activity
- **TBS** picking uses a single "medium" rate as a proxy since the WMS report doesn't split pick complexity into small/medium/large

## File structure

Single file: `index.html` — HTML markup, CSS, and JavaScript all in one file for portability.
