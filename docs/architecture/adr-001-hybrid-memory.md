# ADR-001: Hybrid Memory Architecture

**Status**: Accepted  
**Date**: 2026-04-27  
**Context**: Choosing the foundational memory architecture for WikiNext

## Decision

WikiNext uses a three-layer hybrid architecture: structured storage (source of truth), knowledge graph (relations layer), and compiled wiki views (presentation layer).

## Context

Jones's analysis of Karpathy's wiki vs OpenBrain identifies a fundamental fork: does the AI think at ingest time or query time? Each approach has predictable failure modes at scale.

### Ingest-Time (Karpathy Wiki) Failures

| Failure Mode | Trigger | Effect |
|---|---|---|
| Multi-agent conflicts | Multiple writers to same pages | Merged narrative reflects no one's mental model |
| Update cost explosion | High-frequency operational data | Re-synthesis on each ingest becomes punishing |
| Staleness as misinformation | Neglected pages | Old synthesis reads as confident truth |
| Editorial nuance loss | AI synthesis choices at ingest | Subtle details dropped, invisible to downstream readers |

### Query-Time (OpenBrain/RAG) Failures

| Failure Mode | Trigger | Effect |
|---|---|---|
| Expensive re-derivation | Complex cross-document queries | Same synthesis done and thrown away repeatedly |
| No browsable artifact | Default headless architecture | Must build UI from scratch, no wandering |
| Silent contradictions | Conflicting rows in DB | Never surface unless explicitly audited |
| Fragmented context | Multiple tools querying independently | No single compounding memory across agents |

## The Hybrid Pattern

```
Layer 1: Structured Store (PostgreSQL + pgvector)
├── Every artifact: one row with metadata, tags, timestamps, provenance
├── Immutable source of truth
├── Supports relational/filtered queries, multi-agent access, auditing
└── Scales to 10,000+ artifacts

Layer 2: Knowledge Graph (stored in DB, computed by agents)
├── Topics, entities, projects as nodes
├── Relations: supports, contradicts, extends, depends-on
├── Contradiction pairs flagged as first-class objects
└── Updated incrementally on ingest + periodically via audit agents

Layer 3: Compiled Wiki Views (Markdown, regenerable)
├── Human-readable topic pages generated from graph + store
├── Browsable in Obsidian, web UI, or any Markdown renderer
├── Each page carries freshness marks and source provenance
└── Can be fully regenerated from Layer 1+2 if drift is detected
```

## Key Invariants

1. **Wiki is never the source of truth.** It is a compiled view. If wiki and DB disagree, DB wins.
2. **Facts and narrative are stored separately.** A fact is a row with provenance. A narrative is a compiled synthesis with source links.
3. **Contradictions are first-class.** Two facts that conflict produce a contradiction node in the graph, visible in wiki views.
4. **Staleness is visible.** Every wiki page carries a "last compiled" timestamp and a "source freshness" indicator. Pages older than their source data are flagged.
5. **AI is a maintainer, not an ephemeral oracle.** Agents run on schedules to update the graph and recompile views, not just to answer one-shot queries.

## Consequences

### Positive
- Source data integrity is preserved even when wiki views drift
- Contradictions surface proactively rather than hiding in database rows
- Multi-agent writes go to structured store (transactional, safe) rather than fighting over Markdown files
- Wiki pages can be regenerated from source, preventing institutionalized misinformation
- System compounds understanding over time rather than re-deriving it

### Negative
- More complex than either pure approach
- Requires explicit design of the graph schema and compiler agents
- Two sources to maintain (store + graph) before views are even possible
- Initial setup cost is higher than "just start a wiki" or "just do RAG"

### Risks
- Graph schema may need significant iteration as real data arrives
- Compiler agents may produce inconsistent views if not carefully prompted
- Users may still treat wiki views as source of truth despite design intent

## Scale Breakpoints

| Layer | Comfortable Range | Stress Point | Mitigation |
|---|---|---|---|
| Structured Store | 0 - 100,000+ artifacts | Storage/query cost at extreme scale | Standard DB scaling (indexing, partitioning) |
| Knowledge Graph | 0 - 50,000 edges | Graph traversal cost for deep synthesis | Materialized sub-graphs per topic cluster |
| Compiled Wiki | 10 - 10,000 pages | Recompilation time, inter-page consistency | Incremental compilation, dependency tracking |
