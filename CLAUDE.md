# CLAUDE.md — NEXUS Conflict Intelligence Graph Platform

## Project Overview

**NEXUS** (Network of EXplicit Understanding Structures) is an interactive CLI scaffolding tool that
generates domain-specific conflict analysis applications. Like `create-next-app` but for conflict
intelligence — powered by Neo4j, FastAPI, and AI agents grounded in the Agentic Conflict Ontology
(ACO) v2.0.

Invoked via `uvx nexus-conflict-graph` or `npx nexus-conflict-graph`.

**Part of the [TACITUS](https://tacitus.me) product suite.**
**Built upon [create-context-graph](https://github.com/neo4j-labs/create-context-graph) by Neo4j Labs (Apache 2.0).**

### Conflict Domains (5)
`geopolitical-conflict` · `workplace-dispute` · `commercial-dispute` · `community-conflict` · `interpersonal-conflict`

### Theoretical Foundations (ACO v2.0)
Fisher/Ury · Galtung · Glasl 9-stage escalation · Lederach · PLOVER/CAMEO · UCDP/ACLED

### Status
Phase 1-3 rebranding + CSL base complete. Domains in progress (Prompts 4-11).

---

## Quick Reference

```bash
# Setup
uv venv && uv pip install -e ".[dev]"

# Run tests
source .venv/bin/activate && pytest tests/ -v

# Test scaffold generation
nexus-conflict-graph my-app --domain geopolitical-conflict --framework pydanticai --demo-data

# List conflict domains
nexus-conflict-graph --list-domains
```

---

## Architecture

```
src/nexus_conflict_graph/
├── cli.py              # Click CLI entry point (interactive + flag modes)
├── wizard.py           # Conflict-specialized interactive wizard
├── config.py           # ProjectConfig Pydantic model
├── ontology.py         # YAML ontology loader with CSL base (10 entity types)
├── custom_domain.py    # LLM-powered custom conflict domain generator
├── renderer.py         # Jinja2 template engine
├── generator.py        # LLM-powered conflict data pipeline (4 stages)
├── name_pools.py       # Conflict-specific name pools + Glasl stage data
├── ingest.py           # Neo4j ingestion via neo4j-agent-memory or direct driver
├── neo4j_validator.py  # Neo4j connection testing
├── connectors/         # Data connectors (ACLED, UCDP, SaaS)
│   ├── __init__.py     # BaseConnector ABC, NormalizedData model, registry
│   ├── github_connector.py
│   ├── notion_connector.py
│   ├── jira_connector.py
│   ├── slack_connector.py
│   ├── gmail_connector.py
│   ├── gcal_connector.py
│   ├── salesforce_connector.py
│   └── oauth.py
├── domains/            # Conflict domain YAMLs + _base.yaml (CSL)
├── fixtures/           # Pre-generated conflict scenario fixtures
└── templates/          # Jinja2 templates for generated projects
    ├── base/           # .env, .env.example, Makefile, docker-compose, README, .gitignore
    ├── backend/
    │   ├── shared/     # FastAPI main, config, neo4j client, GDS, vector, models, routes
    │   ├── agents/     # Per-framework agent.py (8 frameworks)
    │   └── connectors/ # Generated connector modules + import_data.py
    ├── frontend/       # Next.js + Chakra UI v3 + NVL conflict visualization
    └── cypher/         # Schema constraints + GDS projections
```

---

## Conflict Structural Layer (CSL) — Base Ontology

`domains/_base.yaml` implements the **ACO v2.0** with 10 primitive entity types:

| Entity | ACO Type | Color | Description |
|--------|----------|-------|-------------|
| Actor | PERSON | #22c55e | Conflict party (individual/group/state) |
| Claim | OBJECT | #ef4444 | Asserted right or position |
| Interest | OBJECT | #f59e0b | Underlying need or goal |
| Constraint | OBJECT | #6366f1 | Structural limit on action |
| Leverage | OBJECT | #ec4899 | Power or pressure instrument |
| Commitment | OBJECT | #14b8a6 | Agreement or obligation |
| ConflictEvent | EVENT | #f97316 | Incident with optional Glasl stage + PLOVER code |
| Narrative | OBJECT | #8b5cf6 | Framing / discourse driving conflict |
| Location | LOCATION | #a855f7 | Territory with strategic value |
| Organization | ORGANIZATION | #3b82f6 | Institution, armed group, IGO, NGO |

**Base relationships (17+):**
ASSERTS · HOLDS · CONSTRAINED_BY · WIELDS · BOUND_BY · TRIGGERED · PROPAGATES ·
OPPOSES · ALLIED_WITH · MEDIATES · LOCATED_IN · MEMBER_OF · CONTESTED_OVER ·
UNDERMINES · ESCALATED_BY · PRECEDED_BY · VIOLATES

---

## Conflict Domain YAML Schema

Each domain YAML must contain:
- `inherits: _base` — merges all 10 CSL entity types + 17 relationships
- `domain:` — id, name, description, tagline, emoji
- `entity_types:` — domain-specific entities (label, pole_type, color, icon, properties)
- `relationships:` — type, source, target
- `document_templates:` — conflict analysis documents (sit-reps, briefs, assessments)
- `decision_traces:` — conflict reasoning traces
- `demo_scenarios:` — name + prompts list for the chat UI
- `agent_tools:` — Cypher-backed tools for conflict graph queries
- `system_prompt:` — ACO-grounded conflict analyst system prompt

**Glasl stage field:** `ConflictEvent.glasl_stage` (integer 1-9) tracks escalation.
**PLOVER code field:** `ConflictEvent.plover_code` (string) encodes event taxonomy.

---

## Key Design Decisions

### Templates are domain-agnostic, data-driven
No per-domain template directories. The ontology YAML drives all customization via Jinja2.
Only `backend/agents/{framework}/agent.py.j2` varies by framework.

### ACO primitives as base entity types
Unlike the original POLE+O base, CSL defines 10 conflict-specific primitives. Domain YAMLs
add domain-specific entity types on top. `pole_type` field maintained for backward compat
with NVL rendering engine.

### Jinja2 + JSX/Python escaping
Templates with JSX curly braces must use `{% raw %}...{% endraw %}` blocks.

### Dual ingestion backends
`ingest.py` tries `neo4j-agent-memory` MemoryClient first (all 3 memory types), falls back
to direct `neo4j` driver. Both paths tag entities with `domain` property.

### Streaming chat via Server-Sent Events (SSE)
Backend emits SSE events as the agent executes: `tool_start`, `tool_end`, `text_delta`, `done`.
Frontend parses via `fetch` + `ReadableStream`. Tool calls render as Chakra UI Timeline.

### Thread-safe async bridging for sync frameworks
CrewAI and Strands use `asyncio.run_coroutine_threadsafe()` for tool calls. `_push_event()`
uses `loop.call_soon_threadsafe()` for worker thread safety.

### Bounded agentic loops
Claude Agent SDK and Anthropic Tools: max 15 iterations, 60s timeout per API call.

---

## Generated Project Structure

```
my-conflict-app/
├── backend/app/          # FastAPI + chosen agent framework
│   ├── main.py, config.py, routes.py, models.py
│   ├── agent.py          # Framework-specific conflict analyst
│   ├── context_graph_client.py  # Neo4j + ACO memory
│   ├── gds_client.py, vector_client.py
│   └── connectors/       # Only if SaaS connectors selected
├── backend/tests/
│   └── test_routes.py    # Health + scenario tests (mocked Neo4j)
├── backend/scripts/
│   └── generate_data.py
├── backend/pyproject.toml
├── frontend/             # Next.js + Chakra UI v3 + NVL conflict graph
│   ├── app/ (layout.tsx, page.tsx)
│   ├── components/
│   │   ├── ChatInterface.tsx      # SSE streaming + Timeline tool viz
│   │   ├── ContextGraphView.tsx   # NVL interactive conflict graph
│   │   ├── DecisionTracePanel.tsx
│   │   └── DocumentBrowser.tsx
│   └── lib/config.ts (NODE_COLORS, RELATIONSHIP_COLORS, GLASL_COLORS)
├── cypher/ (schema.cypher, gds_projections.cypher)
├── data/ (ontology.yaml, _base.yaml, fixtures.json)
├── .env, .env.example, Makefile, docker-compose.yml, README.md
```

**Running a generated app:**
```bash
cp .env.example .env  # fill in NEO4J_URI, NEO4J_PASSWORD, ANTHROPIC_API_KEY
make install && make seed && make start
# Backend: http://localhost:8000/docs
# Frontend: http://localhost:3000
```

---

## Deployment

### Generated App — Backend to Google Cloud Run

```bash
cd my-conflict-app/backend
gcloud builds submit --tag gcr.io/PROJECT/nexus-backend
gcloud run deploy nexus-backend \
  --image gcr.io/PROJECT/nexus-backend \
  --set-env-vars "NEO4J_URI=neo4j+s://xxx.databases.neo4j.io,NEO4J_PASSWORD=pw,ANTHROPIC_API_KEY=key" \
  --allow-unauthenticated
```

### Generated App — Frontend to Vercel

```bash
cd my-conflict-app/frontend
vercel deploy --env NEXT_PUBLIC_API_URL=https://nexus-backend-xxxx.run.app
```

Or connect GitHub repo → Vercel dashboard, set `NEXT_PUBLIC_API_URL`.

---

## Testing

```bash
pytest tests/ -v                    # All tests (no external deps)
pytest tests/test_config.py         # Config model + framework aliases
pytest tests/test_ontology.py       # CSL base + all conflict domains
pytest tests/test_renderer.py       # Template rendering, all 8 frameworks
pytest tests/test_generated_project.py  # Deep validation
pytest tests/ -v -m slow            # Include slow matrix tests
make smoke-test                     # E2E (requires Neo4j + API keys)
```

---

## Makefile Targets

| Target | Description |
|--------|-------------|
| `make test` | Fast unit tests (no external deps) |
| `make test-slow` | Full suite including domain × framework matrix |
| `make test-coverage` | Tests with HTML coverage report |
| `make smoke-test` | E2E smoke tests (requires Neo4j + API keys) |
| `make lint` | Run ruff on `src/` and `tests/` |
| `make scaffold` | Scaffold a test project to `/tmp/test-scaffold` |

---

## Adding a New Conflict Domain

1. Create `src/nexus_conflict_graph/domains/{domain-id}.yaml` with `inherits: _base`
2. Define domain-specific entities that extend the CSL primitives
3. Include rich `system_prompt` encoding conflict analysis methodology for the domain
4. Generate or create `src/nexus_conflict_graph/fixtures/{domain-id}.json`
5. Verify: `pytest tests/test_ontology.py::TestLoadAllDomains -v`

## Adding a New Agent Framework

1. Create `src/nexus_conflict_graph/templates/backend/agents/{framework_key}/agent.py.j2`
2. Add to `SUPPORTED_FRAMEWORKS`, `FRAMEWORK_DISPLAY_NAMES`, `FRAMEWORK_DEPENDENCIES` in `config.py`
3. Export `async def handle_message(message, session_id) -> dict` with `response`, `session_id`, `graph_data`
4. Import and use `get_conversation_history`, `store_message` from `context_graph_client`
5. Optionally add `handle_message_stream()` for text streaming

---

## CI Pipeline (GitHub Actions)

| Job | Trigger |
|-----|---------|
| test | All pushes + PRs — unit tests on Python 3.11, 3.12 |
| lint | All pushes + PRs — ruff |
| matrix | Push to main — all domain × framework combos |
| smoke-test | Push to main (gated on SMOKE_TESTS_ENABLED=true) |

Required secrets for smoke-test: `NEO4J_URI`, `NEO4J_USERNAME`, `NEO4J_PASSWORD`,
`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GOOGLE_API_KEY`.

---

## Attribution

Built upon [create-context-graph](https://github.com/neo4j-labs/create-context-graph) by Neo4j Labs (Apache 2.0).
Fork maintained by TACITUS (gc@tacitus.me). See [ATTRIBUTION.md](ATTRIBUTION.md).
