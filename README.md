# Production Ready

**Stop shipping dashboards that fall apart on first click.**

AI-generated dashboards look finished — until someone tries to use them. Buttons that don't save. Charts hardcoded to fake JSON. Login pages that accept anything. Sidebar links that 404. Settings pages with no persistence. Ten seconds of clicking reveals the truth: it's a screenshot, not a tool.

Production Ready is an AI instruction set that eliminates this failure mode. Every feature ships wired end-to-end — real auth, real data, real permissions, real states, real feedback — across any stack and any industry.

<p align="center">
<strong>36 reference files</strong> &nbsp;·&nbsp; <strong>33 industry verticals</strong> &nbsp;·&nbsp; <strong>24 production requirements</strong> &nbsp;·&nbsp; <strong>4 shippable tiers</strong>
</p>

## Quick start

**Claude Code / Codex:**
```bash
git clone https://github.com/aihxp/production-ready.git
# Add as a skill → ask Claude to build a dashboard → the skill activates automatically
```

**Any AI agent:**
Upload `SKILL.md` and the relevant reference files to your project context. The structured output works with any agent that has a plan-then-execute loop — Claude Code, Codex, Cursor, Windsurf, or custom agents.

**Standalone reference:**
Read `SKILL.md` for the workflow, load reference files as you build each layer.

## The problem this solves

AI agents build dashboards layer by layer — database, then API, then auth, then UI. This produces 80% completion on every layer and 0% completion on any actual feature. The result is a scaffold that *looks* done but does nothing.

Production Ready enforces **vertical-slice discipline**: build one feature completely — schema, API, permissions, UI, states, tests — before touching the next. When a feature is done, a real person can do a real job with it.

## How it works

```
Step 0  Research     → Detect project state (greenfield / existing / audit)
Step 1  Pre-flight   → 12 questions that prevent incompatible decisions
Step 2  Architecture → Stack, auth model, permission model, route map
Step 3  Identity     → Derive unique visual personality from the domain
Step 4  Foundation   → Auth, RBAC, shell, seed data, one working page
Step 5  Features     → Each feature wired end-to-end with all states
Step 5.1 Hollow check → Automated scan catches TODOs and fake data mid-build
Step 6  Cross-cutting → Search, notifications, settings, exports, audit log
Step 7  Tests        → Auth flow, CRUD, permissions, accessibility
Step 8  Harden       → Security headers, performance budget, bundle size
Step 9  Verify       → 20-point smoke test before shipping
```

Each step maps to a **completion tier** — the dashboard is shippable at every checkpoint, not just at the end.

## Three entry modes

Not just for new projects.

| Mode | Trigger | Output |
|---|---|---|
| **Greenfield** | Empty directory | Full scaffolding from zero |
| **Assessment** | Existing codebase | 7-part scan → gap analysis → targeted todos |
| **Audit** | "Verify this dashboard" | Assessment + checklist → severity-sorted fix-it list |

The research output is a structured document any agent's planner can consume to generate precise, gap-filling tasks instead of generic ones.

## Completion tiers

24 requirements, organized into 4 independently shippable tiers:

| Tier | Name | What you can do with it |
|---|---|---|
| 1 | **Foundation** | Sign in, see real data, create/edit/delete one entity, sign out |
| 2 | **Functional** | Role-based access, all states handled, settings that save, profile page |
| 3 | **Polished** | Audit trail, keyboard accessible, responsive to mobile, cross-cutting features wired |
| 4 | **Hardened** | Tested, secure, verified, zero hollow indicators — ready to ship |

The agent declares each tier complete at its checkpoint. For existing codebases, the research phase determines the current tier and works toward the next one.

## What it catches

| What AI agents typically ship | What Production Ready enforces |
|---|---|
| `const mockUsers = [...]` in a component | Data from the real persistence layer via query hooks |
| `onClick={() => console.log("clicked")}` | Real mutation → cache invalidation → success toast |
| Login that accepts any credentials | Hashed passwords, sessions, server-side route guards |
| "Coming soon" pages in the sidebar | Every nav link routes to a real, rendered page |
| Charts that re-randomize on refresh | Charts driven by real data from the data layer |
| Empty table with no guidance | Empty state with explanation + CTA |
| `// TODO: add validation` | Client + server validation with inline field errors |
| Permissions checked only in the UI | Server enforces every mutation; UI reflects as courtesy |
| Every dashboard looks the same | Visual identity derived from domain context — unique colors, typography, personality |

## 33 industry verticals

Generic CRUD misses domain-specific landmines. The skill carries gotchas, compliance requirements, UX patterns, and seed data shapes for 33 industries:

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

**Examples of what domain knowledge prevents:**
- **Healthcare** — building without HIPAA audit logging of every PHI access
- **Financial** — using `float` for money instead of integer cents
- **E-commerce** — modeling inventory as a counter instead of a ledger
- **SaaS** — missing `WHERE org_id = ?` on every query (tenant data leak)
- **Construction** — missing AIA G702/G703 payment applications

## Works with any stack

Pattern-based, not framework-specific:

- **React / Next.js** + TypeScript + Prisma + Auth.js + shadcn/ui + TanStack Query
- **SvelteKit** + TypeScript + Convex/Drizzle + Auth.js + Tailwind
- **Vue / Nuxt** + TypeScript + Drizzle + shadcn-vue
- **Rails** + Hotwire/Turbo + Devise
- **Django** + HTMX + django-allauth
- **Laravel** + Inertia + Breeze

...or anything else. The guidance translates.

## Reference library

36 files, loaded on demand. A typical project reads 4–8, never all 36.

<details>
<summary>View the full reference library</summary>

| File | What it covers |
|---|---|
| **`SKILL.md`** | Core workflow, vertical-slice discipline, haves/have-nots |
| `codebase-research.md` | Project state detection, greenfield/assessment/audit modes, codebase scan protocol |
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
| `animation-and-motion.md` | Transitions, micro-interactions, loading choreography, spring physics |
| `dark-mode-deep-dive.md` | Token remapping, image handling, chart colors, flash prevention |
| `accessibility-deep-dive.md` | Screen readers, ARIA widgets, keyboard navigation, legal compliance |
| `error-pages-and-offline.md` | 404/403/500/503 pages, maintenance mode, PWA offline |
| `file-management-and-uploads.md` | Upload UX, chunked uploads, image crop, file previews |
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
| `headers-and-navigation.md` | Header layout, mega menus, mobile nav, footer design |
| `marketing-and-landing-pages.md` | Hero sections, social proof, feature grids, pricing tables |
| `seo-and-web-standards.md` | Meta tags, Open Graph, robots.txt, llms.txt, RSS feeds, sitemap |
| `social-media-features.md` | Social links, share modals, embeds, social login |
| `email-template-design.md` | HTML email, responsive email, dark mode in email |
| `realtime-and-collaboration.md` | Presence, live cursors, collaborative editing, WebSocket lifecycle |
| **Growth & Operations** | |
| `login-and-auth-pages.md` | Login page design, registration UX, auth flows |
| `migration-and-data-import.md` | CSV/API import, platform migration, phased rollout |
| `expansion-and-scalability.md` | Multi-tenancy, feature flags, i18n prep, billing expansion |
| **Domains** | |
| `domain-considerations.md` | 33 verticals with gotchas, compliance, seed data |

</details>

## Contributing

Gaps, missing domains, outdated guidance — contributions welcome. Open an issue or PR.

## License

[MIT](LICENSE)
