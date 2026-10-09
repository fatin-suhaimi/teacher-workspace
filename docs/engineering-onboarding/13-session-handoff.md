# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91`; docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0; batch 1 (startup, config, middleware); batch 2 (sessions, CSRF); batch 3 (Edupass sign-in); batch 4 (page render, static); batch 5 (API proxy)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Server source: 19/21 inspected. Remaining: `middleware/requestid.go`, `middleware/requestlog.go`.
- Server tests: mostly reviewed by case name; few bodies read.
- Host: bootstrap, routes, preloaded state, `LoginView`, `RootLayout`, `RemoteLoadFallbackView` read. Remaining: `HomeView`, `NotFoundView`, components (`Sidebar`, `AppCard`, `AppSection`, `WelcomeModal`, `ErrorBoundary`), hooks/helpers.
- mock-edupass: all source read; tests by name.
- See `03-coverage-ledger.md`. API catalog (`04-api-catalog.md`) is complete for the server.

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, `/` catch-all. `handler.go:93-105`.
2. Config: defaults, CWD `.env`, `Validate()`. Exactly two remotes. `02-architecture.md`.
3. Sessions: JSON snapshot in memory or Valkey; sliding idle TTL 3h / 30m; `SetUser` rotates ID and CSRF secret; no logout. `subsystems/sessions.md`.
4. Sign-in: OIDC code + PKCE + state + nonce; only `email` kept. `workflows/edupass-sign-in.md`.
5. Page: per-request `{csrfToken, remotes}`; no auth gating; `si` unused; no CSP/frame/HSTS. `subsystems/page-render.md`.
6. API proxy: `/api/posts/*` to Posts backend (`aud=pg`), `/api/student-insights/*` to SI backend (`aud=si`); strips Cookie and backend Set-Cookie; HS256 JWT `{iss: tw, aud, iat, exp}` with 1m TTL; **no session check, no user claim, no CSRF** (Q21, tested as-is); no X-Forwarded or request ID; no per-backend timeout. `workflows/api-proxy.md`.

## Top open questions

Q21 (unauthenticated `/api/`), Q26 (authorisation model), Q30 (how the SPA learns sign-in state), Q29 (security headers), Q31 (no request ID / IP to backends), Q1 (ADR-0001 image). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended order (updated)

| Order | Batch | Files |
| --- | --- | --- |
| **6 (next)** | **Request ID and logging** | `middleware/requestid.go`, `middleware/requestlog.go`, skim their tests |
| 7 | Host shell UI | `HomeView`, `NotFoundView`, `components/*` (skip `ui/`), `hooks`, `helpers`, `App.css` skim |
| 8 | Delivery and ops docs | write `08-infrastructure-and-operations.md`, `09-local-development-and-testing.md` from already-read files |
| 9 | Security consolidation | write `07-security-and-auth.md` from batches 2-5 |
| 10 | Phase 6 consolidation | `05-feature-to-code-map.md`, `10-business-glossary.md`, `12-change-impact-guide.md`, audit |

## Next batch (precise)

**Batch 6: request ID and logging.** Read `server/internal/middleware/requestid.go` and `requestlog.go`; skim `requestid_test.go`, `requestlog_test.go`. Produce `subsystems/observability.md`: request ID source (generated vs trusted inbound header), response header, logger enrichment via `LoggerFromContext`, log line fields, levels, and what is and is not logged (PII, secrets). Confirm the RequestID-then-RequestLog ordering effect and answer the request-ID part of Q31. Small batch: also finishes server source coverage (21/21).

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`, `01`, `02`, `subsystems/*`, `workflows/*` parse with mermaid 11.4.1. Visual layout not reviewed.
