---
description: Show ADP workflow overview and available commands
argument-hint:
---

Display the following overview to the user exactly as written:

# ADP — Agentic Development Playbook

## Workflow

```
/adp:init  →  /adp:new-spec  →  /adp:plan  →  .adp/loop.sh
  setup         write spec       derive prd     implement
```

## Commands

- **`/adp:init`** — Initialize the `.adp/` workspace (PROMPT.md, loop.sh, artifacts folder, gitignore)
- **`/adp:new-spec`** — Create a convention-aware feature spec through guided conversation
- **`/adp:new-insight`** — Route a bug, discovery, or edge case to the right file (spec, convention, or docs)
- **`/adp:plan`** — Derive a prd.json from an approved spec (stories ordered by priority)
- **`/adp:help`** — Show this overview

## File Structure

```
.adp/
├── PROMPT.md                              # implement instructions (committed)
├── simplify.md                            # simplify instructions (committed)
├── review.md                              # review instructions (committed)
├── loop.sh                                # loop script (committed)
└── artifacts/
    └── NNN-feature-name/
        ├── prd.json                       # progress tracker (committed)
        ├── technical-notes.md             # implementation context (committed, optional)
        └── progress.txt                   # learning log (gitignored)
```

## Key Concepts

- **Specs are permanent, plans are disposable.** The spec (`specs/NNN/spec.md`) is the source of truth. prd.json is derived from it.
- **Array order is the plan.** Stories in prd.json are ordered by priority. The agent picks the first `passes: false` (and not blocked) story.
- **Blocked stories are skipped.** Stories with `"blocked": true` are excluded from the loop. Useful for dependencies on other features or manual prerequisites.
- **Technical notes ride along.** Optional `technical-notes.md` in the artifacts folder provides implementation context (schema changes, caching strategies, etc.) without polluting the product spec.
- **Conventions avoid repetition.** Product behavior in `specs/conventions/`, technical rules in `docs/conventions/`.
- **prd.json is committed.** It tracks progress across sessions. Only `progress.txt` is gitignored.

## Running the Loop

```bash
.adp/loop.sh 019-worker-jobs-panel        # default 30 iterations
.adp/loop.sh 019-worker-jobs-panel 10     # limit to 10 iterations
```

## Methodology

The full playbook lives in the plugin's source repo. Start at `playbook/00-start.md`.
