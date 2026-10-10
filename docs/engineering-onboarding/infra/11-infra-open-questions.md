# 11 Infra Open Questions and Conflicts

Infra revision: `main` @ `b345a06`; app revision `5ff58a7`. Phase 0 (2026-10-10). IQ numbers are separate from the app track's Q numbers; cross-references are given where they overlap.

## Conflicts

| # | Topic | Source A says | Source B shows | Resolve by |
| --- | --- | --- | --- | --- |
| IQ2 | Env var names | dev and stg task definitions set `TW_OIDC_ISSUER_URL`, `TW_OIDC_AUTH_URL`, `TW_OIDC_TOKEN_URL`, `TW_OIDC_JWKS_URI`, `TW_OIDC_REDIRECT_URL`, `TW_OIDC_CLIENT_ID`, `TW_OIDC_CLIENT_SECRET`, `TW_API_PROXY_POSTS_SIGNING_KEY` (+ `_STUDENT_INSIGHTS_` in stg) (`acct.lower/.../ecs-services/main/terragrunt.hcl:119-181`) | app at `5ff58a7` reads `TW_EDUPASS_*` (incl. `TW_EDUPASS_JWKS_URL`) and `TW_REMOTE_*`; Edupass settings are required (`config.go:243-258, 429-441`; app doc `02-architecture.md` section 6) | Confirm which image tag runs in dev and stg. If it is `5ff58a7` or later, the containers should exit on config validation. App-side Q3 already notes the stale `TW_OIDC_*` names in the mock README |
| IQ4 | How TW is hosted | `ARCHITECTURE.md:48`: TW "is currently served as a static site from S3 and CloudFront" and will move to ECS | dev and stg run the app on ECS; the CloudFront distributions are commented "Marketing site"; prd has only the marketing site | Update `ARCHITECTURE.md`, or confirm the sentence refers only to prd |
| IQ5 | Parents Gateway private hostname | infra ADR-0001: `pg-internal.parentsgateway.com` | zone `parentsgateway.com.sg` with `stable-tw.pre` (lower) and `prod-tw.prd` (prd) (`acct.*/route53/parentsgateway.com.sg/terragrunt.hcl:15-24`) | Ask whether the ADR example was illustrative; record the real names |
| IQ10 | GitHub org for the TW repo | app `go.mod`: `github.com/String-sg/teacher-workspace`; app CHANGELOG links: `transformteamsg/teacher-workspace` | GitHub OIDC trust: `repo:String-dxd/teacher-workspace:*` and `repo:transformteamsg/*` (`acct.mgmt/iam/github/terragrunt.hcl:53-64`) | Three org names in play. CI pushes from `transformteamsg/teacher-workspace` are covered by the wildcard; confirm the canonical org (app Q4) |
| IQ11 | Edupass client registration hostname | secret descriptions name `stg-tw.digital.moe.gov.sg` (dev and stg) and `tw.digital.moe.gov.sg` (prd) as the Edupass app registrations (`secrets/terragrunt.hcl:26`) | the stg app is served at `stg-teacher-workspace.edutech.works`; `stg-tw.digital.moe.gov.sg` has no record in `digital.moe.gov.sg`; `tw.digital.moe.gov.sg` serves the marketing site, not the app | Confirm the intended app hostnames for stg and prd, and the Edupass redirect URLs registered |

## Open questions

| # | Question | Where to look |
| --- | --- | --- |
| IQ1 | Is `svc.tw-ci` part of Teacher Workspace (e.g. a backend a future remote will call), or an unrelated product that shares the `tw-` prefix? Until confirmed it is excluded from these docs | ask maintainers; `svc.tw-ci/**`, `.gitlab/tw-ci/**` |
| IQ3 | Remote apps are not wired anywhere in infra: no task sets `*_MANIFEST_URL` or `*_BACKEND_BASE_URL`, only signing keys. How are the `pg` remote (likely `svc.tw-pg` CloudFront) and its backend (likely Parents Gateway via the VPC endpoint) meant to be configured? | Phase 1 batches 1, 2, 4 |
| IQ6 | The dev task definitions embed the mock IdP's shared client secret as a plain environment value (both the app and mock-edupass). Acceptable because it is a mock, or should it move to the `init` secret? Value deliberately not reproduced | `ecs-services/main/terragrunt.hcl:157-158`, `ecs-services/mock-edupass/terragrunt.hcl:114-115` |
| IQ7 | ALB health check is `GET /` every 5s; in the app `/` goes through the session middleware and stores a 3h session per probe (app R6/Q23). Is a dedicated health route outside the session layer planned (app Q11)? | `alb/target-groups.hcl:13-23`; app `subsystems/sessions.md` |
| IQ8 | `svc.teacher-workspace/ecs-services/dev-console` contains only a Terragrunt cache and a saved `tfplan`, no config. Leftover local experiment, or a stack whose config was removed? | `git ls-files` on that path (user to run) |
| IQ9 | How are the dev TW app and mock-edupass deployed? `.gitlab/teacher-workspace/` only deploys stg; both dev services need `IMAGE_TAG` at apply time | maintainers; `cicd-templates` (inaccessible) |
| IQ12 | Where is the `transform/mock-edupass` image built? The TW repo's Dockerfile and CI only build `transform/teacher-workspace` | TW repo CI (verified absent); ask maintainers |
| IQ13 | stg Edupass settings are placeholders (`https://edupass.invalid`, `placeholder-not-configured`). Is stg intentionally not signable-in yet? | `acct.stg/.../ecs-services/main/terragrunt.hcl` |
| IQ14 | No prd app stacks (ECS, ALB, Valkey, `init` secret). Is prd go-live planned, and will it follow the dev/stg shape? | maintainers |
| IQ15 | No PG VPC endpoint in stg, while dev and prd have one. Intended? | `acct.stg/env.stg/svc.teacher-workspace` listing |
| IQ16 | Deployed state is invisible from code: applied resources, running image tags, secret values, WAF rules in effect. Read-only AWS console or CLI access would close several Unknowns | access request |
| IQ17 | `ARCHITECTURE.md:131` links ADR 0006 (shared to per-app ALB), but `docs/adr/` stops at 0005. TW uses the per-app pattern | infra docs |
