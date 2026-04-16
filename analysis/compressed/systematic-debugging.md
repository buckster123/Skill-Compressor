---
name: systematic-debugging
description: 4-phase root cause investigation for any bug/test failure/unexpected behavior. NO fixes without understanding first.
version: 1.1.0
author: Hermes Agent (adapted from obra/superpowers)
license: MIT
metadata:
  hermes:
    tags: [debugging, troubleshooting, problem-solving, root-cause, investigation]
    related_skills: [test-driven-development, writing-plans, subagent-driven-development]
---

# Systematic Debugging

NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST — if Phase 1 ¬complete, ¬propose fixes.

**When:** test failures · prod bugs · unexpected behavior · perf problems · build failures · integration issues
**Especially:** under time pressure · "just one quick fix" · already tried multiple fixes · previous fix failed · ¬fully understand issue
**¬skip when:** issue seems simple · in a hurry (rushing ⇒ rework) · someone wants it NOW (systematic > thrashing)

## Phase 1: Root Cause Investigation

BEFORE attempting ANY fix:

### 1. Read Error Messages Carefully
¬skip errors/warnings — they often contain the exact solution. Read stack traces completely. Note line numbers, file paths, error codes.

Use `read_file` on relevant source. Use `search_files` to find error string in codebase.

### 2. Reproduce Consistently
Can you trigger reliably? Exact steps? Every time? If ¬reproducible → gather more data, ¬guess.

```bash
pytest tests/test_module.py::test_name -v
pytest tests/test_module.py -v --tb=long
```

### 3. Check Recent Changes
What changed? Git diff, recent commits, new deps, config changes.

```bash
git log --oneline -10
git diff
git log -p --follow src/problematic_file.py | head -100
```

### 4. Gather Evidence (Multi-Component Systems)
WHEN system has multiple components (API → service → database, CI → build → deploy):
For EACH component boundary: log data in/out, verify env/config propagation, check state at each layer.
Run once → gather evidence showing WHERE it breaks → investigate that component.

### 5. Trace Data Flow
WHEN error is deep in call stack: where does bad value originate? Trace upstream to source. Fix at source, ¬symptom.

```python
search_files("function_name(", path="src/", file_glob="*.py")
search_files("variable_name\\s*=", path="src/", file_glob="*.py")
```

### Phase 1 Completion Checklist
- [ ] Error messages fully read and understood
- [ ] Issue reproduced consistently
- [ ] Recent changes identified and reviewed
- [ ] Evidence gathered (logs, state, data flow)
- [ ] Problem isolated to specific component/code
- [ ] Root cause hypothesis formed

STOP: ¬proceed to Phase 2 until you understand WHY.

## Phase 2: Pattern Analysis

1. **Find working examples** — locate similar working code in same codebase
2. **Compare against references** — read reference impl COMPLETELY, ¬skim
3. **Identify differences** — list every difference, ¬assume "that can't matter"
4. **Understand dependencies** — other components, settings, config, env, assumptions

## Phase 3: Hypothesis and Testing

1. **Single hypothesis** — "X is root cause ∵ Y" — be specific
2. **Test minimally** — smallest possible change, one variable at a time, ¬fix multiple things
3. **Verify** — worked → Phase 4; failed → new hypothesis, ¬add more fixes on top
4. **When unknown** — say "I don't understand X", ask user, research more

## Phase 4: Implementation

1. **Create failing test** — simplest repro, automated, MUST have before fixing (use `test-driven-development` skill)
2. **Single fix** — address root cause, ONE change, ¬"while I'm here" improvements, ¬bundled refactoring
3. **Verify fix:**
```bash
pytest tests/test_module.py::test_regression -v
pytest tests/ -q
```
4. **If fix fails — Rule of Three:** count fixes tried. <3 → return to Phase 1. **≥3 → STOP, question architecture (step 5).** ¬attempt fix #4 without architectural discussion.
5. **3+ fixes failed → question architecture.** Signs: each fix reveals new shared state/coupling elsewhere · fixes require massive refactoring · each fix creates new symptoms. Ask: is pattern fundamentally sound? Should we refactor arch vs. continue fixing symptoms? Discuss with user before more fixes. This is ¬failed hypothesis — this is wrong architecture.

## Red Flags — STOP, return to Phase 1

- "Quick fix now, investigate later"
- "Just try changing X"
- "Add multiple changes, run tests"
- "Skip the test, manually verify"
- "It's probably X, let me fix that"
- "¬fully understand but might work"
- "Pattern says X but I'll adapt differently"
- "Here are the main problems: [lists fixes without investigation]"
- Proposing solutions before tracing data flow
- **"One more fix" (already tried 2+)**
- **Each fix reveals new problem elsewhere**

**3+ fixes failed → question architecture (Phase 4 step 5).**

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "Simple, ¬need process" | Simple bugs have root causes; process is fast |
| "Emergency, no time" | Systematic > guess-and-check thrashing |
| "Try this first, then investigate" | First fix sets pattern; do it right |
| "Test after confirming fix" | Untested fixes ¬stick; test first proves it |
| "Multiple fixes saves time" | Can't isolate what worked; causes new bugs |
| "Reference too long, I'll adapt" | Partial understanding ⇒ bugs; read completely |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause |
| "One more fix" (after 2+ failures) | 3+ failures = architectural problem; ¬fix again |

## Quick Reference

| Phase | Activities | Success |
|---|---|---|
| 1. Root Cause | read errors, reproduce, check changes, gather evidence, trace data flow | understand WHAT+WHY |
| 2. Pattern | find working examples, compare, identify diffs | know what's different |
| 3. Hypothesis | form theory, test minimally, one var at a time | confirmed or new hyp |
| 4. Implementation | regression test, fix root cause, verify | resolved, all tests pass |

## Hermes Agent Integration

**Investigation tools (Phase 1):** `search_files` (error strings, fn calls, patterns) · `read_file` (source w/ line numbers) · `terminal` (tests, git history, repro) · `web_search`/`web_extract` (error msgs, lib docs)

**With `delegate_task`** for multi-component debugging:
```python
delegate_task(
    goal="Investigate why [specific test/behavior] fails",
    context="""
    Follow systematic-debugging skill:
    1. Read the error message carefully
    2. Reproduce the issue
    3. Trace the data flow to find root cause
    4. Report findings — do NOT fix yet

    Error: [paste full error]
    File: [path to failing code]
    Test command: [exact command]
    """,
    toolsets=['terminal', 'file']
)
```

**With `test-driven-development`:** write test reproducing bug (RED) → debug systematically for root cause → fix root cause (GREEN) → test proves fix + prevents regression

## Impact

Systematic: 15-30min to fix · first-time fix rate 95% · new bugs near zero
Random fixes: 2-3hr thrashing · first-time fix rate 40% · new bugs common
