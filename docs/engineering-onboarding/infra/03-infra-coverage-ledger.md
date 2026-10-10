# 03 Infra Coverage Ledger (TW slice)

Infra revision: `main` @ `b345a06`. Last updated: Phase 1 batch 3, data and secrets (2026-10-10). Paths relative to `infra/states/provider.aws/` unless shown otherwise. Statuses: `inspected` (read in full), `partially inspected`, `identified`, `excluded (reason)`, `inaccessible`.

Scope rule: only TW and what it connects to (see `README.md`). Other products' `svc.*` directories are excluded as out of scope and not listed individually.

## Summary

| Area | Files | Inspected | Partially | Identified | Excluded / inaccessible |
| --- | --- | --- | --- | --- | --- | --- |
| `svc.teacher-workspace` (dev, stg, prd) | 22 `.hcl` | 22 | 0 | 0 | `.terraform.lock.hcl` and `.terragrunt-cache` excluded |
| Connected services (`svc.tw-pg`, `svc.mock-pg`) | 4 | 4 | 0 | 0 |  |
| `svc.tw-ci` (relation unknown) | 9 (8 `.hcl` + `init.sql`) | 1 | 1 | 7 | pending IQ1 |
| Shared stacks naming TW | 7 | 2 | 5 | 0 |  |
| Shared platform stacks | 25 (Phase 0's 23 + `env.dev/private-namespace` + `shared/common-waf-config.hcl`; `acct.{lower,stg}/kms` counted below) | 11 | 2 | 12 |
| Account KMS stacks (`acct.{lower,stg}/kms/ap-southeast-1`) | 2 | 2 | 0 | 0 | prd and mgmt not read | rest searched for TW names only |
| Local modules (`infra/modules/aws/*`) | 14 read | 10 | 4 | 0 | only the files TW stacks rely on |
| Pipelines | 7 | 7 | 0 | 0 | external `cicd-templates` inaccessible |
| Repo docs (TW-relevant) | 8 | 5 | 2 | 1 | other ADRs, RFCs, product docs excluded |

## `svc.teacher-workspace`

| Path | Status | Key content |
| --- | --- | --- |
| `acct.lower/env.dev/svc.teacher-workspace/ecs-services/main/terragrunt.hcl` | inspected (line by line, batch 1; `02-infra-runtime.md`) | app service; env vars `TW_*` (IQ2); secrets `TW_SESSION_VALKEY_URL`, `TW_API_PROXY_POSTS_SIGNING_KEY` |
| `.../env.dev/svc.teacher-workspace/ecs-services/mock-edupass/terragrunt.hcl` | inspected (line by line, batch 1) | mock IdP service, `MOCK_EDUPASS_*`, Service Connect |
| `.../env.dev/svc.teacher-workspace/ecs-services/dev-console/` | identified | `.terragrunt-cache` and `tfplan` only, no config (IQ8) |
| `.../env.dev/svc.teacher-workspace/alb/terragrunt.hcl`, `target-groups.hcl`, `https-listener-rules.hcl` | inspected (line by line, batch 2; `04-infra-edge-and-network.md`) | ALB, TGs `main` :3000 and `mock-edupass` :9000, host rules 101/102 |
| `.../env.dev/svc.teacher-workspace/elasticache/cache/terragrunt.hcl` | inspected (line by line, batch 3; `05-infra-data-and-secrets.md`) | Valkey replication group |
| `.../env.dev/svc.teacher-workspace/elasticache/user-group/terragrunt.hcl` | inspected (line by line, batch 3) | named user `teacher-workspace-valkey-default`, manual apply |
| `.../env.dev/svc.teacher-workspace/secrets/terragrunt.hcl` | inspected (line by line, batch 3) | `init`, `edupass/oidc-client-credentials` (`private_key`, `certificate`) |
| `.../env.dev/svc.teacher-workspace/cloudfront/terragrunt.hcl`, `s3/terragrunt.hcl` | inspected | marketing site `dev-tw.edutech.works` |
| `.../env.dev/svc.teacher-workspace/pg-connect/vpc-ep-pg/terragrunt.hcl` | inspected (batch 2) | VPC endpoint to Parents Gateway "pre" endpoint service |
| `acct.stg/env.stg/svc.teacher-workspace/ecs-services/main/terragrunt.hcl` | inspected (full diff vs dev, batch 1) | Edupass placeholders `edupass.invalid`, adds Student Insights signing key |
| `acct.stg/.../alb/*` (3 files), `elasticache/*` (2), `secrets` | inspected (diff vs dev; ALB redone in batch 2, Valkey and secrets in batch 3: identical except the `init` description) | stg host only; otherwise same as dev |
| `acct.prd/env.prd/svc.teacher-workspace/cloudfront`, `s3`, `pg-connect/vpc-ep-pg` | inspected (diff vs dev) | `tw.digital.moe.gov.sg`, prd WAF, prd PG endpoint service |
| `acct.prd/.../secrets/terragrunt.hcl` | inspected (batch 3) | `edupass/oidc-client-credentials` only |
| `acct.prd/.../secrets/.terragrunt-cache/**` | excluded (generated cache) |  |
| all `.terraform.lock.hcl` | excluded (provider lock files) |  |

## Connected services

| Path | Status | Notes |
| --- | --- | --- |
| `acct.lower/env.dev/svc.tw-pg/cloudfront/terragrunt.hcl`, `s3/terragrunt.hcl` | inspected | PG MFE hosting; WAF line commented out; no alias (default CloudFront domain) |
| `acct.lower/env.dev/svc.mock-pg/alb/terragrunt.hcl`, `tw-inc-tunnel/vpc-ep-svc/terragrunt.hcl` | inspected (line by line, batch 2) | spike rig; acceptance not required, allows the lower account root |
| `acct.lower/env.dev/svc.tw-ci/ecs-services/ci/main/terragrunt.hcl` | inspected | read only to judge its relation to TW |
| `acct.lower/env.dev/svc.tw-ci/secrets/terragrunt.hcl` | partially inspected | staged, not reviewed |
| `svc.tw-ci/{bedrock,ecs-tasks,rds,s3,security-groups}/**` | identified | not reviewed pending IQ1 |
| `.gitlab/tw-ci/**` | inspected | dev-only migrate + deploy, image verification against ECR |

## Shared stacks

| Path | Status | TW content |
| --- | --- | --- |
| `acct.lower/route53/edutech.works/terragrunt.hcl` | partially inspected (L1-40, L295-345, zone at L510) | hard-coded ALB names L21, L25; records L35-39, L303-341 |
| `acct.mgmt/route53/digital.moe.gov.sg/terragrunt.hcl` | partially inspected | L15, L151-169 |
| `acct.lower/route53/parentsgateway.com.sg/terragrunt.hcl`, `acct.prd/route53/parentsgateway.com.sg/terragrunt.hcl` | inspected | private zones |
| `acct.mgmt/ecr/terragrunt.hcl` | partially inspected | L516-563 |
| `acct.mgmt/iam/github/terragrunt.hcl` | partially inspected | L40-68 trust conditions (subjects L53-64) |
| `acct.prd/env.prd/wafv2/us-east-1/terragrunt.hcl` | partially inspected | L20-58 |
| `acct.mgmt/iam/gitlab/terragrunt.hcl`, `iam/github/github-policy.json` | identified | GitLab and GitHub role policies (Phase 1 batch 5) |
| `globals.hcl` | inspected (naming, tags, state; personal contact lines deliberately not copied) | `name_prefixes` L34-44, tags L22-31 |
| `acct.lower/acct-vars.hcl`, `acct.stg/acct-vars.hcl` | inspected (VPC, subnets, project name) | no VPC CIDR in these files |
| `acct.lower/env.dev/ecs-cluster/terragrunt.hcl`, `acct.stg/env.stg/ecs-cluster/terragrunt.hcl` | inspected (identical) | `transform-<env>-cluster` |
| `acct.lower/env.dev/private-namespace/terragrunt.hcl` | inspected | namespace `transform-dev` |
| `acct.*/root.hcl` | identified (symlink; target not read) | assumed to point at `globals.hcl` |
| `acct.lower/network/terragrunt.hcl`, `acct.stg/network/terragrunt.hcl` | inspected (batch 2; stg by diff) | CIDRs, subnets, NAT, NACL, firewall allowlist L148-201 |
| `acct.lower/env.dev/wafv2/ap-southeast-1/terragrunt.hcl`, `acct.stg/env.stg/wafv2/ap-southeast-1/terragrunt.hcl` | inspected (batch 2) | regional ACL; dev adds IP set rules L41-66 |
| `shared/common-waf-config.hcl` | inspected (batch 2) | rules shared by every app's ACL |
| `acct.lower/acm/ap-southeast-1`, `acct.stg/acm/ap-southeast-1` | partially inspected (domain names only) | `*.edutech.works` |
| `acct.lower/ecr`, `env.{dev,stg}/alb` (legacy shared ALB), `env.{dev,stg}/cloudwatch/*`, `acct.{lower,stg,prd}/acm/us-east-1`, `acct.prd/network`, `acct.*/vpc-endpoints`, `acct.{prd,mgmt}/acct-vars.hcl` | identified | searched for TW names (no matches except WAF/ACM wildcards); content to be read where a batch needs it |

## Pipelines and repo docs

| Path | Status |
| --- | --- |
| `.gitlab-ci.yml` | inspected |
| `.gitlab/teacher-workspace/trunk-pipeline.yml` | inspected |
| `.gitlab/tw-ci/trunk-pipeline.yml`, `child-pipelines/**` (4 files) | inspected |
| `wog/moe/dxdtransform/dxd-transform/cicd-templates@v2.3.1` (`.plan`, `.deploy`, `.vars:aws-acct:*`, `.rules:pipeline`) | inaccessible (other GitLab project) |
| `README.md`, `CONTRIBUTING.md`, `CONVENTIONS.md`, `CODEOWNERS`, `docs/adr/0001-*` | inspected |
| `ARCHITECTURE.md` | partially inspected (L1-500; TW-relevant parts in full) |
| `AGENTS.md` | partially inspected (headings, Route53 zone table) |
| `docs/github-to-gitlab-mirroring.md` | identified (headings) |
| other ADRs, RFCs, product docs | excluded (not TW) unless a batch needs them |

## Local modules read

| Path | Status | Used for |
| --- | --- | --- |
| `infra/modules/aws/wafv2/main.tf` | partially inspected (default action, rule blocks; L1-63, L360-470) | URI rules are path-only (L406-457) |
| `infra/modules/aws/network/main.tf` | inspected | VPC module wiring |
| `infra/modules/aws/network/terraform-aws-vpc/main.tf` | partially inspected (default routes) | public to firewall L199-209, private to NAT L755-760 |
| `infra/modules/aws/network/terraform-aws-vpc/network-firewall.tf` | partially inspected (routes, subnets) | edge routing L64-95 |
| `infra/modules/aws/network/routes/main.tf` | inspected | not used by TW stacks |
| `infra/modules/aws/network/vpc-endpoints/{main,sgrp,data,variables}.tf` | inspected | PG endpoint; `auto_accept` unused; 443 from VPC CIDR |
| `infra/modules/aws/secrets/{main,variables,data,outputs}.tf` | inspected (batch 3) | containers only, `prevent_destroy`, optional resource policies |
| `infra/modules/aws/kms/**` | identified | how `role/*-exec` patterns become policy conditions not read |

## App files read for comparison (teacher-workspace `5ff58a7`)

| Path                                 | Why                                                    |
| ------------------------------------ | ------------------------------------------------------ |
| `server/internal/config/config.go`   | every `dotenv` name, defaults and validation (batch 1) |
| `server/pkg/dotenv/dotenv.go`        | unknown variables ignored; missing `.env` allowed      |
| `server/cmd/tw/main.go`              | startup order and exit points                          |
| `Dockerfile`                         | image `ENV`, user, port                                |
| `apps/mock-edupass/src/config.ts`    | `MOCK_EDUPASS_*` names and required settings           |
| `server/internal/handler/proxy.go`   | headers forwarded to remote backends (batch 2)         |
| `server/internal/handler/handler.go` | route table vs WAF path rules (batch 2)                |

## Interface and flow coverage

Tracked from Phase 1 onward.

| Flow | Traced | Diagram | Failure paths |
| --- | --- | --- | --- |
| ECS task start: secrets, config load and validation, Valkey client, listen | yes (batch 1) | `02-infra-runtime.md` section 2 | yes (exit points, circuit breaker) |
| Browser to ALB to TW app (dev, stg) | yes (batch 2) | `04` section 2 | yes (404 default, health, timeouts, WAF) |
| TW app to Valkey | yes (batches 2-3) | `05` 4.2 (bootstrap) | yes (URL shape, TLS, auth, single node) |
| TW app to Edupass / mock-edupass | yes for dev mock (batches 1-2); real Edupass host Unknown | `04` section 4 | partial (WAF hairpin risk, IQ26) |
| TW app to Parents Gateway (PrivateLink) | infra side yes (batch 2); not wired in the app (IQ3) | `04` section 4 | partial |
| Browser to `pg` remote (tw-pg CloudFront) | no | provisional only | no |
| Marketing site (dev, prd) | no | provisional only | no |
| Image build to ECR to ECS deploy | no | no | no |
