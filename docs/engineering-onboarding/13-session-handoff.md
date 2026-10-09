# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (re-checked unchanged on 2026-10-09 22:30); docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0; batches 1 (startup, config, middleware), 2 (sessions, CSRF), 3 (Edupass sign-in), 4 (page render, static), 5 (API proxy), 6 (request ID, logging), 7 (host shell UI), 8 (delivery, ops, local dev)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Server source 21/21; host 24/33 (remaining 9 are shadcn `components/ui/*`); mock-edupass source all read; build, CI, tooling and repo docs all read.
- Tests are reviewed mostly by case name; few bodies read.
- Not covered by the repo at all: GitLab deploy, environments, load balancer, Valkey hosting (Q14).
- See `03-coverage-ledger.md`.

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, `/` catch-all. Chain: RequestID, RequestLog, routes.
2. Config: defaults, CWD `.env`, `Validate()`; exactly two remotes, all-or-none. `02-architecture.md`.
3. Sessions: JSON snapshot in memory or Valkey; sliding idle TTL 3h / 30m; no logout. `subsystems/sessions.md`.
4. Sign-in: OIDC code + PKCE + state + nonce; only `email` kept. `workflows/edupass-sign-in.md`.
5. Page: per-request `{csrfToken, remotes}`; no auth gating; no CSP/frame/HSTS. `subsystems/page-render.md`.
6. API proxy: HS256 JWT `{iss, aud, iat, exp}`; **no session check, no user claim, no CSRF** (Q21). `workflows/api-proxy.md`.
7. Observability: random `X-Request-ID`, JSON access log; no metrics, tracing, health, panic recovery. `subsystems/observability.md`.
8. Host shell: loads `pg/Posts`, `pg/Groups`; no API calls; hard-coded catalogue; no error reporting or tests. `subsystems/host-shell.md`.
9. Delivery: 3-stage arm64-only image (Go + built SPA, non-root, no healthcheck); PR CI builds `pr-N-sha` images for same-repo PRs; release-PR model publishes `vX.Y.Z` to ECR then tags; GitLab deploys. CI does not typecheck the host, run mock tests, or (likely) fail on formatting (Q40). Local dev needs three processes; two CONTRIBUTING commands are incomplete (Q41, Q42). `08-*`, `09-*`.

## Top open questions

Q21 (unauthenticated `/api/`), Q26 (authorisation model), Q30/Q38 (sign-in state and CSRF token for SPA and remotes), Q29 (security headers), Q40 (format check), Q42 (CONTRIBUTING remote command), Q1 (ADR-0001 image). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended order (updated)

| Order | Batch | Output |
| --- | --- | --- |
| **9 (next)** | **Security consolidation** | `07-security-and-auth.md`: trust boundaries, identity flow, session and cookie security, CSRF, proxy tokens, secrets handling, headers, supply chain, a prioritised risk register. Built from batches 2-8 with spot re-checks of cited lines; no new subsystems |
| 10 | Data and domain | `06-data-model.md` (session snapshot, preloaded state, JWT claims, config entities, ER-style diagram), `10-business-glossary.md` (Edupass, MFE, remotes, PG, SI, TWSTG, group naming, apps in the catalogue) |
| 11 | Phase 6 consolidation | `05-feature-to-code-map.md`, `12-change-impact-guide.md`, `AUDIT` pass across all docs, refresh `00-project-overview.md` and the README reading path |

## Next batch (precise)

**Batch 9: security consolidation.** Write `07-security-and-auth.md` from the existing docs, re-checking cited lines only where a claim is load-bearing (`proxy.go:48-78`, `middleware/session.go:88-111`, `auth.go:100-249`, `session.go:189-199`, `csrf.go`, `httputil.go:45-54`, `config.go:321-427`). Include an authentication/authorisation diagram, a trust-boundary data-flow diagram, and a risk register ranking Q21, Q8, Q26, Q22/Q27, Q29, Q23, Q31, Q39 by impact and likelihood with suggested fixes.

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`, `01`, `02`, `08`, `09`, `subsystems/*`, `workflows/*` parse with mermaid 11.4.1. Visual layout not reviewed.
