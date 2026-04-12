---
name: dashboard-design
description: "Build production-grade, end-to-end connected dashboards — admin panels, analytics consoles, internal tools, operational control centers, SaaS back-offices — across any stack (React/Next/Vue/Svelte/Remix/Rails/Django/Laravel/etc.). Use this skill whenever the user asks for a 'dashboard,' 'admin panel,' 'control panel,' 'back office,' 'internal tool,' 'analytics view,' 'reporting interface,' 'ops console,' 'metrics view,' 'KPI view,' or describes building any multi-page interface that displays domain data with navigation, auth, and CRUD — even if the word 'dashboard' is never spoken. Triggers also include requests to add a sidebar nav, role-based access, user management, audit logs, charts/graphs over real data, filters, exports, settings pages, or any 'logged-in area' of an app. This skill enforces a no-scaffold-no-placeholder rule — every feature shipped must be wired end-to-end to a real backend (or a real local persistence layer), not stubbed with TODO, fake JSON, or 'you can hook this up later' comments."
---

# Dashboard Design

This skill exists to solve one specific failure mode: AI-generated dashboards that *look* finished but are hollow. Buttons that don't save. Filters that don't filter. "Settings" pages with no persistence. Login screens that accept anything. Charts hardcoded to fake JSON. Sidebars with five items where four route to 404. The user runs the result and within ten seconds discovers it's a museum, not a tool.

The job here is the opposite: ship a dashboard where every visible element is connected to something real. If a button is on the screen, clicking it does the thing. If a chart is on the screen, it reflects actual data from the actual store. If there's a "users" page, you can actually create, edit, delete, and assign roles to users — and those changes persist across reloads.

This is harder than it sounds, because dashboards are *systems*, not screens. A dashboard touches the data model, the API, auth, RBAC, navigation, state, caching, forms, charts, empty states, error states, audit, and exports — and if any one of those layers is faked, the whole thing collapses into a demo. This skill provides the discipline to build all of them.

## Core principle: vertical slices, not horizontal layers

Do not build the dashboard layer-by-layer (database → API → auth → routes → UI → charts). That path produces 80% completion on every layer and 0% completion on any actual feature. Instead, build it feature-by-feature as **vertical slices**:

> One feature at a time, end-to-end, working, before moving on.

A vertical slice for "user management" means: schema migration → API endpoints → permission checks → service layer → query hooks → list page → detail page → create/edit form → delete confirmation → empty state → error state → loading state → audit log entry → tested with two seeded users — *all done* before touching the next feature. When the slice is complete, an actual person can do an actual job with it.

This principle is non-negotiable. It is the single biggest determinant of whether the output is a working dashboard or a scaffold.

## When this skill triggers, do this in order

Follow this sequence. Skipping steps is how dashboards end up hollow.

### 1. Run the pre-flight (mandatory)

Before writing a single line of code, read **`references/preflight-and-verification.md`** and answer the questions in it. The pre-flight establishes: what the dashboard is *for*, who uses it, what the stack is, what the data source is, what auth model exists, what already exists vs. what must be built, and what "done" looks like. You cannot build a connected dashboard without this — you'd be guessing at every fork.

Once you've identified the domain (question #1), read the matching section in **`references/domain-considerations.md`**. It covers 31 verticals with domain-specific data model traps, compliance requirements, and UX patterns that generic CRUD misses. Skipping this is how dashboards end up with float-based currency, single-entry accounting, or missing HIPAA audit logs.

If the user's request is vague ("build me a dashboard"), do not invent a domain. Pick the single most plausible interpretation, state your assumptions in one short paragraph, and proceed. Do not pepper the user with twenty questions. One pass of stated assumptions is enough; they'll redirect if needed.

### 2. Decide the architecture in writing

Produce a short architecture note (5–15 lines, inline in the chat or as a top-of-file comment) covering:

- **Stack** — framework, language, ORM/DB, auth provider, styling, state library, chart library
- **Data source** — real DB? existing API? if neither exists yet, what local persistence will simulate "real" (SQLite + Prisma, Postgres in Docker, Convex, Supabase, etc.)
- **Auth model** — sessions vs. JWT, where the user record lives, how roles attach to users
- **Permission model** — RBAC role list and the resource × action matrix (see `references/auth-and-rbac.md`)
- **Route map** — every page that will exist, with parent → child nesting

Showing this up front prevents the most common failure: building three pages with three incompatible auth assumptions and discovering it on page four.

### 3. Build the foundation slice first

The foundation slice is always the same and must come first:

1. Project bootstrapped, runs locally
2. Database/persistence layer up, migrations applied, seed script with at least one admin user and 5–20 rows of realistic sample data per main entity
3. Auth working — login, logout, session persistence, password hashing, "you must be logged in to see anything" middleware
4. RBAC working — at least two roles (`admin`, `member`), middleware that checks permissions on every protected route, a way to test "what does a non-admin see?"
5. Shell layout working — header with logo + user menu, sidebar nav, content area, breadcrumbs, responsive collapse
6. One real protected page reachable end-to-end (the dashboard home, even if it just shows "Welcome, {user.name}")

Do not move past the foundation until you can: run the app, sign in as the admin, sign out, sign in as a non-admin, and see different things. This is the proof the foundation is real.

### 4. Build feature slices

For each feature (user management, the orders table, billing, settings, etc.), build the full vertical slice. The recipe:

1. Schema/migration if new entities are needed
2. Seed data if the entity is new
3. Server-side: list endpoint (with pagination, filtering, sorting), detail endpoint, create, update, delete — all with permission checks
4. Client-side: query hooks for read, mutation hooks for write, with cache invalidation
5. Pages: list page (with empty/loading/error states), detail page, create/edit form (with validation, submission states, success/error feedback), delete confirmation
6. Audit: every mutation writes an audit log entry
7. Smoke test: do the create→edit→delete loop manually before declaring the slice done

The order of feature slices should be: most-used feature first, riskiest feature second, "nice to have" features last. If the user runs out of patience halfway through, they should still have the most important feature working completely.

### 5. Layer in cross-cutting concerns

After two or three feature slices, add the cross-cutting layer: global search, notifications, user preferences/settings, theme toggle, exports, audit log viewer, profile page. These are easier to do once you have real data flowing — building them on top of empty pages is wasted effort.

### 6. Write tests alongside each slice

A vertical slice isn't done until it has tests. Read `references/testing-and-quality.md` before writing the first test. At minimum, each slice should have:

- An integration test that walks the CRUD flow end-to-end
- A permission test that confirms non-admin users are correctly blocked
- An accessibility test (axe) on every new page

Don't delay testing until "after the build." A dashboard without tests is a dashboard that breaks silently the next time someone changes a query key or a permission rule.

### 7. Harden performance and security

Before the verification pass, read `references/performance-and-security.md` and apply the relevant hardening:

- Set security headers (CSP, HSTS, X-Frame-Options)
- Verify bundle size is under budget (code-split, lazy-load heavy libraries)
- Confirm rate limiting on login, mutations, uploads, and exports
- Run `npm audit` (or equivalent) and fix critical/high vulnerabilities
- Check that no secrets are in git and no server secrets leak to the client bundle

### 8. Run the verification checklist

Before declaring the dashboard "done," walk the verification checklist in `references/preflight-and-verification.md` and the security checklist in `references/performance-and-security.md`. They catch the specific failure modes that make dashboards feel hollow or insecure: dead links, fake data, permissions you forgot to check, missing empty states, console errors, XSS vectors, missing headers, etc.

## The "haves" — every dashboard must have all of these

Memorize this list. If any item is missing when you stop coding, the dashboard isn't done.

1. **Real auth.** A login page that rejects bad credentials, stores a session, and gates every protected route on the server side (not just hidden links in the UI).
2. **Real RBAC.** At least two roles. Server-side permission checks on every mutation. UI that hides actions a user can't perform — but never *only* hides them; the server enforces.
3. **Real data.** Seeded into a real persistence layer. Reloading the page does not reset the data. Two browsers see consistent state.
4. **A shell.** Header (logo top-left, user menu top-right), persistent sidebar (collapsible on mobile), main content area, breadcrumbs on nested pages.
5. **Working navigation.** Every link in the sidebar goes somewhere real. Active state highlights the current page. Sub-menus expand/collapse and remember their state.
6. **A landing page.** The route after login is not a 404 and not "lorem ipsum." It shows something specific and useful — typically 3–6 KPI cards plus one or two key charts plus a recent-activity feed.
7. **At least one full CRUD entity.** List, detail, create, edit, delete — all wired, all persisted, all permissioned.
8. **Loading states, empty states, and error states for every async surface.** Not spinners-everywhere; specific states that explain what's happening.
9. **Form validation.** Client-side for instant feedback, server-side for truth. Errors shown inline next to the field.
10. **Success and error feedback.** Toasts or inline banners after every mutation. Never silent.
11. **Pagination, sorting, and filtering on any table that could exceed 25 rows.** Server-side once it could exceed 200.
12. **A settings page** that actually saves.
13. **A user profile / account page** the logged-in user can edit.
14. **An audit log** of who did what, when. Even a minimal one.
15. **Keyboard accessibility.** Every interactive element reachable by Tab. Focus styles visible. No keyboard traps.
16. **Responsive layout.** Works at 1440, 1024, 768, and 375 wide. Sidebar collapses to a drawer on mobile.
17. **A logout button** that actually invalidates the session.
18. **Tests.** At minimum: auth flow (login, logout, reject bad credentials), one full CRUD flow, one permission denial test, and axe accessibility checks on every page. A dashboard without tests is a dashboard that breaks silently.
19. **Security headers.** CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy set on every response. No secrets in git. Rate limiting on login and mutations.

## The "have nots" — things that disqualify the dashboard

If any of these appear, the dashboard fails the no-scaffold rule and must be fixed before stopping:

- `// TODO`, `// implement later`, `// hook up to API`, `// FIXME` in any file
- `console.log` left in production code paths
- Hardcoded arrays of fake data inside React components (data must come from the data layer)
- `Math.random()` driving any chart or KPI
- `alert()` or `prompt()` for user interaction
- "Coming soon" pages or disabled nav items
- Permission checks that exist *only* on the client (server must enforce)
- Forms that POST and ignore the response
- Buttons whose `onClick` is `() => {}`, `undefined`, or `console.log("clicked")`
- Routes in the sidebar that 404
- Charts that re-randomize on every render
- Login pages that accept any credentials
- Any "demo mode" or "this is a placeholder" copy visible to the user
- Inline styles for what should be design tokens (when the project has a design system)
- Components named `ExampleX`, `DemoY`, `TestZ` shipped to the user
- Dependency on a service the user hasn't been told they need to set up

When you catch yourself about to write any of these, stop and do the real version instead. The real version is almost always 10–30 more lines of code, not 10x more.

## The "dos"

- **Do** seed the database with realistic data. Not `User 1, User 2, User 3` — real-looking names, real-looking emails, realistic timestamps, realistic distributions. Faker libraries exist; use them.
- **Do** put the logo top-left and the user menu top-right. This is where every user's eyes look. Deviating is a costly novelty.
- **Do** group nav items by frequency of use, not alphabetically. Daily-use items at the top, monthly-use in the middle, settings at the bottom.
- **Do** use a server-state library (TanStack Query, SWR, RTK Query, Apollo, urql, or framework-native equivalents) for any non-trivial dashboard. Hand-rolling `useEffect + fetch + useState` for each page is how dashboards get inconsistent and slow.
- **Do** invalidate caches after mutations so the list reflects what just happened. This is the #1 "feels broken" bug.
- **Do** show optimistic updates for actions where the success rate is >95% (toggles, simple edits). Roll back on error.
- **Do** debounce search inputs (250–400ms) and cancel in-flight requests when a new one starts.
- **Do** persist table state (sort, filter, page) in the URL query string so reloads and shares work.
- **Do** put primary actions in the same place on every page (top-right of the page header is the convention).
- **Do** use a single design token system (CSS variables or a theme object). One source of truth for color, spacing, radius, font.
- **Do** pick a chart library and stick to it for the whole dashboard. Mixing Recharts, Chart.js, and ApexCharts is visual chaos.
- **Do** show data freshness ("Updated 2 min ago") on dashboards that use cached or polled data. Users need to know if they're looking at stale state.
- **Do** make destructive actions require confirmation, and make the confirmation type the resource name for irreversible ones.

## The "don'ts"

- **Don't** invent a brand-new layout pattern. Sidebar-left + topbar + content is a convention because it works. Reinvent only when the domain demands it.
- **Don't** put more than 7 items in the top level of the sidebar. Group the rest under sections or move them to settings.
- **Don't** use a pie chart with more than 4 slices. Use a horizontal bar chart instead.
- **Don't** use 3D charts. Ever.
- **Don't** use rainbow palettes for sequential data. Use a single-hue gradient.
- **Don't** display more than 4 KPI cards above the fold on the landing page. More than that and they become wallpaper.
- **Don't** auto-refresh the entire dashboard on a timer unless the use case is operational/realtime monitoring. Let the user pull-to-refresh or use targeted polling per widget.
- **Don't** use modals for anything that needs more than ~5 form fields. Use a side drawer or a dedicated page.
- **Don't** hide critical information behind tooltips. Tooltips are for clarification, not primary content.
- **Don't** use icons without labels in primary navigation. Icon-only nav is for power users who already know the app.
- **Don't** ship a dashboard without testing it on a small viewport. The mobile breakage is always worse than you think.
- **Don't** trust client-supplied IDs in mutations. The server resolves the current user from the session, not from the request body.

## Reference files — load on demand

The body above is enough to start building. For depth on a specific domain, load the matching reference file. Read each file *before* implementing that layer, not after — the cost of re-doing a layer is much higher than the cost of reading 300 lines.

| Reference file | Read before |
|---|---|
| `references/preflight-and-verification.md` | **Always**, at the very start, and again at the end |
| `references/information-architecture.md` | Designing the shell, nav, menus, layout, page structure |
| `references/auth-and-rbac.md` | Implementing login, sessions, roles, permissions, user management |
| `references/data-layer.md` | Wiring API contracts, query/mutation hooks, caching, websockets, file upload |
| `references/data-visualization.md` | Choosing chart types, building KPI cards, designing tables |
| `references/states-and-feedback.md` | Building loading, empty, error states, toasts, optimistic updates |
| `references/workflows-and-actions.md` | Designing forms, multi-step flows, bulk actions, confirmations, exports |
| `references/analytics-and-telemetry.md` | Adding event tracking, audit logs, in-app metrics, usage dashboards |
| `references/ui-design-patterns.md` | Building the visual layer: components, typography, spacing, tokens, motion, micro-copy, dark mode, icons, z-index |
| `references/payments-and-billing.md` | Integrating payments: checkout, subscriptions, invoicing, refunds, webhooks, PCI, revenue metrics |
| `references/internationalization.md` | Multi-language support: i18n architecture, string handling, RTL, date/number/currency formatting, translation workflows |
| `references/notifications-and-email.md` | Multi-channel notifications: in-app, email, push, SMS, Slack; preferences, digests, templates, delivery tracking, shared inbox |
| `references/system-integration.md` | Connecting features: service layer, event bus, cache invalidation chains, entity linking, real-time propagation, job observability, feature flags |
| `references/reporting.md` | Report generation: types, builder UI, PDF/Excel generation, scheduled reports, large datasets, print optimization, report permissions |
| `references/api-and-integrations.md` | External connections: adapter pattern, OAuth2, outbound/inbound webhooks, data sync, integration marketplace, building your own API |
| `references/ai-product-patterns.md` | Building AI products: streaming, context management, model orchestration, prompt management, RAG, evaluation, cost control, moderation, tool calling |
| `references/testing-and-quality.md` | Writing tests for auth, CRUD, permissions, accessibility, visual regression |
| `references/performance-and-security.md` | Optimizing bundle size, queries, caching; hardening CSP, CORS, rate limiting, input sanitization |
| `references/security-deep-dive.md` | Advanced security: session hardening, API security, data protection, secrets management, supply chain, incident response |
| `references/domain-considerations.md` | **During pre-flight**, after identifying the domain. Covers 33 verticals with domain-specific gotchas, compliance, and data model traps |

## How to interpret a vague request

The user rarely says "build me an admin panel with RBAC, audit logs, settings, user management, and a billing module." They say "I need a dashboard for my X." Interpret as follows:

- "Dashboard for my SaaS" → multi-tenant admin: users, organizations, billing, usage, settings, audit
- "Internal tool for my team" → CRUD on the team's domain entities, with simple shared auth, lighter RBAC
- "Analytics dashboard" → KPI cards + 4–6 charts + a filter bar + exports; auth optional but recommended
- "Operational dashboard" / "control panel" → real-time data, health indicators, action buttons (restart, deploy, ack), audit
- "Customer portal" → end-user-facing: their data only, billing/subscription, support, settings
- "Reporting dashboard" → tables-heavy, scheduled reports, exports to CSV/PDF, filters, drill-down
- "CMS" / "blog admin" → content types, draft/publish workflow, media management, SEO, versioning
- "E-commerce admin" / "store manager" → products/variants, orders, inventory, fulfillment, returns
- "AI dashboard" / "LLM admin" → models, prompts, conversations, token/cost tracking, evaluation pipelines
- "Helpdesk" / "support dashboard" → tickets, SLAs, queues, canned responses, customer context
- "CRM" / "sales dashboard" → contacts, deals, pipeline stages, campaigns, forecasting
- "HR dashboard" / "people admin" → employees, payroll, leave, benefits, performance reviews
- "Medical" / "healthcare admin" → patients, encounters, prescriptions, claims, HIPAA compliance
- "Marketplace admin" → two-sided (buyers + sellers), disputes, commissions, trust/safety
- "Gaming admin" / "esports" → players, matches, virtual economy, anti-cheat, seasons
- "Restaurant" / "POS admin" → menus with modifiers, orders, table management, kitchen display
- "Property management" → units, leases, tenants, maintenance, owner reporting
- "Logistics" / "fleet management" → shipments, vehicles, routes, drivers, real-time tracking
- "Legal" / "law firm" → matters, time billing (0.1-hour increments), trust accounting, conflicts
- "Construction" / "field services" → jobs, crews, permits, inspections, offline-first mobile
- "Education" / "LMS" → courses, enrollments, gradebook, assessments, student progress

Pick the closest archetype (or combine if the dashboard spans domains), then read the matching section in `references/domain-considerations.md` during pre-flight. Build the foundation slice the same way regardless, then specialize the feature slices to the archetype.

## Keep going until it's actually done

Dashboards have a long tail. The instinct to stop at "the main pages exist" is wrong — the main pages existing is roughly 40% of the work. The remaining 60% is: empty states, error states, validation, permissions on every mutation, the audit log, the settings page, the profile page, the logout flow, the responsive breakpoints, the keyboard tab order, the loading skeletons, the success toasts, the cache invalidation, the seed data, and the verification pass. Budget for all of it. A "done" dashboard is one a real person can hand to a colleague without a list of caveats.

When in doubt about whether to keep going: open the dashboard yourself and try to do the user's actual job with it. The first thing you can't do is the next thing to build.
