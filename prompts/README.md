# 📝 Prompts dos Agentes

Os system prompts de cada agente vivem aqui, um arquivo por agente.

## Agentes Implementados

- `playbook_author.md` — Gera playbooks Ansible idempotentes (a criar no MVP)
- `reviewer.md` — Revisa playbooks/roles (a criar no MVP)

## Agentes Planejados (Versão Completa)

- `migrator.md` — Converte scripts em Ansible
- `module_builder.md` — Gera módulos Python custom
- `integrator.md` — Integrações com APIs externas
- `doc_writer.md` — Documentação derivada de código

## Princípios de Escrita

Ver [`../docs/prompt-engineering.md`](../docs/prompt-engineering.md).

## Como Referenciar no LiteLLM

Cada prompt é carregado no `litellm-config.yaml` via campo `system_prompt` da entrada `model_info`. Exemplo:

```yaml
- model_name: "📝 Playbook Author (Sonnet)"
  litellm_params:
    model: anthropic/claude-sonnet-4-6
    api_key: os.environ/ANTHROPIC_API_KEY
  model_info:
    system_prompt_file: /app/prompts/playbook_author.md
```
