---
name: init
description: "This skill should be used when the user wants to 'initialize adp', 'set up adp', 'bootstrap the agentic playbook', or needs to create the .adp folder structure in a project."
---

# /init

Initialize the ADP workspace in a project. Creates the `.adp/` folder with operational files at the project root.

## Process

1. **Never delete existing files.** If `.adp/` already exists, that's fine — proceed and overwrite only PROMPT.md, simplify.md, review.md, and loop.sh. Never remove the `artifacts/` folder or anything inside it. Existing prd.json files, progress.txt files, and any other user content must be preserved.

2. **Create the folder structure** (use `mkdir -p`, safe to run if folders exist):
   ```
   .adp/
   ├── PROMPT.md
   ├── simplify.md
   ├── review.md
   ├── loop.sh
   └── artifacts/
   ```

3. **Create `.adp/PROMPT.md`**:
   ```markdown
   You are implementing a feature.

   The prd.json path and feature context will be provided below this prompt by loop.sh.

   ## Your Task
   1. Read the prd.json at the path given below
   2. Read progress.txt (same folder as prd.json) for context from previous iterations — if it exists
   3. Pick the first user story where passes is false
   4. Implement ONLY that user story
   5. Run typecheck, linter, and tests
   6. If failing: fix and retry (max 3 attempts)
   7. If passing: git commit, append progress to progress.txt (same folder as prd.json)
   8. Do NOT set passes to true — the review step handles that
   9. Exit

   ## Rules
   - Work on ONE user story only. Do not look ahead.
   - Every commit must pass typecheck, linter, and tests.
   - Read docs/*.md for conventions when touching relevant code.
   - Keep commits small and descriptive (conventional commits).
   - Print a short status line before each major step (e.g. "Reading prd.json...", "Implementing US-003: add login form", "Running tests...", "Committing...").
   ```

4. **Create `.adp/simplify.md`**:
   ```markdown
   You are simplifying code that was just implemented for a user story.

   The prd.json path and feature context will be provided below this prompt by loop.sh.

   ## Your Task
   1. Read the prd.json at the path given below to identify the current user story (first where passes is false)
   2. Read the git diff of the last commit(s) from this iteration
   3. Simplify the code without changing behavior:
      - Remove dead code and unused imports
      - Extract duplicated logic
      - Simplify conditionals and reduce nesting
      - Improve naming where intent is unclear
      - Remove unnecessary abstractions
   4. Run typecheck, linter, and tests — nothing may break
   5. If you made changes: git commit — use the format `refactor(US-NNN): simplify <short title>` where US-NNN is the story ID and `<short title>` is a brief summary of the user story (e.g. `refactor(US-004): simplify add login form`)
   6. If nothing to simplify: exit without committing
   7. Exit

   ## Rules
   - Do NOT change behavior. Only restructure and clean up.
   - Do NOT add features, fix bugs, or address other stories.
   - If tests fail after your changes, revert and exit.
   - Read docs/*.md for conventions to ensure consistency.
   - Print a short status line before each major step (e.g. "Reading diff...", "Simplifying US-003...", "Running tests...", "Nothing to simplify — skipping.").
   ```

5. **Create `.adp/review.md`**:
   ```markdown
   You are reviewing a completed user story.

   The prd.json path and feature context will be provided below this prompt by loop.sh.

   ## Your Task
   1. Read the prd.json at the path given below
   2. Identify which user story was just implemented (the first where passes is false)
   3. Read the acceptance criteria for that story
   4. Review the git diff (all commits from this iteration) against each acceptance criterion
   5. Decide: APPROVE or REJECT

   ## If APPROVE
   - Set passes to true in prd.json
   - Append "APPROVED: US-NNN — [brief summary]" to progress.txt (same folder as prd.json)

   ## If REJECT
   - Do NOT change passes
   - Append "REJECTED: US-NNN — [specific reasons and what needs to change]" to progress.txt (same folder as prd.json)
   - The next implementation iteration will read this feedback

   ## Rules
   - Print a short status line before each major step (e.g. "Reading AC for US-003...", "Checking criterion: user can log in...", "APPROVED: US-003", "REJECTED: US-003 — missing error handling").

   ## Review Checklist
   - Does the diff satisfy every acceptance criterion? (check each AC individually)
   - Are there regressions in existing functionality?
   - Does the code follow conventions in docs/*.md?
   - Is the implementation minimal and focused (no scope creep)?
   ```

6. **Create `.adp/loop.sh`**:
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

   CONTEXT="

   ---
   Feature: $FEATURE
   PRD path: $PRD
   Progress path: .adp/artifacts/$FEATURE/progress.txt
   ---"

   # Stream filter: reads NDJSON from claude, prints structured progress
   # Event types: system (skip), assistant (text + tool_use in .message.content[]),
   #              tool_result (skip), result (final status in .subtype)
   adp_stream() {
     jq -r --unbuffered '
       if .type == "assistant" then
         [.message.content[]? |
           if .type == "tool_use" then
             if .name == "Read" then "   📖 " + (.input.file_path // "?")
             elif .name == "Write" then "   ✏️  " + (.input.file_path // "?")
             elif .name == "Edit" then "   ✏️  " + (.input.file_path // "?")
             elif .name == "Bash" then "   ⚡ " + (.input.command // "?" | split("\n")[0] | if length > 80 then .[:80] + "..." else . end)
             else "   🔧 " + .name
             end
           elif .type == "text" then
             .text | if . != "" then split("\n") | to_entries | map(if .key == 0 then "   💬 " + .value else "      " + .value end) | join("\n") else empty end
           else empty
           end
         ] | join("\n") | if . != "" then . else empty end
       elif .type == "result" then
         if .subtype == "success" then "   ✅ Done (" + (.duration_ms / 1000 | tostring | split(".")[0]) + "s)"
         else "   ❌ Error: " + (.result // "unknown")
         end
       else empty
       end
     ' 2>/dev/null || true
   }

   for i in $(seq 1 "$MAX_ITERATIONS"); do
     echo ""
     echo "── ADP iteration $i/$MAX_ITERATIONS ─────────────────────────────"
     echo ""
     echo "🔨 IMPLEMENT"
     { cat .adp/PROMPT.md; echo "$CONTEXT"; } | claude -p --dangerously-skip-permissions --output-format stream-json --verbose | adp_stream
     echo ""
     echo "🧹 SIMPLIFY"
     { cat .adp/simplify.md; echo "$CONTEXT"; } | claude -p --dangerously-skip-permissions --output-format stream-json --verbose | adp_stream
     echo ""
     echo "🔍 REVIEW"
     { cat .adp/review.md; echo "$CONTEXT"; } | claude -p --dangerously-skip-permissions --output-format stream-json --verbose | adp_stream
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

7. **Make `loop.sh` executable**: Run `chmod +x .adp/loop.sh`.

8. **Update `.gitignore`**: Check if `.adp/artifacts/**/progress.txt` is already in `.gitignore`. If not, append:
   ```
   # ADP: agent progress logs (ephemeral, per-feature)
   .adp/artifacts/**/progress.txt
   ```
   To check: `grep -q 'adp/artifacts' .gitignore` — skip if it matches.

9. **Confirm to user**: Show what was created.

## Done

The workspace is ready when:
- `.adp/PROMPT.md` exists
- `.adp/simplify.md` exists
- `.adp/review.md` exists
- `.adp/loop.sh` exists and is executable
- `.adp/artifacts/` directory exists
- `.gitignore` has the progress.txt exclusion pattern
