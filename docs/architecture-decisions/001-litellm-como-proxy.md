# ADR 001 — Usar LiteLLM como Proxy Unificado

**Data:** 2026-04-23
**Status:** Aceito
**Decisores:** [Seu Nome]

---

## Contexto

O projeto precisa de uma camada intermediária entre a interface (Open WebUI) e os provedores de LLM (Anthropic API, futuramente Ollama) para:

- Unificar o acesso via API única (compatível com OpenAI)
- Permitir que cada "agente" apareça como modelo selecionável no dropdown
- Tracking de custos e limites de budget
- Injeção automática de system prompts sem lógica customizada
- Facilidade de trocar modelos sem mexer na interface

## Alternativas Consideradas

### A. LiteLLM

**Prós:**
- Proxy OpenAI-compatible maduro e amplamente usado
- Suporta dezenas de provedores (Anthropic, OpenAI, Ollama, Azure, etc.)
- Modelos virtuais com system prompt embutido (via `model_info`)
- Budget tracking nativo
- Deploy simples via Docker

**Contras:**
- Dependência externa a mais na stack
- Configuração via YAML pode ficar verbosa com muitos modelos

### B. Chamadas diretas da Open WebUI

**Prós:**
- Menos componentes na stack
- Open WebUI suporta Anthropic nativamente

**Contras:**
- Sem camada de abstração — trocar provedor exige reconfigurar interface
- Sem budget tracking unificado
- Impossível ter "agentes" como modelos virtuais com system prompts customizados
- Dificuldade de estender para multi-provider no futuro

### C. Camada própria em Python (FastAPI)

**Prós:**
- Controle total sobre a lógica
- Sem dependências de terceiros

**Contras:**
- Reinventar roda — construir o que LiteLLM já oferece
- Manutenção contínua
- Contra objetivo de velocidade de implementação

## Decisão

**Adotar LiteLLM como proxy unificado.**

O ganho de funcionalidades pronto-pra-uso (modelos virtuais, budget, multi-provider) supera amplamente a adição de um componente na stack. Decisão alinhada com a prioridade de velocidade de implementação do MVP.

## Consequências

### Positivas

- Cada agente pode ser configurado como entrada YAML com system prompt embutido, sem código
- Adição de modelos locais (Ollama) no futuro é trivial
- Budget tracking pronto-pra-uso
- Troca de provedor/modelo não afeta Open WebUI

### Negativas

- Uma dependência a mais no stack (mitigado por ser projeto maduro)
- Configuração em YAML cresce conforme adiciona agentes (mitigado por organização em seções)

### Mudanças Necessárias

- Docker Compose inclui serviço LiteLLM
- Open WebUI aponta para LiteLLM (não diretamente para Anthropic)
- Prompts ficam em arquivos separados em `prompts/`, referenciados no `litellm-config.yaml`

## Revisão Futura

Reavaliar esta decisão se:

- LiteLLM se tornar um gargalo de performance
- Aparecer necessidade de lógica complexa que LiteLLM não suporte (nesse caso, adicionar LangGraph como camada adicional, não substituir LiteLLM)
- O projeto perder manutenção ativa
