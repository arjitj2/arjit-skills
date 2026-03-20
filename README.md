# arjit-skills

Personal collection of Claude Code skills.

## Skills

### adding-a-project

Workflow for adding a new project to the personal site (arjit-me) and resume (arjit-resume), compiling the PDF, and pushing live.

### creating-skills

Meta-skill for creating new skills in this repo and symlinking them into agent directories.

### frontend-slides

Create animation-rich HTML presentations from scratch or by converting PowerPoint files.

Originally created by [@zarazhangrui](https://github.com/zarazhangrui/frontend-slides).

### swiftui-design-principles

Design principles for building polished, native-feeling SwiftUI apps and widgets. Ensures proper spacing, typography, colors, and widget implementations.

## Setup

Symlink each skill into `~/.agents/skills/` and `~/.claude/skills/`:

```bash
# ~/.agents/skills/
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/swiftui-design-principles ~/.agents/skills/swiftui-design-principles
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/frontend-slides ~/.agents/skills/frontend-slides

# ~/.claude/skills/
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/swiftui-design-principles ~/.claude/skills/swiftui-design-principles
ln -sf /Users/arjitjaiswal/repos/arjit-skills/skills/frontend-slides ~/.claude/skills/frontend-slides
```
