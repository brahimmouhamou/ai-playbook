---
name: new-insight
description: "This skill should be used when the user reports a 'bug', 'discovery', 'edge case', 'feedback', or asks 'where should I document this', 'where does this belong', or needs to route a finding to the correct file (spec, convention, or docs)."
---

# /new-insight

Route bugs, questions, and discoveries to the right file using a decision tree.

## Decision Tree

### Step 1: Can a user verify it by using the product?

**Yes** → it's product behavior. Go to Step 2.

**No, it can only be verified by reading code or running technical tools** → it's implementation guidance → update or create a doc in `docs/conventions/`.

### Step 2: Is it specific to one feature?

**Yes** → update that feature's spec in `specs/NNN-feature-name/spec.md`:
- Add or modify a user story
- Add or modify an acceptance criterion on an existing story

**No, it applies across features** → update or create a convention in `specs/conventions/`:
- If the convention file exists, add the new behavior
- If no convention covers this concern, create a new one

## Scope Boundary

Do NOT read source code, trace execution paths, debug root causes, or propose implementation fixes. Your job ends at drafting the spec/convention update. The implementation loop handles the fix.

## What If No Spec Exists?

If the decision tree points to a feature spec that doesn't exist yet, do NOT create one. A single bug report is not enough context for a proper spec. Instead:

1. Tell the user no spec exists for this feature
2. Suggest using `/new-spec` to create the spec first
3. Then come back with `/new-insight` to add the bug as an acceptance criterion

## How to Draft the Addition

Write in the appropriate voice for the target file:

- **Spec** (`specs/NNN-feature-name/spec.md`): User-observable behavior only. No technical details. Use Given/When/Then for acceptance criteria.
- **Product convention** (`specs/conventions/`): Product-wide standard. Readable by anyone — developers, analysts, product people.
- **Technical convention** (`docs/conventions/`): Developer and agent guidance. Code-level detail, architecture patterns, naming rules.

If the same issue has BOTH a user-facing and technical aspect, draft entries for both locations.

## Output

1. State which file(s) to update and why
2. Draft the addition in the appropriate voice
3. Present the draft for human review before applying

## Examples

- "Users see a spinner forever when search returns no results" → **specs/conventions/search-behavior.md** (empty state behavior applies across features)
- "The contractor list doesn't paginate past page 10" → **specs/NNN-contractor-list/spec.md** (feature-specific bug, add acceptance criterion)
- "We should always use Effect.gen instead of pipe chains" → **docs/conventions/typescript.md** (technical coding pattern)
- "Error toasts disappear too fast for users to read" → **specs/conventions/error-handling.md** (product behavior) AND **docs/conventions/ux-implementation.md** (technical guidance on toast duration)
