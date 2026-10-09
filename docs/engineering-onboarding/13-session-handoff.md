# 13 Session Handoff

- **Repo:** `teacher-workspace` (local clone at `~/teacher-workspace`), module `github.com/String-sg/teacher-workspace`
- **Branch / revision:** analysed `main` @ `5ff58a78d09a37a6d662794297d7cdf70aeedb91` (re-checked unchanged on 2026-10-09 22:30); docs live on branch `docs/engineering-onboarding` (user pushes to remote `fork`)
- **Objective:** evidence-backed onboarding knowledge base per `CODEBASE_MASTER_PROMPT.md`
- **Completed (2026-10-09):** Phase 0; batches 1 (startup, config, middleware), 2 (sessions, CSRF), 3 (Edupass sign-in), 4 (page render, static), 5 (API proxy), 6 (request ID, logging), 7 (host shell UI)
- **Working constraints:** read-only discovery of the user's folder; no git or shell commands run on the user's behalf. Docs written only to `docs/engineering-onboarding/`. After each batch that changes files, give the user copy-paste `git add` / `git commit -m '...'` (single quotes, conventional commit with backticked scope) / `git push -u fork docs/engineering-onboarding`.

## Coverage highlights

- Server source 21/21 inspected; server tests 17/18 reviewed by case name.
- Host: 24/33 inspected; the 9 remaining are shadcn-generated `components/ui/*` (low priority).
- mock-edupass: all source read; tests by name (`config.test.ts`, `helpers.ts`, `tsconfig.json` not read).
- Build, CI/CD, Docker, tooling, ADRs: read in Phase 0 but not yet written up as `08-*` / `09-*`.
- See `03-coverage-ledger.md`. API catalog complete for the server (`04-api-catalog.md`).

## Key discoveries (evidence-backed)

1. Routes: `/static/` (no session); under Session: `GET /auth/edupass`, `GET /auth/edupass/callback`, `/api/`, `/` catch-all. Chain: RequestID, RequestLog, routes.
2. Config: defaults, CWD `.env`, `Validate()`; exactly two remotes. `02-architecture.md`.
3. Sessions: JSON snapshot in memory or Valkey; sliding idle TTL 3h / 30m; no logout. `subsystems/sessions.md`.
4. Sign-in: OIDC code + PKCE + state + nonce; only `email` kept. `workflows/edupass-sign-in.md`.
5. Page: per-request `{csrfToken, remotes}`; no auth gating; no CSP/frame/HSTS. `subsystems/page-render.md`.
6. API proxy: HS256 JWT `{iss, aud, iat, exp}`, cookies stripped; **no session check, no user claim, no CSRF** (Q21). `workflows/api-proxy.md`.
7. Observability: random `X-Request-ID`, JSON access log; no metrics, tracing, health or panic recovery. `subsystems/observability.md`.
8. Host shell: MF host loading `pg/Posts` and `pg/Groups`; `/students` is a placeholder; hard-coded catalogue of 18 apps; first-visit modal via `localStorage`; `tw:` Tailwind prefix; **no API calls, CSRF token unused, no sign-in awareness**, no top-level error boundary or error reporting; no frontend tests. `subsystems/host-shell.md`.

## Top open questions

Q21 (unauthenticated `/api/`), Q26 (authorisation model), Q30/Q38 (how SPA and remotes learn sign-in state and CSRF token), Q29 (security headers), Q36 (frontend error reporting), Q31 (request ID to backends), Q1 (ADR-0001 image). Full list: `11-open-questions-and-discrepancies.md`.

## Recommended order (updated)

| Order | Batch | Files / output |
| --- | --- | --- |
| **8 (next)** | **Delivery, ops and local dev** | Re-check `Dockerfile`, `compose.yml`, `.github/workflows/ci.yml`, `release.yml`, `mise.toml`, `lefthook.yml`, `package.json`, `pnpm-workspace.yaml`, `CONTRIBUTING.md`, ADR-0002, `.oxlintrc.json`, `.oxfmtrc.json`, mock `test/config.test.ts`; write `08-infrastructure-and-operations.md` (image, CI/CD, release, runtime config, diagrams) and `09-local-development-and-testing.md` |
| 9 | Security consolidation | write `07-security-and-auth.md` from batches 2-7 (no new code) |
| 10 | Phase 6 consolidation | `05-feature-to-code-map.md`, `06-data-model.md`, `10-business-glossary.md`, `12-change-impact-guide.md`, `AUDIT` pass, refresh `00-project-overview.md` |

## Next batch (precise)

**Batch 8: delivery, operations and local development.** Re-read the build and CI files listed in row 8 (mostly read in Phase 0) plus the two lint/format configs. Produce `08-infrastructure-and-operations.md` (Docker stages, runtime user and env, PR image vs release image flow, ECR and GitLab hand-off, CI/CD and release diagrams, rollback options, what lives outside the repo) and `09-local-development-and-testing.md` (toolchain, first-time setup, running with mock-edupass and Valkey, signing in as other fixtures, test commands, what CI does and does not run).

Suggested command: `CONTINUE`

## Diagram validation

All Mermaid blocks in `00`, `01`, `02`, `subsystems/*`, `workflows/*` parse with mermaid 11.4.1. Visual layout not reviewed.
