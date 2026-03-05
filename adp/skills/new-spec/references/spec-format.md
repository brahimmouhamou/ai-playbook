# Feature Spec Format

## Template

```markdown
# Feature: [Name]

> One-line summary of what this feature does from the user's perspective.

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

### US-002: [Short title]
**As a** [role], **I want** [goal], **so that** [benefit].

**Acceptance Criteria**:
- AC-003: Given [context], when [action], then [outcome]
- AC-004: Given [context], when [action], then [outcome]

## Out of Scope
What we're explicitly NOT doing.

## Open Questions
Anything unresolved. Mark unclear items inline with `[NEEDS CLARIFICATION: specific question]`.
All markers must be resolved before moving to plan derivation.
```

## Key Rules

- **Summary line is mandatory.** The `> summary` blockquote right after the title enables efficient scanning across all specs (one grep instead of reading every file).
- **No technical details.** The spec describes what the user experiences, not how the system implements it. No mention of databases, APIs, migrations, components, or file paths.
- **Conventions are feature-level.** Listed once at the top to avoid repetition, even if a convention only applies to some user stories.
- **Links, not names.** Convention references are markdown links so both humans and agents can navigate directly to the source.
- **Keep stories small.** A user story should be completable in one implementation iteration. If it feels too big, split it into smaller stories.
- **Mark unknowns, don't guess.** Use `[NEEDS CLARIFICATION: question]` inline for anything unclear. All markers must be resolved before the spec moves to plan derivation.
- **Specs don't hold state.** No checkboxes, no pass/fail tracking, no progress indicators. The spec defines *what* done looks like. Tracking *whether* it's done is the job of prd.json.

## File Structure

Feature specs live at `specs/NNN-feature-name/spec.md` where NNN is a zero-padded number. The folder name matches the git branch name: `feature/NNN-feature-name`.

Product conventions live at `specs/conventions/` — these describe system-wide product behavior (pagination, auth, error handling) written once, referenced from every feature spec.

Technical conventions live at `docs/conventions/` — these describe coding guidelines and architecture rules for developers and agents.
