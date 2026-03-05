# Key Principles

1. **Specs are human artifacts. Plans are agent scaffolding.** Never let the agent mutate your specs.

2. **Progressive disclosure everywhere.** Small CLAUDE.md, deep docs tree, conventions resolved at derivation time.

3. **Hooks enforce. Docs guide. Skills advise.** Non-negotiable rules are hooks, not markdown instructions.

4. **Fresh context beats long context.** One task per iteration, exit, restart. Context rot is the enemy.

5. **Backpressure is the real orchestrator.** The compiler and test suite gate bad work. You don't need a state machine.

6. **DRY applies to specs too.** Write conventions once, reference everywhere. Feature specs contain only the delta.

7. **Plans are disposable.** If prd.json drifts, regenerate from spec. If progress.txt is stale, delete it.

8. **Start with HITL.** Watch single iterations, tune your prompts, then graduate to overnight runs.
