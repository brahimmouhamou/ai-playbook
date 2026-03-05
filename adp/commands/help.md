---
description: Show ADP workflow overview and available commands
argument-hint:
---

Display the following overview to the user:

# ADP — Agentic Development Playbook

## Workflow

```
/adp:init  →  /adp:new-spec  →  /adp:plan  →  loop.sh
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
├── PROMPT.md                              # loop instructions (committed)
├── loop.sh                                # loop script (committed)
└── artifacts/
    └── NNN-feature-name/
        ├── prd.json                       # progress tracker (committed)
        └── progress.txt                   # learning log (gitignored)
```

## Key Concepts

- **Specs are permanent, plans are disposable.** The spec (`specs/NNN/spec.md`) is the source of truth. prd.json is derived from it.
- **Array order is the plan.** Stories in prd.json are ordered by priority. The agent picks the first `passes: false` story.
- **Conventions avoid repetition.** Product behavior in `specs/conventions/`, technical rules in `docs/conventions/`.
- **prd.json is committed.** It tracks progress across sessions. Only `progress.txt` is gitignored.

## Methodology

The full playbook (philosophy, spec format, loop design, TDD, code review) lives in the plugin's source repo. Read the playbook at `playbook/00-start.md` for the deep dive.
