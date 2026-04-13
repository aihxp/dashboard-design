# Production Ready

Build dashboards that actually work.

AI-generated dashboards have a specific failure mode: they *look* finished but fall apart the moment you try to use them. Buttons that don't save. Charts hardcoded to fake JSON. Login pages that accept anything. Sidebar links that 404. Settings pages with no persistence. A museum piece, not a tool.

This skill fixes that. It's an AI instruction set that ensures every feature is wired end-to-end — real auth, real data, real permissions, real states, real feedback — across any stack and any industry.

## Quick start

**With Claude Code:**
```bash
git clone https://github.com/aihxp/production-ready.git
# Add the repo as a skill in your Claude Code configuration
# Then just ask Claude to build a dashboard — the skill activates automatically
```

**With Claude Chat:**
Upload `SKILL.md` and the relevant reference files to your project context.

**As a standalone reference:**
Read `SKILL.md` for the workflow, then load reference files as you build each layer.

## How it works

The skill enforces a **vertical-slice discipline**: build one feature completely (schema > API > permissions > UI > states > tests) before starting the next. No layer is left half-done.

When you ask Claude to build a dashboard, the skill:

1. **Runs a pre-flight** — 12 questions that prevent incompatible decisions across layers
2. **Identifies your domain** — loads domain-specific gotchas from 33 industry verticals
3. **Derives a visual identity** — picks colors, typography, radius, density, and a signature detail from the domain context so every dashboard looks distinct
4. **Builds the foundation** — auth, RBAC, design tokens, shell layout, seed data, one working protected page
5. **Builds feature slices** — each feature wired end-to-end with real data and all states
6. **Adds cross-cutting concerns** — search, notifications, settings, exports, audit log
7. **Hardens** — security headers, performance budget, tests
8. **Verifies** — a 20-point smoke test that catches hollow features before you ship

## What the skill catches

Things that make dashboards feel broken, and what the skill does about them:

| Hollow pattern | What the skill enforces instead |
|---|---|
| `const mockUsers = [...]` in a component | Data from the real persistence layer via query hooks |
| `onClick={() => console.log("clicked")}` | Real mutation > cache invalidation > success toast |
| Login that accepts any credentials | Hashed passwords, session management, server-side route guards |
| "Coming soon" pages in the sidebar | Every nav link routes to a real, rendered page |
| Charts that re-randomize on refresh | Charts driven by real data from the data layer |
| Empty table with no guidance | Empty state with explanation + CTA to create the first record |
| `// TODO: add validation` | Client-side + server-side validation with field-level errors |
| Permissions checked only in the UI | Server enforces on every mutation; UI reflects as courtesy |
| Every dashboard looks the same | Visual identity system derives unique colors, typography, and personality from the domain |

## 33 industry verticals

The skill knows the domain-specific landmines for 33 industries. Each includes: data model traps, compliance requirements, expected UX patterns, and realistic seed data shapes.

<details>
<summary>View all 33 domains</summary>

| # | Domain | # | Domain |
|---|---|---|---|
| 1 | SaaS / Multi-tenant | 18 | Non-profit / Fundraising |
| 2 | E-commerce / Retail | 19 | Media / Streaming |
| 3 | CMS / Content / Blog | 20 | Agriculture / Farm |
| 4 | Financial / Fintech | 21 | AI / ML / Chat |
| 5 | Healthcare / Medical | 22 | Entertainment / Events |
| 6 | Education / LMS | 23 | Gaming / Esports |
| 7 | Travel / Hospitality | 24 | Cybersecurity / SOC |
| 8 | Sports / Fitness | 25 | Construction / Field Services |
| 9 | Real Estate / Property | 26 | Marketplace / Platform |
| 10 | Logistics / Supply Chain | 27 | Insurance / InsurTech |
| 11 | HR / People / Payroll | 28 | Telecommunications / ISP |
| 12 | Project Management | 29 | Energy / Utilities |
| 13 | Customer Support / Helpdesk | 30 | Government / Public Sector |
| 14 | Marketing / CRM / Sales | 31 | Recruiting / ATS |
| 15 | IoT / Device Management | 32 | Co-working Space |
| 16 | Restaurant / Food Service | 33 | Workflow Automation |
| 17 | Legal / Law Firm | | |

</details>

Examples of what domain knowledge prevents:
- **Healthcare**: building without HIPAA audit logging of every PHI access
- **Financial**: using `float` for money instead of integer cents
- **CRM**: modeling one pipeline when every business eventually needs multiple
- **Gaming**: not tracking embedding model provenance (changing models invalidates all vectors)
- **Construction**: missing AIA G702/G703 payment applications (the industry-standard billing workflow)

## Reference files

35 files, loaded on demand. You read 4-8 per project, never all 35.

<details>
<summary>View the full reference library</summary>

| File | What it covers |
|---|---|
| **`SKILL.md`** | Core workflow, vertical-slice discipline, haves/have-nots |
| **Foundation** | |
| `preflight-and-verification.md` | 12 pre-flight questions, verification checklist, smoke test |
| `information-architecture.md` | 7 layout patterns, navigation, sidebar, responsive |
| `auth-and-rbac.md` | Passkeys, magic links, RBAC, multi-tenant, impersonation |
| `data-layer.md` | Server Components, API contracts, queries, mutations, caching |
| **UI & UX** | |
| `ui-design-patterns.md` | Components, typography, spacing, tokens, visual identity token sets |
| `data-visualization.md` | Charts, KPIs, tables, accessibility, sparklines |
| `states-and-feedback.md` | Loading, empty, error, toasts, undo, offline |
| `workflows-and-actions.md` | Forms, onboarding, drag-and-drop, approval workflows |
| `animation-and-motion.md` | Transitions, micro-interactions, loading choreography, spring physics, View Transitions API |
| `dark-mode-deep-dive.md` | Token remapping, image handling, chart colors, flash prevention, component-level patterns |
| `accessibility-deep-dive.md` | Screen readers, ARIA widgets, keyboard navigation, legal compliance, cognitive accessibility |
| `error-pages-and-offline.md` | 404/403/500/503 pages, maintenance mode, PWA offline, partial failures |
| `file-management-and-uploads.md` | Upload UX, chunked uploads, image crop, file previews, file manager UI, security |
| **Features** | |
| `settings-and-configuration.md` | User/org/system settings, admin panel, multi-tenant |
| `notifications-and-email.md` | In-app, email, push, SMS, Slack, delivery tracking |
| `payments-and-billing.md` | Checkout, subscriptions, invoicing, refunds, PCI |
| `reporting.md` | PDF/Excel generation, scheduling, large datasets |
| `analytics-and-telemetry.md` | Feature adoption, session replay, A/B testing, audit logs |
| **Integration** | |
| `system-integration.md` | Service layer, event bus, cache invalidation, feature flags |
| `api-and-integrations.md` | OAuth2, webhooks, data sync, building your own API |
| `internationalization.md` | i18n, RTL, date/number formatting, translation workflows |
| **Specialized** | |
| `ai-product-patterns.md` | Streaming, RAG, prompt management, eval, cost control |
| `testing-and-quality.md` | E2E, component, load, contract, visual regression testing |
| `performance-and-security.md` | Bundle size, SSR, CSP, CORS, rate limiting |
| `security-deep-dive.md` | Sessions, data protection, secrets, incident response |
| **Frontend & Marketing** | |
| `headers-and-navigation.md` | Header layout, mega menus, mobile nav, footer design, back-to-top |
| `marketing-and-landing-pages.md` | Hero sections, social proof, feature grids, pricing tables, conversion patterns |
| `seo-and-web-standards.md` | Meta tags, Open Graph, robots.txt, llms.txt, RSS feeds, sitemap, favicons |
| `social-media-features.md` | Social links, share modals, social media management, embeds, social login |
| `email-template-design.md` | HTML email, responsive email, dark mode in email, deliverability, frameworks |
| `realtime-and-collaboration.md` | Presence, live cursors, collaborative editing, WebSocket lifecycle |
| **Growth & Operations** | |
| `login-and-auth-pages.md` | Login page design, registration UX, auth flows, login-to-dashboard journey |
| `migration-and-data-import.md` | CSV/API import, platform migration, phased rollout, account merging |
| `expansion-and-scalability.md` | Multi-tenancy, feature flags, i18n prep, billing expansion, RBAC scaling |
| **Domains** | |
| `domain-considerations.md` | 33 verticals with gotchas, compliance, seed data |

</details>

## Works with any stack

The skill is pattern-based, not framework-specific. It works with:

- **React / Next.js** + TypeScript + Prisma + Auth.js + shadcn/ui + TanStack Query
- **SvelteKit** + TypeScript + Convex/Drizzle + Auth.js + Tailwind
- **Vue / Nuxt** + TypeScript + Drizzle + shadcn-vue
- **Rails** + Hotwire/Turbo + Devise
- **Django** + HTMX + django-allauth
- **Laravel** + Inertia + Breeze

...or anything else. The guidance translates.

## Core principles

**Vertical slices, not horizontal layers.** Don't build the database, then the API, then the auth, then the UI. Build one feature end-to-end — working, tested, deployed — before touching the next.

**No scaffolds.** If a button is on the screen, clicking it does the thing. If a chart is on the screen, it reflects real data. If there's a settings page, it actually saves. No TODOs, no fake JSON, no "hook this up later."

**The server enforces, the client reflects.** Every permission is checked server-side on every mutation. The UI hides unauthorized actions as courtesy, never as security.

**Keep going until it's actually done.** The main pages existing is 40% of the work. The remaining 60% is: empty states, error states, validation, permissions, audit log, settings, profile, logout, responsive, keyboard access, loading skeletons, success toasts, cache invalidation, seed data, and the verification pass.

## Contributing

Gaps, missing domains, outdated guidance — contributions are welcome. This is a living document.

## License

[MIT](LICENSE)
