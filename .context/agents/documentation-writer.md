---
type: agent
name: Documentation Writer
description: Create clear, comprehensive documentation
agentType: documentation-writer
phases: [P, C]
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Documentation Writer Agent

## Role

This agent specializes in documentation for Argos Panoptes, a multi-agent system for web hosting customer support built with LangGraph and Python.

## Project Context

Argos Panoptes uses LangGraph to orchestrate specialized AI agents for hosting support. Documentation should help developers understand the architecture, contribute agents, and operate the system.

**Tech Stack**: Python 3.12+, LangGraph, FastAPI, Pydantic v2

## Codebase Structure

- **src/argos/** — Main application package
- **docs/** — Project documentation (specs, roadmap, implementation plan)
- **.context/docs/** — AI-consumable project documentation
- **.context/agents/** — Agent playbooks
- **tests/** — Test suite

## Documentation Types

### User Documentation
- **README.md** — Installation, quick start, architecture overview
- **docs/** — Detailed specifications and plans

### Developer Documentation
- **.context/docs/project-overview.md** — Architecture and components
- **.context/docs/development-workflow.md** — Dev process, commands
- **.context/docs/testing-strategy.md** — Test patterns
- **.context/docs/tooling.md** — Tools and IDE setup
- **Code docstrings** — Google-style Python docstrings

### Agent Documentation
- **.context/agents/*.md** — AI agent playbooks
- Prompts documentation in `src/argos/agents/*/prompts.py`

## Python Documentation Standards

### Module Docstrings
```python
"""Intent classification agent for Argos Panoptes.

This module implements the intent classifier node that analyzes
customer messages and routes them to the appropriate specialized agent.
"""
```

### Function Docstrings (Google Style)
```python
async def classify_intent(state: AgentState) -> Command:
    """Classify customer message intent using structured LLM output.

    Analyzes the last message in the conversation to determine which
    specialized agent should handle the request.

    Args:
        state: Current graph state containing message history.

    Returns:
        Command routing to either the supervisor (complex intents)
        or level1_agent (simple intents).

    Raises:
        ValueError: If no messages exist in state.
    """
```

### Class Docstrings
```python
class IntentClassification(BaseModel):
    """Structured output from the intent classifier.

    Attributes:
        intent: The classified intent category.
        confidence: Classification confidence score (0.0 to 1.0).
        reasoning: Brief explanation of the classification.
        requires_deep_agent: Whether this needs a specialized subgraph.
    """
```

## Responsibilities

1. Keep README.md accurate and helpful
2. Maintain .context documentation
3. Add Google-style docstrings for all public functions/classes
4. Document agent prompts and their reasoning
5. Update agent playbooks when patterns change
6. Keep codebase-map.json current

---
*Created for Argos Panoptes project - August 2026*
