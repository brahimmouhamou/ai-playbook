---
name: distill-conventions
description: "This skill should be used when the user asks to 'distill preferences', 'update conventions from feedback', 'review the preference log', 'turn rejections into rules', or periodically batches accumulated preference signals from `.adp/preferences/*.jsonl` into new or updated convention files under `specs/conventions/` and `docs/conventions/`."
---

# /distill-conventions

Cluster the durable preference logs into proposed convention updates. This is the offline rubric-distillation step — the RLHF-analog equivalent of turning accumulated preference pairs into SFT material (convention docs that load on demand into every future run).

## When to Run

- Every 2-4 weeks on an active project.
- After a feature lands that triggered many REJECTs.
- Before a major refactor, to understand which conventions keep breaking.
- Never as part of `loop.sh`. This is a batch, human-reviewed step.

## Inputs

- `.adp/preferences/rejections.jsonl` — REJECT entries from the review agent
- `.adp/preferences/simplifications.jsonl` — before/after pairs from the simplify agent
- Existing `specs/conventions/*.md` and `docs/conventions/*.md` — what's already codified

## Procedure

### Step 1: Gate

If neither JSONL file exists or both are empty, exit: *"No preferences to distill yet."*

If an argument is provided, treat it as a tag filter (e.g. `/distill-conventions pagination` → only cluster entries tagged `pagination`).

### Step 2: Cluster rejections by tag

Multi-tag rejections must be counted in every tag they carry. Use explode-then-group, not `group_by(.tags[])` (which mis-partitions and non-deterministically labels groups):

```bash
jq -s '
  [.[] | . as $e | .tags[] | {tag: ., entry: $e}]
  | group_by(.tag)
  | map({
      tag: .[0].tag,
      count: length,
      features: (map(.entry.feature) | unique),
      reasons: (map(.entry.reason) | unique),
      samples: (map(.entry) | .[0:3])
    })
  | sort_by(-.count)
' .adp/preferences/rejections.jsonl
```

Drop clusters with `count < 3` or `(features | length) < 2` (below the escalation threshold — not yet a pattern).

### Step 3: Cluster simplifications by convention

```bash
jq -s 'group_by(.convention) | map({convention: .[0].convention, count: length, rationales: (map(.rationale) | unique), samples: .[0:3]})' .adp/preferences/simplifications.jsonl
```

Groups by which convention each refactor was aligned to. Reveals which conventions are repeatedly applied manually — good candidates for stronger automation (hooks) or clearer wording.

### Step 4: Map clusters to target files

For each rejection cluster:

- Can a user observe this issue? → `specs/conventions/<slug>.md`
- Only verifiable by reading code or running tools? → `docs/conventions/<slug>.md`
- Already covered by a convention but still happening? → strengthen wording OR propose a hook (see Step 5).

For each simplification cluster: usually reinforces an existing `docs/conventions/` file. Update examples or clarify rules.

### Step 5: Detect hook candidates

If the same rejection tag appears ≥5 times AND the check could be deterministic (lint rule, grep pattern, tsc rule), flag it as a **hook candidate**, not a convention update. Conventions are advisory; hooks are non-negotiable. From `playbook/04-rules-and-progressive-disclosure.md`:

> Rules that absolutely cannot be violated → hooks.

Example: if `tenant-isolation` has 8 rejections across 4 features, propose a PostToolUse hook that greps edited files for unscoped queries, not another paragraph in a markdown file.

### Step 6: Draft the output

Present the human with:

1. **Top 5 clusters** ranked by frequency.
2. For each: target file (create or update), proposed text (spec/product-convention/tech-convention voice per `/new-insight` rules), sample entries from the log as justification.
3. **Hook candidates** separately. Propose the hook command, not the markdown.
4. **Coverage gaps** — tags that appear in the log but match no existing convention file.

Do NOT apply changes. Output a review document. The human applies approved items manually or via a follow-up `/new-insight` call.

### Step 7: Do not modify the preference logs

The logs are append-only. Never delete entries after distillation. Re-running `/distill-conventions` over the same data should produce the same clusters. If a convention has been applied and issues stop appearing, the tag frequency will naturally fade over time as new entries outweigh old ones.

## Scope Boundary

- Do NOT edit convention files directly. Human approval required.
- Do NOT mutate `.adp/preferences/*.jsonl`.
- Do NOT regenerate prd.json or touch feature specs. That is `/new-insight` and `/new-spec` territory.

## Output Format

```markdown
# Preference Distillation — <date>

## Inputs
- rejections.jsonl: <N> entries, <M> unique tags, <F> features
- simplifications.jsonl: <N> entries, <C> conventions referenced

## Top Clusters

### 1. <tag> (<count> rejections, <N> features)
**Proposed action:** update `<path>` with:
> <draft text>

**Justification samples:**
- <feature>/<story>: <reason>
- <feature>/<story>: <reason>

## Hook Candidates

### <tag>
- Frequency: <N>
- Deterministic check: `<command or pattern>`
- Recommended: add to `.claude/settings.json` as PostToolUse

## Coverage Gaps

- Tag `<slug>` has <N> entries but no matching convention file. Consider creating `docs/conventions/<slug>.md`.
```

## Examples

Run-level examples of what the clustering should surface:

- **Convention update:** tag `error-handling` has 6 rejections across 3 features, no convention file exists → propose creating `specs/conventions/error-handling.md` with the 3 most common reasons as acceptance criteria.
- **Hook upgrade:** tag `tenant-isolation` has 8 rejections, all about missing `tenantId` in queries → propose PostToolUse hook that greps for query builders without tenant filters.
- **Simplification pattern:** 12 simplification entries reference `docs/conventions/clean-architecture.md` with rationale "extracted duplicated validation" → the convention needs a clearer rule about where validation lives, or a hook that rejects inline validation.
