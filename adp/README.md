# Agentic Dev Playbook Plugin

Spec-driven development skills for agentic workflows. Based on Matt Pocock, Geoffrey Huntley, and community best practices.

## Skills

### /new-spec
Guided feature spec creation through conversation. Identifies applicable conventions, generates branch names, drafts user stories with acceptance criteria, and runs parallel review agents (functional analyst, UX expert, UI expert) before presenting the draft for approval.

### /new-insight
Routes bugs, questions, and discoveries to the right file using a decision tree. Determines whether a finding belongs in a feature spec, product convention, or technical convention, and drafts the addition in the appropriate voice. Auto-escalates to a convention file when the same tag pattern appears ≥3 times across ≥2 features in the preference log.

### /distill-conventions
Clusters the durable preference logs under `.adp/preferences/` (rejections + simplifications) into proposed convention updates. Offline batch job — run every 2-4 weeks. Flags deterministic patterns as hook candidates rather than markdown updates. Review the [preference-accumulation playbook section](../playbook/11-preference-accumulation.md) for the RLHF-analog framing.

## Expected Project Structure

This plugin assumes your project has:

```
specs/
├── conventions/           # Product behavior standards
│   ├── pagination.md
│   ├── search-behavior.md
│   └── ...
├── 001-feature-name/
│   └── spec.md
└── ...

docs/
├── conventions/           # Technical coding rules
│   ├── clean-architecture.md
│   └── ...
└── ...
```

## Setup

No configuration required. Install the plugin and the skills will be available when their trigger phrases are detected.
