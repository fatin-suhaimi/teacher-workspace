# 10 Business and Technical Glossary

Revision: `main` @ `5ff58a7`. Batch 10 (2026-10-09).

Each entry says where the term comes from. **Repo** means it appears in this repository, cited. **General** means it is general knowledge not stated in the repo, and should be confirmed with the team if it matters.

## Product and architecture

| Term | Meaning | Source |
| --- | --- | --- |
| Teacher Workspace (TW) | "A unified platform that consolidates teacher-facing applications into day-to-day workflows." This repo is its host shell and host backend | Repo: `README.md`; `iss: "tw"` in `proxy.go:62` |
| Host / host shell | The React app in `apps/host` that provides navigation, layout and routing, and mounts remote apps. Module Federation name `teacher_workspace` | Repo: `rsbuild.config.ts:11`, ADR-0001 |
| Host backend | The Go server (`server/cmd/tw`): serves the shell, owns sessions and sign-in, proxies `/api/` | Repo: ADR-0001, `handler.go` |
| Remote app / remote / MFE | A micro-frontend owned by another team in another repo, loaded into the shell at runtime through Module Federation | Repo: ADR-0001, `CONTRIBUTING.md` |
| Module Federation (MF) | Webpack/Rspack mechanism for loading code from separately deployed bundles at runtime; the host shares `react`, `react-dom` and `react-router` as singletons with remotes | Repo: `rsbuild.config.ts`; General for the mechanism |
| Remote entry / manifest | The file a remote publishes so the host can load it (`mf-manifest.json`). TW passes its URL to the browser as `remotes[].entry` | Repo: `CONTRIBUTING.md`, `index.go:28-31` |
| `registerRemotes` / `loadRemote` | MF runtime calls: register remotes from the preloaded state at boot, then load an exposed module such as `pg/Posts` on a route | Repo: `bootstrap.tsx:11`, `App.tsx:16-32` |
| Preloaded state | JSON (`csrfToken`, `remotes`) the server embeds in `index.html` for the SPA to read at startup | Repo: `index.go:14-22`, `preloaded-state.ts` |
| PG / Parents Gateway | The remote named `pg`. Exposes `Posts` and `Groups`, API prefix `/api/posts`, JWT audience `pg`, dev server port 3004 by convention | Repo: `CONTRIBUTING.md`, `proxy.go:28-34`, `App.tsx:16-32`. "Parents Gateway" expansion: Repo (`CONTRIBUTING.md`) |
| Posts, Groups | Communications features of Parents Gateway, shown under "Communications" in the sidebar | Repo: `Sidebar.tsx:32-35` |
| SI / Student Insights | The remote named `si` ("Holistic insights that help every student thrive"). API prefix `/api/student-insights`, audience `si`. Not yet wired in the host (placeholder at `/students`) | Repo: `HomeView.tsx:30-35`, `proxy.go:35-41`, `App.tsx:44` |
| Signed token / remote backend JWT | Short-lived HS256 JWT TW attaches when proxying to a remote's backend | Repo: `proxy.go:60-67` |
| BFF (backend-for-frontend) | Description of TW's role: the browser talks only to TW, which talks to backends | General term applied in these docs |

## Identity and security

| Term | Meaning | Source |
| --- | --- | --- |
| Edupass | MOE's identity provider used for staff sign-in; TW is an OIDC client of it. Its token error format (`error_codes`, `trace_id`, `correlation_id`) resembles Microsoft Entra ID | Repo: `auth.go`, mock README; Entra resemblance: Inferred |
| mock-edupass | Local OIDC provider (`apps/mock-edupass`) that stands in for Edupass in development and tests; auto-login, 8 fixture accounts | Repo: `apps/mock-edupass/README.md` |
| Edupass group | A string in the ID token's `groups` claim granting a role or attribute at a location | Repo: `provider.ts:14-62`, mock README |
| Group grammar | `<location>_<appCode>_<ROLE or ATTR>_<NAME>`, e.g. `0001_TW_ROLE_TEACHER`, `X_TW_ATTR_PG_ADMIN` | Repo: fixtures (Inferred grammar) |
| Location | School or org code prefix in a group (`0001`, `1001`, `1234`); `X` means global (all locations) | Repo: mock README ("Global role (location=X)") |
| App code `TW` / `TWSTG` | Teacher Workspace production vs pre-production ("staging") app codes in groups | Repo: mock README ("Pre-prod TWSTG app code") |
| Role vs attribute | `ROLE` names a job role (`TEACHER`, `HOD`, `COUNSELLOR`); `ATTR` grants a capability (`PG_ADMIN`). Not yet parsed by TW | Repo: fixtures; Q26 |
| HOD | Head of Department (school role) | General |
| OIDC, authorization code flow | Sign-in protocol TW uses with Edupass | General; implementation `auth.go` |
| PKCE (S256) | Proof Key for Code Exchange: a per-login secret (`code_verifier`) whose hash is sent at authorize time | Repo: `auth.go:50, 60, 173-180` |
| `state` | Random value tying the callback to the login the session started (CSRF protection for the callback) | Repo: `auth.go:48, 123` |
| `nonce` | Random value echoed in the ID token, preventing token replay | Repo: `auth.go:49, 229` |
| `client_secret_post` | Client authenticates to the token endpoint with a shared secret in the form body | Repo: `config.go:232`, `auth.go:177-181` |
| `private_key_jwt` | Client authenticates with a JWT signed by its private key (PS256 here); required by real Edupass per the mock README | Repo: `auth.go:153-176` |
| `x5t#S256` | JWT header holding the base64url SHA-256 thumbprint of the client's X.509 certificate | Repo: `auth.go:164`, `config.go:415-419` |
| Session / session store | Server-side state keyed by the `tw_session` cookie, held in memory or Valkey | Repo: `subsystems/sessions.md` |
| Default TTL / authenticated TTL | Idle timeouts for anonymous (3h) and signed-in (30m) sessions | Repo: `config.go:59-60` |
| CSRF token (masked) | Per-render token derived from the session's secret, embedded in the page; not yet verified | Repo: `csrf.go`, Q8 |
| Valkey | Open-source Redis-compatible key-value store used for sessions; accessed via `valkey-glide` (cgo) | Repo: `go.mod`, `compose.yml` |

## Delivery and tooling

| Term | Meaning | Source |
| --- | --- | --- |
| Release PR | A `release/vX.Y.Z` branch that bumps `package.json` and `CHANGELOG.md`, titled `release: vX.Y.Z`; merging it publishes the image and tag | Repo: ADR-0002, `release.yml` |
| PR image tags | `pr-<n>-<sha>` and `pr-<n>-latest` images pushed for each same-repo PR, for pre-merge testing | Repo: `ci.yml:131-133` |
| ECR `transform/teacher-workspace` | AWS container registry repository for TW images | Repo: `ci.yml:130`, `release.yml:22` |
| GitLab pipeline | The deployment pipeline (outside this repo) that takes a `vX.Y.Z` tag and deploys to staging and production | Repo: ADR-0002 |
| Onward | A sibling project (`transformteamsg/onward`) whose release-PR workflow TW copied and corrected | Repo: ADR-0002 |
| `transformteamsg` / `String-sg` | GitHub organisations: links in CHANGELOG and ADRs use `transformteamsg`, while the Go module path uses `String-sg` (Q4) | Repo: `CHANGELOG.md`, `go.mod` |
| mise | Tool version manager pinning Go, Node, pnpm and golangci-lint, with a checksum lockfile | Repo: `mise.toml` |
| lefthook | Git hook runner; runs oxfmt and oxlint on staged files before commit | Repo: `lefthook.yml` |
| oxlint / oxfmt | Rust-based JS/TS linter and formatter (replacing ESLint and Prettier here) | Repo: `.oxlintrc.json`, `.oxfmtrc.json` |
| Rsbuild | Rspack-based build tool for the host | Repo: `apps/host/package.json` |
| shadcn / Base UI | Component generator (style `base-nova`) and headless component library behind `components/ui` | Repo: `components.json` |
| `tw:` prefix | Tailwind class prefix used throughout the host | Repo: `App.css:2` |
| testcontainers | Go library that starts real Valkey containers in tests | Repo: `go.mod`, `valkeystore_test.go` |

## Apps in the home page catalogue

All from `HomeView.tsx:24-202` (descriptions quoted or condensed from the repo). Acronym expansions marked General are not in the repo.

| App | Repo description | Notes |
| --- | --- | --- |
| Student Insights | Holistic insights that help every student thrive | internal (`/students`), "Beta" |
| School Cockpit | Central hub for school management and daily operations |  |
| SC Mobile | Streamlined attendance management on the go |  |
| SLS | Teaching and learning platform for curriculum-aligned resources | Singapore Student Learning Space (General) |
| All Ears | Personalised forms for students, staff and parents | `forms.moe.edu.sg` |
| Allocate | Simplify your Full SBB class allocation | SBB = Subject-Based Banding (General) |
| SDIS | National School Games, Singapore Youth Festival, OALC and MOE OBS Challenge |  |
| MySEI | Insights for students' social-emotional growth and well-being | SEI = social-emotional (General, partial) |
| Connecto-gram | Social network analysis for student connectedness |  |
| Termly Check-In | Regular well-being check-ins |  |
| SEConnect | Section name: Social-Emotional and Mental Wellbeing |  |
| HeyTalia | AI assistant for drafting parent-friendly school communications | links to `pg.moe.edu.sg` (Parents Gateway) |
| Appraiser | AI-generated draft student testimonials |  |
| LangBuddy | AI chatbot for Mother Tongue Language learning (secondary) |  |
| Workpal | Workplace management companion |  |
| HRP | HR and Payroll portal: leave, claims, HR admin |  |
| OPAL 2.0 | Portal for professional learning |  |
| Glow | Bite-sized daily learning |  |
