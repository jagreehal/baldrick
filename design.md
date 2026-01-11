# Baldrick Design & Philosophy

> *"Skills manufacture work items. Baldrick consumes them until they're done."*

---

## The Core Idea

Baldrick is a **Ralph Wiggum loop**: it runs an AI agent CLI repeatedly until all tasks are complete.

But where Ralph is minimal (~80 lines), Baldrick adds structure, safety, and observability for longer autonomous runs.

The key insight: **separate intelligence from execution**.

- **Skills** (optional) = Turn your intent into specs. Domain-smart, customizable.
- **Baldrick** (core) = Execute specs mechanically. Stable, predictable, observable.

This separation means the execution loop never changes, while intelligence can evolve independently through skills.

---

## Why Not Just Use Ralph?

Ralph is brilliant for its simplicity. It's the minimal viable loop:

```bash
for i in $(seq 1 $MAX_ITERATIONS); do
  OUTPUT=$(cat prompt.md | amp --dangerously-allow-all)
  if echo "$OUTPUT" | grep -q "<promise>COMPLETE</promise>"; then
    exit 0
  fi
done
```

But for longer runs (10+ iterations) on real projects, you start wanting:

| Need | Ralph | Baldrick |
|------|-------|----------|
| **Task selection** | AI decides | Deterministic (bash selects by priority/risk) |
| **Completion check** | Single signal | Redundant (3 signals must agree) |
| **When stuck** | Keep trying | Stuck detection, reviewer mode |
| **What happened?** | Read git log | progress.txt + tradeoffs.md + logs/ |
| **Quality gates** | Manual in prompt | Configurable (BUILD_CMD, TEST_CMD, LINT_CMD) |
| **Focus on one task** | Not built-in | `baldrick once <spec-id>` |

`baldrick once <spec-id>` runs iterations targeting a specific spec until it passes (or `--max` reached).
It uses the same feedback cycles (build/test/lint), reviewer cadence, logging, and tradeoffs/iter artifacts as `baldrick run`.

Baldrick refuses to run if specs don’t validate (`baldrick validate`).

**Baldrick is Ralph with batteries included** - same architecture, more robustness for long-running autonomous work.

---

## The Two-Phase Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                         User Intent                           │
│           "I want to add user authentication"                 │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│           Skills: "Work Item Compiler" (optional)             │
│                                                               │
│  Turn intent into executable spec files.                      │
│                                                               │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐     │
│  │ spec-api  │ │ spec-ui   │ │ spec-vibe │ │ spec-bdd  │     │
│  │ endpoints │ │ components│ │ fast/loose│ │ scenarios │     │
│  └───────────┘ └───────────┘ └───────────┘ └───────────┘     │
│                                                               │
│  Domain-optimized. Opinionated. Customizable.                 │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼ produces
┌───────────────────────────────────────────────────────────────┐
│                         specs/*.md                            │
│                                                               │
│  YAML frontmatter + verifiable "Done When"                   │
│  Skills can be loose about approach, never about done-ness    │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼ consumed by
┌───────────────────────────────────────────────────────────────┐
│        Baldrick Core: "Executor Kernel + Feedback Engine"     │
│                                                               │
│  1. Select next incomplete spec (deterministic)               │
│  2. Run agent with spec + context                             │
│  3. Apply feedback cycles (test/lint/build)                   │
│  4. Detect stuckness, apply corrective pressure               │
│  5. Update state, record decisions                            │
│  6. Repeat until all specs pass                               │
│                                                               │
│  Boring. Mechanical. Never changes.                           │
└───────────────────────────────────────────────────────────────┘
```

### Phase 1: Skills (Optional)

Skills are the "intelligence" layer. They turn vague intent into precise specs.

**What skills do:**
- Ask clarifying questions (or infer defaults)
- Emit different spec styles (API, UI, vibe, BDD)
- Include domain checklists (OpenAPI examples, a11y criteria)
- Encode constraints ("must use existing auth middleware")

**What skills don't do:**
- Run the execution loop
- Modify Baldrick's behavior

**You can skip skills entirely** - just create specs manually. Skills are optional enhancement, not required infrastructure.

### Phase 2: Baldrick Core

Baldrick is the "executor kernel." It doesn't think, it executes.

**What Baldrick does:**
- **Selects** the next incomplete spec (deterministically, by priority/risk)
- **Runs** the agent with the selected spec highlighted
- **Applies** feedback cycles (build, test, lint every iteration)
- **Detects** when stuck (no progress for 2+ iterations)
- **Records** what happened (progress.txt) and why (tradeoffs.md)

**What Baldrick doesn't do:**
- Generate specs
- Infer user intent
- Make domain decisions

This boundary is sacred. Baldrick is deliberately "dumb" so it stays predictable.

---

## Key Design Decisions

### 1. Deterministic Selection (Bash Decides, Not Claude)

In Ralph, the AI picks which task to work on. In Baldrick, bash selects mechanically:

```
priority (high→low) → risk (spike→polish) → created (old→new) → id (A→Z)
```

**Why?**
- **Predictable**: Same state → same selection (auditable)
- **Prevents gaming**: Claude can't skip hard specs
- **Debuggable**: You can trace exactly why a spec was chosen
- **Like Make**: Selection is mechanical, not magical

### 2. Redundant Completion (3 Signals Must Agree)

Ralph checks for one signal: `<promise>COMPLETE</promise>`. Baldrick requires three:

1. `.baldrick-done` file exists
2. `<promise>COMPLETE</promise>` token in output
3. All specs have `passes: true`

If they disagree, something's wrong. Better to keep going than declare false victory.

### 3. Feedback as Core Primitive

Baldrick isn't just "implement", it's "implement + self-correct."

Reviewer cadence is configurable: `REVIEW_EVERY=N` (default 5). Every Nth iteration, Baldrick switches from **Worker** to **Reviewer** mode:

- Checks progress.txt to verify we're on track
- Verifies completed specs actually work (runs tests)
- Fixes issues before continuing
- Tries a different approach if stuck 2+ iterations

This prevents broken code from compounding across iterations.

| Frequency | Feedback | Purpose |
|-----------|----------|---------|
| Every iteration | build, test | Did it break? |
| Every Nth | reviewer mode | Quality check, fix issues |
| On failure | Error analysis | Constrain next iteration |
| On stuck | Alternate strategy | Try different approach |

### 4. Observability as First-Class Output

Baldrick's job isn't just to change code, it's to produce a traceable execution record. Autonomous work without an audit trail is technical debt.

Four artifacts, distinct purposes:

| Artifact | Question Answered | Audience |
|----------|-------------------|----------|
| `progress.txt` | What happened? | Developers |
| `tradeoffs.md` | Why was it done? | Future maintainers |
| `logs/iter-*.json` | Structured iteration data | Automation/analysis |
| `logs/baldrick-*.log` | Raw Claude output | Debugging |

Every decision is traceable. "Why did it do that?" is always answerable.

**Baldrick MUST log a tradeoff when it:**
- Changes or introduces a data model / schema
- Makes a security or auth decision
- Chooses between architectural approaches
- Introduces a new dependency
- Changes a public API contract

---

## When to Use What

### Use Ralph when:
- Maximum simplicity
- Experimenting with the loop concept
- Writing your own prompt from scratch
- Minimal dependencies

### Use Baldrick when:
- Running 10+ iterations autonomously
- Need priority-based task selection
- Want stuck detection and recovery
- Need audit trail (progress + tradeoffs)
- Want optional skills for spec creation
- Running on production codebases

### Migration from Ralph

If you're using Ralph and want to try Baldrick:

1. Copy `baldrick` to your project (single file, self-contained)
2. Convert `prd.json` → `specs/*.md` (one spec per file with YAML frontmatter)
3. Use `baldrick run` instead of `./ralph.sh`

Or install skills for spec creation: `cp -r skills/* ~/.claude/skills/`

---

## The Philosophy

**1. Specs over tasks.** Don't tell the agent what to do, tell it what "done" looks like.

**2. Deterministic over smart.** Let bash pick tasks mechanically. The agent implements, doesn't prioritize.

**3. Feedback over hope.** If tests are red, you're not done. Build self-correction into the loop.

**4. Observable over opaque.** Every iteration writes progress. Every decision logs tradeoffs.

**5. Stable core, evolving skills.** The kernel never changes. Intelligence lives in skills.

These aren't AI-specific ideas. They're just good engineering applied to autonomous coding.

---

## Reference: Spec Format v1

Minimal valid spec:

```yaml
---
title: "Feature Title"
passes: false              # Set true when done
---
```

Recommended spec:

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

**Required fields:**
- `title` - non-empty string
- `passes` - boolean (true/false)

**Validated fields** (warned if invalid):
- `priority` - must be: high | medium | low
- `risk` - must be: spike | integration | standard | polish
- `created` - must be YYYY-MM-DD format

**Optional fields:**
- `id` - unique identifier (needed if other specs depend on this)
- `depends_on` - array of spec **ids** (not filenames)
- `blocked_by` - human-readable blocker description (spec will be skipped)

**Field meanings:**
- `passes: true` means all quality gates have passed (build, test, lint)
- `priority` reflects user/business urgency, not implementation difficulty

Required section: `## Done When` with verifiable checkboxes

See [templates/example.md](templates/example.md) for a complete example.

---

## Reference: Selection Algorithm

```
1. FILTER: Remove specs where:
   - passes = true (already done)
   - blocked_by is set (manually blocked)
   - dependencies unmet (any depends_on spec has passes = false)
   - dependencies unknown (depends_on id not found) → treated as blocked

2. SORT: priority (high→low) → risk (spike→polish) → created (old→new) → id (A→Z)

3. SELECT: Return first spec after sort
```

**Behavior notes:**
- Unknown `depends_on` ids cause the spec to be treated as blocked (not a validation error)
- Specs with unmet dependencies are silently skipped until dependencies pass
- `baldrick validate` checks YAML structure only, not dependency resolution
- `passes` is the source of truth for completion

Risk order: spike → integration → standard → polish (resolve uncertainty first, polish last)

---

## Reference: File Structure

```
project/
├── baldrick                    # Executor kernel (single file, self-contained)
├── skills/                     # Optional: spec creation skills
│   ├── spec-create/SKILL.md
│   ├── spec-vibe/SKILL.md
│   └── ...
├── specs/                      # Work items
│   └── *.md
├── docs/                       # Generated documentation
├── templates/                  # Spec templates
├── progress.txt                # What happened (human narrative)
├── tradeoffs.md                # Why it was done (decisions)
├── baldrick-learnings.md       # Permanent patterns
└── logs/                       # Iteration data
    ├── baldrick-*.log          # Raw Claude output per run
    └── iter-*.json             # Structured data per iteration
```

---

## Summary

Baldrick = Ralph + batteries.

Same core loop, but:
- Deterministic selection (bash decides)
- Redundant completion (3 signals)
- Feedback cycles (test/lint/review)
- Stuck detection (automatic recovery)
- Full observability (progress + tradeoffs + logs)

The two-phase architecture keeps it simple:
- **Skills** compile intent → specs (optional, domain-smart)
- **Baldrick** executes specs → code (stable, mechanical)

For more details on usage, see [README.md](README.md).
