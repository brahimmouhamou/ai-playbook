---
name: new-spec
description: "Create a new feature spec through guided conversation. Use when starting a new feature, enhancement, or significant change."
---

# /new-spec

Create a convention-aware feature spec through guided discovery.

## Process

1. **Understand the feature**: Ask for a brief description. If unclear, ask clarifying questions about:
   - The specific problem being solved
   - User-observable behavior (what changes for the user?)
   - Which parts of the system are affected
   - What's explicitly out of scope

2. **Identify conventions**: Read `specs/conventions/` to determine which conventions apply to this feature.

3. **Generate branch and folder name**:
   - Extract 2-4 keywords in action-noun format (e.g., "job-description-search", "contractor-onboarding")
   - Check existing `specs/` folders, local branches, and remote branches to find the highest number
   - Use the next number: `feature/NNN-short-name`
   - The spec folder and branch share the same name

4. **Draft the spec**: Save to `specs/NNN-short-name/spec.md` following this structure:

   ```markdown
   # Feature: [Name]

   ## Problem
   Why this feature exists. 2-3 sentences from the user's perspective.

   ## Conventions
   - [convention-name](../conventions/convention-name.md)
   - [another-convention](../conventions/another-convention.md)

   ## User Stories

   ### US-001: [Short title]
   **As a** [role], **I want** [goal], **so that** [benefit].

   **Acceptance Criteria**:
   - AC-001: Given [context], when [action], then [outcome]
   - AC-002: Given [context], when [action], then [outcome]

   ## Out of Scope
   What we're explicitly NOT doing.

   ## Open Questions
   Anything unresolved. Mark unclear items inline with `[NEEDS CLARIFICATION: specific question]`.
   All markers must be resolved before moving to plan derivation.
   ```

   Rules for the spec:
   - **No technical details.** Describe what the user experiences, not how the system implements it.
   - **Conventions are feature-level.** Listed once at the top to avoid repetition.
   - **Links, not names.** Convention references are markdown links.
   - **Keep stories small.** Each user story should be completable in one Ralph iteration. If too big, split it.
   - **Mark unknowns, don't guess.** Use `[NEEDS CLARIFICATION: question]` for anything unclear.

5. **Review the draft**: Spawn three parallel Sonnet review agents:
   - **Functional analyst**: Are the user stories complete? Missing edge cases? Acceptance criteria testable?
   - **UX SaaS expert**: Does the flow make sense? Missing states (empty, error, loading)? Consistent with conventions?
   - **UI SaaS expert**: Component patterns feasible? Accessibility gaps? Responsive considerations?

6. **Synthesize feedback**: Incorporate reviewer findings into the draft.

7. **Present to user**: Show the updated draft for human review.

8. **Iterate**: Repeat until approved. Resolve all `[NEEDS CLARIFICATION]` markers before considering the spec complete.

## Model Selection

The orchestrator (driving the conversation, synthesizing feedback, making judgment calls) runs on Opus. The review agents (focused single-lens checks) run on Sonnet — cheaper, faster, and scoped enough that they don't need Opus-level reasoning.

## Done

The spec is ready when:
- All user stories have testable acceptance criteria (Given/When/Then)
- All `[NEEDS CLARIFICATION]` markers are resolved
- Conventions are linked, not repeated
- Out of scope is explicitly defined
- The human has approved the draft

Next step: `/plan` to derive prd.json from the spec.
