# dbStudio Refactor — Python Backend

A Pythonic re-architecture of the dbSherpa Studio workflow engine. Each
node is a single `.yaml` contract + a single `.py` handler. The runtime
auto-discovers them.

> Inspired by the architecture of [rebuild-refactor](https://github.com/sunpratik1772/rebuild-refactor).
> Frontend (React + Vite) and backend (FastAPI) deploy independently.

## Layout

```
backend/
  app/                     # FastAPI HTTP layer
    main.py                # entrypoint, CORS, /healthz, mounts routers
    routers/               # /workflows, /run, /copilot, /node-manifest
  engine/
    context.py             # RunContext (the shared mutable bag)
    registry.py            # auto-discovers NODE_SPEC modules
    node_spec.py           # NodeSpec dataclass + _spec_from_yaml
    dag_runner.py          # topo-sort + per-level executor
    expressions.py         # safe row-expression evaluator
    validator.py           # pure DAG validation
    nodes/                 # ⬅ ONE yaml + ONE py per node
  data_sources/
    registry.py            # loads metadata YAML, exposes datasets
    metadata/              # one yaml per dataset
  copilot/
    workflow_generator.py  # Gemini-driven planner + self-healing loop
  drafts/                  # JSON snapshots of saved workflows
  tests/                   # pytest
  deploy/                  # Cloud Run config (template)
  Dockerfile
  requirements.txt
docs/
  BACKEND_ARCHITECTURE.md
  HOW_TO_DEFINE_A_NODE.md
  ...
```

## Quick start

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env  # add GOOGLE_API_KEY for Copilot + agent node
uvicorn backend.app.main:app --reload --port 8080
```

## Adding a new node

1. `backend/engine/nodes/my_node.yaml` — declarative contract
2. `backend/engine/nodes/my_node.py` — the handler + `NODE_SPEC = _spec_from_yaml(...)`
3. `backend/tests/test_my_node.py` — unit test

The registry auto-discovers it. No central list to edit.
See `docs/HOW_TO_DEFINE_A_NODE.md` for the full pattern.
  