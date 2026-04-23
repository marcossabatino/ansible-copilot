# PRD — Versão Reduzida (Quick Start MVP)
## Sistema de Agentes para Automação Ansible

**Versão:** 1.0
**Data:** 23 de abril de 2026
**Tempo estimado de implementação:** 3 a 5 horas (uma tarde)
**Status:** Pronto para implementação

---

## 1. Sumário Executivo

Sistema pessoal mínimo viável para apoiar um profissional de automação Ansible que mantém múltiplos empregos simultaneamente. Entrega os dois agentes de maior alavancagem — **Playbook Author** e **Reviewer** — via interface web, com seleção manual de modelo Anthropic no dropdown.

**Filosofia:** construir o mínimo que já entrega valor real, usar por 30 dias, deixar o uso real direcionar a evolução.

**Escopo intencionalmente enxuto:** sem LangGraph, sem LLM local, sem serviço Python customizado. Tudo resolvido com Docker Compose + 2 arquivos de configuração.

---

## 2. Objetivos

### 2.1. Objetivos (em ordem de prioridade)

1. **Custo operacional mínimo** — orçamento-alvo: $20–50/mês
2. **Velocidade de implementação** — funcional em uma tarde (3–5h)
3. **Valor imediato** — cobrir 70-80% do trabalho repetitivo do JD
4. **Manutenção próxima de zero** — atualizações via `docker compose pull`

### 2.2. Critério de Sucesso

Após 30 dias de uso:
- Profissional não desativou o sistema
- Economizou tempo líquido considerando overhead mínimo
- Coletou dados suficientes para decidir sobre próxima fase (ou não)

---

## 3. Escopo

### 3.1. Inclui

- Interface web (Open WebUI) com seleção manual de agente via dropdown
- Proxy LLM (LiteLLM) com Anthropic API
- Dois agentes como "modelos virtuais":
  - **Playbook Author** (Sonnet 4.6) — gera playbooks Ansible idempotentes
  - **Reviewer** (Opus 4.7) — revisa playbooks/roles
- Histórico de conversas persistido localmente
- Budget/alertas de custo no LiteLLM
- Setup completo via Docker Compose

### 3.2. Explicitamente Fora do Escopo

- LLM local (Ollama)
- LangGraph ou orquestrador customizado
- Roteamento automático entre agentes
- Agentes adicionais (Migrator, Module Builder, Integrator, Doc Writer)
- Integrações com ferramentas externas (ServiceNow, Splunk, VMware)
- CI/CD próprio
- Biblioteca de templates Ansible
- RAG sobre repositórios

Tudo acima é candidato para a **Versão Completa** (ver PRD separado).

---

## 4. Arquitetura

### 4.1. Visão Geral

```
┌────────────────────┐
│   USUÁRIO          │
│   (navegador)      │
└─────────┬──────────┘
          │ HTTP
          ▼
┌────────────────────┐
│   Open WebUI       │  ← Interface de chat
│   (porta 3000)     │  ← Dropdown de agentes
└─────────┬──────────┘  ← Histórico local
          │ OpenAI API
          ▼
┌────────────────────┐
│   LiteLLM          │  ← Proxy unificado
│   (porta 4000)     │  ← Injeta system prompt
└─────────┬──────────┘  ← Budget tracking
          │ HTTPS
          ▼
┌────────────────────┐
│   Anthropic API    │  ← Claude Sonnet 4.6
│                    │  ← Claude Opus 4.7
└────────────────────┘
```

### 4.2. Por que essa arquitetura é suficiente

- **Open WebUI** tem dropdown de modelos nativo — cada agente aparece como uma opção ("📝 Playbook Author", "🔍 Reviewer")
- **LiteLLM** permite definir "modelos virtuais" com system prompt embutido — cada agente é só uma entrada YAML
- **Anthropic API** faz todo o trabalho pesado de raciocínio

Sem serviços customizados, sem build de imagens, sem código Python. Só YAML e Docker.

---

## 5. Requisitos Funcionais

### RF-01 — Seleção de Agente no Dropdown

O usuário seleciona no dropdown da interface qual agente (+ modelo) usar para a mensagem atual.

**Critérios de aceite:**
- Dropdown lista claramente: "📝 Playbook Author (Sonnet)" e "🔍 Reviewer (Opus)"
- System prompt correto é injetado automaticamente
- Troca de agente no meio da conversa é possível

### RF-02 — Agente Playbook Author

Gera playbooks Ansible idempotentes a partir de descrição em linguagem natural.

**Critérios de aceite:**
- Playbook em YAML válido, formatado corretamente
- Idempotente (módulos nativos sobre shell/command)
- Com handlers, tags, when conditions, no_log quando aplicável
- Comentário inicial explicando propósito
- Seção "Notas de uso" com pré-requisitos e exemplo de invocação

### RF-03 — Agente Reviewer

Analisa playbook/role e lista problemas categorizados.

**Critérios de aceite:**
- Saída em markdown estruturado
- Issues agrupados por severidade (🔴 crítica, 🟡 média, 🟢 sugestão)
- Cada issue cita localização, problema e sugestão concreta
- Seção final com pontos positivos

### RF-04 — Persistência de Conversas

Conversas salvas localmente.

**Critérios de aceite:**
- Histórico sobrevive a restart do container
- Busca por texto funciona
- Exportação para markdown disponível

### RF-05 — Controle de Custo

Budget mensal configurável com alertas.

**Critérios de aceite:**
- Alerta quando uso atinge 80% do limite
- Dashboard LiteLLM mostra custo acumulado do mês
- Custo por agente visível

---

## 6. Requisitos Não-Funcionais

| ID | Categoria | Requisito |
|---|---|---|
| RNF-01 | Performance | Resposta típica em < 30s (depende da API) |
| RNF-02 | Custo | ≤ $50/mês no orçamento-alvo |
| RNF-03 | Segurança | API key em variáveis de ambiente, nunca em código |
| RNF-04 | Setup | 100% via Docker Compose, reproduzível |
| RNF-05 | Manutenção | Atualizações via `docker compose pull` |

---

## 7. Cronograma de Implementação (3–5 horas)

| Etapa | Tempo | Atividade |
|---|---|---|
| 1 | 15 min | Criar pasta do projeto, arquivo `.env` com `ANTHROPIC_API_KEY` e `LITELLM_MASTER_KEY` |
| 2 | 20 min | Escrever `docker-compose.yml` e `litellm-config.yaml` com 1 modelo de teste |
| 3 | 10 min | `docker compose up -d`, acessar Open WebUI em localhost:3000, validar primeira chamada |
| 4 | **1–2h** | **Escrever e iterar os 2 system prompts** (a parte mais importante) |
| 5 | 15 min | Adicionar os agentes como modelos virtuais, reiniciar LiteLLM |
| 6 | 1–2h | Testar com 3–5 casos reais, ajustar prompts |
| 7 | 10 min | Configurar budget de $40/mês no LiteLLM |

**Total: 3h10 a 5h10**, conforme velocidade e nível de iteração nos prompts.

---

## 8. Artefatos a Produzir

1. `docker-compose.yml` — definição dos 2 containers
2. `litellm-config.yaml` — modelos virtuais + system prompts
3. `.env` — variáveis de ambiente (API keys, não comitar)
4. `.env.example` — template sem segredos
5. `.gitignore` — excluir `.env` e volumes
6. `README.md` — instruções de execução
7. `prompts/playbook_author.md` — system prompt em arquivo separado para versionar
8. `prompts/reviewer.md` — idem

---

## 9. Custos

### 9.1. Setup
- Tempo: 3–5h (trabalho próprio)
- Dinheiro: $0

### 9.2. Recorrente (mensal)

**Cenário típico (~60 demandas/mês):**

| Item | Custo |
|---|---|
| Anthropic API (Sonnet para geração, Opus para review) | $20–40 |
| Eletricidade adicional (desprezível) | ~$1 |
| **Total** | **$21–41** |

### 9.3. Otimizações Aplicáveis
- **Prompt caching:** system prompts cacheados automaticamente pelo LiteLLM → até 90% de desconto no input repetido
- **Budget hard limit:** corta abusos

---

## 10. Métricas de Sucesso (avaliar em 30 dias)

| Métrica | Meta |
|---|---|
| Tempo médio por playbook | < 15 min (baseline: 60+ min) |
| % de retrabalho na saída | < 30% |
| Custo mensal | < $50 |
| Frequência de uso | ≥ 5 demandas/semana |
| Tempo gasto em manutenção do sistema | < 30 min/mês |

---

## 11. Riscos

| Risco | Mitigação |
|---|---|
| Qualidade do prompt insuficiente | Iterar com 3–5 casos reais antes de "finalizar" |
| Custo API surpreender | Budget hard limit + alerta em 80% |
| Dados sensíveis vazarem para API | Política pessoal: anonimizar nomes de clientes/servidores antes de enviar |
| Sistema não usado suficientemente | Se após 2 semanas uso < 3 demandas/semana, repensar valor |

---

## 12. Próximos Passos (após 30 dias de uso)

Com dados reais em mãos, decidir:

1. **Manter só a versão reduzida?** Se cobre bem o trabalho atual, não há por que expandir.
2. **Adicionar componentes específicos?** Só o que provou falta no uso real. Exemplos:
   - Custo alto → adicionar Ollama para tarefas simples
   - Cliente exigiu privacidade → ativar Ollama obrigatório para aquele contexto
   - Migração de scripts virou frequente → adicionar agente Migrator
3. **Ir para Versão Completa?** Só se múltiplos sinais justificarem.

**Anti-padrão a evitar:** construir a versão completa "porque seria legal ter". O sistema é ferramenta pessoal, não produto.

---

## 13. Anexos

### Anexo A — Estrutura de Diretórios

```
ansible-agents-mvp/
├── docker-compose.yml
├── litellm-config.yaml
├── .env
├── .env.example
├── .gitignore
├── README.md
└── prompts/
    ├── playbook_author.md
    └── reviewer.md
```

### Anexo B — Decisões Pendentes (para tomar no dia da implementação)

1. Onde rodar o Docker: máquina pessoal ou VPS pequeno?
2. Nome do projeto no repositório Git (se for versionar)
3. Backup dos prompts: local apenas ou também em Git privado?

---

**Fim do documento.**
