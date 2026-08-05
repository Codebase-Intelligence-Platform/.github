# Modules

These are the logical modules of the CodeLens MVP system, not a microservices map. Per GOALS.md's trade-off table, the MVP is an in-process analysis engine (SQLite + NetworkX, no graph database) plus a web client — the module boundaries below are code-organization boundaries, not deployment boundaries, except where noted. See GLOSSARY.md for module/package/component terminology.

## Ingestion & Parser

**Responsibility:** Read a repository from a local path or token-authenticated clone, walk the file tree respecting include/exclude patterns, detect languages, invoke the tree-sitter parser for each file, and produce the raw per-file intermediate representation (IR) — file, class/interface, and function/method-level nodes, plus within-file structural edges. Also computes a content hash per file during ingestion and compares it against the last analyzed commit for the same repository (US-1.4), so a re-ingest re-parses only files whose content actually changed and passes the rest through unmodified.

**Non-responsibility:** Does not resolve cross-file imports or calls — that belongs to Graph Core, which has visibility across the whole repository. Does not perform clustering, cycle detection, or impact analysis. Does not know about the UI. Makes no network calls beyond the initial clone/fetch. Does not decide how a changed-file set gets applied to the graph — it identifies what changed; Graph Core patches the graph.

**Public interface:**
- `parse_file(path, language) -> FileIR`
- `ingest_repository(path, include, exclude) -> IngestionReport`
- `diff_changed_files(repo_path, since_commit) -> list[ChangedFile]` — content-hash comparison against a previously analyzed commit (US-1.4)

Emits `FileIR` objects consumed by Graph Core.

**Technology:** tree-sitter grammars for Python and TypeScript (GOALS.md trade-off table — uniform CST interface across both MVP languages).

## Graph Core (IR, Storage, Algorithms)

**Responsibility:** Own the unified intermediate representation — repository, directory, package, file, class/interface, function/method node levels; contains, imports, calls, extends, implements edges. Resolve cross-file imports and calls where statically possible, assign each edge a confidence level (EPICS.md, Confidence Levels), persist the graph, and expose read/write/query access plus general graph algorithms (path finding, degree/betweenness centrality) to the rest of the system. Also exposes definition and reference lookup over the edges it already resolved (US-4.3) — go-to-definition and find-references are queries over existing `contains`/`imports`/`calls`/`extends`/`implements` edges, not a new analysis subsystem, grouped using the module- and cycle-membership annotations Analysis has already written back onto the graph.

**Non-responsibility:** Does not decide what counts as "impacted" for a PR diff — that's Analysis. Does not run community detection or cycle detection itself, though it hosts the NetworkX graph object those algorithms run over, and reads (but does not compute) the module/cycle annotations Analysis writes back for grouping reference results. Does not render anything.

**Public interface:**
- `resolve_and_merge(file_ir) -> GraphPatch`
- `get_subgraph(node_ids, depth) -> Graph`
- `find_paths(source, target, max_depth) -> list[Path]`
- `find_definition(symbol_id) -> list[Candidate]` — candidates carry their edge's confidence level; a single `resolved` candidate is the definition, otherwise all candidates are returned (US-4.3)
- `find_references(symbol_id) -> list[Reference]` — grouped by inferred module, with cycle-crossing references flagged (US-4.3)
- `apply_patch(patch)` — for incremental updates (Epic 9), and for the changed-file set a cache-aware re-ingest produces (US-1.4)

**Technology:** SQLite for persisted node/edge/metadata storage; NetworkX in-process as the queryable graph structure, held in memory and rehydrated from SQLite on startup (GOALS.md trade-off table — no graph database in the MVP; migrate only when a measured wall is hit).

## Analysis (Impact, Clustering, Cycles)

**Responsibility:** Diff-aware impact analysis (changed lines → changed symbols → reverse-reachability closure → ranked impact set, GOALS.md Goal 3); community detection (Louvain/Leiden) producing inferred modules; divergence computation against declared directories; Tarjan's SCC for cycle detection; and the Epic 6 risk-factor breakdown.

**Non-responsibility:** Does not parse source files or resolve imports — reads the graph Graph Core already resolved. Does not know about HTTP/WebSocket transport or UI rendering. Does not persist its own copy of the graph; it reads from Graph Core and writes results back as graph annotations (module membership, cycle membership, impact-set flags).

**Public interface:**
- `compute_impact(diff) -> ImpactSet`
- `detect_communities(graph) -> list[InferredModule]`
- `compute_divergence(communities, directories) -> list[Divergence]`
- `detect_cycles(graph) -> list[Cycle]`
- `risk_factors(impact_set) -> RiskFactorBreakdown`

**Technology:** NetworkX (`networkx.algorithms.community` for Louvain/Leiden, `networkx.strongly_connected_components` for Tarjan's SCC), plain Python for the diff-to-symbol mapping and impact ranking logic.

## API Layer

**Responsibility:** Expose ingestion, graph query, search, impact analysis, and evaluation-harness results to the web client over HTTP, and push incremental graph updates over WebSocket within the ~2 second budget (Epic 9).

**Non-responsibility:** Does not implement parsing, graph algorithms, or impact logic — it calls into Graph Core and Analysis and serializes their results. Does not embed business rules that duplicate decisions those modules already make (e.g., it does not itself decide what counts as an impacted file).

**Public interface:** REST endpoints for repository connection, ingestion status, graph/subgraph queries, search, and PR impact analysis; one WebSocket channel streaming incremental graph patches (US-9.3).

**Technology:** FastAPI (Python) — kept in the same language as Ingestion/Graph Core/Analysis to avoid a cross-language serialization boundary in the MVP.

## React Web Client

**Responsibility:** Render the architecture graph (module-level default view, collapse/expand, path tracing — Epic 4), the progressive/incomplete-graph state during ingestion (US-1.3), the search UI (Epic 5), the code view with go-to-definition and grouped find-references (US-4.3), the PR impact/blast-radius view with diff overlay (Epic 6), the four Epic 7 charts, and the confidence/limitations indicators from Epic 10.

**Non-responsibility:** Does not compute impact sets, clusters, or cycles — it only renders what the API layer returns. Does not reinterpret or hide confidence levels, and does not omit the Epic 10 limitations indicator (US-10.2) when inferred edges are present in what it's displaying. For the code view specifically: it renders the definition candidates and grouped references Graph Core already resolved — it does not itself walk edges, resolve bindings, or decide module grouping.

**Public interface:** Consumes the API layer's REST endpoints and WebSocket channel. Exposes no interface to other modules — it is the leaf of the dependency chain.

**Technology:** React, with a graph-rendering approach capable of hierarchical collapse/expand at the module/package/file/symbol granularity (GOALS.md Goal 6). Charting library choice is left to implementation; GOALS.md does not mandate one.

## Evaluation Harness

**Responsibility:** Build and store the ground-truth PR dataset (Epic 0, US-0.2), replay each PR's first-commit diff through Analysis's `compute_impact`, and compute and report precision, recall, and F1 against the later-commit ground truth.

**Non-responsibility:** Does not modify the pinned demonstration repository's history. Does not feed ground-truth data back into the impact-analysis algorithm at evaluation time — doing so would invalidate the measurement. Does not run as part of the live product path; it is a development and reporting tool.

**Public interface:** A single command-line entry point producing a machine-readable results file (US-0.3). Reads the ground-truth dataset and calls the same `compute_impact` interface the API layer uses, so the harness measures exactly what a user of the product would see.

**Technology:** Python script(s) driving the Analysis module's public interface directly (no HTTP hop); results stored as JSON/CSV per US-0.3.
