# Dashboard Design Skill

A comprehensive AI skill for building production-grade dashboards — admin panels, analytics consoles, internal tools, SaaS back-offices, CRM systems, AI product interfaces, and any multi-page application with navigation, auth, and CRUD.

**The problem this solves:** AI-generated dashboards that *look* finished but are hollow. Buttons that don't save. Charts hardcoded to fake JSON. Login pages that accept anything. Sidebar links that 404. This skill enforces a no-scaffold rule — every feature shipped must be wired end-to-end to real data.

## What's inside

**22 reference files, ~10,000+ lines of implementation guidance, 33 domain verticals — all research-validated.**

### Core skill

| File | What it covers |
|---|---|
| `SKILL.md` | Core workflow, vertical-slice discipline, mandatory haves/have-nots, archetype routing |

### Reference files (load on demand)

| File | Coverage |
|---|---|
| `preflight-and-verification.md` | 12 pre-flight questions + verification checklist + smoke test |
| `information-architecture.md` | 7 layout patterns, nav, sidebar, breadcrumbs, responsive breakpoints |
| `auth-and-rbac.md` | Passkeys, magic links, social login, RBAC, multi-tenant, user profile, impersonation |
| `data-layer.md` | React Server Components, server actions, API contracts, queries, mutations, serverless pooling |
| `data-visualization.md` | Chart selection tree, KPIs, tables, accessibility, responsive charts, sparklines, widget systems |
| `states-and-feedback.md` | Suspense/streaming, loading, empty, error, toasts, undo, offline |
| `workflows-and-actions.md` | Forms, onboarding, wizards, drag-and-drop, command palette, approval workflows, collaborative editing |
| `analytics-and-telemetry.md` | Feature adoption, session replay, A/B testing, anomaly alerts, audit logs, RUM |
| `settings-and-configuration.md` | User/org/system settings hierarchy, data model, auto-save vs explicit, admin panel, multi-tenant |
| `ui-design-patterns.md` | Components, typography (14px base), spacing (4px grid), tokens, motion, micro-copy, dark mode, icons, z-index |
| `payments-and-billing.md` | Checkout, subscriptions, invoicing, refunds, webhooks, PCI compliance, revenue metrics (MRR/ARR/churn/LTV) |
| `notifications-and-email.md` | Multi-channel (in-app, email, push, SMS, Slack), preferences, digests, templates, delivery tracking, shared inbox |
| `internationalization.md` | i18n architecture, ICU MessageFormat, RTL support, date/number/currency formatting, translation workflows |
| `system-integration.md` | Service layer, event bus, cache invalidation chains, entity linking, real-time propagation, job observability, feature flags |
| `reporting.md` | Report types, builder UI, PDF/Excel generation, scheduled reports, large datasets, print optimization |
| `api-and-integrations.md` | Adapter pattern, OAuth2, outbound/inbound webhooks, data sync, integration marketplace, building your own API |
| `ai-product-patterns.md` | Streaming, context window management, model orchestration, prompt management, RAG, evaluation, cost control, moderation, tool calling |
| `testing-and-quality.md` | E2E, component, API, accessibility, visual regression, load testing, contract testing, test factories |
| `performance-and-security.md` | Bundle splitting, SSR, database queries, CSP, CORS, rate limiting, input sanitization |
| `security-deep-dive.md` | Session hardening, API security, data protection, secrets management, supply chain, incident response |
| `domain-considerations.md` | 33 domain verticals with gotchas, compliance requirements, and seed data shapes |

### 33 domain verticals

Each domain includes: archetype description, core entities, domain-specific gotchas, compliance requirements, expected UX patterns, and realistic seed data shapes.

| # | Domain | # | Domain |
|---|---|---|---|
| 1 | SaaS / Multi-tenant | 18 | Non-profit / Fundraising |
| 2 | E-commerce / Retail | 19 | Media / Streaming |
| 3 | CMS / Content / Blog | 20 | Agriculture / Farm Management |
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

## How to use

### As a Claude skill

1. Download the `.skill` file or clone this repo.
2. Add it to your Claude project or Claude Code configuration.
3. Ask Claude to build a dashboard — the skill activates automatically.

### As a reference

Read `SKILL.md` first for the workflow and principles. Then load specific reference files as needed — you don't need to read all 22 files for every project.

**Typical flow:**
1. Read `SKILL.md` (core principles)
2. Read `preflight-and-verification.md` (answer 12 questions before coding)
3. Read the matching domain section in `domain-considerations.md`
4. Read reference files as you build each layer (auth, data, UI, etc.)
5. Run the verification checklist before declaring done

## Core principles

- **Vertical slices, not horizontal layers.** Build one feature end-to-end before starting the next.
- **No scaffolds.** Every visible element is connected to real data. No TODOs, no fake JSON, no "coming soon."
- **Load on demand.** Read only the reference files relevant to what you're building right now.
- **The server enforces, the client reflects.** Permissions are checked server-side. The UI hides unauthorized actions as courtesy, never as security.

## Stack agnostic

The skill works with any stack: React/Next.js, Vue/Nuxt, Svelte/SvelteKit, Rails, Django, Laravel, or anything else. Guidance is pattern-based, not framework-specific — though framework-specific recommendations are included where relevant.

## Contributing

This skill is maintained as a living document. If you find a gap, a domain that's missing, or guidance that's outdated, contributions are welcome.

## License

MIT License. See [LICENSE](LICENSE) for details.
