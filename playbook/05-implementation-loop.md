# The Implementation Loop

## Principle: Iteration Beats Perfection

A bash loop with fresh context per iteration. Each iteration has three phases: implement, simplify, review. Three agents, three jobs, independent context.

```
implement → simplify → review
make it work   make it clean   verify it's correct
```

---

## PROMPT.md (implementing agent)

```markdown
You are implementing a feature.

The prd.json path and feature context will be provided below this prompt by loop.sh.

## Your Task
1. Read the prd.json at the path given below
2. Read progress.txt (same folder as prd.json) for context from previous iterations — if it exists
3. Pick the first user story where passes is false
4. Read the spec file (path in prd.json) for this story's acceptance criteria, AND read every file in docs/*.md for project conventions. You must follow these conventions in your implementation.
5. **Fast-path check (max 2 minutes):** Before deep exploration, do a quick grep for each AC's key signal — the function name, error message, route path, or UI element it describes. If every AC has matching code, append "Fast-path: all ACs appear implemented — deferring to review" to progress.txt and exit immediately. No commits, no tests, no traceability work. It's OK to be wrong here — the review agent will REJECT if something is actually missing, and the next iteration will fix it.
6. If not already implemented: implement ONLY that user story
7. Run typecheck, linter, and tests
7. If failing: fix and retry (max 3 attempts)
8. If passing: git commit, append progress to progress.txt (same folder as prd.json)
9. Do NOT set passes to true — the review step handles that
10. Exit

## Rules
- Work on ONE user story only. Do not look ahead.
- Every commit must pass typecheck, linter, and tests.
- Keep commits small and descriptive (conventional commits). **Commit messages must be fully lowercase** — no uppercase letters in subject or scope, including abbreviations (write `url` not `URL`, `api` not `API`). Scopes must be kebab-case (write `feat(us-003):` not `feat(US-003):`).
- **Never leave uncommitted changes.** Before exiting, run `git status`. If there are modified or staged files, either commit them or revert them. Leaving uncommitted work causes the review agent to reject the story and wastes a full iteration.
- **Do NOT simplify or refactor.** A separate simplify agent runs after you. Just make it work, commit, and exit. **Skip the pre-commit code-simplifier** mentioned in CLAUDE.md — the ADP loop has a dedicated SIMPLIFY phase that runs after your commit.
- Print a short status line before each major step (e.g. "Reading prd.json...", "Implementing US-003: add login form", "Running tests...", "Committing...").
- **Use tracer bullets.** Build the thinnest possible end-to-end slice first — one path through all layers (e.g. backend endpoint → contract → frontend hook → UI). Get it working, then fill in the remaining cases. This surfaces integration issues early and keeps each commit shippable.
```

---

## simplify.md (simplify agent)

```markdown
You are simplifying code that was just implemented for a user story.

The prd.json path and feature context will be provided below this prompt by loop.sh.

## Your Task
1. Read the prd.json at the path given below to identify the current user story (first where passes is false)
2. Read every file in docs/*.md for project conventions
3. Read the git diff of the last commit(s) from this iteration
4. Simplify the code without changing behavior — ensure the result follows docs/*.md conventions:
   - Remove dead code and unused imports
   - Extract duplicated logic
   - Simplify conditionals and reduce nesting
   - Improve naming where intent is unclear
   - Remove unnecessary abstractions
5. Run typecheck, linter, and tests — nothing may break
6. If you made changes: git commit — use the format `refactor(US-NNN): simplify <short title>` where US-NNN is the story ID and `<short title>` is a brief summary of the user story (e.g. `refactor(US-004): simplify add login form`)
7. If nothing to simplify: exit without committing
8. Exit

## Rules
- Do NOT change behavior. Only restructure and clean up.
- Do NOT add features, fix bugs, or address other stories.
- If tests fail after your changes, revert and exit.
- Print a short status line before each major step (e.g. "Reading diff...", "Simplifying US-003...", "Running tests...", "Nothing to simplify — skipping.").
```

---

## review.md (review agent)

```markdown
You are reviewing a completed user story.

The prd.json path and feature context will be provided below this prompt by loop.sh.

## Your Task
1. Read the prd.json at the path given below
2. Identify which user story was just implemented (the first where passes is false)
3. Read the acceptance criteria for that story AND read every file in docs/*.md for project conventions
4. Review the git diff (all commits from this iteration) against each acceptance criterion and each relevant convention
5. Decide: APPROVE or REJECT

## If APPROVE
- Set passes to true in prd.json
- Append "APPROVED: US-NNN — [brief summary]" to progress.txt (same folder as prd.json)

## If REJECT
- Do NOT change passes
- Append "REJECTED: US-NNN — [specific reasons and what needs to change]" to progress.txt (same folder as prd.json)
- The next implementation iteration will read this feedback

## Rules
- **Do NOT run tests, typecheck, or lint.** The implement agent already verified these. Your only job is to read the diff and check it against the acceptance criteria. Running builds wastes 5–10 minutes.
- **If there is no git diff** (implement agent fast-pathed or left no commits), check working tree changes with `git status`. If there are uncommitted changes relevant to this story, REJECT with "uncommitted changes — implement agent must commit before review."
- Print a short status line before each major step (e.g. "Reading AC for US-003...", "Checking criterion: user can log in...", "APPROVED: US-003", "REJECTED: US-003 — missing error handling").

## Review Checklist
- Does the diff satisfy every acceptance criterion? (check each AC individually)
- Does the code follow conventions in docs/*.md?
- Is the implementation minimal and focused (no scope creep)?
```

---

## adp-stream.sh (output filter)

Standalone stream filter. Reads NDJSON from `claude --output-format stream-json` on stdin, prints structured progress with timestamps and relative paths.

By default shows agent thoughts (💬), results (✅/❌), and reads from `specs/`, `docs/`, or `.adp/` only. Pass `--verbose` as the second argument to also show all tool calls (📖 ✏️ ⚡ 🔧).

```bash
#!/bin/bash
# Usage: claude -p --output-format stream-json --verbose | .adp/adp-stream.sh [project-root] [--verbose]

ROOT="${1:-$(pwd)/}"
VERBOSE="${2:-}"
LABEL="${3:-}"
CURRENT_US="${4:-}"

jq -r --unbuffered --arg root "$ROOT" --arg verbose "$VERBOSE" --arg label "$LABEL" --arg current_us "$CURRENT_US" '
  def strip_root: if startswith($root) then .[$root | length:] else . end;
  def is_context_path: startswith("specs/") or startswith("docs/") or startswith(".adp/");
  if .type == "assistant" then
    [.message.content[]? |
      if .type == "tool_use" then
        if $verbose == "--verbose" then
          if .name == "Read" then "📖 " + ((.input.file_path // "?") | strip_root)
          elif .name == "Write" then "✏️  " + ((.input.file_path // "?") | strip_root)
          elif .name == "Edit" then "✏️  " + ((.input.file_path // "?") | strip_root)
          elif .name == "Bash" then "⚡ " + (.input.command // "?" | split("\n")[0] | if length > 80 then .[:80] + "..." else . end)
          else "🔧 " + .name
          end
        elif .name == "Read" then
          ((.input.file_path // "?") | strip_root) as $path |
          if ($path | is_context_path) then "📖 " + $path else empty end
        else empty
        end
      elif .type == "text" then
        .text | gsub("^\\s+$"; "") | if . != "" then split("\n") | map(select(. != "")) | to_entries | map(if .key == 0 then "💬 " + .value else "      " + .value end) | join("\n") else empty end
      else empty
      end
    ] | map(select(. != "")) | join("\n") | if . != "" then . else empty end
  elif .type == "result" then
    if .subtype == "success" then
      "✅ Done with " + $label + (if $current_us != "" then " · " + $current_us else "" end) + " (" + (.duration_ms / 1000 | tostring | split(".")[0]) + "s)"
    else "❌ Error: " + (.result // "unknown")
    end
  else empty
  end
' 2>/dev/null | while IFS= read -r line; do
  echo "   $(date +%H:%M:%S) $line"
done || true
```

---

## loop.sh (the loop)

```bash
#!/bin/bash
set -euo pipefail

FEATURE="${1:?Usage: .adp/loop.sh <feature-name> [max-iterations] [--verbose]}"
MAX_ITERATIONS="${2:-30}"
VERBOSE="${3:-}"
PRD=".adp/artifacts/$FEATURE/prd.json"

if [ ! -f "$PRD" ]; then
  echo "Error: $PRD not found."
  echo "Run /adp:plan first to generate the prd.json."
  exit 1
fi

CONTEXT="

---
Feature: $FEATURE
PRD path: $PRD
Progress path: .adp/artifacts/$FEATURE/progress.txt
---"

PROJECT_ROOT="$(pwd)/"

IMPLEMENT_MODEL="${IMPLEMENT_MODEL:-}"
REVIEW_MODEL="${REVIEW_MODEL:-haiku}"

run_phase() {
  local prompt="$1"
  local label="$2"
  local model="${3:-}"
  local model_flag=""
  if [ -n "$model" ]; then model_flag="--model $model"; fi
  { cat "$prompt"; echo "$CONTEXT"; } | claude -p --dangerously-skip-permissions $model_flag --output-format stream-json --verbose | .adp/adp-stream.sh "$PROJECT_ROOT" "$VERBOSE" "$label" "$CURRENT_US"
}

for i in $(seq 1 "$MAX_ITERATIONS"); do
  CURRENT_US=$(jq -r '[.userStories[] | select(.passes == false)] | first | "\(.id): \(.title)"' "$PRD" 2>/dev/null || echo "")
  echo ""
  echo "── ADP iteration $i/$MAX_ITERATIONS ─────────────────────────────"
  echo ""

  BEFORE_SHA=$(git rev-parse HEAD 2>/dev/null || echo "none")

  echo "🔨 IMPLEMENT · $CURRENT_US"
  run_phase .adp/PROMPT.md "IMPLEMENT" "$IMPLEMENT_MODEL"

  AFTER_SHA=$(git rev-parse HEAD 2>/dev/null || echo "none")

  if [ "$BEFORE_SHA" != "$AFTER_SHA" ]; then
    echo ""
    echo "🧹 SIMPLIFY · $CURRENT_US"
    run_phase .adp/simplify.md "SIMPLIFY" "$REVIEW_MODEL"
  else
    echo ""
    echo "🧹 SIMPLIFY · $CURRENT_US — skipped (no new commits)"
  fi

  echo ""
  echo "🔍 REVIEW · $CURRENT_US"
  run_phase .adp/review.md "REVIEW" "$REVIEW_MODEL"
  echo ""
  echo "────────────────────────────────────────────────────────────────"

  if jq -e '[.userStories[] | select(.passes == false)] | length == 0' "$PRD" > /dev/null 2>&1; then
    echo ""
    echo "🎉 All user stories complete!"
    exit 0
  fi
done

echo ""
echo "⚠️  Reached max iterations ($MAX_ITERATIONS). Some stories still incomplete."
exit 1
```

---

## Feedback Loops (Backpressure)

The agent can't cheat. These gates reject bad work automatically:

- **TypeScript compiler** — types don't lie
- **Test suite** — behavior verified mechanically
- **Linter** — style enforced deterministically
- **Simplify agent** — cleans up implementation debt before review
- **Review agent** — independent AC verification per story
