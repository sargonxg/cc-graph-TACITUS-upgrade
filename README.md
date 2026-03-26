# NEXUS — Conflict Intelligence Graph Platform

[![TACITUS](https://img.shields.io/badge/TACITUS-suite-navy)](https://tacitus.me)
[![Built on Neo4j Labs](https://img.shields.io/badge/Built_on-Neo4j_Labs_create--context--graph-blue?logo=neo4j)](https://github.com/neo4j-labs/create-context-graph)
[![License](https://img.shields.io/badge/license-Apache_2.0-green)](LICENSE)

> **Part of the [TACITUS](https://tacitus.me) suite** — Built upon [create-context-graph](https://github.com/neo4j-labs/create-context-graph) by Neo4j Labs (Apache 2.0). See [ATTRIBUTION.md](ATTRIBUTION.md).

NEXUS (Network of EXplicit Understanding Structures) is an interactive CLI tool that generates
production-ready conflict analysis applications in under 5 minutes. Select a conflict domain and
AI agent framework — receive a complete full-stack app with a FastAPI backend, Next.js frontend,
Neo4j conflict graph, and an AI agent grounded in the Agentic Conflict Ontology (ACO) v2.0.

---

## Conflict Domains

| Domain | Focus | Use Case |
|--------|-------|----------|
| `geopolitical-conflict` | Interstate/intrastate armed conflict, peace processes, sanctions | UN agencies, think tanks, policy research |
| `workplace-dispute` | HR disputes, harassment, grievances, team dysfunction | HR departments, employment law, mediation firms |
| `commercial-dispute` | Contract breaches, IP conflicts, litigation | Legal teams, arbitration bodies |
| `community-conflict` | Neighborhood tensions, public policy battles, protest movements | Local governments, NGOs, community mediators |
| `interpersonal-conflict` | Family disputes, divorce/custody, neighbor conflicts | Mediation practitioners, family law |

---

## Quick Start

### Python (recommended)

```bash
uvx nexus-conflict-graph
```

### Node.js

```bash
npx nexus-conflict-graph
```

### Non-interactive mode

```bash
uvx nexus-conflict-graph my-app --domain geopolitical-conflict --framework pydanticai --demo-data
```

### List available domains

```bash
uvx nexus-conflict-graph --list-domains
```

---

## Neo4j Setup

NEXUS requires a Neo4j database. Choose the option that fits your environment:

### Option 1: Neo4j Aura (Cloud — recommended, free tier available)

1. Sign up at [neo4j.com/cloud/aura](https://neo4j.com/cloud/aura/) — the **free tier** is sufficient for development
2. Create an instance, download the credentials `.env` file
3. Pass it to NEXUS:
   ```bash
   uvx nexus-conflict-graph my-app --domain geopolitical-conflict \
     --neo4j-aura-env /path/to/Neo4j-Aura-credentials.env
   ```
4. Your generated `.env` is automatically populated with Aura credentials

Your generated app `.env` looks like:
```env
NEO4J_URI=neo4j+s://xxxxxxxx.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your-aura-password
```

### Option 2: Local Neo4j (no Docker required)

```bash
uvx nexus-conflict-graph my-app --domain workplace-dispute --neo4j-local
```

Uses [neo4j-local](https://github.com/johnymontana/neo4j-local) via `npx`. No Docker or Neo4j Desktop needed.
The generated `Makefile` includes `make neo4j-start`, `make neo4j-stop`, `make neo4j-status`.

### Option 3: Docker

```bash
docker run -d \
  --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/yourpassword \
  neo4j:5.26.0
# Browser UI: http://localhost:7474
```

In the NEXUS wizard, choose **"Docker"** and enter your credentials.

### Option 4: Existing Neo4j Instance

If you already have Neo4j running (local, cloud, or self-hosted), choose **"Existing"** in the wizard:

| Variable | Example |
|----------|---------|
| `NEO4J_URI` | `bolt://localhost:7687` or `neo4j+s://xxx.databases.neo4j.io` |
| `NEO4J_USERNAME` | `neo4j` |
| `NEO4J_PASSWORD` | your password |

---

## AI API Keys

The generated `.env.example` shows where to add your keys:

```env
# Anthropic (for Claude-based frameworks: pydanticai, claude-agent-sdk, anthropic-tools, strands, langgraph)
ANTHROPIC_API_KEY=your-anthropic-key

# OpenAI (for openai-agents framework)
OPENAI_API_KEY=your-openai-key

# Google (for google-adk framework)
GOOGLE_API_KEY=your-google-key
```

---

## Supported Agent Frameworks

| Framework | Full Streaming | Default Model |
|-----------|---------------|---------------|
| `pydanticai` | Yes | Claude Sonnet |
| `claude-agent-sdk` | Yes | Claude Sonnet |
| `openai-agents` | Yes | GPT-4o |
| `langgraph` | Yes | Claude Sonnet |
| `crewai` | Tools only | Claude Sonnet |
| `strands` | Tools only | Claude Sonnet |
| `google-adk` | Yes | Gemini Pro |
| `anthropic-tools` | Yes | Claude Sonnet |

---

## Running Your Generated App

After scaffolding:

```bash
cd my-conflict-app

# 1. Copy and fill in credentials
cp .env.example .env
# Edit .env: set NEO4J_URI, NEO4J_PASSWORD, ANTHROPIC_API_KEY (or other AI key)

# 2. Install dependencies
make install

# 3. Seed the conflict knowledge graph
make seed

# 4. Start backend + frontend
make start
# Backend API docs: http://localhost:8000/docs
# Frontend:         http://localhost:3000
```

---

## Deployment

### Backend — Google Cloud Run

The generated FastAPI backend includes a `Dockerfile` pre-configured for Cloud Run.

```bash
cd my-conflict-app/backend

# Authenticate
gcloud auth login
gcloud config set project YOUR_GCP_PROJECT

# Build and push
gcloud builds submit --tag gcr.io/YOUR_GCP_PROJECT/nexus-backend .

# Deploy (replace env vars with your actual values)
gcloud run deploy nexus-backend \
  --image gcr.io/YOUR_GCP_PROJECT/nexus-backend \
  --platform managed \
  --region us-central1 \
  --port 8000 \
  --set-env-vars "NEO4J_URI=neo4j+s://xxxx.databases.neo4j.io" \
  --set-env-vars "NEO4J_USERNAME=neo4j" \
  --set-env-vars "NEO4J_PASSWORD=your-aura-password" \
  --set-env-vars "ANTHROPIC_API_KEY=your-key" \
  --allow-unauthenticated
```

> **Tip:** Use [Neo4j Aura](https://neo4j.com/cloud/aura/) with Cloud Run — its `neo4j+s://` URI is publicly reachable with no VPC configuration.

The deployed backend URL will be: `https://nexus-backend-xxxx-uc.a.run.app`

### Frontend — Vercel (recommended)

```bash
cd my-conflict-app/frontend
npm i -g vercel
vercel deploy
# When prompted for environment variables, set:
# NEXT_PUBLIC_API_URL = https://nexus-backend-xxxx-uc.a.run.app
```

Or connect your GitHub repo to [vercel.com](https://vercel.com) and add:

```
NEXT_PUBLIC_API_URL = https://nexus-backend-xxxx-uc.a.run.app
```

Vercel deploys the Next.js frontend zero-config. Preview URLs are created automatically on every push.

### Frontend — Google Cloud Run (alternative)

```bash
cd my-conflict-app/frontend

# Build the Next.js app
npm run build

# Build container and deploy
gcloud builds submit --tag gcr.io/YOUR_GCP_PROJECT/nexus-frontend .
gcloud run deploy nexus-frontend \
  --image gcr.io/YOUR_GCP_PROJECT/nexus-frontend \
  --platform managed \
  --region us-central1 \
  --set-env-vars "NEXT_PUBLIC_API_URL=https://nexus-backend-xxxx-uc.a.run.app" \
  --allow-unauthenticated
```

### Full Stack — Docker Compose (local or any VM)

```bash
cd my-conflict-app
docker compose up --build
# Neo4j browser: http://localhost:7474
# Backend API:   http://localhost:8000/docs
# Frontend:      http://localhost:3000
```

---

## Development (Contributing to NEXUS itself)

```bash
git clone https://github.com/sargonxg/cc-graph-TACITUS-upgrade.git
cd cc-graph-TACITUS-upgrade

# Set up Python environment
uv venv
source .venv/bin/activate     # Windows: .venv\Scripts\activate
uv pip install -e ".[dev]"

# Run tests (no Neo4j or API keys required)
pytest tests/ -v

# Run the CLI
nexus-conflict-graph --help
nexus-conflict-graph --list-domains

# Scaffold a test app
nexus-conflict-graph /tmp/test-app --domain geopolitical-conflict \
  --framework pydanticai --demo-data --dry-run
```

---

## Architecture

```
src/nexus_conflict_graph/
├── cli.py              # Click CLI entry point
├── wizard.py           # Interactive conflict-specialized wizard
├── config.py           # ProjectConfig Pydantic model
├── ontology.py         # Conflict Structural Layer (CSL) ontology loader
├── custom_domain.py    # LLM-powered custom conflict domain generator
├── renderer.py         # Jinja2 template engine
├── generator.py        # Conflict data generation pipeline
├── name_pools.py       # Conflict-specific name pools + Glasl stage data
├── ingest.py           # Neo4j ingestion (ACO memory + direct driver)
├── neo4j_validator.py  # Connection testing
├── connectors/         # Data connectors (ACLED, UCDP, SaaS)
├── domains/            # Conflict domain YAMLs + _base.yaml (CSL)
├── fixtures/           # Pre-generated conflict scenario fixtures
└── templates/          # Jinja2 templates for scaffolded projects
```

**Theoretical foundations of ACO v2.0:**
Fisher/Ury · Galtung · Glasl 9-stage escalation · Lederach · PLOVER/CAMEO · UCDP/ACLED standards

---

## Attribution

Built upon [create-context-graph](https://github.com/neo4j-labs/create-context-graph) by Neo4j Labs (Apache 2.0).
See [ATTRIBUTION.md](ATTRIBUTION.md).

## License

Apache 2.0 — see [LICENSE](LICENSE).
