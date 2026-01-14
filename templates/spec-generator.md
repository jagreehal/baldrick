# Generate Baldrick Spec

> **How this works:** Chat about your app idea → paste this template → get structured specs → save to `specs/` folder → run `./baldrick run` → Claude builds it.

From the feature/requirements we've been discussing, generate a Baldrick spec using the exact format below. Ask me questions if you need more information or need to clarify anything.

---

## OUTPUT FORMAT

> **Save as:** `specs/[id].md`

````markdown
---
id: feature-name-v1
title: "Feature Title"
passes: false
priority: medium
risk: standard
created: YYYY-MM-DD
depends_on: []
---

# Feature Title

> **One-liner:** What this does in 10 words or less.

## Context

- **User**: Who uses this
- **Trigger**: What kicks it off
- **Success**: How we know it works
- **Platform**: Target platform(s) + fallback behavior
- **Data**: On-device vs server processing; retention policy (if camera/image/audio)

## Scope

**In Scope:**
- What we're building

**Out of Scope:**
- What we're NOT building

## Examples

| Input | Expected |
|-------|----------|
| Happy path input | Concrete result |
| Edge case input | Error or fallback |

## Scenarios

```gherkin
Scenario: Happy path
  Given [concrete precondition]
  When [specific action]
  Then [explicit result]

Scenario: Error case
  Given [error condition]
  When [trigger action]
  Then [expected error handling]

# For multiple similar cases, use Scenario Outline:
Scenario Outline: [name]
  Given [precondition]
  When [action with <variable>]
  Then [result with <expected>]

  Examples:
    | variable | expected |
    | value1   | result1  |
    | value2   | result2  |
```

## Done When

- [ ] **Function**: [Core behavior criterion]
- [ ] **Error**: [Error handling criterion]
- [ ] **UX**: [User-visible state/feedback criterion]
- [ ] **Quality**: Build passes
- [ ] **Quality**: Tests pass
````

---

## EXAMPLE

**Requirements:** "Users need task priority - high, medium, low. High shows red badge, medium yellow, low gray. Filter by priority. Default to medium."

**Output:**

> **Save as:** `specs/task-priority-v1.md`

````markdown
---
id: task-priority-v1
title: "Add Task Priority"
passes: false
priority: high
risk: standard
created: 2026-01-14
depends_on: []
---

# Add Task Priority

> **One-liner:** Allow users to set high/medium/low priority on tasks.

## Context

- **User**: Task owner
- **Trigger**: Creating or editing a task
- **Success**: Priority persists and displays correctly
- **Platform**: Web app (React), desktop and mobile browsers

## Scope

**In Scope:**
- Priority dropdown (high/medium/low)
- Colored badges (red/yellow/gray)
- Filter by priority

**Out of Scope:**
- Priority notifications
- Auto-priority based on due date

## Examples

| Input | Expected |
|-------|----------|
| Set priority "high" | Red badge, task saved |
| Filter by "high" | Only high-priority tasks shown |
| No priority selected | Defaults to medium |
| API receives "urgent" | 400 error "Invalid priority" |

## Scenarios

```gherkin
Scenario: Set priority on new task
  Given I am creating a new task
  When I select "high" priority from the dropdown
  Then the task saves with priority "high"
  And a red badge is displayed

Scenario: Filter by priority
  Given I have tasks with mixed priorities
  When I select "High" in the filter dropdown
  Then only high-priority tasks are shown

Scenario: Invalid priority rejected
  Given I am submitting a task via API
  When I send priority value "urgent"
  Then I receive a 400 error
  And the message is "Invalid priority"
```

## Done When

- [ ] **Function**: Priority dropdown shows high/medium/low options
- [ ] **Function**: Filter dropdown filters tasks correctly
- [ ] **Error**: Invalid priority values return 400 error
- [ ] **UX**: Colored badges visible on task cards (red/yellow/gray)
- [ ] **UX**: Default priority is "medium" when not specified
- [ ] **Quality**: Build passes
- [ ] **Quality**: Tests pass
````

---

## RULES

| Category | Rule |
| -------- | ---- |
| Format | `passes` always `false`; ID: kebab-case-v1; Date: YYYY-MM-DD |
| Format | Start with `> **Save as:** specs/[id].md` |
| Sizing | Split if 2+ systems (UI+API+ML), UI+backend+data/model changes, or 6+ Done When items; ~30 min per spec |
| Priority | `high` (blocking) \| `medium` (normal) \| `low` (nice-to-have) |
| Risk | `spike` (unknown) \| `integration` (external API/ML/OCR) \| `standard` (normal) \| `polish` (refinement) |
| Risk | If OCR/ML/external API → use `risk: integration` (not spike) |
| Risk | If `integration`, `depends_on` must list each external service/model explicitly |
| Scope | Out of Scope: minimum 2 items |
| Platform | Context must include target platforms (e.g., iOS Safari, Android Chrome) + fallback |
| Data | For camera/image/audio: specify on-device vs server processing + retention policy |
| Examples | 2+ rows including edge case with explicit input/output |
| Scenarios | Given/When/Then; include happy path AND error case |
| Done When | Each item prefixed: **Function**, **Error**, **UX**, or **Quality** |
| Clarify | If platform, data handling, or external services unknown → ask before generating |

---

## CHECKLIST (verify before outputting)

Before generating, confirm:

- [ ] Platform specified in Context (e.g., iOS Safari, Android Chrome, mobile web)
- [ ] Data handling specified if camera/image/audio involved
- [ ] Risk is `integration` if depends_on has external services/ML
- [ ] Done When items ≤ 6 (split spec if more)
- [ ] Every Done When item prefixed: **Function**, **Error**, **UX**, or **Quality**
- [ ] Out of Scope has 2+ items
- [ ] Examples table has 2+ rows including edge case

If any fail, fix before outputting or ask for clarification.

---

Now generate my spec from our conversation.
