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
3. Pick the first user story where `passes` is false AND `blocked` is not true. This is "the current story". Do NOT advance to a later story under any circumstance — even if progress.txt says the current story is done. The review agent owns advancement.
4. Read the spec file (path in prd.json) for this story's acceptance criteria.
5. **Read project conventions.** Read every file in `docs/**/*.md` for project conventions. You must follow these conventions in your implementation.
6. **Fast-path check (max 2 minutes).** Before writing any code, grep for each AC's key signal (function name, route path, UI text, error message). Two cases:
   - **Every AC matches existing code.** The story is ready for review. Append "Fast-path: <US-NNN> already implemented — deferring to review" to progress.txt and exit 0. Do not write code. Do not run tests. Do not pick another story.
   - **Some ACs have no matching code.** Proceed with implementation of THIS story only (steps 7-12).
   It's OK to be wrong here — REVIEW will REJECT if something is actually missing, and the next iteration will fix it.
7. If not already implemented: implement ONLY that user story
8. Write or update tests that verify each acceptance criterion for this story. Every AC must have a corresponding test — if a test already exists and covers the criterion, leave it. If not, add one.
9. Run typecheck, linter, and tests
10. If failing: fix and retry (max 3 attempts)
11. If passing: git commit, append progress to progress.txt (same folder as prd.json)
12. Do NOT set passes to true — the review step handles that
13. Exit

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
1. Read the prd.json at the path given below to identify the current user story (first where passes is false AND blocked is not true).
2. Read the git diff of the last commit(s) from this iteration, **excluding lockfiles and generated files**:
   `git diff HEAD~1..HEAD -- ':!**/pnpm-lock.yaml' ':!**/package-lock.json' ':!**/yarn.lock' ':!**/dist/**' ':!**/.astro/**'`
3. Identify which convention files under `docs/conventions/` are clearly relevant to what the diff touches (typically 1-3 files, not all of them). Read only those. If unsure, skip this step — IMPLEMENT already applied conventions.
4. If nothing to simplify: exit immediately without committing. Do not run typecheck, linter, or tests.
5. If there are simplifications, apply them:
   - Remove dead code and unused imports
   - Extract duplicated logic
   - Simplify conditionals and reduce nesting
   - Improve naming where intent is unclear
   - Remove unnecessary abstractions
6. Run typecheck, linter, and tests — nothing may break.
7. If passing: git commit — use the format `refactor(us-NNN): simplify <short title>`.
8. **Only if step 7 produced a new commit, append a pair entry to `.adp/preferences/simplifications.jsonl`.** Captures the before/after pair as DPO-style preference data for later convention distillation. Log-append failure is non-fatal — the pair is recoverable from the `refactor(us-*)` commit in git history.
   **Substitutions required before running:** `<feature-slug>`, `<US-NNN>`, `<one-line what changed and why>`, `<docs/conventions/<file>.md or empty>`.
   ```bash
   # Guard: only run if step 7 committed (HEAD~1 must exist and differ from HEAD).
   if git rev-parse HEAD~1 >/dev/null 2>&1 && [ "$(git rev-parse HEAD~1)" != "$(git rev-parse HEAD)" ]; then
     mkdir -p .adp/preferences
     BEFORE=$(git rev-parse HEAD~1)
     AFTER=$(git rev-parse HEAD)
     jq -nc --arg feature "<feature-slug>" --arg story "<US-NNN>" \
            --arg before "$BEFORE" --arg after "$AFTER" \
            --arg rationale "<one-line what changed and why>" \
            --arg convention "<docs/conventions/<file>.md or empty>" \
       '{ts: now | todateiso8601, feature: $feature, story: $story, before_sha: $before, after_sha: $after, rationale: $rationale, convention: $convention}' \
       >> .adp/preferences/simplifications.jsonl || true
   fi
   ```
9. Exit.

## Rules
- Do NOT change behavior. Only restructure and clean up.
- Do NOT add features, fix bugs, or address other stories.
- **Before removing any code, check it against the story's acceptance criteria in prd.json.** Code that directly implements an AC must not be removed — only restructured.
- **Do NOT run typecheck, linter, or tests before making changes.** The implement agent already verified these. Only run them after your simplifications to confirm nothing broke.
- If tests fail after your changes, revert and exit 0 (non-fatal).
- **Never read `pnpm-lock.yaml`, `package-lock.json`, `yarn.lock`, `dist/`, or `.astro/`** — they will overflow your context.
- If your context is filling up, stop reading files and exit 0. SIMPLIFY is optional polish; the loop will proceed.
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
3. Read the acceptance criteria for that story. Identify which convention files under `docs/conventions/` and `specs/conventions/` are directly relevant to the story's ACs (typically 1-3 files). Read only those.
4. Read the git diff of all commits from this iteration, **excluding lockfiles and generated files**:
   `git diff <iteration-base>..HEAD -- ':!**/pnpm-lock.yaml' ':!**/package-lock.json' ':!**/yarn.lock' ':!**/dist/**' ':!**/.astro/**'`
   If you can't determine the iteration base, use `git show HEAD -- ':!**/*.lock' ':!**/dist/**' ':!**/.astro/**'` for the latest commit only.
5. **Pull few-shot examples from the preference log.** Retrieve up to 3 past rejections whose `tags` overlap with paths in the current diff. Treat matched entries as calibration: apply the same standards they reflect to this review.
   **Substitutions required before running:** `<tag1>`,`<tag2>` — replace with 1-3 tags from the canonical vocabulary in `playbook/11-preference-accumulation.md`. Do not run the block while any `<...>` placeholder remains.
   ```bash
   [ -s .adp/preferences/rejections.jsonl ] && {
     tail -n 500 .adp/preferences/rejections.jsonl | \
       jq -c --argjson tags '["<tag1>","<tag2>"]' \
         'select(.tags | any(. as $t | $tags | index($t)))' | \
       head -3 || true
   }
   ```
6. Review the diff against each acceptance criterion, each relevant convention, and the retrieved few-shot examples.
7. Decide: APPROVE or REJECT

## If APPROVE
- Set passes to true in prd.json
- Append "APPROVED: US-NNN — [brief summary]" to progress.txt (same folder as prd.json)

## If REJECT
- Do NOT change passes
- Append "REJECTED: US-NNN — [specific reasons and what needs to change]" to progress.txt (same folder as prd.json)
- **Append a structured entry to `.adp/preferences/rejections.jsonl`** so the rejection survives past this feature.
  **Substitutions required before running** (all bracketed values must be replaced; do not run with any `<...>` remaining): `<feature-slug>` (from the context block), `<US-NNN>` and `<AC-NNN>` (from prd.json), `<short reason>`, `<tag1>`/`<tag2>` (from the canonical vocabulary in `playbook/11-preference-accumulation.md`).
  ```bash
  mkdir -p .adp/preferences
  jq -nc --arg feature "<feature-slug>" --arg story "<US-NNN>" --arg reason "<short reason>" \
         --arg ac "<AC-NNN>" --arg diff_sha "$(git rev-parse HEAD)" \
         --argjson tags '["<tag1>","<tag2>"]' \
    '{ts: now | todateiso8601, feature: $feature, story: $story, reason: $reason, ac: $ac, diff_sha: $diff_sha, tags: $tags}' \
    >> .adp/preferences/rejections.jsonl
  ```
- The next implementation iteration will read this feedback

## Rules
- **Do NOT run tests, typecheck, or lint.** The implement agent already verified these. Your only job is to read the diff and check it against the acceptance criteria. Running builds wastes 5–10 minutes.
- **Never read `pnpm-lock.yaml`, `package-lock.json`, `yarn.lock`, `dist/`, or `.astro/`.** They will overflow your context without helping you review.
- **If there is no git diff** (implement agent fast-pathed or left no commits), check working tree changes with `git status`. If there are uncommitted changes relevant to this story, REJECT with "uncommitted changes — implement agent must commit before review."
- Print a short status line before each major step (e.g. "Reading AC for US-003...", "Checking criterion: user can log in...", "APPROVED: US-003", "REJECTED: US-003 — missing error handling").

## Review Checklist
- Does the diff satisfy every acceptance criterion? (check each AC individually)
- Does the code follow conventions in `docs/**/*.md`?
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
REVIEW_MODEL="${REVIEW_MODEL:-sonnet}"

# Suppress telemetry/consent prompts so the loop runs unattended
export DISABLE_TELEMETRY=1
export CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1

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
    run_phase .adp/simplify.md "SIMPLIFY" "$REVIEW_MODEL" || echo "   simplify phase errored — continuing to review"
  else
    echo ""
    echo "🧹 SIMPLIFY · $CURRENT_US — skipped (no new commits)"
  fi

  CURRENT_US=$(jq -r '[.userStories[] | select(.passes == false and (.blocked // false) == false)] | first | "\(.id): \(.title)"' "$PRD" 2>/dev/null || echo "")
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
