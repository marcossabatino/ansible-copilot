# 🛡️ Políticas de Segurança — ansible-copilot

**Última atualização:** 23 de abril de 2026
**Status:** Documento vivo — revisar mensalmente

---

## 1. Princípios Fundamentais

1. **Dados sensíveis nunca saem sem anonimização.** Nomes de clientes, hostnames, IPs, credenciais, tokens, CPFs/CNPJs, identificadores internos — tudo deve ser substituído por placeholders antes de enviar para API externa.
2. **Modo privacidade é obrigatório** quando contrato do cliente proibir processamento em infraestrutura externa.
3. **Credenciais vivem fora do código.** Sempre em variáveis de ambiente, cofre (Vault), ou gerenciador de segredos. Nunca em commits.
4. **Logs não devem preservar segredos.** Se um prompt contém dados sensíveis, não deve aparecer em logs persistidos.

---

## 2. O Que Pode Ser Enviado para API Anthropic

### ✅ Permitido

- Conceitos genéricos, patterns, lógica abstrata
- Exemplos sintéticos/fictícios (hostnames como `server01`, IPs como `10.0.0.1`)
- Código público ou que não seja do cliente (seu próprio, open source, exemplos de docs)
- Playbooks genéricos sem dados específicos da infraestrutura real
- Requisitos de alto nível em linguagem natural

### ❌ Proibido

- Nomes reais de clientes ou empresas
- Hostnames, IPs, FQDNs reais da infraestrutura do cliente
- Credenciais, tokens, API keys, senhas (mesmo que aparentemente "antigas")
- Dados pessoais (nomes, emails, CPFs, identificadores internos)
- Configurações específicas que revelem arquitetura interna crítica
- Dumps de banco, exports de CMDB, inventários reais
- Código sob NDA estrito de cliente

---

## 3. Técnicas de Anonimização

Antes de colar no prompt, substitua:

| Original | Placeholder |
|---|---|
| `srv-prod-app-01.cliente.com.br` | `server01.example.com` |
| `10.245.33.18` | `10.0.0.1` |
| `admin_db_cliente` | `admin_user` |
| `Banco Cliente S/A` | `ACME Corp` |
| `sk-live-xxxxxx` | `<REDACTED_TOKEN>` |
| Caminhos absolutos reais | Caminhos genéricos |

**Dica prática:** mantenha um script simples que faça substituições automáticas de termos frequentes. Exemplo em `scripts/anonymize.sh` (a criar).

---

## 4. Quando Usar Modo Local Obrigatoriamente

O modo local (Ollama, quando implementado) deve ser usado **obrigatoriamente** quando:

- Contrato do cliente explicitamente proíbe envio de código/dados a terceiros
- Trabalhando com dados classificados (ex: setor financeiro, governo, saúde)
- Dados que mesmo anonimizados manteriam padrões reconhecíveis
- Primeira exploração de algo que você não sabe ainda se pode compartilhar

Na dúvida: **local por padrão, API quando confirmadamente seguro**.

---

## 5. Gestão de Credenciais

### 5.1. Anthropic API Key

- Armazenar apenas em `.env` (que está no `.gitignore`)
- Nunca comitar, nunca enviar em mensagens, nunca printar em logs
- Rotacionar a cada 90 dias ou ao menor sinal de vazamento
- Usar chaves diferentes para desenvolvimento e uso real (se aplicável)

### 5.2. LiteLLM Master Key

- Gerar string aleatória forte (mínimo 32 caracteres)
- Usar: `openssl rand -base64 32`
- Trocar ao primeiro sinal de acesso indevido

### 5.3. Outras Credenciais (futuro)

Quando integrar com ServiceNow/Splunk/VMware:
- Usar sempre vault (HashiCorp Vault, Ansible Vault, ou similar)
- Nunca hardcode mesmo em arquivos locais "temporários"

---

## 6. Políticas por Contexto de Uso

| Contexto | Política |
|---|---|
| Aprendizado pessoal, testes, exemplos genéricos | API liberada |
| Trabalho rotineiro do Emprego A (cliente com NDA padrão) | API com anonimização obrigatória |
| Trabalho do Emprego B (cliente com NDA estrito) | Apenas local (modo privacidade) |
| Pesquisa de conceitos, leitura de docs públicas | API liberada |
| Dados pessoais próprios (ex: scripts de casa) | API liberada |

**Revisar esta tabela sempre que adicionar novo cliente/empregador.**

---

## 7. Auditoria e Logs

### 7.1. O Que Logar

- Timestamp de cada chamada
- Modelo usado
- Tokens consumidos (input/output)
- Custo estimado
- Agente invocado

### 7.2. O Que NÃO Logar

- Conteúdo completo do prompt (apenas hash ou primeiros 100 caracteres sanitizados)
- Conteúdo completo da resposta
- Qualquer identificador que possa vazar contexto sensível

### 7.3. Retenção

- Logs operacionais: 30 dias
- Conversas no Open WebUI: sob controle do usuário (limpar periodicamente)

---

## 8. Incidentes

Se suspeitar de vazamento ou exposição indevida:

1. Revogar imediatamente a API key Anthropic envolvida
2. Gerar nova key e atualizar `.env`
3. Revisar últimos logs para entender escopo
4. Documentar o incidente em `docs/incidents/` (criar pasta se necessário)
5. Ajustar políticas/processos se aplicável

---

## 9. Revisão

Este documento deve ser revisado:

- Mensalmente (revisão leve)
- Sempre que adicionar novo cliente/empregador
- Sempre que mudar a stack técnica
- Após qualquer incidente
