# prd.json Format

## Structure

```json
{
  "feature": "feature-name",
  "spec": "specs/NNN-feature-name/spec.md",
  "userStories": [
    {
      "id": "US-001",
      "title": "Short title from spec",
      "acceptanceCriteria": [
        "AC-001: Given [context], when [action], then [outcome]",
        "AC-002: Given [context], when [action], then [outcome]"
      ],
      "passes": false
    }
  ]
}
```

## Fields

- **feature**: The short name matching the spec folder (e.g., "job-description-search")
- **spec**: Relative path to the source spec file
- **userStories**: Array of user stories, each with:
  - **id**: Matches the US-NNN from the spec exactly
  - **title**: Short title from the spec exactly
  - **acceptanceCriteria**: List copied from the spec exactly, including AC-NNN prefixes
  - **passes**: Boolean, always starts as `false`

## Key Rules

- **Mirror the spec.** Don't add, remove, or rephrase anything. IDs, titles, and acceptance criteria are copied verbatim.
- **No per-story verification.** Verification is handled globally by hooks (typecheck, linter, tests) — not tracked in prd.json.
- **User stories are the unit of work.** Each story maps to one implementation iteration.
- **The file is disposable.** Regenerate from the spec if it drifts. Delete after the feature ships.

## Location

The file lives at `.adp/prd.json` at the project root. The `.adp/` folder is gitignored.
