# servy fork branch-restructure — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: this is a stateful, sequential ops runbook — execute **inline** with `superpowers:executing-plans` (NOT subagent-per-task; the steps share live repo + GitHub state and need judgment at conflict/verify points). Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make `vnext` the locked fork-default carrying all fork work (caught up to upstream), and reset `main` to a pristine, at-will-syncable mirror of `upstream/main` — the abs-app topology.

**Architecture:** Build + lock `vnext` *before* unprotecting/resetting `main`, so the 192 unsigned upstream commits are grandfathered and only future pushes need signing. All outward mutations are back-loaded (Tasks 3+); Tasks 0–2 are local-only and reversible by deleting the local branch.

**Tech Stack:** git, `gh` CLI, GitHub Rulesets REST API (PUT), PowerShell. SSH-signed commits, no AI attribution.

**Spec:** `docs/superpowers/specs/2026-06-05-servy-fork-branch-restructure-design.md`

**Conventions (every command):**
- `git -C "C:/Users/jscha/source/repos/servy" …` (absolute path; multi-session safety)
- `gh … -R bilbospocketses/servy`
- Commits: signed (`commit.gpgsign=true` already set), conventional style, **zero AI references**
- Scratch files: `C:/Users/jscha/AppData/Local/ClaudeScratch/` (never `%TEMP%`)

---

> **As-built note (2026-06-05):** two method details differ from the steps as written below. (1) Ruleset `16655779` is **`~DEFAULT_BRANCH`**, so swapping the GitHub default to `vnext` moved protection automatically — the "migrate/PUT ruleset include" steps (§3 / Task 4) were effectively a **no-op**; the only ruleset PUTs actually applied were the cosmetic rename ("Protect default branch") and updating the required-check *contexts* to `Build (x64)` + `Test (x64)` after upstream's CI matrix refactor renamed the jobs. (2) `CHANGELOG.md` used a history-preserving merge — upstream's full current history **plus** the fork preface — not a blanket "ours". Authoritative as-built record: `CHANGELOG.md` + `archive/todo_servy_shipped.md`.

## Pre-flight state (as of 2026-06-05, pre-execution)
- Local `vnext` @ rebrand `bd913daa` + spec `7e2d77fb` (+ this plan commit), tracking `origin/main` (fixed in Task 3), **not pushed**.
- `origin/main` @ `bd913daa` = 2 ahead of `upstream/main` (`231c0a9b` empty smoke + `bd913daa` rebrand), 192 behind.
- Ruleset `16655779` → `refs/heads/main`; tag ruleset `16655783` → `refs/tags/v*`. Default branch = `main`.

---

### Task 0: Pre-flight verification + rollback snapshot

**Files:** none (read-only + scratch snapshot)

- [ ] **Step 1: Refresh remotes + confirm starting state**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo fetch upstream --quiet; git -C $repo fetch origin --quiet
git -C $repo branch --show-current            # expect: vnext
git -C $repo log --oneline -3 vnext
git -C $repo rev-list --left-right --count origin/main...upstream/main   # expect: 2  <N>=192+
gh repo view bilbospocketses/servy --json defaultBranchRef --jq .defaultBranchRef.name  # expect: main
```
Expected: on `vnext`; `vnext` HEAD is the plan commit, then spec `7e2d77fb`, then `bd913daa`; default branch `main`.

- [ ] **Step 2: Snapshot the ruleset for rollback**

```powershell
gh api repos/bilbospocketses/servy/rulesets/16655779 > "C:/Users/jscha/AppData/Local/ClaudeScratch/servy-ruleset-16655779-pre.json"
Get-Content "C:/Users/jscha/AppData/Local/ClaudeScratch/servy-ruleset-16655779-pre.json" | Select-String 'refs/heads/main'  # confirm capture
```
Expected: file written; the `include` shows `refs/heads/main`. **Rollback anchor:** `origin/main` = `bd913daa`; ruleset target = `refs/heads/main`; default = `main`.

---

### Task 1: Catch `vnext` up to `upstream/main` (rebase + conflict resolution)

**Files (conflict resolution targets):** `README.md`, `NOTES.md`, `ROADMAP.md`, `VISION.md`, `CHANGELOG.md`, `setup/{choco,scoop,winget}/README.md`, `.github/workflows/test.yml`

- [ ] **Step 1: Rebase vnext onto upstream/main, dropping the empty smoke commit**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo rebase --onto upstream/main 231c0a9b vnext
```
Expected: stops with CONFLICT on the `bd913daa` replay (rebranded docs vs upstream's 192-commit changes). The spec + plan commits replay cleanly afterward (new files).

- [ ] **Step 2: Resolve — fork-rebranded docs win ("ours"); test.yml takes upstream**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
# Keep the fork rebrand verbatim for prose + setup docs (and the new VISION.md):
git -C $repo checkout bd913daa -- README.md NOTES.md ROADMAP.md VISION.md setup/choco/README.md setup/scoop/README.md setup/winget/README.md
# test.yml: take upstream's current version (retargeted to vnext in Task 2):
git -C $repo checkout upstream/main -- .github/workflows/test.yml
```

- [ ] **Step 3: CHANGELOG special-case — keep upstream's full history + re-apply the fork preface**

Blanket "ours" would drop upstream's 192 new CHANGELOG entries. Instead take upstream's CHANGELOG and prepend the fork preface block.

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo checkout upstream/main -- CHANGELOG.md          # upstream's full changelog (keeps 192 entries)
git -C $repo show bd913daa:CHANGELOG.md > "C:/Users/jscha/AppData/Local/ClaudeScratch/servy-changelog-fork.md"
```
Then open the scratch file, copy the fork preface block (everything above upstream's first `## [` release heading), and prepend it to `CHANGELOG.md`. Verify the result has BOTH the preface AND upstream's latest release entries.

> **EXECUTION CHECKPOINT:** confirm with the user that the CHANGELOG = fork preface + upstream's full history before continuing (this refines the approved "ours-win" for the append-only changelog).

- [ ] **Step 4: Stage resolutions + continue the rebase**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo add README.md NOTES.md ROADMAP.md VISION.md CHANGELOG.md setup/choco/README.md setup/scoop/README.md setup/winget/README.md .github/workflows/test.yml
git -C $repo status --short        # expect: no remaining "UU"/conflict markers
git -C $repo rebase --continue
```
Expected: rebase completes; spec + plan commits land on top.

- [ ] **Step 5: Verify vnext is current + linear + rebrand intact**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo rev-list --count vnext..upstream/main          # expect: 0  (vnext contains all upstream)
git -C $repo log --oneline -5 vnext                          # expect: plan, spec, rebrand on top of upstream HEAD
git -C $repo log --format='%G?' -1 vnext                     # expect: G (HEAD signed)
git -C $repo grep -l "bilbospocketses" -- README.md            # expect: README.md (rebrand survived)
```
Expected: `0` commits behind upstream; HEAD signed; rebrand present. **Do NOT proceed if `vnext..upstream/main` != 0.**

---

### Task 2: Retarget workflow triggers on `vnext`

**Files (Modify):** `.github/workflows/build.yml`, `test.yml`, `security.yml`, `loc.yml`

> These now hold upstream's *current* trigger blocks. For each, change the branch filter `main` -> `vnext` in `push` and `pull_request`; preserve `schedule`, `workflow_dispatch`, and everything else. (`sonar` is handled by repo-wide disable in Task 7, not edited here.)

- [ ] **Step 1: Inspect current trigger blocks (they are upstream's latest, post-rebase)**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
foreach ($f in 'build','test','security','loc') {
  Write-Output "===== $f.yml ====="; Get-Content "$repo/.github/workflows/$f.yml" -TotalCount 25
}
```
Use the printed `on:` blocks to write exact Edits. Target end-state for `build`/`test` (`security` additionally keeps its `schedule:` + cron; `loc` keeps only `push`):
```yaml
on:
  push:
    branches:
      - vnext
  pull_request:
    branches:
      - vnext
  workflow_dispatch:
```

- [ ] **Step 2: Apply the edits** (Edit tool, per file — replace `- main` / `[ "main" ]` branch entries with `vnext`, leaving schedule/dispatch intact). Then verify none still gate on `main`:

```powershell
$repo="C:/Users/jscha/source/repos/servy"
Select-String -Path "$repo/.github/workflows/build.yml","$repo/.github/workflows/test.yml","$repo/.github/workflows/security.yml","$repo/.github/workflows/loc.yml" -Pattern '^\s*-?\s*"?main"?\s*$|branches:\s*\[\s*"?main"?' 
```
Expected: **no matches** (all retargeted to `vnext`).

- [ ] **Step 3: Commit (signed, no AI attribution)**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo add .github/workflows/build.yml .github/workflows/test.yml .github/workflows/security.yml .github/workflows/loc.yml
git -C $repo commit -m "ci: retarget build/test/security/loc workflows from main to vnext (fork-default)"
git -C $repo log -1 --format='%h sig=%G? %s'    # expect: sig=G
```

---

### Task 3: Push `vnext` (FIRST remote mutation) + fix tracking

- [ ] **Step 1: Push vnext and set its tracking to origin/vnext**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo push -u origin vnext
```
Expected: `vnext -> vnext` created on origin; local now tracks `origin/vnext` (not `origin/main`).

- [ ] **Step 2: Verify**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo status -sb | Select-Object -First 1     # expect: ## vnext...origin/vnext
gh api repos/bilbospocketses/servy/branches/vnext --jq .name   # expect: vnext
```

---

### Task 4: Migrate ruleset `16655779` `main` -> `vnext`

**Method:** GET → edit `conditions.ref_name.include` → PUT only the editable fields (per `master_github_api`).

- [ ] **Step 1: Build the PUT body from the snapshot, retargeted to vnext**

```powershell
$rs = Get-Content "C:/Users/jscha/AppData/Local/ClaudeScratch/servy-ruleset-16655779-pre.json" -Raw | ConvertFrom-Json
$rs.conditions.ref_name.include = @("refs/heads/vnext")
$body = $rs | Select-Object name, target, enforcement, conditions, rules, bypass_actors | ConvertTo-Json -Depth 30
Set-Content -Path "C:/Users/jscha/AppData/Local/ClaudeScratch/servy-ruleset-vnext.json" -Value $body -Encoding utf8
Select-String -Path "C:/Users/jscha/AppData/Local/ClaudeScratch/servy-ruleset-vnext.json" -Pattern 'refs/heads/vnext'  # confirm
```

- [ ] **Step 2: PUT the retargeted ruleset**

```powershell
gh api -X PUT repos/bilbospocketses/servy/rulesets/16655779 --input "C:/Users/jscha/AppData/Local/ClaudeScratch/servy-ruleset-vnext.json"
```

- [ ] **Step 3: Verify ruleset now guards vnext, not main**

```powershell
gh api repos/bilbospocketses/servy/rulesets/16655779 --jq '{name,enforcement,include:.conditions.ref_name.include,rules:[.rules[].type]}'
```
Expected: `include: ["refs/heads/vnext"]`; rules still include `required_signatures`, `pull_request`, `required_status_checks` (contexts `Build (x64)` + `Test (x64)`). `main` is now unprotected.

---

### Task 5: Swap GitHub default branch -> `vnext`

- [ ] **Step 1: Set default**

```powershell
gh repo edit bilbospocketses/servy --default-branch vnext
gh repo view bilbospocketses/servy --json defaultBranchRef --jq .defaultBranchRef.name   # expect: vnext
```

---

### Task 6: Reset `main` to the upstream mirror

`main` is now unprotected (Task 4), so the force-sync is allowed.

- [ ] **Step 1: Force-sync origin/main from upstream (same command used for future at-will syncs, +`--force` for this one-time divergence discard)**

```powershell
gh repo sync bilbospocketses/servy --branch main --source aelassas/servy --force
```
Expected: `main` updated to match `aelassas/servy:main`.

- [ ] **Step 2: Verify main is a pristine 0-ahead mirror**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo fetch origin --quiet
git -C $repo rev-list --left-right --count origin/main...upstream/main   # expect: 0  0
```
Expected: `0  0` (main == upstream/main exactly). **Future syncs:** same command without `--force`.

- [ ] **Step 3: Realign local main**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo branch -f main origin/main
```

---

### Task 7: Disable the `sonar` workflow repo-wide

- [ ] **Step 1: Disable + verify**

```powershell
gh workflow disable sonar.yml -R bilbospocketses/servy
gh workflow list -R bilbospocketses/servy | Select-String -Pattern 'sonar'   # expect: sonar ... disabled_manually
```
Expected: `sonar` shows disabled. It will no longer run on any branch/event (incl. future `main` syncs).

---

### Task 8: Smoke-verify the locked default end-to-end

Proves the required checks actually run on PRs into `vnext` (the deadlock-reborn risk) before we call it done.

- [ ] **Step 1: Branch off vnext, make a trivial CHANGELOG note**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo switch -c chore/restructure-smoke vnext
```
Add a one-line `[Unreleased]` note to `CHANGELOG.md` recording the branch-restructure (fork-default `vnext` + mirror `main`). Commit (signed):
```powershell
git -C $repo add CHANGELOG.md
git -C $repo commit -m "docs(changelog): note fork branch restructure (vnext default + main upstream mirror)"
git -C $repo push -u origin chore/restructure-smoke
```

- [ ] **Step 2: Open PR into vnext; confirm BOTH required checks run + pass**

```powershell
gh pr create -R bilbospocketses/servy --base vnext --head chore/restructure-smoke --title "chore: changelog note for branch restructure" --body "Smoke + changelog. Validates required checks run on vnext PRs."
# poll (loop, no background watch):
gh pr checks <N> -R bilbospocketses/servy
```
Expected: `Build (x64)` + `Test (x64)` both run and pass (mergeStateStatus CLEAN). **If `Test (x64)`/`Build (x64)` do NOT appear -> trigger retarget (Task 2) is wrong; fix before merging.**

- [ ] **Step 3: Squash-merge (signed web-flow), branch auto-deletes**

```powershell
gh pr merge --squash <N> -R bilbospocketses/servy
gh pr view <N> -R bilbospocketses/servy --json state --jq .state   # expect: MERGED
```

---

### Task 9: Local cleanup + memory/doc updates

- [ ] **Step 1: Prune + realign local, delete the merged rebrand branch**

```powershell
$repo="C:/Users/jscha/source/repos/servy"
git -C $repo fetch origin --prune
git -C $repo switch vnext
git -C $repo pull --ff-only            # absorb the smoke squash on vnext
git -C $repo branch -D docs/fork-rebrand
git -C $repo branch -a
```
Expected: local on `vnext`; `docs/fork-rebrand` gone; remote-tracking pruned.

- [ ] **Step 2: Update memory (Edit/Write — outside the repo, `~/.claude/...memory/`)**
- `breadcrumb_servy.md`: clear the stale "merge PR #3 -> Item #2"; record restructure shipped; next = Item #2.
- `todo_servy.md`: archive the PR #3 / restructure work to `archive/todo_servy_shipped.md`; record the new branch model (vnext default, main mirror, manual `gh repo sync`).
- `project_servy.md`: Fork Divergence (default `vnext`; workflow retargets; sonar disabled) + Lockdown State (ruleset `16655779` now targets `vnext`); add the sync command.
- `project_index.md`: refresh the servy breadcrumb line + todo count/date.

---

## Final verification (definition of done)
```powershell
$repo="C:/Users/jscha/source/repos/servy"
gh repo view bilbospocketses/servy --json defaultBranchRef --jq .defaultBranchRef.name        # vnext
gh api repos/bilbospocketses/servy/rulesets/16655779 --jq .conditions.ref_name.include        # ["refs/heads/vnext"]
git -C $repo rev-list --left-right --count origin/main...upstream/main                          # 0  0
gh workflow list -R bilbospocketses/servy | Select-String sonar                                # disabled
git -C $repo log --format='%G?' -1 vnext                                                      # G
```
All five must match. Task 8 already proved the vnext PR checks run + pass.

## Rollback guide (if aborted before Task 5)
- Tasks 0–2 local-only: `git -C $repo rebase --abort` (mid-rebase) or `git -C $repo branch -D vnext` (after) — nothing on origin yet.
- After Task 3 (vnext pushed) but before Task 4: `gh api -X DELETE repos/bilbospocketses/servy/git/refs/heads/vnext`.
- After Task 4 (ruleset moved): re-PUT the snapshot `servy-ruleset-16655779-pre.json` to restore `main` targeting.
- After Task 5 (default swapped): `gh repo edit bilbospocketses/servy --default-branch main`.
- Task 6 is the point of no easy return for `main`'s 2 fork commits — but they are preserved on `vnext` (rebrand) / intentionally dropped (empty smoke), so there is nothing to recover.
