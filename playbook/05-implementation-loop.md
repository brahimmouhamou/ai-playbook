# The Implementation Loop

## Principle: Iteration Beats Perfection

A bash loop with fresh context per iteration. Each iteration has two phases: implement and review. The implementing agent picks one user story, builds it, and commits. The review agent checks the diff against the acceptance criteria and either approves or rejects. Different agents, independent judgment.

---

## PROMPT.md (implementing agent)

```markdown
You are implementing a feature.

## Your Task
1. Read progress.txt (next to prd.json) for context from previous iterations
2. Find the prd.json in .adp/artifacts/ for the current feature
3. Pick the first user story where passes is false
4. Implement ONLY that user story
5. Run typecheck, linter, and tests
6. If failing: fix and retry (max 3 attempts)
7. If passing: git commit, append progress to the feature's progress.txt
8. Do NOT set passes to true — the review step handles that
9. Exit

## Rules
- Work on ONE user story only. Do not look ahead.
- Every commit must pass typecheck, linter, and tests.
- Read docs/*.md for conventions when touching relevant code.
- Keep commits small and descriptive (conventional commits).
```

---

## review.md (review agent)

```markdown
You are reviewing a completed user story.

## Your Task
1. Find the prd.json in .adp/artifacts/ for the current feature
2. Identify which user story was just implemented (the first where passes is false)
3. Read the acceptance criteria for that story
4. Review the git diff (last commit vs previous) against each acceptance criterion
5. Decide: APPROVE or REJECT

## If APPROVE
- Set passes to true in prd.json
- Append "APPROVED: US-NNN — [brief summary]" to progress.txt

## If REJECT
- Do NOT change passes
- Append "REJECTED: US-NNN — [specific reasons and what needs to change]" to progress.txt
- The next implementation iteration will read this feedback

## Review Checklist
- Does the diff satisfy every acceptance criterion? (check each AC individually)
- Are there regressions in existing functionality?
- Does the code follow conventions in docs/*.md?
- Is the implementation minimal and focused (no scope creep)?
```

---

## The Loop Script

```bash
#!/bin/bash
set -euo pipefail

FEATURE="${1:?Usage: .adp/loop.sh <feature-name> [max-iterations]}"
MAX_ITERATIONS="${2:-30}"
PRD=".adp/artifacts/$FEATURE/prd.json"

if [ ! -f "$PRD" ]; then
  echo "Error: $PRD not found."
  echo "Run /adp:plan first to generate the prd.json."
  exit 1
fi

for i in $(seq 1 "$MAX_ITERATIONS"); do
  echo "=== ADP iteration $i/$MAX_ITERATIONS — implement ==="
  cat .adp/PROMPT.md | claude -p

  echo "=== ADP iteration $i/$MAX_ITERATIONS — review ==="
  cat .adp/review.md | claude -p

  if jq -e '[.userStories[] | select(.passes == false)] | length == 0' "$PRD" > /dev/null 2>&1; then
    echo "All user stories complete!"
    exit 0
  fi
done

echo "Reached max iterations ($MAX_ITERATIONS). Some stories still incomplete."
exit 1
```

---

## Review Layers

The per-story review in the loop is the first layer. After all stories pass:

- **`/simplify`** — post-feature review with parallel agents (simplify, security, performance). Runs once before opening a PR.
- **`/code-review`** — PR-time review with parallel reviewers and confidence scoring. Runs once on the full diff.

---

## Feedback Loops (Backpressure)

The agent can't cheat. These gates reject bad work automatically:

- **TypeScript compiler** — types don't lie
- **Test suite** — behavior verified mechanically
- **Linter** — style enforced deterministically
- **Review agent** — independent AC verification per story
