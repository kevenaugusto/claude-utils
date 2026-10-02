# Guia do Architecture Generator

> **Documento de referência** — não é invocável. Complementa a skill
> `skills/architecture-generator/SKILL.md`, que define o slash command.

---

## Descrição

Esta skill conduz um **brainstorming arquitetural estruturado e interativo** (uma dimensão por vez) para qualquer ideia de projeto de software, e produz um arquivo **`<projeto>-architecture.md`** completo seguindo o padrão do arquivo de referência (`docs/architecture/finance-app-architecture.md`).

---

## Como Funciona

1. **Você invoca** com a descrição do projeto
2. **O agente `architecture-generator`** é invocado via `agent()` tool
3. **O agente conduz** o brainstorming interativo (12 dimensões, uma por vez)
4. **Você decide** a cada dimensão (ou marca como TBD)
5. **Ao final** → resumo consolidado → confirmação → geração do arquivo `.md`
6. **Arquivo salvo** em `docs/architecture/<slug>-architecture.md`

---

## O que o Agente Faz (Resumo das 12 Dimensões)

| # | Dimensão | Pergunta-Chave |
|---|---|---|
| 1 | **Modelo da Aplicação** | "O usuário final interage como?" (Web/Desktop/Mobile/CLI/API/Híbrido) |
| 2 | **Arquitetura de Alto Nível** | "Como os componentes se organizam e comunicam?" (Monolito/Modular/Micro/Serverless/Event-driven) |
| 3 | **Linguagem / Backend** | "Qual stack backend?" (Python/FastAPI, Node/NestJS, Go, Rust, Java, .NET) |
| 4 | **Frontend** | "Qual stack frontend?" (React/Vue/Svelte + TS, Next/Remix/Astro, nativo) |
| 5 | **Banco de Dados** | "Onde os dados vivem?" (SQL/NoSQL/NewSQL/embedded, ORM) |
| 6 | **Infraestrutura / Deploy** | "Onde roda?" (Local-only, Docker, VPS, Cloud, Serverless, K8s) |
| 7 | **Auth / Autorização** | "Quem acessa o quê?" (Próprio/JWT, OAuth/OIDC, mTLS, RBAC/ABAC) |
| 8 | **Comunicação** | "Como serviços conversam?" (REST, GraphQL, gRPC, tRPC, WS, SSE, MQ) |
| 9 | **Observabilidade** | "Como observamos?" (Logs, métricas, tracing, alerting, nível de investimento) |
| 10 | **Qualidade** | "Como garantimos qualidade?" (Testes, lint, CI/CD, contract testing, ADRs) |
| 11 | **Segurança** | "Como protegemos?" (Threat model, secrets, rate limit, CORS, CSP, scanning) |
| 12 | **Dados Sensíveis / LGPD** | "Como tratamos dados pessoais?" (Criptografia, retenção, anonimização, DSR) |

---

## Exemplo de Uso Completo

```bash
# No terminal, dentro do projeto finance-app:
/architecture-generator "SaaS de agendamento para salões: multi-tenant, WhatsApp Business API, pagamentos Stripe, relatórios. Stack: Node + React. Deploy na AWS. Time: 3 devs, 4 meses."
```

**O que acontece:**
1. Agente inicia → "Entendido. Vou conduzir o brainstorming para **saas-agendamento-saloes**. Começamos pela **Dimensão 1: Modelo da Aplicação**."
2. Apresenta opções (Web SPA, Web SSR, Mobile, Desktop, Híbrido) com tabela de prós/contras
3. Você responde "A" ou "Web SPA" ou "TBD"
4. Avança para Dimensão 2... até 12
5. Mostra tabela consolidada → "Tudo correto? Posso gerar?"
6. Gera `docs/architecture/saas-agendamento-saloes-architecture.md` (700+ linhas, diagramas ASCII, tabelas completas)

---

## Arquivos Envolvidos

| Arquivo | Papel |
|---|---|
| `.claude/agents/architecture-generator.md` | **Agente especialista** — contém toda a lógica de brainstorming, templates, validações, geração do arquivo final |
| `.claude/skills/architecture-generator/SKILL.md` | **Este workflow** — define o slash command e o template de output (12 seções) |
| `.claude/references/architecture-generator-guide.md` | **Este arquivo** — guia de uso, exemplos, personalização e troubleshooting |

---

## Invocação Interna (como o skill executa)

Quando você roda `/architecture-generator "..."`, o sistema executa equivalente a:

```python
# Pseudo-código do que acontece internamente
agent_result = await agent(
    prompt=f'''
    Input do usuário: "{user_input}"
    
    Conduza o brainstorming arquitetural completo seguindo TODAS as instruções
    em .claude/agents/architecture-generator.md
    
    Inicie pela Dimensão 1 e interaja com o usuário dimensão a dimensão.
    Ao final, gere o arquivo completo e salve em docs/architecture/.
    ''',
    agent_type="architecture-generator",  # usa o agente definido em .claude/agents/
    schema=ARCHITECTURE_OUTPUT_SCHEMA  # opcional: para output estruturado
)
```

---

## Requisitos do Projeto

- Diretório `.claude/agents/` deve existir (já existe neste projeto)
- Diretório `docs/architecture/` será criado automaticamente se não existir
- O agente usa `Write` tool para salvar o arquivo final

---

## Personalização

Para adaptar o estilo do output, edite `.claude/agents/architecture-generator.md`:
- Seção **Output Final** → estrutura do markdown gerado
- Seção **Estilo de Comunicação** → tom de voz
- Seção **Habilidades Técnicas** → profundidade esperada
- Tabela de **Conflitos Conhecidos** → adicione regras do seu contexto

O template das 12 seções de output vive em `.claude/skills/architecture-generator/SKILL.md`, na seção **Output Final**.

---

## Exemplos Adicionais

```bash
# App mobile offline-first para coleta de dados em campo
/architecture-generator "App mobile offline-first para coleta de dados em campo: formulários dinâmicos, fotos, GPS, sync quando online. Stack: Flutter + Firebase. LGPD obrigatório. Time: 2 devs, 3 meses."

# CLI tool para migração de dados legacy
/architecture-generator "CLI tool para migração de dados legacy Oracle → PostgreSQL: validação de schema, transformação de dados, relatórios de discrepância, rollback. Stack: Python (Typer/Rich). Roda on-premise. Zero dependências cloud."

# API-only para integração B2B
/architecture-generator "API B2B para emissão de notas fiscais: multi-tenant, assinatura digital, webhook callbacks, rate limiting por plano. Stack: Go + PostgreSQL. Deploy Kubernetes on-prem. Compliance: NF-e, LGPD, SOC2."
```

---

## Princípios do Skill

| Princípio | Prática |
|---|---|
| **Uma dimensão por vez** | Não sobrecarregue. Avance só após decisão confirmada. |
| **Sugira com fundamentação** | "Recomendo X porque Y; alternativa viável: Z." |
| **Nunca decida pelo usuário** | Apresente opções claras; o usuário escolhe. |
| **Valide consistência** | Se escolhas conflitarem, aponte e negocie. |
| **Permita "pular por agora"** | Marque como `TBD` e retome no final. |
| **Revise tudo junto ao final** | Antes de gerar arquivo, mostre resumo consolidado para aprovação. |

---

## Troubleshooting

| Problema | Solução |
|----------|---------|
| Agente não encontrado | Verifique se `.claude/agents/architecture-generator.md` existe |
| Arquivo não salvo | Verifique permissão de escrita em `docs/architecture/` |
| Sessão muito longa | Use "TBD" para decisões menores; resolva no final |
| Conflito não detectado | Adicione na tabela de conflitos conhecidos do agente |
| Output incompleto | Verifique se agente completou todas as 12 dimensões antes de gerar |