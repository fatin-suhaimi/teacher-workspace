# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91`; docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0 (inventory); batch 1 (startup, config, middleware composition); batch 2 (sessions and CSRF); batch 3 (Edupass sign-in)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Fully read: `main.go`, `handler.go`, `config.go`, `dotenv/*`, `middleware.go`, `middleware/session.go`, `session/*.go`, both stores, `random.go`, `auth.go`; `proxy.go` L25-78; host bootstrap/routes/preloaded state/`LoginView`; mock-edupass `index.ts`, `app.ts`, `provider.ts`, `config.ts`; all CI/CD, Docker, tooling, ADRs.
- Not yet read: `index.go` (call sites only), rest of `proxy.go`, `htmlutil`, `httputil`, `requestid.go`, `requestlog.go`, `pkg/require`, most test bodies, other host containers/components.
- See `03-coverage-ledger.md` (server source 13/21 inspected, 1 partial).

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, `/`. RequestID outermost, then RequestLog. `handler.go:93-105`.
2. Config: defaults, then CWD `.env` (real env wins), then `Validate()` which joins errors and loads Edupass credentials. Exactly two remotes. `config.go`.
3. Sessions: JSON snapshot in memory or Valkey; saved every request; sliding idle TTL 3h anonymous / 30m signed in; `SetUser` rotates ID and CSRF secret; no logout. `subsystems/sessions.md`.
4. Sign-in: OIDC code flow + PKCE S256 + state + nonce; `client_secret_post` or `private_key_jwt` (PS256, `x5t#S256`); only `email` kept; groups ignored, so any Edupass user with an email can sign in. Failures redirect to `/login?error=oauth2_callback_failed&return_to=...`. `workflows/edupass-sign-in.md`.
5. **`/api/` proxy forwards any caller with an HS256 JWT (`iss=tw`, `aud=pg|si`, `iat`, `exp`), no session check, no user claim** (Q21). **CSRF tokens minted but never verified** (Q8).

## Top open questions

Q21 (unauthenticated `/api/` proxy), Q26 (authorisation model; groups ignored), Q25 (untested callback guards), Q22/Q27 (no logout, no refresh), Q1 (ADR-0001 image). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended order (updated)

| Order | Batch | Files |
| --- | --- | --- |
| **4 (next)** | **Page render and static** | `handler/index.go`, `htmlutil/template.go`, `httputil/httputil.go`, `httputil/error.go`; skim `index_test.go`, `template_test.go`, `httputil_test.go` |
| 5 | API proxy and signed tokens | rest of `handler/proxy.go`, `proxy_test.go`, `pkg/require` |
| 6 | Request ID / logging | `middleware/requestid.go`, `requestlog.go` |
| 7 | Host shell UI | `containers/*`, `components/*` (skip `ui/`) |
| 8 | Delivery and ops docs | write `08-*`, `09-*` from already-read files |
| 9 | Security consolidation | write `07-security-and-auth.md` from batches 2, 3, 5 |

## Next batch (precise)

**Batch 4: page render and static.** Read the files in row 4 above. Produce `subsystems/page-render.md` covering dev vs prod templating, the preloaded-state JSON (`csrfToken`, `remotes` from config), escaping, the `/static/` handler, and the shared response helpers (`RenderPlain`, `RenderJSON`, `Redirect`). Resolve Q9 (is `/login` or any redirect enforced server-side) and Q10 (Student Insights remote vs local `/students` placeholder). Fill the `/` and `/static/*` rows of `04-api-catalog.md`.

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`, `01`, `02`, `subsystems/sessions.md`, `workflows/edupass-sign-in.md` parse with mermaid 11.4.1. Visual layout not reviewed.
