# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (working-copy changes not checked)
- **Objective:** build an evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed:** Phase 0 (2026-10-09): inventory, subsystem map, entry points, provisional system map, ledger, open questions
- **Working constraint:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`.

## Coverage highlights

- Fully read: `main.go`, `handler.go`, host `bootstrap.tsx` / `App.tsx` / `preloaded-state.ts`, mock-edupass `index.ts` / `app.ts`, all CI/CD, Docker, tooling, ADRs.
- Not yet read: all of `config`, `middleware`, `session`, `htmlutil`, `httputil`, `pkg/*`, `auth.go`, `index.go`, `proxy.go`, all Go tests, host components/containers, mock `provider.ts` / `config.ts`.
- See `03-coverage-ledger.md`.

## Key discoveries (evidence-backed)

1. Single Go binary serves everything: `/static/` (no session), and under the Session middleware `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/` (proxy), `/` (page). `handler.go:93-105`.
2. Dev vs prod is a hard switch on `TW_ENV`: dev proxies to the Rsbuild dev server and templates its `index.html`; prod parses `BuildDir/index.html` once at startup and serves files via `os.OpenRoot`. `handler.go:66-85`.
3. Remotes are runtime-configured: the server embeds `{csrfToken, remotes[]}` into `index.html`; the host validates it and calls `registerRemotes` before rendering. `index.html:7-9`, `preloaded-state.ts`, `bootstrap.tsx:11`.
4. Session store is `memory` or `valkey` with no fallback; Valkey TLS is toggled by `?tls=true` on the URL. `main.go:56-89`.
5. Releases are release-PR driven (`release: vX.Y.Z (#N)` on `main`) to ECR, arm64 only; deploy happens in an external GitLab pipeline. `release.yml`, ADR-0002.

## Blockers / gaps

- Deployment infra and GitLab pipeline are outside the repo (Q14).
- Remote apps (`pg`, `si`) and their backends are in other repos (out of scope; interfaces only).

## Top open questions

Q1 (ADR-0001 image not reflected in Dockerfile), Q3 (stale env var names in mock-edupass README), Q6 (proxy routing and JWT claims), Q7 (session contents and role parsing). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended analysis order

| Order | Batch | Phase | Files | Why first |
| --- | --- | --- | --- | --- |
| 1 | Startup and configuration | 1 | `config/config.go` (+ skim `config_test.go`), `pkg/dotenv/*`, `middleware/middleware.go` | Every other subsystem reads `Config`; resolves Q3, Q5 |
| 2 | Sessions and CSRF | 2 | `session/session.go`, `session/csrf.go`, `middleware/session.go`, `memstore`, `valkeystore`, `pkg/random` | Central to auth and proxy; resolves Q7 (part), Q8 |
| 3 | Edupass sign-in | 2/3 | `handler/auth.go`, `auth_test.go`, mock `provider.ts`, `config.ts` | Business-critical login flow; Q7, Q9 |
| 4 | Page render and static | 2/3 | `handler/index.go`, `htmlutil/template.go`, `httputil/*` | How preloaded state and remotes reach the browser; Q10 |
| 5 | API proxy and signed tokens | 2/3 | `handler/proxy.go`, `proxy_test.go`, `pkg/require` | Contract with partner teams; Q6 |
| 6 | Request ID / logging | 2 | `middleware/requestid.go`, `requestlog.go` | Small; observability |
| 7 | Host shell UI | 2/5 | `containers/*`, `components/*` (skip `ui/`) | Navigation, login view, welcome modal |
| 8 | Delivery and ops | 5 | Already read; write `08-*` and `09-*` | Low new reading |

## Next batch (precise)

**PHASE 1, batch 1: startup and configuration.** Read `server/internal/config/config.go`, `server/pkg/dotenv/dotenv.go`, `server/pkg/dotenv/parser.go`, `server/internal/middleware/middleware.go`; skim `config_test.go` for validation rules. Produce `02-architecture.md` (C4 context/container, startup diagram), config reference table, and resolve Q3 and Q5.

Suggested command: `PHASE 1`

## Diagram validation

All 3 Mermaid blocks (`00` x2, `01` x1) parse cleanly with mermaid 11.4.1. Visual layout not reviewed.
