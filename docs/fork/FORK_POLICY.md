---
title: Frictionless Labs Pi Fork Policy
version: 1.2.0
status: ACTIVE
created_date: 2026-09-04
updated_date: 2026-09-06
tags:
  - pi
  - fork-policy
  - governance
confidence: 0.98
owner: Frictionless Labs Repository Maintainer
---

# Frictionless Labs Pi Fork Policy

## Purpose and boundaries

`Frictionless-Labs/pi` is a minimal-delta, reviewable fork of `earendil-works/pi` for controlled
Frictionless use. It preserves upstream architecture and release lineage while adding only owned
fork governance, workflow safety, evidence, and operator controls. Readiness does not authorize
rebranding, independent public distribution, new Pi product features, or upstream mutation.

| Boundary | Policy |
|---|---|
| Authoritative source | `earendil-works/pi` stable, immutable GitHub releases. |
| Current source point | `v0.85.1` at `d981de1229ef899957bbe968bc8dcda02a21f477`. |
| Current recovery point | `frictionless-readiness-baseline-20260904` at `ac4ac9eaf69f2b01ca3af984a5c48f3b99b84278`. |
| Fork delta | Governance/workflow/docs only unless a mandatory test proves a narrow compatibility defect. |
| Publication | Out of scope until a separate Frictionless distribution and credential design is approved. |
| Readiness | Evidence-based state for one immutable candidate-source SHA, indexed by a separately versioned evidence record; never inherited from upstream. |

The v0.85.1 cycle supersedes v0.85.0 because upstream documented a v0.85.0 distribution defect
that broke SDK imports by publishing unintended experimental code and dependencies. The Repository
Maintainer explicitly authorized this retarget on 2026-09-05.

## Target-selection and synchronization cadence

| Trigger | Required action | Decision owner |
|---|---|---|
| Monthly maintenance review | Inspect the latest non-draft/non-prerelease upstream release and decide whether to open a bounded sync cycle. | Repository Maintainer |
| Upstream security release/advisory | Start an out-of-cycle review as soon as practical; do not automatically sync. | Repository Maintainer + Security Reviewer |
| Before every sync mutation | Requery release metadata, resolve the tag SHA, review material release/delta risk, prove ancestry, and freeze one target. | Repository Maintainer |
| New stable release after GUIDE starts | Keep the frozen target unless a material security reason causes a human return to ASSESS and a new evidence cycle. | Repository Maintainer |

This cycle creates a new isolated readiness branch directly at the frozen release after proving the
preserved baseline is its ancestor; that is target alignment, not an executed fork-main
fast-forward. A non-ancestor result stops the cycle. Any later fork-main transition follows the
separately authorized merge path and never permits rebase, history rewrite, force, or improvisation.

## Delta ownership

| Surface | Accountable owner | Required review | Default disposition |
|---|---|---|---|
| Upstream-derived product code | Upstream maintainers for source; Frictionless Maintainer for adoption | Delivery review and full relevant validation | Preserve unchanged. |
| Fork governance/docs | Frictionless Repository Maintainer | Delivery review | Maintain locally. |
| Workflow dispositions/guards | Frictionless Repository Maintainer | Security review | Two KEEP; eight upstream-only DISABLE. |
| Dependencies/lockfiles | Package owners + Repository Maintainer | Supply-chain review | No readiness-only change. |
| GitHub settings/rulesets | Repository Maintainer | Security/Delivery review | Protected `main` and free-tier security controls are enabled and verified. |
| Credentials/environments | Credential owner, currently unknown where upstream-specific | Security review | Never copy/substitute; unavailable means disabled. |
| Release/distribution infrastructure | No Frictionless owner established by this policy | Separate architecture/security/release decision | Prohibited. |

Every local delta must map to a REQ/TASK, use an explicit path allowlist, and produce a reviewed diff.
An unexplained or unowned change blocks the candidate.

## Trust classes and isolation

Pi intentionally operates inside the local user's file/account boundary and does not provide a
security sandbox for arbitrary repository, extension, skill, or prompt content.

| Class | Examples | Permitted boundary | Evidence |
|---|---|---|---|
| TRUSTED | Reviewed repository revision plus reviewed extensions/skills from an approved source | Approved development environment with least required access | Source/revision and reviewer recorded. |
| UNKNOWN | Unreviewed repository, extension, skill, generated script, or prompt-supplied executable content | Treat as UNTRUSTED until reviewed | Classification and unresolved provenance recorded. |
| UNTRUSTED | External code/content or any component that fails trust review | Disposable container, VM, or equivalent verified isolation | Mechanism/version, mounts, network, credentials, disposal result. |
| MIXED | TRUSTED repository with any UNKNOWN/UNTRUSTED component | Least-trusted component controls; use disposable isolation | Component inventory and resulting classification. |

Disposable isolation must not expose ambient SSH agents/keys, cloud/model/production credentials,
sensitive home-directory content, unrelated writable repositories, host Docker/privileged sockets,
privileged devices, or reusable secret-bearing volumes. Network access is denied or allowlisted to
the minimum required endpoints. Writable mounts are limited to the disposable task workspace.

Failure to prove these properties stops the untrusted run. A suspected escape or credential/data
access is a security incident: preserve evidence, contain the boundary, and obtain human direction
before credential revocation, rotation, or host recovery.

## Workflow policy

Only `.github/workflows/ci.yml` and `.github/workflows/npm-audit.yml` have fork readiness value and
are KEEP. The other eight current workflows are DISABLE through TASK-GUIDE-003 repository-identity guards.
The complete per-file contract is in `WORKFLOW_DISPOSITION.md`.

Repository workflow content, GitHub server active state, Actions policy, environments, and secrets
are distinct state planes. The 2026-09-06T04:47:15Z snapshot showed Actions enabled, full-SHA
pinning required, a read-only default workflow token, the readiness branch present, and candidate
CI/audit runs successful. Vulnerability alerts, Dependabot security updates, secret scanning, push
protection, and CodeQL default setup are enabled. Paid-only non-provider-pattern and validity checks
remain disabled to preserve a free-only control plane. Repository counts do not prove organization
or other hidden credentials are absent; upstream-specific jobs remain guarded.

## Merge governance

| Gate | Required state |
|---|---|
| Candidate | Exact immutable candidate-source SHA with scoped diff and no unexplained file. |
| Review | Independent correctness, security, workflow, trust, rollback, and external-effect review. |
| Checks | Candidate-SHA CI, vulnerability, signature, and secret gates PASS. |
| Protection | Required checks/reviews/bypass/force-push policy observed and approved before merge. |
| Approval | Explicit Repository Maintainer GO after evidence review. |
| Merge | Authorized PR/branch path only; no direct autonomous push. |
| Post-check | `main` relationship and all triggered runs/effects observed; TEST-014 PASS. |

The 2026-09-06T04:47:15Z fork snapshot proves `main` protection with strict `build-check-test` and
`audit` checks, admin enforcement, PR-only integration, linear history, conversation resolution,
and force-push/deletion denial. The platform approval count is zero because MIKKOH is the repository's
only collaborator; a nonzero count would deadlock. Current-turn human GO plus independent agent
review is the approved single-maintainer review control.

## Recovery

| Event | Recovery rule |
|---|---|
| Target contradiction or non-ancestor | Stop and return to ASSESS; preserve branch/baseline evidence. |
| Scoped implementation defect before merge | Repair narrowly on the readiness branch; rerun affected and broader gates. |
| Candidate abandoned | Human may delete the readiness branch; never move the archive ref. |
| Defect after merge | Prefer a reviewed forward fix; human may authorize revert without rewriting shared history. |
| Unexpected workflow/external effect | Contain if authorized, preserve logs, identify affected resource/credential, and enter incident review. |
| Credential exposure evidence | Block readiness; human security owner decides revoke/rotate after scoped impact evidence. |

Returning to the baseline restores a Git source point only. It cannot undo a package, GitHub
Release, R2 upload, issue/PR mutation, comment, or credential exposure.

## Human-only operations

Current-turn Repository Maintainer authorization permits the bounded free-tier settings, evidence
commits/pushes, PR, protected merge, and post-merge verification required to complete this cycle.
It grants no standing authority after the cycle and does not authorize credentials, releases,
deployment, publication, or destructive recovery.

- Freeze or change the upstream target after review.
- Push any unrelated branch/ref or approve readiness for a later cycle.
- Change environments, secrets, or workflow server state outside the bounded readiness controls.
- Create/push `v*` tags; run release scripts; publish/unpublish npm packages or GitHub Releases.
- Publish model catalogs or release announcements; write/delete R2 or production infrastructure.
- Copy, create, rotate, revoke, or delete credentials and access grants.
- Perform destructive recovery, data restoration, branch deletion, or incident containment with external effects.

## Readiness decision

READY requires confidence >=98%, 100% mandatory P0 execution PASS, no P0/P1 blocker, complete
candidate-keyed evidence, independent review, verified merge governance, TEST-014, and explicit
human GO. Previous source `99ad976e...` passed fork CI, dependency/signature audit, and redacted
secret scan. Current candidate `38032056...` adds mandatory nested-lock security repairs and has
isolation and merge-governance proof; exact-head remote checks, the protected PR's CodeQL
differential, merge, and post-merge effect proof remain.
