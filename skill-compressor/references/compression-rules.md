# Compression Rules — Detailed Catalog

Read this before compressing. Each rule has a *when to apply* condition. Don't apply rules blindly.

## Rule 1: Factor repeated prefixes

**When:** Same prefix appears 5+ times (tool names, API paths, class prefixes).

**How:** Declare once at the top, drop from body.

```
ORIGINAL:
| `mcp_cerebro_remember` | Store |
| `mcp_cerebro_recall` | Search |
| `mcp_cerebro_associate` | Link |
... ×40 more

COMPRESSED:
### Tools (prefix `mcp_cerebro_` dropped below; all tools carry it)
| tool | fn |
|---|---|
| `remember` | store |
| `recall` | search |
| `associate` | link |
```

**Savings:** ~14 chars × 40 rows = ~560 chars on just the prefix, before any other compression.

## Rule 2: Table cells → telegraphic

**When:** Table cells are full sentences with articles, verbs, subjects.

**How:** Drop articles ("the", "a", "an"). Drop weak verbs ("is", "was", "are"). Drop subject when it's the row key. Use fragments.

```
ORIGINAL:
| `causal` | 0.9 | A caused B | Bug → Outage |
| `semantic` | 0.8 | A is about same concept as B | Two memories about PostgreSQL |

COMPRESSED:
| `causal` | 0.9 | A→B | bug→outage |
| `semantic` | 0.8 | same concept | 2× pg mems |
```

**Keep:** the specific term that's the point of the cell.
**Drop:** grammatical connective tissue.

## Rule 3: Collapse parallel tables

**When:** A "When to use" column is mostly rephrasing the "What it does" column.

**How:** Merge to one column that captures both.

```
ORIGINAL (3 cols):
| Tool | What it does | When to use |
|------|-------------|-------------|
| `send_message` | Message another agent | Cross-agent communication |
| `check_inbox` | Read messages from agents | Check if anyone sent you something |

COMPRESSED (2 cols):
| tool | fn |
|---|---|
| `send_message` | agent msg |
| `check_inbox` | read agent msgs |
```

**Warning:** If the "When to use" column has *different* information from "What it does" (e.g., frequency, context cues), keep both.

## Rule 4: Symbols replace connectives

Apply the symbolic vocabulary from SKILL.md. Common substitutions:

| From | To |
|---|---|
| "leads to" / "results in" / "causes" | `→` |
| "therefore" / "so" / "thus" | `∴` |
| "because" / "since" | `∵` |
| "approximately" / "around" / "roughly" | `≈` |
| "equivalent to" / "same as" / "alias for" | `≡` |
| "is a member of" / "is one of" | `∈` |
| "not" / "avoid" / "don't" | `¬` |
| "if and only if" / "exactly when" | `⇔` |
| "implies" / "means that" | `⇒` |
| "increases" / "goes up" | `↑` |
| "decreases" / "goes down" | `↓` |

**Don't** use symbols where English is already short ("and" → `∧` is a wash; "or" → `∨` is a wash).

## Rule 5: Delete pedagogical scaffolding

**Delete these phrases unless they carry unique information:**

- "Rule of thumb:"
- "Key insight:"
- "Important to note:"
- "One thing to remember:"
- "As mentioned above,"
- "It's worth noting that"
- "Note that"
- "Keep in mind:"
- "Pro tip:"
- Section-opening restatements of the heading

**Keep these** (they're warnings, not framing):
- "Don't..." / "Never..."
- "Warning:"
- "Pitfall:"
- "Caution:"
- "⚠"

## Rule 6: Delete section intros that restate the heading

**When:** First sentence after a heading paraphrases the heading.

```
ORIGINAL:
## Episodes: Recording Event Sequences
Episodes bind memories into temporal narratives — "what happened during X."

COMPRESSED:
## Episodes (temporal narratives)
```

Fold any essential qualifier from the intro into the heading parenthetical. Delete the rest.

## Rule 7: Code examples — keep signatures, drop narration

**When:** Code example is wrapped in prose that narrates what the code does.

**How:** Keep the code. Drop prose. Add inline comment only if the code alone isn't self-documenting.

```
ORIGINAL:
After storing related memories, you can link them like so:

id_a = remember("Deploy failed due to missing env var")
id_b = remember("Added env validation to CI pipeline")
associate(source_id=id_a, target_id=id_b, link_type="causal",
          evidence="The failure led to the fix", weight=0.9)

This creates a causal link between the failure and its fix.

COMPRESSED:
### Pattern
a=remember("deploy fail: missing env")
b=remember("added env validation to CI")
associate(a, b, "causal", evidence="fail→fix", weight=0.9)
```

**Preserve:** parameter names (they're API surface). 
**Drop:** narration, type annotations on var names, verbose string content inside examples.

## Rule 8: Fold bullet lists into comma-separated runs

**When:** Short bullets (each 1-5 words) that are all the same kind of thing.

```
ORIGINAL:
roles:
- `event` - Something that happened (default)
- `context` - Background information, observations
- `outcome` - The result of actions taken
- `reflection` - Lessons learned, insights, feelings

COMPRESSED:
roles: `event`(default) · `context`(bg/obs) · `outcome`(result) · `reflection`(lesson)
```

**Keep as bullets** when:
- Items are long (full sentences)
- Items have sub-structure
- Ordering is critical (numbered steps)

## Rule 9: Collapse "When to X" lists

**When:** A bulleted "When to use this" list has 3-5 items.

**How:** Single line with `·` separators.

```
ORIGINAL:
**When to dream:**
- Weekly maintenance (can be scheduled via cron)
- After large knowledge imports (ingest_file)
- When the user asks to "clean up" or "consolidate" memories
- After intensive multi-session projects

COMPRESSED:
**When:** weekly cron · post-bulk-ingest · user says "consolidate" · post-intensive-project
```

## Rule 10: Preserve pedagogical contrast examples VERBATIM

**When:** Skill uses "Bad: X / Good: Y" pattern to teach quality.

**DO NOT compress these.** The contrast *is* the lesson. A compressed "good" example that's as short as the "bad" example destroys the teaching value.

```
KEEP AS-IS:
BAD:  remember("thing about the database")
GOOD: remember(
        "PostgreSQL connection pool exhaustion caused 502 errors in prod.
         Root cause: default max_connections=100, app opens 20 per worker × 8 workers = 160.
         Fix: set max_connections=300 in postgresql.conf",
        memory_type="semantic",
        tags=["postgresql", "connection-pool", "production-incident"],
        salience=0.7, visibility="shared")
```

Same for long worked-example workflows where each step shows a distinct technique.

## Rule 11: Abbreviate repeated domain terms

**When:** A multi-syllable domain term appears 10+ times.

Common productive abbreviations:
- memory → mem / mems
- procedure → proc / procs
- documentation → docs
- directory → dir
- environment → env
- configuration → config
- development → dev
- production → prod

**Don't** abbreviate:
- Words with only 2 syllables (little savings)
- Words where the abbreviation is ambiguous ("ops" → operations? ops-team?)
- Words that appear 1-3 times

## Rule 12: Collapse YAML frontmatter description

The `description` field is often verbose — it's written for a human deciding whether to load the skill. Compress it like any other prose, but **keep all trigger phrases** (they're what the skill-selector looks for).

```
ORIGINAL:
description: Use CerebroCortex — a brain-analogous AI memory system with 46 MCP tools. Covers when/how to remember, recall, associate, record episodes, track procedures, manage intentions, run dream consolidation, and communicate across agents. Load this skill whenever you see mcp_cerebro_* tools or need to work with persistent memory.

COMPRESSED:
description: CerebroCortex brain-analogous memory (46 MCP tools). Remember/recall/associate/episodes/procedures/intentions/dream-consolidation/agent-messaging. Load on `mcp_cerebro_*` or persistent-memory needs.
```

## Rule 13: When NOT to compress

Flag the user if any of these apply to large portions of the skill:

- Writing style / voice / tone guide (the prose is the skill)
- Contains long verbatim examples that need to stay verbatim (quoted text, sample outputs)
- Already compressed or in a terse format
- Under 100 lines — gains won't exceed ~20%, may not be worth the risk
- Uses extensive numbered procedures where step ordering is load-bearing

In these cases, offer light editing instead (removing pure duplication, fixing verbosity in individual sentences) rather than aggressive structural compression.

## Order of operations

When compressing, apply rules in this order for best results:

1. Factor repeated prefixes (Rule 1) — do this first, it's a prerequisite for table compression
2. Delete pedagogical scaffolding (Rule 5) and section intros (Rule 6)
3. Compress tables (Rules 2, 3)
4. Compress code examples (Rule 7)
5. Fold bullets and "when" lists (Rules 8, 9)
6. Apply symbolic substitutions (Rule 4)
7. Apply abbreviations (Rule 11)
8. Compress frontmatter description (Rule 12)
9. Audit (see audit-checklist.md)

Doing it in this order means later rules operate on already-shortened text and waste less effort.
