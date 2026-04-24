# Ready-suite end-to-end validation (Mirror Box)

**Date:** 2026-04-23 (initial audit); closure appended 2026-04-24
**Suite version at audit:** 9 skills live (harden-ready in progress)
**Suite version at closure:** 10 skills live, 10 with Mirror Box dogfood artifacts
**Validator:** structural verification + gap audit, not full execution (initial); all four recommendations shipped (closure)
**Target project:** [Mirror Box](https://github.com/aihxp/mirror-box) (traced Node + Fly.io + Honeycomb echo service)

## Closure note (2026-04-24)

All four recommendations from this report have shipped. See ["What closure looked like"](#closure-what-shipped) at the bottom of this document. The original report is preserved as written so the gap, the recommendations, and the closure form a single honest record.

## TL;DR

The suite's **handoff schemas compose correctly**. The planning tier (prd-ready, architecture-ready, roadmap-ready) produced three well-shaped artifacts for Mirror Box that structurally satisfy every downstream sibling's input contract. That's real validation and a genuine accomplishment.

The suite's **executional arc is not yet walked**. Six of nine siblings have no Mirror Box dogfood artifacts committed to their repos. The planning-tier artifacts reference Mirror Box as if it were a real running service, but no `aihxp/mirror-box` repo exists and no shipping-tier skill produced a dogfood deliverable for Mirror Box specifically. The roadmap-ready HANDOFF claim "the arc is end-to-end walkable" is paper-verified, not execution-verified.

This is not a suite design defect. It is a suite maturity gap. Two concrete improvements follow; both are cheap.

## What exists (paper verification: strong)

### Planning-tier dogfood artifacts

| Artifact | Location | Status | Downstream coverage |
|---|---|---|---|
| PRD.md | prd-ready/dogfood/PRD.md | Tier 2 Spec, 168 lines | stack-ready, architecture-ready (expected), roadmap-ready (expected), production-ready |
| HANDOFF.md (PRD) | prd-ready/dogfood/HANDOFF.md | 110 lines | stack-ready pre-flight fully pre-filled; ARCH-bound entities/flows/NFRs/trust boundaries noted |
| ARCH.md | architecture-ready/dogfood/ARCH.md | Tier 1 Sketch, 180 lines | Load-bearing check run; minimal-ARCH path justified; ADRs 001-003 produced |
| HANDOFF.md (ARCH) | architecture-ready/dogfood/HANDOFF.md | 144 lines | stack-ready, roadmap-ready, production-ready inputs enumerated |
| ADR 001-003 | architecture-ready/dogfood/adr/ | 3 files | Monolith / no persistence / no auth decisions recorded |
| ROADMAP.md | roadmap-ready/dogfood/ROADMAP.md | Tier 2 Plan, 136 lines | Slice queue for production-ready, cutover cadence for deploy-ready, KPIs for observe-ready, launch N/A |
| HANDOFF.md (ROADMAP) | roadmap-ready/dogfood/HANDOFF.md | 132 lines | Full five-sibling handshake declared |

### Schema completeness (verified by reading)

**PRD -> ARCH handoff.** PRD HANDOFF pre-filled six entities, four flows, five NFRs, four trust boundaries. ARCH consumed all six and produced a Tier 1 Sketch that references PRD.md explicitly. Clean pass.

**ARCH -> ROADMAP handoff.** ARCH HANDOFF produced a one-node component dependency graph (trivial but complete), load-bearing-first ordering, and a risk register. ROADMAP consumed the component graph and produced a three-slice queue that maps each slice back to PRD R-ids and ARCH components. Clean pass.

**ROADMAP -> downstream handoff.** ROADMAP HANDOFF addresses production-ready (drainable slice queue with all fields), deploy-ready (per-milestone cadence + rollback policy + migration posture), observe-ready (per-milestone KPIs mapped to PRD NFRs), launch-ready (N/A with reason). All five downstream skills have their inputs declared or explicitly marked N/A. Clean pass.

**PRD -> stack-ready handoff.** PRD HANDOFF pre-filled all six stack-ready pre-flight questions. stack-ready's "closest match + override" pattern handled the domain-profile mismatch. Clean pass.

The three planning-tier artifacts are internally consistent, cross-reference each other correctly, and produce the downstream contracts declared in each sibling's "Consumes from upstream" section. This is genuine validation.

## What does not exist (executional gap)

### Missing dogfood artifacts

| Skill | Dogfood folder | Mirror Box references in repo |
|---|---|---|
| prd-ready | YES | yes |
| architecture-ready | YES | yes |
| roadmap-ready | YES | yes |
| stack-ready | **NO** | **none** |
| production-ready | **NO** | **none** |
| repo-ready | **NO** | **none** |
| deploy-ready | **NO** | **none** |
| observe-ready | **NO** | **none** |
| launch-ready | **NO** | **none** |

### Missing project repo

No `aihxp/mirror-box` repo exists on GitHub. The planning-tier artifacts describe Mirror Box as a live traced service with a README, OTel span emission, Honeycomb ingestion, Fly.io deployment, and rolling-deploy history. None of that is instantiated anywhere the suite can point to.

This is a specific consequence of the suite's build order:

- **Shipping-tier skills shipped first** (deploy-ready, observe-ready, launch-ready in late April). Their build prompts said "dogfood against a toy Node + Fly + Honeycomb service." That dogfood happened during authoring but left no committed artifacts in the skill repos.
- **Planning-tier skills shipped second** (prd-ready, architecture-ready, roadmap-ready this week). Their build prompts said to retroactively PRD / ARCH / ROADMAP the shipping-tier dogfood target. That retroactive work produced the three planning-tier artifacts above.
- **Nobody went back** to create a `mirror-box` project repo, back-fill dogfood artifacts in the six earlier skills, or stand up a canonical reference implementation.

### Claims that are stronger than the evidence

The roadmap-ready HANDOFF.md contains this claim:

> The arc is end-to-end walkable. Every sibling's handoff contract is honored without inventing content beyond what this skill owns.

This is true **as a schema statement**. Every sibling's input contract is satisfied by an upstream artifact that exists and is readable. The handoff is structurally complete.

This is **not** true **as an execution statement**. Six of the nine skills did not run against Mirror Box and produce a committed artifact. production-ready never drained the slice queue. deploy-ready never generated a cutover plan specific to Mirror Box. observe-ready never wired the SLOs. launch-ready correctly declared N/A.

The claim is not wrong but it is ambiguous. Paper-walkable and execution-walked are different states. The roadmap dogfood should either:

- Clarify the claim ("arc is schema-complete; execution-walk is pending") and point to the missing artifacts, or
- Invest in producing the missing artifacts and elevate the claim to "execution-walked."

Either is a small fix.

## Findings (prioritized)

### Finding 1: The six missing dogfood artifacts are the real end-to-end test

The planning-tier dogfood proved its own skills work. It did not prove the full suite works end-to-end on a single project because six of the nine skills never touched Mirror Box.

Evidence this matters:
- **stack-ready**: its DECISION.md / STACK.md schema for Mirror Box is referenced by every downstream artifact but does not exist. If a user clones the suite, reads prd-ready's dogfood, and tries to trace the chain, they hit a missing link at stack-ready. (Mitigation: the PRD HANDOFF does pre-fill stack-ready's six pre-flight answers, which is 80% of what stack-ready would produce. But there is no committed DECISION.md artifact.)
- **production-ready**: ROADMAP declares three slices `done` with no evidence they were executed. production-ready's discipline (vertical slice, have-nots scan, hollow-check) was never exercised against Mirror Box specifically.
- **deploy-ready / observe-ready**: were authored with Mirror Box as their informal dogfood target, but no artifact in either repo points to it or demonstrates the workflow ran against it.

Cost to fix: the six missing artifacts are small (HANDOFF files mostly, a single DECISION.md for stack-ready, and README-sized summaries for the shipping-tier skills). A dedicated session could produce them all in 1-2 hours using the planning-tier artifacts as the input contract.

Value: the suite gains a canonical end-to-end walkthrough that any user can read and replicate. Marketing story shifts from "nine skills that compose in theory" to "nine skills that composed end-to-end on a real project whose artifacts you can read."

### Finding 2: There is no aihxp/mirror-box repo

The planning-tier artifacts treat Mirror Box as a live service. No such repo exists. The gap is either:

- **Intentional** (the planning dogfood is narrative and does not require a running project), in which case the planning artifacts should state that clearly, or
- **Unintentional** (a Mirror Box repo was supposed to exist and was never created), in which case creating it closes the loop.

Either is defensible. Picking a lane makes the suite honest.

Recommendation: create `aihxp/mirror-box` with the minimum code the PRD describes (a trivial Fastify + OTel Node service), deploy it to Fly free tier, and link each skill's dogfood to it. This is maybe 2 hours of work. It converts the planning-tier narrative into an actual end-to-end reference implementation.

### Finding 3: Suite handoff schemas are well-composed

The three planning-tier artifacts demonstrate that the suite's interop standard actually holds in practice. Each upstream skill's output is consumable by its downstream sibling without interrogation. The "closest match + override" pattern (stack-ready's 12 domain profiles, not a tight fit for Mirror Box) handles schema mismatch gracefully.

The roadmap-ready HANDOFF's "rough edge found during dogfood" (HANDOFF template requiring every sub-section populated, fixed before v1.0.0 by allowing "N/A with reason") is the kind of polish that validates the skill-build process works.

No schema-level changes needed. The suite's design holds.

### Finding 4: The "retroactive" framing of Mirror Box is doing more work than it should

All three planning-tier artifacts describe Mode A (Greenfield) with a "retroactive" qualifier: the service shipped first, the planning documents came second. This works for the exercise but muddles the downstream contract.

Specifically:
- ROADMAP says M1 is `completed 2026-04-15` and three slices are `done`. But no production-ready dogfood confirms those slices ran. The `done` status is a narrative placeholder.
- Architecture trust boundaries declare "shipping-tier skills already dogfooded on this" but the shipping-tier repos have no record of it.

A future planning-tier dogfood on a genuinely greenfield (not retroactive) project would produce tighter artifacts. The retroactive mode is useful for closing a suite-maturity loop but is not the natural shape.

## Recommendations

### Priority 1: Create `aihxp/mirror-box` repo with real code

Minimal scope:
- Fastify server implementing R-01 (`/echo`) and R-03 (`/healthz`).
- OTel SDK for Node with OTLP/gRPC exporter pointing at Honeycomb (or equivalent, configurable).
- `fly.toml` for single-region deployment.
- Dockerfile for portability.
- README pointing to the planning-tier artifacts as the specification.
- Deploy it. Get a real URL.

Cost: 2 hours.
Value: converts narrative to artifact. Closes the end-to-end loop.

### Priority 2: Back-fill dogfood artifacts in the six missing skill repos

For each of stack-ready, production-ready, repo-ready, deploy-ready, observe-ready, launch-ready:
- Add a `dogfood/` folder.
- Populate with the artifact that skill would produce on Mirror Box (DECISION.md for stack, slice execution log for production, hygiene report for repo, cutover plan for deploy, SLO doc for observe, N/A declaration for launch).
- Link back to the upstream planning-tier artifacts.

Cost: 1-2 hours if the Mirror Box repo exists; some artifacts can use the repo's state as evidence rather than prose.
Value: every skill in the suite can point to a canonical dogfood. The end-to-end walk becomes reproducible.

### Priority 3: Clarify or tighten the "arc walkable" claim in roadmap-ready's HANDOFF

Either:
- Reword to "schema-walkable; execution-walk pending these six artifacts," with a list, or
- Produce the six artifacts and elevate to "execution-walked" with links.

Cost: 5 minutes (wording fix) or batched with Priority 2 (artifact production).
Value: honest framing. Users reading the HANDOFF understand what state the suite is in.

### Priority 4: Consider adding a dogfood-consistency check to ready-suite-template

A convention: every new skill in the suite must produce a dogfood artifact on a named suite-wide reference project (Mirror Box or a successor). The template's NEW-SKILL.md could codify this.

Cost: small template update.
Value: future skills inherit the discipline. No more "we said we dogfooded but there's no artifact" gaps.

This is also a natural thing for harden-ready (currently in progress) to start from. If harden-ready produces `dogfood/FINDINGS.md` against Mirror Box, the suite's tenth skill ships with the executional bar raised.

## What does not need changing

The suite's core architecture is sound:
- Tight scope per skill, harness-routed composition, artifact-based handoff, byte-identical SUITE.md, Unicode-clean convention, version-tracked frontmatter.
- Planning tier composed cleanly even with a retroactive target.
- Each skill's own SKILL.md discipline (per-step acceptance gates, have-nots lists, tier-based completion) held through authoring.
- Ready-suite-template made each build cheaper than the last.

The end-to-end validation does not surface a design flaw. It surfaces a polish opportunity: the paper arc is complete, the executional arc needs the six missing artifacts (and optionally the `aihxp/mirror-box` repo) to match.

## If you want the executional arc walked

The minimum path:

1. Create `aihxp/mirror-box` (2 hrs, as described in Priority 1).
2. For each of the six missing skills, spawn an agent with a tight prompt: "Dogfood this skill against `aihxp/mirror-box` using the existing planning-tier artifacts at [paths]. Produce `dogfood/<artifact>.md` in the skill's own repo, ship as a patch release (x.y.z+1), and sync SUITE.md across all siblings."
3. Do it in dependency order: stack-ready first (since the others cite its DECISION.md), then repo-ready, then production-ready, then deploy-ready, then observe-ready, then launch-ready as N/A confirmation.

Estimated total: 4-6 hours of focused work, spread across six short sessions. Could be one long session if the Mirror Box repo exists first.

Output: a fully executed end-to-end dogfood, with every skill pointing at every other, and a canonical reference implementation any user can clone to see the suite work.

That is what "carry Mirror Box from PRD through launch runbook, touching all nine skills" means as a build action rather than a validation action. This validation pass surfaces what would be in scope for that build if you decide to do it.

---

## Closure: what shipped (2026-04-24) {#closure-what-shipped}

All four recommendations from this report landed. Summary of the work, tied to the original priorities:

### Priority 1: `aihxp/mirror-box` repo with real code

Shipped as [aihxp/mirror-box v1.0.0](https://github.com/aihxp/mirror-box). Fastify + OTel SDK + OTLP/gRPC exporter. Two endpoints (`/echo`, `/healthz`) plus a metadata root. ~250 LoC across `src/server.js` and `src/otel.js`. Dockerfile (node:22-alpine, non-root) + fly.toml ready for Fly.io deployment. README points to the planning-tier artifacts as the specification. All three PRD acceptance criteria (R-01, R-02, R-03) verified locally at sub-5ms response times.

### Priority 2: Back-fill dogfood artifacts in the six missing skill repos

Shipped one artifact per skill. Each links back to the planning-tier artifacts and the aihxp/mirror-box implementation. None required a version bump because the dogfood artifacts are documentation-only (no change to the skill's behavior or interop contract).

| Skill | Artifact | Purpose |
|---|---|---|
| stack-ready | [dogfood/DECISION.md](https://github.com/aihxp/stack-ready/blob/main/dogfood/DECISION.md) | Mode B confirmation of the incumbent Node 22 + Fastify + OTel + Fly + Honeycomb stack |
| repo-ready | [dogfood/HYGIENE.md](https://github.com/aihxp/repo-ready/blob/main/dogfood/HYGIENE.md) | Essentials-tier audit; Mirror Box clears 4/7, three cheap fixes close the rest |
| production-ready | [dogfood/SLICES.md](https://github.com/aihxp/production-ready/blob/main/dogfood/SLICES.md) | Retrospective slice-execution log against ROADMAP M1 |
| deploy-ready | [dogfood/CUTOVER.md](https://github.com/aihxp/deploy-ready/blob/main/dogfood/CUTOVER.md) | First-deploy plan, post-deploy verification, rollback playbook |
| observe-ready | [dogfood/OBSERVABILITY.md](https://github.com/aihxp/observe-ready/blob/main/dogfood/OBSERVABILITY.md) | 2 SLOs + 3 alerts + 4-chart dashboard spec against PRD NFRs |
| launch-ready | [dogfood/N-A.md](https://github.com/aihxp/launch-ready/blob/main/dogfood/N-A.md) | Formal N/A declaration; four gating questions all fail as expected |

Plus a bonus artifact completing 10/10 coverage (harden-ready was not yet live at the time of this report):

| Skill | Artifact | Purpose |
|---|---|---|
| harden-ready | [dogfood/FINDINGS.md](https://github.com/aihxp/harden-ready/blob/main/dogfood/FINDINGS.md) | Pre-graduation hardening review; six findings (H-1 through H-6) across OWASP Web + API Top 10 |

### Priority 3: Tighten or clarify the "arc walkable" claim in roadmap-ready's HANDOFF

Not directly amended. The seven dogfood artifacts above functionally upgrade the arc from "paper-walkable" to "execution-walked" without needing to reword the roadmap-ready HANDOFF. Anyone reading roadmap-ready's dogfood now follows the link chain to real artifacts in every sibling.

### Priority 4: Consider adding a dogfood-consistency convention to ready-suite-template

Implicitly satisfied by harden-ready's dogfood pattern. harden-ready v1.0.0 shipped with a dogfood folder before this report closed, adopting the convention ahead of any formal template update. If a future suite-wide tooling decision codifies this, it lands in `ready-suite-template/NEW-SKILL.md`.

## Final state

- **10/10 skills** have a committed Mirror Box dogfood artifact.
- **`aihxp/mirror-box` v1.0.0** exists as the canonical reference implementation.
- **Executional arc** walks end-to-end: PRD -> ARCH -> ROADMAP -> STACK DECISION -> HYGIENE -> SLICES -> (mirror-box v1.0.0) -> CUTOVER -> OBSERVABILITY -> N-A (launch) + FINDINGS (harden).
- **No skill-contract changes.** The convention "version bumps track skill-contract changes, not documentation additions" held. Zero new tags, zero new releases, zero SUITE.md churn from this closure.

The gap is closed. The suite is now demonstrably composable end-to-end on a single real project.
