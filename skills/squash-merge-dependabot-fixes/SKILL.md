---
name: squash-merge-dependabot-fixes
description: Squash-merge already-assessed Dependabot PRs one by one, handling merge conflicts carefully, waiting for fresh CI after Dependabot rebases, checking security alerts, and recording the outcome.
---

# Squash & Merge Dependabot Fixes

Use this skill when the user asks to squash-merge Dependabot PRs after they have been assessed or validated, especially when several dependency PRs need to land one-by-one.

The goal is:
- confirm which Dependabot PRs are still open
- squash-merge them one at a time
- avoid unnecessary rebases or local `main` checkouts
- resolve conflicts only when GitHub reports them
- wait for fresh CI if Dependabot recreates or rebases a branch
- confirm no open Dependabot security alerts remain
- record exactly what merged and what remains

## Preconditions

- `gh` must be authenticated for the repository.
- The user has already approved merging, or the PRs have already been assessed as safe to merge.
- If an automation memory file is named in the thread, read it before merging and update it before responding.
- If a PR has not been assessed yet, do not blindly merge it; first inspect the diff, CI, and package impact.

## Workflow

1. Read prior context.

If this is part of an automation, read its memory file first. For example:

```bash
CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
sed -n '1,220p' "$CODEX_HOME/automations/<automation-id>/memory.md"
```

Then list open Dependabot PRs:

```bash
gh pr list --state open --author app/dependabot \
  --json number,title,headRefName,baseRefName,mergeable,isDraft,statusCheckRollup,url \
  --limit 100
```

2. Choose merge order.

Prefer the already-assessed order if one exists in memory or the user named one. Otherwise:
- merge smaller dev/tooling PRs before runtime PRs when they are independent
- merge older PR numbers before newer ones when there is no better reason
- merge one PR completely before starting the next

Do not pull latest `main` between every PR just to be tidy. Let GitHub tell you whether a PR is mergeable.

3. Squash-merge through GitHub.

Prefer the REST API form because it does not try to checkout or update a local `main` branch:

```bash
gh api -X PUT repos/:owner/:repo/pulls/<pr-number>/merge \
  -f merge_method=squash \
  -f commit_title='<public-facing squash title>'
```

Use the PR title as the squash title unless there is a clear reason to improve it.

After each merge, verify:

```bash
gh pr view <pr-number> --json number,state,mergedAt,mergeCommit,url
```

4. If GitHub reports conflicts, resolve the PR branch.

If the merge API returns a conflict response such as:

```text
Pull Request has merge conflicts
```

then use the `resolve-merge-conflicts` skill. The shortest safe path is:

```bash
git fetch origin <base-branch> <head-branch>
git worktree add /tmp/<repo>-pr<pr-number>-conflict -b <head-branch> origin/<head-branch>
cd /tmp/<repo>-pr<pr-number>-conflict
git merge --no-edit origin/<base-branch>
git diff --name-only --diff-filter=U
```

For dependency lockfile conflicts, prefer regenerating the lockfile with the repo's package manager instead of hand-splicing large YAML/JSON lockfiles. Preserve both sides' intended dependency bumps. Example for pnpm:

```bash
git checkout --theirs pnpm-lock.yaml
pnpm install --lockfile-only
rg -n "^(<<<<<<<|=======|>>>>>>>)" package.json pnpm-lock.yaml
pnpm install --frozen-lockfile
pnpm lint
pnpm typecheck
pnpm build
```

Run the smallest validation that proves the resolved dependency graph works in the target repo. Prefer the repo's own documented checks, package scripts, CI workflow commands, or project-specific testing skill when one exists. For lockfile-only dependency merges, this usually means a frozen install plus lint/typecheck/build and focused or serial tests when local parallelism is known to be noisy.

Example from Skill Index:

```bash
pnpm exec vitest run --no-file-parallelism --maxWorkers=1
```

Then complete the merge commit and push:

```bash
git add <resolved-files>
git commit --no-edit
git push origin <head-branch>
```

5. Handle Dependabot branch updates calmly.

Dependabot may force-update or recreate the branch while you are resolving conflicts. If your push is rejected:

```bash
git fetch origin <head-branch>
git log --oneline --graph --left-right HEAD...origin/<head-branch> --max-count=40
gh pr view <pr-number> --json number,title,mergeable,statusCheckRollup,commits
```

If Dependabot's fresh branch is now mergeable and contains the same intended dependency bump against current `main`, do not force-push your obsolete conflict-resolution commit. Wait for fresh CI and merge the remote PR branch instead:

```bash
gh pr checks <pr-number> --watch --interval 10
gh api -X PUT repos/:owner/:repo/pulls/<pr-number>/merge \
  -f merge_method=squash \
  -f commit_title='<public-facing squash title>'
```

6. Check security alerts after merging.

Always check open Dependabot security alerts before reporting completion:

```bash
gh api 'repos/:owner/:repo/dependabot/alerts?state=open' --paginate \
  --jq '.[] | {number: .number, dependency: .dependency.package.name, severity: .security_advisory.severity, summary: .security_advisory.summary, html_url: .html_url}'
```

If alerts remain and are not covered by the merged PRs, create or ask for the appropriate follow-up work instead of claiming the Dependabot sweep is done.

7. Confirm remaining PR state.

```bash
gh pr list --state open --author app/dependabot --json number,title,url,mergeable --limit 100
gh pr list --state open --json number,title,author,url --limit 100
```

Call out remaining non-Dependabot PRs separately, especially duplicates superseded by a Dependabot PR.

8. Update memory if applicable.

Before responding, append a concise note to the automation memory with:
- PR numbers and merge SHAs
- any conflicts and conflicted files
- validation run for conflict resolutions
- whether security alerts remain
- remaining open PRs that need human attention

## Decision Standards

- Do not merge a PR with failing required checks unless the user explicitly instructs you to bypass them and the repo permits it.
- Do not rebase or pull every PR by habit. Merge one PR, then let GitHub recompute mergeability for the next.
- Do not force-push over a Dependabot branch unless you are certain your local branch contains the latest intended dependency bump and the force update is necessary.
- For lockfile conflicts, keep the current base dependency graph plus the PR's intended bump, then regenerate with the package manager.
- Treat local tool quirks separately from GitHub merge state. For example, `gh pr merge` may fail locally because `main` is checked out in another worktree even though the PR already merged or can merge through the API.

## Common Pitfalls

- assuming a failed local `gh pr merge` means the PR did not merge
- deleting or force-pushing over a Dependabot branch that was just recreated
- hand-editing lockfiles line by line when the package manager can produce a coherent graph
- forgetting to wait for fresh CI after a Dependabot rebase
- reporting only "merged" without merge SHAs or remaining security-alert state

## Final Response To The User

Report back with:
- each PR merged and its squash merge SHA
- any conflicts and how they were resolved
- validation that ran for any conflict resolution
- whether open Dependabot PRs or security alerts remain
- any remaining non-Dependabot PRs the user should know about
