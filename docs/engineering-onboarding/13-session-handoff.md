# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91`; docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0; batches 1 (startup, config, middleware), 2 (sessions, CSRF), 3 (Edupass sign-in), 4 (page render, static), 5 (API proxy), 6 (request ID, logging). **All 21 server source files inspected.**
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Server source: 21/21 inspected. Server tests: 17/18 reviewed by case name (a few bodies read); `handler_test.go` is 16 bytes.
- Host: bootstrap, routes, preloaded state, `LoginView`, `RootLayout`, `RemoteLoadFallbackView`, `StudentsView` read. Remaining: `HomeView`, `NotFoundView`, `Sidebar`, `AppCard`, `AppSection`, `WelcomeModal`, `ErrorBoundary`, `hooks/use-mobile.ts`, `helpers/cn.ts`, `App.css`, `components.json` (`ui/` is shadcn-generated, low priority).
- mock-edupass: all source read; tests by name.
- See `03-coverage-ledger.md`. API catalog complete for the server (`04-api-catalog.md`).

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, `/` catch-all. Chain: RequestID, RequestLog, then routes.
2. Config: defaults, CWD `.env`, `Validate()`. Exactly two remotes. `02-architecture.md`.
3. Sessions: JSON snapshot in memory or Valkey; sliding idle TTL 3h / 30m; `SetUser` rotates ID and CSRF secret; no logout. `subsystems/sessions.md`.
4. Sign-in: OIDC code + PKCE + state + nonce; only `email` kept. `workflows/edupass-sign-in.md`.
5. Page: per-request `{csrfToken, remotes}`; no auth gating; `si` unused; no CSP/frame/HSTS. `subsystems/page-render.md`.
6. API proxy: strips cookies, HS256 JWT `{iss, aud, iat, exp}`; **no session check, no user claim, no CSRF** (Q21). `workflows/api-proxy.md`.
7. Observability: per-request random `X-Request-ID` (inbound ignored), JSON access log (method, path, status, duration); no metrics, tracing, health endpoint or panic recovery; no DEBUG logs. `subsystems/observability.md`.

## Top open questions

Q21 (unauthenticated `/api/`), Q26 (authorisation model), Q30 (SPA sign-in state), Q29 (security headers), Q31 (request ID / IP to backends, spoofable passthrough), Q34 (panics not JSON-logged), Q1 (ADR-0001 image). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended order (updated)

| Order | Batch | Files |
| --- | --- | --- |
| **7 (next)** | **Host shell UI** | `containers/HomeView.tsx`, `NotFoundView.tsx`; `components/Sidebar.tsx`, `AppCard.tsx`, `AppSection.tsx`, `WelcomeModal.tsx`, `ErrorBoundary.tsx`; `hooks/use-mobile.ts`; `helpers/cn.ts`; skim `App.css`, `components.json` |
| 8 | Delivery and ops docs | write `08-infrastructure-and-operations.md`, `09-local-development-and-testing.md` from already-read files (Dockerfile, compose, CI, release, mise, CONTRIBUTING) |
| 9 | Security consolidation | write `07-security-and-auth.md` from batches 2-6 |
| 10 | Phase 6 consolidation | `05-feature-to-code-map.md`, `10-business-glossary.md`, `12-change-impact-guide.md`, audit pass |

## Next batch (precise)

**Batch 7: host shell UI.** Read the files in row 7. Produce `subsystems/host-shell.md`: route tree, layout and sidebar navigation (which apps link where, internal vs external links), home page app catalogue, welcome modal (first-login logic and storage), error boundary, how remotes mount, styling conventions (`tw:` prefix), and frontend diagrams (route/component tree, boot sequence). Check whether any frontend code calls `/api/` or uses `csrfToken`.

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`, `01`, `02`, `subsystems/*`, `workflows/*` parse with mermaid 11.4.1. Visual layout not reviewed.
