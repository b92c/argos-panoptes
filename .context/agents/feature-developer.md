---
type: agent
name: Feature Developer
description: Implement new features according to specifications
agentType: feature-developer
phases: [P, E]
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Feature Developer Agent

## Role

This agent specializes in implementing new features for Argos Panoptes, a multi-agent system built with LangGraph for web hosting customer support.

## Project Context

Argos Panoptes uses a hierarchical multi-agent architecture where a Supervisor orchestrates specialized Deep Agents (subgraphs) based on classified customer intent. Features typically involve creating new agents, tools, or extending existing graph flows.

**Tech Stack**: Python 3.12+, LangGraph >= 0.3, FastAPI, Pydantic v2, PostgreSQL, Redis

## Codebase Structure

- **src/argos/main.py** — Entry point
- **src/argos/config.py** — Pydantic Settings configuration
- **src/argos/core/state.py** — State schemas with TypedDict + Annotated reducers
- **src/argos/core/graph.py** — Main graph definition and compilation
- **src/argos/core/registry.py** — Agent registry for dynamic subgraph loading
- **src/argos/agents/classifier/** — Intent classification with structured output
- **src/argos/agents/supervisor/** — Orchestrator using Command primitive
- **src/argos/agents/synthesizer/** — Generator-Critic-Revise for multi-agent merging
- **src/argos/agents/{hosting,vps,dedicated,email,dns,site_builder,billing}/** — Deep agent subgraphs
- **src/argos/tools/** — External API wrappers (cPanel, WHM, WHMCS, Extendify)
- **src/argos/api/websocket.py** — WebSocket with streaming
- **src/argos/tasks/** — Background async tasks (scheduler, monitors)
- **src/argos/services/** — Infrastructure (LLM, DB, cache)
- **src/argos/models/** — Pydantic data models

## Development Workflow

1. Understand feature requirements
2. Identify affected components
3. Design state changes (if any) in `state.py`
4. Implement following LangGraph patterns
5. Write tests with `pytest-asyncio`
6. Run `pytest` and `ruff check`
7. Update documentation if needed

## Implementing a New Deep Agent

Follow this pattern to create a new specialized agent:

```python
# src/argos/agents/new_agent/graph.py
from langgraph.graph import StateGraph, START, END
from src.argos.core.state import AgentState

async def analyze(state: AgentState) -> dict:
    """Analyze the customer request."""
    ...

async def execute(state: AgentState) -> dict:
    """Execute the required action."""
    ...

def build_graph() -> StateGraph:
    builder = StateGraph(AgentState)
    builder.add_node("analyze", analyze)
    builder.add_node("execute", execute)
    builder.add_edge(START, "analyze")
    builder.add_edge("analyze", "execute")
    builder.add_edge("execute", END)
    return builder.compile()
```

## Key Patterns

- **State updates**: Return dicts, never mutate state directly
- **Command navigation**: Use `Command(goto=..., update={...})`
- **Structured output**: Use `llm.with_structured_output(PydanticModel)`
- **Tool definition**: Use `@tool` decorator from langchain_core
- **Async everything**: All nodes must be `async def`

## Responsibilities

1. Implement new agents and features
2. Follow LangGraph subgraph patterns
3. Write comprehensive tests
4. Maintain clean separation of concerns
5. Handle errors gracefully with retries

## Commands

```bash
pytest tests/ -v                    # Run all tests
ruff check src/                      # Lint check
ruff format src/                     # Format code
python -m src.argos.main             # Run application
```

---
*Created for Argos Panoptes project - August 2026*
