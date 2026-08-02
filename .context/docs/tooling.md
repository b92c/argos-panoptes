---
type: doc
name: tooling
description: Scripts, IDE settings, automation, and developer productivity tips
category: tooling
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Tooling

## Required Tools

- **Python 3.12+** — Primary language (async native, performance improvements)
- **Docker** — Container runtime for infrastructure services
- **Docker Compose** — Multi-container orchestration (PostgreSQL, Redis)

## Python Tools

- **ruff** — Fast linter and formatter (replaces black + isort + flake8)
- **mypy** — Static type checker (strict mode recommended)
- **pytest** — Test runner with async support
- **alembic** — Database migration tool
- **structlog** — Structured logging
- **pyinstrument** — Profiling tool

## Dependencies (from pyproject.toml)

### Core
- **langgraph** >= 0.3 — StateGraph, Command, subgraphs, checkpointing
- **langchain-core** — Base abstractions (messages, tools, prompts)
- **langchain-openai** — OpenAI LLM integration
- **fastapi** — Web framework (WebSocket + REST)
- **uvicorn** — ASGI server
- **pydantic** >= 2.0 — Data validation and settings
- **pydantic-settings** — Environment configuration
- **asyncpg** — Async PostgreSQL driver
- **redis[hiredis]** — Redis client with C extension
- **sqlalchemy** >= 2.0 — Async ORM
- **httpx** — Async HTTP client
- **tenacity** — Retry logic for API calls
- **structlog** — Structured logging
- **apscheduler** — Background task scheduling

### Dev
- **pytest** — Testing
- **pytest-asyncio** — Async test support
- **pytest-mock** — Mocking
- **pytest-cov** — Coverage
- **ruff** — Linting + formatting
- **mypy** — Type checking
- **factory-boy** — Test factories
- **httpx** — FastAPI test client

## IDE Configuration

Recommended VS Code extensions:
- **Python** (Microsoft)
- **Pylance** (type checking, intellisense)
- **Ruff** (linting and formatting)
- **Even Better TOML** (pyproject.toml support)

Recommended settings (`.vscode/settings.json`):
```json
{
  "python.defaultInterpreterPath": ".venv/bin/python",
  "python.analysis.typeCheckingMode": "strict",
  "editor.formatOnSave": true,
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.codeActionsOnSave": {
      "source.fixAll": "explicit",
      "source.organizeImports": "explicit"
    }
  },
  "python.testing.pytestEnabled": true,
  "python.testing.pytestArgs": ["tests/"]
}
```

## Useful Scripts

```bash
# Test WebSocket connection
python scripts/test_ws_client.py

# Seed database with test data
python scripts/seed_db.py

# Visualize LangGraph
python -c "from src.argos.core.graph import graph; graph.get_graph().draw_mermaid_png(output_file_path='graph.png')"
```

## LangSmith Setup

```bash
# Add to .env
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your-key
LANGCHAIN_PROJECT=argos-panoptes
```

LangSmith provides:
- Trace visualization for all LLM calls
- Latency breakdown per node
- Cost tracking per interaction
- Dataset management for prompt evaluation

---
*Created for Argos Panoptes project - August 2026*
