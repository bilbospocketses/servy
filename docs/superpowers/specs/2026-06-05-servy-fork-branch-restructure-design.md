# servy fork — branch restructure for at-will upstream sync

- **Date:** 2026-06-05
- **Status:** Approved (design) — execution gated on runbook review
- **Repo:** `bilbospocketses/servy` (hard fork of `aelassas/servy`, MIT)
- **Author/owner:** Jamie Chapman (fork persona: Bilbo Baggins)

> **As-built note (2026-06-05):** two method details differ from the steps as written below. (1) Ruleset `16655779` is **`~DEFAULT_BRANCH`**, so swapping the GitHub default to `vnext` moved protection automatically — the "migrate/PUT ruleset include" steps (§3 / Task 4) were effectively a **no-op**; the only ruleset PUTs actually applied were the cosmetic rename ("Protect default branch") and updating the required-check *contexts* to `Build (x64)` + `Test (x64)` after upstream's CI matrix refactor renamed the jobs. (2) `CHANGELOG.md` used a history-preserving merge — upstream's full current history **plus** the fork preface — not a blanket "ours". Authoritative as-built record: `CHANGELOG.md` + `archive/todo_servy_shipped.md`.

## Context & problem

Upstream `aelassas/servy` is extremely active (`origin/main` is **192 commits behind**
`upstream/main` as of this date). The fork is **not yet ready to hard-fork** — we want to
keep absorbing upstream's new features at will until the cross-platform / multi-arch /
merge-with-velopack vision work (todo `#2`) begins in earnest.

Today `origin/main` carries the fork divergence directly (lockdown + docs rebrand), so it has
**diverged from upstream and can no longer be fast-forward-synced**. Every upstream catch-up
would be a conflict-laden merge into the protected default branch.

This restructure adopts the **same topology already proven on the `abs-app` fork** (and being
set up in parallel on the `velopack` sister fork), so all three forks share one shape — which
matters because servy + velopack are intended to eventually merge into a single multi-OS/arch
installation-and-service-management tool.

## Goal

Make a clean, **at-will upstream-syncable** branch the upstream mirror, and move all fork work
onto a separate locked default branch — so upstream features flow in with a one-command sync,
and fork work proceeds on its own protected line (and branches off it).

## Chosen approach — A (abs-app-exact)

A new branch becomes the locked default; `main` is reset to a pristine, unprotected upstream
mirror. (The lower-risk inverse — keep `main` as default + a side mirror branch — was
considered and rejected for breaking cross-fork topology symmetry with abs-app/velopack.)

**Locked decisions:**
1. Fork-default branch name: **`vnext`** (matches velopack's fork-default; the upstream mirror
   keeps the upstream's own default branch name — servy's is `main`).
2. Doc-conflict resolution during catch-up: **fork-rebranded versions win ("ours")** on the
   rebranded docs.
3. Upstream sync: **manual `gh repo sync` at will** (matches abs-app; no sync workflow).

## Current-state facts (verified 2026-06-05)

- `origin/main` is **2 commits ahead** of `upstream/main`, **192 behind**:
  - `231c0a9b` "lockdown smoke I.2 (#1)" — **empty commit** (zero files); a lockdown smoke
    marker. **Dropped** in catch-up.
  - `bd913daa` "docs: rebrand … (#3)" — the only real divergence; touches 9 files:
    `README.md`, `NOTES.md`, `ROADMAP.md`, `VISION.md` (new), `CHANGELOG.md`,
    `setup/{choco,scoop,winget}/README.md`, and `.github/workflows/test.yml` (the PR-trigger fix).
- `upstream` remote already wired (`https://github.com/aelassas/servy.git`).
- Branch ruleset `16655779` targets `refs/heads/main`; tag ruleset `16655783` targets
  `refs/tags/v*`.

## Target topology

```
BEFORE                                AFTER
------                                -----
main  (default, LOCKED)               vnext  (default, LOCKED)   <- all fork work + releases
  upstream-base                          = upstream/main (192) + docs-rebrand (replayed, signed)
  + empty smoke + rebrand                ^ merge from main to catch up on future upstream
  192 behind upstream                    |
                                      main  (unprotected mirror)
upstream/main --------------------->     = exact upstream/main, 0 ahead
                                         <- gh repo sync at will
```

## Design

### 1. Branch model
- **`vnext`** — GitHub default, carries ruleset `16655779` (deletion, non_fast_forward,
  required_linear_history, required_signatures, pull_request, required_status_checks =
  `Build (x64)` + `Test (x64)`). All fork work + releases live here; feature branches fork off it.
- **`main`** — unprotected mirror of `upstream/main`, 0 ahead, synced at will. Never receives
  fork commits.

### 2. Catch-up strategy (192 commits)
Catch `vnext` (currently at `origin/main`) up to `upstream/main` by **rebasing it onto
`upstream/main`** — dropping the empty smoke commit (`231c0a9b`) and **replaying only `bd913daa`**
(the rebrand) on top (e.g. `git rebase --onto upstream/main 231c0a9b vnext`). Conflict resolution:
- **`README.md`, `NOTES.md`, `ROADMAP.md`, `CHANGELOG.md`, and `setup/{choco,scoop,winget}/README.md`
  → keep the fork-rebranded versions ("ours").** They are a deliberate identity rewrite;
  upstream's new feature-doc text can be folded back in later if desired.
- **`VISION.md`** is new → no conflict.
- **`.github/workflows/test.yml`** → take upstream's version, then re-apply the trigger fix in
  the workflow audit (Section 4); do not resolve the same conflict twice.

Result: `vnext` is linear, current with upstream, rebrand on top, fully signed.

### 3. Ruleset + default migration
- **Retarget ruleset `16655779`** `refs/heads/main` -> `refs/heads/vnext` (PUT; keep all
  rules incl. required checks). After this, `main` is unprotected and `vnext` is locked.
- **Swap GitHub default** -> `vnext`.
- Tag ruleset `16655783` (`v*`) unchanged.

### 4. Workflow audit

| Workflow | Trigger | Action |
|---|---|---|
| `build`, `test` | push/PR `main` | **Retarget -> `vnext`** (required checks; keeps the deadlock fixed on the new default) |
| `security` (CodeQL) | push/PR `main` + weekly | Retarget push/PR -> `vnext`; keep schedule |
| `loc` | push `main` | Retarget -> `vnext` |
| `sonar` | push `main` | **Disable repo-wide** (`gh workflow disable`) — upstream's SonarCloud (`aelassas_servy`/`aelassas`), unusable in fork, failing since 2026-05-20 |
| `choco` / `scoop` / `winget` | `release` | No action — only fire on a release; gated by the deferred packaging rebrand (todo `#4`) |
| `publish` / `release` / `sbom` / `changelog` / `bump-version` / `wiki` / `dotnet-reflection` | `workflow_dispatch` | No action (manual) |

**Accepted side-effect:** syncing `main` redundantly runs upstream's `build`/`test`/`security`
on the mirror (they pass — it is upstream's own code; cannot be suppressed without diverging the
mirror). `sonar` is disabled repo-wide so it will not recur anywhere.

### 5. Sync mechanism
Manual, at will:
```
gh repo sync bilbospocketses/servy --branch main --source aelassas/servy
```
`main` never diverges, so this always fast-forwards. (A scheduled nightly auto-sync workflow is
an easy future add if the manual step gets old.)

### 6. Execution sequence (order is safety-critical)
1. Local hygiene: ensure `vnext` exists from current `origin/main`; the spec/plan docs live here.
2. Catch up `vnext`: rebase onto `upstream/main`, dropping the empty smoke commit, replaying
   the rebrand; resolve conflicts per Section 2.
3. Apply workflow retargets (Section 4) + commit on `vnext` (signed, no AI attribution).
4. Push `vnext`; set its upstream tracking to `origin/vnext`.
5. Migrate ruleset `16655779` `main` -> `vnext`; swap GitHub default -> `vnext`.
6. **Now** force-reset `main` -> `upstream/main` (allowed — `main` is unprotected).
7. `gh workflow disable sonar.yml`.
8. Local cleanup: track new `main` mirror, delete merged `docs/fork-rebrand`, prune.

The signature ordering works because `vnext` is fully built + pushed **before** it is locked:
the 192 unsigned upstream commits are grandfathered, and only future pushes must be signed
(exactly how abs-app's locked default already operates).

### 7. Post: memory + docs
- This spec + the writing-plans runbook committed on `vnext`.
- Breadcrumb: clear the stale "merge PR #3 -> Item #2" handoff; record the restructure.
- `todo_servy`: archive PR #3 shipped; add the new branch model + the restructure outcome.
- `project_servy`: Fork Divergence (new default `vnext`, workflow retargets, sonar disabled) +
  Lockdown State (ruleset `16655779` now targets `vnext`).
- `project_index.md`: refresh the servy breadcrumb line.

## Risks & trade-offs
- **Rebranded docs re-conflict** on every future upstream sync of those files (acceptable, by
  design — they are intentional fork divergence; resolution is always "ours").
- **Mirror redundant CI** (see Section 4 accepted side-effect).
- **Default-branch swap** changes clone/PR defaults to `vnext` — intended.

## Out of scope
- Packaging rebrand (winget/choco/scoop manifests, `.iss`, signing) — todo `#4`, gated on fork
  releases.
- Hard-fork vision work — proceeds on `vnext` (feature branches off it) when todo `#2` (initiative brainstorm) starts.
- Automated upstream-sync workflow — optional future enhancement.

## Verification (definition of done)
- `vnext` is the GitHub default, locked by ruleset `16655779`; a smoke PR into `vnext` runs
  and passes both `Build (x64)` + `Test (x64)` (proving the trigger retarget).
- `main` is 0 commits ahead of `upstream/main` and unprotected; `gh repo sync --branch main`
  fast-forwards cleanly.
- `sonar` shows disabled; no new sonar failures.
- `vnext` contains upstream's 192 commits + the rebrand, linear history, HEAD signed.
