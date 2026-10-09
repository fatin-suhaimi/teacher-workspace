# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (source unchanged as of 2026-10-09 22:50); docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0; batches 1 (startup, config, middleware), 2 (sessions, CSRF), 3 (Edupass sign-in), 4 (page render, static), 5 (API proxy), 6 (request ID, logging), 7 (host shell UI), 8 (delivery, ops, local dev), 9 (security consolidation), 10 (data model, glossary)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Server source 21/21; host 24/33 (remaining 9 are shadcn `components/ui/*`); mock-edupass source all read; build, CI, tooling and repo docs all read. Tests reviewed mostly by case name.
- Docs written: `00`-`04`, `06`-`11`, `13`, `subsystems/{sessions,page-render,observability,host-shell}`, `workflows/{edupass-sign-in,api-proxy}`. Not yet written: `05-feature-to-code-map.md`, `12-change-impact-guide.md`.
- See `03-coverage-ledger.md`.

## Key discoveries (evidence-backed)

1. No database: the only persisted data is the session snapshot `{id, csrf_token, user{email}, data}` in Valkey or memory with idle TTL. Everything else is config or data in transit (`06-data-model.md`).
2. Routes and flows: `04-api-catalog.md`, `workflows/*`.
3. Security: identity stops at the session (no SPA flag, no backend claim); R1 unauthenticated `/api/`, R2 CSRF unenforced, R3 no authorisation (`07-security-and-auth.md`).
4. Domain: remotes `pg` (Parents Gateway: Posts, Groups) and `si` (Student Insights); Edupass groups `<location>_<TW|TWSTG>_<ROLE|ATTR>_<NAME>` not yet used (`10-business-glossary.md`).
5. Delivery and local dev: arm64 image to ECR via release PRs; three local processes; two CONTRIBUTING gaps (`08-*`, `09-*`).

## Top open questions

Q21/Q8/Q38 (proxy auth, CSRF, token delivery), Q26 (authorisation model), Q30 (SPA sign-in state), Q29 (headers), Q40 (format check), Q41/Q42 (CONTRIBUTING gaps), Q1 (ADR-0001 image), Q4 (canonical GitHub org). Full list: `11-open-questions-and-discrepancies.md`.

## Remaining work

| Order | Batch | Output |
| --- | --- | --- |
| **11 (next)** | **Phase 6 consolidation, part 1** | `05-feature-to-code-map.md` (features: sign-in, session, page shell and catalogue, remote mounting for Posts/Groups, API proxy, welcome modal, observability; each mapped to entry point, routes, handlers, data, external deps, tests, diagrams, blast radius) and `12-change-impact-guide.md` (common changes: new remote, new config, user claims, CSRF enforcement, new route, catalogue edit, session format; files, tests and partner impact for each) |
| 12 | Phase 6 consolidation, part 2 (`AUDIT`) | Cross-check every doc for stale citations, contradictions and Inferred items later verified; refresh `00-project-overview.md` (still marked provisional) and `01-repository-map.md` statuses; tidy `11-*` (close Q6, now answered by batch 5); final README reading path; mark Phase 6 complete only if the ledger supports it |

## Next batch (precise)

**Batch 11: feature-to-code map and change-impact guide.** Build both from existing docs, spot-checking cited lines where a mapping is load-bearing. No new source files.

Suggested command: `CONTINUE`

## Diagram validation

All 29 Mermaid blocks parse with mermaid 11.4.1. Visual layout not reviewed.
