# CodeLens Organization — Overview

This repository contains organization-level documentation for the CodeLens platform: the problem statement, the MVP scope, the user stories, the module breakdown, the end-to-end demo workflow, and the terminology used consistently across all of it.

## Contents

- [GOALS.md](./GOALS.md) — the MVP definition: what to build first and why, including the trade-offs the team must be able to defend.
- [SCOPE.md](./SCOPE.md) — the single source of truth for what is MVP, Phase 2, or deferred, story by story.
- [EPICS.md](./EPICS.md) — the full set of user stories and testable acceptance criteria.
- [MODULES.md](./MODULES.md) — responsibility, non-responsibility, interface, and technology for each system module.
- [WORKFLOW.md](./WORKFLOW.md) — the single connected demo story, step by step.
- [GLOSSARY.md](./GLOSSARY.md) — one meaning per term for words used inconsistently elsewhere (module, component, blast radius vs. impact set, confirmed vs. inferred edge, and more).

## MVP Statement

CodeLens's MVP ingests a single repository written in two languages — Python and TypeScript — into a unified code graph, reconstructs its architecture by running community detection against the folder structure and surfacing where they disagree, detects cycles as its one anomaly class, and, as the demo's centerpiece, turns a commit or pull-request diff into a ranked, confidence-scored impact set within about two seconds. Everything else described in the problem statement below — polyrepo, semantic search, ownership heatmaps, lineage tracking, refactoring assistance, CI integration — is deliberately out of MVP scope; see [SCOPE.md](./SCOPE.md) for what that means and why.

## Positioning

CodeLens does not compete with Sourcegraph or CodeSee on the axes those tools are built for. Sourcegraph's go-to-definition and find-references are backed by real compilers and language servers, across dozens of languages, indexed at organizational scale across many repositories. CodeLens uses tree-sitter with heuristic import resolution and no type inference ([GOALS.md](./GOALS.md)'s trade-off table), covers two languages, and analyzes one repository at a time. On reference precision, language coverage, and scale, CodeLens is behind, and there's no MVP-scope path that closes that gap — see [SCOPE.md](./SCOPE.md)'s Phase 2 entries for type-inference-backed code intelligence, cross-repository search, and distributed indexing.

What CodeLens does instead is answer a different question. Conventional code-intelligence tools answer "where is this symbol used." CodeLens answers "what are this codebase's real module boundaries, where do they disagree with the folder structure, and what does this diff actually reach." Architecture reconstruction and divergence (Epic 8) and measured, ranked change impact (Epic 6) are the claim — not a better version of what those tools already do well.

That claim is falsifiable, which is the point: Epic 0 runs the impact predictor against 30–50 real merged pull requests from the pinned demonstration repository and reports precision, recall, and F1. That number, not a feature list, is the actual positioning argument — a graph that looks right and an impact set that's measured to be right are different claims, and only Epic 0 backs the second one.

None of this requires new technology. The [Technology Stack](#technology-stack) table further down is unchanged by anything in this document — US-4.3's code view is a UI over edges Graph Core already resolves, not a new subsystem.

| Axis | Sourcegraph / CodeSee | CodeLens MVP |
|---|---|---|
| Reference precision | Compiler/type-checker-backed | Heuristic, confidence-labeled (`resolved`/`heuristic`/`inferred`) |
| Language coverage | Dozens of languages | Two (Python, TypeScript) |
| Scale | Many repositories, organization-wide | One repository |
| Module boundary inference | Not a focus | Community detection vs. folder structure (Epic 8) |
| Change-impact ranking | Not a focus | Diff-aware, confidence-ranked (Epic 6) |
| Accuracy claim | Asserted by tool maturity | Measured: precision/recall/F1 against real PRs (Epic 0) |

## Languages and Demonstration Repository

- **Languages:** Python and TypeScript ([GOALS.md](./GOALS.md)).
- **Demonstration repository:** `TODO(decision)` — not yet pinned. See [SCOPE.md](./SCOPE.md) for the selection criteria (size, language mix, PR history depth) and current status.

## Technology Stack

| Layer | Technology |
|---|---|
| Parsing | tree-sitter (Python and TypeScript grammars) |
| Graph storage & algorithms | SQLite + NetworkX, in-process (no graph database in the MVP) |
| API | FastAPI |
| Web client | React |
| Evaluation harness | Python scripts producing machine-readable (JSON/CSV) results |

See [MODULES.md](./MODULES.md) for how these fit together, and [GOALS.md](./GOALS.md) for why each was chosen over the alternative.

## Problem Statement

### Real-Time Codebase Intelligence, Architecture Reconstruction, and Impact Analysis Platform

This project focuses on developing a real-time codebase intelligence and architecture reasoning
platform that supports developers in understanding, navigating, and analyzing large-scale or
polyglot codebases. Instead of producing static dependency diagrams or manually maintained
architecture charts, this system should dynamically reconstruct architecture from the underlying code—visualizing module boundaries, dependency relationships, call flows, communication
paths between services, and change-impact forecasts. The platform must help engineers gain
deep insight into complex codebases, enabling tasks such as onboarding new developers, identifying risky components and hotspots, validating architectural adherence, and reasoning about the consequences of proposed modifications. Key objectives include scalability across large monorepos and distributed polyrepos, multi-language code parsing, and real-time responsiveness to continuous changes in the repository.
This project goes beyond the capabilities of existing tools like Sourcegraph or CodeSee by offering advanced semantic analysis, interactive exploration graphs, anomaly detection (including
cyclic dependencies, unstable modules, or dead code clusters), ownership and contribution heat
maps based on commit history, and diff-aware architectural impact visualization. Features such
as semantic search, lineage tracking, refactoring assistance, and CI-driven architecture integrity
validation can contribute to the project’s engineering depth. Students should justify visualization approaches, underlying data processing pipelines, clustering and graph algorithms used,
and trade-offs in architectural reconstruction fidelity versus performance.

## Demo Success Criterion

From [GOALS.md](./GOALS.md), the project's acceptance bar:

> "I open a PR that changes three lines in a utility function, and the tool tells me within two seconds that it touches four modules the author didn't know about, including one across a cycle they didn't know existed."

If the MVP can do this reliably on the pinned demonstration repository, everything else in the problem statement above is a feature list to prioritize by whatever time remains — not a blocker.
