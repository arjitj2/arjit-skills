# arjit-skills

Personal collection of Claude Code skills.

## Install

Install a single skill from this repo:

```bash
npx skills add arjitj2/arjit-skills --skill swiftui-design-principles
```

Install all skills from this repo:

```bash
npx skills add arjitj2/arjit-skills --all
```

You can also list available skills before installing:

```bash
npx skills add arjitj2/arjit-skills --list
```

## Skills

### adding-a-project

Workflow for adding a new project to the personal site (arjit-me) and resume (arjit-resume), compiling the PDF, and pushing live.

### address-copilot-review

Handle GitHub Copilot PR review comments end-to-end, including code fixes, verification, replies, and thread resolution.

### create-public-facing-pr

Prepare clean public-facing GitHub pull requests with scoped staging, validation, concise commit messaging, polished PR title/body, push, and final verification.

### creating-skills

Meta-skill for creating new skills in this repo and symlinking them into agent directories.

### deslop

Coordinate an eight-lane cleanup of duplicated code, shared types, unused code, circular dependencies, weak types, defensive fallbacks, legacy paths, and AI-style clutter.

### frontend-design

Create distinctive, production-grade frontend interfaces with high design quality.

### frontend-slides

Create animation-rich HTML presentations from scratch or by converting PowerPoint files.

Originally created by [@zarazhangrui](https://github.com/zarazhangrui/frontend-slides).

### ios-simulator-testing

End-to-end iOS simulator testing using blitz-iphone MCP and XcodeBuildMCP. Covers which tool to use and when, gesture mechanics, screenshot discipline, and interaction patterns.

### resolve-merge-conflicts

Resolve PR merge conflicts end-to-end by merging the base branch locally, reconciling conflicted files, validating the result, and pushing the updated branch.

### swiftui-design-principles

Design principles for building polished, native-feeling SwiftUI apps and widgets. Ensures proper spacing, typography, colors, and widget implementations.

## Development Setup

For local development, symlink each skill into `~/.agents/skills/` and `~/.claude/skills/`:

```bash
# ~/.agents/skills/
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/adding-a-project ~/.agents/skills/adding-a-project
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/address-copilot-review ~/.agents/skills/address-copilot-review
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/create-public-facing-pr ~/.agents/skills/create-public-facing-pr
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/creating-skills ~/.agents/skills/creating-skills
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/deslop ~/.agents/skills/deslop
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/frontend-design ~/.agents/skills/frontend-design
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/frontend-slides ~/.agents/skills/frontend-slides
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/ios-simulator-testing ~/.agents/skills/ios-simulator-testing
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/resolve-merge-conflicts ~/.agents/skills/resolve-merge-conflicts
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/swiftui-design-principles ~/.agents/skills/swiftui-design-principles

# ~/.claude/skills/
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/adding-a-project ~/.claude/skills/adding-a-project
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/address-copilot-review ~/.claude/skills/address-copilot-review
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/create-public-facing-pr ~/.claude/skills/create-public-facing-pr
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/creating-skills ~/.claude/skills/creating-skills
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/deslop ~/.claude/skills/deslop
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/frontend-design ~/.claude/skills/frontend-design
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/frontend-slides ~/.claude/skills/frontend-slides
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/ios-simulator-testing ~/.claude/skills/ios-simulator-testing
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/resolve-merge-conflicts ~/.claude/skills/resolve-merge-conflicts
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/swiftui-design-principles ~/.claude/skills/swiftui-design-principles
```
