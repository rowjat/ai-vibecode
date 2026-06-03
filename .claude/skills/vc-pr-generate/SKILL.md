---
name: github-pr-generate
description: >
  Generate a complete, high-quality GitHub Pull Request from staged changes, a branch diff, a commit log, or a plain description of what was done. Use this skill whenever the user wants to: open a PR, write a PR description, draft a pull request, generate PR title and body, summarize changes for a PR, create a PR from a diff or commit list, or produce PR-ready markdown. Trigger even if the user says things like "write up my changes", "make a PR for this", "what should I put in this PR", or "help me open a pull request". This skill produces structured, ready-to-paste GitHub PR output including title, body with sections (summary, motivation, changes, testing), and optional labels/checklist.
---

# GitHub PR Generator

Generates a complete, well-structured GitHub Pull Request (title + body) from any of the following inputs:

- `git diff` or patch output
- `git log` (commit messages)
- A plain English description of changes
- A mix of the above

---

## Workflow

### Step 1 — Gather Input

Check what the user has provided. Accept any combination of:

| Input type | How to use it |
|---|---|
| `git diff HEAD~N..HEAD` or patch | Primary source of truth for what changed |
| `git log --oneline HEAD~N..HEAD` | Summarize intent from commit messages |
| File paths / changed modules | Scope the impact area |
| Plain description | Use as motivation / context |
| Ticket / issue number | Link in body |
| Target branch | Mention in body if not `main`/`master` |

If the user hasn't provided a diff or commit log but wants a PR, ask:
> "Can you share the output of `git diff main...HEAD` or `git log --oneline main...HEAD`? Or describe what you changed and I'll draft from that."

If the user says "just draft from description", proceed with what you have.

---

### Step 2 — Analyze the Changes

Before writing, mentally classify the change:

- **Type**: feat / fix / refactor / chore / docs / test / perf / ci
- **Scope**: which module, service, or file area is affected
- **Risk**: low / medium / high (breaking changes, migrations, auth, data)
- **Size**: small (1–3 files) / medium / large

Use this classification to set tone and detail level.

---

### Step 3 — Generate the PR

Produce the following sections. All output should be **GitHub Flavored Markdown**, ready to paste.

#### Title

Format: `[type]: short imperative description (≤72 chars)`

Examples:
- `feat: add OAuth2 login with Google`
- `fix: resolve race condition in job queue`
- `chore: upgrade Postgres driver to v3`

#### Body

Use this template, filling in only the relevant sections. Omit sections that don't apply.

```markdown
## Summary
<!-- 2–4 sentences. What does this PR do and why? -->

## Motivation / Context
<!-- Why is this change needed? Link issues: Closes #123 -->

## Changes
<!-- Bullet list of what was changed. Be specific. -->
- 
- 

## Screenshots / Demo
<!-- If UI changes, add before/after. Otherwise omit. -->

## Testing
<!-- How was this tested? What should a reviewer verify? -->
- [ ] Unit tests added/updated
- [ ] Manually tested: <describe scenario>
- [ ] No new tests needed: <reason>

## Migration / Breaking Changes
<!-- If this changes APIs, DB schema, env vars, etc. -->
> ⚠️ Breaking: ...

## Checklist
- [ ] Tests pass locally
- [ ] Docs updated (if applicable)
- [ ] No secrets or debug code committed
```

---

### Step 4 — Output Format

Always output:

1. **A fenced code block** with the full PR body (for easy copy-paste)
2. **Suggested labels** (1–3): e.g. `bug`, `enhancement`, `breaking-change`, `needs-review`
3. **Suggested reviewers** — only if the user mentioned team members
4. **Optional**: a one-liner for the squash-merge commit message

---

## Quality Rules

- Title uses imperative mood ("add", "fix", "remove" — not "added", "fixes", "removing")
- Summary is honest about scope — don't oversell a minor fix
- Changes section uses specific file/function names when available from the diff
- Testing section is always filled — if no tests, say why
- Flag breaking changes prominently with ⚠️
- Keep body under ~600 words unless change is large/complex

---

## Examples

### Input: git log
```
abc1234 fix null pointer in UserService.getById
def5678 add index on users.email column
ghi9012 update migration script
```

### Output title:
`fix: resolve null pointer in UserService and optimize email lookup`

### Output body:
```markdown
## Summary
Fixes a null pointer exception that occurred when fetching users by ID with a missing record, and adds a database index on `users.email` to speed up lookup queries.

## Motivation / Context
Resolves #214 — production errors logged when user lookup returns null.

## Changes
- `UserService.getById`: add null check before accessing user fields
- `migrations/add_email_index.sql`: new index on `users.email`
- `migrations/`: updated script ordering to apply index after table creation

## Testing
- [x] Unit tests updated for `UserService.getById` null case
- [x] Manually tested: confirmed no errors on missing user lookup
- [x] Index verified in local Postgres via `\d users`

## Checklist
- [x] Tests pass locally
- [x] No secrets or debug code committed
```

**Labels**: `bug`, `performance`
**Squash commit**: `fix: null pointer in UserService + email index (#214)`
