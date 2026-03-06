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
- **No technical details — at all.** The spec describes what the user sees, clicks, and reads. It never describes how the system works. This is the most important rule and the most commonly violated. See the "Technical Leakage" section below.
- **Conventions are feature-level.** Listed once at the top to avoid repetition, even if a convention only applies to some user stories.
- **Links, not names.** Convention references are markdown links so both humans and agents can navigate directly to the source.
- **Keep stories small.** A user story should be completable in one implementation iteration. If it feels too big, split it into smaller stories.
- **Mark unknowns, don't guess.** Use `[NEEDS CLARIFICATION: question]` inline for anything unclear. All markers must be resolved before the spec moves to plan derivation.
- **Specs don't hold state.** No checkboxes, no pass/fail tracking, no progress indicators. The spec defines *what* done looks like. Tracking *whether* it's done is the job of prd.json.
- **Background section is brief.** 2-4 sentences max. Describe the current user experience and what's missing. Do not explain system architecture, data flow, or precedence rules.

## Technical Leakage

This is the single biggest failure mode. Even when told "no technical details", specs drift into implementation because the input prompt contains technical context. **Strip it. The spec is for a product person, not an engineer.**

### Never include

- Database columns, table names, migrations, JSONB, schemas
- Entity names, DTOs, domain models, service classes
- Event names, message queues, pub/sub topics
- API endpoints, request/response shapes, HTTP verbs
- Component names, file paths, folder structures
- Cache layers, key formats, TTLs
- State machines, status enums beyond what the user sees
- Data precedence rules, conflict resolution logic
- "The system publishes/consumes/persists/queries…"

### How to rewrite technical language

| ❌ Technical (don't write this) | ✅ User-observable (write this) |
|---|---|
| "A `WorkerDocumentStoredEvent` is published that triggers the enrichment pipeline" | "After documents are retrieved, enrichment starts automatically" |
| "Fields are persisted on the `worker_documents` record along with `enrichedAt`" | "The document shows its classification and expiry date" |
| "The `EnrichmentStatusService` uses the same Valkey key format" | "The status indicator works the same way as for uploaded documents" |
| "Nullable `document_classification` (varchar) column is added" | *(don't mention — this is implementation)* |
| "The event carries the worker document ID and worker ID" | *(don't mention — this is implementation)* |
| "Falls back to displaying provider metadata" | "Shows the available document information" |
| "No namespace collision risk with UUIDs" | *(don't mention — this is implementation)* |

### The test

Read each acceptance criterion aloud and ask: **"Could a product manager verify this by using the product?"** If the answer is no — if they'd need to check a database, read a log, or inspect an event — rewrite it or remove it.

## File Structure

Feature specs live at `specs/NNN-feature-name/spec.md` where NNN is a zero-padded number. The folder name matches the git branch name: `feature/NNN-feature-name`.

Product conventions live at `specs/conventions/` — these describe system-wide product behavior (pagination, auth, error handling) written once, referenced from every feature spec.

Technical conventions live at `docs/conventions/` — these describe coding guidelines and architecture rules for developers and agents.
