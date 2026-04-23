# 💡 Examples

Casos de uso reais **anonimizados** servindo como:

1. Testes de regressão informais — após ajustar um prompt, rodar os exemplos e verificar se qualidade se manteve
2. Documentação viva — mostra do que o sistema é capaz
3. Onboarding para você mesmo daqui a 3 meses

## Regras

- **Sempre anonimizar** antes de salvar aqui (ver [`../docs/security-policies.md`](../docs/security-policies.md))
- Nomear arquivos descritivamente: `<agente>-<cenário>.md`
- Incluir: prompt enviado, resposta recebida, avaliação breve da qualidade

## Estrutura Sugerida por Exemplo

```markdown
# [Título do caso]

**Agente:** [nome]
**Modelo:** [Claude Sonnet 4.6, etc.]
**Data:** YYYY-MM-DD
**Avaliação:** ⭐⭐⭐⭐⭐ (1-5)

## Prompt

[texto enviado]

## Resposta

[output recebido]

## Observações

[O que foi bom, o que ficou aquém, ajustes necessários no prompt]
```
