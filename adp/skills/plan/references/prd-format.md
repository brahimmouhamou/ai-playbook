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
      "passes": false,
      "blocked": false
    },
    {
      "id": "US-003",
      "title": "Blocked story example",
      "acceptanceCriteria": [
        "AC-005: Given [context], when [action], then [outcome]"
      ],
      "passes": false,
      "blocked": true,
      "blockedReason": "Depends on US-001 from spec 019 which is not yet implemented"
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
  - **blocked**: Boolean, defaults to `false`. When `true`, the story is skipped by the implementation loop. Use this for stories that depend on work outside this feature (e.g., another spec must be implemented first, or a human is handling a prerequisite).
  - **blockedReason** *(optional)*: Short explanation of why the story is blocked. Shown in loop output and progress.txt.

## Key Rules

- **Mirror the spec.** Don't add, remove, or rephrase anything. IDs, titles, and acceptance criteria are copied verbatim.
- **Array order is the plan.** Stories are ordered by implementation priority — dependencies first. No separate implementation plan file needed.
- **No per-story verification.** Verification is handled globally by hooks (typecheck, linter, tests) — not tracked in prd.json.
- **User stories are the unit of work.** Each story maps to one implementation iteration.
- **Blocked stories are skipped.** Stories with `"blocked": true` are excluded from the implementation loop. They don't count toward completion. Use this when a story depends on external work (another feature, a manual step, infrastructure not yet available). Unblock by setting `blocked` to `false`.
- **Committable.** prd.json tracks progress and belongs in git. Only progress.txt is gitignored.

## Location

The file lives at `.adp/artifacts/NNN-feature-name/prd.json`. Each feature gets its own folder under `.adp/artifacts/`.

## Technical Notes (Optional)

A `technical-notes.md` file can be placed alongside prd.json at `.adp/artifacts/NNN-feature-name/technical-notes.md`. This file contains implementation-specific technical details that the implementing agent should know but that don't belong in the product spec.

Examples of what belongs in technical-notes.md:
- "Store enriched company data in Valkey with 7-day TTL matching token expiry"
- "Persist recipient company name in the connection record from VAT validation at creation time"
- "Use fire-and-forget pattern (Effect.forkDaemon) for async enrichment"
- "The existing `contractor-company-connection` schema needs a `status` column and `recipientEmail` column"

The `/plan` skill creates this file during planning if the user provides technical context. The implementing agent reads it alongside the spec and conventions.
