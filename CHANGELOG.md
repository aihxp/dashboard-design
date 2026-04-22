# Changelog

## v1.4.0 (2026-04-22)

Quality-polish release. Adds per-step acceptance tests, a fourth project-state mode for migrations, canonical-scope headers on overlapping references, and a focused trigger surface. Frontmatter description trimmed; long trigger list moved to README. No breaking changes.

### Added

- **Per-step acceptance tests in SKILL.md.** Every workflow step now ends with a one-line "Passes when" gate that makes step completion auditable.
- **Mode D: Migration** in `codebase-research.md` and SKILL.md Step 0. Detects off-the-shelf templates (Retool, Appsmith, shadcn admin kit), prior-agent work, and cross-framework ports. Produces a keep/rewrite/discard disposition inventory and a phased migration plan. Prevents the "preserve hollow patterns because they're already there" failure mode.
- **Canonical-scope headers on 11 overlapping reference files.** Each file now states its owned scope and cross-references related files. Kills the split-brain between `auth-and-rbac.md` and `login-and-auth-pages.md`, `security-deep-dive.md` and `performance-and-security.md`, `data-layer.md` and `api-and-integrations.md`, `information-architecture.md` and `headers-and-navigation.md`, and others.
- **"When Production Ready should trigger" section in README.** Full trigger surface moved out of frontmatter: positive triggers, implied triggers, mode triggers, negative triggers, and the repo-ready pairing rule. Agents read the concise frontmatter first, then the detailed list when needed.

### Changed

- **Frontmatter description trimmed** from ~1,100 to ~620 characters. Retains core trigger signal ("dashboard," "admin panel," "internal tool"), enforcement rule, and pairing. Full trigger list now lives in README.
- **"Rules for working with existing codebases"** in `codebase-research.md` now applies to Modes B, C, and D (was B and C only), with a Mode D caveat: rules govern only "keep" items; "rewrite" and "discard" items are re-decided per the migration plan.

## v1.3.0 (2026-04-22)

Incident-aligned release. Closes the dominant AI-coding complaint gaps surfaced by industry research (METR 2025, Veracode 2025, Socket/slopsquatting, Stack Overflow 2025, and 2025 incidents at Moltbook, Lovable, Base44, Replit). Trims SKILL.md, extracts naming conventions to a new reference, adds a repo-ready companion handoff, and introduces session-state handoff plus ADRs for cross-session continuity.

### Added

- **Threat model bullet in Step 2 architecture note.** Three required answers: attacker gain, highest-blast-radius mutation, trust boundaries. Compliance blast radius for regulated domains.
- **Slopsquatting defense in Step 4.** Pre-install verification of every unfamiliar dependency (publisher, registry presence, download count). Hollow check now scans `package.json`, `requirements.txt`, and `Gemfile` for ghost names; any `GHOST:` hit is always high-severity.
- **ADR-per-slice in Step 5.** Three-line decision record in `.production-ready/adr/NNN-slug.md` for any non-obvious choice a future maintainer would have to reverse-engineer.
- **Session state and handoff section with `.production-ready/STATE.md` template.** Updated at every tier boundary and context compaction; read first on resume.
- **Graduated contract tests in Step 7 and Tier 4.** Any public API signature, server action, or exported type that crosses slice boundaries gets a schema-level contract test. Mandatory the moment a second slice consumes the signature.
- **Tier remap for regulated domains** (`domain-considerations.md`). Healthcare, government, education, finance, legal, HR, gaming with minors, and cybersecurity remap specific Tier 3 items to Tier 1 legal obligations. Includes seed-data compliance guardrails.
- **Vibe coding vs. vibe engineering framing in README.** Cites real 2025 incidents (Moltbook, Lovable CVE-2025-48757, Base44, Replit production-DB wipe, Veracode 45%, Socket 20% slopsquatting, METR 19% slowdown, Stack Overflow 66% "almost right"). Adopts Willison's vibe-engineering framing.
- **"What this skill prevents" incident table in README.** Every prevention maps to a documented incident or research finding.
- **Repo-ready companion skill pairing.** Explicit handoff section in SKILL.md for repo hygiene (README, LICENSE, CI, release automation). Harness-aware instructions for skill-invocation tools.
- **`naming.md` reference file.** Cross-layer naming conventions extracted from SKILL.md. Tables for code, navigation, casing boundary rule, and anti-patterns.
- **Frontmatter metadata.** `version`, `updated`, `changelog`, `compatible_with`, `pairs_with` for machine-readable version legibility and STATE.md cross-check.
- **"When this skill does NOT apply" routing section.** Explicit scope fence against single-component requests, repo hygiene, marketing sites, and greenfield scaffolding.
- **Cross-stack hollow check.** Python, Ruby, PHP, and universal markers added to the grep protocol (was JS/TS only).
- **Tier 1 proof test now verifies design-token application.** "Inspect one button and verify its color traces to `--color-primary`, not a library default."

### Changed

- **SKILL.md trimmed from 643 to approximately 360 lines.** Visual identity framework moved into `ui-design-patterns.md`. Naming conventions moved into `naming.md`. "How to interpret a vague request" table moved into `domain-considerations.md`.
- **Visual identity decision framework now single-sourced** in `ui-design-patterns.md` (archetype table, 5 decisions, token block, anti-patterns) immediately above the existing 10 token sets.
- **Documentation rewritten without em dashes** across SKILL.md, README.md, `naming.md`, and the new sections of `ui-design-patterns.md` and `domain-considerations.md`.
- **GitHub repo description** updated to mention the repo-ready pairing.
- **Reference file count** updated from 36 to 37.

### Added to have-nots

- Installing a package without confirming it exists on the registry with a real publisher (slopsquatting protection).
- A non-obvious architectural decision made with no corresponding ADR in `.production-ready/adr/`.
- Running a destructive command (`rm -rf`, `DROP TABLE`, `git push --force`, `prisma migrate reset`) without an explicit confirmation step and a known backup.

---

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
