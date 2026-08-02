# 🏛️ Argos Panoptes

> *O gigante de cem olhos que tudo vê* — Sistema multi-agentes para atendimento ao cliente de hospedagem de sites.

## Sobre

**Argos Panoptes** é um sistema multi-agentes construído com **LangGraph** e **Python** para atendimento ao cliente de uma empresa de hospedagem de sites. O sistema utiliza uma arquitetura hierárquica de supervisão onde agentes especializados (Deep Agents) são orquestrados para resolver demandas em múltiplos domínios.

### Domínios Atendidos

| Domínio | Agente | Descrição |
|:--------|:-------|:----------|
| 🖥️ Hospedagem Compartilhada | `HostingAgent` | Gestão de contas cPanel, sites, recursos |
| ☁️ VPS | `VPSAgent` | Provisionamento e gestão de VPS |
| 🏢 Servidores Dedicados | `DedicatedAgent` | Administração de servidores dedicados |
| 📧 E-mail | `EmailAgent` | Criação, configuração e troubleshooting de e-mails |
| 🌐 DNS/SSL | `DNSAgent` | Gestão de DNS, certificados SSL, registros |
| 🎨 Criação de Sites | `SiteBuilderAgent` | Criação de sites via Extendify/WordPress |
| 💰 Faturamento | `BillingAgent` | Faturas, pagamentos, upgrades (WHMCS) |

## Arquitetura

```
Cliente → WebSocket → Intent Classifier → Supervisor → Deep Agent (Subgraph) → Resposta
                                              ↓ (multi-domínio)
                                         Synthesizer
```

- **Intent Classifier**: Classifica a intenção do cliente com structured output (Pydantic)
- **Supervisor**: Orquestra os Deep Agents via `Command` primitive do LangGraph
- **Deep Agents**: Subgraphs independentes com tools especializados
- **Synthesizer**: Merge de resultados quando múltiplos agentes colaboram

## Tech Stack

| Tecnologia | Propósito |
|:-----------|:----------|
| Python 3.12+ | Linguagem principal |
| LangGraph >= 0.3 | Framework de agentes (StateGraph, subgraphs, Command) |
| FastAPI | WebSocket + REST API |
| Pydantic v2 | Validação de estado e dados |
| PostgreSQL 16 | Checkpointing e persistência |
| Redis 7 | Cache e sessões |
| LangSmith | Observabilidade e tracing |
| structlog | Logging estruturado |

## Quick Start

```bash
# Clone
git clone https://github.com/b92c/argos-panoptes.git
cd argos-panoptes

# Setup
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

# Infraestrutura
docker compose up -d  # PostgreSQL + Redis

# Configuração
cp .env.example .env  # Edite com suas API keys

# Executar
uvicorn src.argos.api.app:app --reload --port 8000
```

## Documentação

| Documento | Descrição |
|:----------|:----------|
| [Especificação Técnica](docs/technical-specification.md) | Arquitetura completa, schemas, padrões |
| [Roadmap](docs/roadmap.md) | Trilha de aprendizado e cronograma |
| [Plano de Implementação](docs/implementation-plan.md) | Guia fase por fase com código |

## AI Context

Este projeto utiliza uma estrutura de contexto para agentes de IA:

- **`AGENTS.md`** — Ponto de entrada para agentes de IA
- **`.context/agents/`** — Playbooks especializados por tipo de tarefa
- **`.context/docs/`** — Documentação consumível por agentes

## Dojo

Este projeto é um **dojo pessoal** para atingir nível **Senior/Staff** em:
- Python avançado (async/await, typing, protocols)
- LangGraph (multi-agent, subgraphs, streaming)
- Arquitetura de Sistemas Agênticos
- Engenharia de Contexto e Produção

## Licença

MIT
