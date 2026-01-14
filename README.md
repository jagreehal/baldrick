# baldrick

**Autonomous AI coding loops that actually finish.**

![Version](https://img.shields.io/badge/version-1.3.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Works With](https://img.shields.io/badge/works%20with-Node%20%7C%20Python%20%7C%20Go-orange)
![Zero Dependencies](https://img.shields.io/badge/zero%20dependencies-%E2%9C%94-green)

![Baldrick](baldrick.png)

> *"I have a cunning plan!"* - Baldrick, Blackadder

## Quick Start

```bash
curl -sO https://raw.githubusercontent.com/jagreehal/baldrick/main/baldrick && chmod +x baldrick && ./baldrick init
```

Create a spec, run `./baldrick run 5`, watch Claude implement it.

---

## What Is This?

Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/) — define what "done" looks like, run Claude repeatedly until it gets there. Baldrick adds structure:

- **Deterministic selection** — bash picks tasks mechanically (priority/risk), prevents AI from skipping hard work
- **Redundant completion** — 3 signals must agree (done file + token + all specs pass), prevents false victories
- **Feedback loops** — build/test/lint every iteration, reviewer mode every Nth (configurable, default 5) checks progress and fixes issues, self-correcting
- **Full observability** — progress.txt (what happened), tradeoffs.md (why), logs/ (raw data)
- **Works with any language** — configure build/test/lint commands for your stack (see [Configuration](#configuration))

See [design.md](design.md) for the full philosophy.

---

## Why Loops Work

Every AI coding session accumulates context: files read, commands run, wrong turns taken, half-baked plans. You can add but can't delete. Eventually it starts repeating itself, "fixing" the same bug different ways, confidently undoing its own work. That's **context pollution**.

Baldrick doesn't try to clean the context. It throws it away and starts fresh. Progress persists in files (specs, progress.txt, git). Failures evaporate with the session. Each iteration reconstructs reality from the filesystem, not chat history.

---

## Getting Started

After running `./baldrick init`, you'll have:

```text
specs/                  # Your feature specs go here
templates/              # Spec templates (minimal, standard, detailed, example)
progress.txt            # Session progress log
baldrick-learnings.md   # Permanent patterns
tradeoffs.md            # Decision log (auto-created if missing)
logs/                   # Iteration logs (auto-created if missing)
```

**Requires:** [Claude Code CLI](https://docs.anthropic.com/claude-code) (configured with appropriate permissions)

Now create a spec. This tells baldrick what "done" looks like:

**Quick way:**
```bash
./baldrick new hello --title "Add Hello Function"
```

**Or manually:**
```bash
cat > specs/hello.md << 'EOF'
---
title: "Add Hello Function"
passes: false
---
# Add Hello Function
## Done When
- [ ] Function `hello(name)` returns "Hello, {name}!"
- [ ] Tests pass
EOF
```

> For better results, see [Spec Templates](#spec-templates) or use Claude Code skills (spec-create, spec-vibe, spec-detailed)

`passes: true` means the spec's Done When criteria are met **and** quality gates (build/test/lint) are green.

Run it:

```bash
./baldrick run 1           # Run one iteration
./baldrick run 10 --verbose # Run with streaming output
./baldrick status          # Check progress
```

That's it. Claude implements the feature, runs quality gates (build/test/lint), and commits when all checks pass.

**Run options:**
- `--dry-run` - Show what would run without executing
- `--verbose` or `-v` - Stream Claude output in real-time (instead of spinner)
- `--stream=PATH` - Enable TUI event streaming (advanced)
- `--control=PATH` - Enable TUI control socket (advanced)

---

## For Non-Technical Users

Don't want to write specs manually? Use any AI assistant (ChatGPT, Gemini, Claude):

1. **Chat about your feature** - Describe what you want to build
2. **Paste the generator template** - Copy [`templates/spec-generator.md`](templates/spec-generator.md) into your chat
3. **Get a Baldrick spec** - The AI generates a properly formatted spec
4. **Save to specs folder** - Copy output to `specs/your-feature.md` (Baldrick auto-discovers specs here)
5. **Run Baldrick** - `./baldrick run` or `./baldrick once your-feature` to run a specific spec

This bridges conversational AI (great for brainstorming) with Baldrick (great for execution).

**Tip:** If your feature is large, the AI will generate multiple specs. Save each one as a separate file.

---

## The Development Cycle

```
┌─────────────────────────────────────────────────────────────────┐
│                      Your specs/*.md                            │
│                                                                 │
│  YAML frontmatter + verifiable done criteria                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Baldrick Core (the loop)                      │
│                                                                 │
│  1. Select next incomplete spec (by priority)                   │
│  2. Run Claude with spec as context                             │
│  3. Claude implements the feature                               │
│  4. Run tests/build/lint                                        │
│  5. Commit changes                                              │
│  6. Repeat until all specs pass                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                      Working code + commits
```

The workflow is simple:

1. **Define specs** — Write what "done" looks like
2. **Run baldrick** — `./baldrick run 10`
3. **Review code** — Check `git log`, `git diff`, read the changes
4. **Repeat** — Add more specs as needed

Each iteration rebuilds context from files (specs + progress + learnings), not chat history—so it runs indefinitely without memory drift.

---

## Commands

```bash
# Execution
./baldrick run [n] [--dry-run] [--verbose|-v]    # Run n iterations (default: 10)
./baldrick once <name> [--max n] [--verbose|-v]  # Focus on single spec until done
./baldrick tui                                  # Launch interactive TUI

# Status
./baldrick status                 # Show specs and progress
./baldrick validate               # Check specs for errors
./baldrick doctor                 # Check configuration

# Utilities
./baldrick init                   # Setup project structure
./baldrick new <name>             # Create new spec interactively
./baldrick archive                # Archive current session
./baldrick archive list           # List archived sessions
./baldrick archive restore <name> # Restore archived session
./baldrick logs [show|tail|list|activity|clean]  # View iteration logs
./baldrick skip <spec>            # Temporarily skip a spec
./baldrick unskip <spec>          # Re-enable a skipped spec
./baldrick --version              # Show version
```

**Aliases:** `s` → status, `r` → run, `v` → validate, `n` → new, `l` → logs

---

## Spec Templates

Don't want to start from scratch? Download a template:

| Template | Description | Download |
|----------|-------------|----------|
| **Minimal** | Just `title` and `passes` | [View](templates/minimal.md) |
| **Standard** | Adds context, examples, scenarios | [View](templates/standard.md) |
| **Detailed** | Full spec with diagrams, dependencies | [View](templates/detailed.md) |
| **Example** | Filled-in spec (task priority feature) | [View](templates/example.md) |
| **Spec Generator** | Paste into ChatGPT/Gemini to generate specs | [View](templates/spec-generator.md) |

Or run `./baldrick init` to create templates locally in `templates/`.

---

## Configuration

Set these environment variables to customize baldrick:

| Variable | Default | Description |
|----------|---------|-------------|
| `BUILD_CMD` | `npm run build` | Build command |
| `TEST_CMD` | `npm test` | Test command |
| `LINT_CMD` | `npm run lint` | Lint command |
| `QUALITY_TIER` | `production` | Hint to Claude: prototype (speed) / production (robust) / library (API stability) |
| `REVIEW_EVERY` | `5` | Reviewer mode every N iterations |
| `CLAUDE_TIMEOUT` | `300` | Max seconds per Claude call (prevents hung loops) |
| `NOTIFY_CMD` | (none) | Notification webhook |

**For non-Node projects**, set these before running:

```bash
# Python
export BUILD_CMD="python -m py_compile *.py"
export TEST_CMD="pytest"
export LINT_CMD="ruff check ."

# Go
export BUILD_CMD="go build ./..."
export TEST_CMD="go test ./..."
export LINT_CMD="golangci-lint run"

# Or edit defaults in baldrick script (lines 30-32)
```

**Check your setup:**

```bash
./baldrick doctor
```

---

## Essential Tips

### 1. Baldrick Is A Loop

AI coding evolved through phases: vibe coding → planning → multi-phase. Baldrick simplifies everything by running the same prompt in a loop:

```bash
for ((i=1; i<=n; i++)); do
  SPEC=$(get_next_spec)  # Deterministic: priority → risk → date → id
  claude -p "## SELECTED SPEC
@$SPEC
@progress.txt @baldrick-learnings.md @specs/*.md ..."
done
```

The key: **Baldrick selects the task mechanically (by priority/risk). Claude implements it.**

### 2. Write Clear Specs

Instead of "implement login," describe exactly what login means:

```yaml
---
title: "Login"
passes: false
priority: high    # high | medium | low (optional)
risk: integration # spike | integration | standard | polish (optional)
---

# Login

## Done When
- [ ] POST `/api/auth/login` returns JWT
- [ ] Invalid credentials return 401
- [ ] Tests pass
```

**Spec should be so clear that Claude can implement it without asking questions.**

**Right-sized specs:**

| Too Big | Right Size |
|---------|------------|
| "Build entire auth" | "Add login endpoint" |
| "Implement checkout" | "Add cart summary" |
| "Refactor API layer" | "Extract validation" |

### 3. Use Feedback Loops

The more feedback loops, the higher quality code:

| Feedback Loop | What It Catches |
|---------------|-----------------|
| Build command | Compilation errors, missing deps |
| Type checking | Type mismatches, missing props |
| Unit tests | Broken logic, regressions |
| Linting | Code style, potential bugs |

**Claude can't declare victory if the tests are red.**

### 4. Track Progress

Baldrick maintains `progress.txt` across iterations. Claude reads `## Codebase Patterns` first before starting work:

```markdown
# Baldrick Progress Log

## Codebase Patterns
- Error handling: use CustomError from src/errors.ts
- API responses: always wrap in { data, error } shape

## Key Files
- src/auth/login.ts - authentication logic
```

For patterns that survive archiving, use `baldrick-learnings.md`. This acts as **guardrails**: mistakes evaporate with each session, but lessons accumulate permanently. The same mistake shouldn't happen twice.

### 5. Handle Problems

**Reviewer Mode:** Every Nth iteration (configurable via `REVIEW_EVERY`, default 5), baldrick switches from Worker to Reviewer mode. The reviewer:
- Checks progress.txt to verify we're on track
- Verifies completed specs actually work (runs tests)
- Fixes issues before continuing
- Tries a different approach if stuck 2+ iterations

This prevents broken code from compounding across iterations.

**Stuck Detection:** If nothing changes for 2+ iterations, baldrick warns Claude to try a different approach. If truly stuck, outputs `<promise>BLOCKED</promise>` to stop.

**Prevention:** Smaller specs, clear requirements, stable tests.

### 6. Customize for Your Project

```bash
# AFK notifications
NOTIFY_CMD="curl -d 'Baldrick done' ntfy.sh/my-topic"

# macOS notification
NOTIFY_CMD="osascript -e 'display notification \"Done\"'"

# Quality tier
QUALITY_TIER=production   # prototype | production | library
```

---

## Enhancing with Skills

Skills add intelligence on top of baldrick's stable core. They're **optional** - baldrick works without them if you create specs manually.

### Installing Skills

```bash
# Global (all projects)
cp -r skills/* ~/.claude/skills/

# Per-project
cp -r skills/ .claude/skills/
```

### Spec Creation Skills

Turn natural language intent into structured specs:

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `spec-create` | Interactive spec with questions | Default for new features |
| `spec-vibe` | Fast spec, no questions | Quick prototyping |
| `spec-detailed` | Comprehensive with diagrams | Complex features |

**Usage** (in Claude Code, type this as a prompt):

```
Load the spec-create skill and create a spec for user authentication
```

Skills are prompt files in `.claude/skills/`—tell Claude to load one, then describe your intent.

### Loop Skills

Specialized iteration patterns:

| Skill | Purpose |
|-------|---------|
| `coverage-loop` | Increase test coverage to target % |
| `lint-loop` | Fix all linting errors |
| `entropy-loop` | Reduce code smells and duplication |
| `pr-docs` | Generate PR documentation |

Load the skill and describe your goal. The skill iterates until complete.

---

## Interactive TUI

Baldrick includes TUI integration for real-time monitoring and control:

```bash
./baldrick tui
```

The TUI provides:
- Real-time progress updates
- Pause/resume/stop controls
- Live log streaming
- Spec status visualization

Requires `baldrick-tui` to be installed separately.

## Advanced Setup

### From Marketplace

```bash
# Add the marketplace
/plugin marketplace add jagreehal/baldrick

# Install the plugin
/plugin install baldrick@baldrick-marketplace
```

Or add to `.claude/settings.local.json`:

```json
{
  "extraKnownMarketplaces": {
    "baldrick-marketplace": {
      "source": {
        "source": "github",
        "repo": "jagreehal/baldrick"
      }
    }
  },
  "enabledPlugins": {
    "baldrick@baldrick-marketplace": true
  }
}
```

### Testing & Mocking

```bash
# Source without executing
source ./baldrick

# Disable colors for test output
NO_COLOR=1 ./baldrick status

# Mock Claude for integration tests
CLAUDE_RUNNER=./mock-claude.sh ./baldrick run 1
```

### Files Reference

| File | Purpose |
|------|---------|
| `baldrick` | Core loop script (single file, self-contained) |
| `specs/*.md` | Feature specs with YAML frontmatter |
| `progress.txt` | Session learnings |
| `baldrick-learnings.md` | Permanent patterns |
| `logs/baldrick-*.log` | Raw Claude output |
| `logs/iter-*.json` | Structured iteration data |
| `logs/activity.log` | Activity audit trail |

### Logs Command

View iteration logs and activity:

```bash
./baldrick logs              # Show latest Claude output
./baldrick logs tail          # Follow latest log in real-time
./baldrick logs list          # List all log files
./baldrick logs activity      # Show activity audit trail
./baldrick logs clean         # Remove all log files
```

### Skip/Unskip Specs

Temporarily exclude specs from selection:

```bash
./baldrick skip <spec-name>   # Mark spec as blocked
./baldrick unskip <spec-name>  # Re-enable a skipped spec
```

Skipped specs are excluded from selection until unskipped. Useful for temporarily blocking specs that need external dependencies or are waiting on other work.

---

## When NOT to Use

- **Exploratory work** - Spikes, prototypes, "what if..."
- **Major refactors** - Cross-system changes without clear criteria
- **Security-critical code** - Auth, payments, crypto need human review
- **Cross-team dependencies** - Blocked by external APIs

For these, stay in human-in-the-loop mode.

---

## Philosophy

1. **Specs over tasks.** Don't tell Claude what to do—tell it what done looks like.
2. **Deterministic selection.** Baldrick picks the task mechanically (priority/risk). Claude implements.
3. **Quality is explicit.** The agent doesn't know if this is prototype or production.
4. **Small steps compound.** Many small commits beat one large commit.
5. **Feedback loops are non-negotiable.** If tests are red, you're not done.

These aren't AI-specific ideas. They're just good engineering. Baldrick makes them automatic.

---

## License

MIT

---

*Baldrick is named after the character from Blackadder who always had "a cunning plan." Unlike the TV character, this Baldrick's plans actually work.*
