# Audit Checklist

Run this after compression, before presenting. Compression without audit is deletion with confidence.

## Quick audit (all skills)

```bash
# Basic size comparison
echo "== ORIGINAL =="  ; wc -l -w -c <original>
echo "== COMPRESSED ==" ; wc -l -w -c <compressed>
```

## Programmatic invariant check

Use this Python pattern. Adapt the regexes to the specific skill's conventions.

```python
import re
orig = open("<original>").read()
comp = open("<compressed>").read()

# 1. Tool / function names — customize regex for the skill's naming pattern
# Examples: mcp_foo_*, google_drive_*, Salesforce:*, function calls, etc.
tool_pattern = r'mcp_\w+_(\w+)'  # adjust this
orig_tools = set(re.findall(tool_pattern, orig))
# Tools in compressed may have prefix factored out — check for bare names
comp_identifiers = set(re.findall(r'`([a-z][a-z_0-9]+)`', comp))
missing_tools = orig_tools - comp_identifiers
print(f"Tools: {len(orig_tools)} in orig, missing in comp: {missing_tools}")
assert not missing_tools, f"LOST TOOLS: {missing_tools}"

# 2. Numeric specs — ANY decimal or integer surrounded by context
# Pull both and verify every orig number appears in comp
orig_nums = set(re.findall(r'\b\d+(?:\.\d+)?\b', orig))
comp_nums = set(re.findall(r'\b\d+(?:\.\d+)?\b', comp))
lost_nums = orig_nums - comp_nums
# Allow false positives (e.g., version numbers that got simplified)
# but print them for human review
print(f"Numbers in orig not in comp: {sorted(lost_nums)}")

# 3. Enum values — typically backticked lowercase identifiers in tables
# Extract any backticked token that appears in a table row
orig_enums = set(re.findall(r'\| `([a-z_]+)` \|', orig))
comp_enums = set(re.findall(r'\| `([a-z_]+)` \|', comp))
missing_enums = orig_enums - comp_enums
print(f"Enum values missing: {missing_enums}")
assert not missing_enums, f"LOST ENUMS: {missing_enums}"

# 4. Warnings / "don't" statements — these are easy to lose
orig_warnings = len(re.findall(r'(?i)(don\'t|never|avoid|warning|pitfall|caution|⚠)', orig))
comp_warnings = len(re.findall(r'(?i)(don\'t|never|avoid|warning|pitfall|caution|⚠|¬)', comp))
print(f"Warnings: orig={orig_warnings}, comp={comp_warnings}")
# Expect comp to be close to orig; significant drop means you lost warnings

# 5. Headings — structure should survive
orig_headings = [l.strip() for l in orig.split('\n') if l.startswith('#')]
comp_headings = [l.strip() for l in comp.split('\n') if l.startswith('#')]
print(f"Headings: {len(orig_headings)} → {len(comp_headings)}")
# Some consolidation expected; drop >30% needs review
```

## Manual spot-checks

Numbers of things to verify by eye, not just regex:

1. **YAML frontmatter:** `name`, `version`, `author`, `license` unchanged. `description` tightened but contains all trigger phrases.

2. **Aliases:** If the skill mentions `X ≡ Y` (aliases), both names still present.

3. **Code example parameter names:** Open a random code example in compressed; every parameter name also appears in the original example.

4. **Numeric blends / formulas:** If the skill has a weighted formula (e.g., "35% vector + 30% recency + ..."), the compressed version has an equivalent formula with exact weights.

5. **Range specifications:** Bands like "0.8-1.0 critical / 0.6-0.8 important" — all bands and boundaries preserved.

6. **Layer / stage sequences:** If skill has "A → B → C → D", all stages present in same order.

## Red flags that mean STOP and redo

- Compression ratio > 65% — almost always means information loss
- Any tool name missing from compressed
- Any enum value missing
- Pitfall count drops by > 20%
- A numeric threshold changed (check exact values, not just presence)
- YAML frontmatter lost required fields

## Reporting format

Present audit results inline with compression numbers:

```
**Compression results:**
- Chars: 20,186 → 10,386 (48.5% reduction)
- Words: 2,735 → 1,288 (52.9% reduction)
- ~Tokens: ~5,176 → ~2,967 (~43% reduction)

**Audit:**
- ✓ 40/40 tool names preserved
- ✓ 6/6 memory types preserved
- ✓ 9/9 link types with exact weights
- ✓ All numeric specs preserved (recall blend, thresholds, times)
- ✓ All 10 pitfalls preserved
- ✓ YAML frontmatter required fields intact

**Techniques applied:**
- Factored `mcp_cerebro_` prefix (60× → 1 declaration)
- Table cells → telegraphic fragments
- Prose connectives → symbols (→ ⇒ ∴ ∵ ≈ ≡)
- Dropped pedagogical framing, kept warnings
- Collapsed parallel "when-to-use" bullets
```

If any audit line is ✗, fix before presenting. Don't hand the user a damaged skill.

## Token estimation (no tokenizer available)

If you don't have tiktoken / an actual tokenizer, estimate tokens from chars:

- **English prose:** ~3.9 chars/token
- **Dense symbolic text:** ~3.5 chars/token (fewer chars per token because unicode symbols and rare abbreviations tokenize less efficiently individually, but there are far fewer of them — net win)
- **Code:** ~3.0 chars/token

The estimate is approximate — don't quote it to 3 decimal places. A range like "~43% token reduction" is honest; "43.2861%" is not.
