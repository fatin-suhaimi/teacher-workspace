# Teacher Workspace: Deployment Infrastructure (infra track)

Evidence-backed onboarding docs for how Teacher Workspace (TW) is deployed, built from the organisation's infrastructure repository. This track complements the application docs one level up (`../README.md`).

- **Source repo:** `dxd-transform-infrastructure` (local clone `~/dxd-transform-infrastructure`), title "MOE DIVA Infra"
- **Revision analysed:** `main` @ `b345a0632a34c1e8400bb111262c5f0352763ef1` (from `.git/refs/heads/main`; uncommitted changes not checked)
- **App revision these docs compare against:** teacher-workspace `main` @ `5ff58a7`
- **Status:** Phase 0 (inventory and plan) and Phase 1 batches 1-4 (runtime; edge and network; data and secrets; static sites) complete, 2026-10-10. Docs not yet traced by a Phase 1 batch stay provisional.
- **Started fresh on 2026-10-10.** Earlier infra notes from 2026-10-09/10 were discarded at the user's request and not reused.

## Scope

The infra repo holds infrastructure for every Transform product. This track covers **only** Teacher Workspace and the things it connects to:

| In scope | Why |
| --- | --- |
| `svc.teacher-workspace` in dev, stg, prd | The TW app, its mock IdP, cache, secrets, ALB, marketing site, Parents Gateway endpoint |
| `svc.tw-pg` (dev) | Hosts the Parents Gateway (`pg`) micro-frontend that TW loads |
| `svc.mock-pg` (dev) | The PrivateLink test rig for TW to Parents Gateway (infra ADR-0001) |
| `.gitlab/teacher-workspace/` | TW deploy pipeline |
| Shared stacks, only where they name TW | DNS records, ECR repositories, GitHub OIDC trust, prd WAF for TW, Parents Gateway private zones |
| Shared platform context TW depends on | ECS cluster, VPC and subnet tiers, KMS keys, Atlantis, secrets pattern (described, not inventoried) |

Explicitly **out of scope**: every other `svc.*` product. `svc.tw-ci` is identified but **not documented** until its relation to TW is confirmed (IQ1).

## Status labels

| Label | Meaning |
| --- | --- |
| Verified | Read in the infra repo at the stated revision |
| Inferred | Strongly suggested by names, comments or Terraform/AWS behaviour, but not shown by code |
| Unknown | No evidence in either repo (often: deployed state, external pipelines, other teams) |
| Conflicts | Infra code, infra docs and/or TW app code disagree |

Note: the infra repo describes **intended** state. What is actually deployed (applied state, running image tags, secret values) is not visible from code and is always Unknown here unless stated otherwise.

## Documents

| File | Purpose |
| --- | --- |
| [00-infra-overview.md](00-infra-overview.md) | Provisional deployment map: environments, hostnames, components, connections |
| [01-infra-repository-map.md](01-infra-repository-map.md) | How the infra repo is organised, tooling, TW footprint inventory, entry points, dependencies |
| [02-infra-runtime.md](02-infra-runtime.md) | ECS services and task definitions per environment, env var and secret mapping against the app's config, startup outcome, IAM and security groups |
| [03-infra-coverage-ledger.md](03-infra-coverage-ledger.md) | Per-file review status for the TW slice of the infra repo |
| [04-infra-edge-and-network.md](04-infra-edge-and-network.md) | Network layout, inbound path (DNS, ALB, TLS, WAF, health checks), outbound path (NAT, firewall allowlist), PrivateLink to Parents Gateway |
| [05-infra-data-and-secrets.md](05-infra-data-and-secrets.md) | Valkey session store (TLS, auth, expected URL shape), Secrets Manager containers and their keys, Edupass credential wiring, KMS access, bootstrap order |
| [06-infra-static-sites.md](06-infra-static-sites.md) | Marketing site (dev, prd) and the `pg` micro-frontend host (`svc.tw-pg`): CloudFront, S3, OAC, WAF allowlists, uploaders, fitness as a remote manifest host |
| [11-infra-open-questions.md](11-infra-open-questions.md) | Infra conflicts and open questions (IQ numbers) |
| [13-infra-session-handoff.md](13-infra-session-handoff.md) | Resume checkpoint and next batch |
