# Handoff: DPSS Fleet Portal

## Overview
A web portal for the U-M Division of Public Safety & Security (DPSS) fleet. It serves two roles, **Officer** and **Supervisor**, across two locations, **Ann Arbor** and **Dearborn**. It covers:
- daily vehicle pre/post-checks
- work orders
- vehicle assignments (bid-period plans, daily rosters and a posted sheet)
- service scheduling
- vehicle ordering, with an Excel build sheet emailed to Renee (fleet purchasing)
- fleet overview and budget forecasting

## About the Design Files
The files in `pages/` are **design references built in HTML**. They are working prototypes of the intended look and behavior, not production code. Recreate them in the target codebase using its existing framework and patterns. If there is no codebase yet, choose a suitable stack; React plus a small backend is a reasonable default.

Each `*.dc.html` file is self-contained: markup with inline styles, plus a `class Component` logic block at the bottom that holds all state and behavior. `support.js` is only the prototype runtime and should not be ported. To run the prototypes locally, serve the `pages/` folder, for example with `npx serve pages`, and open `Supervisor Home.dc.html`. Opening the files directly from disk won't work because they fetch the JSON files.

`reference/dpss-fleet-23.html` is the user's earlier all-in-one working app. It contains `window.FLEET_DATA` (VEH, ROSTER, PKG, CONTACTS, FLOW, STAGES), which is the source for the vehicle and roster data.

## Fidelity
**High-fidelity.** Colors, type, spacing and interactions are final. Match them closely.

## Global shell (every page)
- **Utility bar**:
  - Background `#FAFAFA`, bottom border `1px #F0F0F0`, padding `6px 40px`.
  - Left: the label "homeLocation", then a location selector (Ann Arbor / Dearborn).
  - Right: "Viewing as", then a role selector (Officer / Supervisor) styled as a pill `<select>`.
- **Header**:
  - White, min-height 67px, padding `14px 40px`, bottom border `1px #E5E7EB`.
  - Left: the wordmark "DPSS FLEET" (18px / 700 / letter-spacing 1.8px / `#00274C`).
  - Right: nav links (14px; active link 700 `#00274C`, others 500 `#6B7280`).
  - Far right: the primary pill button **+ Work Order** (navy background, white text, padding `12px 26px`).
- **Nav by role**:
  - Officer: Home, Assignments.
  - Supervisor: Home, Overview, Budget Forecast, Ordering, Service Calendar, Assignments.
- **Content column**: max-width 1280px, padding `48px 28px 88px`, vertical gap 22px.
- **Page title**: 42px / 800 / line-height 1.12 / letter-spacing −1px / `#00274C`. A 13px `#6B7280` subtitle sits above it.
- **Stored in localStorage**:
  - `dpss-role`: the selected role.
  - `dpss-location`: the selected location.

## Screens

### Officer Home (`Officer Home.dc.html`)
- The officer's vehicle for today.
- Pre-check and post-check flows, run as checklists.
- **Duplicate detection**: if a reported issue matches an open work order on that vehicle, the officer is shown the existing work order instead of creating a new one.
- Officer views deliberately leave out data-integrity flags.

### Officer Schedule (`Officer Schedule.dc.html`)
The officer's upcoming shifts and vehicles.

### Supervisor Home (`Supervisor Home.dc.html`)
- **Needs Attention**: a single card listing open **work orders only** (currently 11), grouped by vehicle, each with its work-order status.
- Clicking a vehicle opens a modal where the supervisor can move the work order between stages.

### Vehicle Assignments (`Shift Assignments.dc.html`)
This screen is the core workflow. Both locations use the same dropdown-row model, which comes from Dearborn's roster workbook.

**Supervisor tabs** (segmented pill): **Period plan**, **Daily roster**, **Posted sheet**. Officers see only the Posted sheet, plus a navy card showing their own vehicle.

- **Bid periods**:
  - Sep–Dec, Jan–Apr and May–Aug.
  - Each is planned **4 months ahead**. The card shows "Plan by <start − 4 months>".
  - On load, the page selects the next period whose planning window is open.
  - Status per card: Not started, Draft, Posted Rev N, In effect, or due.
- **Period plan**:
  - Pick a weekday (Sun–Sat tabs, each with a row count). The week repeats for the whole period.
  - Three shift blocks per day. Ann Arbor: Days, Afternoons, Midnights. Dearborn: Days, Noons, Nights.
  - Each block has rows of dropdowns: **Officer, Vehicle, Taser (—/Assigned/Loaner), Rifle (1–10), Kit (1–5), Additional (duty)**, then Notes and a remove (×) button.
  - A dashed "+ Add officer to <shift>" dropdown sits at the bottom of each block.
  - "Copy <day> to [days] → Copy" duplicates a day onto the chosen days.
  - An empty period can start from the previous period's plan or from blank.
  - A sticky status bar shows the draft or posted state. It has **Undo changes** and **Post Rev N & email officers** (maize button) actions.
- **Two-officer cars**:
  - "+ Add second officer" under a solo vehicle opens a "Riding with…" dropdown, which has a × to cancel.
  - Choosing an officer inserts a row with the same vehicle and duty.
  - The second row gets a "Remove second officer" link.
- **Daily roster**:
  - Date stepper (← date → Today). The Today button is maize when the selected date is today and grey otherwise.
  - Same blocks and rows as the Period plan.
  - An empty date offers "Start from <weekday> plan" (from the posted plan) or "Copy <previous day>". Nothing is auto-assigned.
  - Notes flag any vehicle that differs from the plan ("Planned: X").
  - A footer card has **Mark entered in TeleStaff**. It records a timestamp and flags when the roster changes after entry.
- **Posted sheet**:
  - Read-only grid: vehicles as rows, Sunday–Saturday as columns. Cells hold shift-colored name chips.
  - The legend filters by shift. The officer's own name is outlined.
  - **Print / PDF**: opens a landscape printable table.
  - **Export to Excel**: downloads a CSV.
- **Validation**, shown as red select styling plus a note:
  - officer listed twice in the same day
  - duplicate rifle or kit within a shift
  - no vehicle when the duty needs one (Regular Duty, Overtime, In FTO)
  - vehicle out of service (daily roster only)
  - Ann Arbor Days/Afternoons overlap from 1600 to 1700 on the same car: amber hand-off note
- **Vehicle tags**:
  - R = radar
  - CE = community engagement
  - K9
  - Command (Ann Arbor 25 and 405) is kept out of bidding.
- **localStorage keys**:
  - `dpss-plans`: `{ "<loc>|<periodId>": { rev, at, posted: Week, draft: Week } }`
  - `dpss-rosters2`: `{ "<loc>|<yyyy-mm-dd>": { d, a, m, entered, sig } }`
  - `Week` is 7 × `{ d: Row[], a: Row[], m: Row[] }`.
  - `Row` is `{ n, u, taser, rifle, kit, duty }`.

### Service Calendar (`Service Calendar.dc.html`)
- An embedded Google Calendar.
- 14 upcoming service dates.
- Vendor contact list.

### Vehicle Ordering (`Vehicle Ordering.dc.html`)
- A multi-step order request form with all required fields. There's no process-steps display.
- The Review step shows the email to Renee, including a note about follow-up.
- **Build sheet attachment**:
  - An `.xlsx` generated from the saved spec for the chosen vehicle model (`data/build-specs.json`, taken from Renee's existing build sheets).
  - The layout matches Renee's template (`data/build-sheet-template.xlsx`).
  - Header row: Item#, U-M vehicle# (the units being replaced), and "Total of N".
  - Spec lines are filled from the saved spec. The ACCEPTANCE column is left as "Select one".
  - The Department Authorization block is left blank for a real signature.
  - The file is attached to the email.

### Overview (`Overview.dc.html`) and Budget Forecast (`Budget Forecast.dc.html`)
- Supervisor fleet summary.
- Replacement and budget forecasting (split out of the original Overview).

### Fleet Ordering (`Fleet Ordering.dc.html`)
An earlier or alternate ordering screen, kept for reference.

## Design Tokens
- **Colors**:
  - Navy `#00274C` (primary; hover `#003A6E`)
  - Maize `#FFCB05` (accent, primary action, "today")
  - Page `#F7F8FA`, cards `#FFFFFF`
  - Borders `#E5E7EB` / `#F0F0F0` / `#F5F5F5`
  - Text `#131516` / `#374151` / `#6B7280` / `#9CA3AF`
  - Info `#2A5993` (tint `#E8EEF7` / `#A9BFDD`)
  - Success `#2A7D46` (tint `#EDF7F1`)
  - Warning `#92610A` (tint `#FDF4DC`)
  - Error `#C0362C` (tint `#FDECEC` / `#F2B8B3`)
- **Shift colors** (both locations):
  - Days: background `#D8F1F8`, text `#0B5566`, swatch `#3FB5D3`
  - Afternoons/Noons: background `#F8DDEE`, text `#7A1F5C`, swatch `#D14FA0`
  - Midnights/Nights: background `#FFF1BF`, text `#6B4A00`, swatch `#F2C200`
- **Tag colors**: R `#00274C`, CE `#1F5E35`, K9 `#5B3A8C`. White text, 10px/700, radius 4px.
- **Type**: DM Sans (Google Fonts, weights 400–900). Scale: 42/800 title, 22/700 modal title, 16–19/700 section and unit, 14 body, 13 small, 12 meta, 11/600 uppercase labels (letter-spacing 0.66px).
- **Radii**: pills 999px, cards 16px, hero/modal 18px, cells and chips 6px.
- **Borders**: cards `0.77px solid #E5E7EB`.
- **Shadows**: sticky bar `0 6px 20px rgba(0,39,76,0.06)`, modal `0 24px 60px rgba(0,20,40,0.25)`.
- **Buttons**: all pills.
  - Primary: navy with white text.
  - Emphasis: maize with navy text.
  - Secondary: white with a navy or grey hairline border.
  - Padding `12px 22px`, 14px/600–700.

## Data
- `data/assign-data.json`: Ann Arbor cars as `[unit, year, model, role, status]`, plus the roster.
- `data/dearborn-roster.json`: parsed from Dearborn's June 2026 roster workbook. Contains officers, vehicles, duty options, and per-day d/a/m rows `[name, vehicle, taser, rifle, kit, duty]` with call-offs.
- `data/build-specs.json`: spec rows per vehicle model for the build sheet.
- `data/build-sheet-template.xlsx`: Renee's template.
- `reference/dpss-fleet-23.html` → `window.FLEET_DATA`: vehicles, packages, contacts and work-order stages.

## Integration still to do
- **Live data**: vehicles, equipment, rosters and shift sheets from Google Sheets or Apps Script. localStorage is the prototype's stand-in.
- **Identity** from SSO. The role and location switchers are preview-only.
- **Email**: send the posted sheet to officers, and the order plus build sheet to Renee.
- **TeleStaff entry**: currently a manual confirmation.
- **+ Work Order page**: shared by both roles, not built yet.
- **Open questions**:
  - Ann Arbor Midnight hours.
  - Dearborn shift hours.
  - Whether Dearborn uses TeleStaff.
  - Whether Ann Arbor tracks taser, rifle and kit.

## Files
- `pages/*.dc.html`: one file per screen, listed above.
- `pages/support.js`: prototype runtime only.
- `pages/*.json`: copies of the data files so the prototypes run.
- `data/`: data and the xlsx template.
- `reference/`: the original working app.
