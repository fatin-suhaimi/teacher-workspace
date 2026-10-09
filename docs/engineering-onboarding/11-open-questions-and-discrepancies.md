# 11 Open Questions and Discrepancies

Revision: `main` @ `5ff58a7`. Ordered by engineering impact. Each item says how to resolve it.

## Conflicts with documentation

| # | Topic | Docs say | Code shows | Resolve by |
| --- | --- | --- | --- | --- |
| Q1 | Local-dev image for remote teams | ADR-0001 (Accepted): one image bundling host shell, host backend, datastores and a local OIDC provider | `Dockerfile` builds only `apps/host` + `server`; no mock-edupass or Valkey in the image; `compose.yml` runs Valkey only | Ask maintainers whether ADR-0001 is not yet implemented, or implemented elsewhere (e.g. a separate compose/image repo) |
| Q2 | Release-title guard | ADR-0002: "a PR check on `release/**` branches rejects a malformed title" | `ci.yml` has no title check job; `release.yml` validates the title only after merge. ADR's own Consequences admit the gap | Check GitHub repo settings / rulesets; otherwise a missing CI job |
| Q3 | mock-edupass env var names | `apps/mock-edupass/README.md` says client ID must match `TW_OIDC_CLIENT_ID` and redirect must match `TW_OIDC_REDIRECT_URL` | **Verified:** the server reads only `TW_EDUPASS_CLIENT_ID` / `TW_EDUPASS_REDIRECT_URL` (`config.go:249-250`); no `TW_OIDC_*` tag exists | Stale README. Small docs fix PR if wanted |
| Q21 | `/api/` proxy auth | ADR-0001 hop 3: host backend "authenticates the session, strips the cookie, and signs a short-lived JWT scoped to that one app" | **Verified:** cookie stripped and JWT scoped by `aud` (`proxy.go:61-66, 86-89`), but no session check and no user claim; the test pins the claim set `{iss, aud, iat, exp}` (`proxy_test.go:254-262`) and calls the proxy without a session | Ask maintainers whether auth gating and a user claim are planned before partner backends hold real data. Highest-impact item. See `workflows/api-proxy.md` section 6 |
| Q4 | Repo identity | Module path `github.com/String-sg/teacher-workspace` (`go.mod`) | CHANGELOG links point to `github.com/transformteamsg/teacher-workspace` | Ask which org is canonical (possibly a transfer or mirror) |

## Unknowns to resolve in upcoming batches

| # | Question | Where to look |
| --- | --- | --- |
| ~~Q5~~ | **Resolved:** `Chain` makes the first middleware outermost, so RequestID wraps RequestLog wraps routes (`middleware.go:10-18`, `middleware_test.go:11-45`) |  |
| Q6 | How does `/api/` pick a remote backend (path prefix per remote?), what claims does the signed JWT carry, and which algorithm (HMAC given the 32-byte signing keys)? | `handler/proxy.go`, `proxy_test.go`, `config.go` |
| ~~Q7~~ | **Resolved:** sign-in decodes only the `email` claim and stores `User{Email}`; Edupass `groups`, `sub` and `name` are ignored and no user is rejected for role (`auth.go:235-249`). Follow-up in Q26 |  |
| Q8 | **Resolved (server side):** tokens are minted into the page (`index.go:44`) but `VerifyCSRFToken` has no non-test caller; only `SameSite=Lax` protects unsafe requests. Open: is enforcement planned, and which header will the host use? | maintainers; host fetch code |
| ~~Q9~~ | **Resolved:** no. The server renders the shell for anonymous sessions and `RootLayout` has no guard; the preloaded state has no user flag, so the SPA cannot know whether the user is signed in (`index.go:14-56`, `RootLayout.tsx`). Follow-up: Q30 |  |
| ~~Q10~~ | **Resolved:** the `si` remote is registered when configured (`index.go:30-31`) but no route loads it; `/students/*` renders the local placeholder (`App.tsx:44`). Wiring is presumably pending |  |
| Q11 | Health/readiness endpoint for the Go server: none in the route table. How does the deploy platform probe it? | `handler.go:93-105` (Verified none); ask platform team |
| Q12 | Frontend is never typechecked or built in PR CI, and mock-edupass tests/typecheck are not run in CI (only Docker build compiles the host) | `ci.yml` (Verified); ask whether intentional |
| Q13 | `Dockerfile` downloads `pnpm-linux-arm64` explicitly, consistent with arm64-only publishing; an amd64 build would fail at that step | `Dockerfile` L17-19 (Verified) |
| Q14 | Deployment pipeline (GitLab), environments, and runtime infra (load balancer, Valkey hosting, TLS) live outside this repo | Evidence gap: request access or docs |
| Q15 | Uncommitted local changes in the working copy were not checked (git status not run) | User can run `git status` and share output |
| Q16 | **Partly resolved:** both TTLs are sliding idle timeouts, re-applied on every request (`middleware/session.go:25-31, 88-111`). Open: rationale for 3h anonymous vs 30m signed-in | maintainers |
| Q17 | `parser.go:16` `strings.ReplaceAll(s, "\n", "\n")` is a no-op; probably meant to normalise lone `\r`. Low impact | Code owner; `parser_test.go` |
| Q18 | Set-but-empty env vars override defaults (e.g. `TW_REMOTE_POSTS_MANIFEST_URL=` counts as partially registering Posts and fails startup). Intended? | `dotenv.go:44-52`, `dotenv_test.go:162`, `config.go:450` |
| Q19 | The `private_key_jwt` certificate's expiry is checked only at startup (`config.go:408-410`); a long-running process keeps using an expired cert. Is rotation handled by redeploys? | Ops / maintainers |
| Q20 | Remotes are hard-coded as two named apps (Posts, Student Insights) rather than a list, so each new remote is a code change across config, index and proxy. Planned to generalise? | `config.go:429-536`; batches 4-5 |
| Q22 | No logout route and no absolute session lifetime: an active session never ends except by 30m idle. Is this the agreed policy? | `handler.go:93-105`, `middleware/session.go`; security review |
| Q23 | Every cookieless request stores a new 3h session; memstore never sweeps unread expired entries. Acceptable for probes/bots in production (Valkey memory) and long dev runs? | `middleware/session.go:65`, `memstore.go` |
| Q24 | Concurrent requests on one session are last-write-wins over the whole snapshot (no locking/versioning). Fine while only sign-in mutates; revisit if more state is stored | `middleware/session.go:65-129` |
| Q25 | Callback guard branches have no named tests: no pending login, state mismatch, provider `error` param, ID token verification failure, nonce mismatch, missing email (`auth.go:109-247`). Worth adding before changing sign-in | `auth_test.go` |
| Q26 | Any Edupass user with an email can sign in; mock fixtures (staff-4 role conflict, staff-7 non-TW role, staff-5 `TWSTG`) imply planned role parsing and rejection. What is the intended authorisation model and where should it live? | maintainers; `auth.go:243-249` |
| Q27 | No refresh token use, no stored ID token, no logout (local or Edupass). Users re-authenticate after 30m idle. Intended? | `auth.go`, `workflows/edupass-sign-in.md` A4 |
| Q28 | `email_verified` is not checked; does Edupass guarantee verified emails? | Edupass docs (external) |
| Q29 | No CSP, `X-Frame-Options` / `frame-ancestors`, `Referrer-Policy` or HSTS on the page (`httputil.go:45-54`). Are these set by the load balancer or CDN? | platform team |
| Q30 | How should the SPA learn the user is signed in (to show `/login` or user details)? Options: add a field to `PreloadedState`, or a "me" endpoint. Neither exists | `index.go:14-17`, `preloaded-state.ts` |
| Q31 | Proxy forwards no client IP (`X-Forwarded-*`) and no request ID to remote backends, so backends cannot correlate logs with TW. Intended? | `proxy.go:83-90`; batch 6 for RequestID |
| Q32 | No per-backend timeout: a slow backend is cut only by the server `WriteTimeout` (30s), and uploads are bound by `ReadTimeout` (15s). Are these right for partner APIs (e.g. file uploads)? | `proxy.go:82`, `config.go:52-54` |
| Q33 | Three identifiers per app (`/api/posts`, `aud=pg`, remote `pg`; `/api/student-insights`, `aud=si`, remote `si`). Should the API prefix match the remote name? Worth documenting for partner teams | `proxy.go:28-41`, `index.go:27-31` |
