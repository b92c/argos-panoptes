# Documentation Index

Welcome to the Argos Panoptes repository knowledge base. Start with the project overview, then dive into specific guides as needed.

## Core Guides
- [Project Overview](./project-overview.md) — Architecture, components, and tech stack
- [Development Workflow](./development-workflow.md) — Setup, commands, branching, and contribution
- [Testing Strategy](./testing-strategy.md) — pytest patterns, async testing, and coverage
- [Tooling & Productivity Guide](./tooling.md) — Python tools, IDE config, dependencies

## Repository Snapshot
- `src/argos/` — Main application package
- `src/argos/core/` — State schemas, main graph, agent registry
- `src/argos/agents/` — All agent implementations (classifier, supervisor, deep agents)
- `src/argos/api/` — FastAPI app, WebSocket, REST endpoints
- `src/argos/tools/` — External API wrappers (cPanel, WHM, WHMCS, Extendify)
- `src/argos/services/` — Infrastructure services (LLM, database, cache)
- `src/argos/tasks/` — Background async tasks
- `src/argos/models/` — Pydantic data models
- `tests/` — Test suite (unit, integration, e2e)
- `docs/` — Detailed project documentation
- `docker-compose.yml` — Development environment (PostgreSQL, Redis)
- `pyproject.toml` — Dependencies and build configuration
- `AGENTS.md` — AI context references

## Document Map
| Guide | File | Primary Inputs |
| --- | --- | --- |
| Project Overview | `project-overview.md` | README, architecture, codebase structure |
| Development Workflow | `development-workflow.md` | pyproject.toml, Docker setup, commands |
| Testing Strategy | `testing-strategy.md` | pytest patterns, coverage commands |
| Tooling & Productivity Guide | `tooling.md` | Python tools, IDE configs, dependencies |

## Quick Reference

### Key Commands
```bash
# Setup
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

# Run
uvicorn src.argos.api.app:app --reload --port 8000

# Test
pytest tests/ -v --cov=src/argos

# Lint
ruff check src/ tests/
ruff format src/ tests/

# Type check
mypy src/ --strict

# Docker (PostgreSQL + Redis)
docker compose up -d
```

### Tech Stack
- **Language**: Python 3.12+
- **Agent Framework**: LangGraph >= 0.3
- **Web Framework**: FastAPI
- **State Validation**: Pydantic v2
- **Persistence**: PostgreSQL + Redis
- **Observability**: LangSmith + structlog
