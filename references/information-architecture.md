# Information Architecture

The shell of a dashboard — the layout, navigation, header, sidebar, breadcrumbs, page structure — is where convention is your friend. Users learn dashboards by transferring expectations from every other dashboard they've used. Deviating from the conventions costs more than it gains. This file is the rules for the shell.

## The canonical layout

Adopt this layout unless the domain demands otherwise. It is the convention used by Stripe, Linear, Notion, Vercel, GitHub, GitLab, AWS, Datadog, Tableau, Power BI, Looker, Mixpanel, Segment, Asana, Jira, Atlassian, ServiceNow, Salesforce, HubSpot, Intercom, Shopify admin, Airtable, Retool, Metabase, and roughly every internal tool you've used.

```
┌─────────────────────────────────────────────────────────────┐
│ [LOGO]   ┊ optional global search ┊         [user menu ▾]  │  ← header (56–64px)
├──────────┬──────────────────────────────────────────────────┤
│          │  Breadcrumbs > nested > location                 │
│ Sidebar  │  ┌──────────────────────────────────────────┐    │
│  nav     │  │  Page header                              │    │
│          │  │  Page title          [primary action ▸]   │    │
│  240px   │  │  optional subtitle / description          │    │
│          │  └──────────────────────────────────────────┘    │
│  collap- │                                                    │
│  sible   │  Main content area                                 │
│  to 64px │                                                    │
│          │                                                    │
│          │                                                    │
│          │                                                    │
└──────────┴──────────────────────────────────────────────────┘
```

### Why this layout

- **Logo top-left**: where everyone looks first. Click → home. Anchors brand identity.
- **User menu top-right**: where everyone goes for "log out" and "settings." Standard since at least Gmail (2004).
- **Sidebar left**: vertical lists scale better than horizontal ones. You can fit 12 nav items on a sidebar; you cannot fit 12 on a topbar.
- **Persistent sidebar, not hamburger**: on desktop, hiding nav behind a click is friction. The sidebar collapses to icons on narrow screens, to a drawer on mobile — but on a typical 1440px monitor, it stays open.
- **Content area on the right**: gets the most pixels because it's where work happens.
- **Breadcrumbs above the page header**: orient the user inside hierarchy. Skip on top-level pages; use on every nested page.
- **Page header inside the content area**: title, subtitle, primary action button. Same shape on every page so the eye knows where to land.

### Dimensions that work

- Header: 56–64px tall
- Sidebar: 240–280px wide expanded, 56–64px collapsed
- Content max width: 1280–1440px (centered) or full-width depending on density needs — full-width for tables and dashboards, centered for forms and reading
- Page padding: 24–32px on desktop, 16px on mobile
- Section spacing: 24–32px between major sections, 12–16px between rows

## When to deviate

Some dashboards have legitimate reasons to use a different layout. Recognize them so you don't reinvent for no reason.

- **Operational / NOC dashboards** — large screens, no nav at all, all real-time tiles. The user doesn't navigate; they monitor. Skip the sidebar, fill the screen with widgets.
- **Single-screen analytics** — one page with filter bar on top and charts below. Nav is unnecessary if there's only one view.
- **Wizard / onboarding flows** — linear progression with a step indicator instead of a sidebar.
- **Embedded dashboards** — when the dashboard lives inside another app (e.g., a customer-facing report inside a CRM), the host already has nav. Don't duplicate.

For everything else — admin panels, internal tools, SaaS back-offices, customer portals — use the canonical layout.

## The header

### Logo

- **Position**: top-left corner. Vertically centered in the header bar.
- **Size**: 28–36px tall. Width whatever the logo demands.
- **Behavior**: clickable, links to dashboard home (`/` after login). On hover, very subtle feedback (slight opacity change). No big "click me" treatment.
- **For unbranded internal tools**: a simple wordmark or the project name in a distinctive font is better than no logo. The header feels naked without something there.

### Optional global search

If the dashboard has more than ~50 navigable items (records, pages, users, projects), put a global search in the center of the header (or center-left of the content area). Keyboard shortcut to focus it: `⌘K` / `Ctrl+K`. This is the "command palette" pattern from Linear, Notion, Slack, Vercel, etc.

If the dashboard has fewer items, skip global search — it adds clutter for nothing.

### User menu

Top-right corner. Avatar (or initials) with the user's name beside it on desktop, just the avatar on mobile. Click opens a dropdown:

```
┌───────────────────────┐
│ Hanns C.              │
│ admin@acme.com        │
├───────────────────────┤
│ Profile               │
│ Account settings      │
│ Theme           ▸     │  ← optional submenu
├───────────────────────┤
│ Help                  │
│ Keyboard shortcuts    │
├───────────────────────┤
│ Sign out              │
└───────────────────────┘
```

The order matters: identity at top, account/personal settings in the middle, help below, sign out at the bottom. This is the convention. Sign out is always last.

### Notifications bell

Optional. Place it just left of the user menu. Show a dot or count when there are unread items. Click opens a dropdown of recent notifications with a "view all" link to a full notifications page.

Skip the bell entirely if the dashboard doesn't generate notifications. An empty notifications icon is worse than no icon.

## The sidebar

### Item ordering

Order items by **frequency of use**, not alphabetically and not by section weight. Most-used at the top, least-used (settings, admin) at the bottom.

A typical order for a SaaS admin:

```
Dashboard       ← always first; the landing page
Inbox           ← if there are user-actionable items
Customers
Orders
Products
─────
Reports
Audit log
─────
Settings        ← always last (or in the user menu)
```

Group related items into sections separated by a thin divider or a small section label. Don't over-section — three sections max for a typical dashboard. More than three and you're describing an app, not a dashboard.

### Item count

- **Sweet spot**: 5–8 top-level items
- **Maximum**: 10 top-level items
- **Beyond that**: collapse the bottom half into "More" or move into settings

If the dashboard genuinely has 15 distinct top-level features, it might be three apps in a trench coat. Push back on the user before building it as one.

### Sub-menus

Use a sub-menu when a top-level area has 2–6 child pages that share the parent. Render it as an indented list under the parent that expands when the parent is clicked or active.

```
▾ Customers
    All customers
    Segments
    Activity
    Imports
```

Rules:
- Sub-menus expand when you visit any child page (so the active path is always visible)
- The expanded/collapsed state for each parent is remembered across sessions (localStorage)
- Don't nest more than two levels deep — third-level navigation belongs inside the page, not in the sidebar
- Clicking the parent label navigates to the parent's index page; the chevron handles expand/collapse separately

### Icons

Each top-level item gets an icon. Icons help scanning, and they make the collapsed sidebar usable. Use a single icon family (Lucide, Heroicons, Phosphor, Tabler) — don't mix.

Sub-menu items typically don't get icons; they're text-only and indented under the parent's icon.

### Active state

The current page must be visually distinct in the sidebar. The conventions:
- A solid background fill (slightly darker or accent-colored) on the active item
- A 2–3px vertical bar on the left edge of the active item
- Bolder text weight on the active item

Pick one or two; don't pile all three. Make sure the active state is obvious from across the room.

### Collapsed state (icon rail)

When the sidebar collapses on narrow screens:
- Width drops to ~64px
- Only icons remain
- Hovering an icon shows a tooltip with the item label
- The active state still shows (vertical bar or background fill)
- A toggle button at the top or bottom expands it back

### Mobile (drawer)

Below ~768px the sidebar should not occupy persistent space. Replace it with a hamburger button in the header that opens the sidebar as an overlay drawer from the left edge. Tapping outside or pressing `Esc` closes it.

## Breadcrumbs

Show breadcrumbs on every page deeper than the top level. They orient the user and provide one-click escape upward.

```
Dashboard / Customers / Acme Corp / Orders / #4521
```

Rules:
- Each segment except the last is clickable
- The last segment is the current page (not a link)
- Use `/` or `>` as the separator — pick one and stay consistent
- Truncate long names with ellipsis if the breadcrumb threatens to wrap
- Don't show breadcrumbs on the dashboard home — there's nothing above it
- Don't include the user's name or any state in breadcrumbs — only structural location

## The page header

Every content page has the same header shape. Consistency is the point.

```
┌────────────────────────────────────────────────────┐
│ Page title                       [Secondary] [Primary action] │
│ Optional subtitle or description                              │
└────────────────────────────────────────────────────┘
```

- **Title**: the noun this page is about. "Customers." "Order #4521." "Settings — Billing."
- **Subtitle**: optional. One sentence of context. Often the count, the last-updated time, or a description.
- **Primary action**: top-right of the header. The most common action a user takes on this page. "New customer," "Export," "Save changes."
- **Secondary actions**: to the left of the primary. "Filter," "Import," "Refresh." Use icon-only buttons for very common ones, labeled buttons for less common ones.

The page header is sticky on long-scrolling pages so the primary action is always reachable.

## The landing page

The page after login is special. It's the first thing the user sees every session. Treat it as a real designed page, not a placeholder.

The conventional landing page for a dashboard:

```
┌────────────────────────────────────────────────────┐
│ Welcome back, Hanns                                │
│ Here's what's happened since you were last here.   │
├──────────┬──────────┬──────────┬──────────────────┤
│ KPI 1    │ KPI 2    │ KPI 3    │ KPI 4            │
│ 1,247    │ $42.1k   │ 98.2%    │ 12               │
│ ↑ 12% MoM│ ↓ 3% WoW │ → flat   │ +3 today         │
├──────────┴──────────┴──────────┴──────────────────┤
│ Main chart (revenue over time, etc.)               │
│                                                     │
├─────────────────────┬──────────────────────────────┤
│ Secondary chart     │ Recent activity feed         │
│                     │ • Alice created Project X    │
│                     │ • Bob updated Order #123     │
│                     │ • System restarted at …      │
└─────────────────────┴──────────────────────────────┘
```

Rules:
- 3–6 KPI cards across the top — never more than 6, four is the sweet spot
- Each KPI shows: the number, the unit/format, the comparison (vs. previous period or target), and the direction
- One large chart below the KPIs that tells the main story
- One secondary panel: another smaller chart, or a recent-activity feed, or a "needs attention" list
- All data is real, sourced from the data layer, and reflects the actual current state of the database
- A "last updated" timestamp somewhere near the top so the user knows the freshness

Do not put 12 charts on the landing page. The landing page answers "is everything OK?" in 5 seconds. Drill-down lives on dedicated pages.

## Page templates

There are five page templates that cover ~95% of dashboard pages. Build them once as reusable components and reuse them.

### 1. Dashboard / overview

KPIs + charts + activity feed. Used for the landing page and any "summary" page (e.g., "Customer overview").

### 2. List / table

A single primary table with filter bar, search, pagination, and per-row actions. Used for "all customers," "all orders," etc.

```
┌────────────────────────────────────────────────────┐
│ Customers                              [+ New]     │
│ 1,247 total                                         │
├────────────────────────────────────────────────────┤
│ [Search]  [Status ▾] [Plan ▾] [Created ▾]  [↓]    │  ← filter bar
├────────────────────────────────────────────────────┤
│ ☐ Name     │ Email      │ Plan │ Created │  ⋯    │
│ ☐ Acme Co  │ ...        │ Pro  │ Jan 12  │  ⋯    │
│ ☐ ...      │ ...        │ ...  │ ...     │  ⋯    │
├────────────────────────────────────────────────────┤
│ Showing 1–25 of 1,247         ‹ 1 2 3 ... 50 ›    │
└────────────────────────────────────────────────────┘
```

### 3. Detail

A header with the entity's identity + tabs or sections for different facets. Used for "Customer #1234," "Order #4521," "User profile."

```
┌────────────────────────────────────────────────────┐
│ ← Back to Customers                                │
│ Acme Corp                            [Edit] [⋯]    │
│ acme.com · Pro plan · Customer since Jan 2023       │
├────────────────────────────────────────────────────┤
│ Overview · Orders · Users · Billing · Activity      │  ← tabs
├────────────────────────────────────────────────────┤
│ Tab content here                                    │
└────────────────────────────────────────────────────┘
```

### 4. Form

Used for create / edit. Single column, generous spacing, fields grouped into sections, primary action sticky at the bottom.

```
┌────────────────────────────────────────────────────┐
│ New customer                                        │
├────────────────────────────────────────────────────┤
│ Basics                                              │
│   Name           [_____________________________]    │
│   Email          [_____________________________]    │
│   Phone          [_____________________________]    │
│                                                      │
│ Plan & billing                                      │
│   Plan           [Free ▾]                           │
│   Trial days     [14]                                │
├────────────────────────────────────────────────────┤
│                                  [Cancel] [Create]  │  ← sticky footer
└────────────────────────────────────────────────────┘
```

### 5. Settings

Two-column: secondary nav on the left listing settings sections, content on the right. Each section is its own form that saves independently.

```
┌────────────────────────────────────────────────────┐
│ Settings                                            │
├──────────────┬─────────────────────────────────────┤
│ Profile      │ Profile                              │
│ Security     │   Name        [_________________]    │
│ Notifications│   Email       [_________________]    │
│ Team         │   Avatar      [_________________]    │
│ Billing      │                              [Save]  │
│ API keys     │                                      │
└──────────────┴─────────────────────────────────────┘
```

## Density: how much fits on a page

Two density modes are common:

- **Comfortable**: row height 48–56px, padding 16–24px. Default. Easier to scan, friendlier to new users.
- **Compact**: row height 32–40px, padding 8–12px. For power users with lots of data. Offer it as a toggle on data-heavy tables.

Pick one as the default and only build the second mode if users will be living in the dashboard all day. Most dashboards only need comfortable.

## Responsive breakpoints

Think in three sizes. Test all three.

- **Desktop (>1024px)**: full layout, persistent sidebar, multi-column content where appropriate
- **Tablet (768–1024px)**: collapsed icon-rail sidebar, content stacks where it doesn't fit
- **Mobile (<768px)**: hamburger drawer for nav, single-column content, tables become cards (each row a stacked card), filter bar collapses into a "Filters" button that opens a drawer

Tables are the hardest part of mobile dashboards. Two strategies:

1. **Horizontal scroll inside the table** with a "frozen" first column. Acceptable for power-user dashboards.
2. **Card layout per row**. Each row becomes a stacked card showing the most important fields. Better for end-user dashboards.

Pick based on who the user is. Don't ship desktop-only and call it done.

## Theming and tokens

Use a single source of truth for design tokens: CSS custom properties or a theme object. Define:

- 1 background, 1 surface, 1 surface-elevated
- 1 border, 1 border-strong
- 1 text-primary, 1 text-secondary, 1 text-muted
- 1 brand color (the accent), 1 brand-hover, 1 brand-foreground
- 4 status colors: success, warning, danger, info — each with foreground and subtle background variants
- A radius scale (4, 6, 8, 12), a spacing scale (4, 8, 12, 16, 24, 32, 48), a font-size scale

Light and dark mode are *both* defined by swapping the values of these tokens. Build dark mode from day one — retrofitting it later is painful and the result usually looks like inverted light mode. Test both modes before finishing.

## Iconography

Pick one icon family. Stick to it. Lucide and Heroicons are the safe defaults. Phosphor, Tabler, and Material Symbols are also good. Mixing families produces visual chaos.

Use icons consistently:
- Same icon for the same concept across the app (one icon for "delete," one for "edit," one for "settings")
- Same icon size in the same context
- Icons in primary nav are filled or outlined consistently

## What to avoid

- **Hamburger menu on desktop.** Hides the nav behind a click for no reason. Use a persistent sidebar.
- **Top nav with 8+ items.** Horizontal nav doesn't scale. Use a sidebar.
- **Two sidebars.** Causes orientation problems. If you need secondary nav, put it inside the page (tabs or sub-nav above the content).
- **Sticky everything.** Sticky header, sticky sidebar, sticky page header, sticky footer, sticky filter bar — and the user has 200px of actual content in the middle. Pick the few things that genuinely need to stay visible.
- **Carousels of widgets on the landing page.** If it's important enough to show, it's important enough to not hide behind a "next" button.
- **Modal-on-modal-on-modal.** Two layers max. Beyond that, use a dedicated page or a side drawer.
- **Cute branded loading screens.** They're charming the first time and infuriating the tenth. A subtle skeleton or top-of-page progress bar is enough.
- **Animations longer than 200ms** on routine interactions. The dashboard is a tool, not a presentation. Snappier is better.
- **Tooltips on everything.** A dashboard littered with `(?)` icons is a dashboard with a labeling problem. Fix the labels.
