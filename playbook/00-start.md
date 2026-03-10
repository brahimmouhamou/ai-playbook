# Agentic Development Playbook

**Based on**: Matt Pocock, Geoffrey Huntley, community best practices (Jan-Mar 2026)

---

## How to Use This Playbook

This playbook is organized as a layered file structure. This index provides a brief summary of each section so you (or an AI agent) can decide which files to read in full. Each section is a self-contained document — read only what's relevant to your current question or task.

---

## Sections

### [Philosophy & Architecture](./01-philosophy-and-architecture.md)
The foundational mental model behind agentic development. Covers why simple loops beat complex orchestrators, how the spec becomes the primary engineering artifact, and provides the full architecture diagram showing human-owned files, agent-owned files, enforcement hooks, and skills. Start here if you're new to the playbook or need the big picture.

### [Specification System](./02-specify.md)
How we write specs using DRY conventions. Convention files (`specs/conventions/`) capture cross-cutting concerns like pagination, auth, and error handling — written once, referenced everywhere. Feature specs (`specs/NNN-feature-name/spec.md`) contain only the delta: what's unique to that feature. Includes the full spec template and explains why this approach saves time for writers, agents, analysts, and new developers.

### [Plan Derivation](./03-plan.md)
The process of turning a human-readable spec into an agent-executable plan. Conventions are resolved and inlined at derivation time so the agent never needs to chase references during its loop. Covers the derivation prompt, prd.json structure, and the principle that plans are disposable scaffolding while specs are permanent.

### [Rules & Progressive Disclosure](./04-rules-and-progressive-disclosure.md)
How to structure CLAUDE.md and code convention docs for optimal agent performance. The core principle: keep the root file tiny (~50-60 lines of pointers) and put detailed rules in `docs/` files the agent reads on demand. Includes the three-tier enforcement model — hooks enforce, docs guide, skills advise — and an example CLAUDE.md.

### [The Implementation Loop](./05-implementation-loop.md)
The iteration engine: a bash loop that gives the agent fresh context per user story. Each iteration picks one user story from prd.json, implements it, verifies it, commits, and exits. Covers PROMPT.md, the loop script, and the four feedback loops (TypeScript compiler, test suite, linter, stop hook auto-review) that act as backpressure.

### [TDD Integration](./06-tdd-integration.md)
How to enforce Red-Green-Refactor inside the implementation loop using isolated subagents. Separate agents for writing tests, implementing, and refactoring prevent context pollution where the same agent "cheats" by writing tests that match its own implementation. Includes the TDD skill definition.

### [Code Review](./07-code-review.md)
Three review checkpoints: stop hook auto-review (every implementation pass), /simplify (post-feature, pre-PR), and /code-review (PR-time with parallel reviewers and confidence scoring). Review happens at boundaries, not inline, to keep the implementation agent focused.

### [Spec Lifecycle Skills](./08-spec-lifecycle-skills.md)
Two complementary skills for keeping specs alive. `/new-spec` guides you through creating a convention-aware feature spec from scratch. `/new-insight` routes bugs, questions, and discoveries to the right file (spec vs convention vs docs) using a decision tree. Together they ensure the system gets smarter over time.

### [Implementation Plan](./09-implementation-plan.md)
A phased, day-by-day rollout plan for adopting this playbook. Eight phases from foundation setup through first real run, covering file structure, spec templates, skills, plan derivation, implementation loop, TDD, code review, and graduation from HITL to AFK mode.

### [Key Principles](./10-key-principles.md)
The eight core principles distilled into a quick-reference summary. Covers spec ownership, progressive disclosure, three-tier enforcement, fresh context, backpressure, DRY specs, disposable plans, and the HITL-first approach. Read this for a fast refresher on the playbook's philosophy.
