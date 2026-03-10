---
name: plan
description: "This skill should be used when the user wants to 'create a plan', 'derive a plan', 'generate prd.json', 'start implementation planning', or needs to turn an approved spec into a prd.json."
---

# /plan

Derive a prd.json from an approved feature spec. For prd.json structure details, read `references/prd-format.md`.

## Prerequisites

The spec must be approved before running this skill:
- All `[NEEDS CLARIFICATION]` markers resolved
- Human has signed off on the spec

If the spec isn't ready, stop and tell the user to finish it first.

## Process

1. **Locate the spec**: Ask which feature to plan. Find the spec at `specs/NNN-feature-name/spec.md`. If ambiguous, list available specs and ask.

2. **Read the spec and its conventions**: Read the spec file. For each file referenced in the Conventions section, read it too. Conventions are resolved at plan time — the agent needs their content to implement correctly.

3. **Prepare the artifacts folder**:
   ```bash
   mkdir -p .adp/artifacts/NNN-feature-name
   ```

4. **Generate `.adp/artifacts/NNN-feature-name/prd.json`**: For each user story in the spec, create an entry following the format in `references/prd-format.md`.

   Rules:
   - **Mirror the spec exactly.** IDs, titles, and acceptance criteria come straight from the spec. Don't add, remove, or rephrase.
   - **Order by priority.** Place stories in implementation order — dependencies first. The first `passes: false` story is the next one the agent picks up.
   - **Every story has `blocked: false` by default.** If the user indicates a story depends on external work (another spec, a manual prerequisite, infrastructure not yet available), set `blocked: true` and add an optional `blockedReason` string. Blocked stories are skipped by the loop.
   - **Smart merge with existing prd.json.** If a prd.json already exists, compare each story by ID and acceptance criteria:
     - If the story ID exists in the old prd.json **and its acceptance criteria are identical** → copy `passes` and `blocked`/`blockedReason` from the old entry.
     - If the story is new or its acceptance criteria changed → set `passes: false`, `blocked: false`.
     - Append a line to `progress.txt` listing which stories were reset and why (e.g. `RE-PLANNED: US-002 — new AC added: AC-016`).
   - If no prd.json exists yet, all stories start as `passes: false`, `blocked: false`.
   - **No per-story verification.** Verification is global via hooks (typecheck, linter, tests).
   - **Feature and spec path are required.** The prd.json must reference the source spec.

5. **Create technical notes (if applicable)**: If the user has provided implementation-specific technical context (database schema changes, caching strategies, service wiring, data flow details), create `.adp/artifacts/NNN-feature-name/technical-notes.md` with that information. Ask the user if they have technical notes to include. If they don't, skip this step.

6. **Present to user**: Show the file(s) for review.

## Key Principles

- **Artifacts are per-feature.** Each feature gets its own folder under `.adp/artifacts/`. This allows multiple features to be in flight simultaneously.
- **prd.json is committable.** It tracks progress and should be in git so teammates can pick up where you left off. Only `progress.txt` is gitignored.
- **The spec is permanent, the plan is not.** The spec is the source of truth. The plan is a derived artifact. If the plan drifts (e.g. a `/new-insight` updated the spec), regenerate from the spec. Only stories with changed ACs get re-run — already-passing stories are preserved.
- **Conventions resolve at plan time.** The spec links conventions for human readability. The agent reads them during planning to inform implementation.
- **Array order is the plan.** No separate implementation plan file. The story order in prd.json determines what gets built first.

## Done

The plan is ready when:
- prd.json has an entry for every user story in the spec
- IDs and acceptance criteria match the spec exactly
- Stories are ordered by implementation priority (dependencies first)
- The human has reviewed the file

Next step: Start the implementation loop — pick the first `passes: false` story and implement it.
