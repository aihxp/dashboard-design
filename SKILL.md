---
name: production-ready
description: "Build production-grade, end-to-end connected dashboards across any stack (React/Next/Vue/Svelte/Remix/Rails/Django/Laravel/etc.). Covers admin panels, analytics consoles, internal tools, operational control centers, SaaS back-offices. Triggers on: 'dashboard,' 'admin panel,' 'control panel,' 'back office,' 'internal tool,' 'analytics view,' 'reporting interface,' 'ops console,' 'metrics view,' 'KPI view,' or any multi-page interface with navigation, auth, and CRUD over domain data. Also triggers on requests to add sidebar nav, role-based access, user management, audit logs, charts over real data, filters, exports, or any 'logged-in area.' Enforces a no-scaffold-no-placeholder rule: every feature ships wired end-to-end to a real backend or real local persistence, not stubbed with TODO, fake JSON, or 'hook this up later.'"
---

# Production Ready

This skill exists to solve one specific failure mode. AI-generated dashboards that *look* finished but are hollow. Buttons that don't save. Filters that don't filter. Charts hardcoded to fake JSON. Sidebars where half the links 404. Login screens that accept anything. The user runs it, and within ten seconds discovers it's a museum, not a tool.

The job here is the opposite. Ship a dashboard where every visible element is connected to something real. If a button is on the screen, clicking it does the thing. If a chart is on the screen, it reflects actual data. If there's a "users" page, you can actually create, edit, delete, and assign roles, and those changes persist across reloads.

This is harder than it sounds because dashboards are *systems*, not screens. A dashboard touches the data model, the API, auth, RBAC, navigation, state, caching, forms, charts, empty states, error states, audit, and exports. If any one layer is faked, the whole thing collapses into a demo.

## Core principle: vertical slices, not horizontal layers

Do not build the dashboard layer-by-layer (database, then API, then auth, then routes, then UI, then charts). That path produces 80% completion on every layer and 0% completion on any actual feature. Instead, build **vertical slices**:

> One feature at a time, end-to-end, working, before moving on.

A vertical slice for "user management" means: schema migration, API endpoints, permission checks, service layer, query hooks, list page, detail page, create/edit form, delete confirmation, empty and error and loading states, audit log entry, tested with seeded users, *all done* before touching the next feature. When the slice is complete, an actual person can do an actual job with it.

This principle is non-negotiable. It is the single biggest determinant of whether the output is a working dashboard or a scaffold.

## When this skill does NOT apply

Route elsewhere if the request is:
- **Single-component or single-page** ("add one chart," "build a landing page"). This skill is for multi-page systems.
- **Repo hygiene** (README, LICENSE, CI, CONTRIBUTING, issue templates, release automation). Delegate to the `repo-ready` skill.
- **Marketing site, blog, or docs site.** Frontend-only content sites do not have the auth, data, RBAC, and audit surface this skill enforces.
- **Greenfield repo scaffolding only.** If the user wants "set up my repo," that's `repo-ready`. This skill takes over once app features are being built.

## Workflow

Follow this sequence. Skipping steps is how dashboards end up hollow.

### Step 0. Detect project state and research

Read `references/codebase-research.md` and run the mode detection protocol.

- **Empty or boilerplate** routes to Mode A (Greenfield). Quick external-constraints scan, then proceed.
- **Existing source code** routes to Mode B (Assessment). Run the full scan before step 1. Its output replaces guesswork and constrains the architecture step.
- **User asked to audit/verify/harden** routes to Mode C (Audit). Verification-focused scan, produce a fix-it list, skip steps 1 through 6, go directly to fixing what's broken.

### Step 1. Pre-flight

Read `references/preflight-and-verification.md`. Answer the 12 questions in writing. In Mode B, pre-fill answers from the assessment output and focus on the gaps.

Once the domain is identified, read the matching section of `references/domain-considerations.md`. It carries domain-specific data-model traps, compliance requirements, and UX patterns that generic CRUD misses. Skipping this is how dashboards end up with float-based currency, single-entry accounting, or missing HIPAA audit logs.

If the request is vague ("build me a dashboard"), do not interrogate the user. Pick the most plausible interpretation, state assumptions in one short paragraph, and proceed. See the "vague request interpretation" table in `references/domain-considerations.md`.

### Step 2. Architecture note

Produce a short note (5 to 15 lines) covering:

- **Stack.** Framework, language, ORM/DB, auth provider, styling, state library, chart library.
- **Data source.** Real DB, existing API, or local persistence that behaves like real (SQLite plus Prisma, Convex, Supabase, Postgres in Docker). Never in-memory state or hardcoded JSON.
- **Auth model.** Sessions vs. JWT, where the user record lives, how roles attach.
- **Permission model.** RBAC role list and the resource by action matrix.
- **Route map.** Every page, parent to child nesting.
- **Threat model.** Three answers, one line each: (1) what does an attacker gain from this system (data, money, identity, control); (2) what is the highest-blast-radius mutation (who can `DELETE`, who can alter billing, who can impersonate); (3) where is each trust boundary (network edge, session, role, tenant). For regulated domains (healthcare, finance, legal, HR, government), add the compliance blast radius: what kind of incident would trigger a disclosure obligation. This is not a deep threat model; it is the minimum signal needed so the server-side checks in Step 4 aren't guesswork.
- **Visual identity.** Archetype, palette, typography, radius, density, signature detail (see step 3).

**In Mode B:** don't decide, document. Architecture note adopts what exists and adds only what's missing.

### Step 3. Derive the visual identity

Before any component code, commit to a visual personality derived from the domain, not from component-library defaults. Read `references/ui-design-patterns.md` and work through its "Visual identity" decision framework:

1. Pick an aesthetic archetype from the 10 in that file.
2. Make 5 decisions: color palette, typography pairing, border radius, density, signature detail.
3. Output the CSS design-token block and apply it globally.

Two users asking for "a SaaS dashboard" must get two dashboards that look different. Shipping with unmodified shadcn/Radix/MUI defaults is the visual equivalent of `// TODO`.

### Step 4. Build the foundation slice

Always the same, always first:

1. Project bootstrapped, runs locally.
2. **Verify every dependency exists before installing.** AI suggestions produce ghost packages roughly 20% of the time (Socket, 2025). Before running `npm install`, `pip install`, `bundle add`, `composer require`, or equivalent, confirm for each unfamiliar package: (a) it actually exists on the registry; (b) the publisher name matches what you expect (`@tanstack/react-query` is real, `@tansatck/react-query` is a squat); (c) weekly download count is non-trivial (1k plus is a sane floor for serious deps). A quick check: `npm view <pkg> name maintainers` or the registry's web page. Reject ghost packages. Never install a registry name an LLM produced without this check. See `references/performance-and-security.md` for the supply-chain deep dive.
3. Design tokens applied globally. Fonts loaded. Every component inherits from tokens.
4. Database or persistence up, migrations applied, seed script with at least one admin and 5 to 20 realistic rows per main entity.
5. Auth working: login, logout, session persistence, password hashing (argon2 or bcrypt, never plain, never sha256), middleware that gates protected routes server-side.
6. RBAC working: at least two roles, server-side permission checks, a way to verify what a non-admin sees. Every check maps back to a trust boundary from the Step 2 threat model.
7. Shell layout: header (logo top-left, user menu top-right), sidebar nav, content area, breadcrumbs, responsive collapse. Tokens applied, not default theme.
8. One real protected page reachable end-to-end.

**Proof test before declaring Tier 1:** run the app, sign in as admin, see the landing page with real data, visit a CRUD page, create, edit, delete, sign out, sign in as non-admin, confirm different UI and 403 on forbidden mutations, reload, confirm data persists. Inspect one button and verify its color traces to `--color-primary`, not a library default. Confirm every dependency in `package.json` (or equivalent) is a real, published package with a known publisher. If this loop works, Tier 1 is complete. Run the hollow check.

### Step 5. Build feature slices

For each feature, follow this recipe:

1. Schema or migration if new entities.
2. Seed data.
3. Server: list (paginate, filter, sort), detail, create, update, delete, all with permission checks that map to the threat model's trust boundaries.
4. Client: query hooks for read, mutation hooks for write, cache invalidation.
5. Pages: list (empty, loading, error states), detail, create/edit form (client and server validation, submission states, success and error feedback), delete confirmation.
6. Audit: every mutation writes an audit log entry.
7. Smoke test: walk create, edit, delete manually before declaring the slice done.
8. **Decision record (ADR) if the slice made a non-obvious choice.** Three lines in `.production-ready/adr/NNN-slug.md`: what was decided, why, what was rejected and why. Skip for obvious decisions ("used the already-chosen ORM"). Write for decisions a future maintainer would have to reverse-engineer from code: "chose event sourcing over CRUD for this entity," "split `orders` and `order_items` as separate aggregates," "bypass the query layer for this report because it aggregates across tenants." The ADR is the only way the next human or agent inherits your reasoning rather than rediscovering it. Three lines. If you need more, you are over-documenting.

Order of slices: most-used first, riskiest second, nice-to-haves last. If the user runs out of patience halfway through, the most important feature is still complete.

### Step 5.1. Hollow-check protocol (after every slice)

Runs in 30 seconds, catches hollows while they're cheap to fix. Adapt the patterns to your stack.

```
# TypeScript / JavaScript
grep -rnE "// (TODO|FIXME|HACK|TEMP)" src/
grep -rnE "(\(\)\s*=>\s*\{\s*\}|onClick=\{\(\)\s*=>\s*\{\s*\}\})" src/ --include="*.tsx" --include="*.ts"
grep -rn "console\.log" src/ --include="*.tsx" --include="*.ts"
grep -rnE "(mock|fake|dummy)Data\b|Lorem|lorem ipsum" src/
grep -rn "Math\.random()" src/

# Python (Django, FastAPI, Flask)
grep -rnE "# (TODO|FIXME|XXX)" . --include="*.py"
grep -rn "raise NotImplementedError" . --include="*.py"
grep -rn "^\s*pass\s*#" . --include="*.py"

# Ruby (Rails)
grep -rnE "# (TODO|FIXME)" app/ lib/
grep -rn "raise ['\"]TODO" app/ lib/
grep -rn "binding\.pry\|byebug\|debugger" app/ lib/

# PHP (Laravel)
grep -rnE "// (TODO|FIXME)" app/ resources/
grep -rn "dd(\|dump(\|var_dump(" app/ resources/

# Universal
grep -rnE "Coming soon|Not implemented|Placeholder|TBD|WIP" .

# Ghost dependency check (run after any package add)
# npm / pnpm / yarn
jq -r '.dependencies, .devDependencies | keys[]?' package.json 2>/dev/null | while read -r pkg; do
  npm view "$pkg" name > /dev/null 2>&1 || echo "GHOST: $pkg not published on npm"
done
# Python
pip install --dry-run -r requirements.txt 2>&1 | grep -i "could not find"
# Ruby
bundle check 2>&1 | grep -i "could not find"
```

**Rule:** zero high-severity hits before the next slice. If the scan finds a TODO you wrote ten minutes ago, fix it now. Legitimate `console.error` or structured logger calls in catch blocks are fine; raw `console.log`, `dd()`, or `binding.pry` in production paths are not. Any `GHOST:` hit is always high-severity: stop, remove the package, re-verify the real name, re-install.

**When to run:** after each slice (scan touched files only), and at each tier boundary (scan everything). Tier 4 requires zero hits at any severity.

### Step 6. Cross-cutting concerns

After two or three slices, layer in: global search, notifications, preferences, theme toggle, exports, audit log viewer, profile page. Easier once real data flows.

### Step 7. Tests alongside each slice

A slice isn't done until it has tests. Read `references/testing-and-quality.md`. Minimum per slice:

- Integration test walking the CRUD flow end-to-end.
- Permission test confirming non-admin users are blocked.
- Accessibility test (axe) on every new page.

### Step 8. Harden performance and security

Before verification, read `references/performance-and-security.md` and apply:

- Security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy).
- Bundle budget (code-split, lazy-load heavy libs).
- Rate limiting on login, mutations, uploads, exports.
- `npm audit` (or equivalent); fix critical and high.
- No secrets in git; no server secrets leaking to the client bundle.

### Step 9. Verification

Walk the verification checklist in `references/preflight-and-verification.md` and the security checklist in `references/performance-and-security.md`. If the repo also lacks CI, LICENSE, SECURITY.md, or release automation, invoke the `repo-ready` skill. Do not duplicate that work here.

## Completion tiers

24 requirements across 4 tiers. The agent aims for the highest tier the session allows. Declare each tier explicitly at its boundary. Never leave a tier half-done. Finish the current slice and declare the last complete tier.

In Mode B, the research output's gap analysis maps directly to unfulfilled tier requirements. Determine the current tier, then work toward the next.

### Tier 1: Foundation (8)

The dashboard runs, authenticates, and shows real data. One person can do one real job with it.

| # | Requirement |
|---|---|
| 1 | **Real auth.** Login rejects bad credentials, stores a session, gates every protected route server-side. |
| 2 | **Real data.** Seeded into a real persistence layer. Reloads persist. Two browsers see consistent state. |
| 3 | **A shell.** Header (logo top-left, user menu top-right), persistent sidebar, content area. |
| 4 | **Working navigation.** Every sidebar link routes somewhere real. Active state highlights current page. |
| 5 | **A landing page.** Route after login shows something useful: KPI cards, a chart, or an activity feed. Not lorem ipsum. |
| 6 | **At least one full CRUD entity.** List, detail, create, edit, delete, wired and persisted. |
| 7 | **A logout button** that actually invalidates the session. |
| 8 | **A visual identity.** Design tokens applied: palette, typography, radius, density. Not default component-library styling. |

### Tier 2: Functional (7)

Multiple features, roles, edge cases. A team can use it daily.

| # | Requirement |
|---|---|
| 9 | **Real RBAC.** At least two roles. Server-side permission checks on every mutation. UI hides unauthorized actions as courtesy, never as security. |
| 10 | **Loading, empty, and error states** for every async surface. Specific, not generic spinners. |
| 11 | **Form validation.** Client for instant feedback, server for truth. Inline field-level errors. |
| 12 | **Success and error feedback.** Toasts or banners after every mutation. Never silent. |
| 13 | **Pagination, sorting, filtering** on any table that could exceed 25 rows. State preserved in URL. |
| 14 | **A settings page** that actually saves. |
| 15 | **A user profile or account page** the logged-in user can edit. |

### Tier 3: Polished (5)

Cross-cutting concerns and accessibility. Ready for internal use or beta.

| # | Requirement |
|---|---|
| 16 | **An audit log** of who did what, when. |
| 17 | **Keyboard accessibility.** Every interactive element Tab-reachable. Focus styles visible. No keyboard traps. |
| 18 | **Responsive layout.** Works at 1440, 1024, 768, 375. Sidebar collapses to drawer on mobile. |
| 19 | **Breadcrumbs** on nested pages. Sub-menus expand and collapse, and remember state. |
| 20 | **Cross-cutting concerns wired:** search, exports, notifications, or theme toggle, whichever the domain needs. |

### Tier 4: Hardened (4)

Ship-ready. Tested, secure, verified.

| # | Requirement |
|---|---|
| 21 | **Tests.** Auth flow, one CRUD flow, one permission denial, axe on every page. |
| 22 | **Security headers.** CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy. No secrets in git. Rate limits on login and mutations. |
| 23 | **Verification checklist passed.** Full checklist from `references/preflight-and-verification.md` walked and green. |
| 24 | **No hollow indicators.** Zero TODOs, zero raw `console.log`, `dd()`, or `binding.pry` in production paths, zero hardcoded data, zero empty handlers. |

**Proof test:** build succeeds clean, tests pass, lint passes, DevTools shows zero red errors and zero failed requests. If the repo lacks CI, LICENSE, or release automation, hand off to `repo-ready`.

## The "have-nots": things that disqualify the dashboard at any tier

If any of these appear, the dashboard fails the no-scaffold rule and must be fixed:

- `// TODO`, `# TODO`, `// FIXME`, `# implement later`, `// hook up to API` in any file
- Raw `console.log`, `dd()`, `binding.pry`, `var_dump()`, or debug `print()` left in production paths
- Hardcoded arrays of fake data inside components (data must come from the data layer)
- `Math.random()` driving any chart or KPI
- `alert()` or `prompt()` for user interaction
- "Coming soon" pages or disabled nav items
- Permission checks that exist *only* on the client (server must enforce)
- Forms that POST and ignore the response
- Buttons whose `onClick` is `() => {}`, `undefined`, or `console.log("clicked")`
- Routes in the sidebar that 404
- Charts that re-randomize on every render
- Login pages that accept any credentials
- Any "demo mode" or "placeholder" copy visible to the user
- Components named `ExampleX`, `DemoY`, `TestZ` shipped to the user
- Unmodified shadcn/Radix/MUI default theme visible to the user
- Installing a package without confirming it exists on the registry with a real publisher (slopsquatting protection)
- A non-obvious architectural decision made with no corresponding ADR in `.production-ready/adr/`
- Running a destructive command (`rm -rf`, `DROP TABLE`, `git push --force`, `prisma migrate reset`) without an explicit confirmation step and a known backup

When you catch yourself about to write any of these, do the real version instead. The real version is almost always 10 to 30 more lines, not 10x more.

## Reference files: load on demand

The body above is enough to start. Load each reference *before* implementing that layer, not after.

| Reference file | When to load |
|---|---|
| `codebase-research.md` | **Always.** Start of every session |
| `preflight-and-verification.md` | **Always.** Start and end |
| `domain-considerations.md` | **Always.** During pre-flight, after identifying domain |
| `naming.md` | **Tier 1.** When naming anything cross-layer (tables, APIs, nav, events, permissions) |
| `information-architecture.md` | **Tier 1.** Shell, nav, layout |
| `auth-and-rbac.md` | **Tier 1.** Login, sessions, roles, permissions |
| `data-layer.md` | **Tier 1.** API, queries, mutations, caching |
| `ui-design-patterns.md` | **Tier 1.** Components, tokens, visual identity decision framework |
| `states-and-feedback.md` | **Tier 2.** Loading, empty, error, toasts |
| `workflows-and-actions.md` | **Tier 2.** Forms, bulk actions, exports |
| `settings-and-configuration.md` | **Tier 2.** Settings hierarchy, data model |
| `analytics-and-telemetry.md` | **Tier 3.** Audit log, event tracking |
| `system-integration.md` | **Tier 3.** Service layer, event bus, feature flags |
| `testing-and-quality.md` | **Tier 4.** Test strategy, integration tests |
| `performance-and-security.md` | **Tier 4.** CSP, CORS, rate limiting, bundle size |
| `security-deep-dive.md` | **Tier 4.** Session hardening, secrets, incident response |
| `data-visualization.md` | **On demand.** Charts, KPIs, tables |
| `payments-and-billing.md` | **On demand.** Checkout, subscriptions, PCI |
| `internationalization.md` | **On demand.** i18n, RTL, translation |
| `notifications-and-email.md` | **On demand.** Multi-channel notifications |
| `reporting.md` | **On demand.** Report generation, PDF and Excel |
| `api-and-integrations.md` | **On demand.** External APIs, webhooks |
| `ai-product-patterns.md` | **On demand.** AI and LLM features |

## Session state and handoff

Long builds span sessions. Without a state file, every resume rediscovers context from the code and drops the decisions made in earlier sessions. The second diagnosis of context-entropy failures (Pragmatic Engineer, r/ExperiencedDevs) is always the same: "the AI's context window can only see fragments, and developers have no map to find a stable version."

Maintain `.production-ready/STATE.md` as the map. Update it at every tier boundary, every session end, and whenever the harness compacts context. Read it first on resume. If it conflicts with the code, trust the code and update the file.

**Template:**

```markdown
# Production-Ready State

## Current tier
Working toward Tier [N]. Last completed tier: [N-1]. Declared at [ISO date].

## Slices completed
- [x] users (Tier 1, completed 2026-04-18)
- [x] orders (Tier 2, completed 2026-04-20)
- [ ] billing (in progress, started 2026-04-22)

## Active architectural decisions
- Stack: Next.js 15 + Prisma + SQLite (dev) / Postgres (prod) + Better Auth + shadcn/ui + Tailwind + TanStack Query + Recharts
- Auth: session-based, httpOnly cookie, argon2 password hash
- RBAC: roles = [owner, admin, member], server-enforced via middleware + row-level ownership checks
- Visual identity: Clean corporate archetype, DM Sans, 8px radius, comfortable density, accent-header-bar signature
- Threat model: tenant isolation is the critical boundary; highest-blast-radius op is cross-tenant `DELETE` in admin panel

## ADRs written
- 001-prisma-over-drizzle.md
- 002-session-cookie-not-jwt.md
- 003-audit-log-as-append-only-table.md

## Open questions blocking work
- Stripe vs. Lemon Squeezy for billing (user to decide before slice starts)

## Next slice
Billing. Needs: Stripe webhook endpoint, `subscription` + `invoice` schema, plan-change flow, dunning emails.

## Last session note
[2026-04-22] Finished orders audit log. Skipped analytics page for now (Tier 3 item). Hollow check clean. Ghost dep check clean.
```

**Rules:**
- STATE.md is the contract with the next session. If it is out of date, the next session will do the wrong thing.
- At every `/compact`, `/clear`, or context reset, update STATE.md first. Do not rely on the agent's memory.
- The `Active architectural decisions` block is the short version of the ADR corpus. Agents reading STATE.md on resume should not need to open every ADR to know the stack.
- Never delete STATE.md. If an entry is wrong, correct it in place with a dated note.

## Handoff: repo hygiene is not this skill's job

If the work needs README, LICENSE, SECURITY.md, CONTRIBUTING, CI pipelines, issue or PR templates, branch protection, dependency scanning, CODEOWNERS, or release automation, delegate to `repo-ready`. The two skills compose: production-ready owns the app wiring; repo-ready owns the repo scaffolding. Do not duplicate either.

**If your harness exposes a skill-invocation tool** (e.g., Claude Code's Skill tool), invoke `repo-ready` directly when the handoff trigger fires. **Otherwise**, surface the handoff to the user: "This step needs `repo-ready`. Install it or handle the repo hygiene layer separately." Do not attempt to generate CI, LICENSE, or release automation inline from this skill.

## Keep going until it's actually done

The main pages existing is roughly 40% of the work. The remaining 60% is empty states, error states, validation, permissions on every mutation, the audit log, the settings page, the profile page, the logout flow, the responsive breakpoints, the keyboard tab order, the loading skeletons, the success toasts, the cache invalidation, the seed data, and the verification pass. Budget for all of it.

A "done" dashboard is one a real person can hand to a colleague without a list of caveats. When in doubt, open the dashboard and try to do the user's actual job with it. The first thing you can't do is the next thing to build.
