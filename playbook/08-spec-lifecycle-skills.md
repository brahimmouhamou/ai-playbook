# Spec Lifecycle Skills

## /new-spec — Creating Feature Specs

Used once per feature, at the start of work. Guides you through creating a clean, convention-aware spec through conversation. An Opus orchestrator drives the process, then three parallel Sonnet reviewers (functional analyst, UX SaaS expert, UI SaaS expert) check the draft before presenting it for human approval.

Full skill definition: [skills/new-spec/SKILL.md](./skills/new-spec/SKILL.md)

---

## /new-insight — Routing Feedback to the Right Files

Used constantly throughout a feature's life. When a bug, question, or discovery arrives, this skill uses a decision tree to determine what existing files need updating — spec, product convention, or technical convention — and drafts the additions in the appropriate voice.

Full skill definition: [skills/new-insight/SKILL.md](./skills/new-insight/SKILL.md)

---

## The Pair

```
/new-spec      →  "I need to build something new"
/new-insight   →  "I learned something, update the right files"
```

Together they keep specs, conventions, and docs improving over time. Every bug, every stakeholder question, every code review finding makes the system smarter through /new-insight. Every new feature starts clean and convention-aware through /new-spec.

---

## Fixing Bugs in Completed Features

A feature is done. Days later you find a bug or a missing edge case. Here's the workflow:

```
find bug → /adp:new-insight → updates the spec
         → /adp:plan        → fresh prd.json (all stories start passes:false)
         → loop.sh           → skips what's done, fixes what's broken
```

**Why this works without tracking previous progress:**

The spec is the source of truth. The code is the record of what's implemented. prd.json is disposable — it's a work tracker, not a history log.

When you re-plan a spec that was already implemented, the plan skill creates a fresh prd.json with all stories starting at `passes: false`. That looks wasteful, but it isn't. The loop handles it naturally:

1. The **implement agent** reads the code, sees the story is already satisfied, commits nothing
2. The **simplify agent** sees no new diff, skips
3. The **review agent** checks AC against the current code, marks `passes: true`, moves on

Stories that are already done cost a few seconds of review each. The story covering the bug gets real work. No special commands, no incremental planning, no progress tracking between runs.

**When the insight changes a convention instead of a spec:**

If `/adp:new-insight` updates a convention file rather than a spec, you may not need to re-plan at all. The loop's simplify and review agents read `docs/**/*.md` every iteration. If existing code now violates the updated convention, the next loop run on any feature will catch it.

**Key principle:** don't persist prd.json after a feature is done. Delete it or let it be overwritten. The spec + code tell you everything you need to know.

**Exception — the preference log is durable.** `.adp/preferences/rejections.jsonl` and `.adp/preferences/simplifications.jsonl` accumulate across every feature and are never deleted. They are the RLHF-analog corpus: each line is a preference signal that calibrates future review and simplify agents. See [11-preference-accumulation.md](./11-preference-accumulation.md) for how the log is written, read, and distilled into conventions.
