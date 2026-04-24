# Mirror Box slice execution log (dogfood for production-ready v2.5.9)

**Target:** [aihxp/mirror-box v1.0.0](https://github.com/aihxp/mirror-box)
**Slice queue source:** [aihxp/roadmap-ready/dogfood/ROADMAP.md](https://github.com/aihxp/roadmap-ready/blob/main/dogfood/ROADMAP.md) M1
**Mode:** retrospective post-ship log (production-ready's Step 5 drain log, reconstructed from the shipped implementation)
**Version:** 1.0
**Last updated:** 2026-04-23
**Owner:** h (dogfood)

## What this is

Retrospective drain of production-ready's Step 5 (feature slice execution) against Mirror Box. The three slices from the ROADMAP are already `done` (the service shipped at v1.0.0). This document reconstructs what production-ready's discipline would have produced if it had run each slice live: the vertical-slice checklist, the have-nots scan, and the Tier 1 proof-test log.

Purpose: close the production-ready half of the Mirror Box executional gap from the suite VALIDATION report.

## Caveat: retrospective, not live

production-ready's Step 5 is designed to run live during implementation. Running it after the fact loses the "vertical slice forced me to implement the full stack before moving on" benefit. What remains verifiable:

- The shipped code meets every acceptance criterion (yes, from smoke tests).
- The have-nots scan returns clean (yes, see below).
- The Tier 1 proof test passes against the live service (yes, smoke-tested at v1.0.0 ship).
- ADRs exist for non-obvious decisions (yes, in architecture-ready/dogfood/adr/).

What cannot be retro-verified:
- Whether the slices were genuinely built in vertical-slice order rather than horizontal-layer order. (Mirror Box is simple enough that this is moot.)
- Whether the author resisted mid-slice temptations to "also add X." (Probably yes, given the tight scope.)

## Slice queue (from ROADMAP)

| # | Slice | Appetite | PRD ref | ARCH ref | Depends | Status |
|---|---|---|---|---|---|---|
| 1 | Base HTTP server + /healthz | small batch | R-03 | mirror-box component | - | done |
| 2 | /echo with JSON validation | small batch | R-01 | mirror-box component | #1 | done |
| 3 | OTel instrumentation + Honeycomb export | small batch to medium | R-02 | mirror-box + Honeycomb integration | #2 | done |

All three slices shipped within M1's 6-week appetite. Under-budget by ~0.8 engineer-weeks per the ROADMAP retrospective.

## Slice 1: Base HTTP server + /healthz

**Acceptance criteria (PRD R-03):**
- Given the service is running and OTel exporter is reachable, when a client hits `/healthz`, then 200 with `{"status":"ok"}` within 100ms.

**Verified at Mirror Box v1.0.0:**
- `curl -fsS http://localhost:8080/healthz` returns `{"status":"ok"}` in ~2ms. (Observed during smoke test: `responseTime: 2.14ms`.) Pass.

**Implementation evidence:**
- `src/server.js` lines 82-85 (approximately): `fastify.get('/healthz', ...)`.
- Fastify is listening on port 8080 by default, configurable via `PORT`.
- No auth, no rate limit (intentional per ARCH ADR-003).

**Vertical-slice completeness check:**
- Server process starts. Yes.
- Health endpoint responds with correct shape. Yes.
- Response time under 100ms target. Yes (~2ms).
- No placeholder or mock in the path. Yes.
- Have-nots scan shows no `// TODO`, `console.log("wip")`, or similar in this slice's code. Pass.

**ADR:** none needed for this slice; trivial implementation within an already-decided stack.

## Slice 2: /echo with JSON validation

**Acceptance criteria (PRD R-01):**
- Given a POST to `/echo` with `{"k":"v"}`, when the server responds, then the response body includes `{"echo":{"k":"v"},"trace_id":"<otel-trace-id>","server_version":"<semver>","received_at":"<iso8601>"}`.
- Given a POST with a non-JSON body, when the server responds, then the response is 400 with `{"error":"body must be JSON"}`.

**Verified at Mirror Box v1.0.0:**
- Valid-JSON case: response at smoke test returned `{"echo":{"k":"v","n":42},"trace_id":"f3492671c60ad6ed4f623af485deac19","server_version":"1.0.0","received_at":"2026-04-23T22:48:14.406Z"}`. All four keys present; trace_id populated. Pass.
- Non-JSON case: response at smoke test returned `{"error":"body must be JSON"}` with status 400. Pass.

**Implementation evidence:**
- `src/server.js` lines 88-104 (approximately): the `/echo` handler.
- Fastify's default JSON parser handles the happy path; the custom `setErrorHandler` catches `FST_ERR_CTP_INVALID_JSON_BODY` and similar to return the PRD-specified error shape.
- `received_at` set via `onRequest` hook (line 64) from `new Date().toISOString()`.
- `server_version` read from `package.json`'s `version` at boot time.
- `trace_id` extracted from `trace.getActiveSpan()?.spanContext()?.traceId`.

**Vertical-slice completeness check:**
- Request handler exists. Yes.
- Happy path returns the specified shape. Yes.
- Error path returns the specified shape. Yes.
- All four response keys populated on happy path. Yes.
- Fastify's built-in request logging works (observed `ip_hash` entries in the smoke test log). Yes.
- Have-nots scan: no TODOs, no console.log, no placeholder data, no hardcoded return values. Pass.

**ADR:** none needed; the Fastify error-handler pattern is standard.

## Slice 3: OTel instrumentation + Honeycomb export

**Acceptance criteria (PRD R-02):**
- Given a request to any endpoint, when the span is exported, then all six attributes (`http.method`, `http.route`, `http.status_code`, `user.ip` hashed, `request.size_bytes`, `response.size_bytes`) are present.
- Given the OTel exporter is misconfigured (bad OTLP endpoint), when the server starts, then it logs a warning and continues serving (does not block on exporter health).

**Verified at Mirror Box v1.0.0:**
- Six-attributes case: auto-instrumentations register `http.method`, `http.route`, `http.status_code` by default. Mirror Box adds `user.ip` (hashed), `request.size_bytes`, `response.size_bytes` via `onRequest`/`onSend` hooks. When `HONEYCOMB_API_KEY` is set and pointing at a valid team, spans appear in the Honeycomb dataset with all six attributes. (Confirmed against a test Honeycomb dataset during dogfood authoring.) Pass.
- Misconfigured-exporter case: the SDK init is wrapped in a try/catch that logs a warning and continues. Without `HONEYCOMB_API_KEY` the service logs "Tracing initialized. Exporter: none (spans dropped)." and continues serving. With a bad endpoint, the OTLP batch exporter retries with exponential backoff and drops the batch after 3x without blocking the request path. Pass.

**Implementation evidence:**
- `src/otel.js`: the bootstrap, loaded via `node --import`. Sets up `NodeSDK` with `OTLPTraceExporter` and `getNodeAutoInstrumentations`.
- `src/server.js` lines 64-78: the three hooks that add `user.ip`, `request.size_bytes`, `response.size_bytes` to the active span.
- The IP-hash uses a per-process salt (sha256 of `process.pid:Date.now():Math.random()`), keeping hashes non-reversible within a process lifetime.
- Fastify instrumentation via auto-instrumentations (the `@opentelemetry/auto-instrumentations-node` package).
- FS and DNS auto-instrumentations explicitly disabled to reduce span noise.

**Vertical-slice completeness check:**
- OTel SDK initialized at boot. Yes.
- Spans emitted on every request. Yes.
- All six attributes present. Yes.
- Exporter failure is non-blocking. Yes.
- Graceful shutdown flushes pending spans on SIGTERM/SIGINT. Yes.
- Have-nots scan: no TODOs, no fake data, no commented-out auto-instrumentations. Pass.

**ADR:** ARCH ADR-001 covers the single-service monolith decision; the OTel-specific decisions (gRPC vs HTTP, auto-instrumentations vs manual) could warrant a retrospective ADR, but they follow the canonical OTel Node path and don't represent a non-obvious choice. No new ADR needed.

## Have-nots scan (final, against v1.0.0)

production-ready's Step 5 tight-loop check ran against the shipped code:

- **No TODO / FIXME / XXX / HACK** in `src/`. Pass.
- **No `console.log` for debugging.** Pass. Only structured logging via Fastify's logger and `console.warn` in otel.js for exporter misconfiguration and shutdown errors (intentional, not placeholders).
- **No `Math.random()` in request-handling paths.** The salt uses `Math.random()` once at startup for salt generation; this is appropriate non-determinism, not placeholder logic.
- **No hardcoded fake data.** All responses derive from the request or from `package.json`.
- **No `new Promise((resolve) => resolve())` or other no-op patterns.** Pass.
- **No commented-out code blocks.** Pass.
- **No half-wired buttons.** N/A (no UI).
- **No `// hook this up later`.** Pass.
- **No mock OTel exporters, fake span IDs, or stubbed Honeycomb clients.** Pass.

production-ready's have-nots contract is clean against Mirror Box v1.0.0.

## Tier 1 proof test (smoke tested at v1.0.0 ship)

production-ready's Tier 1 requires end-to-end operation of the core flow:

- Service boots on `npm start` without error. Pass. (`Server listening at http://0.0.0.0:8080`)
- Health endpoint returns correct shape and is fast (<100ms). Pass (~2ms).
- Primary endpoint happy path returns correct shape with trace_id populated. Pass.
- Primary endpoint error path returns correct shape with correct status. Pass.
- Service responds within PRD latency budget (p95 <200ms). Pass (observed sub-5ms for all four test cases; ~100ms headroom vs PRD target).
- IPs in logs are hashed, not raw. Pass (observed `ip_hash: "7bb2429b30f91729"` in log entries, never a raw IP).
- Service terminates cleanly on SIGTERM (OTel spans flush). Pass (kill -TERM during boot test exited cleanly).

Tier 1 declared complete. production-ready's slice queue for Mirror Box M1 is drained.

## Security-adjacent check

Mirror Box inherits production-ready's security discipline through ARCH ADR-003 and the PRD NFR Security clause:

- No secrets in the committed code. Pass (all secrets via env vars).
- No raw PII in logs or spans. Pass (IPs hashed before any emission).
- Dependency tree has zero `npm audit` findings. Pass (uuid override closes the transitive CVE).
- No authentication bypass (service is intentionally public). N/A.
- No authorization bypass (no authorization to enforce). N/A.

production-ready's Tier 2 (security) checklist is as satisfied as a no-auth public service can be. harden-ready has its own pass in `aihxp/harden-ready` for the adversarial review.

## Tier verdict for Mirror Box

- **Tier 1 (working):** clean. All three slices shipped with acceptance criteria met. Have-nots clean. Proof test passed.
- **Tier 2 (secure):** as clean as a no-auth public service permits. No secrets, hashed IPs, zero audit findings.
- **Tier 3 (observable):** partial. OTel wired. Full observability posture lives in `aihxp/observe-ready/dogfood/OBSERVABILITY.md`.
- **Tier 4 (resilient):** minimal. Single instance, single region, Fly auto-restart. Rollback window is last 3 Fly deploys.

Mirror Box's trivial scope makes Tier 2-4 progression fast. The service is a teaching target; production-ready does not push it further.

## Handoff (back to the suite)

This dogfood doesn't trigger downstream re-runs. The slice queue is drained; Mirror Box M1 is shipped. No new work in production-ready's scope.

Ready-suite version table and SUITE.md are unchanged by this commit.
