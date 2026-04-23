# 🔧 Scripts

Utilitários shell e Python para operação do projeto.

## Scripts Planejados

- `setup.sh` — Setup inicial: valida pré-requisitos, cria `.env` a partir do template, sobe a stack
- `anonymize.sh` — Aplica substituições automáticas em inputs antes de copiar para prompt
- `backup-prompts.sh` — Backup versionado dos prompts (garantia extra além do Git)
- `export-conversations.sh` — Exporta histórico do Open WebUI em markdown
- `cost-report.sh` — Resumo mensal de gastos via API do LiteLLM

## Convenções

- Scripts shell em Bash, começando com `#!/usr/bin/env bash` e `set -euo pipefail`
- Cada script com bloco de comentários no topo explicando propósito e uso
- Testar no macOS e Linux (evitar dependências GNU-only em scripts)
