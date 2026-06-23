# BRI Sydney DC Planner - Future Exception Report Decision Brief

Date updated: 2026-06-23

## Current live app

The current published planner is a static GitHub Pages app. Operations uploads the WMS Outstanding Orders Report into the browser, reviews mapped customer workload, resolves alerts, enters manual unmapped/customer activity hours, and uses the **Print Plan** tab to produce the daily labour plan summary.

Current operator-facing exceptions are handled inside the browser:

- Sidebar alert for orders requiring review, such as `Did Not Allocate`.
- Sidebar alert for unmapped customer codes, with a clickable A-Z summary and drill-through order detail.
- Customer-card review badges and status/count pills that open filtered order detail.
- Missing manual quantities for customers such as Ten Tops where the WMS file does not provide the required UOM.

The automation described below is optional future work. It is not required for the current GitHub Pages planner.

## Current constraint

The existing planner calculates headcount only after a human uploads the WMS Outstanding Orders Report. A fully automated daily exception email cannot be reliable until the day's WMS file, or a parsed planner snapshot, is available to a backend service without relying only on a browser session.

Recommended interim path: add Supabase storage for the latest parsed planner snapshot. The browser app can upload the normalized calculation result through a Vercel API route after the file is loaded and reviewed. A scheduled report can then send from the latest snapshot. If no current snapshot exists, the report should send a missing-file exception.

Agreed architecture:

`Planner UI -> Vercel API routes -> Supabase -> Vercel Cron -> Resend`

## Decisions needed

1. WMS file source

   Preferred: WMS auto-exports the Outstanding Orders Report to email, SFTP, SharePoint, or another location that can be pulled into Supabase/server-side processing.

   Interim: operations uploads the file in the existing planner before 2:30 PM, and the planner stores the parsed snapshot in Supabase.

2. Exception definition

   Recommended initial rules:

   - Any unmapped customer code with planned work in the active planning window.
   - Any order with WMS status exactly matching or containing "Did Not Allocate".
   - Any mapped customer requiring at least 4.0 people for due-tomorrow/overdue work.
   - Any mandatory manual quantity still missing, especially Ten Tops total tasks/cartons.
   - Missing current WMS/planner snapshot by 2:30 PM Sydney time.
   - Optional later rule: actual roster variance, once actual rostered headcount is stored somewhere reliable.

3. Recipients

   Recommended initial groups:

   - Operations Manager: receives every report and every exception type.
   - Supervisors/Team Leaders: receive all exceptions at first.
   - Later refinement: add workstream/customer scoping, such as high-reach or admin-only exceptions.

4. Reporting time and daylight saving

   Target business time: to be confirmed with operations, likely mid-afternoon Australia/Sydney on weekdays.

   Sydney is UTC+10 during AEST and UTC+11 during AEDT. Instead of manually changing cron twice per year, schedule two UTC cron attempts and have the Vercel API route send only when the current Sydney time matches the agreed report time and today's report has not already been sent.

## Recommended build sequence

1. Create a new isolated Supabase project for `bri-planner`.
2. Apply the schema in `supabase/schema.sql`.
3. Deploy the Vercel API routes in `api/`.
4. Configure secrets:

   - `SUPABASE_URL`
   - `SUPABASE_SERVICE_ROLE_KEY`
   - `RESEND_API_KEY`
   - `REPORT_FROM_EMAIL`
   - `SNAPSHOT_WRITE_TOKEN` if the snapshot endpoint should require a shared token

5. Add Team Leader/Supervisor records and assign active report recipients.
6. Update the browser planner to write one `order_snapshots` row after upload/calculation.
7. Test `/api/cron-exception-report?force=true&dry_run=true`.
8. Enable the cron schedule after manual send succeeds.

## Browser app integration point

After `index.html` finishes calculating the planner result, post a normalized snapshot to Supabase:

- `business_date`: Sydney local date of the report run.
- `planning_due_date`: the due date being planned, usually next business day.
- `source_name`: uploaded XLSX filename.
- `summary`: top-level totals used by the sidebar, Print Plan report, and grand total.
- `snapshot`: structured details, including mapped customer cards, unmapped customers, order rows, manual-entry status, manual pallet activities, alert status, and status text.

The report route is deliberately tolerant of the first snapshot shape. It looks for common keys such as `unmappedCustomers`, `orders`, `customers`, `manualEntries`, `kpiPeopleRequired`, and `grandTotalPeople`.

## Open items to confirm with operations

- Can WMS export the daily XLSX automatically, and where can it send it?
- Should the people-required threshold be 4.0, 5.0, or customer-specific?
- Should future-due work ever appear as an exception, or only active planning-window work?
- Should Team Leaders receive every exception initially, or only exceptions scoped to their workstream?
- Who owns report failure follow-up if the 2:30 PM job cannot find a current snapshot?

## Source notes checked

- QIO reference architecture uses Vercel Cron to call a serverless route, Supabase for state, and Resend for email.
- BRI will use Vercel Cron rather than Supabase `pg_cron` for the first implementation.
