---
name: deslop
description: Use when the user asks to deslop, clean up, de-slop, harden, simplify, deduplicate, or improve code quality across an existing codebase, especially when the work spans unused code, circular dependencies, weak types, legacy paths, defensive fallbacks, and AI-generated clutter.
---

# Deslop

Use this skill for broad code-quality cleanup where the goal is to make the codebase smaller, clearer, more strongly typed, and less defensive or legacy-laden.

## Operating Model

Treat this as coordinated research plus implementation, not a blind cleanup sprint.

1. Inspect the repo first: language, package manager, test commands, lint/typecheck commands, dependency graph tooling, and current git status.
2. If a subagent or multi-agent tool is not already visible, search for one before falling back. If subagents are available, dispatch exactly 8 independent subagents, one for each lane below. If subagents are unavailable, run the same lanes sequentially and say so.
3. Give each lane a narrow, evidence-based prompt. Require detailed research, a critical assessment of current code, recommendations with confidence levels, and implementation of only high-confidence recommendations.
4. Integrate the results yourself. Review every diff, reconcile conflicts, and reject speculative or risky changes.
5. Verify with the strongest practical checks: typecheck, lint, targeted tests, dependency analysis, and any affected app build.

## The 8 Lanes

1. **Deduplication and DRY**
   Find duplicated logic, repeated utilities, copied UI, repeated config, and near-identical workflows. Consolidate only when it reduces complexity or clarifies ownership.

2. **Shared Type Definitions**
   Find local type/interface/model/schema definitions that should be shared. Consolidate types into existing shared modules or a minimal new shared home that matches repo conventions.

3. **Unused Code**
   Use tools such as `knip`, language-native analyzers, compiler diagnostics, static search, and test/build evidence to find unused exports, files, dependencies, scripts, and dead routes. Remove only after proving no real references remain.

4. **Circular Dependencies**
   Use tools such as `madge`, compiler/module diagnostics, dependency graph commands, and import tracing to find cycles. Break cycles by moving shared contracts, inverting dependencies, or splitting mixed-responsibility modules.

5. **Weak Types**
   Find `any`, `unknown`, casts, unchecked dictionaries, untyped errors, nullable escape hatches, and language equivalents. Research call sites, package types, runtime schemas, and domain models before replacing with strong types.

6. **Defensive Error Handling**
   Find `try/catch` blocks and equivalent fallback/error-hiding patterns. Remove defensive handling that has no clear boundary role. Keep and clarify handling for unknown input, external systems, parsing, IO, network, user data, and intentional recovery paths.

7. **Deprecated, Legacy, and Fallback Paths**
   Find deprecated APIs, legacy compatibility branches, duplicate old/new flows, migration leftovers, feature-flag fallbacks, and stale adapter paths. Remove when the current code path is clear and tests/builds confirm the old path is no longer needed.

8. **AI Slop, Stubs, LARP, and Comments**
   Find stubs, placeholder logic, fake implementations, TODO theater, unnecessary comments, comments about in-progress replacement work, and verbose AI-style explanation. Remove them or replace with concise comments that help a new maintainer understand non-obvious code.

## Subagent Prompt Requirements

Each lane prompt must include:

- the exact lane scope and files/tools likely to matter
- instructions to research before editing
- instructions to write a critical assessment before implementation
- permission to implement only high-confidence recommendations
- a warning not to perform broad rewrites, speculative removals, or style churn
- required output: findings, evidence, recommendations not taken, files changed, and verification run

## Decision Standards

- Prefer deletion over abstraction when code is genuinely unused or obsolete.
- Prefer existing repo patterns over new architecture.
- Do not consolidate code just because it looks similar; shared abstractions must reduce cognitive load.
- Do not replace weak types with fake precision. Strong types must be backed by actual data shape evidence.
- Do not remove error handling at trust boundaries.
- Do not hide uncertainty. Leave lower-confidence recommendations as assessment notes rather than code changes.
- Preserve user work in a dirty git tree. Never revert unrelated changes.

## Verification

Before finalizing, run the repo's relevant checks. Common examples:

```bash
pnpm typecheck
pnpm lint
pnpm test
pnpm exec knip
pnpm exec madge --circular .
```

Use the repo's actual package manager and commands. If a tool is missing, install only with user approval or use the nearest existing analyzer.

## Final Response

Report:

- the eight lane outcomes
- the highest-impact changes implemented
- notable recommendations intentionally left unimplemented
- verification commands and results
- any remaining risk or follow-up that needs human judgment
