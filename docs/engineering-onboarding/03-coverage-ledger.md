# 03 Coverage Ledger

Revision: `main` @ `5ff58a7`. Last updated: batch 10, data model and glossary (2026-10-09; no new files read, structures spot-checked). All server source files and all hand-written host source files are now inspected; only shadcn-generated `components/ui/*` remain. Statuses: `inspected`, `partially inspected`, `identified`, `excluded (reason)`, `inaccessible`. "Inspected" means read in full at this revision, not that every call has been traced.

## Summary

| Area | Files | Inspected | Partially | Identified only | Excluded |
| --- | --- | --- | --- | --- | --- |
| Server Go source (non-test) | 21 | 21 | 0 | 0 | 0 |
| Server Go tests | 18 | 1 | 16 | 1 | 0 |
| Host app source + config (excl. assets, dist) | 33 | 24 | 0 | 9 | 0 |
| mock-edupass source + tests + config | 10 | 7 | 2 | 1 | 0 |
| Build, CI/CD, tooling, root config | 22 | 18 | 0 | 0 | 4 (lockfiles, `.env`) |
| Repo docs | 7 | 7 | 0 | 0 | 0 |

## Server (Go)

| Path | Subsystem | Status | Key symbols / notes | Docs |
| --- | --- | --- | --- | --- |
| `server/cmd/tw/main.go` | S1 | inspected | `main`, `shutdownTimeout`; store switch L56-89; server L104-116; shutdown L118-138 | 00, 01, 02 |
| `server/internal/config/config.go` | S2 | inspected | `Config`, `Default`, `Validate`, per-section `validate`, `EdupassClientCredentials`, `IsPostsRegistered`, `IsStudentInsightsRegistered` | 02 |
| `server/internal/config/config_test.go` | S2 | partially inspected | Test/case names all listed; `TestDefault` bodies read; other bodies sampled | 02 |
| `server/pkg/dotenv/dotenv.go` | S2 | inspected | `Load`, `decode`, `environToMap`, `stringToURLFunc`, `stringToLogLevelFunc` | 02 |
| `server/pkg/dotenv/parser.go` | S2 | inspected | `parse`, `lineRE`; no-op replace at L16 (Q17) | 02 |
| `server/pkg/dotenv/dotenv_test.go`, `parser_test.go` | S2 | partially inspected | `TestLoad` read; `TestDecode` URL and override cases read; `TestParse` not read | 02 |
| `server/internal/handler/handler.go` | S3 | inspected | `Handler`, `New`, `Routes`, `newDevServerProxy` | 00, 01 |
| `server/internal/handler/auth.go` | S3 | inspected | `authEdupass`, `authEdupassCallback`, `safeReturnTo`, `loginFailedURL`, session keys | workflows/edupass-sign-in |
| `server/internal/handler/auth_test.go` | S3 | partially inspected | all case names reviewed; some bodies sampled | workflows/edupass-sign-in |
| `server/internal/handler/index.go` | S3 | inspected | `PreloadedState`, `Remote`, `index()`, `static` | subsystems/page-render |
| `server/internal/handler/index_test.go` | S3 | partially inspected | case names reviewed; CSRF embedding case read | subsystems/page-render |
| `server/internal/handler/proxy.go` | S3 | inspected | `proxy()`, `remoteBackend`, `newRemoteBackendProxy` (Rewrite, ModifyResponse, ErrorHandler) | workflows/api-proxy |
| `server/internal/handler/proxy_test.go` | S3 | partially inspected | case names reviewed; JWT claims test read | workflows/api-proxy |
| `server/internal/handler/handler_test.go` | S3 | identified | 16 bytes (package clause only, presumably) |  |
| `server/internal/middleware/middleware.go` | S4 | inspected | `Middleware`, `Chain` (first = outermost) | 02 |
| `server/internal/middleware/requestid.go` | S4 | inspected | `RequestID`, `RequestIDFromContext`, `LoggerFromContext`, `WithLogger`, `X-Request-ID` | subsystems/observability |
| `server/internal/middleware/requestlog.go` | S4 | inspected | `RequestLog`, `requestLogResponseWriter` | subsystems/observability |
| `server/internal/middleware/session.go` | S4/S5 | inspected | `Session`, `SessionOptions`, `SessionFromContext`, `WithSession`, `sessionResponseWriter` | sessions |
| `server/internal/middleware/middleware_test.go` | S4 | inspected | `TestChain` | 02 |
| `server/internal/middleware/session_test.go` | S4/S5 | partially inspected | 40 subtest names reviewed | sessions |
| `server/internal/middleware/{requestid,requestlog}_test.go` | S4 | partially inspected | case names reviewed | subsystems/observability |
| `server/internal/session/session.go` | S5 | inspected | `Store`, `Session`, `snapshot`, `User`, `New`, `Load`, `Save`, `SetUser`, CSRF accessors, data accessors | sessions |
| `server/internal/session/csrf.go` | S5 | inspected | `mask`, `unmask`, `maskToken`, `verifyToken` | sessions |
| `server/internal/session/memstore/memstore.go` | S5 | inspected | `Store`, `New`, `WithClock`, `Prepare`/`Commit`/`Drop` | sessions |
| `server/internal/session/valkeystore/valkeystore.go` | S5 | inspected | `Store`, `New`, `WithPrefix`, GET / SET EX / DEL | sessions |
| `server/internal/session/**/_test.go` (4 files) | S5 | partially inspected | all subtest names reviewed; valkeystore tests use testcontainers | sessions |
| `server/internal/htmlutil/template.go` | S6 | inspected | `Template`, `URLTemplate`, `FileTemplate`, `fetchTimeout` | subsystems/page-render |
| `server/internal/htmlutil/template_test.go` | S6 | partially inspected | case names reviewed | subsystems/page-render |
| `server/internal/httputil/httputil.go`, `error.go` | S6 | inspected | `RenderPlain`, `RenderHTML`, `RenderJSON`, `Redirect`, `ErrorResponse`, header and MIME constants | subsystems/page-render |
| `server/internal/httputil/httputil_test.go` | S6 | partially inspected | case names reviewed | subsystems/page-render |
| `server/pkg/random/random.go` | S6 | inspected | `Alphanumeric` (crypto/rand, rejection sampling), `Base62`, `Base58` | sessions |
| `server/pkg/random/random_test.go` | S6 | partially inspected | subtest names reviewed | sessions |
| `server/pkg/require/require.go` | S6 | inspected | test-only assertion helpers (`Equal`, `NotEqual`, `True`, `False`, `NoError`, `HasError`) | workflows/api-proxy |

## Host frontend (`apps/host`)

| Path | Status | Notes |
| --- | --- | --- | --- |
| `package.json`, `rsbuild.config.ts`, `tsconfig.json`, `index.html` | inspected | MF host `teacher_workspace`, `remotes: {}`, shared singletons; path alias `~/*`; `@mf-types` path |
| `src/index.ts`, `src/bootstrap.tsx`, `src/App.tsx` | inspected | entry, `registerRemotes`, route table |
| `src/stores/preloaded-state.ts` | inspected | `PreloadedState {csrfToken, remotes}`, runtime validation, throws if invalid |
| `src/containers/StudentsView.tsx` | inspected | Placeholder heading only |
| `src/containers/LoginView.tsx` | inspected | login link with `return_to`, error toast for `oauth2_callback_failed` |
| `src/containers/RootLayout.tsx`, `RemoteLoadFallbackView.tsx` | inspected | layout (sidebar, welcome modal, no auth guard); remote error fallback | subsystems/page-render |
| `src/containers/HomeView.tsx`, `NotFoundView.tsx` | inspected | `APP_SECTIONS` catalogue (8 sections, 18 cards), greeting; 404 view | subsystems/host-shell |
| `src/components/{AppCard,AppSection,ErrorBoundary,Sidebar,WelcomeModal}.tsx` | inspected | card (internal vs external link), section grid, boundary (no reporting), nav items, first-visit modal (`localStorage`) | subsystems/host-shell |
| `src/components/ui/*.tsx` (9) | identified | shadcn-generated (CONTRIBUTING: regenerate, do not hand-edit); low review priority |
| `src/hooks/use-mobile.ts`, `src/helpers/cn.ts`, `src/env.d.ts`, `src/App.css` | inspected | 768px breakpoint hook; `cn`; Rsbuild types; Tailwind `tw` prefix and design tokens | subsystems/host-shell |
| `components.json` | inspected | shadcn `base-nova`, prefix `tw`, aliases | subsystems/host-shell |
| `src/assets/**` | excluded (binary assets: SVG logos, PNG, MP4) |  |
| `dist/**` | excluded (build output) |  |
| `node_modules/**`, `.claude/.cc-writes` | excluded (dependencies / tool state) |  |

## mock-edupass (`apps/mock-edupass`)

| Path | Status | Notes |
| --- | --- | --- | --- |
| `package.json`, `README.md`, `src/index.ts`, `src/app.ts` | inspected | auto-login, 8 fake accounts per README |
| `src/config.ts`, `src/provider.ts` | inspected | `loadConfig`; `createProvider`, `accounts` (8 fixtures), PS256 + `x5t#S256` check, PKCE required, `account` extra param |
| `test/api.test.ts` (~57 KB) | partially inspected | case names reviewed |
| `test/config.test.ts` | partially inspected | case names reviewed | 09 |
| `test/helpers.ts` | identified |  |
| `tsconfig.json` | inspected | NodeNext, strict, includes `src` and `test` | 09 |

## Build, CI/CD, tooling, root

| Path | Status |
| --- | --- |
| `Dockerfile`, `.dockerignore`, `compose.yml` | inspected |
| `.github/workflows/ci.yml`, `.github/workflows/release.yml`, `.github/PULL_REQUEST_TEMPLATE.md` | inspected |
| `go.mod`, `package.json`, `pnpm-workspace.yaml`, `mise.toml`, `lefthook.yml`, `.golangci.yaml`, `.gitignore` | inspected |
| `.env.example` | inspected (keys and comments; values not reproduced) |
| `.vscode/settings.json` | inspected |
| `.vscode/extensions.json` | inspected |
| `.oxlintrc.json`, `.oxfmtrc.json` | inspected |
| `go.sum`, `pnpm-lock.yaml`, `mise.lock` | excluded (lockfiles) |
| `.env` | excluded (local secrets; deliberately not read) |
| `.git/**` | excluded (only `HEAD` and `refs/heads/main` read for the revision) |

## Repo documentation

| Path | Status | Note |
| --- | --- | --- |
| `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `CHANGELOG.md` | inspected | Treated as intent, not proof |
| `docs/adr/0001-*.md`, `docs/adr/0002-*.md` | inspected | See discrepancies Q1, Q2 |
| `docs/go-test-conventions.md` | inspected | summarised in 09 section 4 |

## Interface coverage

Tracked from Phase 3 onward. Discovered so far (no flows traced yet):

| Interface | Handler | Traced | Sequence diagram | Error paths |
| --- | --- | --- | --- | --- |
| `GET /auth/edupass` | `Handler.authEdupass` | yes | yes | yes |
| `GET /auth/edupass/callback` | `Handler.authEdupassCallback` | yes | yes | yes |
| `/api/` (all methods) | `Handler.proxy()` | yes | yes | yes |
| `/` (catch-all) | `Handler.index()` | yes | yes | yes |
| `/static/` | `Handler.static` | yes | no (flowchart only) | yes |
| mock-edupass `GET /health`, `GET /interaction/:uid`, OIDC endpoints | `createApp`, `oidc-provider` | partial | no | no |
