# Explicit Assumptions

Documenting assumptions upfront because requirement errors become dramatically more costly in later phases. Each assumption should be validated or invalidated during discovery.

## Architecture Assumptions

| ID | Assumption | Confidence | Validation Method | Status |
|---|---|---|---|---|
| A1 | PostgreSQL + pgvector can serve as both relational store and vector store for our scale | High | Benchmark with 10K documents | Untested |
| A2 | A knowledge graph stored in relational tables (not a dedicated graph DB) is sufficient for our relation complexity | Medium | Prototype with 1K edges, measure query performance | Untested |
| A3 | Markdown is a viable intermediate format for compiled wiki views | High | Generate 50 pages, test in Obsidian + web renderer | Untested |
| A4 | LLM-based contradiction detection at ingest time can achieve >0.7 precision without being too expensive | Medium | Test with 100 known-contradictory fact pairs | Untested |
| A5 | Incremental wiki compilation (only recompile changed topics) is feasible and produces consistent results | Medium | Compile 10 topics, change 2, recompile, check consistency | Untested |

## User Assumptions (Solo Researcher)

| ID | Assumption | Confidence | Validation Method | Status |
|---|---|---|---|---|
| U1 | A solo researcher will check provenance when making technical decisions (p-values, model params) | High | Self-test with real research workflow | Untested |
| U2 | Context recovery (<30s to re-enter a domain) is more valuable than search quality | High | Time current context-switching cost vs system-assisted | Untested |
| U3 | Cross-domain connections are the highest-value output (not within-domain summaries) | High | Track which wiki views are accessed most | Untested |
| U4 | Artifact volume is 5-30/week across all domains, with bursts during active research phases | Medium | Log actual ingest rate for 1 month | Untested |
| U5 | Failed experiments are referenced as often as successful ones when making decisions | Medium | Track access patterns for experiment artifacts by outcome | Untested |
| U6 | The researcher uses 3+ tools (Claude, Cursor, Jupyter, etc.) and fragments knowledge across them | High | Audit current tool usage | Untested |
| U7 | Hypothesis tracking across domains is a workflow the researcher will maintain (not abandon after novelty wears off) | Medium | Test with 10 real hypotheses over 2 months | Untested |

## Cost Assumptions

| ID | Assumption | Confidence | Validation Method | Status |
|---|---|---|---|---|
| C1 | Tier 1 ingest can stay under $0.01/document using embedding models + lightweight classification | High | Measure actual API costs for 100 documents | Untested |
| C2 | Tier 2 synthesis for one topic cluster costs < $1.00 with current LLM pricing | Medium | Run compiler agent on 5 real topic clusters | Untested |
| C3 | Graph updates are computationally cheap relative to LLM calls | High | Profile graph operations | Untested |

## Process Assumptions

| ID | Assumption | Confidence | Validation Method | Status |
|---|---|---|---|---|
| P1 | Starting with discovery phase will reduce total rework cost | High | Track assumption invalidations vs what they would have cost post-build | Ongoing |
| P2 | ADRs are sufficient for capturing architecture decisions at this stage | High | Review after 3 months -- are they referenced? Updated? | Ongoing |
| P3 | A prototype with 50-100 documents is representative enough to validate the architecture | Medium | Compare prototype behavior with eventual real-scale behavior | Future |
