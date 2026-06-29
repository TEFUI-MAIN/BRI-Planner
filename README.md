# BRI Sydney DC — Daily Resource Planner

A browser-based HTML tool that calculates daily DC labour requirements from the WMS Outstanding Orders Report, based on agreed customer KPI rates and operator-entered manual activity.

## What it does

Upload the daily "Outstanding Orders Report" export (xlsx) from the WMS, and the planner:

- Calculates required labour hours and headcount for KPI-mapped customers.
- Flags overdue orders, review-required orders, and unmapped customer codes in the left sidebar.
- Provides clickable status/count buttons throughout the app so operators can drill into the order rows behind each number.
- Lets operators enter manual hours for unmapped customers; completed unmapped entries turn green and roll into the grand total.
- Captures additional manual pallet activity for inbound unload, inbound receipt, inbound putaway, outbound loading, and container unload.
- Shows a 5-day order pipeline, including future work beyond the immediate planning window.
- Generates a printable **Print Plan** report with mapped customers, unmapped customers, manual pallet activities, team leaders, and final headcount.

## Mapped customers (as of this build)

| Customer | WMS code | Current planning logic |
|---|---|---|
| BBW B2B | AVBBBW | Orders, future orders, and picked-complete visibility with B2B-specific status grouping |
| Tory Burch | AVBTRB | Pick by line at 20 lines/hr |
| BBW EComm | AVRBBW | Pick open work, pack picked-complete/closed orders, and pack open orders |
| Flying Tiger | FLYTSYD-WH-F | Pre-pick, carton pick with manual carton override, and post-pick |
| Makita | MAKAUSSYD | Batch pick, Pack station, Labels, Pallet wrap |
| TBS | TBS | SO Orders, Transfers, and Return to Supplier with dedicated pill colours and KPI lines |
| Ten Tops | TENTSYD-WH | Store Orders, X-DOCK visibility, manual task/carton entry, and wrap pallets |

Any other customer code present in the file appears in the **Unmapped customers** table at the bottom of the Roster view. The sidebar unmapped alert also opens a quick A-Z summary; each order count can be clicked to see that customer code's order detail.

## Running it

The live planner is a static GitHub Pages app: `index.html` plus image assets in `assets/`.

- **Local test server:** `python3 -m http.server 5173 --directory /Users/admin/documents/github/bri-planner`
- **Local URL:** `http://localhost:5173/`
- **Hosted URL:** `https://tefui-main.github.io/BRI-Planner/`

The XLSX parsing library ([SheetJS](https://sheetjs.com)) loads from a CDN (`cdn.jsdelivr.net`). Serve the page over `http(s)://` for normal operation.

## Updating KPI rates

All customer/task/rate definitions live in the `CUSTOMERS` array near the top of the `<script>` block in `index.html`. Each task has:

```js
{ id, team, label, driver, rate, unit, note }
```

- `driver` determines what WMS field feeds the calculation (`lines`, `orders`, `units`, `pickedCompleteOrders`, `transferOrders`, `returnSupplierOrders`, `waveTasks`, `estCartons`, `estPallets`, `wrapPallets`, or `manualEntry` for tasks requiring an operator-entered quantity)
- `rate` is units-per-hour from the agreed KPI sheet
- Rates can also be overridden live in the UI per session (stored in browser memory only, not saved)

## Known data caveats

- **Flying Tiger** carton quantity is operator-entered because carton quantity is not reliably available from the WMS import.
- **Ten Tops** requires operator-entered total tasks and total cartons to pack. Wrap pallets are estimated from cartons using the current conversion in the customer task config.
- **TBS** uses dedicated treatment for `TFR_` transfer orders and `RS_` return-to-supplier orders.
- **BBW EComm** includes pack lines for both picked-complete/closed orders and open orders.
- **BBW B2B** totals include current orders, future orders, and picked-complete orders in the customer header.
- **Review alerts** currently cover WMS statuses that require operator review, such as `Did Not Allocate`.

## Print Plan report

After the operator finishes review/manual entries, the **Print Plan** tab generates a printable report with:

- Mapped customers first, A-Z by customer.
- Unmapped customers next, A-Z by customer code.
- Manual pallet activities.
- Grand total headcount, with Team Leaders always shown at the bottom of the category list before the final total.
- Source file, plan date, prepared timestamp, and productive-hours assumption.

The print stylesheet hides the planner UI and prints only the report content.

## Future automation notes

`docs/exception-report-plan.md` captures the optional future path for automated exception reporting. It is not required for the current GitHub Pages planner.

## File structure

- `index.html` - HTML markup, CSS, and JavaScript for the planner
- `assets/BRI-Icon-Circle.webp` - sidebar logo served with the GitHub Pages site
- `docs/exception-report-plan.md` - optional future automation decision brief
