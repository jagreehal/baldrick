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

Each spec includes: context, scope boundaries, examples, BDD scenarios, mermaid diagrams, and done criteria.

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
