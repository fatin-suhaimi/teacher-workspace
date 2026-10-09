# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91`; docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0 (inventory); Phase 1 batch 1 (startup, config, middleware composition); Phase 2 batch 2 (sessions and CSRF)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Fully read: `main.go`, `handler.go`, `config.go`, `dotenv/*`, `middleware.go`, `middleware/session.go`, `session/session.go`, `session/csrf.go`, `memstore.go`, `valkeystore.go`, `random.go`; `proxy.go` L25-78; host bootstrap/routes/preloaded state; mock-edupass entry/router; all CI/CD, Docker, tooling, ADRs.
- Not yet read: `auth.go` (call sites only), `index.go` (call sites only), rest of `proxy.go`, `htmlutil`, `httputil`, `requestid.go`, `requestlog.go`, `pkg/require`, most test bodies, host components/containers, mock `provider.ts` / `config.ts`.
- See `03-coverage-ledger.md` (server source 12/21 inspected, 1 partial).

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, `/`. RequestID outermost, then RequestLog. `handler.go:93-105`, `middleware.go:10-18`.
2. Config: defaults, then CWD `.env` (real env wins), then `Validate()` which joins all errors and loads Edupass credentials. Edupass has no defaults. Exactly two remotes, all-or-none config. `config.go`.
3. Sessions: JSON snapshot `{id, csrf_token, user{email}, data}` in memory or Valkey (`session:<id>`); saved on every request before first write; sliding idle TTL 3h anonymous / 30m signed in; `SetUser` rotates ID and CSRF secret and clears data; no logout, no absolute lifetime. `subsystems/sessions.md`.
4. **CSRF tokens are minted into the page but never verified** in server code (Q8).
5. **`/api/` proxy forwards any caller with a signed HS256 JWT (`iss=tw`, `aud=pg|si`, `iat`, `exp`), with no session check and no user claim** (`proxy.go:25-78`). Conflicts with ADR-0001 (Q21). Highest-impact finding so far.

## Top open questions

Q21 (unauthenticated `/api/` proxy), Q1 (ADR-0001 image vs Dockerfile), Q7 (does sign-in use Edupass roles at all?), Q22 (no logout / absolute lifetime), Q16 (TTL rationale). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended order (updated)

| Order | Batch | Files |
| --- | --- | --- |
| **3 (next)** | **Edupass sign-in** | `handler/auth.go`, skim `handler/auth_test.go`; `apps/mock-edupass/src/provider.ts`, `src/config.ts`; skim `test/api.test.ts` |
| 4 | Page render and static | `handler/index.go`, `htmlutil/template.go`, `httputil/*` |
| 5 | API proxy and signed tokens | rest of `handler/proxy.go`, `proxy_test.go`, `pkg/require` |
| 6 | Request ID / logging | `middleware/requestid.go`, `requestlog.go` |
| 7 | Host shell UI | `containers/*`, `components/*` (skip `ui/`) |
| 8 | Delivery and ops docs | write `08-*`, `09-*` from already-read files |
| 9 | Security consolidation | write `07-security-and-auth.md` from batches 2, 3, 5 |

## Next batch (precise)

**PHASE 2/3, batch 3: Edupass sign-in.** Read `server/internal/handler/auth.go` in full and skim `auth_test.go`; read mock-edupass `provider.ts` and `config.ts`. Produce `workflows/edupass-sign-in.md` with a sequence diagram (browser, TW, Edupass) covering state/nonce/PKCE, token exchange (`client_secret_post` and `private_key_jwt`), ID token verification, claims to `User`, `return_to` safety, and failure paths. Resolve Q7 and Q9; start `04-api-catalog.md` rows for the two auth routes.

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`, `01`, `02`, `subsystems/sessions.md` parse with mermaid 11.4.1. Visual layout not reviewed.
