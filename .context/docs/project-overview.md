---
type: doc
name: project-overview
description: High-level overview of the project, its purpose, and key components
category: overview
generated: 2026-08-01
status: filled
scaffoldVersion: "2.0.0"
---

# Project Overview

## Summary

Argos Panoptes ("the all-seeing") is a multi-agent AI system for web hosting customer support, built with LangGraph and Python. Named after the mythological giant with a hundred eyes, it provides omniscient visibility into customer needs across multiple hosting domains: shared hosting, VPS, dedicated servers, email, DNS, site creation, and billing.

The system uses a hierarchical supervisor architecture where an Intent Classifier routes customer messages to specialized Deep Agent subgraphs, each with domain-specific tools and knowledge. Communication happens via WebSocket for real-time chat, and background async routines handle monitoring and automation tasks.

## Architecture

```
argos-panoptes/
├── src/
│   └── argos/
│       ├── main.py                # Application entry point
│       ├── config.py              # Pydantic Settings configuration
│       ├── core/                  # State schemas, main graph, registry
│       │   ├── state.py           # TypedDict + Pydantic state models
│       │   ├── graph.py           # Main StateGraph assembly
│       │   └── registry.py        # Dynamic agent registry
│       ├── agents/                # All agent implementations
│       │   ├── classifier/        # Intent classification (structured output)
│       │   ├── supervisor/        # Orchestrator (Command-based routing)
│       │   ├── synthesizer/       # Multi-agent result merger (Gen-Critic-Revise)
│       │   ├── hosting/           # Shared hosting deep agent
│       │   ├── vps/               # VPS deep agent
│       │   ├── dedicated/         # Dedicated server deep agent
│       │   ├── email/             # Email management deep agent
│       │   ├── dns/               # DNS/SSL deep agent
│       │   ├── site_builder/      # Site creation deep agent (Extendify)
│       │   └── billing/           # Billing deep agent (WHMCS)
│       ├── tools/                 # External API wrappers
│       │   ├── cpanel.py          # cPanel UAPI client
│       │   ├── whm.py             # WHM API client
│       │   ├── whmcs.py           # WHMCS API client
│       │   └── extendify.py       # Extendify integration
│       ├── api/                   # FastAPI app layer
│       │   ├── app.py             # FastAPI application setup
│       │   ├── websocket.py       # WebSocket handler with streaming
│       │   ├── rest.py            # REST health/admin endpoints
│       │   └── middleware.py      # Auth, CORS, rate limiting
│       ├── services/              # Infrastructure services
│       │   ├── llm.py             # LLM provider abstraction
│       │   ├── database.py        # PostgreSQL + asyncpg
│       │   └── cache.py           # Redis caching
│       ├── tasks/                 # Background async routines
│       │   ├── scheduler.py       # APScheduler setup
│       │   ├── health_monitor.py  # Server health checks
│       │   ├── ssl_checker.py     # SSL expiration monitoring
│       │   └── usage_alerts.py    # Resource usage alerts
│       ├── models/                # Pydantic data models
│       └── utils/                 # Shared utilities
├── tests/                         # Test suite
│   ├── unit/                      # Unit tests
│   ├── integration/               # Integration tests
│   └── e2e/                       # End-to-end scenarios
├── docs/                          # Detailed documentation
├── .context/                      # AI-consumable context
│   ├── agents/                    # Agent playbooks
│   └── docs/                      # Project knowledge base
├── docker-compose.yml
├── pyproject.toml
└── AGENTS.md
```

## Key Components

- **Intent Classifier** (`agents/classifier/`): Analyzes customer messages using structured LLM output (Pydantic schema) to determine intent category and confidence. Routes to Level 1 (simple) or Supervisor (complex).

- **Supervisor** (`agents/supervisor/`): Central orchestrator that delegates to specialized Deep Agents using the LangGraph `Command` primitive. Can request multi-agent collaboration via the Synthesizer.

- **Deep Agents** (`agents/{hosting,vps,email,...}/`): Independent subgraphs compiled and added as nodes to the parent graph. Each has domain-specific tools, prompts, and multi-step logic.

- **Synthesizer** (`agents/synthesizer/`): Handles cases where multiple agents need to collaborate (e.g., creating a site requires DNS + Hosting). Uses Generator-Critic-Revise loop.

- **WebSocket Gateway** (`api/websocket.py`): Real-time bidirectional communication with streaming support via `astream_events`.

- **Background Tasks** (`tasks/`): Async routines for health monitoring, SSL checks, usage alerts, powered by APScheduler.

## Core Data Flow

1. Client connects via WebSocket (`/ws/{thread_id}`)
2. Message received → Intent Classifier analyzes intent
3. Simple intents → Level 1 Agent responds directly
4. Complex intents → Supervisor delegates to Deep Agent subgraph
5. Multi-domain → Synthesizer merges results from multiple agents
6. Response streamed back via WebSocket (`astream_events`)
7. State checkpointed to PostgreSQL for conversation persistence

## Design Decisions

- **Hybrid agent approach**: Simple queries use single-node agents (low latency, low cost). Complex queries use full subgraph agents (deep domain expertise).
- **Self-hosted over LangGraph Platform**: For deep learning of internals.
- **WebSocket over SSE**: Enables bidirectional communication (server can push alerts from background tasks).
- **PostgresSaver for checkpointing**: Production-grade persistence, never InMemorySaver.

---
*Created for Argos Panoptes project - August 2026*
