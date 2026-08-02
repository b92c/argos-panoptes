---
type: doc
name: development-workflow
description: Day-to-day engineering processes, setup, and contribution guidelines
category: workflow
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Development Workflow

## Prerequisites

- Python 3.12+ installed
- Docker and Docker Compose (for PostgreSQL and Redis)
- A valid LLM API key (OpenAI, Anthropic, or Google)

## Quick Start

```bash
# Clone and setup
git clone https://github.com/b92c/argos-panoptes.git
cd argos-panoptes

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # macOS/Linux

# Install dependencies (including dev tools)
pip install -e ".[dev]"

# Start infrastructure
docker compose up -d  # PostgreSQL + Redis

# Copy environment config
cp .env.example .env
# Edit .env with your API keys

# Run the application
uvicorn src.argos.api.app:app --reload --port 8000

# In another terminal, test WebSocket
python scripts/test_ws_client.py
```

## Branching Strategy

- `main` — Production-ready code
- `fix/*` — Bug fixes
- `feat/*` — New features (agents, tools, nodes)
- `refactor/*` — Code improvements
- `docs/*` — Documentation updates
- `chore/*` — Maintenance tasks

## Development Process

1. Create a branch from `main`
2. Check `.context/agents/` for relevant agent playbook
3. Implement changes following Python/LangGraph patterns
4. Write tests: `pytest tests/ -v`
5. Lint: `ruff check src/ tests/`
6. Type check: `mypy src/ --strict`
7. Create PR for review

## Key Commands

```bash
# Application
uvicorn src.argos.api.app:app --reload --port 8000   # Run dev server
python -m src.argos.main                              # Run standalone

# Testing
pytest tests/ -v                                      # All tests
pytest tests/unit/ -v                                  # Unit tests
pytest tests/integration/ -v                           # Integration tests
pytest tests/ --cov=src/argos --cov-report=html       # Coverage report
pytest tests/ -k "test_classifier" -v                  # Specific tests

# Code Quality
ruff check src/ tests/                                 # Lint
ruff format src/ tests/                                # Format
mypy src/ --strict                                     # Type check

# Infrastructure
docker compose up -d                                   # Start PostgreSQL + Redis
docker compose down                                    # Stop
docker compose logs -f                                 # View logs

# Database
alembic upgrade head                                   # Run migrations
alembic revision --autogenerate -m "description"       # Create migration
```

## Docker Development Environment

```yaml
# docker-compose.yml provides:
# - PostgreSQL 16 (checkpointing + data storage)
# - Redis 7 (caching + session state)
```

## Code Organization Rules

- **src/argos/core/**: Core graph logic only — no external API calls
- **src/argos/agents/**: Each agent in its own directory with graph.py, nodes.py, tools.py, prompts.py
- **src/argos/tools/**: Thin wrappers around external APIs — no business logic
- **src/argos/services/**: Infrastructure abstractions — injectable, mockable
- **src/argos/tasks/**: Background tasks — independent of graph execution
- **src/argos/models/**: Pure Pydantic models — no logic

## Adding a New Deep Agent

1. Create directory: `src/argos/agents/new_agent/`
2. Add files: `__init__.py`, `graph.py`, `nodes.py`, `tools.py`, `prompts.py`
3. Define subgraph in `graph.py` using `StateGraph`
4. Register in `src/argos/core/registry.py`
5. Add intent mapping in classifier prompts
6. Add routing in supervisor
7. Write tests in `tests/unit/test_new_agent.py`
8. Update `.context/agents/` if new playbook needed

---
*Created for Argos Panoptes project - August 2026*
