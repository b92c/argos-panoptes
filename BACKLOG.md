# 👁️ Argos Panoptes — Backlog de Implementação Completo e Detalhado

> **Última atualização:** Agosto 2026
>
> Este documento é o **desdobramento operacional** e pragmático do [ROADMAP.md](./roadmap.md).
> Cada tarefa é um card executável com título, descrição, critérios de aceite e checklist.
> Marque os checkboxes conforme for completando cada item.

---

## Como usar este documento

1. **Siga a ordem numérica** — as tarefas estão sequenciadas por dependência de infraestrutura e lógica.
2. **Marque o checkbox `[x]`** quando concluir cada etapa dentro da tarefa.
3. **Marque a tarefa inteira** quando todos os sub-itens estiverem concluídos.
4. Tarefas com 🔗 indicam **dependência explícita** de outra tarefa.

### Legenda de Labels

| Label | Significado |
|-------|-------------|
| `INFRA` | Estrutura de diretórios, dependências e containers |
| `GRAPH` | Definição de estados, nós e transições do LangGraph |
| `AGENT` | Implementação de agentes especialistas ou subgrafos |
| `TOOL` | Integrações com APIs externas (cPanel, WHM, WHMCS, Extendify) |
| `API` | Servidor FastAPI, WebSocket e endpoints REST |
| `TASK` | Rotinas assíncronas em segundo plano (Scheduler/APScheduler) |
| `TEST` | Testes de unidade, integração e E2E |
| `OBS` | LangSmith, tracing, log estruturado e métricas |
| `DB` | Migrations, persistência SQLAlchemy, esquemas Redis |

---

## 📦 FASE 0 — Estrutura Base & Tooling

---

### - [x] Tarefa 001 — Criar estrutura de diretórios do projeto
**Label:** `INFRA`
**Descrição:** Criar toda a árvore de diretórios do projeto seguindo a arquitetura limpa proposta para a aplicação Python com FastAPI e LangGraph.

**Checklist:**
- [x] Criar pasta `src/argos/`
- [x] Criar subpastas em `src/argos/`: `api/`, `core/`, `agents/`, `tools/`, `models/`, `services/`, `tasks/`, `utils/`
- [x] Criar pastas para cada agente em `src/argos/agents/`: `classifier/`, `supervisor/`, `synthesizer/`, `hosting/`, `vps/`, `dedicated/`, `email/`, `dns/`, `site_builder/`, `billing/`
- [x] Adicionar arquivos `__init__.py` vazios em todas as pastas e subpastas para marcar os pacotes Python
- [x] Criar estrutura do diretório `tests/` com subpastas `unit/`, `integration/` e `e2e/`
- [x] Criar pasta `docs/` e garantir que `roadmap.md`, `technical-specification.md` e `implementation-plan.md` estejam organizados lá

---

### - [x] Tarefa 002 — Configurar dependências e empacotamento com Poetry
**Label:** `INFRA`
**Dependência:** 🔗 Tarefa 001
**Descrição:** Configurar o arquivo `pyproject.toml` usando o Poetry para gerenciar as dependências do projeto, linter/formatter (Ruff) e gerador de tipos (Mypy).

**Checklist:**
- [x] Criar `pyproject.toml` na raiz do projeto com Poetry
- [x] Configurar dependências de runtime: `fastapi`, `uvicorn`, `langgraph`, `langchain`, `langchain-openai`, `pydantic`, `pydantic-settings`, `asyncpg`, `sqlalchemy`, `redis`, `apscheduler`, `structlog`
- [x] Configurar dependências de desenvolvimento: `pytest`, `pytest-asyncio`, `pytest-mock`, `pytest-cov`, `ruff`, `mypy`, `httpx`
- [x] Executar `poetry install` e criar o virtual environment (`.venv`)
- [x] Validar que Ruff e Mypy estão instalados no virtualenv executando `ruff --version` e `mypy --version`

---

### - [x] Tarefa 003 — Configurar ferramentas de qualidade de código (Ruff, Mypy)
**Label:** `INFRA`
**Dependência:** 🔗 Tarefa 002
**Descrição:** Adicionar as configurações estritas no `pyproject.toml` para o Ruff (linter e formatter) e Mypy (type checker) para garantir qualidade no nível Staff.

**Checklist:**
- [x] Configurar Ruff no `pyproject.toml` com regras estritas: `E`, `F`, `I`, `N`, `UP`, `B`, `A`, `C4`, `T20`, `RET`, `SIM`, `ARG`, `PTH`, `ERA`, `PL`, `RUF`
- [x] Configurar Mypy em modo estrito (`strict = true`, `disallow_untyped_defs = true`, `warn_unused_ignores = true`)
- [x] Criar arquivo `.vscode/settings.json` com configurações de format on save utilizando o Ruff
- [x] Rodar `poetry run ruff check .` e `poetry run mypy src/` e verificar que executam sem erros

---

### - [ ] Tarefa 004 — Criar docker-compose.yml com PostgreSQL e Redis
**Label:** `INFRA`
**Dependência:** 🔗 Tarefa 001
**Descrição:** Configurar o docker-compose.yml para subir os serviços locais de banco de dados (PostgreSQL) e cache/session store (Redis).

**Checklist:**
- [ ] Criar `docker-compose.yml` com instâncias do PostgreSQL 16 (porta 5432) e Redis 7 (porta 6379)
- [ ] Configurar volumes nomeados para persistência: `postgres_data` e `redis_data`
- [ ] Adicionar healthchecks apropriados no PostgreSQL e Redis para garantir que estejam prontos antes da inicialização do app
- [ ] Testar a subida com `docker compose up -d` e validar status com `docker compose ps`

---

### - [ ] Tarefa 005 — Configurar variáveis de ambiente e Settings
**Label:** `INFRA`
**Dependência:** 🔗 Tarefa 003
**Descrição:** Criar o arquivo de configurações base utilizando `pydantic-settings` para carregamento seguro do `.env`.

**Checklist:**
- [ ] Criar `.env.example` com todas as chaves necessárias (OpenAI, LangSmith, DB, Redis, APIs externas)
- [ ] Criar o arquivo de configurações base `src/argos/config.py` usando `pydantic-settings`
- [ ] Definir a classe `Settings` com tipos corretos e validação (e.g. `postgres_port` como int)
- [ ] Criar o arquivo `.env` local para desenvolvimento baseado no `.env.example`

---

### - [ ] Tarefa 006 — Configurar conexões assíncronas do banco de dados (SQLAlchemy 2.0)
**Label:** `DB`
**Dependência:** 🔗 Tarefa 005
**Descrição:** Implementar a inicialização assíncrona do banco de dados utilizando SQLAlchemy 2.0 com `asyncpg`.

**Checklist:**
- [ ] Criar `src/argos/services/database.py`
- [ ] Implementar o `AsyncEngine` utilizando a URL do banco carregada do `settings`
- [ ] Criar `async_sessionmaker` para fornecer sessões do banco
- [ ] Definir a classe base declarativa `Base` para modelos ORM
- [ ] Implementar dependência assíncrona `get_db_session` para injeção de dependências
- [ ] Escrever teste de unidade validando conexão e ping assíncrono com o banco de dados

---

### - [ ] Tarefa 007 — Configurar conexão assíncrona do Redis
**Label:** `DB`
**Dependência:** 🔗 Tarefa 005
**Descrição:** Implementar a inicialização do cliente Redis assíncrono para cache e controle de sessões.

**Checklist:**
- [ ] Criar `src/argos/services/cache.py`
- [ ] Instanciar o cliente `redis.asyncio.Redis` usando a URL do redis no `settings`
- [ ] Implementar funções helper: `get_cache`, `set_cache` com suporte a expiração (TTL) e serialização JSON
- [ ] Escrever teste de unidade validando operações de get/set assíncronas no Redis

---

## 🧠 FASE 1 — Core do Grafo, State & Roteamento

---

### - [ ] Tarefa 008 — Implementar esquemas de estado (State Schemas)
**Label:** `GRAPH`
**Dependência:** 🔗 Tarefa 005
**Descrição:** Definir as estruturas de dados compartilhadas do grafo usando `TypedDict` do Python e classes Pydantic. Adicionar o reducer `add_messages` para o histórico de conversas.

**Checklist:**
- [ ] Criar `src/argos/core/state.py`
- [ ] Implementar classe `CustomerContext` herdando de `BaseModel` (contendo dados de clientes e serviços ativos)
- [ ] Implementar classe `IntentClassification` herdando de `BaseModel` (contendo intent, confidence e reasoning)
- [ ] Criar o tipo `AgentState` usando `TypedDict`
- [ ] Utilizar `Annotated[List[BaseMessage], add_messages]` no campo `messages` para assegurar acumulação correta de histórico
- [ ] Escrever teste de unidade garantindo o comportamento acumulador das mensagens no estado

---

### - [ ] Tarefa 009 — Implementar nó classificador de intenção (Intent Classifier)
**Label:** `GRAPH`, `AGENT`
**Dependência:** 🔗 Tarefa 008
**Descrição:** Criar o nó classificador de intenção. O classificador utiliza chamada de modelo estruturada (structured output via Pydantic) para discernir a intenção do cliente de forma determinística antes de rotear para o supervisor.

**Checklist:**
- [ ] Criar `src/argos/agents/classifier/prompts.py` com o prompt do sistema para classificação (focado no domínio de hospedagem)
- [ ] Criar `src/argos/agents/classifier/node.py` contendo a função assíncrona `classify_intent_node`
- [ ] Configurar chamada LLM usando `.with_structured_output(IntentClassification)` para forçar saída tipada
- [ ] Tratar exceções de falha de parsing do LLM retornando uma classificação segura de fallback ("general") com confiança baixa
- [ ] Escrever testes de unidade injetando diferentes inputs textuais e assegurando a classificação correta do intent em `tests/unit/test_classifier.py`

---

### - [ ] Tarefa 010 — Implementar roteador condicional por intenção
**Label:** `GRAPH`
**Dependência:** 🔗 Tarefa 009
**Descrição:** Implementar a função que serve como conditional edge no LangGraph para direcionar o fluxo baseado na intenção.

**Checklist:**
- [ ] Criar a função `intent_router` em `src/argos/agents/classifier/node.py`
- [ ] Mapear as intenções para os respectivos nós dos agentes especialista
- [ ] Mapear intenção "general" para o nó "synthesizer"
- [ ] Mapear intenção "escalate" para o nó "human_escalation"
- [ ] Mapear casos inválidos para um fallback seguro
- [ ] Testar a função de roteamento com mocks de estado de agente

---

### - [ ] Tarefa 011 — Implementar nó Supervisor
**Label:** `GRAPH`, `AGENT`
**Dependência:** 🔗 Tarefa 008
**Descrição:** Implementar o supervisor que atua como o ponto de decisão e orquestração do grafo principal de nível 2.

**Checklist:**
- [ ] Criar `src/argos/agents/supervisor/prompts.py` com as instruções para orquestração
- [ ] Criar `src/argos/agents/supervisor/node.py` com a função assíncrona `supervisor_node`
- [ ] Garantir que o supervisor injete o contexto do usuário (como dados do WHMCS ou cPanel) caso ele ainda não esteja preenchido no estado
- [ ] Escrever testes de unidade para validar a injeção correta de dados no estado pelo supervisor

---

### - [ ] Tarefa 012 — Configurar persistência com PostgresSaver (Checkpointer)
**Label:** `DB`, `GRAPH`
**Dependência:** 🔗 Tarefa 006, 🔗 Tarefa 008
**Descrição:** Configurar o PostgresSaver para persistir o histórico das threads do grafo no banco PostgreSQL de forma resiliente.

**Checklist:**
- [ ] Criar `src/argos/core/persistence.py`
- [ ] Implementar a conexão e o setup de tabelas do PostgresSaver
- [ ] Exportar a factory `get_postgres_checkpointer`
- [ ] Escrever teste de integração de gravação e recuperação de estado do grafo usando uma thread de teste

---

### - [ ] Tarefa 013 — Montar e Compilar o Grafo Principal
**Label:** `GRAPH`
**Dependência:** 🔗 Tarefa 010, 🔗 Tarefa 011, 🔗 Tarefa 012
**Descrição:** Montar e compilar as peças individuais no arquivo de montagem do grafo central.

**Checklist:**
- [ ] Criar `src/argos/core/graph.py`
- [ ] Instanciar `StateGraph(AgentState)` no construtor
- [ ] Adicionar os nós criados: `classifier`, `supervisor` e placeholders temporários para os subgrafos
- [ ] Conectar os nós com arestas diretas (`add_edge`) e arestas condicionais (`add_conditional_edges`) usando `intent_router`
- [ ] Compilar o grafo passando o checkpointer do PostgresSaver
- [ ] Exportar a instância compilada `app_graph`
- [ ] Escrever teste de integração simples rodando o grafo compilado e validando o percurso entre nós

---

## 🖥️ FASE 2 — Primeiro Agente Profundo (Hosting) & API WebSocket

---

### - [ ] Tarefa 014 — Criar ferramentas (Tools) da API do cPanel
**Label:** `TOOL`
**Dependência:** 🔗 Tarefa 005
**Descrição:** Desenvolver wrappers HTTP assíncronos e a definição de ferramenta do Langchain para consultas à API cPanel.

**Checklist:**
- [ ] Criar `src/argos/tools/cpanel.py` com wrappers HTTP assíncronos usando `httpx`
- [ ] Implementar a ferramenta `@tool` `check_cpanel_disk_usage`
- [ ] Implementar a ferramenta `@tool` `clear_cpanel_cache`
- [ ] Adicionar lógica de fallback robusta nas ferramentas para carregar dados mockados caso `CPANEL_API_TOKEN` esteja vazio
- [ ] Escrever testes de unidade isolando as chamadas HTTP (usando `pytest-mock` ou `respx`)

---

### - [ ] Tarefa 015 — Desenvolver Subgrafo do Hosting Agent (ReAct)
**Label:** `AGENT`
**Dependência:** 🔗 Tarefa 014, 🔗 Tarefa 013
**Descrição:** Criar o subgrafo independente `hosting_subgraph` usando o padrão ReAct com ferramentas específicas de cPanel.

**Checklist:**
- [ ] Criar `src/argos/agents/hosting/prompts.py` com o prompt de especialista em hospedagem (`HOSTING_SYSTEM_PROMPT`)
- [ ] Criar `src/argos/agents/hosting/nodes.py` implementando a criação do ReAct agent via `create_react_agent`
- [ ] Criar `src/argos/agents/hosting/graph.py` definindo a função `hosting_subgraph` que encapsula o executável do ReAct agent e traduz o estado para o grafo pai
- [ ] Registrar o nó `hosting_agent` no grafo pai em `src/argos/core/graph.py` e apontar o roteamento condicional para ele
- [ ] Escrever teste de integração chamando o subgrafo diretamente com um mock de estado e checando a execução das ferramentas

---

### - [ ] Tarefa 016 — Configurar FastAPI e middlewares base
**Label:** `API`
**Dependência:** 🔗 Tarefa 005
**Descrição:** Inicializar a aplicação FastAPI com middlewares obrigatórios de segurança, CORS e tratamento global de exceções.

**Checklist:**
- [ ] Criar `src/argos/api/app.py` configurando a instância do FastAPI
- [ ] Adicionar middleware `CORSMiddleware` limitando origens seguras
- [ ] Adicionar middleware customizado para logging de requisições REST
- [ ] Adicionar tratamento global de exceções retornando respostas estruturadas no padrão RFC 7807 (Problem Details)
- [ ] Criar rota `/health` com verificação de status do banco de dados e Redis

---

### - [ ] Tarefa 017 — Implementar rota de WebSocket para Chat
**Label:** `API`
**Dependência:** 🔗 Tarefa 016, 🔗 Tarefa 013
**Descrição:** Implementar a rota de WebSocket para conversação em tempo real.

**Checklist:**
- [ ] Criar `src/argos/api/websocket.py`
- [ ] Definir schemas de mensagens WebSocket (JSON) usando Pydantic
- [ ] Implementar gerenciador de conexões WebSocket com suporte a desconexões limpas
- [ ] Configurar endpoint `/ws/chat` aceitando conexões de clientes
- [ ] Criar teste de unidade para testar o handshake da rota `/ws/chat`

---

### - [ ] Tarefa 018 — Implementar Streaming via WebSocket usando astream_events
**Label:** `API`, `GRAPH`
**Dependência:** 🔗 Tarefa 017
**Descrição:** Integrar a execução do LangGraph no loop de eventos do WebSocket, enviando tokens de forma incremental para o cliente.

**Checklist:**
- [ ] Capturar a mensagem do usuário recebida via WebSocket e passá-la para `app_graph.astream_events`
- [ ] Identificar eventos `on_chat_model_stream` e enviar tokens individuais de resposta em formato JSON para o cliente
- [ ] Identificar eventos de início de execução de ferramentas (`on_tool_start`) e notificar o usuário com um status de carregamento
- [ ] Garantir o encerramento seguro e o envio de payload de conclusão (`done`)
- [ ] Desenvolver o script de teste `scripts/test_ws_client.py` e validar o streaming completo via terminal

---

## ☁️ FASE 3 — Expansão dos Agentes Especializados (Deep Agents)

---

### - [ ] Tarefa 019 — Criar ferramentas da API do WHM
**Label:** `TOOL`
**Dependência:** 🔗 Tarefa 005
**Descrição:** Desenvolver wrappers HTTP assíncronos para a API do WHM focados na administração de servidores.

**Checklist:**
- [ ] Criar `src/argos/tools/whm.py`
- [ ] Implementar a ferramenta `@tool` `check_vps_resources` (load avg, RAM, disco)
- [ ] Implementar a ferramenta `@tool` `restart_vps_service` (HTTPd, MySQL, Exim)
- [ ] Configurar mocks com dados estáticos quando chaves API não fornecidas no `.env`
- [ ] Escrever testes unitários para a API do WHM

---

### - [ ] Tarefa 020 — Implementar VPS Agent
**Label:** `AGENT`
**Dependência:** 🔗 Tarefa 019, 🔗 Tarefa 013
**Descrição:** Criar o subgrafo do VPS Agent e integrá-lo no fluxo principal.

**Checklist:**
- [ ] Criar pasta `src/argos/agents/vps/`
- [ ] Criar prompt do VPS Agent com instruções detalhadas sobre Linux, administração de serviços e load average
- [ ] Implementar o subgrafo `vps_subgraph` acoplando as ferramentas do WHM
- [ ] Registrar o nó `vps_agent` no grafo principal (`graph.py`)
- [ ] Atualizar o `intent_router` e o classificador para rotear intenções de VPS para o novo agente

---

### - [ ] Tarefa 021 — Implementar Email Agent (Exim/Entregabilidade)
**Label:** `AGENT`, `TOOL`
**Dependência:** 🔗 Tarefa 014, 🔗 Tarefa 013
**Descrição:** Criar ferramentas e o subgrafo especializado em resolução de problemas de e-mails corporativos.

**Checklist:**
- [ ] Criar pasta `src/argos/agents/email/`
- [ ] Implementar ferramentas em `email/tools.py`: `check_mail_queue` (cPanel), `validate_spf_dkim`, `verify_rbl_blacklist` (HTTP queries para RBLs conhecidas)
- [ ] Criar prompt focando em entregabilidade, SMTP, MX e retransmissões
- [ ] Criar o subgrafo `email_subgraph` e integrá-lo no grafo principal
- [ ] Adicionar testes cobrindo a execução lógica do email agent

---

### - [ ] Tarefa 022 — Implementar DNS Agent (dig/SSL)
**Label:** `AGENT`, `TOOL`
**Dependência:** 🔗 Tarefa 014, 🔗 Tarefa 013
**Descrição:** Criar ferramentas de diagnóstico de DNS/SSL e o subgrafo especializado.

**Checklist:**
- [ ] Criar pasta `src/argos/agents/dns/`
- [ ] Implementar ferramentas: `check_dns_propagation` (consultando servidores DNS via dig assíncrono), `verify_ssl_certificate`
- [ ] Criar prompt focado em propagação DNS, registros A/AAAA/CNAME/TXT e validade de certificados SSL
- [ ] Desenvolver subgrafo `dns_subgraph` e integrá-lo no grafo principal
- [ ] Adicionar testes de unidade para as ferramentas e fluxo do DNS Agent

---

### - [ ] Tarefa 023 — Criar ferramentas da API do WHMCS (Faturamento)
**Label:** `TOOL`
**Dependência:** 🔗 Tarefa 005
**Descrição:** Implementar wrappers HTTP assíncronos para a API do WHMCS focados em faturamento de clientes.

**Checklist:**
- [ ] Criar `src/argos/tools/whmcs.py`
- [ ] Implementar ferramenta `@tool` `list_customer_invoices`
- [ ] Implementar ferramenta `@tool` `get_invoice_link`
- [ ] Configurar fallback mockado na ausência de credenciais WHMCS
- [ ] Adicionar testes de unidade cobrindo requests simulados da API WHMCS

---

### - [ ] Tarefa 024 — Implementar Billing Agent
**Label:** `AGENT`
**Dependência:** 🔗 Tarefa 023, 🔗 Tarefa 013
**Descrição:** Criar o subgrafo do Billing Agent focado em faturamento e integrá-lo no fluxo do supervisor.

**Checklist:**
- [ ] Criar pasta `src/argos/agents/billing/`
- [ ] Definir o prompt do Billing Agent contendo diretrizes estritas de segurança de dados e conformidade (LGPD)
- [ ] Criar o subgrafo `billing_subgraph` associado às ferramentas do WHMCS
- [ ] Registrar o nó `billing_agent` no grafo principal e no classificador de intenção
- [ ] Escrever testes unitários e de fluxo para o Billing Agent

---

### - [ ] Tarefa 025 — Implementar Site Builder Agent (Extendify/WordPress)
**Label:** `AGENT`, `TOOL`
**Dependência:** 🔗 Tarefa 010
**Descrição:** Criar subgrafo de Site Builder especializado em WordPress e no provisionamento automatizado de templates Gutenberg/Extendify.

**Checklist:**
- [ ] Criar pasta `src/argos/agents/site_builder/`
- [ ] Criar ferramentas em `site_builder/tools.py`: `check_wp_plugins`, `install_extendify_theme` (ambos mockando chamadas via SSH/WP-CLI)
- [ ] Definir o prompt focado em temas de blocos, plugins WordPress e templates do Gutenberg
- [ ] Criar o subgrafo `site_builder_subgraph` e registrá-lo no grafo principal
- [ ] Escrever testes de fluxo simulando problemas no WordPress e checando se o agente sugere as ferramentas adequadas

---

### - [ ] Tarefa 026 — Implementar Dedicated Server Agent (Bare-metal)
**Label:** `AGENT`, `TOOL`
**Dependência:** 🔗 Tarefa 010
**Descrição:** Criar subgrafo para diagnóstico de hardware de servidores dedicados físicos.

**Checklist:**
- [ ] Criar pasta `src/argos/agents/dedicated/`
- [ ] Criar ferramentas em `dedicated/tools.py`: `check_raid_status`, `check_ipmi_status`
- [ ] Definir o prompt focado em hardware, IPMI, volumes RAID físicos e monitoramento de rede local
- [ ] Configurar subgrafo `dedicated_subgraph` e integrá-lo no grafo principal
- [ ] Escrever testes cobrindo fluxo lógico de diagnóstico do dedicado

---

## 🔄 FASE 4 — Sintetizador Colaborativo & Background Tasks

---

### - [ ] Tarefa 027 — Implementar Sintetizador com Loop Critic/Revise
**Label:** `GRAPH`, `AGENT`
**Dependência:** 🔗 Tarefa 013
**Descrição:** Criar o nó sintetizador avançado aplicando o padrão Generator-Critic-Revise para compilar a resposta final unificada ao cliente.

**Checklist:**
- [ ] Criar pasta `src/argos/agents/synthesizer/`
- [ ] Criar `src/argos/agents/synthesizer/prompts.py` definindo instruções de unificação de tom da marca e validação
- [ ] Desenvolver no `node.py` o loop Generator-Critic-Revise:
  - [ ] LLM gera rascunho de resposta (`Generator`)
  - [ ] Um prompt secundário valida se todas as solicitações originais do usuário foram respondidas (`Critic`)
  - [ ] Se houver pendências de dados de outro domínio, solicita ao supervisor um redirecionamento ou refina o texto final (`Revise`)
- [ ] Substituir o placeholder do sintetizador no grafo principal pelo novo nó assíncrono implementado
- [ ] Escrever testes de integração simulando uma resposta fragmentada de dois agentes especialistas e validando que o sintetizador produz uma resposta única coerente

---

### - [ ] Tarefa 028 — Configurar APScheduler para Tarefas em Segundo Plano (Background Tasks)
**Label:** `TASK`
**Dependência:** 🔗 Tarefa 016
**Descrição:** Configurar o APScheduler para gerenciar rotinas assíncronas do sistema e inicializá-lo junto com o startup da aplicação FastAPI.

**Checklist:**
- [ ] Criar `src/argos/tasks/scheduler.py` configurando a instância do `AsyncIOScheduler`
- [ ] Integrar a inicialização do scheduler nos eventos de startup da aplicação em `src/argos/api/app.py`
- [ ] Configurar graceful shutdown para aguardar a finalização de tarefas pendentes no encerramento da API
- [ ] Validar inicialização correta do scheduler rodando o uvicorn

---

### - [ ] Tarefa 029 — Criar rotina assíncrona: SSL Checker
**Label:** `TASK`
**Dependência:** 🔗 Tarefa 028
**Descrição:** Implementar a tarefa proativa de checagem periódica de expiração de certificados SSL das contas hospedadas.

**Checklist:**
- [ ] Criar `src/argos/tasks/ssl_checker.py` contendo a tarefa assíncrona `check_expiring_ssls`
- [ ] Configurar a rota na tabela para listar domínios locais e verificar o vencimento do SSL via socket do Python
- [ ] Registrar a tarefa no scheduler com recorrência programada (diária/semanal)
- [ ] Escrever testes unitários e de integração mockando domínios vencidos e assegurando a geração de eventos de alerta no sistema

---

### - [ ] Tarefa 030 — Criar rotina assíncrona: Health Monitor
**Label:** `TASK`
**Dependência:** 🔗 Tarefa 028
**Descrição:** Implementar o monitoramento periódico de saúde e disponibilidade dos servidores.

**Checklist:**
- [ ] Criar `src/argos/tasks/health_monitor.py` contendo a tarefa `monitor_servers_health`
- [ ] Configurar ping HTTP/TCP assíncrono para a lista de servidores
- [ ] Registrar a tarefa no scheduler com recorrência curta (ex: a cada 5 minutos)
- [ ] Integrar logs estruturados em caso de indisponibilidade de servidor (HTTP != 200 ou timeout)
- [ ] Escrever testes de integração mockando falhas de servidores e garantindo a correta sinalização

---

## 🛡️ FASE 5 — Observabilidade, Segurança & Testes de Produção

---

### - [ ] Tarefa 031 — Configurar Observabilidade com LangSmith
**Label:** `OBS`
**Dependência:** 🔗 Tarefa 005
**Descrição:** Configurar o tracing completo das execuções e chamadas LLM integrando-se nativamente com o LangSmith.

**Checklist:**
- [ ] Configurar as variáveis de ambiente necessárias para o LangSmith no `.env`
- [ ] Garantir que o tracing é iniciado com o bootstrap do grafo principal
- [ ] Verificar que cada iteração e chamada de ferramenta é registrada com os inputs e outputs correspondentes
- [ ] Validar visualmente os grafos e tempos de resposta direto no dashboard do LangSmith

---

### - [ ] Tarefa 032 — Configurar logs estruturados com Structlog
**Label:** `OBS`
**Dependência:** 🔗 Tarefa 003
**Descrição:** Configurar log estruturado em console/arquivo no formato JSON para ambientes de produção e amigável em ambiente de desenvolvimento.

**Checklist:**
- [ ] Criar `src/argos/utils/logging.py`
- [ ] Configurar o `structlog` com os processadores necessários (timestamp, level, JSON ou ConsoleRender)
- [ ] Substituir chamadas do print nativo por logging estruturado nos nós e nos serviços internos
- [ ] Injetar dados de metadados como `thread_id` nos logs de eventos dos WebSockets de forma automática
- [ ] Escrever teste validando a saída correta dos logs estruturados

---

### - [ ] Tarefa 033 — Implementar Human-in-the-Loop (HITL) para Operações Críticas
**Label:** `GRAPH`, `API`
**Dependência:** 🔗 Tarefa 018
**Descrição:** Configurar interrupções ("interrupts") no LangGraph para cenários sensíveis, exigindo aprovação manual ou ação humana.

**Checklist:**
- [ ] Configurar checkpoints com interrupções no grafo principal usando o parâmetro `interrupt_before` ou `interrupt_after` em nós sensíveis de faturamento ou hardware
- [ ] Criar rotas HTTP em `src/argos/api/rest.py` para permitir que o painel administrativo aprova ou rejeita a ação que causou a interrupção
- [ ] Notificar o cliente/operador via WebSocket do estado de espera por aprovação humana
- [ ] Escrever teste de unidade validando que a execução do grafo congela no nó seguro e retoma apenas após a chamada de aprovação na API

---

### - [ ] Tarefa 034 — Escrever testes E2E com cenários de múltiplos domínios
**Label:** `TEST`
**Dependência:** 🔗 Tarefa 018, 🔗 Tarefa 027
**Descrição:** Desenvolver testes de ponta a ponta simulando as principais interações de clientes no suporte da hospedagem, cruzando múltiplos agentes especialistas.

**Checklist:**
- [ ] Criar arquivo `tests/e2e/test_scenarios.py`
- [ ] Escrever teste simulando fluxo: cliente reclama do site fora do ar -> agente de DNS diagnostica problema de propagação e sugere correção -> cliente solicita upgrade de plano e é direcionado para o Billing Agent
- [ ] Mockar de forma segura o tráfego do LLM e validar a transição lógica dos nós do grafo
- [ ] Validar se as mensagens finais geradas contêm informações acumuladas de ambos os domínios

---

### - [ ] Tarefa 035 — Configurar GitHub Actions CI
**Label:** `INFRA`
**Dependência:** 🔗 Tarefa 002
**Descrição:** Criar a esteira de CI simples para validação do código em cada pull request.

**Checklist:**
- [ ] Criar `.github/workflows/ci.yml`
- [ ] Configurar o workflow para instalar Python, Poetry e as dependências
- [ ] Adicionar passos de verificação: Ruff check/format, Mypy type check e pytest
- [ ] Garantir que o CI passe localmente simulando as etapas do Actions
