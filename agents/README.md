# 🤖 Agents

Código Python dos agentes e orquestrador.

## Status Atual

**Vazio no MVP** — a Versão Reduzida não precisa de código Python customizado. Toda a lógica de agentes fica no `litellm-config.yaml` como modelos virtuais com system prompts embutidos.

## Planejado para Versão Completa

Quando o projeto evoluir para incluir LangGraph como orquestrador, esta pasta conterá:

```
agents/
├── __init__.py
├── main.py              # FastAPI expondo endpoint OpenAI-compatible
├── router.py            # Classificador de intenção + matriz de decisão
├── playbook_author.py   # Lógica do agente (se precisar mais que prompt)
├── reviewer.py
├── migrator.py
├── module_builder.py
├── integrator.py
└── doc_writer.py
```

## Decisão Arquitetural

Ver [`../docs/architecture-decisions/`](../docs/architecture-decisions/) para o raciocínio por trás de adotar (ou não) LangGraph em cada fase.
