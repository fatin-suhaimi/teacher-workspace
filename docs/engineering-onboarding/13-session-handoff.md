# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (unchanged as of 2026-10-09 23:45); docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Status:** **Phases 0-6 complete** (2026-10-09): Phase 0 inventory; batches 1-11 analysis and docs; batch 12 AUDIT. Limits are recorded in `03-coverage-ledger.md` (audit summary).
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each change to files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage

- Every in-scope first-party source and config file inspected (server 21/21, host 24/24 hand-written, mock-edupass 8/8 non-test-body, build and CI all).
- Excluded with reasons: shadcn `components/ui/*`, assets, `dist`, `node_modules`, lockfiles, `.env`.
- Tests: case names reviewed across all suites; selected bodies read.
- Audit: 584 citations range-checked, 34 content-sampled; stale labels fixed; no contradictions remain.

## Key discoveries

1. Identity stops at the session: the SPA has no signed-in flag; `/api/` accepts anonymous callers and sends backends a JWT with no user claim; CSRF tokens are minted but never verified (`07-security-and-auth.md` R1-R3).
2. No database: the session snapshot is the only persisted state (`06-data-model.md`).
3. Remote identifiers are hard-coded in four places (`05-*`, `12-*` C1).
4. Local dev needs three processes; two `CONTRIBUTING.md` commands are incomplete (`09-*`, Q41, Q42).
5. CI may not fail on formatting, and does not typecheck the host or run mock tests (Q40, Q12).

## Questions for maintainers

See the shortlist at the top of `11-open-questions-and-discrepancies.md` (Q21/Q8/Q38, Q26, Q22/Q27, Q29/Q11/Q35, Q1, Q40/Q12, Q41/Q42/Q3, Q4).

## Suggested next steps (optional, on request)

| Command | When |
| --- | --- |
| `UPDATE <scope>` | after `main` moves past `5ff58a7`: re-inspect changed files and update affected docs |
| `TRACE <route>` / `FEATURE <name>` / `IMPACT <change>` | targeted questions; answer from existing docs plus fresh code reads |
| Test-body review | if deeper evidence of test coverage is wanted (start with `auth_test.go` callback cases, `proxy_test.go`, `middleware/session_test.go`) |
| Small docs PRs to the repo | Q3, Q41, Q42 fixes to `CONTRIBUTING.md` and `apps/mock-edupass/README.md` (separate branch, not this docs branch) |

## Diagram validation

All Mermaid blocks parse with mermaid 11.4.1. Visual layout not reviewed.
