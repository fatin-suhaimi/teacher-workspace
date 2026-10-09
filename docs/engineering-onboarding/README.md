# Engineering Onboarding Knowledge Base

Evidence-backed onboarding docs for Teacher Workspace, built incrementally following `CODEBASE_MASTER_PROMPT.md` (in the claude.ai Project "Teacher Workspace 2.0").

- **Revision analysed:** `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (read from `.git/refs/heads/main`; uncommitted local changes were not checked)
- **Current phase:** Phase 2 in progress. Done: Phase 0, batch 1 (startup/config), batch 2 (sessions and CSRF), batch 3 (Edupass sign-in).
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
| [04-api-catalog.md](04-api-catalog.md) | Every route/interface | Started (auth routes complete) |
| 05-feature-to-code-map.md | Feature to code traceability | Not started |
| 06-data-model.md | Session data, tokens, stored state | Not started |
| 07-security-and-auth.md | Edupass OIDC, sessions, CSRF, signed tokens | Not started |
| 08-infrastructure-and-operations.md | Docker, ECR, release, runtime config | Not started |
| 09-local-development-and-testing.md | Running and testing locally | Not started |
| 10-business-glossary.md | Domain terms | Not started |
| [subsystems/sessions.md](subsystems/sessions.md) | Session model, middleware lifecycle, TTLs, cookie, CSRF, stores, risks | Phase 2 batch 2 |
| [workflows/edupass-sign-in.md](workflows/edupass-sign-in.md) | Edupass OIDC sign-in end to end: PKCE, client auth, validation, failures, claims | Batch 3 |
| [11-open-questions-and-discrepancies.md](11-open-questions-and-discrepancies.md) | Unknowns and doc/code conflicts | Phase 0 |
| 12-change-impact-guide.md | Where changes ripple | Not started |
| [13-session-handoff.md](13-session-handoff.md) | Resume checkpoint for the next session | Current |

## Suggested reading path (so far)

1. `00-project-overview.md`
2. `01-repository-map.md`
3. `02-architecture.md`
4. Repo's own docs: `CONTRIBUTING.md`, `docs/adr/0001-*`, `docs/adr/0002-*`
5. `13-session-handoff.md` for what to look at next
