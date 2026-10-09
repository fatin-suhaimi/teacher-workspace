# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91`; docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0; batch 1 (startup, config, middleware); batch 2 (sessions, CSRF); batch 3 (Edupass sign-in); batch 4 (page render, static, response helpers)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Server source: 17/21 inspected, `proxy.go` partial (L25-78). Remaining: rest of `proxy.go`, `requestid.go`, `requestlog.go`, `pkg/require`.
- Host: bootstrap, routes, preloaded state, `LoginView`, `RootLayout`, `RemoteLoadFallbackView` read. Remaining: `HomeView`, `NotFoundView`, components (`Sidebar`, `AppCard`, `AppSection`, `WelcomeModal`, `ErrorBoundary`), hooks/helpers.
- mock-edupass: all source read; tests only by name.
- See `03-coverage-ledger.md`.

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, and `/` catch-all for any method and path. RequestID outermost, then RequestLog. `handler.go:93-105`.
2. Config: defaults, CWD `.env` (real env wins), `Validate()` joins errors and loads Edupass credentials. Exactly two remotes. `config.go`.
3. Sessions: JSON snapshot in memory or Valkey; saved every request; sliding idle TTL 3h / 30m; `SetUser` rotates ID and CSRF secret; no logout. `subsystems/sessions.md`.
4. Sign-in: OIDC code + PKCE + state + nonce; `client_secret_post` or `private_key_jwt`; only `email` kept; groups ignored. `workflows/edupass-sign-in.md`.
5. Page: per-request preloaded state `{csrfToken, remotes[{name: pg|si, entry}]}` rendered into `index.html` (dev: fetched from Rsbuild each request; prod: parsed once). No auth gating of pages; SPA has no signed-in flag. `si` registered but unused. No CSP/frame/HSTS headers. `subsystems/page-render.md`.
6. **`/api/` proxy forwards any caller with an HS256 JWT, no session check, no user claim** (Q21). **CSRF tokens never verified** (Q8).

## Top open questions

Q21 (unauthenticated `/api/`), Q26 (authorisation model), Q30 (how the SPA learns sign-in state), Q29 (security headers), Q25 (untested callback guards), Q1 (ADR-0001 image). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended order (updated)

| Order | Batch | Files |
| --- | --- | --- |
| **5 (next)** | **API proxy and signed tokens** | `handler/proxy.go` (full), `proxy_test.go`, `pkg/require/require.go` |
| 6 | Request ID / logging | `middleware/requestid.go`, `requestlog.go` (+ tests) |
| 7 | Host shell UI | `HomeView`, `NotFoundView`, `components/*` (skip `ui/`), `hooks`, `helpers` |
| 8 | Delivery and ops docs | write `08-*`, `09-*` from already-read files |
| 9 | Security consolidation | write `07-security-and-auth.md` from batches 2, 3, 4, 5 |

## Next batch (precise)

**Batch 5: API proxy and signed tokens.** Read `server/internal/handler/proxy.go` in full (especially `newRemoteBackendProxy`: header rewriting, cookie stripping, `Authorization` header, error handling), skim `proxy_test.go`, and read `pkg/require/require.go`. Produce `workflows/api-proxy.md` with a sequence diagram (browser, TW, remote backend), the JWT contract for partner teams, path mapping examples, and failure paths. Finish the `/api/` row in `04-api-catalog.md` and update Q21 (cookie stripping).

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`, `01`, `02`, `subsystems/*`, `workflows/*` parse with mermaid 11.4.1. Visual layout not reviewed.
