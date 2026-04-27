# WikiNext

A hybrid AI knowledge system for solo researchers working across multiple deep technical domains. Combines structured fact storage with compiled narrative synthesis -- avoiding the failure modes of both pure wiki (Karpathy-style) and pure database (OpenBrain-style) approaches.

Built for researchers who work across domains like bioinformatics, robotics AI/ML, edgeML/tinyML, health monitoring, algorithm discovery, and multi-omics -- where the highest value is in **cross-domain connections** that conventional tools miss entirely.

## Core Thesis

The fundamental architectural choice in AI knowledge systems is **when the AI thinks**: at ingest time (wiki) or at query time (database). Both break in predictable ways. WikiNext takes a third path: **structured storage as source of truth, with a graph + wiki compiler that produces browsable narrative views on demand**.

### Why Not Just Wiki?

- Staleness reads as confident prose, not missing data
- Multi-agent/team writes produce merge conflicts and incoherent narratives
- High-frequency updates (tickets, events, CRM) overwhelm re-synthesis costs
- Early editorial choices silently drop nuance from raw sources

### Why Not Just RAG/Database?

- Deep cross-document synthesis is re-derived from scratch every query
- No persistent browsable artifact -- headless by default
- Contradictions sit silently unless you ask the exact right question
- "Real cognitive work done and thrown away" on every query

### The Hybrid Pattern

```
[Raw Sources] --> [Structured DB + Embeddings] --> [Knowledge Graph] --> [Compiled Wiki Views]
                  (source of truth)                (relations layer)    (regenerable artifacts)
```

- **Structured DB** is authoritative: every fact has metadata, tags, timestamps, provenance
- **Knowledge Graph** maps relations, contradictions, and topic clusters
- **Wiki Views** are compiled artifacts: human-readable, browsable, but regenerable from source
- **AI agents** are long-running maintainers of the graph + wiki, not ephemeral answer engines

## What It Solves

The core pain for a multi-domain researcher is **context-switching cost**: every time you move from multi-omics to edge ML, you spend 30-60 minutes rebuilding context you had last week. You re-read papers you've already synthesized, forget which approaches you rejected (and why), and miss connections across domains because you're deep in one when the insight from another would be relevant.

WikiNext solves this by:
- **Instant context recovery**: Switch domains, get a compiled page with current state, open questions, recent experiments, and cross-domain connections
- **Cross-domain bridge detection**: Proactive identification of connections (e.g., "your edge ML quantization paper is relevant to the bioinformatics pipeline you're optimizing for portable sequencers")
- **Experimental memory**: Failed experiments are first-class artifacts -- "you tried this config on March 15 and it failed because of gradient instability at low precision"
- **Hypothesis tracking**: Active hypotheses accumulate evidence across domains over time
- **Decision trail**: Revisit a domain after months and understand not just *what* you decided but *why*

## Architecture

See [docs/architecture/](docs/architecture/) for detailed design documents:

- [ADR-001: Hybrid Memory Architecture](docs/architecture/adr-001-hybrid-memory.md) -- Three-layer pattern
- [ADR-002: Ingest vs Query Time Processing](docs/architecture/adr-002-processing-strategy.md) -- Tiered processing strategy
- [ADR-003: Contradiction and Staleness Detection](docs/architecture/adr-003-contradiction-detection.md) -- First-class contradiction handling
- [ADR-004: Domain Model](docs/architecture/adr-004-domain-model.md) -- Data model for multi-domain researcher

## Discovery

See [docs/discovery/](docs/discovery/) for research phase documents:

- [Persona: Solo Multi-Domain Researcher](docs/discovery/persona-solo-researcher.md)
- [Design Questions](docs/discovery/design-questions.md) -- Resolved and open questions
- [Failure Mode Analysis](docs/discovery/failure-modes.md) -- What breaks and how we prevent it
- [Explicit Assumptions](docs/discovery/assumptions.md) -- What we believe, how confident, how to validate

## Background

This architecture is informed by [Jones's analysis](https://ppl-ai-file-upload.s3.amazonaws.com) of Karpathy's AI wiki vs OpenBrain, which frames the core fork as: does the AI think at ingest time or query time? WikiNext takes both -- structured storage for facts, periodic synthesis for narrative, and on-demand analysis for novel questions.

Key references:
- Karpathy's wiki concept: ingest-time synthesis, compounding understanding
- OpenBrain: query-time structured memory, faithful storage with on-demand synthesis
- Jones's hybrid pattern: structured DB as source of truth, graph + wiki as compiled views

## Project Status

**Phase: Discovery & Architecture Definition**

This project is deliberately in a deep discovery phase. Early definition work matters more than speed to execution -- requirement errors become dramatically more costly in later phases.

## License

MIT
