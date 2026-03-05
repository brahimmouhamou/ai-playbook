# Code Review

## Principle: Review at Boundaries, Not Inline

Code review happens at specific checkpoints, not as a peer agent in an orchestrator.

---

## Three Review Points

### 1. Stop Hook Auto-Review (every implementation pass)
- Fires when Claude finishes working
- Spawns review subagent with fresh context
- Reviews only modified files
- Catches obvious issues before you look

### 2. /simplify (on-demand after feature completion)
- Spawns 3 parallel agents: code reuse, quality, efficiency
- Aggregates findings and applies fixes
- Run after implementing a feature, before PR

### 3. /code-review (PR-time)
- 5 independent reviewers in parallel
- Confidence scoring (0-100), only posts issues ≥80
- Reduces noise dramatically
