# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/transformteamsg/teacher-workspace` (renamed from `String-sg` in #204)
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (2026-10-09); **synced to upstream `main` @ `abe54576c604b5879679bf98a7cfd1fd3d0e7e8e` on 2026-10-16** (`documentation-sync-state.json`, `15-documentation-sync-log.md`). Docs live on branch `docs/engineering-onboarding` in the clone `~/teacher-workspace-fatin`, whose remote `origin` is the personal fork `fatin-suhaimi/teacher-workspace` (there is no `upstream` remote; sync fetches `https://github.com/transformteamsg/teacher-workspace.git` directly)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Status:** **Phases 0-6 complete** (2026-10-09): Phase 0 inventory; batches 1-11 analysis and docs; batch 12 AUDIT. Limits are recorded in `03-coverage-ledger.md` (audit summary).
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each change to files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage

- Every in-scope first-party source and config file inspected (server 21/21, host 24/24 hand-written, mock-edupass 8/8 non-test-body, build and CI all).
- Excluded with reasons: shadcn `components/ui/*`, assets, `dist`, `node_modules`, lockfiles, `.env`.
- Tests: case names reviewed across all suites; selected bodies read.
- Audit: 584 citations range-checked, 34 content-sampled; stale labels fixed; no contradictions remain.

## Key discoveries

1. Identity mostly stops at the session: since #199/#212 sign-in requires one base role and `/api/` forwards `role`/`attributes`, but the SPA has no signed-in flag, the JWT has no `sub`, and CSRF tokens are minted but never verified (`07-security-and-auth.md` R1-R3).
2. No database: the session snapshot is the only persisted state (`06-data-model.md`).
3. Remote identifiers are hard-coded in four places (`05-*`, `12-*` C1).
4. Local dev needs three processes; two `CONTRIBUTING.md` commands are incomplete (`09-*`, Q41, Q42).
5. CI may not fail on formatting, and does not typecheck the host or run mock tests (Q40, Q12).

## Infrastructure track

Complete (2026-10-10), from `dxd-transform-infrastructure` @ `b345a06`, TW scope only. Docs are in `infra/`; the resume point and the prioritised maintainer action list are in `infra/13-infra-session-handoff.md`. Verified deployment facts are folded into `02` (deployed values), `07` (R12 update, R15-R18), `08` (topology, deploy, health probe) and `11` (Q11, Q14 resolved, cross-links).

Headline: the dev and stg ECS task definitions use env var names (`TW_OIDC_*`, `TW_API_PROXY_*`) that the app at `5ff58a7` does not read (infra IQ2).

## Questions for maintainers

See the shortlist at the top of `11-open-questions-and-discrepancies.md` (Q21/Q8/Q38, Q26, Q22/Q27, Q29/Q11/Q35, Q1, Q40/Q12, Q41/Q42/Q3, Q4, then the infra list).

## Suggested next steps (optional, on request)

| Command | When |
| --- | --- |
| `SYNC DOCUMENTATION` | weekly (scheduled Fridays 5pm SGT) or after `main` moves past the checkpoint in `documentation-sync-state.json`: follow `DOCUMENTATION_SYNC_PROMPT.md` |
| `TRACE <route>` / `FEATURE <name>` / `IMPACT <change>` | targeted questions; answer from existing docs plus fresh code reads |
| Test-body review | if deeper evidence of test coverage is wanted (start with `auth_test.go` callback cases, `proxy_test.go`, `middleware/session_test.go`) |
| Small docs PRs to the repo | Q3, Q41, Q42 fixes to `CONTRIBUTING.md` and `apps/mock-edupass/README.md` (separate branch, not this docs branch) |

## Diagram validation

All Mermaid blocks parse with mermaid 11.4.1. Visual layout not reviewed.
