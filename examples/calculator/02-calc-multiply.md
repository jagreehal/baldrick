---
id: calc-multiply-v1
title: "Calculator Multiply and Divide"
passes: false
priority: medium
risk: standard
created: 2026-01-13
depends_on: [calc-basic-v1]
---

# Calculator Multiply and Divide

> **One-liner:** Add multiply and divide operations to calculator

## Context

**User:** Developer testing Baldrick loop
**Trigger:** After basic operations are working
**Success:** Calculator can multiply and divide numbers

## Scope

**In:**
- Multiply (*) button and operation
- Divide (/) button and operation
- Proper operator precedence NOT required (left-to-right is fine)

**Out:**
- Decimal numbers
- Error handling for divide by zero (next spec)

## Examples

| Input | Expected Output |
|-------|-----------------|
| `6 * 7 =` | `42` |
| `20 / 4 =` | `5` |
| `5 + 3 * 2 =` | `16` (left-to-right: (5+3)*2) |

## Scenarios

### Scenario: Multiplication
```gherkin
Given the calculator display shows "0"
When I press "6", "*", "7", "="
Then the display shows "42"
```

### Scenario: Division
```gherkin
Given the calculator display shows "0"
When I press "2", "0", "/", "4", "="
Then the display shows "5"
```

### Scenario: Chained operations
```gherkin
Given the calculator display shows "0"
When I press "1", "0", "+", "5", "*", "2", "="
Then the display shows "30"
```

## Done When

- [ ] Multiply button (*) exists and works
- [ ] Divide button (/) exists and works
- [ ] Chained operations work left-to-right
- [ ] All previous functionality still works
