## The framing question first

Your MVP shouldn't try to be a smaller version of the full platform. It should be the **smallest thing that proves the riskiest claim** in the problem statement — that you can reconstruct architecture from code and forecast the blast radius of a change more usefully than a static diagram.

Everything else in the brief (ownership heatmaps, semantic search, refactoring assistance, CI gates) is additive value on top of a working code graph. If the graph is wrong, none of it matters. So build the graph and the impact loop first.

## Core MVP goals

**1. Multi-language ingestion into a unified code graph.**
Pick exactly two languages with different module semantics — Python and TypeScript is a good pair. Use tree-sitter for parsing so you get a uniform CST interface. Extract nodes at three levels (file, module/class, function/symbol) and three edge types (imports, calls, inherits). Two languages is enough to force you to design a language-agnostic IR; three is just grinding.

**2. Architecture reconstruction that disagrees with the folder tree.**
Run community detection (Louvain or Leiden) over the dependency graph and compare the inferred clusters against the declared directory structure. The *divergence* is your product: "these two directories are one module in practice," "this package has three unrelated concerns fused together." A diagram that just redraws the folder tree proves nothing.

**3. Diff-aware impact analysis.**
This is the differentiator, so make it the demo centerpiece. Given a commit, PR, or uncommitted working-tree diff: map changed line ranges → changed symbols → reverse-reachability closure over the call/import graph → ranked list of affected files, modules, and test files. Rank by graph distance and edge confidence, not just set membership, or the output degenerates into "half the repo is affected."

**4. Incremental updates.**
Real-time responsiveness is an explicit objective, so you need to show *something*. Target: file save → re-parse only that file → patch the affected subgraph → UI reflects it in under ~2 seconds on a mid-sized repo. Full reparse on every change is the thing you're explicitly claiming to beat.

**5. Exactly one anomaly detector: cycles.**
Tarjan's SCC over the module graph. It's cheap, visually obvious, and genuinely actionable. Resist adding dead-code detection to the MVP — it's badly unsound in dynamic languages and will generate false positives that undermine trust in the whole tool during your demo.

**6. A graph UI with collapse/expand and a diff overlay.**
Module-level view by default, click to expand into files and symbols. Overlay the impact set from goal 3 in a distinct color. Do *not* render 50,000 nodes and call it a visualization — hierarchical collapse is the feature.

## Explicitly out of scope (and say so in your report)

Polyrepo and cross-service communication inference, ownership/contribution heatmaps, embedding-based semantic search, lineage tracking, refactoring suggestions, CI integration, and languages three through eight. Naming these as deliberate deferrals with reasoning reads as engineering judgment; leaving them unmentioned reads as failure to deliver.

## The thing most teams skip

**Build an evaluation harness in week two, not week nine.** Take 30–50 historical merged PRs from your target repo. For each, feed your tool only the first commit's diff and ask: does the predicted impact set contain the files that were actually touched in later commits of that same PR? Does it contain the tests that were modified?

That gives you precision and recall numbers for impact prediction — which is the single most convincing thing you can put in a report and the only way to honestly answer "fidelity vs. performance." Without it you're demoing a pretty graph and asserting it's correct.

## Trade-offs you'll need to defend

| Decision | MVP choice | Why it's defensible |
|---|---|---|
| Call resolution | Syntactic + heuristic import resolution, no type inference | Full type inference costs 10× the effort; measure the precision gap instead of hiding it |
| Unsoundness direction | Over-approximate (include uncertain edges, mark confidence) | Missing an impacted file is worse than flagging an extra one |
| Storage | SQLite + NetworkX in-process | Neo4j is real infrastructure overhead; migrate only when you hit a measured wall |
| Graph scope | Symbol-level within files, module-level across | Full symbol-level cross-module graphs blow up node counts without improving impact ranking much |

The honest limitation to state upfront: dynamic dispatch, reflection, dependency injection, and string-based routing are invisible to static analysis. Documenting *where* your reconstruction is blind is more impressive than pretending it isn't.

## Demo success criterion

One sentence you should be able to say in the viva: *"I open a PR that changes three lines in a utility function, and the tool tells me within two seconds that it touches four modules the author didn't know about, including one across a cycle they didn't know existed."*

If your MVP can do that reliably on a real repo, everything else is a feature list you can prioritize by whatever time remains.