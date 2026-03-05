---
name: plan
description: "This skill should be used when the user wants to 'create a plan', 'derive a plan', 'generate prd.json', 'start implementation planning', or needs to turn an approved spec into a prd.json and implementation plan."
---

# /plan

Derive a prd.json and implementation plan from an approved feature spec. For prd.json structure details, read `references/prd-format.md`.

## Prerequisites

The spec must be approved before running this skill:
- All `[NEEDS CLARIFICATION]` markers resolved
- Human has signed off on the spec

If the spec isn't ready, stop and tell the user to finish it first.

## Process

1. **Locate the spec**: Ask which feature to plan. Find the spec at `specs/NNN-feature-name/spec.md`. If ambiguous, list available specs and ask.

2. **Read the spec and its conventions**: Read the spec file. For each file referenced in the Conventions section, read it too. Conventions are resolved at plan time — the agent needs their content to implement correctly.

3. **Ensure `.adp/` exists**: Create `.adp/` at the project root if it doesn't exist. Verify it's in `.gitignore`.

4. **Generate `.adp/prd.json`**: For each user story in the spec, create an entry following the format in `references/prd-format.md`.

   Rules:
   - **Mirror the spec exactly.** IDs, titles, and acceptance criteria come straight from the spec. Don't add, remove, or rephrase.
   - **All stories start as `passes: false`.** The implementation loop flips them.
   - **No per-story verification.** Verification is global via hooks (typecheck, linter, tests).
   - **Feature and spec path are required.** The prd.json must reference the source spec.

5. **Generate `.adp/implementation-plan.md`**: Write a short plan covering:
   - Suggested story ordering (which to implement first based on dependencies)
   - Any notes on shared groundwork that multiple stories depend on
   - Keep it brief — this is a guide, not a second spec

6. **Present to user**: Show both files for review.

## Key Principles

- **Everything in `.adp/` is disposable.** If the plan drifts (e.g. a `/new-insight` updated the spec), regenerate from the spec. Delete the whole folder after the feature ships.
- **The spec is permanent, the plan is not.** The spec is the source of truth. The plan is a derived artifact.
- **Conventions resolve at plan time.** The spec links conventions for human readability. The agent reads them during planning to inform implementation.

## Done

The plan is ready when:
- prd.json has an entry for every user story in the spec
- IDs and acceptance criteria match the spec exactly
- implementation-plan.md has a suggested story order
- `.adp/` is in `.gitignore`
- The human has reviewed both files

Next step: Start the implementation loop — pick the first user story and implement it.
