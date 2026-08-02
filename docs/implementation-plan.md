# Plano de Implementação — Argos Panoptes

## Visão Geral
Este documento detalha o plano de implementação em fases para o **Argos Panoptes**, um sistema multiagente focado no suporte ao cliente de hospedagem web. O sistema utiliza a arquitetura de grafos baseada no **LangGraph**, interface de comunicação assíncrona bidirecional via **FastAPI WebSocket**, e é construído com **Python 3.12+**. O desenvolvimento segue uma abordagem progressiva, iniciando pela infraestrutura fundacional, avançando para a orquestração do grafo centralizada, seguida pela implementação do primeiro agente profundo, integração WebSocket e culminando na complexa orquestração de múltiplos agentes especialistas e tarefas em segundo plano.

---

## Fase 1: Fundação do Projeto

### 1.1 Estrutura do Projeto
A seguinte estrutura de diretórios acomodará a arquitetura limpa e modular do projeto:

```text
argos-panoptes/
├── .github/
│   └── workflows/
│       └── ci.yml
├── docs/
│   ├── roadmap.md
│   ├── technical-specification.md
│   └── implementation-plan.md
├── src/
│   └── argos/
│       ├── __init__.py
│       ├── main.py
│       ├── config.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── app.py
│       │   ├── websocket.py
│       │   ├── rest.py
│       │   └── middleware.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── state.py
│       │   ├── graph.py
│       │   └── registry.py
│       ├── agents/
│       │   ├── __init__.py
│       │   ├── classifier/
│       │   │   ├── __init__.py
│       │   │   ├── node.py
│       │   │   └── prompts.py
│       │   ├── supervisor/
│       │   │   ├── __init__.py
│       │   │   ├── node.py
│       │   │   └── prompts.py
│       │   ├── synthesizer/
│       │   │   ├── __init__.py
│       │   │   ├── node.py
│       │   │   └── prompts.py
│       │   ├── hosting/
│       │   │   ├── __init__.py
│       │   │   ├── graph.py
│       │   │   ├── nodes.py
│       │   │   ├── tools.py
│       │   │   └── prompts.py
│       │   ├── vps/
│       │   │   ├── __init__.py
│       │   │   ├── graph.py
│       │   │   ├── nodes.py
│       │   │   ├── tools.py
│       │   │   └── prompts.py
│       │   ├── dedicated/
│       │   │   ├── __init__.py
│       │   │   ├── graph.py
│       │   │   ├── nodes.py
│       │   │   ├── tools.py
│       │   │   └── prompts.py
│       │   ├── email/
│       │   │   ├── __init__.py
│       │   │   ├── graph.py
│       │   │   ├── nodes.py
│       │   │   ├── tools.py
│       │   │   └── prompts.py
│       │   ├── dns/
│       │   │   ├── __init__.py
│       │   │   ├── graph.py
│       │   │   ├── nodes.py
│       │   │   ├── tools.py
│       │   │   └── prompts.py
│       │   ├── site_builder/
│       │   │   ├── __init__.py
│       │   │   ├── graph.py
│       │   │   ├── nodes.py
│       │   │   ├── tools.py
│       │   │   └── prompts.py
│       │   └── billing/
│       │       ├── __init__.py
│       │       ├── graph.py
│       │       ├── nodes.py
│       │       ├── tools.py
│       │       └── prompts.py
│       ├── tools/
│       │   ├── __init__.py
│       │   ├── cpanel.py
│       │   ├── whm.py
│       │   ├── whmcs.py
│       │   ├── extendify.py
│       │   └── base.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── customer.py
│       │   ├── ticket.py
│       │   ├── server.py
│       │   └── conversation.py
│       ├── services/
│       │   ├── __init__.py
│       │   ├── llm.py
│       │   ├── database.py
│       │   ├── cache.py
│       │   └── knowledge_base.py
│       ├── tasks/
│       │   ├── __init__.py
│       │   ├── scheduler.py
│       │   ├── health_monitor.py
│       │   ├── ssl_checker.py
│       │   ├── usage_alerts.py
│       │   └── backup_verifier.py
│       └── utils/
│           ├── __init__.py
│           ├── logging.py
│           ├── errors.py
│           └── validators.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   │   ├── test_classifier.py
│   │   ├── test_supervisor.py
│   │   ├── test_state.py
│   │   └── test_tools.py
│   ├── integration/
│   │   ├── test_graph.py
│   │   ├── test_websocket.py
│   │   └── test_agents.py
│   └── e2e/
│       └── test_scenarios.py
├── scripts/
│   ├── seed_db.py
│   └── test_ws_client.py
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
├── .env.example
├── .gitignore
├── langgraph.json
└── README.md
```

### 1.2 Configuração do Ambiente

#### `pyproject.toml`
```toml
[tool.poetry]
name = "argos-panoptes"
version = "0.1.0"
description = "Multi-agent system for web hosting customer support"
authors = ["Argos Team <team@argos.local>"]
readme = "README.md"
packages = [{include = "argos", from = "src"}]

[tool.poetry.dependencies]
python = "^3.12"
fastapi = "^0.110.0"
uvicorn = {extras = ["standard"], version = "^0.29.0"}
langchain = "^0.1.13"
langchain-core = "^0.1.33"
langchain-openai = "^0.1.1"
langgraph = "^0.0.30"
pydantic = "^2.6.4"
pydantic-settings = "^2.2.1"
asyncpg = "^0.29.0"
sqlalchemy = "^2.0.29"
redis = {extras = ["hiredis"], version = "^5.0.3"}
websockets = "^12.0"
httpx = "^0.27.0"
apscheduler = "^3.10.4"

[tool.poetry.group.dev.dependencies]
pytest = "^8.1.1"
pytest-asyncio = "^0.23.6"
black = "^24.3.0"
isort = "^5.13.2"
flake8 = "^7.0.0"
mypy = "^1.9.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

#### `.env.example`
```env
# Application
PROJECT_NAME="Argos Panoptes"
DEBUG=True
ENVIRONMENT=development
API_HOST=0.0.0.0
API_PORT=8000

# LLM / LangChain
OPENAI_API_KEY=sk-proj-...
LANGCHAIN_TRACING_V2=true
LANGCHAIN_ENDPOINT="https://api.smith.langchain.com"
LANGCHAIN_API_KEY=lsv2_...
LANGCHAIN_PROJECT="argos-panoptes"

# Database
POSTGRES_USER=argos
POSTGRES_PASSWORD=argos_pass
POSTGRES_DB=argos_db
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# Redis
REDIS_URL=redis://localhost:6379/0

# External APIs (cPanel, WHM, WHMCS)
CPANEL_API_TOKEN=...
WHM_API_TOKEN=...
WHMCS_API_IDENTIFIER=...
WHMCS_API_SECRET=...
```

#### `docker-compose.yml`
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: argos
      POSTGRES_PASSWORD: argos_pass
      POSTGRES_DB: argos_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U argos"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  postgres_data:
  redis_data:
```

#### `.gitignore`
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
*.egg-info/
.installed.cfg
*.egg

# Virtual Environment
venv/
.venv/
env/
.env

# IDEs
.idea/
.vscode/
*.swp
*.swo

# LangGraph & DB
langgraph.sqlite
*.db
```

### 1.3 Configuração (`config.py`)
```python
# src/argos/config.py
from typing import Optional
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    # App
    project_name: str = "Argos Panoptes"
    debug: bool = False
    environment: str = "development"
    
    # Model
    openai_api_key: str
    model_name: str = "gpt-4-turbo-preview"
    
    # Observability
    langchain_tracing_v2: bool = True
    langchain_api_key: Optional[str] = None
    langchain_project: str = "argos-panoptes"
    
    # DB
    postgres_user: str
    postgres_password: str
    postgres_db: str
    postgres_host: str = "localhost"
    postgres_port: int = 5432
    
    # Redis
    redis_url: str = "redis://localhost:6379/0"
    
    # External APIs
    cpanel_api_token: Optional[str] = None
    whm_api_token: Optional[str] = None
    whmcs_api_identifier: Optional[str] = None
    whmcs_api_secret: Optional[str] = None

    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

settings = Settings()
```

---

## Fase 2: Core do Grafo

### 2.1 State Schema (`core/state.py`)
```python
# src/argos/core/state.py
import operator
from typing import Annotated, TypedDict, List, Optional, Any, Dict
from langchain_core.messages import BaseMessage
from pydantic import BaseModel, Field

class CustomerContext(BaseModel):
    customer_id: str
    active_services: List[str] = Field(default_factory=list)
    recent_tickets: List[str] = Field(default_factory=list)
    is_vip: bool = False

class AgentState(TypedDict):
    messages: Annotated[List[BaseMessage], operator.add]
    customer_context: Optional[CustomerContext]
    current_intent: Optional[str]
    next_node: str
    # Estado genérico para armazenar artefatos gerados pelos agentes (e.g. domínios consultados)
    artifacts: Dict[str, Any]
```

### 2.2 Intent Classifier (`agents/classifier/node.py`)
```python
# src/argos/agents/classifier/node.py
from typing import Literal
from langchain_core.messages import SystemMessage, HumanMessage
from langchain_openai import ChatOpenAI
from pydantic import BaseModel, Field
from argos.core.state import AgentState
from argos.config import settings

INTENTS = Literal[
    "hosting", "vps", "dedicated", "email", 
    "dns", "site_builder", "billing", "general", "escalate"
]

class IntentClassification(BaseModel):
    intent: INTENTS = Field(description="The primary intent of the user's message.")
    confidence: float = Field(description="Confidence score between 0 and 1.")
    reasoning: str = Field(description="Brief explanation of the classification.")

classifier_prompt = """You are the Argos Panoptes routing agent.
Analyze the user's message and classify it into one of the available intents.
Focus on web hosting contexts. If it's a generic greeting, choose 'general'.
If the user demands a human immediately, choose 'escalate'."""

def classify_intent_node(state: AgentState) -> dict:
    llm = ChatOpenAI(model=settings.model_name, temperature=0)
    structured_llm = llm.with_structured_output(IntentClassification)
    
    messages = [SystemMessage(content=classifier_prompt)] + state["messages"]
    classification = structured_llm.invoke(messages)
    
    return {
        "current_intent": classification.intent,
        # O Supervisor usará o intent para definir o próximo node, mas por padrão 
        # enviamos de volta ao supervisor.
        "next_node": "supervisor" 
    }

def intent_router(state: AgentState) -> str:
    """Função de roteamento condicional (Conditional Edge)."""
    intent = state.get("current_intent")
    
    # Mapeamento do intent para o agente específico
    route_map = {
        "hosting": "hosting_agent",
        "vps": "vps_agent",
        "dedicated": "dedicated_agent",
        "email": "email_agent",
        "dns": "dns_agent",
        "site_builder": "site_builder_agent",
        "billing": "billing_agent",
        "escalate": "human_escalation",
        "general": "synthesizer" # Responde diretamente
    }
    
    return route_map.get(intent, "synthesizer")
```

### 2.3 Supervisor Node (`agents/supervisor/node.py`)
```python
# src/argos/agents/supervisor/node.py
from argos.core.state import AgentState

def supervisor_node(state: AgentState) -> dict:
    """
    O supervisor atua como um nó de passagem que pode realizar 
    validações de segurança, injetar contexto do cliente ou 
    preparar o estado antes de passar para o sub-agente.
    """
    # Exemplo: Se o contexto não existir, poderíamos buscar na base de dados
    if not state.get("customer_context"):
        # Dummy fetch
        pass
        
    return {"next_node": "continue"}
```

### 2.4 Graph Assembly (`core/graph.py`)
```python
# src/argos/core/graph.py
from langgraph.graph import StateGraph, END
from argos.core.state import AgentState
from argos.agents.classifier.node import classify_intent_node, intent_router
from argos.agents.supervisor.node import supervisor_node
from argos.agents.hosting.graph import hosting_subgraph
from argos.agents.synthesizer.node import synthesizer_node

def create_main_graph():
    workflow = StateGraph(AgentState)
    
    # Adicionando os nós principais
    workflow.add_node("classifier", classify_intent_node)
    workflow.add_node("supervisor", supervisor_node)
    workflow.add_node("synthesizer", synthesizer_node)
    
    # Adicionando sub-grafos / agentes
    workflow.add_node("hosting_agent", hosting_subgraph)
    
    # (Outros agentes seriam adicionados aqui...)
    
    # Definindo as arestas (Edges)
    workflow.set_entry_point("classifier")
    
    workflow.add_edge("classifier", "supervisor")
    
    # Do supervisor, nós roteamos com base no intent classificado
    workflow.add_conditional_edges(
        "supervisor",
        intent_router,
        {
            "hosting_agent": "hosting_agent",
            # ... mapeamento de outros agentes ...
            "synthesizer": "synthesizer"
        }
    )
    
    # Todos os agentes retornam ao sintetizador para compilar a resposta final
    workflow.add_edge("hosting_agent", "synthesizer")
    workflow.add_edge("synthesizer", END)
    
    return workflow.compile()

app_graph = create_main_graph()
```

---

## Fase 3: Primeiro Deep Agent (Hosting)

### 3.1 Hosting Subgraph

#### `agents/hosting/tools.py`
```python
# src/argos/agents/hosting/tools.py
from langchain_core.tools import tool

@tool
def check_cpanel_disk_usage(username: str) -> str:
    """Verifica o uso de disco de uma conta cPanel específica."""
    # Dummy implementation
    return f"A conta {username} está usando 85% do espaço em disco (8.5GB/10GB)."

@tool
def clear_cpanel_cache(username: str) -> str:
    """Limpa o cache LiteSpeed/Redis para uma conta de hospedagem compartilhada."""
    return f"Cache limpo com sucesso para {username}."
```

#### `agents/hosting/prompts.py`
```python
# src/argos/agents/hosting/prompts.py
HOSTING_SYSTEM_PROMPT = """Você é o Especialista de Hospedagem Compartilhada do Argos Panoptes.
Sua função é resolver problemas relacionados a cPanel, limites de inode, uso de CPU/RAM (CloudLinux) 
e permissões de arquivos. Utilize as ferramentas disponíveis para investigar a conta do usuário.
Responda sempre com clareza e ofereça o próximo passo de solução."""
```

#### `agents/hosting/nodes.py`
```python
# src/argos/agents/hosting/nodes.py
from langchain_core.messages import SystemMessage
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent
from argos.config import settings
from argos.agents.hosting.tools import check_cpanel_disk_usage, clear_cpanel_cache
from argos.agents.hosting.prompts import HOSTING_SYSTEM_PROMPT

def create_hosting_agent():
    llm = ChatOpenAI(model=settings.model_name, temperature=0.2)
    tools = [check_cpanel_disk_usage, clear_cpanel_cache]
    
    # O sub-agente usa o padrão ReAct
    agent = create_react_agent(
        llm, 
        tools, 
        state_modifier=HOSTING_SYSTEM_PROMPT
    )
    return agent
```

#### `agents/hosting/graph.py`
```python
# src/argos/agents/hosting/graph.py
from argos.core.state import AgentState
from argos.agents.hosting.nodes import create_hosting_agent

hosting_agent_runnable = create_hosting_agent()

def hosting_subgraph(state: AgentState) -> dict:
    """Wrapper para integrar o agente ReAct ao grafo principal."""
    response = hosting_agent_runnable.invoke({"messages": state["messages"]})
    # Atualiza as mensagens com o que o agente gerou
    return {"messages": response["messages"]}
```

---

## Fase 4: WebSocket Integration

### 4.1 FastAPI App & WebSocket

#### `api/app.py`
```python
# src/argos/api/app.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from argos.api.websocket import ws_router
from argos.config import settings

app = FastAPI(title=settings.project_name)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(ws_router)

@app.get("/health")
async def health_check():
    return {"status": "ok"}
```

#### Message Schema & `api/websocket.py`
```python
# src/argos/api/websocket.py
import json
import asyncio
from fastapi import APIRouter, WebSocket, WebSocketDisconnect
from pydantic import BaseModel
from langchain_core.messages import HumanMessage
from argos.core.graph import app_graph

ws_router = APIRouter()

class WSMessage(BaseModel):
    content: str
    customer_id: str

@ws_router.websocket("/ws/chat")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    
    try:
        while True:
            data = await websocket.receive_text()
            payload = WSMessage.parse_raw(data)
            
            state_input = {
                "messages": [HumanMessage(content=payload.content)],
                "customer_context": {"customer_id": payload.customer_id} # Simplificado
            }
            
            # Execução em modo streaming usando eventos do LangGraph
            async for event in app_graph.astream_events(state_input, version="v1"):
                kind = event["event"]
                
                if kind == "on_chat_model_stream":
                    chunk = event["data"]["chunk"].content
                    if chunk:
                        await websocket.send_json({
                            "type": "token",
                            "data": chunk
                        })
                        
                elif kind == "on_tool_start":
                    await websocket.send_json({
                        "type": "tool_start",
                        "data": f"Executando ação: {event['name']}..."
                    })
                    
            await websocket.send_json({"type": "done"})
            
    except WebSocketDisconnect:
        print("Cliente desconectado")
```

---

## Fase 5: Agentes Especializados Adicionais

Cada agente abaixo seguirá o mesmo padrão arquitetural do Agente de Hosting (Sub-grafo ReAct com ferramentas específicas e system prompts especializados).

1. **VPS Agent**: Diagnosticador de servidores virtuais. Ferramentas: checar load average, analisar logs de kernel (OOM killer), reiniciar container/VM (via WHM/KVM API).
2. **Dedicated Server Agent**: Especializado em bare-metal. Ferramentas: checar IPMI, status de RAID por hardware, interfaces de rede físicas.
3. **Email Agent**: Focado em entregabilidade. Ferramentas: verificar bloqueios de IP (RBLs), validar SPF/DKIM/DMARC, checar fila do Exim.
4. **DNS Agent**: Troubleshooting de zonas. Ferramentas: consulta `dig` distribuída (global propagation), checagem de glue records e DNSSEC.
5. **Site Builder Agent**: Suporte a CMS (WordPress) e Construtores Visuais. Ferramentas: gerenciar plugins WP via WP-CLI, checar versões de PHP e erros fatais.
6. **Billing Agent**: Financeiro e assinaturas. Ferramentas: integração WHMCS, listar faturas pendentes, emitir 2ª via, verificar fraudes.

---

## Fase 6: Sintetizador

### 6.1 Synthesizer Node
Quando uma solicitação envolve mais de um domínio (ex: "Meu site caiu e acho que minha fatura venceu"), o **Sintetizador** atua após a execução paralela ou sequencial de vários agentes. Ele é responsável pelo loop: *Generator -> Critic -> Revise*.

```python
# src/argos/agents/synthesizer/node.py
from langchain_core.messages import SystemMessage
from langchain_openai import ChatOpenAI
from argos.core.state import AgentState
from argos.config import settings

def synthesizer_node(state: AgentState) -> dict:
    llm = ChatOpenAI(model=settings.model_name, temperature=0.4)
    
    # O System Prompt instrui o modelo a unificar o histórico
    # das ferramentas e respostas dos agentes intermediários num tom uníssono.
    prompt = """Você é o porta-voz final do suporte Argos.
    Revise as informações coletadas pelos agentes especialistas anteriores.
    Sintetize a resposta para o cliente final. Seja claro, empático e conciso.
    Não mencione que você é uma IA ou um conjunto de agentes."""
    
    messages = [SystemMessage(content=prompt)] + state["messages"]
    
    # O ideal seria implementar um sub-grafo Generator-Critic, mas como MVP
    # fazemos uma síntese direta.
    response = llm.invoke(messages)
    
    return {"messages": [response]}
```

*Nota Avançada*: No padrão colaborativo pleno do Argos, o supervisor criará tarefas em paralelo. O `StateSchema` coletará as saídas em `artifacts`, e o Sintetizador rodará uma validação (Critic) para garantir que todas as perguntas do usuário foram respondidas. Se o Critic apontar falhas, ele re-roteará para o(s) agente(s) faltante(s) antes de responder.

---

## Fase 7: Background Tasks

Para suporte proativo e alertas.

```python
# src/argos/tasks/scheduler.py
import asyncio
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from argos.tasks.ssl_checker import check_expiring_ssls

scheduler = AsyncIOScheduler()

def start_scheduler():
    # Executa a verificação de SSLs que vencem em breve a cada dia à 01:00
    scheduler.add_job(check_expiring_ssls, 'cron', hour=1, minute=0)
    scheduler.start()

# Chamado durante o startup da aplicação FastAPI
```

```python
# src/argos/tasks/ssl_checker.py
async def check_expiring_ssls():
    """Busca certificados vencendo em < 7 dias e gera tickets preventivos."""
    print("[Task] Verificando certificados SSL...")
    # Lógica de integração com banco e criação de alerta no Argos
    # Pode disparar um evento (trigger) diretamente para o grafo 
    # iniciar uma conversa com o Billing/Hosting agent.
```

---

## Fase 8: Observabilidade & Testes

**Testes**:
- Framework: `pytest` + `pytest-asyncio`.
- Integração: Testar cada subgrafo com `create_react_agent` de forma isolada, zombando (mock) as ferramentas.
- E2E: Testar a conexão WebSocket e o streaming de tokens completos.

**Observabilidade**:
- O projeto usa o **LangSmith** ativado via `.env` (`LANGCHAIN_TRACING_V2=true`). Isso garante que cada chamada LLM, invocação de ferramenta, roteamento do Supervisor e tempo de resposta sejam registrados de forma granular (trace-level).
- Para logs de aplicação genéricos (FastAPI, BD, background tasks), utilizaremos o módulo nativo `logging` do Python encapsulado no `utils.logging.py` no formato estruturado (JSON).

---

## Checklist de Implementação

- [ ] **Fase 1: Fundação**
  - [ ] Criar repositório e estrutura de diretórios base.
  - [ ] Inicializar ambiente Poetry com dependências.
  - [ ] Configurar Docker e base de dados.
  - [ ] Implementar `config.py` e carregar variáveis.
- [ ] **Fase 2: Core do Grafo**
  - [ ] Definir o schema de estado (`state.py`).
  - [ ] Implementar e testar prompt de Classificação.
  - [ ] Implementar Supervisor e roteador condicional.
  - [ ] Integrar todos no `graph.py` principal.
- [ ] **Fase 3: Agente de Hosting (MVP)**
  - [ ] Implementar dummy tools (cPanel/WHM).
  - [ ] Criar sub-grafo de Hosting usando `create_react_agent`.
  - [ ] Conectar Agente de Hosting ao grafo principal.
- [ ] **Fase 4: WebSocket API**
  - [ ] Criar servidor FastAPI.
  - [ ] Implementar rota `/ws/chat`.
  - [ ] Testar streaming (async iterator) do LangGraph.
- [ ] **Fase 5: Especialistas (Expansão)**
  - [ ] Agente VPS.
  - [ ] Agente Dedicated.
  - [ ] Agente Email.
  - [ ] Agente DNS.
  - [ ] Agente Billing.
  - [ ] Agente Site Builder.
- [ ] **Fase 6 & 7: Avançado**
  - [ ] Aprimorar Sintetizador com Critic Loop.
  - [ ] Implementar APScheduler no startup da API.
  - [ ] Criar task de monitoramento SSL.
- [ ] **Fase 8: Refinamento**
  - [ ] Garantir 100% de traces visíveis no LangSmith.
  - [ ] Alcançar 80%+ de code coverage.
  - [ ] Escrever README final de execução.
