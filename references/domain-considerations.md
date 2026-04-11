# Domain Considerations

This file covers the domain-specific landmines for 26 common dashboard verticals. The generic dashboard discipline — auth, RBAC, CRUD, vertical slices, states, validation — applies identically to all of them. What differs is the 20% of domain-specific knowledge that, if missed, causes a rewrite, a compliance violation, or a broken product.

**How to use this file:** During pre-flight (question #1: "who uses this, for what job?"), identify which domain archetype matches. Read that section before designing the schema or writing the first migration. The gotchas listed here are things that generic CRUD misses — they're the landmines, not the full terrain.

If the dashboard spans multiple domains (e.g., a SaaS that includes billing — that's SaaS + Financial), read both sections.

---

## 1. SaaS / Multi-tenant

**Archetype:** Platform admin for a multi-tenant product — managing organizations, subscriptions, users, feature access, and usage.

**Core entities:** Tenant/Organization, Subscription/Plan, Seat/License, Feature Flag, Usage Metric, Invoice, API Key

**Gotchas:**
- **Tenant isolation is not just auth** — a missing `WHERE org_id = ?` leaks data across tenants. Every query, every cache key, every background job must be tenant-scoped. Shared DB vs. schema-per-tenant vs. DB-per-tenant is an architectural decision that's nearly impossible to change later.
- **Plan limits are runtime constraints, not billing metadata** — "5 seats," "10GB storage," "1000 API calls/day" must be enforced in real-time at the application layer, not just checked at invoice time.
- **Subscription lifecycle has ~12 states, not 3** — trialing, active, past_due, paused, cancelled-but-active-until-period-end, downgraded-pending, upgraded-prorated, churned, reactivated, comped. Modeling this as "active/inactive" breaks.
- **Feature flags are per-tenant, per-plan, AND per-user** — a feature might be enabled for the plan but toggled off for the org, or enabled as a beta for one specific tenant. Three-dimensional matrix.
- **Usage-based billing requires idempotent event ingestion** — double-counted API calls mean incorrect invoices, chargebacks, and broken trust.
- **Impersonation/support access must be audited** — support staff viewing a tenant's dashboard must leave an audit trail and must not be able to modify billing.
- **White-labeling affects every layer** — custom domains, logos, and email from-addresses require SSL cert provisioning, DNS validation, and email deliverability config.

**Compliance:** SOC 2 Type II (expected by enterprise buyers), GDPR data processing agreements per tenant, data residency requirements (EU tenants may require EU-hosted data).

**UX users expect:** Tenant switcher in nav, plan comparison/upgrade flow inline, usage dashboards with limits as gauges, "impersonating [tenant]" banner for support staff.

---

## 2. E-commerce / Retail

**Archetype:** Store admin managing products, orders, inventory, and fulfillment.

**Core entities:** Product/Variant/SKU, Order, Cart, Inventory Location, Fulfillment, Return/Refund, Promotion/Discount

**Gotchas:**
- **A product is not a row — it's a tree.** Product > Variant (size/color) > SKU > Inventory per location. Flattening this causes data integrity nightmares when a size is out of stock at one warehouse but available at another.
- **Price is not a single number** — list price, sale price, member price, bulk/tiered pricing, currency-specific pricing, tax-inclusive vs. tax-exclusive. A "price" column is always wrong.
- **Inventory is a ledger, not a counter** — decrementing a quantity field causes race conditions. Inventory should be event-sourced: reserved, allocated, shipped, returned, adjusted, damaged. "Available" is a computed projection.
- **Order state machines are complex and partially reversible** — pending > paid > partially_shipped > shipped > partially_returned > refunded. Many transitions happen in parallel.
- **Tax calculation is locale-specific and changes constantly** — US sales tax varies by state/county/city, EU VAT varies by product category and buyer location. Use a tax service API (Avalara, TaxJar). Never hardcode rates.
- **Promotions interact combinatorially** — "20% off + free shipping + buy-2-get-1" — which stack? Promotion engines are their own domain.
- **Returns are not the inverse of orders** — partial refunds, restocking fees, store credit vs. original payment method, inventory going to a different location.

**Compliance:** PCI DSS if handling card data (use tokenized payment providers), consumer protection laws (EU cooling-off periods), ADA/WCAG for storefront.

**UX users expect:** Drag-to-reorder product images, bulk inventory update via spreadsheet, order timeline with every state transition, SKU matrix editor (size x color grid), real-time inventory badges.

---

## 3. CMS / Content / Blog

**Archetype:** Content team managing articles, pages, media, and publishing workflows.

**Core entities:** Content Item (Post/Page), Content Type/Schema, Media Asset, Taxonomy (Category/Tag), Revision/Version, Workflow Stage, Author

**Gotchas:**
- **Content is not a single "body" text field** — content types have structured fields (hero image, SEO metadata, excerpt, related posts, custom fields). Storing everything in one rich text blob blocks structured queries, API delivery, and multi-channel publishing.
- **Draft/publish lifecycle requires revision history** — every save creates a version. Published content and draft edits coexist simultaneously. Scheduled publishing requires a background job.
- **Slugs and URLs are a data integrity concern** — changing a slug must create redirects from the old URL. Duplicate slugs, slug collisions, and locale-specific slugs cause SEO damage if missed.
- **Media management is its own subsystem** — images need multiple generated sizes, alt text, focal point cropping, CDN delivery, and deduplication. Deleting a media asset referenced by 50 posts is a cascade problem.
- **Localization is not just translation** — different locales may have entirely different content. Fallback chains (show English if French is missing), RTL support, and locale-specific URLs are structural decisions.
- **Content workflows are org-specific** — "Draft > Review > Legal > Scheduled > Published" vs. "Draft > Published." The workflow should be configurable, not hardcoded.

**Compliance:** WCAG 2.1 AA for published content, GDPR (comment forms, cookies), copyright/licensing metadata for media, DMCA takedown process.

**UX users expect:** WYSIWYG with structured blocks (Notion-style), side-by-side preview, version diff view, drag-to-reorder blocks, inline media upload, SEO score indicator, editorial calendar view.

---

## 4. Financial / Fintech / Accounting

**Archetype:** Finance team managing ledgers, transactions, invoices, and reconciliation.

**Core entities:** Account (asset/liability/equity/revenue/expense), Transaction/Journal Entry, Ledger, Invoice, Reconciliation, Chart of Accounts, Fiscal Period

**Gotchas:**
- **Double-entry bookkeeping is non-negotiable** — every transaction must have equal debits and credits. A single-entry "transactions" table requires a full rewrite when an accountant reviews it.
- **Money must never use floating point** — use integer cents or a decimal type with fixed precision. $0.01 rounding error across 100,000 transactions is $1,000 that fails an audit.
- **Financial records are append-only** — never edit or delete a transaction. Create a correcting/reversing entry. The original record must remain with a linked reversal.
- **Fiscal periods close and lock** — once a month/quarter/year is "closed," no transactions post to it without a formal reopening process. Not optional.
- **Multi-currency requires exchange rates at transaction time** — store both the original currency amount AND the converted amount at the rate used. Rates change daily; retroactive revaluation is a separate process.
- **Reconciliation is a first-class workflow** — matching bank statement lines to ledger entries is core, not an afterthought. Unreconciled items must be surfaced prominently.
- **Rounding rules vary by jurisdiction** — some countries round to nearest 0.05, some use banker's rounding, tax rounding has locale-specific rules.

**Compliance:** SOX (public companies), GAAP/IFRS accounting standards, PCI DSS for payment data, AML/KYC for fintech, data retention 7+ years.

**UX users expect:** T-account visualization, trial balance report, aged receivables/payables, bank reconciliation matching interface, chart of accounts tree, journal entry form with auto-balancing, fiscal period lock indicators.

---

## 5. Healthcare / Medical

**Archetype:** Clinical or admin staff managing patients, encounters, diagnoses, prescriptions, and claims.

**Core entities:** Patient, Provider, Encounter/Visit, Diagnosis (ICD code), Procedure (CPT code), Medication/Prescription, Insurance Claim, Care Plan

**Gotchas:**
- **HIPAA is not optional and it's not just encryption** — it requires access controls, audit logging of every PHI access (who viewed what patient record when), minimum necessary access, Business Associate Agreements with every vendor, breach notification procedures, and regular risk assessments. Violations cost $50K-$1.8M per incident.
- **Patient identity matching is an unsolved hard problem** — patients lack universal IDs. Name + DOB matching creates duplicates, and merging duplicates is a complex workflow (which record's allergies win?).
- **Clinical data uses standardized code systems** — ICD-10 for diagnoses (~70,000 codes), CPT for procedures, LOINC for labs, RxNorm for medications. Free-text fields are not acceptable for these.
- **Interoperability standards are mandated** — HL7 FHIR is required for certified EHR systems. You will need to both consume and produce FHIR resources.
- **Consent management is granular and revocable** — a patient may consent to treatment but not to sharing data with a specific insurer. This is not a simple boolean.
- **Drug interactions and allergy checking are safety-critical** — prescribing workflows must check drug-drug interactions and contraindications. Getting this wrong can kill someone.

**Compliance:** HIPAA, HITECH Act, 21st Century Cures Act (information blocking), GDPR (EU health data is "special category"), FDA for clinical software, state-specific consent laws.

**UX users expect:** Problem/medication/allergy list always visible, clinical note templates (SOAP format), encounter timeline, medication reconciliation workflow, e-prescribing integration, patient portal with role-restricted views.

---

## 6. Education / EdTech / LMS

**Archetype:** Instructors and admins managing courses, enrollments, grades, and student progress.

**Core entities:** Course, Module/Lesson, Assignment/Assessment, Enrollment, Grade/Score, Student, Instructor, Certificate

**Gotchas:**
- **Grading is not just a percentage** — letter grades, pass/fail, weighted categories (homework 30%, exams 40%), curved grades, dropped lowest scores, incomplete grades, grade overrides. The gradebook is a computation engine.
- **Academic calendar structures everything** — semesters, terms, registration periods, add/drop deadlines, grade submission deadlines create temporal boundaries that affect what actions are available when.
- **Student data has specific protections** — FERPA restricts who can see student records. Parents of minors have rights; parents of adults (18+) do not unless the student grants access.
- **Content sequencing and prerequisites are a DAG** — "complete Module 3 before Module 4" and "pass Course 101 before enrolling in 201" create dependency graphs enforced at enrollment and content access time.
- **Assessment integrity is a feature** — proctoring, time limits, randomized question pools, plagiarism detection, and lockdown browser integration are expected.
- **Accessibility is legally mandated** — ADA/Section 508 compliance is required for educational institutions. All content must work with screen readers, video must have captions.

**Compliance:** FERPA (US student records), COPPA (under-13), GDPR (EU students), ADA/Section 508 accessibility, state-specific K-12 data privacy laws.

**UX users expect:** Gradebook grid (students x assignments), progress tracker (completion %), assignment submission workflow with rubric, discussion forum threading, attendance tracker, certificate builder, student/parent/teacher portal views.

---

## 7. Travel / Hospitality / Booking

**Archetype:** Property or tour operator managing listings, availability, reservations, and guest communication.

**Core entities:** Listing/Property/Room, Reservation/Booking, Availability, Rate/Pricing, Guest, Channel (OTA), Calendar Block

**Gotchas:**
- **Availability is a time-slot matrix, not a boolean** — a hotel room is available per-night, a tour per-time-slot. Overbooking is intentional in some domains (airlines) and catastrophic in others (vacation rentals).
- **Rate management is multi-dimensional** — rates vary by date, day of week, season, length of stay, occupancy, booking channel, lead time, member status, and promo code. A "price" field is meaningless; you need a rate engine.
- **Channel management creates double-booking risk** — listings on Booking.com, Airbnb, Expedia, and direct must sync availability in near-real-time. A booking on one channel must instantly block dates on all others.
- **Cancellation policies are complex business logic** — "Free until 48 hours before check-in, 50% for 24-48 hours, no refund within 24 hours" with timezone considerations (whose timezone?), force majeure, and partial cancellations.
- **Timezone handling is critical and pervasive** — a hotel in Tokyo booked by a user in New York: checkout is 11 AM JST but the confirmation email shows the guest's local time. Cancellation deadlines evaluate in the property's timezone.
- **Guest communication has timing requirements** — confirmation immediately, pre-arrival info 3 days before, check-in instructions day-of, review request post-checkout. Automated sequences tied to booking dates.

**Compliance:** PCI DSS for payment, local tourism/lodging taxes and reporting, ADA for US properties, anti-discrimination laws, data retention for tax.

**UX users expect:** Calendar grid for availability (green/yellow/red), drag-to-block dates, rate calendar (price heatmap), reservation timeline (Gantt-style), channel distribution status, guest communication timeline, dynamic pricing suggestions.

---

## 8. Sports / Fitness

**Archetype:** Gym, studio, or club managing members, classes, schedules, and trainer assignments.

**Core entities:** Member/Athlete, Class/Session, Schedule, Membership Plan, Trainer/Coach, Booking/Check-in, Workout/Program, Body Metrics

**Gotchas:**
- **Class capacity and waitlists are real-time constraints** — a class with 20 spots must enforce capacity at booking time, manage a waitlist with auto-promotion on cancellation, and handle no-shows (which may incur penalties).
- **Recurring schedules with exceptions are hard** — "Spinning every Tuesday/Thursday at 6 PM, except holidays, with a substitute instructor on March 5th" is a recurring event with overrides. iCal RRULE + EXDATE is the minimum viable approach.
- **Membership freezes, pauses, and transfers are not edge cases** — members freeze for injury/travel, pause seasonally, transfer locations, or convert monthly to annual. Each has proration and billing implications.
- **Wearable/device integration generates high-volume time-series data** — heart rate every second, GPS tracks, rep counts. This needs time-series storage and aggregation, not relational rows.
- **Late cancellation and no-show policies drive revenue** — "Cancel 2+ hours before or lose a class credit" is business-critical logic. Grace periods, penalty credits, and strike systems are expected.
- **Family/group accounts share billing but have individual profiles** — a parent pays for the family membership but each member has their own bookings and progress.

**Compliance:** Health data privacy (jurisdiction-dependent), minor athlete protections (parental consent, safeguarding), PCI for recurring billing, liability waivers (digital signature).

**UX users expect:** Weekly schedule grid, class capacity indicators, member check-in kiosk mode, workout builder (sets x reps x weight), progress charts, leaderboards, trainer schedule calendar.

---

## 9. Real Estate / Property Management

**Archetype:** Property manager handling units, tenants, leases, maintenance, and owner reporting.

**Core entities:** Property, Unit, Tenant, Lease, Maintenance Request, Rent Payment, Listing, Inspection

**Gotchas:**
- **Lease terms are legal contracts with complex date logic** — start/end dates, renewal options (auto-renew vs. month-to-month), rent escalation clauses, security deposit rules (state-specific maximums and return timelines), early termination penalties.
- **Rent accounting has trust account requirements** — many jurisdictions require security deposits in separate escrow/trust accounts. Commingling operating funds with tenant deposits is illegal in most US states.
- **Maintenance requests have legal response timelines** — habitability issues (heat, water, structural) have legally mandated response times. Treating all requests equally causes legal violations.
- **Property financials need owner-level reporting** — each owner needs P&L statements, 1099 generation, reserve fund accounting, and CAM reconciliation. This is per-property-owner accounting.
- **Vacancy and turnover is a workflow** — move-out inspection > deposit itemization > unit make-ready > listing > showing > application > lease signing > move-in inspection. Each step has timelines and dependencies.
- **Fair housing compliance is non-negotiable** — screening criteria must be applied uniformly. The dashboard must not surface protected class information to decision-makers during application review.

**Compliance:** Fair Housing Act, state-specific landlord-tenant laws (vary wildly), security deposit laws, lead paint disclosure (pre-1978), habitability requirements, eviction process requirements, 1099-MISC for owner payments.

**UX users expect:** Portfolio overview with vacancy rates, unit-level P&L, lease expiration timeline, maintenance request kanban, tenant ledger, inspection checklist with photo capture, comparative market analysis.

---

## 10. Logistics / Supply Chain / Fleet

**Archetype:** Operations team managing shipments, vehicles, drivers, routes, and warehousing.

**Core entities:** Shipment/Consignment, Vehicle/Asset, Route, Driver/Operator, Warehouse/Depot, Load/Cargo, Proof of Delivery, Geofence

**Gotchas:**
- **Location tracking is a high-frequency data stream** — vehicles report GPS every 5-30 seconds. This is time-series data needing specialized storage (TimescaleDB, InfluxDB). PostgreSQL rows will fail at scale.
- **Route optimization is NP-hard** — vehicle capacity, delivery time windows, driver hours-of-service, traffic, multi-stop optimization. Integrate with an optimization engine (Google OR-Tools, HERE, Routific). Don't build one.
- **Chain of custody and proof of delivery are legal documents** — photos, signatures, timestamps, GPS at delivery are evidence in disputes. This data must be immutable.
- **Hours of Service regulations are strict** — US DOT: max 11 hours driving after 10 hours off, 14-hour window, 30-minute break after 8 hours. EU tachograph rules differ. Violations mean heavy fines.
- **ETA calculations must account for real-world constraints** — loading/unloading time, traffic, driver breaks, customs for cross-border, weight stations. Naive distance/speed is always wrong.
- **Vehicle maintenance is predictive, not just calendar-based** — intervals based on mileage, engine hours, AND time. Deferred maintenance must trigger escalating alerts.

**Compliance:** DOT/FMCSA, EU driving/rest regulations, hazmat transport rules, customs/import-export docs, cold chain compliance (temperature logging for perishables), ELD mandate.

**UX users expect:** Real-time map with vehicle positions, route visualization, driver HOS status bars, shipment tracking timeline, warehouse slot grid, vehicle maintenance dashboard with red/yellow/green, geofence alerts.

---

## 11. HR / People / Payroll

**Archetype:** HR team managing employees, compensation, benefits, leave, and compliance.

**Core entities:** Employee, Position/Job, Department, Pay Period, Payroll Run, Leave/PTO Balance, Benefits Enrollment, Performance Review

**Gotchas:**
- **Payroll is the most regulation-heavy computation most developers will ever implement** — gross pay, pre-tax deductions (401k, HSA), federal withholding (W-4), state tax (50 different rules), local tax, FICA (Social Security caps change annually), Medicare (additional 0.9% above $200K), post-tax deductions, garnishments (with priority rules). Any error is a legal violation.
- **Multi-state employment is a tax nightmare** — remote employees: which state's taxes apply? Reciprocity agreements, nexus rules, home state vs. work state all matter.
- **PTO accrual is not a simple balance** — rates vary by tenure, accrual may be per-pay-period, some states require payout at termination, some cap accrual ("use it or lose it" is illegal in California).
- **Effective dating is pervasive** — salary changes mid-pay-period, title changes effective next month, benefits effective first-of-month after 30 days. Every attribute needs effective dates, not just current values.
- **Compensation data is the most sensitive data in the company** — even HR staff may have tiered access (can see ICs but not executives).
- **Termination is a multi-system workflow** — final paycheck (timing is state-regulated), COBRA, access revocation, equipment return, exit interview, 401k rollover. Missing any step has legal consequences.

**Compliance:** FLSA (overtime, exempt vs. non-exempt), FMLA (leave), ADA (accommodations), ACA (benefits), EEOC, W-2/W-4/I-9/1099 forms, COBRA, ERISA (retirement).

**UX users expect:** Org chart, payroll run preview with exceptions highlighted, PTO calendar (team view), benefits enrollment wizard, performance review workflow, compensation band visualization, headcount reporting.

---

## 12. Project Management / Collaboration

**Archetype:** Teams managing tasks, sprints, timelines, and resource allocation.

**Core entities:** Project, Task/Issue, Sprint/Iteration, Board/View, Milestone, Time Entry, Comment/Activity, Workflow Status

**Gotchas:**
- **Task dependencies create scheduling constraints** — finish-to-start, start-to-start, with lag/lead times. Changing one task's dates must cascade through the dependency graph (critical path). Circular dependencies must be detected.
- **Custom workflows per team are expected** — one team uses Kanban (3 statuses), another uses a 7-status workflow with approval gates and transition rules ("only QA can move to Done").
- **Time tracking has billing implications** — entries may be billable/non-billable, associated with a client, subject to approval. Rounding rules and overtime add complexity.
- **Permissions are project-scoped, not global** — admin on Project A, member on Project B, viewer on Project C. Guest/external collaborator access with limited visibility is common.
- **Activity feeds must be real-time** — every field change generates an event. Users expect WebSocket-driven updates. The activity log is also the audit trail.
- **Reporting requires cross-project aggregation** — portfolio health, resource utilization, burn-down charts, velocity. These are expensive queries needing materialized views.

**Compliance:** SOC 2 for enterprise, GDPR (right to deletion conflicts with audit trails).

**UX users expect:** Kanban board with drag-and-drop, Gantt chart with dependency arrows, sprint backlog with story points, inline time tracking, @-mentions with notifications, customizable views (list/board/calendar/timeline).

---

## 13. Customer Support / Helpdesk

**Archetype:** Support team managing tickets, SLAs, queues, and customer communication.

**Core entities:** Ticket/Case, Queue, Agent, SLA, Canned Response/Macro, Knowledge Base Article, Customer, Escalation

**Gotchas:**
- **SLA tracking is a contractual obligation** — "First response within 1 business hour" must account for business hours, timezone of the contract, holidays, and SLA pausing when waiting on customer response. A ticket opened Friday at 4:30 PM with a 4-hour SLA is due Monday, not Saturday.
- **Ticket routing is multi-factor** — skills-based (language, product expertise), round-robin with capacity limits, priority queue jumping, VIP routing, and follow-the-sun for global teams.
- **Merging duplicate tickets must preserve context** — keep all messages from both, notify both requesters, consolidate SLA timers (use the earlier/stricter one).
- **Collision detection prevents duplicate work** — two agents viewing the same ticket need real-time presence indicators ("Agent Smith is typing").
- **Escalation paths are time-triggered and condition-triggered** — "If no response in 30 minutes, escalate to Tier 2. If Enterprise plan, escalate immediately." Automated rules running continuously.
- **CSAT surveys have timing requirements** — too early or too late = low response. Survey fatigue must be managed (don't survey the same customer every time).

**Compliance:** GDPR (right to delete, but ticket history may be needed for disputes), PCI DSS (mask card numbers in messages), data retention policies.

**UX users expect:** Queue with SLA countdown timers (red when breaching), split-pane (list + detail), customer context sidebar, canned response insertion, internal notes vs. public replies, collision indicators, satisfaction trends.

---

## 14. Marketing / CRM / Sales

**Archetype:** Sales and marketing teams managing leads, deals, campaigns, and pipelines.

**Core entities:** Contact/Lead, Account/Company, Opportunity/Deal, Pipeline Stage, Campaign, Activity/Touchpoint, Segment, Attribution

**Gotchas:**
- **Contact deduplication is an ongoing battle** — same person from a webform, a CSV import, and LinkedIn. Merge logic must handle conflicting data and cascade to all related activities and opportunities.
- **Pipeline stages are not linear** — deals move backward, skip stages, sit for months (needing stale deal alerts), and have stage-specific required fields.
- **Attribution modeling is an analytics problem masquerading as a data model** — first-touch, last-touch, linear, time-decay require capturing every touchpoint with timestamps. Changing the model must recompute all historical data.
- **Email deliverability is a feature** — bounce handling, unsubscribe management (CAN-SPAM one-click), suppression lists, sending reputation, SPF/DKIM/DMARC. Sending to a bad list blacklists your domain.
- **Forecast accuracy requires pipeline weighting** — stage-based probability vs. AI-predicted vs. rep-override are three different values. Forecasting by team, quarter, with commit/best-case/pipeline is expected.
- **Activity logging must be effortless or it won't happen** — auto-logging from email integration, calendar sync, and call recording is expected. Manual logging alone means empty CRMs.

**Compliance:** CAN-SPAM / CASL / GDPR for email, TCPA for phone/SMS, Do Not Call registries, consent tracking for B2C.

**UX users expect:** Pipeline board (Kanban by stage with deal values), contact timeline, email template builder with merge fields, campaign performance dashboard, forecast table (reps x months), lead scoring indicators.

---

## 15. IoT / Device Management

**Archetype:** Operations team monitoring devices, telemetry, firmware, and alerts at scale.

**Core entities:** Device, Sensor, Telemetry Reading, Alert/Alarm, Firmware Version, Device Group/Fleet, Command, Configuration Profile

**Gotchas:**
- **Telemetry volume is orders of magnitude beyond CRUD** — 10,000 devices x 10 metrics x every 10 seconds = 600,000 data points/minute. PostgreSQL won't handle this. Use a time-series database (InfluxDB, TimescaleDB, QuestDB) with retention policies and downsampling.
- **Devices are offline by default** — commands queue and retry. Telemetry arrives out of order and in batches. "Offline" is normal, not an error, with its own alerting thresholds.
- **OTA firmware updates are high-risk deployments** — a bad update bricks thousands of devices. Staged rollouts (1% > 10% > 50% > 100%), rollback, and success reporting are critical.
- **Alert fatigue kills usefulness** — deduplication, suppression windows (maintenance mode), severity escalation, acknowledgment workflows, and alert correlation (100 devices in same location = one root cause).
- **Device provisioning is a security boundary** — unique credentials (certificates), device attestation, secure enrollment. Compromised credentials must be revocable per-device.
- **Edge vs. cloud processing determines architecture** — some computation must happen at the edge. The dashboard manages edge gateway config, local rule engines, and data forwarding.

**Compliance:** FDA (medical devices), FCC (RF devices), GDPR (if devices collect personal data), IEC 62443 (industrial IoT), data sovereignty for cross-border deployments.

**UX users expect:** Real-time device map, telemetry time-series charts with zoom/pan, alert list with severity and acknowledgment, live device metrics, fleet health scores, firmware rollout tracker, threshold configuration.

---

## 16. Restaurant / Food Service

**Archetype:** Restaurant operator managing menus, orders, tables, kitchen workflow, and multi-platform delivery.

**Core entities:** Menu Item, Modifier/Addon, Order (dine-in/takeout/delivery), Table, Reservation, Ingredient/Inventory, Shift, POS Terminal

**Gotchas:**
- **Menu structure is deeply nested** — Category > Subcategory > Item > Size > Modifiers (with modifier groups: "Choose protein" allows 1, "Choose toppings" allows up to 3). Modifiers have prices, some are free, some required, availability may differ by item.
- **Kitchen Display requires order routing** — a single order's items may go to different stations (grill, fryer, bar). Each station sees only its items, tracks prep time, and coordinates firing (apps before entrees).
- **Menu availability is time-sensitive** — breakfast menu until 11 AM, happy hour 4-6 PM, 86'd items (out of stock mid-service). The POS must prevent ordering unavailable items in real-time.
- **Ingredient-level inventory affects menu availability** — running out of chicken affects 12 menu items. The system must trace ingredient-to-item relationships.
- **Split checks and partial payments are the norm** — "Split 4 ways," "pay for my items only," "put $30 on this card and the rest on that." Tip distribution across splits adds complexity.
- **Delivery integration is multi-platform** — Uber Eats, DoorDash, Grubhub arrive through different APIs with different menu mappings, commissions, and delivery areas.

**Compliance:** Food safety (HACCP, FDA Food Code), allergen disclosure (EU requires 14 allergen categories), calorie labeling, tip reporting (IRS), liquor licensing.

**UX users expect:** Visual menu builder with modifier groups, table map with status colors, KDS ticket display with timers, 86 board, daily sales summary with prior period comparison, labor cost gauge, delivery aggregation view.

---

## 17. Legal / Law Firm

**Archetype:** Law firm managing clients, cases, documents, time billing, and trust accounting.

**Core entities:** Client/Matter, Case, Document, Time Entry, Trust Account (IOLTA), Invoice, Court Deadline, Conflict Check

**Gotchas:**
- **Conflict of interest checking is mandatory** — before engagement, search all current and past clients, opposing parties, and related entities. A missed conflict means malpractice and bar discipline. Must be documented.
- **Trust accounting (IOLTA) has strict rules** — client funds cannot commingle with operating funds. Each client's trust balance is tracked individually. Over-disbursing is a disbarrable offense. The trust ledger must reconcile to the penny monthly.
- **Legal billing is idiosyncratic** — hourly in 6-minute increments (0.1 hour), flat fee, contingency, blended rates, LEDES format for corporate clients, pre-bills (drafts partners edit before sending). Time narratives must be detailed but not reveal privileged info.
- **Privilege classification governs document access** — documents are privileged, work product, or neither. During litigation, a privilege log must list all withheld documents. Inadvertent disclosure is a crisis.
- **Statute of limitations tracking is malpractice prevention** — calculate deadlines from triggering events, account for tolling and jurisdiction-specific rules. Provide escalating reminders. Missing a deadline is the #1 malpractice claim.
- **Document management needs version control with metadata** — drafts, redlines, executed copies, court-filed versions, opposing counsel versions must be distinguishable.

**Compliance:** Bar association rules (vary by state), ABA Model Rules, IOLTA trust rules, client confidentiality, e-discovery requirements, LEDES billing standards.

**UX users expect:** Conflict check search, trust ledger with per-client sub-ledgers, time entry with timer and 0.1-hour increments, pre-bill editing workflow, matter-centric document management, court deadline calendar with countdown.

---

## 18. Non-profit / Fundraising

**Archetype:** Non-profit staff managing donors, donations, grants, programs, and fund restrictions.

**Core entities:** Donor, Donation/Gift, Campaign/Appeal, Grant, Program/Project, Beneficiary, Volunteer, Fund/Restriction

**Gotchas:**
- **Fund accounting is fundamentally different from corporate accounting** — track by restriction: unrestricted, temporarily restricted (donor-specified purpose), permanently restricted (endowment principal never spent). Spending a restricted gift on operations is a legal violation.
- **Donor stewardship has lifecycle expectations** — acknowledgment within 48 hours, year-end tax receipt (IRS requires written acknowledgment for gifts >$250), impact report showing gift use. Missing the window damages relationships and may violate IRS rules.
- **Grant management is compliance-heavy** — proposal > award > reporting. Grants have spending restrictions, reporting deadlines, matching requirements, and indirect cost caps. Expenses must be grant-allocable.
- **Recurring giving has unique churn dynamics** — credit card expiration is the #1 failure cause. Card updater services, retry logic (1st and 15th), and donor notification workflows are essential.
- **In-kind donations and volunteer hours need valuation** — donated goods must be valued at fair market value for tax receipts. Getting valuations wrong creates IRS problems.
- **Beneficiary data has extra sensitivity** — vulnerable populations (refugees, domestic violence survivors) require stronger privacy. Case worker access must be role-restricted, some data (shelter addresses) must never appear in reports.

**Compliance:** IRS 501(c)(3) rules, Form 990, grant compliance (OMB Uniform Guidance for federal grants), state charitable solicitation registration, GDPR for international donors, quid pro quo donation rules.

**UX users expect:** Donor giving history timeline, acknowledgment letter generator, fund balance dashboard (restricted vs. unrestricted), grant deadline tracker, campaign thermometer, recurring giving dashboard with failed payment alerts, volunteer hours logging.

---

## 19. Media / Streaming

**Archetype:** Content team managing a media catalog, licensing rights, distribution, and subscriber analytics.

**Core entities:** Title/Asset, Episode/Season/Series, License/Rights Window, Catalog Entry, Subscription Tier, Viewing Session, Content Rating, Distribution Channel

**Gotchas:**
- **Content rights are territorial and temporal** — a movie may be licensed for US streaming Jan-Dec 2026, UK from March, unavailable in Germany. Every availability check evaluates: territory + date range + platform + rights type (SVOD/TVOD/AVOD). Expired rights must auto-remove content.
- **Metadata standards are industry-specific** — EIDR for title IDs, per-territory content ratings (PG-13 US, 12A UK, FSK 12 Germany), available subtitles/dubs, audio formats (Stereo, 5.1, Atmos), HDR formats, aspect ratios.
- **Content ingestion is a pipeline, not an upload** — source master > quality check > transcode to multiple formats/bitrates > generate thumbnails > extract captions > DRM packaging > CDN. Each step can fail. A single title may produce 20+ output files.
- **Parental controls and ratings require player-level enforcement** — age-gated profiles, PIN for mature content, kids profiles must never serve tracked ads (COPPA).
- **Playback analytics drive the business** — start rate, completion rate, drop-off points, buffering events, concurrent stream limits, device fingerprinting for DRM. High-volume event data.
- **Content windowing is business-critical** — theatrical > premium VOD > standard VOD > pay TV > free. The dashboard must manage release windows and prevent premature availability.

**Compliance:** COPPA (kids content), regional rating board requirements, accessibility (audio description, closed captions mandated), GDPR for viewing history, royalty reporting to rights holders.

**UX users expect:** Catalog browser with territory/rights filters, rights availability calendar heatmap, ingestion pipeline status, viewing analytics with engagement curves, subscriber cohorts, content performance rankings, editorial curation tools.

---

## 20. Agriculture / Farm Management

**Archetype:** Farm operator managing fields, crops, livestock, inputs, yields, and compliance records.

**Core entities:** Field/Plot, Crop/Variety, Livestock/Herd, Input Application (seed/fertilizer/pesticide), Harvest/Yield, Weather Data, Equipment, Compliance Record

**Gotchas:**
- **Spatial data is fundamental** — fields are polygons, not names. Soil types vary within a field. Variable-rate application requires GIS integration. The dashboard needs map-based visualization with overlays.
- **Crop rotation is a multi-year dataset** — what was planted for the last 5 years affects what can be planted this year. The data model must track field-year-crop relationships across seasons.
- **Input application records are regulatory** — every pesticide application must record: product, EPA registration number, rate per acre, application method, wind speed, temperature, applicator certification, restricted entry interval. Missing any field means fines.
- **Weather integration is essential** — growing degree days, precipitation, frost dates, evapotranspiration drive decisions. On-farm weather station data must ingest automatically.
- **Livestock tracking is individual-animal granular** — ear tag/RFID, breed, birthdate, lineage, vaccinations, weight history, health events, movement history. For dairy: individual milk production records.
- **Harvest must reconcile with storage and sales** — bushels from Field 3 go to Bin A, sold to Elevator B at a specific grade and moisture. Moisture adjustment, drying costs, and basis pricing are standard calculations.

**Compliance:** EPA pesticide reporting, USDA organic certification (3-year transition), FSMA Produce Safety Rule, animal welfare regulations, water use permits, nutrient management plans, country-of-origin labeling, traceability (EU Farm to Fork).

**UX users expect:** Map-centric interface with field polygons, field activity timeline, input application form with regulatory fields, yield comparison (this year vs. 5-year average), livestock cards, weather dashboard with GDD accumulation, grain bin tracker.

---

## 21. AI / ML / Chat

**Archetype:** Team managing AI models, prompts, conversations, usage/cost tracking, and evaluation pipelines.

**Core entities:** Model/Deployment, Prompt/Template, Conversation/Thread, Message, Usage Record (tokens/cost), Evaluation/Benchmark, Knowledge Base/RAG Source, Fine-tune Job

**Gotchas:**
- **Token cost tracking requires per-request granularity** — input tokens, output tokens, cached tokens, and model-specific pricing all differ. Aggregate by user, by feature, by model, by time period. Costs can spike 100x overnight if a prompt change increases output length. Budget alerts and per-user/per-org rate limits are essential.
- **Streaming responses require a fundamentally different UX pattern** — the response builds token-by-token. The UI must handle: streaming display (token-by-token or chunk-by-chunk), cancellation mid-stream (AbortController), retry on failure mid-stream, and saving the final complete response. Standard request/response patterns don't work.
- **Prompt versioning is as critical as code versioning** — a prompt change can silently degrade output quality across the product. Track prompt versions, link them to evaluation scores, and support rollback. Treat prompts as deployment artifacts, not strings in a database.
- **Conversation history has context window limits** — you cannot send the entire conversation to the model forever. Implement truncation strategies (sliding window, summarization, hybrid) and make the strategy visible/configurable. Users will notice when the model "forgets."
- **Evaluation is not optional and is domain-specific** — "accuracy" means different things for summarization vs. code generation vs. classification. Build evaluation pipelines with domain-appropriate metrics, human review workflows, and A/B testing between prompt versions or models.
- **RAG (Retrieval-Augmented Generation) adds a whole subsystem** — document ingestion, chunking strategy, embedding generation, vector storage, retrieval quality measurement, source attribution in responses. Each has tuneable parameters that dramatically affect quality.
- **Content moderation is both input and output** — filter harmful inputs before they reach the model AND filter harmful outputs before they reach the user. Log moderation events. False positives frustrate users; false negatives create liability.
- **Model availability and latency vary unpredictably** — different models have different rate limits, different uptime, different latency profiles. Build fallback chains (try Model A, fall back to Model B), queue management, and latency monitoring per model.

**Compliance:** Data processing agreements with model providers, GDPR (conversation data is personal data if it contains PII), EU AI Act (risk classification, transparency requirements), CCPA, industry-specific rules if AI processes financial/medical/legal data, content moderation obligations.

**UX users expect:** Chat interface with streaming responses, conversation history with search, prompt playground/testing sandbox, usage/cost dashboard with per-model breakdown, model performance comparison charts, knowledge base/document management for RAG, evaluation results dashboard, system prompt editor with version history.

---

## 22. Entertainment / Events

**Archetype:** Event organizer or venue managing shows, ticketing, artists, and attendee experience.

**Core entities:** Event, Venue/Location, Ticket/Tier, Performer/Artist, Attendee, Seating Map, Promotion/Discount, Settlement/Payout

**Gotchas:**
- **Seating maps are spatial data with business logic** — reserved seating requires a visual seat picker where each seat has a section, row, seat number, price tier, accessibility designation, and obstructed-view flag. Hold-release patterns (seats held for 10 minutes during checkout) prevent double-selling.
- **Ticket inventory has high-concurrency pressure** — 10,000 people trying to buy 500 tickets in 30 seconds. Standard database locks won't hold. Queue-based purchase flows, waiting rooms, and atomic inventory decrement (optimistic locking or database-level constraints) are required.
- **Dynamic/surge pricing is expected** — prices change based on demand, time-to-event, and remaining inventory. Early bird, last-minute, and demand-based tiers. The pricing engine must be fast and auditable.
- **Refund and transfer policies vary per event** — some events allow transfers (name change on ticket), some allow refunds until X days before, some are final sale. Policies must be per-event configurable.
- **Settlement/payout to artists and venues is a financial workflow** — ticket revenue minus fees, taxes, and venue costs, split among multiple parties. This is accounting with multiple payees per event.
- **Check-in and access control are real-time** — QR code/barcode scanning at doors, preventing duplicate entry (a scanned ticket can't enter twice), VIP vs. GA routing, and real-time attendance counts.
- **Multi-day and recurring events add calendar complexity** — festivals with multiple stages and overlapping acts, weekly show series, season passes valid for specific dates.

**Compliance:** ADA accessibility for venues, local fire code capacity limits, sales tax on tickets (varies by jurisdiction), age restrictions for certain events, anti-scalping laws (jurisdiction-specific), payment card industry compliance.

**UX users expect:** Visual seating map editor, drag-and-drop event scheduling, ticket sales dashboard with real-time counters, check-in scanner interface (mobile-optimized), settlement/payout breakdown per event, marketing campaign tracking (promo code performance), attendee demographics.

---

## 23. Gaming / Esports

**Archetype:** Game studio or platform managing players, matches, virtual economies, seasons, and competitive integrity.

**Core entities:** Player/Account, Match/Game Session, Leaderboard/Ranking, Virtual Item/Currency, Season/Battle Pass, Tournament/Bracket, Ban/Sanction, Achievement

**Gotchas:**
- **Virtual economies require fintech-level rigor** — in-game currency (earned and purchased), item trading, marketplace transactions. Use integer math for all currency. Duplication exploits (creating currency/items from nothing) are the #1 game-breaking bug. Transaction logs must be append-only and auditable.
- **Matchmaking is a real-time system with fairness constraints** — skill-based rating (Elo, Glicko-2, TrueSkill), queue times vs. match quality trade-offs, party/group matching, regional latency requirements, and smurf detection (high-skill players on new accounts).
- **Leaderboards at scale need specialized data structures** — sorted sets (Redis ZSET) or materialized views, not `ORDER BY score DESC LIMIT 100` on every page load. Real-time vs. periodic refresh. Per-region, per-season, per-mode leaderboards multiply the problem.
- **Season/battle pass progression is a time-bound engagement system** — XP accumulation, tier rewards, premium vs. free track, catch-up mechanics for late joiners, end-of-season reward distribution. Resetting and archiving season data while preserving rewards is a migration event.
- **Anti-cheat and ban management is an ongoing war** — detection signals (impossible stats, speed hacks, aimbot patterns), ban types (temporary, permanent, shadow ban, hardware ban), appeal workflows, and ban evasion detection (same hardware/IP with new account).
- **Loot box / gacha compliance varies by country** — Belgium and Netherlands ban paid loot boxes, Japan regulates "kompu gacha," China requires published drop rates, Apple/Google require disclosure. The same game may need different monetization per region.
- **Player data for minors triggers COPPA/GDPR-K** — under-13 accounts need parental consent, restricted chat, restricted spending, no behavioral advertising. Age gating must be real, not a "click yes" checkbox.
- **Real-time multiplayer state is not a database concern** — match state lives in game servers, not in PostgreSQL. The dashboard reads from match result records, player stats aggregations, and event streams. Don't try to query live match state from the admin panel.

**Compliance:** COPPA (minors), GDPR (player data, right to deletion of accounts), loot box regulations (Belgium, Netherlands, Japan, China), ESRB/PEGI rating compliance, gambling regulations (if real-money is involved), platform ToS (Steam, PlayStation, Xbox, App Store policies).

**UX users expect:** Player lookup with match history, ban/moderation queue with evidence viewer, leaderboard browser by season/region/mode, virtual economy dashboard (currency in circulation, inflation metrics), tournament bracket editor, season pass progression analytics, report/appeal workflow, real-time concurrent player counts.

---

## 24. Cybersecurity / SOC

**Archetype:** Security team monitoring threats, investigating incidents, managing vulnerabilities, and tracking compliance.

**Core entities:** Alert/Event, Incident, Vulnerability, Asset/Host, Threat Intelligence Feed, Investigation/Case, Policy/Rule, Compliance Control

**Gotchas:**
- **Alert volume is overwhelming by design** — a SOC receives thousands to millions of events per day. Without correlation (grouping related alerts into incidents), deduplication, and automated triage, the dashboard is unusable. Build for filtering and prioritization, not for showing everything.
- **MITRE ATT&CK framework is the shared language** — every alert and incident should map to ATT&CK tactics and techniques. Analysts expect to see TTPs (Tactics, Techniques, Procedures), not just raw log lines.
- **Mean Time to Detect (MTTD) and Mean Time to Respond (MTTR) are the KPIs** — the dashboard must track these per incident type, per analyst, over time. They drive staffing and process decisions.
- **Investigation workflows are non-linear** — an analyst pivots from an IP to a host to a user to a file hash to another host. The dashboard must support this "investigation graph" pattern — click an IOC (Indicator of Compromise), see everything related.
- **Threat intelligence feeds are external data integrations** — STIX/TAXII format feeds, commercial threat intel (Recorded Future, CrowdStrike), OSINT feeds. IOCs (IPs, domains, file hashes) must be enriched and correlated against internal telemetry.
- **Chain of custody matters for forensics** — evidence collected during an investigation (memory dumps, disk images, log exports) must be hashed, timestamped, and stored immutably. This is legal evidence.
- **Compliance posture tracking is continuous** — SOC 2, ISO 27001, NIST CSF, PCI DSS, HIPAA controls mapped to evidence. Each control has a status (met/partially met/not met) with supporting evidence and review dates.

**Compliance:** SOC 2, ISO 27001, NIST CSF, PCI DSS, HIPAA (if healthcare data), GDPR (incident notification within 72 hours), industry-specific regulations, breach notification laws (vary by state/country).

**UX users expect:** Alert queue with severity/priority, incident timeline with event correlation, threat map (geographic), MITRE ATT&CK matrix heatmap, vulnerability scanner results with CVSS scores, asset inventory with risk scores, compliance control matrix with evidence status, runbook/playbook execution tracker.

---

## 25. Construction / Field Services

**Archetype:** Construction firm or field service company managing jobs, crews, inspections, permits, and equipment.

**Core entities:** Project/Job Site, Work Order, Crew/Worker, Inspection, Permit, Equipment/Asset, Daily Log, Change Order, Drawing/Plan

**Gotchas:**
- **The dashboard must work offline and on mobile** — field workers are on job sites with poor connectivity. Forms for daily logs, inspections, and time tracking must work offline and sync when connectivity returns. This is a fundamental architecture decision, not a nice-to-have.
- **Change orders are how construction projects actually work** — the original scope changes constantly. Each change order has a cost impact, schedule impact, approval workflow, and audit trail. The budget is the original contract + sum of approved change orders. Tracking "budget vs. actual" without change orders is meaningless.
- **Permit tracking has jurisdictional dependencies** — different municipalities have different permit types, fee structures, inspection requirements, and processing times. A permit's status (applied > under review > approved > inspection scheduled > passed) is a state machine with external dependencies.
- **Safety compliance is non-negotiable and heavily regulated** — OSHA incident reporting (within 24 hours for hospitalizations, 8 hours for fatalities), daily safety logs, toolbox talks, PPE tracking, and near-miss reporting. The dashboard must make safety reporting fast and frictionless.
- **Drawing/plan management requires version control with superseding** — Revision A is superseded by Revision B. Workers must always see the latest revision. Building from an outdated drawing is a catastrophic (and common) mistake.
- **Progress tracking is visual and percentage-based** — "Foundation 100%, Framing 60%, Electrical 20%, Plumbing 0%." Progress photos linked to dates and locations. Earned value management (EVM) for financial progress vs. schedule progress.
- **Prevailing wage requirements apply to government contracts** — Davis-Bacon Act (federal), state prevailing wage laws. Workers on covered projects must be paid specific rates by trade classification. Tracking certified payroll is a reporting requirement.

**Compliance:** OSHA (safety), Davis-Bacon (prevailing wage for government work), state contractor licensing, building codes (vary by jurisdiction), environmental regulations (stormwater, hazmat), lien laws and payment bond requirements.

**UX users expect:** Mobile-first daily log entry, photo capture with GPS tagging, punch list with completion tracking, Gantt chart with critical path, equipment utilization calendar, safety incident dashboard, budget tracker with change order waterfall, drawing viewer with revision indicator.

---

## 26. Marketplace / Platform

**Archetype:** Two-sided platform connecting buyers and sellers (or providers and consumers), managing trust, transactions, and disputes.

**Core entities:** Seller/Provider, Buyer/Consumer, Listing, Transaction/Order, Review/Rating, Dispute, Commission/Fee, Payout, Verification

**Gotchas:**
- **Two-sided means two dashboards (at least)** — sellers see their listings, orders, payouts, and performance. Buyers see their orders, reviews, and saved items. Platform admins see everything plus disputes, compliance, and marketplace health. Three distinct RBAC contexts, not one.
- **Trust and safety is a dedicated function** — user verification (ID, business license, background check), content moderation (listing photos, descriptions), fraud detection (fake reviews, shill bidding, payment fraud), and abuse reporting. This is not an afterthought; it's a full team's worth of tooling.
- **Escrow/payment splitting is complex** — buyer pays > platform holds > seller fulfills > platform releases to seller minus commission. Refund flows, dispute holds, partial fulfillment, split payments to multiple sellers in one order. You need a payment orchestration layer (Stripe Connect, PayPal for Marketplaces).
- **Commission structures are rarely simple** — they vary by category, by seller tier, by volume, by promotional period. "15% on electronics, 8% on books, 5% for Gold sellers, 0% for the first 90 days" is a real commission table.
- **Search and discovery are core product features** — not just a search bar. Relevance ranking, category navigation, filters (price, location, rating, availability), promoted/sponsored listings, and personalized recommendations. Search quality directly drives GMV.
- **Dispute resolution is a three-party workflow** — buyer claims item not received, seller provides tracking, platform arbitrates. Escalation tiers, evidence submission, resolution deadlines, and appeal processes. The dispute dashboard is as important as the order dashboard.
- **Review integrity affects the entire marketplace** — fake review detection, review gating (only verified purchasers), review response from sellers, rating aggregation with recency weighting, and review moderation for prohibited content.
- **Geographic and regulatory complexity** — a marketplace operating in multiple countries must handle: local payment methods, local tax collection and remittance (marketplace facilitator laws in the US), local consumer protection, and local seller verification requirements.

**Compliance:** Marketplace facilitator tax laws (US, state-by-state), consumer protection (EU Consumer Rights Directive, right of withdrawal), payment services regulations (PSD2 in EU), KYC/AML for seller onboarding, digital services act (EU, content moderation obligations), platform liability laws.

**UX users expect:** Seller dashboard with earnings/payouts/performance, buyer order tracking, admin moderation queue, dispute resolution workflow with evidence viewer, marketplace health dashboard (GMV, take rate, seller churn, buyer satisfaction), category management, search analytics (top queries, zero-result queries), promoted listing management.

---

## Cross-cutting patterns

Several patterns emerged across all 26 domains. If your domain isn't listed above, check whether these apply — they almost certainly do:

1. **State machines, not status columns** — every domain has at least one entity with a lifecycle more complex than a simple enum. Orders, tickets, cases, bookings, grants, permits, matches — all have conditional transitions, parallel states, and reversal paths. Model them explicitly.

2. **Temporal complexity beyond timestamps** — business hours, fiscal periods, timezone-aware deadlines, seasonal schedules, effective dates, rights windows, SLA clocks that pause. Almost no domain uses simple datetime comparisons.

3. **Multi-dimensional pricing** — nearly no domain has a single price field. Pricing varies by time, geography, customer tier, channel, volume, and promotional period. A "price" column is almost always wrong.

4. **Compliance causes rewrites, not technical debt** — technical shortcuts are tolerable; compliance violations are not. The domains with the harshest consequences (healthcare, finance, legal, HR, gaming with minors) are where "we'll add that later" is most dangerous.

5. **Append-only audit trails** — financial, healthcare, legal, HR, cybersecurity, and gaming domains all require immutable records. Soft-delete is insufficient; you need reversing entries and full audit logs.

6. **Domain-specific code systems** — ICD-10 (medical), CPT (procedures), NAICS (industry), HS codes (customs), LEDES (legal billing), STIX (threat intel), EIDR (entertainment). If your domain has a standard taxonomy, use it — don't invent your own.

7. **Two audiences, one platform** — marketplaces, LMS (student/teacher), healthcare (patient/provider), support (agent/customer), and gaming (player/admin) all have at least two fundamentally different user types that need different views of the same data.

8. **Offline/mobile-first for field domains** — construction, agriculture, logistics, sports, and restaurant (KDS/POS) all have users who are not sitting at a desk with good WiFi. If the dashboard doesn't work on a phone with spotty connectivity, it doesn't work.
