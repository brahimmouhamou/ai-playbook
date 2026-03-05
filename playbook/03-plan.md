# Create a Plan

## Principle: Resolve Conventions at Derivation Time

The spec references conventions for human readability. The plan inlines them for agent reliability. Different audiences, different formats, same source of truth.

---

## The Derivation Prompt

Run once before starting a implementation loop:

```
Read specs/NNN-feature-name/spec.md.
For each file referenced in the Conventions section, read it too.
Generate:
1. .adp/prd.json — each user story as an entry with:
   - id (matching US-NNN from spec)
   - title
   - acceptanceCriteria (list, copied from spec)
   - passes: false
2. .adp/implementation-plan.md — story ordering with dependencies

The agent reads convention files and docs/ during implementation
to figure out the technical approach. The plan just mirrors
the spec's user stories.
```

---

## Output: prd.json

The plan mirrors the spec structure. User stories are the unit of work. Verification is handled globally by hooks (typecheck, tests, linter) — not per story.

```json
{
  "feature": "job-description-search",
  "spec": "specs/001-job-description-search/spec.md",
  "userStories": [
    {
      "id": "US-001",
      "title": "Search job descriptions",
      "acceptanceCriteria": [
        "AC-001: Given a recruiter on the job list, when they type 3+ characters, then matching jobs appear",
        "AC-002: Given search results, when results exceed one page, then pagination controls appear"
      ],
      "passes": false
    },
    {
      "id": "US-002",
      "title": "Filter by department",
      "acceptanceCriteria": [
        "AC-003: Given a recruiter on the job list, when they select a department, then only jobs in that department appear",
        "AC-004: Given an active department filter, when they clear it, then all jobs appear again"
      ],
      "passes": false
    }
  ]
}
```

---

## Key Principles

Everything in `.adp/` is disposable — both `prd.json` and `implementation-plan.md`. If the plan drifts (e.g. because a `/new-insight` updated the spec), regenerate it from the spec. Delete the whole folder after the feature ships. The spec is permanent, the plan is not.

The `.adp/` folder lives at the project root, not inside `specs/`. This keeps a clean separation between human-owned files (`specs/`, `docs/`) and agent-owned derived artifacts. Add it to `.gitignore`:

```
# Agent-derived artifacts (disposable, regenerate from spec)
.adp/
```
