---
name: baldrick-run
description: "Execute Baldrick iterations on existing specs. Use when: run baldrick, execute specs, implement features, start working. Triggers on: baldrick run, run specs, implement, execute features."
---

# Baldrick Run Skill

Execute autonomous iterations on feature specifications until all are complete.

---

## The Job

Each iteration:

1. Read `## Codebase Patterns` at top of progress.txt FIRST
2. Read `baldrick-learnings.md` for additional patterns
3. Read all specs in `specs/*.md` where `passes: false`
4. Pick the HIGHEST PRIORITY incomplete spec
5. Implement that single spec
6. Run build, test, lint commands
7. Update spec to `passes: true`
8. Append progress with learnings
9. Add reusable patterns to progress.txt and baldrick-learnings.md
10. Commit changes

---

## Priority Order

Specs are selected in this order:

1. **Priority**: `high` > `medium` > `low`
2. **Within same priority, Risk**: `spike` > `integration` > `standard` > `polish`

This ensures:
- Critical work happens first
- Unknowns (spikes) are resolved early
- Polish happens last

---

## Progress Format

After completing each spec, append to progress.txt:

```markdown
## 2025-01-08 14:30 - feature-name
Session: baldrick-20250108143000-12345
- What was done: Implemented X, added Y, fixed Z
- Test output: (paste actual output)
- Learnings: (patterns discovered, gotchas encountered)
- Next: Related work or follow-ups
```

---

## Pattern Consolidation

If you discover a REUSABLE pattern during implementation:

1. Add to `## Codebase Patterns` at TOP of progress.txt
2. Add to `baldrick-learnings.md` for permanent storage

**Good patterns:**
- "Migrations: always use IF NOT EXISTS"
- "API responses: wrap in { data, error } shape"
- "Tests: mock database with jest.mock"

**Do NOT add:**
- Story-specific implementation details
- Temporary debugging notes

---

## Completion Signals

**Per-spec completion:**
- Update YAML frontmatter: `passes: true`
- Commit changes

**All specs complete:**
1. Run final verification
2. Create done marker: `touch .baldrick-done`
3. Output: `<promise>COMPLETE</promise>`

---

## Quality Requirements

Every iteration must:
- Pass typecheck
- Pass tests
- Pass lint
- Keep commits focused and minimal
- Follow existing code patterns

Do NOT commit broken code.

---

## Commands

```bash
# Run 10 iterations (default)
./baldrick run

# Run specific number of iterations
./baldrick run 20

# Preview without executing
./baldrick run 5 --dry-run

# Work on single spec until done
./baldrick once login-feature
./baldrick once login-feature --max 15
```

---

## Reviewer Mode

Every 5th iteration, switch from Worker to Reviewer:

1. Check progress.txt - are we on track?
2. Verify completed specs actually work
3. Run tests to confirm
4. Fix issues before continuing
5. If stuck 2+ iterations, try different approach

This prevents broken code from compounding.

---

## Alternative Loops (Skills)

Beyond feature specs, use these skills for goal-oriented loops:

- **coverage-loop** - Increase test coverage to target %
- **lint-loop** - Fix all linting errors
- **entropy-loop** - Reduce code smells and duplication

Load the skill and describe your goal. The skill iterates until complete.

---

## Session Tracking

Each run is tracked with:
- **Session ID**: `baldrick-YYYYMMDDHHMMSS-PID`
- **Log file**: `logs/baldrick-YYYYMMDD-HHMMSS.log`
- **Last session**: `.last-session` file

When switching branches or features, previous sessions are automatically archived.

---

## Stuck Detection

If the same spec fails twice without changes:
1. Warning is added to prompt
2. Claude advised to try different approach
3. Consider breaking spec into smaller pieces

---

## Archive on Session Change

When you switch git branches or start a new feature:
1. Previous specs, progress, and logs are archived
2. Archive saved to `archive/YYYY-MM-DD-feature-name/`
3. Fresh progress.txt created for new session

This prevents mixing work across features.
