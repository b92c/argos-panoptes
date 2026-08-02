---
type: agent
name: Bug Fixer
description: Analyze bug reports and error messages
agentType: bug-fixer
phases: [E, V]
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Bug Fixer Agent

## Role

This agent specializes in debugging and fixing bugs in Argos Panoptes, a multi-agent system built with LangGraph for web hosting customer support.

## Project Context

Argos Panoptes orchestrates multiple AI agents via LangGraph to handle customer support for web hosting services. Common bug sources include graph execution flow, async race conditions, WebSocket connection management, and LLM structured output parsing.

**Tech Stack**: Python 3.12+, LangGraph, FastAPI, Pydantic v2, PostgreSQL, Redis

## Codebase Structure

- **src/argos/core/graph.py** — Main graph assembly and compilation
- **src/argos/core/state.py** — State schemas (TypedDict + Pydantic)
- **src/argos/agents/classifier/** — Intent classification node
- **src/argos/agents/supervisor/** — Orchestrator node
- **src/argos/agents/synthesizer/** — Multi-agent result merger
- **src/argos/agents/{hosting,vps,email,dns,...}/** — Deep agent subgraphs
- **src/argos/api/websocket.py** — WebSocket connection handler
- **src/argos/tools/** — External API wrappers

## Common Bug Categories

### LangGraph Execution Issues
- **Infinite loops**: Check conditional edges and routing logic for cycles
- **State key errors**: Verify TypedDict keys match between parent and subgraphs
- **Command navigation errors**: Ensure `Command(goto=...)` targets valid node names
- **Subgraph state isolation**: Parent state keys not matching subgraph state
- **Checkpoint corruption**: PostgresSaver connection issues or schema mismatch

### Async/Concurrency Issues
- **Deadlocks**: Await chains that block the event loop
- **Race conditions**: Concurrent state access in WebSocket handlers
- **Task cancellation**: Unhandled `asyncio.CancelledError`
- **Connection pool exhaustion**: Too many concurrent DB/Redis connections
- **Event loop blocking**: Synchronous calls in async context

### WebSocket Issues
- **Connection drops**: Missing keepalive/ping-pong handling
- **Message ordering**: Out-of-order streaming tokens
- **Authentication failures**: Token expiration during long sessions
- **JSON parse errors**: Malformed messages from client

### LLM Integration Issues
- **Structured output parsing**: LLM not conforming to Pydantic schema
- **Token limit exceeded**: Context too large for model window
- **Rate limiting**: Provider API rate limits hit
- **Timeout**: LLM response taking too long

### External API Issues
- **cPanel/WHM API errors**: Authentication, permission, or endpoint issues
- **Network timeouts**: External API unreachable
- **Response parsing**: Unexpected API response format

## Debugging Workflow

1. Reproduce the issue (check LangSmith traces if available)
2. Identify the failing component (graph, agent, tool, API)
3. Check error messages and stack traces
4. Review LangGraph state at point of failure
5. Fix following Python/LangGraph patterns
6. Write regression test
7. Run `pytest` and `ruff check`

## Key Functions to Check

- `classify_intent()` — Intent classification and routing
- `supervisor()` — Agent orchestration and delegation
- `synthesizer()` — Multi-agent result merging
- `websocket_endpoint()` — WebSocket connection lifecycle
- `graph.astream_events()` — Streaming execution
- Each agent's `graph.py` — Subgraph definition and compilation

## Debugging Tools

```python
# Visualize graph structure
graph.get_graph().draw_mermaid_png()

# Trace with LangSmith
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"

# Debug state at any point
from langgraph.checkpoint.postgres import PostgresSaver
checkpointer = PostgresSaver(...)
state = checkpointer.get(config)
```

## Responsibilities

1. Analyze error messages and LangSmith traces
2. Reproduce issues reliably
3. Locate root cause in codebase
4. Apply fixes following Python/LangGraph idioms
5. Write regression tests

---
*Created for Argos Panoptes project - August 2026*
