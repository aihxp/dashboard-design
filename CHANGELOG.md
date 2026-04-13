# Changelog

## v1.2.0 — 2026-04-12

### Added

- **Mid-build hollow check protocol** (Step 5.1 in SKILL.md) — automated grep-based scan that runs after every feature slice and at every tier checkpoint. Catches TODOs, empty handlers, hardcoded data, and console.logs while they're cheap to fix, not at the end.

- **Decision trees added to 6 reference files** — each file now opens with actionable routing guidance before the deep content:
  - `auth-and-rbac.md` — "Which auth method?" decision tree (internal/SaaS/sensitive → method)
  - `data-layer.md` — Concrete Next.js App Router pattern (server component + server action + client component)
  - `information-architecture.md` — "Which layout?" table (page count × task → layout type)
  - `testing-and-quality.md` — Test prioritization table (priority 1–4 by regression damage)
  - `settings-and-configuration.md` — Default hybrid architecture + auto-save vs explicit save rule
  - `domain-considerations.md` — Top 13 domain gotchas frequency table

### Changed

- **Reference table is now tier-annotated** — each file tagged with when to load it (Always / Tier 1 / Tier 2 / Tier 3 / Tier 4 / On demand) instead of vague descriptions
- **Hollow check reminders** added to all tier checkpoint annotations
- **"Haves" section condensed** — replaced vestigial header with one-line note pointing to tiers (the tiers ARE the requirements list)
- **Tier 4 checkpoint** now explicitly requires zero hollow check hits at any severity

---

## v1.1.0 — 2026-04-12

### Added

- **Codebase research phase** (`references/codebase-research.md`) — New Step 0 in the workflow that runs before the pre-flight. Detects whether the project is greenfield, an existing codebase, or an audit request, then produces a structured assessment any agent's planner can consume.
  - **Mode A (Greenfield):** 60-second scan for external constraints (deployment config, pre-installed deps, design docs, existing APIs)
  - **Mode B (Assessment):** Full 7-part codebase scan — stack inventory, route/page map, data layer state, auth state, UI inventory, conventions scan, hollow indicators — produces a tier-tagged gap analysis
  - **Mode C (Audit):** Assessment + verification checklist walkthrough, produces a severity-sorted fix-it list

- **Completion tiers** — 4-tier system (Foundation → Functional → Polished → Hardened) that defines shippable checkpoints throughout the build. Each tier has concrete requirements, a proof test, and a checkpoint annotation in the workflow steps.
  - Tier 1 (Foundation): Auth + real data + shell + one CRUD entity + visual identity
  - Tier 2 (Functional): + RBAC + all states + validation + settings + profile
  - Tier 3 (Polished): + audit log + accessibility + responsive + cross-cutting
  - Tier 4 (Hardened): + tests + security headers + verification green + zero hollows

- **Decision tree** — Visual routing logic at the top of the tiers section that directs the agent based on project state (greenfield vs. existing codebase vs. audit) and current tier

- **Tier-aware research outputs** — Assessment gap analysis and audit fix-it lists now tag every item with its tier, so planners build todos in the correct order

### Changed

- **Workflow steps renumbered 0–9** (was 1–8 with duplicate step 4). Step 0 is the new research phase.
- **Step 1 (pre-flight)** now consumes research output to pre-fill answers for existing codebases
- **Step 2 (architecture)** branches: in assessment mode, document existing conventions instead of deciding fresh
- **Reference file count** updated from 35 to 36

### Fixed

- Duplicate step numbering (two "### 4" headings) corrected to sequential 0–9

---

## v1.0.0 — 2026-04-11

Initial release as Production Ready (renamed from Dashboard Design Skill).

- 10-step vertical-slice workflow
- 12-question pre-flight
- 33 industry vertical domain knowledge
- Visual identity derivation system (10 aesthetic archetypes)
- 35 reference files covering auth, data, UI, testing, security, and more
- 20-point verification checklist
- Naming conventions guide
- Works with any stack (React/Next/Vue/Svelte/Rails/Django/Laravel)
