---
title: Pi Fork Workflow Disposition
version: 1.4.0
status: READY
created_date: 2026-09-04
updated_date: 2026-09-06
tags:
  - pi
  - github-actions
  - workflow-safety
confidence: 0.99
owner: Frictionless Labs Repository Maintainer
---

# Pi Fork Workflow Disposition

## Decision rule

`ci.yml` and `npm-audit.yml` are KEEP. Every other current workflow is DISABLE in
`Frictionless-Labs/pi` through a TASK-GUIDE-003 repository-identity guard on every job and a
`disabled_manually` GitHub server state. DISABLE preserves the upstream workflow source while the
guard makes its jobs executable only when `github.repository == 'earendil-works/pi'`. Unknown
credential ownership or missing services cannot be replaced with personal/Frictionless credentials.

The post-merge fork snapshot reported all ten repository workflows present, two KEEP workflows
active, eight DISABLE workflows `disabled_manually`, SHA pinning required, and a read-only default
token. Exact-head CI run `34015185239` and audit run `34015193772` succeeded on merged PR#6 head.
Secret scanning, push protection, Dependabot, and extended CodeQL are enabled. Workflow references
below are source interfaces; repository counts do not prove hidden credentials absent.

## Complete inventory

| Workflow | Trigger | Effective permissions | Secret names/interfaces | Environments | External mutation | Upstream coupling | Fork value / risk | Disposition | Validation |
|---|---|---|---|---|---|---|---|---|---|
| `.github/workflows/approve-contributor.yml` | `issue_comment: created` | Job: `contents: write`, `issues: write`, `pull-requests: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Edits `.github/APPROVED_CONTRIBUTORS`, commits/pushes default branch, comments on issue | Upstream contributor approval file/process and maintainer semantics | No readiness value / can mutate default branch and public issue state | **DISABLE** | TASK-GUIDE-003: upstream-only guard implemented on `approve`; static validation and post-merge disabled-state proof passed. |
| `.github/workflows/build-binaries.yml` | Push tag `v*`; manual dispatch with `tag`/`source_ref` | Top `{}`; jobs use `contents: read/write`, `actions: read`, `id-token: write` | `GITHUB_TOKEN`; `PI_ARTIFACTS_R2_ACCESS_KEY_ID`; `PI_ARTIFACTS_R2_SECRET_ACCESS_KEY`; npm OIDC trusted-publishing interface | `npm-publish`; `pi-model-upload` | Builds/uploads artifacts; creates/deletes/publishes GitHub Releases; publishes npm packages; writes pi.dev release marker to R2 | Earendil npm identities, pi.dev/R2 endpoint, release process and protected environments | No readiness value / irreversible public distribution and credential-dependent writes | **DISABLE** | TASK-GUIDE-003: upstream-only guards cover all seven jobs, including `always()` cleanup; static validation and post-merge disabled-state proof passed. |
| `.github/workflows/ci.yml` | Push to `main` or `codex/pi-fork-production-readiness`; pull request to `main`; manual dispatch | Workflow: `contents: read`; checkout uses `persist-credentials: false` | Automatic read-only `GITHUB_TOKEN` interface; not persisted by checkout | None | No repository/external write declared; installs, builds, checks, tests | Canonical repository scripts only | Essential candidate health evidence / runner and supply-chain execution risk bounded by reviewed source | **KEEP** | TASK-DEPLOY-001 controls passed; PR and post-merge exact-SHA runs completed successfully with retained jobs, pins, commands, read permission, and non-persistent checkout token. |
| `.github/workflows/issue-analysis.yml` | Issue labeled; issue comment created | Workflow: `contents: read`, `issues: write` | `EARENDIL_ORG_READ_TOKEN`; `PI_AUTH_JSON`; `PI_AUTH_UPDATE_TOKEN`; `PI_GIST_TOKEN`; automatic `GITHUB_TOKEN` | `pi-analyze` | Reads Earendil staff membership; labels/comments issues; runs credentialed Pi; updates environment secret; creates private gist | `earendil-works/staff`, `@issuron`, pi.dev session sharing, Earendil-owned credential process | No readiness value / organization auth, model credentials, secret write, gist and issue mutations | **DISABLE** | TASK-GUIDE-003: upstream-only guards cover `authorize` and `analyze`; static validation and post-merge disabled-state proof passed. |
| `.github/workflows/issue-gate.yml` | Issue opened | Job: `contents: read`, `issues: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Comments, labels, and closes public issues | Upstream approved-contributor file, trusted bot list, contribution policy | No readiness value / can close legitimate fork issues and send upstream-branded guidance | **DISABLE** | TASK-GUIDE-003: upstream-only guard covers `check-contributor`; static validation and post-merge disabled-state proof passed. |
| `.github/workflows/issue-triage-labels.yml` | Issue reopened or labeled | Job: `issues: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Adds/removes labels and closes issues | Upstream label taxonomy and triage flow | No readiness value / bulk public issue mutation with unadopted governance | **DISABLE** | TASK-GUIDE-003: upstream-only guard covers `update-labels`; static validation and post-merge disabled-state proof passed. |
| `.github/workflows/npm-audit.yml` | Daily schedule `37 7 * * *`; manual dispatch | Workflow: `contents: read` | None | None | No external write; installs without lifecycle scripts, queries npm audit/signature services | Canonical npm registry/security policy only | Essential dependency/signature evidence / registry availability and changing advisory data | **KEEP** | Exact merged-head run `34015193772` passed audit and signature verification; rerun on future candidate heads. |
| `.github/workflows/pr-gate.yml` | `pull_request_target: opened` | Job: `contents: read`, `issues: write`, `pull-requests: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Comments on and closes public PRs | Upstream approved-contributor gate, bot list, contribution policy | No readiness value / privileged base-context event can close fork PRs and post upstream policy | **DISABLE** | TASK-GUIDE-003: upstream-only guard covers `check-contributor`; static validation and post-merge disabled-state proof passed. |
| `.github/workflows/publish-model-catalog.yml` | Successful `CI` workflow run on `main`; selected PR paths; weekday schedule `17 8-13 * * 1-5`; manual dispatch | Workflow: `contents: read` | `PI_ARTIFACTS_R2_ACCESS_KEY_ID`; `PI_ARTIFACTS_R2_SECRET_ACCESS_KEY` | `pi-model-upload` | Uploads GitHub artifact; may write production model catalog to R2 | pi.dev production R2 bucket/endpoint, Vienna publication window, upstream CI name | Generation has diagnostic value but publication is unnecessary / automatic chain can mutate production R2 | **DISABLE** | TASK-GUIDE-003: upstream-only guards cover `generate` and `publish`; static validation and post-merge disabled-state proof passed. |
| `.github/workflows/remove-inprogress-on-close.yml` | Issue closed | Job: `issues: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Removes `inprogress` label from public issue | Upstream issue label taxonomy | No readiness value / unadopted public issue mutation | **DISABLE** | TASK-GUIDE-003: upstream-only guard covers `remove-label`; static validation and post-merge disabled-state proof passed. |

## Acceptance and follow-up

- Inventory cardinality is exactly ten unique current workflow paths: two KEEP and eight DISABLE.
- TASK-GUIDE-003 guards every DISABLE job, not only workflow entrypoints or a subset of dependency
  roots; immutable-candidate, PR, and post-merge server-state evidence passed.
- Existing job conditions, including `always()` cleanup and `workflow_run` filters, must remain
  effective after composing the repository-identity condition.
- `npm-audit.yml` remains byte-unchanged. The authorized `ci.yml` KEEP delta is limited to an empty
  `workflow_dispatch`, the exact `codex/pi-fork-production-readiness` push addition, workflow-level
  `contents: read`, and checkout `persist-credentials: false`; `main` push, `main` pull-request,
  jobs, pins, and commands remain unchanged. An authorized candidate push must prove workflow source,
  checkout, and run SHA equality before TEST-005 can PASS.
- Full-SHA action pins and existing explicit permission scopes must not be weakened.
- Static guards and server disablement are both required. Candidate and post-merge run evidence
  shows no DISABLE job or unauthorized external effect in the fork.
