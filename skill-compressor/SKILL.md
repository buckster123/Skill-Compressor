---
name: skill-compressor
description: Compress a skill file to reduce token count while preserving all technical information. Use whenever the user drops in a SKILL.md, skill file, system prompt, or agent instruction document and asks to "compress", "shrink", "densify", "reduce tokens", "tighten", or "make smaller" — or when they mention a skill is "too long", "bloated", or "costs too much to load". Applies structural compression (collapsing repeated prefixes, telegraphic tables, symbolic notation) to English-prose skills. Target 40-55% token reduction with zero information loss on technical content.
license: MIT
metadata:
  hermes:
    version: "1.0.0"
    author: Hermes
    tags: [skill-tooling, compression, tokens, optimization, meta]
---

# Skill Compressor

Compresses verbose English-prose skills into token-efficient AI-readable form. Preserves every tool name, parameter, number, enum, threshold, and edge case. Deletes pedagogical scaffolding, repeated prefixes, and prose that restates what's obvious from structure.

**Typical result:** 40-55% token reduction. Higher ratios (60%+) usually indicate information loss — audit carefully.

## When this skill applies (and when it doesn't)

**Good candidates** (high compression yield, safe):
- Technical reference skills with many tables, tool lists, enum definitions
- Skills with repeated prefixes (`mcp_foo_*`, `anthropic_*`, `google_drive_*`)
- Skills heavy in "rule of thumb", "key insight", "when to use" pedagogical framing
- Skills with parallel structure (tier tables, type tables, level tables)

**Poor candidates** (skip or warn user):
- Writing style / tone guides — the prose *is* the content
- Skills <100 lines — compression overhead won't pay off, just tighten lightly
- Skills that are mostly code or mostly examples — examples are the content
- Skills whose value is teaching a human, not instructing an AI

If the user drops in a skill that's a poor candidate, say so before compressing. Offer light editing instead.

## Workflow

```
1. READ  → measure baseline (wc -l -w -c), classify content types
2. PLAN  → tell user what you'll do, flag any poor-candidate sections
3. COMPRESS → apply type-specific rules (see references/compression-rules.md)
4. AUDIT → programmatically verify invariants preserved (references/audit-checklist.md)
5. REPORT → show before/after numbers + audit results + present file
```

Read `references/compression-rules.md` for the full technique catalog before starting. Read `references/audit-checklist.md` when done — don't skip the audit, it's what separates compression from damage.

## Core principle: classify, then compress

Every line of a skill is one of these content types. Each has its own rule.

| Content type | Rule |
|---|---|
| Tool names / API signatures | **Preserve exactly.** Factor repeated prefixes to a header declaration. |
| Numeric specs (weights, thresholds, %, times) | **Preserve exactly.** Never round, never paraphrase. |
| Enum values (types, levels, roles) | **Preserve exactly.** All of them. |
| Descriptive prose | Compress aggressively — symbols, telegraphic phrasing, drop articles |
| Pedagogical framing ("Rule of thumb:", "Key insight:") | **Delete** — the adjacent content already makes the point |
| Tables | Keep structure; compress cells to telegraphic fragments |
| Code examples | Keep signatures + minimal comments; drop prose narration around them |
| Good vs Bad examples showing a pedagogical contrast | **Keep verbose** — the contrast *is* the point |
| Section intros restating the heading | **Delete** |

## Invariants — never lose these

Before calling done, verify via grep/audit:

1. Every tool/function/API name present in original appears in compressed form
2. Every numeric value (weights, %, thresholds, time units) exact match
3. Every enum member (type values, level names, role names) preserved
4. Every configuration parameter name preserved
5. Every filepath, URL, identifier preserved
6. Every edge case / exception / "don't do this" warning preserved (may be tersely worded but the information must be there)
7. The skill's YAML frontmatter fields preserved (can tighten the `description` but keep required fields)

See `references/audit-checklist.md` for a runnable audit script pattern.

## Symbolic vocabulary (load into compressed skill)

Use these consistently. They're unambiguous to AI readers:

```
→    leads to / causes / produces
⇒    therefore / implies (stronger than →)
∴    therefore (conclusion)
∵    because
≡    equivalent to / alias of
≈    approximately
∈    element of / in set
∉    not in
⇔    if and only if / bidirectional
¬    not
∨    or
∧    and
×    times / multiplied by
↑ ↓  increase / decrease
≥ ≤  at least / at most
t½   half-life
∞    permanent / unbounded
⊃    contains / superset
```

Don't invent new symbols — if a concept doesn't map to one of these, use a short English word.

## Abbreviation conventions

Apply consistently across the compressed output:

```
memory → mem (plural: mems)
procedure → proc
documentation → docs
directory → dir
environment → env
configuration → config
start of session → SOS
end of session → EOS
long-term → LT
working memory → WM
```

Declare non-obvious ones once at the top if the skill uses them heavily.

## Before/after example

```
ORIGINAL (82 chars):
**Rule of thumb:** Store memories as if writing a note for a colleague
who has no context.

COMPRESSED (52 chars):
∴ write as note to context-free colleague.
```

Same information; 37% shorter; no loss.

```
ORIGINAL (110 chars):
Over time, frequently-successful procedures rank higher in
find_relevant_procedures(). Failed ones get demoted.

COMPRESSED (47 chars):
Successful procs ↑ rank; failed ↓.
```

Same information; 57% shorter.

## The audit phase is not optional

Compression without audit is just deletion with confidence. After compressing:

1. Extract all tool names from original → verify each appears in compressed
2. Extract all numbers (regex `\d+\.?\d*`) → verify each appears in compressed
3. Extract all enum values (backticked lowercase words in tables) → verify preserved
4. Spot-check every "Don't" or "Warning" — easy to lose these when removing pedagogical framing

If any invariant fails, fix it before presenting. Show the user the audit results — this is how they know you didn't silently damage their skill.

## Reporting template

```
**Compression results:**
- Chars: X → Y (Z% reduction)
- Words: X → Y (Z% reduction)
- Estimated tokens: ~X → ~Y (~Z% reduction)

**Audit (all must pass):**
- ✓ N/N tool names preserved
- ✓ N/N numeric specs preserved
- ✓ N/N enum values preserved
- ✓ N/N edge-case warnings preserved

**Techniques applied:**
- [bullet list of the main structural moves]
```

If any audit line is ✗, show what was lost and fix before presenting.

## Common failure modes (self-check)

1. **Dropping a warning while cutting "pedagogical framing"** — warnings look like pedagogy but carry critical information. When in doubt, keep.
2. **Rounding a number** — "~20" and "approximately 20" compress to "≈20" not "~few dozen". Never paraphrase numbers.
3. **Consolidating two similar tools into one** — aliases ARE two separate tool names. Note the alias relationship but preserve both names.
4. **Losing a parameter name from a code example** — even if the example is verbose, the parameter names are part of the API surface.
5. **Over-compressing YAML frontmatter** — `description` can tighten, but `name`, `version`, `author` stay.
6. **Applying symbols inconsistently** — if you use `→` for "leads to", don't also use it for "then". Pick one meaning per symbol.
