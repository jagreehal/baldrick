---
name: entropy-loop
description: "Iteratively reduce code entropy by fixing smells and improving quality. Use when: cleanup, code smells, entropy, refactor loop, baldrick entropy. Triggers on: entropy, cleanup, code smells, reduce entropy."
---

# Entropy Reduction Loop

Iteratively scan and fix code smells to improve codebase quality.

> **Philosophy:** Impactful cleanup over nitpicking. Focus on changes that matter.

---

## The Job

1. Scan codebase for code smells
2. Identify ONE smell to fix
3. Fix with minimal changes
4. Run tests to verify
5. Commit the fix
6. Repeat until no more meaningful improvements

---

## Iteration Flow

```
┌─────────────────────────────────────────────────┐
│ 1. Scan for code smells                         │
│ 2. If no meaningful smells → COMPLETE           │
│ 3. Pick ONE smell to fix                        │
│ 4. Fix with minimal changes                     │
│ 5. Run tests to verify                          │
│ 6. Commit the fix                               │
│ 7. Append to progress.txt                       │
│ 8. Loop back to step 1                          │
└─────────────────────────────────────────────────┘
```

---

## Code Smells to Look For

**High Impact:**
- Duplicate code that should be a function
- Unused exports and dead code
- Overly complex functions (break them down)
- Inconsistent patterns (use the established pattern)

**Medium Impact:**
- Missing type annotations
- Unreachable code branches
- Deeply nested conditionals
- Long parameter lists

**Low Impact (skip unless asked):**
- Minor formatting
- Comment improvements
- Variable renaming (unless confusing)

---

## Priority Order

1. **Unused code** - Remove dead code first
2. **Duplicates** - Extract shared logic
3. **Complexity** - Break down large functions
4. **Inconsistency** - Align with patterns
5. **Types** - Add missing annotations

---

## Progress Format

Append to progress.txt after each fix:

```
## Entropy Fix - [Date/Time]
- Smell: Duplicate email validation in 3 files
- Fix: Extracted to `src/utils/validation.ts`
- Files: auth.ts, profile.ts, settings.ts
- Impact: Reduced code by 45 lines
---
```

---

## Stop Condition

When no more meaningful improvements exist:
1. Output: `<promise>COMPLETE</promise>`

**"Meaningful" means:**
- Reduces actual complexity
- Improves maintainability
- Removes real duplication
- NOT just cosmetic changes

---

## Example Usage

User: "Run entropy loop to clean up the codebase"

The skill will:
1. Scan for code smells
2. Fix one smell per iteration
3. Continue until codebase is clean

---

## Checklist Per Iteration

- [ ] Identified ONE meaningful code smell
- [ ] Fix is minimal and focused
- [ ] Tests still pass
- [ ] Committed the fix
- [ ] Documented in progress.txt
- [ ] Did NOT make unrelated changes
