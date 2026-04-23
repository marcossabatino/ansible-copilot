# 🤖 ansible-copilot

**Status:** 🟡 Em desenvolvimento (MVP)
**Tipo:** Sistema pessoal de agentes de IA
**Criado em:** Abril de 2026

Sistema pessoal de agentes de IA para acelerar o trabalho de automação Ansible. Combina modelos comerciais (Anthropic Claude) com modelos locais opcionais (Ollama), via interface web (Open WebUI) e proxy unificado (LiteLLM).

---

## 🎯 Propósito

Apoiar um profissional de automação Ansible que precisa entregar mais rápido e dedicar menos tempo, cobrindo tarefas frequentes e repetitivas do trabalho:

- Geração de playbooks idempotentes
- Revisão de código Ansible
- Migração de scripts (PowerShell/Bash → Ansible) — *fase futura*
- Criação de módulos Python custom — *fase futura*
- Integração com ferramentas externas (ServiceNow, Splunk, VMware) — *fase futura*
- Documentação derivada de código — *fase futura*

---

## 📦 Stack

| Componente | Função |
|---|---|
| [Open WebUI](https://github.com/open-webui/open-webui) | Interface de chat com dropdown de agentes |
| [LiteLLM](https://github.com/BerriAI/litellm) | Proxy unificado para múltiplos LLMs |
| [Anthropic Claude](https://www.anthropic.com/) | LLMs comerciais (Haiku/Sonnet/Opus) |
| [Ollama](https://ollama.com/) | LLMs locais (fase futura) |
| [LangGraph](https://langchain-ai.github.io/langgraph/) | Orquestrador (fase futura) |

---

## 🚀 Quick Start

> **Pré-requisitos:** Docker, Docker Compose, chave de API Anthropic.

```bash
# 1. Clonar o repositório
git clone git@github.com:SEU_USUARIO/ansible-copilot.git
cd ansible-copilot

# 2. Configurar variáveis de ambiente
cp .env.example .env
# Edite .env e adicione sua ANTHROPIC_API_KEY

# 3. Subir a stack
docker compose up -d

# 4. Acessar a interface
# Abra http://localhost:3000 no navegador
```

Detalhes completos em [`docs/PRD-reduzida.md`](docs/PRD-reduzida.md).

---

## 🗂️ Estrutura do Repositório

```
ansible-copilot/
├── docs/                  # PRDs, ADRs, políticas
├── prompts/               # System prompts versionados
├── agents/                # Código Python (orquestração futura)
├── scripts/               # Utilitários (setup, backup, export)
├── examples/              # Casos de uso reais anonimizados
├── docker-compose.yml     # Stack principal
└── litellm-config.yaml    # Configuração dos agentes
```

---

## 🤖 Agentes Disponíveis

### Versão Reduzida (MVP)

| Agente | Modelo | Propósito |
|---|---|---|
| 📝 Playbook Author | Claude Sonnet 4.6 | Gera playbooks Ansible idempotentes |
| 🔍 Reviewer | Claude Opus 4.7 | Revisa playbooks/roles e aponta problemas |

### Versão Completa (fase futura)

| Agente | Modelo | Propósito |
|---|---|---|
| 🔄 Migrator | Claude Sonnet 4.6 | Converte PowerShell/Bash em Ansible |
| 🐍 Module Builder | Claude Opus 4.7 | Cria módulos Python custom |
| 🔌 Integrator | Claude Sonnet 4.6 | Integrações com APIs externas |
| 📚 Doc Writer | Claude Haiku 4.5 | Documentação derivada de código |
| 🤖 Auto-Route | Orquestrador | Escolhe agente + modelo automaticamente |
| 🔒 Modo Privacidade | Local (Ollama) | Processamento 100% local |

---

## 🛡️ Políticas de Segurança

Antes de usar com dados de qualquer cliente/empregador, consulte [`docs/security-policies.md`](docs/security-policies.md). Regra de ouro: **dados sensíveis são anonimizados antes de qualquer envio a API externa**.

---

## 📊 Roadmap

- [x] Estrutura inicial do repositório
- [ ] MVP (Versão Reduzida): Open WebUI + LiteLLM + 2 agentes
- [ ] 30 dias de uso + coleta de dados
- [ ] Decisão sobre Versão Completa
- [ ] (se justificado) Adição de Ollama + modo privacidade
- [ ] (se justificado) Orquestrador com auto-route
- [ ] (se justificado) Agentes adicionais

---

## 📝 Licença

Proprietária. Uso pessoal apenas. Não distribuir.

---

## 🔗 Projetos Relacionados

Outros repositórios da mesma família (a criar conforme necessidade):

- `email-copilot` — futuro sistema para triagem e redação de email
- `research-copilot` — futuro sistema para pesquisa e síntese de conteúdo
- `pr-review-copilot` — futuro sistema para revisão de pull requests
