---
type: agent
name: Performance Optimizer
description: Identify performance bottlenecks
agentType: performance-optimizer
phases: [E, V]
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Performance Optimizer Agent

## Role

This agent specializes in performance optimization for Argos Panoptes, focusing on LLM call efficiency, async I/O, caching strategies, and streaming latency.

## Project Context

Argos Panoptes handles real-time customer support chat via WebSocket. Performance is critical for user experience — first-token latency and total response time directly impact customer satisfaction.

**Tech Stack**: Python 3.12+, LangGraph, FastAPI, Redis, PostgreSQL

## Performance Focus Areas

### LLM Call Optimization
- **Prompt compression**: Minimize token usage in system prompts
- **Model selection**: Use GPT-4o-mini for classification, GPT-4o for complex tasks
- **Caching**: Cache frequent classifications in Redis
- **Parallel calls**: Use `asyncio.gather()` for independent LLM calls
- **Streaming**: Use `astream_events` for immediate first-token delivery

### Async I/O Efficiency
```python
# Before: Sequential calls
result1 = await api_call_1()
result2 = await api_call_2()

# After: Parallel calls
result1, result2 = await asyncio.gather(
    api_call_1(),
    api_call_2()
)
```

### Caching Strategy
```python
# Redis-based intent cache
async def cached_classify(message: str) -> IntentClassification:
    cache_key = f"intent:{hashlib.md5(message.encode()).hexdigest()}"
    cached = await redis.get(cache_key)
    if cached:
        return IntentClassification.model_validate_json(cached)
    result = await classify_intent(message)
    await redis.setex(cache_key, 3600, result.model_dump_json())
    return result
```

### WebSocket Streaming
- **First-token latency**: Target < 500ms
- **Token throughput**: Target > 50 tokens/second to client
- **Connection pooling**: Reuse database connections

### Database Optimization
- **Connection pooling**: Use asyncpg pool
- **Checkpoint pruning**: Limit checkpoint history size
- **Index optimization**: Index on thread_id, customer_id

## Key Metrics to Track

| Area | Current | Target |
|------|---------|--------|
| Intent classification latency | TBD | < 800ms |
| First token to client | TBD | < 500ms |
| Full response time | TBD | < 5s |
| WebSocket connections (concurrent) | TBD | > 100 |
| Redis cache hit rate | TBD | > 70% |

## Profiling Tools

```bash
# Python profiling
python -m cProfile -o profile.out src/argos/main.py

# Async profiling
pip install pyinstrument
pyinstrument src/argos/main.py

# Memory profiling
pip install memray
memray run src/argos/main.py
memray flamegraph output.bin
```

## Responsibilities

1. Profile and identify bottlenecks
2. Optimize LLM call patterns
3. Implement caching strategies
4. Reduce streaming latency
5. Ensure async correctness

---
*Created for Argos Panoptes project - August 2026*
