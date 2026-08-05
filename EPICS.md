# Epic User Stories

This document lists every user story for CodeLens with testable acceptance criteria. Every story's tier (MVP, Phase 2, or Deferred) is recorded in [SCOPE.md](./SCOPE.md), not here — this file describes what a story does when it is built, not when. Terms below (module, component, blast radius, confidence, etc.) follow [GLOSSARY.md](./GLOSSARY.md).

Several stories below were originally written to the full problem statement rather than the MVP defined in [GOALS.md](./GOALS.md). Where that happened, the story is downgraded to what the MVP actually supports rather than deleted, and the downgrade is noted inline. See [SCOPE.md](./SCOPE.md) for the full accounting.

## Definition of Done

A story is done when:

1. Every acceptance criterion is met and has been checked against the pinned demonstration repository (SCOPE.md), not a synthetic toy example only.
2. Every numeric acceptance criterion (latency, throughput, node count, recall floor) has been measured, not estimated, and the measurement is reproducible.
3. Confidence levels (below) are attached to every edge or flow the story produces, wherever the story's acceptance criteria mention confidence.
4. The story's tier in SCOPE.md is MVP. If a story turns out to depend on a Phase 2 or Deferred capability, the dependency is recorded and the story is re-tiered rather than partially shipped.
5. Any limitation from Epic 10 that applies to the story's output is either handled or explicitly surfaced in the UI per US-10.2.

## Confidence Levels

Every edge and reconstructed flow in the graph carries one of three confidence levels, assigned at the point the edge is created:

| Level | Assigned when | Included in impact analysis? |
|---|---|---|
| `resolved` | The reference was statically bound to a unique declaration — e.g., an import resolved to a specific file/symbol, or a call bound to a symbol reachable through a resolved import. | Yes, full weight. |
| `heuristic` | The reference matches a declaration by name or convention but could not be uniquely proven — e.g., a call matching multiple candidate declarations, or an import resolved by directory convention rather than an explicit resolver. | Yes, included and visually marked. |
| `inferred` | The reference is asserted from a weak signal only — e.g., a dynamic-looking construct partially matched to a symbol name. | Yes, included and visually marked. |

CodeLens over-approximates: uncertain edges are included, not dropped, and always marked with their confidence level. This is a deliberate trade-off from `GOALS.md` — missing a truly impacted file is worse than flagging an extra one that turns out not to matter. Every acceptance criterion below that mentions "confidence" refers to this three-level scale.

---

# Epic 0: Ground Truth and Evaluation Harness

## Epic Goal

Establish a measured baseline for impact-set accuracy before building further features on top of an unverified graph. This is the single highest-value addition to the original epic set (GOALS.md: "the thing most teams skip").

## US-0.1: Select and Pin the Demonstration Repository

**As a developer, I want to select and pin one demonstration repository so that all analysis, evaluation, and the final demo are measured against a stable, known target.**

### Acceptance Criteria

- A single repository is selected as the pinned demonstration repository for the project (see SCOPE.md; a `TODO(decision)` marker with selection criteria stands in until this is resolved).
- The repository predominantly uses the two MVP languages: Python and TypeScript.
- The repository has at least 30 merged pull requests with multi-commit history, sufficient to build the Epic 0 ground-truth dataset.
- The pinned repository's name, URL, and analyzed commit range are recorded in SCOPE.md and do not change without an explicit, recorded decision update.
- Repository size (file count, approximate lines of code) is documented to contextualize every later performance number in this document.

## US-0.2: Build the Ground-Truth PR Dataset

**As a developer, I want a ground-truth dataset of historical merged PRs so that impact predictions can be checked against what actually changed.**

### Acceptance Criteria

- 30–50 historical merged pull requests are extracted from the pinned demonstration repository.
- For each PR, the dataset records: the first commit's diff, every file touched by any later commit in the same PR, and which of those files are test files.
- The dataset is stored in a machine-readable format (JSON) versioned alongside the evaluation harness.
- PRs with fewer than 2 commits are excluded — there is nothing for the harness to predict against.
- Dataset generation is reproducible from a single script run against the pinned repository.

## US-0.3: Run the Impact-Prediction Evaluation Harness

**As a developer, I want a harness that measures impact-set accuracy against the ground-truth dataset so that I can report real precision and recall, not an assertion.**

### Acceptance Criteria

- The harness runs from a single command (e.g., `make eval` or an equivalent single script invocation) with no manual steps between dataset and result.
- For each PR in the ground-truth dataset, the harness feeds CodeLens only the first commit's diff and captures the predicted impact set.
- The harness compares the predicted impact set against the files and test files actually touched by later commits of the same PR.
- Per-PR and aggregate precision, recall, and F1 are computed.
- Results are emitted as a machine-readable file (JSON or CSV) with one row per PR plus an aggregate summary row.
- A full run against the pinned repository's PR set completes in under 30 minutes on a single developer machine, so it can be re-run during development.

## US-0.4: Report the Baseline Accuracy

**As a developer, I want the evaluation results surfaced in the project's documentation so that the impact analysis is measured, not just demonstrated.**

### Acceptance Criteria

- Aggregate precision, recall, and F1 from US-0.3 are recorded in a results summary alongside the pinned repository and commit range used.
- The baseline is re-run and re-recorded whenever the impact-analysis algorithm (GOALS.md Goal 3) changes.
- The recall figure is the number quoted in the demo (WORKFLOW.md's closing step); precision is reported alongside it, never omitted.
- If recall falls below the target floor stated in US-6.2 (0.7), this is stated explicitly in the report rather than the number being left out.

---

# Epic 1: Connect and Analyze a Monorepo

## Epic Goal

Allow a developer to connect one repository and prepare it for architecture analysis. (Polyrepo is Phase 2 — see SCOPE.md; the MVP analyzes a single repository, monorepo or not.)

## US-1.1: Connect a Repository

**As a developer, I want to connect a repository so that CodeLens can analyze it.**

*Downgraded from GitHub OAuth to local-path / token-clone connection — full OAuth with a repository picker is real integration effort with little value for demonstrating the core reconstruction-and-impact claim (see SCOPE.md). Full OAuth is Phase 2.*

### Acceptance Criteria

- The user provides either a local filesystem path to an existing clone, or a Git remote URL plus a personal access token.
- CodeLens validates the path or clone before ingestion begins: the local path exists and is a Git repository, or the remote URL clones successfully with the supplied token.
- Any supplied token is stored encrypted at rest and is never logged or displayed after entry.
- The user selects the branch or commit to analyze; the default is the repository's current HEAD.
- The repository's resolved commit SHA is recorded and displayed after connection.

## US-1.2: Configure and Ingest the Repository

**As a developer, I want CodeLens to identify and ingest relevant directories so that generated and irrelevant files are excluded from analysis.**

*Downgraded: "applications, services, libraries" as distinct detected node types are Phase 2 (see GLOSSARY.md — the MVP graph does not model applications or services separately from packages). This story covers packages and directories.*

### Acceptance Criteria

- CodeLens detects top-level packages and source directories using file-extension and manifest heuristics (e.g., `package.json`, `pyproject.toml`, `setup.py`, `requirements.txt`).
- Users can define include and exclude path glob patterns before ingestion runs.
- `node_modules`, `dist`, `build`, `.git`, vendor directories, virtual environments, and other common generated-file directories are excluded by default.
- The interface displays ingestion progress — files discovered, files parsed, files skipped — updated at least once per second or per completed file batch.
- A single file's parse failure does not abort ingestion of the remaining files.
- Ingestion of the full pinned demonstration repository completes in under 5 minutes on a single developer machine.
- On completion, the user sees exactly one of: success (all files parsed), partial success (N files failed, each listed with a reason), or failure (ingestion could not proceed) — always with a numeric count of parsed vs. skipped files.

## US-1.3: Progressive Graph Rendering During Ingestion

**As a developer connecting a repository for the first time, I want to see and start exploring a usable graph while analysis is still running, so that ingestion feels immediate rather than blocking.**

*Import edges resolve fast; call resolution is the slow part. Package-level nodes and import edges can therefore render while symbol extraction and call resolution continue underneath — this story is about sequencing what US-1.2 and US-3.1 already produce, not computing anything new or faster.*

### Acceptance Criteria

- A first meaningful graph — package and directory nodes plus import edges — renders within 10 seconds of ingestion starting, well inside the 5-minute ingestion budget (US-1.2) and the 60-second post-parse graph-build budget (US-3.1). Ten seconds is chosen because import-edge extraction is a lightweight, single-pass scan per file that doesn't require the symbol- and call-resolution work the longer budgets account for, so it's achievable without a new code path, and it's short enough that the user never perceives ingestion as stalled.
- The graph populates incrementally as parsing proceeds, reusing the Epic 9 patch mechanism (`apply_patch` / the WebSocket channel, `MODULES.md`) rather than a second, parallel streaming path.
- The view remains interactable — zoom, pan, expand — throughout ingestion, and stays inside the 300-node render ceiling from US-4.1 at every point, not just once ingestion completes.
- A persistent, clearly visible indicator states that the graph is incomplete while ingestion is running, and names which analyses aren't valid yet.
- Community detection (US-8.1), divergence (US-8.2), and cycle detection (US-8.3) are shown as pending, not computed against a partial graph and silently corrected later — a module boundary that quietly changes underneath the user is worse than an honest "still computing."
- Impact analysis (Epic 6) is unavailable, not merely inaccurate, until the graph is complete.

## US-1.4: Cache and Re-Ingest

**As a developer, I want a previously analyzed repository to load without a full rebuild, so that reconnecting or moving to a newer commit is fast.**

### Acceptance Criteria

- CodeLens detects that a repository and commit have already been analyzed, keyed on content (e.g., a hash of the repository tree at that commit) rather than on the local path or remote URL used to connect it, and rehydrates the persisted graph from SQLite into NetworkX (`MODULES.md`, Graph Core) instead of re-parsing. A cached repository/commit loads in under 5 seconds for the pinned demonstration repository — this is I/O and in-memory graph reconstruction, not re-analysis, so it sits well inside every parsing or graph-build budget.
- **Same repository, newer commit:** only files whose content hash changed since the last analyzed commit are re-parsed, and the graph is patched rather than rebuilt, reusing Epic 9's incremental machinery (US-9.1–US-9.3) rather than a parallel mechanism. This is the criterion that matters most — it makes incremental analysis work across commits, not just across file saves, which is what a real PR review needs.
  - `TODO(decision):` Epic 9's ~2 second end-to-end budget (US-9.2) is defined for a single-file save. Moving between commits can change an arbitrary number of files, and applying the per-file re-parse budget (500ms, US-9.1) file-by-file to, say, a 50-file commit would take well over 2 seconds — this criterion cannot inherit the single-file budget unchanged. Whether the resolution is a budget stated per changed file rather than end-to-end, a cap on file count before falling back to US-1.3's progressive-rendering pattern, or something else, is not resolved here.
- Cache invalidation is explicit, not inferred: a parser version change, an IR schema change (`platform-contracts`), an include/exclude pattern change, or a change to an algorithm that affects stored annotations (community membership, cycle membership) all invalidate the cache. A stale cache that silently serves a graph built under different rules is a correctness bug, not a performance one.
- If the cache is corrupt or unreadable, CodeLens falls back to a full rebuild and states that it has done so, rather than failing silently or serving partial data.
- A manual force-rebuild path exists, bypassing the cache regardless of whether it appears valid.

---

# Epic 2: Parse a Multi-Language Codebase

## Epic Goal

Extract a normalized representation from the two MVP languages inside the repository.

## US-2.1: Detect and Parse Supported Languages

**As a developer, I want CodeLens to detect and parse every supported language in the repository so that the complete architecture can be reconstructed.**

*Downgraded: "every supported language" is exactly two languages in the MVP — Python and TypeScript (GOALS.md Goal 1). Additional languages are Deferred (SCOPE.md).*

### Acceptance Criteria

- Python and TypeScript files are detected automatically by extension and, where ambiguous, by content sniffing.
- Parsing uses tree-sitter grammars for both languages, giving a uniform CST interface (GOALS.md trade-off table).
- A parser failure in one file does not prevent any other file from parsing (verified: a syntax error injected into one file among N leaves the other N−1 files parsed successfully).
- Language distribution (file counts, parsed vs. unsupported) is stored and shown in the interface.
- Files in languages other than Python or TypeScript are recorded as unsupported and do not block analysis.
- Parsing of the full pinned demonstration repository completes in under 5 minutes on a single developer machine (same budget as ingestion in US-1.2, since the MVP pipeline parses during ingestion).

## US-2.2: Extract Symbols and Relationships

**As a developer, I want CodeLens to extract symbols and code relationships so that I can understand how the repository is structured and connected.**

*Downgraded: API endpoint extraction is Phase 2, tied to the deferred API/service detection capability (SCOPE.md).*

### Acceptance Criteria

- The system extracts packages, classes, interfaces, functions, and methods — the module/class and function/symbol levels from GOALS.md Goal 1.
- Imports and exports are extracted for both languages.
- Function and method calls are extracted where statically resolvable; each extracted call carries a confidence level (`resolved` or `heuristic`).
- Local (in-repository) and external dependencies are distinguished.
- Source file, line number, language, and symbol type are retained for every extracted symbol and relationship.
- Relationships that cannot be statically resolved are labeled `inferred`, never presented as `resolved`.
- Both languages produce the same normalized intermediate representation, queryable without knowing which language a given node came from.
- API endpoint extraction is Phase 2 — see SCOPE.md.

---

# Epic 3: Reconstruct the Architecture

## Epic Goal

Convert parsed source-code information into a unified architecture graph.

## US-3.1: Build the Architecture Graph

**As a developer, I want CodeLens to reconstruct an architecture graph so that I can understand the repository beyond its folder structure.**

*Downgraded: node and edge types are trimmed to what the MVP's ingestion, parsing, and analysis modules actually produce. Service, application, API endpoint, and database-entity nodes, and the reads-from/writes-to/communicates-with edges, are Phase 2 — see SCOPE.md.*

### Acceptance Criteria

The graph contains these node types in the MVP:

- repository
- directory
- package
- file
- class
- interface
- function / method
- external dependency

The graph contains these edge types in the MVP:

- contains
- imports
- calls
- extends
- implements
- depends on (an aggregate view over imports/calls between packages)

Phase 2 additions (not built in the MVP — see SCOPE.md): application and service nodes, API endpoint and database entity nodes, and reads-from / writes-to / communicates-with edges.

Additional criteria:

- Duplicate nodes are prevented (a symbol parsed from the same file and location twice produces one node, not two).
- Every edge retains evidence of its source file and location.
- Every edge carries a confidence level (`resolved`, `heuristic`, or `inferred`); confirmed and inferred relationships are visually distinguishable in any rendering of the graph.
- The graph is associated with the analyzed commit SHA.
- Building the full graph for the pinned demonstration repository completes within 60 seconds of parse completion (US-2.1).

## US-3.2: Identify Architectural Boundaries and Flows

**As a developer, I want CodeLens to identify structural boundaries so that I can understand the major units of the repository before looking at inferred modules.**

*Downgraded: this story covers declared/structural grouping only (packages, directories, dependency direction). Inferred-module clustering is Epic 8. API-to-handler paths, service-to-service communication, and API-to-database paths are Phase 2 — see SCOPE.md.*

### Acceptance Criteria

- Top-level packages and directories are identified from the graph's `contains` hierarchy and package manifests.
- Packages are grouped using directory structure, package manifests (`package.json`, `pyproject.toml`, etc.), and declared dependency relationships — not inference.
- Internal (in-repository) and external dependencies are distinguished for every package.
- Every reconstructed flow carries a confidence level (see Confidence Levels above).
- API-to-handler path reconstruction, service-to-service communication detection, and API-to-database path display are Phase 2 — see SCOPE.md.

---

# Epic 4: Explore the Architecture in React

## Epic Goal

Provide a responsive React interface for architecture exploration, charts, and engineering insights.

## US-4.1: Explore the Interactive Architecture Graph

**As a developer, I want to interactively explore the architecture graph so that I can move from a system overview to implementation-level details.**

*Downgraded: the default view groups by package and inferred module (Epic 8), not by "application/service" — those node types don't exist in the MVP graph.*

### Acceptance Criteria

- The initial view shows top-level packages and inferred modules (Epic 8 community-detection clusters); individual files and symbols are collapsed.
- Users can zoom, pan, expand, collapse, and reposition nodes.
- Users can filter by language, node type, package, and inferred module.
- If an expanded view would render more than 300 nodes simultaneously, the excess stays collapsed and the user is prompted to filter or drill down further — the graph never renders the whole symbol-level repository at once (GOALS.md Goal 6).
- Selecting a node displays its metadata, dependencies, dependents, and source location.
- Users can open the associated source file.
- Expanding a node loads its immediate subgraph within 1 second for the pinned demonstration repository.

## US-4.2: Trace Dependencies and Call Paths

**As a developer, I want to trace dependencies and call paths so that I can understand how two components are connected.**

### Acceptance Criteria

- Users can select a start node and destination node.
- The interface displays one or more available paths between them.
- Relationship types are visible on every path.
- Direct and transitive dependencies are distinguishable (see GLOSSARY.md).
- Recursive and cyclic paths are marked, cross-referencing Epic 8's cycle detection where applicable.
- Path depth is configurable, with a default maximum of 5 hops and an adjustable ceiling of 15.
- Inferred connections display their confidence level.

## US-4.3: Code View with Definitions and References

**As a developer, I want to open a symbol in a code view and navigate to its definition and references, so that I can move between architecture-level and line-level understanding without leaving CodeLens.**

*This story adds a view over data Graph Core already produces — the `resolved`-confidence edges in the graph are a reference index (`MODULES.md`). It does not introduce a new analysis subsystem, type inference, an LSP, or a SCIP indexer; `GOALS.md`'s trade-off table rules out type inference for the MVP, and this story must not quietly reintroduce it under a different name.*

### Acceptance Criteria

- A code view shows the source of a selected file with the selected symbol highlighted. It is reachable from a graph node (US-4.1), a search result (US-5.1), and an impact-set entry (US-6.2).
- Go-to-definition resolves a symbol under the cursor by walking `contains` and `imports` edges to their target declaration.
  - A unique `resolved`-confidence binding navigates directly to the definition.
  - A `heuristic` or `inferred` binding presents the candidate list instead of picking one, with each candidate's confidence level shown. Over-approximation applies here as everywhere in the system: candidates are shown and marked, never silently dropped or auto-resolved.
- Find-references returns inbound `calls`, `imports`, `extends`, and `implements` edges for the selected symbol, each labeled with its confidence level.
- References are grouped by inferred module (Epic 8), not presented as a flat list, and any reference that crosses a detected cycle (US-8.3) is flagged. This is the one place the code view does something a conventional code-intelligence tool does not — it is a requirement, not an enhancement.
- Go-to-definition resolves within 500ms, matching the US-5.1 search budget — it is a bounded lookup over already-resolved edges, not a new computation. Find-references, including module grouping and cycle-crossing flags, resolves within 1 second, matching US-4.1's subgraph-expansion budget — it does strictly more work than go-to-definition, so it inherits the heavier of the two existing budgets rather than a new one.
- When no binding is found — the reference is invisible to static analysis, see Epic 10 — the view states this explicitly and links to the Epic 10 limitations surface (US-10.2), rather than showing an empty panel.

---

# Epic 5: Search the Codebase

## Epic Goal

Allow users to locate files, packages, and symbols without knowing the exact path or name, using text and structural queries.

*Downgraded from natural-language semantic search to lexical and structural search — embedding-based semantic search is explicitly out of MVP scope per GOALS.md. Downgraded rather than removed because WORKFLOW.md's Step 6 needs a search capability behind it; semantic search is Phase 2 (SCOPE.md).*

## US-5.1: Search the Codebase by Name and Structure

**As a developer, I want to search by symbol name, text, and structural filters so that I can find relevant code across the repository.**

Example queries:

- `AuthService` (exact/fuzzy symbol name)
- `retry` (substring match across identifiers and text)
- `type:function lang:python path:billing/` (structural filter combination)

### Acceptance Criteria

- Search matches against symbol names, file paths, and identifier text using lexical matching (exact, prefix, and fuzzy substring) — no embedding-based semantic matching in the MVP.
- Search supports structural filters: language, node type (file/package/class/function), and package/module scope.
- Results are ranked by lexical match quality (exact > prefix > fuzzy) combined with structural relevance (e.g., an exact symbol-name match outranks a substring match inside a docstring).
- Every result includes a code snippet, source path, and line number.
- Search returns results within 500ms for the pinned demonstration repository.
- Search results identify the analyzed commit.
- Natural-language semantic querying (e.g., "where is authentication implemented?") is Phase 2 — see SCOPE.md.

## US-5.2: Explore Search Results in Architecture Context

**As a developer, I want to open a search result inside the architecture graph so that I can understand its surrounding dependencies and call flow.**

### Acceptance Criteria

- Selecting a result focuses its graph node.
- Immediate dependencies and dependents can be displayed.
- Callers and callees can be expanded.
- The user can trace a path from the result to another node (US-4.2).
- Search context (the query and result list) remains available while navigating the graph.

---

# Epic 6: Analyze Pull-Request Blast Radius

## Epic Goal

Show how a pull request changes the architecture and which components may be affected.

## US-6.1: Analyze a Pull Request

**As a pull-request reviewer, I want CodeLens to compare the base and head revisions so that I can understand the architectural consequences of the proposed changes.**

### Acceptance Criteria

- The system receives or retrieves the pull-request base and head commits.
- Added, modified, renamed, and deleted files are identified.
- Changed classes, functions, methods, and packages are identified.
- Added and removed dependencies are detected.
- The architecture graph is updated incrementally for the changed code, within the ~2 second budget defined in Epic 9 (Real-Time Incremental Analysis).
- Analysis failures are reported without producing a misleading partial result — a failed analysis is shown as failed, not as a smaller impact set.

## US-6.2: Calculate and Visualize Blast Radius

**As a reviewer, I want to see the direct and transitive impact set of a pull request so that I can judge its risk before merging.**

*Downgraded: impacted APIs, services, and database interactions are Phase 2 — see SCOPE.md. This story covers packages, files, symbols, and tests.*

### Acceptance Criteria

The analysis identifies:

- directly modified symbols
- directly affected dependents
- transitively affected dependents
- impacted packages and inferred modules
- potentially affected tests
- new or removed dependencies

Phase 2 additions (not built in the MVP — see SCOPE.md): impacted services, impacted API endpoints, impacted database interactions.

Additional criteria:

- Direct and transitive impact are displayed separately (GLOSSARY.md).
- The architectural diff visually distinguishes added, removed, and modified nodes and edges.
- Every impacted component includes an explanation (which changed symbol reached it, and by what path).
- Every impacted component's confidence level is shown; confirmed and probable impacts are visually distinguished.
- Traversal depth is configurable, sharing the default (5) and ceiling (15) from US-4.2.
- Cycles are handled without infinite traversal — verified against a synthetic cyclic import graph, which must produce a finite, terminating impact-set computation.
- The impact set targets recall ≥ 0.7 against the Epic 0 ground-truth PR set for the pinned demonstration repository (files and tests actually touched in later commits of the same PR). This is a target, not an assumed result — see Definition of Done; it is validated by Epic 0, not asserted here.
- Precision is reported alongside recall. Per the over-approximation policy (GOALS.md, Confidence Levels above), recall is prioritized over precision: extra flagged files are acceptable, missed files are not.

## US-6.3: Report Explainable Pull-Request Risk Factors

**As a tech lead, I want a transparent breakdown of change-risk factors so that I can prioritize review and testing effort.**

*Downgraded from a composite risk score to a factor breakdown for the MVP. A composite score requires calibrated weights, and the only source of calibration data is the Epic 0 evaluation harness; until enough demonstration-repo PRs have been scored, any weights would themselves be an unexplained, unvalidated number — exactly what this story exists to avoid. A weighted composite score is Phase 2, contingent on Epic 0 producing enough scored PRs to calibrate weights empirically — see SCOPE.md.*

### Acceptance Criteria

The following factors are computed and displayed for every analyzed pull request, each as its own labeled number — not combined into a single score:

- changed symbol count
- affected package / inferred-module count
- direct dependent count
- transitive dependent count
- architectural centrality of changed symbols (degree or betweenness centrality within the dependency graph)
- dependency additions and removals
- presence or absence of a corresponding test file in the impact set
- membership of changed symbols in a detected cycle (Epic 8, US-8.3)

Additional criteria:

- Database write-path involvement is not computed in the MVP — API-to-database path reconstruction is deferred (SCOPE.md).
- Each factor links to the underlying graph evidence it was computed from.
- No composite score, categorical label (low/medium/high/critical), or AI-generated value is presented in the MVP.
- Users can inspect the underlying metrics behind every factor.

---

# Epic 7: Present Insights and Generate Reports

## Epic Goal

Convert architecture and impact-analysis data into understandable React dashboards and exportable reports.

## US-7.1: View Architecture Charts and Insights

**As a developer, I want architecture metrics and charts so that I can understand the repository without manually inspecting the complete graph.**

*Cut from eight charts to four — the ones backed by data the MVP genuinely computes. Language distribution, dependency-type detail beyond imports/calls/inheritance, full architecture composition (services/API endpoints/database entities), most-critical-components ranking, and architectural diff summary are moved to Phase 2 in SCOPE.md; several of those could be partially computed but are cut here to keep the chart set small and fully honest.*

### React Dashboard Charts

The interface includes exactly these charts in the MVP.

#### 1. Language Distribution

Chart type: donut chart or horizontal bar chart.

Shows: files by language, lines of code by language, parsed versus unsupported files.

#### 2. Package / Module Dependency Count

Chart type: horizontal bar chart.

Shows: incoming dependencies per package or inferred module, outgoing dependencies per package or inferred module, highest-coupled packages/modules.

#### 3. Pull-Request Impact Summary

Chart type: stacked bar chart or summary cards.

Shows: changed components, directly affected components, transitively affected components, affected tests.

(Affected APIs and affected database paths are Phase 2 — see SCOPE.md.)

#### 4. Pull-Request Risk-Factor Breakdown

Chart type: weighted horizontal bars (one bar per factor from US-6.3 — not a radar chart implying a composite score, since none exists in the MVP).

Shows: the eight factors listed in US-6.3.

### Acceptance Criteria

- Charts use analysis data from the selected repository and commit.
- Selecting a chart element filters or focuses the architecture graph.
- Charts display loading, empty, and error states.
- Metrics include clear descriptions.
- Users can view the source components behind each insight.
- Charts are responsive on common desktop screen sizes.
- The dashboard never reports a conclusion the underlying data doesn't support (e.g., no API/database charts, since that data isn't computed).

## US-7.2: Generate an Impact-Analysis Report

**As a reviewer, I want an impact-analysis report so that I can share and preserve the architectural review of a pull request.**

*Downgraded: the "Impacted APIs and database paths" section is cut (Phase 2 — SCOPE.md). The risk section reports the US-6.3 factor breakdown, not a composite score. All other sections survive because they're backed by MVP-computed data once service/API/database language is removed.*

### Report Sections

The report contains:

1. Repository and pull-request information
2. Base and head commits
3. Executive impact summary
4. Changed files and symbols
5. Architecture changes
6. Directly affected components
7. Transitively affected components
8. Impacted packages and inferred modules
9. Potentially affected tests
10. Dependency additions and removals
11. Risk-factor breakdown (US-6.3 — not a composite score)
12. Architecture charts (the four from US-7.1)
13. Blast-radius graph
14. Recommended review order (directly impacted symbols first, then transitive) and the list of tests in the impact set — derived entirely from already-computed data, not a new recommendation algorithm
15. Analysis assumptions and confidence levels (Epic 10)

("Impacted APIs and database paths" is cut from the MVP report — Phase 2, see SCOPE.md.)

### Acceptance Criteria

- The report can be generated from a completed pull-request analysis.
- Repository, branch, commit, and pull-request information are included.
- The report includes graph visualizations and the four React dashboard charts from US-7.1.
- Confirmed and inferred impacts are labeled separately, using the confidence levels defined above.
- Every significant conclusion links to underlying graph evidence.
- The report can be exported as PDF or HTML.
- The report records the analysis timestamp and graph version (commit SHA).
- Sensitive source-code snippets are limited and configurable.

---

# Epic 8: Architecture Divergence and Anomaly Detection

## Epic Goal

Surface where the codebase's real structure disagrees with its folder structure, and detect the one class of anomaly the MVP claims to find: cycles. This is the project's actual differentiator (GOALS.md Goal 2 and Goal 5) and was missing entirely from the original epic set.

## US-8.1: Detect Communities Over the Dependency Graph

**As a developer, I want CodeLens to run community detection over the dependency graph so that it can identify modules that exist in practice but not in the folder tree.**

### Acceptance Criteria

- Community detection (Louvain or Leiden) runs over the file- or package-level dependency graph after every full or incremental graph build.
- Each detected community is assigned a stable ID that persists across incremental updates where its membership is unchanged, so the graph doesn't relabel modules on every save.
- Detected communities are stored as inferred modules (GLOSSARY.md), distinct from declared directories and packages.
- Community detection completes within the ~2 second incremental-update budget (Epic 9) for a single-file change, and within the 60 second graph-build budget (US-3.1) for a full rebuild on the pinned demonstration repository.
- Singleton communities (a single file with no strong internal cohesion) are not reported as modules.

## US-8.2: Surface Divergence Between Inferred Modules and the Folder Tree

**As a developer, I want to see where inferred modules disagree with the declared directory structure so that I can find hidden coupling and misplaced code.**

### Acceptance Criteria

- For each inferred module, CodeLens computes its overlap with declared directories/packages using a set-overlap (purity) metric.
- Directories whose files split across more than one inferred module, such that no single module accounts for at least 80% of the directory's files, are flagged as "fused" — multiple concerns living in one declared unit.
- Inferred modules whose files span more than one declared top-level directory, by the same 80% threshold, are flagged as "split" — one real module spread across declared units.
- Flagged divergences list the specific files responsible, not just a directory name.
- The divergence view is reachable from the architecture graph (Epic 4) and is the artifact WORKFLOW.md surfaces between reconstruction and exploration.

## US-8.3: Detect Cycles via Tarjan's SCC

**As a developer, I want cyclic dependencies detected and shown so that I can find genuinely risky coupling.**

*Cycle detection is the only anomaly detector in the MVP. Dead-code detection and instability detection are Phase 2 — both are unsound in dynamic languages and would generate false positives that undermine trust in the tool during the demo (GOALS.md Goal 5).*

### Acceptance Criteria

- Tarjan's strongly connected components (SCC) algorithm runs over the package/module-level dependency graph.
- Every strongly connected component with more than one member is reported as a cycle, listing its member packages/modules and the edges forming the cycle.
- Cycle detection re-runs on every incremental update and completes within the ~2 second incremental budget (Epic 9) for a single-file change.
- Cycles are visually distinguished in the architecture graph (Epic 4) and included in the risk-factor breakdown (US-6.3).
- Dead-code detection and instability detection are not built in the MVP — see SCOPE.md and Epic 10.

---

# Epic 9: Real-Time Incremental Analysis

## Epic Goal

Prove the "real-time" claim in the project's name with a measured, bounded update loop rather than a full reparse on every change. "Real-time" is in the project title and no epic owned it before this one.

## US-9.1: Re-Parse Only the Changed File on Save

**As a developer, I want CodeLens to re-parse only the file I saved so that feedback is fast enough to use while coding.**

### Acceptance Criteria

- On file save, CodeLens re-parses only the changed file, not the repository.
- Re-parsing produces the same IR (nodes, edges, symbols) that a full parse of that file would produce.
- A parse error in the saved file does not affect the previously-parsed state of any other file.
- Re-parsing a single file completes within 500ms for files up to the largest file size in the pinned demonstration repository, leaving headroom inside the ~2 second end-to-end budget.

## US-9.2: Patch the Subgraph Instead of Rebuilding

**As a developer, I want only the affected subgraph updated so that incremental changes don't cost as much as a full rebuild.**

### Acceptance Criteria

- A single-file re-parse adds, removes, or modifies only the nodes and edges touching that file; the rest of the graph is untouched (verified: node/edge object identity is preserved for unaffected parts of the graph).
- If the changed file's symbols affect community membership (US-8.1), the affected inferred module(s) are recomputed; modules with no member in the changed file are not recomputed.
- If the change creates, breaks, or alters a strongly connected component, cycle detection (US-8.3) re-runs for the affected subgraph and the UI's cycle indicators update to match.
- End-to-end — file save → subgraph patch → UI reflects the change — completes in ~2 seconds on the pinned demonstration repository. This is the numeric budget from GOALS.md Goal 4.

## US-9.3: Reflect the Update in the UI Without a Full Reload

**As a developer, I want the graph UI to update live so that I can see the effect of a change without manually refreshing.**

### Acceptance Criteria

- The architecture graph view (Epic 4) updates the affected nodes/edges in place, without discarding the user's current pan/zoom/expansion state.
- Nodes/edges added, removed, or modified by the update are visually distinguished for 3 seconds after the update lands.
- If the update changes an inferred module the user currently has expanded, the UI indicates the module's membership changed rather than silently re-clustering under the user.
- The update reaches the UI within the same ~2 second budget as US-9.2 — this is the end of that budget, not an additional one.

---

# Epic 10: Limitations and Assumptions

## Epic Goal

Document, and surface in the product itself, where static reconstruction is blind — so the tool's confidence claims are honest rather than implied. This is a deliverable, not an afterthought.

## US-10.1: Document Static-Analysis Blind Spots

**As a developer, I want the limitations of static reconstruction documented so that users and evaluators know what the graph cannot see.**

### Acceptance Criteria

- The documentation set states, in one place, that dynamic dispatch, reflection, dependency injection, string-based routing, monkey-patching, and dynamic imports are not resolved by static analysis and are therefore not represented as confirmed (`resolved`) edges.
- Each blind spot lists the concrete language construct it affects for at least one of the two MVP languages — e.g., Python's `getattr`-based dispatch and dependency-injection containers, TypeScript's dynamic `import()` and string-keyed routing tables.
- The document states the consequence: affected edges are either missing entirely or included only as low-confidence `inferred` edges (Confidence Levels, above).

## US-10.2: Surface Limitations in the UI

**As a developer, I want the tool to tell me when it might be wrong so that I don't mistake an incomplete graph for a complete one.**

### Acceptance Criteria

- Any view showing an impact set, blast radius, or dependency path displays a visible indicator when the underlying analysis relied on `inferred` (not `resolved`) edges.
- The indicator states, in plain language, that the result may under-report impact when dynamic-dispatch-style constructs are present, and links to the limitations documentation (US-10.1).
- The UI never presents an impact set or graph view as complete or exhaustive without this caveat being at least available on demand — it does not have to be forced into view every time, but it is never fully absent.
- This requirement applies to every story in Epic 4 and Epic 6 that surfaces impact or dependency data — it is not a separate, optional feature.
