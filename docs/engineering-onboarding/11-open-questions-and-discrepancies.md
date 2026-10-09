# 11 Open Questions and Discrepancies

Revision: `main` @ `5ff58a7`. Ordered by engineering impact. Each item says how to resolve it. Audited in batch 12 (2026-10-09).

## Shortlist for maintainers

The questions most worth asking, in priority order. Answers would resolve the largest documented gaps.

1. **Q21 / Q8 / Q38:** Is requiring sign-in on `/api/`, adding a user claim to the backend JWT, and enforcing CSRF planned before partner backends hold real data? How should remotes obtain the CSRF token?
2. **Q26:** What is the intended authorisation model from Edupass `groups` (roles, attributes, locations, `TW` vs `TWSTG`)?
3. **Q22 / Q27:** Is "no logout, 30m idle timeout, no absolute lifetime" the agreed session policy (shared school devices)?
4. **Q29 / Q11 / Q35:** Does the load balancer or CDN add security headers, health probes and monitoring, given the app has none?
5. **Q1:** Is the ADR-0001 developer image (host, backend, datastores, local IdP) still planned, or built elsewhere?
6. **Q40 / Q12:** Does the CI Format job actually fail on unformatted code? Is skipping host typecheck and mock tests in CI intentional?
7. **Q41 / Q42 / Q3:** Happy to receive small docs PRs for `CONTRIBUTING.md` (third process, full remote env vars) and the mock-edupass README variable names?
8. **Q4:** Which GitHub org is canonical, `String-sg` or `transformteamsg`?

## Conflicts with documentation

| # | Topic | Docs say | Code shows | Resolve by |
| --- | --- | --- | --- | --- |
| Q1 | Local-dev image for remote teams | ADR-0001 (Accepted): one image bundling host shell, host backend, datastores and a local OIDC provider | `Dockerfile` builds only `apps/host` + `server`; no mock-edupass or Valkey in the image; `compose.yml` runs Valkey only | Ask maintainers whether ADR-0001 is not yet implemented, or implemented elsewhere (e.g. a separate compose/image repo) |
| Q2 | Release-title guard | ADR-0002: "a PR check on `release/**` branches rejects a malformed title" | `ci.yml` has no title check job; `release.yml` validates the title only after merge. ADR's own Consequences admit the gap | Check GitHub repo settings / rulesets; otherwise a missing CI job |
| Q3 | mock-edupass env var names | `apps/mock-edupass/README.md` says client ID must match `TW_OIDC_CLIENT_ID` and redirect must match `TW_OIDC_REDIRECT_URL` | **Verified:** the server reads only `TW_EDUPASS_CLIENT_ID` / `TW_EDUPASS_REDIRECT_URL` (`config.go:249-250`); no `TW_OIDC_*` tag exists | Stale README. Small docs fix PR if wanted |
| Q21 | `/api/` proxy auth | ADR-0001 hop 3: host backend "authenticates the session, strips the cookie, and signs a short-lived JWT scoped to that one app" | **Verified:** cookie stripped and JWT scoped by `aud` (`proxy.go:61-66, 86-89`), but no session check and no user claim; the test pins the claim set `{iss, aud, iat, exp}` (`proxy_test.go:254-262`) and calls the proxy without a session | Ask maintainers whether auth gating and a user claim are planned before partner backends hold real data. Highest-impact item. See `workflows/api-proxy.md` section 6 |
| Q42 | Running against a local remote | `CONTRIBUTING.md` "Running against a local remote": `TW_REMOTE_POSTS_MANIFEST_URL=... go run ./server/cmd/tw` | **Verified:** config validation requires the manifest URL, backend base URL and signing key together, or none (`config.go:450-485`); with `.env.example` (which leaves the others commented out) that command should fail at startup | Docs fix: show all three variables (see `09-local-development-and-testing.md` section 3) |
| Q41 | Running locally | `README.md` and `CONTRIBUTING.md` "Running locally" list two processes (host dev server, Go server) | Signing in needs mock-edupass on `:9000` as well (`.env.example` Edupass URLs); it is documented only in `apps/mock-edupass/README.md` | Docs fix: add the third terminal |
| Q4 | Repo identity | Module path `github.com/String-sg/teacher-workspace` (`go.mod`) | CHANGELOG links point to `github.com/transformteamsg/teacher-workspace` | Ask which org is canonical (possibly a transfer or mirror) |

## Open questions

Struck-through items are resolved; the resolution stays for traceability.

| # | Question | Where to look |
| --- | --- | --- |
| ~~Q5~~ | **Resolved:** `Chain` makes the first middleware outermost, so RequestID wraps RequestLog wraps routes (`middleware.go:10-18`, `middleware_test.go:11-45`) |  |
| ~~Q6~~ | **Resolved (batch 5):** `/api/<app>/` picks the backend by the first path segment (`posts`, `student-insights`); the JWT is HS256 with exactly `iss`, `aud`, `iat`, `exp` (`proxy.go:51-67`; `workflows/api-proxy.md`) |  |
| ~~Q7~~ | **Resolved:** sign-in decodes only the `email` claim and stores `User{Email}`; Edupass `groups`, `sub` and `name` are ignored and no user is rejected for role (`auth.go:235-249`). Follow-up in Q26 |  |
| Q8 | **Resolved (server side):** tokens are minted into the page (`index.go:44`) but `VerifyCSRFToken` has no non-test caller; only `SameSite=Lax` protects unsafe requests. Open: is enforcement planned, and which header will the host use? | maintainers; host fetch code |
| ~~Q9~~ | **Resolved:** no. The server renders the shell for anonymous sessions and `RootLayout` has no guard; the preloaded state has no user flag, so the SPA cannot know whether the user is signed in (`index.go:14-56`, `RootLayout.tsx`). Follow-up: Q30 |  |
| ~~Q10~~ | **Resolved:** the `si` remote is registered when configured (`index.go:30-31`) but no route loads it; `/students/*` renders the local placeholder (`App.tsx:44`). Wiring is presumably pending |  |
| Q11 | Health/readiness endpoint for the Go server: none in the route table. How does the deploy platform probe it? | `handler.go:93-105` (Verified none); ask platform team |
| Q12 | PR CI has no host typecheck or standalone host build (the host is compiled only inside the image job, skipped for forks), and mock-edupass tests and typecheck never run in CI. Intentional? | `ci.yml` (Verified); maintainers |
| Q13 | `Dockerfile` downloads `pnpm-linux-arm64` explicitly, consistent with arm64-only publishing; an amd64 build would fail at that step | `Dockerfile` L17-19 (Verified) |
| Q14 | Deployment pipeline (GitLab), environments, and runtime infra (load balancer, Valkey hosting, TLS) live outside this repo | Evidence gap: request access or docs |
| Q15 | Uncommitted local changes in the working copy were not checked (git status not run). Source file timestamps were unchanged across all batches | user can run `git status` |
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
| Q31 | Proxy forwards no client IP (`X-Forwarded-*`) and not TW's request ID; worse, a client-supplied `X-Request-ID` is passed through unchanged, so a backend may log a spoofable value. TW also ignores inbound request IDs from the load balancer (`requestid.go:22`, `proxy.go:83-90`). Intended? | maintainers |
| Q32 | No per-backend timeout: a slow backend is cut only by the server `WriteTimeout` (30s), and uploads are bound by `ReadTimeout` (15s). Are these right for partner APIs (e.g. file uploads)? | `proxy.go:82`, `config.go:52-54` |
| Q33 | Three identifiers per app (`/api/posts`, `aud=pg`, remote `pg`; `/api/student-insights`, `aud=si`, remote `si`). Should the API prefix match the remote name? Worth documenting for partner teams | `proxy.go:28-41`, `index.go:27-31` |
| Q34 | No panic-recovery middleware and no `http.Server.ErrorLog`: a handler panic is logged by `net/http` as plain text on stderr (not JSON) with no access-log line or request ID. Acceptable for log ingestion? | `main.go:105-116`; `subsystems/observability.md` |
| Q35 | No metrics or tracing. Is platform-level monitoring (load balancer metrics, log-based alerts) the intended approach? | platform team |
| Q36 | The host has no top-level error boundary and no error reporting: a render error outside the remote routes blanks the app, and remote errors are swallowed by `ErrorBoundary` with no `componentDidCatch`. Is client error monitoring planned? | `App.tsx`, `ErrorBoundary.tsx` |
| Q37 | The home page app catalogue (18 external and internal links) is hard-coded in `HomeView.tsx`. Will it move to server config or a CMS, and should it vary by role or school? | `HomeView.tsx:24-202` |
| Q38 | How will remotes get the CSRF token (and later user info)? Nothing is passed as props or shared modules; a remote would have to read `#preloaded-state` from the DOM. Needs a documented contract before Q8 enforcement | `rsbuild.config.ts:13-26`, `App.tsx:16-32` |
| Q39 | Dockerfile downloads the pnpm tarball with `wget` and no checksum check, and base images are pinned by tag rather than digest, while local tools are checksum-verified by `mise.lock`. Worth tightening? | `Dockerfile:6, 17-19, 37, 55` |
| Q40 | CI "Format" job runs `pnpm format`, which rewrites files (no `--check`). If oxfmt exits 0 after rewriting, unformatted PRs pass. Confirm with a deliberately unformatted PR, or switch to a check mode | `package.json` `format`, `ci.yml:36-37` |
| Q43 | lefthook covers only JS/TS/MD/HTML/CSS/JSON/YAML/TOML; Go formatting is not hooked, only checked by `golangci-lint` in CI | `lefthook.yml`, `.golangci.yaml` |
| Q44 | `mise.lock` records verification only for `macos-arm64`; developers on Linux or Intel Macs install tools without the lockfile check | `mise.toml` `lockfile_platforms` |
