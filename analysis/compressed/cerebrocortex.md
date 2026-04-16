---
name: cerebrocortex
description: CerebroCortex brain-analogous mem (46 MCP tools). Remember/recall/associate/episodes/procs/intentions/dream-consolidation/agent-messaging. Load on `mcp_cerebro_*` or persistent-mem needs.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [cerebrocortex, memory, mcp, brain, cognitive-architecture, multi-agent]
    related_skills: [native-mcp, hermes-memory-provider-plugin]
---

# CerebroCortex

Persistent, associative, emotionally-weighted mem that strengthens w/ use, decays w/ neglect, reorganizes during dream consolidation. Modeled after neuroscience, ¬ a vector DB.

**Load when:** `mcp_cerebro_*` tools · store/recall across sessions · link ideas · track episodes · manage TODOs · agent comms · mem maintenance.

## Mental Model

```
Input → Thalamus(gate) → Temporal Lobe(concepts) → Amygdala(emotion)
      → Association Cortex(link) → Hippocampus(episodes) / Cerebellum(procs)
      → Prefrontal Cortex(rank) → Neocortex(patterns)
      → Dream Engine(offline consolidation)
```

You ¬ manually trigger brain regions. `remember()` runs full encoding pipeline; `recall()` runs ranking pipeline. Regions are engines under the hood.

## The 46 Tools

### Prefix: all tools carry `mcp_cerebro_` (dropped below)

### Tier 1: Daily Drivers (80% usage)

| tool | fn |
|---|---|
| `remember` | store mem (full pipeline) — facts, decisions, discoveries |
| `recall` | semantic search — "what do I know about X?" |
| `associate` | link 2 mems — cause, support, context |
| `session_save` | save session summary — end of session |
| `session_recall` | load past session notes — start of session |
| `store_intention` | save TODO/reminder |
| `list_intentions` | check pending TODOs |
| `resolve_intention` | mark TODO done |
| `send_message` | msg another agent |
| `check_inbox` | read agent msgs |

### Tier 2: Knowledge Building

| tool | fn |
|---|---|
| `store_procedure` | save workflow/how-to |
| `find_relevant_procedures` | find matching procs |
| `record_procedure_outcome` | mark proc success/fail |
| `create_schema` | extract pattern/principle |
| `find_matching_schemas` | find relevant patterns |
| `episode_start` | begin recording sequence |
| `episode_add_step` | add step to episode |
| `episode_end` | finish episode w/ summary |
| `ingest_file` | import file → searchable mems |

### Tier 3: Graph Navigation

| tool | fn |
|---|---|
| `memory_neighbors` | get linked mems |
| `common_neighbors` | find shared connections |
| `find_path` | shortest path between mems |
| `get_memory` | full mem details + metadata |
| `update_memory` | edit content/tags/salience |
| `delete_memory` | permanently remove |
| `share_memory` | change visibility (private↔shared) |

### Tier 4: System & Analytics

| tool | fn |
|---|---|
| `dream_run` | run offline consolidation |
| `dream_status` | check dream engine status |
| `memory_health` | health report |
| `cortex_stats` | raw system stats |
| `memory_graph_stats` | graph structure metrics |
| `emotional_summary` | emotional tone breakdown |
| `list_agents` | show registered agents |
| `register_agent` | register new agent |

### Aliases

`memory_store` ≡ `remember` · `memory_search` ≡ `recall`

### Utility (MCP meta-tools)

`list_prompts` · `get_prompt` · `list_resources` · `read_resource`

---

## How to Remember Well

### Mem Types

Auto-classifies if omitted, but explicit is better:

| type | use for |
|---|---|
| `episodic` | events — "deploy failed at 3pm due to OOM" |
| `semantic` | facts — "PG max_connections default=100" |
| `procedural` | how-to — "deploy: test→build→push→apply" |
| `affective` | emotional context — "user frustrated after 3rd CI fail" |
| `prospective` | future intentions — "upgrade Python before Friday" |
| `schematic` | patterns — "services skipping staging have 3x more incidents" |

### Salience (0.0–1.0) — survival against decay

| range | meaning | examples |
|---|---|---|
| 0.8–1.0 | critical | user corrections, security, core prefs |
| 0.6–0.8 | important | key decisions, arch choices, milestones |
| 0.4–0.6 | normal | regular facts, session events, standard procs |
| 0.2–0.4 | low | transient context, temp notes |
| 0.0–0.2 | ephemeral | raw observations likely superseded |

Auto-estimation works well; override for critical items.

### Visibility

| level | who | use for |
|---|---|---|
| `shared` | all agents | facts, procs, cross-team knowledge |
| `private` | storing agent only | personal notes, drafts, agent state |
| `thread` | same conversation | conversation-scoped context |

### Tags — consistent conventions

Project: `project:cerebrocortex` · Category: `deployment`, `debugging` · Status: `resolved`, `in-progress`, `blocked` · Agent: auto-tagged, add `team:backend` etc.

### Good vs Bad Memory Storage

```
BAD:  remember("thing about the database")
GOOD: remember(
        "PostgreSQL connection pool exhaustion caused 502 errors in prod.
         Root cause: default max_connections=100, app opens 20 per worker × 8 workers = 160.
         Fix: set max_connections=300 in postgresql.conf",
        memory_type="semantic",
        tags=["postgresql", "connection-pool", "production-incident"],
        salience=0.7,
        visibility="shared"
      )
```

Store mems as if writing a note for a colleague w/ no context. Include what, why, resolution.

---

## Recall

### Basic
```
recall(query="database connection issues", top_k=5)
```

**4-signal blend:** 35% vector similarity · 30% ACT-R activation (recency+frequency) · 20% FSRS retrievability (spaced-repetition) · 15% salience

### Filtered
```python
recall(query="deploy", memory_types=["procedural"])
recall(query="security", min_salience=0.7)
recall(query="database", explain=True)          # score breakdown
recall(query="what we discussed", conversation_thread="thread_123")
recall(query="migration", context_ids=["mem_abc", "mem_def"])  # boost connected
```

### Strategy

| situation | approach |
|---|---|
| "what about X?" | `recall(query="X", top_k=10)` |
| "how to X?" | `find_relevant_procedures(concepts=["X"])` |
| "pattern for X?" | `find_matching_schemas(concepts=["X"])` |
| "what happened during X?" | `recall(query="X", memory_types=["episodic"])` + episodes |
| "connected to mem M?" | `memory_neighbors(memory_id="M")` |
| "feelings about X?" | `recall(query="X", memory_types=["affective"])` |

---

## Associative Network

### Link Types

Links enable spreading activation — found mem → linked mems get relevance boost.

| type | weight | when | example |
|---|---|---|---|
| `causal` | 0.9 | A→B | bug→outage |
| `semantic` | 0.8 | same concept | 2× PG mems |
| `supports` | 0.8 | A evidences B | test→hypothesis |
| `part_of` | 0.8 | A component of B | chapter→book |
| `contextual` | 0.7 | shared context | same project/meeting |
| `derived_from` | 0.7 | B abstracted from A | episodes→schema |
| `temporal` | 0.6 | near in time | sequential events |
| `affective` | 0.5 | similar feeling | 2× frustrating experiences |
| `contradicts` | 0.3 | A opposes B | conflicting recs |

### Linking Patterns
```
a = remember("Deploy failed due to missing env var")
b = remember("Added env validation to CI pipeline")
associate(source_id=a, target_id=b, link_type="causal",
          evidence="fail→fix", weight=0.9)
```
```
associate(source_id=fact_id, target_id=procedure_id,
          link_type="supports", evidence="fact validates proc")
```

**Don't over-link.** Link on genuine relationship. Dream Engine discovers implicit connections.

---

## Episodes (temporal narratives)

### Pattern: Recording a Task
```
1. episode_start(title="Debugging OOM in prod")
2. remember("Noticed 502 errors at 14:30") → add_step(role="event")
3. remember("Checked logs, found OOM killer") → add_step(role="context")
4. remember("Increased memory limit to 4GB") → add_step(role="event")
5. remember("Service recovered after restart") → add_step(role="outcome")
6. remember("Should add memory alerts to monitoring") → add_step(role="reflection")
7. episode_end(summary="OOM in prod, fixed by increasing limits", valence="mixed")
```

roles: `event`(default) · `context`(bg/obs) · `outcome`(result) · `reflection`(lesson)

Episode vs individual: episode for multi-step process/debug/meeting/milestone · individual for standalone facts/prefs/one-offs

---

## Procs (tracked workflows)

Track success/fail over time. Frequently-successful procs rank ↑; failed ones ↓.

```
proc_id = store_procedure(
    content="Deploy to prod: 1) tests 2) build 3) push 4) apply 5) verify",
    tags=["deployment", "production"])
record_procedure_outcome(procedure_id=proc_id, success=True)
```

---

## Schemas (abstract patterns)

Principles derived from multiple specific mems.

```
create_schema(
    content="Services skipping staging have ≈3x more prod incidents",
    source_ids=["mem_incident_1", "mem_incident_2", "mem_incident_3"],
    tags=["deployment", "best-practices"])
```

Dream Engine also creates schemas automatically during consolidation.

---

## Intentions (smart TODOs)

Prospective mems that surface when relevant. Unlike simple TODOs, intentions participate in recall — intention about "upgrade Python" surfaces when someone asks about Python.

```
store_intention(content="Upgrade Python to 3.12 before deploying new service", salience=0.8)
list_intentions()
resolve_intention(memory_id="mem_xxx")
```

---

## Multi-Agent Communication

Multiple agents share one mem store.

```
send_message(to="CLAUDE-HAILO", content="EEG plugin ready for testing on your Pi")
send_message(to="all", content="CerebroCortex v0.2.0 deployed, dream engine improved")
```
Messages bypass gating — always delivered. Auto-tagged w/ sender/recipient.

```
check_inbox()
check_inbox(from_agent="CLAUDE-OPUS", since="2026-04-14T00:00:00")
```

New agents must register before store/recall:
```
register_agent(
    agent_id="MY-AGENT", display_name="My Agent",
    generation=1,           # -1=origin, 0=primary, 1+=descendant
    lineage="hermes", specialization="what this agent is good at")
```

---

## Session Lifecycle

### Start
```python
session_recall(limit=5)     # recent sessions
list_intentions()            # pending?
check_inbox()                # messages?
```

### End
```python
session_save(
    session_summary="Debugged OOM in prod, deployed fix, updated monitoring",
    key_discoveries=["PostgreSQL default connections too low for our worker count"],
    unfinished_business=["Add memory usage alerts to Grafana"],
    if_disoriented=["Check the monitoring PR in GitHub"],
    session_type="technical", priority="HIGH")
```

---

## Dream Engine (offline consolidation)

6 phases modeled after biological sleep:
1. **Replay** — revisit recent mems, strengthen important
2. **Cluster** — group by embedding similarity
3. **Abstract** — extract schemas from clusters (LLM)
4. **Prune** — remove low-activation unlinked mems
5. **Promote** — working→long-term→cortex
6. **REM Recombine** — unexpected cross-domain connections (LLM)

```
dream_run()                  # full cycle (≈20 LLM calls)
dream_run(max_llm_calls=10)  # lighter
dream_status()               # last run results
```

**When:** weekly cron · post-bulk-ingest · user says "consolidate" · post-intensive-project

**Pitfall:** Dream runs use LLM calls (token cost). On cost-conscious setups, use `max_llm_calls=10`.

---

## Mem Layers & Decay

```
Sensory (6hr half-life) → Working (3-day) → Long-Term (30-day) → Cortex (permanent)
```

- Sensory→Working: auto on 2nd access
- Working→Long-Term: 5+ accesses, 24h+ old
- Long-Term→Cortex: Dream Engine only (crystallized knowledge)

System promotes based on access patterns. Frequently-recalled mems survive; neglected fade.

**To keep a mem alive:** recall it. Every recall strengthens activation + stability.

---

## Common Workflows

### Research & Learning
```
1. ingest_file(file_path="/path/to/paper.md", tags=["research", "topic"])
2. recall("key concepts from the paper")
3. remember() important findings w/ high salience
4. associate() related findings
5. create_schema() when patterns emerge
```

### Debugging Session
```
1. episode_start(title="Debugging: service X failing")
2. remember(observation) → episode_add_step()
3. When solved: remember(solution, memory_type="procedural")
4. episode_end(summary="...", valence="positive")
5. associate(problem_id, solution_id, "causal")
```

### Project Kickoff
```
1. recall("similar past projects")
2. find_relevant_procedures(concepts=["project-type"])
3. store_intention("Project milestones and deadlines")
4. remember("Project goals and constraints", salience=0.8)
```

### Cross-Agent Handoff
```
1. session_save(summary="...", unfinished_business=["task X"])
2. send_message(to="OTHER-AGENT", content="Continue task X, see session notes")
3. Other agent: check_inbox() → session_recall() → continue
```

---

## Pitfalls

1. **¬ store raw data dumps.** CerebroCortex is for distilled knowledge, ¬ logs. Summarize first. Use ingest_file for large docs — auto-chunks.

2. **¬ over-link.** Quality > quantity. Few strong causal/semantic links beat 50 weak contextual ones. Dream Engine discovers implicit connections.

3. **¬ forget to recall.** Mems never recalled → decay → pruned. If important, access periodically. By design — real mem works this way.

4. **Aliases exist.** `memory_store` ≡ `remember`, `memory_search` ≡ `recall`. ¬ call both thinking they're different.

5. **explain mode for debugging.** If recall results seem wrong, `explain=True` → score breakdown (vector similarity, ACT-R, FSRS, salience). Reveals whether issue is semantic, temporal, or importance-based.

6. **Dream Engine costs tokens.** Each run ≈20 LLM calls. Schedule wisely on prepaid API credits.

7. **Visibility matters in multi-agent.** Private mems invisible to other agents. Shared = default for collab. Thread-scoped for conversation-local context.

8. **Session notes = your compass.** Always save at end, recall at start.

9. **Intentions surface in recall.** Stored intention about "upgrade Python" appears when someone asks about Python. Feature, ¬ bug — prospective mem working as designed.

10. **Hardware context:** Built for Raspberry Pi 5 (4GB RAM). Lightweight by design. Dream engine = heaviest op — avoid running during other intensive tasks.
