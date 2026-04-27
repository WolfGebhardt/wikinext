# ADR-004: Domain Model for Solo Multi-Domain Researcher

**Status**: Accepted  
**Date**: 2026-04-27  
**Context**: Defining the data model for a solo researcher working across bioinformatics, robotics AI/ML, edgeML/tinyML, health monitoring, algorithm discovery, and multi-omics

## Decision

The system uses a flexible, typed knowledge graph with domain-aware artifact storage, hypothesis tracking as a first-class concept, and explicit cross-domain bridge detection.

## Data Model

### Artifacts (Structured Store)

Every piece of knowledge enters as an artifact with rich, typed metadata:

```sql
artifacts:
  id              UUID PRIMARY KEY
  type            ENUM(paper, code, experiment, model, dataset, pipeline,
                       hypothesis, decision, validation, note)
  title           TEXT
  content         TEXT           -- raw content, never summarized
  content_hash    TEXT           -- detect duplicates and changes
  source_url      TEXT           -- where it came from (URL, file path, DOI)
  source_type     ENUM(arxiv, biorxiv, github, local_note, jupyter, config, manual)
  created_at      TIMESTAMP
  ingested_at     TIMESTAMP
  domains         TEXT[]         -- e.g., ['edge_ml', 'health_monitoring']
  tags            TEXT[]         -- freeform, user-defined
  embedding       VECTOR(1536)  -- for semantic search
  metadata        JSONB          -- type-specific fields (see below)
```

### Type-Specific Metadata (JSONB)

```yaml
paper:
  authors: [str]
  venue: str
  date: date
  claims: [str]           # key claims extracted
  methods: [str]          # methods used
  doi: str

code:
  language: str
  framework: str
  repository: str
  function_name: str      # if referencing a specific function
  solves: str             # what problem this code addresses

experiment:
  outcome: enum(success, failure, partial, inconclusive)
  parameters: jsonb       # what was tried
  metrics: jsonb          # what was measured
  failure_reason: str     # if failed, why -- this is high-value data

model:
  task: str
  architecture: str
  constraints:            # latency_ms, memory_mb, power_mw
  performance: jsonb      # accuracy, f1, inference_time, etc.
  quantization: str       # if applicable

dataset:
  modalities: [str]       # e.g., ['transcriptomics', 'proteomics']
  size: str
  access: str             # how to get it
  known_issues: [str]

pipeline:
  steps: [str]
  dependencies: [str]
  runtime: str
  failure_modes: [str]

hypothesis:
  statement: str
  confidence: float       # 0-1, updated as evidence accumulates
  status: enum(active, supported, weakened, invalidated, merged)
  supporting_evidence: [artifact_id]
  contradicting_evidence: [artifact_id]
  domains: [str]          # which domains this hypothesis spans

decision:
  chosen: str
  alternatives: [str]
  rationale: str
  still_valid: bool       # updated if new evidence arrives

validation:
  target: artifact_id     # what was being validated
  metrics: jsonb
  conditions: jsonb
  statistical_significance: str
  reproducible: bool
```

### Knowledge Graph Edges

```sql
edges:
  id              UUID PRIMARY KEY
  source_id       UUID REFERENCES artifacts
  target_id       UUID REFERENCES artifacts
  relation        ENUM(
    -- epistemic
    supports, contradicts, extends, supersedes, replicates,
    -- methodological
    uses_method_from, adapts_pipeline_from, shares_constraint_with,
    -- structural
    part_of, depends_on, derived_from, validates,
    -- cross-domain (the high-value ones)
    transfers_technique_to, analogous_to, enables, blocks
  )
  confidence      FLOAT
  domains         TEXT[]         -- which domains this edge spans
  is_cross_domain BOOLEAN        -- true if source and target are in different domains
  created_at      TIMESTAMP
  created_by      ENUM(ingest_classifier, synthesis_agent, user, query_discovery)
  note            TEXT           -- optional context for why this edge exists
```

### Domain Registry

```sql
domains:
  id              TEXT PRIMARY KEY  -- e.g., 'edge_ml', 'bioinformatics'
  display_name    TEXT              -- e.g., 'Edge ML / TinyML / Edge AI'
  description     TEXT
  parent_domain   TEXT              -- for hierarchical domains
  related_domains TEXT[]            -- known cross-domain connections
  keywords        TEXT[]            -- for auto-classification
```

Initial domains:
- `bioinformatics` -- pipelines, sequence analysis, alignment, annotation
- `robotics_ml` -- robot perception, control, planning with ML
- `edge_ml` -- TinyML, edge AI, on-device inference, quantization, pruning
- `health_monitoring` -- wearable sensors, continuous monitoring, clinical algorithms
- `algorithm_discovery` -- ML training, validation, hyperparameter optimization, NAS
- `multi_omics` -- genomics + proteomics + metabolomics integration, biomarker discovery

## Cross-Domain Bridge Detection

The most valuable insights for a multi-domain researcher are connections *across* domains. The system actively hunts for these:

### At Ingest (Tier 1)
- When a new artifact is classified into domain A, check semantic similarity against artifacts in domains B, C, D...
- If similarity > threshold on a cross-domain pair, create a candidate bridge edge
- Tag candidate bridges for review in the next Tier 2 synthesis

### At Synthesis (Tier 2)
- Dedicated "bridge agent" walks candidate cross-domain edges
- Evaluates whether the connection is meaningful: shared method, analogous problem, transferable technique, enabling dependency
- Promotes meaningful bridges to confirmed edges with typed relations
- Compiles a "Cross-Domain Bridges" wiki page showing recent connections

### Bridge Categories

| Category | Description | Example |
|---|---|---|
| Method transfer | Same algorithm/technique applies in different domain | Quantization-aware training (edge ML) → portable sequencer analysis (bioinformatics) |
| Constraint sharing | Same physical/computational constraint appears | Latency budget from robotics inference ≈ latency budget for wearable health device |
| Pipeline reuse | A pipeline structure from one domain works in another | Bioinformatics alignment pipeline adapted for multi-omics integration |
| Biomarker→sensor | Discovery in one domain creates a target for another | Multi-omics biomarker → real-time health monitoring algorithm |
| Validation pattern | A validation strategy from one domain strengthens another | ML cross-validation methodology applied to multi-omics biomarker discovery |

## Hypothesis Lifecycle

Hypotheses are living artifacts that evolve over time:

```
PROPOSED → ACTIVE → SUPPORTED / WEAKENED → VALIDATED / INVALIDATED
                          ↓
                       MERGED (combined with another hypothesis)
```

- When new evidence arrives (paper, experiment result), the synthesis agent checks if it supports or contradicts any active hypothesis
- Hypothesis confidence is updated incrementally
- The "Hypotheses" wiki view shows all active hypotheses ranked by confidence, with evidence trails
- Invalidated hypotheses are kept (not deleted) -- knowing what was wrong and why is valuable

## Consequences

### What This Enables
- Context recovery in <30 seconds: switch to a domain, see compiled wiki page with current state
- Cross-domain insight surfacing: bridge detection finds connections you might miss
- Experimental memory: failed experiments are first-class, searchable, with failure reasons
- Decision trail: revisit a domain after months and understand not just what but why

### What This Requires
- Rich metadata extraction at ingest (not just text → embedding)
- Typed edge relations (not just "related to")
- A bridge detection agent that runs cross-domain similarity checks
- Hypothesis tracking as an ongoing, evolving data model
- Code/config parsing for non-text artifacts (YAML, JSON, Jupyter)
