# BRI Planner

Static labour planning dashboard for BRI Sydney supervisors.

The planner lets supervisors upload the daily Outstanding Orders report and calculates outbound labour requirements for KPI-mapped customer accounts. Files are processed locally in the browser; no order data is uploaded to a server.

Live site: https://tefui-main.github.io/BRI-Planner/

## Current Scope

- Upload daily Outstanding Orders Excel report.
- Include only configured KPI customers.
- Exclude orders with status `Picked Complete`.
- Use 6.85 productive hours per person.
- Calculate required people by work team:
  - Picking
  - Packing
  - Order prep / admin
- Show client KPI assumptions and editable scenario rates.
- Keep inbound workload visible as a future section for putaway, receipts, container unloads, and pallet movement.

## KPI Baseline

The first baseline is hardcoded from `DASHBOARD - RESOURCES.xlsx`.

Customer mappings:

- `TBS` -> TBS
- `MAKAUSSYD` -> Makita
- `TENTSYD-WH` -> Ten Tops
- `FLYTSYD-WH-F` -> Flying Tiger
- `AVBBBW` -> BBW B2B
- `AVRBBW` -> BBW EComm

## Use

Open `index.html` in a browser or serve it through GitHub Pages, then upload the daily Outstanding Orders report.

## Privacy

The Excel workbook is parsed in the browser. The planner does not upload order data to GitHub, a backend, or any third-party service.
