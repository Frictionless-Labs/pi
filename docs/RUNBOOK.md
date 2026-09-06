---
title: Pi Fork Production Readiness Runbook
version: 1.4.0
status: READY
created_date: 2026-09-04
updated_date: 2026-09-06
tags:
  - pi
  - asgd
  - runbook
confidence: 0.99
owner: Frictionless Labs Repository Maintainer
---

# Pi Fork Production Readiness Runbook

## Execution wrapper

**Terminal Command**

````bash
cd /Users/mikkohchen/Developer/frictionless-labs/pi-production-readiness-v0851
````

**Session Title:** `CODEX_PI_FORK_READINESS_<TASK-ID>`

**Runbook objective:** Execute SCAN → ASSESS → STRATEGIZE → GUIDE → DEPLOY → VERIFY without
skipping gates, widening a task surface, or treating local/static evidence as remote/runtime proof.

**Allowed paths:** Only the active task's named surface. If the task does not name a write surface,
it is read-only.

**Global stop condition:** `CODEX_PI_FORK_READINESS_BLOCKED_<REASON>` on wrong repository, wrong
branch, unrelated dirty state that overlaps the task, unsafe external effect, failed ancestry,
secret exposure, scope breach, destructive requirement, or unowned decision.

**After completion**

| Outcome | Next action |
|---|---|
| Task PASS and next dependency satisfied | Start only the next task in this runbook after fresh preflight. |
| `INTRODUCED` failure | Repair the smallest root cause; rerun narrow then required broader validation. |
| `PRE-EXISTING` failure | Prove independently; record it; do not call the gate PASS or mask it. |
| `ENVIRONMENT` or `DEPENDENCY` failure | Record reproduction; restore the required environment/service without source churn. |
| `ARCHITECTURE` or `PERMISSION` failure | Return to ASSESS/STRATEGIZE and obtain the named human decision. |
| Authorized settings, evidence push, PR, or merge step | Execute only the bounded current-cycle operation; reverify state and evidence immediately after. |
| Credential, release, publication, deployment, or destructive step | Stop at the human gate; these remain outside current authorization. |

## Deterministic phase gates

| Phase | Entry | Required output | Exit gate |
|---|---|---|---|
| SCAN | Repository read access | Current local/runtime/repository truth | Identity, work state, tools, sources, and rollback candidate known. |
| ASSESS | SCAN PASS | Frozen target and complete workflow/security risk disposition | Stable target and P0 trace are decision-complete. |
| STRATEGIZE | ASSESS PASS | Complete contracts, task briefs, trust and fork policy | Every P0 maps REQ → FEAT → MOD → TASK → TEST/control/outcome. |
| GUIDE | STRATEGIZE PASS | Preserved baseline, target alignment, guards, docs, local validation | Minimal candidate exists; unsafe workflows cannot run in fork. |
| DEPLOY | GUIDE PASS | Candidate-SHA CI/security/governance evidence | Every mandatory gate PASS and human GO recorded. |
| VERIFY | DEPLOY evidence available | Independent findings, post-merge evidence, readiness decision | READY only at >=98% with no P0/P1 blocker. |

## VIZ-07 — Execution, validation, failure, rollback, and human gate

````mermaid
flowchart TD
    S[SCAN] --> A[ASSESS]
    A --> ST[STRATEGIZE]
    ST --> G[GUIDE]
    G --> L0[L0 Static]
    L0 --> L1[L1 Targeted]
    L1 --> L2[L2 Module]
    L2 --> L3[L3 Integration]
    L3 --> L4[L4 Repository]
    L4 --> L5[L5 Release and Security]
    L5 --> L6[L6 Runtime]
    L6 --> D[DEPLOY evidence]
    D --> V[VERIFY independent review]
    V --> H{Human GO?}
    H -->|yes, all gates pass| M[Authorized merge]
    M --> P[Post-merge TEST-014]
    H -->|no| B[BLOCKED or CONDITIONAL]
    L0 -. failure .-> F{Classify}
    L1 -. failure .-> F
    L2 -. failure .-> F
    L3 -. failure .-> F
    L4 -. failure .-> F
    L5 -. failure .-> F
    L6 -. failure .-> F
    F -->|INTRODUCED| R[Smallest repair and rerun]
    F -->|PRE-EXISTING| E[Prove and record]
    F -->|ENVIRONMENT or DEPENDENCY| ENV[Correct environment or wait]
    F -->|ARCHITECTURE or PERMISSION| A
    F -->|destructive or external| H
    R --> L0
    E --> B
    ENV --> L0
    P -->|failure| RB[Human-authorized rollback or forward fix]
    RB --> A
````

Alt: phases proceed deterministically through escalating validation and independent review; every
failure is classified, repair or rollback loops back to an earlier gate, and merge stays human-only.

## Validation ladder

| Level | Proves | Escalation rule |
|---|---|---|
| L0 Static | Markdown/YAML shape, workflow syntax, identifiers, scope, generated-file absence | Must pass before targeted execution. |
| L1 Targeted | Changed artifact's observable contract | Run the narrowest repository-approved check. |
| L2 Module | Affected package/module behavior | Required for code behavior changes; docs-only tasks record N/A. |
| L3 Integration | Cross-module or workflow contract | Required when interfaces or automation change. |
| L4 Repository | Canonical check/test/build policy | Code changes use repository commands; CI is authoritative where local commands are prohibited. |
| L5 Release/Security | Permissions, pins, dependency, signatures, secrets, publication quarantine | Any unknown is BLOCKED/PENDING. |
| L6 Runtime | Fork-local run on exact candidate SHA and absence of unintended effects | Required before READY; upstream runs do not substitute. |

## Task briefs

### TASK-SCAN-001 — Capture repository, runtime, and Codex truth

| Field | Contract |
|---|---|
| Objective | Capture immutable local and toolchain evidence before any mutation. |
| Why | Prevent stale strategy values or wrong-worktree assumptions from controlling the run. |
| Context | Fork `Frictionless-Labs/pi`; upstream `earendil-works/pi`; supplied baseline is a hypothesis until verified. |
| Read first | `AGENTS.md`, `package.json`, `SECURITY.md`, applicable nested `AGENTS.md`, `.github/workflows/*.yml`. |
| Requirements | REQ-001, REQ-009. |
| In scope | Read-only cwd/root/branch/SHA/status/remotes, versions, engine, instructions, Codex config/help, workflow and script inventory. |
| Out of scope | Ref creation, fetch, install, edit, settings, secret values. |
| Allowed surface | None; AUDIT only. |
| Protected surface | Entire worktree, `.git`, credentials, user config. |
| Implementation requirements | Record actual values; label unverified fields; inspect installed Codex before relying on flags. |
| Stable interfaces | `ForkBaseline` candidate and environment evidence fields in `SPEC.md`. |
| Edge cases | Detached HEAD, dirty tree, missing CLI, multiple instruction files, CLI/docs contradiction. |
| Exact validation | `pwd`; `git rev-parse --show-toplevel`; `git branch --show-current`; `git status --short`; `git log --oneline -1`; `git remote -v`; `node --version`; `npm --version`; `codex --version`; relevant `codex --help`. |
| Completion evidence | Commands, exit codes, full SHA, status, versions, instruction/config inventory, contradiction record. |
| Failure rule | Wrong repo/branch or overlapping dirty state stops mutation; missing Codex marks only Codex mechanics UNVERIFIED. |
| Final response contract | `DONE` or `BLOCKED`; observed identity/SHA; exact checks; unknowns; next dependency TASK-ASSESS-001. |

### TASK-ASSESS-001 — Freeze the stable upstream target

| Field | Contract |
|---|---|
| Objective | Reverify and freeze the latest reviewed non-prerelease upstream release. |
| Why | A moving or stale target makes synchronization and evidence non-reproducible. |
| Context | Current frozen target is `v0.85.1` at `d981de1229ef899957bbe968bc8dcda02a21f477`. |
| Read first | `SPEC.md` MOD-01/02, upstream release metadata, local refs, release notes/delta. |
| Requirements | REQ-002, REQ-003. |
| In scope | Read release state, tag resolution, ancestry, delta, material compatibility/security notes. |
| Out of scope | Sync, target-tag mutation, upstream push, automatic tracking of `main`. |
| Allowed surface | Evidence artifact only if the task explicitly authorizes its update. |
| Protected surface | Product/workflow code, `.git` refs unless TASK-GUIDE-001 follows. |
| Implementation requirements | Record supplied versus observed target and human freeze decision; use one target per cycle. |
| Stable interfaces | `UpstreamTarget` and `TargetAlignment` preconditions. |
| Edge cases | New release during run, moved/missing tag, draft/prerelease latest, non-ancestor target. |
| Exact validation | GitHub release API metadata; `git rev-parse <tag>^{}`; `git merge-base --is-ancestor <baseline> <target>`; `git rev-list --count <baseline>..<target>`; `git diff --shortstat <baseline>..<target>`. |
| Completion evidence | Tag/SHA/stable flags/publish and observe UTC, ancestry exit 0, reviewed delta, human freeze owner. |
| Failure rule | Non-ancestor or unresolved target contradiction is `ARCHITECTURE` and blocks GUIDE. |
| Final response contract | `DONE` or `BLOCKED`; frozen tag/SHA; ancestry/delta; material risks; next TASK-ASSESS-002. |

The v0.85.1 cycle supersedes v0.85.0 after upstream documented that v0.85.0 published unintended
experimental code and dependencies that broke SDK imports. The Repository Maintainer authorized
the return to ASSESS and retarget on 2026-09-05; this does not authorize a later target change.

### TASK-ASSESS-002 — Classify every workflow

| Field | Contract |
|---|---|
| Objective | Produce exactly one fork disposition for every current workflow. |
| Why | All ten workflows are active server-side and eight can mutate upstream-oriented external state. |
| Context | KEEP `ci.yml` and `npm-audit.yml`; DISABLE the other eight via TASK-GUIDE-003 identity guards. |
| Read first | All ten `.github/workflows/*.yml`, `docs/fork/WORKFLOW_DISPOSITION.md`, SPEC MOD-03. |
| Requirements | REQ-004, REQ-011. |
| In scope | Triggers, permissions, secret names, environments, effects, coupling, value/risk, validation. |
| Out of scope | Secret values, workflow edits, server disable actions, credential creation. |
| Allowed surface | `docs/fork/WORKFLOW_DISPOSITION.md` when authorized. |
| Protected surface | `.github/workflows`, GitHub settings, secrets/environments. |
| Implementation requirements | One row per path; unknown org/environment credentials remain unknown; no hidden-state inference. |
| Stable interfaces | `WorkflowDisposition`; dispositions KEEP or DISABLE for the current inventory. |
| Edge cases | `workflow_run`, schedule, `pull_request_target`, manual publish input, implicit token, absent repo secrets. |
| Exact validation | Sorted workflow file list/count; parse/review every complete file; unique path/disposition count equals ten. |
| Completion evidence | Ten-row table and exact fork guard requirement for each DISABLE row. |
| Failure rule | Missing/duplicate row or unknown mutating path is `BLOCK_RELEASE`. |
| Final response contract | `DONE` or `BLOCKED`; KEEP/DISABLE counts; unknown interfaces; next TASK-STRATEGIZE-001. |

### TASK-STRATEGIZE-001 — Complete the control plane and fork governance

| Field | Contract |
|---|---|
| Objective | Materialize coherent `AGENTS.md`, `SPEC.md`, `docs/RUNBOOK.md`, and fork governance. |
| Why | Synchronization and automation changes require a deterministic, repository-native contract. |
| Context | The user kickoff request says the execution pack is incomplete; both named Downloads documents were absent and **[UNVERIFIED]** this cycle. |
| Read first | Originating request; current repo instructions; package/security/workflow sources; either named Downloads document only if it becomes available and is then read completely. |
| Requirements | REQ-009, REQ-012, REQ-013, REQ-014. |
| In scope | Conservative instruction merge; MOD-01–07; ASGD tasks; evidence schema; policy and trace. |
| Out of scope | Workflow/product/dependency/Git metadata changes and external actions. |
| Allowed surface | `AGENTS.md`, `SPEC.md`, `docs/RUNBOOK.md`, `docs/fork/*.md`. |
| Protected surface | Every other path. |
| Implementation requirements | Preserve current rules; use live facts; include YAML frontmatter where required; map 100% P0. |
| Stable interfaces | REQ-001–014, TEST-001–014, TASK IDs, evidence and readiness contracts. |
| Edge cases | Stale attached values, current CLI contradiction, remote state unavailable, overlapping edits. |
| Exact validation | Identifier cardinality/coverage checks; Markdown/YAML checks; diff-name scope; full diff review. |
| Completion evidence | Six governance artifacts, checks with exit codes, report, no workflow/code/Git mutation. |
| Failure rule | Missing P0 trace or scope breach blocks TASK-GUIDE-003; preserve valid current rules. |
| Final response contract | Required short status; changed paths; one-line check summary; concerns. |

### TASK-STRATEGIZE-002 — Define and prove the Pi isolation policy

| Field | Contract |
|---|---|
| Objective | Classify trusted versus unknown/untrusted Pi execution and define enforceable isolation. |
| Why | `SECURITY.md` says Pi has no sandbox and shares the local user's trust boundary. |
| Context | Unknown repositories, extensions, skills, and prompt-supplied executable content are untrusted. |
| Read first | `SECURITY.md`, SPEC MOD-05, `docs/fork/FORK_POLICY.md`, chosen isolation documentation. |
| Requirements | REQ-008. |
| In scope | Trust classes, network/mount/credential posture, disposal evidence, mixed-trust rule. |
| Out of scope | Claiming Pi is a sandbox, provisioning production infrastructure, copying credentials. |
| Allowed surface | Fork policy/evidence docs; isolation config only under a separately authorized task. |
| Protected surface | Home data, SSH/cloud credentials, production tokens, unrelated repositories, host sockets. |
| Implementation requirements | Unknown defaults to untrusted; choose an available verified container/VM boundary per run. |
| Stable interfaces | `TrustClassification` and TEST-008 evidence record. |
| Edge cases | Shared agents/sockets, symlinks, writable host mounts, persistent caches, egress, mixed extensions. |
| Exact validation | Review policy against prohibited ambient surfaces; capture mechanism/version/mount/network/credential/disposal evidence for an actual untrusted run before use. |
| Completion evidence | Reviewed policy plus operator evidence; no ambient restricted surface. |
| Failure rule | Missing or unproved isolation stops the untrusted run; escape becomes a security incident. |
| Final response contract | `DONE`, `DONE_WITH_CONCERNS`, or `BLOCKED`; class/boundary evidence; concerns; next TASK-GUIDE-001. |

### TASK-GUIDE-001 — Preserve the baseline ref

| Field | Contract |
|---|---|
| Objective | Create or verify the non-release archive ref at the exact pre-sync SHA. |
| Why | Recovery requires immutable provenance without triggering `v*` release automation. |
| Context | Required ref is `frictionless-readiness-baseline-20260904` at `ac4ac9e...`. |
| Read first | AGENTS Git rules, SPEC MOD-01, current refs/status/remotes. |
| Requirements | REQ-001. |
| In scope | Verify existing exact ref or create the named local ref when explicitly authorized. |
| Out of scope | Push, move/force ref, release tag, worktree cleanup. |
| Allowed surface | The one named local ref only. |
| Protected surface | All other refs, branches, worktree files, remotes. |
| Implementation requirements | Fresh preflight; never overwrite a mismatch; use non-`v*` name. |
| Stable interfaces | `ForkBaseline`. |
| Edge cases | Ref exists exact, exists mismatched, baseline object missing, dirty overlapping work. |
| Exact validation | `git show-ref --verify refs/tags/frictionless-readiness-baseline-20260904`; `git rev-parse refs/tags/frictionless-readiness-baseline-20260904^{}`. |
| Completion evidence | Ref and dereferenced SHA equal exact baseline; no remote change. |
| Failure rule | Mismatch is `BLOCK_RELEASE`; do not force-update. |
| Final response contract | `DONE` or `BLOCKED`; ref/SHA; commands/exits; next TASK-GUIDE-002. |

### TASK-GUIDE-002 — Create and verify the target-aligned branch

| Field | Contract |
|---|---|
| Objective | Create or verify the isolated readiness branch directly at the frozen target before fork-specific edits. |
| Why | Preserves upstream lineage and exposes the complete reviewed delta. |
| Context | Baseline `ac4ac9e...`; target `v0.85.1`/`d981de...`. |
| Read first | SPEC MOD-01/02, status/branch/refs, target delta, AGENTS Git rules. |
| Requirements | REQ-003, REQ-013. |
| In scope | Fresh read checks, baseline-to-target ancestry proof, and target-aligned readiness-branch creation or verification. |
| Out of scope | Fork-main synchronization, rebase, merge commit, reset, force, cleanup, or main push. |
| Allowed surface | Current readiness branch ref/worktree only. |
| Protected surface | Archive ref, unrelated branches/files, remotes. |
| Implementation requirements | Baseline ancestry exit 0; preserve work; freeze target; capture branch base and prove fork `main` was not changed. |
| Stable interfaces | `TargetAlignment` operation=`branch-created-at-target`. |
| Edge cases | Non-ancestor, collision, concurrent change, moved tag, new stable release. |
| Exact validation | Preflight; `git merge-base --is-ancestor <baseline> <target>`; `git rev-parse HEAD`; branch/worktree identity; delta stat; fork-main ref observation. |
| Completion evidence | Branch base equals frozen target; baseline is its ancestor; no fork-main change, merge, rebase, or reset; baseline ref unchanged. |
| Failure rule | Non-ancestor state, branch/target mismatch, fork-main mutation, or threatened work loss stops and returns to ASSESS. |
| Final response contract | `DONE` or `BLOCKED`; old/new/target SHA; delta; concerns; next TASK-GUIDE-003. |

### TASK-GUIDE-003 — Apply fork repository-identity workflow guards

| Field | Contract |
|---|---|
| Objective | Prevent every DISABLE workflow job from running in `Frictionless-Labs/pi`. |
| Why | All ten workflows are active; eight mutate releases, packages, catalogs, or issue/PR state. |
| Context | This task implements repository-identity job guards; KEEP workflows remain runnable. |
| Read first | All workflows, disposition table, SPEC MOD-03, GitHub expression/job-condition docs. |
| Requirements | REQ-004, REQ-011. |
| In scope | Only eight DISABLE workflow files and contract-level guard validation. |
| Out of scope | KEEP workflow edits, server settings, secrets, environments, publication, product code. |
| Allowed surface | The eight paths named DISABLE in `WORKFLOW_DISPOSITION.md`. |
| Protected surface | `ci.yml`, `npm-audit.yml`, all non-workflow files, GitHub settings. |
| Implementation requirements | Add guard to every job while preserving existing conditions and dependency semantics; exact upstream identity only. |
| Stable interfaces | `WorkflowDisposition`; repository identity is `github.repository`. |
| Edge cases | Existing job `if`, `always()`, dependency cleanup jobs, `workflow_run`, `pull_request_target`. |
| Exact validation | Parse all workflow YAML; enumerate jobs; prove every DISABLE job has effective upstream-only guard; prove KEEP files unchanged; review diff. |
| Completion evidence | Eight guarded workflows; two unchanged KEEP workflows; no job executable in fork; pins/permissions preserved. |
| Failure rule | One unguarded job, invalid YAML, or altered KEEP workflow is `BLOCK_RELEASE`. |
| Final response contract | `DONE` or `BLOCKED`; guarded job/file counts; checks/exits; concerns; next TASK-GUIDE-004. |

### TASK-GUIDE-004 — Materialize governance and evidence artifacts

| Field | Contract |
|---|---|
| Objective | Create complete repository-native fork policy, workflow disposition, and evidence registry. |
| Why | A maintainer must reproduce readiness without the original conversation. |
| Context | This artifact set uses the live v0.85.1 cycle evidence rather than attached-pack or superseded v0.85.0 observations. |
| Read first | Current control-plane artifacts, verified kickoff request, workflows, and current-cycle evidence; mark the absent strategy/execution-pack files **[UNVERIFIED]** until available and fully read. |
| Requirements | REQ-009, REQ-012, REQ-013, REQ-014. |
| In scope | `AGENTS.md`, `SPEC.md`, `docs/RUNBOOK.md`, `docs/fork/*.md`. |
| Out of scope | Workflows, code, dependencies, lockfiles, generated files, Git metadata, external actions. |
| Allowed surface | The six named governance artifacts. |
| Protected surface | Every other tracked/untracked repository path. |
| Implementation requirements | YAML frontmatter; complete modules/tasks; ten workflow rows; observed records; P0 trace 100%. |
| Stable interfaces | All REQ, FEAT, MOD, TASK, TEST, RISK, CTRL, KPI identifiers and the source labels defined in `SPEC.md`. |
| Edge cases | Evidence unavailable, current state changes, contradictory source, accidental readiness claim. |
| Exact validation | Identifier/content checker; workflow-row set equality; changed-path allowlist; Markdown frontmatter; full diff review. |
| Completion evidence | Artifact list, requirement coverage, exact checks/exits, concerns, no commit. |
| Failure rule | Missing field/ID, stale live fact, or out-of-scope diff blocks completion. |
| Final response contract | `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, or `BLOCKED`; paths; one-line checks; concerns. |

### TASK-GUIDE-005 — Validate candidate scope and local contracts

| Field | Contract |
|---|---|
| Objective | Escalate from static to applicable local/integration validation before remote execution. |
| Why | Remote evidence is wasteful and unsafe if scope, docs, YAML, or targeted contracts already fail. |
| Context | Documentation uses content/schema checks; workflow guard changes require YAML/job-condition checks. |
| Read first | Diff, task validation contract, root scripts, affected package instructions. |
| Requirements | REQ-005, REQ-013. |
| In scope | L0–L4 checks applicable to changed surfaces. |
| Out of scope | Prohibited local `npm run build`/`npm test`, real-provider tests, publication. |
| Allowed surface | Test/repair only within the originating task's allowed paths. |
| Protected surface | Unrelated source, assertions, security gates, credentials. |
| Implementation requirements | Run narrow first; code changes require `npm run check`; modified tests run specifically; CI owns full build/test. |
| Stable interfaces | Validation ladder and evidence record. |
| Edge cases | Formatter writes, poisoned env, missing deps, docs-only changes, pre-existing failure. |
| Exact validation | Task-specific L0/L1; applicable package test; `npm run check` after code; full diff/status; record every unrun gate. |
| Completion evidence | Commands/exits, output sufficient to classify failures, allowlisted diff only. |
| Failure rule | Never weaken checks; classify and loop according to VIZ-07. |
| Final response contract | `DONE` or `BLOCKED`; validation matrix; exact failures; candidate SHA; next TASK-DEPLOY-001. |

### TASK-DEPLOY-001 — Produce fork-local CI evidence

| Field | Contract |
|---|---|
| Objective | Run the KEEP CI workflow on the exact guarded candidate SHA. |
| Why | Upstream/local evidence cannot prove fork CI; exact candidate execution is required. |
| Context | Main CI run `34014481660` failed because `runClient()` unsubscribed before replicated `run_end` delivery. Candidate source `f2858ead77530b5bf86bd787b141a83e7a2daa41` repairs that race; push run `34014966796`, PR run `34014979030`, and post-merge run `34015185239` all passed. |
| Read first | `ci.yml`, candidate diff, GUIDE validation, GitHub Actions state. |
| Requirements | REQ-005. |
| In scope | Human-authorized push of the immutable candidate to `codex/pi-fork-production-readiness`; observe the resulting run/jobs. |
| Out of scope | Release tags/workflows, publication credentials, bypassing checks. |
| Allowed surface | Current-cycle readiness-branch evidence pushes, PR, and resulting GitHub run state under explicit current-turn authorization. |
| Protected surface | Releases, packages, R2, secrets, repository settings. |
| Implementation requirements | Use the exact readiness-branch push path once; workflow source SHA, checkout SHA, and run head SHA must each equal the pushed immutable candidate-source SHA. Retain stable run URL/ID, jobs, conclusion, UTC, and the later evidence-record revision that indexes the run. |
| Stable interfaces | TEST-005 `EvidenceRecord`. |
| Edge cases | Zero runs, queue/runner outage, workflow source or checkout on another SHA, run on merge SHA, canceled superseded run, flaky failure. |
| Exact validation | Confirm the push ref is exactly `codex/pi-fork-production-readiness`; workflow source SHA, checkout SHA, and run head SHA equal the pushed candidate SHA; all canonical build/check/test steps conclude success; no DISABLE workflow job runs. |
| Completion evidence | Stable run ID/URL, candidate-source SHA, job conclusions, timestamps, failure/rerun trail, and evidence-record revision. |
| Failure rule | Absent/mismatched/failed run is not PASS; classify and remediate without workflow relaxation. |
| Final response contract | `DONE` or `BLOCKED`; run/SHA/conclusion; failures; next TASK-DEPLOY-002. |

### TASK-DEPLOY-002 — Produce dependency, signature, and secret evidence

| Field | Contract |
|---|---|
| Objective | Prove dependency integrity and zero live credentials introduced by the candidate. |
| Why | Dependency, signature, and secret evidence must be candidate-keyed even when server controls are enabled. |
| Context | Run `34012634857` passed audit/signature checks on evidence head `3064da9f...`; nested candidate audits also pass. Secret scanning and push protection are enabled; two inherited deleted-path alerts remain open with unknown validity. |
| Read first | `npm-audit.yml`, dependency policy, candidate diff/history scope, approved scanner docs. |
| Requirements | REQ-006, REQ-007. |
| In scope | Candidate-keyed audit/signature run and approved redacted secret scan. |
| Out of scope | Printing values, enabling settings, rotating credentials, accepting live secrets. |
| Allowed surface | Evidence records; GitHub run after explicit remote authorization. |
| Protected surface | Secrets, credential stores, package publication, source outside remediation task. |
| Implementation requirements | Use exact repository commands; no lifecycle scripts; scan defined delta; redact all findings. |
| Stable interfaces | TEST-006/007 `EvidenceRecord`; candidate-source SHA is distinct from the later evidence-record revision; optional waiver with owner/reason/expiry where policy permits. |
| Edge cases | Registry outage, advisory change, disabled scanning API, false positive, upstream pre-existing finding. |
| Exact validation | `npm audit --omit=dev --audit-level=moderate`; `npm audit signatures --omit=dev`; approved scanner on candidate/diff; verify SHA and zero validated live secrets. |
| Completion evidence | Commands/exits/run ID/SHA/timestamps; redacted counts; waiver or remediation/rerun. |
| Failure rule | Signature failure or validated live secret is unwaivable `BLOCK_RELEASE`; never paste the secret. |
| Final response contract | `DONE` or `BLOCKED`; audit/signature/secret results; redacted concerns; next TASK-DEPLOY-003. |

### TASK-DEPLOY-003 — Verify merge governance and independent review

| Field | Contract |
|---|---|
| Objective | Verify GitHub merge controls and review the candidate as another engineer's work. |
| Why | Merge controls and independent review must be proven before integration. |
| Context | `main` protection now enforces strict CI/audit/two-CodeQL checks, PRs, admin enforcement, linear history, conversation resolution, and force-push/deletion denial. MIKKOH is the sole collaborator, so a nonzero platform approval count would deadlock this public fork; explicit human GO plus independent agent review is the approved control. |
| Read first | Branch/ruleset API evidence, full candidate diff, all TEST evidence, fork policy. |
| Requirements | REQ-010, REQ-012, REQ-014. |
| In scope | Settings verification, independent 15-dimension review, and bounded protection changes needed for required candidate checks. |
| Out of scope | Dismissing findings, bypassing protection, or weakening required checks. |
| Allowed surface | Evidence docs and the authorized `main` protection/API settings. |
| Protected surface | Release refs, credentials, environments, packages, catalogs, and deployments. |
| Implementation requirements | Review correctness, trace, provenance, workflow/supply-chain/secret/trust safety, tests, failure, rollback, scope, drift, effects. |
| Stable interfaces | Finding: severity, REQ/TASK, evidence, fix, validation, release impact. |
| Edge cases | Missing API scope, stale evidence, candidate changed during review, reviewer conflict. |
| Exact validation | Query rulesets/protection/required checks/reviews/bypass; verify candidate SHA; inspect full diff; ensure every finding disposition is evidenced. |
| Completion evidence | Settings result, reviewer identity/UTC, findings, repairs/reruns, human decision required. |
| Failure rule | Absent or unknown merge controls and any unresolved release blocker prevent human GO. |
| Final response contract | `DONE`, `DONE_WITH_CONCERNS`, or `BLOCKED`; settings evidence; findings; decision owner; next TASK-DEPLOY-004. |

### TASK-DEPLOY-004 — Human-authorized merge and post-check

| Field | Contract |
|---|---|
| Objective | Merge only after GO and prove intended main state without unintended side effects. |
| Why | Merge and triggered workflows are externally consequential and can invalidate candidate evidence. |
| Context | Repository Maintainer MIKKOH granted explicit current-turn GO for the bounded readiness PR, merge, and post-check on 2026-09-06. |
| Read first | Final evidence registry, review, branch controls, workflow disposition, rollback decision. |
| Requirements | REQ-011, REQ-014. |
| In scope | Exact-target lineage-preserving fast-forward, reviewed fork-delta PR integration, and read-only post-check under explicit authorization. |
| Out of scope | Release tag, npm/GitHub Release/model-catalog/R2 publication, credential/settings changes. |
| Allowed surface | Authorized PR/branch merge only; evidence artifact update. |
| Protected surface | Upstream, release refs, packages, catalogs, credentials, infrastructure. |
| Implementation requirements | Reconfirm candidate SHA/gates immediately before merge; record human GO; observe every resulting workflow/effect. |
| Stable interfaces | `ReadinessDecision`, TEST-013/014. |
| Edge cases | Merge commit changes SHA, new push arrives, checks stale, DISABLE workflow unexpectedly starts. |
| Exact validation | Verify merged `main`, checks, runs, and attribution of every release/npm/catalog/R2/issue/PR/comment effect; block on any unintended effect. |
| Completion evidence | PR/merge identifier, source/result SHA, approver, UTC, checks, workflow runs, effect audit. |
| Failure rule | Unexpected job/effect triggers containment; preserve evidence; rollback or forward fix is separately human-authorized. |
| Final response contract | `DONE`, `DONE_WITH_CONCERNS`, or `BLOCKED`; merge/post-check proof; incident if any; next TASK-VERIFY-001. |

### TASK-VERIFY-001 — Calculate final readiness

| Field | Contract |
|---|---|
| Objective | Independently reconcile all requirements, findings, evidence, and effects into one readiness state. |
| Why | Passing a subset of checks does not prove production readiness. |
| Context | READY requires >=98% confidence, 100% mandatory P0 PASS, no P0/P1 blocker, and human GO. |
| Read first | `SPEC.md`, full diff, all fork docs, run/security/settings evidence, review findings. |
| Requirements | REQ-001 through REQ-014. |
| In scope | Counterexample review, evidence freshness/SHA checks, trace and status calculation. |
| Out of scope | New mutation, silent waiver, retroactive evidence inference. |
| Allowed surface | Final evidence/readiness record when authorized. |
| Protected surface | Code, workflows, Git/GitHub state, prior evidence history. |
| Implementation requirements | Recompute rather than trust summaries; every PASS must cite same-cycle evidence. |
| Stable interfaces | `ReadinessDecision`: READY, CONDITIONAL, or BLOCKED. |
| Edge cases | P1 blocker, stale run, unknown hidden credential, post-merge SHA, conditional waiver expiry. |
| Exact validation | Check 14 evidence records, 11/11 P0 design traces, mandatory execution PASS rate, findings, diff/status, post-merge TEST-014, human GO. |
| Completion evidence | Final status/confidence, candidate/result SHA, trace/pass rates, blockers, human-gated actions, next dependency. |
| Failure rule | Unknown or missing mandatory evidence lowers status; never inflate confidence. |
| Final response contract | STATUS, RESULT, FILES, REPOSITORY STATE, WORKFLOWS, VALIDATION, TRACE, SECURITY, BLOCKERS, HUMAN GATES, NEXT DEPENDENCY. |

## Final executed outcome

| Field | Observed result |
|---|---|
| Decision | **READY at 99% confidence** for controlled internal fork use under `FORK_POLICY.md`. |
| Source | Candidate source `f2858ead77530b5bf86bd787b141a83e7a2daa41`; one narrow product repair is justified by mandatory CI failure `34014481660`. |
| Integration | PR#6 merged at exact head `f2858ead77530b5bf86bd787b141a83e7a2daa41` on 2026-09-06T05:55:38Z. |
| Checks | Candidate push/PR and post-merge CI, npm audit/signatures, both CodeQL analyses, 20 repeated focused tests, and the 26-test remote-runtime file completed successfully. |
| Workflow safety | Two KEEP workflows active; eight DISABLE workflows guarded and `disabled_manually`; no unauthorized job/effect observed. |
| Security | Dependabot open count zero; 532 inherited CodeQL alerts and two deleted-path historical secret alerts remain disclosed and open. |
| Governance | Strict four-check protection, admin enforcement, linear history, conversation resolution, and force/delete denial restored and verified. |
| Free-only constraint | Standard public-repository controls and runners only; paid validity/non-provider secret scanning disabled. |
| Excluded effects | No release/tag/package/model-catalog/R2/deployment/credential or disabled-workflow issue/comment effect occurred. Dependabot auto-closed redundant PR#1/PR#2 and posted two attributed bot comments. |
| Stop condition | `CODEX_PI_FORK_PRODUCTION_READINESS_COMPLETE`. |

## Rollback and recovery protocol

| Failure point | Recovery |
|---|---|
| Before target alignment | Stop; baseline ref remains authoritative; repair evidence or target decision. |
| Baseline is not an ancestor of target or branch base differs from target | Do not merge/rebase/reset; return to ASSESS with ancestry and fork-delta evidence. |
| TASK-GUIDE-003 guard defect | Repair only guarded workflow files; rerun static/contract validation before remote execution. |
| Candidate check fails | Classify; repair smallest cause on readiness branch; regenerate candidate-keyed evidence. |
| Unintended external effect | Stop workflows where authorized, preserve logs, identify resources/credentials, and open human incident gate. |
| Pre-merge abandonment | Human may delete the readiness branch; archive baseline remains immutable. |
| Post-merge defect | Prefer reviewed forward fix. Human may authorize revert to a known safe commit; never force-reset shared `main`. |

Rollback is not proof of readiness. Restoring code/ref state does not undo a public package, release,
R2 write, comment, issue/PR mutation, or credential exposure; those require separate containment.

## Review protocol

The independent reviewer must test counterexamples across requirement correctness, P0 traceability,
provenance, target selection, workflow safety, supply chain, secrets, Pi trust, Codex semantics, test
integrity, failure handling, rollback, scope creep, documentation drift, and external effects. Every
finding records severity, affected REQ/TASK, evidence, required fix, validation, and release impact.
An unresolved release-blocking finding prevents READY and human GO.
