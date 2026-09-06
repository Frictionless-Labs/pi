---
title: Pi Fork Workflow Disposition
version: 1.1.0
status: BLOCKED
created_date: 2026-09-04
updated_date: 2026-09-05
tags:
  - pi
  - github-actions
  - workflow-safety
confidence: 0.98
owner: Frictionless Labs Repository Maintainer
---

# Pi Fork Workflow Disposition

## Decision rule

`ci.yml` and `npm-audit.yml` are KEEP. Every other current workflow is DISABLE in
`Frictionless-Labs/pi` through a TASK-GUIDE-003 repository-identity guard on every job. DISABLE preserves
the upstream workflow source while making its jobs executable only when
`github.repository == 'earendil-works/pi'`. Unknown credential ownership or missing services cannot
be replaced with personal/Frictionless credentials under this readiness program.

The current fork snapshot at 2026-09-06T00:18:28Z reported all ten workflows present, Actions
enabled, `allowed_actions=all`, `sha_pinning_required=false`, no readiness branch, zero target-SHA
runs, zero repository environments, and zero repository Actions secrets. It reported all listed
secret-scanning controls and Dependabot security updates disabled. Dependabot alerts returned
disabled (HTTP 403) and code scanning returned no analysis (HTTP 404); both queries also reported
missing `admin:repo_hook`, so alert details remain `PERMISSION`-limited. Workflow references below
are source interfaces; repository counts do not prove organization or other hidden credentials absent.

## Complete inventory

| Workflow | Trigger | Effective permissions | Secret names/interfaces | Environments | External mutation | Upstream coupling | Fork value / risk | Disposition | Validation |
|---|---|---|---|---|---|---|---|---|---|
| `.github/workflows/approve-contributor.yml` | `issue_comment: created` | Job: `contents: write`, `issues: write`, `pull-requests: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Edits `.github/APPROVED_CONTRIBUTORS`, commits/pushes default branch, comments on issue | Upstream contributor approval file/process and maintainer semantics | No readiness value / can mutate default branch and public issue state | **DISABLE** | TASK-GUIDE-003: upstream-only identity guard implemented on `approve`; static guard, trigger, permission, and pin validation passed. Runtime evidence remains pending. |
| `.github/workflows/build-binaries.yml` | Push tag `v*`; manual dispatch with `tag`/`source_ref` | Top `{}`; jobs use `contents: read/write`, `actions: read`, `id-token: write` | `GITHUB_TOKEN`; `PI_ARTIFACTS_R2_ACCESS_KEY_ID`; `PI_ARTIFACTS_R2_SECRET_ACCESS_KEY`; npm OIDC trusted-publishing interface | `npm-publish`; `pi-model-upload` | Builds/uploads artifacts; creates/deletes/publishes GitHub Releases; publishes npm packages; writes pi.dev release marker to R2 | Earendil npm identities, pi.dev/R2 endpoint, release process and protected environments | No fork-readiness value / irreversible public distribution and credential-dependent writes | **DISABLE** | TASK-GUIDE-003: upstream-only guard implemented on all seven jobs, including `always()` cleanup; v0.85.1 packed-consumer smoke step preserved. Static validation passed; runtime evidence remains pending. |
| `.github/workflows/ci.yml` | Push to `main` or `codex/pi-fork-production-readiness`; pull request to `main`; manual dispatch | Workflow: `contents: read`; checkout uses `persist-credentials: false` | Automatic read-only `GITHUB_TOKEN` interface; not persisted by checkout | None | No repository/external write declared; installs, builds, checks, tests | Canonical repository scripts only | Essential candidate health evidence / runner and supply-chain execution risk bounded by reviewed source | **KEEP** | TASK-DEPLOY-001 path: empty manual dispatch, exact readiness-branch push trigger, workflow-level read permission, and non-persistent checkout token; pull-request gate, jobs, pins, and commands retained. Prove workflow source, checkout, and run SHA equal the pushed candidate source. |
| `.github/workflows/issue-analysis.yml` | Issue labeled; issue comment created | Workflow: `contents: read`, `issues: write` | `EARENDIL_ORG_READ_TOKEN`; `PI_AUTH_JSON`; `PI_AUTH_UPDATE_TOKEN`; `PI_GIST_TOKEN`; automatic `GITHUB_TOKEN` | `pi-analyze` | Reads Earendil staff membership; labels/comments issues; runs credentialed Pi; updates environment secret; creates private gist | `earendil-works/staff`, `@issuron`, pi.dev session sharing, Earendil-owned credential process | No readiness value / organization auth, model credentials, secret write, gist and issue mutations | **DISABLE** | TASK-GUIDE-003: upstream-only guard implemented on `authorize` and `analyze`, composed with the existing analysis condition; static validation passed. Runtime evidence remains pending. |
| `.github/workflows/issue-gate.yml` | Issue opened | Job: `contents: read`, `issues: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Comments, labels, and closes public issues | Upstream approved-contributor file, trusted bot list, contribution policy | No readiness value / can close legitimate fork issues and send upstream-branded guidance | **DISABLE** | TASK-GUIDE-003: upstream-only identity guard implemented on `check-contributor`; static guard, trigger, permission, and pin validation passed. Runtime evidence remains pending. |
| `.github/workflows/issue-triage-labels.yml` | Issue reopened or labeled | Job: `issues: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Adds/removes labels and closes issues | Upstream label taxonomy and triage flow | No readiness value / bulk public issue mutation with unadopted governance | **DISABLE** | TASK-GUIDE-003: upstream-only guard implemented on `update-labels`; static guard, trigger, permission, and pin validation passed. Runtime evidence remains pending. |
| `.github/workflows/npm-audit.yml` | Daily schedule `37 7 * * *`; manual dispatch | Workflow: `contents: read` | None | None | No external write; installs without lifecycle scripts, queries npm audit/signature services | Canonical npm registry/security policy only | Essential dependency/signature evidence / registry availability and changing advisory data | **KEEP** | Run on exact candidate SHA; both audit commands exit 0; record run ID/UTC; full-SHA action pins remain. **Currently BLOCKED: zero runs.** |
| `.github/workflows/pr-gate.yml` | `pull_request_target: opened` | Job: `contents: read`, `issues: write`, `pull-requests: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Comments on and closes public PRs | Upstream approved-contributor gate, bot list, contribution policy | No readiness value / privileged base-context event can close fork PRs and post upstream policy | **DISABLE** | TASK-GUIDE-003: upstream-only guard implemented on `check-contributor`; static guard, trigger, permission, and pin validation passed. Runtime evidence remains pending. |
| `.github/workflows/publish-model-catalog.yml` | Successful `CI` workflow run on `main`; selected PR paths; weekday schedule `17 8-13 * * 1-5`; manual dispatch | Workflow: `contents: read` | `PI_ARTIFACTS_R2_ACCESS_KEY_ID`; `PI_ARTIFACTS_R2_SECRET_ACCESS_KEY` | `pi-model-upload` | Uploads GitHub artifact; may write production model catalog to R2 | pi.dev production R2 bucket/endpoint, Vienna publication window, upstream CI name | Generation has diagnostic value but publication is unnecessary / automatic chain can mutate production R2 | **DISABLE** | TASK-GUIDE-003: upstream-only guard implemented on `generate` and `publish`, composed with their existing conditions; static validation passed. Runtime evidence remains pending. |
| `.github/workflows/remove-inprogress-on-close.yml` | Issue closed | Job: `issues: write` | Automatic `GITHUB_TOKEN` interface; no named stored secret | None | Removes `inprogress` label from public issue | Upstream issue label taxonomy | No readiness value / unadopted public issue mutation | **DISABLE** | TASK-GUIDE-003: upstream-only guard implemented on `remove-label`; static guard, trigger, permission, and pin validation passed. Runtime evidence remains pending. |

## Acceptance and follow-up

- Inventory cardinality is exactly ten unique current workflow paths: two KEEP and eight DISABLE.
- The completed local TASK-GUIDE-003 delta guards every DISABLE job, not only workflow entrypoints or a subset
  of dependency roots; immutable-candidate and post-merge runtime evidence remain pending.
- Existing job conditions, including `always()` cleanup and `workflow_run` filters, must remain
  effective after composing the repository-identity condition.
- `npm-audit.yml` remains byte-unchanged. The authorized `ci.yml` KEEP delta is limited to an empty
  `workflow_dispatch`, the exact `codex/pi-fork-production-readiness` push addition, workflow-level
  `contents: read`, and checkout `persist-credentials: false`; `main` push, `main` pull-request,
  jobs, pins, and commands remain unchanged. An authorized candidate push must prove workflow source,
  checkout, and run SHA equality before TEST-005 can PASS.
- Full-SHA action pins and existing explicit permission scopes must not be weakened.
- Static guards are necessary but not sufficient: candidate/post-merge run evidence must show no
  DISABLE job or unauthorized external effect in the fork.
