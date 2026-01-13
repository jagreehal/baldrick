---
id: calc-display-v1
title: "Calculator Display Formatting"
passes: false
priority: medium
risk: polish
created: 2026-01-13
depends_on: [calc-multiply-v1]
---

# Calculator Display Formatting

> **One-liner:** Add decimal support and display formatting

## Context

**User:** Developer testing Baldrick loop
**Trigger:** After multiply/divide are working
**Success:** Calculator handles decimals and formats large numbers

## Scope

**In:**
- Decimal point (.) button
- Decimal arithmetic
- Large number formatting (comma separators for display)
- Result truncation for long decimals (max 10 digits)

**Out:**
- Scientific notation
- Negative number entry (can result from subtraction though)

## Examples

| Input | Expected Output |
|-------|-----------------|
| `3.14 + 2.86 =` | `6` |
| `10 / 3 =` | `3.333333333` (truncated) |
| `1000000 + 1 =` | `1,000,001` (formatted) |
| `0.1 + 0.2 =` | `0.3` |

## Scenarios

### Scenario: Decimal input
```gherkin
Given the calculator display shows "0"
When I press "3", ".", "1", "4"
Then the display shows "3.14"
```

### Scenario: Decimal arithmetic
```gherkin
Given the calculator display shows "0"
When I press "1", "0", "/", "3", "="
Then the display shows a number starting with "3.33"
```

### Scenario: Large number formatting
```gherkin
Given the calculator display shows "0"
When I press "9", "9", "9", "9", "9", "9", "+", "1", "="
Then the display shows "1,000,000"
```

## Done When

- [ ] Decimal point button (.) exists and works
- [ ] Decimal arithmetic is accurate
- [ ] Large numbers display with commas
- [ ] Long decimals are truncated to fit display
- [ ] All previous functionality still works
