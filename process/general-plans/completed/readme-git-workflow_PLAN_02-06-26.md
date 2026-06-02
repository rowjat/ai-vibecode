# PLAN: README Git Workflow Summary

## Overview
This plan addresses the "Gap" in AI-assisted development where agents might inadvertently commit directly to the `master` or `dev` branches. We are adding a "Git Workflow Quick Start" section to the `README.md` to make these rules explicit and discoverable.

This implementation serves as a demonstration of the RIPER-5 protocol, specifically showing how the `EXECUTE` phase is strictly bound by a plan that enforces branch safety.

## Complexity: COMPLEX (Demonstration)

## Touchpoints
- `README.md`

## Public Contracts
None (Documentation only).

## Blast Radius
Low. Changes are limited to the project's root `README.md` file.

## Implementation Checklist

### Phase 1: Preparation & Branch Safety
1. **[MANDATORY]** Create a new git branch named `ai/readme-git-workflow-summary` from the current `master` branch.
   - Command: `git checkout -b ai/readme-git-workflow-summary`
2. Verify the branch was created and is active.

### Phase 2: Content Implementation
3. Locate the "🚀 Install" section in `README.md`.
4. Insert a new section titled "🛡️ Git Workflow (Team Safety)" immediately following the Install section.
5. Add the following content:
   - **The One Rule:** No direct commits to `main` or `master`.
   - **Branching:** Use `ai/` prefix for AI sessions, `feature/` for humans.
   - **Hygiene:** Squash merges are mandatory for AI branches to keep history clean.
   - **Signing:** All commits must be signed (GPG/SSH).
6. Reference `process/development-protocols/git-workflow.md` for full details.

### Phase 3: Verification
7. Run `markdownlint` (if available) on `README.md`.
8. Visually inspect the `README.md` to ensure the new section is correctly formatted and positioned.
9. Verify that no changes were accidentally made to the `master` branch.

## Verification Evidence
- [ ] Successful checkout of `ai/readme-git-workflow-summary`.
- [ ] `git branch --show-current` returns the new branch name.
- [ ] `README.md` contains the new section with the specified rules.
- [ ] `git diff master` shows only the intended changes on the new branch.

## Resume and Execution Handoff
To implement this plan, the `vc-execute-agent` MUST:
1. **Check out the branch first** before any file edits.
2. Use the content specified in Phase 2.
3. Report the `touched_files` as `["README.md"]`.

**Status:** DONE
**Summary:** Implementation plan for README update created, focusing on demonstrating branch safety and closing the "gap" in AI workflows.
**Concerns/Blockers:** None.
