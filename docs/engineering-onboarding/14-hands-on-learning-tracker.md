# 14 Hands-On Learning Tracker

Started 2026-10-11 per `HANDS_ON_LEARNING_MASTER_PROMPT.md`. Baseline revision: `main` @ `5ff58a7` (confirmed current after `git fetch` on 2026-10-11: `git diff --stat 5ff58a7 origin/main` is empty).

Familiarity: 0 not explored, 1 read docs, 2 inspected code, 3 traced runtime, 4 modified and tested, 5 confident. Documented = covered by docs 00-13 (analysis, not hands-on evidence).

## 1. Coverage matrix

Doc paths are relative to `docs/engineering-onboarding/` on the `docs/engineering-onboarding` branch unless they start with a repo path. `§` = section number in that doc.

| # | Subsystem | Key paths | Documented | Hands-on | Exercises | Open issues | Docs to keep open |
| --- | --- | --- | --- | --- | --- | --- | --- |
| S1 | Toolchain and local dev | `mise.toml`, `.env.example`, `compose.yml`, `CONTRIBUTING.md` | yes (09) | 1 | E1 | [#113](https://github.com/transformteamsg/teacher-workspace/issues/113) | `09` §1-3 (setup, running), §7 (troubleshooting); `01` (layout, entry points); repo `CONTRIBUTING.md`, `apps/mock-edupass/README.md`; `11` Q41, Q42 |
| S2 | Server startup and config | `server/cmd/tw/main.go`, `internal/config/config.go`, `pkg/dotenv` | yes (02) | 1 | E5, E11, G1 | [#149](https://github.com/transformteamsg/teacher-workspace/issues/149) ([#135](https://github.com/transformteamsg/teacher-workspace/issues/135), [#171](https://github.com/transformteamsg/teacher-workspace/issues/171) stale) | `02` §3 (startup), §5 (config loading), §6 (config reference); `12` C2 (add a server setting); `09` §4 (tests) |
| S3 | Middleware and observability | `internal/middleware/{requestid,requestlog,middleware}.go` | yes | 1 | E4, G3 | none (mock side: [#177](https://github.com/transformteamsg/teacher-workspace/issues/177)) | `subsystems/observability.md` §2-4 (behaviour, format, log catalog), §6 (debugging); `02` §7 and `01` (request pipeline); `12` C10 (add a middleware) |
| S4 | Sessions and CSRF | `middleware/session.go`, `internal/session/*` | yes | 1 | E2, E8, G1, G7 | [#149](https://github.com/transformteamsg/teacher-workspace/issues/149), [#145](https://github.com/transformteamsg/teacher-workspace/issues/145), [#185](https://github.com/transformteamsg/teacher-workspace/issues/185) | `subsystems/sessions.md` §4 (request lifecycle), §5 (TTLs), §6 (CSRF), §7 (stores), §8 (risks), §10 (tests); `06` §3 (session snapshot); `07` §6 (risks R1-R3); `12` C5, C9; `09` §3 (inspecting Valkey) |
| S5 | Edupass sign-in and roles | `handler/auth.go`, `handler/handler.go` | yes | 1 | E3, E6, G4, G7, G8 | [#198](https://github.com/transformteamsg/teacher-workspace/issues/198), [#160](https://github.com/transformteamsg/teacher-workspace/issues/160), [#184](https://github.com/transformteamsg/teacher-workspace/issues/184), [#172](https://github.com/transformteamsg/teacher-workspace/issues/172), [#145](https://github.com/transformteamsg/teacher-workspace/issues/145), [#185](https://github.com/transformteamsg/teacher-workspace/issues/185), [#143](https://github.com/transformteamsg/teacher-workspace/issues/143), [#51](https://github.com/transformteamsg/teacher-workspace/issues/51), [#12](https://github.com/transformteamsg/teacher-workspace/issues/12), [#17](https://github.com/transformteamsg/teacher-workspace/issues/17), [#174](https://github.com/transformteamsg/teacher-workspace/issues/174) (copy for sign-in failures; no code, see S8) | `workflows/edupass-sign-in.md` §3 (happy path), §5 (failure paths), §7 (identity data), §8 (local mock); `07` §2 (auth flow), §4 (authorisation); `06` §5 (claims not stored); `12` C3, C4, C6; `10` (identity glossary); `infra/05-infra-data-and-secrets.md` (for [#172](https://github.com/transformteamsg/teacher-workspace/issues/172)) |
| S6 | Page render and preloaded state | `handler/index.go`, `htmlutil/template.go`, `httputil` | yes | 1 | E2, E7, G10 | [#161](https://github.com/transformteamsg/teacher-workspace/issues/161) | `subsystems/page-render.md` §2 (routing by env), §3 (render sequence), §4 (preloaded state contract), §6 (httputil); `12` C7 (change preloaded state); `05` §3 (blast radius) |
| S7 | API proxy and JWT | `handler/proxy.go` | yes | 1 | E9, G5 | [#79](https://github.com/transformteamsg/teacher-workspace/issues/79), [#163](https://github.com/transformteamsg/teacher-workspace/issues/163), [#152](https://github.com/transformteamsg/teacher-workspace/issues/152) | `workflows/api-proxy.md` §2 (flow), §3 (path mapping), §4 (backend contract), §5 (failures, timeouts); `12` C4; `07` §6; repo `docs/adr/0001-local-development-for-remote-apps.md`; `infra/04-infra-edge-and-network.md` (WAF header, IQ21) |
| S8 | Host shell SPA | `apps/host/src/*` | yes | 1 | E2, E7, E10 | [#161](https://github.com/transformteamsg/teacher-workspace/issues/161), [#164](https://github.com/transformteamsg/teacher-workspace/issues/164), [#14](https://github.com/transformteamsg/teacher-workspace/issues/14) ([#174](https://github.com/transformteamsg/teacher-workspace/issues/174) shown here via `LoginView.tsx`; home row S5) | `subsystems/host-shell.md` §2 (boot), §3 (routes), §4-5 (nav, catalogue), §6 (welcome modal), §7 (styling), §10 (where to change); `12` C8; design sandbox repo [transformteamsg/design-teacher-workspace](https://github.com/transformteamsg/design-teacher-workspace) (prototype for the home page, sidebar and profile menu work) |
| S9 | Module Federation remotes | `rsbuild.config.ts`, `App.tsx`, `ErrorBoundary.tsx` | yes | 1 | E12, E14 | [#114](https://github.com/transformteamsg/teacher-workspace/issues/114) ([#118](https://github.com/transformteamsg/teacher-workspace/issues/118), [#111](https://github.com/transformteamsg/teacher-workspace/issues/111) partly obsolete) | `subsystems/host-shell.md` §8 (errors), §9 (integration with server and remotes); `page-render.md` §4; `12` C1 (add a remote), C11; repo `docs/adr/0001-local-development-for-remote-apps.md`; `infra/06-infra-static-sites.md` (remote hosting) |
| S10 | mock-edupass | `apps/mock-edupass/*` | yes | 1 | E3, E13, G3, G6 | [#177](https://github.com/transformteamsg/teacher-workspace/issues/177), [#173](https://github.com/transformteamsg/teacher-workspace/issues/173), [#162](https://github.com/transformteamsg/teacher-workspace/issues/162) ([#64](https://github.com/transformteamsg/teacher-workspace/issues/64) partly done, [#63](https://github.com/transformteamsg/teacher-workspace/issues/63) stale) | `04` (mock-edupass routes); `workflows/edupass-sign-in.md` §4 (client auth), §8 (local mock); `09` §3 (signing in, `private_key_jwt`); repo `apps/mock-edupass/README.md` |
| S11 | Go tests and testcontainers | `docs/go-test-conventions.md`, `*_test.go` | partial | 1 | E5, E8, G1 | [#167](https://github.com/transformteamsg/teacher-workspace/issues/167) | repo `docs/go-test-conventions.md`; `09` §4 (testing), §5 (lint, hooks); `07` §8 (security test coverage); `03` (test files in the ledger); `11` Q12, Q40 |
| S12 | Build, CI, release | `Dockerfile`, `.github/workflows/*` | yes (08) | 1 | E15, G12 | [#168](https://github.com/transformteamsg/teacher-workspace/issues/168) | `08` §1 (artefact), §2 (CI), §3 (release), §6 (supply chain); repo `docs/adr/0002-release-strategy.md`; `infra/07-infra-delivery.md` |
| S13 | Infra | `~/dxd-transform-infrastructure` | yes (infra/) | 1 | E15, G4, G12 | [#124](https://github.com/transformteamsg/teacher-workspace/issues/124), [#162](https://github.com/transformteamsg/teacher-workspace/issues/162), [#172](https://github.com/transformteamsg/teacher-workspace/issues/172) | `infra/00-infra-overview.md`, `02` (runtime), `04` (edge, network), `05` (data, secrets), `07` (delivery), `11` (IQ list); app `08` §4 (topology); `infra/13-infra-session-handoff.md` (action list) |

Related repos: design sandbox [transformteamsg/design-teacher-workspace](https://github.com/transformteamsg/design-teacher-workspace) (UX prototypes; issues such as [#185](https://github.com/transformteamsg/teacher-workspace/issues/185) link its hosted prototype); infra `dxd-transform-infrastructure` (local `~/dxd-transform-infrastructure`, not on GitHub).

Cross-cutting, useful for every exercise: `00-project-overview.md` (start here), `05-feature-to-code-map.md` (feature to file), `12-change-impact-guide.md` (change recipes), `11-open-questions-and-discrepancies.md`, `10-business-glossary.md`.

## 2. Roadmap

1. **Run it** (S1): nothing else is hands-on until the three processes are up.
2. **One request end to end** (S3, S4, S6, S8): request ID, session, rendered page, SPA boot.
3. **Identity** (S5, S10): the OIDC redirect dance, live and then in the debugger.
4. **Config and tests** (S2, S11): learn the test conventions by writing tests in them.
5. **Partners** (S7, S9): proxy, JWT contract, remote loading.
6. **Delivery** (S12, S13): read-only.

Stages 3 to 5 can go in any order once stage 2 is done.

## 3. Codebase exercises (E1-E15), ranked by learning value

| ID | Exercise | Category | Covers | Level | Time | Status |
| --- | --- | --- | --- | --- | --- | --- |
| E1 | Bring up all three processes, plus Valkey via compose | Runtime / infra | S1 | Beg | 1-2h | not started |
| E2 | DevTools: follow one `GET /` from cookie to preloaded state to React boot | Runtime / DevTools | S4, S6, S8 | Beg | 1h | not started |
| E3 | Sign in with mock-edupass; capture every redirect in the Network tab | Runtime / API trace | S5, S10 | Int | 1-2h | not started |
| E4 | Add a `user_present` field to the request log line | Small change | S3, S4 | Int | 2h | not started |
| E5 | Write a config validation test for the all-or-none rule on `TW_REMOTE_POSTS_*` | Testing | S2, S11 | Int | 2h | not started |
| E6 | Delve breakpoints through `authEdupassCallback` | Debugging | S5, S4 | Int | 2h | not started |
| E7 | Change the `PreloadedState` shape on the server only, watch the host fail, fix both sides | Change / E2E | S6, S8 | Int | 2h | not started |
| E8 | Inspect live session JSON in Valkey; watch the ID rotate on sign-in | Data | S4 | Beg | 1h | not started (partly superseded by G1) |
| E9 | Point the proxy at a throwaway local echo server; decode the JWT it sends | E2E / API | S7 | Int | 2-3h | not started |
| E10 | Add a static catalogue card and a sidebar route | Small change | S8 | Beg | 1h | not started |
| E11 | Trace `main.go` startup: dotenv, `Validate`, store choice, mux | Code exploration | S2 | Beg | 1h | not started |
| E12 | Research Module Federation; build a tiny local remote and load it | Research / PoC | S9 | Adv | 4h+ | not started |
| E13 | Run mock-edupass tests; add one test for an `account` fixture | Testing | S10 | Int | 1-2h | not started (superseded by G6 if taken) |
| E14 | Trigger `RemoteLoadFallbackView` with a bad manifest URL; read `ErrorBoundary` | Debugging | S9 | Beg | 1h | not started |
| E15 | Build the Docker image locally; map `ci.yml` / `release.yml` to `infra/07-infra-delivery.md` | Infra | S12, S13 | Int | 2h | not started |

## 4. Issue-based exercises (G1-G12)

| ID | Issue | Mode | Covers | Level | Status |
| --- | --- | --- | --- | --- | --- |
| G1 | [#149](https://github.com/transformteamsg/teacher-workspace/issues/149) memstore bound | shadow on fork (assigned, in review), then compare with the PR | S4, S2, S11 | Int | not started |
| G2 | Issue hygiene: verify [#135](https://github.com/transformteamsg/teacher-workspace/issues/135), [#171](https://github.com/transformteamsg/teacher-workspace/issues/171), [#63](https://github.com/transformteamsg/teacher-workspace/issues/63), [#64](https://github.com/transformteamsg/teacher-workspace/issues/64), [#118](https://github.com/transformteamsg/teacher-workspace/issues/118) against the code | read-only; draft comments for the user to post if they choose | S2, S7, S9, S10 | Beg | not started |
| G3 | [#177](https://github.com/transformteamsg/teacher-workspace/issues/177) mock-edupass logging | real PR candidate (unassigned) | S10, S3 | Beg-Int | not started |
| G4 | [#172](https://github.com/transformteamsg/teacher-workspace/issues/172) Edupass client timeout vs infra (`handler.go:41`) | real PR candidate (needs Core infra input) | S5, S13 | Int | not started |
| G5 | [#79](https://github.com/transformteamsg/teacher-workspace/issues/79) proxy transport spike | research spike (unassigned) | S7 | Int-Adv | not started |
| G6 | [#173](https://github.com/transformteamsg/teacher-workspace/issues/173) mock RP-initiated logout | real PR candidate; pairs with [#185](https://github.com/transformteamsg/teacher-workspace/issues/185) | S10, S5 | Int | not started |
| G7 | [#145](https://github.com/transformteamsg/teacher-workspace/issues/145) + [#185](https://github.com/transformteamsg/teacher-workspace/issues/185) route guard and sign-out | PR review study (assigned to others) | S4, S5, S8 | Int | not started |
| G8 | [#160](https://github.com/transformteamsg/teacher-workspace/issues/160) + [#198](https://github.com/transformteamsg/teacher-workspace/issues/198) + [#184](https://github.com/transformteamsg/teacher-workspace/issues/184) role resolution | PR review study (assigned to others) | S5 | Int | not started |
| G10 | [#161](https://github.com/transformteamsg/teacher-workspace/issues/161) role in preloaded state | E2E, after [#160](https://github.com/transformteamsg/teacher-workspace/issues/160) merges | S6, S8 | Int-Adv | blocked |
| G12 | [#124](https://github.com/transformteamsg/teacher-workspace/issues/124) immutable ECR tags | read-only infra trace | S13, S12 | Beg | not started |

(G9 and G11 dropped: [#171](https://github.com/transformteamsg/teacher-workspace/issues/171) probably satisfied, [#118](https://github.com/transformteamsg/teacher-workspace/issues/118) premise partly obsolete.)

Suggested order: E1, E2, E3, then G1 or G3 (first real PR: G3 or G6). Confirm with the team before claiming an issue.

## 5. Detailed starting exercises

### E1. Bring the stack up

- **Learn:** the three-process topology, how `.env` drives the server, the failure modes in `09` section 7.
- **Steps:**
  1. Setup per `CONTRIBUTING.md`: `mise trust`, `mise install`, `cp .env.example .env`, `pnpm install`.
  2. Three terminals at the repo root: `pnpm dev`; `go run ./server/cmd/tw`; `pnpm --filter @teacher-workspace/mock-edupass dev`.
  3. Open `http://localhost:3000`.
  4. Break it on purpose, once each: stop the Rsbuild dev server; stop mock-edupass. Note each failure.
  5. `docker compose up -d`, restart the server with `TW_SESSION_STORE_PROVIDER=valkey` (URL form in `09` section 3).
- **Observe:** which failure gives a 500 and which only breaks sign-in; that the server starts without mock-edupass.
- **Done when:** you can explain why `README.md` lists only two terminals and what goes wrong without the third (Q41).
- **Safety:** local only.

### E2. DevTools trace of one page load

- **Steps:**
  1. Network tab, document request for `/`: note `Set-Cookie: tw_session`, `Cache-Control: no-store`, `X-Request-ID`.
  2. Elements: find the preloaded state injected via `apps/host/index.html`.
  3. Application: cookie flags vs `TW_SESSION_SECURE`; `tw_welcome_modal_seen` in localStorage.
  4. Sources: breakpoints in `stores/preloaded-state.ts` and `bootstrap.tsx`; reload.
- **Hypotheses before reading code:** does reload reuse your session or mint a new one? What is in `remotes` with no `TW_REMOTE_*` set?
- **Confirm in code:** `index.go:24-57`, `middleware/session.go`.
- **Done when:** you can narrate browser, middleware chain, `Handler.index`, template, `PreloadedState`, React mount without notes, and match the request ID to its log line.
- **Safety:** read-only.

### E4. Add `user_present` to the request log

- **Branch:** `feat/learning-log-user-present` on the fork.
- **Objective:** each request log line says whether the session has a signed-in user.
- **The design question:** `RequestLog` wraps the Session middleware, so the session is not in context when the logger starts (`main.go:107-111`, `handler.go:93-105`). How do you get the value out, and should you?
- **Pointers:** `middleware/requestlog.go`, `LoggerFromContext`, `session.SetUser`, `requestlog_test.go` log subtests (`slog.JSONHandler` buffer), `docs/go-test-conventions.md`.
- **Done when:** a new subtest passes under `go test -race ./server/internal/middleware`; the field flips false to true across a live sign-in; you can defend the ordering choice.
- **Safety:** local branch only.

## 6. Issue status vs code (verified 2026-10-11 at `5ff58a7`)

| Issue | Status | Evidence |
| --- | --- | --- |
| [#135](https://github.com/transformteamsg/teacher-workspace/issues/135) | stale: `validateHTTPURL`, `APIProxyConfig`, `RemoteConfig` no longer exist; config is now `EdupassConfig` / `RemoteAppsConfig` | `config.go:278`, `:443` |
| [#172](https://github.com/transformteamsg/teacher-workspace/issues/172) | valid, path stale: timeout is `edupassHTTPClient`, 10 s | `handler.go:41` (not `internal/oidc/oidc.go`) |
| [#171](https://github.com/transformteamsg/teacher-workspace/issues/171) | probably satisfied: each remote is optional and all-or-none | `config.go:463-556` |
| [#118](https://github.com/transformteamsg/teacher-workspace/issues/118), [#111](https://github.com/transformteamsg/teacher-workspace/issues/111) premise | partly obsolete: no `PostsView.tsx`; manifest URL comes from `TW_REMOTE_POSTS_MANIFEST_URL` via preloaded state | `App.tsx:16`, `index.go:27-31` |
| [#63](https://github.com/transformteamsg/teacher-workspace/issues/63) | stale: mock is `mock-edupass`, query response mode, not `form_post` | `04-api-catalog.md` |
| [#64](https://github.com/transformteamsg/teacher-workspace/issues/64) | partly done: per-boot key and `private_key_jwt` exist | `04-api-catalog.md` |
| [#149](https://github.com/transformteamsg/teacher-workspace/issues/149), [#79](https://github.com/transformteamsg/teacher-workspace/issues/79), [#185](https://github.com/transformteamsg/teacher-workspace/issues/185), [#173](https://github.com/transformteamsg/teacher-workspace/issues/173), [#198](https://github.com/transformteamsg/teacher-workspace/issues/198), [#160](https://github.com/transformteamsg/teacher-workspace/issues/160), [#184](https://github.com/transformteamsg/teacher-workspace/issues/184), [#145](https://github.com/transformteamsg/teacher-workspace/issues/145) | valid: behaviour absent | grep: no memstore bound or prod check, no proxy `Transport`, no logout, no `cesEmail` / `groups` |

Sync note (2026-10-16, `abe5457`): the grep evidence "no `cesEmail` / `groups`" for #198, #160 and #184 is now stale: `cesEmail` and `groups` are read at sign-in since #199/#200/#202, and #212 adds role claims to the proxy JWT. Whether those issues are closed was not checked (no GitHub access in the sync run).

Side finding: the app renamed `TW_OIDC_*` to `TW_EDUPASS_*`; infra task definitions and some issues still use the old names, so infra is the lagging side of IQ2.

Not code (product, epics, design, copy): [#107](https://github.com/transformteamsg/teacher-workspace/issues/107), [#23](https://github.com/transformteamsg/teacher-workspace/issues/23), [#55](https://github.com/transformteamsg/teacher-workspace/issues/55), [#41](https://github.com/transformteamsg/teacher-workspace/issues/41), [#80](https://github.com/transformteamsg/teacher-workspace/issues/80), [#50](https://github.com/transformteamsg/teacher-workspace/issues/50), [#132](https://github.com/transformteamsg/teacher-workspace/issues/132), [#164](https://github.com/transformteamsg/teacher-workspace/issues/164); [#174](https://github.com/transformteamsg/teacher-workspace/issues/174) is a copy task under S5 (code follow-up: per-reason `error` values in `loginFailedURL`, `auth.go:325-332`, mapped in `LoginView.tsx`). Deprecated: [#182](https://github.com/transformteamsg/teacher-workspace/issues/182), [#183](https://github.com/transformteamsg/teacher-workspace/issues/183).

## 7. Completed exercises

None yet.

## 8. Next

User selects an exercise (E1-E15 or G1-G12).
