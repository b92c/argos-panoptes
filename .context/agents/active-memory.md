# Active Memory - Argos Panoptes Session

**Data**: 2026-08-01
**Status**: Fase 0 — Planejamento e Estruturação

---

## Estado Atual

### Fase 0 - Planejamento
- Estrutura de diretórios definida em `docs/implementation-plan.md`
- Especificação técnica completa em `docs/technical-specification.md`
- Roadmap de aprendizado em `docs/roadmap.md`
- Estrutura `.context/` criada com agents e docs
- Decisões arquiteturais documentadas:
  - Abordagem híbrida: Agent L1 (ordinário) + Deep Agents (subgraphs)
  - WebSocket para comunicação bidirecional
  - Self-hosted (FastAPI + PostgresSaver) para aprendizado
  - Supervisor pattern como orquestrador principal

---

## Próximos Passos

1. Setup do projeto (pyproject.toml, docker-compose, .env)
2. Implementar State Schema com Pydantic/TypedDict
3. Implementar Intent Classifier com structured output
4. Implementar Supervisor Node
5. Primeiro Deep Agent: HostingAgent

---

## Notas Técnicas Importantes
- O estado usa `Annotated[list[BaseMessage], add_messages]` para histórico de mensagens
- Cada Deep Agent é um subgraph compilado independente
- O Synthesizer usa padrão Generator-Critic-Revise
- Background tasks usam APScheduler com AsyncIOScheduler
- Checkpointing com PostgresSaver (nunca InMemorySaver em produção)

---

**Encerrado em**: 2026-08-01
**Retomar por**: Iniciar Fase 1 (Setup do ambiente e Hello World LangGraph)
