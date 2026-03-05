# Philosophy & Architecture

## Philosophy

Simple loops beat complex orchestrators. The models are good enough now that you don't need state machines or multi-agent frameworks — you need clear specs, strong feedback loops, and fresh context per iteration.

Your job shifts from writing code to writing specifications and engineering backpressure. The spec is the engineering artifact. The code is downstream output.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    HUMAN-OWNED                          │
│                                                         │
│  specs/                                                 │
│  ├── conventions/           System-wide standards       │
│  │   ├── pagination.md      (write once, reference      │
│  │   ├── search-behavior.md  from every feature)        │
│  │   ├── api-auth.md                                    │
│  │   ├── api-authorization.md                           │
│  │   ├── ux-patterns.md                                 │
│  │   ├── error-handling.md                              │
│  │   └── performance.md                                 │
│  │                                                      │
│  ├── 001-job-description-search/                        │
│  │   └── spec.md            Feature delta only,         │
│  ├── 002-contractor-onboarding/  references conventions │
│  │   └── spec.md                                        │
│  └── ...                                                │
│                                                         │
│  docs/                                                  │
│  ├── conventions/           Technical rules             │
│  │   ├── clean-architecture.md  (progressive disclosure │
│  │   ├── testing-patterns.md     for the agent)         │
│  │   ├── domain-modeling.md                             │
│  │   └── api-contracts.md                               │
│  └── ...                    Other project docs          │
│                                                         │
│  CLAUDE.md                  Tiny (~50-60 lines).        │
│                             Pointers only.              │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    AGENT-OWNED                          │
│                                                         │
│  .adp/                      Agent workspace              │
│  ├── PROMPT.md              Implement instructions       │
│  ├── review.md              Review instructions          │
│  ├── loop.sh                Loop script                  │
│  └── artifacts/                                          │
│      └── NNN-feature/                                    │
│          ├── prd.json       User stories (committed)     │
│          └── progress.txt   Learning log (gitignored)    │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    ENFORCEMENT                          │
│                                                         │
│  Hooks (deterministic, non-negotiable)                  │
│  ├── PostToolUse: tsc --noEmit after file edits         │
│  ├── PreCommit: tests must pass                         │
│  ├── Stop: auto-review subagent on modified files       │
│  └── PostCommit: append to progress.txt                 │
│                                                         │
│  Skills (on-demand workflows)                           │
│  ├── /new-spec              Guided feature spec creation│
│  ├── /new-insight           Route bugs/feedback to the  │
│  │                          right files (spec/conv/doc) │
│  ├── /plan                  Spec → prd.json             │
│  ├── /tdd-cycle             Red-Green-Refactor          │
│  ├── /simplify              3-agent parallel review     │
│  └── /code-review           PR review with scoring      │
└─────────────────────────────────────────────────────────┘
```
