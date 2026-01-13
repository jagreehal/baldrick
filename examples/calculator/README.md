# Calculator Example

A complete example of Baldrick building a calculator through 4 autonomous iterations.

## The Specs

This example uses 4 specs with dependencies:

```
01-calc-basic (high priority)
    ↓
02-calc-multiply (medium, depends on basic)
    ↓
03-calc-display (medium, depends on multiply)
    ↓
04-calc-errors (low, depends on display)
```

## Running This Example

```bash
# Copy specs to your project
cp specs/*.md /path/to/your/project/specs/

# Configure for HTML (no build system)
export BUILD_CMD='echo "No build needed"'
export TEST_CMD='test -f calculator.html && echo "calculator.html exists"'
export LINT_CMD='echo "No lint configured"'
export QUALITY_TIER=prototype

# Run baldrick
./baldrick run 4
```

## What Gets Built

After all 4 iterations complete, you'll have `calculator.html` with:

- Number buttons (0-9)
- Basic operations (+, -, *, /)
- Decimal point support
- Display formatting (commas, truncation)
- Error handling (division by zero)
- Error recovery (press number to clear error)

## Results

From an actual test run:

| Iteration | Spec | Time | What Happened |
|-----------|------|------|---------------|
| 1 | calc-basic | ~80s | Created calculator.html with +/- |
| 2 | calc-multiply | ~80s | Verified */÷ already implemented |
| 3 | calc-display | ~15s | Added decimal + formatting |
| 4 | calc-errors | ~10s | Added error handling |

Total: ~5 minutes of autonomous work.

## Key Observations

1. **Fresh context each iteration**: Iteration 2 "discovered" that multiply/divide were already implemented by reading the file fresh.

2. **Dependencies work**: Spec 2 couldn't start until spec 1 was done. Baldrick enforced the ordering.

3. **Progress accumulates**: Each iteration added to `progress.txt` with learnings that future iterations could reference.

4. **Git-backed safety**: Each spec completion was committed, so interruptions wouldn't lose work.

## Sample progress.txt Output

```markdown
## 2026-01-13 14:02 - Calculator Basic Operations
Session: baldrick-20260113140149-99954
- What: Created calculator.html with basic add/subtract functionality
- Learnings:
  - Used accumulator pattern with state machine for calculator logic
  - Key variables: currentInput, previousValue, operator, shouldResetInput

## 2026-01-13 14:07 - Calculator Multiply and Divide
Session: baldrick-20260113140724-27390
- What: Verified multiply and divide operations already implemented
- Learnings:
  - Previous spec implementation was forward-thinking
  - Spec validation: always read existing code before implementing
```

Notice how iteration 2 noted that the previous implementation was "forward-thinking" — this Claude had no memory of the previous conversation but could see the results in the code.
