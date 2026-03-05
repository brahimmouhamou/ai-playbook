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
