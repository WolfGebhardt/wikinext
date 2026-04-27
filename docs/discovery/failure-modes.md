# Failure Mode Analysis

Catalogued failure modes from Jones's analysis of Karpathy wiki vs OpenBrain, plus general project failure research. This document exists to ensure we design against these specific breakdowns.

## Conventional AI-Over-Documents Failures

These are the default failures of "just use RAG" or "just feed PDFs and ask questions":

1. **Re-derivation waste**: The model re-derives the same synthesis every time, doing "real cognitive work and then throwing it away"
2. **No persistent cross-document understanding**: Connections and contradictions are not kept as first-class artifacts
3. **Hyper-fragmentation**: Queries across ChatGPT, NotebookLM, Claude, etc. never benefit from a single compounding memory
4. **No compounding**: Understanding doesn't get better over time -- it's recomputed fresh each time

## Wiki-Specific Failures (Karpathy-style)

| ID | Failure | Trigger | Detection Signal |
|---|---|---|---|
| W1 | Multi-agent merge conflicts | >1 writer to same topic | Incoherent narratives, contradictory edits |
| W2 | Update cost explosion | >50 operational updates/day | Ingest latency spike, API cost jump |
| W3 | Staleness as misinformation | Pages not recompiled after source changes | Decisions made on outdated synthesis |
| W4 | Editorial nuance loss | AI makes editorial choices at ingest | Users can't access details AI deemed unimportant |
| W5 | Wiki becomes trusted as source of truth | Convenience of reading prose > checking raw sources | Team stops verifying claims against originals |

## Database-Specific Failures (OpenBrain-style)

| ID | Failure | Trigger | Detection Signal |
|---|---|---|---|
| D1 | Expensive deep synthesis | Queries spanning 15+ facts | High query latency, high API cost per question |
| D2 | No browsable artifact | Users want to "wander" through knowledge | Requests for "just show me everything about X" |
| D3 | Silent contradictions | Conflicting rows on same entity | Wrong decisions because conflict wasn't surfaced |
| D4 | Unstable synthesis | Same query gives different answers | Users lose trust in system outputs |

## General Project Failures (Novel/Complex Space)

From Astadia, Saritasa, and construction project failure research:

| ID | Failure | Root Cause | WikiNext Mitigation |
|---|---|---|---|
| G1 | Bad problem framing | Building before validating what users need | Discovery phase with assumption testing |
| G2 | Shallow planning | Rough estimates, vague assumptions | ADRs with explicit scale breakpoints |
| G3 | Requirement vagueness | Generic workflows, no forced precision | Design questions encoded in architecture |
| G4 | Hidden dependency chains | Spreadsheets can't show interdependencies | Knowledge graph makes relations explicit |
| G5 | Late-surfacing complexity | Edge cases appear during implementation | Failure mode analysis upfront (this document) |
| G6 | Scope creep without cost visibility | Changes accumulate without showing true cost | Processing tiers with explicit cost profiles |
| G7 | Poor knowledge transfer | Outcomes captured poorly, teams repeat mistakes | Contradiction resolution history, synthesis changelogs |

## Design Responses

Each failure has a specific architectural response:

- **W1, D3**: Transactional structured store as source of truth (no file-level merge conflicts)
- **W2**: Tiered processing -- lightweight ingest, batched synthesis
- **W3, W5**: Freshness marks, provenance links, regenerable wiki views
- **W4**: Raw sources always accessible alongside synthesis
- **D1**: Persistent graph + cached synthesis reduces re-derivation
- **D2**: Compiled wiki views provide browsable artifacts
- **D4**: Deterministic graph traversal + consistent compilation prompts
- **G1-G7**: Discovery-first approach, explicit architecture decisions, documented assumptions
