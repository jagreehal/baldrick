---
title: "[FEATURE_TITLE]"
passes: false
priority: medium    # high | medium | low
risk: standard      # spike | integration | standard | polish
created: [DATE]
---

<!--
Optional fields (defaults if omitted):
- priority: medium
- risk: standard
- created: (unset)
-->

# [FEATURE_TITLE]

> **One-liner:** [What this does in 10 words or less]

## Context

**User:** [Who uses this?]
**Trigger:** [What kicks this off?]
**Success:** [What does "working" look like?]

## Scope

**In:** [What we're building]
**Out:** [What we're NOT building]

## Examples

| Input | Expected Output |
|-------|-----------------|
| `[real example input]` | `[real example output]` |
| `[edge case input]` | `[edge case handling]` |
| `[error trigger]` | `[error response]` |

## Scenarios

> Gherkin syntax is documentation for Claude—not executable Cucumber tests.

### Scenario: [Happy path]
```gherkin
Given [concrete precondition]
When [specific action]
Then [explicit result]
```

### Scenario Outline: [Validation cases]
```gherkin
Given [precondition]
When [action with <input>]
Then [expected <output>]

Examples:
  | input           | output              |
  | valid_value     | success response    |
  | invalid_value   | error message       |
  | edge_case       | handled gracefully  |
```

## Done When

- [ ] [Specific criterion 1]
- [ ] [Specific criterion 2]
- [ ] Build passes
- [ ] Tests pass
<!-- For UI specs only: -->
<!-- - [ ] Screenshot verified: UI matches expected layout -->
