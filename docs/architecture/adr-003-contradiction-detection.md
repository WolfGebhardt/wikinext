# ADR-003: Contradiction and Staleness Detection

**Status**: Accepted  
**Date**: 2026-04-27  
**Context**: Ensuring contradictions surface visibly rather than being smoothed away or sitting silently

## Decision

Contradictions and staleness are **first-class objects** in the system, with dedicated detection at both ingest and synthesis time, and mandatory visibility in wiki views.

## Problem

Both pure approaches fail at contradiction handling:

- **Wiki approach**: AI smooths contradictions into coherent narrative at ingest time. The user reads confident prose and has no idea two sources disagreed. Staleness looks like truth.
- **Database approach**: Contradicting rows sit side by side silently. Unless you ask "do any sources disagree about X?", you'll never know. Contradictions are invisible by default.

## Design

### Contradiction as a First-Class Entity

```
contradictions table:
  id
  fact_a_id        -> references structured_store
  fact_b_id        -> references structured_store
  topic_id         -> references knowledge_graph.topics
  type             -> enum: direct_conflict, temporal_supersession, scope_mismatch, ambiguity
  confidence       -> float (0-1, how confident the system is this is a real contradiction)
  detected_at      -> timestamp
  detected_by      -> enum: ingest_check, synthesis_audit, user_report, query_discovery
  resolution       -> nullable: which fact won, or "unresolved"
  resolution_note  -> nullable: why
```

### Detection Points

**At Ingest (Tier 1)**:
- When a new fact is classified, check existing facts on the same entities/topics
- Lightweight semantic similarity + entity overlap check
- If confidence > 0.7, create a contradiction record immediately
- Flag in the ingest response: "This contradicts [existing fact] from [source]"

**At Synthesis (Tier 2)**:
- Full audit agent walks each topic cluster
- Compares all facts within a cluster for logical consistency
- Catches contradictions that span topics (entity A says X in topic 1, but topic 2 implies not-X)
- Updates contradiction confidence scores based on broader context

**At Query (Tier 3)**:
- If a query synthesis encounters conflicting sources, record the contradiction
- Surface it in the response: "Note: sources disagree on this point"
- Propose contradiction record for persistence

### Staleness Detection

Every compiled wiki page carries:
```yaml
---
topic: "example-topic"
last_compiled: 2026-04-27T14:30:00Z
source_facts_count: 23
newest_source: 2026-04-27T12:00:00Z
oldest_source: 2025-11-03T08:15:00Z
staleness_status: fresh | stale | unknown
stale_reason: null | "3 new facts since last compilation" | "source updated"
open_contradictions: 2
---
```

**Staleness rules**:
- If `newest_source > last_compiled`: page is **stale** (new data exists that hasn't been synthesized)
- If `last_compiled` is more than [configurable threshold] old: page is **potentially stale**
- If `open_contradictions > 0`: page carries a visible warning banner

### Visibility in Wiki Views

Compiled wiki pages MUST show:
1. **Freshness badge**: "Last updated [date], based on [N] sources"
2. **Contradiction callouts**: inline markers where sources disagree, with links to both sources
3. **Provenance links**: every claim links back to its source fact(s) in the structured store
4. **Confidence indicators**: where synthesis required judgment, mark it as interpretation vs established fact

Example wiki output:
```markdown
## Topic: Widget Pricing Strategy

Widget pricing was set at $49/unit in Q1 2026. [source: pricing-memo-2026-01]

> **Contradiction detected**: The sales team deck from March states $39/unit 
> for enterprise. [source: sales-deck-2026-03] This conflicts with the 
> pricing memo. **Status: Unresolved** [View details →]

The margin analysis assumes the $49 price point. [source: margin-report-2026-02]

---
*Last compiled: 2026-04-27 | 1 open contradiction | Sources: 3 documents*
```

## Consequences

### Positive
- Contradictions cannot hide -- they surface in both the graph and wiki views
- Users develop calibrated trust: they can see where the system is confident vs uncertain
- Staleness is visible as staleness, not as confident misinformation
- Resolution history creates institutional memory of how conflicts were decided

### Negative
- More visual noise in wiki pages (contradiction banners, freshness marks)
- Requires careful UX to avoid "alert fatigue" from low-confidence contradictions
- Contradiction detection adds cost to ingest and synthesis

### Open Questions
- What confidence threshold triggers a visible contradiction callout vs quiet logging?
- How to handle temporal supersession (newer fact replaces older) vs genuine disagreement?
- Should users be able to "resolve" contradictions, and does that feed back into AI behavior?
