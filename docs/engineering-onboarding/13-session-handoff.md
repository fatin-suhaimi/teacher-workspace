# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (source files unchanged as of 2026-10-09 22:43); docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0; batches 1 (startup, config, middleware), 2 (sessions, CSRF), 3 (Edupass sign-in), 4 (page render, static), 5 (API proxy), 6 (request ID, logging), 7 (host shell UI), 8 (delivery, ops, local dev), 9 (security consolidation)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Server source 21/21; host 24/33 (remaining 9 are shadcn `components/ui/*`); mock-edupass source all read; build, CI, tooling and repo docs all read.
- Tests reviewed mostly by case name.
- Outside the repo: GitLab deploy, environments, load balancer, Valkey hosting (Q14).
- See `03-coverage-ledger.md`.

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, `/` catch-all. `04-api-catalog.md`.
2. Config: defaults, CWD `.env`, `Validate()`; two hard-coded remotes. `02-architecture.md`.
3. Sessions: JSON snapshot in memory or Valkey; sliding idle TTL 3h / 30m; rotation on sign-in; no logout. `subsystems/sessions.md`.
4. Sign-in: OIDC code + PKCE + state + nonce; only `email` kept. `workflows/edupass-sign-in.md`.
5. API proxy: HS256 JWT `{iss, aud, iat, exp}`; no session check, no user claim, no CSRF. `workflows/api-proxy.md`.
6. Page, host shell, observability, delivery: `subsystems/*`, `08-*`, `09-*`.
7. **Security risk register (`07-security-and-auth.md`):** R1 unauthenticated `/api/` with no identity (High); R2 CSRF unenforced (Medium, High after R1); R3 no authorisation from Edupass groups; R4 no security headers; R5 no logout or absolute lifetime; then hygiene items R6-R14. Suggested fix order: R1+R2 together, R3, R5, R4.

## Top open questions

Q21/Q8/Q38 (proxy auth, CSRF, token delivery to remotes), Q26 (authorisation model), Q29 (headers at the edge?), Q22/Q27 (logout policy), Q40 (format check), Q41/Q42 (CONTRIBUTING gaps), Q1 (ADR-0001 image). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended order (updated)

| Order | Batch | Output |
| --- | --- | --- |
| **10 (next)** | **Data model and glossary** | `06-data-model.md`: session snapshot, pending-login keys, preloaded state, JWT claims (TW to backends, client assertion to Edupass, ID token claims used), config entities, ER/class diagram, session state machine reference. `10-business-glossary.md`: Edupass, MFE / remote / host, PG, SI, TWSTG, group naming (`<loc>_<APP>_<ROLE or ATTR>_<NAME>`), catalogue apps, Onward, DXD/Transform terms seen in the repo. No new code reading expected beyond spot checks |
| 11 | Phase 6 consolidation | `05-feature-to-code-map.md`, `12-change-impact-guide.md`, `AUDIT` pass across all docs (stale citations, contradictions, Inferred items that later batches verified), refresh `00-project-overview.md` and README reading path |

## Next batch (precise)

**Batch 10: data model and glossary.** Write `06-data-model.md` and `10-business-glossary.md` from existing docs, spot-checking `session.go:43-61`, `index.go:14-22`, `proxy.go:61-66`, `auth.go:22-27, 155-164, 235-237`, `config.go:27-441`, mock `provider.ts:7-62`. Include a class/ER diagram of persisted and transmitted structures and a "who reads/writes what" table.

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`-`09`, `subsystems/*`, `workflows/*` parse with mermaid 11.4.1. Visual layout not reviewed.
