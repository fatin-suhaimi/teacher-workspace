# 15 Documentation Sync Log

One entry per incremental sync run (`DOCUMENTATION_SYNC_PROMPT.md`). The checkpoint itself is `documentation-sync-state.json`.

## 2026-10-16: `5ff58a7` to `abe5457`

### A. Comparison

| Item | Value |
| --- | --- |
| Baseline (BASE) | `5ff58a78d09a37a6d662794297d7cdf70aeedb91`, the revision recorded in `README.md` and `13-session-handoff.md` (no sync-state file existed before this run) |
| Target (TARGET) | `abe54576c604b5879679bf98a7cfd1fd3d0e7e8e`, upstream `transformteamsg/teacher-workspace` `main`, fetched over HTTPS at the start of the run |
| Ancestry | BASE is an ancestor of TARGET (`git merge-base --is-ancestor`) |
| Commits | 9: #199, #204, #205, #200, #202, #211, #212, #222, #223 |
| Files | 33 changed (2085+, 361-); 1 added (`.github/CODEOWNERS`), 0 deleted, 0 renamed |
| Subsystems | Edupass sign-in (S3), API proxy (S3), sessions (S5), config (S2), mock-edupass (S8), host shell (S7), repo identity / CI (S9) |

### B. Source changes

| Subsystem | Change | Kind |
| --- | --- | --- |
| Sign-in | `groups` claim resolved into one base role plus attributes (`resolveEdupassGroups`, allow-lists of 22 roles and 9 attributes); sign-in refused unless exactly one base role (#199) | behaviour change, potentially breaking for users without one role |
| Sign-in | MOE Kindergarten base roles dropped before resolution (#202) | behaviour change |
| Sign-in | ID token must carry `id`, `name`, `cesEmail`; `iat` must exist and be at most 5 min in the future; `email` no longer read (#200) | behaviour change |
| Sign-in | single `Exchange` call with shared options; every `RetrieveError` logged with `edupass_*` fields (#211) | refactor plus log-shape change |
| Logging | `provider=edupass` attached via `logger.With`; message `login failed at provider` renamed to `failed to get authorization code` (#205, #211) | log-shape change |
| Session | `User` gains `ID`, `Name`, `Role`, `Attributes` (#199) | data structure change (additive JSON) |
| Config | `TW_EDUPASS_APP_CODE` (`TW` default, or `TWSTG`), validated (#199) | new config |
| Proxy | requires a session user with a role and non-nil attributes (500 otherwise); JWT gains `role`, `effective_role`, `attributes` (#212) | behaviour and contract change for partner backends |
| mock-edupass | accounts keyed by `id`; emits `sub`, `iat`, `nbf`, `id`, `name`, `cesEmail`, `groups`; fixtures staff-9, staff-10 added (#199, #200, #202) | dev tooling |
| Repo | Go module renamed to `github.com/transformteamsg/teacher-workspace` (#204); `CODEOWNERS` default reviewers (#223) | refactor / repo config |
| Host | `@fontsource-variable/inter` 5.3.0, `--font-sans: 'Inter Variable'` (#222) | dependency change, visual only |
| Tests | large additions in `auth_test.go`, `proxy_test.go`, `config_test.go`, `api.test.ts`; import-path updates elsewhere | tests |

### C. Documentation changes

| File | Change |
| --- | --- |
| `workflows/edupass-sign-in.md` | rewritten for the new callback: sequence and failure flowcharts updated; new section 7.1 with a group-resolution flowchart (new diagram) and fixture outcome table; observations A1, A3, A5 updated, A7-A10 added; refs remapped |
| `workflows/api-proxy.md` | sequence diagram adds the 500 branch and role claims; partner contract and example payload; ADR-0001 gap table; failure table; change table |
| `subsystems/sessions.md` | `User` class diagram and state diagram; R1 and R6 updated; purpose line |
| `06-data-model.md` | class diagram (`User`, ID token), data-flow flowchart, snapshot JSON, claims table, change table |
| `07-security-and-auth.md` | identity sequence diagram; R1, R3, R9 and test-coverage rows; trust-boundary and sensitive-data rows |
| `subsystems/observability.md` | Edupass and proxy log catalogue; troubleshooting row |
| `11-open-questions-and-discrepancies.md` | Q4 resolved; Q7, Q21, Q25, Q26, Q28 updated; shortlist |
| `12-change-impact-guide.md` | C3 and C4 marked largely / partly done, file and test pointers updated |
| `00`, `01`, `02`, `03`, `04`, `05`, `08`, `09`, `10`, `13`, `14`, `README.md`, `subsystems/host-shell.md`, `subsystems/page-render.md` | targeted text updates (module path, CODEOWNERS, fixtures, `TW_EDUPASS_APP_CODE`, font, glossary terms) and remapped line citations |
| `infra/README.md`, `infra/11-infra-open-questions.md` | cross-reference notes for #204 (IQ10) and `TW_EDUPASS_APP_CODE`; infra repo not re-read |
| `documentation-sync-state.json`, this file | created |

No documentation files or diagrams were removed. Diagrams updated: 8; created: 1 (`edupass-sign-in.md` section 7.1).

### D. Verification

- Read the full diff for every non-test source file, and test case names for `auth_test.go`, `proxy_test.go`; inspected `auth.go`, `proxy.go`, `index.go`, `session.go`, `config.go` and mock `provider.ts`/`app.ts` at TARGET with `git show`.
- Line citations: 128 citations in unchanged prose remapped mechanically with a line-level diff of each changed source file (`5ff58a7` to `abe5457`); 28 that fell into changed code were fixed by hand or superseded by the two workflow rewrites. All 656 resolvable `file:line` citations in the app docs are within range at TARGET; the `auth.go` and `proxy.go` citations in the two workflow docs were content-checked line by line.
- Mermaid: all 30 blocks in the changed docs parse with mermaid 11.4.1 (jsdom). Two failures found (semicolons in labels) and fixed. Visual layout not reviewed.
- Not run: Go tests, mock-edupass tests, the app itself. Fixture outcomes in section 7.1 for staff-1 to staff-8 are derived from code; staff-9 and staff-10 also have direct test cases.

### E. Remaining work and uncertainties

- `DOCUMENTATION_SYNC_PROMPT.md` names `/Users/Fatin_SUHAIMI/teacher-workspace` and `docs/onboarding/`; the docs actually live in `~/teacher-workspace-fatin` under `docs/engineering-onboarding/`, and that clone has only `origin` (the fork), no `upstream`. The sync-state file was placed next to the docs. Update the prompt if this layout is intended.
- The clone's `origin/main` ref is behind the merged history (the docs branch already merged #199 and #205 from somewhere). Not a problem for sync, which compares SHAs fetched from upstream directly.
- Issue status in `14-hands-on-learning-tracker.md` section 6 not re-checked against GitHub.
- Runtime check worth doing: sign in with each mock fixture and call `/api/posts/` to confirm the 500/200 outcomes and JWT payload.
- Infra track: whether stg points at pre-prod Edupass (would need `TW_EDUPASS_APP_CODE=TWSTG`) is unknown.

### F. Checkpoint

`last_reviewed_source_commit` = `abe54576c604b5879679bf98a7cfd1fd3d0e7e8e`.
