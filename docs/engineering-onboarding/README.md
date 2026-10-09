# Engineering Onboarding Knowledge Base

Evidence-backed onboarding docs for Teacher Workspace, built incrementally following `CODEBASE_MASTER_PROMPT.md` (in the claude.ai Project "Teacher Workspace 2.0").

- **Revision analysed:** `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (read from `.git/refs/heads/main`; uncommitted local changes were not checked)
- **Current phase:** Phase 2 in progress. Done: Phase 0, batch 1 (startup/config), batch 2 (sessions and CSRF), batch 3 (Edupass sign-in), batch 4 (page render and static), batch 5 (API proxy), batch 6 (request ID and logging), batch 7 (host shell UI), batch 8 (delivery and local development), batch 9 (security consolidation). All server source and all hand-written host source are now inspected.
- **Last updated:** 2026-10-09

## Status labels used throughout

| Label | Meaning |
| --- | --- |
| Verified | Read in source at the stated revision |
| Inferred | Strongly suggested by names, config, docs or tests, but the implementing code has not been read |
| Unknown | No evidence yet |
| Conflicts with documentation | Code and repo docs disagree |

## Documents

| File | Purpose | Status |
| --- | --- | --- |
| [00-project-overview.md](00-project-overview.md) | What the system is, tech stack, provisional system map | Phase 0 (provisional) |
| [01-repository-map.md](01-repository-map.md) | Directory layout, subsystems, entry points, dependencies | Phase 0 |
| [02-architecture.md](02-architecture.md) | Layering, containers, startup/shutdown, config loading and reference, middleware order | Phase 1 batch 1 |
| [03-coverage-ledger.md](03-coverage-ledger.md) | Per-file review status | Phase 0 |
| [04-api-catalog.md](04-api-catalog.md) | Every route/interface | Server routes complete; mock-edupass routes listed |
| 05-feature-to-code-map.md | Feature to code traceability | Not started |
| 06-data-model.md | Session data, tokens, stored state | Not started |
| [07-security-and-auth.md](07-security-and-auth.md) | Trust boundaries, identity flow, controls in place, authorisation gap, sensitive data, prioritised risk register (R1-R14) | Batch 9 |
| [08-infrastructure-and-operations.md](08-infrastructure-and-operations.md) | Image build, CI, release, runtime topology (provisional), operational characteristics, supply-chain guards | Batch 8 |
| [09-local-development-and-testing.md](09-local-development-and-testing.md) | Toolchain, setup, running all local processes, signing in, tests, lint/format/hooks, conventions, troubleshooting | Batch 8 |
| 10-business-glossary.md | Domain terms | Not started |
| [subsystems/sessions.md](subsystems/sessions.md) | Session model, middleware lifecycle, TTLs, cookie, CSRF, stores, risks | Phase 2 batch 2 |
| [subsystems/page-render.md](subsystems/page-render.md) | Dev vs prod page rendering, preloaded state contract, static serving, response helpers | Batch 4 |
| [subsystems/observability.md](subsystems/observability.md) | Request ID, access log, full log catalog, what is missing (metrics, tracing, health, panic recovery), debugging guide | Batch 6 |
| [subsystems/host-shell.md](subsystems/host-shell.md) | React host: boot sequence, routes, navigation, app catalogue, welcome modal, styling conventions, error handling, integration points | Batch 7 |
| [workflows/edupass-sign-in.md](workflows/edupass-sign-in.md) | Edupass OIDC sign-in end to end: PKCE, client auth, validation, failures, claims | Batch 3 |
| [workflows/api-proxy.md](workflows/api-proxy.md) | `/api/` reverse proxy: path mapping, JWT contract for remote backends, failure paths, ADR-0001 gap | Batch 5 |
| [11-open-questions-and-discrepancies.md](11-open-questions-and-discrepancies.md) | Unknowns and doc/code conflicts | Phase 0 |
| 12-change-impact-guide.md | Where changes ripple | Not started |
| [13-session-handoff.md](13-session-handoff.md) | Resume checkpoint for the next session | Current |

## Suggested reading path (so far)

1. `00-project-overview.md`
2. `01-repository-map.md`
3. `02-architecture.md`
4. `09-local-development-and-testing.md` (get it running)
5. `07-security-and-auth.md` (the risk register is the best summary of what is unfinished)
6. Repo's own docs: `CONTRIBUTING.md`, `docs/adr/0001-*`, `docs/adr/0002-*`
7. `13-session-handoff.md` for what to look at next
