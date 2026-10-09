# 11 Open Questions and Discrepancies

Revision: `main` @ `5ff58a7`. Ordered by engineering impact. Each item says how to resolve it.

## Conflicts with documentation

| # | Topic | Docs say | Code shows | Resolve by |
| --- | --- | --- | --- | --- |
| Q1 | Local-dev image for remote teams | ADR-0001 (Accepted): one image bundling host shell, host backend, datastores and a local OIDC provider | `Dockerfile` builds only `apps/host` + `server`; no mock-edupass or Valkey in the image; `compose.yml` runs Valkey only | Ask maintainers whether ADR-0001 is not yet implemented, or implemented elsewhere (e.g. a separate compose/image repo) |
| Q2 | Release-title guard | ADR-0002: "a PR check on `release/**` branches rejects a malformed title" | `ci.yml` has no title check job; `release.yml` validates the title only after merge. ADR's own Consequences admit the gap | Check GitHub repo settings / rulesets; otherwise a missing CI job |
| Q3 | mock-edupass env var names | `apps/mock-edupass/README.md` says client ID must match `TW_OIDC_CLIENT_ID` and redirect must match `TW_OIDC_REDIRECT_URL` | **Verified:** the server reads only `TW_EDUPASS_CLIENT_ID` / `TW_EDUPASS_REDIRECT_URL` (`config.go:249-250`); no `TW_OIDC_*` tag exists | Stale README. Small docs fix PR if wanted |
| Q4 | Repo identity | Module path `github.com/String-sg/teacher-workspace` (`go.mod`) | CHANGELOG links point to `github.com/transformteamsg/teacher-workspace` | Ask which org is canonical (possibly a transfer or mirror) |

## Unknowns to resolve in upcoming batches

| # | Question | Where to look |
| --- | --- | --- |
| ~~Q5~~ | **Resolved:** `Chain` makes the first middleware outermost, so RequestID wraps RequestLog wraps routes (`middleware.go:10-18`, `middleware_test.go:11-45`) |  |
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
| Q16 | Default `TW_SESSION_DEFAULT_TTL` (3h) is longer than `TW_SESSION_AUTHENTICATED_TTL` (30m). Is the authenticated TTL sliding (refreshed per request) or absolute? Why does a pre-login session outlive a signed-in one? | `middleware/session.go` (batch 2); ask maintainers for rationale |
| Q17 | `parser.go:16` `strings.ReplaceAll(s, "\n", "\n")` is a no-op; probably meant to normalise lone `\r`. Low impact | Code owner; `parser_test.go` |
| Q18 | Set-but-empty env vars override defaults (e.g. `TW_REMOTE_POSTS_MANIFEST_URL=` counts as partially registering Posts and fails startup). Intended? | `dotenv.go:44-52`, `dotenv_test.go:162`, `config.go:450` |
| Q19 | The `private_key_jwt` certificate's expiry is checked only at startup (`config.go:408-410`); a long-running process keeps using an expired cert. Is rotation handled by redeploys? | Ops / maintainers |
| Q20 | Remotes are hard-coded as two named apps (Posts, Student Insights) rather than a list, so each new remote is a code change across config, index and proxy. Planned to generalise? | `config.go:429-536`; batches 4-5 |
