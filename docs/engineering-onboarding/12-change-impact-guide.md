# 12 Change-Impact Guide

Revision: `main` @ `5ff58a7`. Batch 11 (2026-10-09). Recipes for common changes, built from the verified analysis in batches 1 to 10. Each lists the files to touch, the tests to update or add, other docs to update, who outside this repo is affected, and the release impact under ADR-0002's SemVer rule. Under that rule, major is reserved for breaking "the host-shell and proxy interface, or the image's runtime contract". While the project is at `0.0.x`, these are guidance only.

## Quick lookup

| I want to...                                                 | Section |
| ------------------------------------------------------------ | ------- |
| add a remote app                                             | C1      |
| add a server setting                                         | C2      |
| use Edupass roles, or store more about the user              | C3      |
| require sign-in on `/api/` and tell backends who the user is | C4      |
| enforce CSRF                                                 | C5      |
| add a server route (logout, health, me)                      | C6      |
| change what the page embeds for the SPA                      | C7      |
| edit the home catalogue or sidebar                           | C8      |
| change session TTLs, cookie or storage                       | C9      |
| add a middleware (e.g. security headers, panic recovery)     | C10     |
| wire Student Insights                                        | C11     |
| bump React or React Router                                   | C12     |
| change Edupass client auth or scopes                         | C13     |
| change the image or CI                                       | C14     |

## C1. Add a remote app

|  |  |
| --- | --- |
| Files | `config.go`: three fields in `RemoteAppsConfig`, a validation block and an `Is<App>Registered` helper (copy the Posts pattern, `config.go:429-536`); `index.go:25-32`: append `Remote{Name, Entry}`; `proxy.go:28-41`: add a `remoteBackends["<api-prefix>"]` entry with its audience; `apps/host/src/App.tsx`: lazy `loadRemote('<name>/<Module>')` route wrapped in `ErrorBoundary` + `Suspense`; `Sidebar.tsx` and/or `HomeView.tsx` links; `.env.example` |
| Tests | `config_test.go` (accept/reject tables), `index_test.go` (remote embedding), `proxy_test.go` (routing, audience, prefix stripping) |
| Docs | `02-architecture.md` config reference, `workflows/api-proxy.md` path mapping, `10-business-glossary.md`, `CONTRIBUTING.md` local-remote section |
| Outside the repo | the remote team: MF name, exposed modules, shared singleton versions, manifest URL, and verifying the JWT (`aud`, key); deployment config for the three new env vars |
| Release | minor (new capability) |
| Watch out | four places must agree on identifiers (Q33); a half-set triple stops startup (`config.go` all-or-none). Generalising remotes to a list is the long-term fix (Q20) |

## C2. Add a server setting

|  |  |
| --- | --- |
| Files | `Config` field with a `dotenv:"TW_..."` tag in the right section, a default in `Default()` if safe, a rule in that section's `validate()` (`config.go`); `.env.example` with a comment |
| Tests | `config_test.go`: default value (`TestDefault`), accept and reject cases; follow `docs/go-test-conventions.md` |
| Docs | `02-architecture.md` section 6 |
| Outside | deployment env (GitLab / platform) if there is no safe default |
| Release | patch or minor; **major** if a previously optional setting becomes required (image runtime contract) |
| Watch out | set-but-empty overrides the default (Q18); `.env` is CWD-relative; keyed struct literals only (CONTRIBUTING) |

## C3. Use Edupass roles or store more user data

|  |  |
| --- | --- |
| Files | claims struct `auth.go:235-237` (add `sub`, `name`, `groups`); parse groups and reject users without a TW role before `SetUser` (`auth.go:243-249`); extend `session.User` (`session.go:58-61`) |
| Tests | `auth_test.go`: new cases per mock fixture (staff-4 conflict, staff-5 `TWSTG`, staff-7 non-TW role, staff-8 empty groups); also add the missing guard-branch tests (Q25) |
| Docs | `06-data-model.md`, `workflows/edupass-sign-in.md` section 7, `07-security-and-auth.md` R3, `10-business-glossary.md` |
| Outside | product decision on the role model (Q26); Edupass must emit `groups` for this client |
| Release | minor; behaviourally significant if users start being rejected |
| Watch out | session JSON is additive, so no migration; `SetUser` clears `data`, so read anything needed first; `TWSTG` vs `TW` app code must match the environment |

## C4. Require sign-in on `/api/` and add user identity to the JWT

|  |  |
| --- | --- |
| Files | `proxy.go:48-59`: `SessionFromContext`, `IsAuthenticated`, return 401 JSON; `proxy.go:61-66`: custom claims struct with `sub` (and email or roles from C3) |
| Tests | `proxy_test.go`: the claim-set assertion at `proxy_test.go:254-262` must change; add an "anonymous caller gets 401" case; existing tests call `h.proxy()` without a session and will need one |
| Docs | `workflows/api-proxy.md` sections 4 and 6, `07-security-and-auth.md` R1, `04-api-catalog.md`, close Q21 |
| Outside | **every partner backend**: new claims to read; remotes must handle 401 (e.g. send the user to `/login`) |
| Release | **major** under ADR-0002 (proxy interface change); coordinate with partner teams; consider an ADR |
| Watch out | do C5 in the same change (R1 + R2), or cookies gain authority without CSRF protection; the host has no "signed in" signal yet (C7) |

## C5. Enforce CSRF

|  |  |
| --- | --- |
| Files | a middleware (or a check in `proxy()`) for unsafe methods that reads a header (e.g. `X-CSRF-Token`) and calls `sess.VerifyCSRFToken`; wire it in `Handler.Routes` around `/api/` (`handler.go:93-105`) |
| Tests | new middleware tests (accepts a minted token, rejects missing or foreign tokens, ignores GET/HEAD/OPTIONS); `proxy_test.go` setup gains a token |
| Docs | `subsystems/sessions.md` section 6, `07-security-and-auth.md` R2, close Q8 |
| Outside | remotes must send the header; decide how they obtain the token (read `#preloaded-state`, or a shared module from the host) and document it (Q38) |
| Release | **major** (remotes must change) unless rolled out in report-only mode first |
| Watch out | tokens are invalidated at sign-in (`SetUser` rotates the secret), so a page loaded before sign-in must reload |

## C6. Add a server route (logout, health, me)

|  |  |
| --- | --- |
| Files | `Handler.Routes` (`handler.go:93-105`). Session-aware routes go on the inner `app` mux; probes such as health go on the outer `mux` next to `/static/` so they create no sessions; new handler file in `internal/handler` |
| Notes per route | **logout**: needs a way to drop the session (the handler has no store; add a "destroy" flag the middleware honours, or give `Handler` the store), make it POST + CSRF; **health**: outer mux, no session, consider checking Valkey; **me**: returns `session.User`, an alternative to C7 |
| Tests | handler tests in `internal/handler`, following `index_test.go` patterns |
| Docs | `04-api-catalog.md`, the relevant subsystem doc |
| Release | minor |
| Watch out | the `/` catch-all matches every unregistered path and method, so a typo in a pattern silently serves the SPA |

## C7. Change the preloaded state

|  |  |
| --- | --- |
| Files | `PreloadedState` (`index.go:14-17`) and `index()` (`index.go:43-46`); host `stores/preloaded-state.ts`: the interface and `isPreloadedState` validator |
| Tests | `index_test.go` (embedding cases) |
| Docs | `subsystems/page-render.md` section 4, `06-data-model.md` |
| Outside | any remote that reads `#preloaded-state` from the DOM (none known) |
| Release | minor for added fields; host-shell interface change if fields are removed or renamed |
| Watch out | the host **throws on boot** if validation fails, so server and host must change together (they ship in one image, but local dev mixes `pnpm dev` and `go run`) |

## C8. Edit the home catalogue or sidebar

|  |  |
| --- | --- |
| Files | `APP_SECTIONS` in `HomeView.tsx:24-202` (logo in `src/assets/logos/`); `navItems` / `communicationsItems` in `Sidebar.tsx:27-35` |
| Tests | none exist |
| Docs | `subsystems/host-shell.md` sections 4-5, `10-business-glossary.md` catalogue table |
| Release | patch or minor |
| Watch out | `href` starting with `http` opens in a new tab; anything else is an internal `Link` (`AppCard.tsx:85-97`); card `key` is the title within a section |

## C9. Change session TTLs, cookie or storage

|  |  |
| --- | --- |
| Files | TTL defaults `config.go:57-66`; cookie attributes `middleware/session.go:103-111`; snapshot shape `session.go:50-56`; Valkey key scheme `valkeystore.go` |
| Tests | `middleware/session_test.go` (TTL, cookie), `session_test.go` (load/save), store tests |
| Docs | `subsystems/sessions.md`, `06-data-model.md`, `CONTRIBUTING.md` valkey-cli section if keys change |
| Release | patch; renaming `id` or `csrf_token`, or changing the key prefix, signs everyone out on deploy |
| Watch out | TTLs below 1s are rejected for a reason (`Max-Age=0` quirk, `config.go:179-180`); `SameSite=Strict` would break the Edupass redirect back (the callback must carry the cookie on a cross-site top-level navigation) (Inferred, browser behaviour) |

## C10. Add a middleware for every request

|  |  |
| --- | --- |
| Files | `main.go:107-111` `middleware.Chain(...)`: first argument is outermost; or `Handler.Routes` for app routes only |
| Placement | panic recovery inside `RequestLog` (after it in the list) so the 500 is logged; security headers anywhere, but CSP must allow remote origins (manifest hosts) |
| Tests | a `*_test.go` in `internal/middleware` following `requestlog_test.go` |
| Docs | `02-architecture.md` section 7, `subsystems/observability.md` |
| Release | patch or minor; a strict CSP can break remotes, so test with each remote |

## C11. Wire Student Insights

|  |  |
| --- | --- |
| Files | `App.tsx:44`: replace `StudentsView` with a lazy `loadRemote('si/<Module>')` route wrapped like Posts |
| Outside | the SI team's exposed module name; `TW_REMOTE_STUDENT_INSIGHTS_*` in each environment |
| Release | minor |
| Watch out | the catalogue and sidebar already link to `/students`; without a registered `si` remote the fallback view shows |

## C12. Bump React or React Router

|  |  |
| --- | --- |
| Files | `apps/host/package.json`, `requiredVersion` in `rsbuild.config.ts:13-26`, `pnpm-lock.yaml` |
| Outside | **every remote** shares these as singletons and must be compatible |
| Release | treat a major library bump as a host-shell interface change |
| Watch out | `minimumReleaseAge` (7 days) blocks very new versions; host and remotes must also match dev vs prod builds |

## C13. Change Edupass client auth or scopes

|  |  |
| --- | --- |
| Files | `TW_EDUPASS_CLIENT_AUTH_METHOD` and credential vars (config only), or `Scopes` in `handler.go:54`; assertion shape `auth.go:153-176` |
| Tests | `auth_test.go` client auth sections, `config_test.go` credential sections; mock-edupass `api.test.ts` for the mock side |
| Outside | the Edupass client registration; mock-edupass config (`MOCK_EDUPASS_TW_*`) for local dev |
| Watch out | the key must be PKCS#8 RSA >= 2048 bits and match the cert, and cert expiry is only checked at startup (Q19) |

## C14. Change the image or CI

|  |  |
| --- | --- |
| Files | `Dockerfile`, `.github/workflows/ci.yml`, `release.yml` |
| Notes | amd64 needs the pnpm download changed (`Dockerfile:17-19`), `platforms` updated in both workflows, and an amd64 or emulated runner (cgo); new CI jobs should be added to the image job's `needs` |
| Outside | GitLab deploy pipeline and partner developers pulling the image (ADR-0001) |
| Release | changes to the runtime contract (port, required env, user, paths) are **major** under ADR-0002 |
