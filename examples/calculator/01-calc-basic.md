---
id: calc-basic-v1
title: "Calculator Basic Operations"
passes: false
priority: high
risk: standard
created: 2026-01-13
depends_on: []
---

# Calculator Basic Operations

> **One-liner:** Create HTML calculator with add and subtract

## Context

**User:** Developer testing Baldrick loop
**Trigger:** First spec in the calculator feature set
**Success:** A working HTML page with calculator that can add and subtract

## Scope

**In:**
- Single HTML file (`calculator.html`)
- Basic UI with number buttons (0-9)
- Add (+) and subtract (-) operations
- Equals (=) button to compute result
- Clear (C) button to reset
- Display showing current input and result

**Out:**
- Multiply/divide (next spec)
- Error handling (later spec)
- Styling beyond basic functionality

## Examples

| Input | Expected Output |
|-------|-----------------|
| `5 + 3 =` | `8` |
| `10 - 4 =` | `6` |
| `5 + 3 + 2 =` | `10` |
| `C` | `0` (display cleared) |

## Scenarios

### Scenario: Basic addition
```gherkin
Given the calculator display shows "0"
When I press "5", "+", "3", "="
Then the display shows "8"
```

### Scenario: Basic subtraction
```gherkin
Given the calculator display shows "0"
When I press "9", "-", "4", "="
Then the display shows "5"
```

### Scenario: Clear resets display
```gherkin
Given the calculator display shows "42"
When I press "C"
Then the display shows "0"
```

## Done When

- [ ] `calculator.html` exists in project root
- [ ] Calculator displays numbers when buttons pressed
- [ ] Addition works correctly
- [ ] Subtraction works correctly
- [ ] Clear button resets to 0
- [ ] File can be opened in browser and works
