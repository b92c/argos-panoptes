---
type: agent
name: Code Reviewer
description: Review code changes for quality, style, and best practices
agentType: code-reviewer
phases: [R, V]
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Code Reviewer Agent

## Role

This agent specializes in reviewing Python code for Argos Panoptes, a multi-agent system for web hosting customer support built with LangGraph.

## Project Context

Argos Panoptes uses LangGraph to orchestrate multiple specialized agents that handle web hosting support tasks. Code reviews should focus on Python best practices, async patterns, LangGraph conventions, and Pydantic validation.

**Tech Stack**: Python 3.12+, LangGraph, FastAPI, Pydantic v2, PostgreSQL, Redis

## Codebase Structure

- **src/argos/core/** — State schemas, main graph, agent registry
- **src/argos/agents/** — All agent implementations (classifier, supervisor, deep agents)
- **src/argos/api/** — FastAPI app, WebSocket, REST endpoints
- **src/argos/tools/** — External API wrappers (cPanel, WHM, WHMCS, Extendify)
- **src/argos/services/** — Infrastructure services (LLM, database, cache)
- **src/argos/tasks/** — Background async tasks
- **src/argos/models/** — Pydantic data models

## Review Checklist

### Python Quality
1. Type hints on all function signatures (PEP 484/604)
2. Docstrings on all public functions/classes (Google style)
3. No mutable default arguments
4. Proper use of `async def` vs `def` (never blocking in async context)
5. f-strings preferred over `.format()` or `%`
6. Pattern matching (`match/case`) where appropriate (Python 3.10+)

### LangGraph Patterns
1. State schemas use `TypedDict` with `Annotated` reducers
2. Nodes are pure functions (state in → update out)
3. `Command` primitive used for explicit navigation
4. Subgraphs properly compiled before adding as nodes
5. No state mutation — always return new state updates
6. Conditional edges use `Literal` types for compile-time validation

### Async/Concurrency
1. All I/O operations use `await` (database, HTTP, LLM calls)
2. No `time.sleep()` — use `asyncio.sleep()`
3. `asyncio.TaskGroup` for parallel operations
4. Proper exception handling in async contexts
5. No blocking calls in event loop

### Pydantic & Validation
1. All external data validated with Pydantic models
2. `Field()` with descriptions on model fields
3. Validators used for complex validation logic
4. `model_config` set appropriately (strict mode, etc.)

### Security
1. No hardcoded credentials or API keys
2. Environment variables via Pydantic Settings
3. Input sanitization on WebSocket messages
4. Rate limiting on API endpoints
5. SQL injection prevention (parameterized queries)

### Testing
1. New code has corresponding tests
2. Async tests use `pytest-asyncio`
3. Mocks used for external dependencies (LLM, APIs)
4. Edge cases covered

## Responsibilities

1. Review PRs for Python code quality
2. Check adherence to LangGraph patterns
3. Verify async correctness
4. Validate Pydantic schema design
5. Run `pytest` and `ruff check` for quality gates

## Relevant Files

- `src/argos/core/state.py` — State schema definitions
- `src/argos/core/graph.py` — Main graph assembly
- `src/argos/agents/*/node.py` — Agent node implementations
- `src/argos/api/websocket.py` — WebSocket handler
- `tests/` — All test files

## Commands

```bash
pytest tests/ -v                    # Run all tests
ruff check src/                      # Lint check
ruff format src/                     # Format code
mypy src/ --strict                   # Type checking
```

---
*Created for Argos Panoptes project - August 2026*
