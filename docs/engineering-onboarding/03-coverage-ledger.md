# 03 Coverage Ledger

Revision: `main` @ `5ff58a7`. Last updated: Phase 1 batch 1 (2026-10-09). Statuses: `inspected`, `partially inspected`, `identified`, `excluded (reason)`, `inaccessible`. "Inspected" means read in full at this revision, not that every call has been traced.

## Summary

| Area | Files | Inspected | Partially | Identified only | Excluded |
| --- | --- | --- | --- | --- | --- |
| Server Go source (non-test) | 21 | 6 | 0 | 15 | 0 |
| Server Go tests | 18 | 1 | 3 | 14 | 0 |
| Host app source + config (excl. assets, dist) | 33 | 9 | 0 | 24 | 0 |
| mock-edupass source + tests + config | 10 | 4 | 0 | 6 | 0 |
| Build, CI/CD, tooling, root config | 22 | 15 | 0 | 3 | 4 (lockfiles, `.env`) |
| Repo docs | 7 | 6 | 1 | 0 | 0 |

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
| `server/internal/handler/auth.go` | S3 | identified | `authEdupass`, `authEdupassCallback` |  |
| `server/internal/handler/auth_test.go` | S3 | identified | ~57 KB |  |
| `server/internal/handler/index.go` | S3 | identified | `index`, likely `static` |  |
| `server/internal/handler/index_test.go` | S3 | identified |  |  |
| `server/internal/handler/proxy.go` | S3 | identified | `proxy` |  |
| `server/internal/handler/proxy_test.go` | S3 | identified |  |  |
| `server/internal/handler/handler_test.go` | S3 | identified | 16 bytes (package clause only, presumably) |  |
| `server/internal/middleware/middleware.go` | S4 | inspected | `Middleware`, `Chain` (first = outermost) | 02 |
| `server/internal/middleware/requestid.go` | S4 | identified | `RequestID` |  |
| `server/internal/middleware/requestlog.go` | S4 | identified | `RequestLog`, `LoggerFromContext` (used in handler.go L119) |  |
| `server/internal/middleware/session.go` | S4/S5 | identified | `Session`, `SessionOptions` |  |
| `server/internal/middleware/middleware_test.go` | S4 | inspected | `TestChain` | 02 |
| `server/internal/middleware/{requestid,requestlog,session}_test.go` | S4 | identified | `session_test.go` ~48 KB |  |
| `server/internal/session/session.go` | S5 | identified | `Store` interface |  |
| `server/internal/session/csrf.go` | S5 | identified |  |  |
| `server/internal/session/memstore/memstore.go` | S5 | identified | `New` |  |
| `server/internal/session/valkeystore/valkeystore.go` | S5 | identified | `New`, `WithPrefix` |  |
| `server/internal/session/**/_test.go` (4 files) | S5 | identified | valkeystore tests use testcontainers |  |
| `server/internal/htmlutil/template.go` | S6 | identified | `Template`, `NewURLTemplate`, `NewFileTemplate` |  |
| `server/internal/htmlutil/template_test.go` | S6 | identified |  |  |
| `server/internal/httputil/httputil.go`, `error.go` | S6 | identified | `RenderPlain` |  |
| `server/internal/httputil/httputil_test.go` | S6 | identified |  |  |
| `server/pkg/random/random.go` (+ test) | S6 | identified |  |  |
| `server/pkg/require/require.go` | S6 | identified | no test file |  |

## Host frontend (`apps/host`)

| Path | Status | Notes |
| --- | --- | --- |
| `package.json`, `rsbuild.config.ts`, `tsconfig.json`, `index.html` | inspected | MF host `teacher_workspace`, `remotes: {}`, shared singletons; path alias `~/*`; `@mf-types` path |
| `src/index.ts`, `src/bootstrap.tsx`, `src/App.tsx` | inspected | entry, `registerRemotes`, route table |
| `src/stores/preloaded-state.ts` | inspected | `PreloadedState {csrfToken, remotes}`, runtime validation, throws if invalid |
| `src/containers/StudentsView.tsx` | inspected | Placeholder heading only |
| `src/containers/{LoginView,RootLayout,HomeView,NotFoundView,RemoteLoadFallbackView}.tsx` | identified |  |
| `src/components/{AppCard,AppSection,ErrorBoundary,Sidebar,WelcomeModal}.tsx` | identified |  |
| `src/components/ui/*.tsx` (9) | identified | shadcn-generated (CONTRIBUTING: regenerate, do not hand-edit); low review priority |
| `src/hooks/use-mobile.ts`, `src/helpers/cn.ts`, `src/env.d.ts`, `src/App.css` | identified |  |
| `components.json` | identified | shadcn config |
| `src/assets/**` | excluded (binary assets: SVG logos, PNG, MP4) |  |
| `dist/**` | excluded (build output) |  |
| `node_modules/**`, `.claude/.cc-writes` | excluded (dependencies / tool state) |  |

## mock-edupass (`apps/mock-edupass`)

| Path | Status | Notes |
| --- | --- | --- |
| `package.json`, `README.md`, `src/index.ts`, `src/app.ts` | inspected | auto-login, 8 fake accounts per README |
| `src/config.ts`, `src/provider.ts` | identified | `loadConfig`, `createProvider`, `accounts` |
| `test/api.test.ts` (~57 KB), `test/config.test.ts`, `test/helpers.ts` | identified |  |
| `tsconfig.json` | identified |  |

## Build, CI/CD, tooling, root

| Path | Status |
| --- | --- |
| `Dockerfile`, `.dockerignore`, `compose.yml` | inspected |
| `.github/workflows/ci.yml`, `.github/workflows/release.yml`, `.github/PULL_REQUEST_TEMPLATE.md` | inspected |
| `go.mod`, `package.json`, `pnpm-workspace.yaml`, `mise.toml`, `lefthook.yml`, `.golangci.yaml`, `.gitignore` | inspected |
| `.env.example` | inspected (keys and comments; values not reproduced) |
| `.vscode/settings.json` | inspected |
| `.vscode/extensions.json` | identified |
| `.oxlintrc.json`, `.oxfmtrc.json` | identified |
| `go.sum`, `pnpm-lock.yaml`, `mise.lock` | excluded (lockfiles) |
| `.env` | excluded (local secrets; deliberately not read) |
| `.git/**` | excluded (only `HEAD` and `refs/heads/main` read for the revision) |

## Repo documentation

| Path | Status | Note |
| --- | --- | --- |
| `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `CHANGELOG.md` | inspected | Treated as intent, not proof |
| `docs/adr/0001-*.md`, `docs/adr/0002-*.md` | inspected | See discrepancies Q1, Q2 |
| `docs/go-test-conventions.md` | partially inspected | First section only |

## Interface coverage

Tracked from Phase 3 onward. Discovered so far (no flows traced yet):

| Interface | Handler | Traced | Sequence diagram | Error paths |
| --- | --- | --- | --- | --- |
| `GET /auth/edupass` | `Handler.authEdupass` | no | no | no |
| `GET /auth/edupass/callback` | `Handler.authEdupassCallback` | no | no | no |
| `/api/` (all methods) | `Handler.proxy()` | no | no | no |
| `/` (catch-all) | `Handler.index()` | no | no | no |
| `/static/` | `Handler.static` | no | no | no |
| mock-edupass `GET /health`, `GET /interaction/:uid`, OIDC endpoints | `createApp`, `oidc-provider` | partial | no | no |
