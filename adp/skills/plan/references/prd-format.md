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

- **feature**: The short name matching the spec folder (e.g., "019-worker-jobs-panel")
- **spec**: Relative path to the source spec file
- **userStories**: Array of user stories, ordered by implementation priority (dependencies first). The agent picks the first `passes: false` story each iteration.
  - **id**: Matches the US-NNN from the spec exactly
  - **title**: Short title from the spec exactly
  - **acceptanceCriteria**: List copied from the spec exactly, including AC-NNN prefixes
  - **passes**: Boolean, always starts as `false`

## Key Rules

- **Mirror the spec.** Don't add, remove, or rephrase anything. IDs, titles, and acceptance criteria are copied verbatim.
- **Array order is the plan.** Stories are ordered by implementation priority — dependencies first. No separate implementation plan file needed.
- **No per-story verification.** Verification is handled globally by hooks (typecheck, linter, tests) — not tracked in prd.json.
- **User stories are the unit of work.** Each story maps to one implementation iteration.
- **Committable.** prd.json tracks progress and belongs in git. Only progress.txt is gitignored.

## Location

The file lives at `.adp/artifacts/NNN-feature-name/prd.json`. Each feature gets its own folder under `.adp/artifacts/`.
