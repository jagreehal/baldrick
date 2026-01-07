# baldrick

**AI-assisted feature execution for Claude Code.**

👋 Hi ya, Ralph. Meet Baldrick.

> "I have a cunning plan!" - Baldrick, Blackadder

## Quick Start

```bash
curl -O https://raw.githubusercontent.com/jagreehal/baldrick/main/baldrick
chmod +x baldrick
./baldrick init
```

Edit `baldrick-features.json`, then:

```bash
./baldrick run 10
./baldrick status
```

That's it.

---

## How It Works

```json
{
  "project": "My App",
  "features": [
    {
      "category": "setup",
      "description": "Initialise project with TypeScript and tests",
      "passes": false
    }
  ]
}
```

Claude picks the first incomplete feature (`passes: false`), implements it, marks it done (`passes: true`), moves on.

---

## Spec Mode

For complex features with BDD specs, mermaid diagrams, and structured examples:

```bash
./baldrick init spec

./baldrick create login       # interactive - asks clarifying questions
./baldrick vibe signup        # no questions - sensible defaults
./baldrick spec checkout      # comprehensive - mermaid, ASCII, BDD

./baldrick run 10
./baldrick once login         # focus on single spec
./baldrick status
./baldrick docs login         # generate PR docs
```

Same simple tracking: `passes: false` → `passes: true`

More documentation before coding: BDD scenarios, example tables, scope boundaries, done criteria.

### Example Spec

```markdown
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

### Scenario: Invalid credentials
```gherkin
Given a user exists with email "alice@test.com" and password "Secret123"
When I POST to "/api/auth/login" with wrong password
Then I receive a 401 response with "Invalid credentials"
```

## Done When

- [ ] POST `/api/auth/login` returns JWT for valid credentials
- [ ] Invalid credentials return 401
- [ ] Malformed email returns 400
- [ ] Build passes
- [ ] Tests pass
```

Claude reads this spec, implements exactly what's defined, then sets `passes: true`.

---

## Commands

```bash
./baldrick init                    # setup simple mode (JSON)
./baldrick init spec               # setup spec mode (markdown)
./baldrick run [n]                 # run n iterations (default: 10)
./baldrick once <name> [--max n]  # run single spec until done (max iterations)
./baldrick status                  # show status
./baldrick create <name> [desc]   # create spec (interactive, optional description)
./baldrick vibe <name> [desc]     # create spec (no questions, optional description)
./baldrick spec <name> [desc]     # create comprehensive spec (optional description)
./baldrick docs <name>            # generate PR docs
./baldrick validate                # check specs for errors
./baldrick help                    # show all commands
```

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

---

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/claude-code)
- Run `/sandbox` in Claude Code first

---

## License

MIT
