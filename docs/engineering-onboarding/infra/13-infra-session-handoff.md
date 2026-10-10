# 13 Infra Session Handoff

- **Source repo:** `dxd-transform-infrastructure` (local `~/dxd-transform-infrastructure`), `main` @ `b345a0632a34c1e8400bb111262c5f0352763ef1`
- **App repo compared against:** teacher-workspace `main` @ `5ff58a7`
- **Docs location:** teacher-workspace fork, branch `docs/engineering-onboarding`, folder `docs/engineering-onboarding/infra/`. Nothing is written to the infra repo.
- **Scope:** Teacher Workspace and what it connects to only (see `README.md` scope table). Other products are out of scope; `svc.tw-ci` is excluded until IQ1 is answered.
- **History:** started fresh 2026-10-10. Earlier infra notes (2026-10-09/10) were discarded at the user's request and must not be reused.
- **Completed:**
  - Phase 0 (2026-10-10): inventory, footprint per environment, hostnames, entry points, provisional deployment map, ledger, open questions.
  - Phase 1 batch 1, runtime (2026-10-10): `02-infra-runtime.md`.
  - Phase 1 batch 2, edge and network (2026-10-10): `04-infra-edge-and-network.md`.
  - Phase 1 batch 3, data and secrets (2026-10-10): `05-infra-data-and-secrets.md`.
- **Working constraints:**
  - Discovery is read-only (list and stage files).
  - Never run git, terraform, terragrunt, aws or any other shell command on the user's machine. When something needs running (for example `git ls-files`), give the exact command for the user to run and paste back.
  - After each batch that changes files, give the copy-paste `git add` / `git commit -m '...'` (single quotes, backticked scope) / `git push -u fork docs/engineering-onboarding`.
  - Never reproduce secret values; cite file and line instead.

## Key discoveries (provisional)

1. **Per-environment footprint.**
   - dev: TW app (ECS), mock-edupass (ECS), per-app ALB, Valkey 9.1, a marketing site and a PG VPC endpoint.
   - stg: app, ALB and Valkey only, with Edupass placeholders.
   - prd: only the marketing site at `tw.digital.moe.gov.sg` (WAF allows SSOE and SEED IPs only), the Edupass client credential secret and a PG VPC endpoint.
2. **The task definitions' env var names do not match the app at `5ff58a7`** (traced in batch 1).
   - They set `TW_OIDC_*` and `TW_API_PROXY_*`; the app reads `TW_EDUPASS_*` (`JWKS_URL`, not `JWKS_URI`) and `TW_REMOTE_*`.
   - Unknown names are silently ignored, and the seven Edupass settings are required, so an image from `5ff58a7` exits 1 at `Validate` in dev and stg (IQ2).
   - The dev mock-edupass values agree with the TW task's values, so dev works once the names are fixed.
3. **Remote apps are unconfigured in infra.** Only signing keys are set, with no manifest or backend URLs. The app treats each remote as all-or-nothing, so renaming the keys alone would also break startup (IQ3). `svc.tw-pg` hosts the PG micro-frontend bundle.
4. **Deploy paths.**
   - Only stg has a GitLab deploy job.
   - How dev is deployed is unknown (IQ9).
   - The Valkey user group is applied by hand.
   - Secret values are uploaded out of band.
5. **Runtime shape (batch 1).** One Fargate task per environment (0.25 vCPU, 512 MB, ARM64), ECS Exec on, no container health check, task security group open to `0.0.0.0/0` on 3000 (IQ18), no `wait_for_steady_state` on `main` (IQ19). The real Edupass credential secret is not read by any task (IQ20). TW `main` is not a Service Connect client; it reaches mock-edupass via its public hostname.
6. **Edge and network (batch 2).**
   - Internet-facing per-app ALB in the web subnets; TLS `*.edutech.works`; plain HTTP to the task.
   - Shared environment WAF: other apps' path allow rules are not host-scoped (IQ25); the Common Rule Set body limits apply to `/api/` (IQ25); the default action injects a secret header that the TW proxy would forward to remote backends (IQ21).
   - Egress: NAT, then Network Firewall domain allowlist (`.edutech.works`, `.gov.sg`, `.amazonaws.com` allowed). In dev the server reaches mock-edupass through the internet and back through the WAF (IQ26).
   - PrivateLink to PG in dev and prd only; endpoint open to the whole VPC (IQ22). IQ5 resolved.
   - ALB idle 300 s vs app keep-alive 60 s (IQ23). Health checks create at least about 4,300 live sessions per environment (IQ7).
7. **Data and secrets (batch 3).**
   - Valkey: one `cache.t3.micro` node, TLS required, RBAC user `teacher-workspace-valkey-default` (not `default`), user group applied by hand with `VALKEY_PASSWORD` (password in state, IQ28). Single node logs everyone out on node events (IQ29).
   - Required URL shape: `valkey://teacher-workspace-valkey-default:<encoded password>@<primary endpoint>:6379?tls=true`.
   - Secrets are empty containers filled by hand; `init` keys are documented only in the task definitions (IQ31). The Edupass secret holds `private_key` and `certificate` for `private_key_jwt`; wiring steps in `05` 3.3 (IQ20). No cert-expiry alert (IQ30).
   - KMS `ServicesSecretsKey` lets every `*-exec`/`*-task` role decrypt, so IAM is the only gate (IQ31).
8. **Doc conflicts.**
   - `ARCHITECTURE.md` says TW is a static site (IQ4).
   - Infra ADR-0001's PG hostname differs from the repo's (IQ5).
   - The TW repo is referred to under three GitHub org names (IQ10).

## Recommended analysis order

| Order | Batch | Phase | Files | Output |
| --- | --- | --- | --- | --- |
| 1 (done) | Runtime: ECS services and task config | 1 | `svc.teacher-workspace/ecs-services/main` (dev, stg), `ecs-services/mock-edupass` (dev); `env.{dev,stg}/ecs-cluster`; `env.dev/private-namespace`; `globals.hcl` naming; `acct-vars.hcl` subnets and VPC | `infra/02-infra-runtime.md`: task definition per env, a full env-var and secret mapping table against app config (`02-architecture.md` section 6), IAM roles, security groups, deploy settings, Service Connect; resolve or sharpen IQ2, IQ3, IQ13 |
| 2 (done) | Edge and network | 1 | `svc.teacher-workspace/alb/*` (dev, stg), `env.{dev,stg}/wafv2/*`, ACM, Route53 records, `acct.*/network` (firewall egress allowlist: Edupass, PG, remote hosts), `pg-connect`, `parentsgateway.com.sg` zones, `svc.mock-pg` | `infra/04-infra-edge-and-network.md`: request path diagram, health checks, WAF, TLS, egress, PrivateLink; the TW task's path to `dev-mock-edupass.edutech.works` (out and back through the ALB); IQ5, IQ7, IQ15, IQ18 |
| 3 (done) | Data and secrets | 1 | `elasticache/*` (dev, stg), `secrets` (all envs), `acct.*/kms`, `infra/modules/aws/secrets` | `infra/05-infra-data-and-secrets.md`: Valkey TLS and auth vs the app's `valkey://` URL parsing, secret keys and owners, manual steps; IQ6 |
| **4 (next)** | **Static sites** | 1 | marketing `cloudfront`/`s3` (dev, prd), `env.prd/wafv2/us-east-1`, `svc.tw-pg/*` | `infra/06-infra-static-sites.md`: marketing site and PG MFE hosting, who uploads, WAF |
| 5 | Delivery | 1 | `acct.mgmt/ecr`, `acct.mgmt/iam/{github,gitlab}`, `.gitlab-ci.yml`, `.gitlab/teacher-workspace/*`, Atlantis docs (`docs/atlantis-quirks.md` TW-relevant parts) | `infra/07-infra-delivery.md`: image build to ECR to ECS deploy diagram (joining app `release.yml`/`ci.yml`), rollback; IQ9, IQ10, IQ12 |
| 6 | Consolidation | 6 | all infra docs, plus app docs `08`, `11`, `07`, `02` | fold verified deployment facts into the app docs (replace Unknowns), audit citations, final handoff |

## Next batch (precise)

**Phase 1, batch 4: static sites.** Read, line by line:

- `acct.lower/env.dev/svc.teacher-workspace/{cloudfront,s3}/terragrunt.hcl` and the prd equivalents (marketing site).
- `acct.prd/env.prd/wafv2/us-east-1/terragrunt.hcl` and `acct.lower/env.dev/wafv2/us-east-1/terragrunt.hcl` (CloudFront WAFs).
- `acct.lower/env.dev/svc.tw-pg/{cloudfront,s3}/terragrunt.hcl` (PG micro-frontend hosting).
- The `tw` record in `acct.mgmt/route53/digital.moe.gov.sg` (L161-169) and `dev-tw` in `edutech.works` (L35-39).
- `acct.{lower,prd}/acm/us-east-1`.
- App side: how the host loads a remote manifest (app `subsystems/remote-apps.md` or equivalent) against the `svc.tw-pg` distribution (CORS, cache headers, whether it is a manifest host).

Produce `infra/06-infra-static-sites.md`: who uploads what, OAC, WAF and geo rules, TLS, and whether `svc.tw-pg` can serve as `TW_REMOTE_POSTS_MANIFEST_URL` (IQ3).

Suggested command: `CONTINUE`

## Diagram validation

Mermaid blocks in `00-infra-overview.md` (1), `01-infra-repository-map.md` (1), `02-infra-runtime.md` (1), `04-infra-edge-and-network.md` (2) and `05-infra-data-and-secrets.md` (1) are parse-checked with mermaid 11.4.1; visual layout not reviewed.
