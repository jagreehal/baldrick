---
id: calc-errors-v1
title: "Calculator Error Handling"
passes: false
priority: low
risk: polish
created: 2026-01-13
depends_on: [calc-display-v1]
---

# Calculator Error Handling

> **One-liner:** Handle division by zero and invalid operations gracefully

## Context

**User:** Developer testing Baldrick loop
**Trigger:** After display formatting is complete
**Success:** Calculator shows helpful error messages instead of crashing

## Scope

**In:**
- Division by zero shows "Error"
- Multiple decimal points prevented
- Multiple operators in a row prevented (last one wins)
- Error clears on next number press

**Out:**
- Recovery from JavaScript errors
- Input validation messages

## Examples

| Input | Expected Output |
|-------|-----------------|
| `5 / 0 =` | `Error` |
| `1..2` | `1.2` (second decimal ignored) |
| `5 + - 3 =` | `2` (last operator wins: 5 - 3) |
| After error, press `5` | `5` (error cleared) |

## Scenarios

### Scenario: Division by zero
```gherkin
Given the calculator display shows "0"
When I press "5", "/", "0", "="
Then the display shows "Error"
```

### Scenario: Error recovery
```gherkin
Given the calculator display shows "Error"
When I press "5"
Then the display shows "5"
And the calculator is ready for new input
```

### Scenario: Prevent multiple decimals
```gherkin
Given the display shows "3.14"
When I press "."
Then the display still shows "3.14"
```

### Scenario: Operator replacement
```gherkin
Given I have entered "5", "+"
When I press "-"
Then the pending operation is subtraction
And when I press "3", "="
Then the display shows "2"
```

## Done When

- [ ] Division by zero shows "Error"
- [ ] Multiple decimal points are prevented
- [ ] Consecutive operators replace previous
- [ ] Error state clears on number press
- [ ] All previous functionality still works
