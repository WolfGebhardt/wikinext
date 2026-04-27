# Design Questions

These questions must be concretely answered by the system design. Derived from Jones's framework for evaluating AI knowledge architectures.

## Resolved

### Q1: Ingest-time or query-time processing?
**Answer**: Both, via tiered processing (ADR-002). Lightweight classification at ingest, periodic deep synthesis via scheduled agents, on-demand query-time analysis for novel questions.

### Q2: What is the source of truth?
**Answer**: Structured database (PostgreSQL + pgvector). Wiki pages are compiled views, never primary storage. If wiki and DB disagree, DB wins.

### Q3: How are facts and narrative separated?
**Answer**: Facts are rows in structured store with provenance, timestamps, and metadata. Narrative is compiled wiki pages generated from facts + graph. Separate storage, separate update cycles.

### Q4: What is AI's job description?
**Answer**: Three roles via processing tiers:
- Tier 1 (Ingest): Librarian/classifier -- tags, embeds, light contradiction check
- Tier 2 (Synthesis): Editor/cartographer -- updates graph, compiles wiki views
- Tier 3 (Query): Research analyst -- deep-dives when existing artifacts don't suffice

### Q5: What is the target user and use case?
**Answer**: Solo researcher with multiple deep, technically diverse domains: bioinformatics pipelines, robotics AI/ML, edgeML/tinyML/edgeAI, health monitoring pipelines/algorithm discovery/training/validation, multi-omics discovery and validation.

**Cascading implications**:
- No multi-user conflicts (simplifies storage layer significantly)
- Cross-domain connections are the killer feature -- the graph must actively detect bridges
- Technical precision is non-negotiable -- lossy summarization of p-values, model params, or pipeline configs is harmful
- Artifacts are heterogeneous: papers, code, experiments, model configs, pipeline YAML, Jupyter notebooks
- Hypothesis tracking is a first-class concept that evolves over time
- Primary pain is context-switching cost, not information retrieval
- Cost tolerance is personal budget, not enterprise

See [persona-solo-researcher.md](persona-solo-researcher.md) and [ADR-004](../architecture/adr-004-domain-model.md).

## Open

### Q6: At what artifact count does each layer break?
**Hypothesis** (needs validation):
- Structured store: comfortable to 100K+ artifacts
- Knowledge graph: comfortable to 50K edges, stress at 100K+
- Compiled wiki: comfortable to 10K pages, stress when inter-page consistency matters at scale

### Q7: How should temporal supersession work?
For a researcher, understanding *evolves* -- a hypothesis from 3 months ago may be invalidated by new data. The system should:
- Keep the full temporal trail visible (not just latest state)
- Auto-detect temporal supersession (newer paper updates older finding)
- But preserve the evolution: "You believed X in March based on evidence A, but evidence B from April weakened it"
- For hypotheses specifically, track confidence over time as evidence accumulates

### Q8: What front-end serves the compiled wiki?
**Options**:
- Obsidian vault (Markdown files, local-first)
- Custom web UI (Next.js or similar)
- Both: Markdown as intermediate format, web UI as primary, Obsidian as power-user option

### Q9: How do tools feed the system?
Solo researcher uses multiple tools (Claude Code, Cursor, ChatGPT, Jupyter, CLI). How do they ingest into the same store?
- CLI tool for manual ingest (`wikinext ingest <file>`)
- Watch folder / file system monitor for automatic ingest
- API endpoint for programmatic ingest from scripts and notebooks
- Each source gets a provenance tag (which tool, when, what context)
- No write conflict concern (single user), but dedup needed (same paper ingested from two tools)

### Q10: What is the cost budget per tier?
- Tier 1 (per ingest): target < $0.01 per document
- Tier 2 (per synthesis cycle): target < $1.00 per topic cluster
- Tier 3 (per query): variable, but cached results amortize cost

### Q11: How do we validate the architecture before full build?
**Proposed validation approach**:
1. Load 50-100 real documents into structured store
2. Manually define 5-10 topic clusters in the graph
3. Run compiler agent to produce wiki pages
4. Evaluate: Do the pages reflect genuine understanding? Are contradictions visible? Is provenance useful?
5. Test staleness: add new documents, check if freshness detection works
