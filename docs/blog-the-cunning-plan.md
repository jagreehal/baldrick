# The Cunning Plan for AI Coding That Actually Works

*How to build features autonomously without context rot*

---

## Why Your AI Keeps Forgetting What You Asked For

You're 45 minutes into building a feature with Claude or ChatGPT. The code is taking shape. Then you ask for one more change and suddenly:

*"I don't see any code in our conversation. Could you share what you're working with?"*

Your AI just forgot everything.

This is **context rot** — the slow decay of understanding that happens when AI conversations get too long. The model's attention spreads thin across thousands of tokens, and your carefully built context crumbles.

If you've ever felt like you're babysitting an AI that keeps losing the plot, you're not imagining it. And if you've ever wished you could just hand off a project and have it *done* when you come back, keep reading.

---

## The Problem Isn't the AI. It's How We Use It.

Here's what most people do:

1. Open Claude/ChatGPT
2. Describe the entire project
3. Ask for feature after feature
4. Watch quality degrade as context grows
5. Eventually start over

This is like asking someone to memorize an entire book while simultaneously writing new chapters. Eventually, something gives.

**The insight**: What if instead of one long conversation, you had many short ones — each laser-focused on a single task, with a system that tracks progress between them?

---

## Enter Baldrick: "I Have a Cunning Plan"

Baldrick is a bash script that runs Claude Code in a loop. Each iteration:

- Starts with **fresh context** (no memory of previous conversations)
- Receives **exactly one spec** to implement
- Has access to **shared files** that carry knowledge forward
- **Commits its work** to git before finishing

The key innovation: **Claude doesn't remember. The filesystem does.**

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Iteration 1 │────▶│ Iteration 2 │────▶│ Iteration 3 │
│ Spec: Login │     │ Spec: Auth  │     │ Spec: API   │
│ (fresh)     │     │ (fresh)     │     │ (fresh)     │
└─────────────┘     └─────────────┘     └─────────────┘
       │                  │                   │
       ▼                  ▼                   ▼
   ┌──────────────────────────────────────────────┐
   │          Git Repository (persists)           │
   │          progress.txt (learnings)            │
   │          specs/*.md (requirements)           │
   └──────────────────────────────────────────────┘
```

Each Claude session is like a fresh employee who:
- Gets a clear task description (the spec)
- Can read what previous employees learned (progress.txt)
- Does their work and documents what they discovered
- Clocks out, and the next employee picks up seamlessly

---

## A Real Test: Building a Calculator in 4 Autonomous Iterations

I wanted to prove this actually works. So I created 4 specs:

| # | Spec | Priority | Depends On |
|---|------|----------|------------|
| 1 | Basic Operations (+, -) | High | None |
| 2 | Multiply & Divide | Medium | Spec 1 |
| 3 | Display Formatting | Medium | Spec 2 |
| 4 | Error Handling | Low | Spec 3 |

Then I ran:

```bash
baldrick run 4
```

And walked away.

### What Happened

**Iteration 1** (~80 seconds): Claude created `calculator.html` from scratch with add/subtract functionality.

**Iteration 2** (~80 seconds): A *completely fresh* Claude read the existing file, noticed multiply/divide were already implemented, verified they worked, and marked the spec done.

**Iteration 3** (~15 seconds): Fresh Claude added decimal support and display formatting.

**Iteration 4** (quick): Fresh Claude added division-by-zero handling and error recovery.

**Total time**: ~5 minutes of autonomous work.

**Result**: A fully functional calculator with all features implemented correctly.

---

## The Magic Moment

Here's the entry from `progress.txt` after Iteration 2:

```
## 2026-01-13 14:07 - Calculator Multiply and Divide
- What: Verified multiply and divide operations already implemented
- Learnings:
  - Previous spec implementation was forward-thinking, included all four basic operations
  - Spec validation: always read existing code before implementing
```

Notice: *"Previous spec implementation was forward-thinking"*

This Claude had **zero memory** of the previous Claude's conversation. It discovered this by reading the code fresh. The system worked exactly as designed — each iteration starts clean but benefits from accumulated work.

---

## Why This Matters

### For Developers

**No more context babysitting.** Write specs once, let the loop run. Come back to committed, working code.

**Specs become living documentation.** Each spec is a BDD-style document with:
- Clear scenarios in Gherkin format
- Explicit "Done When" criteria
- Dependencies that ensure proper ordering

**Git history stays clean.** Each spec completion is a focused commit, not a sprawling "implemented stuff" message.

### For Teams

**Onboard AI to your project, not vice versa.** The specs define what you want. The AI figures out how to deliver it within your existing codebase patterns.

**Progress is visible.** `baldrick status` shows exactly what's done, what's in progress, and what's blocked:

```
╔═════════════════════════════════════════════════════════════╗
║                     Baldrick Status                         ║
╠═════════════════════════════════════════════════════════════╣
║ ✓ │ high     │ standard    │ 01-calc-basic                  ║
║ ✓ │ medium   │ standard    │ 02-calc-multiply               ║
║ ⭕ │ medium   │ polish      │ 03-calc-display                ║
║ 🚫 │ low      │ polish      │ 04-calc-errors                 ║
╚═════════════════════════════════════════════════════════════╝

Progress: 2/4 (50%)
```

### For Non-Developers

Think of Baldrick as a **project manager for AI work**. You write down what you want (specs), and it keeps feeding tasks to the AI, one at a time, until everything is done.

The AI might "forget" between tasks, but Baldrick doesn't. It tracks:
- What's completed
- What's still pending
- What's blocked waiting on other work
- Lessons learned along the way

---

## How to Get Started

### 1. Install

```bash
curl -sO https://raw.githubusercontent.com/jagreehal/baldrick/main/baldrick
chmod +x baldrick
./baldrick init
```

### 2. Write Your First Spec

```yaml
---
id: my-feature-v1
title: "My Awesome Feature"
passes: false
priority: high
---

# My Awesome Feature

> **One-liner:** Does the thing users need

## Scenarios

### Scenario: Happy path
Given the system is ready
When user does the action
Then the expected result happens

## Done When
- [ ] Feature works as described
- [ ] Tests pass
- [ ] No regressions
```

### 3. Run the Loop

```bash
./baldrick run 10
```

Baldrick will:
1. Select the highest-priority spec with satisfied dependencies
2. Invoke a fresh Claude session with that spec
3. Claude implements, tests, and commits
4. Claude marks `passes: true` when done
5. Loop continues to the next spec

### 4. Check Status Anytime

```bash
./baldrick status
```

---

## What to Expect

### It's Not Magic

- **Some specs take multiple iterations.** Complex features might need a few passes.
- **Quality commands matter.** Configure your build/test/lint commands so Claude can verify its work.
- **Specs should be specific.** Vague specs produce vague results.

### It Is Remarkably Consistent

- **Fresh context = fresh thinking.** No accumulated confusion from earlier missteps.
- **Progress is durable.** Each iteration commits to git. Interrupted? Just resume.
- **Learnings compound.** The `progress.txt` file accumulates wisdom that future iterations benefit from.

---

## The Philosophy: Trust the System, Not the Session

Traditional AI coding is like a long phone call — you maintain context by staying on the line. Baldrick is like exchanging letters — each message is self-contained, but the relationship progresses through accumulated correspondence.

This isn't a limitation. It's a feature.

Because when you stop fighting context limits and start designing around them, you can build things that would be impossible in a single conversation:

- **Multi-day projects** that proceed without babysitting
- **Complex features** broken into digestible chunks
- **Team workflows** where AI work is visible and auditable

---

## Try It

Baldrick is a single bash script. No dependencies beyond Claude Code itself.

```bash
# Get started
./baldrick init
./baldrick doctor  # Verify setup

# Create specs in specs/ directory

# Run the loop
./baldrick run 10
```

The cunning plan isn't about making AI smarter. It's about making AI *work smarter* — one focused task at a time, with perfect handoffs between sessions.

*"I have a cunning plan, my lord."*

This time, it actually works.

---

## Resources

- [GitHub Repository](https://github.com/jagreehal/baldrick)
- [Full Documentation](../README.md)
- [Design Philosophy](../design.md)
- [Spec Templates](../templates/)

---

*Baldrick is named after the character from Blackadder who always had "a cunning plan." Unlike the TV character, this Baldrick's plans actually work.*
