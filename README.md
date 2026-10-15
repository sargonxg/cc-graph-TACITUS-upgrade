# NEXUS — historical TACITUS prototype

[![TACITUS](https://img.shields.io/badge/TACITUS-historical%20prototype-6B7280)](https://tacitus.me)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)

> **Status: superseded and no longer maintained.** This repository is retained as an architectural experiment and attribution record. It is not part of the current TACITUS product surface.

NEXUS explored whether a conflict-analysis application generator could combine a FastAPI backend, Next.js interface, Neo4j graph, agent framework, and a typed conflict ontology. It was built on Neo4j Labs' `create-context-graph` work and adapted for policy, mediation, workplace, commercial, community, and interpersonal conflict scenarios.

The experiment helped clarify several requirements that now sit elsewhere in the TACITUS architecture: source-level provenance, review gates, portable context objects, temporal state, and a stable contract between analytical engines and the user-facing workbench.

## Why this repository remains public

- It preserves the experimental implementation and its commit history.
- It records the upstream dependency and TACITUS-specific adaptations.
- It shows an earlier design path that has since been consolidated into newer projects.

## Current successors

- [DIALECTICA](https://github.com/sargonxg/A3_DIALECTICAbyTACITUS_v3) — the active context-capsule and review engine.
- [Tacitus Policy Workbench](https://github.com/sargonxg/tacitus-political-policy-second-brain) — the active local-first policy-workbench integration lab.
- [AGON](https://github.com/sargonxg/AGON) — experimental evidence and conflict-perception engine.
- [KAIROS](https://github.com/sargonxg/KAIROS-temporal-vision-TACITUS) — experimental temporal-analysis engine.

No support, release cadence, hosted service, or compatibility guarantee is provided for NEXUS.

## Attribution and license

Built upon [Neo4j Labs' create-context-graph](https://github.com/neo4j-labs/create-context-graph). See [ATTRIBUTION.md](ATTRIBUTION.md) and [LICENSE](LICENSE). The repository is retained under the Apache License 2.0 terms recorded in the project.
