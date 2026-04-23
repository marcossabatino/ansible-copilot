# PRD — Versão Completa
## Sistema Híbrido de Agentes para Automação Ansible

**Versão:** 1.0
**Data:** 23 de abril de 2026
**Tempo estimado de implementação:** 25–40 horas distribuídas em 3–5 semanas
**Status:** Para implementação após validação da Versão Reduzida

---

## 1. Sumário Executivo

Este documento especifica a versão madura e completa do sistema de agentes para automação Ansible. Expande o MVP (Versão Reduzida) adicionando:

- **LLM local** via Ollama para tarefas simples e modo privacidade
- **Orquestrador inteligente** (LangGraph) com roteamento automático entre modelos e agentes
- **Agentes adicionais** cobrindo migração, módulos custom, integrações e documentação
- **Biblioteca de templates** Ansible reutilizáveis
- **Pipeline CI/CD próprio** e devcontainer padronizado
- **Observabilidade avançada** e métricas customizadas

**Pré-requisito absoluto:** a Versão Reduzida deve estar em produção há pelo menos 30 dias, com dados de uso coletados. Este PRD deve ser recalibrado com base nesses dados antes da implementação.

---

## 2. Contexto e Justificativa

### 2.1. Quando implementar esta versão

Esta versão **só faz sentido** se, após 30 dias rodando a Versão Reduzida, pelo menos três dos seguintes sinais aparecerem:

- Uso consistente (≥ 5 demandas/semana) comprovando valor
- Custo API mensal > $80 (local seria economicamente justificável)
- Demanda recorrente por migração de scripts, módulos custom, ou integrações
- Restrição contratual de cliente exigindo processamento local
- Necessidade repetida de documentação derivada de código
- Estimativa honesta de disponibilidade de 25-40h + 2-5h/mês de manutenção

Se nenhum desses sinais aparecer, **não implemente esta versão**. A Versão Reduzida está entregando o que precisava.

### 2.2. Filosofia

A Versão Completa **não substitui** a Reduzida — ela evolui a partir dela. Cada componente adicional deve ser justificado por dados reais de uso, não por especulação sobre necessidades futuras.

---

## 3. Objetivos

### 3.1. Objetivos (em ordem de prioridade)

1. **Manter custo operacional controlado** — alvo: < $80/mês mesmo com uso intenso
2. **Cobertura funcional ampla** — atender ~95% do JD do cargo
3. **Privacidade opcional** — modo local forçável para contextos sensíveis
4. **Qualidade máxima** — roteamento que usa o modelo certo para cada tarefa
5. **Escalabilidade** — fácil adicionar novos agentes sem refatoração

### 3.2. Não-Objetivos

- Não é um produto comercializável; permanece ferramenta pessoal
- Não substitui decisões de arquitetura/design do profissional
- Não automatiza execução em produção (só gera código)

---

## 4. Escopo

### 4.1. Inclui (além do que já está na Versão Reduzida)

#### 4.1.1. Camada de Execução Local
- Ollama instalado e configurado
- Modelos locais: Qwen3-Coder (para geração) e Llama 3.3 8B (para classificação/roteamento)
- Modo privacidade: flag que força roteamento 100% local

#### 4.1.2. Orquestração Inteligente
- LangGraph como serviço Python expondo API compatível com OpenAI
- Classificador de intenção (qual agente + qual modelo)
- Modo "auto-route" selecionável no dropdown ao lado dos agentes específicos
- Lógica de fallback (se local falha, tenta API; se budget estourou, avisa)

#### 4.1.3. Conjunto Completo de Agentes
- Playbook Author (já existe)
- Reviewer (já existe)
- **Migrator** — converte PowerShell/Bash/scheduled tasks em Ansible
- **Module/Collection Builder** — gera módulos Python seguindo padrão Ansible
- **Integrator** — especialista em ServiceNow/Splunk/VMware/SQL APIs
- **Doc Writer** — produz README, docs de role, knowledge transfer

#### 4.1.4. Biblioteca de Templates
- Repositório privado com roles canônicas (patching Linux/Windows, compliance, instalação de agentes comuns)
- Estrutura de collection esqueleto com testes e workflow de release

#### 4.1.5. CI/CD Próprio
- GitHub Actions reutilizável: yamllint → ansible-lint → molecule test → publish em AAP
- Configurável para reuso entre os múltiplos empregos

#### 4.1.6. Devcontainer
- Imagem Docker com ansible-core, lint tools, Molecule, Python + libs comuns, WinRM/SSH configurados

#### 4.1.7. Observabilidade
- Métricas customizadas: uso por agente, taxa de retrabalho estimada, tempo economizado
- Alertas em budget e em comportamento anômalo

### 4.2. Fora do Escopo (mesmo na versão completa)

- Multi-usuário
- Aplicativo mobile
- RAG sobre repositórios (fica para uma possível "Versão Avançada" futura)
- Execução direta em AAP (sistema só gera código, execução continua manual ou via pipelines existentes)

---

## 5. Arquitetura Técnica

### 5.1. Visão Geral

```
┌─────────────────────────────────────────────────────────┐
│                       USUÁRIO                           │
│                  (navegador web)                        │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    Open WebUI                           │
│  Dropdown expandido:                                    │
│   • 📝 Playbook Author (Sonnet)                         │
│   • 🔍 Reviewer (Opus)                                  │
│   • 🔄 Migrator (Sonnet)                                │
│   • 🐍 Module Builder (Opus)                            │
│   • 🔌 Integrator (Sonnet)                              │
│   • 📚 Doc Writer (Haiku)                               │
│   • 🤖 Auto-Route (orquestrador escolhe)                │
│   • 🔒 Modo Privacidade (força local)                   │
└──────────────────────────┬──────────────────────────────┘
                           │ OpenAI-compatible API
                           ▼
┌─────────────────────────────────────────────────────────┐
│                      LiteLLM                            │
│  - Proxy unificado                                      │
│  - Roteamento inicial por nome do modelo                │
│  - Budget/rate limiting                                 │
│  - Métricas e logs centralizados                        │
└─────┬────────────────┬─────────────────┬────────────────┘
      │                │                 │
      ▼                ▼                 ▼
┌──────────┐    ┌──────────────┐   ┌────────────────────┐
│Anthropic │    │   Ollama     │   │  LangGraph Service │
│   API    │    │  (local)     │   │  (orquestrador)    │
│          │    │              │   │                    │
│Haiku 4.5 │    │qwen3-coder   │   │ - Router/classifier│
│Sonnet4.6 │    │llama-3.3-8b  │   │ - Multi-step flows │
│Opus 4.7  │    │              │   │ - Fallback logic   │
└──────────┘    └──────────────┘   └────────────────────┘
                                            │
                                            ▼
                                    ┌────────────────┐
                                    │  Chama Anthropic│
                                    │  ou Ollama via  │
                                    │  LiteLLM        │
                                    └────────────────┘

┌─────────────────────────────────────────────────────────┐
│            REPOSITÓRIOS DE SUPORTE                      │
│  • templates/ (roles canônicas)                         │
│  • collection-skeleton/ (boilerplate de collections)    │
│  • .github/workflows/ (CI reutilizável)                 │
│  • .devcontainer/ (ambiente padronizado)                │
└─────────────────────────────────────────────────────────┘
```

### 5.2. Componentes

| Componente | Tecnologia | Função | Origem |
|---|---|---|---|
| Interface | Open WebUI | UI de chat + dropdown expandido | Reduzida (mantida) |
| Proxy | LiteLLM | Unificador de APIs, tracking | Reduzida (mantida) |
| LLM Comercial | Anthropic API | Claude 4.x | Reduzida (mantida) |
| LLM Local | Ollama | Qwen3-Coder + Llama 3.3 8B | **Novo** |
| Orquestrador | LangGraph (FastAPI) | Router inteligente, multi-step | **Novo** |
| Persistência | SQLite (Open WebUI) + PostgreSQL (métricas) | Histórico + analytics | Parcialmente novo |
| Templates | Git privado | Roles canônicas reutilizáveis | **Novo** |
| CI/CD | GitHub Actions | Lint + test + publish | **Novo** |
| Devcontainer | Dockerfile + devcontainer.json | Ambiente padronizado | **Novo** |

### 5.3. Fluxo de Roteamento Automático (Modo Auto)

1. Usuário escolhe "🤖 Auto-Route" no dropdown
2. Open WebUI → LiteLLM → LangGraph
3. LangGraph usa **Llama 3.3 8B local** para classificar intenção:
   - "Isso é geração de playbook? Revisão? Migração? Módulo? Integração? Doc?"
   - "É simples ou complexo?"
   - "Há indicação de dados sensíveis?"
4. Aplica matriz de decisão (seção 5.4)
5. Chama modelo escolhido com system prompt do agente adequado
6. Retorna resposta com log visível: "Classificado como: geração complexa → Roteado para Sonnet 4.6 (Playbook Author)"

### 5.4. Matriz de Decisão do Auto-Route

| Classificação | Complexidade | Privacidade exigida | Modelo/Agente escolhido |
|---|---|---|---|
| Classificação/triagem | Qualquer | Qualquer | Llama 3.3 8B (local) |
| Geração de boilerplate simples | Baixa | Não | Haiku 4.5 + Playbook Author |
| Geração de boilerplate simples | Baixa | Sim | Qwen3-Coder (local) + Playbook Author |
| Geração de playbook complexo | Média/Alta | Não | Sonnet 4.6 + Playbook Author |
| Geração de playbook complexo | Média/Alta | Sim | Qwen3-Coder (local) + Playbook Author |
| Revisão normal | Média | Não | Sonnet 4.6 + Reviewer |
| Revisão crítica (compliance, segurança) | Alta | Não | Opus 4.7 + Reviewer |
| Migração script → Ansible | Média | Não | Sonnet 4.6 + Migrator |
| Módulo Python custom | Alta | Não | Opus 4.7 + Module Builder |
| Integração com API externa | Média | Não | Sonnet 4.6 + Integrator |
| Documentação a partir de código | Baixa | Não | Haiku 4.5 + Doc Writer |

---

## 6. Requisitos Funcionais

### RF-01 — Todos os Requisitos da Versão Reduzida
Mantidos integralmente (RF-01 a RF-05 do PRD Reduzido).

### RF-06 — LLM Local via Ollama

**Descrição:** Sistema deve permitir execução local de LLMs para tarefas apropriadas e modo privacidade.

**Critérios de aceite:**
- Ollama rodando com ao menos Qwen3-Coder e Llama 3.3 8B
- LiteLLM expõe modelos Ollama como opções no dropdown
- Performance aceitável (< 60s para playbooks típicos)
- Uso de VRAM/RAM documentado e dentro de limites do hardware escolhido

### RF-07 — Modo Privacidade

**Descrição:** Opção que força roteamento 100% local, bloqueando qualquer chamada externa.

**Critérios de aceite:**
- Toggle ou opção no dropdown claramente identificado
- Quando ativo, qualquer tentativa de roteamento para API externa é rejeitada
- Log explícito de que modo privacidade está ativo
- Indicador visual persistente enquanto ativo

### RF-08 — Orquestrador (Auto-Route)

**Descrição:** Modo que delega a escolha de agente + modelo ao orquestrador baseado em classificação da demanda.

**Critérios de aceite:**
- Disponível como opção "🤖 Auto-Route" no dropdown
- Classificador local (Llama 3.3 8B) decide em < 3s
- Decisão visível para o usuário com justificativa
- Usuário pode sobrescrever a decisão re-enviando com agente específico

### RF-09 — Agente Migrator

**Descrição:** Converte scripts PowerShell/Bash/scheduled tasks em equivalentes Ansible.

**Critérios de aceite:**
- Aceita script colado ou arquivo upload
- Devolve playbook/role usando módulos nativos quando possível
- Identifica e marca tarefas que precisaram de shell/command com justificativa
- Destaca diferenças semânticas (ex: "no script original havia retry implícito, aqui usei until/retries")

### RF-10 — Agente Module/Collection Builder

**Descrição:** Gera módulos Ansible em Python seguindo padrões oficiais.

**Critérios de aceite:**
- Estrutura correta: AnsibleModule, argument_spec, return values, documentação em YAML
- check_mode suportado
- Testes unitários gerados (pytest + Molecule se aplicável)
- Seguir padrão de naming e organização de collections

### RF-11 — Agente Integrator

**Descrição:** Gera código Ansible/Python para integração com ServiceNow, Splunk, VMware e SQL/Oracle.

**Critérios de aceite:**
- Autenticação correta (OAuth, token, basic)
- Tratamento de paginação, retry, rate limiting
- Não hardcode de credenciais (uso de vault)
- Exemplos funcionais testados em sandbox quando possível

### RF-12 — Agente Doc Writer

**Descrição:** Gera documentação a partir de código Ansible.

**Critérios de aceite:**
- README de role com: propósito, variáveis, dependências, exemplo
- Documentação de módulo custom em formato Ansible (DOCUMENTATION, EXAMPLES, RETURN)
- Documento de knowledge transfer em markdown com diagramas textuais quando útil

### RF-13 — Biblioteca de Templates

**Descrição:** Repositório com roles canônicas pré-construídas para os casos mais frequentes.

**Critérios de aceite:**
- Mínimo de 5 roles prontas e testadas (ex: patching-linux, patching-windows, cis-baseline-rhel, splunk-agent-install, monitoring-agent-install)
- Cada role com README, defaults, handlers, testes Molecule
- Versionamento semântico
- Agentes podem referenciar essas roles como contexto

### RF-14 — CI/CD Reutilizável

**Descrição:** Workflow GitHub Actions que pode ser aplicado a qualquer repositório de automação.

**Critérios de aceite:**
- Estágios: yamllint → ansible-lint → molecule test → publish em AAP (opcional, configurável)
- Reutilizável como `uses:` em outros repositórios
- Configuração via inputs (versão Ansible, caminho de collections, etc.)

### RF-15 — Devcontainer

**Descrição:** Ambiente Docker padronizado para desenvolvimento de automações.

**Critérios de aceite:**
- Inclui: ansible-core, ansible-lint, yamllint, Molecule, Python 3.11+, PowerShell Core, WinRM/SSH tools
- `devcontainer.json` configurado para VS Code
- Tempo de build inicial < 5 min, reuso via cache

### RF-16 — Observabilidade Avançada

**Descrição:** Métricas e dashboard além do básico do LiteLLM.

**Critérios de aceite:**
- Uso por agente, por modelo, por dia/semana/mês
- Tempo médio de resposta por agente
- Taxa de retrabalho estimada (feedback manual opcional)
- Estimativa de tempo economizado vs. baseline

---

## 7. Requisitos Não-Funcionais

| ID | Categoria | Requisito | Métrica |
|---|---|---|---|
| RNF-01 | Performance | Resposta API | < 30s para 95% dos casos |
| RNF-02 | Performance | Resposta local | < 60s para 95% dos casos |
| RNF-03 | Disponibilidade | Uptime | 95% no horário de trabalho |
| RNF-04 | Custo | Gasto mensal API | ≤ $80/mês |
| RNF-05 | Segurança | Secrets | Em vault ou env vars apenas |
| RNF-06 | Privacidade | Modo local | 100% offline, 0 chamadas externas |
| RNF-07 | Manutenibilidade | Setup | Docker Compose único comando |
| RNF-08 | Manutenibilidade | Esforço mensal | ≤ 5h/mês de manutenção |
| RNF-09 | Observabilidade | Logs estruturados | Cada chamada com timestamp, modelo, tokens, custo |
| RNF-10 | Extensibilidade | Adicionar agente | ≤ 2h para novo agente (prompt + config) |

---

## 8. Cronograma de Implementação (25–40 horas em 3–5 semanas)

Implementação **incremental**, em fases que entregam valor cumulativamente.

### Fase 0 — Ponto de Partida
A Versão Reduzida já está rodando há 30+ dias e fornece dados de uso para priorizar.

### Fase 1 — LLM Local e Modo Privacidade (4–6h)

| Passo | Tempo |
|---|---|
| Decidir e preparar hardware (ver seção 9) | variável |
| Instalar Ollama e baixar Qwen3-Coder + Llama 3.3 8B | 1h |
| Configurar LiteLLM para expor modelos locais | 30 min |
| Adicionar opções no dropdown | 15 min |
| Implementar toggle de modo privacidade | 1h |
| Testar qualidade local vs API em casos reais | 1–2h |
| Ajustar roteamento conforme resultados | 30 min |

**Entrega:** sistema híbrido funcional, modo privacidade operacional.

### Fase 2 — Orquestrador com Auto-Route (6–10h)

| Passo | Tempo |
|---|---|
| Estruturar serviço LangGraph (FastAPI) | 1h |
| Implementar classificador usando Llama local | 2h |
| Implementar matriz de decisão | 1h |
| Expor endpoint OpenAI-compatible | 1h |
| Integrar com LiteLLM e Open WebUI | 1h |
| Implementar logging de decisões | 30 min |
| Testes com 10+ demandas reais variadas | 2–3h |
| Ajuste da matriz de decisão | 1h |

**Entrega:** modo auto-route funcional com decisões visíveis e ajustáveis.

### Fase 3 — Agentes Adicionais (8–12h)

Cada agente: ~2h = prompt + config + testes com 2-3 casos reais.

| Agente | Tempo | Prioridade |
|---|---|---|
| Migrator | 2h | Alta (se migração for frequente) |
| Module/Collection Builder | 2–3h | Média (pede mais qualidade no prompt) |
| Integrator | 2–3h | Média (depende de quais integrações aparecem) |
| Doc Writer | 1–2h | Baixa (tarefa mais simples) |

**Implementar apenas os agentes que os dados de uso da Fase 0 justificarem.**

### Fase 4 — Templates, CI/CD, Devcontainer (4–8h)

| Passo | Tempo |
|---|---|
| Estruturar repositório de templates | 1h |
| Criar 5 roles canônicas (iterativamente ao longo do uso) | 2–4h |
| Workflow GitHub Actions reutilizável | 1–2h |
| Devcontainer base | 1h |

**Entrega:** fundação de reuso estabelecida.

### Fase 5 — Observabilidade Avançada (3–4h)

| Passo | Tempo |
|---|---|
| Definir métricas-alvo | 30 min |
| Implementar coleta (log structured + PostgreSQL) | 1–2h |
| Dashboard simples (Grafana ou Metabase) | 1–2h |

**Entrega:** visibilidade completa de uso e ROI.

### Totais

| Fase | Tempo mínimo | Tempo máximo |
|---|---|---|
| 1 — LLM Local | 4h | 6h |
| 2 — Orquestrador | 6h | 10h |
| 3 — Agentes adicionais | 8h | 12h |
| 4 — Templates/CI/Devcontainer | 4h | 8h |
| 5 — Observabilidade | 3h | 4h |
| **Total** | **25h** | **40h** |

Distribuído em 3–5 semanas assumindo 6–10h por semana dedicadas.

---

## 9. Decisão: Onde Rodar o LLM Local

Recalibrar com base nos 30 dias da Versão Reduzida:

### 9.1. Opção A — Máquina Pessoal
**Escolher se:** já possui Mac M-series ≥ 16GB ou PC com GPU NVIDIA ≥ 12GB VRAM.
**Custo:** $0 adicional + ~$5-15/mês de eletricidade.
**Recomendação:** melhor opção se hardware já existe.

### 9.2. Opção B — Cloud GPU sob Demanda (RunPod/Vast.ai)
**Escolher se:** sem hardware local, uso esporádico.
**Custo:** ~$35–60/mês para 4h/dia.
**Cuidado:** dados saem da sua rede (mas em ambiente isolado).

### 9.3. Opção C — Servidor Dedicado / Homelab
**Escolher se:** uso intenso e contínuo, outros usos para o servidor.
**Custo:** $1500–3000 inicial + $20–60/mês.
**Payback vs. cloud:** 24+ meses.

### 9.4. Recomendação Padrão

Se após 30 dias de Versão Reduzida:
- Gasto < $60/mês → **não adicionar local ainda**, custo-benefício não justifica
- Gasto $60–100/mês + hardware disponível → **Opção A**
- Gasto > $100/mês sem hardware → **Opção B** inicialmente, migrar para A/C se mantiver
- Demanda de cliente exige privacidade → **Opção A** ou **B** obrigatória

---

## 10. Custos

### 10.1. Setup
- Tempo: 25–40h (trabalho próprio)
- Hardware: $0 a $3000 dependendo da opção escolhida para LLM local
- Software: $0 (tudo open source ou pay-as-you-go)

### 10.2. Recorrente Mensal

**Cenário Moderado (~100 demandas/mês, híbrido balanceado):**

| Item | Custo |
|---|---|
| Anthropic API (Sonnet/Opus para complexo) | $30–50 |
| Cloud GPU (se Opção B) | $0–60 |
| Eletricidade (se Opção A) | $5–15 |
| **Total** | **$35–125** |

**Cenário Intenso (~200+ demandas/mês):**

| Item | Custo |
|---|---|
| Anthropic API | $60–100 |
| Infra local | $5–60 |
| **Total** | **$65–160** |

### 10.3. ROI

Se o sistema completo economizar **4h/semana** vs. baseline atual, o payback é imediato mesmo no cenário mais caro.

---

## 11. Métricas de Sucesso (avaliar em 90 dias)

| Métrica | Meta |
|---|---|
| Uso efetivo de 4+ agentes diferentes | Sim |
| % de demandas resolvidas via auto-route sem override manual | > 70% |
| Custo mensal total | < $130 |
| Tempo economizado estimado vs. baseline | > 4h/semana |
| Tempo de manutenção do sistema | < 5h/mês |
| Quantidade de templates reutilizados | ≥ 3 em demandas reais |

### Critério de Sucesso Geral

Sistema se mantém em uso ativo após 90 dias **e** profissional reporta que não voltaria para fluxo anterior.

---

## 12. Riscos e Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Overengineering: componentes que não são usados | Alta | Médio | Implementação incremental, cada fase só após validação |
| Custo API estourar orçamento | Média | Alto | Budget hard limit + auto-route privilegiando local |
| Qualidade do LLM local insuficiente | Média | Médio | Testes A/B; caso insatisfatório, manter como apenas fallback |
| Vazamento de dados sensíveis | Baixa | Crítico | Política documentada + modo privacidade + anonimização |
| Manutenção excede benefício | Média | Alto | Métrica explícita de tempo gasto; simplificar se > 5h/mês |
| LangGraph/dependências quebram em update | Média | Médio | Pinning de versões, `docker compose pull` manual e testado |
| Agentes adicionais pouco usados | Alta | Baixo | Implementar só os que dados de uso justificarem |
| Hardware local insuficiente | Média | Médio | Começar com Opção B (cloud) antes de investir em Opção C |

---

## 13. Decisões Arquiteturais Críticas

### 13.1. Por que LangGraph e não continuar só com LiteLLM?

Na Versão Reduzida, LiteLLM sozinho é suficiente porque cada agente é um "modelo virtual" estático. Na Completa, precisamos de:
- Classificação dinâmica antes de escolher o modelo
- Multi-step flows (ex: classificar → chamar → possivelmente reencaminhar)
- Fallback lógico (local falhou → tenta API)

LiteLLM não oferece isso nativamente. LangGraph é a ferramenta adequada.

### 13.2. Por que Open WebUI continua sendo a interface?

Considerado troca por LibreChat, mas Open WebUI:
- Já está em produção na Versão Reduzida
- Dropdown nativo para múltiplos modelos funciona bem
- Familiaridade > troca por ganho marginal

### 13.3. Por que SQLite + PostgreSQL (e não só um)?

- SQLite já é usado pelo Open WebUI para histórico de conversas; sem motivo para trocar
- PostgreSQL para métricas/analytics é padrão e integra bem com Grafana/Metabase
- Coexistem sem atrito

---

## 14. Evolução Pós-Versão Completa

Candidatos para uma hipotética "Versão Avançada" (não no escopo deste PRD):

- RAG sobre repositórios próprios — agentes leem seu histórico de playbooks como contexto automático
- Agente "Architect" que desenha soluções multi-playbook
- Integração direta com AAP (dispara execução após aprovação humana)
- Feedback loop automático: PR aceito/rejeitado alimenta melhoria contínua dos prompts
- Suporte multi-idioma (atualmente português + inglês apenas)

Todas essas ideias ficam arquivadas aqui, mas **só serão consideradas após a Versão Completa rodar 90+ dias** com valor comprovado.

---

## 15. Anexos

### Anexo A — Stack Tecnológica Completa

| Camada | Tecnologia |
|---|---|
| Linguagem principal | Python 3.11+ |
| Containerização | Docker + Docker Compose |
| Interface | Open WebUI |
| Proxy LLM | LiteLLM |
| Orquestração | LangGraph + FastAPI |
| LLM API | Anthropic Claude 4.x |
| LLM Local | Ollama + Qwen3-Coder + Llama 3.3 |
| Persistência conversas | SQLite (Open WebUI) |
| Persistência métricas | PostgreSQL |
| Dashboard | Grafana ou Metabase |
| CI/CD | GitHub Actions |
| IaC/Dev env | Dockerfile + devcontainer.json |
| Versionamento | Git + GitHub privado |

### Anexo B — Estrutura de Repositórios

```
~/work/
├── ansible-agents/                 # Sistema principal
│   ├── docker-compose.yml
│   ├── litellm-config.yaml
│   ├── langgraph-service/
│   │   ├── agents/
│   │   ├── prompts/
│   │   ├── router.py
│   │   └── main.py
│   └── README.md
├── ansible-templates/              # Biblioteca de roles
│   ├── roles/
│   ├── collection-skeleton/
│   └── README.md
├── ansible-ci-actions/             # CI/CD reutilizável
│   └── .github/workflows/
└── ansible-devcontainer/           # Ambiente padrão
    └── .devcontainer/
```

### Anexo C — Pré-Requisitos para Iniciar

1. Versão Reduzida rodando há 30+ dias
2. Dados de uso coletados (logs LiteLLM)
3. Análise honesta: quais agentes/componentes os dados justificam?
4. Hardware decidido (local vs cloud vs homelab)
5. Bloco de tempo reservado: 6–10h/semana por 3–5 semanas

### Anexo D — Critérios de Não-Implementação

**Não implementar esta versão se:**
- Uso da Versão Reduzida < 3 demandas/semana após 30 dias
- Custo API atual já está confortavelmente abaixo de $50/mês
- Não há demanda concreta dos empregos por migração/módulos custom/integrações
- Não há disponibilidade real de 25–40h nas próximas 5 semanas
- Manutenção da Reduzida já está incomodando (adicionar complexidade piorará)

Em qualquer um desses casos: **mantenha a Reduzida** e reavalie em 60-90 dias.

---

**Fim do documento.**
