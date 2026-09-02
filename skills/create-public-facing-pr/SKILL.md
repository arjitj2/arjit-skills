---
name: create-public-facing-pr
description: Use when creating, publishing, or updating a public-facing GitHub pull request that should read cleanly for maintainers and users. Guides branch hygiene, staging, validation, commit wording, PR title/body, pushing, and final verification without exposing agent workflow details.
---

# Create Public Facing PR

Use this for GitHub pull requests that should read like they were prepared for the project, not copied from an agent transcript.

## Workflow

1. Inspect scope before staging.

Use quick local checks:

```bash
git status -sb
git diff --stat
git diff
```

Stage only the files that belong in the PR.

2. Keep unrelated work out of the PR.

If the current branch already backs another PR, create a new `arjit/<description>` branch from the repository's mainline branch before editing.

3. Run the smallest relevant validation before publishing.

Prefer the repository's documented checks. Use targeted validation for the touched area first, then broaden only when the change or project norms require it. Be explicit if environment issues prevent validation.

4. Commit with a terse, public-facing message.

Avoid mentioning agents, automation, internal workflow, or implementation trivia that does not help maintainers understand the change.

5. Push with tracking.

```bash
git push -u origin "$(git branch --show-current)"
```

6. Create the PR as ready for review unless the user explicitly asks for a draft.

7. Attach relevant image or video evidence with `--attach` when it exists.

8. Request a Copilot review when available unless the user explicitly opts out.

9. Verify the created PR before reporting it back.

```bash
gh pr view --json title,isDraft,body,url,headRefName,baseRefName
```

## PR Title

- Do not prefix with `[codex]`.
- Do not mention agents, automation, or internal workflow.
- Use a concise verb phrase that describes the actual change.

Good:

```text
Show download progress
```

Bad:

```text
[codex] Implement download progress modal
```

## PR Body

Write only what changed and what validated it. Use Markdown when it improves scanability: short bullets for distinct changes, inline code for commands or paths, and compact sections. Keep formatting restrained; avoid padded summaries, nested lists, decorative headings, or anything that makes the PR read like an agent transcript.

Baseline template:

```markdown
## Changes

- <specific change>

## Validation

- `<command or check>`
```

Avoid filler such as "This PR", "improves the experience", "polishes", "streamlines", or extra summary bullets unless they add specific information.

Use a temporary body file with real newlines when creating or editing PRs. Pass Markdown through `--body-file` so bullets, headings, and backticks stay readable:

```bash
tmp_body=$(mktemp)
cat > "$tmp_body" <<'EOF'
## Changes

- <specific change>

## Validation

- `<command or check>`
EOF
gh pr create --base <base-branch> --head "$(git branch --show-current)" --title "<title>" --body-file "$tmp_body"
```

## Media Evidence

When a screenshot or video materially demonstrates a UI change, visual regression, rendered output, or interaction, include it in the PR body. Do not attach irrelevant media or fabricate evidence.

`--attach` requires GitHub CLI v2.99.0 or newer. Check `gh --version` and `gh pr create --help` before use. Reference each local path in the body with useful alt text, then pass the same path through the repeatable flag so GitHub preserves its placement:

```markdown
![Settings loading state](output/verification/settings-loading.png)
```

```bash
gh pr create ... \
  --body-file "$tmp_body" \
  --attach output/verification/settings-loading.png
```

For an existing PR, use `gh pr edit PR-NUMBER --attach <path>`. Verify `gh pr view PR-NUMBER --json body` rewrote local paths to `https://github.com/user-attachments/...`. If v2.99.0+ cannot be used, report why the evidence was not attached.

## Copilot Review

After creating or materially updating a PR, request Copilot when the repository supports it:

```bash
gh pr edit PR-NUMBER --add-reviewer @copilot
```

Do not trust exit status alone. Verify a pending request, Copilot review, or `review_requested` timeline event. If the CLI command creates no request, use GitHub's documented reviewer identity:

```bash
gh api --method POST repos/OWNER/REPO/pulls/PR-NUMBER/requested_reviewers \
  -f 'reviewers[]=copilot-pull-request-reviewer[bot]'
```

Treat a clear unavailable/not-enabled response as non-blocking and report it. Do not wait for Copilot to finish unless the user asks.

If the user gives title, body, draft, base branch, or publishing instructions, those instructions override this skill.
