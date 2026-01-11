---
id: task-priority-v1
title: "Add Task Priority"
passes: false
priority: high
risk: standard
created: 2026-01-11
depends_on: []
---

# Add Task Priority

> **One-liner:** Allow users to set high/medium/low priority on tasks.

## Context

| | |
|---|---|
| **User** | Task owner |
| **Trigger** | Creating or editing a task |
| **Success** | Priority persists and displays correctly |

## Scope

**In Scope:**

- Priority field with high/medium/low options
- Colored badge display (red/yellow/gray)
- Priority filter on task list

**Out of Scope:**

- Priority-based notifications
- Auto-priority based on due date
- Priority inheritance from parent tasks

## Dependencies

- [ ] Tasks table exists in database
- [ ] Task form component exists

## Examples

### Happy Path

| Input | Expected |
|-------|----------|
| Set priority to "high" | Red badge appears, task saved |
| Set priority to "low" | Gray badge appears, task saved |
| Filter by "high" | Only high-priority tasks shown |

### Edge Cases

| Input | Condition | Expected |
|-------|-----------|----------|
| No priority selected | Default behavior | Medium priority assigned |
| Change from high to low | Existing task | Badge color updates immediately |

### Error Responses

| Trigger | Code | Response |
|---------|------|----------|
| Invalid priority value | 400 | `{error: "Invalid priority"}` |
| Task not found | 404 | `{error: "Task not found"}` |

## Scenarios

### Set priority on new task

Given I am creating a new task
When I select "high" priority
Then the task is saved with priority "high"
And a red badge is displayed

### Filter by priority

Given I have tasks with mixed priorities
When I select "High" in the filter dropdown
Then only high-priority tasks are shown
And the URL updates with `?priority=high`

### Scenario Outline: Priority badge colors

Given I have a task with priority "<priority>"
When I view the task card
Then I see a "<color>" badge

Examples:
  | priority | color  |
  | high     | red    |
  | medium   | yellow |
  | low      | gray   |

### Invalid priority rejected

Given I am editing a task via API
When I submit with priority "urgent"
Then I receive a 400 response
And the error message is "Invalid priority"

## Files

```
src/
├── components/
│   └── tasks/
│       ├── task-card.tsx        # Add priority badge
│       └── task-form.tsx        # Add priority dropdown
├── actions/
│   └── tasks.ts                 # Update save logic
├── db/
│   └── schema.ts                # Add priority column
└── tests/
    └── tasks.test.ts            # Priority tests
```

## Done When

- [ ] Priority dropdown in task form (high/medium/low)
- [ ] Colored badge on task cards (red=high, yellow=medium, gray=low)
- [ ] Filter dropdown filters tasks by priority
- [ ] Default priority is "medium" when not specified
- [ ] Invalid priority values return 400 error
- [ ] Build passes
- [ ] Tests pass with new priority tests
