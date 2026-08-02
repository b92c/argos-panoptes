# Agent Handbook

This directory contains ready-to-customize playbooks for AI agents collaborating on the Argos Panoptes repository.

## Project Context

**Argos Panoptes** is a multi-agent system for web hosting customer support built with LangGraph and Python. It provides real-time chat support via WebSocket, integrates with hosting control panels (cPanel/WHM), and runs asynchronous background monitoring routines.

## Available Agents
- [Code Reviewer](./code-reviewer.md) — Review Python code for quality, async patterns, and LangGraph best practices
- [Bug Fixer](./bug-fixer.md) — Debug LangGraph graph execution, WebSocket issues, and async race conditions
- [Feature Developer](./feature-developer.md) — Implement new agents, tools, and graph nodes
- [Refactoring Specialist](./refactoring-specialist.md) — Optimize Python code, improve type safety, refactor graph architecture
- [Test Writer](./test-writer.md) — Write pytest tests for agents, tools, graph flows, and WebSocket endpoints
- [Documentation Writer](./documentation-writer.md) — Maintain README, code docstrings, and context docs
- [Performance Optimizer](./performance-optimizer.md) — Optimize LLM calls, async I/O, caching, and streaming latency
- [Agent Architect](./agent-architect.md) — Design new LangGraph subgraphs, state schemas, and multi-agent patterns
- [Prompt Engineer](./prompt-engineer.md) — Craft and optimize system prompts, structured outputs, and classification logic

## How To Use These Playbooks
1. Pick the agent that matches your task.
2. Review the agent's responsibilities and relevant files.
3. Follow Python conventions and project patterns.
4. Run `pytest` and `ruff check` before completing.

## Tech Stack Reference
- **Language**: Python 3.12+
- **Agent Framework**: LangGraph >= 0.3 (StateGraph, Command, subgraphs)
- **LLM Integration**: LangChain Core
- **Web Framework**: FastAPI (WebSocket + REST)
- **State Validation**: Pydantic v2
- **Persistence**: PostgreSQL (checkpointing) + Redis (cache)
- **Testing**: pytest + pytest-asyncio
- **Linting**: ruff
- **Observability**: LangSmith + structlog

## Related Resources
- [Documentation Index](../docs/README.md)
- [Agent Knowledge Base](../../AGENTS.md)
- [Project README](../../README.md)
