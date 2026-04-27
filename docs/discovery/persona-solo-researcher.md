# Persona: Solo Researcher with Multiple Deep Domains

**Date**: 2026-04-27  
**Status**: Primary target user

## Profile

A solo researcher who works across multiple technically deep, partially overlapping domains simultaneously. Not a generalist skimming surfaces -- deeply engaged in each domain, but constantly context-switching between them.

## Active Domains

```
                        ┌─────────────────┐
                        │   Robotics      │
                        │   AI/ML         │
                        └───────┬─────────┘
                                │ inference at the edge
            ┌───────────────────┼───────────────────┐
            │                   │                   │
   ┌────────▼────────┐  ┌──────▼──────────┐  ┌─────▼──────────┐
   │  Edge ML /      │  │  Health          │  │  Bioinformatics │
   │  TinyML /       │  │  Monitoring      │  │  Pipelines      │
   │  Edge AI        │  │  Pipelines       │  │                 │
   └────────┬────────┘  └──────┬──────────┘  └─────┬──────────┘
            │                   │                   │
            │    signal processing, validation      │
            │           ┌──────▼──────────┐         │
            └──────────►│  Algorithm      │◄────────┘
                        │  Discovery /    │
                        │  Training /     │
                        │  Validation     │
                        └──────┬──────────┘
                               │
                        ┌──────▼──────────┐
                        │  Multi-Omics    │
                        │  Discovery &    │
                        │  Validation     │
                        └─────────────────┘
```

### Cross-Domain Connections (the high-value edges)

These are the connections a conventional tool would miss entirely:

| From | To | Connection Type | Example |
|---|---|---|---|
| Edge ML | Health Monitoring | Deployment target | TinyML model running inference on a wearable sensor |
| Health Monitoring | Algorithm Discovery | Method transfer | Signal processing technique from ECG applied to genomic time-series |
| Bioinformatics | Multi-Omics | Pipeline reuse | Alignment pipeline adapted for proteomics + transcriptomics integration |
| Robotics AI/ML | Edge ML | Constraint sharing | Latency/power constraints from robotics inference apply to edge health devices |
| Algorithm Discovery | Multi-Omics | Validation patterns | Training/validation split strategy from ML applied to multi-omics biomarker discovery |
| Multi-Omics | Health Monitoring | Biomarker→sensor | Discovered biomarker becomes target for a real-time monitoring algorithm |
| Edge ML | Bioinformatics | Compute constraints | Quantization techniques for edge models applied to portable sequencing analysis |

## Core Pain: Context Switching Cost

The primary problem is not "I can't find information" -- it's **"every time I switch domains, I spend 30-60 minutes rebuilding context that I had last week."**

This manifests as:
1. Re-reading papers you've already synthesized
2. Forgetting which approach you decided against (and why) in a different domain
3. Missing cross-domain connections because you're deep in one domain when the insight from another would be relevant
4. Losing experimental context: "I tried X configuration last month, it failed because of Y, but I can't find my notes"
5. Duplicating analysis: running the same comparison you ran 3 weeks ago because you don't remember the result

## Artifact Types

This researcher doesn't just read papers. The system must handle:

| Artifact Type | Examples | Key Metadata |
|---|---|---|
| Papers/preprints | arXiv, bioRxiv, conference papers | Authors, venue, date, claims, methods |
| Code references | GitHub repos, specific functions, pipeline configs | Language, framework, dependencies, what it solves |
| Experimental notes | "Tried X, got Y result because Z" | Date, domain, outcome (success/failure/partial), parameters |
| Model architectures | Network designs, quantization configs, training recipes | Task, constraints (latency, memory, power), performance metrics |
| Dataset descriptions | Multi-omics datasets, sensor data, benchmarks | Modalities, size, access method, known issues |
| Pipeline definitions | Bioinformatics workflows, training pipelines, deployment configs | Steps, dependencies, runtime, known failure modes |
| Hypotheses | "I think X because of evidence Y and Z" | Confidence, supporting evidence, contradicting evidence, status |
| Decision records | "I chose approach A over B because C" | Date, domain, alternatives considered, rationale |
| Validation results | Benchmark runs, statistical tests, ablation studies | Metrics, conditions, statistical significance, reproducibility notes |

## What Success Looks Like

1. **Instant context recovery**: Switch from multi-omics to edge ML and within 30 seconds have a compiled page showing current state of understanding, open questions, recent experimental results, and connections to other domains
2. **Cross-domain insight surfacing**: The system proactively notes "your new edge ML paper on quantization-aware training is relevant to the bioinformatics pipeline you're optimizing for portable sequencers"
3. **Experimental memory**: "You tried this hyperparameter configuration on 2026-03-15 and it failed because of gradient instability at low precision -- here's your note"
4. **Hypothesis tracking**: See all your active hypotheses across domains, with supporting/contradicting evidence accumulating automatically as you ingest new sources
5. **Decision trail**: When you revisit a domain after 2 months, understand not just what you decided but *why*, and whether new evidence has changed the picture

## Implications for Architecture

### Simplifications (vs team/enterprise)
- **No multi-user conflicts**: Single writer, so wiki compilation races and merge conflicts are irrelevant
- **No access control**: Everything is yours, no permissions layer needed
- **No organizational politics in synthesis**: The AI serves one mental model, not competing stakeholders
- **Cost tolerance is personal budget**: Optimize for value per dollar, not enterprise pricing

### Complications (vs simpler use cases)
- **Domain taxonomy is fluid**: New sub-domains emerge as research evolves; the graph schema must be flexible, not hardcoded
- **Cross-domain edges are the core value**: The graph must actively identify connections across domain boundaries, not just within them
- **Technical precision is non-negotiable**: Lossy summarization that drops a p-value, a model architecture detail, or a pipeline parameter is actively harmful
- **Artifacts are heterogeneous**: Papers, code, experimental notes, model configs -- not just "documents"
- **Temporal evolution matters**: A hypothesis from 3 months ago may be invalidated; understanding evolves and the system must show that evolution, not just the latest state
- **Provenance is personal memory**: "Where did I read that?" is a constant question; every synthesized claim must trace back to source

### Key Design Consequences

1. **The graph needs typed edges with domain annotations**: Not just "related to" but "uses same method as", "contradicts finding in", "deploys model from", "validates hypothesis from"
2. **Wiki views should be per-domain + a cross-domain connections page**: Each domain gets its own compiled view, but there's an explicit "bridges" view showing inter-domain connections
3. **Experimental notes are first-class, not afterthoughts**: Failed experiments are as valuable as successful ones; the system must treat "I tried X and it didn't work because Y" as a high-value artifact
4. **Hypothesis tracking needs its own data model**: Hypotheses accumulate evidence over time across domains; they're not static facts
5. **Ingest must handle code and configs, not just text**: Pipeline YAML, model architecture definitions, Jupyter notebooks -- these carry structured information that pure text extraction would mangle
