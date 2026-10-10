# Engineering Onboarding Knowledge Base

Evidence-backed onboarding docs for Teacher Workspace, built incrementally following `CODEBASE_MASTER_PROMPT.md` (in the claude.ai Project "Teacher Workspace 2.0").

- **Revision analysed:** `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (read from `.git/refs/heads/main`; uncommitted local changes were not checked)
- **Status:** Phases 0 to 6 complete as of 2026-10-09, within the limits recorded in `03-coverage-ledger.md` (audit summary): every in-scope first-party source and config file inspected; test files reviewed by case name; deployment infrastructure outside the repo.
- **Keeping it current:** when `main` moves, re-inspect changed files and update the affected docs (`UPDATE <scope>` in the master prompt). Citations are `file:line` at the revision above.

## Status labels used throughout

| Label | Meaning |
| --- | --- |
| Verified | Read in source at the stated revision |
| Inferred | Strongly suggested by names, config, docs, tests or library/platform behaviour, but not shown by code in this repo |
| Unknown | No evidence yet |
| Conflicts with documentation | Code and repo docs disagree |

## Documents

| File | Purpose |
| --- | --- |
| [00-project-overview.md](00-project-overview.md) | What the system is, tech stack, verified system context, what to know first |
| [01-repository-map.md](01-repository-map.md) | Directory layout, subsystems, entry points, runtime dependencies |
| [02-architecture.md](02-architecture.md) | Layering, containers, startup and shutdown, config loading and full config reference, middleware order |
| [03-coverage-ledger.md](03-coverage-ledger.md) | Per-file review status, interface coverage, audit summary |
| [04-api-catalog.md](04-api-catalog.md) | Every route (TW server and mock-edupass) with auth, inputs, outputs, failures, tests |
| [05-feature-to-code-map.md](05-feature-to-code-map.md) | 15 features traced from UI to routes, handlers, data, external deps, tests and docs; blast radius |
| [06-data-model.md](06-data-model.md) | Every persisted and transmitted structure, who writes and reads it, invariants, identity gap |
| [07-security-and-auth.md](07-security-and-auth.md) | Trust boundaries, identity flow, controls in place, authorisation gap, sensitive data, risk register R1-R14 |
| [08-infrastructure-and-operations.md](08-infrastructure-and-operations.md) | Image build, CI, release, runtime topology (provisional: outside the repo), operations, supply chain |
| [09-local-development-and-testing.md](09-local-development-and-testing.md) | Toolchain, setup, running all local processes, signing in, tests, lint and hooks, conventions, troubleshooting |
| [10-business-glossary.md](10-business-glossary.md) | Product, architecture, identity, delivery and catalogue terms, each with its source |
| [11-open-questions-and-discrepancies.md](11-open-questions-and-discrepancies.md) | Maintainer shortlist, doc/code conflicts, open questions Q1-Q44 |
| [12-change-impact-guide.md](12-change-impact-guide.md) | 14 change recipes: files, tests, docs, partner impact, release impact |
| [13-session-handoff.md](13-session-handoff.md) | Resume checkpoint for future sessions |
| [subsystems/sessions.md](subsystems/sessions.md) | Session model, middleware lifecycle, TTLs, cookie, CSRF, stores, risks |
| [subsystems/page-render.md](subsystems/page-render.md) | Dev vs prod page rendering, preloaded state contract, static serving, response helpers |
| [subsystems/observability.md](subsystems/observability.md) | Request ID, access log, full log catalog, what is missing, debugging guide |
| [subsystems/host-shell.md](subsystems/host-shell.md) | React host: boot, routes, navigation, catalogue, welcome modal, styling, error handling |
| [workflows/edupass-sign-in.md](workflows/edupass-sign-in.md) | Edupass OIDC sign-in end to end: PKCE, client auth, validation, failures, claims |
| [infra/README.md](infra/README.md) | Deployment infrastructure for TW from the org infra repo `dxd-transform-infrastructure` (TW scope only): per-environment footprint, hostnames, provisional deployment map, infra open questions. Phase 0 |
| [workflows/api-proxy.md](workflows/api-proxy.md) | `/api/` reverse proxy: path mapping, JWT contract for partner backends, failures, ADR-0001 gap |

## Reading path

**Day 1: orientation**

1. `00-project-overview.md`
2. `10-business-glossary.md` (keep it open)
3. `01-repository-map.md`, then `02-architecture.md`
4. `09-local-development-and-testing.md`: get all three processes running and sign in as a mock user
5. The repo's own `CONTRIBUTING.md`, `docs/adr/0001-*`, `docs/adr/0002-*`

**Day 2: how requests work**

6. `subsystems/sessions.md`, then `workflows/edupass-sign-in.md`
7. `subsystems/page-render.md`, then `subsystems/host-shell.md`
8. `workflows/api-proxy.md`, then `04-api-catalog.md`
9. `subsystems/observability.md`

**Day 3: what to work on and how**

10. `07-security-and-auth.md` (the risk register is the best summary of what is unfinished)
11. `06-data-model.md`, `05-feature-to-code-map.md`
12. `12-change-impact-guide.md` before your first change
13. `11-open-questions-and-discrepancies.md` (maintainer shortlist) for your first conversations with the team
