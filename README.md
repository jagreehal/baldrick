# baldrick

**AI-assisted feature execution for Claude Code.**

👋 Hi Ralph, meet the man with a cunning plan, Baldrick.

## Quick Start

```bash
curl -O https://raw.githubusercontent.com/jagreehal/baldrick/main/baldrick
chmod +x baldrick
./baldrick init spec
./baldrick create login "user authentication"
./baldrick run 10
./baldrick status
```

---

## How It Works

1. Define features as markdown specs with `passes: false`
2. Claude picks the first incomplete spec, implements it
3. Once done criteria pass, mark `passes: true`
4. Repeat

Each spec includes: context, scope boundaries, examples, BDD scenarios, diagrams, and done criteria.

> **Note:** The gherkin syntax is for Claude to understand expected behaviour—it's documentation, not executable Cucumber tests. Claude reads these scenarios and implements code + tests to match.

### The loop (every iteration)

1. Read specs (`passes: false` first), `progress.txt`, and logs
2. Implement the next spec only
3. Run build, tests, and lint (`BUILD_CMD`, `TEST_CMD`, `LINT_CMD`)
4. Flip the spec to `passes: true` when it satisfies Done When
5. Append evidence and learnings to `progress.txt`
6. Commit your work (the prompt asks Claude to do this)
7. Stop when all specs pass or `.baldrick-done` exists

Pseudo-loop:

```bash
for ((i=1; i<=n; i++)); do
  claude --permission-mode acceptEdits \
    -p "@specs/*.md @progress.txt ... instructions"
done
```

### Reviewer mode (every 5th iteration)

Every 5th iteration, Claude switches from **Worker** to **Reviewer** mode:

1. Check progress.txt — are we on track?
2. Verify completed specs — do they actually work?
3. Run tests to validate previous work
4. Fix issues before moving on
5. If stuck 2+ iterations, try a different approach

This prevents broken code from compounding across iterations.

### State and files

- `specs/*.md` — feature specs with YAML frontmatter
- `progress.txt` — session learnings (per-run context)
- `baldrick-learnings.md` — permanent codebase patterns (survives across specs)
- `logs/` — per-run logs
- `docs/` — generated PR docs
- `.baldrick-done` — file-based completion signal
- `baldrick-features.json` — simple mode (JSON) if you use `./baldrick init`
- `templates/` — spec templates shipped with the script

After `./baldrick init spec`:

```text
specs/
progress.txt
logs/
docs/
templates/detailed/{feature.md, feature-detailed.md}
```

### Progress format (append in `progress.txt`)

```markdown
## 2025-01-07 - login
- What was done: Implemented login happy path + errors
- Test output: npm test (paste real output)
- Next: Add rate limiting later
```

### Example Spec

````markdown
---
title: "Login"
passes: false
created: 2025-01-06
---

# Login

> **One-liner:** Users authenticate with email and password

## Context

**User:** Registered users with existing accounts
**Trigger:** User submits login form with email and password
**Success:** User receives valid JWT token and can access protected routes

## Scope

**In:** Email/password form, validation, JWT token, error messages
**Out:** OAuth, password reset, remember me, rate limiting

## Flow

```text
┌────────┐  credentials   ┌────────┐  lookup   ┌────────┐
│  User  │───────────────>│  API   │──────────>│   DB   │
└────────┘                └───┬────┘           └────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               │               ▼
        ┌──────────┐          │         ┌──────────┐
        │ 200 JWT  │          │         │ 401 Error│
        └──────────┘          │         └──────────┘
                        valid?/invalid?
```

## Examples

| Input | Expected Output |
|-------|-----------------|
| `{email: "alice@test.com", password: "Secret123"}` | `200` + JWT token |
| `{email: "wrong@test.com", password: "Secret123"}` | `401 Invalid credentials` |
| `{email: "not-an-email", password: "x"}` | `400 Invalid email format` |

## Scenarios

### Scenario: Successful login
```gherkin
Given a user exists with email "alice@test.com" and password "Secret123"
When I POST to "/api/auth/login" with those credentials
Then I receive a 200 response with a valid JWT token
```

### Scenario Outline: Login validation
```gherkin
Given a user exists with email "alice@test.com" and password "Secret123"
When I POST to "/api/auth/login" with <email> and <password>
Then I receive a <status> response with <body>

Examples:
  | email              | password    | status | body                    |
  | alice@test.com     | Secret123   | 200    | valid JWT token         |
  | alice@test.com     | wrong       | 401    | "Invalid credentials"   |
  | unknown@test.com   | Secret123   | 401    | "Invalid credentials"   |
  | not-an-email       | x           | 400    | "Invalid email format"  |
```

## Done When

- [ ] POST `/api/auth/login` returns JWT for valid credentials
- [ ] Invalid credentials return 401
- [ ] Malformed email returns 400
- [ ] Build passes
- [ ] Tests pass
````

Claude reads this spec, implements exactly what's defined, then sets `passes: true`.

---

## Commands

| Command | Description | Example |
| ------- | ----------- | ------- |
| `init` | Setup simple mode (JSON) | `./baldrick init` |
| `init spec` | Setup spec mode (markdown) | `./baldrick init spec` |
| `run [n]` | Run n iterations (default: 10) | `./baldrick run 5` |
| `once <name> [--max n]` | Run single spec until done | `./baldrick once login --max 3` |
| `status` | Show progress status | `./baldrick status` |
| `create <name> [desc]` | Create spec interactively | `./baldrick create login "user auth"` |
| `vibe <name> [desc]` | Create spec (no questions) | `./baldrick vibe signup` |
| `spec <name> [desc]` | Create comprehensive spec | `./baldrick spec checkout` |
| `docs <name>` | Generate PR documentation | `./baldrick docs login` |
| `validate` | Check specs for errors | `./baldrick validate` |
| `help` | Show all commands | `./baldrick help` |

---

## Built-in Safeguards

- Stuck detection: Warns after two unchanged iterations; nudges Claude to change approach.
- Evidence required: Paste real build/test/lint output into `progress.txt`.
- Completion signals: Either set all specs to `passes: true` or create `.baldrick-done`.
- Single-spec focus: Each iteration works on one spec only to avoid thrash.

---

## Monitoring

Quick commands to check progress:

```bash
# Spec status (requires jq)
cat specs/*.md | grep -E "^(title|passes):" | paste - -

# Recent commits
git log --oneline -10

# Learnings so far
cat progress.txt | tail -50
```

---

## Learnings Guidance

**Add to `baldrick-learnings.md`:**

- "When modifying X, also update Y"
- "This module uses pattern Z"
- "Tests require dev server running"

**Don't add:**

- Spec-specific details (goes in progress.txt)
- Temporary notes
- Info already documented elsewhere

---

## Configuration

Edit lines 23-25 in `baldrick` for your project:

```bash
# Node.js (default)
BUILD_CMD="npm run build"
TEST_CMD="npm test"
LINT_CMD="npm run lint"

# Python
BUILD_CMD=""
TEST_CMD="pytest"
LINT_CMD="ruff check ."

# Rust
BUILD_CMD="cargo build"
TEST_CMD="cargo test"
LINT_CMD="cargo clippy"
```

**Templates** — Both `feature.md` and `feature-detailed.md` include optional screenshot verification (commented out). Uncomment for UI specs.

---

## Critical Success Factors

**Small specs** — Each spec must fit in one context window. Break large features into smaller pieces.

| Too Big | Right Size |
| ------- | ---------- |
| "Build entire auth system" | "Add login form" |
| "Implement checkout flow" | "Add cart summary component" |

**Fast feedback** — Build and tests must run quickly. Without fast feedback, broken code compounds.

**Explicit criteria** — Done When must be verifiable, not vague.

| Vague | Explicit |
| ----- | -------- |
| "Users can log in" | "POST `/api/login` returns JWT for valid credentials" |
| "Handle errors" | "Invalid email returns 400 with message" |

**Learnings compound** — By spec 10, Claude knows patterns from specs 1-9 via progress.txt.

---

## Common Gotchas

**Stuck detection** — Triggers after 2 unchanged iterations. If stuck, try a different approach or break the spec smaller.

**Idempotent migrations** — Use `IF NOT EXISTS` patterns:

```sql
ALTER TABLE users ADD COLUMN IF NOT EXISTS email TEXT;
```

**Interactive prompts** — Bypass with empty input:

```bash
echo -e "\n\n\n" | npm run db:generate
```

**Schema changes cascade** — After editing schema, check: server actions, UI components, API routes. Fixing related files is OK—not scope creep.

**progress.txt grows** — Consider pruning between projects or moving permanent patterns to a learnings file.

---

## When NOT to Use

- **Exploratory work** — Spikes, prototypes, "what if we tried..."
- **Major refactors** — Cross-system changes without clear acceptance criteria
- **Security-critical code** — Auth, payments, crypto—needs human review
- **Cross-team dependencies** — Blocked by external APIs or other teams

---

## Browser Testing (UI specs only)

For specs with UI changes, verify with screenshots before marking complete. Skip this for API-only specs.

```bash
# Using Playwright (install: npm i -D playwright)
npx playwright screenshot http://localhost:3000/your-page screenshot.png

# Or with a simple script
cat > verify-ui.js << 'EOF'
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.setViewportSize({ width: 1280, height: 900 });
  await page.goto(process.env.URL || 'http://localhost:3000');
  await page.screenshot({ path: 'screenshot.png' });
  await browser.close();
})();
EOF
node verify-ui.js
```

Add to your spec's Done When:

- `[ ] Screenshot verified: UI matches expected layout`

---

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/claude-code)
- Run `/sandbox` in Claude Code first

---

## License

MIT
