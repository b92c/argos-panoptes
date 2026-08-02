# Especificação Técnica — Argos Panoptes

## 1. Visão Geral do Sistema
- **Nome do projeto**: Argos Panoptes (referência ao gigante da mitologia grega com 100 olhos - que tudo vê)
- **Propósito**: Sistema multi-agentes construído com LangGraph para atendimento automatizado de suporte ao cliente de uma empresa de hospedagem de sites. O sistema atua de forma proativa e reativa.
- **Stack**: Python 3.12+, LangGraph, FastAPI, WebSocket, PostgreSQL, Redis.
- **Domínios atendidos**: 
  - Hospedagem Compartilhada
  - VPS (Virtual Private Server)
  - Servidores Dedicados
  - E-mail (Configuração, Troubleshooting, Blacklists)
  - Criação de Sites (Integração com Extendify)
  - DNS (Zonas, Propagação, Registros)
  - SSL (Emissão, Renovação, Instalação)
  - Faturamento e Cobrança (Billing)

## 2. Arquitetura de Alto Nível

A arquitetura do Argos Panoptes é baseada em um orquestrador central (Supervisor) que delega tarefas para agentes especialistas (Subgraphs), operando através de uma interface de comunicação em tempo real bidirecional.

```mermaid
graph TD
    Client[Cliente/Frontend] <-->|WebSocket JSON| WSGateway[WebSocket Gateway\nFastAPI]
    
    WSGateway -->|Stream Eventos| IntentClassifier[Intent Classifier Node]
    
    subgraph Orquestração LangGraph
        IntentClassifier --> Supervisor[Supervisor Node\nOrchestrator]
        
        Supervisor -->|Command| HostingAgent[HostingAgent\nSubgraph]
        Supervisor -->|Command| VPSAgent[VPSAgent\nSubgraph]
        Supervisor -->|Command| DedicatedAgent[DedicatedAgent\nSubgraph]
        Supervisor -->|Command| EmailAgent[EmailAgent\nSubgraph]
        Supervisor -->|Command| SiteBuilderAgent[SiteBuilderAgent\nSubgraph]
        Supervisor -->|Command| DNSAgent[DNSAgent\nSubgraph]
        Supervisor -->|Command| BillingAgent[BillingAgent\nSubgraph]
        
        HostingAgent --> Synthesizer[Synthesizer Node]
        VPSAgent --> Synthesizer
        DedicatedAgent --> Synthesizer
        EmailAgent --> Synthesizer
        SiteBuilderAgent --> Synthesizer
        DNSAgent --> Synthesizer
        BillingAgent --> Synthesizer
        
        Synthesizer --> Supervisor
    end
    
    subgraph Assíncrono & Estado
        Scheduler[Background Task Scheduler\nCelery/Arq]
        StateStore[(State Store\nPostgreSQL + Redis)]
    end
    
    Orquestração LangGraph <..> StateStore
    Scheduler <..> StateStore
    
    subgraph APIs Externas
        cPanel[cPanel/WHM API]
        Extendify[Extendify API]
        WHMCS[WHMCS Billing]
        CloudLinux[CloudLinux API]
    end
    
    HostingAgent --> cPanel
    EmailAgent --> cPanel
    BillingAgent --> WHMCS
    SiteBuilderAgent --> Extendify
    VPSAgent --> CloudLinux
```

## 3. Stack Tecnológica Detalhada

- **Python 3.12+**: Utilizado por suas melhorias de performance e recursos nativos de concorrência e tipagem essenciais para o LangGraph.
- **LangGraph >= 0.3**: Framework principal para construção do fluxo de agentes. Utiliza `StateGraph`, `Command` para navegação inteligente entre nós, e subgrafos para agentes especialistas complexos.
- **LangChain Core**: Fornece as abstrações base para mensagens, modelos (LLMs) e ferramentas.
- **FastAPI**: Framework web de alta performance para gerenciar as conexões WebSocket bidirecionais e expor APIs REST auxiliares.
- **Pydantic v2**: Utilizado intensamente para definição e validação de schemas do estado do grafo, garantindo consistência dos dados trafegados entre os agentes.
- **PostgreSQL 16**: Banco de dados relacional principal. Utilizado para persistência de longo prazo e checkpointing do grafo através do `PostgresSaver`.
- **Redis**: Armazenamento in-memory em alta velocidade. Usado para caching, gerenciamento de estado efêmero de sessão, limitador de taxa (rate limiting) e pub/sub de eventos do sistema.
- **SQLAlchemy 2.0 + asyncpg**: ORM moderno e driver assíncrono para comunicação sem bloqueios de I/O com o PostgreSQL.
- **LangSmith**: Plataforma indispensável para observabilidade, rastreamento de execução (tracing) dos agentes, debug e avaliação de desempenho dos LLMs.
- **Docker + Docker Compose**: Containerização de toda a stack para desenvolvimento e facilidade de deploy.
- **pytest + pytest-asyncio**: Suíte de testes automatizados, crucial para validar comportamentos assíncronos e simulações do grafo.
- **Alembic**: Ferramenta oficial do SQLAlchemy para controle de versões do esquema de banco de dados (migrations).
- **Structlog**: Biblioteca para logging estruturado (JSON), vital para sistemas distribuídos e envio de logs para agregadores como ELK ou Datadog.
- **Tenacity**: Utilitário robusto para implementação de lógica de retry com exponential backoff ao se comunicar com APIs de terceiros.

## 4. Design do Estado (State Schema)

O estado global do grafo é a "memória" compartilhada durante o ciclo de vida do atendimento. Ele é atualizado progressivamente pelos agentes.

```python
import operator
from typing import Annotated, Literal, TypedDict
from pydantic import BaseModel, Field
from langchain_core.messages import AnyMessage

def add_messages(left: list[AnyMessage], right: list[AnyMessage]) -> list[AnyMessage]:
    """Função redutora para acumular mensagens no estado."""
    return left + right

class CustomerProfile(BaseModel):
    """Informações contextuais sobre o cliente atual."""
    client_id: int = Field(description="ID único do cliente no WHMCS")
    name: str = Field(description="Nome completo do cliente")
    tier: Literal["standard", "premium", "vip"] = Field(description="Nível de suporte do cliente")
    active_services: list[str] = Field(description="Lista de serviços ativos (ex: 'vps-ssd-1', 'shared-host-pro')")
    last_login: str | None = None

class IntentClassification(BaseModel):
    """Resultado da classificação de intenção do usuário."""
    primary_intent: Literal["hosting", "vps", "dedicated", "email", "site_builder", "dns", "billing", "general"]
    confidence_score: float = Field(ge=0.0, le=1.0)
    requires_escalation: bool = Field(default=False)
    entities: dict = Field(default_factory=dict, description="Entidades extraídas (ex: domínio, id_fatura)")

class ToolExecutionResult(BaseModel):
    """Resultado de uma chamada de ferramenta externa."""
    tool_name: str
    success: bool
    output: str | dict
    execution_time_ms: int

class AgentResponse(BaseModel):
    """Resposta estruturada gerada por um sub-agente."""
    agent_name: str
    message_to_user: str
    internal_notes: str | None = None
    suggested_actions: list[str] = Field(default_factory=list)

class ConversationContext(BaseModel):
    """Contexto efêmero da conversa atual."""
    current_active_agent: str | None = None
    is_human_escalated: bool = False
    pending_user_confirmation: bool = False
    
class AgentState(TypedDict):
    """
    Estado Principal do Grafo (StateGraph).
    Utiliza Annotated para acumulação de listas de mensagens.
    """
    # Histórico de mensagens. O redutor 'add_messages' garante o append seguro.
    messages: Annotated[list[AnyMessage], add_messages]
    
    # Perfil injetado na inicialização da conexão
    customer: CustomerProfile
    
    # Intenção atual para roteamento
    intent: IntentClassification | None
    
    # Lista de resultados de ferramentas executadas (acumulativo)
    tool_results: Annotated[list[ToolExecutionResult], operator.add]
    
    # Respostas finais geradas pelos agentes
    agent_responses: Annotated[list[AgentResponse], operator.add]
    
    # Contexto geral de controle de fluxo
    context: ConversationContext
```

## 5. Padrões Arquiteturais

A arquitetura do Argos Panoptes emprega diversos padrões avançados do LangGraph:

1. **Supervisor Pattern (Orquestração)**: Um nó central (Supervisor LLM) analisa o estado atual e decide qual agente especialista invocar utilizando o primitivo `Command`.
2. **Subgraph Pattern**: Cada domínio (ex: `EmailAgent`, `VPSAgent`) não é apenas uma prompt, mas sim um subgrafo inteiro com sua própria lógica de raciocínio, ferramentas de diagnóstico e recuperação de erros locais.
3. **Command Primitive**: Utilizado no LangGraph 0.3+ para delegar controle de fluxo de forma declarativa, permitindo que um subgrafo indique quem deve executar em seguida (ex: `Command(goto="Synthesizer")`).
4. **Conditional Edges**: Arestas de roteamento dinâmico utilizadas após a extração de intenção. Dependendo de `intent.primary_intent`, o grafo encaminha para subgrafos específicos de maneira determinística.
5. **Human-in-the-Loop**: Para operações destrutivas ou críticas (ex: reiniciar um servidor de produção, alterar zonas DNS, processar reembolso), o grafo pausa a execução (interrupt) e solicita um `Command(resume=True)` com aprovação explícita do usuário (ou de um operador humano N2).
6. **Generator-Critic-Revise (Synthesizer)**: Quando múltiplos agentes são consultados para um problema complexo (ex: e-mail caindo devido a IP de VPS em blacklist), os resultados de ambos os agentes vão para o nó `Synthesizer`, que revisa, critica e unifica uma resposta final coerente para o cliente.
7. **Accumulator Pattern**: Utilizado no `AgentState` com `Annotated[list, add_messages]`, garantindo que o histórico do chat seja mantido e apensado, em vez de sobrescrito, lidando perfeitamente com interações multipassos.

## 6. Agentes Ordinários vs Multi-Agentes: Análise de Trade-offs

O Argos Panoptes utiliza uma abordagem híbrida inteligente baseada na complexidade da tarefa.

| Característica | Agente Ordinário (Single Agent) | Multi-Agentes (Subgraphs / Swarm) |
| :--- | :--- | :--- |
| **Complexidade da Tarefa** | Simples, Linear, 1-2 passos | Complexa, Multi-domínio, Ramificada |
| **Latência** | Baixa (Poucas chamadas LLM) | Alta (Múltiplas chamadas LLM seq/paralelas) |
| **Custo de Token** | Baixo | Alto |
| **Escopo das Ferramentas** | Genéricas ou de leitura (RAG, FAQ, Status) | Especializadas, ações de escrita (Restart, Config) |
| **Recuperação de Falhas** | Limitada, prompt simples de retry | Robusta, loops locais dentro do subgrafo |
| **Casos de Uso (Argos)** | "Qual o status do meu servidor?", "Como acesso o painel?" | "Meu e-mail não chega, mas o site funciona e o disco do VPS parece cheio." |

**Recomendação Híbrida**: O sistema utiliza o nó inicial (Nível 1) como um Agente Ordinário para triagem e resolução imediata de problemas óbvios (RAG sobre a base de conhecimento). Se a ferramenta de resolução rápida falhar ou a intenção exigir diagnóstico aprofundado, o fluxo ativa a Orquestração Multi-Agentes (Nível 2+), injetando o contexto no Subgrafo correspondente.

## 7. Integração WebSocket

A comunicação com os clientes ocorre via WebSocket sobre FastAPI para suportar interações fluídas, streaming e eventos assíncronos.

- **Ciclo de Vida da Conexão**: O cliente conecta enviando o JWT token. O FastAPI valida e inicializa um novo `ConversationContext` recuperando dados do WHMCS.
- **Protocolo de Mensagens**: Comunicação baseada em JSON padronizado.
  ```json
  {
    "type": "user_message",
    "payload": { "text": "Meu site está fora do ar." }
  }
  ```
- **Streaming de Tokens**: Utilização da API `astream_events` do LangGraph. Conforme o nó (como o Synthesizer) gera a resposta, tokens parciais são enviados pelo WebSocket para criar o efeito de digitação (typing effect), diminuindo a percepção de latência.
- **Pooling e Reconexão**: O Gateway mantém um gerenciador de conexões em memória. Se a conexão cair, o front-end possui backoff algorítmico. Ao reconectar informando o `thread_id`, o estado é restaurado automaticamente do `PostgresSaver`.
- **Autenticação**: Via token JWT validado no handshake inicial do WebSocket.

## 8. Rotinas Assíncronas (Background Tasks)

O Argos não é apenas reativo. Agentes autônomos operam em background (acionados via APScheduler ou Celery) que podem inserir eventos no Grafo de um usuário específico, notificando-o ativamente via WebSocket.

- **Monitoramento de Saúde**: Pings regulares e verificações de load average em Servidores Dedicados. Caso ocorra degradação, um evento proativo "high_load_alert" é enviado ao grafo, que avisa o usuário: "Notei que seu servidor X está com carga alta. Deseja que eu reinicie o serviço Apache?".
- **Renovação SSL**: Scripts verificam diariamente certificados AutoSSL expirando em <7 dias.
- **Verificação de Backups**: Confirmação se as rotinas R1Soft/JetBackup executaram com sucesso na madrugada.
- **Alertas de Uso**: Notificação sobre uso de Disco, Banda, ou limites de inodes (CloudLinux LVE).
- **Relatórios**: Geração consolidada de relatórios mensais de uptime.

## 9. Segurança

- **API Tokens**: O acesso às APIs externas (WHMCS, cPanel) usa tokens armazenados no Vault/AWS Secrets Manager, nunca no código.
- **RBAC**: Baseado no `CustomerProfile.tier` e permissões do usuário, o sistema habilita ou desabilita ferramentas dentro do grafo.
- **Sanitização de Input**: Toda entrada passa por validação severa com Pydantic para prevenir injeções de prompt (Prompt Injection) e execução arbitrária.
- **Rate Limiting**: Configurado no Redis, limitando requisições WebSocket e invocações do grafo (ex: max 10 mensagens / minuto por usuário).
- **Trilha de Auditoria**: Qualquer alteração de estado (restart, mudança de DNS) executada por uma ferramenta gera log em uma tabela imutável `audit_logs` associada ao `thread_id` e `client_id`.

## 10. Observabilidade

- **LangSmith**: Todos os grafos são executados com `LANGCHAIN_TRACING_V2=true`. Permite visualizar o caminho exato que o Supervisor tomou, duração de cada nó e custos de tokens.
- **Structlog**: Logs no formato JSON injetando `thread_id` automaticamente usando contexto de variáveis (ContextVars) do Python.
- **Métricas**: Exposição de métricas no `/metrics` via Prometheus client no FastAPI (contadores de execução de agentes, taxa de sucesso de ferramentas, latência do WebSocket).
- **Alertas**: Alertas no Slack via webhook configurados se a taxa de "Human-in-the-Loop" (escalonamento manual) superar 20% das sessões ativas na última hora.

## 11. Requisitos Não-Funcionais

- **Latência de Resposta (Time to First Token)**: Máximo de 2 segundos para mensagens simples; máximo de 6 segundos para resoluções complexas que demandam ferramentas externas (via exibição de indicativos de carregamento "Estou verificando seu servidor...").
- **Throughput**: Capacidade para 5.000 conexões WebSocket ativas simultâneas por pod do Gateway.
- **Disponibilidade**: 99.9% de uptime (SLA), suportado pelo deploy redundante no Kubernetes.
- **Escalabilidade**: O sistema é *stateless* a nível de aplicação (estado guardado no Postgres/Redis), permitindo escalabilidade horizontal imediata adicionando mais workers do LangGraph/FastAPI.
