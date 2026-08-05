The final project demonstration should follow one connected developer story.

Every step below is tagged with its `SCOPE.md` tier. All eleven steps are MVP — this document describes the MVP demo path end to end, not a feature list; it doesn't cover Phase 2 or Deferred capability. Where a step was originally written to a broader scope than the MVP supports, it's downgraded here rather than cut, and the downgrade is noted.

## Step 1: Connect — MVP (downgraded)

The user provides a local clone path or a Git remote URL with a token-authenticated clone, and selects the branch or commit to analyze. (Full GitHub OAuth with a repository picker is Phase 2 — see `SCOPE.md`; it's real integration effort with little value for demonstrating the core reconstruction-and-impact claim.) If this repository and commit were analyzed before, CodeLens loads the cached graph directly instead of re-ingesting (US-1.4) — this is why a second run of the demo is fast.

## Step 2: Ingest — MVP (downgraded)

CodeLens detects source languages, top-level packages, and relevant source directories and files, excluding generated and vendor paths by default. (The original step assumed detection of "applications" and "services" as distinct node types; the MVP graph doesn't model those separately — see `GLOSSARY.md` — so this step is described in terms of packages, directories, and files.) A first meaningful graph — package and directory nodes plus import edges — appears within seconds and populates incrementally as parsing continues (US-1.3), so the graph is there to look at while ingestion is still running rather than only after it finishes; the view stays interactable throughout, with a clear indicator of what's still pending.

## Step 3: Reconstruct — MVP

The platform parses the repository and builds a graph of packages, files, classes, functions, imports, calls, and inheritance relationships, each edge carrying a confidence level (`EPICS.md`, Confidence Levels).

## Step 4: Surface Divergence and Cycles — MVP

Before the user explores anything, CodeLens runs community detection over the dependency graph and compares the inferred modules against the declared directory structure (Epic 8), and runs cycle detection over the module graph. The divergence and any cycles are the first thing the user sees after reconstruction — this is the moment that distinguishes the demo from a rendered folder tree, and it didn't have a workflow step before this revision.

## Step 5: Explore — MVP

The user opens the React architecture explorer, which defaults to the module-level view produced in Step 4, expands a package, traces a dependency path, and opens a source symbol — jumping to its definition and viewing its references grouped by inferred module, with any reference that crosses a detected cycle flagged (US-4.3).

## Step 6: Search — MVP (downgraded)

The user searches by symbol name or text — for example `AuthService` or `retry` — with structural filters (language, node type, package). CodeLens returns matching files, packages, and functions and highlights them in the graph. (Natural-language semantic search — e.g. "Where is user authentication implemented?" — is deferred per `GOALS.md`; this step is lexical and structural symbol search over the graph instead. Semantic search is Phase 2 — see `SCOPE.md` and `EPICS.md` Epic 5.)

## Step 7: Analyze a Pull Request — MVP

The user selects an open pull request. CodeLens compares its base and head commits and identifies changed symbols and the packages, files, and functions connected to them.

## Step 8: Show Blast Radius — MVP (downgraded)

The UI highlights directly and transitively affected packages, files, symbols, and tests. (The original step assumed database-path and API tracing; those are Phase 2 — see `SCOPE.md` — so this step covers packages, files, symbols, and tests only.)

## Step 9: Present Insights — MVP (downgraded)

The React dashboard displays the four MVP-backed charts from Epic 7: language distribution, package/module dependency counts, pull-request impact summary, and pull-request risk-factor breakdown. (The original eight-chart list included charts backed by data the MVP doesn't compute — service counts, API and database paths; those are Phase 2, see `SCOPE.md`.)

## Step 10: Generate the Report — MVP (downgraded)

The user generates an impact-analysis report containing the pull-request changes, architecture diff, risk-factor breakdown (not a composite score — see `EPICS.md` US-6.3), blast-radius graph, impacted packages/files/symbols, and test priorities.

## Step 11: Close on Measured Accuracy — MVP

The demo ends by showing the Epic 0 evaluation harness's baseline: precision, recall, and F1 for the impact set against the pinned demonstration repository's 30–50 historical pull requests (`EPICS.md` Epic 0, `SCOPE.md`). The demo closes on these measured numbers, not on a screenshot of the graph.
