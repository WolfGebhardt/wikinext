# WikiNext

A hybrid AI knowledge system for solo researchers working across multiple deep technical domains. Combines structured fact storage with compiled narrative synthesis -- avoiding the failure modes of both pure wiki and pure database approaches.

Built for researchers who work across domains like bioinformatics, robotics AI/ML, edgeML/tinyML, health monitoring, algorithm discovery, and multi-omics -- where the highest value is in **cross-domain connections** that conventional tools miss entirely.

## Why This Exists

Two important open-source projects define the current landscape for AI-powered personal knowledge:

- [**Karpathy's Wiki LLM**](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) -- An elegant concept where the AI maintains a personal wiki at ingest time. New sources are read, synthesized into topic pages, cross-referenced, and contradictions flagged immediately. Queries are cheap because understanding is pre-built. But the wiki *is* the source of truth, staleness looks like confident prose, and it breaks under multi-domain scale.

- [**Nate B. Jones's OpenBrain (OB1)**](https://github.com/NateBJones-Projects/OB1) -- Working infrastructure (Postgres + pgvector + MCP) where facts are stored faithfully as structured rows with metadata and embeddings. No synthesis at ingest -- queries pull relevant entries and the AI synthesizes on the fly. Clean, honest, deployable. But deep cross-document synthesis is re-derived every time, contradictions sit silently, and there's no browsable artifact to wander through.

**WikiNext asks: what if you took the best of both and fixed what breaks?**

## Core Thesis

The fundamental architectural choice in AI knowledge systems is **when the AI thinks**: at ingest time (wiki) or at query time (database). Both break in predictable ways. WikiNext takes a third path: **structured storage as source of truth, with a knowledge graph + wiki compiler that produces browsable narrative views on demand**.

### How WikiNext Differs

| Capability | Karpathy Wiki | OpenBrain (OB1) | WikiNext |
|---|---|---|---|
| Source of truth | Wiki pages (risky -- staleness reads as truth) | Database rows (correct) | Database rows; wiki is a regenerable compiled view |
| When AI thinks | All at ingest (expensive per source) | All at query (expensive per question) | Three tiers: light ingest, periodic synthesis, on-demand query |
| Contradictions | Mentioned in "lint" pass, no resolution | Schema supports edges, no auto-detection | First-class objects with typed resolution and provenance |
| Cross-domain connections | Not modeled | Not modeled | Primary feature -- active bridge detection across domains |
| Artifact types | Generic markdown pages | Generic "thoughts" + JSONB metadata | 9 typed schemas: paper, code, experiment, model, dataset, pipeline, hypothesis, decision, validation |
| Hypothesis tracking | None | None | Full lifecycle: proposed → active → supported/weakened → validated/invalidated |
| Failed experiments | Not modeled | Not modeled | First-class artifacts with failure reasons and parameters |
| Staleness detection | Wiki drifts silently | No mechanism | Freshness marks, source provenance, regeneration from DB |
| Browsable output | Markdown wiki (the only layer) | None (headless by design) | Compiled wiki views regenerated from graph + DB |

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

- [ADR-001: Hybrid Memory Architecture](docs/architecture/adr-001-hybrid-memory.md) -- Three-layer pattern with explicit failure mode mapping
- [ADR-002: Ingest vs Query Time Processing](docs/architecture/adr-002-processing-strategy.md) -- Tiered processing with cost profiles per tier
- [ADR-003: Contradiction and Staleness Detection](docs/architecture/adr-003-contradiction-detection.md) -- Contradictions as first-class entities
- [ADR-004: Domain Model](docs/architecture/adr-004-domain-model.md) -- Typed artifact schema, knowledge graph edges, hypothesis lifecycle

## Discovery

See [docs/discovery/](docs/discovery/) for research phase documents:

- [Persona: Solo Multi-Domain Researcher](docs/discovery/persona-solo-researcher.md)
- [Design Questions](docs/discovery/design-questions.md) -- Resolved and open questions
- [Failure Mode Analysis](docs/discovery/failure-modes.md) -- What breaks in each approach and how WikiNext prevents it
- [Explicit Assumptions](docs/discovery/assumptions.md) -- What we believe, how confident, how to validate

## Standing on the Shoulders

WikiNext wouldn't exist without the thinking behind both projects:

- **Karpathy's key insight**: LLMs should be long-running *maintainers* of knowledge artifacts, not ephemeral answer engines. The wiki-as-compounding-understanding pattern is powerful. WikiNext preserves this by having AI agents maintain the graph and compile wiki views -- but treats the wiki as a regenerable artifact, not the source of truth.

- **Jones's key insight (OpenBrain)**: Facts should be stored faithfully with provenance, not lossy-summarized into prose. The structured store + MCP bridge pattern is correct infrastructure. WikiNext builds on this by adding a knowledge graph layer and a compilation step that produces the browsable wiki Karpathy envisioned -- without sacrificing the data integrity OpenBrain provides.

- **What WikiNext adds**: The explicit hybrid architecture (three processing tiers), cross-domain bridge detection, typed artifact schemas for research workflows, hypothesis lifecycle tracking, and first-class contradiction handling with resolution provenance.

## Project Status

**Phase: Discovery & Architecture Definition**

This project is deliberately in a deep discovery phase. Early definition work matters more than speed to execution -- requirement errors become dramatically more costly in later phases.

Next milestone: prototype with 50-100 real documents to validate the three-layer architecture against actual research workflows.

## License

MIT
