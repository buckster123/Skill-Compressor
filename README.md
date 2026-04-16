# Skill-Compressor

A compression skill for AI agent instruction documents. Shrinks `SKILL.md` files, system prompts, and similar structured instructions by 40–55% while preserving every tool name, parameter, threshold, enum value, and edge case.

Built after compressing a real 2,700-word skill down to 1,300 words with zero information loss, then reverse-engineering what made that work.

## What it does

AI instruction documents accumulate verbose English prose that's cheap for humans to read but expensive to load into context every turn. A skill that costs 5,000 tokens per activation costs the same whether it runs once or a thousand times.

This skill takes a verbose input and produces a token-dense equivalent by:

- **Factoring repeated prefixes.** `mcp_cerebro_remember`, `mcp_cerebro_recall`, `mcp_cerebro_associate` × 40 more → declare the prefix once, drop from body.
- **Compressing tables to telegraphic form.** "A is about the same concept as B" → "same concept".
- **Replacing prose connectives with symbols.** `→ ⇒ ∴ ∵ ≈ ≡ ∈ ¬ ⇔` are unambiguous to AI readers and 3–8 characters shorter than their English equivalents.
- **Dropping pedagogical scaffolding.** "Rule of thumb:", "Key insight:", "It's worth noting that" — the adjacent content already makes the point.
- **Abbreviating domain-heavy terms.** memory→mem, procedure→proc, directory→dir, when they appear 10+ times.
- **Auditing afterward.** Every tool name, number, enum value, and warning gets programmatically verified against the original. If anything was lost, the skill fixes it before presenting.

## What it preserves (inviolate)

- Every tool / function / API name
- Every numeric value — weights, thresholds, percentages, times
- Every enum member — type values, level names, role names
- Every parameter name in code examples
- Every "don't do this" warning (even when dropping surrounding prose)
- YAML frontmatter required fields
- Pedagogical contrast examples (Bad/Good pairs) — the verbosity *is* the lesson

## What it doesn't compress

The skill actively refuses or flags these:

- Writing style / voice / tone guides — the prose is the content
- Skills under 100 lines — gains don't exceed ~20%
- Skills that are mostly code or mostly worked examples
- Already-compressed or naturally terse inputs

For these, light editing is offered instead of aggressive structural compression.

## Typical results

Tested against 3 diverse Hermes Agent skills using tiktoken (cl100k_base) and a 24-question semantic equivalence evaluation:

| Input | Original | Compressed | Reduction |
|---|---|---|---|
| CerebroCortex (46 MCP tools, tables, enums, weights) | 4,956 tokens | 3,715 tokens | 25.0% |
| Systematic Debugging (4-phase methodology, pedagogical) | 2,498 tokens | 1,725 tokens | 30.9% |
| GitHub PR Workflow (CLI commands, API URLs, code blocks) | 2,895 tokens | 2,308 tokens | 20.3% |
| **Total** | **10,349** | **7,748** | **25.1%** |

### Semantic equivalence

Compressed skills were tested with 24 technical questions (numeric precision, enum completeness, API knowledge, procedural workflows) answered by Claude Sonnet using each version as context:

| Metric | Value |
|---|---|
| Average original score | 9.7/10 |
| Average compressed score | 9.6/10 |
| Equivalent answers | 18/24 (75%) |
| Original better | 3/24 (12.5%) |
| Compressed better | 3/24 (12.5%) |
| Both wrong | 0/24 (0%) |

**Zero loss on:** numeric precision, enum completeness, range specifications, API knowledge.

See [`analysis/REPORT.md`](analysis/REPORT.md) for the full evidence report, methodology, per-question breakdowns, and system-wide projections for Hermes Agent.

## Beyond skills

The compression patterns apply to any structured AI instruction document:

- Custom system prompts
- Agent instructions / persona definitions
- Tool manifests and API reference docs
- Cursor / Windsurf / Copilot rule files
- MCP server descriptions
- Long in-context reference materials

Anything where you're paying tokens every turn to load English prose written for a human reader.

## Installation

### As a Claude skill

Drop `skill-compressor.skill` into your skills directory. It'll activate when you paste in a skill, system prompt, or instruction document and ask to "compress", "shrink", "tighten", "densify", or "reduce tokens".

### As a reference

The skill is three plain markdown files. You can also just read them and apply the techniques by hand, or feed them to any AI agent as context:

- `SKILL.md` — decision-making, workflow, invariants, reporting template
- `references/compression-rules.md` — 13 detailed rules with before/after examples
- `references/audit-checklist.md` — runnable Python audit pattern + manual spot-checks

## How it works

The skill is structured as a decision tree, not a substitution list.

**Step 1 — Classify.** Every line of a skill is one of these content types: tool names, numeric specs, enum values, descriptive prose, pedagogical framing, tables, code examples, contrast examples, section intros. Each has its own rule.

**Step 2 — Apply type-specific rules.** Tool names preserve exactly. Prose compresses aggressively. Pedagogical framing deletes. Contrast examples stay verbatim. See `compression-rules.md` for the full catalog.

**Step 3 — Audit.** A Python pass extracts every tool name, number, and enum value from the original and verifies each appears in the compressed output. Warnings are counted. Headings are tracked. Anything missing gets restored before presenting.

**Step 4 — Report.** Before/after numbers, audit results with ✓/✗ per invariant class, and a list of techniques applied.

The audit is what separates compression from damage. Most "make this shorter" attempts skip verification and silently drop edge cases. This one doesn't.

## Example: the symbolic vocabulary

These symbols are used consistently across compressed outputs. They're unambiguous to AI readers and save 3–8 characters each vs their English equivalents:

```
→    leads to / causes / produces
⇒    therefore / implies (stronger than →)
∴    therefore (conclusion)
∵    because
≡    equivalent to / alias of
≈    approximately
∈    element of / in set
⇔    if and only if / bidirectional
¬    not
×    times / multiplied by
↑ ↓  increase / decrease
≥ ≤  at least / at most
t½   half-life
∞    permanent / unbounded
```

The skill doesn't invent new symbols — if a concept doesn't map to one of these, a short English word is used instead.

## Caveats

**Readability trade-off.** A compressed skill is less pleasant for a human to read. That's the point — the intended reader is an AI loading it as context, not a person skimming the repo. If a skill is meant to onboard new humans to a system, keep the verbose version.

**Token estimates are approximate.** Without a live tokenizer, estimates use ~3.9 chars/token for English prose and ~3.5 for dense symbolic text. Real token counts can vary ±10%. The skill reports ranges, not false precision.

**Regex audit patterns need tuning.** The audit script's regex for extracting tool names assumes common conventions (`mcp_foo_*`, backticked identifiers in tables). Unusual naming schemes need a quick adjustment to the pattern. The skill flags this.

**This isn't semantic compression.** The skill doesn't summarize content or restate meaning in fewer words. It removes redundancy and rewrites connective tissue. If your input has genuinely five paragraphs of unique information, it'll still have roughly five paragraphs of unique information afterward — just denser.

## License

MIT

## Credit

Built with Claude (Anthropic). The initial test case and primary validation was a 46-tool brain-analogous memory system skill called CerebroCortex.
