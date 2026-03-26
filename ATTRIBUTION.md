# Attribution

NEXUS is built upon [create-context-graph](https://github.com/neo4j-labs/create-context-graph)
by Neo4j Labs, licensed under the Apache License 2.0. We are grateful to the Neo4j Labs team for
creating this exceptional scaffolding framework.

## Original Project

- **Name:** create-context-graph
- **Author:** Neo4j Labs contributors (primary: William Lyon)
- **Repository:** https://github.com/neo4j-labs/create-context-graph
- **License:** Apache 2.0
- **Version forked from:** v0.5.0

## Fork

- **Name:** NEXUS — Conflict Intelligence Graph Platform
- **Maintained by:** TACITUS (https://tacitus.me)
- **Contact:** gc@tacitus.me
- **Repository:** https://github.com/sargonxg/cc-graph-TACITUS-upgrade

## What Changed

This fork specializes create-context-graph entirely for conflict analysis and dispute resolution:

- Package renamed: `create_context_graph` → `nexus_conflict_graph`
- CLI command renamed: `create-context-graph` → `nexus-conflict-graph`
- Generic industry domains replaced with conflict-specialized domains based on the
  Agentic Conflict Ontology (ACO) v2.0
- Base ontology evolved from POLE+O to Conflict Structural Layer (CSL) grounded in
  Fisher/Ury, Galtung, Glasl, Lederach, PLOVER/CAMEO
- Conflict-specific data connectors (ACLED, UCDP, GDELT)
- TACITUS design system for generated frontends

## License

All original code remains under Apache 2.0. New contributions are also Apache 2.0.
See LICENSE for full terms.
