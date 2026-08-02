---
type: doc
name: testing-strategy
description: Test frameworks, patterns, coverage requirements, and quality gates
category: testing
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Testing Strategy

## Framework

- **pytest** — Primary test runner
- **pytest-asyncio** — Async test support (`@pytest.mark.asyncio`)
- **pytest-mock** — Mocking utilities
- **pytest-cov** — Coverage reporting
- **httpx** — Async HTTP/WebSocket client for FastAPI testing
- **factory-boy** — Test data factories

## Running Tests

```bash
pytest tests/ -v                            # All tests
pytest tests/unit/ -v                        # Unit tests only
pytest tests/integration/ -v                 # Integration tests
pytest tests/e2e/ -v                         # End-to-end tests
pytest tests/ --cov=src/argos --cov-report=html  # Coverage with HTML report
pytest tests/ -k "classifier" -v             # Filter by name
pytest tests/ --tb=short                     # Short tracebacks
```

## Test Patterns

### Unit Tests — Agent Nodes
Test individual node functions with mock state and mock LLM:
```python
@pytest.mark.asyncio
async def test_classify_hosting_intent(mock_llm, sample_state):
    sample_state["messages"] = [HumanMessage(content="Meu site está fora do ar")]
    with patch("src.argos.services.llm.get_llm", return_value=mock_llm):
        result = await classify_intent(sample_state)
    assert result["intent"].intent == "hosting"
```

### Integration Tests — Graph Flows
Test compiled subgraphs end-to-end:
```python
@pytest.mark.asyncio
async def test_hosting_agent_full_flow():
    result = await hosting_agent.ainvoke(initial_state)
    assert result["agent_results"]["hosting"] is not None
```

### WebSocket Tests
Use httpx AsyncClient with ASGI transport:
```python
async def test_websocket_chat():
    async with AsyncClient(transport=ASGITransport(app=app)) as client:
        async with client.websocket_connect("/ws/test") as ws:
            await ws.send_json({"message": "test"})
            response = await ws.receive_json()
            assert "content" in response
```

### Mocking Strategy
- **LLM calls**: Always mock in unit tests, use real in integration (optional)
- **External APIs**: Always mock (cPanel, WHM, WHMCS)
- **Database**: Use test database with rollback fixtures
- **Redis**: Use fakeredis or test instance

## Coverage Requirements

| Package | Target | Focus |
|---------|--------|-------|
| `core/` | 90%+ | State validation, graph compilation |
| `agents/classifier/` | 90%+ | Classification accuracy |
| `agents/supervisor/` | 80%+ | Routing correctness |
| `agents/*/` (deep) | 70%+ | Node logic, tool calls |
| `tools/` | 80%+ | API error handling |
| `api/` | 70%+ | WebSocket lifecycle |
| `tasks/` | 60%+ | Background execution |

## Quality Gates

1. All tests pass: `pytest tests/ -v`
2. Coverage meets targets: `pytest --cov=src/argos`
3. No lint errors: `ruff check src/ tests/`
4. Type checking passes: `mypy src/ --strict`
5. No security issues (manual review)

## Test File Structure

```
tests/
├── conftest.py              # Shared fixtures
├── unit/
│   ├── test_classifier.py   # Intent classification tests
│   ├── test_supervisor.py   # Routing logic tests
│   ├── test_state.py        # State schema tests
│   ├── test_tools.py        # API wrapper tests
│   └── test_synthesizer.py  # Synthesizer logic tests
├── integration/
│   ├── test_graph.py        # Full graph execution
│   ├── test_websocket.py    # WebSocket communication
│   └── test_agents.py       # Agent subgraph flows
└── e2e/
    └── test_scenarios.py    # Real-world support scenarios
```

---
*Created for Argos Panoptes project - August 2026*
