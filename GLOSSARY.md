# Glossary

The terms below were used inconsistently across earlier drafts of this documentation set. Each now has exactly one meaning, applied consistently in `README.md`, `GOALS.md`, `EPICS.md`, `MODULES.md`, `WORKFLOW.md`, and `SCOPE.md`.

### Module

An **inferred** cluster of files or symbols produced by community detection (Louvain/Leiden) over the dependency graph (`EPICS.md` Epic 8). A module is a graph-analysis output, not something declared in source. Contrast with **package** and **directory** below. "Module-level view" (`GOALS.md` Goal 6) means the graph UI's default view groups nodes by inferred module, not by folder.

### Directory

A folder in the repository's file tree, exactly as declared by the repository's structure. No inference involved.

### Package

A directory or set of files that the target language's own tooling recognizes as an importable or organizational unit — a Python package (`__init__.py` or an implicit namespace package) or a TypeScript/JavaScript module resolved via `package.json`/`tsconfig` paths. Declared, not inferred. Contrast with **module**, above.

### Application / Service

Human-assigned, declared groupings some repositories use for top-level directories (e.g., `apps/checkout`, `services/billing`). The MVP graph does not detect or type these as distinct node types — they're treated as packages/directories like any other, and service-to-service communication inference is Phase 2 (`SCOPE.md`). These terms appear elsewhere in this documentation set only in that Phase 2 context, or when quoting the original problem statement in `README.md`.

### Component

A generic term for any addressable unit in the architecture graph — file, package, module, class, function, or symbol — used when the specific node type doesn't matter (e.g., "affected components" in an impact set). Distinct from the phrase "component repo" in earlier `README.md` drafts, which referred to a sibling repository in the CodeLens GitHub organization (Backend, Frontend, Intelligence, `platform-contracts`) — a repo-organization concept, unrelated to graph terminology. This documentation set no longer uses "component repo"; see `MODULES.md` for the module breakdown instead.

### Symbol

A named, addressable code entity below file level: a class, interface, function, or method. The smallest node at the function/symbol level of the IR (`GOALS.md` Goal 1).

### Definition vs. Reference

**Definition:** the declaration site of a symbol — where a class, interface, function, or method is defined, as opposed to used. Go-to-definition (`EPICS.md` US-4.3) resolves to this. **Reference:** an inbound `calls`, `imports`, `extends`, or `implements` edge pointing at a symbol — a usage site, not the declaration. Find-references (US-4.3) returns these, grouped by inferred module. Neither term is a synonym for a generic graph edge: every reference is backed by an edge, but "reference" specifically means an edge surfaced in the code-navigation context, carrying the same confidence level (`resolved`/`heuristic`/`inferred`) as the edge itself.

### Blast Radius vs. Impact Set

**Impact set** is the literal output of impact analysis: the concrete list of files, symbols, and tests flagged as affected by a change (direct ∪ transitive), each carrying a confidence level. **Blast radius** is the same data in its user-facing framing — UI copy, chart titles, casual discussion ("the blast radius view"). It is not a separate computation. Use "impact set" in technical contexts (interfaces, acceptance criteria, the evaluation harness); use "blast radius" only in user-facing copy referring to that same impact set.

### Direct vs. Transitive Impact

**Direct impact:** a file or symbol with a one-hop edge from a changed symbol in the reverse-reachability closure. **Transitive impact:** reachable from a changed symbol only through one or more intermediate nodes — two or more hops.

### Confirmed vs. Inferred Edge

See "Confidence Levels" in `EPICS.md` for the full three-level scale (`resolved` / `heuristic` / `inferred`). Shorthand: **confirmed** = `resolved` confidence — statically bound to a unique declaration. **Inferred** = `heuristic` or `inferred` confidence — bound by name-matching or a weak signal, not proof. CodeLens over-approximates: inferred edges are included and marked, never silently dropped, because a missed impacted file is worse than an extra flagged one (`GOALS.md` trade-off table).
