# ADR-002: Ingest vs Query Time Processing Strategy

**Status**: Accepted  
**Date**: 2026-04-27  
**Context**: Defining when AI does "hard thinking" across the system

## Decision

WikiNext uses a **tiered processing strategy**: lightweight classification at ingest, periodic deep synthesis via scheduled agents, and on-demand query-time analysis for novel questions.

## The Core Tradeoff

Jones frames this as the single most consequential fork:

> Does the AI do hard thinking when information comes in, or when you ask about it?

Neither extreme works. We use three tiers:

### Tier 1: Ingest-Time (Lightweight)

**When**: Every new source arrives  
**Cost**: Low (classification + embedding, not full synthesis)  
**What happens**:
- Extract structured metadata (source, date, entities mentioned, type)
- Generate embeddings for semantic search
- Tag with topics from existing graph taxonomy
- Run lightweight contradiction check against existing facts on same entities
- Flag high-confidence contradictions for immediate attention
- Store as immutable row in structured store

**What does NOT happen**:
- No full narrative synthesis
- No wiki page updates
- No deep cross-document reasoning

This avoids the Karpathy wiki failure of "every ingest triggers expensive re-synthesis" while still capturing enough structure for the graph.

### Tier 2: Scheduled Synthesis (Deep, Periodic)

**When**: On schedule (configurable: hourly, daily, or on-demand)  
**Cost**: Medium-High (full graph traversal + wiki compilation)  
**What happens**:
- Graph maintenance agent walks recent additions
- Updates relation edges (new connections, strengthened/weakened links)
- Runs full contradiction audit across topic clusters
- Recompiles wiki pages for topics with changed source data
- Marks unchanged pages as still-fresh, changed pages as updated
- Produces a "synthesis changelog" showing what understanding shifted

This is where compounding happens. Unlike pure query-time systems, this work is persisted -- you don't re-derive it on every question.

### Tier 3: Query-Time (On-Demand)

**When**: User asks a question not fully answered by existing wiki/graph  
**Cost**: Variable (depends on scope)  
**What happens**:
- First check: does a compiled wiki page already answer this? (cheap)
- Second check: can the graph + structured store answer via filtered query? (medium)
- Third: full synthesis across relevant artifacts (expensive, but results cached back into graph)
- Novel synthesis results are proposed as graph updates for the next Tier 2 cycle

This ensures query-time work is not wasted -- it feeds back into the persistent knowledge base.

## Cost Profile

| Operation | Pure Wiki | Pure RAG | WikiNext Hybrid |
|---|---|---|---|
| Ingest 1 document | Expensive (full synthesis) | Cheap (store + embed) | Low-Medium (classify + embed + light check) |
| Simple factual query | Cheap (read wiki) | Medium (retrieve + synthesize) | Cheap (read wiki or structured query) |
| Deep cross-document query | Cheap (if wiki is current) | Expensive (full re-synthesis) | Medium (graph traversal + targeted synthesis) |
| 100 documents/day operational load | Very Expensive | Cheap | Low (Tier 1 only, Tier 2 batched) |

## AI's Job Description Per Tier

| Tier | AI Role | Analogy |
|---|---|---|
| Tier 1: Ingest | Librarian/classifier | Filing clerk who tags and shelves, doesn't write summaries |
| Tier 2: Synthesis | Editor/cartographer | Periodically updates the map of understanding |
| Tier 3: Query | Research analyst | Deep-dives when the map doesn't cover the territory |

## Consequences

- Ingest is fast enough for operational data (Slack, tickets, CRM events)
- Deep understanding still compounds over time via Tier 2
- Query-time work feeds back into persistent knowledge (no "think and throw away")
- Cost is predictable: Tier 2 is the main budget item, and it's scheduled/controllable
