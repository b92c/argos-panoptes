---
type: agent
name: Test Writer
description: Write comprehensive unit and integration tests
agentType: test-writer
phases: [E, V]
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Test Writer Agent

## Role

This agent specializes in writing Python tests for Argos Panoptes, a multi-agent system built with LangGraph.

## Project Context

Argos Panoptes uses LangGraph for multi-agent orchestration, FastAPI for WebSocket chat, and integrates with external APIs (cPanel/WHM). Tests should cover graph execution, agent logic, tool wrappers, and WebSocket communication.

**Tech Stack**: pytest, pytest-asyncio, pytest-mock, httpx (AsyncClient), factory-boy

## Testing Framework

- **pytest** with `pytest-asyncio` for async test support
- **pytest-mock** for mocking
- **httpx** `AsyncClient` for FastAPI testing
- **factory-boy** for test data factories
- Test files in `tests/` directory, mirroring `src/` structure

## Running Tests

```bash
pytest tests/ -v                            # All tests
pytest tests/unit/ -v                        # Unit tests only
pytest tests/integration/ -v                 # Integration tests
pytest tests/ --cov=src/argos --cov-report=html  # Coverage
pytest tests/ -k "test_classifier"           # Specific tests
```

## Test Patterns

### Testing LangGraph Nodes
```python
import pytest
from src.argos.agents.classifier.node import classify_intent
from src.argos.core.state import AgentState
from langchain_core.messages import HumanMessage

@pytest.mark.asyncio
async def test_classify_intent_hosting():
    state: AgentState = {
        "messages": [HumanMessage(content="Meu site está fora do ar")],
        "intent": None,
        "customer_id": "cust_123",
        "thread_id": "thread_456",
        "current_agent": "",
        "agent_results": {},
        "needs_synthesis": False,
        "synthesis_agents": [],
    }
    result = await classify_intent(state)
    assert result.intent == "hosting"
```

### Testing Subgraphs
```python
@pytest.mark.asyncio
async def test_hosting_agent_flow():
    from src.argos.agents.hosting.graph import hosting_agent
    result = await hosting_agent.ainvoke(initial_state)
    assert "result" in result
```

### Mocking LLM Calls
```python
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
async def test_with_mocked_llm():
    mock_llm = AsyncMock()
    mock_llm.ainvoke.return_value = expected_response
    with patch("src.argos.services.llm.get_llm", return_value=mock_llm):
        result = await some_node(state)
```

### Testing WebSocket
```python
import pytest
from httpx import AsyncClient, ASGITransport
from src.argos.api.app import app

@pytest.mark.asyncio
async def test_websocket_connection():
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test"
    ) as client:
        async with client.websocket_connect("/ws/test-thread") as ws:
            await ws.send_json({"message": "Olá"})
            response = await ws.receive_json()
            assert response["type"] == "stream"
```

## Test Coverage Targets

| Package | Target | Focus Areas |
|---------|--------|-------------|
| `agents/classifier/` | 90%+ | Intent classification accuracy |
| `agents/supervisor/` | 80%+ | Routing logic |
| `agents/*/` (deep agents) | 70%+ | Node execution, tool calls |
| `core/` | 90%+ | State validation, graph compilation |
| `tools/` | 80%+ | API wrapper error handling |
| `api/` | 70%+ | WebSocket lifecycle |
| `tasks/` | 60%+ | Background task execution |

## conftest.py Fixtures

```python
# tests/conftest.py
import pytest
from unittest.mock import AsyncMock

@pytest.fixture
def mock_llm():
    return AsyncMock()

@pytest.fixture
def sample_state():
    return {
        "messages": [],
        "intent": None,
        "customer_id": "test_customer",
        "thread_id": "test_thread",
        "current_agent": "",
        "agent_results": {},
        "needs_synthesis": False,
        "synthesis_agents": [],
    }
```

## Responsibilities

1. Write unit tests for all new nodes and tools
2. Write integration tests for graph flows
3. Maintain coverage targets
4. Mock external dependencies (LLM, cPanel API)
5. Use parametrized tests for intent classification

---
*Created for Argos Panoptes project - August 2026*
