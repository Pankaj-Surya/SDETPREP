## Git, Version Control & Branching Strategy

## Q1. What is the version control strategy (Git) you follow?

**Plain English**
- A version control strategy defines how your team organizes branches, merges code, and releases changes — so multiple people can work on the same codebase without stepping on each other's work, and releases stay stable

**Most common strategy in real teams — Git Flow / trunk-based with feature branches**
- `main`/`master` branch — always holds stable, production-ready code
- `develop` branch — integration branch where features come together before a release
- `feature/*` branches — one branch per feature or bug fix, created from `develop`
- `release/*` branches — created from `develop` when preparing a release, used for final testing/bug fixes before merging to `main`
- `hotfix/*` branches — created directly from `main` for urgent production fixes, then merged back to both `main` and `develop`

**Real-time example — banking**
- A developer creates `feature/schedule-recurring-transfer` branch off `develop` to build the new recurring transfer feature. Once done and reviewed, it merges into `develop`. Before the monthly release, a `release/v2.5` branch is cut from `develop` for final QA regression testing. Once approved, it merges into `main` and gets deployed to production. If a critical bug is found in production (e.g., wrong interest calculation), a `hotfix/interest-calc-fix` branch is created directly from `main`, fixed, tested, and merged into both `main` and `develop`.

## Q2. Explain version control strategies

**Plain English overview of common strategies (broader than just one team's workflow)**

- **Git Flow** — the structured model described in Q1 above (main, develop, feature, release, hotfix branches); good for projects with scheduled releases and longer testing cycles
- **Trunk-Based Development** — everyone commits small, frequent changes directly (or via very short-lived branches) into a single `main`/`trunk` branch; relies heavily on feature flags to hide incomplete work and strong CI/automated testing to keep `main` always deployable; suits teams deploying multiple times a day
- **GitHub Flow** — a simpler model: `main` is always deployable, every change goes through a short-lived feature branch and a pull request, then merges straight to `main` and deploys; no separate `develop`/`release` branches; popular with teams doing continuous deployment
- **Forking Workflow** — each contributor works on their own full copy (fork) of the repo, then submits changes via pull request to the main repo; common in open-source projects, less common in typical internal company teams

**Real-time example — e-commerce**
- A fast-moving e-commerce startup deploying 5–10 times a day uses **Trunk-Based Development** with feature flags — a new "buy now pay later" feature is merged into `main` early but hidden behind a flag, tested in production with 1% of users, then gradually rolled out — versus a large bank with monthly release cycles and heavy compliance/regression needs, which is better suited to **Git Flow** with dedicated release branches for thorough testing before each release.

## Q3. Git merge vs rebase, and handling merge conflicts (with examples)

**Merge — plain English**
- Combines two branches by creating a new "merge commit" that ties both histories together — keeps the full original history of both branches, including all individual commits, exactly as they happened

**Rebase — plain English**
- Takes your branch's commits and replays them one by one on top of the latest code from the target branch — creates a clean, linear history, as if you'd started your work *after* all the latest changes, with no separate "merge commit"

**Key difference to remember**
- Merge = "combine and keep both histories as they actually happened" (messier but more truthful)
- Rebase = "rewrite my commits as if I started fresh from the latest code" (cleaner history but rewrites commit history)

**Real-time example — banking**
- Two developers are working on `develop`: Developer A adds a new "block card" feature, Developer B adds a new "set spending limit" feature, both branching off the same starting point
- If Developer A **merges** `develop` into their branch before finishing, Git creates a merge commit showing both sets of changes coming together, preserving exactly how it happened
- If Developer A **rebases** their branch onto the latest `develop` instead, their "block card" commits get replayed on top of B's already-merged "spending limit" commits, making it look like A started their work after B's changes — clean linear history, easier to read later

**Handling merge conflicts — real-time example**
- Both Developer A and Developer B modify the same line in `AccountService.java` — A changes a validation rule, B changes a formatting rule on the exact same line
- Git can't automatically decide which change is correct, so it marks the file as conflicted, showing both versions with conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
- Resolution: open the file, manually review both changes, decide the correct combined version (or talk to the other developer if unclear), remove the conflict markers, save the file, then `git add` the resolved file and continue the merge/rebase (`git commit` for merge, `git rebase --continue` for rebase)

## Q4. Git commands you have used

**Your list is correct — here's the cleaned-up, correctly formatted version with plain-English purpose**

- `git add <filename>` or `git add .` — stage changes for commit
- `git commit -m "message"` — save staged changes to local history with a message
- `git push` — send local commits to the remote repository
- `git pull` — fetch and merge remote changes into your current local branch
- `git checkout <branch>` — switch to an existing branch
- `git checkout -b <branch>` — create a new branch and switch to it
- `git fetch` — download remote changes without merging them into your branch yet
- `git merge <branch>` — combine another branch's changes into your current branch
- `git diff` — show the differences between changes (note: it's `git diff`, not `git --diff`)
- `git log` — view commit history
- Additional commonly used ones worth mentioning: `git status` (see current changes/staged files), `git stash` (temporarily save uncommitted changes), `git branch` (list branches), `git rebase` (replay commits on top of another branch), `git reset` (undo commits/staged changes), `git revert` (safely undo a commit by creating a new opposite commit)

## Q5. How do you resolve conflicts in Git?

**Step-by-step**
- Run `git status` to see which files are conflicted
- Open each conflicted file — Git marks the conflicting sections with `<<<<<<< HEAD`, `=======`, and `>>>>>>> branch-name`
- Manually review both versions and decide the correct final content — sometimes it's picking one side, sometimes it's combining both
- Remove the conflict markers once resolved
- Stage the resolved file with `git add <filename>`
- Complete the operation — `git commit` if it was a merge, or `git rebase --continue` if it was a rebase
- Test the code after resolving to confirm nothing broke in the merge

**Real-time example — e-commerce**
- Merging a `feature/discount-coupon` branch into `develop` conflicts on `PricingService.java` because `develop` already had a change to the same discount calculation function. After manually reviewing, the developer realizes both changes need to coexist — combines both pieces of logic, tests locally to confirm coupon + existing discount logic both still work correctly, then commits the resolved merge.

## Q6. Git commands: fetch, pull, clone — use cases

- **`git clone <url>`** — download a full copy of a remote repository (including all history) to your local machine for the first time; used once when starting work on a project
- **`git fetch`** — check the remote repository for new changes and download them, but **don't** merge them into your current branch yet — lets you review what changed before deciding to merge
- **`git pull`** — does `git fetch` **and** `git merge` in one step, immediately bringing remote changes into your current branch

**Real-time example — banking**
- First day joining the project: `git clone` the banking app's repository to get the full codebase locally
- Before starting work each morning: `git fetch` to see what teammates pushed overnight without touching your current branch, review the changes, then decide to `git pull` if you want them merged into your local `develop` branch right away

## Q7. Explain Git branching models

*(Overlaps with Q2 — here's the focused version specifically on branching models)*

- **Git Flow** — main, develop, feature, release, hotfix branches (detailed structure, best for scheduled releases)
- **GitHub Flow** — main + short-lived feature branches only, merge via PR directly to main, deploy immediately (best for continuous deployment)
- **Trunk-Based Development** — everyone works off one main branch with very short-lived branches or direct small commits, relies on feature flags (best for very high deploy frequency)
- **Release branching** — a variation where a dedicated `release/x.x` branch is cut before each release for final stabilization/bug fixes, without touching ongoing feature development on `develop`

**Real-time example — banking vs e-commerce**
- A banking system with monthly regulated releases uses Git Flow with dedicated release branches for thorough compliance testing before each release
- A fast-moving e-commerce startup deploying daily uses GitHub Flow — every feature branch merges straight to main via PR and deploys immediately after passing automated checks

## Q8. Explain your Git branching strategy or version control workflow

*(Same as Q1 — this is typically asked as a follow-up to describe your personal/team's actual workflow, so give the concrete lived version)*

**Real-time example to say in interview — banking**
"In my project, we follow a Git Flow-based strategy. All feature work happens in `feature/*` branches created off `develop`. Once a feature is code-reviewed and passes CI checks (automated smoke tests), it's merged into `develop`. Before each release, we cut a `release/*` branch for final QA regression testing — this is where I run the full automated regression suite and do exploratory testing. Once signed off, it merges to `main` and deploys to production. If a critical bug is found in production, we create a `hotfix/*` branch directly from `main`, fix and test it quickly, then merge back to both `main` and `develop` so the fix isn't lost in the next regular release."

## Q9. What is the difference between git pull and git fetch? (clarifying — likely meant fetch, not "patch")

*(Note: this is most likely meant to ask about `git fetch` vs `git pull`, since "git patch" isn't a standard core Git command — clarifying and answering both interpretations)*

**git pull vs git fetch**
- `git fetch` — downloads new commits from the remote but leaves your current branch untouched; you review changes before merging
- `git pull` — downloads new commits AND immediately merges them into your current branch in one step

**Real-time example — e-commerce**
- Before merging your feature branch, you run `git fetch` to see if `develop` has moved forward, review the incoming changes with `git log develop..origin/develop`, and only then decide to merge/rebase — safer than blindly running `git pull` and getting surprise conflicts immediately

**If genuinely asking about "git patch" — brief note**
- Git does support creating and applying patch files (`git format-patch`, `git apply`) — a way to export commits as `.patch` files and apply them to another repository without direct network access; rarely used day-to-day, mostly relevant in offline/air-gapped environments or open-source contribution workflows

## Q10. What is git rebase and git merge?

*(Covered in detail in Q3 above — quick summary)*
- `git merge` — combines two branches, preserving full history with a merge commit
- `git rebase` — replays your branch's commits on top of another branch, creating a clean, linear history without a merge commit

## Q11. When to use git rebase?

- Use rebase when you want a **clean, linear commit history** — especially useful before merging a feature branch, to make it look like your work started from the latest code, making the project history easier to read later
- Use rebase to update your **local feature branch** with the latest changes from `develop`/`main` before raising a pull request, so your PR shows a clean diff without unrelated merge commits cluttering it

**When NOT to use rebase — important caveat to mention**
- Never rebase a branch that others have already pulled/are working off of — rebase rewrites commit history, so anyone else who already has the old commits will run into confusing conflicts when they try to sync
- Safe rule of thumb: rebase your own local/feature branches freely before sharing; avoid rebasing shared/public branches like `main` or `develop`

**Real-time example — banking**
- A developer working on `feature/card-block` for 3 days rebases their branch onto the latest `develop` each morning before continuing work, so when they finally raise a PR, it applies cleanly on top of the current codebase with a tidy, easy-to-review commit history — instead of a messy set of merge commits showing every sync point.

## Q12. How do you save local changes without committing or pushing to remote?

**Answer: `git stash`**

**Plain English**
- `git stash` temporarily saves your uncommitted changes (both staged and unstaged) into a separate storage area and reverts your working directory back to clean — useful when you need to quickly switch branches or pull latest changes without committing half-finished work
- `git stash pop` brings those saved changes back later, restoring exactly where you left off

**Real-time example — e-commerce**
- You're midway through modifying the checkout page's discount logic when an urgent production bug comes in that needs immediate attention on a different branch. Instead of committing incomplete/broken code just to switch branches, you run `git stash` to safely park your changes, switch to `hotfix/checkout-crash`, fix and deploy the urgent issue, switch back to your original branch, then run `git stash pop` to resume exactly where you left off on the discount logic.
