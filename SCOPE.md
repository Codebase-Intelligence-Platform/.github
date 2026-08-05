# Scope

This is the single source of truth for what CodeLens is and is not building. `GOALS.md` defines the MVP; `EPICS.md` was originally written to the full problem statement in `README.md` as if all of it were MVP scope. Where the two disagreed, `GOALS.md` won, and every story below was either kept as-is, downgraded, or moved to Phase 2 / Deferred — never silently dropped. See `EPICS.md` for the downgrade notes on each story and `GLOSSARY.md` for the terms used in the rationale column.

Tiers:
- **MVP** — built and demoed now.
- **Phase 2** — a natural next increment with a reasonably clear design path from the MVP; not built now.
- **Deferred** — explicitly out of scope per GOALS.md, with no design or acceptance criteria in this documentation set yet.

## Languages

Python and TypeScript (GOALS.md Goal 1 — "pick exactly two languages with different module semantics"). Decided, not open.

## Demonstration Repository

`TODO(decision):` no repository has been pinned yet. Selection criteria (EPICS.md US-0.1):

- **Size:** medium — roughly 200–2,000 source files. Large enough that folder-vs-module divergence (Epic 8) is likely to exist and be interesting; small enough to stay inside the ingestion and graph-build time budgets (EPICS.md US-1.2, US-3.1) on a single developer machine.
- **Language mix:** majority Python and TypeScript, with both meaningfully represented — not a 95/5 split, or the two-language IR claim (GOALS.md Goal 1) isn't actually exercised.
- **PR history depth:** at least 30 merged, multi-commit pull requests, ideally 50–150 for headroom above the Epic 0 minimum (EPICS.md US-0.2) and to allow excluding low-signal PRs (single-file typo fixes, dependency bumps) without dropping below 30.
- **License:** permissive enough to clone and analyze freely for a student project.
- **Test suite:** a real, non-trivial test suite, so "affected tests" in the ground-truth dataset (US-0.2) and in the impact set (US-6.2) means something.

Once selected, the repository name, URL, and analyzed commit range are recorded here and in `README.md`, and this `TODO(decision)` is removed.

## Why These Deferrals

GOALS.md's argument is that an MVP proving architecture reconstruction and diff-aware impact analysis is worth more than a wider MVP that proves nothing reliably: "if the graph is wrong, none of it matters." Everything below scoped out of MVP — polyrepo, semantic search, service inference, ownership heatmaps, lineage, refactoring assistance, CI integration, languages beyond two — is additive value on top of that graph, not a prerequisite for it. Each is deferred with a specific reason (data the MVP doesn't compute, effort disproportionate to what it proves, or unsoundness risk that would undermine trust in the demo), not because it's unimportant. Naming these as deliberate deferrals, with reasoning, is meant to read as engineering judgment; leaving them unmentioned would read as failure to deliver.

## User Stories

| ID | Story | Tier | Rationale |
|---|---|---|---|
| US-0.1 | Select and pin the demonstration repository | MVP | Nothing else in the evaluation harness or demo is meaningful without a fixed target repo. |
| US-0.2 | Build the ground-truth PR dataset | MVP | Required input for measuring impact-set accuracy — GOALS.md's "thing most teams skip." |
| US-0.3 | Run the impact-prediction evaluation harness | MVP | Turns impact analysis from an assertion into a measured claim. |
| US-0.4 | Report the baseline accuracy | MVP | The demo closes on this number (WORKFLOW.md's final step). |
| US-1.1 | Connect a repository | MVP (downgraded) | Local clone / token auth only; full GitHub OAuth is real integration effort with little academic value now. |
| US-1.2 | Configure and ingest the repository | MVP (downgraded) | Detects packages/directories, not applications/services — those node types aren't part of the MVP graph. |
| US-1.3 | Progressive graph rendering during ingestion | MVP | Renders package/import structure within the existing ingestion and graph-build budgets instead of after them — no new computation, only sequencing of what US-1.2/US-3.1 already produce. |
| US-1.4 | Cache and re-ingest | MVP | Makes incremental analysis work across commits, not just file saves — required for the demo's second run (and any real PR review) to be fast; reuses Epic 9's machinery rather than adding a parallel mechanism. |
| US-2.1 | Detect and parse supported languages | MVP (downgraded) | Exactly two languages (Python, TypeScript) per GOALS.md, not an open language list. |
| US-2.2 | Extract symbols and relationships | MVP (downgraded) | API endpoint extraction dropped — tied to service/API detection, which is Phase 2. |
| US-3.1 | Build the architecture graph | MVP (downgraded) | Node/edge types trimmed to file/package/class/interface/function plus imports/calls/inherits/contains; service, API endpoint, database entity, and communicates-with/reads-from/writes-to are Phase 2. |
| US-3.2 | Identify architectural boundaries and flows | MVP (downgraded) | Structural package/directory grouping only; API-to-handler paths, service-to-service communication, and API-to-database paths are Phase 2. |
| US-4.1 | Explore the interactive architecture graph | MVP (downgraded) | Default view is packages plus inferred modules; service filter removed. |
| US-4.2 | Trace dependencies and call paths | MVP | Core to the "graph disagrees with the folder tree" claim; no dependency on deferred capability. |
| US-4.3 | Code view with definitions and references | MVP | A view over `resolved`-confidence edges Graph Core already produces, not a new analysis subsystem — no type inference, no LSP, no SCIP indexer. |
| US-5.1 | Search the codebase | MVP (downgraded) | Lexical and structural search only; natural-language semantic search is Phase 2. |
| US-5.2 | Explore search results in architecture context | MVP | Doesn't depend on semantic search itself. |
| US-6.1 | Analyze a pull request | MVP | The differentiator (GOALS.md Goal 3). |
| US-6.2 | Calculate and visualize blast radius | MVP (downgraded) | Impacted APIs/database interactions/services dropped; packages, files, symbols, tests, and dependency changes remain. |
| US-6.3 | Report explainable pull-request risk factors | MVP (downgraded to factor breakdown) | No composite score in the MVP — weights would be unvalidated without Epic 0 data; database write-path factor dropped, tied to deferred DB-path reconstruction. |
| US-7.1 | View architecture charts and insights | MVP (downgraded to 4 charts) | Language distribution, package/module dependency count, PR impact summary, PR risk-factor breakdown are the only charts fully backed by MVP-computed data. |
| US-7.2 | Generate an impact-analysis report | MVP (downgraded) | "Impacted APIs and database paths" section dropped; risk section reports factors, not a composite score. |
| US-8.1 | Detect communities over the dependency graph | MVP | GOALS.md Goal 2 — the architecture-reconstruction differentiator. |
| US-8.2 | Surface divergence between inferred modules and the folder tree | MVP | Same — "the divergence is your product" (GOALS.md). |
| US-8.3 | Detect cycles via Tarjan's SCC | MVP | GOALS.md Goal 5 — the one anomaly detector in the MVP. |
| US-9.1 | Re-parse only the changed file on save | MVP | GOALS.md Goal 4 — real-time responsiveness is an explicit objective. |
| US-9.2 | Patch the subgraph instead of rebuilding | MVP | Same. |
| US-9.3 | Reflect the update in the UI without a full reload | MVP | Same. |
| US-10.1 | Document static-analysis blind spots | MVP | GOALS.md: "documenting where your reconstruction is blind is more impressive than pretending it isn't." |
| US-10.2 | Surface limitations in the UI | MVP | A documentation-only limitations story wouldn't meet GOALS.md's framing; the UI has to carry it. |

## Deferred and Phase 2 Items Without a Dedicated Story

These were named in `README.md`'s problem statement and/or `EPICS.md`'s original acceptance criteria but never had their own user story — they're recorded here so nothing that was mentioned gets silently dropped.

| Item | Tier | Rationale |
|---|---|---|
| Full GitHub OAuth + repository picker | Phase 2 | Downgraded from US-1.1; real effort, little academic value for the MVP's demo. |
| Application/service detection as distinct node types | Phase 2 | Downgraded from US-1.2/US-3.1/US-3.2; a prerequisite for service-to-service inference to be meaningful. |
| Service-to-service communication inference | Phase 2 | Explicitly out of MVP scope per GOALS.md. |
| API endpoint extraction & API-to-handler path reconstruction | Phase 2 | Explicitly out of MVP scope per GOALS.md; downgraded from US-2.2/US-3.2. |
| API-to-database path reconstruction & database entity modeling | Phase 2 | Explicitly out of MVP scope per GOALS.md; downgraded from US-3.1/US-3.2/US-6.2/US-6.3/US-7.2. |
| Natural-language / embedding-based semantic search | Phase 2 | Explicitly out of MVP scope per GOALS.md; downgraded from US-5.1. |
| Polyrepo / cross-repository analysis | Phase 2 | Explicitly out of MVP scope per GOALS.md. |
| Composite weighted risk score | Phase 2 | Downgraded from US-6.3; needs Epic 0 data to calibrate weights honestly rather than assert them. |
| Dead-code detection | Phase 2 | GOALS.md: unsound in dynamic languages, would generate false positives that undermine trust in the demo. |
| Instability detection | Phase 2 | Same reasoning as dead-code detection. |
| Architecture Composition chart (full: apps/services/API endpoints/database entities) | Phase 2 | Downgraded from US-7.1; only a subset (packages, modules, files, symbols, external dependencies) is MVP-computable. |
| Dependency-Type Distribution chart | Phase 2 | Downgraded from US-7.1; includes API/database relationship types the MVP doesn't compute. |
| Most Critical Components chart | Phase 2 | Downgraded from US-7.1; cut to keep the MVP chart count at four. |
| Architectural Diff Summary chart | Phase 2 | Downgraded from US-7.1; cut to keep the MVP chart count at four. |
| Type-inference-backed precise code intelligence | Phase 2 | US-4.3's go-to-definition/find-references are a view over existing `resolved`/`heuristic`/`inferred` edges; a type checker backing precise, single-answer resolution is real effort `GOALS.md`'s trade-off table explicitly rules out for the MVP. |
| Cross-repository reference search | Phase 2 | Consistent with the existing polyrepo deferral — US-4.3 resolves references within the one analyzed repository only. |
| Distributed or pre-built reference/code-navigation indexing | Phase 2 | In-process by design per the `GOALS.md` storage trade-off (SQLite + NetworkX, no dedicated index service); US-4.3 reuses Graph Core's existing edges rather than building a separate index. |
| Ownership and contribution heatmaps | Deferred | Explicitly out of MVP scope per GOALS.md; no design or acceptance criteria exist in this documentation set. |
| Lineage tracking | Deferred | Same. |
| Refactoring assistance | Deferred | Same. |
| CI-driven architecture integrity validation | Deferred | Same. |
| Languages beyond Python and TypeScript | Deferred | GOALS.md: "two languages is enough to force a language-agnostic IR; three is just grinding" — a scope discipline, not a near-term roadmap item. |
