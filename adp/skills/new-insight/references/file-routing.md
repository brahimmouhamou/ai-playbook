# File Routing Guide

## Where Files Live

| Location | What it contains | Audience | Voice |
|----------|-----------------|----------|-------|
| `specs/NNN-feature-name/spec.md` | Feature-specific user stories and acceptance criteria | Anyone | User-observable behavior, Given/When/Then |
| `specs/conventions/` | Product-wide behavior standards (pagination, auth, UX patterns) | Anyone | Product standard, stakeholder-readable |
| `docs/conventions/` | Technical coding rules and architecture guidelines | Developers, agents | Code-level detail, patterns, naming rules |

## Decision Summary

1. **Can a user verify it by using the product?**
   - Yes + feature-specific → `specs/NNN-feature-name/spec.md`
   - Yes + applies across features → `specs/conventions/`
   - No (only verified by reading code) → `docs/conventions/`

2. **Both user-facing AND technical?** → Draft entries for both locations.

## Voice Examples

**Spec voice** (specs/NNN/spec.md):
> AC-005: Given a recruiter on the job list, when search returns no results, then an empty state message appears with a suggestion to broaden the search.

**Product convention voice** (specs/conventions/search-behavior.md):
> Empty state: When a search returns zero results, display a message suggesting the user broaden their search terms. Do not show an empty table.

**Technical convention voice** (docs/conventions/error-handling.md):
> Use Effect.catchTag for domain errors. Map each error tag to an HTTP status code in the API layer. Never expose internal error messages to the client.
