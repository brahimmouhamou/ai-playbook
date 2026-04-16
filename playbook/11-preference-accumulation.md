# Preference Accumulation (RLHF-Analog at the Prompt Layer)

## Principle: Compound Learning Across Features

You cannot fine-tune Claude's weights. You can accumulate preferences in a durable log and feed them back into future runs. That is the RLHF pattern, implemented at the prompt/context layer instead of the gradient layer.

Andrej Karpathy's framing of post-training distinguishes:

- **SFT (Supervised Fine-Tuning)** — a corpus of "good examples"
- **RLHF** — a reward model + policy optimization over preference pairs
- **RLVR (RL from Verifiable Rewards)** — RL against objective checkers (tests, compilers, graders). Karpathy argues RLVR is the stronger paradigm because the reward is hard to hack.

ADP already does **RLVR**: typecheck, linter, and tests are the verifier gates. The implement agent cannot cheat them. This doc adds the **RLHF-analog** layer — preference signals that are not reducible to pass/fail tests, such as "this code is convention-violating" or "this is better written as X."

---

## The Two Preference Logs

Both live under `.adp/preferences/`. Both are append-only NDJSON. Both are durable across feature lifecycles. Neither is regenerated — they are the long-term memory.

### `.adp/preferences/rejections.jsonl`

Written by the review agent when it REJECTs a user story. One line per rejection. Schema:

```json
{
  "ts": "2026-04-16T14:22:10Z",
  "feature": "042-search-ui",
  "story": "US-003",
  "reason": "missing error handling on empty-result branch",
  "ac": "AC-002",
  "diff_sha": "abc123...",
  "tags": ["error-handling", "ui"]
}
```

Tags are 1-3 slugs drawn from the canonical vocabulary below. They are the retrieval key: both the review agent (few-shot) and `/new-insight` (escalation) match on `tags` with `index($t)` exact matching, so free-form tags will fragment the vocabulary and silently drop recall.

### Canonical Tag Vocabulary

Pick 1-3 from this list. Only invent a new tag if no existing tag fits; if you invent one, add it here in the same change.

| Tag | Covers |
|---|---|
| `error-handling` | Exceptions, error responses, empty states, retry/backoff, failure UX |
| `pagination` | Page/cursor semantics, limits, total counts, empty pages |
| `tenant-isolation` | Multi-tenant scoping, tenant filters, cross-tenant leaks |
| `validation` | Input validation, schema checks, sanitization |
| `auth` | Authentication, session, token handling |
| `authorization` | Permissions, roles, access control, scoping rules |
| `ui` | React/frontend components, layout, a11y, visible behavior |
| `api` | HTTP endpoints, contracts, request/response shape |
| `db` | Schema, queries, migrations, transactions, N+1 |
| `tests` | Missing tests, mocking, test isolation, flaky tests |
| `naming` | Identifier names, file names, conventions |
| `types` | Type safety, generics, narrowing, schema types |
| `performance` | Latency, N+1, unnecessary work, hot paths |
| `security` | Injection, XSS, CSRF, secrets handling, OWASP top 10 |
| `logging` | Observability, log format, redaction, log levels |
| `dead-code` | Unused imports, unreachable branches, stale flags |
| `docs` | README, inline comments, API docs, convention files |

Apply this vocabulary uniformly in: the rejection writer (review.md step "If REJECT"), the few-shot reader (review.md step 5), and `/new-insight` Step 0.

### `.adp/preferences/simplifications.jsonl`

Written by the simplify agent after each `refactor(us-NNN):` commit. Captures the before/after pair — DPO-style preference data. Schema:

```json
{
  "ts": "2026-04-16T14:30:02Z",
  "feature": "042-search-ui",
  "story": "US-003",
  "before_sha": "abc...",
  "after_sha": "def...",
  "rationale": "extracted duplicated validation into a helper",
  "convention": "docs/conventions/clean-architecture.md"
}
```

The pair plus rationale is the signal. The diff between `before_sha` and `after_sha` is recoverable from git, so we do not duplicate it in the log.

---

## How the Logs Are Used

Three read paths. All are cheap — `jq` over NDJSON, no vector DB.

### 1. Few-shot retrieval in the review agent

Before deciding APPROVE/REJECT, the review agent pulls up to 3 past rejections whose `tags` overlap with the files in the current diff. They calibrate its rubric: if the team has consistently rejected missing-error-handling on UI code, this review applies the same standard.

Implementation: see the "Pull few-shot examples" step inside the review.md block in `playbook/05-implementation-loop.md`.

### 2. Auto-escalation in `/new-insight`

When a new bug or discovery comes in, `/new-insight` scans the log first. If the same tag pattern has produced **≥3 rejections across ≥2 distinct features**, the finding is no longer a feature-specific quirk — it is a pattern. The skill forces routing to `specs/conventions/` or `docs/conventions/` instead of the single feature spec.

Threshold rationale: 3 strikes / 2 features = small enough to catch real patterns, large enough to filter one-off noise. Tune for your team.

Implementation: see `adp/skills/new-insight/SKILL.md` Step 0.

### 3. Offline distillation via `/distill-conventions`

Batched. Periodic. A skill that reads both JSONL logs, clusters by `tags`, and proposes new or updated convention files. Human approves. This is the reward-model-rubric-distillation analog — turning accumulated preferences into durable SFT material (the convention docs that load on demand into every future run).

Implementation: `adp/skills/distill-conventions/SKILL.md`.

---

## The Mapping to Post-Training

| Post-training stage | ADP equivalent |
|---|---|
| Pretraining | Claude base weights (Anthropic-owned, out of scope) |
| SFT corpus | `CLAUDE.md`, `docs/conventions/*.md`, `specs/conventions/*.md` — loaded on demand |
| Reward model | `review.md` agent |
| Preference pairs (for DPO/PPO) | `.adp/preferences/simplifications.jsonl` |
| Rejection feedback (for negative examples) | `.adp/preferences/rejections.jsonl` |
| Policy | The implement agent + `loop.sh` iteration |
| Verifier rewards (RLVR) | typecheck, linter, tests, review agent's AC check |
| Offline rubric distillation | `/distill-conventions` (batched convention-file updates) |

ADP's loop already covers the RLVR half. The preference logs close the RLHF-analog half — without adding a second agent, a vector DB, or an external training run.

---

## What Gets Committed vs. Gitignored

Start ignored. Promote once stable.

Initial setup — add to `.gitignore`:

```
.adp/preferences/
```

Once the schemas stabilize (after ~2 weeks of use), promote by removing the gitignore entry. Commit the files. They become team-shared learning, reviewable in PRs. Cross-developer preference accumulation.

Alternative: keep `rejections.jsonl` gitignored (noisy, per-developer) and commit only `simplifications.jsonl` (curated, post-review).

---

## Cost Control

- **Few-shot retrieval** is capped at 3 entries per review. ~600 tokens per iteration worst case.
- **Escalation check** in `/new-insight` is one `jq` call on startup. Zero extra agent turns.
- **Distillation** runs on demand, not every iteration.

Measured budget impact on a review iteration: <5% tokens. No change to the verifier gates. No change to `loop.sh`.

---

## When to Skip This Layer

- Solo prototype project, <10 iterations of `loop.sh` total. The log cannot accumulate enough signal to be worth the overhead.
- Projects where every feature is genuinely unconnected. Preference signal does not transfer.
- When the team does not yet have stable conventions — premature accumulation will lock in noise.

Build up to it: run plain ADP first, graduate to preference accumulation once you see the same rejection twice.
