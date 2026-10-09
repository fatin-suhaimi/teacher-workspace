# 11 Open Questions and Discrepancies

Revision: `main` @ `5ff58a7`. Ordered by engineering impact. Each item says how to resolve it.

## Conflicts with documentation

| # | Topic | Docs say | Code shows | Resolve by |
| --- | --- | --- | --- | --- |
| Q1 | Local-dev image for remote teams | ADR-0001 (Accepted): one image bundling host shell, host backend, datastores and a local OIDC provider | `Dockerfile` builds only `apps/host` + `server`; no mock-edupass or Valkey in the image; `compose.yml` runs Valkey only | Ask maintainers whether ADR-0001 is not yet implemented, or implemented elsewhere (e.g. a separate compose/image repo) |
| Q2 | Release-title guard | ADR-0002: "a PR check on `release/**` branches rejects a malformed title" | `ci.yml` has no title check job; `release.yml` validates the title only after merge. ADR's own Consequences admit the gap | Check GitHub repo settings / rulesets; otherwise a missing CI job |
| Q3 | mock-edupass env var names | `apps/mock-edupass/README.md` says client ID must match `TW_OIDC_CLIENT_ID` and redirect must match `TW_OIDC_REDIRECT_URL` | Server keys are `TW_EDUPASS_CLIENT_ID` / `TW_EDUPASS_REDIRECT_URL` (`.env.example`) | Confirm in `config.go` during Phase 1; likely a stale README after a rename |
| Q4 | Repo identity | Module path `github.com/String-sg/teacher-workspace` (`go.mod`) | CHANGELOG links point to `github.com/transformteamsg/teacher-workspace` | Ask which org is canonical (possibly a transfer or mirror) |

## Unknowns to resolve in upcoming batches

| # | Question | Where to look |
| --- | --- | --- |
| Q5 | Execution order of `middleware.Chain(h, RequestID, RequestLog)`: does RequestID run before RequestLog? | `server/internal/middleware/middleware.go` |
| Q6 | How does `/api/` pick a remote backend (path prefix per remote?), what claims does the signed JWT carry, and which algorithm (HMAC given the 32-byte signing keys)? | `handler/proxy.go`, `proxy_test.go`, `config.go` |
| Q7 | What does the session hold (user identity, Edupass claims, roles/groups like `0001_TW_ROLE_TEACHER`)? How are role groups parsed and conflicts (staff-4 fixture) handled? | `session/session.go`, `handler/auth.go`, `auth_test.go`, mock `provider.ts` |
| Q8 | Is CSRF enforced on `/api/` mutating requests, and how does the host send the token? | `session/csrf.go`, `middleware/session.go`, host fetch code (none seen yet) |
| Q9 | Is `/login` (frontend) and an unauthenticated redirect enforced server-side, or only in the SPA? | `handler/index.go`, `containers/LoginView.tsx`, `RootLayout.tsx` |
| Q10 | Route `/students/*` renders a local placeholder, while config defines a Student Insights (`si`) remote. Is wiring pending? | `App.tsx`, `index.go` remote list |
| Q11 | Health/readiness endpoint for the Go server: none in the route table. How does the deploy platform probe it? | `handler.go:93-105` (Verified none); ask platform team |
| Q12 | Frontend is never typechecked or built in PR CI, and mock-edupass tests/typecheck are not run in CI (only Docker build compiles the host) | `ci.yml` (Verified); ask whether intentional |
| Q13 | `Dockerfile` downloads `pnpm-linux-arm64` explicitly, consistent with arm64-only publishing; an amd64 build would fail at that step | `Dockerfile` L17-19 (Verified) |
| Q14 | Deployment pipeline (GitLab), environments, and runtime infra (load balancer, Valkey hosting, TLS) live outside this repo | Evidence gap: request access or docs |
| Q15 | Uncommitted local changes in the working copy were not checked (git status not run) | User can run `git status` and share output |
