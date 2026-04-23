# ✍️ Prompt Engineering — Princípios do Projeto

**Última atualização:** 23 de abril de 2026

Este documento consolida os princípios adotados na escrita dos system prompts dos agentes deste projeto. Serve como referência para criar novos agentes e manter consistência.

---

## 1. Princípios Gerais

### 1.1. Especificidade > generalidade

Prompt genérico gera resposta genérica. Sempre incluir:
- Papel/função do agente (quem ele é)
- Escopo explícito (o que faz e o que NÃO faz)
- Formato exato da saída esperada
- Critérios de qualidade mensuráveis

### 1.2. Exemplos few-shot quando possível

Para tarefas com output estruturado, 1-2 exemplos de input → output ajudam o modelo a acertar formato e tom. Não precisa ser longo — pode ser um exemplo simplificado.

### 1.3. Restrições explícitas

Modelos tendem a ser verbosos. Se você quer concisão, peça. Se quer apenas código sem preamble, peça. Se há padrões a evitar, liste.

### 1.4. Estrutura com markdown

System prompts com `##` e listas numeradas funcionam melhor que parágrafos corridos. Facilitam para o modelo identificar seções e seguir instruções.

### 1.5. Critérios de clarificação

Definir quando o agente deve fazer perguntas antes de responder (input ambíguo, informação crítica faltando) vs. quando deve assumir padrões razoáveis.

---

## 2. Estrutura Padrão de System Prompt

Todos os agentes deste projeto seguem esta estrutura:

```markdown
# [Nome do Agente]

## Papel
[Quem é o agente, senioridade implícita, especialidade]

## Escopo
[O que faz]

## Fora de Escopo
[O que explicitamente NÃO faz — redireciona ou recusa]

## Regras de Qualidade
[Lista numerada de requisitos que a saída DEVE atender]

## Formato de Saída
[Estrutura exata esperada]

## Comportamento em Ambiguidade
[Quando perguntar, quando assumir]

## Exemplo
[Opcional: 1 exemplo curto de input → output ideal]
```

---

## 3. Regras Específicas para Agentes de Código

### 3.1. Sempre produzir código executável

Se o agente gera código, o output deve ser colável e funcional. Evitar pseudocódigo exceto quando explicitamente pedido.

### 3.2. Comentários explicando "por quê", não "o quê"

Código bem escrito é auto-explicativo no "o quê". Comentários devem capturar intenção e decisões não-óbvias.

### 3.3. Seguir convenções do ecossistema

- Ansible: `ansible.builtin.*`, FQCNs, idempotência, handlers, tags
- Python: PEP 8, type hints quando útil, docstrings
- YAML: 2 espaços, aspas quando necessário, consistência

### 3.4. Segurança por padrão

- Nunca hardcode de credenciais no output
- `no_log: true` em tasks com segredos
- Uso de variáveis e vaults para valores sensíveis

---

## 4. Regras Específicas para Agentes de Revisão

### 4.1. Severidade calibrada

Usar 3 níveis claros (crítica / alta-média / sugestão), não 5+ que viram subjetivos. Crítica = bloqueia merge; alta-média = deve corrigir; sugestão = opcional.

### 4.2. Sempre oferecer correção

Não basta apontar problema. Cada issue deve ter proposta concreta de como corrigir, idealmente com snippet.

### 4.3. Reconhecer o que está bem

Seção de pontos positivos não é bajulação — é calibração. Ajuda o autor a saber o que manter e replicar.

---

## 5. Anti-Padrões a Evitar

### 5.1. "Seja útil" / "Faça o melhor"

Instruções vagas geram saída medíocre. Sempre operacionalizar em critérios.

### 5.2. Listas de regras sem priorização

Se tudo é prioridade, nada é. Em conflito entre regras, o prompt deve indicar qual vence.

### 5.3. Tamanho excessivo

Prompt de 2000+ palavras raramente é melhor que um de 500 focado. Cada parágrafo adicional custa atenção do modelo.

### 5.4. Instruções contraditórias

"Seja conciso. Explique detalhadamente cada decisão. Use no máximo 100 palavras. Dê 3 exemplos." — não dá para cumprir tudo. Revisar prompts buscando contradições.

### 5.5. Assumir contexto não compartilhado

O modelo não sabe sua stack, padrões da empresa, preferências pessoais. Explicitar ou dar exemplos.

---

## 6. Iteração de Prompts

### 6.1. Ciclo de refinamento

1. Prompt v1 com base nos princípios acima
2. Testar com 3 casos reais diversos
3. Identificar onde a saída ficou aquém
4. Ajustar prompt (adicionar regra, exemplo, restrição)
5. Repetir até qualidade estável (geralmente 2-3 ciclos)

### 6.2. Versionamento

Cada prompt fica em arquivo dedicado em `prompts/`. Mudanças significativas commitadas com mensagem descritiva. Histórico Git é a trilha de evolução.

### 6.3. Testes de regressão informais

Manter em `examples/` 2-3 casos que representam "bom funcionamento". Após mudança no prompt, rodar esses casos e verificar se qualidade se manteve ou melhorou.

---

## 7. Recursos de Referência

- [Anthropic Prompt Engineering Guide](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Anthropic Prompt Library](https://docs.claude.com/en/prompt-library)
- Histórico de conversas do próprio Open WebUI — prompts que geraram bons resultados são fonte de padrões
