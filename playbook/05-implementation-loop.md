# The Implementation Loop

## Principle: Iteration Beats Perfection

A bash loop with fresh context per iteration. The agent picks one user story, implements it, verifies it, commits, exits. Next iteration picks the next story with zero context rot.

---

## PROMPT.md (fed to agent every iteration)

```markdown
You are implementing a feature for NineID.

## Your Task
1. Read .adp/artifacts/<feature>/prd.json
2. Pick the first user story where passes is false
3. Implement ONLY that user story
4. Run typecheck, linter, and tests
5. If passing: set passes to true, git commit,
   append progress to .adp/artifacts/<feature>/progress.txt
6. If failing: fix and retry (max 3 attempts)
7. Exit

## Rules
- Work on ONE user story only. Do not look ahead.
- Every commit must pass typecheck, linter, and tests.
- Read docs/*.md for conventions when touching relevant code.
- Keep commits small and descriptive (conventional commits).
```

---

## The Loop Script

```bash
#!/bin/bash
FEATURE="019-worker-jobs-panel"
MAX_ITERATIONS=30
for i in $(seq 1 $MAX_ITERATIONS); do
  echo "=== ADP iteration $i ==="
  cat .adp/PROMPT.md | claude -p --model opus

  # Check if all user stories pass
  if jq -e '[.userStories[] | select(.passes == false)] | length == 0' .adp/artifacts/$FEATURE/prd.json > /dev/null 2>&1; then
    echo "All user stories complete!"
    break
  fi
done
```

---

## Feedback Loops (Backpressure)

The agent can't cheat. These gates reject bad work automatically:

- **TypeScript compiler** — types don't lie
- **Test suite** — behavior verified mechanically
- **Linter** — style enforced deterministically
- **Stop hook auto-review** — subagent catches what the implementer missed
