# Baldrick Agent Instructions

## Overview

Baldrick is an autonomous AI agent loop that runs Claude Code repeatedly until all specs are complete. Each iteration is a fresh Claude instance with clean context.

The core loop **deterministically selects** the highest priority incomplete spec, so you don't need to choose - just implement the `## SELECTED SPEC` section in your context.

## Commands

```bash
# Run the loop
./baldrick run [n] [--dry-run]  # Run n iterations (default: 10)
./baldrick once <spec> [--max n] # Focus on single spec until done

# Check status
./baldrick status               # Show specs and progress
./baldrick validate             # Check specs for errors
./baldrick doctor               # Check configuration

# Utilities
./baldrick init                 # Setup project structure
./baldrick archive              # Archive current session
./baldrick archive list         # List archived sessions
./baldrick archive restore <n>  # Restore archived session
```

## Key Files

| File | Purpose |
|------|---------|
| `baldrick` | The bash execution loop (single file, self-contained) |
| `specs/*.md` | Feature specs with YAML frontmatter |
| `progress.txt` | Session progress log (read `## Codebase Patterns` first) |
| `baldrick-learnings.md` | Permanent patterns that survive archiving |
| `tradeoffs.md` | Decision log (why choices were made) |
| `logs/iter-*.json` | Structured iteration data |
| `logs/baldrick-*.log` | Raw Claude output |
| `templates/example.md` | Example spec format |

## Spec Format v1 (Quick Reference)

**Minimal spec** (required fields only):
```yaml
---
title: "Feature Title"
passes: false              # Set true when done
---
```

**Recommended spec**:
```yaml
---
id: feature-name-v1        # Unique identifier (for depends_on)
title: "Feature Title"
passes: false              # Set true when done
priority: high             # high | medium | low (default: medium)
risk: standard             # spike | integration | standard | polish (default: standard)
created: 2026-01-11        # YYYY-MM-DD format
depends_on: []             # Array of spec IDs
---
```

**Selection order:** priority (high→low) → risk (spike→polish) → created (old→new) → id (A→Z)

**Important:** `passes: true` means the spec's Done When criteria are met **and** quality gates (build/test/lint) are green.

## Workflow Per Iteration

1. Read `## Codebase Patterns` in progress.txt FIRST
2. Read `baldrick-learnings.md` for permanent patterns
3. Find `## SELECTED SPEC` in context (this is your target)
4. Implement that single spec following BDD scenarios
5. Run quality checks (build, test, lint)
6. Commit changes if all checks pass
7. Set `passes: true` in the spec
8. Append progress to progress.txt

## Stop Signals

- `<promise>COMPLETE</promise>` - All specs done, create `.baldrick-done` file
- `<promise>BLOCKED</promise>` - Human intervention needed
- `<promise>STUCK</promise>` - Multiple iterations with no progress

## Quality Requirements

- ALL commits must pass quality checks (build, test, lint)
- Do NOT set `passes: true` until ALL checks pass
- Paste actual command output in progress notes
- `passes: true` requires both Done When criteria met AND quality gates green

## Log Tradeoffs

When you make a significant decision, APPEND to `tradeoffs.md`:

```
## [Date] - [Decision Title]
Spec: [spec name]
Decision: [what you chose]
Alternatives: [what you considered]
Rationale: [why you chose this]
---
```

**You MUST log a tradeoff when you:**
- Change or introduce a data model / schema
- Make a security or auth decision
- Choose between architectural approaches
- Introduce a new dependency
- Change a public API contract

Create `tradeoffs.md` if it doesn't exist. This file answers "why was it done this way?" for future maintainers.

## Patterns

<!-- Add reusable patterns here as you discover them -->
- Read existing patterns in progress.txt before starting work
- Keep commits small and focused
- Follow existing code conventions in the target repo
- Work on ONE spec per iteration
- Commit frequently with conventional commit messages
