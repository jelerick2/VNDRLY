# VNDRLY.ai — Production Readiness Test Script

**Version 1.0 — August 12, 2026**

| | |
|---|---|
| **Web target** | https://vndrly.ai |
| **iOS target** | VNDRLY Field Mobile via TestFlight (production API baked in: `vndrly.ai`) |
| **Tester** | ______________________________ |
| **Date started** | ______________________________ |
| **Build / deploy tested** | ______________________________ |

**Purpose.** A single, direct walkthrough that (A) creates a brand-new Partner and Vendor through the onboarding tool, (B) drives one work ticket through the entire lifecycle while touching every web-app surface, (C) verifies the iOS app end to end, and (D) ends with a go/no-go checklist for production. Follow it top to bottom; each step has a checkbox and an expected result. When a step fails, log it in the Defect Log (last page) with the step number and keep going unless the failure blocks later steps.

---

## How to use this script

1. **Work top to bottom.** Later steps depend on data created in earlier steps (the new Partner/Vendor orgs and the ticket).
2. **Checkboxes:** mark ☐ → ☑ when the expected result matches, or ✗ and log a defect.
3. **Test data naming:** everything you create is prefixed **UAT-** so it is easy to find and clean up afterwards (e.g. partner "UAT Petroleum Co", vendor "UAT Field Services").
4. **Two browsers help.** The lifecycle section switches between Partner, Vendor, and Admin sessions frequently. Use two browser profiles (or one normal + one incognito) to avoid constant logout/login.
5. **Time required:** roughly 3–4 hours for the web pass, 1–1.5 hours for iOS.

### Known platform limitations (do NOT log these as defects)

| Area | Current state |
|---|---|
| **Outbound email** | Disabled in code ("Outbound email skipped"). Password-reset, invite, invoice, and 1099 e-delivery **emails will not arrive**. Use on-screen links: employee invites can be copied from the Employees page; skip email-arrival checks. |
| **Ask VNDRLY (AskV)** | Requires the Anthropic key on the server. If unconfigured, expect a clear failure message — verify the panel opens and fails gracefully. Voice input additionally requires the OpenAI key. |
| **QuickBooks Online / OpenAccountant** | Require Intuit/OA secrets on the server. If unconfigured, the Connect buttons should fail with a clear configuration error, not a crash. |
| **Maps** | Crew Map / Site Map / site pickers require the Mapbox token. Blank map tiles = missing token, not an app bug. |

### Test accounts (canonical — do not change these passwords)

| Login | Password | Role |
|---|---|---|
| `admin` | `vndrly123` | VNDRLY Admin |
| `mach` | `mach123` | Partner (Mach) |
| `exxon` | `exxon123` | Partner (Exxon) |
| `baker` | `baker123` | Vendor (Baker) |
| `winchester` | `winchester2` | Vendor (Winchester) |
| `joe.boggs@winchester.com` | `winchester2` | Field employee (Winchester) |

Plus the accounts **you create** in Part A (write them here):

| Org | Login (email) | Password |
|---|---|---|
| UAT Petroleum Co (partner admin) | ______________________ | ______________________ |
| UAT Field Services (vendor admin) | ______________________ | ______________________ |
| UAT field employee | ______________________ | ______________________ |

### Pre-flight (5 minutes)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | 0.1 Open https://vndrly.ai in a fresh browser session | Login page loads: dark theme ("vdark"), brand mark, **Sign In to Portal** button |
| ☐ | 0.2 Check https://vndrly.ai/api/health | `{"status":"ok"}` |
| ☐ | 0.3 On the login page, toggle theme (top-left) and language (English/Spanish) | Page flips to light theme; labels switch to Spanish and back |
| ☐ | 0.4 Log in as `admin` / `vndrly123`, then sign out | Admin Portal dashboard loads; sign-out returns to login |

---

# PART A — Onboarding (creates all test data for the rest of the script)

## A1. Create a Partner with the onboarding tool (~15 min)

Route: `/signup` → **Begin Partner Onboarding** (direct: `/onboarding/partner`). Do this logged out.

| ☐ | Step | Expected |
|---|---|---|
| ☐ | A1.1 From the login page click **Onboard your organization →** | Chooser page `/signup` shows Partner and Vendor onboarding options |
| ☐ | A1.2 Click **Begin Partner Onboarding** | Wizard "Partner Onboarding" opens on step **Company Basics** with a 7-step stepper: Company Basics · Platform Agreement · Branding · First Site · Tax & Billing · Preferences · Invite Team |
| ☐ | A1.3 Fill Company Basics: Company Name **UAT Petroleum Co**, Your Name **UAT Partner Admin**, a real email you control, phone, password (8+ chars), confirm | All fields accept input; a duplicate-name warning appears only if a similar partner exists (if so, tick "I'm sure this is a different partner") |
| ☐ | A1.4 Click **Create account** | Dialog: **"Congratulations! Your account has been successfully created!"** with Continue onboarding / Go to VNDRLY.ai now. You are now signed in; an email-verification banner may appear (email is disabled — ignore, do not log) |
| ☐ | A1.5 Continue onboarding → **Platform Agreement**: scroll the EULA, tick the acceptance checkbox, Continue | Cannot continue without scrolling/accepting; step completes |
| ☐ | A1.6 **Branding**: upload a horizontal logo and a square logo, pick a primary and accent color | Live header preview updates with your logo/colors |
| ☐ | A1.7 **First Site**: name **UAT Test Pad #1**, a real street address, keep the generated site code, geofence radius default (1609 m) | Address accepted; site code shown (format `SITE-XXXXXXXX`) |
| ☐ | A1.8 **Tax & Billing**: Federal Tax ID (e.g. 12-3456789), State Tax ID, physical + billing addresses | Fields accept input |
| ☐ | A1.9 **Preferences**: hours of operation text, default operating radius (e.g. 100 miles) | Accepted |
| ☐ | A1.10 **Invite Team**: enter one teammate email → click **Finish setup** | Wizard completes and lands on the **Partner Portal** dashboard |
| ☐ | A1.11 Sidebar → **Partner** | Your partner detail page shows the branding, tax IDs, and addresses you entered |
| ☐ | A1.12 Sidebar → **Site Locations** | **UAT Test Pad #1** exists with your radius and site code |
| ☐ | A1.13 (Resume check) Log out, log back in as the UAT partner | If any step was skipped, dashboard shows a **Finish setting up your account** banner that deep-links to the skipped step |

## A2. Create a Vendor with the onboarding tool (~20 min)

Route: `/signup` → **Begin Vendor Onboarding** (direct: `/onboarding/vendor`). Do this logged out (new browser profile is easiest).

| ☐ | Step | Expected |
|---|---|---|
| ☐ | A2.1 `/signup` → **Begin Vendor Onboarding** | Wizard "Vendor Onboarding" with 8-step stepper: Account · Platform Agreement · Branding · Tax IDs · Service Area & Work Types · Compliance · Rates & 1099 · First Employee |
| ☐ | A2.2 **Account**: Company **UAT Field Services**, Your Name **UAT Vendor Admin**, a second real email, phone, password | **Create account** → congratulations dialog, signed in |
| ☐ | A2.3 **Platform Agreement**: scroll + accept | Completes |
| ☐ | A2.4 **Branding**: upload a logo, pick a brand color | Live preview updates |
| ☐ | A2.5 **Tax IDs**: federal + state tax IDs, physical + billing addresses | Accepted |
| ☐ | A2.6 **Service Area & Work Types**: set operating radius (e.g. 50 mi); check **at least 2 work types** from the catalog | Checkbox catalog grouped by category; banner notes pricing is set later under "Your services & rates" |
| ☐ | A2.7 **Compliance**: carrier, policy number, future expiration date, **upload a COI file** (any PDF), coverage notes | Upload succeeds and shows the file |
| ☐ | A2.8 **Rates & 1099**: baseline hourly rate (e.g. 85), daily OT after (8), weekly OT after (40), OT multiplier (1.5); choose **Yes** for 1099 e-delivery | Accepted |
| ☐ | A2.9 **First Employee**: first/last name **UAT Fieldhand**, a third email, phone → **Finish setup** | Wizard completes → Vendor Portal dashboard |
| ☐ | A2.10 Sidebar → **Your services & rates** (`/vendor-catalog`) | The work types you checked are listed; set a **Price / Unit** on each and save | 
| ☐ | A2.11 Sidebar → **Employees** | **UAT Fieldhand** exists (role `field`) |
| ☐ | A2.12 Sidebar → **Vendor** | Vendor detail shows branding, tax IDs, insurance, and rates from the wizard |

## A3. Field employee invite (~10 min)

Email is disabled, so send the invite from the UI and copy the link.

| ☐ | Step | Expected |
|---|---|---|
| ☐ | A3.1 As UAT vendor: **Employees** → open **UAT Fieldhand** → **Send onboarding invite** | Invite created; the invite **URL is copied/shown** (`/onboarding/field/<token>`) |
| ☐ | A3.2 Open the invite link in a logged-out browser | 3-step wizard: **Personal Info → Photo & Certs → Set Password** |
| ☐ | A3.3 Personal Info: confirm name, phone, language **English**, role **Field** | Accepted |
| ☐ | A3.4 Photo & Certs: upload a profile photo (required); optionally tick PEC SafeLandUSA + expiration | Cannot continue without photo |
| ☐ | A3.5 Set Password (8+) → **Finish setup** | Lands signed-in on the **Field Employee Portal** (`/field`) with tabs Home · Schedule · Scan · Profile |
| ☐ | A3.6 (Alternate path) As vendor, open another employee → **Enable web portal login** → set email + temp password + force-change flag | Login is created; signing in with it forces a password change |

## A4. Connect the new orgs (admin, ~10 min)

The new Partner and Vendor need a relationship and assignments to transact. 

| ☐ | Step | Expected |
|---|---|---|
| ☐ | A4.1 Log in as `admin`. Sidebar → **Partners** | **UAT Petroleum Co** appears in the list with your branding |
| ☐ | A4.2 Sidebar → **Vendors** | **UAT Field Services** appears |
| ☐ | A4.3 Open **UAT Petroleum Co** → partner–vendor approvals/relationships → approve/link **UAT Field Services** (exact control: partner detail relationship section) | Vendor is now related to the partner |
| ☐ | A4.4 As UAT partner: **Site Locations** → open **UAT Test Pad #1** → work assignments → assign **UAT Field Services** for one of its work types | Assignment saved; this site/work-type pair is what the ticket in Part B uses |
| ☐ | A4.5 As `admin`: **Platform catalog** (`/catalog`) | Platform-wide work types list loads; add one test entry **UAT-Widget-Service** and confirm it appears (used to verify catalog CRUD) |

---

# PART B — Web application: full functional pass

## B1. Hotlist marketplace (Partner posts, Vendor bids) (~15 min)

The Hotlist lives on the **Dashboard** (`/`), not its own route.

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B1.1 As UAT partner, Dashboard → Hotlist → **Post Hotlist Job**: site **UAT Test Pad #1**, a work type your vendor offers, description, amount | Job appears in the Hotlist with a live (SSE) indicator |
| ☐ | B1.2 **Print Hotlist** | `/print-hotlist` renders the open jobs |
| ☐ | B1.3 As UAT vendor, Dashboard → Hotlist | The job is visible (vendor in radius and offers the work type) |
| ☐ | B1.4 Place a **bid** (amount, ETA, note); add a **comment** | Bid + comment appear; partner side updates live without refresh |
| ☐ | B1.5 As UAT partner, review bids → **Award** the UAT vendor's bid | Bid marked awarded |
| ☐ | B1.6 **Convert awarded bid → ticket** | New ticket in **awaiting_acceptance**; note its number: **#______** |

## B2. Ticket lifecycle — happy path (~30 min)

Use the ticket from B1.6 (or Partner → Tracking → **Create New Job**).

| ☐ | Step | Actor | Expected |
|---|---|---|---|
| ☐ | B2.1 **Tracking** (`/tickets`): find ticket #______; try each status filter pill | Partner | Pill filters work; ticket shows **awaiting_acceptance** |
| ☐ | B2.2 Open ticket → **Accept** | Vendor | Status → **initiated**; audit trail records it |
| ☐ | B2.3 **Schedule ticket**: assign **UAT Fieldhand**, set date | Vendor | Saved; appears on the employee's Schedule |
| ☐ | B2.4 As **UAT Fieldhand** (`/field`): open the ticket and progress it (En Route → On Location → **Check In**) | Field | Status → **in_progress**, lifecycle → on_site; GPS events in ticket **History** with "View on map" |
| ☐ | B2.5 Add **Parts & Labor**: one Labor line and one Part line | Field/Vendor | Lines save; **tax preview** computes; totals correct |
| ☐ | B2.6 Add a **comment with a photo** | Field | Comment + thumbnail visible to vendor/partner live |
| ☐ | B2.7 **Check Out** → **Send for Review** | Field | Status → **pending_review** |
| ☐ | B2.8 **Submit for partner approval** | Vendor | Status → **submitted** (requires line items) |
| ☐ | B2.9 **Approve** (+ optional vendor rating) | Partner | Status → **approved**; rating recorded |
| ☐ | B2.10 **Disperse Funds**: method ETF, reference, note, receipt upload | Partner AP | Status → **funds_dispersed**; Payment Details visible |
| ☐ | B2.11 **Print ticket** | Any | `/print-ticket/:id` renders lines, totals, history |
| ☐ | B2.12 Audit trail → **export CSV and PDF** | Admin | Both download and match on-screen events |

## B3. Ticket lifecycle — branches (~20 min)

Create three quick tickets (Partner → Tracking → **Create New Job**) against UAT Test Pad #1.

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B3.1 Ticket 2: vendor **Deny** with reason | Status → **denied**; partner sees **Find Vendor** to re-route |
| ☐ | B3.2 Ticket 3: run to **submitted**, partner **Kickback** with reason → fix → resubmit | **kicked_back** → editable → **submitted** again |
| ☐ | B3.3 Ticket 4: vendor office **Cancel job** | **cancelled**; confirm a field/foreman login has NO cancel button |
| ☐ | B3.4 On an in_progress ticket: vendor **Mark Awaiting Payment** | **awaiting_payment**; partner AP can Disperse from here |
| ☐ | B3.5 As `admin` on the dispersed ticket: **Reverse dispersal** | Returns to **approved**; audit trail records reversal |
| ☐ | B3.6 **Flag** a ticket; check **Flagged** page (`/flagged`); unflag | Appears and clears for admin/partner/vendor |
| ☐ | B3.7 **Nudge** up (vendor→partner) and down (partner→field) | Recipient's bell shows the nudge; no status change |

## B4. Sites, QR codes, visitors (~15 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B4.1 Partner: **Site Locations** → **Add Site**: try **Use my location**, drag the map pin, adjust radius, upload wellhead photo | Address ⇄ coordinates geocode both ways; map preview with radius circle; auto site code |
| ☐ | B4.2 Site detail → **Print visitor QR** | `/print-visitor-qr/:id` renders a scannable QR |
| ☐ | B4.3 Logged out, open `/visit/<site-code>` | Public visit page loads with site name |
| ☐ | B4.4 Visitor check-in (name + safety acknowledgment) then check out | Visit recorded |
| ☐ | B4.5 Partner: **Visitors** page → open the visit | Visit listed; detail shows map pin |
| ☐ | B4.6 Site detail: **Deactivate** then reactivate | While inactive, new tickets against it are blocked with a clear message |

## B5. Live maps (~10 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B5.1 Vendor: **Crew Map** | Map renders; **Live** pill connected (SSE); on-clock employees appear |
| ☐ | B5.2 Click a pin → **Crew replay** | Day-replay timeline of pings |
| ☐ | B5.3 Partner: **Site Map** → select UAT Test Pad #1 | Employees near site, recent tickets/trips, compliance cards |
| ☐ | B5.4 Verify: partner has no Crew Map nav; vendor has no Site Map nav | Role scoping correct |

## B6. Money: invoices, bills, statements (~20 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B6.1 Vendor: **Invoices** — filter by period/number; open an invoice | Filters work; detail shows lines and ledger |
| ☐ | B6.2 Invoice detail: **Send** (email disabled — expect graceful no-send), **Record payment**, **Credit memo**, **Late fees** | Ledger updates after each action |
| ☐ | B6.3 Partner: **Bills to Pay** | Approved/unpaid work appears as payables |
| ☐ | B6.4 **Statements** (`/statement`) as vendor, partner, and admin | Party statement renders; admin can pick any party |
| ☐ | B6.5 Invoice detail: multi-select lines → **Set 1099 category** → **Undo** | Applies and reverts |

## B7. Reports, 1099, accounting connections (~20 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B7.1 Vendor **Reports**: A/R aging, revenue by partner / work type / AFE, sales tax collected, crew cost | Cards load (Baker/Winchester have history) |
| ☐ | B7.2 Partner **Reports**: A/P aging, spend by vendor / work type / AFE, sales tax paid | Cards load |
| ☐ | B7.3 1099 section (either role): NEC worksheet, MISC, K preview — JSON/CSV/PDF exports | Files download and open |
| ☐ | B7.4 Admin **Reports**: 1099 filing dashboard, category change log, **IRS FIRE export** (TXT) | FIRE file downloads |
| ☐ | B7.5 Vendor Reports: **Connect QuickBooks Online** | With secrets: OAuth window; without: clear config error, no crash |
| ☐ | B7.6 Vendor Reports: **Connect OpenAccountant** (OAuth and API-key dialog) | Same expectation as B7.5 |
| ☐ | B7.7 Admin: **1099 transmitter** (`/admin/1099-transmitter`) — edit a T-record field, save | Saves; history paginates |

## B8. Safety (~15 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B8.1 Any role: **Report safety issue** (`/safety-report`, from dashboard card or ticket) — type "near miss", description; try the **anonymous** toggle | Event submits |
| ☐ | B8.2 Submit a second report with **stop-work** against a UAT site | Site becomes inactive; new tickets blocked |
| ☐ | B8.3 Partner (HSE) or admin: **Safety inbox** (`/safety`) → open the event → add note → **Close event** | Event closes |
| ☐ | B8.4 Reactivate the stopped site (site detail, HSE/admin gate) | Site active again; ticket creation works |
| ☐ | B8.5 **Safety training** (`/safety-training`, via dashboard banner) — complete a module | Completion recorded; banner clears |

## B9. Notifications, AskV, account plumbing (~15 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B9.1 Bell → **Notifications** inbox: mark read/unread, delete, open a ticket link | All actions work; deep link opens the ticket |
| ☐ | B9.2 **Notification preferences**: toggle categories, set DND hours | Saves and persists on reload |
| ☐ | B9.3 **Ask VNDRLY** pane (any portal): open, send "what tickets are in progress?" | With Anthropic key: role-aware answer; without: clear failure, no crash |
| ☐ | B9.4 **Context switching**: log in as a user with 2+ memberships (or your UAT user if invited to both orgs) | Blocking org picker after login; sidebar switcher swaps context and data |
| ☐ | B9.5 **Forgot password** (`/forgot-password`) | Form submits gracefully (email disabled — no mail arrives; do not log) |
| ☐ | B9.6 Dark/light toggle + English/Spanish toggle inside the app | Theme and locale switch and persist |

## B10. Admin-only surfaces (~10 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B10.1 **VNDRLY** (`/admin/vndrly`): edit platform brand color; **Add VNDRLY Employee** (temp password shown) | Saves; new admin can log in and is forced to change password |
| ☐ | B10.2 **Rate limits** (`/admin/rate-limits`) | Live throttle budgets table; Refresh works |
| ☐ | B10.3 **Removed comments** (`/admin/removed-comments`) | Soft-deleted comments audit (delete a test comment on a ticket first, then find it here) |
| ☐ | B10.4 **Employees** (`/field-employees`): open UAT Fieldhand → verify PEC cert fields, **disable credentials**, re-enable | Credential disable blocks login; re-enable restores |
| ☐ | B10.5 **Catalog health** (`/catalog-health`) | Data-quality report loads |
| ☐ | B10.6 Confirm partner/vendor logins get 403/redirect (not a crash) on `/admin/vndrly` and `/admin/rate-limits` | Admin routes protected |

## B11. Foreman portal + field web portal (~15 min)

Give one UAT employee the **foreman** role (vendor → Employees → edit role), then log in as them.

| ☐ | Step | Expected |
|---|---|---|
| ☐ | B11.1 Login lands on **Foreman Portal** (`/foreman`) with tabs Today · Schedule · Map · Crews · Analytics · Scan · Profile | Correct portal for `vendorRole: foreman/both` |
| ☐ | B11.2 **Today**: quick actions — Alerts, Start job, Schedule, Safety Reports | All four open |
| ☐ | B11.3 **Crews**: save a crew preset with UAT Fieldhand | Preset saved, reusable when scheduling |
| ☐ | B11.4 **Start job** → create a ticket; add crew check-in from **Crew & Time**; **Close for Review** | Works; foreman has NO cancel option |
| ☐ | B11.5 **Analytics**: my tickets, on-site today, awaiting review, kickback trend | Cards render |
| ☐ | B11.6 As plain field user (`/field`): Home · Schedule · Scan · Profile tabs; **Send for Review** on a ticket; Profile → language/theme/org switch; **Compliance Card** | Field portal complete; compliance card shows QR |

---

# PART C — iOS app (VNDRLY Field Mobile via TestFlight)

**Setup:** install the latest TestFlight build on a physical iPhone (GPS needed). The build talks to production (`vndrly.ai`) — the same data as Part A/B. Have the UAT Fieldhand, foreman, vendor, and partner credentials ready. Bring the phone to a site inside a geofence if possible; otherwise note geofence steps as "not testable at desk".

## C1. Login, consent, and permissions (~10 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | C1.1 Launch app logged out | **VNDRLY / Field Employee Portal** login; English/Spanish toggle top-right |
| ☐ | C1.2 Toggle Spanish, then back | All labels flip and persist |
| ☐ | C1.3 Sign in as **UAT Fieldhand** | Prompt **Enable Face ID?** — accept; lands on tabs |
| ☐ | C1.4 First field login → **Share your live location while on the clock?** | Consent screen forced; **Accept & turn on** → iOS location permission prompts (While Using + Always) |
| ☐ | C1.5 Kill and relaunch the app | Face ID auto-prompt signs you in |
| ☐ | C1.6 Tap the EULA link on the login screen (log out first) | Opens `vndrly.ai/legal/eula` in browser |

## C2. Field employee core loop (~25 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | C2.1 **Home** tab: section **Tracking** lists open jobs; pull-to-refresh | The ticket scheduled in B2.3 (or a new one) is listed |
| ☐ | C2.2 Tab **Scan** → grant camera → scan the printed site QR from B4.2 | Opens **Start New Job** with the site prefilled (note: authenticated Scan starts a job, not a visitor check-in) |
| ☐ | C2.3 Complete **Start New Job**: work type(s), description, answer **"Are you on site now?" → Not yet** | Ticket created in **Pending Arrival**; opens **Tracking** screen |
| ☐ | C2.4 Tap **En Route** | **Starting Mileage** modal (Skip/Submit); lifecycle → en_route; **Live location: active** pill appears |
| ☐ | C2.5 Tap **On Location** when at/near the site | Lifecycle → on_location |
| ☐ | C2.6 Tap **Check In** inside the geofence | Toast **"Checked in — you're on site"**; status → in_progress, lifecycle → on_site. (Outside the geofence: expect a clear geofence rejection) |
| ☐ | C2.7 Web check: vendor **Crew Map** on desktop | Your phone's pin is live on the map |
| ☐ | C2.8 On the ticket: add a **Parts & Labor** line (Part, qty, unit price) | Line saves; totals update |
| ☐ | C2.9 **Comments**: take a photo with the camera and post | Photo uploads; visible on web ticket immediately |
| ☐ | C2.10 Tap **Check Out** | **Ending Mileage** modal; lifecycle → off_site |
| ☐ | C2.11 Tap **Close for Review** | Status → pending_review; office is notified |
| ☐ | C2.12 **History** from Home | The completed job appears |
| ☐ | C2.13 **Schedule** tab | Assigned/scheduled jobs for the next 14 days |
| ☐ | C2.14 **Profile**: stop location sharing, restart it; change language; **Edit Profile** photo; **Compliance Card** QR | All persist; card renders |

## C3. Foreman on mobile (~15 min)

Sign in as the foreman-role user from B11.

| ☐ | Step | Expected |
|---|---|---|
| ☐ | C3.1 Home tab shows **Foreman Portal** with **Quick actions** (Alerts, Start job, Schedule, Safety Reports) and **Active jobs** | Extra tabs visible: **Map**, **Crews**, **Comms** |
| ☐ | C3.2 **Crews**: create/edit a crew | Saved |
| ☐ | C3.3 On a ticket: **Crew & Time** — add crew member, check in/out an employee, bulk check-out | Labor summary updates |
| ☐ | C3.4 **Schedule** tab → **Schedule Ticket** → pick ticket, crew, reminders | Saved; **Crew Tracker** link appears on the ticket |
| ☐ | C3.5 **Map** tab | Live crew pins; tap → day replay |
| ☐ | C3.6 **Comms** tab: pick an active ticket, **Hold to talk** | PTT records/sends; non-foreman deep link shows locked message |
| ☐ | C3.7 Close a ticket for review while crew still checked in | Prompt **Crew still checked in → Check Out & Close** |

## C4. Office roles on mobile (~15 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | C4.1 Sign in as UAT **vendor** admin | Home = **Vendor Portal**, list **Team tickets**; pending work offers show **Commit/Pass** |
| ☐ | C4.2 On an awaiting_acceptance ticket: **Accept** / **Deny** banner | Works like web |
| ☐ | C4.3 On an in_progress ticket: **Awaiting Payment** → confirm modal | Status → awaiting_payment |
| ☐ | C4.4 Sign in as UAT **partner** | Home = **Partner Portal**, **Site tickets**; **Map** tab = Site crew map; **Add site** CTA works |
| ☐ | C4.5 Partner (AP): on an approved ticket → **Disperse Funds** (method, reference, receipt photo) | Status → funds_dispersed; **Payment Details** visible; reverse if permitted |
| ☐ | C4.6 While a field ticket is open, have the vendor remove the site/work-type assignment on web | Mobile shows **Your assignment was changed** banner; lifecycle buttons grey out; **Cancel this ticket** offered |

## C5. Safety, notifications, AskV, visitor mode (~15 min)

| ☐ | Step | Expected |
|---|---|---|
| ☐ | C5.1 **Report safety issue** (Home or ticket) — submit one report | Success; appears in **My safety reports** |
| ☐ | C5.2 Open a safety event detail from the list | Renders event data |
| ☐ | C5.3 **Safety training** from Profile banner — play a module | Completion recorded |
| ☐ | C5.4 **Notifications**: categories filter, mark read, delete, open a ticket deep link; gear → **Notification settings** (toggles + DND) | All work; push notification (from a nudge or crew punch) deep-links correctly |
| ☐ | C5.5 **AskV** tab: send a message | With Anthropic key: role-aware reply; without: graceful failure |
| ☐ | C5.6 **Flagged** tab | Flagged jobs list matches web |
| ☐ | C5.7 Sign out → **Continue as Visitor** → Visitor Sign-In (name + safety ack) → scan site QR → check in → check out | Guest 24h session; visit appears in web **Visitors** page; no portal tabs for guests |
| ☐ | C5.8 Org switching: sign in as a multi-membership user | **Choose your organization** modal; Home header org tap switches; Profile → Active organization works |

---

# PART D — Production go / no-go

## D1. Release gate checklist

All of these must be true before promoting to production for real customers:

| ☐ | Gate |
|---|---|
| ☐ | Part A: Partner and Vendor onboarding completed start-to-finish with no blocking defect |
| ☐ | Part B2: full ticket lifecycle (awaiting_acceptance → funds_dispersed) verified with real GPS events |
| ☐ | Part B3: every branch (deny, kickback, cancel, awaiting_payment, reverse) behaves and is audit-trailed |
| ☐ | Part B6/B7: money surfaces (invoices, bills, statements, 1099 exports) produce correct totals |
| ☐ | Part C2: iOS field loop (scan → en route → check in → parts/photos → check out → close for review) verified on a physical device against production |
| ☐ | Live maps show real phone pings (B5.1 + C2.7) |
| ☐ | Role scoping verified: no admin surfaces reachable by partner/vendor/field logins (B10.6); foreman cannot cancel (B3.3/B11.4) |
| ☐ | No defect marked **P0/P1** remains open in the log below |
| ☐ | Decision on known limitations: outbound email OFF, and any unconfigured integrations (QBO/OA/AskV/Mapbox) are either configured or consciously deferred |

## D2. Post-test cleanup

| ☐ | Step |
|---|---|
| ☐ | Remove or deactivate the UAT orgs (admin → Partners/Vendors → UAT Petroleum Co / UAT Field Services) once testing is done — or keep them as permanent staging orgs (recommended) |
| ☐ | Cancel any leftover open UAT tickets |
| ☐ | Delete the UAT-Widget-Service platform catalog entry (A4.5) |

## D3. Defect log

| # | Step | Severity (P0 blocker / P1 major / P2 minor / P3 cosmetic) | Description | Status |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |
| 6 | | | | |
| 7 | | | | |
| 8 | | | | |
| 9 | | | | |
| 10 | | | | |

## D4. Sign-off

| Role | Name | Date | Go / No-Go |
|---|---|---|---|
| Tester | | | |
| Product owner | | | |

*Generated from the VNDRLY source (`vndrly/VNDRLY.ai`) on 2026-08-12: routes, wizard steps, button labels, and lifecycle states were read directly from the code, so the labels in this script match the UI verbatim.*
