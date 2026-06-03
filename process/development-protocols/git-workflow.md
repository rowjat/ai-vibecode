# Git Workflow

How we use git across all projects. This applies to everyone:
developers, designers, AI coding tools (Claude Code, Gemini, OpenCode, Cursor).

**Version:** 2.0 — June 2026

---

## The One Rule

**Nobody commits to `main` directly.** Not humans. Not AI tools. `main` is
production. Everything goes through a pull request.

---

## Branch Types

| Prefix | Who uses it | Example |
|---|---|---|
| `feature/` | Developers | `feature/proggya-chart-filters` |
| `fix/` | Any dev, fixing a bug | `fix/data-dialogue-timeout` |
| `ai/` | AI coding sessions (any tool) | `ai/proggya-f01-f03-2026-05-11` |
| `design/` | External designer | `design/token-update-may` |
| `hotfix/` | Tech Lead only, emergency prod fix | `hotfix/nfdms-auth-bypass` |
| `chore/` | Any dev, routine maintenance | `chore/upgrade-react-19` |
| `release/` | Tech Lead, release prep | `release/v2.5.0` |
| `docs/` | Any dev, documentation only | `docs/api-endpoint-ref` |
| `spike/` | Any dev, experimental/R&D | `spike/webrtc-poc` |

### Branch Type Guidelines

| Type | Lifetime | Merges to | Requires review |
|---|---|---|---|
| `feature/` | Days–weeks | `main` | Yes |
| `fix/` | Hours–days | `main` | Yes |
| `ai/` | Session-length | `main` or dev branch | Tech Lead |
| `design/` | Days–weeks | `main` | Tech Lead + CEO |
| `hotfix/` | Hours | `main` | Tech Lead self-review + CEO notification |
| `chore/` | Hours–days | `main` | Yes |
| `release/` | Days | `main` | Tech Lead |
| `docs/` | Hours | `main` | Peer spot-check |
| `spike/` | Hours–days | Discard or `main` | Tech Lead (if merging) |

---

## Starting Work

Before writing any code:

```bash
# 1. Get latest main
git checkout main && git pull origin main

# 2. Create your branch
git checkout -b [prefix]/[descriptive-name]

# 3. Confirm you are on your branch (NOT main)
git branch --show-current
```

Branch from `main` unless told otherwise. Do not branch from a stale
local copy — always pull first.

---

## Rules

**Rule 1 — One branch per task.** Don't accumulate multiple unrelated changes
on one branch. If you're fixing a bug and notice another bug, make a second
branch for the second bug.

**Rule 2 — PR before going offline.** If you're done for the day and your
work isn't merged, push your branch and open a draft PR. This tells the team
where you are and prevents lost work.

**Rule 3 — AI branches need human review.** Every `ai/` branch requires
the Tech Lead's approval before merge. No exceptions. AI-generated code is
fast but not always contextually aware of what other modules expect.

**Rule 4 — Delete branches after merge.** Merged branches are dead. Delete
them on GitHub and locally: `git branch -d [branch]` + click "delete branch"
on the merged PR page.

**Rule 5 — Never force push to a shared branch.** If someone else might be
working on or reviewing a branch, do not `git push --force`. Rebase locally
or create a new branch.

**Rule 6 — Sign your commits.** All commits must be signed with a GPG or SSH
key. Configure once:

```bash
# GPG
git config --global user.signingkey <key-id>
git config --global commit.gpgsign true

# SSH (GitHub recommends this on macOS)
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Unsigned commits will be flagged in CI and may block the PR merge.

---

## Merge Strategy

| Scenario | Strategy | Command |
|---|---|---|
| `feature/` → `main` | Squash merge | Keep one clean commit per feature |
| `fix/` → `main` | Squash merge | Single atomic fix |
| `ai/` → `main` | Squash merge | AI sessions generate many noisy commits; squash to one |
| `hotfix/` → `main` | Rebase merge | Preserve the hotfix chain, then revert if needed |
| `release/` → `main` | Merge commit | Preserves the release commit for tagging |
| `chore/` → `main` | Squash merge | Single maintenance commit |
| `docs/` → `main` | Squash merge | Single documentation commit |
| `spike/` → discard | Delete branch | No merge needed |
| `design/` → `main` | Squash merge | One commit per design token set |

### Why Squash for AI Branches

AI coding sessions produce frequent checkpoints, experiment commits, and
backtracks. Squashing before merge keeps `main` history readable:

```bash
# Option A — squash via GitHub PR (recommended)
# Select "Squash and merge" in the PR dropdown

# Option B — squash locally before pushing
git rebase -i main
# Mark all but the first commit as `squash`
git push --force-with-lease origin ai/branch-name
```

**Never squash a shared branch** — only squash when you are the sole author.

---

## Version Tagging

Tags follow semantic versioning: `vMAJOR.MINOR.PATCH`

```bash
# Create and push a tag
git tag -a v2.5.0 -m "v2.5.0 — FMCG drill-down charts"
git push origin v2.5.0

# List tags
git tag -l "v2.*"

# Delete a tag (local + remote)
git tag -d v2.5.0
git push origin :refs/tags/v2.5.0
```

### When to Tag

| Event | Tag | Who |
|---|---|---|
| Production release | `vMAJOR.MINOR.PATCH` | Tech Lead |
| Hotfix release | `vMAJOR.MINOR.PATCH-hotfix.N` | Tech Lead |
| Pre-release / RC | `vMAJOR.MINOR.PATCH-rc.N` | Tech Lead |
| First deploy after feature complete | `vMAJOR.MINOR.PATCH` | Any committer |

### Auto-Generated Changelog

Tags feed changelog generation. After tagging:

```bash
# Generate changelog between two tags
git log --oneline v2.4.0..v2.5.0 --no-merges
```

For full automation, see the `gh` release workflow in CI.

---

## Changelog / Release Notes

Every tagged release should produce a changelog entry. Use this format:

```markdown
## [v2.5.0] — 2026-06-02

### Added
- [dashboard] FMCG category drill-down chart

### Fixed
- [auth] Token expiry not resetting on refresh

### Changed
- [design] Spacing tokens migrated to 8pt grid

### Removed
- [legacy] Deprecated v1 API endpoint

### Dependencies
- react 18.3 → 19.0
```

### Automation

Enable changelog generation from conventional commits:

```bash
# Generate draft changelog from commit log
git log --oneline --no-merges v2.4.0..HEAD | sort
```

For projects using `[module]` prefix tags, the release manager groups
changes manually. If you adopt Conventional Commits (see below), tools
like `git-cliff` or `standard-version` can automate the entire process.

### Conventional Commits (Optional Upgrade)

The existing `[module]` prefix is mandatory and sufficient. Teams that want
automated changelog, semver bumping, and CI gating can upgrade to the full
[Conventional Commits](https://www.conventionalcommits.org/) standard:

| Current | Conventional equivalent |
|---|---|
| `[auth] fix token expiry` | `fix(auth): token expiry not resetting on refresh` |
| `[dashboard] add drill-down` | `feat(dashboard): add drill-down to FMCG category chart` |
| `[design] update spacing` | `refactor(design): update spacing tokens to 8pt grid` |
| `[ai-pipeline] F03 output` | `feat(ai-pipeline): F03 output — entity extraction agent complete` |

**Decision:** Each project can adopt Conventional Commits independently.
The `[module]` prefix remains the minimum standard across all projects.
If you adopt Conventional Commits, add `conventionalcommits: true` to the
project's `.github/settings.yml`.

---

## Pull Request Process

When your work is ready for review:

```bash
# Push your branch
git push origin [your-branch-name]
```

Then open a PR on GitHub. Every PR must include:

**Title format:**
```
[module] Brief description
```

**Description (required, not optional):**
```markdown
## What This Does
One paragraph explaining the change.

## Files Changed
- list the main files

## How to Test
Steps someone can follow to verify this works.

## Checklist
- [ ] Branch is up to date with main (no conflicts)
- [ ] Code runs locally
- [ ] No debug logs or commented-out code left in
- [ ] No hardcoded credentials, API keys, or secrets
- [ ] Commits are signed
- [ ] If touching a shared interface — tagged the relevant developer
```

### Using `gh` CLI (Recommended)

```bash
# Create PR from current branch
gh pr create \
  --title "[module] Brief description" \
  --body "$(cat PR_TEMPLATE.md)" \
  --base main

# View PR status
gh pr status

# Merge after approval (squash)
gh pr merge --squash --delete-branch
```

---

## Who Reviews What

| Change type | Primary reviewer | Secondary |
|---|---|---|
| AI-generated code (`ai/` branch) | Tech Lead | — |
| Backend module | Tech Lead | Peer dev spot-check |
| Frontend / UI | Tech Lead | Peer dev spot-check |
| Design tokens / system | Tech Lead | CEO (for brand-level changes) |
| Hotfix | Tech Lead self-reviews, notifies CEO | — |
| Chore / maintenance | Peer dev | Tech Lead (if touching deps) |
| Documentation | Peer dev | — |
| Spike (if merged) | Tech Lead | — |

---

## Review Response Time

| PR type | Expected review time |
|---|---|
| Bug fix / small feature | Same day |
| AI-generated (`ai/` branch) | Within 24 hours |
| Module addition | Within 48 hours |
| Design tokens / UI changes | Within 24 hours |
| Chore / maintenance | Within 48 hours |
| Hotfix | ASAP |

---

## Commit Message Format

```
[module] short description (max 72 chars)

Optional body if the change needs explanation.
Separate from subject with blank line.
```

Examples:

```
[auth] fix token expiry not resetting on refresh
[dashboard] add drill-down to FMCG category chart
[ai-pipeline] F03 output — entity extraction agent complete
[design] update spacing tokens to 8pt grid
[chore] upgrade react 18.3 → 19.0
[docs] add API endpoint reference for v2
```

The `[module]` tag is mandatory. It makes `git log` scannable and maps to
Trello cards cleanly.

---

## AI Coding Sessions — Additional Rules

These rules apply when using Claude Code, Gemini, OpenCode, Cursor, or any
AI coding tool that writes code.

### Branch Naming

```
ai/[project-slug]-[phase]-[YYYY-MM-DD]
```

The `ai/` prefix makes AI-generated branches immediately distinguishable
from human branches in the GitHub UI.

### Before Starting an AI Session

```bash
# 1. Pull latest main
git checkout main && git pull origin main

# 2. Check for active AI sessions on overlapping scope
git branch -r | grep "ai/"
# If a branch exists for the same project/module, STOP.
# Talk to the Tech Lead before proceeding — two AI sessions on the
# same files will produce conflicting outputs.

# 3. Create session branch
git checkout -b ai/[project-slug]-[phase]-[YYYY-MM-DD]
```

### Parallel AI Sessions with Git Worktrees

When running multiple AI agents in parallel (e.g., via `vc-team`), use git
worktrees instead of switching branches to avoid context thrashing:

```bash
# Each worktree is an independent checkout directory
git worktree add ../ai-vibecode-feature-x ai/feature-x-session-2026-06-02
git worktree add ../ai-vibecode-fix-y ai/fix-y-session-2026-06-02
```

Each worktree has its own working directory, index, and branch. Agents
work without stepping on each other. When done:

```bash
# Remove a worktree after its branch is merged
git worktree remove ../ai-vibecode-feature-x

# List active worktrees
git worktree list
```

See `process/development-protocols/parallel-fan-out.md` for when to
recommend parallel AI agents. The `vc-merge-worktree` skill handles
cleanup automatically.

### Using `vc-git-manager` Agent

This repo includes a `vc-git-manager` agent that automates commit hygiene
for AI sessions. After an AI coding session:

```bash
# 1. Show what changed
git status --short

# 2. Let vc-git-manager split changes into logical commits
#    Invoke via the agent system with:
#    "organize working changes into logical commits"
#    or use the manual commands below:

# 3. Stage by file group (example)
git add src/auth/ && git commit -m "[auth] fix token expiry handling"
git add src/dashboard/ && git commit -m "[dashboard] add drill-down chart"
git add tests/ && git commit -m "[test] add auth and dashboard tests"

# 4. Push and create PR
git push origin ai/[branch-name]
gh pr create --title "[AI] [module] Summary" --body "..."
```

The agent handles:
- Diff analysis against the plan
- Commit splitting by module
- PR description generation from the plan
- Draft PR creation

### After an AI Session

```bash
# Push session branch
git push origin ai/[branch-name]

# Open PR with context
gh pr create \
  --title "[AI] [Project] [Phase range] — [brief description]" \
  --body "## Session Summary

**Phases completed:** F0X–F0Y
**Files changed:** [list]
**Tests:** [pass/fail/count]
**Known issues:** [any]
**Review focus:** [what the reviewer should look at]

## Checklist
- [ ] Tests pass
- [ ] Commits are signed
- [ ] No hardcoded credentials
- [ ] No changes outside declared scope" \
  --base main
```

The PR description is not optional. The reviewer needs context to assess
AI-generated code efficiently.

### When Another Developer Is Active on the Same Files

If another developer has an active branch touching the same files:

1. AI session checks out from that developer's branch, not from `main`
2. Branch name: `ai/[project]-[phase]-from-[dev-branch]-[date]`
3. PR targets the developer's branch, not `main`
4. The developer reviews AI additions before their own PR merges to `main`

The human developer has final say over code that touches their module.

---

## Handling Merge Conflicts

### For Human Developers

```bash
# 1. Update your branch from main
git checkout main && git pull origin main
git checkout [my-branch] && git merge main

# 2. Resolve conflicts in your editor
#    Look for <<<<<<<, =======, >>>>>>> markers

# 3. After resolving
git add [resolved-files]
git commit -m "[module] resolve merge conflicts with main"

# 4. Push
git push origin [my-branch]
```

### For AI Sessions

AI tools should never auto-resolve merge conflicts without human review.
If an AI session encounters a conflict:

1. **Stop.** The AI must report the conflict to the developer.
2. The developer reviews the conflicting changes manually.
3. If the conflict is straightforward, the developer can resolve and push.
4. If the conflict is complex, the AI creates a new branch and the developer
   resolves before merge.

```bash
# AI workflow for conflict avoidance:
# Before starting, rebase onto main to minimize drift
git fetch origin main
git rebase origin/main

# If conflicts occur during rebase:
echo "CONFLICT: Cannot auto-resolve. Requires human intervention."
exit 1
```

---

## Git Workflow Automation

### Git Hooks

This repo provides automation hooks in `.codex/hooks/` and `.claude/hooks/`.
Key hooks:

| Hook | Purpose |
|---|---|
| `pre-commit` | Lint staged files, check for secrets, verify commit message format |
| `commit-msg` | Validate `[module]` prefix, enforce 72-char subject, check for WIP |
| `pre-push` | Run tests, check branch name matches prefix rules, verify no unsigned commits |

Enable hooks:

```bash
# These hooks are installed automatically by the harness setup.
# To verify:
ls -la .git/hooks/ | grep -v sample
```

### CI Gates

Every push triggers these automated checks:

1. **Commit signing verification** — all commits must be signed
2. **Commit message format** — `[module]` prefix required
3. **Branch name validation** — must match `[prefix]/[description]`
4. **Lint & typecheck** — code quality gates
5. **Tests** — unit + integration
6. **Merge conflict detection** — PR must be up to date with `main`

CI configuration lives in `.github/workflows/`. If a check fails, the PR
cannot merge until resolved.

### `vc-git-manager` Agent Automation

The `vc-git-manager` agent provides automated git operations during AI
coding sessions. Available commands:

| Command | What it does |
|---|---|
| `organize working changes into logical commits` | Splits unstaged changes by module |
| `create PR for current branch` | Pushes and opens a draft PR with context |
| `squash and merge current branch` | Squashes all commits and merges via CLI |
| `sync branch with main` | Rebases or merges main into the current branch |
| `clean up merged branches` | Deletes local merged branches |

To invoke: route to the `vc-git-manager` agent via the orchestrator or
call it directly when in EXECUTE mode.

---

## What Happens When Rules Are Broken

If someone pushes to `main` directly (bypassing a PR):

1. It's not the end of the world — but do not make it a habit
2. Notify the Tech Lead immediately
3. If the push broke something, the Tech Lead creates a `hotfix/` branch to fix it
4. We treat it as a learning moment, not a disciplinary issue

If an unsigned commit is pushed:

1. CI will flag it
2. The author must rebase and re-sign: `git rebase --exec 'git commit --amend --no-edit -S'`
3. Force push with lease: `git push --force-with-lease`

Currently GitHub branch protection is not enabled (requires Pro plan).
The discipline is on us, not on a tool.

### Alternatives to Branch Protection (No Pro)

| Approach | What it does |
|---|---|
| CI gate with `if: github.ref == 'refs/heads/main'` | Blocks main pushes that bypass PR in CI |
| Server-side `pre-receive` hook | Rejects direct pushes to `main` on the server |
| `git config --global push.autoSetupRemote` | Prevents accidental pushes to the wrong remote |

Recommended: implement the CI gate first, then a server-side hook if you
have server access.

---

## Quick Reference

| I want to... | Command |
|---|---|
| Start new work | `git checkout main && git pull && git checkout -b feature/name` |
| See all branches | `git branch -a` |
| Switch branches | `git checkout [branch-name]` |
| Push my branch | `git push origin [branch-name]` |
| Update from main | `git checkout main && git pull && git checkout [my-branch] && git merge main` |
| Rebase onto main | `git fetch origin main && git rebase origin/main` |
| Delete merged branch | `git branch -d [branch-name]` |
| Check if I'm on main | `git branch --show-current` |
| List worktrees | `git worktree list` |
| Add a worktree | `git worktree add ../path branch-name` |
| Remove a worktree | `git worktree remove ../path` |
| Squash last N commits | `git reset --soft HEAD~N && git commit -m "[module] message"` |
| Create and push a tag | `git tag -a v1.0.0 -m "message" && git push origin v1.0.0` |
| Sign previous commit | `git commit --amend --no-edit -S` |
| View commit log by module | `git log --oneline --grep="^\[auth\]"` |
