# 13 Infra Session Handoff

- **Source repo:** `dxd-transform-infrastructure` (local `~/dxd-transform-infrastructure`), `main` @ `b345a0632a34c1e8400bb111262c5f0352763ef1`
- **App repo compared against:** teacher-workspace `main` @ `5ff58a7`
- **Docs location:** teacher-workspace fork, branch `docs/engineering-onboarding`, folder `docs/engineering-onboarding/infra/`. Nothing is written to the infra repo.
- **Scope:** Teacher Workspace and what it connects to only (see `README.md` scope table). Other products are out of scope; `svc.tw-ci` is excluded until IQ1 is answered.
- **History:** started fresh 2026-10-10. Earlier infra notes (2026-10-09/10) were discarded at the user's request and must not be reused.
- **Status: complete (2026-10-10).**
  - Phase 0: inventory, footprint, hostnames, entry points, map, ledger, open questions.
  - Phase 1 batch 1, runtime: `02-infra-runtime.md`.
  - Phase 1 batch 2, edge and network: `04-infra-edge-and-network.md`.
  - Phase 1 batch 3, data and secrets: `05-infra-data-and-secrets.md`.
  - Phase 1 batch 4, static sites: `06-infra-static-sites.md`.
  - Phase 1 batch 5, delivery: `07-infra-delivery.md`.
  - Batch 6, consolidation and audit:
    - verified deployment facts folded into app docs `02`, `07`, `08` and `11`;
    - `00` finalised;
    - ledger summary rebuilt;
    - 118 line citations in `02` to `07` range-checked against the staged sources (0 out of range);
    - all Mermaid blocks parse.
- **Working constraints:**
  - Discovery is read-only (list and stage files).
  - Never run git, terraform, terragrunt, aws or any other shell command on the user's machine. When something needs running, give the exact command for the user to run and paste back.
  - After each batch that changes files, give the copy-paste `git add` / `git commit -m '...'` (single quotes, backticked scope) / `git push -u fork docs/engineering-onboarding`.
  - Never reproduce secret values; cite file and line instead. Do not copy personal contact details from `globals.hcl` or elsewhere.
  - No em-dashes in the docs.

## Key discoveries

1. **The task definitions and the app disagree on configuration (IQ2, IQ3).**
   - dev and stg set `TW_OIDC_*` and `TW_API_PROXY_*`; the app at `5ff58a7` reads `TW_EDUPASS_*` and `TW_REMOTE_*`, and ignores unknown names.
   - An image from `5ff58a7` would therefore exit at config validation and be rolled back.
   - Renaming the signing keys without the remote URLs would also fail validation.
2. **What exists where.**
   - dev and stg each run one 0.25 vCPU ARM64 Fargate task behind an internet-facing per-app ALB, with the shared environment WAF and single-node Valkey (TLS, named RBAC user).
   - prd has only the marketing site, the Edupass credential secret and a Parents Gateway endpoint (IQ14).
3. **Nothing about remotes is wired.**
   - No manifest or backend URLs are set.
   - `svc.tw-pg` lacks CORS, cache control and a stable hostname (`06` 3.3).
   - stg has no Parents Gateway endpoint (IQ15).
4. **Edge behaviour that affects the app.**
   - The `GET /` health check creates a session per probe (IQ7).
   - The ALB idle timeout is 300 s vs the app's 60 s keep-alive (IQ23).
   - Shared WAF rules apply to `/api/` (IQ25), and a WAF secret header would be forwarded to partner backends (IQ21).
   - In dev, the server's calls to mock-edupass loop out through NAT and back in through the WAF (IQ26).
5. **Secrets and data.**
   - Secrets Manager containers are filled by hand; the `init` keys are documented only in task definitions (IQ31).
   - The Edupass secret is built for `private_key_jwt` and is not wired (IQ20).
   - The Valkey password is applied by hand and sits in Terraform state (IQ28).
6. **Delivery.**
   - GitHub Actions build images; a GitLab web pipeline deploys stg only; dev is applied from a laptop (IQ9).
   - ECR tags are mutable with no lifecycle policy (IQ37), and the shared GitHub role can overwrite any repo's tags (IQ35).

## Action list for maintainers (priority order)

| # | Action | IQ | Where |
| --- | --- | --- | --- |
| 1 | Confirm the running image tags in dev and stg; then rename the task definitions' Edupass variables (including `JWKS_URI` to `JWKS_URL`) and drop or complete the remote signing keys | IQ2, IQ3, IQ16 | `02` 3, 5, 7 |
| 2 | Decide the real Edupass setup: `private_key_jwt` wiring from the existing secret, hostnames and redirect URLs (`tw.digital.moe.gov.sg` is held by the marketing site) | IQ20, IQ11, IQ4 | `05` 3.3, `06` 2.4 |
| 3 | Restrict who can push TW images (per-repo ECR push, immutable `v*` tags) and confirm GitLab protections on the stg deploy | IQ35, IQ36, IQ10 | `07` 3, 7 |
| 4 | Before configuring remotes: strip `x-amzn-waf-*` in the proxy, plan WAF handling for `/api/`, and add CORS, cache control and a hostname to `svc.tw-pg` (plus stg and prd copies and a stg PG endpoint) | IQ21, IQ25, IQ3, IQ15, IQ34 | `04` 3, `06` 3 |
| 5 | Add a health route outside the session layer and point the ALB at it; raise the app keep-alive above the ALB idle timeout | IQ7, IQ23 | `04` 2 |
| 6 | Tighten the task security group to the ALB, and document the `init` keys and bootstrap order | IQ18, IQ31 | `02` 4.2, `05` 4.2 |
| 7 | Decide dev deployment (pipeline or documented manual path) and how the mock-edupass image is built | IQ9, IQ12 | `07` 4.3 |
| 8 | Plan prd (instances, Valkey replica, monitoring) | IQ14, IQ29 | `05` 2.4 |

## Commands for the user (optional; paste output back)

| Purpose | Command | Where to run |
| --- | --- | --- |
| When the variable rename happened | `git log -S 'TW_OIDC_' --oneline -- server/` | teacher-workspace repo |
| Running image and service health | `aws ecs describe-services --cluster transform-dev-cluster --services transform-dev-teacher-workspace-main --query 'services[0].{td:taskDefinition,running:runningCount,events:events[:5].message}'` (and stg) | terminal with read-only AWS access |
| Who checks the WAF header | `git grep -n 'alb-expecting-this-header'` | infra repo |

## Suggested next steps (on request)

| Command | When |
| --- | --- |
| `UPDATE infra` | after the infra repo moves past `b345a06`: re-stage the files listed in `03` and update the affected docs |
| `TRACE <flow>` | for example "stg deploy" or "sign-in in dev", answered from these docs plus fresh reads |
| IQ1 follow-up | if `svc.tw-ci` turns out to be part of TW, document it as a new batch |

## Diagram validation

Mermaid blocks in `00-infra-overview.md` (1), `01-infra-repository-map.md` (1), `02-infra-runtime.md` (1), `04-infra-edge-and-network.md` (2), `05-infra-data-and-secrets.md` (1), `06-infra-static-sites.md` (1) and `07-infra-delivery.md` (1), plus the new topology in app doc `08` (section 4), parse with mermaid 11.4.1. Visual layout not reviewed.
