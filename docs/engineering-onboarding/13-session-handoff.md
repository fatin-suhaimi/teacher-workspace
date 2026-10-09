# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (unchanged as of 2026-10-09 23:29); docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0; batches 1-11 (startup/config, sessions/CSRF, Edupass sign-in, page render, API proxy, observability, host shell, delivery/local dev, security, data model/glossary, feature map/change-impact)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Server source 21/21; host 24/33 (remaining 9 are shadcn `components/ui/*`); mock-edupass source all read; build, CI, tooling and repo docs all read. Tests reviewed mostly by case name.
- All planned docs now exist: `00`-`13`, `subsystems/{sessions,page-render,observability,host-shell}`, `workflows/{edupass-sign-in,api-proxy}`.
- See `03-coverage-ledger.md`.

## Key discoveries (evidence-backed)

1. 15 features mapped end to end (`05-feature-to-code-map.md`); highest blast radius: `internal/config`, the Session middleware, the `PreloadedState` contract, remote identifiers in four places, the JWT claim set, and MF shared singletons.
2. 14 change recipes with tests, docs, partner and release impact (`12-change-impact-guide.md`). C4 (auth on `/api/` plus user claims) and C5 (CSRF) are major-version changes under ADR-0002 and should ship together.
3. Security: R1 unauthenticated `/api/`, R2 CSRF unenforced, R3 no authorisation (`07-security-and-auth.md`).
4. Data: no database; the session snapshot is the only persisted state (`06-data-model.md`).

## Top open questions

Q21/Q8/Q38 (proxy auth, CSRF, token delivery), Q26 (authorisation model), Q30 (SPA sign-in state), Q29 (headers), Q40 (format check), Q41/Q42 (CONTRIBUTING gaps), Q1 (ADR-0001 image), Q4 (canonical GitHub org).

## Remaining work

| Order | Batch | Output |
| --- | --- | --- |
| **12 (next, last planned)** | **`AUDIT`: Phase 6 consolidation** | Cross-check every doc: stale or wrong line citations (sample-verify against source), contradictions between docs, items still labelled Inferred or Unknown that later batches verified; refresh `00-project-overview.md` (still marked provisional; system map now verified) and `01-repository-map.md` subsystem statuses; tidy `11-*` (close Q6, answered by batch 5; strike resolved items; add a short "questions for maintainers" shortlist); final README reading path and phase status; mark Phase 6 complete only if the ledger supports it |
| Later (optional) | Test-body review | read test bodies for the highest-risk areas (callback guards, proxy, session middleware) if deeper test-coverage evidence is wanted |
| Later (on change) | `UPDATE <scope>` | re-inspect when `main` moves past `5ff58a7` |

## Next batch (precise)

**Batch 12: `AUDIT`.** Re-read all docs; sample-verify about 30 citations against the staged sources (prioritise `07`, `workflows/api-proxy.md`, `subsystems/sessions.md`); fix contradictions; update `00`, `01`, `11`, README; produce an audit summary section in `03-coverage-ledger.md`.

Suggested command: `CONTINUE` (or `AUDIT`)

## Diagram validation

All 30 Mermaid blocks parse with mermaid 11.4.1. Visual layout not reviewed.
