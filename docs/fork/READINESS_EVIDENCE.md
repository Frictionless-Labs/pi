---
title: Pi Fork Readiness Evidence Registry
version: 1.4.0
status: READY
created_date: 2026-09-04
updated_date: 2026-09-06
tags:
  - pi
  - readiness-evidence
  - audit
confidence: 0.99
owner: Frictionless Labs Repository Maintainer
---

# Pi Fork Readiness Evidence Registry

## Evidence rules and schema

PASS means the named observable gate ran or was directly inspected against the exact observed
ref/state. PENDING means the gate has not run or awaits an immutable candidate/human action.
BLOCKED means current evidence proves a required condition is absent or unsafe. Upstream results,
plans, static configuration, and uncommitted worktree content do not substitute for candidate-SHA
runtime evidence.

| Field | Required meaning |
|---|---|
| Evidence ID | Stable `EVD-YYYYMMDD-NNN` identifier. |
| Candidate SHA | Exact immutable candidate-source commit whose implementation is under test. Use `N/A` only for the narrow non-candidate observation exception below. |
| Observed ref/state | Exact ref/SHA, worktree base plus state, remote resource snapshot, or explicit `NOT_RUN` state actually addressed by the record. |
| Requirement | One or more REQ IDs. |
| Test | One or more TEST IDs whose observable contract this record addresses. |
| Task | Producing TASK ID. |
| Control | Related CTRL/RISK identifier. |
| Actor/runner | Human role, Codex task role, local tool, or remote runner that produced the observation; state `no runner` when pending. |
| Validation | Exact command, API observation, or human review method. |
| Result | PASS, FAIL, BLOCKED, or PENDING. |
| Source class | `SOURCE_FACT`, `EXECUTION_OBSERVATION`, `INFERENCE`, or `HUMAN_DECISION`. |
| Source | Local command, repository file, GitHub API/run, or reviewed artifact reference. |
| Evidence reference | Repository section, stable GitHub API/run/artifact URL or ID, or other durable location where the evidence can be inspected; `/tmp` may be a diagnostic supplement but never the sole reference. |
| Timestamp | Actual observation UTC; never an estimated time. |
| Notes | Redacted exception, failure classification, blocker owner, and next evidence. |

**Narrow non-candidate exception.** `Candidate SHA` is `N/A` when no immutable post-readiness
candidate exists or when the record is explicitly a source/ref/server-state observation rather than
candidate execution. Such a row must identify the exact `Observed ref/state`, actor/runner, method,
UTC, result, and evidence reference, and its notes must include `[NON-CANDIDATE OBSERVATION]`.
TEST-001 through TEST-003 may PASS against exact pre-candidate provenance/synchronization refs; no
other candidate-dependent gate may PASS with `Candidate SHA = N/A`. An executed BLOCKED observation
is still executed evidence of a missing/unsafe condition, not an unrun gate.

Evidence records never contain secret values, auth headers, environment dumps, or unredacted
credential findings. A rerun appends or replaces a result only when the immutable state and method
are explicit; stale evidence remains historical, not current proof.

### Candidate-source and evidence-record revisions

The **candidate-source revision** is the immutable commit containing the workflow and governance
implementation under test. The **evidence-record revision** is the later Git commit/blob containing
the registry row that indexes results for that candidate source. They are intentionally separate:
a source commit cannot contain its own SHA or post-push CI result.

- Before the candidate-source commit exists, its SHA is `N/A`; the uncommitted registry is a
  worktree observation and cannot satisfy a candidate-dependent PASS gate.
- After source commit `S` exists, every candidate-dependent row records `S`. The commit/blob that
  contains that row is evidence-record revision `R`, identified by the Git revision and path used
  to read this file; `R` may postdate `S` and need not self-embed its own SHA.
- Evidence-only revisions may update this registry without changing the tested workflow/product
  source. Any tested-source change creates a new candidate source and invalidates prior candidate-
  keyed execution evidence for the changed candidate.
- Remote evidence uses stable workflow/run/job IDs or artifact URLs. Local summaries required for
  review are embedded below; temporary logs remain supplemental diagnostics only.

## Active v0.85.1 records

Candidate source `f2858ead77530b5bf86bd787b141a83e7a2daa41` contains the reviewed readiness
implementation, two nested lockfile security repairs, and the terminal-event ordering repair proven
necessary by failed main CI run `34014481660`. Evidence before that revision remains historical.
The later evidence-record revision containing this table changes only governance documents and is
intentionally distinct from the candidate source.

| Evidence ID | Candidate SHA | Observed ref/state | Requirement | Test | Task | Control | Actor/runner | Validation | Result | Source class | Evidence reference | Timestamp | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| EVD-20260906-001 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | baseline ref `ac4ac9e...` | REQ-001 | TEST-001 | TASK-GUIDE-001 | CTRL-001 | Local Git 2.50.1 | Dereference archive ref | **PASS** | EXECUTION_OBSERVATION | `SOURCE-002`, `FINAL-LIVE-005` | 2026-09-06T06:04:49Z | Non-`v*` recovery ref still resolves exactly to the pre-sync baseline. |
| EVD-20260906-002 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | upstream release `v0.85.1` = `d981de1...` | REQ-002 | TEST-002 | TASK-ASSESS-001 | CTRL-001 | GitHub API + local Git | Release metadata and tag resolution | **PASS** | EXECUTION_OBSERVATION | `SOURCE-002`, `FINAL-LIVE-005` | 2026-09-06T06:04:49Z | Stable, published, non-draft, non-prerelease target remains frozen. |
| EVD-20260906-003 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | fork `main=f2858ead...`; target lineage retained | REQ-003 | TEST-003 | TASK-GUIDE-002 | CTRL-007 | Local Git + fork API | Ancestry, exact fast-forwards, ref verification | **PASS** | EXECUTION_OBSERVATION | `ALIGN-002`, `FINAL-LIVE-005` | 2026-09-06T06:04:49Z | Baseline remains an ancestor of target; main equals candidate and all transitions retained exact identities. |
| EVD-20260906-004 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | ten workflows; 2 KEEP/8 DISABLE; 16 guarded jobs | REQ-004 | TEST-004 | TASK-ASSESS-002, TASK-GUIDE-003 | CTRL-002, CTRL-006 | Local validator + GitHub API | YAML inventory, guards, action pins, server state | **PASS** | EXECUTION_OBSERVATION | `STATIC-005`, `FINAL-LIVE-005` | 2026-09-06T06:04:49Z | Candidate repair does not touch workflows; all eight DISABLE workflows remain server-disabled. |
| EVD-20260906-005 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | PR#6 and merged `main` exact head | REQ-005 | TEST-005 | TASK-DEPLOY-001 | CTRL-008 | GitHub Actions | Build, check, test on exact head | **PASS** | EXECUTION_OBSERVATION | `CI-REPAIR-005`; runs `34014966796`, `34014979030`, `34015185239` | 2026-09-06T06:08:35Z | Run `34014481660` failed on missing `run_end`; the source repair then passed 20 final focused reruns, push, PR, and post-merge full CI. |
| EVD-20260906-006 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | PR#6 and merged head plus nested lockfiles | REQ-006 | TEST-006 | TASK-DEPLOY-002 | CTRL-009 | npm 11.19.0 + GitHub Actions | Production audit and signature verification | **PASS** | EXECUTION_OBSERVATION | Runs `34014984627`, `34015193772`; `SUPPLY-002` | 2026-09-06T05:56:10Z | Candidate and post-merge audit/signatures passed; nested audits and Dependabot report zero open vulnerabilities. |
| EVD-20260906-007 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | target-to-candidate history and diff | REQ-007 | TEST-007 | TASK-DEPLOY-002 | CTRL-004 | Gitleaks 8.30.1 | Redacted seven-commit history and 145.43 KB diff scans | **PASS** | EXECUTION_OBSERVATION | `SCAN-004` | 2026-09-06T06:05:23Z | Both valid scans found zero leaks; an invalid zero-commit worktree-container attempt is excluded. Two inherited server alerts remain open under `SECURITY-002`. |
| EVD-20260906-008 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | disposable Docker container policy/probe | REQ-008 | TEST-008 | TASK-STRATEGIZE-002 | CTRL-005 | Docker Desktop 4.89.0 / Engine 29.7.2 | Inspect boundary, execute probe, verify disposal | **PASS** | EXECUTION_OBSERVATION | `ISOLATION-002`; unchanged policy diff | 2026-09-06T04:47:53Z | Source repair does not alter isolation controls; prior current-cycle probe proved the boundary and disposal. |
| EVD-20260906-009 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | v1.4.0 governance documents | REQ-009 | TEST-009 | TASK-STRATEGIZE-001, TASK-GUIDE-004 | CTRL-007 | Local validators | Frontmatter, identifiers, agreement, annotation repair, diff check | **PASS** | EXECUTION_OBSERVATION | `STATIC-005` | 2026-09-06T06:07:56Z | Five frontmatters, 14 REQs, 14 TESTs, seven modules, 15 tasks, and 14 active records validate. |
| EVD-20260906-010 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | protected fork `main@f2858ead...` and free controls | REQ-010 | TEST-010 | TASK-DEPLOY-003 | CTRL-011 | Fork GitHub API | Protection, Actions, repo security, collaborators | **PASS** | EXECUTION_OBSERVATION | `SERVER-004`, `FINAL-LIVE-005` | 2026-09-06T06:04:49Z | Four strict checks, admin enforcement, linear history, conversation resolution, no force/delete; approval count remains zero for the sole collaborator. |
| EVD-20260906-011 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | 2 KEEP active; 8 DISABLE server-disabled and guarded | REQ-011 | TEST-011 | TASK-GUIDE-003, TASK-DEPLOY-004 | CTRL-002 | GitHub Actions/API | Workflow state, runs, comments, releases, tags, effects | **PASS** | EXECUTION_OBSERVATION | `WORKFLOW-SERVER-005` | 2026-09-06T06:04:49Z | Only approved verification ran; the only issue comments remain the two attributed Dependabot comments; no release/tag or disabled-job effect exists. |
| EVD-20260906-012 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | target plus bounded 21-path fork delta | REQ-012, REQ-013 | TEST-012 | TASK-GUIDE-005 | CTRL-007, CTRL-010 | Local Git + scoped review | Allowlist, full diff, whitespace, counterexample review | **PASS** | EXECUTION_OBSERVATION | `SCOPE-004`, `STATIC-005` | 2026-09-06T06:07:56Z | Twenty-one paths include the justified four-file source/changelog repair and five evidence documents; no release, credential, publication, generated, or Git metadata change. Annotation repairs remain intact. |
| EVD-20260906-013 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | 14 active evidence records in v1.4.0 | REQ-014 | TEST-013 | TASK-GUIDE-005 | CTRL-010 | Local validator | Record cardinality, IDs, state reconciliation | **PASS** | EXECUTION_OBSERVATION | This registry; `STATIC-005` | 2026-09-06T06:07:56Z | Fourteen unique active IDs reconcile candidate, failure/repair, merge, post-merge, workflow, security, and bot effects. |
| EVD-20260906-014 | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | PR#6 merged as exact candidate head | REQ-011, REQ-014 | TEST-014 | TASK-DEPLOY-004, TASK-VERIFY-001 | CTRL-002, CTRL-010 | Repository Maintainer + protected GitHub path | PR, exact checks, fast-forward, effect audit | **PASS** | HUMAN_DECISION | PR#6; `POSTMERGE-006` | 2026-09-06T06:04:49Z | MIKKOH GO recorded; exact candidate and post-merge checks, restored protection, and effect audit passed. |

## Active command/result ledger

| Reference | Method | Result |
|---|---|---|
| `SOURCE-002` | Git archive-ref/tag resolution plus upstream release API | Exit 0; baseline `ac4ac9e...`; target `d981de1...`; stable release. |
| `ALIGN-002` | `git merge-base --is-ancestor`; `git rev-list`; `git diff --shortstat` | Exit 0; 720 commits; 883 files; target-aligned branch. |
| `SUPPLY-002` | Nested `npm audit --package-lock-only --omit=dev --audit-level=moderate --workspaces=false` | Both exit 0; zero vulnerabilities after two transitive lock updates. |
| `STATIC-002` | Ruby YAML/frontmatter/identifier/record validators; `git diff --check`; Actionlint with shellcheck disabled | Exit 0; workflows parse; 16 guards and 39 pins; schemas/counts pass. Upstream ShellCheck warnings remain separate non-regression observations. |
| `STATIC-003` | Final v1.3.0 frontmatter, identifier, active-record, workflow, annotation-repair, and diff validators | Exit 0; final evidence documents agree and contain no active PENDING gate. |
| `STATIC-004` | v1.3.1 structure, effect-attribution, annotation-repair, workflow, and diff validators | Exit 0; all active evidence remains PASS and expected bot effects are attributed. |
| `STATIC-005` | v1.4.0 frontmatter, identifier, active-record, workflow, annotation-repair, path-scope, and diff validators | Exit 0; all five documents agree on `f2858ead...`, all active records PASS, and the terminal-event repair is explicitly bounded. |
| `SCOPE-002` | Candidate name/status/diff review, whitespace check, prior final-review repair verification | Exit 0; 17 bounded paths; `SPEC.md` makes no claim from unread strategy; RUNBOOK consistently uses `TargetAlignment`. |
| `SCOPE-003` | Final five-document diff review, v1.3.0 structural validator, whitespace check, and Actionlint 1.7.12 | Exit 0; 14 active rows all PASS; annotation repairs intact; workflow guards and pins unchanged. |
| `SCAN-002` | Official Gitleaks 8.30.1 container, network disabled, worktree/Git metadata read-only, redaction enabled | Exit 0; current content plus nonzero 2-commit delta and 686-commit lineage scans report zero leaks. Invalid 0-commit attempts were rejected and rerun. |
| `SCAN-003` | Official Gitleaks 8.30.1 container; network disabled; source read-only; redaction enabled; final v1.3.0 worktree content | Exit 0; 17.78 MB scanned; no leaks found. |
| `SCAN-004` | Official Gitleaks 8.30.1 container; network disabled; worktree and owning Git repository read-only; redaction enabled | Exit 0; seven target-to-candidate commits and 145.43 KB candidate diff scanned with no leaks. The first worktree-only history attempt resolved zero commits and is excluded. |
| `ISOLATION-002` | Hardened cached `alpine:3.22` container; config inspection, runtime assertions, exit/disposal check | Exit 0; boundary properties in EVD-20260906-008 proven; no container remains. |
| `SERVER-002` | GitHub repository, Actions-permission, protection, security/default-setup, collaborator APIs | Free controls enabled as recorded; paid-only validity and non-provider scanning disabled. |
| `SECURITY-002` | Secret metadata with `hide_secret=true`; location/history without reading values | Two alerts predate baseline; source path was deleted 2026-04-30 and is absent at baseline/target. Validity remains `unknown`; alerts remain open and visible. |
| `MAIN-ALIGN-002` | Temporarily remove admin enforcement; exact non-force push of `d981de1...` to `main`; restore enforcement; reread protection | Push exit 0; `main` equals target; all four checks and protection fields retained; admin enforcement restored. |
| `WORKFLOW-SERVER-002` | Disable eight unsafe workflow IDs; reread inventory and PR effects | Exactly CI/audit active and eight repository workflows `disabled_manually`; PR comments/reviews zero. |
| `CODEQL-ADOPTION-002` | Extended CodeQL run `34012647603`; paginated PR-head alerts and changed-path intersection | Both jobs succeeded. 532 target-inherited alerts remain: 444 test, 76 runtime, 4 workflow, 6 scripts, 2 examples. Four alerts intersect fork-changed paths; all four predate the fork delta. |
| `PR-CODEQL-003` | PR#3 default-setup CodeQL run `34013039523`; PR alert API | Both analysis jobs succeeded on `1f346820...`; zero fork-delta alerts. |
| `POSTMERGE-003` | PR#3 merge API; exact-SHA CI `34013262552`, audit `34013305833`, and CodeQL `34013261982`; ref/effect checks | PR#3 merged at exact head `1f346820...`; all post-merge verification succeeded; `main` and readiness ref match. |
| `CI-REPAIR-005` | Failed main run `34014481660`; source inspection; build/type/static gates; 20 focused reruns; 26-test runtime file; PR#6 checks | The old source missed `run_end` in 1/2158 tests. The RPC response could overtake transcript replication. `f2858ead...` retains the subscription through the matching terminal event; all local and hosted reruns passed. |
| `PR-CODEQL-006` | PR#6 CodeQL run `34014978444`; PR alert API | Actions and JavaScript/TypeScript analyses succeeded on `f2858ead...`; zero open PR alerts. |
| `POSTMERGE-006` | PR#6 merge API; exact-SHA CI `34015185239`, audit `34015193772`, and CodeQL `34015184916`; ref/effect checks | PR#6 merged at exact candidate head; all post-merge verification succeeded; `main` and readiness ref match. |
| `SERVER-003` | Post-merge protection, Actions, security, collaborator, and workflow APIs | Four strict checks and admin enforcement restored; free-only controls enabled; 2 KEEP active and 8 DISABLE workflows disabled. |
| `WORKFLOW-SERVER-003` | Initial post-merge workflow/run and effect inventory | Verification paths ran; later repository-wide comment inspection supersedes its overbroad no-comment inference. |
| `WORKFLOW-SERVER-004` | Final workflow/run, release/tag, issue-comment, and PR-effect inventory | No release/tag/publication/deployment/credential or disabled-workflow effect. Dependabot auto-closed PR#1/PR#2 and posted comments `5557114268`/`5557114858`. |
| `FINAL-LIVE-005` | 2026-09-06T06:04:49Z Git/ref/release, workflow, protection, Actions-permission, security, PR, release/tag, and issue-comment API snapshot | Candidate and refs match; protection restored; free controls active; 0 Dependabot alerts, 2 historical secret alerts, 532 inherited CodeQL alerts, 0 releases/tags, and exactly 2 attributed Dependabot comments. |
| `SERVER-004` | Post-PR#6 protection and Actions API snapshot | Four strict checks at GitHub Actions app ID 15368, admin enforcement, linear history, conversation resolution, no force/delete, SHA pinning, and read-only default token verified. |
| `WORKFLOW-SERVER-005` | Post-PR#6 workflow/run and effect inventory | Two KEEP workflows plus dynamic Dependabot/CodeQL are active; eight unsafe repository workflows remain `disabled_manually`; no unauthorized effect observed. |
| `SCOPE-004` | `git diff --name-only/--shortstat d981de1...f2858ead...`, full patch review, and fresh annotation checks | 21 changed paths, 1471 insertions, and 13 deletions; the four-file terminal repair is mandatory-gate-driven and all other changes remain within the reviewed readiness/security/evidence allowlist. |

## Active security disposition

| Finding | Class | Decision |
|---|---|---|
| Nine Dependabot alerts in nested example lockfiles, including critical `shell-quote` and high `undici` advisories | `RESOLVED` | Patched in the retained candidate lineage; nested audits and default-branch Dependabot report zero open alerts; duplicate Dependabot PR#1 and PR#2 closed automatically. |
| Two Google OAuth-pattern alerts from commit `c359023c...` in a path deleted by `fe66edd9...` before the fork baseline | `PRE-EXISTING` | Keep open and visible. Do not claim revocation/validity. They are absent from baseline, target, and candidate content and were not introduced or re-exposed by this delta. |
| 532 CodeQL alerts inherited from the frozen upstream target | `PRE-EXISTING` | Keep extended suite and alerts open. Controlled internal use requires the documented trust boundary; arbitrary untrusted execution remains isolation-only. PR#6 produced zero open PR alerts. |
| Main CI `34014481660` omitted `run_end` from one remote-runtime event stream | `RESOLVED` | `FIX_NOW`: source `f2858ead...` retains the subscription until matching `run_end` or `run_suspend`; 20 focused reruns, all 26 runtime tests, two candidate CI runs, and post-merge CI passed. |

## Active readiness calculation

| Measure | Current result | Basis |
|---|---|---|
| P0 design trace coverage | **100% (11/11)** | Verified kickoff request and repository source set; Downloads strategy/execution files remain absent and **[UNVERIFIED]**. |
| Candidate implementation | **PASS** | Readiness controls and security lock repairs are merged; this governance record is the separate final evidence revision. |
| Fork CI/security | **PASS with inherited backlog disclosed** | Exact-head CI/audit/Gitleaks and PR/post-merge CodeQL passed; all fork-changed CodeQL alerts are pre-existing. |
| Isolation | **PASS** | Hardened disposable boundary and disposal directly observed. |
| Merge governance | **PASS** | Free-tier protected-main controls and current human GO directly observed. |
| Current readiness | **READY; confidence 99%** | Mandatory P0 execution is 100% PASS, TEST-014 passed, MIKKOH GO is recorded, and no P0/P1 blocker remains. |

## Active next dependency

None for this readiness cycle. Future upstream synchronization, release, publication, deployment,
credential, or paid-feature adoption requires a new evidence cycle and separate authorization.

## Pre-candidate observations (superseded)

The v0.85.1 observation window began 2026-09-06T00:08:10Z. The governance files are an
uncommitted worktree delta over `d981de1229ef899957bbe968bc8dcda02a21f477`; TASK-GUIDE-005 and
TASK-DEPLOY-002/003 refreshed local, supply-chain, scanner, and fork-server observations on
2026-09-06. Candidate-dependent
gates remain PENDING or BLOCKED until an immutable candidate exists. The preserved v0.85.0
registry is historical only: v0.85.1 superseded that cycle after upstream fixed the v0.85.0 SDK
import failure caused by unintended published experimental code and dependencies.

| Evidence ID | Candidate SHA | Observed ref/state | Requirement | Test | Task | Control | Actor/runner | Validation | Result | Source class | Source | Evidence reference | Timestamp | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| EVD-20260905-001 | N/A | `HEAD=d981de1229ef899957bbe968bc8dcda02a21f477`; `refs/tags/frictionless-readiness-baseline-20260904^{}=ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278` | REQ-001 | TEST-001 | TASK-GUIDE-001 | CTRL-001, RISK-010 | Codex retarget Task 1; local Git 2.50.1 | Verify and dereference the named archive ref | **PASS** | EXECUTION_OBSERVATION | Local Git exit 0 | Embedded ledger `SOURCE-001` | 2026-09-06T00:08:10Z | [NON-CANDIDATE OBSERVATION] Exact rollback ref passed; no remote-ref claim. |
| EVD-20260905-002 | N/A | `upstream:v0.85.1^{}` and local `v0.85.1^{}` = `d981de1229ef899957bbe968bc8dcda02a21f477` | REQ-002 | TEST-002 | TASK-ASSESS-001 | CTRL-001, RISK-001, RISK-011 | Codex retarget Task 1; GitHub API and local Git | Release API plus local tag resolution | **PASS** | EXECUTION_OBSERVATION | GitHub API + local Git | Embedded ledger `SOURCE-001`; `GET /repos/earendil-works/pi/releases/tags/v0.85.1` | 2026-09-06T00:08:10Z | [NON-CANDIDATE OBSERVATION] Published 2026-09-05T12:29:01Z; non-draft; non-prerelease. |
| EVD-20260905-003 | N/A | `codex/pi-fork-production-readiness-v0851@d981de1229ef899957bbe968bc8dcda02a21f477`; baseline `ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278` | REQ-003 | TEST-003 | TASK-GUIDE-002 | CTRL-007, RISK-001, RISK-005 | Codex retarget Task 1; local Git 2.50.1 | Ancestry, branch-base, rev-list count, diff shortstat, and HEAD | **PASS** | EXECUTION_OBSERVATION | Local Git exits 0 | Embedded ledger `ALIGN-001` | 2026-09-06T00:08:10Z | [NON-CANDIDATE OBSERVATION] Readiness branch was created directly at the target; baseline ancestry and 720-commit/883-file lineage passed. Fork `main` was not synchronized. |
| EVD-20260905-004 | N/A | `HEAD=d981de1229ef899957bbe968bc8dcda02a21f477`; uncommitted 15-path delta; local/fork inventories each 10 workflows | REQ-004 | TEST-004 | TASK-ASSESS-002, TASK-GUIDE-003 | CTRL-002, CTRL-006, RISK-003 | Codex retarget validation; Ruby Psych/local Git/GitHub API | Parse YAML; reconcile inventory; inspect every DISABLE job guard; exits 0 | **PENDING** | EXECUTION_OBSERVATION | Local worktree + fork GitHub API | Embedded ledger `STATIC-001`; fork API routes in `SERVER-001` | 2026-09-06T00:18:28Z | [NON-CANDIDATE OBSERVATION] 2 KEEP/8 DISABLE, 16/16 guards, and 39/39 SHA pins passed locally; immutable candidate/runtime proof remains pending. |
| EVD-20260905-005 | N/A | `Frictionless-Labs/pi` readiness branch absent; target-SHA Actions run count 0 | REQ-005 | TEST-005 | TASK-GUIDE-005, TASK-DEPLOY-001 | CTRL-008, RISK-005, RISK-008 | Codex retarget validation; GitHub API/local Git | Query branch ref and Actions runs by exact target SHA | **BLOCKED** | EXECUTION_OBSERVATION | Fork GitHub API + origin remote | Embedded ledger `SERVER-001`; fork Git ref and Actions-run API routes | 2026-09-06T00:18:28Z | [NON-CANDIDATE OBSERVATION] One candidate commit and the separately authorized one-time readiness-branch push are required before exact-SHA fork CI; the specification grants no standing push authority. |
| EVD-20260905-006 | N/A | Uncommitted v0.85.1 delta; `npm ci`, production audit, and signature audit all exit 0 | REQ-006 | TEST-006 | TASK-DEPLOY-002 | CTRL-009, RISK-007 | Codex retarget validation; npm 11.19.0 | `npm ci --ignore-scripts`; `npm audit --omit=dev --audit-level=high`; `npm audit signatures` | **PENDING** | EXECUTION_OBSERVATION | Local npm registry validation | Embedded ledger `SUPPLY-001` | 2026-09-06T00:16:30Z | [NON-CANDIDATE OBSERVATION] 0 production vulnerabilities; 307 signatures and 60 attestations verified. Re-run on immutable candidate. |
| EVD-20260905-007 | N/A | Frozen target range/content and pre-final-review patch SHA-256 `3ea23af167c79cc377e9095d68ab8d90d8e44f4f890d3ad790a906a626cf126c` | REQ-007 | TEST-007 | TASK-DEPLOY-002 | CTRL-004, RISK-007 | Codex retarget validation; checksum-verified Gitleaks 8.30.1 | Official GitHub release asset checksum; redacted history/content/pre-fix-patch scans | **PENDING** | EXECUTION_OBSERVATION | Local scanner; reports outside repository | Embedded ledger `SCAN-001`; temporary detailed log is supplemental | 2026-09-06T00:22:39Z | [NON-CANDIDATE OBSERVATION] Historical pre-fix scan only, not the complete current candidate. All three inputs scanned with 0 leaks; exact lineage is 720 while Gitleaks progress reported 684. A complete post-fix patch scan and later immutable-candidate scan remain required. |
| EVD-20260905-008 | N/A | v0.85.1 policy uncommitted; isolation operator proof = `NOT_RUN` | REQ-008 | TEST-008 | TASK-STRATEGIZE-002 | CTRL-005, RISK-004 | No isolation runner | Review policy and execute one independently evidenced isolation classification | **PENDING** | SOURCE_FACT | Repository policy + retarget state | `SECURITY.md`; `docs/fork/FORK_POLICY.md` | 2026-09-06T00:08:10Z | [NON-CANDIDATE OBSERVATION] Policy exists; runtime boundary proof remains required. |
| EVD-20260905-009 | N/A | `HEAD=d981de1229ef899957bbe968bc8dcda02a21f477`; uncommitted v1.1.0 control plane | REQ-009 | TEST-009 | TASK-STRATEGIZE-001, TASK-GUIDE-004 | CTRL-007, RISK-009 | Codex retarget validation; Ruby/local Git | Validate frontmatter, all 15 task schemas, evidence IDs, target consistency, and path scope; exits 0 | **PENDING** | EXECUTION_OBSERVATION | Local worktree | Embedded ledger `STATIC-001` | 2026-09-06T00:16:18Z | [NON-CANDIDATE OBSERVATION] Local structure passed; immutable candidate and independent review remain pending. |
| EVD-20260905-010 | N/A | Fork `main`: branch protection absent; repository rulesets count 0 | REQ-010 | TEST-010 | TASK-DEPLOY-003 | CTRL-011, RISK-006 | Codex retarget validation; GitHub API | Query branch protection and repository rulesets | **BLOCKED** | EXECUTION_OBSERVATION | Fork GitHub API | Embedded ledger `SERVER-001`; branch-protection and ruleset API routes | 2026-09-06T00:18:28Z | [NON-CANDIDATE OBSERVATION] Current merge controls are absent; maintainer governance decision and separately authorized configuration are required. |
| EVD-20260905-011 | N/A | `HEAD=d981de1229ef899957bbe968bc8dcda02a21f477`; uncommitted 8-workflow guard delta | REQ-011 | TEST-011 | TASK-GUIDE-003, TASK-DEPLOY-004 | CTRL-002, RISK-002, RISK-003 | Codex retarget validation; Ruby Psych | Inspect DISABLE job conditions and action pins; 16/16 guards and 39/39 pins; exit 0 | **PENDING** | EXECUTION_OBSERVATION | Local worktree | Embedded ledger `STATIC-001` | 2026-09-06T00:16:18Z | [NON-CANDIDATE OBSERVATION] Static guard proof passed; immutable candidate/runtime side-effect proof remains pending. |
| EVD-20260905-012 | N/A | `HEAD=d981de1229ef899957bbe968bc8dcda02a21f477`; uncommitted 15-path readiness delta | REQ-012, REQ-013 | TEST-012 | TASK-GUIDE-004, TASK-GUIDE-005 | CTRL-007, CTRL-010, RISK-010, RISK-012 | Codex retarget validation; local Git | `git diff --check`; strict changed-path allowlist; preserve upstream build smoke; exits 0 | **PENDING** | EXECUTION_OBSERVATION | Local worktree | Embedded ledger `SCOPE-001` | 2026-09-06T00:16:18Z | [NON-CANDIDATE OBSERVATION] Local scope passed; immutable candidate and independent review remain pending. |
| EVD-20260905-013 | N/A | `HEAD=d981de1229ef899957bbe968bc8dcda02a21f477`; 14 current-cycle records | REQ-014 | TEST-013 | TASK-GUIDE-004, TASK-GUIDE-005 | CTRL-010 | Codex retarget validation; Ruby | Validate ordered evidence IDs and 14-record cardinality; exit 0 | **PENDING** | EXECUTION_OBSERVATION | Local worktree | Embedded ledger `STATIC-001`; candidate/evidence revision model above | 2026-09-06T00:16:18Z | [NON-CANDIDATE OBSERVATION] Registry structure passed; no immutable candidate or candidate-keyed remote set exists. |
| EVD-20260905-014 | N/A | Merge authorization absent; post-merge state = `NOT_RUN` | REQ-011, REQ-014 | TEST-014 | TASK-DEPLOY-004, TASK-VERIFY-001 | CTRL-002, CTRL-010 | No runner; Repository Maintainer decision pending | Await separately authorized merge, then inspect result and effects | **PENDING** | HUMAN_DECISION | No merge authorization or execution | `docs/RUNBOOK.md`, TASK-DEPLOY-004 | 2026-09-06T00:08:10Z | [NON-CANDIDATE OBSERVATION] Retarget authorization does not authorize merge. |

## Current-cycle embedded command/result ledger

These summaries are repository inspection points for the current non-candidate observations. They
do not promote a candidate-dependent gate to PASS. Detailed temporary logs are diagnostic copies,
not the sole evidence reference.

| Reference | CWD and exact command/method | Exit/result | Observed UTC |
|---|---|---|---|
| `SOURCE-001` | CWD `/Users/mikkohchen/Developer/frictionless-labs/pi-production-readiness-v0851`; `git rev-parse refs/tags/frictionless-readiness-baseline-20260904^{}`; `git rev-parse v0.85.1^{}`; `GET /repos/earendil-works/pi/releases/tags/v0.85.1` | Git/API exit 0; archive ref equals `ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278`; target equals `d981de1229ef899957bbe968bc8dcda02a21f477`; release published 2026-09-05T12:29:01Z, non-draft, non-prerelease. | 2026-09-06T00:08:10Z |
| `ALIGN-001` | CWD `/Users/mikkohchen/Developer/frictionless-labs/pi-production-readiness-v0851`; `git merge-base --is-ancestor ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278 d981de1229ef899957bbe968bc8dcda02a21f477`; `git rev-list --count ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278..d981de1229ef899957bbe968bc8dcda02a21f477`; `git diff --shortstat ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278..d981de1229ef899957bbe968bc8dcda02a21f477`; `git rev-parse HEAD` | All exit 0; ancestry compatible; 720 commits; 883 files, 111675 insertions, 27866 deletions; readiness branch base equals target. The operation was branch creation at target, not fork-main synchronization. | 2026-09-06T00:08:10Z |
| `STATIC-001` | Same CWD; Ruby Psych validator parsed all workflow YAML and checked inventory/guards/pins/document schemas; `git diff --check` | Exit 0; 10 workflows, 2 KEEP/8 DISABLE, 16/16 upstream-only guards, 39/39 full-SHA pins, five valid frontmatters, 14 records, 15 task schemas. This result predates final-review edits and remains non-candidate evidence. | 2026-09-06T00:16:18Z |
| `SUPPLY-001` | Same CWD; `npm ci --ignore-scripts`; `npm audit --omit=dev --audit-level=high`; `npm audit signatures` | Each exit 0; 323 packages hydrated, 0 reported production vulnerabilities, 307 signatures and 60 attestations verified. | 2026-09-06T00:16:21Z–00:16:30Z |
| `SCAN-001` | Same CWD; official `gitleaks/gitleaks` v8.30.1 Darwin arm64 asset and checksum file downloaded with `gh release download`; archive verified with `shasum -a 256 -c -`; `gitleaks git` and `gitleaks dir` invoked with `--redact` on the baseline-to-target history, archived target content, and pre-final-review patch SHA-256 `3ea23af167c79cc377e9095d68ab8d90d8e44f4f890d3ad790a906a626cf126c` | Download, checksum, version, and three scans exit 0; 0 leaks. This does not cover later final-review edits or an immutable candidate. | 2026-09-06T00:22:35Z–00:22:39Z |
| `SERVER-001` | Fork API: `GET /repos/Frictionless-Labs/pi/actions/workflows`; readiness branch ref; Actions runs filtered by target SHA; `GET /repos/Frictionless-Labs/pi/branches/main/protection`; `GET /repos/Frictionless-Labs/pi/rulesets`; repository/security, Actions-permission, Dependabot-alert, secret-count, environment-count, and code-scanning routes | 10 workflows; readiness branch 404; target runs 0; `main` not protected (404); rulesets `[]`; Actions enabled/all and SHA pinning not required; listed scanning controls disabled; secrets/environments 0. Dependabot alerts 403 and code scanning 404 also reported missing `admin:repo_hook`; details are `PERMISSION`-limited. | 2026-09-06T00:18:28Z–00:18:31Z |
| `SCOPE-001` | Same CWD; `git diff --check`; `git status --porcelain=v1 -z --untracked-files=all` reconciled to the 15-path allowlist; `git diff --exit-code HEAD -- .github/workflows/npm-audit.yml` | Exit 0; exactly 15 allowed changed paths, no diff whitespace errors, `npm-audit.yml` byte-unchanged, v0.85.1 packed-consumer smoke retained. This result predates final-review edits and requires final repetition. | 2026-09-06T00:16:18Z |

## Current-cycle GitHub security and governance observation

This fork snapshot was captured from 2026-09-06T00:18:28Z through 00:18:31Z. It is current for the
v0.85.1 observation window but remains subject to recapture immediately before a settings or merge
decision.

| Control surface | Current observation | Readiness effect |
|---|---|---|
| Actions policy | Enabled; `allowed_actions=all`; `sha_pinning_required=false`; default workflow permission fields returned `null` | File-level least privilege, checkout hardening, and full-SHA pins remain required; server policy is not enforcement proof. |
| Workflows/runs | Ten workflows; readiness branch absent; target-SHA runs 0 | Candidate-source CI and runtime workflow-safety evidence are BLOCKED/PENDING. |
| Environments/secrets | Zero repository environments; zero repository Actions secrets | No inference about organization or other hidden credentials; upstream-specific workflows stay disabled. |
| Repository security settings | Dependabot security updates, secret scanning, non-provider patterns, validity checks, and push protection all reported disabled | Server-side security posture does not satisfy TEST-006/007. |
| Dependabot alerts | API returned `Dependabot alerts are disabled for this repository.` (HTTP 403) and reported missing `admin:repo_hook` | Disabled state observed; alert detail is `PERMISSION`-limited and cannot be called clear. |
| Code scanning | API returned `no analysis found` (HTTP 404) and reported missing `admin:repo_hook` | No analysis observed; configuration/alert detail is `PERMISSION`-limited. |
| Merge controls | Rulesets `[]`; `main` returned `Branch not protected` (HTTP 404) | TEST-010 is BLOCKED; human-approved governance is required before merge. |

## Superseded-cycle GitHub security and governance observations

These values were last observed on 2026-09-04 during the v0.85.0 cycle. They are historical risk
inputs only and do not replace the current-cycle snapshot above.

| Control surface | Last verified evidence | Readiness effect |
|---|---|---|
| Repository | Public; issues disabled; authenticated role reports admin capability | Public exposure raises workflow/secret review importance; role does not authorize settings mutation. |
| Actions policy | Enabled; all actions allowed; SHA pinning not required by server policy | Existing file-level full-SHA pins must remain; server policy is not an enforcement proof. |
| Workflows/runs | Ten active workflows; zero runs | Current-cycle static and remote evidence remain required. |
| Environments/secrets | Zero repository environments; zero repository Actions secrets | Upstream-named workflows cannot work as written; no inference about org/hidden secrets. |
| Dependabot/vulnerability alerts | Dependabot security updates disabled; `Vulnerability alerts are disabled. (HTTP 404)` | Remote dependency-security evidence is blocked/pending; npm audit gates still must run. |
| Secret protection | Secret scanning, non-provider patterns, validity checks, and push protection disabled | TEST-007 blocked until an approved scan runs; docs do not repair server posture. |
| Code scanning | No analysis observed; API access also reported missing `admin:repo_hook` scope | Mark PENDING; insufficient evidence to claim configured or clear. |
| Merge controls | Zero rulesets; `main` branch protection absent | TEST-010 blocked; human must establish/adjudicate governance before merge. |

## Superseded readiness calculation

| Measure | Current result | Basis |
|---|---|---|
| P0 design trace coverage | **100% (11/11) for the verified kickoff request and live repository source set** | `SPEC.md` defines and maps every current P0 REQ to FEAT, MOD, TASK, TEST/evidence, risk/control, and outcome. The two named Downloads documents were absent and remain **[UNVERIFIED]**. |
| P0 execution pass coverage | **27.3% (3/11)** | REQ-001/002/003 have current-cycle PASS evidence; local dependency, scanner, and control checks await immutable-candidate repetition. |
| Fork-local mandatory CI | **BLOCKED** | No v0.85.1 candidate or exact-SHA fork run exists. |
| Workflow safety | **PENDING** | Current-cycle static validation found 16/16 upstream-only guards; candidate and post-merge runtime evidence remain absent. |
| Remote security | **BLOCKED** | Local dependency/signature and redacted scanner checks passed, but no immutable candidate or candidate-keyed remote evidence exists. |
| Merge governance | **BLOCKED** | Current fork snapshot proves no `main` protection and zero repository rulesets. |
| Current readiness | **BLOCKED; confidence 97%** | Critical P0 execution evidence and merge controls are absent. |

## Superseded next dependency

Complete final review fixes and post-edit validation before creating the candidate-source revision.
The separate cycle authorization permits one reviewed candidate commit and one push to
`codex/pi-fork-production-readiness`; this specification grants no standing push authority. Repeat
candidate-keyed controls after that commit and record them in a later evidence-record revision. A
PR, merge, release, deployment, publication, settings change, and any later push remain unauthorized.
