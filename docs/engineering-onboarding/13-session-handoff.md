# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91`; docs live on local branch `docs/engineering-onboarding` (uncommitted at time of writing)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed:** Phase 0 (inventory); Phase 1 batch 1 (startup, configuration, middleware composition), both 2026-10-09
- **Working constraint:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`.

## Coverage highlights

- Fully read: `main.go`, `handler.go`, `config.go`, `dotenv.go`, `parser.go`, `middleware.go` (+ test), host bootstrap/routes/preloaded state, mock-edupass entry/router, all CI/CD, Docker, tooling, ADRs.
- Not yet read: `session/**`, `middleware/{requestid,requestlog,session}.go`, `htmlutil`, `httputil`, `pkg/{random,require}`, `auth.go`, `index.go`, `proxy.go`, most Go tests, host components/containers, mock `provider.ts` / `config.ts`.
- See `03-coverage-ledger.md` (server source 6/21 inspected).

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, `/`. Outer chain is RequestID (outermost), then RequestLog. `handler.go:93-105`, `middleware.go:10-18`.
2. Config = `Default()` then `./.env` (CWD-relative; real env wins; file values exported) then `Validate()`, which joins all errors and also loads Edupass secrets/keys/certs into `cfg.Edupass.ClientCredentials`. `dotenv.go`, `config.go:78-427`.
3. Edupass settings have no defaults, so the server never starts without them, even in dev. `config_test.go:264`.
4. Exactly two remotes (Posts, Student Insights), each needing manifest URL + backend base URL + >=32-byte HS256 key, all or none. `config.go:429-536`.
5. Startup fails fast on: bad config, unreachable Valkey, missing prod `index.html`. Shutdown drains for 30s on SIGINT/SIGTERM. No health endpoint.

## Top open questions

Q1 (ADR-0001 image vs Dockerfile), Q6 (proxy routing and JWT claims), Q7 (session contents, role parsing), Q16 (why default TTL 3h > authenticated TTL 30m; sliding or absolute?), Q18 (empty env vars override defaults). Q3 and Q5 resolved. Full list: `11-open-questions-and-discrepancies.md`.

## Remaining Phase 1 items (deferred, low new reading)

- Environment differences (local vs PR image vs release) beyond `TW_ENV`: needs deployment info outside the repo (Q14).
- C4 level 3 components per subsystem: built up during Phase 2 batches.

## Recommended order (updated)

| Order | Batch | Files |
| --- | --- | --- |
| **2 (next)** | **Sessions and CSRF** | `session/session.go`, `session/csrf.go`, `middleware/session.go`, `session/memstore/memstore.go`, `session/valkeystore/valkeystore.go`, `pkg/random/random.go`; skim `middleware/session_test.go`, `session/session_test.go` |
| 3 | Edupass sign-in | `handler/auth.go`, `auth_test.go`, mock `provider.ts`, `config.ts` |
| 4 | Page render and static | `handler/index.go`, `htmlutil/template.go`, `httputil/*` |
| 5 | API proxy and signed tokens | `handler/proxy.go`, `proxy_test.go`, `pkg/require` |
| 6 | Request ID / logging | `middleware/requestid.go`, `requestlog.go` |
| 7 | Host shell UI | `containers/*`, `components/*` (skip `ui/`) |
| 8 | Delivery and ops docs | write `08-*`, `09-*` from already-read files |

## Next batch (precise)

**PHASE 2, batch 2: sessions and CSRF.** Read the files in row 2 above. Produce `subsystems/sessions.md` (Store contract, session lifecycle and TTLs, cookie attributes, CSRF token issue/verify, Valkey key layout), a session state diagram, and a request sequence through the Session middleware. Resolve Q16 and Q8 (server side), and partly Q7.

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`, `01`, `02` parse with mermaid 11.4.1. Visual layout not reviewed.
