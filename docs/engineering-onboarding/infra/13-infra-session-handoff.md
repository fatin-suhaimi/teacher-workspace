# 13 Infra Session Handoff

- **Source repo:** `dxd-transform-infrastructure` (local `~/dxd-transform-infrastructure`), `main` @ `b345a0632a34c1e8400bb111262c5f0352763ef1`
- **App repo compared against:** teacher-workspace `main` @ `5ff58a7`
- **Docs location:** teacher-workspace fork, branch `docs/engineering-onboarding`, folder `docs/engineering-onboarding/infra/`. Nothing is written to the infra repo.
- **Scope:** Teacher Workspace and what it connects to only (see `README.md` scope table). Other products are out of scope; `svc.tw-ci` is excluded until IQ1 is answered.
- **History:** started fresh 2026-10-10. Earlier infra notes (2026-10-09/10) were discarded at the user's request and must not be reused.
- **Completed:** Phase 0 (2026-10-10): inventory, footprint per environment, hostnames, entry points, provisional deployment map, ledger, open questions.
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
2. **The task definitions' env var names do not match the app at `5ff58a7`.** They set `TW_OIDC_*` and `TW_API_PROXY_*`; the app reads `TW_EDUPASS_*` and `TW_REMOTE_*` (IQ2).
3. **Remote apps are unconfigured in infra.** Only signing keys are set, with no manifest or backend URLs. `svc.tw-pg` hosts the PG micro-frontend bundle (IQ3).
4. **Deploy paths.**
   - Only stg has a GitLab deploy job.
   - How dev is deployed is unknown (IQ9).
   - The Valkey user group is applied by hand.
   - Secret values are uploaded out of band.
5. **Doc conflicts.**
   - `ARCHITECTURE.md` says TW is a static site (IQ4).
   - Infra ADR-0001's PG hostname differs from the repo's (IQ5).
   - The TW repo is referred to under three GitHub org names (IQ10).

## Recommended analysis order

| Order | Batch | Phase | Files | Output |
| --- | --- | --- | --- | --- |
| **1 (next)** | **Runtime: ECS services and task config** | 1 | `svc.teacher-workspace/ecs-services/main` (dev, stg), `ecs-services/mock-edupass` (dev); `env.{dev,stg}/ecs-cluster`; `env.dev/private-namespace`; `globals.hcl` naming; `acct-vars.hcl` subnets and VPC | `infra/02-infra-runtime.md`: task definition per env, a full env-var and secret mapping table against app config (`02-architecture.md` section 6), IAM roles, security groups, deploy settings, Service Connect; resolve or sharpen IQ2, IQ3, IQ13 |
| 2 | Edge and network | 1 | `svc.teacher-workspace/alb/*` (dev, stg), `env.{dev,stg}/wafv2/*`, ACM, Route53 records, `acct.*/network` (firewall egress allowlist: Edupass, PG, remote hosts), `pg-connect`, `parentsgateway.com.sg` zones, `svc.mock-pg` | `infra/04-infra-edge-and-network.md`: request path diagram, health checks, WAF, TLS, egress, PrivateLink; IQ5, IQ7, IQ15 |
| 3 | Data and secrets | 1 | `elasticache/*` (dev, stg), `secrets` (all envs), `acct.*/kms`, `infra/modules/aws/secrets` | `infra/05-infra-data-and-secrets.md`: Valkey TLS and auth vs the app's `valkey://` URL parsing, secret keys and owners, manual steps; IQ6 |
| 4 | Static sites | 1 | marketing `cloudfront`/`s3` (dev, prd), `env.prd/wafv2/us-east-1`, `svc.tw-pg/*` | `infra/06-infra-static-sites.md`: marketing site and PG MFE hosting, who uploads, WAF |
| 5 | Delivery | 1 | `acct.mgmt/ecr`, `acct.mgmt/iam/{github,gitlab}`, `.gitlab-ci.yml`, `.gitlab/teacher-workspace/*`, Atlantis docs (`docs/atlantis-quirks.md` TW-relevant parts) | `infra/07-infra-delivery.md`: image build to ECR to ECS deploy diagram (joining app `release.yml`/`ci.yml`), rollback; IQ9, IQ10, IQ12 |
| 6 | Consolidation | 6 | all infra docs, plus app docs `08`, `11`, `07`, `02` | fold verified deployment facts into the app docs (replace Unknowns), audit citations, final handoff |

## Next batch (precise)

**Phase 1, batch 1: runtime (ECS services and task configuration).** Re-read `acct.lower/env.dev/svc.teacher-workspace/ecs-services/main/terragrunt.hcl`, `.../ecs-services/mock-edupass/terragrunt.hcl` and `acct.stg/env.stg/svc.teacher-workspace/ecs-services/main/terragrunt.hcl` line by line. Then read `env.{dev,stg}/ecs-cluster/terragrunt.hcl`, `env.dev/private-namespace`, `globals.hcl` (name prefixes, tags, subnet inputs) and `acct.{lower,stg}/acct-vars.hcl`. Produce `infra/02-infra-runtime.md` with a variable-by-variable mapping against the app's config reference.

Suggested command: `CONTINUE`

## Diagram validation

Mermaid blocks in `00-infra-overview.md` (1) and `01-infra-repository-map.md` (1) are parse-checked with mermaid 11.4.1; visual layout not reviewed.
