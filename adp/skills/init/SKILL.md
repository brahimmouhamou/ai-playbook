---
name: init
description: "This skill should be used when the user wants to 'initialize adp', 'set up adp', 'bootstrap the agentic playbook', or needs to create the .adp folder structure in a project."
---

# /init

Initialize the ADP workspace in a project. Creates the `.adp/` folder with operational files at the project root.

## Process

1. **Check if `.adp/` already exists**: If it does, warn the user and ask before overwriting.

2. **Create the folder structure**:
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

4. **Create `.adp/loop.sh`**:
   ```bash
   #!/bin/bash
   FEATURE="${1:?Usage: ./loop.sh <feature-name>}"
   MAX_ITERATIONS=${2:-30}

   if [ ! -f ".adp/artifacts/$FEATURE/prd.json" ]; then
     echo "Error: .adp/artifacts/$FEATURE/prd.json not found"
     echo "Run /adp:plan first to generate the prd.json"
     exit 1
   fi

   for i in $(seq 1 $MAX_ITERATIONS); do
     echo "=== ADP iteration $i ==="
     cat .adp/PROMPT.md | claude -p --model opus

     if jq -e '[.userStories[] | select(.passes == false)] | length == 0' .adp/artifacts/$FEATURE/prd.json > /dev/null 2>&1; then
       echo "All user stories complete!"
       break
     fi
   done
   ```

5. **Make `loop.sh` executable**: `chmod +x .adp/loop.sh`

6. **Update `.gitignore`**: Append the following if not already present:
   ```
   # ADP: agent progress logs (ephemeral, per-feature)
   .adp/artifacts/**/progress.txt
   ```

7. **Confirm to user**: Show what was created and explain the workflow.

## Done

The workspace is ready when:
- `.adp/PROMPT.md` exists
- `.adp/loop.sh` exists and is executable
- `.adp/artifacts/` directory exists
- `.gitignore` has the progress.txt exclusion pattern
