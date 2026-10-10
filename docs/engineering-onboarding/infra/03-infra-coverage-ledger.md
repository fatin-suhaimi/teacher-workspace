# 03 Infra Coverage Ledger (TW slice)

Infra revision: `main` @ `b345a06`. Last updated: Phase 0 (2026-10-10). Paths relative to `infra/states/provider.aws/` unless shown otherwise. Statuses: `inspected` (read in full), `partially inspected`, `identified`, `excluded (reason)`, `inaccessible`.

Scope rule: only TW and what it connects to (see `README.md`). Other products' `svc.*` directories are excluded as out of scope and not listed individually.

## Summary

| Area | Files | Inspected | Partially | Identified | Excluded / inaccessible |
| --- | --- | --- | --- | --- | --- |
| `svc.teacher-workspace` (dev, stg, prd) | 22 `.hcl` | 22 | 0 | 0 | `.terraform.lock.hcl` and `.terragrunt-cache` excluded |
| Connected services (`svc.tw-pg`, `svc.mock-pg`) | 4 | 4 | 0 | 0 |  |
| `svc.tw-ci` (relation unknown) | 9 (8 `.hcl` + `init.sql`) | 1 | 1 | 7 | pending IQ1 |
| Shared stacks naming TW | 7 | 2 | 5 | 0 |  |
| Shared platform stacks | 23 | 0 | 0 | 23 | searched for TW names only |
| Pipelines | 7 | 7 | 0 | 0 | external `cicd-templates` inaccessible |
| Repo docs (TW-relevant) | 8 | 5 | 2 | 1 | other ADRs, RFCs, product docs excluded |

## `svc.teacher-workspace`

| Path | Status | Key content |
| --- | --- | --- |
| `acct.lower/env.dev/svc.teacher-workspace/ecs-services/main/terragrunt.hcl` | inspected | app service; env vars `TW_*` (IQ2); secrets `TW_SESSION_VALKEY_URL`, `TW_API_PROXY_POSTS_SIGNING_KEY` |
| `.../env.dev/svc.teacher-workspace/ecs-services/mock-edupass/terragrunt.hcl` | inspected | mock IdP service, `MOCK_EDUPASS_*`, Service Connect |
| `.../env.dev/svc.teacher-workspace/ecs-services/dev-console/` | identified | `.terragrunt-cache` and `tfplan` only, no config (IQ8) |
| `.../env.dev/svc.teacher-workspace/alb/terragrunt.hcl`, `target-groups.hcl`, `https-listener-rules.hcl` | inspected | ALB, TGs `main` :3000 and `mock-edupass` :9000, host rules 101/102 |
| `.../env.dev/svc.teacher-workspace/elasticache/cache/terragrunt.hcl` | inspected | Valkey replication group |
| `.../env.dev/svc.teacher-workspace/elasticache/user-group/terragrunt.hcl` | inspected | default user, manual apply |
| `.../env.dev/svc.teacher-workspace/secrets/terragrunt.hcl` | inspected | `init`, `edupass/oidc-client-credentials` |
| `.../env.dev/svc.teacher-workspace/cloudfront/terragrunt.hcl`, `s3/terragrunt.hcl` | inspected | marketing site `dev-tw.edutech.works` |
| `.../env.dev/svc.teacher-workspace/pg-connect/vpc-ep-pg/terragrunt.hcl` | inspected | VPC endpoint to Parents Gateway "pre" endpoint service |
| `acct.stg/env.stg/svc.teacher-workspace/ecs-services/main/terragrunt.hcl` | inspected (diff vs dev) | Edupass placeholders `edupass.invalid`, adds Student Insights signing key |
| `acct.stg/.../alb/*` (3 files), `elasticache/*` (2), `secrets` | inspected (diff vs dev) | stg host only; otherwise same as dev |
| `acct.prd/env.prd/svc.teacher-workspace/cloudfront`, `s3`, `pg-connect/vpc-ep-pg` | inspected (diff vs dev) | `tw.digital.moe.gov.sg`, prd WAF, prd PG endpoint service |
| `acct.prd/.../secrets/terragrunt.hcl` | inspected | `edupass/oidc-client-credentials` only |
| `acct.prd/.../secrets/.terragrunt-cache/**` | excluded (generated cache) |  |
| all `.terraform.lock.hcl` | excluded (provider lock files) |  |

## Connected services

| Path | Status | Notes |
| --- | --- | --- |
| `acct.lower/env.dev/svc.tw-pg/cloudfront/terragrunt.hcl`, `s3/terragrunt.hcl` | inspected | PG MFE hosting; WAF line commented out; no alias (default CloudFront domain) |
| `acct.lower/env.dev/svc.mock-pg/alb/terragrunt.hcl`, `tw-inc-tunnel/vpc-ep-svc/terragrunt.hcl` | inspected | spike rig; acceptance not required, allows the lower account root |
| `acct.lower/env.dev/svc.tw-ci/ecs-services/ci/main/terragrunt.hcl` | inspected | read only to judge its relation to TW |
| `acct.lower/env.dev/svc.tw-ci/secrets/terragrunt.hcl` | partially inspected | staged, not reviewed |
| `svc.tw-ci/{bedrock,ecs-tasks,rds,s3,security-groups}/**` | identified | not reviewed pending IQ1 |
| `.gitlab/tw-ci/**` | inspected | dev-only migrate + deploy, image verification against ECR |

## Shared stacks

| Path | Status | TW content |
| --- | --- | --- |
| `acct.lower/route53/edutech.works/terragrunt.hcl` | partially inspected | TW records at L21-39, L303-341 |
| `acct.mgmt/route53/digital.moe.gov.sg/terragrunt.hcl` | partially inspected | L15, L151-169 |
| `acct.lower/route53/parentsgateway.com.sg/terragrunt.hcl`, `acct.prd/route53/parentsgateway.com.sg/terragrunt.hcl` | inspected | private zones |
| `acct.mgmt/ecr/terragrunt.hcl` | partially inspected | L516-563 |
| `acct.mgmt/iam/github/terragrunt.hcl` | partially inspected | L40-68 trust conditions (subjects L53-64) |
| `acct.prd/env.prd/wafv2/us-east-1/terragrunt.hcl` | partially inspected | L20-58 |
| `acct.mgmt/iam/gitlab/terragrunt.hcl`, `iam/github/github-policy.json` | identified | GitLab and GitHub role policies (Phase 1 batch 5) |
| `acct.lower/ecr`, `env.{dev,stg}/{alb,ecs-cluster,wafv2}`, `env.{dev,stg}/cloudwatch/*`, `acct.{lower,stg,prd}/acm/*`, `acct.{lower,stg,prd}/network`, `globals.hcl`, `acct.*/acct-vars.hcl` | identified | searched for TW names (no matches except WAF/ACM wildcards); content to be read where a batch needs it |

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

## Interface and flow coverage

Tracked from Phase 1 onward.

| Flow                                      | Traced | Diagram          | Failure paths |
| ----------------------------------------- | ------ | ---------------- | ------------- |
| Browser to ALB to TW app (dev, stg)       | no     | provisional only | no            |
| TW app to Valkey                          | no     | provisional only | no            |
| TW app to Edupass / mock-edupass          | no     | provisional only | no            |
| TW app to Parents Gateway (PrivateLink)   | no     | provisional only | no            |
| Browser to `pg` remote (tw-pg CloudFront) | no     | provisional only | no            |
| Marketing site (dev, prd)                 | no     | provisional only | no            |
| Image build to ECR to ECS deploy          | no     | no               | no            |
