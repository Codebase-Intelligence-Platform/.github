# CodeLens Organization — Overview

This repository contains organization-level documentation and pointers for the CodeLens platform. Below are concise summaries of each component repo to help new contributors quickly understand responsibilities, boundaries, and where to look for details.

## Backend (Backend/)
- Purpose: ingestion, parsing, graph construction, persistence, and API gateway (REST + WebSocket).
- Key responsibilities: Git ingestion, parsers (Tree-sitter), Neo4j graph persistence, PostgreSQL metadata, Kafka messaging, CI-driven infra, and DevOps artifacts (Docker, Helm, Terraform).
- Tech highlights: FastAPI + Go for APIs, Neo4j for graph storage, Kafka for eventing, Redis caching, MinIO/S3 for objects, TimescaleDB for time-series, OpenSearch for search.
- Exposes the canonical APIs used by the Frontend and routes intelligence requests to the Intelligence service.

## Frontend (Frontend/)
- Purpose: interactive UI for exploring real-time graphs, semantic search, impact analysis, and CI dashboards.
- Key responsibilities: rendering large-scale dependency graphs, semantic search panels, diff/impact visualizations, ownership heat maps, and UX integration with Backend APIs (REST + WS).
- Tech highlights: React + TypeScript, Vite, Cytoscape/D3 for graph rendering, Zustand + React Query for state, Tailwind + shadcn for design system, Playwright + Vitest for tests.

## Intelligence (Intelligence/)
- Purpose: AI/ML reasoning layer — embeddings, retrieval, LLM orchestration, and advanced impact/risk analysis.
- Key responsibilities: generate embeddings, run vector search, RAG pipelines, architecture anomaly detection, probabilistic risk scoring, refactoring suggestions, and explanation generation.
- Tech highlights: FastAPI/gRPC gateway, Qdrant/pgvector for vectors, PyTorch & PyG for modeling, vLLM/ONNX/Triton for inference, and MLflow for tracking.

## Platform Contracts (platform-contracts/)
- Purpose: single source of truth for cross-repo contracts — OpenAPI, Protobufs, JSON Schemas, error envelopes, and event schemas.
- Key conventions: canonical error format (code/message/details/request_id/trace), required `x-request-id` propagation, canonical WebSocket envelope shape, and CI contract validation (contract bumps require governance).
- Includes: `openapi/`, `proto/`, `schemas/`, event/topic definitions, prompt registry pointers, and generated types used by Backend, Frontend, and Intelligence.

## How these pieces fit
- The Backend is the ingestion and serving boundary the Frontend talks to; Intelligence provides semantic and LLM reasoning behind the Backend's APIs. `platform-contracts` holds the canonical API and event schemas that coordinate compatibility.

## Where to go next
- Read the component READMEs for detailed setup, quick-starts, and architecture diagrams: [Backend/](../Backend/), [Frontend/](../Frontend/), [Intelligence/](../Intelligence/), [platform-contracts/](../platform-contracts/).

If you want, I can expand any of these summaries, add quick-start commands, or generate a contributor checklist for onboarding.

## Tech Stack
- Languages & runtimes: Python 3.12 (Backend/Intelligence), Go 1.22 (graph/servers), Node.js 20+ (Frontend/Tooling), TypeScript.
- APIs & transport: REST (OpenAPI), WebSocket for streaming, gRPC for internal inference/streaming where applicable.
- Databases & storage: Neo4j (graph), PostgreSQL (metadata + TimescaleDB for time-series), Redis (cache), MinIO/S3 (objects), Qdrant/pgvector (vector store), OpenSearch/Elasticsearch (text search).
- Message bus & queueing: Apache Kafka for ingestion/eventing; Celery/Redis or Kafka consumers for background jobs.
- ML & inference: PyTorch, PyG, vLLM/ONNX/Triton for model serving; MLflow for experiments.
- Frontend stack: React 18 + TypeScript, Vite, Cytoscape/D3 for graphs, Zustand + React Query, Tailwind CSS.

## DevOps & Infrastructure
- Containerization: Docker and OCI images for all services.
- Orchestration: Kubernetes (Helm charts provided) for dev/staging/prod deployments.
- IaC: Terraform modules for cloud resources and environment provisioning.
- CI/CD: GitHub Actions for build/test; ArgoCD or GitOps for continuous deployment to clusters.
- Local dev: Docker Compose compose files to spin up Neo4j, Postgres, Kafka, MinIO, Redis, OpenSearch for local testing.
- Observability: Prometheus + Grafana (metrics/alerts), Tempo/OpenTelemetry (traces), Loki (logs) and structured JSON logging.
- Secrets & config: environment-driven `.env` + secret manager integration (cloud provider or Vault recommended).

## Features (Platform Capabilities)
- Repository ingestion: Git polling, webhook receivers, and direct uploads to ingest codebases.
- Multi-language parsing: Tree-sitter-based parsers to convert source into ASTs and graph nodes.
- Graph model: canonical graph of modules, files, symbols, and edges (imports, calls, dependencies).
- Real-time APIs: REST + WebSocket endpoints for live graph updates, impact streams, and diff streaming.
- Semantic search & RAG: embeddings, vector retrieval, and RAG pipelines for grounded LLM answers.
- Impact & risk analysis: architecture diffs, blast-radius computations, and probabilistic risk scoring.
- Recommendations & automation: refactoring suggestions, policy checks, and CI-integrated architecture validation.
- Visualization: large-scale interactive graph rendering, ownership heatmaps, and diff-aware overlays.
- Governance & contracts: OpenAPI/Protobuf/JSON Schema contracts in `platform-contracts` with CI validation and contract bump process.


