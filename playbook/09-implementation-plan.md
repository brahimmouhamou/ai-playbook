# Step-by-Step Implementation Plan

## Phase 1: Foundation (Day 1)

**Goal**: Set up the file structure and CLAUDE.md

1. Create `docs/` folder with code convention files:
   - Extract Clean Architecture rules from your existing knowledge into `docs/clean-architecture.md`
   - Same for `testing-patterns.md`, `domain-modeling.md`, `api-contracts.md`, `typescript-conventions.md`

2. Trim your CLAUDE.md to ~50-60 lines:
   - Project overview, monorepo structure, build commands
   - Pointers to docs/ files
   - 2-3 critical "never" rules only

3. Create `specs/conventions/` folder:
   - Start with the conventions you repeat most: `pagination.md`, `api-authentication.md`, `api-authorization.md`
   - Add `search-behavior.md`, `ux-patterns.md`, `error-handling.md` as needed
   - Each file: one concern, written once, complete enough that a feature spec never needs to repeat it

---

## Phase 2: Apply Spec Format (Day 1-2)

**Goal**: Refactor existing specs to follow the format defined in [02-specify](./02-specify.md)

4. Refactor your existing `specs/001-job-description-search/spec.md`:
   - Move pagination, auth, and performance details to `specs/conventions/`
   - Rewrite as user stories with acceptance criteria (Given/When/Then)
   - Reduce spec to feature-delta only
   - Verify it's still complete when read alongside the conventions

---

## Phase 3: Spec Lifecycle Skills (Day 2-3)

**Goal**: Automate spec creation and feedback routing

6. Create `/new-spec` skill (`.claude/skills/new-spec/SKILL.md`):
   - Guided conversation for new feature specs
   - Reads conventions folder to identify applicable standards
   - Generates branch name and spec folder from feature description
   - Produces spec in the lean template format

7. Create `/new-insight` skill (`.claude/skills/new-insight/SKILL.md`):
   - Routes bugs, questions, discoveries to the right files
   - Decision tree: spec vs convention vs docs
   - Drafts additions in the appropriate voice

8. Test both skills:
   - `/new-spec` on a small upcoming feature
   - `/new-insight` on a known bug or edge case in job-description-search

---

## Phase 4: Plan Derivation (Day 3-4)

**Goal**: Automate spec → prd.json generation

9. Write your plan skill (save as `.claude/skills/plan/SKILL.md`):
   - Reads feature spec + all referenced convention files
   - Generates prd.json with user stories and acceptance criteria
   - Stories ordered by implementation priority (dependencies first)

10. Test plan derivation on your job-description-search spec:
    - Run the derivation
    - Verify prd.json mirrors all user stories from the spec
    - Verify acceptance criteria are copied correctly
    - Verify story order makes sense (dependencies first)

---

## Phase 5: Implementation Loop Setup (Day 4-5)

**Goal**: Get the basic loop running

11. Create `.adp/PROMPT.md`:
    - Instructions for picking one user story, implementing, verifying, committing
    - Reference to docs/ for conventions
    - Progress.txt append instruction

12. Create the loop script (`loop.sh`):
    - Bash loop with iteration cap (start with 5-10)
    - Completion check (all user stories passes: true)
    - Start with HITL mode (single iterations, you watch)

13. Set up hooks:
    - PostToolUse: `tsc --noEmit` after file edits
    - Pre-commit: typecheck, linter, and tests must pass
    - Post-commit: append to progress.txt

---

## Phase 6: TDD Integration (Day 5-6)

**Goal**: Add the Red-Green-Refactor cycle

14. Create TDD skill (`.claude/skills/tdd-cycle/SKILL.md`):
    - Red/Green/Refactor with isolated subagents
    - Each phase gets fresh context

15. Create subagent definitions:
    - `.claude/agents/test-writer.md` — writes failing tests only
    - `.claude/agents/implementer.md` — writes minimal code to pass
    - `.claude/agents/refactorer.md` — cleans up, runs simplify checks

---

## Phase 7: Code Review (Day 6-7)

**Goal**: Automated quality gates

16. Set up Stop hook auto-review:
    - Review subagent with fresh context
    - Reviews only modified files since last review
    - Blocks commit if critical issues found

17. Create or install `/code-review` skill for PR-time reviews:
    - Parallel reviewers with confidence scoring
    - Integrates with your PR workflow

---

## Phase 8: First Real Run (Day 7-8)

**Goal**: End-to-end test with a real feature

18. Run the full workflow on job-description-search:
    - Use /new-spec to verify your existing spec is clean (or refactor it)
    - Derive prd.json from spec
    - Start in HITL mode (single iterations, you watch)
    - Observe TDD cycle, review quality, progress tracking
    - Iterate on prompts based on what you observe

19. Refine based on observations:
    - Tune PROMPT.md phrasing where agent struggles
    - Adjust hook sensitivity
    - Add missing convention docs for things the agent gets wrong

20. Graduate to AFK mode:
    - Increase iteration cap to 20-30
    - Run in Docker sandbox for safety
    - Monitor via progress.txt and git log
