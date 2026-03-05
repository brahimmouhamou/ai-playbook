# Spec System

## Principle: DRY Specs via Convention Files

Cross-cutting concerns (pagination, auth, debounce, error handling) are identical across features. Extract them into convention files that are written once and referenced from every feature spec.

---

## Convention Files (`specs/conventions/`)

Each file documents one system-wide standard. Written by humans, reviewed in PRs, versioned in git. Examples:

- **pagination.md** — page/pageSize params, defaults, max values, empty page behavior, total count contract
- **search-behavior.md** — debounce timing, minimum characters, case-insensitive by default, loading states
- **api-authentication.md** — auth context extraction, token validation, 401 response shape
- **api-authorization.md** — permission checks, tenant scoping, 403 response shape
- **ux-patterns.md** — icon button tooltips, loading skeletons, empty states, confirmation dialogs
- **error-handling.md** — error response shapes, retry behavior, user-facing messages
- **performance-requirements.md** — response time budgets, index strategies, monitoring

---

## Feature Specs (`specs/NNN-feature-name/spec.md`)

Each feature spec contains ONLY the functional scope — what's unique about this feature from the user's perspective. No technical implementation details, no database schemas, no file paths. It references conventions but never repeats them.

### Structure

```markdown
# Feature: [Name]

> One-line summary of what this feature does from the user's perspective.

## Problem
Why this feature exists. 2-3 sentences from the user's perspective.

## Conventions
- [pagination](../conventions/pagination.md)
- [search-behavior](../conventions/search-behavior.md)
- [error-handling](../conventions/error-handling.md)

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

### Key rules

- **Summary line is mandatory.** The `> summary` blockquote right after the title enables efficient scanning across all specs (one grep instead of reading every file).
- **No technical details.** The spec describes *what* the user experiences, not *how* the system implements it.
- **Conventions are feature-level.** Listed once at the top to avoid repetition, can be applicable to one or more user stories.
- **Links, not names.** Convention references are markdown links so both humans and agents can navigate directly to the source.
- **Keep stories small.** A user story should be completable in one implementation iteration. If it feels too big, split it into smaller stories in the spec.
- **Mark unknowns, don't guess.** Use `[NEEDS CLARIFICATION: question]` inline for anything unclear. All markers must be resolved before the spec moves to plan derivation.
- **Specs don't hold state.** No checkboxes, no pass/fail tracking, no progress indicators. The spec defines *what* done looks like. Tracking *whether* it's done is the job of prd.json.

---

## Why This Matters

- **Writing specs**: 20 minutes instead of an hour. You're not re-documenting pagination.
- **For the agent**: Conventions get resolved during plan derivation (see [Plan Derivation](./03-plan.md)).
- **For analysts**: They learn conventions once, then review only feature deltas.
- **For bug reports**: Convention violation → check `conventions/*.md`. Feature bug → check feature `spec.md`.
- **For onboarding**: New developer reads conventions folder → understands the whole system.
