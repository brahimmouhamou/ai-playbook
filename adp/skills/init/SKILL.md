---
name: init
description: "This skill should be used when the user wants to 'initialize adp', 'set up adp', 'bootstrap the agentic playbook', or needs to create the .adp folder structure in a project."
---

# /init

Initialize the ADP workspace in a project. Creates the `.adp/` folder with operational files at the project root.

## Process

1. **Never delete existing files.** If `.adp/` already exists, that's fine — proceed and overwrite only PROMPT.md and loop.sh. Never remove the `artifacts/` folder or anything inside it. Existing prd.json files, progress.txt files, and any other user content must be preserved.

2. **Create the folder structure** (use `mkdir -p`, safe to run if folders exist):
   ```
   .adp/
   ├── PROMPT.md
   ├── loop.sh
   └── artifacts/
   ```

3. **Create `.adp/PROMPT.md`**:
   ```markdown
   You are implementing a feature.

   ## Your Task
   1. Find the prd.json in .adp/artifacts/ for the current feature
   2. Pick the first user story where passes is false
   3. Implement ONLY that user story
   4. Run typecheck, linter, and tests
   5. If passing: set passes to true, git commit,
      append progress to the feature's progress.txt (next to prd.json)
   6. If failing: fix and retry (max 3 attempts)
   7. Exit

   ## Rules
   - Work on ONE user story only. Do not look ahead.
   - Every commit must pass typecheck, linter, and tests.
   - Read docs/*.md for conventions when touching relevant code.
   - Keep commits small and descriptive (conventional commits).
   ```

4. **Create `.adp/loop.sh`**:
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
     echo "=== ADP iteration $i/$MAX_ITERATIONS ==="
     cat .adp/PROMPT.md | claude -p

     if jq -e '[.userStories[] | select(.passes == false)] | length == 0' "$PRD" > /dev/null 2>&1; then
       echo "All user stories complete!"
       exit 0
     fi
   done

   echo "Reached max iterations ($MAX_ITERATIONS). Some stories still incomplete."
   exit 1
   ```

5. **Make `loop.sh` executable**: Run `chmod +x .adp/loop.sh`.

6. **Update `.gitignore`**: Check if `.adp/artifacts/**/progress.txt` is already in `.gitignore`. If not, append:
   ```
   # ADP: agent progress logs (ephemeral, per-feature)
   .adp/artifacts/**/progress.txt
   ```
   To check: `grep -q 'adp/artifacts' .gitignore` — skip if it matches.

7. **Confirm to user**: Show what was created.

## Done

The workspace is ready when:
- `.adp/PROMPT.md` exists
- `.adp/loop.sh` exists and is executable
- `.adp/artifacts/` directory exists
- `.gitignore` has the progress.txt exclusion pattern
