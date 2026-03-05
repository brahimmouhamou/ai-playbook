# TDD Integration

## Principle: Context Isolation Per Phase

When the same agent writes tests and implementation, it "cheats." Separate subagents per TDD phase prevent context pollution.

---

## The Cycle (per task)

```
RED    → Test subagent writes failing tests (no implementation knowledge)
GREEN  → Implement subagent writes minimal code to pass (fresh context)
REFACTOR → Simplify subagent cleans up (runs /simplify)
```

---

## TDD Skill (`.claude/skills/tdd-cycle/SKILL.md`)

```markdown
---
name: tdd-cycle
description: Enforce Red-Green-Refactor using isolated subagents.
  Triggers on "implement", "add feature", "build".
---
## Process
1. Spawn RED subagent: write failing tests for the acceptance criteria.
   Stop after tests are written. Do not implement.
2. Spawn GREEN subagent: read test files, write minimal code to pass.
   Do not modify tests.
3. Run tests. If failing, iterate GREEN until passing.
4. Spawn REFACTOR subagent: review implementation + tests against
   refactoring checklist. Apply improvements. Verify tests still pass.
5. Commit with conventional commit message.
```
