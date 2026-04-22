---
name: resolve-merge-conflicts
description: Resolve PR merge conflicts end-to-end by identifying the base branch, merging it locally, fixing conflicted files carefully, validating the result, and pushing the updated branch. Use when the user asks to look at merge conflicts on a PR, resolve them, and push the branch.
---

# Resolve Merge Conflicts

Use this skill when a user wants merge conflicts on an active PR resolved all the way through, not just inspected.

The goal is:
- identify the PR base branch
- merge the base into the current branch locally
- resolve the conflicted files without discarding intentional branch work
- validate the merged result
- commit the merge
- push the updated branch

## Preconditions

- the current branch should already be the PR branch you intend to update
- `gh` should be authenticated for the repo if you need to inspect PR metadata
- do not assume a conflict is already present locally; check first

## Workflow

1. Inspect the current branch and PR context.

Start with:

```bash
git status --short --branch
git diff --name-only --diff-filter=U
gh pr view --json number,title,baseRefName,headRefName,url
```

Notes:
- if `git diff --name-only --diff-filter=U` is empty, there may be no active conflicted merge in the worktree yet
- if needed, verify the branch matches `headRefName` from the PR

2. Fetch the latest base branch state.

Typical command:

```bash
git fetch origin
```

3. Merge the PR base branch into the current branch locally.

Typical command:

```bash
git merge --no-edit origin/<base-branch>
```

Examples:

```bash
git merge --no-edit origin/main
git merge --no-edit origin/develop
```

If the merge reports conflicts, capture the conflicted files:

```bash
git diff --name-only --diff-filter=U
```

4. Read each conflicted file before editing.

Look for conflict markers:

```bash
rg -n "^(<<<<<<<|=======|>>>>>>>)" <file>
```

Guidance:
- keep the current branch's feature intent intact
- absorb compatible changes from the base branch
- prefer the newer shared API or wiring from base when it does not invalidate the feature branch's design
- remove conflict markers completely
- do not blindly take one side for large files; reconcile the behavior

5. Resolve the conflicts carefully.

When reconciling:
- preserve branch-specific feature work
- preserve legitimate upstream fixes from base
- adapt old single-file assumptions if the branch has moved to package-level or directory-level behavior
- update tests when upstream added coverage that still assumes old shapes

After edits, verify there are no remaining conflict markers:

```bash
rg -n "^(<<<<<<<|=======|>>>>>>>)" src
```

6. Run the smallest useful validation for the merged surfaces.

Recommended baseline:

```bash
pnpm typecheck
```

Then add focused tests for the files or systems touched by the conflict resolution. Examples:

```bash
pnpm exec vitest run src/main/issue-resolution.test.ts
pnpm exec vitest run src/main/skill-inventory.test.ts
pnpm exec vitest run src/renderer/src/views/InventoryViewChrome.test.tsx
```

If the conflict changes visible UI, also run a real browser check with Playwright.

7. Stage the resolved files and complete the merge commit.

Typical flow:

```bash
git add <resolved-files>
git status
git commit --no-edit
```

Notes:
- during a merge, `git commit --no-edit` will usually finish the merge commit using the default merge message
- only use a custom commit message if there is a clear reason

8. Push the updated PR branch.

Typical command:

```bash
git push origin <current-branch>
```

Example:

```bash
git push origin codex/skills-are-directories-not-files
```

9. Report back clearly.

Include:
- which files had conflicts
- the core reconciliation decisions
- the exact validation you ran
- whether the branch was pushed successfully

## Decision Standards

- Do not throw away either side without understanding what behavior it carried.
- Prefer the feature branch's architectural direction when that is the purpose of the PR.
- Prefer the base branch's new shared APIs, tests, and wiring when they can be integrated cleanly.
- If upstream changed assumptions that conflict with the PR's design, reconcile the newer code to the PR's model instead of silently reverting the feature.
- Keep the result coherent; conflict resolution is integration work, not line-level arbitration.

## Common Pitfalls

- assuming conflicts already exist locally when they do not
- resolving only the compile errors and missing test expectation drift
- leaving one file still marked `UU` because it was edited but not staged
- forgetting that merge commits are still in progress until `git commit`
- pushing before re-running targeted validation

## Final Response To The User

Report back with:
- what conflicted
- what you chose in the reconciliation
- what checks passed
- confirmation that the branch was pushed
