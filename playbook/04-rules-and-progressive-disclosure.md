# Rules & Progressive Disclosure

## Principle: Small Root, Deep Tree

LLMs can follow ~150-200 instructions consistently. Every token in CLAUDE.md loads on every request. Keep it tiny, point elsewhere.

---

## CLAUDE.md (~50-60 lines max)

Contains ONLY:
- One-sentence project description
- Monorepo structure overview
- Package manager and build commands
- Pointer to convention index (one link, not individual files)
- The most critical "never do this" rules (2-3 max)

### Example

```markdown
# Acme Core Platform

Multi-tenant contractor compliance SaaS. Monorepo with pnpm workspaces.

## Structure
- apps/core-backend — Node.js API (Clean Architecture, Effect-TS)
- apps/workforce-web — React frontend (TanStack Router)
- packages/core-contract — Shared API contracts (Effect Schema)

## Commands
- typecheck: `pnpm run typecheck`
- test: `pnpm run test`
- test single: `pnpm vitest run path/to/file`
- lint: `pnpm run lint`

## Technical Conventions
See docs/conventions/index.md for all coding conventions and guidelines.

## Critical Rules
- NEVER modify migration files that have already been applied
- ALWAYS run typecheck before committing
- Tenant isolation: every query MUST filter by tenant context
```

---

## Technical Conventions (`docs/conventions/`)

Technical rules and coding guidelines that the agent reads on demand. Not pre-loaded — the agent navigates here when it touches relevant code.

- **clean-architecture.md** — Layer boundaries, dependency direction, port/adapter patterns
- **testing-patterns.md** — Test naming, fixture patterns, what to mock vs integrate
- **domain-modeling.md** — Aggregate design rules, value objects, invariant enforcement
- **api-contracts.md** — Effect Schema conventions, error shapes, versioning

Note: these are separate from `specs/conventions/` which holds product behavior standards (pagination, auth, UX patterns) referenced by feature specs. The split:

- `specs/conventions/` — **what the product does** (functional, readable by anyone)
- `docs/conventions/` — **how to build it** (technical, for developers and agents)

---

## Three-Tier Rule Enforcement

| Tier | Mechanism | Nature | Example |
|------|-----------|--------|---------|
| **Enforce** | Hooks | Deterministic, cannot be ignored | `tsc --noEmit`, test runner, linter |
| **Guide** | CLAUDE.md + docs/ | Advisory, agent follows when loaded | Clean Architecture layers, naming |
| **Advise** | Skills | On-demand, loaded when triggered | TDD workflow, code review checklist |

Rules that absolutely cannot be violated → hooks.
Rules the agent should follow when working in an area → docs (progressive disclosure).
Complex workflows → skills.
