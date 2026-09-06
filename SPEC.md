---
title: Pi Fork Production Readiness Specification
version: 1.4.0
status: READY
created_date: 2026-09-04
updated_date: 2026-09-06
tags:
  - pi
  - fork-governance
  - production-readiness
confidence: 0.99
owner: Frictionless Labs Repository Maintainer
---

# Pi Fork Production Readiness Specification

## Scope and live baseline

This specification governs the minimal readiness delta for public fork
`Frictionless-Labs/pi`, sourced from `earendil-works/pi`. Current-turn Repository Maintainer
authorization permits the bounded GitHub settings, evidence-only commits, readiness-branch pushes,
PR, merge, and verification needed to finish this cycle. Product changes remain out of scope except
for a narrow repair proven necessary by a failing mandatory gate. Credentials, release tags,
package/GitHub Release/model-catalog publication, R2 writes, and deployment remain out of scope.

| Fact | Current observed value | Authority |
|---|---|---|
| Pre-sync baseline | `ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278` | Local Git |
| Rollback ref | `frictionless-readiness-baseline-20260904` resolves to the baseline | Local Git |
| Frozen stable target | `v0.85.1` at `d981de1229ef899957bbe968bc8dcda02a21f477` | Git + GitHub release |
| Target release state | Published 2026-09-05T12:29:01Z; non-draft; non-prerelease | GitHub API |
| Candidate-source revision | `f2858ead77530b5bf86bd787b141a83e7a2daa41` | Local Git + fork GitHub API |
| Merged readiness head | `f2858ead77530b5bf86bd787b141a83e7a2daa41`; PR#6 merged 2026-09-06T05:55:38Z | Git + fork GitHub API |
| Fork `main` | Merged readiness head `f2858ead77530b5bf86bd787b141a83e7a2daa41`; upstream and fork commit identities preserved | Fork GitHub API |
| Baseline-to-target lineage delta | 720 commits; 883 files; 111675 insertions; 27866 deletions | Local Git |
| Required runtime | Node `>=22.19.0` | `package.json` |
| Observed runtime | macOS 26.6.2 arm64; Git 2.50.1; Node v26.7.0; npm 11.19.0 | Local CLI |
| Codex runtime | 0.145.0; `read-only`, `workspace-write`, `danger-full-access`; `untrusted`, `on-request`, `never` | Installed CLI help |
| Repository-local Codex config | None | Worktree scan |
| GitHub Actions state | Observed 2026-09-06T06:03Z: enabled; SHA pinning required; default token `read`; CI/audit active; eight upstream-only workflows `disabled_manually` | Fork GitHub API |
| GitHub governance | Observed 2026-09-06T06:03Z: protected `main`; four strict CI/audit/CodeQL checks; admin enforcement restored; PR-only, linear-history, conversation-resolution, no-force-push, no-delete; zero platform approvals because MIKKOH is the sole collaborator | Fork GitHub API |
| GitHub security | Observed 2026-09-06T04:47:15Z: vulnerability alerts, Dependabot security updates, secret scanning, and push protection enabled; paid-only non-provider and validity checks disabled; CodeQL default setup uses the extended suite on standard runners | Fork GitHub API |
| CodeQL evidence | Adoption run `34012647603` found 532 inherited alerts; PR#6 run `34014978444` succeeded with zero open PR alerts | Fork GitHub API |
| Post-merge verification | Exact-SHA CI `34015185239`, audit `34015193772`, and CodeQL `34015184916`; all completed successfully | Fork GitHub API |

## Source availability and authority

| Source | Cycle status | Use in this specification |
|---|---|---|
| User kickoff request, "PI Fork Production Readiness — Codex End-to-End Execution Megaprompt v1.0.0" | **[VERIFIED]** Read completely from the user-supplied attachment | Binding request for source precedence, execution doctrine, required artifacts, readiness gates, and identifier-chain shape. |
| Current repository, Git, CLI, workflow, package-registry, and fork API observations | **[VERIFIED]** Read or executed in this cycle | Authority for live implementation and evidence facts. |
| `strategy-pi-fork-production-readiness.md` in the named Downloads location | **[UNVERIFIED]** File absent when checked this cycle; content unread | No requirement, risk, KPI, or readiness claim is attributed to its unread content. |
| `execution-pack-pi-fork-production-readiness.md` in the named Downloads location | **[UNVERIFIED]** File absent when checked this cycle; content unread | No module, task, or validation claim is attributed to its unread content. |

The REQ/FEAT/RISK/CTRL/KPI ledger below is **[DERIVED]** from the verified kickoff request and live
repository evidence. Its coverage calculation proves internal coverage of that read source set; it
does not claim alignment with the two unavailable Downloads documents.

Installed Codex CLI help for version 0.145.0 exposes `untrusted`; no current official page was
verified for that specific token in this cycle. This project still recommends trusted
repository configuration with `sandbox_mode = "workspace-write"` and
`approval_policy = "on-request"`. The no-override product default for `codex exec` is read-only;
deterministic runs must pass explicit `--sandbox` and `--ask-for-approval` flags or record verified
effective configuration. `--full-auto` is deprecated, and `codex exec resume --last` is current
according to official documentation. No local `.codex/config.toml` is created by this task.

## Architecture and shared contracts

````mermaid
flowchart TD
    M1[MOD-01 Provenance] --> M2[MOD-02 Synchronization]
    M2 --> M3[MOD-03 Workflow Safety]
    M1 --> M5[MOD-05 Trust Policy]
    M1 --> M6[MOD-06 Codex Control Plane]
    M3 --> M4[MOD-04 CI and Security Evidence]
    M5 --> M4
    M6 --> M4
    M4 --> M7[MOD-07 Governance and Decision]
````

| Contract | Required fields | Owner | Invariant |
|---|---|---|---|
| `ForkBaseline` | repository, SHA, archive ref, captured UTC | Git | Archive ref is non-`v*`, immutable, and resolves to SHA. |
| `UpstreamTarget` | repository, tag, SHA, published UTC, stable flags, observed UTC | Git/GitHub | Frozen after GUIDE begins; tag is immutable and stable. |
| `TargetAlignment` | baseline SHA, target SHA, branch base SHA, alignment operation, ancestry result, delta, observed UTC | Git | Readiness work begins on a new isolated branch created at the frozen target; fork `main` is not changed by this observation. |
| `WorkflowDisposition` | path, trigger, permissions, secrets, environments, effects, disposition, validation | Repository docs | Exactly one record exists per workflow. |
| `TrustClassification` | source class, boundary, network, credentials, mounts, retention | Fork policy | Untrusted execution never uses the ordinary credential-bearing user boundary. |
| `EvidenceRecord` | ID, candidate SHA, REQ, TEST, TASK, control, method, result, source, UTC, notes | Evidence registry | Result is never PASS without an observed gate on the named SHA. |
| `ReadinessDecision` | status, candidate SHA, blockers, evidence set, human approver, UTC | Human maintainer | READY requires every mandatory gate and explicit human GO. |

Git owns refs and code state. GitHub owns workflow runs and repository settings. Repository docs own
policy and evidence records. The Repository Maintainer owns irreversible decisions and final GO.

## MOD-01 — Provenance and Upstream Selection

| Field | Specification |
|---|---|
| Requirements | REQ-001, REQ-002 |
| Inputs | Fork identity, HEAD, remotes, release metadata, archive-ref lookup. |
| Outputs | `ForkBaseline`, `UpstreamTarget`, TEST-001/002 evidence. |
| Contract | Inspect is read-idempotent. Archive-ref creation is idempotent only when the existing ref resolves to the exact baseline. Target must be a resolvable non-draft/non-prerelease release. |
| State ownership | Git owns the archive ref; GitHub owns release metadata. Baseline is write-once. Target may change only before synchronization and human target freeze. |
| Dependencies | Local Git objects, read access to public upstream release metadata. |
| Edge cases | Dirty worktree, detached HEAD, missing/moved tag, archive collision, API outage, a newer release appearing mid-cycle. |
| Errors | Mismatched archive ref or unresolvable tag is `BLOCK_RELEASE`; API outage is `ENVIRONMENT`; dirty unrelated work is `PROVE_NOW` and stops mutation. |
| Security | No credentials are required. No archive-ref push or `v*` readiness tag is allowed. |
| Observability | Record remotes, branch, status, full SHAs, stable flags, publish/observation UTC, and ref resolution. |
| Tests | TEST-001, TEST-002. |

**Exact acceptance**

- `origin` is `Frictionless-Labs/pi`; upstream source is `earendil-works/pi`.
- `frictionless-readiness-baseline-20260904^{}` equals
  `ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278` and the name does not match `v*`.
- `v0.85.1^{}` equals `d981de1229ef899957bbe968bc8dcda02a21f477`; release metadata reports
  `draft=false` and `prerelease=false`.
- This cycle supersedes v0.85.0 because v0.85.1 fixes the published 0.85.0 SDK import failure
  caused by unintended experimental code and dependencies; the maintainer explicitly authorized
  the return to ASSESS and retarget on 2026-09-05.
- Any later target change after GUIDE begins stops execution and returns to ASSESS.

**Done gate:** TEST-001 and TEST-002 are PASS on current evidence before MOD-02 proceeds.

## MOD-02 — Target Alignment and Delta Control

| Field | Specification |
|---|---|
| Requirements | REQ-003, REQ-013 |
| Inputs | Accepted baseline, frozen target, status, ancestry, and diff evidence. |
| Outputs | `TargetAlignment`, target-aligned branch, explicit fork-delta boundary. |
| Contract | `merge-base --is-ancestor(baseline,target)` must succeed before target alignment. This cycle creates a new isolated readiness branch directly at the frozen target; it does not synchronize fork `main`. A later fork-main transition must preserve the proven ancestry and use its separately authorized merge path. |
| State ownership | The readiness branch owns mutable work; the archive ref remains immutable. |
| Dependencies | MOD-01 and a preservation-safe worktree. |
| Edge cases | Non-ancestor target, ref unavailable locally, colliding untracked file, concurrent same-file edit, branch already exists. |
| Errors | Non-ancestor is `ARCHITECTURE`; collision is `INTRODUCED` or `PRE-EXISTING` based on evidence; inability to preserve work is `BLOCK_RELEASE`. |
| Security | Never stash, clean, broad-restore, force-push, or rewrite history. No remote mutation is implicit. |
| Observability | Record before/after SHA, ancestor exit code, operation, commit/file/line delta, and fork-only diff. |
| Tests | TEST-003, TEST-012. |

**Exact acceptance**

- Baseline is an ancestor of target and current pre-readiness HEAD equals target.
- The observed operation began with target-aligned branch creation at `v0.85.1`; the preserved
  baseline is an ancestor of that target with a 720-commit, 883-file lineage delta. The authorized
  integration then fast-forwarded `main` exactly to the target, the readiness controls, and the
  repaired PR#6 head `f2858ead...` without rewriting any commit identity.
- All fork readiness edits are attributable after target alignment and remain within approved paths.
- Any required product-code edit must be separately justified by a failing mandatory gate.

**Done gate:** TEST-003 is PASS and the current fork-specific diff is scope-reviewed before MOD-03.

## MOD-03 — Workflow Safety Layer

| Field | Specification |
|---|---|
| Requirements | REQ-004, REQ-011 |
| Inputs | All `.github/workflows/*.yml`, GitHub server workflow state, triggers, permissions, secrets, environments, effects. |
| Outputs | Ten `WorkflowDisposition` records and TASK-GUIDE-003 identity-guard implementation evidence. |
| Contract | `ci.yml` and `npm-audit.yml` are KEEP. The other eight workflows are DISABLE in the fork through a repository-identity guard on every job. Unknown ownership never becomes ADAPT. |
| State ownership | Workflow YAML owns executable job conditions; GitHub owns active/disabled server state; `WORKFLOW_DISPOSITION.md` owns policy. |
| Dependencies | MOD-02, complete workflow inventory, TASK-GUIDE-003 implementation. |
| Edge cases | Scheduled triggers, `workflow_run`, `pull_request_target`, manual publish inputs, job dependency chains, absent repository secrets, hidden organization secrets. |
| Errors | Missing inventory row is `INTRODUCED`; an unguarded mutating job is `BLOCK_RELEASE`; unavailable upstream credential is `PERMISSION`, never a reason to substitute credentials. |
| Security | Preserve full-SHA action pins and least privilege. Never add publication credentials or expose secret values. |
| Observability | Record file count, guard checks, trigger tests, active server state, run list, and post-merge side-effect evidence. |
| Tests | TEST-004, TEST-011, TEST-014. |

**Exact acceptance**

- Workflow disposition row count equals the ten current workflow files with no duplicate path.
- Only the two KEEP workflows can start jobs in `Frictionless-Labs/pi`.
- Each DISABLE workflow keeps its upstream behavior available only when repository identity is
  `earendil-works/pi`; no fork-owned publication credential is introduced.
- No readiness tag matches `v*`; no external publication or issue/PR mutation runs in the fork.

**Done gate:** TASK-GUIDE-003 guard diff and TEST-004/011 must PASS before candidate workflows execute.

## MOD-04 — CI and Security Evidence Layer

| Field | Specification |
|---|---|
| Requirements | REQ-005, REQ-006, REQ-007 |
| Inputs | Guarded candidate SHA, KEEP workflows, canonical commands, approved secret scanner. |
| Outputs | Candidate-keyed CI, vulnerability, registry-signature, and secret-scan `EvidenceRecord` values. |
| Contract | CI PASS requires build/check/test success on the exact candidate SHA. Audit PASS requires both repository commands exit 0. Secret PASS requires zero validated live credentials in the defined candidate/diff scope. |
| State ownership | GitHub Actions owns authoritative remote run results; the evidence registry indexes immutable results. |
| Dependencies | MOD-03, MOD-05, MOD-06, runner and npm availability. |
| Edge cases | Zero run history, runner/registry outage, flaky test, advisory database change, disabled secret scanning, pre-existing upstream finding. |
| Errors | Classify as `INTRODUCED`, `PRE-EXISTING`, `ENVIRONMENT`, `DEPENDENCY`, `ARCHITECTURE`, or `PERMISSION`; do not weaken a gate. |
| Security | Install with lifecycle scripts disabled. Use faux providers. Redact findings and never persist secret values. |
| Observability | Record workflow/run/job ID, SHA, command, exit/conclusion, timestamps, redacted finding count, remediation, rerun, and waiver owner/expiry. |
| Tests | TEST-005, TEST-006, TEST-007. |

**Exact acceptance**

- `ci.yml` starts from an authorized push of the immutable candidate to
  `codex/pi-fork-production-readiness`, which is the only added push trigger. Manual dispatch may
  also run CI against a selected ref, but supplies TEST-005 evidence only when its workflow source,
  checkout SHA, and run SHA each equal the immutable candidate and build, check, and test succeed.
- The KEEP CI workflow declares workflow-level `contents: read`, and checkout does not persist the
  workflow token in the local Git configuration.
- `npm audit --omit=dev --audit-level=moderate` and `npm audit signatures --omit=dev` exit 0,
  or an explicitly authorized non-release-blocking waiver is recorded; signatures cannot be waived.
- An approved secret scan reports zero validated live credentials introduced or exposed by the
  readiness delta. Disabled GitHub secret scanning is BLOCKED, not PASS.
- Pre-existing or environmental failures remain non-PASS until the defined repair or waiver exists.

**Done gate:** TEST-005, TEST-006, and TEST-007 are PASS before final readiness review.

## MOD-05 — Trust Boundary and Operator Policy

| Field | Specification |
|---|---|
| Requirements | REQ-008 |
| Inputs | `SECURITY.md`, repository/extension/skill provenance, intended mounts, credentials, network policy. |
| Outputs | `TrustClassification`, isolation decision, operator evidence. |
| Contract | Trusted repository plus trusted extensions/skills may use an approved development environment. Any unknown/untrusted component or prompt-supplied executable content requires disposable isolation. |
| State ownership | `FORK_POLICY.md` owns the policy; the operator owns each run classification; the isolation platform owns runtime enforcement. |
| Dependencies | Reviewed inputs and an available container, VM, or equivalent verified disposable boundary. |
| Edge cases | Trusted repo loads unknown extension, shared SSH agent, writable home mount, symlink escape, host Docker socket, network egress, persistent cache. |
| Errors | Unknown classification defaults to untrusted. Missing isolation is `BLOCK_RELEASE` for that run. Boundary escape is a security incident. |
| Security | No ambient SSH/cloud/model/production credentials, sensitive home data, unrelated writable repos, privileged device/socket, or reusable secret-bearing volume. |
| Observability | Record classification, isolation mechanism/version, mounts, credential posture, network posture, disposal result, and reviewer. |
| Tests | TEST-008. |

**Exact acceptance**

- Policy states Pi is not itself a sandbox and identifies trusted, unknown, and untrusted classes.
- Every unknown/untrusted run uses disposable isolation with the prohibited ambient surfaces absent.
- Mixed-trust inputs take the least-trusted classification; failure to prove a boundary prevents run.

**Done gate:** TEST-008 passes through policy review and one independently evidenced operator
classification before untrusted Pi use.

## MOD-06 — Codex Control Plane

| Field | Specification |
|---|---|
| Requirements | REQ-009, REQ-013 |
| Inputs | Current root `AGENTS.md`, this spec, runbook, installed Codex help, official Codex documentation, repository rules. |
| Outputs | Conservatively merged `AGENTS.md`, complete `SPEC.md`, deterministic `docs/RUNBOOK.md`. |
| Contract | Deeper valid `AGENTS.md` instructions take precedence. Recommended mutation mode is trusted project configuration plus `workspace-write` and `on-request`; no fabricated flags or Claude-only commands. |
| State ownership | Repository owns versioned controls; user config remains user-owned; no repository `.codex/config.toml` is implied. |
| Dependencies | MOD-01 facts, official docs, installed CLI corroboration, current repository instructions. |
| Edge cases | CLI/doc version drift, nested future `AGENTS.md`, dirty tree, approval unavailable, task brief lacks an allowed surface. |
| Errors | Contradiction is recorded and resolved by source precedence; unverified mechanic is marked `PROVE_NOW`; scope breach is `BLOCK_RELEASE`. |
| Security | Never use `danger-full-access` or approval bypass for routine readiness; treat project config as trusted code; preserve user work. |
| Observability | Record CLI version/help, official-doc correction, config presence, changed paths, validation results, and unrun gates. |
| Tests | TEST-009, TEST-012. |

**Exact acceptance**

- All three control-plane artifacts exist, agree on source, target, workflow policy, trust policy,
  evidence semantics, failure classes, rollback, and human gates.
- Every runbook task includes all required deterministic brief fields and exact validation.
- Artifact checks find all MOD, REQ, TEST, TASK, VIZ-07, and VIZ-08 identifiers with no stale
  claim that `untrusted` was removed.
- The readiness delta contains one mandatory-gate-driven product repair, two security lockfile
  repairs, and no generated or Git metadata edit; workflow edits remain limited to reviewed job
  guards plus the KEEP CI trigger and token hardening.

**Done gate:** TEST-009 and documentation-scope validation PASS before TASK-GUIDE-003 execution.

## MOD-07 — Governance and Readiness Evidence

| Field | Specification |
|---|---|
| Requirements | REQ-010, REQ-012, REQ-014 |
| Inputs | Fork policy, workflow dispositions, all TEST evidence, branch/ruleset state, review findings, human decisions. |
| Outputs | Evidence registry, `ReadinessDecision`, rollback/recovery and final handoff. |
| Contract | Candidate execution evidence is keyed to an immutable candidate-source SHA. The Git commit/blob containing the registry row is a separate evidence-record revision and may postdate that source. READY requires 100% mandatory P0 PASS, no P0/P1 blocker, confidence >=98%, independent review, verified merge controls, and human GO. |
| State ownership | Repository docs own records; GitHub owns settings; human maintainer owns adjudication and merge. |
| Dependencies | MOD-01 through MOD-06 and access to relevant GitHub evidence. |
| Edge cases | No rulesets/protection, stale run on another SHA, partial permission, contradictory reviewer findings, merge changes SHA, post-merge workflow trigger. |
| Errors | Unknown setting remains PENDING/BLOCKED. Candidate mismatch invalidates evidence. Any unintended external effect triggers incident containment. |
| Security | Docs store identifiers and redacted evidence only. Settings, merge, push, publication, credential, and rollback mutations remain human-only unless separately authorized. |
| Observability | Record source/actor, exact SHA/ref, UTC, method, result, evidence reference, blocker, decision owner, and next action. |
| Tests | TEST-010, TEST-012, TEST-013, TEST-014. |

**Exact acceptance**

- Fork purpose, cadence, delta ownership, trust classes, merge policy, recovery, and human-only
  operations are documented and reviewed.
- Every TEST-001 through TEST-014 has a current candidate-keyed record; PENDING/BLOCKED is explicit.
- Branch/ruleset/required-check evidence is verified or blocks human GO; absence is not inferred safe.
- After authorized merge, `main` and expected candidate relationship is proven and no unintended
  release, npm, model-catalog, R2, issue, PR, or comment mutation occurred. Dependabot's expected
  auto-closure of redundant PR#1/PR#2 and its two closure comments are explicitly attributed.

**Done gate:** PASS. The Repository Maintainer gave explicit GO; the failed terminal-event gate was
repaired, all mandatory gates and PR#6 CodeQL passed, and exact fast-forward integration plus
post-merge verification completed successfully.

## Test catalog

| Test | Observable contract |
|---|---|
| TEST-001 | Archive ref is non-release-named and resolves exactly to the pre-sync baseline SHA. |
| TEST-002 | Selected upstream release is current at freeze, immutable, non-draft, non-prerelease, and resolves to the recorded SHA. |
| TEST-003 | Baseline is an ancestor of target; the isolated readiness branch was created at the exact target without changing fork `main` or using merge/rebase/reset. |
| TEST-004 | Workflow inventory has exactly one safe disposition for every current workflow and enforced job guard for every DISABLE workflow. |
| TEST-005 | Fork-local canonical CI succeeds on the exact candidate SHA. |
| TEST-006 | Production vulnerability audit and registry-signature audit succeed under the recorded policy. |
| TEST-007 | Approved scan finds zero validated live credentials introduced/exposed by the readiness delta. |
| TEST-008 | Policy and operator evidence enforce disposable isolation for unknown/untrusted Pi inputs. |
| TEST-009 | `AGENTS.md`, `SPEC.md`, and `docs/RUNBOOK.md` exist, agree, and use verified Codex semantics. |
| TEST-010 | Branch protection/rulesets, required checks, review and bypass policy are observed and satisfy approved merge governance. |
| TEST-011 | Candidate cannot execute unauthorized upstream publication or organization-maintenance jobs in the fork. |
| TEST-012 | Fork purpose/recovery are documented and candidate diff contains only approved minimal readiness surfaces. |
| TEST-013 | Every required gate has candidate SHA, method, UTC, actor/source, result, and evidence reference. |
| TEST-014 | Post-merge evidence proves intended `main` state and no unintended external workflow effect. |

## Identifier and source ledger

All identifiers used by this control plane are defined here. **[DERIVED]** means the definition is
derived from the verified kickoff request plus the named live repository evidence; it is not
attributed to either unavailable Downloads document.

### Requirements

| Identifier | Canonical meaning | Source label |
|---|---|---|
| REQ-001 | Preserve and verify the exact pre-readiness baseline at an immutable non-release archive ref. | **[DERIVED]** Kickoff §§2, 5 + local Git refs. |
| REQ-002 | Select, verify, and freeze one immutable stable upstream release. | **[DERIVED]** Kickoff §§1, 5 + upstream release/tag evidence. |
| REQ-003 | Prove baseline-to-target ancestry and create the isolated readiness branch at the exact target without changing fork `main`. | **[DERIVED]** Kickoff §§5–6 + local Git/worktree evidence. |
| REQ-004 | Classify every workflow and enforce the approved fork disposition. | **[DERIVED]** Kickoff §7 + current workflow inventory. |
| REQ-005 | Obtain canonical fork-local CI on the exact immutable candidate-source SHA. | **[DERIVED]** Kickoff §§9–10 + `ci.yml`. |
| REQ-006 | Prove dependency vulnerability and registry-signature gates on the candidate source. | **[DERIVED]** Kickoff §§9–10, 13 + npm policy/workflow. |
| REQ-007 | Run an approved redacted secret scan against the exact candidate source and defined history/content scope. | **[DERIVED]** Kickoff §§2.5, 9–10, 13. |
| REQ-008 | Define and evidence disposable isolation for unknown or untrusted Pi execution. | **[DERIVED]** Kickoff §8 + `SECURITY.md`. |
| REQ-009 | Maintain current `AGENTS.md`, complete `SPEC.md`, and deterministic `docs/RUNBOOK.md` with verified Codex semantics. | **[DERIVED]** Kickoff §§3–4 + current instructions/CLI evidence. |
| REQ-010 | Observe and approve branch/ruleset/review/bypass governance before merge. | **[DERIVED]** Kickoff §11 + fork API evidence. |
| REQ-011 | Prevent unauthorized publication and organization-maintenance workflow execution in the fork. | **[DERIVED]** Kickoff §§2.6–2.7, 7 + workflow inspection. |
| REQ-012 | Document fork purpose, owned delta, recovery, and human-only boundaries. | **[DERIVED]** Kickoff §§2, 13, 17–20. |
| REQ-013 | Keep the candidate within the reviewed minimal path scope and attribute every result to exact state. | **[DERIVED]** Kickoff §§2.3, 6, 14, 16. |
| REQ-014 | Maintain durable readiness evidence, independent review, human decisions, and post-merge effect verification. | **[DERIVED]** Kickoff §§12, 14, 18–20. |

### Capabilities

| Identifier | Canonical meaning | Source label |
|---|---|---|
| FEAT-001 | Provenance and rollback-point verification. | **[DERIVED]** REQ-001 and MOD-01. |
| FEAT-002 | Stable target selection and target-aligned branch provenance. | **[DERIVED]** REQ-002/003 and MOD-01/02. |
| FEAT-003 | Complete workflow disposition and upstream-only job enforcement. | **[DERIVED]** REQ-004/011 and MOD-03. |
| FEAT-004 | Candidate-keyed CI, dependency, signature, and secret evidence. | **[DERIVED]** REQ-005/006/007 and MOD-04. |
| FEAT-005 | Trust classification and disposable isolation policy. | **[DERIVED]** REQ-008 and MOD-05. |
| FEAT-006 | Repository-native Codex control plane and deterministic validation. | **[DERIVED]** REQ-009/013 and MOD-06. |
| FEAT-007 | Merge governance, evidence reconciliation, recovery, and human readiness decision. | **[DERIVED]** REQ-010/012/014 and MOD-07. |

### Risks and controls

| Identifier | Canonical meaning | Source label |
|---|---|---|
| RISK-001 | Stale, moved, or wrongly selected upstream target invalidates provenance. | **[DERIVED]** Kickoff §§1, 5, 19. |
| RISK-002 | An upstream publication or maintenance workflow executes in the fork. | **[DERIVED]** Kickoff §§2.6–2.7, 7. |
| RISK-003 | Workflow privilege, credentials, or external effects escape the approved fork boundary. | **[DERIVED]** Kickoff §§2.5–2.6, 7. |
| RISK-004 | Untrusted Pi inputs access ambient credentials, home data, repositories, network, or privileged host resources. | **[DERIVED]** Kickoff §8. |
| RISK-005 | Evidence runs against a different ref/SHA or an unproven lineage. | **[DERIVED]** Kickoff §§5–6, 10, 14. |
| RISK-006 | Missing merge controls or unauthorized bypass permits unreviewed integration. | **[DERIVED]** Kickoff §§11, 13. |
| RISK-007 | Dependency, registry, or credential integrity failure reaches the candidate. | **[DERIVED]** Kickoff §§2.4–2.5, 9. |
| RISK-008 | Local/upstream/flaky CI is misrepresented as exact fork-candidate evidence. | **[DERIVED]** Kickoff §§9–10. |
| RISK-009 | Stale or fabricated Codex mechanics weaken scope, sandbox, or approval controls. | **[DERIVED]** Kickoff §§0, 3–4. |
| RISK-010 | Destructive Git action or unexplained path drift loses work or obscures the candidate. | **[DERIVED]** Kickoff §§2.2–2.3, 5–6. |
| RISK-011 | A stable release contains a material unresolved compatibility or security defect. | **[DERIVED]** Kickoff §§1, 5, 19. |
| RISK-012 | Recovery or post-merge verification cannot contain or attribute an external effect. | **[DERIVED]** Kickoff §§17–20. |
| CTRL-001 | Exact ref/release resolution, stability review, freeze, and archive-ref verification. | **[DERIVED]** Kickoff §§5–6. |
| CTRL-002 | Exact upstream repository-identity guard on every DISABLE workflow job. | **[DERIVED]** Kickoff §7 + current guard design. |
| CTRL-004 | Checksum-verified approved scanner, redacted output, and exact candidate/history/content scope. | **[DERIVED]** Kickoff §§2.5, 9, 14. |
| CTRL-005 | Disposable isolation with prohibited ambient surfaces absent and disposal evidenced. | **[DERIVED]** Kickoff §8. |
| CTRL-006 | One reconciled disposition record for every current workflow. | **[DERIVED]** Kickoff §7. |
| CTRL-007 | Fresh preflight, instruction precedence, ancestry, exact path allowlist, and candidate-state checks. | **[DERIVED]** Kickoff §§0, 2, 5, 16. |
| CTRL-008 | Canonical fork CI with workflow source, checkout, and run SHA equal to the candidate source. | **[DERIVED]** Kickoff §10 + `ci.yml`. |
| CTRL-009 | Script-disabled install plus repository-proven vulnerability and registry-signature commands. | **[DERIVED]** Kickoff §§2.4–2.5, 9. |
| CTRL-010 | Durable evidence-record revisions, independent review, explicit human decisions, and effect audit. | **[DERIVED]** Kickoff §§12, 14, 18. |
| CTRL-011 | Observed and approved branch/ruleset/check/review/bypass/force-push governance. | **[DERIVED]** Kickoff §11. |

### Outcomes

| Identifier | Canonical measurable outcome | Source label |
|---|---|---|
| KPI-001 | Every required gate has exact state, method, actor/source, UTC, result, and durable reference. | **[DERIVED]** Kickoff §§14, 18. |
| KPI-002 | Target branch base equals the frozen target and baseline ancestry is proven, with no fork-main mutation. | **[DERIVED]** Kickoff §§5–6. |
| KPI-003 | Ten workflows have one disposition each; all 16 DISABLE jobs are upstream-only. | **[DERIVED]** Kickoff §7 + current workflow inventory. |
| KPI-004 | Canonical fork CI succeeds with workflow source, checkout, and run SHA equal to the candidate source. | **[DERIVED]** Kickoff §10. |
| KPI-005 | Candidate vulnerability and registry-signature commands exit 0 under the recorded policy. | **[DERIVED]** Kickoff §§9, 13. |
| KPI-006 | Approved redacted scan reports zero validated live credentials in the defined candidate scope. | **[DERIVED]** Kickoff §§2.5, 9, 13. |
| KPI-007 | Required control-plane artifacts and task schemas are complete, consistent, and scope-valid. | **[DERIVED]** Kickoff §§3–4, 20. |
| KPI-008 | One actual unknown/untrusted Pi run proves the required isolation posture and disposal. | **[DERIVED]** Kickoff §§8, 13. |

## P0 traceability

Within the **[DERIVED]** request-and-repository ledger above, P0 requirements are REQ-001 through
REQ-009, REQ-011, and REQ-013. The table maps all 11, so internal design trace coverage for the
actually read sources is 11/11 = 100%; alignment with the two unavailable Downloads documents is
**[UNVERIFIED]**. Candidate CI, audit, secret scan, isolation, governance, CodeQL, PR merge, and
post-merge verification have direct evidence. Mandatory P0 execution coverage is complete.

### VIZ-08 — Requirement-to-outcome traceability

````mermaid
flowchart LR
    R[REQ requirement] --> F[FEAT capability]
    F --> M[MOD contract]
    M --> T[TASK deterministic work]
    T --> E[TEST or evidence]
    E --> O[KPI or technical outcome]
    K[RISK] --> C[CTRL]
    C --> T
    E --> H[Human readiness gate]
````

Alt: each requirement maps through a feature, module, deterministic task, evidence, safety control,
and measurable outcome before the human readiness gate.

| REQ | Priority | FEAT | MOD | TASK | TEST/evidence | RISK / CTRL | KPI or criterion |
|---|---|---|---|---|---|---|---|
| REQ-001 | P0 | FEAT-001 | MOD-01 | TASK-SCAN-001, TASK-GUIDE-001 | TEST-001 | RISK-010 / CTRL-001, CTRL-007 | KPI-001 |
| REQ-002 | P0 | FEAT-002 | MOD-01 | TASK-ASSESS-001 | TEST-002 | RISK-001, RISK-011 / CTRL-001 | KPI-002 |
| REQ-003 | P0 | FEAT-002 | MOD-02 | TASK-GUIDE-002 | TEST-003 | RISK-001, RISK-005 / CTRL-007 | KPI-002 |
| REQ-004 | P0 | FEAT-003 | MOD-03 | TASK-ASSESS-002, TASK-GUIDE-003 | TEST-004 | RISK-003 / CTRL-002, CTRL-006 | KPI-003 |
| REQ-005 | P0 | FEAT-004 | MOD-04 | TASK-GUIDE-005, TASK-DEPLOY-001 | TEST-005 | RISK-005, RISK-008 / CTRL-008 | KPI-004 |
| REQ-006 | P0 | FEAT-004 | MOD-04 | TASK-DEPLOY-002 | TEST-006 | RISK-007 / CTRL-009 | KPI-005 |
| REQ-007 | P0 | FEAT-004 | MOD-04 | TASK-DEPLOY-002 | TEST-007 | RISK-003, RISK-007 / CTRL-004 | KPI-006 |
| REQ-008 | P0 | FEAT-005 | MOD-05 | TASK-STRATEGIZE-002 | TEST-008 | RISK-004 / CTRL-005 | KPI-008 |
| REQ-009 | P0 | FEAT-006 | MOD-06 | TASK-SCAN-001, TASK-STRATEGIZE-001, TASK-GUIDE-004 | TEST-009 | RISK-009 / CTRL-007 | KPI-007 |
| REQ-010 | P1 | FEAT-007 | MOD-07 | TASK-DEPLOY-003 | TEST-010 | RISK-006 / CTRL-011 | Verified merge-control criterion |
| REQ-011 | P0 | FEAT-003 | MOD-03 | TASK-GUIDE-003, TASK-DEPLOY-004 | TEST-011, TEST-014 | RISK-002, RISK-003, RISK-012 / CTRL-002 | KPI-003 |
| REQ-012 | P1 | FEAT-007 | MOD-07 | TASK-STRATEGIZE-001, TASK-GUIDE-004 | TEST-012 | RISK-001, RISK-012 / CTRL-010 | Reviewed governance criterion |
| REQ-013 | P0 | FEAT-002, FEAT-006 | MOD-02, MOD-06 | TASK-GUIDE-002, TASK-GUIDE-004 | TEST-012 | RISK-005, RISK-010 / CTRL-007 | Minimal-diff criterion |
| REQ-014 | P1 | FEAT-007 | MOD-07 | TASK-GUIDE-004, TASK-DEPLOY-004 | TEST-013 | RISK-006 / CTRL-010 | KPI-001 evidence integrity |
